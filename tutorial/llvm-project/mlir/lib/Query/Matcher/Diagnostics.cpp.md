# Diagnostics.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Query/Matcher/Diagnostics.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements MLIR query facilities, matcher infrastructure, or command-line querying behavior.
  - **CN**: 实现 MLIR 查询能力、匹配器基础设施或命令行查询行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- Diagnostic.cpp -----------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及高层摘要。

### Lines 8-12
```cpp

#include "Diagnostics.h"
#include "mlir/Query/Matcher/ErrorBuilder.h"

namespace mlir::query::matcher::internal {
```
- **EN**: Pulls in the declarations needed by this translation unit, including `Diagnostics.h`, `mlir/Query/Matcher/ErrorBuilder.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `Diagnostics.h`, `mlir/Query/Matcher/ErrorBuilder.h`。

### Lines 13-19
```cpp

Diagnostics::ArgStream &
Diagnostics::ArgStream::operator<<(const llvm::Twine &arg) {
  out->push_back(arg.str());
  return *this;
}

```
- **EN**: Implements logic around `operator`, `push_back`.
- **CN**: 围绕 `operator`、`push_back` 实现具体逻辑。

### Lines 20-29
```cpp
Diagnostics::ArgStream Diagnostics::addError(SourceRange range,
                                             ErrorType error) {
  errorValues.emplace_back();
  ErrorContent &last = errorValues.back();
  last.contextStack = contextStack;
  last.messages.emplace_back();
  last.messages.back().range = range;
  last.messages.back().type = error;
  return ArgStream(&last.messages.back().args);
}
```
- **EN**: Implements logic around `addError`, `emplace_back`, `back`, `ArgStream`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `addError`、`emplace_back`、`back`、`ArgStream` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 30-39
```cpp

static llvm::StringRef errorTypeToFormatString(ErrorType type) {
  switch (type) {
  case ErrorType::RegistryMatcherNotFound:
    return "Matcher not found: $0";
  case ErrorType::RegistryWrongArgCount:
    return "Incorrect argument count. (Expected = $0) != (Actual = $1)";
  case ErrorType::RegistryWrongArgType:
    return "Incorrect type for arg $0. (Expected = $1) != (Actual = $2)";
  case ErrorType::RegistryValueNotFound:
```
- **EN**: Implements logic around `errorTypeToFormatString`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `errorTypeToFormatString` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 40-49
```cpp
    return "Value not found: $0";
  case ErrorType::RegistryNotBindable:
    return "Matcher does not support binding.";

  case ErrorType::ParserStringError:
    return "Error parsing string token: <$0>";
  case ErrorType::ParserNoOpenParen:
    return "Error parsing matcher. Found token <$0> while looking for '('.";
  case ErrorType::ParserNoCloseParen:
    return "Error parsing matcher. Found end-of-code while looking for ')'.";
```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 50-59
```cpp
  case ErrorType::ParserNoComma:
    return "Error parsing matcher. Found token <$0> while looking for ','.";
  case ErrorType::ParserNoCode:
    return "End of code found while looking for token.";
  case ErrorType::ParserNotAMatcher:
    return "Input value is not a matcher expression.";
  case ErrorType::ParserInvalidToken:
    return "Invalid token <$0> found when looking for a value.";
  case ErrorType::ParserTrailingCode:
    return "Unexpected end of code.";
```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 60-69
```cpp
  case ErrorType::ParserOverloadedType:
    return "Input value has unresolved overloaded type: $0";
  case ErrorType::ParserMalformedChainedExpr:
    return "Period not followed by valid chained call.";
  case ErrorType::ParserChainedExprInvalidArg:
    return "Missing/Invalid argument for the chained call.";
  case ErrorType::ParserChainedExprNoCloseParen:
    return "Missing ')' for the chained call.";
  case ErrorType::ParserChainedExprNoOpenParen:
    return "Missing '(' for the chained call.";
```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 70-78
```cpp
  case ErrorType::ParserFailedToBuildMatcher:
    return "Failed to build matcher: $0.";

  case ErrorType::None:
    return "<N/A>";
  }
  llvm_unreachable("Unknown ErrorType value.");
}

