# MCAsmParser.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/MC/MCParser/MCAsmParser.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements Abstract Asm Parser Interface.
  - **CN**: 实现 LLVM MC 汇编解析、伪指令处理、表达式解析以及面向 streamer 的解析辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- MCAsmParser.cpp - Abstract Asm Parser Interface -------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 8-21
```cpp

#include "llvm/MC/MCParser/MCAsmParser.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/ADT/Twine.h"
#include "llvm/Config/llvm-config.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCParser/AsmLexer.h"
#include "llvm/MC/MCParser/MCParsedAsmOperand.h"
#include "llvm/MC/MCParser/MCTargetAsmParser.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/SMLoc.h"
#include "llvm/Support/raw_ostream.h"
#include <cassert>
```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/MCParser/MCAsmParser.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/Twine.h`, `llvm/Config/llvm-config.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/MCParser/MCAsmParser.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/Twine.h`, `llvm/Config/llvm-config.h`。

### Lines 22-30
```cpp

using namespace llvm;

namespace llvm {
cl::opt<unsigned> AsmMacroMaxNestingDepth(
    "asm-macro-max-nesting-depth", cl::init(20), cl::Hidden,
    cl::desc("The maximum nesting depth allowed for assembly macros."));
}

```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 31-42
```cpp
MCAsmParser::MCAsmParser(MCContext &Ctx, MCStreamer &Out, SourceMgr &SM,
                         const MCAsmInfo &MAI)
    : Ctx(Ctx), Out(Out), SrcMgr(SM), MAI(MAI), Lexer(MAI) {}

MCAsmParser::~MCAsmParser() = default;

void MCAsmParser::setTargetParser(MCTargetAsmParser &P) {
  assert(!TargetParser && "Target parser is already initialized!");
  TargetParser = &P;
  TargetParser->Initialize(*this);
}

```
- **EN**: Implements logic around `MCAsmParser`, `Ctx`, `~MCAsmParser`, `setTargetParser`, and 2 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `MCAsmParser`, `Ctx`, `~MCAsmParser`, `setTargetParser`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为。

### Lines 43-51
```cpp
const AsmToken &MCAsmParser::getTok() const {
  return getLexer().getTok();
}

bool MCAsmParser::parseTokenLoc(SMLoc &Loc) {
  Loc = getTok().getLoc();
  return false;
}

```
- **EN**: Implements logic around `getTok`, `getLexer`, `parseTokenLoc`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `getTok`, `getLexer`, `parseTokenLoc` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 52-58
```cpp
bool MCAsmParser::parseEOL() {
  if (getTok().getKind() != AsmToken::EndOfStatement)
    return Error(getTok().getLoc(), "expected newline");
  Lex();
  return false;
}

```
- **EN**: Implements logic around `parseEOL`, `Error`, `Lex`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseEOL`, `Error`, `Lex` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 59-65
```cpp
bool MCAsmParser::parseEOL(const Twine &Msg) {
  if (getTok().getKind() != AsmToken::EndOfStatement)
    return Error(getTok().getLoc(), Msg);
  Lex();
  return false;
}

```
- **EN**: Implements logic around `parseEOL`, `Error`, `Lex`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseEOL`, `Error`, `Lex` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 66-74
```cpp
bool MCAsmParser::parseToken(AsmToken::TokenKind T, const Twine &Msg) {
  if (T == AsmToken::EndOfStatement)
    return parseEOL(Msg);
  if (getTok().getKind() != T)
    return Error(getTok().getLoc(), Msg);
  Lex();
  return false;
}

```
- **EN**: Implements logic around `parseToken`, `parseEOL`, `Error`, `Lex`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseToken`, `parseEOL`, `Error`, `Lex` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 75-82
```cpp
bool MCAsmParser::parseIntToken(int64_t &V, const Twine &Msg) {
  if (getTok().getKind() != AsmToken::Integer)
    return TokError(Msg);
  V = getTok().getIntVal();
  Lex();
  return false;
}

```
- **EN**: Implements logic around `parseIntToken`, `TokError`, `getTok`, `Lex`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseIntToken`, `TokError`, `getTok`, `Lex` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 83-89
```cpp
bool MCAsmParser::parseOptionalToken(AsmToken::TokenKind T) {
  bool Present = (getTok().getKind() == T);
  if (Present)
    parseToken(T);
  return Present;
}

