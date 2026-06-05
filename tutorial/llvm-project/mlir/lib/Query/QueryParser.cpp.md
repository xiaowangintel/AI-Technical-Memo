# QueryParser.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Query/QueryParser.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements MLIR query facilities, matcher infrastructure, or command-line querying behavior.
  - **CN**: 实现 MLIR 查询能力、匹配器基础设施或命令行查询行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===---- QueryParser.cpp - mlir-query command parser ---------------------===//
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

#include "QueryParser.h"
#include "llvm/ADT/StringSwitch.h"

namespace mlir::query {

// Lex any amount of whitespace followed by a "word" (any sequence of
// non-whitespace characters) from the start of region [begin,end).  If no word
// is found before end, return StringRef(). begin is adjusted to exclude the
// lexed region.
llvm::StringRef QueryParser::lexWord() {
  // Don't trim newlines.
  line = line.ltrim(" \t\v\f\r");

```
- **EN**: Pulls in the declarations needed by this translation unit, including `QueryParser.h`, `llvm/ADT/StringSwitch.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `QueryParser.h`, `llvm/ADT/StringSwitch.h`。

### Lines 22-35
```cpp
  if (line.empty())
    // Even though the line is empty, it contains a pointer and
    // a (zero) length. The pointer is used in the LexOrCompleteWord
    // code completion.
    return line;

  llvm::StringRef word;
  if (line.front() == '#') {
    word = line.substr(0, 1);
  } else {
    word = line.take_until([](char c) {
      // Don't trim newlines.
      return llvm::StringRef(" \t\v\f\r").contains(c);
    });
```
- **EN**: Implements logic around `empty`, `front`, `substr`, `take_until`, and 1 more symbols.
- **CN**: 围绕 `empty`、`front`、`substr`、`take_until` 等另外 1 个符号 实现具体逻辑。

### Lines 36-45
```cpp
  }

  line = line.drop_front(word.size());
  return word;
}

// This is the StringSwitch-alike used by LexOrCompleteWord below. See that
// function for details.
template <typename T>
struct QueryParser::LexOrCompleteWord {
```
- **EN**: Introduces declarations for `QueryParser::LexOrCompleteWord`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `QueryParser::LexOrCompleteWord` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 46-53
```cpp
  llvm::StringRef word;
  llvm::StringSwitch<T> stringSwitch;

  QueryParser *queryParser;
  // Set to the completion point offset in word, or StringRef::npos if
  // completion point not in word.
  size_t wordCompletionPos;

```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 54-67
```cpp
  // Lexes a word and stores it in word. Returns a LexOrCompleteword<T> object
  // that can be used like a llvm::StringSwitch<T>, but adds cases as possible
  // completions if the lexed word contains the completion point.
  LexOrCompleteWord(QueryParser *queryParser, llvm::StringRef &outWord)
      : word(queryParser->lexWord()), stringSwitch(word),
        queryParser(queryParser), wordCompletionPos(llvm::StringRef::npos) {
    outWord = word;
    if (queryParser->completionPos &&
        queryParser->completionPos <= word.data() + word.size()) {
      if (queryParser->completionPos < word.data())
        wordCompletionPos = 0;
      else
        wordCompletionPos = queryParser->completionPos - word.data();
    }
```
- **EN**: Implements logic around `LexOrCompleteWord`, `word`, `queryParser`, `data`.
- **CN**: 围绕 `LexOrCompleteWord`、`word`、`queryParser`、`data` 实现具体逻辑。

### Lines 68-79
```cpp
  }

  LexOrCompleteWord &Case(llvm::StringLiteral caseStr, const T &value,
                          bool isCompletion = true) {

    if (wordCompletionPos == llvm::StringRef::npos)
      stringSwitch.Case(caseStr, value);
    else if (!caseStr.empty() && isCompletion &&
             wordCompletionPos <= caseStr.size() &&
             caseStr.substr(0, wordCompletionPos) ==
                 word.substr(0, wordCompletionPos)) {

```
- **EN**: Implements logic around `Case`, `empty`, `size`, `substr`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `Case`、`empty`、`size`、`substr` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 80-86
```cpp
      queryParser->completions.emplace_back(
          (caseStr.substr(wordCompletionPos) + " ").str(),
          std::string(caseStr));
    }
    return *this;
  }

```
- **EN**: Implements logic around `emplace_back`, `substr`, `string`.
- **CN**: 围绕 `emplace_back`、`substr`、`string` 实现具体逻辑。

### Lines 87-93
```cpp
  T Default(T value) { return stringSwitch.Default(value); }
};

QueryRef QueryParser::endQuery(QueryRef queryRef) {
  llvm::StringRef extra = line;
  llvm::StringRef extraTrimmed = extra.ltrim(" \t\v\f\r");

```
- **EN**: Implements logic around `Default`, `endQuery`, `ltrim`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `Default`、`endQuery`、`ltrim` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 94-107
```cpp
  if (extraTrimmed.starts_with('\n') || extraTrimmed.starts_with("\r\n"))
    queryRef->remainingContent = extra;
  else {
    llvm::StringRef trailingWord = lexWord();
    if (trailingWord.starts_with('#')) {
      line = line.drop_until([](char c) { return c == '\n'; });
      line = line.drop_while([](char c) { return c == '\n'; });
      return endQuery(queryRef);
    }
    if (!trailingWord.empty()) {
      return new InvalidQuery("unexpected extra input: '" + extra + "'");
    }
  }
  return queryRef;
```
- **EN**: Implements logic around `starts_with`, `lexWord`, `drop_until`, `drop_while`, and 3 more symbols.
- **CN**: 围绕 `starts_with`、`lexWord`、`drop_until`、`drop_while` 等另外 3 个符号 实现具体逻辑。

### Lines 108-120
```cpp
}

namespace {

enum class ParsedQueryKind {
  Invalid,
  Comment,
  NoOp,
  Help,
  Match,
  Quit,
};

```
- **EN**: Introduces declarations for `ParsedQueryKind`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `ParsedQueryKind` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 121-129
```cpp
QueryRef
makeInvalidQueryFromDiagnostics(const matcher::internal::Diagnostics &diag) {
  std::string errStr;
  llvm::raw_string_ostream os(errStr);
  diag.print(os);
  return new InvalidQuery(errStr);
}
} // namespace

```
- **EN**: Implements logic around `makeInvalidQueryFromDiagnostics`, `os`, `print`, `InvalidQuery`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `makeInvalidQueryFromDiagnostics`、`os`、`print`、`InvalidQuery` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 130-140
```cpp
QueryRef QueryParser::completeMatcherExpression() {
  std::vector<matcher::MatcherCompletion> comps =
      matcher::internal::Parser::completeExpression(
          line, completionPos - line.begin(), qs.getRegistryData(),
          &qs.namedValues);
  for (const auto &comp : comps) {
    completions.emplace_back(comp.typedText, comp.matcherDecl);
  }
  return QueryRef();
}

```
- **EN**: Implements logic around `completeMatcherExpression`, `completeExpression`, `begin`, `emplace_back`, and 1 more symbols; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `completeMatcherExpression`、`completeExpression`、`begin`、`emplace_back` 等另外 1 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 141-154
```cpp
QueryRef QueryParser::doParse() {

  llvm::StringRef commandStr;
  ParsedQueryKind qKind =
      LexOrCompleteWord<ParsedQueryKind>(this, commandStr)
          .Case("", ParsedQueryKind::NoOp)
          .Case("#", ParsedQueryKind::Comment, /*isCompletion=*/false)
          .Case("help", ParsedQueryKind::Help)
          .Case("m", ParsedQueryKind::Match, /*isCompletion=*/false)
          .Case("match", ParsedQueryKind::Match)
          .Case("q", ParsedQueryKind::Quit, /*IsCompletion=*/false)
          .Case("quit", ParsedQueryKind::Quit)
          .Default(ParsedQueryKind::Invalid);

```
- **EN**: Implements logic around `doParse`, `LexOrCompleteWord`, `Case`, `Default`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `doParse`、`LexOrCompleteWord`、`Case`、`Default` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 155-163
```cpp
  switch (qKind) {
  case ParsedQueryKind::Comment:
  case ParsedQueryKind::NoOp:
    line = line.drop_until([](char c) { return c == '\n'; });
    line = line.drop_while([](char c) { return c == '\n'; });
    if (line.empty())
      return new NoOpQuery;
    return doParse();

```
- **EN**: Implements logic around `drop_until`, `drop_while`, `empty`, `doParse`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `drop_until`、`drop_while`、`empty`、`doParse` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 164-174
```cpp
  case ParsedQueryKind::Help:
    return endQuery(new HelpQuery);

  case ParsedQueryKind::Quit:
    return endQuery(new QuitQuery);

  case ParsedQueryKind::Match: {
    if (completionPos) {
      return completeMatcherExpression();
    }

```
- **EN**: Implements logic around `endQuery`, `completeMatcherExpression`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `endQuery`、`completeMatcherExpression` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 175-188
```cpp
    matcher::internal::Diagnostics diag;
    auto matcherSource = line.ltrim();
    auto origMatcherSource = matcherSource;
    std::optional<matcher::DynMatcher> matcher =
        matcher::internal::Parser::parseMatcherExpression(
            matcherSource, qs.getRegistryData(), &qs.namedValues, &diag);
    if (!matcher) {
      return makeInvalidQueryFromDiagnostics(diag);
    }
    auto actualSource = origMatcherSource.substr(0, origMatcherSource.size() -
                                                        matcherSource.size());
    QueryRef query = new MatchQuery(actualSource, *matcher);
    query->remainingContent = matcherSource;
    return query;
```
- **EN**: Implements logic around `ltrim`, `parseMatcherExpression`, `getRegistryData`, `makeInvalidQueryFromDiagnostics`, and 3 more symbols; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `ltrim`、`parseMatcherExpression`、`getRegistryData`、`makeInvalidQueryFromDiagnostics` 等另外 3 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 189-197
```cpp
  }

  case ParsedQueryKind::Invalid:
    return new InvalidQuery("unknown command: " + commandStr);
  }

  llvm_unreachable("Invalid query kind");
}

```
- **EN**: Implements logic around `InvalidQuery`, `llvm_unreachable`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `InvalidQuery`、`llvm_unreachable` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 198-207
```cpp
QueryRef QueryParser::parse(llvm::StringRef line, const QuerySession &qs) {
  return QueryParser(line, qs).doParse();
}

std::vector<llvm::LineEditor::Completion>
QueryParser::complete(llvm::StringRef line, size_t pos,
                      const QuerySession &qs) {
  QueryParser queryParser(line, qs);
  queryParser.completionPos = line.data() + pos;

```
- **EN**: Implements logic around `parse`, `QueryParser`, `complete`, `queryParser`, and 1 more symbols; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `parse`、`QueryParser`、`complete`、`queryParser` 等另外 1 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 208-212
```cpp
  queryParser.doParse();
  return queryParser.completions;
}

} // namespace mlir::query
```
- **EN**: Implements logic around `doParse`.
- **CN**: 围绕 `doParse` 实现具体逻辑。

## Key Concepts / 关键概念

- **IR querying / IR 查询**:
  - **EN**: Supports querying or filtering IR according to structural or semantic predicates.
  - **CN**: 支持依据结构或语义谓词对 IR 进行查询或过滤。
- **SSA and region-based IR / SSA 与基于 Region 的 IR**:
  - **EN**: The file works with MLIR operations, blocks, values, or regions that form MLIR's hierarchical SSA IR.
  - **CN**: 该文件处理 MLIR 操作、块、值或 Region，它们共同构成 MLIR 的层次化 SSA IR。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `QueryParser.h`, `llvm/ADT/StringSwitch.h`
- **Subsystem categories / 子系统类别**: LLVM ADT containers and utilities / LLVM ADT 容器与工具 (1)
