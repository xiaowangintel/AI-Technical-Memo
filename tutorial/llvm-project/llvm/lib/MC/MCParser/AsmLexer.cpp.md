# AsmLexer.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/MC/MCParser/AsmLexer.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements Lexer for Assembly Files.
  - **CN**: 实现 LLVM MC 汇编解析、伪指令处理、表达式解析以及面向 streamer 的解析辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28
```cpp
//===- AsmLexer.cpp - Lexer for Assembly Files ----------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This class implements the lexer for assembly files.
//
//===----------------------------------------------------------------------===//

#include "llvm/MC/MCParser/AsmLexer.h"
#include "llvm/ADT/APInt.h"
#include "llvm/ADT/ArrayRef.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/Support/Compiler.h"
#include "llvm/Support/SMLoc.h"
#include "llvm/Support/SaveAndRestore.h"
#include "llvm/Support/raw_ostream.h"
#include <cassert>
#include <cctype>
#include <cstdio>
#include <cstring>
#include <string>

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/MCParser/AsmLexer.h`, `llvm/ADT/APInt.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/StringExtras.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/MCParser/AsmLexer.h`, `llvm/ADT/APInt.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/StringExtras.h`。

### Lines 29-56
```cpp
using namespace llvm;

SMLoc AsmToken::getLoc() const { return SMLoc::getFromPointer(Str.data()); }

SMLoc AsmToken::getEndLoc() const {
  return SMLoc::getFromPointer(Str.data() + Str.size());
}

SMRange AsmToken::getLocRange() const { return SMRange(getLoc(), getEndLoc()); }

void AsmToken::dump(raw_ostream &OS) const {
  switch (Kind) {
  case AsmToken::Error:
    OS << "error";
    break;
  case AsmToken::Identifier:
    OS << "identifier: " << getString();
    break;
  case AsmToken::Integer:
    OS << "int: " << getString();
    break;
  case AsmToken::Real:
    OS << "real: " << getString();
    break;
  case AsmToken::String:
    OS << "string: " << getString();
    break;

```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 57-88
```cpp
    // clang-format off
  case AsmToken::Amp:                OS << "Amp"; break;
  case AsmToken::AmpAmp:             OS << "AmpAmp"; break;
  case AsmToken::At:                 OS << "At"; break;
  case AsmToken::BackSlash:          OS << "BackSlash"; break;
  case AsmToken::BigNum:             OS << "BigNum"; break;
  case AsmToken::Caret:              OS << "Caret"; break;
  case AsmToken::Colon:              OS << "Colon"; break;
  case AsmToken::Comma:              OS << "Comma"; break;
  case AsmToken::Comment:            OS << "Comment"; break;
  case AsmToken::Dollar:             OS << "Dollar"; break;
  case AsmToken::Dot:                OS << "Dot"; break;
  case AsmToken::EndOfStatement:     OS << "EndOfStatement"; break;
  case AsmToken::Eof:                OS << "Eof"; break;
  case AsmToken::Equal:              OS << "Equal"; break;
  case AsmToken::EqualEqual:         OS << "EqualEqual"; break;
  case AsmToken::Exclaim:            OS << "Exclaim"; break;
  case AsmToken::ExclaimEqual:       OS << "ExclaimEqual"; break;
  case AsmToken::Greater:            OS << "Greater"; break;
  case AsmToken::GreaterEqual:       OS << "GreaterEqual"; break;
  case AsmToken::GreaterGreater:     OS << "GreaterGreater"; break;
  case AsmToken::Hash:               OS << "Hash"; break;
  case AsmToken::HashDirective:      OS << "HashDirective"; break;
  case AsmToken::LBrac:              OS << "LBrac"; break;
  case AsmToken::LCurly:             OS << "LCurly"; break;
  case AsmToken::LParen:             OS << "LParen"; break;
  case AsmToken::Less:               OS << "Less"; break;
  case AsmToken::LessEqual:          OS << "LessEqual"; break;
  case AsmToken::LessGreater:        OS << "LessGreater"; break;
  case AsmToken::LessLess:           OS << "LessLess"; break;
  case AsmToken::Minus:              OS << "Minus"; break;
  case AsmToken::MinusGreater:       OS << "MinusGreater"; break;
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 89-109
```cpp
  case AsmToken::Percent:            OS << "Percent"; break;
  case AsmToken::Pipe:               OS << "Pipe"; break;
  case AsmToken::PipePipe:           OS << "PipePipe"; break;
  case AsmToken::Plus:               OS << "Plus"; break;
  case AsmToken::Question:           OS << "Question"; break;
  case AsmToken::RBrac:              OS << "RBrac"; break;
  case AsmToken::RCurly:             OS << "RCurly"; break;
  case AsmToken::RParen:             OS << "RParen"; break;
  case AsmToken::Slash:              OS << "Slash"; break;
  case AsmToken::Space:              OS << "Space"; break;
  case AsmToken::Star:               OS << "Star"; break;
  case AsmToken::Tilde:              OS << "Tilde"; break;
    // clang-format on
  }

  // Print the token string.
  OS << " (\"";
  OS.write_escaped(getString());
  OS << "\")";
}

```
- **EN**: Implements logic around `write_escaped`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `write_escaped` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为。

### Lines 110-126
```cpp
AsmLexer::AsmLexer(const MCAsmInfo &MAI) : MAI(MAI) {
  // For COFF targets, this is true, while for ELF targets, it should be false.
  // Currently, @specifier parsing depends on '@' being included in the token.
  AllowAtInIdentifier = !StringRef(MAI.getCommentString()).starts_with("@") &&
                        MAI.useAtForSpecifier();
  LexMotorolaIntegers = MAI.shouldUseMotorolaIntegers();

  CurTok.emplace_back(AsmToken::Space, StringRef());
}

void AsmLexer::setBuffer(StringRef Buf, const char *ptr,
                         bool EndStatementAtEOF) {
  // Buffer must be NULL-terminated. NULL terminator must reside at `Buf.end()`.
  // It must be safe to dereference `Buf.end()`.
  assert(*Buf.end() == '\0' &&
         "Buffer provided to AsmLexer lacks null terminator.");

```
- **EN**: Implements logic around `AsmLexer`, `StringRef`, `useAtForSpecifier`, `shouldUseMotorolaIntegers`, and 3 more symbols; this block parses assembly syntax or operands.
- **CN**: 围绕 `AsmLexer`, `StringRef`, `useAtForSpecifier`, `shouldUseMotorolaIntegers`, and 3 more symbols 实现具体逻辑；这一段解析汇编语法或操作数。

### Lines 127-142
```cpp
  CurBuf = Buf;

  if (ptr)
    CurPtr = ptr;
  else
    CurPtr = CurBuf.begin();

  TokStart = nullptr;
  this->EndStatementAtEOF = EndStatementAtEOF;
}

