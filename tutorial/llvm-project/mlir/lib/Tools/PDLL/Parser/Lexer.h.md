# Lexer.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Tools/PDLL/Parser/Lexer.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Declares textual MLIR parsing support.
  - **CN**: 声明 MLIR 文本解析支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- Lexer.h - MLIR PDLL Frontend Lexer -----------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及高层摘要。

### Lines 8-15
```cpp

#ifndef LIB_TOOLS_PDLL_PARSER_LEXER_H_
#define LIB_TOOLS_PDLL_PARSER_LEXER_H_

#include "mlir/Support/LLVM.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/Support/SMLoc.h"

```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/Support/LLVM.h`, `llvm/ADT/StringRef.h`, `llvm/Support/SMLoc.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/Support/LLVM.h`, `llvm/ADT/StringRef.h`, `llvm/Support/SMLoc.h`。

### Lines 16-22
```cpp
namespace llvm {
class SourceMgr;
} // namespace llvm

namespace mlir {
namespace pdll {
class CodeCompleteContext;
```
- **EN**: Introduces declarations for `llvm`, `SourceMgr`, `mlir`, `pdll`, and 1 more symbols, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `llvm`、`SourceMgr`、`mlir`、`pdll` 等另外 1 个符号 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 23-30
```cpp

namespace ast {
class DiagnosticEngine;
} // namespace ast

//===----------------------------------------------------------------------===//
// Token
//===----------------------------------------------------------------------===//
```
- **EN**: Introduces declarations for `ast`, `DiagnosticEngine`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `ast`、`DiagnosticEngine` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 31-38
```cpp

class Token {
public:
  enum Kind {
    /// Markers.
    eof,
    error,
    /// Token signifying a code completion location.
```
- **EN**: Introduces declarations for `Token`, `Kind`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `Token`、`Kind` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 39-45
```cpp
    code_complete,
    /// Token signifying a code completion location within a string.
    code_complete_string,

    /// Keywords.
    KW_BEGIN,
    /// Dependent keywords, i.e. those that are treated as keywords depending on
```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 46-52
```cpp
    /// the current parser context.
    KW_DEPENDENT_BEGIN,
    kw_attr,
    kw_op,
    kw_type,
    KW_DEPENDENT_END,

```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 53-66
```cpp
    /// General keywords.
    kw_Attr,
    kw_erase,
    kw_let,
    kw_Constraint,
    kw_not,
    kw_Op,
    kw_OpName,
    kw_Pattern,
    kw_replace,
    kw_return,
    kw_rewrite,
    kw_Rewrite,
    kw_Type,
```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 67-73
```cpp
    kw_TypeRange,
    kw_Value,
    kw_ValueRange,
    kw_with,
    KW_END,

    /// Punctuation.
```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 74-81
```cpp
    arrow,
    colon,
    comma,
    dot,
    equal,
    equal_arrow,
    semicolon,
    /// Paired punctuation.
```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 82-91
```cpp
    less,
    greater,
    l_brace,
    r_brace,
    l_paren,
    r_paren,
    l_square,
    r_square,
    underscore,

```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 92-100
```cpp
    /// Tokens.
    directive,
    identifier,
    integer,
    string_block,
    string
  };
  Token(Kind kind, StringRef spelling) : kind(kind), spelling(spelling) {}

```
- **EN**: Implements logic around `Token`.
- **CN**: 围绕 `Token` 实现具体逻辑。

### Lines 101-107
```cpp
  /// Given a token containing a string literal, return its value, including
  /// removing the quote characters and unescaping the contents of the string.
  std::string getStringValue() const;

