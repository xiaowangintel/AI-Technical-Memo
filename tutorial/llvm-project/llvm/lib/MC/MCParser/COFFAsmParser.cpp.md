# COFFAsmParser.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/MC/MCParser/COFFAsmParser.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements COFF Assembly Parser.
  - **CN**: 实现 LLVM MC 汇编解析、伪指令处理、表达式解析以及面向 streamer 的解析辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24
```cpp
//===- COFFAsmParser.cpp - COFF Assembly Parser ---------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/ADT/StringRef.h"
#include "llvm/ADT/StringSwitch.h"
#include "llvm/ADT/Twine.h"
#include "llvm/BinaryFormat/COFF.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCDirectives.h"
#include "llvm/MC/MCParser/AsmLexer.h"
#include "llvm/MC/MCParser/MCAsmParserExtension.h"
#include "llvm/MC/MCSectionCOFF.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/Support/SMLoc.h"
#include "llvm/TargetParser/Triple.h"
#include <cassert>
#include <cstdint>
#include <limits>

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ADT/StringRef.h`, `llvm/ADT/StringSwitch.h`, `llvm/ADT/Twine.h`, `llvm/BinaryFormat/COFF.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ADT/StringRef.h`, `llvm/ADT/StringSwitch.h`, `llvm/ADT/Twine.h`, `llvm/BinaryFormat/COFF.h`。

### Lines 25-36
```cpp
using namespace llvm;

namespace {

class COFFAsmParser : public MCAsmParserExtension {
  template<bool (COFFAsmParser::*HandlerMethod)(StringRef, SMLoc)>
  void addDirectiveHandler(StringRef Directive) {
    MCAsmParser::ExtensionDirectiveHandler Handler = std::make_pair(
        this, HandleDirective<COFFAsmParser, HandlerMethod>);
    getParser().addDirectiveHandler(Directive, Handler);
  }

```
- **EN**: Introduces declarations for `llvm`, `COFFAsmParser`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `COFFAsmParser` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 37-49
```cpp
  bool parseSectionSwitch(StringRef Section, unsigned Characteristics);

  bool parseSectionSwitch(StringRef Section, unsigned Characteristics,
                          StringRef COMDATSymName, COFF::COMDATType Type,
                          unsigned UniqueID);

  bool parseSectionName(StringRef &SectionName);
  bool parseSectionFlags(StringRef SectionName, StringRef FlagsString,
                         unsigned *Flags);
  void Initialize(MCAsmParser &Parser) override {
    // Call the base implementation.
    MCAsmParserExtension::Initialize(Parser);

```
- **EN**: Implements logic around `parseSectionSwitch`, `parseSectionName`, `parseSectionFlags`, `Initialize`; this block parses assembly syntax or operands; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `parseSectionSwitch`, `parseSectionName`, `parseSectionFlags`, `Initialize` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 50-73
```cpp
    addDirectiveHandler<&COFFAsmParser::parseSectionDirectiveText>(".text");
    addDirectiveHandler<&COFFAsmParser::parseSectionDirectiveData>(".data");
    addDirectiveHandler<&COFFAsmParser::parseSectionDirectiveBSS>(".bss");
    addDirectiveHandler<&COFFAsmParser::parseDirectiveSection>(".section");
    addDirectiveHandler<&COFFAsmParser::parseDirectivePushSection>(
        ".pushsection");
    addDirectiveHandler<&COFFAsmParser::parseDirectivePopSection>(
        ".popsection");
    addDirectiveHandler<&COFFAsmParser::parseDirectiveDef>(".def");
    addDirectiveHandler<&COFFAsmParser::parseDirectiveScl>(".scl");
    addDirectiveHandler<&COFFAsmParser::parseDirectiveType>(".type");
    addDirectiveHandler<&COFFAsmParser::parseDirectiveEndef>(".endef");
    addDirectiveHandler<&COFFAsmParser::parseDirectiveSecRel32>(".secrel32");
    addDirectiveHandler<&COFFAsmParser::parseDirectiveSymIdx>(".symidx");
    addDirectiveHandler<&COFFAsmParser::parseDirectiveSafeSEH>(".safeseh");
    addDirectiveHandler<&COFFAsmParser::parseDirectiveSecIdx>(".secidx");
    addDirectiveHandler<&COFFAsmParser::parseDirectiveLinkOnce>(".linkonce");
    addDirectiveHandler<&COFFAsmParser::parseDirectiveRVA>(".rva");
    addDirectiveHandler<&COFFAsmParser::parseDirectiveSymbolAttribute>(".weak");
    addDirectiveHandler<&COFFAsmParser::parseDirectiveSymbolAttribute>(
        ".weak_anti_dep");
    addDirectiveHandler<&COFFAsmParser::parseDirectiveCGProfile>(".cg_profile");
    addDirectiveHandler<&COFFAsmParser::parseDirectiveSecNum>(".secnum");
    addDirectiveHandler<&COFFAsmParser::parseDirectiveSecOffset>(".secoffset");
```
- **EN**: Implements logic around `parseSectionDirectiveText>`, `parseSectionDirectiveData>`, `parseSectionDirectiveBSS>`, `parseDirectiveSection>`, and 16 more symbols; this block parses assembly syntax or operands; updates MC section or symbol state.
- **CN**: 围绕 `parseSectionDirectiveText>`, `parseSectionDirectiveData>`, `parseSectionDirectiveBSS>`, `parseDirectiveSection>`, and 16 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态。