/// ReturnError - Set the error to the specified string at the specified
/// location.  This is defined to always return AsmToken::Error.
AsmToken AsmLexer::ReturnError(const char *Loc, const std::string &Msg) {
  SetError(SMLoc::getFromPointer(Loc), Msg);

```
- **EN**: Implements logic around `begin`, `ReturnError`, `SetError`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `begin`, `ReturnError`, `SetError` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 143-158
```cpp
  return AsmToken(AsmToken::Error, StringRef(Loc, CurPtr - Loc));
}

int AsmLexer::getNextChar() {
  if (CurPtr == CurBuf.end())
    return EOF;
  return (unsigned char)*CurPtr++;
}

int AsmLexer::peekNextChar() {
  if (CurPtr == CurBuf.end())
    return EOF;
  return (unsigned char)*CurPtr;
}

/// The leading integral digit sequence and dot should have already been
```
- **EN**: Implements logic around `AsmToken`, `getNextChar`, `peekNextChar`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `AsmToken`, `getNextChar`, `peekNextChar` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 159-175
```cpp
/// consumed, some or all of the fractional digit sequence *can* have been
/// consumed.
AsmToken AsmLexer::LexFloatLiteral() {
  // Skip the fractional digit sequence.
  while (isDigit(*CurPtr))
    ++CurPtr;

  if (*CurPtr == '-' || *CurPtr == '+')
    return ReturnError(CurPtr, "invalid sign in float literal");

  // Check for exponent
  if ((*CurPtr == 'e' || *CurPtr == 'E')) {
    ++CurPtr;

    if (*CurPtr == '-' || *CurPtr == '+')
      ++CurPtr;

```
- **EN**: Implements logic around `LexFloatLiteral`, `ReturnError`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `LexFloatLiteral`, `ReturnError` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 176-194
```cpp
    while (isDigit(*CurPtr))
      ++CurPtr;
  }

  return AsmToken(AsmToken::Real,
                  StringRef(TokStart, CurPtr - TokStart));
}

/// LexHexFloatLiteral matches essentially (.[0-9a-fA-F]*)?[pP][+-]?[0-9a-fA-F]+
/// while making sure there are enough actual digits around for the constant to
/// be valid.
///
/// The leading "0x[0-9a-fA-F]*" (i.e. integer part) has already been consumed
/// before we get here.
AsmToken AsmLexer::LexHexFloatLiteral(bool NoIntDigits) {
  assert((*CurPtr == 'p' || *CurPtr == 'P' || *CurPtr == '.') &&
         "unexpected parse state in floating hex");
  bool NoFracDigits = true;

```
- **EN**: Implements logic around `AsmToken`, `StringRef`, `LexHexFloatLiteral`, `assert`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `AsmToken`, `StringRef`, `LexHexFloatLiteral`, `assert` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 195-215
```cpp
  // Skip the fractional part if there is one
  if (*CurPtr == '.') {
    ++CurPtr;

    const char *FracStart = CurPtr;
    while (isHexDigit(*CurPtr))
      ++CurPtr;

    NoFracDigits = CurPtr == FracStart;
  }

  if (NoIntDigits && NoFracDigits)
    return ReturnError(TokStart, "invalid hexadecimal floating-point constant: "
                                 "expected at least one significand digit");

  // Make sure we do have some kind of proper exponent part
  if (*CurPtr != 'p' && *CurPtr != 'P')
    return ReturnError(TokStart, "invalid hexadecimal floating-point constant: "
                                 "expected exponent part 'p'");
  ++CurPtr;

```
- **EN**: Implements logic around `ReturnError`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `ReturnError` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 216-231
```cpp
  if (*CurPtr == '+' || *CurPtr == '-')
    ++CurPtr;

  // N.b. exponent digits are *not* hex
  const char *ExpStart = CurPtr;
  while (isDigit(*CurPtr))
    ++CurPtr;

  if (CurPtr == ExpStart)
    return ReturnError(TokStart, "invalid hexadecimal floating-point constant: "
                                 "expected at least one exponent digit");

  return AsmToken(AsmToken::Real, StringRef(TokStart, CurPtr - TokStart));
}

/// LexIdentifier: [a-zA-Z_$.@?][a-zA-Z0-9_$.@#?]*
```
- **EN**: Implements logic around `ReturnError`, `AsmToken`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `ReturnError`, `AsmToken` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 232-249
```cpp
static bool isIdentifierChar(char C, bool AllowAt, bool AllowHash) {
  return isAlnum(C) || C == '_' || C == '$' || C == '.' || C == '?' ||
         (AllowAt && C == '@') || (AllowHash && C == '#');
}

AsmToken AsmLexer::LexIdentifier() {
  // Check for floating point literals.
  if (CurPtr[-1] == '.' && isDigit(*CurPtr)) {
    // Disambiguate a .1243foo identifier from a floating literal.
    while (isDigit(*CurPtr))
      ++CurPtr;

    if (!isIdentifierChar(*CurPtr, AllowAtInIdentifier,
                          AllowHashInIdentifier) ||
        *CurPtr == 'e' || *CurPtr == 'E')
      return LexFloatLiteral();
  }

```
- **EN**: Implements logic around `isIdentifierChar`, `isAlnum`, `LexIdentifier`, `LexFloatLiteral`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `isIdentifierChar`, `isAlnum`, `LexIdentifier`, `LexFloatLiteral` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 250-268
```cpp
  while (isIdentifierChar(*CurPtr, AllowAtInIdentifier, AllowHashInIdentifier))
    ++CurPtr;

  // Handle . as a special case.
  if (CurPtr == TokStart+1 && TokStart[0] == '.')
    return AsmToken(AsmToken::Dot, StringRef(TokStart, 1));

  return AsmToken(AsmToken::Identifier, StringRef(TokStart, CurPtr - TokStart));
}

