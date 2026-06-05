# Parser.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Query/Matcher/Parser.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Recursive parser implementation for the matcher expression grammar.
  - **CN**: 实现 MLIR 查询能力、匹配器基础设施或命令行查询行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===- Parser.cpp - Matcher expression parser -----------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Recursive parser implementation for the matcher expression grammar.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及高层摘要。

### Lines 12-22
```cpp

#include "Parser.h"

#include <vector>

namespace mlir::query::matcher::internal {

// Simple structure to hold information for one token from the parser.
struct Parser::TokenInfo {
  TokenInfo() = default;

```
- **EN**: Pulls in the declarations needed by this translation unit, including `Parser.h`, `vector`.
- **CN**: 引入该编译单元所需的声明，其中包括 `Parser.h`, `vector`。

### Lines 23-37
```cpp
  // Method to set the kind and text of the token
  void set(TokenKind newKind, llvm::StringRef newText) {
    kind = newKind;
    text = newText;
  }

  // Known identifiers.
  static const char *const idExtract;

  llvm::StringRef text;
  TokenKind kind = TokenKind::Eof;
  SourceRange range;
  VariantValue value;
};

```
- **EN**: Implements logic around `set`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `set` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 38-47
```cpp
const char *const Parser::TokenInfo::idExtract = "extract";

class Parser::CodeTokenizer {
public:
  // Constructor with matcherCode and error
  explicit CodeTokenizer(llvm::StringRef matcherCode, Diagnostics *error)
      : code(matcherCode), startOfLine(matcherCode), error(error) {
    nextToken = getNextToken();
  }

```
- **EN**: Introduces declarations for `Parser::CodeTokenizer`, `Parser`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `Parser::CodeTokenizer`、`Parser` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 48-58
```cpp
  // Constructor with matcherCode, error, and codeCompletionOffset
  CodeTokenizer(llvm::StringRef matcherCode, Diagnostics *error,
                unsigned codeCompletionOffset)
      : code(matcherCode), startOfLine(matcherCode), error(error),
        codeCompletionLocation(matcherCode.data() + codeCompletionOffset) {
    nextToken = getNextToken();
  }

  // Peek at next token without consuming it
  const TokenInfo &peekNextToken() const { return nextToken; }

```
- **EN**: Implements logic around `CodeTokenizer`, `code`, `codeCompletionLocation`, `getNextToken`, and 1 more symbols.
- **CN**: 围绕 `CodeTokenizer`、`code`、`codeCompletionLocation`、`getNextToken` 等另外 1 个符号 实现具体逻辑。

### Lines 59-72
```cpp
  // Consume and return the next token
  TokenInfo consumeNextToken() {
    TokenInfo thisToken = nextToken;
    nextToken = getNextToken();
    return thisToken;
  }

  // Skip any newline tokens
  TokenInfo skipNewlines() {
    while (nextToken.kind == TokenKind::NewLine)
      nextToken = getNextToken();
    return nextToken;
  }

```
- **EN**: Implements logic around `consumeNextToken`, `getNextToken`, `skipNewlines`.
- **CN**: 围绕 `consumeNextToken`、`getNextToken`、`skipNewlines` 实现具体逻辑。

### Lines 73-82
```cpp
  // Consume and return next token, ignoring newlines
  TokenInfo consumeNextTokenIgnoreNewlines() {
    skipNewlines();
    return nextToken.kind == TokenKind::Eof ? nextToken : consumeNextToken();
  }

  // Return kind of next token
  TokenKind nextTokenKind() const { return nextToken.kind; }

private:
```
- **EN**: Implements logic around `consumeNextTokenIgnoreNewlines`, `skipNewlines`, `consumeNextToken`, `nextTokenKind`.
- **CN**: 围绕 `consumeNextTokenIgnoreNewlines`、`skipNewlines`、`consumeNextToken`、`nextTokenKind` 实现具体逻辑。

### Lines 83-97
```cpp
  // Helper function to get the first character as a new StringRef and drop it
  // from the original string
  llvm::StringRef firstCharacterAndDrop(llvm::StringRef &str) {
    assert(!str.empty());
    llvm::StringRef firstChar = str.substr(0, 1);
    str = str.drop_front();
    return firstChar;
  }

  // Get next token, consuming whitespaces and handling different token types
  TokenInfo getNextToken() {
    consumeWhitespace();
    TokenInfo result;
    result.range.start = currentLocation();

```
- **EN**: Implements logic around `firstCharacterAndDrop`, `assert`, `substr`, `drop_front`, and 3 more symbols.
- **CN**: 围绕 `firstCharacterAndDrop`、`assert`、`substr`、`drop_front` 等另外 3 个符号 实现具体逻辑。