### Lines 74-97
```cpp

    // Win64 EH directives.
    addDirectiveHandler<&COFFAsmParser::parseSEHDirectiveStartProc>(
        ".seh_proc");
    addDirectiveHandler<&COFFAsmParser::parseSEHDirectiveEndProc>(
        ".seh_endproc");
    addDirectiveHandler<&COFFAsmParser::parseSEHDirectiveEndFuncletOrFunc>(
        ".seh_endfunclet");
    addDirectiveHandler<&COFFAsmParser::parseSEHDirectiveSplitChained>(
        ".seh_splitchained");
    addDirectiveHandler<&COFFAsmParser::parseSEHDirectiveHandler>(
        ".seh_handler");
    addDirectiveHandler<&COFFAsmParser::parseSEHDirectiveHandlerData>(
        ".seh_handlerdata");
    addDirectiveHandler<&COFFAsmParser::parseSEHDirectiveAllocStack>(
        ".seh_stackalloc");
    addDirectiveHandler<&COFFAsmParser::parseSEHDirectiveEndProlog>(
        ".seh_endprologue");
    addDirectiveHandler<&COFFAsmParser::ParseSEHDirectiveBeginEpilog>(
        ".seh_startepilogue");
    addDirectiveHandler<&COFFAsmParser::ParseSEHDirectiveEndEpilog>(
        ".seh_endepilogue");
    addDirectiveHandler<&COFFAsmParser::ParseSEHDirectiveUnwindV2Start>(
        ".seh_unwindv2start");
```
- **EN**: Implements logic around `parseSEHDirectiveStartProc>`, `parseSEHDirectiveEndProc>`, `parseSEHDirectiveEndFuncletOrFunc>`, `parseSEHDirectiveSplitChained>`, and 7 more symbols; this block parses assembly syntax or operands.
- **CN**: 围绕 `parseSEHDirectiveStartProc>`, `parseSEHDirectiveEndProc>`, `parseSEHDirectiveEndFuncletOrFunc>`, `parseSEHDirectiveSplitChained>`, and 7 more symbols 实现具体逻辑；这一段解析汇编语法或操作数。

### Lines 98-113
```cpp
    addDirectiveHandler<&COFFAsmParser::ParseSEHDirectiveUnwindVersion>(
        ".seh_unwindversion");
  }

  bool parseSectionDirectiveText(StringRef, SMLoc) {
    return parseSectionSwitch(".text", COFF::IMAGE_SCN_CNT_CODE |
                                           COFF::IMAGE_SCN_MEM_EXECUTE |
                                           COFF::IMAGE_SCN_MEM_READ);
  }

  bool parseSectionDirectiveData(StringRef, SMLoc) {
    return parseSectionSwitch(".data", COFF::IMAGE_SCN_CNT_INITIALIZED_DATA |
                                           COFF::IMAGE_SCN_MEM_READ |
                                           COFF::IMAGE_SCN_MEM_WRITE);
  }

```
- **EN**: Implements logic around `ParseSEHDirectiveUnwindVersion>`, `parseSectionDirectiveText`, `parseSectionSwitch`, `parseSectionDirectiveData`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `ParseSEHDirectiveUnwindVersion>`, `parseSectionDirectiveText`, `parseSectionSwitch`, `parseSectionDirectiveData` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 114-137
```cpp
  bool parseSectionDirectiveBSS(StringRef, SMLoc) {
    return parseSectionSwitch(".bss", COFF::IMAGE_SCN_CNT_UNINITIALIZED_DATA |
                                          COFF::IMAGE_SCN_MEM_READ |
                                          COFF::IMAGE_SCN_MEM_WRITE);
  }

  bool parseDirectiveSection(StringRef, SMLoc);
  bool parseSectionArguments(StringRef, SMLoc);
  bool parseDirectivePushSection(StringRef, SMLoc);
  bool parseDirectivePopSection(StringRef, SMLoc);
  bool parseDirectiveDef(StringRef, SMLoc);
  bool parseDirectiveScl(StringRef, SMLoc);
  bool parseDirectiveType(StringRef, SMLoc);
  bool parseDirectiveEndef(StringRef, SMLoc);
  bool parseDirectiveSecRel32(StringRef, SMLoc);
  bool parseDirectiveSecIdx(StringRef, SMLoc);
  bool parseDirectiveSafeSEH(StringRef, SMLoc);
  bool parseDirectiveSymIdx(StringRef, SMLoc);
  bool parseCOMDATType(COFF::COMDATType &Type);
  bool parseDirectiveLinkOnce(StringRef, SMLoc);
  bool parseDirectiveRVA(StringRef, SMLoc);
  bool parseDirectiveCGProfile(StringRef, SMLoc);
  bool parseDirectiveSecNum(StringRef, SMLoc);
  bool parseDirectiveSecOffset(StringRef, SMLoc);
```
- **EN**: Implements logic around `parseSectionDirectiveBSS`, `parseSectionSwitch`, `parseDirectiveSection`, `parseSectionArguments`, and 16 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `parseSectionDirectiveBSS`, `parseSectionSwitch`, `parseDirectiveSection`, `parseSectionArguments`, and 16 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 138-152
```cpp

  // Win64 EH directives.
  bool parseSEHDirectiveStartProc(StringRef, SMLoc);
  bool parseSEHDirectiveEndProc(StringRef, SMLoc);
  bool parseSEHDirectiveEndFuncletOrFunc(StringRef, SMLoc);
  bool parseSEHDirectiveSplitChained(StringRef, SMLoc);
  bool parseSEHDirectiveHandler(StringRef, SMLoc);
  bool parseSEHDirectiveHandlerData(StringRef, SMLoc);
  bool parseSEHDirectiveAllocStack(StringRef, SMLoc);
  bool parseSEHDirectiveEndProlog(StringRef, SMLoc);
  bool ParseSEHDirectiveBeginEpilog(StringRef, SMLoc);
  bool ParseSEHDirectiveEndEpilog(StringRef, SMLoc);
  bool ParseSEHDirectiveUnwindV2Start(StringRef, SMLoc);
  bool ParseSEHDirectiveUnwindVersion(StringRef, SMLoc);

```
- **EN**: Implements logic around `parseSEHDirectiveStartProc`, `parseSEHDirectiveEndProc`, `parseSEHDirectiveEndFuncletOrFunc`, `parseSEHDirectiveSplitChained`, and 8 more symbols; this block parses assembly syntax or operands.
- **CN**: 围绕 `parseSEHDirectiveStartProc`, `parseSEHDirectiveEndProc`, `parseSEHDirectiveEndFuncletOrFunc`, `parseSEHDirectiveSplitChained`, and 8 more symbols 实现具体逻辑；这一段解析汇编语法或操作数。

