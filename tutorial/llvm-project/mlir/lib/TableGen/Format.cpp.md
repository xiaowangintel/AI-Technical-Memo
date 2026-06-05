# Format.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/TableGen/Format.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file defines utilities for formatting strings. They are specially tailored to the needs of TableGen'ing op definitions and rewrite rules, so they are not expected to be used as widely applicable utilities.
  - **CN**: 实现生成 MLIR 声明、定义或辅助代码的 TableGen 后端。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- Format.cpp - Utilities for String Format ---------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及高层摘要。

### Lines 8-14
```cpp
//
// This file defines utilities for formatting strings. They are specially
// tailored to the needs of TableGen'ing op definitions and rewrite rules,
// so they are not expected to be used as widely applicable utilities.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 15-22
```cpp
#include "mlir/TableGen/Format.h"
#include "llvm/ADT/StringSwitch.h"
#include "llvm/ADT/Twine.h"
#include <cctype>

using namespace mlir;
using namespace mlir::tblgen;

```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/TableGen/Format.h`, `llvm/ADT/StringSwitch.h`, `llvm/ADT/Twine.h`, `cctype`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/TableGen/Format.h`, `llvm/ADT/StringSwitch.h`, `llvm/ADT/Twine.h`, `cctype`。

### Lines 23-30
```cpp
// Marker to indicate an error happened when replacing a placeholder.
const char *const kMarkerForNoSubst = "<no-subst-found>";

FmtContext::FmtContext(ArrayRef<std::pair<StringRef, StringRef>> subs) {
  for (auto &sub : subs)
    addSubst(sub.first, sub.second);
}

```
- **EN**: Implements logic around `FmtContext`, `addSubst`; this block processes TableGen records or generates derived code.
- **CN**: 围绕 `FmtContext`、`addSubst` 实现具体逻辑；该代码块处理 TableGen 记录或生成派生代码。

### Lines 31-40
```cpp
FmtContext &FmtContext::addSubst(StringRef placeholder, const Twine &subst) {
  customSubstMap[placeholder] = subst.str();
  return *this;
}

FmtContext &FmtContext::withBuilder(Twine subst) {
  builtinSubstMap[PHKind::Builder] = subst.str();
  return *this;
}

```
- **EN**: Implements logic around `addSubst`, `str`, `withBuilder`; this block processes TableGen records or generates derived code.
- **CN**: 围绕 `addSubst`、`str`、`withBuilder` 实现具体逻辑；该代码块处理 TableGen 记录或生成派生代码。

### Lines 41-54
```cpp
FmtContext &FmtContext::withSelf(Twine subst) {
  builtinSubstMap[PHKind::Self] = subst.str();
  return *this;
}

std::optional<StringRef>
FmtContext::getSubstFor(FmtContext::PHKind placeholder) const {
  if (placeholder == FmtContext::PHKind::None ||
      placeholder == FmtContext::PHKind::Custom)
    return {};
  auto it = builtinSubstMap.find(placeholder);
  if (it == builtinSubstMap.end())
    return {};
  return StringRef(it->second);
```
- **EN**: Implements logic around `withSelf`, `str`, `getSubstFor`, `find`, and 2 more symbols; this block processes TableGen records or generates derived code.
- **CN**: 围绕 `withSelf`、`str`、`getSubstFor`、`find` 等另外 2 个符号 实现具体逻辑；该代码块处理 TableGen 记录或生成派生代码。

### Lines 55-63
```cpp
}

std::optional<StringRef> FmtContext::getSubstFor(StringRef placeholder) const {
  auto it = customSubstMap.find(placeholder);
  if (it == customSubstMap.end())
    return {};
  return StringRef(it->second);
}

