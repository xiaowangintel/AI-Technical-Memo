# WasmAsmParser.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/MC/MCParser/WasmAsmParser.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file contains processing for generic directives implemented using MCTargetStreamer, the ones that depend on WebAssemblyTargetStreamer are in WebAssemblyAsmParser.
  - **CN**: 实现 LLVM MC 汇编解析、伪指令处理、表达式解析以及面向 streamer 的解析辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17
```cpp
//===- WasmAsmParser.cpp - Wasm Assembly Parser -----------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
// --
//
// Note, this is for wasm, the binary format (analogous to ELF), not wasm,
// the instruction set (analogous to x86), for which parsing code lives in
// WebAssemblyAsmParser.
//
// This file contains processing for generic directives implemented using
// MCTargetStreamer, the ones that depend on WebAssemblyTargetStreamer are in
// WebAssemblyAsmParser.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 18-30
```cpp

#include "llvm/ADT/StringExtras.h"
#include "llvm/BinaryFormat/Wasm.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCObjectFileInfo.h"
#include "llvm/MC/MCParser/AsmLexer.h"
#include "llvm/MC/MCParser/MCAsmParser.h"
#include "llvm/MC/MCParser/MCAsmParserExtension.h"
#include "llvm/MC/MCSectionWasm.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/MC/MCSymbolWasm.h"
#include <optional>

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ADT/StringExtras.h`, `llvm/BinaryFormat/Wasm.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCObjectFileInfo.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ADT/StringExtras.h`, `llvm/BinaryFormat/Wasm.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCObjectFileInfo.h`。

### Lines 31-43
```cpp
using namespace llvm;

namespace {

class WasmAsmParser : public MCAsmParserExtension {
  MCAsmParser *Parser = nullptr;
  AsmLexer *Lexer = nullptr;

  template<bool (WasmAsmParser::*HandlerMethod)(StringRef, SMLoc)>
  void addDirectiveHandler(StringRef Directive) {
    MCAsmParser::ExtensionDirectiveHandler Handler = std::make_pair(
        this, HandleDirective<WasmAsmParser, HandlerMethod>);

```
- **EN**: Introduces declarations for `llvm`, `WasmAsmParser`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `WasmAsmParser` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 44-55
```cpp
    getParser().addDirectiveHandler(Directive, Handler);
  }

public:
  WasmAsmParser() { BracketExpressionsSupported = true; }

  void Initialize(MCAsmParser &P) override {
    Parser = &P;
    Lexer = &Parser->getLexer();
    // Call the base implementation.
    this->MCAsmParserExtension::Initialize(*Parser);

```
- **EN**: Implements logic around `getParser`, `WasmAsmParser`, `Initialize`, `getLexer`; this block parses assembly syntax or operands.
- **CN**: 围绕 `getParser`, `WasmAsmParser`, `Initialize`, `getLexer` 实现具体逻辑；这一段解析汇编语法或操作数。

### Lines 56-71
```cpp
    addDirectiveHandler<&WasmAsmParser::parseSectionDirectiveText>(".text");
    addDirectiveHandler<&WasmAsmParser::parseSectionDirectiveData>(".data");
    addDirectiveHandler<&WasmAsmParser::parseSectionDirective>(".section");
    addDirectiveHandler<&WasmAsmParser::parseDirectiveSize>(".size");
    addDirectiveHandler<&WasmAsmParser::parseDirectiveType>(".type");
    addDirectiveHandler<&WasmAsmParser::ParseDirectiveIdent>(".ident");
    addDirectiveHandler<
      &WasmAsmParser::ParseDirectiveSymbolAttribute>(".weak");
    addDirectiveHandler<
      &WasmAsmParser::ParseDirectiveSymbolAttribute>(".local");
    addDirectiveHandler<
      &WasmAsmParser::ParseDirectiveSymbolAttribute>(".internal");
    addDirectiveHandler<
      &WasmAsmParser::ParseDirectiveSymbolAttribute>(".hidden");
  }

```
- **EN**: Implements logic around `parseSectionDirectiveText>`, `parseSectionDirectiveData>`, `parseSectionDirective>`, `parseDirectiveSize>`, and 3 more symbols; this block parses assembly syntax or operands; updates MC section or symbol state.
- **CN**: 围绕 `parseSectionDirectiveText>`, `parseSectionDirectiveData>`, `parseSectionDirective>`, `parseDirectiveSize>`, and 3 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态。

### Lines 72-82
```cpp
  bool error(const StringRef &Msg, const AsmToken &Tok) {
    return Parser->Error(Tok.getLoc(), Msg + Tok.getString());
  }