/// LexSlash: Slash: /
///           C-Style Comment: /* ... */
///           C-style Comment: // ...
AsmToken AsmLexer::LexSlash() {
  if (!MAI.shouldAllowAdditionalComments()) {
    IsAtStartOfStatement = false;
    return AsmToken(AsmToken::Slash, StringRef(TokStart, 1));
  }

```
- **EN**: Implements logic around `AsmToken`, `LexSlash`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `AsmToken`, `LexSlash` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 269-300
```cpp
  switch (*CurPtr) {
  case '*':
    IsAtStartOfStatement = false;
    break; // C style comment.
  case '/':
    ++CurPtr;
    return LexLineComment();
  default:
    IsAtStartOfStatement = false;
    return AsmToken(AsmToken::Slash, StringRef(TokStart, 1));
  }

  // C Style comment.
  ++CurPtr;  // skip the star.
  const char *CommentTextStart = CurPtr;
  while (CurPtr != CurBuf.end()) {
    switch (*CurPtr++) {
    case '*':
      // End of the comment?
      if (*CurPtr != '/')
        break;
      // If we have a CommentConsumer, notify it about the comment.
      if (CommentConsumer) {
        CommentConsumer->HandleComment(
            SMLoc::getFromPointer(CommentTextStart),
            StringRef(CommentTextStart, CurPtr - 1 - CommentTextStart));
      }
      ++CurPtr;   // End the */.
      return AsmToken(AsmToken::Comment,
                      StringRef(TokStart, CurPtr - TokStart));
    }
  }
```
- **EN**: Implements logic around `LexLineComment`, `AsmToken`, `HandleComment`, `getFromPointer`, and 1 more symbols; this block parses assembly syntax or operands; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `LexLineComment`, `AsmToken`, `HandleComment`, `getFromPointer`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 301-318
```cpp
  return ReturnError(TokStart, "unterminated comment");
}

/// LexLineComment: Comment: #[^\n]*
///                        : //[^\n]*
AsmToken AsmLexer::LexLineComment() {
  // Mark This as an end of statement with a body of the
  // comment. While it would be nicer to leave this two tokens,
  // backwards compatability with TargetParsers makes keeping this in this form
  // better.
  const char *CommentTextStart = CurPtr;
  int CurChar = getNextChar();
  while (CurChar != '\n' && CurChar != '\r' && CurChar != EOF)
    CurChar = getNextChar();
  const char *NewlinePtr = CurPtr;
  if (CurChar == '\r' && CurPtr != CurBuf.end() && *CurPtr == '\n')
    ++CurPtr;

```
- **EN**: Implements logic around `ReturnError`, `LexLineComment`, `getNextChar`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `ReturnError`, `LexLineComment`, `getNextChar` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 319-336
```cpp
  // If we have a CommentConsumer, notify it about the comment.
  if (CommentConsumer) {
    CommentConsumer->HandleComment(
        SMLoc::getFromPointer(CommentTextStart),
        StringRef(CommentTextStart, NewlinePtr - 1 - CommentTextStart));
  }

  IsAtStartOfLine = true;
  // This is a whole line comment. leave newline
  if (IsAtStartOfStatement)
    return AsmToken(AsmToken::EndOfStatement,
                    StringRef(TokStart, CurPtr - TokStart));
  IsAtStartOfStatement = true;

  return AsmToken(AsmToken::EndOfStatement,
                  StringRef(TokStart, CurPtr - 1 - TokStart));
}

```
- **EN**: Implements logic around `HandleComment`, `getFromPointer`, `StringRef`, `AsmToken`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `HandleComment`, `getFromPointer`, `StringRef`, `AsmToken` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 337-359
```cpp
static void SkipIgnoredIntegerSuffix(const char *&CurPtr) {
  // Skip case-insensitive ULL, UL, U, L and LL suffixes.
  if (CurPtr[0] == 'U' || CurPtr[0] == 'u')
    ++CurPtr;
  if (CurPtr[0] == 'L' || CurPtr[0] == 'l')
    ++CurPtr;
  if (CurPtr[0] == 'L' || CurPtr[0] == 'l')
    ++CurPtr;
}

// Look ahead to search for first non-hex digit, if it's [hH], then we treat the
// integer as a hexadecimal, possibly with leading zeroes.
static unsigned doHexLookAhead(const char *&CurPtr, unsigned DefaultRadix,
                               bool LexHex) {
  const char *FirstNonDec = nullptr;
  const char *LookAhead = CurPtr;
  while (true) {
    if (isDigit(*LookAhead)) {
      ++LookAhead;
    } else {
      if (!FirstNonDec)
        FirstNonDec = LookAhead;

```
- **EN**: Implements logic around `SkipIgnoredIntegerSuffix`, `doHexLookAhead`.
- **CN**: 围绕 `SkipIgnoredIntegerSuffix`, `doHexLookAhead` 实现具体逻辑。

### Lines 360-380
```cpp
      // Keep going if we are looking for a 'h' suffix.
      if (LexHex && isHexDigit(*LookAhead))
        ++LookAhead;
      else
        break;
    }
  }
  bool isHex = LexHex && (*LookAhead == 'h' || *LookAhead == 'H');
  CurPtr = isHex || !FirstNonDec ? LookAhead : FirstNonDec;
  if (isHex)
    return 16;
  return DefaultRadix;
}

static const char *findLastDigit(const char *CurPtr, unsigned DefaultRadix) {
  while (hexDigitValue(*CurPtr) < DefaultRadix) {
    ++CurPtr;
  }
  return CurPtr;
}

```
- **EN**: Implements logic around `findLastDigit`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `findLastDigit` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 381-401
```cpp
static AsmToken intToken(StringRef Ref, APInt &Value) {
  if (Value.isIntN(64))
    return AsmToken(AsmToken::Integer, Ref, Value);
  return AsmToken(AsmToken::BigNum, Ref, Value);
}

static std::string radixName(unsigned Radix) {
  switch (Radix) {
  case 2:
    return "binary";
  case 8:
    return "octal";
  case 10:
    return "decimal";
  case 16:
    return "hexadecimal";
  default:
    return "base-" + std::to_string(Radix);
  }
}