```
- **EN**: Implements logic around `getSubstFor`, `find`, `end`, `StringRef`; this block processes TableGen records or generates derived code.
- **CN**: 围绕 `getSubstFor`、`find`、`end`、`StringRef` 实现具体逻辑；该代码块处理 TableGen 记录或生成派生代码。

### Lines 64-71
```cpp
FmtContext::PHKind FmtContext::getPlaceHolderKind(StringRef str) {
  return StringSwitch<FmtContext::PHKind>(str)
      .Case("_builder", FmtContext::PHKind::Builder)
      .Case("_self", FmtContext::PHKind::Self)
      .Case("", FmtContext::PHKind::None)
      .Default(FmtContext::PHKind::Custom);
}

```
- **EN**: Implements logic around `getPlaceHolderKind`, `PHKind>`, `Case`, `Default`; this block processes TableGen records or generates derived code.
- **CN**: 围绕 `getPlaceHolderKind`、`PHKind>`、`Case`、`Default` 实现具体逻辑；该代码块处理 TableGen 记录或生成派生代码。

### Lines 72-85
```cpp
std::pair<FmtReplacement, StringRef>
FmtObjectBase::splitFmtSegment(StringRef fmt) {
  size_t begin = fmt.find_first_of('$');
  if (begin == StringRef::npos) {
    // No placeholders: the whole format string should be returned as a
    // literal string.
    return {FmtReplacement{fmt}, StringRef()};
  }
  if (begin != 0) {
    // The first placeholder is not at the beginning: we can split the format
    // string into a literal string and the rest.
    return {FmtReplacement{fmt.substr(0, begin)}, fmt.substr(begin)};
  }

```
- **EN**: Implements logic around `splitFmtSegment`, `find_first_of`, `StringRef`, `substr`; this block processes TableGen records or generates derived code.
- **CN**: 围绕 `splitFmtSegment`、`find_first_of`、`StringRef`、`substr` 实现具体逻辑；该代码块处理 TableGen 记录或生成派生代码。

### Lines 86-92
```cpp
  // The first placeholder is at the beginning

  if (fmt.size() == 1) {
    // The whole format string just contains '$': treat as literal.
    return {FmtReplacement{fmt}, StringRef()};
  }

```
- **EN**: Implements logic around `size`, `StringRef`; this block processes TableGen records or generates derived code.
- **CN**: 围绕 `size`、`StringRef` 实现具体逻辑；该代码块处理 TableGen 记录或生成派生代码。

### Lines 93-100
```cpp
  // Allow escaping dollar with '$$'
  if (fmt[1] == '$') {
    return {FmtReplacement{fmt.substr(0, 1)}, fmt.substr(2)};
  }

  // First try to see if it's a positional placeholder, and then handle special
  // placeholders.

```
- **EN**: Implements logic around `substr`; this block processes TableGen records or generates derived code.
- **CN**: 围绕 `substr` 实现具体逻辑；该代码块处理 TableGen 记录或生成派生代码。

### Lines 101-109
```cpp
  size_t end =
      fmt.find_if_not([](char c) { return std::isdigit(c); }, /*From=*/1);
  if (end != 1) {
    // We have a positional placeholder. Parse the index.
    size_t index = 0;
    if (fmt.substr(1, end - 1).consumeInteger(0, index)) {
      llvm_unreachable("invalid replacement sequence index");
    }

```
- **EN**: Implements logic around `find_if_not`, `substr`, `llvm_unreachable`; this block parses or prints textual MLIR representations; processes TableGen records or generates derived code.
- **CN**: 围绕 `find_if_not`、`substr`、`llvm_unreachable` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示，并处理 TableGen 记录或生成派生代码。

### Lines 110-117
```cpp
    // Check if this is the part of a range specification.
    if (fmt.substr(end, 3) == "...") {
      // Currently only ranges without upper bound are supported.
      return {
          FmtReplacement{fmt.substr(0, end + 3), index, FmtReplacement::kUnset},
          fmt.substr(end + 3)};
    }

```
- **EN**: Implements logic around `substr`; this block processes TableGen records or generates derived code.
- **CN**: 围绕 `substr` 实现具体逻辑；该代码块处理 TableGen 记录或生成派生代码。

### Lines 118-124
```cpp
    if (end == StringRef::npos) {
      // All the remaining characters are part of the positional placeholder.
      return {FmtReplacement{fmt, index}, StringRef()};
    }
    return {FmtReplacement{fmt.substr(0, end), index}, fmt.substr(end)};
  }

```
- **EN**: Implements logic around `StringRef`, `substr`; this block processes TableGen records or generates derived code.
- **CN**: 围绕 `StringRef`、`substr` 实现具体逻辑；该代码块处理 TableGen 记录或生成派生代码。

### Lines 125-133
```cpp
  end = fmt.find_if_not([](char c) { return std::isalnum(c) || c == '_'; }, 1);
  auto placeholder = FmtContext::getPlaceHolderKind(fmt.substr(1, end - 1));
  if (end == StringRef::npos) {
    // All the remaining characters are part of the special placeholder.
    return {FmtReplacement{fmt, placeholder}, StringRef()};
  }
  return {FmtReplacement{fmt.substr(0, end), placeholder}, fmt.substr(end)};
}