### Lines 98-111
```cpp
    // Code completion case
    if (codeCompletionLocation && codeCompletionLocation <= code.data()) {
      result.set(TokenKind::CodeCompletion,
                 llvm::StringRef(codeCompletionLocation, 0));
      codeCompletionLocation = nullptr;
      return result;
    }

    // End of file case
    if (code.empty()) {
      result.set(TokenKind::Eof, "");
      return result;
    }

```
- **EN**: Implements logic around `data`, `set`, `StringRef`, `empty`.
- **CN**: 围绕 `data`、`set`、`StringRef`、`empty` 实现具体逻辑。

### Lines 112-131
```cpp
    // Switch to handle specific characters
    switch (code[0]) {
    case '#':
      code = code.drop_until([](char c) { return c == '\n'; });
      return getNextToken();
    case ',':
      result.set(TokenKind::Comma, firstCharacterAndDrop(code));
      break;
    case '.':
      result.set(TokenKind::Period, firstCharacterAndDrop(code));
      break;
    case '\n':
      ++line;
      startOfLine = code.drop_front();
      result.set(TokenKind::NewLine, firstCharacterAndDrop(code));
      break;
    case '(':
      result.set(TokenKind::OpenParen, firstCharacterAndDrop(code));
      break;
    case ')':
```
- **EN**: Implements logic around `drop_until`, `getNextToken`, `set`, `drop_front`.
- **CN**: 围绕 `drop_until`、`getNextToken`、`set`、`drop_front` 实现具体逻辑。

