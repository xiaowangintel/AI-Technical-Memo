# Parser.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Query/Matcher/Parser.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Simple matcher expression parser.
  - **CN**: 声明 MLIR 查询能力、匹配器基础设施或命令行查询行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===--- Parser.h - Matcher expression parser -------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及高层摘要。

### Lines 8-21
```cpp
//
// Simple matcher expression parser.
//
// This file contains the Parser class, which is responsible for parsing
// expressions in a specific format: matcherName(Arg0, Arg1, ..., ArgN). The
// parser can also interpret simple types, like strings.
//
// The actual processing of the matchers is handled by a Sema object that is
// provided to the parser.
//
// The grammar for the supported expressions is as follows:
// <Expression>        := <Literal> | <MatcherExpression>
// <Literal>           := <StringLiteral> | <NumericLiteral> | <BooleanLiteral>
// <StringLiteral>     := "quoted string"
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 22-28
```cpp
// <BooleanLiteral>    := "true" | "false"
// <NumericLiteral>    := [0-9]+
// <MatcherExpression> := <MatcherName>(<ArgumentList>)
// <MatcherName>       := [a-zA-Z]+
// <ArgumentList>      := <Expression> | <Expression>,<ArgumentList>
//
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 29-40
```cpp

#ifndef MLIR_TOOLS_MLIRQUERY_MATCHER_PARSER_H
#define MLIR_TOOLS_MLIRQUERY_MATCHER_PARSER_H

#include "Diagnostics.h"
#include "RegistryManager.h"
#include "llvm/ADT/ArrayRef.h"
#include "llvm/ADT/StringMap.h"
#include "llvm/ADT/StringRef.h"
#include <memory>
#include <vector>

```
- **EN**: Pulls in the declarations needed by this translation unit, including `Diagnostics.h`, `RegistryManager.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/StringMap.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `Diagnostics.h`, `RegistryManager.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/StringMap.h`。

### Lines 41-47
```cpp
namespace mlir::query::matcher::internal {

// Matcher expression parser.
class Parser {
public:
  // Different possible tokens.
  enum class TokenKind {
```
- **EN**: Introduces declarations for `mlir::query::matcher::internal`, `Parser`, `TokenKind`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `mlir::query::matcher::internal`、`Parser`、`TokenKind` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 48-60
```cpp
    Eof,
    NewLine,
    OpenParen,
    CloseParen,
    Comma,
    Period,
    Literal,
    Ident,
    InvalidChar,
    CodeCompletion,
    Error
  };

```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 61-67
```cpp
  // Interface to connect the parser with the registry and more. The parser uses
  // the Sema instance passed into parseMatcherExpression() to handle all
  // matcher tokens.
  class Sema {
  public:
    virtual ~Sema();

```
- **EN**: Introduces declarations for `Sema`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `Sema` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 68-77
```cpp
    // Process a matcher expression. The caller takes ownership of the Matcher
    // object returned.
    virtual VariantMatcher actOnMatcherExpression(
        MatcherCtor ctor, SourceRange nameRange, llvm::StringRef functionName,
        llvm::ArrayRef<ParserValue> args, Diagnostics *error) = 0;

    // Look up a matcher by name in the matcher name found by the parser.
    virtual std::optional<MatcherCtor>
    lookupMatcherCtor(llvm::StringRef matcherName) = 0;

```
- **EN**: Declares APIs around `actOnMatcherExpression`, `lookupMatcherCtor`; this block parses or prints textual MLIR representations.
- **CN**: 声明与 `actOnMatcherExpression`、`lookupMatcherCtor` 相关的 API；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 78-86
```cpp
    // Compute the list of completion types for Context.
    virtual std::vector<ArgKind> getAcceptedCompletionTypes(
        llvm::ArrayRef<std::pair<MatcherCtor, unsigned>> Context);

