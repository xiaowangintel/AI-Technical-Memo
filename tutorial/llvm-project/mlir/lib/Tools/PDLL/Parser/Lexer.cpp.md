# Lexer.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Tools/PDLL/Parser/Lexer.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements textual MLIR parsing support.
  - **CN**: 实现 MLIR 文本解析支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```cpp
//===- Lexer.cpp ----------------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "Lexer.h"
#include "mlir/Tools/PDLL/AST/Diagnostic.h"
#include "mlir/Tools/PDLL/Parser/CodeComplete.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/ADT/StringSwitch.h"
#include "llvm/Support/SourceMgr.h"

```
- **EN**: Pulls in the declarations needed by this translation unit, including `Lexer.h`, `mlir/Tools/PDLL/AST/Diagnostic.h`, `mlir/Tools/PDLL/Parser/CodeComplete.h`, `llvm/ADT/StringExtras.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `Lexer.h`, `mlir/Tools/PDLL/AST/Diagnostic.h`, `mlir/Tools/PDLL/Parser/CodeComplete.h`, `llvm/ADT/StringExtras.h`。

### Lines 16-26
```cpp
using namespace mlir;
using namespace mlir::pdll;

//===----------------------------------------------------------------------===//
// Token
//===----------------------------------------------------------------------===//

std::string Token::getStringValue() const {
  assert(getKind() == string || getKind() == string_block ||
         getKind() == code_complete_string);

```
- **EN**: Implements logic around `getStringValue`, `assert`, `getKind`.
- **CN**: 围绕 `getStringValue`、`assert`、`getKind` 实现具体逻辑。

### Lines 27-42
```cpp
  // Start by dropping the quotes.
  StringRef bytes = getSpelling();
  if (is(string))
    bytes = bytes.drop_front().drop_back();
  else if (is(string_block))
    bytes = bytes.drop_front(2).drop_back(2);

  std::string result;
  result.reserve(bytes.size());
  for (unsigned i = 0, e = bytes.size(); i != e;) {
    auto c = bytes[i++];
    if (c != '\\') {
      result.push_back(c);
      continue;
    }

```
- **EN**: Implements logic around `getSpelling`, `is`, `drop_front`, `reserve`, and 2 more symbols.
- **CN**: 围绕 `getSpelling`、`is`、`drop_front`、`reserve` 等另外 2 个符号 实现具体逻辑。

### Lines 43-59
```cpp
    assert(i + 1 <= e && "invalid string should be caught by lexer");
    auto c1 = bytes[i++];
    switch (c1) {
    case '"':
    case '\\':
      result.push_back(c1);
      continue;
    case 'n':
      result.push_back('\n');
      continue;
    case 't':
      result.push_back('\t');
      continue;
    default:
      break;
    }

```
- **EN**: Implements logic around `assert`, `push_back`.
- **CN**: 围绕 `assert`、`push_back` 实现具体逻辑。

### Lines 60-69
```cpp
    assert(i + 1 <= e && "invalid string should be caught by lexer");
    auto c2 = bytes[i++];

    assert(llvm::isHexDigit(c1) && llvm::isHexDigit(c2) && "invalid escape");
    result.push_back((llvm::hexDigitValue(c1) << 4) | llvm::hexDigitValue(c2));
  }

  return result;
}

```
- **EN**: Implements logic around `assert`, `push_back`.
- **CN**: 围绕 `assert`、`push_back` 实现具体逻辑。

### Lines 70-81
```cpp
//===----------------------------------------------------------------------===//
// Lexer
//===----------------------------------------------------------------------===//