```
- **EN**: Implements logic around `find_if_not`, `getPlaceHolderKind`, `StringRef`, `substr`; this block processes TableGen records or generates derived code.
- **CN**: 围绕 `find_if_not`、`getPlaceHolderKind`、`StringRef`、`substr` 实现具体逻辑；该代码块处理 TableGen 记录或生成派生代码。

### Lines 134-144
```cpp
std::vector<FmtReplacement> FmtObjectBase::parseFormatString(StringRef fmt) {
  std::vector<FmtReplacement> replacements;
  FmtReplacement repl;
  while (!fmt.empty()) {
    std::tie(repl, fmt) = splitFmtSegment(fmt);
    if (repl.type != FmtReplacement::Type::Empty)
      replacements.push_back(repl);
  }
  return replacements;
}

```
- **EN**: Implements logic around `parseFormatString`, `empty`, `tie`, `push_back`; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations; processes TableGen records or generates derived code.
- **CN**: 围绕 `parseFormatString`、`empty`、`tie`、`push_back` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示，并处理 TableGen 记录或生成派生代码。

### Lines 145-154
```cpp
void FmtObjectBase::format(raw_ostream &s) const {
  for (auto &repl : replacements) {
    if (repl.type == FmtReplacement::Type::Empty)
      continue;

    if (repl.type == FmtReplacement::Type::Literal) {
      s << repl.spec;
      continue;
    }

```
- **EN**: Implements logic around `format`; this block manipulates core MLIR IR objects; processes TableGen records or generates derived code.
- **CN**: 围绕 `format` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并处理 TableGen 记录或生成派生代码。

### Lines 155-168
```cpp
    if (repl.type == FmtReplacement::Type::SpecialPH) {
      if (repl.placeholder == FmtContext::PHKind::None) {
        s << repl.spec;
      } else if (!context) {
        // We need the context to replace special placeholders.
        s << repl.spec << kMarkerForNoSubst;
      } else {
        std::optional<StringRef> subst;
        if (repl.placeholder == FmtContext::PHKind::Custom) {
          // Skip the leading '$' sign for the custom placeholder
          subst = context->getSubstFor(repl.spec.substr(1));
        } else {
          subst = context->getSubstFor(repl.placeholder);
        }
```
- **EN**: Implements logic around `getSubstFor`; this block manipulates core MLIR IR objects; processes TableGen records or generates derived code.
- **CN**: 围绕 `getSubstFor` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并处理 TableGen 记录或生成派生代码。

### Lines 169-176
```cpp
        if (subst)
          s << *subst;
        else
          s << repl.spec << kMarkerForNoSubst;
      }
      continue;
    }

```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 177-190
```cpp
    if (repl.type == FmtReplacement::Type::PositionalRangePH) {
      if (repl.index >= adapters.size()) {
        s << repl.spec << kMarkerForNoSubst;
        continue;
      }
      auto range = llvm::ArrayRef(adapters);
      range = range.drop_front(repl.index);
      if (repl.end != FmtReplacement::kUnset)
        range = range.drop_back(adapters.size() - repl.end);
      llvm::interleaveComma(range, s,
                            [&](auto &x) { x->format(s, /*Options=*/""); });
      continue;
    }

```
- **EN**: Implements logic around `size`, `ArrayRef`, `drop_front`, `drop_back`, and 2 more symbols; this block manipulates core MLIR IR objects; processes TableGen records or generates derived code.
- **CN**: 围绕 `size`、`ArrayRef`、`drop_front`、`drop_back` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并处理 TableGen 记录或生成派生代码。

### Lines 191-200
```cpp
    assert(repl.type == FmtReplacement::Type::PositionalPH);

    if (repl.index >= adapters.size()) {
      s << repl.spec << kMarkerForNoSubst;
      continue;
    }
    adapters[repl.index]->format(s, /*Options=*/"");
  }
}