```
- **EN**: Implements logic around `intToken`, `AsmToken`, `radixName`, `to_string`; this block parses assembly syntax or operands; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `intToken`, `AsmToken`, `radixName`, `to_string` 实现具体逻辑；这一段解析汇编语法或操作数，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 402-433
```cpp
/// LexDigit: First character is [0-9].
///   Local Label: [0-9][:]
///   Forward/Backward Label: [0-9][fb]
///   Binary integer: 0b[01]+
///   Octal integer: 0[0-7]+
///   Hex integer: 0x[0-9a-fA-F]+ or [0x]?[0-9][0-9a-fA-F]*[hH]
///   Decimal integer: [1-9][0-9]*
AsmToken AsmLexer::LexDigit() {
  // MASM-flavor binary integer: [01]+[yY] (if DefaultRadix < 16, [bByY])
  // MASM-flavor octal integer: [0-7]+[oOqQ]
  // MASM-flavor decimal integer: [0-9]+[tT] (if DefaultRadix < 16, [dDtT])
  // MASM-flavor hexadecimal integer: [0-9][0-9a-fA-F]*[hH]
  if (LexMasmIntegers && isdigit(CurPtr[-1])) {
    const char *FirstNonBinary =
        (CurPtr[-1] != '0' && CurPtr[-1] != '1') ? CurPtr - 1 : nullptr;
    const char *FirstNonDecimal =
        (CurPtr[-1] < '0' || CurPtr[-1] > '9') ? CurPtr - 1 : nullptr;
    const char *OldCurPtr = CurPtr;
    while (isHexDigit(*CurPtr)) {
      switch (*CurPtr) {
      default:
        if (!FirstNonDecimal) {
          FirstNonDecimal = CurPtr;
        }
        [[fallthrough]];
      case '9':
      case '8':
      case '7':
      case '6':
      case '5':
      case '4':
      case '3':
```
- **EN**: Implements logic around `LexDigit`; this block parses assembly syntax or operands; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `LexDigit` 实现具体逻辑；这一段解析汇编语法或操作数，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 434-451
```cpp
      case '2':
        if (!FirstNonBinary) {
          FirstNonBinary = CurPtr;
        }
        break;
      case '1':
      case '0':
        break;
      }
      ++CurPtr;
    }
    if (*CurPtr == '.') {
      // MASM float literals (other than hex floats) always contain a ".", and
      // are always written in decimal.
      ++CurPtr;
      return LexFloatLiteral();
    }

```
- **EN**: Implements logic around `LexFloatLiteral`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `LexFloatLiteral` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 452-483
```cpp
    if (LexMasmHexFloats && (*CurPtr == 'r' || *CurPtr == 'R')) {
      ++CurPtr;
      return AsmToken(AsmToken::Real, StringRef(TokStart, CurPtr - TokStart));
    }

    unsigned Radix = 0;
    if (*CurPtr == 'h' || *CurPtr == 'H') {
      // hexadecimal number
      ++CurPtr;
      Radix = 16;
    } else if (*CurPtr == 't' || *CurPtr == 'T') {
      // decimal number
      ++CurPtr;
      Radix = 10;
    } else if (*CurPtr == 'o' || *CurPtr == 'O' || *CurPtr == 'q' ||
               *CurPtr == 'Q') {
      // octal number
      ++CurPtr;
      Radix = 8;
    } else if (*CurPtr == 'y' || *CurPtr == 'Y') {
      // binary number
      ++CurPtr;
      Radix = 2;
    } else if (FirstNonDecimal && FirstNonDecimal + 1 == CurPtr &&
               DefaultRadix < 14 &&
               (*FirstNonDecimal == 'd' || *FirstNonDecimal == 'D')) {
      Radix = 10;
    } else if (FirstNonBinary && FirstNonBinary + 1 == CurPtr &&
               DefaultRadix < 12 &&
               (*FirstNonBinary == 'b' || *FirstNonBinary == 'B')) {
      Radix = 2;
    }
```
- **EN**: Implements logic around `AsmToken`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `AsmToken` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 484-501
```cpp

    if (Radix) {
      StringRef Result(TokStart, CurPtr - TokStart);
      APInt Value(128, 0, true);

      if (Result.drop_back().getAsInteger(Radix, Value))
        return ReturnError(TokStart, "invalid " + radixName(Radix) + " number");

      // MSVC accepts and ignores type suffices on integer literals.
      SkipIgnoredIntegerSuffix(CurPtr);

      return intToken(Result, Value);
    }

    // default-radix integers, or floating point numbers, fall through
    CurPtr = OldCurPtr;
  }

```
- **EN**: Implements logic around `Result`, `Value`, `ReturnError`, `SkipIgnoredIntegerSuffix`, and 1 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `Result`, `Value`, `ReturnError`, `SkipIgnoredIntegerSuffix`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 502-522
```cpp
  // MASM default-radix integers: [0-9a-fA-F]+
  // (All other integer literals have a radix specifier.)
  if (LexMasmIntegers && UseMasmDefaultRadix) {
    CurPtr = findLastDigit(CurPtr, 16);
    StringRef Result(TokStart, CurPtr - TokStart);

    APInt Value(128, 0, true);
    if (Result.getAsInteger(DefaultRadix, Value)) {
      return ReturnError(TokStart,
                         "invalid " + radixName(DefaultRadix) + " number");
    }

    return intToken(Result, Value);
  }

  // Motorola hex integers: $[0-9a-fA-F]+
  if (LexMotorolaIntegers && CurPtr[-1] == '$') {
    const char *NumStart = CurPtr;
    while (isHexDigit(CurPtr[0]))
      ++CurPtr;

```
- **EN**: Implements logic around `findLastDigit`, `Result`, `Value`, `ReturnError`, and 2 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `findLastDigit`, `Result`, `Value`, `ReturnError`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 523-539
```cpp
    APInt Result(128, 0);
    if (StringRef(NumStart, CurPtr - NumStart).getAsInteger(16, Result))
      return ReturnError(TokStart, "invalid hexadecimal number");

    return intToken(StringRef(TokStart, CurPtr - TokStart), Result);
  }

  // Motorola binary integers: %[01]+
  if (LexMotorolaIntegers && CurPtr[-1] == '%') {
    const char *NumStart = CurPtr;
    while (*CurPtr == '0' || *CurPtr == '1')
      ++CurPtr;

    APInt Result(128, 0);
    if (StringRef(NumStart, CurPtr - NumStart).getAsInteger(2, Result))
      return ReturnError(TokStart, "invalid binary number");

```
- **EN**: Implements logic around `Result`, `ReturnError`, `intToken`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `Result`, `ReturnError`, `intToken` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 540-559
```cpp
    return intToken(StringRef(TokStart, CurPtr - TokStart), Result);
  }

  // Decimal integer: [1-9][0-9]*
  // HLASM-flavour decimal integer: [0-9][0-9]*
  // FIXME: Later on, support for fb for HLASM has to be added in
  // as they probably would be needed for asm goto
  if (LexHLASMIntegers || CurPtr[-1] != '0' || CurPtr[0] == '.') {
    unsigned Radix = doHexLookAhead(CurPtr, 10, LexMasmIntegers);

    if (!LexHLASMIntegers) {
      bool IsHex = Radix == 16;
      // Check for floating point literals.
      if (!IsHex && (*CurPtr == '.' || *CurPtr == 'e' || *CurPtr == 'E')) {
        if (*CurPtr == '.')
          ++CurPtr;
        return LexFloatLiteral();
      }
    }

```
- **EN**: Implements logic around `intToken`, `doHexLookAhead`, `LexFloatLiteral`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `intToken`, `doHexLookAhead`, `LexFloatLiteral` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 560-585
```cpp
    StringRef Result(TokStart, CurPtr - TokStart);

    APInt Value(128, 0, true);
    if (Result.getAsInteger(Radix, Value))
      return ReturnError(TokStart, "invalid " + radixName(Radix) + " number");

    if (!LexHLASMIntegers)
      // The darwin/x86 (and x86-64) assembler accepts and ignores type
      // suffices on integer literals.
      SkipIgnoredIntegerSuffix(CurPtr);

    return intToken(Result, Value);
  }

  if (!LexMasmIntegers && ((*CurPtr == 'b') || (*CurPtr == 'B'))) {
    ++CurPtr;
    // See if we actually have "0b" as part of something like "jmp 0b\n"
    if (!isDigit(CurPtr[0])) {
      --CurPtr;
      StringRef Result(TokStart, CurPtr - TokStart);
      return AsmToken(AsmToken::Integer, Result, 0);
    }
    const char *NumStart = CurPtr;
    while (CurPtr[0] == '0' || CurPtr[0] == '1')
      ++CurPtr;

```
- **EN**: Implements logic around `Result`, `Value`, `ReturnError`, `SkipIgnoredIntegerSuffix`, and 2 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `Result`, `Value`, `ReturnError`, `SkipIgnoredIntegerSuffix`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 586-602
```cpp
    // Requires at least one binary digit.
    if (CurPtr == NumStart)
      return ReturnError(TokStart, "invalid binary number");

    StringRef Result(TokStart, CurPtr - TokStart);

    APInt Value(128, 0, true);
    if (Result.substr(2).getAsInteger(2, Value))
      return ReturnError(TokStart, "invalid binary number");

    // The darwin/x86 (and x86-64) assembler accepts and ignores ULL and LL
    // suffixes on integer literals.
    SkipIgnoredIntegerSuffix(CurPtr);

    return intToken(Result, Value);
  }

```
- **EN**: Implements logic around `ReturnError`, `Result`, `Value`, `SkipIgnoredIntegerSuffix`, and 1 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `ReturnError`, `Result`, `Value`, `SkipIgnoredIntegerSuffix`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 603-621
```cpp
  if ((*CurPtr == 'x') || (*CurPtr == 'X')) {
    ++CurPtr;
    const char *NumStart = CurPtr;
    while (isHexDigit(CurPtr[0]))
      ++CurPtr;

    // "0x.0p0" is valid, and "0x0p0" (but not "0xp0" for example, which will be
    // diagnosed by LexHexFloatLiteral).
    if (CurPtr[0] == '.' || CurPtr[0] == 'p' || CurPtr[0] == 'P')
      return LexHexFloatLiteral(NumStart == CurPtr);

    // Otherwise requires at least one hex digit.
    if (CurPtr == NumStart)
      return ReturnError(CurPtr-2, "invalid hexadecimal number");

    APInt Result(128, 0);
    if (StringRef(TokStart, CurPtr - TokStart).getAsInteger(0, Result))
      return ReturnError(TokStart, "invalid hexadecimal number");

```
- **EN**: Implements logic around `LexHexFloatLiteral`, `ReturnError`, `Result`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `LexHexFloatLiteral`, `ReturnError`, `Result` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 622-639
```cpp
    // Consume the optional [hH].
    if (LexMasmIntegers && (*CurPtr == 'h' || *CurPtr == 'H'))
      ++CurPtr;

    // The darwin/x86 (and x86-64) assembler accepts and ignores ULL and LL
    // suffixes on integer literals.
    SkipIgnoredIntegerSuffix(CurPtr);

    return intToken(StringRef(TokStart, CurPtr - TokStart), Result);
  }

  // Either octal or hexadecimal.
  APInt Value(128, 0, true);
  unsigned Radix = doHexLookAhead(CurPtr, 8, LexMasmIntegers);
  StringRef Result(TokStart, CurPtr - TokStart);
  if (Result.getAsInteger(Radix, Value))
    return ReturnError(TokStart, "invalid " + radixName(Radix) + " number");

```
- **EN**: Implements logic around `SkipIgnoredIntegerSuffix`, `intToken`, `Value`, `doHexLookAhead`, and 2 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `SkipIgnoredIntegerSuffix`, `intToken`, `Value`, `doHexLookAhead`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 640-657
```cpp
  // Consume the [hH].
  if (Radix == 16)
    ++CurPtr;

  // The darwin/x86 (and x86-64) assembler accepts and ignores ULL and LL
  // suffixes on integer literals.
  SkipIgnoredIntegerSuffix(CurPtr);

  return intToken(Result, Value);
}