### Lines 153-176
```cpp
  bool parseAtUnwindOrAtExcept(bool &unwind, bool &except);
  bool parseDirectiveSymbolAttribute(StringRef Directive, SMLoc);

public:
  COFFAsmParser() = default;
};

} // end anonymous namespace.

bool COFFAsmParser::parseSectionFlags(StringRef SectionName,
                                      StringRef FlagsString, unsigned *Flags) {
  enum {
    None = 0,
    Alloc = 1 << 0,
    Code = 1 << 1,
    Load = 1 << 2,
    InitData = 1 << 3,
    Shared = 1 << 4,
    NoLoad = 1 << 5,
    NoRead = 1 << 6,
    NoWrite = 1 << 7,
    Discardable = 1 << 8,
    Info = 1 << 9,
  };
```
- **EN**: Implements logic around `parseAtUnwindOrAtExcept`, `parseDirectiveSymbolAttribute`, `COFFAsmParser`, `parseSectionFlags`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `parseAtUnwindOrAtExcept`, `parseDirectiveSymbolAttribute`, `COFFAsmParser`, `parseSectionFlags` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 177-193
```cpp

  bool ReadOnlyRemoved = false;
  unsigned SecFlags = None;

  for (char FlagChar : FlagsString) {
    switch (FlagChar) {
    case 'a':
      // Ignored.
      break;

    case 'b': // bss section
      SecFlags |= Alloc;
      if (SecFlags & InitData)
        return TokError("conflicting section flags 'b' and 'd'.");
      SecFlags &= ~Load;
      break;

```
- **EN**: Implements logic around `TokError`; this block parses assembly syntax or operands; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `TokError` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 194-207
```cpp
    case 'd': // data section
      SecFlags |= InitData;
      if (SecFlags & Alloc)
        return TokError("conflicting section flags 'b' and 'd'.");
      SecFlags &= ~NoWrite;
      if ((SecFlags & NoLoad) == 0)
        SecFlags |= Load;
      break;

    case 'n': // section is not loaded
      SecFlags |= NoLoad;
      SecFlags &= ~Load;
      break;

```
- **EN**: Implements logic around `TokError`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `TokError` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 208-220
```cpp
    case 'D': // discardable
      SecFlags |= Discardable;
      break;

    case 'r': // read-only
      ReadOnlyRemoved = false;
      SecFlags |= NoWrite;
      if ((SecFlags & Code) == 0)
        SecFlags |= InitData;
      if ((SecFlags & NoLoad) == 0)
        SecFlags |= Load;
      break;

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 221-232
```cpp
    case 's': // shared section
      SecFlags |= Shared | InitData;
      SecFlags &= ~NoWrite;
      if ((SecFlags & NoLoad) == 0)
        SecFlags |= Load;
      break;

    case 'w': // writable
      SecFlags &= ~NoWrite;
      ReadOnlyRemoved = true;
      break;

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 233-244
```cpp
    case 'x': // executable section
      SecFlags |= Code;
      if ((SecFlags & NoLoad) == 0)
        SecFlags |= Load;
      if (!ReadOnlyRemoved)
        SecFlags |= NoWrite;
      break;

    case 'y': // not readable
      SecFlags |= NoRead | NoWrite;
      break;

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 245-258
```cpp
    case 'i': // info
      SecFlags |= Info;
      break;

    default:
      return TokError("unknown flag");
    }
  }

  *Flags = 0;

  if (SecFlags == None)
    SecFlags = InitData;

```
- **EN**: Implements logic around `TokError`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `TokError` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 259-278
```cpp
  if (SecFlags & Code)
    *Flags |= COFF::IMAGE_SCN_CNT_CODE | COFF::IMAGE_SCN_MEM_EXECUTE;
  if (SecFlags & InitData)
    *Flags |= COFF::IMAGE_SCN_CNT_INITIALIZED_DATA;
  if ((SecFlags & Alloc) && (SecFlags & Load) == 0)
    *Flags |= COFF::IMAGE_SCN_CNT_UNINITIALIZED_DATA;
  if (SecFlags & NoLoad)
    *Flags |= COFF::IMAGE_SCN_LNK_REMOVE;
  if ((SecFlags & Discardable) ||
      MCSectionCOFF::isImplicitlyDiscardable(SectionName))
    *Flags |= COFF::IMAGE_SCN_MEM_DISCARDABLE;
  if ((SecFlags & NoRead) == 0)
    *Flags |= COFF::IMAGE_SCN_MEM_READ;
  if ((SecFlags & NoWrite) == 0)
    *Flags |= COFF::IMAGE_SCN_MEM_WRITE;
  if (SecFlags & Shared)
    *Flags |= COFF::IMAGE_SCN_MEM_SHARED;
  if (SecFlags & Info)
    *Flags |= COFF::IMAGE_SCN_LNK_INFO;