Lexer::Lexer(llvm::SourceMgr &mgr, ast::DiagnosticEngine &diagEngine,
             CodeCompleteContext *codeCompleteContext)
    : srcMgr(mgr), diagEngine(diagEngine), addedHandlerToDiagEngine(false),
      codeCompletionLocation(nullptr) {
  curBufferID = mgr.getMainFileID();
  curBuffer = srcMgr.getMemoryBuffer(curBufferID)->getBuffer();
  curPtr = curBuffer.begin();

```
- **EN**: Implements logic around `Lexer`, `srcMgr`, `codeCompletionLocation`, `getMainFileID`, and 2 more symbols.
- **CN**: 围绕 `Lexer`、`srcMgr`、`codeCompletionLocation`、`getMainFileID` 等另外 2 个符号 实现具体逻辑。

### Lines 82-101
```cpp
  // Set the code completion location if necessary.
  if (codeCompleteContext) {
    codeCompletionLocation =
        codeCompleteContext->getCodeCompleteLoc().getPointer();
  }

  // If the diag engine has no handler, add a default that emits to the
  // SourceMgr.
  if (!diagEngine.getHandlerFn()) {
    diagEngine.setHandlerFn([&](const ast::Diagnostic &diag) {
      srcMgr.PrintMessage(diag.getLocation().Start, diag.getSeverity(),
                          diag.getMessage());
      for (const ast::Diagnostic &note : diag.getNotes())
        srcMgr.PrintMessage(note.getLocation().Start, note.getSeverity(),
                            note.getMessage());
    });
    addedHandlerToDiagEngine = true;
  }
}

```
- **EN**: Implements logic around `getCodeCompleteLoc`, `getHandlerFn`, `setHandlerFn`, `PrintMessage`, and 2 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `getCodeCompleteLoc`、`getHandlerFn`、`setHandlerFn`、`PrintMessage` 等另外 2 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 102-113
```cpp
Lexer::~Lexer() {
  if (addedHandlerToDiagEngine)
    diagEngine.setHandlerFn(nullptr);
}

LogicalResult Lexer::pushInclude(StringRef filename, SMRange includeLoc) {
  std::string includedFile;
  int bufferID =
      srcMgr.AddIncludeFile(filename.str(), includeLoc.End, includedFile);
  if (!bufferID)
    return failure();

```
- **EN**: Implements logic around `~Lexer`, `setHandlerFn`, `pushInclude`, `AddIncludeFile`, and 1 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `~Lexer`、`setHandlerFn`、`pushInclude`、`AddIncludeFile` 等另外 1 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 114-133
```cpp
  curBufferID = bufferID;
  curBuffer = srcMgr.getMemoryBuffer(curBufferID)->getBuffer();
  curPtr = curBuffer.begin();
  return success();
}

Token Lexer::emitError(SMRange loc, const Twine &msg) {
  diagEngine.emitError(loc, msg);
  return formToken(Token::error, loc.Start.getPointer());
}
Token Lexer::emitErrorAndNote(SMRange loc, const Twine &msg, SMRange noteLoc,
                              const Twine &note) {
  diagEngine.emitError(loc, msg)->attachNote(note, noteLoc);
  return formToken(Token::error, loc.Start.getPointer());
}
Token Lexer::emitError(const char *loc, const Twine &msg) {
  return emitError(
      SMRange(SMLoc::getFromPointer(loc), SMLoc::getFromPointer(loc + 1)), msg);
}