/// LexSingleQuote: Integer: 'b'
AsmToken AsmLexer::LexSingleQuote() {
  int CurChar = getNextChar();

  if (LexHLASMStrings)
    return ReturnError(TokStart, "invalid usage of character literals");

```
- **EN**: Implements logic around `SkipIgnoredIntegerSuffix`, `intToken`, `LexSingleQuote`, `getNextChar`, and 1 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `SkipIgnoredIntegerSuffix`, `intToken`, `LexSingleQuote`, `getNextChar`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 658-675
```cpp
  if (LexMasmStrings) {
    while (CurChar != EOF) {
      if (CurChar != '\'') {
        CurChar = getNextChar();
      } else if (peekNextChar() == '\'') {
        // In MASM single-quote strings, doubled single-quotes mean an escaped
        // single quote, so should be lexed in.
        (void)getNextChar();
        CurChar = getNextChar();
      } else {
        break;
      }
    }
    if (CurChar == EOF)
      return ReturnError(TokStart, "unterminated string constant");
    return AsmToken(AsmToken::String, StringRef(TokStart, CurPtr - TokStart));
  }

```
- **EN**: Implements logic around `getNextChar`, `ReturnError`, `AsmToken`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `getNextChar`, `ReturnError`, `AsmToken` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 676-691
```cpp
  if (CurChar == '\\')
    CurChar = getNextChar();

  if (CurChar == EOF)
    return ReturnError(TokStart, "unterminated single quote");

  CurChar = getNextChar();

  if (CurChar != '\'')
    return ReturnError(TokStart, "single quote way too long");

  // The idea here being that 'c' is basically just an integral
  // constant.
  StringRef Res = StringRef(TokStart,CurPtr - TokStart);
  long long Value;

```
- **EN**: Implements logic around `getNextChar`, `ReturnError`, `StringRef`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `getNextChar`, `ReturnError`, `StringRef` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 692-708
```cpp
  if (Res.starts_with("\'\\")) {
    char theChar = Res[2];
    switch (theChar) {
      default: Value = theChar; break;
      case '\'': Value = '\''; break;
      case 't': Value = '\t'; break;
      case 'n': Value = '\n'; break;
      case 'b': Value = '\b'; break;
      case 'f': Value = '\f'; break;
      case 'r': Value = '\r'; break;
    }
  } else
    Value = TokStart[1];

  return AsmToken(AsmToken::Integer, Res, Value);
}

