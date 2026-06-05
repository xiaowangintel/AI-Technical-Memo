# COFFMasmParser.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/MC/MCParser/COFFMasmParser.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements COFF MASM Assembly Parser.
  - **CN**: 实现 LLVM MC 汇编解析、伪指令处理、表达式解析以及面向 streamer 的解析辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```cpp
//===- COFFMasmParser.cpp - COFF MASM Assembly Parser ---------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/ADT/StringRef.h"
#include "llvm/ADT/Twine.h"
#include "llvm/BinaryFormat/COFF.h"
#include "llvm/MC/MCAsmMacro.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCParser/AsmLexer.h"
#include "llvm/MC/MCParser/MCAsmParserExtension.h"
#include "llvm/MC/MCParser/MCTargetAsmParser.h"
#include "llvm/MC/MCSectionCOFF.h"
#include "llvm/MC/MCStreamer.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ADT/StringRef.h`, `llvm/ADT/Twine.h`, `llvm/BinaryFormat/COFF.h`, `llvm/MC/MCAsmMacro.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ADT/StringRef.h`, `llvm/ADT/Twine.h`, `llvm/BinaryFormat/COFF.h`, `llvm/MC/MCAsmMacro.h`。

### Lines 19-27
```cpp
#include "llvm/MC/MCSymbolCOFF.h"
#include "llvm/MC/SectionKind.h"
#include "llvm/Support/SMLoc.h"
#include <cstdint>

using namespace llvm;

namespace {

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/MCSymbolCOFF.h`, `llvm/MC/SectionKind.h`, `llvm/Support/SMLoc.h`, `cstdint`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/MCSymbolCOFF.h`, `llvm/MC/SectionKind.h`, `llvm/Support/SMLoc.h`, `cstdint`。

### Lines 28-37
```cpp
class COFFMasmParser : public MCAsmParserExtension {
  template <bool (COFFMasmParser::*HandlerMethod)(StringRef, SMLoc)>
  void addDirectiveHandler(StringRef Directive) {
    MCAsmParser::ExtensionDirectiveHandler Handler =
        std::make_pair(this, HandleDirective<COFFMasmParser, HandlerMethod>);
    getParser().addDirectiveHandler(Directive, Handler);
  }

  bool parseSectionSwitch(StringRef SectionName, unsigned Characteristics);

```
- **EN**: Introduces declarations for `COFFMasmParser`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `COFFMasmParser` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 38-48
```cpp
  bool parseSectionSwitch(StringRef SectionName, unsigned Characteristics,
                          StringRef COMDATSymName, COFF::COMDATType Type,
                          Align Alignment);

  bool parseDirectiveProc(StringRef, SMLoc);
  bool parseDirectiveEndProc(StringRef, SMLoc);
  bool parseDirectiveSegment(StringRef, SMLoc);
  bool parseDirectiveSegmentEnd(StringRef, SMLoc);
  bool parseDirectiveIncludelib(StringRef, SMLoc);
  bool parseDirectiveOption(StringRef, SMLoc);

```
- **EN**: Implements logic around `parseSectionSwitch`, `parseDirectiveProc`, `parseDirectiveEndProc`, `parseDirectiveSegment`, and 3 more symbols; this block parses assembly syntax or operands; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `parseSectionSwitch`, `parseDirectiveProc`, `parseDirectiveEndProc`, `parseDirectiveSegment`, and 3 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 49-60
```cpp
  bool parseDirectiveAlias(StringRef, SMLoc);

  bool parseSEHDirectiveAllocStack(StringRef, SMLoc);
  bool parseSEHDirectiveEndProlog(StringRef, SMLoc);