```
- **EN**: Implements logic around `isImplicitlyDiscardable`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `isImplicitlyDiscardable` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 279-293
```cpp
  return false;
}

/// ParseDirectiveSymbolAttribute
///  ::= { ".weak", ... } [ identifier ( , identifier )* ]
bool COFFAsmParser::parseDirectiveSymbolAttribute(StringRef Directive, SMLoc) {
  MCSymbolAttr Attr = StringSwitch<MCSymbolAttr>(Directive)
    .Case(".weak", MCSA_Weak)
    .Case(".weak_anti_dep", MCSA_WeakAntiDep)
    .Default(MCSA_Invalid);
  assert(Attr != MCSA_Invalid && "unexpected symbol attribute directive!");
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    while (true) {
      MCSymbol *Sym;

```
- **EN**: Implements logic around `parseDirectiveSymbolAttribute`, `StringSwitch<MCSymbolAttr>`, `Case`, `Default`, and 1 more symbols; this block parses assembly syntax or operands; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveSymbolAttribute`, `StringSwitch<MCSymbolAttr>`, `Case`, `Default`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 294-307
```cpp
      if (getParser().parseSymbol(Sym))
        return TokError("expected identifier in directive");

      getStreamer().emitSymbolAttribute(Sym, Attr);

      if (getLexer().is(AsmToken::EndOfStatement))
        break;

      if (getLexer().isNot(AsmToken::Comma))
        return TokError("unexpected token in directive");
      Lex();
    }
  }

```
- **EN**: Implements logic around `TokError`, `getStreamer`, `Lex`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `TokError`, `getStreamer`, `Lex` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 308-321
```cpp
  Lex();
  return false;
}

bool COFFAsmParser::parseDirectiveCGProfile(StringRef S, SMLoc Loc) {
  return MCAsmParserExtension::parseDirectiveCGProfile(S, Loc);
}

bool COFFAsmParser::parseSectionSwitch(StringRef Section,
                                       unsigned Characteristics) {
  return parseSectionSwitch(Section, Characteristics, "", (COFF::COMDATType)0,
                            MCSection::NonUniqueID);
}

```
- **EN**: Implements logic around `Lex`, `parseDirectiveCGProfile`, `parseSectionSwitch`; this block parses assembly syntax or operands; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `Lex`, `parseDirectiveCGProfile`, `parseSectionSwitch` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 322-333
```cpp
bool COFFAsmParser::parseSectionSwitch(StringRef Section,
                                       unsigned Characteristics,
                                       StringRef COMDATSymName,
                                       COFF::COMDATType Type,
                                       unsigned UniqueID) {
  if (getLexer().isNot(AsmToken::EndOfStatement))
    return TokError("unexpected token in section switching directive");
  Lex();

  getStreamer().switchSection(getContext().getCOFFSection(
      Section, Characteristics, COMDATSymName, Type, UniqueID));

```
- **EN**: Implements logic around `parseSectionSwitch`, `TokError`, `Lex`, `getStreamer`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `parseSectionSwitch`, `TokError`, `Lex`, `getStreamer` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 334-345
```cpp
  return false;
}

bool COFFAsmParser::parseSectionName(StringRef &SectionName) {
  if (!getLexer().is(AsmToken::Identifier) && !getLexer().is(AsmToken::String))
    return true;

  SectionName = getTok().getIdentifier();
  Lex();
  return false;
}

```
- **EN**: Implements logic around `parseSectionName`, `getTok`, `Lex`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseSectionName`, `getTok`, `Lex` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 346-368
```cpp
bool COFFAsmParser::parseDirectiveSection(StringRef directive, SMLoc loc) {
  return parseSectionArguments(directive, loc);
}

// .section name [, "flags"] [, identifier [ identifier ], identifier]
// .pushsection <same as above>
//
// Supported flags:
//   a: Ignored.
//   b: BSS section (uninitialized data)
//   d: data section (initialized data)
//   n: "noload" section (removed by linker)
//   D: Discardable section
//   r: Readable section
//   s: Shared section
//   w: Writable section
//   x: Executable section
//   y: Not-readable section (clears 'r')
//
// Subsections are not supported.
bool COFFAsmParser::parseSectionArguments(StringRef, SMLoc) {
  StringRef SectionName;

```
- **EN**: Implements logic around `parseDirectiveSection`, `parseSectionArguments`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveSection`, `parseSectionArguments` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 369-381
```cpp
  if (parseSectionName(SectionName))
    return TokError("expected identifier in directive");

  unsigned Flags = COFF::IMAGE_SCN_CNT_INITIALIZED_DATA |
                   COFF::IMAGE_SCN_MEM_READ |
                   COFF::IMAGE_SCN_MEM_WRITE;

  if (getLexer().is(AsmToken::Comma)) {
    Lex();

    if (getLexer().isNot(AsmToken::String))
      return TokError("expected string in directive");

```
- **EN**: Implements logic around `TokError`, `Lex`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `TokError`, `Lex` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 382-395
```cpp
    StringRef FlagsStr = getTok().getStringContents();
    Lex();

    if (parseSectionFlags(SectionName, FlagsStr, &Flags))
      return true;
  }

  COFF::COMDATType Type = (COFF::COMDATType)0;
  StringRef COMDATSymName;
  if (getLexer().is(AsmToken::Comma) &&
      getLexer().peekTok().getString() != "unique") {
    Type = COFF::IMAGE_COMDAT_SELECT_ANY;
    Lex();

```
- **EN**: Implements logic around `getTok`, `Lex`, `getLexer`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getTok`, `Lex`, `getLexer` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 396-408
```cpp
    Flags |= COFF::IMAGE_SCN_LNK_COMDAT;

    if (!getLexer().is(AsmToken::Identifier))
      return TokError("expected comdat type such as 'discard' or 'largest' "
                      "after protection bits");

    if (parseCOMDATType(Type))
      return true;

    if (getLexer().isNot(AsmToken::Comma))
      return TokError("expected comma in directive");
    Lex();

```
- **EN**: Implements logic around `TokError`, `Lex`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `TokError`, `Lex` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 409-428
```cpp
    if (getParser().parseIdentifier(COMDATSymName))
      return TokError("expected identifier in directive");
  }

  int64_t UniqueID = MCSection::NonUniqueID;
  if (maybeParseUniqueID(UniqueID))
    return true;

  if (getLexer().isNot(AsmToken::EndOfStatement))
    return TokError("unexpected token in directive");

  if (Flags & COFF::IMAGE_SCN_CNT_CODE) {
    const Triple &T = getContext().getTargetTriple();
    if (T.getArch() == Triple::arm || T.getArch() == Triple::thumb)
      Flags |= COFF::IMAGE_SCN_MEM_16BIT;
  }
  parseSectionSwitch(SectionName, Flags, COMDATSymName, Type, UniqueID);
  return false;
}

```
- **EN**: Implements logic around `TokError`, `getContext`, `parseSectionSwitch`; this block parses assembly syntax or operands; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `TokError`, `getContext`, `parseSectionSwitch` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 429-445
```cpp
bool COFFAsmParser::parseDirectivePushSection(StringRef directive, SMLoc loc) {
  getStreamer().pushSection();

  if (parseSectionArguments(directive, loc)) {
    getStreamer().popSection();
    return true;
  }

  return false;
}

bool COFFAsmParser::parseDirectivePopSection(StringRef, SMLoc) {
  if (!getStreamer().popSection())
    return TokError(".popsection without corresponding .pushsection");
  return false;
}

```
- **EN**: Implements logic around `parseDirectivePushSection`, `getStreamer`, `parseDirectivePopSection`, `TokError`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectivePushSection`, `getStreamer`, `parseDirectivePopSection`, `TokError` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 446-457
```cpp
bool COFFAsmParser::parseDirectiveDef(StringRef, SMLoc) {
  MCSymbol *Sym;

  if (getParser().parseSymbol(Sym))
    return TokError("expected identifier in directive");

  getStreamer().beginCOFFSymbolDef(Sym);

  Lex();
  return false;
}

```
- **EN**: Implements logic around `parseDirectiveDef`, `TokError`, `getStreamer`, `Lex`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveDef`, `TokError`, `getStreamer`, `Lex` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 458-470
```cpp
bool COFFAsmParser::parseDirectiveScl(StringRef, SMLoc) {
  int64_t SymbolStorageClass;
  if (getParser().parseAbsoluteExpression(SymbolStorageClass))
    return true;

  if (getLexer().isNot(AsmToken::EndOfStatement))
    return TokError("unexpected token in directive");

  Lex();
  getStreamer().emitCOFFSymbolStorageClass(SymbolStorageClass);
  return false;
}

```
- **EN**: Implements logic around `parseDirectiveScl`, `TokError`, `Lex`, `getStreamer`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveScl`, `TokError`, `Lex`, `getStreamer` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 471-483
```cpp
bool COFFAsmParser::parseDirectiveType(StringRef, SMLoc) {
  int64_t Type;
  if (getParser().parseAbsoluteExpression(Type))
    return true;

  if (getLexer().isNot(AsmToken::EndOfStatement))
    return TokError("unexpected token in directive");

  Lex();
  getStreamer().emitCOFFSymbolType(Type);
  return false;
}

```
- **EN**: Implements logic around `parseDirectiveType`, `TokError`, `Lex`, `getStreamer`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveType`, `TokError`, `Lex`, `getStreamer` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 484-502
```cpp
bool COFFAsmParser::parseDirectiveEndef(StringRef, SMLoc) {
  Lex();
  getStreamer().endCOFFSymbolDef();
  return false;
}

bool COFFAsmParser::parseDirectiveSecRel32(StringRef, SMLoc) {
  MCSymbol *Symbol;
  if (getParser().parseSymbol(Symbol))
    return TokError("expected identifier in directive");

  int64_t Offset = 0;
  SMLoc OffsetLoc;
  if (getLexer().is(AsmToken::Plus)) {
    OffsetLoc = getLexer().getLoc();
    if (getParser().parseAbsoluteExpression(Offset))
      return true;
  }

```
- **EN**: Implements logic around `parseDirectiveEndef`, `Lex`, `getStreamer`, `parseDirectiveSecRel32`, and 2 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveEndef`, `Lex`, `getStreamer`, `parseDirectiveSecRel32`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 503-516
```cpp
  if (getLexer().isNot(AsmToken::EndOfStatement))
    return TokError("unexpected token in directive");

  if (Offset < 0 || Offset > std::numeric_limits<uint32_t>::max())
    return Error(
        OffsetLoc,
        "invalid '.secrel32' directive offset, can't be less "
        "than zero or greater than std::numeric_limits<uint32_t>::max()");

  Lex();
  getStreamer().emitCOFFSecRel32(Symbol, Offset);
  return false;
}

```
- **EN**: Implements logic around `TokError`, `Error`, `max`, `Lex`, and 1 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `TokError`, `Error`, `max`, `Lex`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 517-530
```cpp
bool COFFAsmParser::parseDirectiveRVA(StringRef, SMLoc) {
  auto parseOp = [&]() -> bool {
    MCSymbol *Symbol;
    if (getParser().parseSymbol(Symbol))
      return TokError("expected identifier in directive");

    int64_t Offset = 0;
    SMLoc OffsetLoc;
    if (getLexer().is(AsmToken::Plus) || getLexer().is(AsmToken::Minus)) {
      OffsetLoc = getLexer().getLoc();
      if (getParser().parseAbsoluteExpression(Offset))
        return true;
    }

```
- **EN**: Implements logic around `parseDirectiveRVA`, `TokError`, `getLexer`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveRVA`, `TokError`, `getLexer` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 531-545
```cpp
    if (Offset < std::numeric_limits<int32_t>::min() ||
        Offset > std::numeric_limits<int32_t>::max())
      return Error(OffsetLoc, "invalid '.rva' directive offset, can't be less "
                              "than -2147483648 or greater than "
                              "2147483647");

    getStreamer().emitCOFFImgRel32(Symbol, Offset);
    return false;
  };

  if (getParser().parseMany(parseOp))
    return addErrorSuffix(" in directive");
  return false;
}