### Lines 132-151
```cpp
      result.set(TokenKind::CloseParen, firstCharacterAndDrop(code));
      break;
    case '"':
    case '\'':
      consumeStringLiteral(&result);
      break;
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
      consumeNumberLiteral(&result);
      break;
    default:
      parseIdentifierOrInvalid(&result);
```
- **EN**: Implements logic around `set`, `consumeStringLiteral`, `consumeNumberLiteral`, `parseIdentifierOrInvalid`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `set`、`consumeStringLiteral`、`consumeNumberLiteral`、`parseIdentifierOrInvalid` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 152-170
```cpp
      break;
    }

    result.range.end = currentLocation();
    return result;
  }

  void consumeNumberLiteral(TokenInfo *result) {
    StringRef original = code;
    unsigned value = 0;
    if (!code.consumeInteger(0, value)) {
      size_t numConsumed = original.size() - code.size();
      result->text = original.take_front(numConsumed);
      result->kind = TokenKind::Literal;
      result->value = static_cast<int64_t>(value);
      return;
    }
  }

```
- **EN**: Implements logic around `currentLocation`, `consumeNumberLiteral`, `consumeInteger`, `size`, and 2 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `currentLocation`、`consumeNumberLiteral`、`consumeInteger`、`size` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 171-190
```cpp
  // Consume a string literal, handle escape sequences and missing closing
  // quote.
  void consumeStringLiteral(TokenInfo *result) {
    bool inEscape = false;
    const char marker = code[0];
    for (size_t length = 1; length < code.size(); ++length) {
      if (inEscape) {
        inEscape = false;
        continue;
      }
      if (code[length] == '\\') {
        inEscape = true;
        continue;
      }
      if (code[length] == marker) {
        result->kind = TokenKind::Literal;
        result->text = code.substr(0, length + 1);
        result->value = code.substr(1, length - 1);
        code = code.drop_front(length + 1);
        return;
```
- **EN**: Implements logic around `consumeStringLiteral`, `size`, `substr`, `drop_front`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `consumeStringLiteral`、`size`、`substr`、`drop_front` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 191-201
```cpp
      }
    }
    llvm::StringRef errorText = code;
    code = code.drop_front(code.size());
    SourceRange range;
    range.start = result->range.start;
    range.end = currentLocation();
    error->addError(range, ErrorType::ParserStringError) << errorText;
    result->kind = TokenKind::Error;
  }

```
- **EN**: Implements logic around `drop_front`, `currentLocation`, `addError`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `drop_front`、`currentLocation`、`addError` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 202-221
```cpp
  void parseIdentifierOrInvalid(TokenInfo *result) {
    if (isalnum(code[0])) {
      // Parse an identifier
      size_t tokenLength = 1;

      while (true) {
        // A code completion location in/immediately after an identifier will
        // cause the portion of the identifier before the code completion
        // location to become a code completion token.
        if (codeCompletionLocation == code.data() + tokenLength) {
          codeCompletionLocation = nullptr;
          result->kind = TokenKind::CodeCompletion;
          result->text = code.substr(0, tokenLength);
          code = code.drop_front(tokenLength);
          return;
        }
        if (tokenLength == code.size() || !(isalnum(code[tokenLength])))
          break;
        ++tokenLength;
      }
```
- **EN**: Implements logic around `parseIdentifierOrInvalid`, `isalnum`, `data`, `substr`, and 2 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `parseIdentifierOrInvalid`、`isalnum`、`data`、`substr` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 222-241
```cpp
      llvm::StringRef token = code.substr(0, tokenLength);
      code = code.drop_front(tokenLength);
      // Check if the identifier is a boolean literal
      if (token == "true") {
        result->text = "false";
        result->kind = TokenKind::Literal;
        result->value = true;
      } else if (token == "false") {
        result->text = "false";
        result->kind = TokenKind::Literal;
        result->value = false;
      } else {
        // Otherwise it is treated as a normal identifier
        result->kind = TokenKind::Ident;
        result->text = token;
      }
    } else {
      result->kind = TokenKind::InvalidChar;
      result->text = code.substr(0, 1);
      code = code.drop_front(1);
```
- **EN**: Implements logic around `substr`, `drop_front`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `substr`、`drop_front` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 242-255
```cpp
    }
  }

  // Consume all leading whitespace from code, except newlines
  void consumeWhitespace() { code = code.ltrim(" \t\v\f\r"); }

  // Returns the current location in the source code
  SourceLocation currentLocation() {
    SourceLocation location;
    location.line = line;
    location.column = code.data() - startOfLine.data() + 1;
    return location;
  }

```
- **EN**: Implements logic around `consumeWhitespace`, `currentLocation`, `data`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `consumeWhitespace`、`currentLocation`、`data` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 256-265
```cpp
  llvm::StringRef code;
  llvm::StringRef startOfLine;
  unsigned line = 1;
  Diagnostics *error;
  TokenInfo nextToken;
  const char *codeCompletionLocation = nullptr;
};

Parser::Sema::~Sema() = default;

```
- **EN**: Implements logic around `~Sema`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `~Sema` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 266-275
```cpp
std::vector<ArgKind> Parser::Sema::getAcceptedCompletionTypes(
    llvm::ArrayRef<std::pair<MatcherCtor, unsigned>> context) {
  return {};
}

std::vector<MatcherCompletion>
Parser::Sema::getMatcherCompletions(llvm::ArrayRef<ArgKind> acceptedTypes) {
  return {};
}

```
- **EN**: Implements logic around `getAcceptedCompletionTypes`, `getMatcherCompletions`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `getAcceptedCompletionTypes`、`getMatcherCompletions` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 276-285
```cpp
// Entry for the scope of a parser
struct Parser::ScopedContextEntry {
  Parser *parser;

  ScopedContextEntry(Parser *parser, MatcherCtor c) : parser(parser) {
    parser->contextStack.emplace_back(c, 0u);
  }

  ~ScopedContextEntry() { parser->contextStack.pop_back(); }

```
- **EN**: Introduces declarations for `Parser::ScopedContextEntry`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `Parser::ScopedContextEntry` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 286-299
```cpp
  void nextArg() { ++parser->contextStack.back().second; }
};

// Parse and validate expressions starting with an identifier.
// This function can parse named values and matchers. In case of failure, it
// will try to determine the user's intent to give an appropriate error message.
bool Parser::parseIdentifierPrefixImpl(VariantValue *value) {
  const TokenInfo nameToken = tokenizer->consumeNextToken();

  if (tokenizer->nextTokenKind() != TokenKind::OpenParen) {
    // Parse as a named value.
    if (auto namedValue = namedValues ? namedValues->lookup(nameToken.text)
                                      : VariantValue()) {

```
- **EN**: Implements logic around `nextArg`, `parseIdentifierPrefixImpl`, `consumeNextToken`, `nextTokenKind`, and 2 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `nextArg`、`parseIdentifierPrefixImpl`、`consumeNextToken`、`nextTokenKind` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 300-311
```cpp
      if (tokenizer->nextTokenKind() != TokenKind::Period) {
        *value = namedValue;
        return true;
      }

      if (!namedValue.isMatcher()) {
        error->addError(tokenizer->peekNextToken().range,
                        ErrorType::ParserNotAMatcher);
        return false;
      }
    }

```
- **EN**: Implements logic around `nextTokenKind`, `isMatcher`, `addError`; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `nextTokenKind`、`isMatcher`、`addError` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 312-331
```cpp
    if (tokenizer->nextTokenKind() == TokenKind::NewLine) {
      error->addError(tokenizer->peekNextToken().range,
                      ErrorType::ParserNoOpenParen)
          << "NewLine";
      return false;
    }

    // If the syntax is correct and the name is not a matcher either, report
    // an unknown named value.
    if ((tokenizer->nextTokenKind() == TokenKind::Comma ||
         tokenizer->nextTokenKind() == TokenKind::CloseParen ||
         tokenizer->nextTokenKind() == TokenKind::NewLine ||
         tokenizer->nextTokenKind() == TokenKind::Eof) &&
        !sema->lookupMatcherCtor(nameToken.text)) {
      error->addError(nameToken.range, ErrorType::RegistryValueNotFound)
          << nameToken.text;
      return false;
    }
    // Otherwise, fallback to the matcher parser.
  }
```
- **EN**: Implements logic around `nextTokenKind`, `addError`, `lookupMatcherCtor`; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `nextTokenKind`、`addError`、`lookupMatcherCtor` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 332-342
```cpp

  tokenizer->skipNewlines();

  assert(nameToken.kind == TokenKind::Ident);
  TokenInfo openToken = tokenizer->consumeNextToken();
  if (openToken.kind != TokenKind::OpenParen) {
    error->addError(openToken.range, ErrorType::ParserNoOpenParen)
        << openToken.text;
    return false;
  }

```
- **EN**: Implements logic around `skipNewlines`, `assert`, `consumeNextToken`, `addError`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `skipNewlines`、`assert`、`consumeNextToken`、`addError` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 343-356
```cpp
  std::optional<MatcherCtor> ctor = sema->lookupMatcherCtor(nameToken.text);

  // Parse as a matcher expression.
  return parseMatcherExpressionImpl(nameToken, openToken, ctor, value);
}

bool Parser::parseChainedExpression(std::string &argument) {
  // Parse the parenthesized argument to .extract("foo")
  // Note: EOF is handled inside the consume functions and would fail below when
  // checking token kind.
  const TokenInfo openToken = tokenizer->consumeNextToken();
  const TokenInfo argumentToken = tokenizer->consumeNextTokenIgnoreNewlines();
  const TokenInfo closeToken = tokenizer->consumeNextTokenIgnoreNewlines();

```
- **EN**: Implements logic around `lookupMatcherCtor`, `parseMatcherExpressionImpl`, `parseChainedExpression`, `consumeNextToken`, and 1 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `lookupMatcherCtor`、`parseMatcherExpressionImpl`、`parseChainedExpression`、`consumeNextToken` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 357-368
```cpp
  if (openToken.kind != TokenKind::OpenParen) {
    error->addError(openToken.range, ErrorType::ParserChainedExprNoOpenParen);
    return false;
  }

  if (argumentToken.kind != TokenKind::Literal ||
      !argumentToken.value.isString()) {
    error->addError(argumentToken.range,
                    ErrorType::ParserChainedExprInvalidArg);
    return false;
  }

```
- **EN**: Implements logic around `addError`, `isString`; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `addError`、`isString` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 369-378
```cpp
  if (closeToken.kind != TokenKind::CloseParen) {
    error->addError(closeToken.range, ErrorType::ParserChainedExprNoCloseParen);
    return false;
  }

  // If all checks passed, extract the argument and return true.
  argument = argumentToken.value.getString();
  return true;
}

```
- **EN**: Implements logic around `addError`, `getString`; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `addError`、`getString` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 379-390
```cpp
// Parse the arguments of a matcher
bool Parser::parseMatcherArgs(std::vector<ParserValue> &args, MatcherCtor ctor,
                              const TokenInfo &nameToken, TokenInfo &endToken) {
  ScopedContextEntry sce(this, ctor);

  while (tokenizer->nextTokenKind() != TokenKind::Eof) {
    if (tokenizer->nextTokenKind() == TokenKind::CloseParen) {
      // end of args.
      endToken = tokenizer->consumeNextToken();
      break;
    }

```
- **EN**: Implements logic around `parseMatcherArgs`, `sce`, `nextTokenKind`, `consumeNextToken`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `parseMatcherArgs`、`sce`、`nextTokenKind`、`consumeNextToken` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 391-400
```cpp
    if (!args.empty()) {
      // We must find a , token to continue.
      TokenInfo commaToken = tokenizer->consumeNextToken();
      if (commaToken.kind != TokenKind::Comma) {
        error->addError(commaToken.range, ErrorType::ParserNoComma)
            << commaToken.text;
        return false;
      }
    }

```
- **EN**: Implements logic around `empty`, `consumeNextToken`, `addError`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `empty`、`consumeNextToken`、`addError` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 401-414
```cpp
    ParserValue argValue;
    tokenizer->skipNewlines();

    argValue.text = tokenizer->peekNextToken().text;
    argValue.range = tokenizer->peekNextToken().range;
    if (!parseExpressionImpl(&argValue.value)) {
      return false;
    }

    tokenizer->skipNewlines();
    args.push_back(argValue);
    sce.nextArg();
  }

```
- **EN**: Implements logic around `skipNewlines`, `peekNextToken`, `parseExpressionImpl`, `push_back`, and 1 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `skipNewlines`、`peekNextToken`、`parseExpressionImpl`、`push_back` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 415-428
```cpp
  return true;
}

// Parse and validate a matcher expression.
bool Parser::parseMatcherExpressionImpl(const TokenInfo &nameToken,
                                        const TokenInfo &openToken,
                                        std::optional<MatcherCtor> ctor,
                                        VariantValue *value) {
  if (!ctor) {
    error->addError(nameToken.range, ErrorType::RegistryMatcherNotFound)
        << nameToken.text;
    // Do not return here. We need to continue to give completion suggestions.
  }

```
- **EN**: Implements logic around `parseMatcherExpressionImpl`, `addError`; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `parseMatcherExpressionImpl`、`addError` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 429-444
```cpp
  std::vector<ParserValue> args;
  TokenInfo endToken;

  tokenizer->skipNewlines();

  if (!parseMatcherArgs(args, ctor.value_or(nullptr), nameToken, endToken)) {
    return false;
  }

  // Check for the missing closing parenthesis
  if (endToken.kind != TokenKind::CloseParen) {
    error->addError(openToken.range, ErrorType::ParserNoCloseParen)
        << nameToken.text;
    return false;
  }

```
- **EN**: Implements logic around `skipNewlines`, `parseMatcherArgs`, `addError`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `skipNewlines`、`parseMatcherArgs`、`addError` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 445-460
```cpp
  std::string functionName;
  if (tokenizer->peekNextToken().kind == TokenKind::Period) {
    tokenizer->consumeNextToken();
    TokenInfo chainCallToken = tokenizer->consumeNextToken();
    if (chainCallToken.kind == TokenKind::CodeCompletion) {
      addCompletion(chainCallToken, MatcherCompletion("extract(\"", "extract"));
      return false;
    }

    if (chainCallToken.kind != TokenKind::Ident ||
        chainCallToken.text != TokenInfo::idExtract) {
      error->addError(chainCallToken.range,
                      ErrorType::ParserMalformedChainedExpr);
      return false;
    }

```
- **EN**: Implements logic around `peekNextToken`, `consumeNextToken`, `addCompletion`, `addError`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `peekNextToken`、`consumeNextToken`、`addCompletion`、`addError` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 461-478
```cpp
    if (chainCallToken.text == TokenInfo::idExtract &&
        !parseChainedExpression(functionName))
      return false;
  }

  if (!ctor)
    return false;
  // Merge the start and end infos.
  SourceRange matcherRange = nameToken.range;
  matcherRange.end = endToken.range.end;
  VariantMatcher result = sema->actOnMatcherExpression(
      *ctor, matcherRange, functionName, args, error);
  if (result.isNull())
    return false;
  *value = result;
  return true;
}

```
- **EN**: Implements logic around `parseChainedExpression`, `actOnMatcherExpression`, `isNull`; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `parseChainedExpression`、`actOnMatcherExpression`、`isNull` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 479-488
```cpp
// If the prefix of this completion matches the completion token, add it to
// completions minus the prefix.
void Parser::addCompletion(const TokenInfo &compToken,
                           const MatcherCompletion &completion) {
  if (llvm::StringRef(completion.typedText).starts_with(compToken.text)) {
    completions.emplace_back(completion.typedText.substr(compToken.text.size()),
                             completion.matcherDecl);
  }
}

```
- **EN**: Implements logic around `addCompletion`, `StringRef`, `emplace_back`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `addCompletion`、`StringRef`、`emplace_back` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 489-502
```cpp
std::vector<MatcherCompletion>
Parser::getNamedValueCompletions(llvm::ArrayRef<ArgKind> acceptedTypes) {
  if (!namedValues)
    return {};

  std::vector<MatcherCompletion> result;
  for (const auto &entry : *namedValues) {
    std::string decl =
        (entry.getValue().getTypeAsString() + " " + entry.getKey()).str();
    result.emplace_back(entry.getKey(), decl);
  }
  return result;
}

```
- **EN**: Implements logic around `getNamedValueCompletions`, `getValue`, `emplace_back`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `getNamedValueCompletions`、`getValue`、`emplace_back` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 503-513
```cpp
void Parser::addExpressionCompletions() {
  const TokenInfo compToken = tokenizer->consumeNextTokenIgnoreNewlines();
  assert(compToken.kind == TokenKind::CodeCompletion);

  // We cannot complete code if there is an invalid element on the context
  // stack.
  for (const auto &entry : contextStack) {
    if (!entry.first)
      return;
  }

```
- **EN**: Implements logic around `addExpressionCompletions`, `consumeNextTokenIgnoreNewlines`, `assert`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `addExpressionCompletions`、`consumeNextTokenIgnoreNewlines`、`assert` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 514-523
```cpp
  auto acceptedTypes = sema->getAcceptedCompletionTypes(contextStack);
  for (const auto &completion : sema->getMatcherCompletions(acceptedTypes)) {
    addCompletion(compToken, completion);
  }

  for (const auto &completion : getNamedValueCompletions(acceptedTypes)) {
    addCompletion(compToken, completion);
  }
}

```
- **EN**: Implements logic around `getAcceptedCompletionTypes`, `getMatcherCompletions`, `addCompletion`, `getNamedValueCompletions`.
- **CN**: 围绕 `getAcceptedCompletionTypes`、`getMatcherCompletions`、`addCompletion`、`getNamedValueCompletions` 实现具体逻辑。