```
- **EN**: Implements logic around `AsmToken`; this block parses assembly syntax or operands; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `AsmToken` 实现具体逻辑；这一段解析汇编语法或操作数，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 709-732
```cpp
/// LexQuote: String: "..."
AsmToken AsmLexer::LexQuote() {
  int CurChar = getNextChar();
  if (LexHLASMStrings)
    return ReturnError(TokStart, "invalid usage of string literals");

  if (LexMasmStrings) {
    while (CurChar != EOF) {
      if (CurChar != '"') {
        CurChar = getNextChar();
      } else if (peekNextChar() == '"') {
        // In MASM double-quoted strings, doubled double-quotes mean an escaped
        // double quote, so should be lexed in.
        (void)getNextChar();
        CurChar = getNextChar();
      } else {
        break;
      }
    }
    if (CurChar == EOF)
      return ReturnError(TokStart, "unterminated string constant");
    return AsmToken(AsmToken::String, StringRef(TokStart, CurPtr - TokStart));
  }

```
- **EN**: Implements logic around `LexQuote`, `getNextChar`, `ReturnError`, `AsmToken`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `LexQuote`, `getNextChar`, `ReturnError`, `AsmToken` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 733-750
```cpp
  while (CurChar != '"') {
    if (CurChar == '\\') {
      // Allow \", etc.
      CurChar = getNextChar();
    }

    if (CurChar == EOF)
      return ReturnError(TokStart, "unterminated string constant");

    CurChar = getNextChar();
  }

  return AsmToken(AsmToken::String, StringRef(TokStart, CurPtr - TokStart));
}

StringRef AsmLexer::LexUntilEndOfStatement() {
  TokStart = CurPtr;

```
- **EN**: Implements logic around `getNextChar`, `ReturnError`, `AsmToken`, `LexUntilEndOfStatement`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `getNextChar`, `ReturnError`, `AsmToken`, `LexUntilEndOfStatement` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 751-767
```cpp
  while (!isAtStartOfComment(CurPtr) &&     // Start of line comment.
         !isAtStatementSeparator(CurPtr) && // End of statement marker.
         *CurPtr != '\n' && *CurPtr != '\r' && CurPtr != CurBuf.end()) {
    ++CurPtr;
  }
  return StringRef(TokStart, CurPtr-TokStart);
}

StringRef AsmLexer::LexUntilEndOfLine() {
  TokStart = CurPtr;

  while (*CurPtr != '\n' && *CurPtr != '\r' && CurPtr != CurBuf.end()) {
    ++CurPtr;
  }
  return StringRef(TokStart, CurPtr-TokStart);
}

```
- **EN**: Implements logic around `isAtStatementSeparator`, `end`, `StringRef`, `LexUntilEndOfLine`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `isAtStatementSeparator`, `end`, `StringRef`, `LexUntilEndOfLine` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 768-784
```cpp
size_t AsmLexer::peekTokens(MutableArrayRef<AsmToken> Buf,
                            bool ShouldSkipSpace) {
  SaveAndRestore SavedTokenStart(TokStart);
  SaveAndRestore SavedCurPtr(CurPtr);
  SaveAndRestore SavedAtStartOfLine(IsAtStartOfLine);
  SaveAndRestore SavedAtStartOfStatement(IsAtStartOfStatement);
  SaveAndRestore SavedSkipSpace(SkipSpace, ShouldSkipSpace);
  SaveAndRestore SavedIsPeeking(IsPeeking, true);
  std::string SavedErr = getErr();
  SMLoc SavedErrLoc = getErrLoc();

  size_t ReadCount;
  for (ReadCount = 0; ReadCount < Buf.size(); ++ReadCount) {
    AsmToken Token = LexToken();

    Buf[ReadCount] = Token;

```
- **EN**: Implements logic around `peekTokens`, `SavedTokenStart`, `SavedCurPtr`, `SavedAtStartOfLine`, and 6 more symbols; this block parses assembly syntax or operands.
- **CN**: 围绕 `peekTokens`, `SavedTokenStart`, `SavedCurPtr`, `SavedAtStartOfLine`, and 6 more symbols 实现具体逻辑；这一段解析汇编语法或操作数。

### Lines 785-800
```cpp
    if (Token.is(AsmToken::Eof)) {
      ReadCount++;
      break;
    }
  }

  SetError(SavedErrLoc, SavedErr);
  return ReadCount;
}