```
- **EN**: Implements logic around `assert`, `size`, `format`; this block manipulates core MLIR IR objects; processes TableGen records or generates derived code.
- **CN**: 围绕 `assert`、`size`、`format` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并处理 TableGen 记录或生成派生代码。

### Lines 201-208
```cpp
FmtStrVecObject::FmtStrVecObject(StringRef fmt, const FmtContext *ctx,
                                 ArrayRef<std::string> params)
    : FmtObjectBase(fmt, ctx, params.size()) {
  parameters.reserve(params.size());
  for (std::string p : params)
    parameters.push_back(
        llvm::support::detail::build_format_adapter(std::move(p)));

```
- **EN**: Implements logic around `FmtStrVecObject`, `FmtObjectBase`, `reserve`, `push_back`, and 1 more symbols; this block processes TableGen records or generates derived code.
- **CN**: 围绕 `FmtStrVecObject`、`FmtObjectBase`、`reserve`、`push_back` 等另外 1 个符号 实现具体逻辑；该代码块处理 TableGen 记录或生成派生代码。

### Lines 209-219
```cpp
  adapters.reserve(parameters.size());
  for (auto &p : parameters)
    adapters.push_back(&p);
}

FmtStrVecObject::FmtStrVecObject(FmtStrVecObject &&that)
    : FmtObjectBase(std::move(that)), parameters(std::move(that.parameters)) {
  adapters.reserve(parameters.size());
  for (auto &p : parameters)
    adapters.push_back(&p);
}
```
- **EN**: Implements logic around `reserve`, `push_back`, `FmtStrVecObject`, `FmtObjectBase`; this block processes TableGen records or generates derived code.
- **CN**: 围绕 `reserve`、`push_back`、`FmtStrVecObject`、`FmtObjectBase` 实现具体逻辑；该代码块处理 TableGen 记录或生成派生代码。

## Key Concepts / 关键概念

- **Code generation via TableGen / 借助 TableGen 生成代码**:
  - **EN**: Transforms declarative descriptions into generated MLIR declarations and definitions.
  - **CN**: 把声明式描述转换为生成的 MLIR 声明与定义。
- **SSA and region-based IR / SSA 与基于 Region 的 IR**:
  - **EN**: The file works with MLIR operations, blocks, values, or regions that form MLIR's hierarchical SSA IR.
  - **CN**: 该文件处理 MLIR 操作、块、值或 Region，它们共同构成 MLIR 的层次化 SSA IR。
- **TableGen-driven generation / TableGen 驱动生成**:
  - **EN**: Declarative records are converted into generated MLIR declarations or implementation fragments.
  - **CN**: 把声明式记录转换为生成的 MLIR 声明或实现片段。
- **Streaming output / 流式输出**:
  - **EN**: Text, diagnostics, or generated content are emitted through LLVM stream APIs.
  - **CN**: 通过 LLVM 流 API 输出文本、诊断或生成内容。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `mlir/TableGen/Format.h`, `llvm/ADT/StringSwitch.h`, `llvm/ADT/Twine.h`
- **Standard-library headers / 标准库头文件**: `<cctype>`
- **Subsystem categories / 子系统类别**: LLVM ADT containers and utilities / LLVM ADT 容器与工具 (2), TableGen backend support / TableGen 后端支持 (1)