```
- **EN**: Implements logic around `max`, `Error`, `getStreamer`, `addErrorSuffix`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `max`, `Error`, `getStreamer`, `addErrorSuffix` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 546-558
```cpp
bool COFFAsmParser::parseDirectiveSafeSEH(StringRef, SMLoc) {
  MCSymbol *Symbol;
  if (getParser().parseSymbol(Symbol))
    return TokError("expected identifier in directive");

  if (getLexer().isNot(AsmToken::EndOfStatement))
    return TokError("unexpected token in directive");

  Lex();
  getStreamer().emitCOFFSafeSEH(Symbol);
  return false;
}

```
- **EN**: Implements logic around `parseDirectiveSafeSEH`, `TokError`, `Lex`, `getStreamer`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveSafeSEH`, `TokError`, `Lex`, `getStreamer` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 559-571
```cpp
bool COFFAsmParser::parseDirectiveSecIdx(StringRef, SMLoc) {
  MCSymbol *Symbol;
  if (getParser().parseSymbol(Symbol))
    return TokError("expected identifier in directive");

  if (getLexer().isNot(AsmToken::EndOfStatement))
    return TokError("unexpected token in directive");

  Lex();
  getStreamer().emitCOFFSectionIndex(Symbol);
  return false;
}

```
- **EN**: Implements logic around `parseDirectiveSecIdx`, `TokError`, `Lex`, `getStreamer`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveSecIdx`, `TokError`, `Lex`, `getStreamer` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 572-584
```cpp
bool COFFAsmParser::parseDirectiveSymIdx(StringRef, SMLoc) {
  MCSymbol *Symbol;
  if (getParser().parseSymbol(Symbol))
    return TokError("expected identifier in directive");

  if (getLexer().isNot(AsmToken::EndOfStatement))
    return TokError("unexpected token in directive");

  Lex();
  getStreamer().emitCOFFSymbolIndex(Symbol);
  return false;
}