```
- **EN**: Implements logic around `parseOptionalToken`, `getTok`, `parseToken`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseOptionalToken`, `getTok`, `parseToken` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 90-99
```cpp
bool MCAsmParser::check(bool P, const Twine &Msg) {
  return check(P, getTok().getLoc(), Msg);
}

bool MCAsmParser::check(bool P, SMLoc Loc, const Twine &Msg) {
  if (P)
    return Error(Loc, Msg);
  return false;
}

```
- **EN**: Implements logic around `check`, `Error`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `check`, `Error` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 100-110
```cpp
bool MCAsmParser::TokError(const Twine &Msg, SMRange Range) {
  return Error(getLexer().getLoc(), Msg, Range);
}

bool MCAsmParser::Error(SMLoc L, const Twine &Msg, SMRange Range) {
  MCPendingError PErr;
  PErr.Loc = L;
  Msg.toVector(PErr.Msg);
  PErr.Range = Range;
  PendingErrors.push_back(PErr);

```
- **EN**: Implements logic around `TokError`, `Error`, `toVector`, `push_back`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `TokError`, `Error`, `toVector`, `push_back` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 111-118
```cpp
  // If we threw this parsing error after a lexing error, this should
  // supercede the lexing error and so we remove it from the Lexer
  // before it can propagate
  if (getTok().is(AsmToken::Error))
    getLexer().Lex();
  return true;
}

```
- **EN**: Implements logic around `getLexer`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `getLexer` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 119-127
```cpp
bool MCAsmParser::addErrorSuffix(const Twine &Suffix) {
  // Make sure lexing errors have propagated to the parser.
  if (getTok().is(AsmToken::Error))
    Lex();
  for (auto &PErr : PendingErrors)
    Suffix.toVector(PErr.Msg);
  return true;
}

```
- **EN**: Implements logic around `addErrorSuffix`, `Lex`, `toVector`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `addErrorSuffix`, `Lex`, `toVector` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 128-141
```cpp
bool MCAsmParser::parseMany(function_ref<bool()> parseOne, bool hasComma) {
  if (parseOptionalToken(AsmToken::EndOfStatement))
    return false;
  while (true) {
    if (parseOne())
      return true;
    if (parseOptionalToken(AsmToken::EndOfStatement))
      return false;
    if (hasComma && parseToken(AsmToken::Comma))
      return true;
  }
  return false;
}

```
- **EN**: Implements logic around `parseMany`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseMany` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 142-155
```cpp
bool MCAsmParser::parseExpression(const MCExpr *&Res) {
  SMLoc L;
  return parseExpression(Res, L);
}

bool MCAsmParser::parseGNUAttribute(SMLoc L, int64_t &Tag,
                                    int64_t &IntegerValue) {
  // Parse a .gnu_attribute with numerical tag and value.
  StringRef S(L.getPointer());
  SMLoc TagLoc;
  TagLoc = getTok().getLoc();
  const AsmToken &Tok = getTok();
  if (Tok.isNot(AsmToken::Integer))
    return false;
```
- **EN**: Implements logic around `parseExpression`, `parseGNUAttribute`, `S`, `getTok`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseExpression`, `parseGNUAttribute`, `S`, `getTok` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 156-165
```cpp
  Tag = Tok.getIntVal();
  Lex(); // Eat the Tag
  Lex(); // Eat the comma
  if (Tok.isNot(AsmToken::Integer))
    return false;
  IntegerValue = Tok.getIntVal();
  Lex(); // Eat the IntegerValue
  return true;
}

```
- **EN**: Implements logic around `getIntVal`, `Lex`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `getIntVal`, `Lex` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 166-174
```cpp
bool MCAsmParser::parseSymbol(MCSymbol *&Res) {
  StringRef Name;
  if (parseIdentifier(Name))
    return true;

  Res = getContext().parseSymbol(Name);
  return false;
}

```
- **EN**: Implements logic around `parseSymbol`, `getContext`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseSymbol`, `getContext` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 175-182
```cpp
void MCParsedAsmOperand::dump() const {
  // Cannot completely remove virtual function even in release mode.
#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
  MCTargetOptions Opts;
  dbgs() << "  ";
  print(dbgs(), MCAsmInfo(Opts));
#endif
}
```
- **EN**: Implements logic around `dump`, `dbgs`, `print`; this block parses assembly syntax or operands.
- **CN**: 围绕 `dump`, `dbgs`, `print` 实现具体逻辑；这一段解析汇编语法或操作数。

## Key Concepts / 关键概念

- **Assembly parsing / 汇编解析**:
  - **EN**: Tokenizes assembly text, interprets directives, and builds MC expressions or instructions
  - **CN**: 对汇编文本做词法与语法解析，并构建 MC 表达式或指令
- **Directive handling / 伪指令处理**:
  - **EN**: Interprets assembler directives and maps them to streamer operations
  - **CN**: 解释汇编伪指令并把它们映射为 streamer 操作
- **MC instruction model / MC 指令模型**:
  - **EN**: Uses MCInst/MCOperand abstractions to represent machine instructions generically
  - **CN**: 使用 MCInst/MCOperand 抽象以通用方式表示机器指令
- **Debug/unwind metadata / 调试与展开元数据**:
  - **EN**: Emits or manages metadata needed for debugging and stack unwinding
  - **CN**: 输出或管理调试与栈展开所需的元数据

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/MC/MCParser/MCAsmParser.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/Twine.h`, `llvm/Config/llvm-config.h`, `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCParser/AsmLexer.h`, `llvm/MC/MCParser/MCParsedAsmOperand.h`, `llvm/MC/MCParser/MCTargetAsmParser.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/SMLoc.h`, `llvm/Support/raw_ostream.h` ... (+1 more)
- **LLVM subsystems / LLVM 子系统**: MC, Support