bool AsmLexer::isAtStartOfComment(const char *Ptr) {
  if (MAI.isHLASM() && !IsAtStartOfStatement)
    return false;

  StringRef CommentString = MAI.getCommentString();

```
- **EN**: Implements logic around `SetError`, `isAtStartOfComment`, `getCommentString`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `SetError`, `isAtStartOfComment`, `getCommentString` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 801-820
```cpp
  if (CommentString.size() == 1)
    return CommentString[0] == Ptr[0];

  // Allow # preprocessor comments also be counted as comments for "##" cases
  if (CommentString[1] == '#')
    return CommentString[0] == Ptr[0];

  return strncmp(Ptr, CommentString.data(), CommentString.size()) == 0;
}

bool AsmLexer::isAtStatementSeparator(const char *Ptr) {
  return strncmp(Ptr, MAI.getSeparatorString(),
                 strlen(MAI.getSeparatorString())) == 0;
}

AsmToken AsmLexer::LexToken() {
  TokStart = CurPtr;
  // This always consumes at least one character.
  int CurChar = getNextChar();

```
- **EN**: Implements logic around `strncmp`, `isAtStatementSeparator`, `strlen`, `LexToken`, and 1 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `strncmp`, `isAtStatementSeparator`, `strlen`, `LexToken`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 821-836
```cpp
  if (!IsPeeking && CurChar == '#' && IsAtStartOfStatement) {
    // If this starts with a '#', this may be a cpp
    // hash directive and otherwise a line comment.
    AsmToken TokenBuf[2];
    MutableArrayRef<AsmToken> Buf(TokenBuf, 2);
    size_t num = peekTokens(Buf, true);
    // There cannot be a space preceding this
    if (IsAtStartOfLine && num == 2 && TokenBuf[0].is(AsmToken::Integer) &&
        TokenBuf[1].is(AsmToken::String)) {
      CurPtr = TokStart; // reset curPtr;
      StringRef s = LexUntilEndOfLine();
      UnLex(TokenBuf[1]);
      UnLex(TokenBuf[0]);
      return AsmToken(AsmToken::HashDirective, s);
    }

```
- **EN**: Implements logic around `Buf`, `peekTokens`, `is`, `LexUntilEndOfLine`, and 2 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `Buf`, `peekTokens`, `is`, `LexUntilEndOfLine`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 837-852
```cpp
    if (MAI.shouldAllowAdditionalComments())
      return LexLineComment();
  }

  if (isAtStartOfComment(TokStart)) {
    StringRef CommentString = MAI.getCommentString();
    // For multi-char comment strings, advance CurPtr only if we matched the
    // full string. This stops us from accidentally eating the newline if the
    // current line ends in a single comment char.
    if (CommentString.size() > 1 &&
        StringRef(TokStart, CommentString.size()) == CommentString) {
      CurPtr += CommentString.size() - 1;
    }
    return LexLineComment();
  }

```
- **EN**: Implements logic around `LexLineComment`, `getCommentString`, `StringRef`, `size`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `LexLineComment`, `getCommentString`, `StringRef`, `size` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 853-879
```cpp
  if (isAtStatementSeparator(TokStart)) {
    CurPtr += strlen(MAI.getSeparatorString()) - 1;
    IsAtStartOfLine = true;
    IsAtStartOfStatement = true;
    return AsmToken(AsmToken::EndOfStatement,
                    StringRef(TokStart, strlen(MAI.getSeparatorString())));
  }

  // If we're missing a newline at EOF, make sure we still get an
  // EndOfStatement token before the Eof token.
  if (CurChar == EOF && !IsAtStartOfStatement && EndStatementAtEOF) {
    IsAtStartOfLine = true;
    IsAtStartOfStatement = true;
    return AsmToken(AsmToken::EndOfStatement, StringRef(TokStart, 0));
  }
  IsAtStartOfLine = false;
  bool OldIsAtStartOfStatement = IsAtStartOfStatement;
  IsAtStartOfStatement = false;
  switch (CurChar) {
  default:
    // Handle identifier: [a-zA-Z_.$@#?][a-zA-Z0-9_.$@#?]*
    // Whether or not the lexer accepts '$', '@', '#' and '?' at the start of
    // an identifier is target-dependent. These characters are handled in the
    // respective switch cases.
    if (isalpha(CurChar) || CurChar == '_' || CurChar == '.')
      return LexIdentifier();

```
- **EN**: Implements logic around `strlen`, `AsmToken`, `StringRef`, `LexIdentifier`; this block parses assembly syntax or operands; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `strlen`, `AsmToken`, `StringRef`, `LexIdentifier` 实现具体逻辑；这一段解析汇编语法或操作数，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 880-911
```cpp
    // Unknown character, emit an error.
    return ReturnError(TokStart, "invalid character in input");
  case EOF:
    if (EndStatementAtEOF) {
      IsAtStartOfLine = true;
      IsAtStartOfStatement = true;
    }
    return AsmToken(AsmToken::Eof, StringRef(TokStart, 0));
  case 0:
  case ' ':
  case '\t':
    IsAtStartOfStatement = OldIsAtStartOfStatement;
    while (*CurPtr == ' ' || *CurPtr == '\t')
      CurPtr++;
    if (SkipSpace)
      return LexToken(); // Ignore whitespace.
    else
      return AsmToken(AsmToken::Space, StringRef(TokStart, CurPtr - TokStart));
  case '\r': {
    IsAtStartOfLine = true;
    IsAtStartOfStatement = true;
    // If this is a CR followed by LF, treat that as one token.
    if (CurPtr != CurBuf.end() && *CurPtr == '\n')
      ++CurPtr;
    return AsmToken(AsmToken::EndOfStatement,
                    StringRef(TokStart, CurPtr - TokStart));
  }
  case '\n':
    IsAtStartOfLine = true;
    IsAtStartOfStatement = true;
    return AsmToken(AsmToken::EndOfStatement, StringRef(TokStart, 1));
  case ':': return AsmToken(AsmToken::Colon, StringRef(TokStart, 1));
```
- **EN**: Implements logic around `ReturnError`, `AsmToken`, `LexToken`, `StringRef`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `ReturnError`, `AsmToken`, `LexToken`, `StringRef` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 912-943
```cpp
  case '+': return AsmToken(AsmToken::Plus, StringRef(TokStart, 1));
  case '~': return AsmToken(AsmToken::Tilde, StringRef(TokStart, 1));
  case '(': return AsmToken(AsmToken::LParen, StringRef(TokStart, 1));
  case ')': return AsmToken(AsmToken::RParen, StringRef(TokStart, 1));
  case '[': return AsmToken(AsmToken::LBrac, StringRef(TokStart, 1));
  case ']': return AsmToken(AsmToken::RBrac, StringRef(TokStart, 1));
  case '{': return AsmToken(AsmToken::LCurly, StringRef(TokStart, 1));
  case '}': return AsmToken(AsmToken::RCurly, StringRef(TokStart, 1));
  case '*': return AsmToken(AsmToken::Star, StringRef(TokStart, 1));
  case ',': return AsmToken(AsmToken::Comma, StringRef(TokStart, 1));
  case '$': {
    if (LexMotorolaIntegers && isHexDigit(*CurPtr))
      return LexDigit();
    if (MAI.doesAllowDollarAtStartOfIdentifier())
      return LexIdentifier();
    return AsmToken(AsmToken::Dollar, StringRef(TokStart, 1));
  }
  case '@':
    if (MAI.doesAllowAtAtStartOfIdentifier())
      return LexIdentifier();
    return AsmToken(AsmToken::At, StringRef(TokStart, 1));
  case '#':
    if (MAI.isHLASM())
      return LexIdentifier();
    return AsmToken(AsmToken::Hash, StringRef(TokStart, 1));
  case '?':
    if (MAI.doesAllowQuestionAtStartOfIdentifier())
      return LexIdentifier();
    return AsmToken(AsmToken::Question, StringRef(TokStart, 1));
  case '\\': return AsmToken(AsmToken::BackSlash, StringRef(TokStart, 1));
  case '=':
    if (*CurPtr == '=') {
```
- **EN**: Implements logic around `AsmToken`, `LexDigit`, `LexIdentifier`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `AsmToken`, `LexDigit`, `LexIdentifier` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 944-975
```cpp
      ++CurPtr;
      return AsmToken(AsmToken::EqualEqual, StringRef(TokStart, 2));
    }
    return AsmToken(AsmToken::Equal, StringRef(TokStart, 1));
  case '-':
    if (*CurPtr == '>') {
      ++CurPtr;
      return AsmToken(AsmToken::MinusGreater, StringRef(TokStart, 2));
    }
    return AsmToken(AsmToken::Minus, StringRef(TokStart, 1));
  case '|':
    if (*CurPtr == '|') {
      ++CurPtr;
      return AsmToken(AsmToken::PipePipe, StringRef(TokStart, 2));
    }
    return AsmToken(AsmToken::Pipe, StringRef(TokStart, 1));
  case '^': return AsmToken(AsmToken::Caret, StringRef(TokStart, 1));
  case '&':
    if (*CurPtr == '&') {
      ++CurPtr;
      return AsmToken(AsmToken::AmpAmp, StringRef(TokStart, 2));
    }
    return AsmToken(AsmToken::Amp, StringRef(TokStart, 1));
  case '!':
    if (*CurPtr == '=') {
      ++CurPtr;
      return AsmToken(AsmToken::ExclaimEqual, StringRef(TokStart, 2));
    }
    return AsmToken(AsmToken::Exclaim, StringRef(TokStart, 1));
  case '%':
    if (LexMotorolaIntegers && (*CurPtr == '0' || *CurPtr == '1')) {
      return LexDigit();
```
- **EN**: Implements logic around `AsmToken`, `LexDigit`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `AsmToken`, `LexDigit` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 976-1007
```cpp
    }
    return AsmToken(AsmToken::Percent, StringRef(TokStart, 1));
  case '/':
    IsAtStartOfStatement = OldIsAtStartOfStatement;
    return LexSlash();
  case '\'': return LexSingleQuote();
  case '"': return LexQuote();
  case '0': case '1': case '2': case '3': case '4':
  case '5': case '6': case '7': case '8': case '9':
    return LexDigit();
  case '<':
    switch (*CurPtr) {
    case '<':
      ++CurPtr;
      return AsmToken(AsmToken::LessLess, StringRef(TokStart, 2));
    case '=':
      ++CurPtr;
      return AsmToken(AsmToken::LessEqual, StringRef(TokStart, 2));
    case '>':
      ++CurPtr;
      return AsmToken(AsmToken::LessGreater, StringRef(TokStart, 2));
    default:
      return AsmToken(AsmToken::Less, StringRef(TokStart, 1));
    }
  case '>':
    switch (*CurPtr) {
    case '>':
      ++CurPtr;
      return AsmToken(AsmToken::GreaterGreater, StringRef(TokStart, 2));
    case '=':
      ++CurPtr;
      return AsmToken(AsmToken::GreaterEqual, StringRef(TokStart, 2));
```
- **EN**: Implements logic around `AsmToken`, `LexSlash`, `LexSingleQuote`, `LexQuote`, and 1 more symbols; this block parses assembly syntax or operands; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `AsmToken`, `LexSlash`, `LexSingleQuote`, `LexQuote`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 1008-1017
```cpp
    default:
      return AsmToken(AsmToken::Greater, StringRef(TokStart, 1));
    }

  // TODO: Quoted identifiers (objc methods etc)
  // local labels: [0-9][:]
  // Forward/backward labels: [0-9][fb]
  // Integers, fp constants, character constants.
  }
}
```
- **EN**: Implements logic around `AsmToken`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `AsmToken` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

## Key Concepts / 关键概念

- **Assembly parsing / 汇编解析**:
  - **EN**: Tokenizes assembly text, interprets directives, and builds MC expressions or instructions
  - **CN**: 对汇编文本做词法与语法解析，并构建 MC 表达式或指令
- **Directive handling / 伪指令处理**:
  - **EN**: Interprets assembler directives and maps them to streamer operations
  - **CN**: 解释汇编伪指令并把它们映射为 streamer 操作

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/MC/MCParser/AsmLexer.h`, `llvm/ADT/APInt.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/StringExtras.h`, `llvm/ADT/StringRef.h`, `llvm/MC/MCAsmInfo.h`, `llvm/Support/Compiler.h`, `llvm/Support/SMLoc.h`, `llvm/Support/SaveAndRestore.h`, `llvm/Support/raw_ostream.h`, `cassert`, `cctype` ... (+3 more)
- **LLVM subsystems / LLVM 子系统**: MC, Support