  bool IgnoreDirective(StringRef, SMLoc) {
    while (!getLexer().is(AsmToken::EndOfStatement)) {
      Lex();
    }
    return false;
  }

```
- **EN**: Implements logic around `parseDirectiveAlias`, `parseSEHDirectiveAllocStack`, `parseSEHDirectiveEndProlog`, `IgnoreDirective`, and 1 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveAlias`, `parseSEHDirectiveAllocStack`, `parseSEHDirectiveEndProlog`, `IgnoreDirective`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 61-70
```cpp
  void Initialize(MCAsmParser &Parser) override {
    // Call the base implementation.
    MCAsmParserExtension::Initialize(Parser);

    // x64 directives
    addDirectiveHandler<&COFFMasmParser::parseSEHDirectiveAllocStack>(
        ".allocstack");
    addDirectiveHandler<&COFFMasmParser::parseSEHDirectiveEndProlog>(
        ".endprolog");

```
- **EN**: Implements logic around `Initialize`, `parseSEHDirectiveAllocStack>`, `parseSEHDirectiveEndProlog>`; this block parses assembly syntax or operands.
- **CN**: 围绕 `Initialize`, `parseSEHDirectiveAllocStack>`, `parseSEHDirectiveEndProlog>` 实现具体逻辑；这一段解析汇编语法或操作数。

### Lines 71-87
```cpp
    // Code label directives
    // label
    // org

    // Conditional control flow directives
    // .break
    // .continue
    // .else
    // .elseif
    // .endif
    // .endw
    // .if
    // .repeat
    // .until
    // .untilcxz
    // .while

```
- **EN**: Documents the next declaration group or explains a subsystem-specific rule.
- **CN**: 为后续声明分组提供说明，或解释某个子系统的专用规则。

### Lines 88-105
```cpp
    // Data allocation directives
    // align
    // even
    // mmword
    // tbyte
    // xmmword
    // ymmword

    // Listing control directives
    addDirectiveHandler<&COFFMasmParser::IgnoreDirective>(".cref");
    addDirectiveHandler<&COFFMasmParser::IgnoreDirective>(".list");
    addDirectiveHandler<&COFFMasmParser::IgnoreDirective>(".listall");
    addDirectiveHandler<&COFFMasmParser::IgnoreDirective>(".listif");
    addDirectiveHandler<&COFFMasmParser::IgnoreDirective>(".listmacro");
    addDirectiveHandler<&COFFMasmParser::IgnoreDirective>(".listmacroall");
    addDirectiveHandler<&COFFMasmParser::IgnoreDirective>(".nocref");
    addDirectiveHandler<&COFFMasmParser::IgnoreDirective>(".nolist");
    addDirectiveHandler<&COFFMasmParser::IgnoreDirective>(".nolistif");
```
- **EN**: Implements logic around `IgnoreDirective>`; this block parses assembly syntax or operands.
- **CN**: 围绕 `IgnoreDirective>` 实现具体逻辑；这一段解析汇编语法或操作数。

### Lines 106-114
```cpp
    addDirectiveHandler<&COFFMasmParser::IgnoreDirective>(".nolistmacro");
    addDirectiveHandler<&COFFMasmParser::IgnoreDirective>("page");
    addDirectiveHandler<&COFFMasmParser::IgnoreDirective>("subtitle");
    addDirectiveHandler<&COFFMasmParser::IgnoreDirective>(".tfcond");
    addDirectiveHandler<&COFFMasmParser::IgnoreDirective>("title");

    // Macro directives
    // goto

```
- **EN**: Implements logic around `IgnoreDirective>`; this block parses assembly syntax or operands.
- **CN**: 围绕 `IgnoreDirective>` 实现具体逻辑；这一段解析汇编语法或操作数。

### Lines 115-125
```cpp
    // Miscellaneous directives
    addDirectiveHandler<&COFFMasmParser::parseDirectiveAlias>("alias");
    // assume
    // .fpo
    addDirectiveHandler<&COFFMasmParser::parseDirectiveIncludelib>(
        "includelib");
    addDirectiveHandler<&COFFMasmParser::parseDirectiveOption>("option");
    // popcontext
    // pushcontext
    // .safeseh

```
- **EN**: Implements logic around `parseDirectiveAlias>`, `parseDirectiveIncludelib>`, `parseDirectiveOption>`; this block parses assembly syntax or operands.
- **CN**: 围绕 `parseDirectiveAlias>`, `parseDirectiveIncludelib>`, `parseDirectiveOption>` 实现具体逻辑；这一段解析汇编语法或操作数。

### Lines 126-143
```cpp
    // Procedure directives
    addDirectiveHandler<&COFFMasmParser::parseDirectiveEndProc>("endp");
    // invoke (32-bit only)
    addDirectiveHandler<&COFFMasmParser::parseDirectiveProc>("proc");
    // proto

    // Processor directives; all ignored
    addDirectiveHandler<&COFFMasmParser::IgnoreDirective>(".386");
    addDirectiveHandler<&COFFMasmParser::IgnoreDirective>(".386p");
    addDirectiveHandler<&COFFMasmParser::IgnoreDirective>(".387");
    addDirectiveHandler<&COFFMasmParser::IgnoreDirective>(".486");
    addDirectiveHandler<&COFFMasmParser::IgnoreDirective>(".486p");
    addDirectiveHandler<&COFFMasmParser::IgnoreDirective>(".586");
    addDirectiveHandler<&COFFMasmParser::IgnoreDirective>(".586p");
    addDirectiveHandler<&COFFMasmParser::IgnoreDirective>(".686");
    addDirectiveHandler<&COFFMasmParser::IgnoreDirective>(".686p");
    addDirectiveHandler<&COFFMasmParser::IgnoreDirective>(".k3d");
    addDirectiveHandler<&COFFMasmParser::IgnoreDirective>(".mmx");
```
- **EN**: Implements logic around `parseDirectiveEndProc>`, `parseDirectiveProc>`, `IgnoreDirective>`; this block parses assembly syntax or operands.
- **CN**: 围绕 `parseDirectiveEndProc>`, `parseDirectiveProc>`, `IgnoreDirective>` 实现具体逻辑；这一段解析汇编语法或操作数。

### Lines 144-157
```cpp
    addDirectiveHandler<&COFFMasmParser::IgnoreDirective>(".xmm");

    // Scope directives
    // comm
    // externdef

    // Segment directives
    // .alpha (32-bit only, order segments alphabetically)
    // .dosseg (32-bit only, order segments in DOS convention)
    // .seq (32-bit only, order segments sequentially)
    addDirectiveHandler<&COFFMasmParser::parseDirectiveSegmentEnd>("ends");
    // group (32-bit only)
    addDirectiveHandler<&COFFMasmParser::parseDirectiveSegment>("segment");

```
- **EN**: Implements logic around `IgnoreDirective>`, `parseDirectiveSegmentEnd>`, `parseDirectiveSegment>`; this block parses assembly syntax or operands.
- **CN**: 围绕 `IgnoreDirective>`, `parseDirectiveSegmentEnd>`, `parseDirectiveSegment>` 实现具体逻辑；这一段解析汇编语法或操作数。

### Lines 158-171
```cpp
    // Simplified segment directives
    addDirectiveHandler<&COFFMasmParser::parseSectionDirectiveCode>(".code");
    // .const
    addDirectiveHandler<&COFFMasmParser::parseSectionDirectiveInitializedData>(
        ".data");
    addDirectiveHandler<
        &COFFMasmParser::parseSectionDirectiveUninitializedData>(".data?");
    // .exit
    // .fardata
    // .fardata?
    addDirectiveHandler<&COFFMasmParser::IgnoreDirective>(".model");
    // .stack
    // .startup

```
- **EN**: Implements logic around `parseSectionDirectiveCode>`, `parseSectionDirectiveInitializedData>`, `parseSectionDirectiveUninitializedData>`, `IgnoreDirective>`; this block parses assembly syntax or operands; updates MC section or symbol state.
- **CN**: 围绕 `parseSectionDirectiveCode>`, `parseSectionDirectiveInitializedData>`, `parseSectionDirectiveUninitializedData>`, `IgnoreDirective>` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态。

### Lines 172-182
```cpp
    // String directives, written <name> <directive> <params>
    // catstr (equivalent to <name> TEXTEQU <params>)
    // instr (equivalent to <name> = @InStr(<params>))
    // sizestr (equivalent to <name> = @SizeStr(<params>))
    // substr (equivalent to <name> TEXTEQU @SubStr(<params>))

    // Structure and record directives
    // record
    // typedef
  }

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 183-194
```cpp
  bool parseSectionDirectiveCode(StringRef, SMLoc) {
    return parseSectionSwitch(".text", COFF::IMAGE_SCN_CNT_CODE |
                                           COFF::IMAGE_SCN_MEM_EXECUTE |
                                           COFF::IMAGE_SCN_MEM_READ);
  }