```
- **EN**: Implements logic around `parseDirectiveSymIdx`, `TokError`, `Lex`, `getStreamer`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveSymIdx`, `TokError`, `Lex`, `getStreamer` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 585-597
```cpp
bool COFFAsmParser::parseDirectiveSecNum(StringRef, SMLoc) {
  MCSymbol *Symbol;
  if (getParser().parseSymbol(Symbol))
    return TokError("expected identifier in directive");

  if (getLexer().isNot(AsmToken::EndOfStatement))
    return TokError("unexpected token in directive");

  Lex();
  getStreamer().emitCOFFSecNumber(Symbol);
  return false;
}

```
- **EN**: Implements logic around `parseDirectiveSecNum`, `TokError`, `Lex`, `getStreamer`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveSecNum`, `TokError`, `Lex`, `getStreamer` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 598-610
```cpp
bool COFFAsmParser::parseDirectiveSecOffset(StringRef, SMLoc) {
  MCSymbol *Symbol;
  if (getParser().parseSymbol(Symbol))
    return TokError("expected identifier in directive");

  if (getLexer().isNot(AsmToken::EndOfStatement))
    return TokError("unexpected token in directive");

  Lex();
  getStreamer().emitCOFFSecOffset(Symbol);
  return false;
}

```
- **EN**: Implements logic around `parseDirectiveSecOffset`, `TokError`, `Lex`, `getStreamer`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveSecOffset`, `TokError`, `Lex`, `getStreamer` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 611-624
```cpp
/// ::= [ identifier ]
bool COFFAsmParser::parseCOMDATType(COFF::COMDATType &Type) {
  StringRef TypeId = getTok().getIdentifier();

  Type = StringSwitch<COFF::COMDATType>(TypeId)
    .Case("one_only", COFF::IMAGE_COMDAT_SELECT_NODUPLICATES)
    .Case("discard", COFF::IMAGE_COMDAT_SELECT_ANY)
    .Case("same_size", COFF::IMAGE_COMDAT_SELECT_SAME_SIZE)
    .Case("same_contents", COFF::IMAGE_COMDAT_SELECT_EXACT_MATCH)
    .Case("associative", COFF::IMAGE_COMDAT_SELECT_ASSOCIATIVE)
    .Case("largest", COFF::IMAGE_COMDAT_SELECT_LARGEST)
    .Case("newest", COFF::IMAGE_COMDAT_SELECT_NEWEST)
    .Default((COFF::COMDATType)0);

```
- **EN**: Implements logic around `parseCOMDATType`, `getTok`, `COMDATType>`, `Case`, and 1 more symbols; this block parses assembly syntax or operands; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `parseCOMDATType`, `getTok`, `COMDATType>`, `Case`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 625-640
```cpp
  if (Type == 0)
    return TokError(Twine("unrecognized COMDAT type '" + TypeId + "'"));

  Lex();

  return false;
}

/// ParseDirectiveLinkOnce
///  ::= .linkonce [ identifier ]
bool COFFAsmParser::parseDirectiveLinkOnce(StringRef, SMLoc Loc) {
  COFF::COMDATType Type = COFF::IMAGE_COMDAT_SELECT_ANY;
  if (getLexer().is(AsmToken::Identifier))
    if (parseCOMDATType(Type))
      return true;

```
- **EN**: Implements logic around `TokError`, `Lex`, `parseDirectiveLinkOnce`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `TokError`, `Lex`, `parseDirectiveLinkOnce` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 641-652
```cpp
  const MCSectionCOFF *Current =
      static_cast<const MCSectionCOFF *>(getStreamer().getCurrentSectionOnly());

  if (Type == COFF::IMAGE_COMDAT_SELECT_ASSOCIATIVE)
    return Error(Loc, "cannot make section associative with .linkonce");

  if (Current->getCharacteristics() & COFF::IMAGE_SCN_LNK_COMDAT)
    return Error(Loc, Twine("section '") + Current->getName() +
                          "' is already linkonce");

  Current->setSelection(Type);

```
- **EN**: Implements logic around `getStreamer`, `Error`, `setSelection`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getStreamer`, `Error`, `setSelection` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 653-666
```cpp
  if (getLexer().isNot(AsmToken::EndOfStatement))
    return TokError("unexpected token in directive");

  return false;
}