  bool isNext(AsmToken::TokenKind Kind) {
    auto Ok = Lexer->is(Kind);
    if (Ok)
      Lex();
    return Ok;
  }

```
- **EN**: Implements logic around `error`, `Error`, `isNext`, `is`, and 1 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `error`, `Error`, `isNext`, `is`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 83-94
```cpp
  bool expect(AsmToken::TokenKind Kind, const char *KindName) {
    if (!isNext(Kind))
      return error(std::string("Expected ") + KindName + ", instead got: ",
                   Lexer->getTok());
    return false;
  }

  bool parseSectionDirectiveText(StringRef, SMLoc) {
    // FIXME: .text currently no-op.
    return false;
  }

```
- **EN**: Implements logic around `expect`, `error`, `getTok`, `parseSectionDirectiveText`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `expect`, `error`, `getTok`, `parseSectionDirectiveText` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 95-112
```cpp
  bool parseSectionDirectiveData(StringRef, SMLoc) {
    auto *S = getContext().getObjectFileInfo()->getDataSection();
    getStreamer().switchSection(S);
    return false;
  }

  uint32_t parseSectionFlags(StringRef FlagStr, bool &Passive, bool &Group) {
    uint32_t flags = 0;
    for (char C : FlagStr) {
      switch (C) {
      case 'p':
        Passive = true;
        break;
      case 'G':
        Group = true;
        break;
      case 'T':
        flags |= wasm::WASM_SEG_FLAG_TLS;
```
- **EN**: Implements logic around `parseSectionDirectiveData`, `getContext`, `getStreamer`, `parseSectionFlags`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `parseSectionDirectiveData`, `getContext`, `getStreamer`, `parseSectionFlags` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 113-126
```cpp
        break;
      case 'S':
        flags |= wasm::WASM_SEG_FLAG_STRINGS;
        break;
      case 'R':
        flags |= wasm::WASM_SEG_FLAG_RETAIN;
        break;
      default:
        return -1U;
      }
    }
    return flags;
  }

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 127-144
```cpp
  bool parseGroup(StringRef &GroupName) {
    if (Lexer->isNot(AsmToken::Comma))
      return TokError("expected group name");
    Lex();
    if (Lexer->is(AsmToken::Integer)) {
      GroupName = getTok().getString();
      Lex();
    } else if (Parser->parseIdentifier(GroupName)) {
      return TokError("invalid group name");
    }
    if (Lexer->is(AsmToken::Comma)) {
      Lex();
      StringRef Linkage;
      if (Parser->parseIdentifier(Linkage))
        return TokError("invalid linkage");
      if (Linkage != "comdat")
        return TokError("Linkage must be 'comdat'");
    }
```
- **EN**: Implements logic around `parseGroup`, `TokError`, `Lex`, `getTok`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseGroup`, `TokError`, `Lex`, `getTok` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 145-155
```cpp
    return false;
  }