### Lines 524-539
```cpp
// Parse an <Expresssion>
bool Parser::parseExpressionImpl(VariantValue *value) {
  switch (tokenizer->nextTokenKind()) {
  case TokenKind::Literal:
    *value = tokenizer->consumeNextToken().value;
    return true;
  case TokenKind::Ident:
    return parseIdentifierPrefixImpl(value);
  case TokenKind::CodeCompletion:
    addExpressionCompletions();
    return false;
  case TokenKind::Eof:
    error->addError(tokenizer->consumeNextToken().range,
                    ErrorType::ParserNoCode);
    return false;

```
- **EN**: Implements logic around `parseExpressionImpl`, `nextTokenKind`, `consumeNextToken`, `parseIdentifierPrefixImpl`, and 2 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `parseExpressionImpl`、`nextTokenKind`、`consumeNextToken`、`parseIdentifierPrefixImpl` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 540-554
```cpp
  case TokenKind::Error:
    // This error was already reported by the tokenizer.
    return false;
  case TokenKind::NewLine:
  case TokenKind::OpenParen:
  case TokenKind::CloseParen:
  case TokenKind::Comma:
  case TokenKind::Period:
  case TokenKind::InvalidChar:
    const TokenInfo token = tokenizer->consumeNextToken();
    error->addError(token.range, ErrorType::ParserInvalidToken)
        << (token.kind == TokenKind::NewLine ? "NewLine" : token.text);
    return false;
  }

```
- **EN**: Implements logic around `consumeNextToken`, `addError`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `consumeNextToken`、`addError` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 555-565
```cpp
  llvm_unreachable("Unknown token kind.");
}

Parser::Parser(CodeTokenizer *tokenizer, const Registry &matcherRegistry,
               const NamedValueMap *namedValues, Diagnostics *error)
    : tokenizer(tokenizer),
      sema(std::make_unique<RegistrySema>(matcherRegistry)),
      namedValues(namedValues), error(error) {}

Parser::RegistrySema::~RegistrySema() = default;

```
- **EN**: Implements logic around `llvm_unreachable`, `Parser`, `tokenizer`, `sema`, and 2 more symbols; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `llvm_unreachable`、`Parser`、`tokenizer`、`sema` 等另外 2 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 566-577
```cpp
std::optional<MatcherCtor>
Parser::RegistrySema::lookupMatcherCtor(llvm::StringRef matcherName) {
  return RegistryManager::lookupMatcherCtor(matcherName, matcherRegistry);
}

VariantMatcher Parser::RegistrySema::actOnMatcherExpression(
    MatcherCtor ctor, SourceRange nameRange, llvm::StringRef functionName,
    llvm::ArrayRef<ParserValue> args, Diagnostics *error) {
  return RegistryManager::constructMatcher(ctor, nameRange, functionName, args,
                                           error);
}

```
- **EN**: Implements logic around `lookupMatcherCtor`, `actOnMatcherExpression`, `constructMatcher`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `lookupMatcherCtor`、`actOnMatcherExpression`、`constructMatcher` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 578-587
```cpp
std::vector<ArgKind> Parser::RegistrySema::getAcceptedCompletionTypes(
    llvm::ArrayRef<std::pair<MatcherCtor, unsigned>> context) {
  return RegistryManager::getAcceptedCompletionTypes(context);
}

std::vector<MatcherCompletion> Parser::RegistrySema::getMatcherCompletions(
    llvm::ArrayRef<ArgKind> acceptedTypes) {
  return RegistryManager::getMatcherCompletions(acceptedTypes, matcherRegistry);
}

```
- **EN**: Implements logic around `getAcceptedCompletionTypes`, `getMatcherCompletions`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `getAcceptedCompletionTypes`、`getMatcherCompletions` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 588-605
```cpp
bool Parser::parseExpression(llvm::StringRef &code,
                             const Registry &matcherRegistry,
                             const NamedValueMap *namedValues,
                             VariantValue *value, Diagnostics *error) {
  CodeTokenizer tokenizer(code, error);
  Parser parser(&tokenizer, matcherRegistry, namedValues, error);
  if (!parser.parseExpressionImpl(value))
    return false;
  auto nextToken = tokenizer.peekNextToken();
  if (nextToken.kind != TokenKind::Eof &&
      nextToken.kind != TokenKind::NewLine) {
    error->addError(tokenizer.peekNextToken().range,
                    ErrorType::ParserTrailingCode);
    return false;
  }
  return true;
}

```
- **EN**: Implements logic around `parseExpression`, `tokenizer`, `parser`, `parseExpressionImpl`, and 2 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `parseExpression`、`tokenizer`、`parser`、`parseExpressionImpl` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 606-615
```cpp
std::vector<MatcherCompletion>
Parser::completeExpression(llvm::StringRef &code, unsigned completionOffset,
                           const Registry &matcherRegistry,
                           const NamedValueMap *namedValues) {
  Diagnostics error;
  CodeTokenizer tokenizer(code, &error, completionOffset);
  Parser parser(&tokenizer, matcherRegistry, namedValues, &error);
  VariantValue dummy;
  parser.parseExpressionImpl(&dummy);

```
- **EN**: Implements logic around `completeExpression`, `tokenizer`, `parser`, `parseExpressionImpl`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `completeExpression`、`tokenizer`、`parser`、`parseExpressionImpl` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 616-635
```cpp
  return parser.completions;
}

std::optional<DynMatcher> Parser::parseMatcherExpression(
    llvm::StringRef &code, const Registry &matcherRegistry,
    const NamedValueMap *namedValues, Diagnostics *error) {
  VariantValue value;
  if (!parseExpression(code, matcherRegistry, namedValues, &value, error))
    return std::nullopt;
  if (!value.isMatcher()) {
    error->addError(SourceRange(), ErrorType::ParserNotAMatcher);
    return std::nullopt;
  }
  std::optional<DynMatcher> result = value.getMatcher().getDynMatcher();
  if (!result) {
    error->addError(SourceRange(), ErrorType::ParserOverloadedType)
        << value.getTypeAsString();
  }
  return result;
}
```
- **EN**: Implements logic around `parseMatcherExpression`, `parseExpression`, `isMatcher`, `addError`, and 2 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `parseMatcherExpression`、`parseExpression`、`isMatcher`、`addError` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 636-637
```cpp

} // namespace mlir::query::matcher::internal
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

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `Parser.h`
- **Standard-library headers / 标准库头文件**: `<vector>`