  bool parseSectionDirectiveInitializedData(StringRef, SMLoc) {
    return parseSectionSwitch(".data", COFF::IMAGE_SCN_CNT_INITIALIZED_DATA |
                                           COFF::IMAGE_SCN_MEM_READ |
                                           COFF::IMAGE_SCN_MEM_WRITE);
  }

```
- **EN**: Implements logic around `parseSectionDirectiveCode`, `parseSectionSwitch`, `parseSectionDirectiveInitializedData`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `parseSectionDirectiveCode`, `parseSectionSwitch`, `parseSectionDirectiveInitializedData` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 195-204
```cpp
  bool parseSectionDirectiveUninitializedData(StringRef, SMLoc) {
    return parseSectionSwitch(".bss", COFF::IMAGE_SCN_CNT_UNINITIALIZED_DATA |
                                          COFF::IMAGE_SCN_MEM_READ |
                                          COFF::IMAGE_SCN_MEM_WRITE);
  }

  /// Stack of active procedure definitions.
  SmallVector<StringRef, 1> CurrentProcedures;
  SmallVector<bool, 1> CurrentProceduresFramed;

```
- **EN**: Implements logic around `parseSectionDirectiveUninitializedData`, `parseSectionSwitch`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `parseSectionDirectiveUninitializedData`, `parseSectionSwitch` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 205-216
```cpp
public:
  COFFMasmParser() = default;
};

} // end anonymous namespace.