  bool parseSectionDirective(StringRef, SMLoc loc) {
    StringRef Name;
    if (Parser->parseIdentifier(Name))
      return TokError("expected identifier in directive");

    if (expect(AsmToken::Comma, ","))
      return true;

```
- **EN**: Implements logic around `parseSectionDirective`, `TokError`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseSectionDirective`, `TokError` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 156-172
```cpp
    if (Lexer->isNot(AsmToken::String))
      return error("expected string in directive, instead got: ", Lexer->getTok());

    auto Kind = StringSwitch<std::optional<SectionKind>>(Name)
                    .StartsWith(".data", SectionKind::getData())
                    .StartsWith(".tdata", SectionKind::getThreadData())
                    .StartsWith(".tbss", SectionKind::getThreadBSS())
                    .StartsWith(".rodata", SectionKind::getReadOnly())
                    .StartsWith(".text", SectionKind::getText())
                    .StartsWith(".custom_section", SectionKind::getMetadata())
                    .StartsWith(".bss", SectionKind::getBSS())
                    // See use of .init_array in WasmObjectWriter and
                    // TargetLoweringObjectFileWasm
                    .StartsWith(".init_array", SectionKind::getData())
                    .StartsWith(".debug_", SectionKind::getMetadata())
                    .Default(SectionKind::getData());

```
- **EN**: Implements logic around `error`, `optional<SectionKind>>`, `StartsWith`, `Default`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `error`, `optional<SectionKind>>`, `StartsWith`, `Default` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 173-182
```cpp
    // Update section flags if present in this .section directive
    bool Passive = false;
    bool Group = false;
    uint32_t Flags =
        parseSectionFlags(getTok().getStringContents(), Passive, Group);
    if (Flags == -1U)
      return TokError("unknown flag");

    Lex();

```
- **EN**: Implements logic around `parseSectionFlags`, `TokError`, `Lex`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseSectionFlags`, `TokError`, `Lex` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 183-192
```cpp
    if (expect(AsmToken::Comma, ",") || expect(AsmToken::At, "@"))
      return true;

    StringRef GroupName;
    if (Group && parseGroup(GroupName))
      return true;

    if (expect(AsmToken::EndOfStatement, "eol"))
      return true;

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 193-201
```cpp
    // TODO: Parse UniqueID
    MCSectionWasm *WS = getContext().getWasmSection(
        Name, *Kind, Flags, GroupName, MCSection::NonUniqueID);

    if (WS->getSegmentFlags() != Flags)
      Parser->Error(loc, "changed section flags for " + Name +
                             ", expected: 0x" +
                             utohexstr(WS->getSegmentFlags()));

```
- **EN**: Implements logic around `getContext`, `Error`, `utohexstr`; this block parses assembly syntax or operands; updates MC section or symbol state.
- **CN**: 围绕 `getContext`, `Error`, `utohexstr` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态。

### Lines 202-211
```cpp
    if (Passive) {
      if (!WS->isWasmData())
        return Parser->Error(loc, "Only data sections can be passive");
      WS->setPassive();
    }

    getStreamer().switchSection(WS);
    return false;
  }

```
- **EN**: Implements logic around `Error`, `setPassive`, `getStreamer`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `Error`, `setPassive`, `getStreamer` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 212-229
```cpp
  // TODO: This function is almost the same as ELFAsmParser::ParseDirectiveSize
  // so maybe could be shared somehow.
  bool parseDirectiveSize(StringRef, SMLoc Loc) {
    MCSymbol *Sym;
    if (Parser->parseSymbol(Sym))
      return TokError("expected identifier in directive");
    if (expect(AsmToken::Comma, ","))
      return true;
    const MCExpr *Expr;
    if (Parser->parseExpression(Expr))
      return true;
    if (expect(AsmToken::EndOfStatement, "eol"))
      return true;
    auto WasmSym = static_cast<const MCSymbolWasm *>(Sym);
    if (WasmSym->isFunction()) {
      // Ignore .size directives for function symbols.  They get their size
      // set automatically based on their content.
      Warning(Loc, ".size directive ignored for function symbols");
```
- **EN**: Implements logic around `parseDirectiveSize`, `TokError`, `Warning`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveSize`, `TokError`, `Warning` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 230-247
```cpp
    } else {
      getStreamer().emitELFSize(Sym, Expr);
    }
    return false;
  }

  bool parseDirectiveType(StringRef, SMLoc) {
    // This could be the start of a function, check if followed by
    // "label,@function"
    if (!Lexer->is(AsmToken::Identifier))
      return error("Expected label after .type directive, got: ",
                   Lexer->getTok());
    auto *WasmSym = static_cast<MCSymbolWasm *>(
        getStreamer().getContext().parseSymbol(Lexer->getTok().getString()));
    Lex();
    if (!(isNext(AsmToken::Comma) && isNext(AsmToken::At) &&
          Lexer->is(AsmToken::Identifier)))
      return error("Expected label,@type declaration, got: ", Lexer->getTok());
```
- **EN**: Implements logic around `getStreamer`, `parseDirectiveType`, `error`, `getTok`, and 2 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getStreamer`, `parseDirectiveType`, `error`, `getTok`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 248-264
```cpp
    auto TypeName = Lexer->getTok().getString();
    if (TypeName == "function") {
      WasmSym->setType(wasm::WASM_SYMBOL_TYPE_FUNCTION);
      auto *Current =
          static_cast<MCSectionWasm *>(getStreamer().getCurrentSectionOnly());
      if (Current->getGroup())
        WasmSym->setComdat(true);
    } else if (TypeName == "global")
      WasmSym->setType(wasm::WASM_SYMBOL_TYPE_GLOBAL);
    else if (TypeName == "object")
      WasmSym->setType(wasm::WASM_SYMBOL_TYPE_DATA);
    else
      return error("Unknown WASM symbol type: ", Lexer->getTok());
    Lex();
    return expect(AsmToken::EndOfStatement, "EOL");
  }

```
- **EN**: Implements logic around `getTok`, `setType`, `getStreamer`, `setComdat`, and 3 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getTok`, `setType`, `getStreamer`, `setComdat`, and 3 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 265-279
```cpp
  // FIXME: Shared with ELF.
  /// ParseDirectiveIdent
  ///  ::= .ident string
  bool ParseDirectiveIdent(StringRef, SMLoc) {
    if (getLexer().isNot(AsmToken::String))
      return TokError("unexpected token in '.ident' directive");
    StringRef Data = getTok().getIdentifier();
    Lex();
    if (getLexer().isNot(AsmToken::EndOfStatement))
      return TokError("unexpected token in '.ident' directive");
    Lex();
    getStreamer().emitIdent(Data);
    return false;
  }

```
- **EN**: Implements logic around `ParseDirectiveIdent`, `TokError`, `getTok`, `Lex`, and 1 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `ParseDirectiveIdent`, `TokError`, `getTok`, `Lex`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 280-297
```cpp
  // FIXME: Shared with ELF.
  /// ParseDirectiveSymbolAttribute
  ///  ::= { ".local", ".weak", ... } [ identifier ( , identifier )* ]
  bool ParseDirectiveSymbolAttribute(StringRef Directive, SMLoc) {
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
        MCSymbol *Sym;
        if (getParser().parseSymbol(Sym))
          return TokError("expected identifier in directive");
        getStreamer().emitSymbolAttribute(Sym, Attr);
```
- **EN**: Implements logic around `ParseDirectiveSymbolAttribute`, `StringSwitch<MCSymbolAttr>`, `Case`, `Default`, and 3 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `ParseDirectiveSymbolAttribute`, `StringSwitch<MCSymbolAttr>`, `Case`, `Default`, and 3 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 298-309
```cpp
        if (getLexer().is(AsmToken::EndOfStatement))
          break;
        if (getLexer().isNot(AsmToken::Comma))
          return TokError("unexpected token in directive");
        Lex();
      }
    }
    Lex();
    return false;
  }
};

```
- **EN**: Implements logic around `TokError`, `Lex`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `TokError`, `Lex` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 310-312
```cpp
} // end anonymous namespace

MCAsmParserExtension *llvm::createWasmAsmParser() { return new WasmAsmParser; }
```
- **EN**: Implements logic around `createWasmAsmParser`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `createWasmAsmParser` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

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

- **Direct includes / 直接包含**: `llvm/ADT/StringExtras.h`, `llvm/BinaryFormat/Wasm.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCObjectFileInfo.h`, `llvm/MC/MCParser/AsmLexer.h`, `llvm/MC/MCParser/MCAsmParser.h`, `llvm/MC/MCParser/MCAsmParserExtension.h`, `llvm/MC/MCSectionWasm.h`, `llvm/MC/MCStreamer.h`, `llvm/MC/MCSymbolWasm.h`, `optional`
- **LLVM subsystems / LLVM 子系统**: MC, BinaryFormat