```
- **EN**: Implements logic around `llvm_unreachable`; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `llvm_unreachable` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 79-88
```cpp
static void formatErrorString(llvm::StringRef formatString,
                              llvm::ArrayRef<std::string> args,
                              llvm::raw_ostream &os) {
  while (!formatString.empty()) {
    std::pair<llvm::StringRef, llvm::StringRef> pieces =
        formatString.split("$");
    os << pieces.first.str();
    if (pieces.second.empty())
      break;

```
- **EN**: Implements logic around `formatErrorString`, `empty`, `split`, `str`.
- **CN**: 围绕 `formatErrorString`、`empty`、`split`、`str` 实现具体逻辑。

### Lines 89-98
```cpp
    const char next = pieces.second.front();
    formatString = pieces.second.drop_front();
    if (next >= '0' && next <= '9') {
      const unsigned index = next - '0';
      if (index < args.size()) {
        os << args[index];
      } else {
        os << "<Argument_Not_Provided>";
      }
    }
```
- **EN**: Implements logic around `front`, `drop_front`, `size`.
- **CN**: 围绕 `front`、`drop_front`、`size` 实现具体逻辑。

### Lines 99-107
```cpp
  }
}

static void maybeAddLineAndColumn(SourceRange range, llvm::raw_ostream &os) {
  if (range.start.line > 0 && range.start.column > 0) {
    os << range.start.line << ":" << range.start.column << ": ";
  }
}

```
- **EN**: Implements logic around `maybeAddLineAndColumn`.
- **CN**: 围绕 `maybeAddLineAndColumn` 实现具体逻辑。

### Lines 108-115
```cpp
void Diagnostics::printMessage(
    const Diagnostics::ErrorContent::Message &message, const llvm::Twine prefix,
    llvm::raw_ostream &os) const {
  maybeAddLineAndColumn(message.range, os);
  os << prefix;
  formatErrorString(errorTypeToFormatString(message.type), message.args, os);
}

```
- **EN**: Implements logic around `printMessage`, `maybeAddLineAndColumn`, `formatErrorString`; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `printMessage`、`maybeAddLineAndColumn`、`formatErrorString` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 116-125
```cpp
void Diagnostics::printErrorContent(const Diagnostics::ErrorContent &content,
                                    llvm::raw_ostream &os) const {
  if (content.messages.size() == 1) {
    printMessage(content.messages[0], "", os);
  } else {
    for (size_t i = 0, e = content.messages.size(); i != e; ++i) {
      if (i != 0)
        os << "\n";
      printMessage(content.messages[i],
                   "Candidate " + llvm::Twine(i + 1) + ": ", os);
```
- **EN**: Implements logic around `printErrorContent`, `size`, `printMessage`, `Twine`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `printErrorContent`、`size`、`printMessage`、`Twine` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 126-135
```cpp
    }
  }
}

void Diagnostics::print(llvm::raw_ostream &os) const {
  for (const ErrorContent &error : errorValues) {
    if (&error != &errorValues.front())
      os << "\n";
    printErrorContent(error, os);
  }
```
- **EN**: Implements logic around `print`, `front`, `printErrorContent`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `print`、`front`、`printErrorContent` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 136-138
```cpp
}

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
- **Structured diagnostics / 结构化诊断**:
  - **EN**: Failures are surfaced through `LogicalResult`, diagnostics, or related reporting helpers.
  - **CN**: 通过 `LogicalResult`、诊断对象或相关报告工具显式传播失败。
- **Streaming output / 流式输出**:
  - **EN**: Text, diagnostics, or generated content are emitted through LLVM stream APIs.
  - **CN**: 通过 LLVM 流 API 输出文本、诊断或生成内容。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `Diagnostics.h`, `mlir/Query/Matcher/ErrorBuilder.h`
- **Subsystem categories / 子系统类别**: query infrastructure / 查询基础设施 (1)