```
- **EN**: Implements logic around `getMemoryBuffer`, `begin`, `success`, `emitError`, and 3 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `getMemoryBuffer`、`begin`、`success`、`emitError` 等另外 3 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 134-144
```cpp
int Lexer::getNextChar() {
  char curChar = *curPtr++;
  switch (curChar) {
  default:
    return static_cast<unsigned char>(curChar);
  case 0: {
    // A nul character in the stream is either the end of the current buffer
    // or a random nul in the file. Disambiguate that here.
    if (curPtr - 1 != curBuffer.end())
      return 0;

```
- **EN**: Implements logic around `getNextChar`, `char>`, `end`.
- **CN**: 围绕 `getNextChar`、`char>`、`end` 实现具体逻辑。

### Lines 145-159
```cpp
    // Otherwise, return end of file.
    --curPtr;
    return EOF;
  }
  case '\n':
  case '\r':
    // Handle the newline character by ignoring it and incrementing the line
    // count. However, be careful about 'dos style' files with \n\r in them.
    // Only treat a \n\r or \r\n as a single line.
    if ((*curPtr == '\n' || (*curPtr == '\r')) && *curPtr != curChar)
      ++curPtr;
    return '\n';
  }
}

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 160-175
```cpp
Token Lexer::lexToken() {
  while (true) {
    const char *tokStart = curPtr;

    // Check to see if this token is at the code completion location.
    if (tokStart == codeCompletionLocation)
      return formToken(Token::code_complete, tokStart);

    // This always consumes at least one character.
    int curChar = getNextChar();
    switch (curChar) {
    default:
      // Handle identifiers: [a-zA-Z_]
      if (isalpha(curChar) || curChar == '_')
        return lexIdentifier(tokStart);

```
- **EN**: Implements logic around `lexToken`, `formToken`, `getNextChar`, `isalpha`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `lexToken`、`formToken`、`getNextChar`、`isalpha` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 176-189
```cpp
      // Unknown character, emit an error.
      return emitError(tokStart, "unexpected character");
    case EOF: {
      // Return EOF denoting the end of lexing.
      Token eof = formToken(Token::eof, tokStart);

      // Check to see if we are in an included file.
      SMLoc parentIncludeLoc = srcMgr.getParentIncludeLoc(curBufferID);
      if (parentIncludeLoc.isValid()) {
        curBufferID = srcMgr.FindBufferContainingLoc(parentIncludeLoc);
        curBuffer = srcMgr.getMemoryBuffer(curBufferID)->getBuffer();
        curPtr = parentIncludeLoc.getPointer();
      }

```
- **EN**: Implements logic around `emitError`, `formToken`, `getParentIncludeLoc`, `isValid`, and 3 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `emitError`、`formToken`、`getParentIncludeLoc`、`isValid` 等另外 3 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 190-209
```cpp
      return eof;
    }

    // Lex punctuation.
    case '-':
      if (*curPtr == '>') {
        ++curPtr;
        return formToken(Token::arrow, tokStart);
      }
      return emitError(tokStart, "unexpected character");
    case ':':
      return formToken(Token::colon, tokStart);
    case ',':
      return formToken(Token::comma, tokStart);
    case '.':
      return formToken(Token::dot, tokStart);
    case '=':
      if (*curPtr == '>') {
        ++curPtr;
        return formToken(Token::equal_arrow, tokStart);
```
- **EN**: Implements logic around `formToken`, `emitError`; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `formToken`、`emitError` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 210-222
```cpp
      }
      return formToken(Token::equal, tokStart);
    case ';':
      return formToken(Token::semicolon, tokStart);
    case '[':
      if (*curPtr == '{') {
        ++curPtr;
        return lexString(tokStart, /*isStringBlock=*/true);
      }
      return formToken(Token::l_square, tokStart);
    case ']':
      return formToken(Token::r_square, tokStart);

```
- **EN**: Implements logic around `formToken`, `lexString`.
- **CN**: 围绕 `formToken`、`lexString` 实现具体逻辑。

### Lines 223-241
```cpp
    case '<':
      return formToken(Token::less, tokStart);
    case '>':
      return formToken(Token::greater, tokStart);
    case '{':
      return formToken(Token::l_brace, tokStart);
    case '}':
      return formToken(Token::r_brace, tokStart);
    case '(':
      return formToken(Token::l_paren, tokStart);
    case ')':
      return formToken(Token::r_paren, tokStart);
    case '/':
      if (*curPtr == '/') {
        lexComment();
        continue;
      }
      return emitError(tokStart, "unexpected character");

```
- **EN**: Implements logic around `formToken`, `lexComment`, `emitError`; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `formToken`、`lexComment`、`emitError` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 242-253
```cpp
    // Ignore whitespace characters.
    case 0:
    case ' ':
    case '\t':
    case '\n':
      return lexToken();

    case '#':
      return lexDirective(tokStart);
    case '"':
      return lexString(tokStart, /*isStringBlock=*/false);

```
- **EN**: Implements logic around `lexToken`, `lexDirective`, `lexString`.
- **CN**: 围绕 `lexToken`、`lexDirective`、`lexString` 实现具体逻辑。