bool COFFAsmParser::parseSEHDirectiveStartProc(StringRef, SMLoc Loc) {
  MCSymbol *Symbol;
  if (getParser().parseSymbol(Symbol))
    return true;

  if (getLexer().isNot(AsmToken::EndOfStatement))
    return TokError("unexpected token in directive");

```
- **EN**: Implements logic around `TokError`, `parseSEHDirectiveStartProc`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `TokError`, `parseSEHDirectiveStartProc` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 667-683
```cpp
  Lex();
  getStreamer().emitWinCFIStartProc(Symbol, Loc);
  return false;
}

bool COFFAsmParser::parseSEHDirectiveEndProc(StringRef, SMLoc Loc) {
  Lex();
  getStreamer().emitWinCFIEndProc(Loc);
  return false;
}

bool COFFAsmParser::parseSEHDirectiveEndFuncletOrFunc(StringRef, SMLoc Loc) {
  Lex();
  getStreamer().emitWinCFIFuncletOrFuncEnd(Loc);
  return false;
}

```
- **EN**: Implements logic around `Lex`, `getStreamer`, `parseSEHDirectiveEndProc`, `parseSEHDirectiveEndFuncletOrFunc`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `Lex`, `getStreamer`, `parseSEHDirectiveEndProc`, `parseSEHDirectiveEndFuncletOrFunc` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 684-707
```cpp
bool COFFAsmParser::parseSEHDirectiveSplitChained(StringRef, SMLoc Loc) {
  Lex();
  getStreamer().emitWinCFISplitChained(Loc);
  return false;
}