bool COFFMasmParser::parseSectionSwitch(StringRef SectionName,
                                        unsigned Characteristics) {
  return parseSectionSwitch(SectionName, Characteristics, "",
                            (COFF::COMDATType)0, Align(16));
}

```
- **EN**: Implements logic around `COFFMasmParser`, `parseSectionSwitch`, `Align`; this block parses assembly syntax or operands; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `COFFMasmParser`, `parseSectionSwitch`, `Align` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 217-225
```cpp
bool COFFMasmParser::parseSectionSwitch(StringRef SectionName,
                                        unsigned Characteristics,
                                        StringRef COMDATSymName,
                                        COFF::COMDATType Type,
                                        Align Alignment) {
  if (getLexer().isNot(AsmToken::EndOfStatement))
    return TokError("unexpected token in section switching directive");
  Lex();

```
- **EN**: Implements logic around `parseSectionSwitch`, `TokError`, `Lex`; this block parses assembly syntax or operands; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `parseSectionSwitch`, `TokError`, `Lex` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 226-240
```cpp
  MCSection *Section = getContext().getCOFFSection(SectionName, Characteristics,
                                                   COMDATSymName, Type);
  Section->setAlignment(Alignment);
  getStreamer().switchSection(Section);

  return false;
}

bool COFFMasmParser::parseDirectiveSegment(StringRef Directive, SMLoc Loc) {
  StringRef SegmentName;
  if (!getLexer().is(AsmToken::Identifier))
    return TokError("expected identifier in directive");
  SegmentName = getTok().getIdentifier();
  Lex();

```
- **EN**: Implements logic around `getContext`, `setAlignment`, `getStreamer`, `parseDirectiveSegment`, and 3 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `getContext`, `setAlignment`, `getStreamer`, `parseDirectiveSegment`, and 3 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 241-254
```cpp
  StringRef SectionName = SegmentName;
  SmallVector<char, 247> SectionNameVector;

  StringRef Class;
  if (SegmentName == "_TEXT" || SegmentName.starts_with("_TEXT$")) {
    if (SegmentName.size() == 5) {
      SectionName = ".text";
    } else {
      SectionName =
          (".text$" + SegmentName.substr(6)).toStringRef(SectionNameVector);
    }
    Class = "CODE";
  }

```
- **EN**: Implements logic around `substr`; this block updates MC section or symbol state.
- **CN**: 围绕 `substr` 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 255-272
```cpp
  // Parse all options to end of statement.
  // Alignment defaults to PARA if unspecified.
  int64_t Alignment = 16;
  // Default flags are used only if no characteristics are set.
  bool DefaultCharacteristics = true;
  unsigned Flags = 0;
  // "obsolete" according to the documentation, but still supported.
  bool Readonly = false;
  while (getLexer().isNot(AsmToken::EndOfStatement)) {
    switch (getTok().getKind()) {
    default:
      break;
    case AsmToken::String: {
      // Class identifier; overrides Kind.
      Class = getTok().getStringContents();
      Lex();
      break;
    }
```
- **EN**: Implements logic around `getTok`, `Lex`; this block parses assembly syntax or operands; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `getTok`, `Lex` 实现具体逻辑；这一段解析汇编语法或操作数，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 273-290
```cpp
    case AsmToken::Identifier: {
      SMLoc KeywordLoc = getTok().getLoc();
      StringRef Keyword;
      if (getParser().parseIdentifier(Keyword)) {
        llvm_unreachable("failed to parse identifier at an identifier token");
      }
      if (Keyword.equals_insensitive("byte")) {
        Alignment = 1;
      } else if (Keyword.equals_insensitive("word")) {
        Alignment = 2;
      } else if (Keyword.equals_insensitive("dword")) {
        Alignment = 4;
      } else if (Keyword.equals_insensitive("para")) {
        Alignment = 16;
      } else if (Keyword.equals_insensitive("page")) {
        Alignment = 256;
      } else if (Keyword.equals_insensitive("align")) {
        if (getParser().parseToken(AsmToken::LParen) ||
```
- **EN**: Implements logic around `getTok`, `llvm_unreachable`; this block parses assembly syntax or operands.
- **CN**: 围绕 `getTok`, `llvm_unreachable` 实现具体逻辑；这一段解析汇编语法或操作数。

### Lines 291-308
```cpp
            getParser().parseIntToken(Alignment,
                                      "Expected integer alignment") ||
            getParser().parseToken(AsmToken::RParen)) {
          return Error(getTok().getLoc(),
                       "Expected (n) following ALIGN in SEGMENT directive");
        }
        if (!isPowerOf2_64(Alignment) || Alignment > 8192) {
          return Error(KeywordLoc,
                       "ALIGN argument must be a power of 2 from 1 to 8192");
        }
      } else if (Keyword.equals_insensitive("alias")) {
        if (getParser().parseToken(AsmToken::LParen) ||
            !getTok().is(AsmToken::String))
          return Error(
              getTok().getLoc(),
              "Expected (string) following ALIAS in SEGMENT directive");
        SectionName = getTok().getStringContents();
        Lex();
```
- **EN**: Implements logic around `getParser`, `Error`, `Expected`, `getTok`, and 1 more symbols; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getParser`, `Error`, `Expected`, `getTok`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 309-326
```cpp
        if (getParser().parseToken(AsmToken::RParen))
          return Error(
              getTok().getLoc(),
              "Expected (string) following ALIAS in SEGMENT directive");
      } else if (Keyword.equals_insensitive("readonly")) {
        Readonly = true;
      } else {
        unsigned Characteristic =
            StringSwitch<unsigned>(Keyword)
                .CaseLower("info", COFF::IMAGE_SCN_LNK_INFO)
                .CaseLower("read", COFF::IMAGE_SCN_MEM_READ)
                .CaseLower("write", COFF::IMAGE_SCN_MEM_WRITE)
                .CaseLower("execute", COFF::IMAGE_SCN_MEM_EXECUTE)
                .CaseLower("shared", COFF::IMAGE_SCN_MEM_SHARED)
                .CaseLower("nopage", COFF::IMAGE_SCN_MEM_NOT_PAGED)
                .CaseLower("nocache", COFF::IMAGE_SCN_MEM_NOT_CACHED)
                .CaseLower("discard", COFF::IMAGE_SCN_MEM_DISCARDABLE)
                .Default(-1);
```
- **EN**: Implements logic around `Error`, `getTok`, `Expected`, `StringSwitch<unsigned>`, and 2 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `Error`, `getTok`, `Expected`, `StringSwitch<unsigned>`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 327-338
```cpp
        if (Characteristic == static_cast<unsigned>(-1)) {
          return Error(KeywordLoc,
                       "Expected characteristic in SEGMENT directive; found '" +
                           Keyword + "'");
        }
        Flags |= Characteristic;
        DefaultCharacteristics = false;
      }
    }
    }
  }

```
- **EN**: Implements logic around `Error`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `Error` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 339-356
```cpp
  SectionKind Kind = StringSwitch<SectionKind>(Class)
                         .CaseLower("data", SectionKind::getData())
                         .CaseLower("code", SectionKind::getText())
                         .CaseLower("const", SectionKind::getReadOnly())
                         .Default(SectionKind::getData());
  if (Kind.isText()) {
    if (DefaultCharacteristics) {
      Flags |= COFF::IMAGE_SCN_MEM_EXECUTE | COFF::IMAGE_SCN_MEM_READ;
    }
    Flags |= COFF::IMAGE_SCN_CNT_CODE;
  } else {
    if (DefaultCharacteristics) {
      Flags |= COFF::IMAGE_SCN_MEM_READ | COFF::IMAGE_SCN_MEM_WRITE;
    }
    Flags |= COFF::IMAGE_SCN_CNT_INITIALIZED_DATA;
  }
  if (Readonly) {
    Flags &= ~COFF::IMAGE_SCN_MEM_WRITE;
```
- **EN**: Implements logic around `StringSwitch<SectionKind>`, `CaseLower`, `Default`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `StringSwitch<SectionKind>`, `CaseLower`, `Default` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 357-367
```cpp
  }

  MCSection *Section = getContext().getCOFFSection(SectionName, Flags, "",
                                                   (COFF::COMDATType)(0));
  if (Alignment != 0) {
    Section->setAlignment(Align(Alignment));
  }
  getStreamer().switchSection(Section);
  return false;
}

```
- **EN**: Implements logic around `getContext`, `setAlignment`, `getStreamer`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `getContext`, `setAlignment`, `getStreamer` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 368-380
```cpp
/// parseDirectiveSegmentEnd
///  ::= identifier "ends"
bool COFFMasmParser::parseDirectiveSegmentEnd(StringRef Directive, SMLoc Loc) {
  StringRef SegmentName;
  if (!getLexer().is(AsmToken::Identifier))
    return TokError("expected identifier in directive");
  SegmentName = getTok().getIdentifier();

  // Ignore; no action necessary.
  Lex();
  return false;
}

```
- **EN**: Implements logic around `parseDirectiveSegmentEnd`, `TokError`, `getTok`, `Lex`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveSegmentEnd`, `TokError`, `getTok`, `Lex` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 381-398
```cpp
/// parseDirectiveIncludelib
///  ::= "includelib" identifier
bool COFFMasmParser::parseDirectiveIncludelib(StringRef Directive, SMLoc Loc) {
  StringRef Lib;
  if (getParser().parseIdentifier(Lib))
    return TokError("expected identifier in includelib directive");

  unsigned Flags = COFF::IMAGE_SCN_MEM_PRELOAD | COFF::IMAGE_SCN_MEM_16BIT;
  getStreamer().pushSection();
  getStreamer().switchSection(getContext().getCOFFSection(
      ".drectve", Flags, "", (COFF::COMDATType)(0)));
  getStreamer().emitBytes("/DEFAULTLIB:");
  getStreamer().emitBytes(Lib);
  getStreamer().emitBytes(" ");
  getStreamer().popSection();
  return false;
}

```
- **EN**: Implements logic around `parseDirectiveIncludelib`, `TokError`, `getStreamer`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveIncludelib`, `TokError`, `getStreamer` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 399-416
```cpp
/// parseDirectiveOption
///  ::= "option" option-list
bool COFFMasmParser::parseDirectiveOption(StringRef Directive, SMLoc Loc) {
  auto parseOption = [&]() -> bool {
    StringRef Option;
    if (getParser().parseIdentifier(Option))
      return TokError("expected identifier for option name");
    if (Option.equals_insensitive("prologue")) {
      StringRef MacroId;
      if (parseToken(AsmToken::Colon) || getParser().parseIdentifier(MacroId))
        return TokError("expected :macroId after OPTION PROLOGUE");
      if (MacroId.equals_insensitive("none")) {
        // Since we currently don't implement prologues/epilogues, NONE is our
        // default.
        return false;
      }
      return TokError("OPTION PROLOGUE is currently unsupported");
    }
```
- **EN**: Implements logic around `parseDirectiveOption`, `TokError`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveOption`, `TokError` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 417-430
```cpp
    if (Option.equals_insensitive("epilogue")) {
      StringRef MacroId;
      if (parseToken(AsmToken::Colon) || getParser().parseIdentifier(MacroId))
        return TokError("expected :macroId after OPTION EPILOGUE");
      if (MacroId.equals_insensitive("none")) {
        // Since we currently don't implement prologues/epilogues, NONE is our
        // default.
        return false;
      }
      return TokError("OPTION EPILOGUE is currently unsupported");
    }
    return TokError("OPTION '" + Option + "' is currently unsupported");
  };

```
- **EN**: Implements logic around `TokError`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `TokError` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 431-439
```cpp
  if (parseMany(parseOption))
    return addErrorSuffix(" in OPTION directive");
  return false;
}

/// parseDirectiveProc
/// TODO(epastor): Implement parameters and other attributes.
///  ::= label "proc" [[distance]]
///          statements
```
- **EN**: Implements logic around `addErrorSuffix`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `addErrorSuffix` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 440-457
```cpp
///      label "endproc"
bool COFFMasmParser::parseDirectiveProc(StringRef Directive, SMLoc Loc) {
  if (!getStreamer().getCurrentFragment())
    return Error(getTok().getLoc(), "expected section directive");

  MCSymbol *Sym;
  if (getParser().parseSymbol(Sym))
    return Error(Loc, "expected identifier for procedure");
  if (getLexer().is(AsmToken::Identifier)) {
    StringRef nextVal = getTok().getString();
    SMLoc nextLoc = getTok().getLoc();
    if (nextVal.equals_insensitive("far")) {
      // TODO(epastor): Handle far procedure definitions.
      Lex();
      return Error(nextLoc, "far procedure definitions not yet supported");
    } else if (nextVal.equals_insensitive("near")) {
      Lex();
      nextVal = getTok().getString();
```
- **EN**: Implements logic around `parseDirectiveProc`, `Error`, `getTok`, `Lex`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveProc`, `Error`, `getTok`, `Lex` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 458-467
```cpp
      nextLoc = getTok().getLoc();
    }
  }

  // Define symbol as simple external function
  auto *COFFSym = static_cast<MCSymbolCOFF *>(Sym);
  COFFSym->setExternal(true);
  COFFSym->setType(COFF::IMAGE_SYM_DTYPE_FUNCTION
                   << COFF::SCT_COMPLEX_TYPE_SHIFT);

```
- **EN**: Implements logic around `getTok`, `setExternal`, `setType`; this block parses assembly syntax or operands; updates MC section or symbol state.
- **CN**: 围绕 `getTok`, `setExternal`, `setType` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态。

### Lines 468-476
```cpp
  bool Framed = false;
  if (getLexer().is(AsmToken::Identifier) &&
      getTok().getString().equals_insensitive("frame")) {
    Lex();
    Framed = true;
    getStreamer().emitWinCFIStartProc(Sym, Loc);
  }
  getStreamer().emitLabel(Sym, Loc);

```
- **EN**: Implements logic around `getTok`, `Lex`, `getStreamer`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `getTok`, `Lex`, `getStreamer` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为。

### Lines 477-486
```cpp
  CurrentProcedures.push_back(Sym->getName());
  CurrentProceduresFramed.push_back(Framed);
  return false;
}
bool COFFMasmParser::parseDirectiveEndProc(StringRef Directive, SMLoc Loc) {
  StringRef Label;
  SMLoc LabelLoc = getTok().getLoc();
  if (getParser().parseIdentifier(Label))
    return Error(LabelLoc, "expected identifier for procedure end");

```
- **EN**: Implements logic around `push_back`, `parseDirectiveEndProc`, `getTok`, `Error`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `push_back`, `parseDirectiveEndProc`, `getTok`, `Error` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 487-500
```cpp
  if (CurrentProcedures.empty())
    return Error(Loc, "endp outside of procedure block");
  else if (!CurrentProcedures.back().equals_insensitive(Label))
    return Error(LabelLoc, "endp does not match current procedure '" +
                               CurrentProcedures.back() + "'");

  if (CurrentProceduresFramed.back()) {
    getStreamer().emitWinCFIEndProc(Loc);
  }
  CurrentProcedures.pop_back();
  CurrentProceduresFramed.pop_back();
  return false;
}

```
- **EN**: Implements logic around `Error`, `back`, `getStreamer`, `pop_back`; this block manipulates JITLink graph structures or link-time passes; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `Error`, `back`, `getStreamer`, `pop_back` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 501-511
```cpp
bool COFFMasmParser::parseDirectiveAlias(StringRef Directive, SMLoc Loc) {
  std::string AliasName, ActualName;
  if (getTok().isNot(AsmToken::Less) ||
      getParser().parseAngleBracketString(AliasName))
    return Error(getTok().getLoc(), "expected <aliasName>");
  if (getParser().parseToken(AsmToken::Equal))
    return addErrorSuffix(" in " + Directive + " directive");
  if (getTok().isNot(AsmToken::Less) ||
      getParser().parseAngleBracketString(ActualName))
    return Error(getTok().getLoc(), "expected <actualName>");

```
- **EN**: Implements logic around `parseDirectiveAlias`, `getParser`, `Error`, `addErrorSuffix`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveAlias`, `getParser`, `Error`, `addErrorSuffix` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 512-529
```cpp
  MCSymbol *Alias = getContext().parseSymbol(AliasName);
  MCSymbol *Actual = getContext().parseSymbol(ActualName);

  getStreamer().emitWeakReference(Alias, Actual);

  return false;
}

bool COFFMasmParser::parseSEHDirectiveAllocStack(StringRef Directive,
                                                 SMLoc Loc) {
  int64_t Size;
  SMLoc SizeLoc = getTok().getLoc();
  if (getParser().parseAbsoluteExpression(Size))
    return Error(SizeLoc, "expected integer size");
  if (Size % 8 != 0)
    return Error(SizeLoc, "stack size must be a multiple of 8");
  getStreamer().emitWinCFIAllocStack(static_cast<unsigned>(Size), Loc);
  return false;
```
- **EN**: Implements logic around `getContext`, `getStreamer`, `parseSEHDirectiveAllocStack`, `getTok`, and 1 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getContext`, `getStreamer`, `parseSEHDirectiveAllocStack`, `getTok`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 530-540
```cpp
}

bool COFFMasmParser::parseSEHDirectiveEndProlog(StringRef Directive,
                                                SMLoc Loc) {
  getStreamer().emitWinCFIEndProlog(Loc);
  return false;
}

MCAsmParserExtension *llvm::createCOFFMasmParser() {
  return new COFFMasmParser;
}
```
- **EN**: Implements logic around `parseSEHDirectiveEndProlog`, `getStreamer`, `createCOFFMasmParser`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `parseSEHDirectiveEndProlog`, `getStreamer`, `createCOFFMasmParser` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

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

- **Direct includes / 直接包含**: `llvm/ADT/StringRef.h`, `llvm/ADT/Twine.h`, `llvm/BinaryFormat/COFF.h`, `llvm/MC/MCAsmMacro.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCParser/AsmLexer.h`, `llvm/MC/MCParser/MCAsmParserExtension.h`, `llvm/MC/MCParser/MCTargetAsmParser.h`, `llvm/MC/MCSectionCOFF.h`, `llvm/MC/MCStreamer.h`, `llvm/MC/MCSymbolCOFF.h`, `llvm/MC/SectionKind.h` ... (+2 more)
- **LLVM subsystems / LLVM 子系统**: MC, Support, BinaryFormat