### Lines 254-268
```cpp
    case '0':
    case '1':
    case '2':
    case '3':
    case '4':
    case '5':
    case '6':
    case '7':
    case '8':
    case '9':
      return lexNumber(tokStart);
    }
  }
}

```
- **EN**: Implements logic around `lexNumber`.
- **CN**: 围绕 `lexNumber` 实现具体逻辑。

### Lines 269-288
```cpp
/// Skip a comment line, starting with a '//'.
void Lexer::lexComment() {
  // Advance over the second '/' in a '//' comment.
  assert(*curPtr == '/');
  ++curPtr;

  while (true) {
    switch (*curPtr++) {
    case '\n':
    case '\r':
      // Newline is end of comment.
      return;
    case 0:
      // If this is the end of the buffer, end the comment.
      if (curPtr - 1 == curBuffer.end()) {
        --curPtr;
        return;
      }
      [[fallthrough]];
    default:
```
- **EN**: Implements logic around `lexComment`, `assert`, `end`.
- **CN**: 围绕 `lexComment`、`assert`、`end` 实现具体逻辑。

### Lines 289-299
```cpp
      // Skip over other characters.
      break;
    }
  }
}

Token Lexer::lexDirective(const char *tokStart) {
  // Match the rest with an identifier regex: [0-9a-zA-Z_]*
  while (isalnum(*curPtr) || *curPtr == '_')
    ++curPtr;

```
- **EN**: Implements logic around `lexDirective`, `isalnum`.
- **CN**: 围绕 `lexDirective`、`isalnum` 实现具体逻辑。

### Lines 300-319
```cpp
  StringRef str(tokStart, curPtr - tokStart);
  return Token(Token::directive, str);
}

Token Lexer::lexIdentifier(const char *tokStart) {
  // Match the rest of the identifier regex: [0-9a-zA-Z_]*
  while (isalnum(*curPtr) || *curPtr == '_')
    ++curPtr;

  // Check to see if this identifier is a keyword.
  StringRef str(tokStart, curPtr - tokStart);
  Token::Kind kind = StringSwitch<Token::Kind>(str)
                         .Case("attr", Token::kw_attr)
                         .Case("Attr", Token::kw_Attr)
                         .Case("erase", Token::kw_erase)
                         .Case("let", Token::kw_let)
                         .Case("Constraint", Token::kw_Constraint)
                         .Case("not", Token::kw_not)
                         .Case("op", Token::kw_op)
                         .Case("Op", Token::kw_Op)
```
- **EN**: Implements logic around `str`, `Token`, `lexIdentifier`, `isalnum`, and 2 more symbols.
- **CN**: 围绕 `str`、`Token`、`lexIdentifier`、`isalnum` 等另外 2 个符号 实现具体逻辑。

### Lines 320-336
```cpp
                         .Case("OpName", Token::kw_OpName)
                         .Case("Pattern", Token::kw_Pattern)
                         .Case("replace", Token::kw_replace)
                         .Case("return", Token::kw_return)
                         .Case("rewrite", Token::kw_rewrite)
                         .Case("Rewrite", Token::kw_Rewrite)
                         .Case("type", Token::kw_type)
                         .Case("Type", Token::kw_Type)
                         .Case("TypeRange", Token::kw_TypeRange)
                         .Case("Value", Token::kw_Value)
                         .Case("ValueRange", Token::kw_ValueRange)
                         .Case("with", Token::kw_with)
                         .Case("_", Token::underscore)
                         .Default(Token::identifier);
  return Token(kind, str);
}

```
- **EN**: Implements logic around `Case`, `Default`, `Token`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `Case`、`Default`、`Token` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 337-346
```cpp
Token Lexer::lexNumber(const char *tokStart) {
  assert(isdigit(curPtr[-1]));

  // Handle the normal decimal case.
  while (isdigit(*curPtr))
    ++curPtr;

  return formToken(Token::integer, tokStart);
}

```
- **EN**: Implements logic around `lexNumber`, `assert`, `isdigit`, `formToken`.
- **CN**: 围绕 `lexNumber`、`assert`、`isdigit`、`formToken` 实现具体逻辑。

