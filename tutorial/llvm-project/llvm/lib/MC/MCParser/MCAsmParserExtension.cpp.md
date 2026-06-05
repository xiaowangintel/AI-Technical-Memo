# MCAsmParserExtension.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/MC/MCParser/MCAsmParserExtension.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements Asm Parser Hooks.
  - **CN**: 实现 LLVM MC 汇编解析、伪指令处理、表达式解析以及面向 streamer 的解析辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- MCAsmParserExtension.cpp - Asm Parser Hooks ------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 8-14
```cpp

#include "llvm/MC/MCParser/MCAsmParserExtension.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCParser/AsmLexer.h"
#include "llvm/MC/MCStreamer.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/MCParser/MCAsmParserExtension.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCParser/AsmLexer.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/MCParser/MCAsmParserExtension.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCParser/AsmLexer.h`。

### Lines 15-20
```cpp
using namespace llvm;

MCAsmParserExtension::MCAsmParserExtension() = default;

MCAsmParserExtension::~MCAsmParserExtension() = default;

```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 21-25
```cpp
void MCAsmParserExtension::Initialize(MCAsmParser &Parser) {
  this->Parser = &Parser;
}

/// parseDirectiveCGProfile
```
- **EN**: Implements logic around `Initialize`; this block parses assembly syntax or operands.
- **CN**: 围绕 `Initialize` 实现具体逻辑；这一段解析汇编语法或操作数。

### Lines 26-32
```cpp
///  ::= .cg_profile identifier, identifier, <number>
bool MCAsmParserExtension::parseDirectiveCGProfile(StringRef, SMLoc) {
  StringRef From;
  SMLoc FromLoc = getLexer().getLoc();
  if (getParser().parseIdentifier(From))
    return TokError("expected identifier in directive");

```
- **EN**: Implements logic around `parseDirectiveCGProfile`, `getLexer`, `TokError`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveCGProfile`, `getLexer`, `TokError` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 33-41
```cpp
  if (getLexer().isNot(AsmToken::Comma))
    return TokError("expected a comma");
  Lex();

  StringRef To;
  SMLoc ToLoc = getLexer().getLoc();
  if (getParser().parseIdentifier(To))
    return TokError("expected identifier in directive");

```
- **EN**: Implements logic around `TokError`, `Lex`, `getLexer`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `TokError`, `Lex`, `getLexer` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 42-49
```cpp
  if (getLexer().isNot(AsmToken::Comma))
    return TokError("expected a comma");
  Lex();

  int64_t Count;
  if (getParser().parseIntToken(Count))
    return true;

```
- **EN**: Implements logic around `TokError`, `Lex`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `TokError`, `Lex` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 50-55
```cpp
  if (getLexer().isNot(AsmToken::EndOfStatement))
    return TokError("unexpected token in directive");

  MCSymbol *FromSym = getContext().parseSymbol(From);
  MCSymbol *ToSym = getContext().parseSymbol(To);

```
- **EN**: Implements logic around `TokError`, `getContext`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `TokError`, `getContext` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 56-61
```cpp
  getStreamer().emitCGProfileEntry(
      MCSymbolRefExpr::create(FromSym, getContext(), FromLoc),
      MCSymbolRefExpr::create(ToSym, getContext(), ToLoc), Count);
  return false;
}

```
- **EN**: Implements logic around `getStreamer`, `create`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getStreamer`, `create` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 62-71
```cpp
bool MCAsmParserExtension::maybeParseUniqueID(int64_t &UniqueID) {
  AsmLexer &L = getLexer();
  if (L.isNot(AsmToken::Comma))
    return false;
  Lex();
  StringRef UniqueStr;
  if (getParser().parseIdentifier(UniqueStr))
    return TokError("expected identifier");
  if (UniqueStr != "unique")
    return TokError("expected 'unique'");
```
- **EN**: Implements logic around `maybeParseUniqueID`, `getLexer`, `Lex`, `TokError`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `maybeParseUniqueID`, `getLexer`, `Lex`, `TokError` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 72-81
```cpp
  if (L.isNot(AsmToken::Comma))
    return TokError("expected commma");
  Lex();
  if (getParser().parseAbsoluteExpression(UniqueID))
    return true;
  if (UniqueID < 0)
    return TokError("unique id must be positive");
  if (!isUInt<32>(UniqueID) || UniqueID == ~0U)
    return TokError("unique id is too large");
  return false;
```
- **EN**: Implements logic around `TokError`, `Lex`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `TokError`, `Lex` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 82-82
```cpp
}
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

## Key Concepts / 关键概念

- **Assembly parsing / 汇编解析**:
  - **EN**: Tokenizes assembly text, interprets directives, and builds MC expressions or instructions
  - **CN**: 对汇编文本做词法与语法解析，并构建 MC 表达式或指令
- **Directive handling / 伪指令处理**:
  - **EN**: Interprets assembler directives and maps them to streamer operations
  - **CN**: 解释汇编伪指令并把它们映射为 streamer 操作

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/MC/MCParser/MCAsmParserExtension.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCParser/AsmLexer.h`, `llvm/MC/MCStreamer.h`
- **LLVM subsystems / LLVM 子系统**: MC