    // Compute the list of completions that match any of acceptedTypes.
    virtual std::vector<MatcherCompletion>
    getMatcherCompletions(llvm::ArrayRef<ArgKind> acceptedTypes);
  };

```
- **EN**: Declares APIs around `getAcceptedCompletionTypes`, `getMatcherCompletions`.
- **CN**: 声明与 `getAcceptedCompletionTypes`、`getMatcherCompletions` 相关的 API。

### Lines 87-94
```cpp
  // An implementation of the Sema interface that uses the matcher registry to
  // process tokens.
  class RegistrySema : public Parser::Sema {
  public:
    RegistrySema(const Registry &matcherRegistry)
        : matcherRegistry(matcherRegistry) {}
    ~RegistrySema() override;

```
- **EN**: Introduces declarations for `RegistrySema`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `RegistrySema` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 95-103
```cpp
    std::optional<MatcherCtor>
    lookupMatcherCtor(llvm::StringRef matcherName) override;

    VariantMatcher actOnMatcherExpression(MatcherCtor Ctor,
                                          SourceRange NameRange,
                                          StringRef functionName,
                                          ArrayRef<ParserValue> Args,
                                          Diagnostics *Error) override;

```
- **EN**: Declares APIs around `lookupMatcherCtor`, `actOnMatcherExpression`; this block parses or prints textual MLIR representations.
- **CN**: 声明与 `lookupMatcherCtor`、`actOnMatcherExpression` 相关的 API；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 104-110
```cpp
    std::vector<ArgKind> getAcceptedCompletionTypes(
        llvm::ArrayRef<std::pair<MatcherCtor, unsigned>> context) override;

    std::vector<MatcherCompletion>
    getMatcherCompletions(llvm::ArrayRef<ArgKind> acceptedTypes) override;

  private:
```
- **EN**: Declares APIs around `getAcceptedCompletionTypes`, `getMatcherCompletions`.
- **CN**: 声明与 `getAcceptedCompletionTypes`、`getMatcherCompletions` 相关的 API。

### Lines 111-124
```cpp
    const Registry &matcherRegistry;
  };

  using NamedValueMap = llvm::StringMap<VariantValue>;

  // Methods to parse a matcher expression and return a DynMatcher object,
  // transferring ownership to the caller.
  static std::optional<DynMatcher>
  parseMatcherExpression(llvm::StringRef &matcherCode,
                         const Registry &matcherRegistry,
                         const NamedValueMap *namedValues, Diagnostics *error);
  static std::optional<DynMatcher>
  parseMatcherExpression(llvm::StringRef &matcherCode,
                         const Registry &matcherRegistry, Diagnostics *error) {
```
- **EN**: Implements logic around `parseMatcherExpression`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `parseMatcherExpression` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 125-133
```cpp
    return parseMatcherExpression(matcherCode, matcherRegistry, nullptr, error);
  }

  // Methods to parse any expression supported by this parser.
  static bool parseExpression(llvm::StringRef &code,
                              const Registry &matcherRegistry,
                              const NamedValueMap *namedValues,
                              VariantValue *value, Diagnostics *error);

```
- **EN**: Declares APIs around `parseMatcherExpression`, `parseExpression`; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 声明与 `parseMatcherExpression`、`parseExpression` 相关的 API；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 134-147
```cpp
  static bool parseExpression(llvm::StringRef &code,
                              const Registry &matcherRegistry,
                              VariantValue *value, Diagnostics *error) {
    return parseExpression(code, matcherRegistry, nullptr, value, error);
  }

  // Methods to complete an expression at a given offset.
  static std::vector<MatcherCompletion>
  completeExpression(llvm::StringRef &code, unsigned completionOffset,
                     const Registry &matcherRegistry,
                     const NamedValueMap *namedValues);
  static std::vector<MatcherCompletion>
  completeExpression(llvm::StringRef &code, unsigned completionOffset,
                     const Registry &matcherRegistry) {
```
- **EN**: Implements logic around `parseExpression`, `completeExpression`; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `parseExpression`、`completeExpression` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 148-154
```cpp
    return completeExpression(code, completionOffset, matcherRegistry, nullptr);
  }

private:
  class CodeTokenizer;
  struct ScopedContextEntry;
  struct TokenInfo;
```
- **EN**: Introduces declarations for `CodeTokenizer`, `ScopedContextEntry`, `TokenInfo`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `CodeTokenizer`、`ScopedContextEntry`、`TokenInfo` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 155-162
```cpp