### Lines 347-358
```cpp
Token Lexer::lexString(const char *tokStart, bool isStringBlock) {
  while (true) {
    // Check to see if there is a code completion location within the string. In
    // these cases we generate a completion location and place the currently
    // lexed string within the token (without the quotes). This allows for the
    // parser to use the partially lexed string when computing the completion
    // results.
    if (curPtr == codeCompletionLocation) {
      return formToken(Token::code_complete_string,
                       tokStart + (isStringBlock ? 2 : 1));
    }

```
- **EN**: Implements logic around `lexString`, `formToken`; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `lexString`、`formToken` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 359-378
```cpp
    switch (*curPtr++) {
    case '"':
      // If this is a string block, we only end the string when we encounter a
      // `}]`.
      if (!isStringBlock)
        return formToken(Token::string, tokStart);
      continue;
    case '}':
      // If this is a string block, we only end the string when we encounter a
      // `}]`.
      if (!isStringBlock || *curPtr != ']')
        continue;
      ++curPtr;
      return formToken(Token::string_block, tokStart);
    case 0: {
      // If this is a random nul character in the middle of a string, just
      // include it. If it is the end of file, then it is an error.
      if (curPtr - 1 != curBuffer.end())
        continue;
      --curPtr;
```
- **EN**: Implements logic around `formToken`, `end`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `formToken`、`end` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 379-392
```cpp

      StringRef expectedEndStr = isStringBlock ? "}]" : "\"";
      return emitError(curPtr - 1,
                       "expected '" + expectedEndStr + "' in string literal");
    }

    case '\n':
    case '\v':
    case '\f':
      // String blocks allow multiple lines.
      if (!isStringBlock)
        return emitError(curPtr - 1, "expected '\"' in string literal");
      continue;

```
- **EN**: Implements logic around `emitError`; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `emitError` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 393-405
```cpp
    case '\\':
      // Handle explicitly a few escapes.
      if (*curPtr == '"' || *curPtr == '\\' || *curPtr == 'n' ||
          *curPtr == 't') {
        ++curPtr;
      } else if (llvm::isHexDigit(*curPtr) && llvm::isHexDigit(curPtr[1])) {
        // Support \xx for two hex digits.
        curPtr += 2;
      } else {
        return emitError(curPtr - 1, "unknown escape in string literal");
      }
      continue;

```
- **EN**: Implements logic around `isHexDigit`, `emitError`; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `isHexDigit`、`emitError` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 406-410
```cpp
    default:
      continue;
    }
  }
}
```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

## Key Concepts / 关键概念

- **Assembly parsing / 汇编解析**:
  - **EN**: Parses textual MLIR into structured IR objects with diagnostics and recovery paths.
  - **CN**: 把文本形式的 MLIR 解析为结构化 IR 对象，并提供诊断与恢复路径。
- **Tooling support / 工具链支持**:
  - **EN**: Builds reusable infrastructure for opt-like tools, translation drivers, and CLIs.
  - **CN**: 为类似 opt 的工具、翻译驱动和命令行程序构建可复用基础设施。
- **SSA and region-based IR / SSA 与基于 Region 的 IR**:
  - **EN**: The file works with MLIR operations, blocks, values, or regions that form MLIR's hierarchical SSA IR.
  - **CN**: 该文件处理 MLIR 操作、块、值或 Region，它们共同构成 MLIR 的层次化 SSA IR。
- **Structured diagnostics / 结构化诊断**:
  - **EN**: Failures are surfaced through `LogicalResult`, diagnostics, or related reporting helpers.
  - **CN**: 通过 `LogicalResult`、诊断对象或相关报告工具显式传播失败。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `Lexer.h`, `mlir/Tools/PDLL/AST/Diagnostic.h`, `mlir/Tools/PDLL/Parser/CodeComplete.h`, `llvm/ADT/StringExtras.h`, `llvm/ADT/StringSwitch.h`, `llvm/Support/SourceMgr.h`
- **Subsystem categories / 子系统类别**: tooling support declarations / 工具支持声明 (2), LLVM ADT containers and utilities / LLVM ADT 容器与工具 (2), LLVM support-library helpers / LLVM Support 库辅助工具 (1)