bool COFFAsmParser::parseSEHDirectiveHandler(StringRef, SMLoc Loc) {
  MCSymbol *handler;
  if (getParser().parseSymbol(handler))
    return true;

  if (getLexer().isNot(AsmToken::Comma))
    return TokError("you must specify one or both of @unwind or @except");
  Lex();
  bool unwind = false, except = false;
  if (parseAtUnwindOrAtExcept(unwind, except))
    return true;
  if (getLexer().is(AsmToken::Comma)) {
    Lex();
    if (parseAtUnwindOrAtExcept(unwind, except))
      return true;
  }
  if (getLexer().isNot(AsmToken::EndOfStatement))
    return TokError("unexpected token in directive");
```
- **EN**: Implements logic around `parseSEHDirectiveSplitChained`, `Lex`, `getStreamer`, `parseSEHDirectiveHandler`, and 1 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseSEHDirectiveSplitChained`, `Lex`, `getStreamer`, `parseSEHDirectiveHandler`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 708-719
```cpp

  Lex();
  getStreamer().emitWinEHHandler(handler, unwind, except, Loc);
  return false;
}

bool COFFAsmParser::parseSEHDirectiveHandlerData(StringRef, SMLoc Loc) {
  Lex();
  getStreamer().emitWinEHHandlerData();
  return false;
}

```
- **EN**: Implements logic around `Lex`, `getStreamer`, `parseSEHDirectiveHandlerData`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `Lex`, `getStreamer`, `parseSEHDirectiveHandlerData` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 720-732
```cpp
bool COFFAsmParser::parseSEHDirectiveAllocStack(StringRef, SMLoc Loc) {
  int64_t Size;
  if (getParser().parseAbsoluteExpression(Size))
    return true;

  if (getLexer().isNot(AsmToken::EndOfStatement))
    return TokError("unexpected token in directive");

  Lex();
  getStreamer().emitWinCFIAllocStack(Size, Loc);
  return false;
}

```
- **EN**: Implements logic around `parseSEHDirectiveAllocStack`, `TokError`, `Lex`, `getStreamer`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `parseSEHDirectiveAllocStack`, `TokError`, `Lex`, `getStreamer` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 733-744
```cpp
bool COFFAsmParser::parseSEHDirectiveEndProlog(StringRef, SMLoc Loc) {
  Lex();
  getStreamer().emitWinCFIEndProlog(Loc);
  return false;
}

bool COFFAsmParser::ParseSEHDirectiveBeginEpilog(StringRef, SMLoc Loc) {
  Lex();
  getStreamer().emitWinCFIBeginEpilogue(Loc);
  return false;
}

```
- **EN**: Implements logic around `parseSEHDirectiveEndProlog`, `Lex`, `getStreamer`, `ParseSEHDirectiveBeginEpilog`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `parseSEHDirectiveEndProlog`, `Lex`, `getStreamer`, `ParseSEHDirectiveBeginEpilog` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 745-756
```cpp
bool COFFAsmParser::ParseSEHDirectiveEndEpilog(StringRef, SMLoc Loc) {
  Lex();
  getStreamer().emitWinCFIEndEpilogue(Loc);
  return false;
}

bool COFFAsmParser::ParseSEHDirectiveUnwindV2Start(StringRef, SMLoc Loc) {
  Lex();
  getStreamer().emitWinCFIUnwindV2Start(Loc);
  return false;
}

```
- **EN**: Implements logic around `ParseSEHDirectiveEndEpilog`, `Lex`, `getStreamer`, `ParseSEHDirectiveUnwindV2Start`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `ParseSEHDirectiveEndEpilog`, `Lex`, `getStreamer`, `ParseSEHDirectiveUnwindV2Start` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 757-772
```cpp
bool COFFAsmParser::ParseSEHDirectiveUnwindVersion(StringRef, SMLoc Loc) {
  int64_t Version;
  if (getParser().parseIntToken(Version, "expected unwind version number"))
    return true;

  if ((Version < 1) || (Version > UINT8_MAX))
    return Error(Loc, "invalid unwind version");

  if (getLexer().isNot(AsmToken::EndOfStatement))
    return TokError("unexpected token in directive");

  Lex();
  getStreamer().emitWinCFIUnwindVersion(Version, Loc);
  return false;
}

```
- **EN**: Implements logic around `ParseSEHDirectiveUnwindVersion`, `Error`, `TokError`, `Lex`, and 1 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `ParseSEHDirectiveUnwindVersion`, `Error`, `TokError`, `Lex`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 773-789
```cpp
bool COFFAsmParser::parseAtUnwindOrAtExcept(bool &unwind, bool &except) {
  StringRef identifier;
  if (getLexer().isNot(AsmToken::At) && getLexer().isNot(AsmToken::Percent))
    return TokError("a handler attribute must begin with '@' or '%'");
  SMLoc startLoc = getLexer().getLoc();
  Lex();
  if (getParser().parseIdentifier(identifier))
    return Error(startLoc, "expected @unwind or @except");
  if (identifier == "unwind")
    unwind = true;
  else if (identifier == "except")
    except = true;
  else
    return Error(startLoc, "expected @unwind or @except");
  return false;
}

```
- **EN**: Implements logic around `parseAtUnwindOrAtExcept`, `TokError`, `getLexer`, `Lex`, and 1 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseAtUnwindOrAtExcept`, `TokError`, `getLexer`, `Lex`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 790-790
```cpp
MCAsmParserExtension *llvm::createCOFFAsmParser() { return new COFFAsmParser; }
```
- **EN**: Implements logic around `createCOFFAsmParser`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `createCOFFAsmParser` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

## Key Concepts / 关键概念

- **Assembly parsing / 汇编解析**:
  - **EN**: Tokenizes assembly text, interprets directives, and builds MC expressions or instructions
  - **CN**: 对汇编文本做词法与语法解析，并构建 MC 表达式或指令
- **Directive handling / 伪指令处理**:
  - **EN**: Interprets assembler directives and maps them to streamer operations
  - **CN**: 解释汇编伪指令并把它们映射为 streamer 操作
- **Object layout / 目标文件布局**:
  - **EN**: Organizes fragments into sections and computes final offsets
  - **CN**: 把片段组织到节区中并计算最终偏移

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/ADT/StringRef.h`, `llvm/ADT/StringSwitch.h`, `llvm/ADT/Twine.h`, `llvm/BinaryFormat/COFF.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCDirectives.h`, `llvm/MC/MCParser/AsmLexer.h`, `llvm/MC/MCParser/MCAsmParserExtension.h`, `llvm/MC/MCSectionCOFF.h`, `llvm/MC/MCStreamer.h`, `llvm/Support/SMLoc.h`, `llvm/TargetParser/Triple.h` ... (+3 more)
- **LLVM subsystems / LLVM 子系统**: MC, Support, BinaryFormat, Target/TargetParser