  Parser(CodeTokenizer *tokenizer, const Registry &matcherRegistry,
         const NamedValueMap *namedValues, Diagnostics *error);

  bool parseChainedExpression(std::string &argument);

  bool parseExpressionImpl(VariantValue *value);

```
- **EN**: Declares APIs around `Parser`, `parseChainedExpression`, `parseExpressionImpl`; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 声明与 `Parser`、`parseChainedExpression`、`parseExpressionImpl` 相关的 API；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 163-170
```cpp
  bool parseMatcherArgs(std::vector<ParserValue> &args, MatcherCtor ctor,
                        const TokenInfo &nameToken, TokenInfo &endToken);

  bool parseMatcherExpressionImpl(const TokenInfo &nameToken,
                                  const TokenInfo &openToken,
                                  std::optional<MatcherCtor> ctor,
                                  VariantValue *value);

```
- **EN**: Declares APIs around `parseMatcherArgs`, `parseMatcherExpressionImpl`; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 声明与 `parseMatcherArgs`、`parseMatcherExpressionImpl` 相关的 API；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 171-179
```cpp
  bool parseIdentifierPrefixImpl(VariantValue *value);

  void addCompletion(const TokenInfo &compToken,
                     const MatcherCompletion &completion);
  void addExpressionCompletions();

  std::vector<MatcherCompletion>
  getNamedValueCompletions(llvm::ArrayRef<ArgKind> acceptedTypes);

```
- **EN**: Declares APIs around `parseIdentifierPrefixImpl`, `addCompletion`, `addExpressionCompletions`, `getNamedValueCompletions`; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 声明与 `parseIdentifierPrefixImpl`、`addCompletion`、`addExpressionCompletions`、`getNamedValueCompletions` 相关的 API；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 180-186
```cpp
  CodeTokenizer *const tokenizer;
  std::unique_ptr<RegistrySema> sema;
  const NamedValueMap *const namedValues;
  Diagnostics *const error;

  using ContextStackTy = std::vector<std::pair<MatcherCtor, unsigned>>;

```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 187-193
```cpp
  ContextStackTy contextStack;
  std::vector<MatcherCompletion> completions;
};

} // namespace mlir::query::matcher::internal

#endif // MLIR_TOOLS_MLIRQUERY_MATCHER_PARSER_H
```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

## Key Concepts / 关键概念

- **IR querying / IR 查询**:
  - **EN**: Supports querying or filtering IR according to structural or semantic predicates.
  - **CN**: 支持依据结构或语义谓词对 IR 进行查询或过滤。
- **SSA and region-based IR / SSA 与基于 Region 的 IR**:
  - **EN**: The file works with MLIR operations, blocks, values, or regions that form MLIR's hierarchical SSA IR.
  - **CN**: 该文件处理 MLIR 操作、块、值或 Region，它们共同构成 MLIR 的层次化 SSA IR。
- **LLVM ADT containers / LLVM ADT 容器**:
  - **EN**: The implementation uses LLVM containers for performance-conscious storage and traversal.
  - **CN**: 该实现使用 LLVM 容器来进行兼顾性能的存储与遍历。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `Diagnostics.h`, `RegistryManager.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/StringMap.h`, `llvm/ADT/StringRef.h`
- **Standard-library headers / 标准库头文件**: `<memory>`, `<vector>`
- **Subsystem categories / 子系统类别**: LLVM ADT containers and utilities / LLVM ADT 容器与工具 (3)