  /// Returns true if the current token is a string literal.
  bool isString() const { return isAny(Token::string, Token::string_block); }

```
- **EN**: Implements logic around `getStringValue`, `isString`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getStringValue`、`isString` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 108-114
```cpp
  /// Returns true if the current token is a keyword.
  bool isKeyword() const {
    return kind > Token::KW_BEGIN && kind < Token::KW_END;
  }

  /// Returns true if the current token is a keyword in a dependent context, and
  /// in any other situation (e.g. variable names) may be treated as an
```
- **EN**: Implements logic around `isKeyword`.
- **CN**: 围绕 `isKeyword` 实现具体逻辑。

### Lines 115-122
```cpp
  /// identifier.
  bool isDependentKeyword() const {
    return kind > Token::KW_DEPENDENT_BEGIN && kind < Token::KW_DEPENDENT_END;
  }

  /// Return the bytes that make up this token.
  StringRef getSpelling() const { return spelling; }

```
- **EN**: Implements logic around `isDependentKeyword`, `getSpelling`.
- **CN**: 围绕 `isDependentKeyword`、`getSpelling` 实现具体逻辑。

### Lines 123-132
```cpp
  /// Return the kind of this token.
  Kind getKind() const { return kind; }

  /// Return true if this token is one of the specified kinds.
  bool isAny(Kind k1, Kind k2) const { return is(k1) || is(k2); }
  template <typename... T>
  bool isAny(Kind k1, Kind k2, Kind k3, T... others) const {
    return is(k1) || isAny(k2, k3, others...);
  }

```
- **EN**: Implements logic around `getKind`, `isAny`, `is`.
- **CN**: 围绕 `getKind`、`isAny`、`is` 实现具体逻辑。

### Lines 133-139
```cpp
  /// Return if the token does not have the given kind.
  bool isNot(Kind k) const { return k != kind; }
  template <typename... T>
  bool isNot(Kind k1, Kind k2, T... others) const {
    return !isAny(k1, k2, others...);
  }

```
- **EN**: Implements logic around `isNot`, `isAny`.
- **CN**: 围绕 `isNot`、`isAny` 实现具体逻辑。

### Lines 140-149
```cpp
  /// Return if the token has the given kind.
  bool is(Kind k) const { return kind == k; }

  /// Return a location for the start of this token.
  SMLoc getStartLoc() const { return SMLoc::getFromPointer(spelling.data()); }
  /// Return a location at the end of this token.
  SMLoc getEndLoc() const {
    return SMLoc::getFromPointer(spelling.data() + spelling.size());
  }
  /// Return a location for the range of this token.
```
- **EN**: Implements logic around `is`, `getStartLoc`, `getEndLoc`, `getFromPointer`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `is`、`getStartLoc`、`getEndLoc`、`getFromPointer` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 150-156
```cpp
  SMRange getLoc() const { return SMRange(getStartLoc(), getEndLoc()); }

private:
  /// Discriminator that indicates the kind of token this is.
  Kind kind;

  /// A reference to the entire token contents; this is always a pointer into
```
- **EN**: Implements logic around `getLoc`.
- **CN**: 围绕 `getLoc` 实现具体逻辑。

### Lines 157-163
```cpp
  /// a memory buffer owned by the source manager.
  StringRef spelling;
};

//===----------------------------------------------------------------------===//
// Lexer
//===----------------------------------------------------------------------===//
```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 164-170
```cpp

class Lexer {
public:
  Lexer(llvm::SourceMgr &mgr, ast::DiagnosticEngine &diagEngine,
        CodeCompleteContext *codeCompleteContext);
  ~Lexer();

```
- **EN**: Introduces declarations for `Lexer`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `Lexer` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 171-177
```cpp
  /// Return a reference to the source manager used by the lexer.
  llvm::SourceMgr &getSourceMgr() { return srcMgr; }

  /// Return a reference to the diagnostic engine used by the lexer.
  ast::DiagnosticEngine &getDiagEngine() { return diagEngine; }

  /// Push an include of the given file. This will cause the lexer to start
```
- **EN**: Implements logic around `getSourceMgr`, `getDiagEngine`; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `getSourceMgr`、`getDiagEngine` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 178-184
```cpp
  /// processing the provided file. Returns failure if the file could not be
  /// opened, success otherwise.
  LogicalResult pushInclude(StringRef filename, SMRange includeLoc);

  /// Lex the next token and return it.
  Token lexToken();

```
- **EN**: Declares APIs around `pushInclude`, `lexToken`; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 声明与 `pushInclude`、`lexToken` 相关的 API；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 185-194
```cpp
  /// Change the position of the lexer cursor. The next token we lex will start
  /// at the designated point in the input.
  void resetPointer(const char *newPointer) { curPtr = newPointer; }

  /// Emit an error to the lexer with the given location and message.
  Token emitError(SMRange loc, const Twine &msg);
  Token emitError(const char *loc, const Twine &msg);
  Token emitErrorAndNote(SMRange loc, const Twine &msg, SMRange noteLoc,
                         const Twine &note);

```
- **EN**: Implements logic around `resetPointer`, `emitError`, `emitErrorAndNote`; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects.
- **CN**: 围绕 `resetPointer`、`emitError`、`emitErrorAndNote` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象。

### Lines 195-202
```cpp
private:
  Token formToken(Token::Kind kind, const char *tokStart) {
    return Token(kind, StringRef(tokStart, curPtr - tokStart));
  }

  /// Return the next character in the stream.
  int getNextChar();

```
- **EN**: Implements logic around `formToken`, `Token`, `getNextChar`.
- **CN**: 围绕 `formToken`、`Token`、`getNextChar` 实现具体逻辑。

### Lines 203-209
```cpp
  /// Lex methods.
  void lexComment();
  Token lexDirective(const char *tokStart);
  Token lexIdentifier(const char *tokStart);
  Token lexNumber(const char *tokStart);
  Token lexString(const char *tokStart, bool isStringBlock);

```
- **EN**: Declares APIs around `lexComment`, `lexDirective`, `lexIdentifier`, `lexNumber`, and 1 more symbols.
- **CN**: 声明与 `lexComment`、`lexDirective`、`lexIdentifier`、`lexNumber` 等另外 1 个符号 相关的 API。

### Lines 210-217
```cpp
  llvm::SourceMgr &srcMgr;
  int curBufferID;
  StringRef curBuffer;
  const char *curPtr;

  /// The engine used to emit diagnostics during lexing/parsing.
  ast::DiagnosticEngine &diagEngine;

```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 218-227
```cpp
  /// A flag indicating if we added a default diagnostic handler to the provided
  /// diagEngine.
  bool addedHandlerToDiagEngine;

  /// The optional code completion point within the input file.
  const char *codeCompletionLocation;
};
} // namespace pdll
} // namespace mlir

```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 228-228
```cpp
#endif // LIB_TOOLS_PDLL_PARSER_LEXER_H_
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

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `mlir/Support/LLVM.h`, `llvm/ADT/StringRef.h`, `llvm/Support/SMLoc.h`
- **Subsystem categories / 子系统类别**: shared MLIR support helpers / 共享的 MLIR 支持工具 (1), LLVM ADT containers and utilities / LLVM ADT 容器与工具 (1), LLVM support-library helpers / LLVM Support 库辅助工具 (1)
