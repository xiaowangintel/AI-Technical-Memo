# FormatVariadic.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/FormatVariadic.cpp`
- Repository: `llvm-project`
- Purpose (EN): A maximum of 2 characters at the beginning can be used for something other than the width.
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `FormatVariadic` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===- FormatVariadic.cpp - Format string parsing and analysis ----*-C++-*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//===----------------------------------------------------------------------===//

#include "llvm/Support/FormatVariadic.h"
#include <cassert>
#include <optional>

using namespace llvm;

static std::optional<AlignStyle> translateLocChar(char C) {
  switch (C) {
  case '-':
    return AlignStyle::Left;
  case '=':
    return AlignStyle::Center;
  case '+':
    return AlignStyle::Right;
  default:
    return std::nullopt;
  }
  LLVM_BUILTIN_UNREACHABLE;
}

static bool consumeFieldLayout(StringRef &Spec, AlignStyle &Where,
                               unsigned &Align, char &Pad) {
  Where = AlignStyle::Right;
  Align = 0;
  Pad = ' ';
  if (Spec.empty())
    return true;

  if (Spec.size() > 1) {
    // A maximum of 2 characters at the beginning can be used for something
    // other than the width.
    // If Spec[1] is a loc char, then Spec[0] is a pad char and Spec[2:...]
    // contains the width.
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 3 direct dependencies, including `llvm/Support/FormatVariadic.h`, `cassert`, `optional`.
  CN: 引入了 3 个直接依赖，其中包括 `llvm/Support/FormatVariadic.h`, `cassert`, `optional`。
- EN: This section centers on `translateLocChar`, `consumeFieldLayout` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `translateLocChar`, `consumeFieldLayout` 等符号展开，负责实现局部控制流程与状态维护。

### Lines 41-80

```cpp
    // Otherwise, if Spec[0] is a loc char, then Spec[1:...] contains the width.
    // Otherwise, Spec[0:...] contains the width.
    if (auto Loc = translateLocChar(Spec[1])) {
      Pad = Spec[0];
      Where = *Loc;
      Spec = Spec.drop_front(2);
    } else if (auto Loc = translateLocChar(Spec[0])) {
      Where = *Loc;
      Spec = Spec.drop_front(1);
    }
  }

  bool Failed = Spec.consumeInteger(0, Align);
  return !Failed;
}

static std::optional<ReplacementItem> parseReplacementItem(StringRef Spec) {
  StringRef RepString = Spec.trim("{}");

  // If the replacement sequence does not start with a non-negative integer,
  // this is an error.
  char Pad = ' ';
  unsigned Align = 0;
  AlignStyle Where = AlignStyle::Right;
  StringRef Options;
  unsigned Index = ~0U;
  RepString = RepString.ltrim();

  // If index is not specified, keep it ~0U to indicate unresolved index.
  RepString.consumeInteger(0, Index);

  if (RepString.consume_front(",")) {
    if (!consumeFieldLayout(RepString, Where, Align, Pad)) {
      assert(false && "Invalid replacement field layout specification!");
      return std::nullopt;
    }
  }
  RepString = RepString.ltrim();
  if (RepString.consume_front(":")) {
    Options = RepString;
```
- EN: This section centers on `parseReplacementItem`, `assert` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `parseReplacementItem`, `assert` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 81-120

```cpp
    RepString = StringRef();
  }
  RepString = RepString.trim();
  if (!RepString.empty()) {
    assert(0 && "Unexpected characters found in replacement string!");
    return std::nullopt;
  }

  return ReplacementItem(Spec, Index, Align, Where, Pad, Options);
}

static std::pair<std::optional<ReplacementItem>, StringRef>
splitLiteralAndReplacement(StringRef Fmt) {
  assert(!Fmt.empty());
  // Everything up until the first brace is a literal.
  if (Fmt.front() != '{') {
    size_t BO = Fmt.find_first_of('{');
    return {ReplacementItem{Fmt.substr(0, BO)}, Fmt.substr(BO)};
  }

  StringRef Braces = Fmt.take_while([](char C) { return C == '{'; });
  // If there is more than one brace, then some of them are escaped.  Treat
  // these as replacements.
  if (Braces.size() > 1) {
    size_t NumEscapedBraces = Braces.size() / 2;
    StringRef Middle = Fmt.take_front(NumEscapedBraces);
    StringRef Right = Fmt.drop_front(NumEscapedBraces * 2);
    return {ReplacementItem(Middle), Right};
  }
  // An unterminated open brace is undefined. Assert to indicate that this is
  // undefined and that we consider it an error. When asserts are disabled,
  // build a replacement item with an error message.
  size_t BC = Fmt.find_first_of('}');
  if (BC == StringRef::npos) {
    assert(false &&
           "Unterminated brace sequence. Escape with {{ for a literal brace.");
    return {ReplacementItem("Unterminated brace sequence. Escape with {{ for a "
                            "literal brace."),
            StringRef()};
  }
```
- EN: This section centers on `assert`, `ReplacementItem`, `splitLiteralAndReplacement` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `assert`, `ReplacementItem`, `splitLiteralAndReplacement` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 121-160

```cpp

  // Even if there is a closing brace, if there is another open brace before
  // this closing brace, treat this portion as literal, and try again with the
  // next one.
  size_t BO2 = Fmt.find_first_of('{', 1);
  if (BO2 < BC)
    return {ReplacementItem(Fmt.substr(0, BO2)), Fmt.substr(BO2)};

  StringRef Spec = Fmt.slice(1, BC);
  StringRef Right = Fmt.substr(BC + 1);

  return {parseReplacementItem(Spec), Right};
}

#ifndef NDEBUG
#define ENABLE_VALIDATION 1
#else
#define ENABLE_VALIDATION 0 // Conveniently enable validation in release mode.
#endif

SmallVector<ReplacementItem, 2>
formatv_object_base::parseFormatString(StringRef Fmt, size_t NumArgs,
                                       bool Validate) {
  SmallVector<ReplacementItem, 2> Replacements;
  unsigned NextAutomaticIndex = 0;

#if ENABLE_VALIDATION
  const StringRef SavedFmtStr = Fmt;
  unsigned NumExpectedArgs = 0;
  bool HasExplicitIndex = false;
#endif

  while (!Fmt.empty()) {
    std::optional<ReplacementItem> I;
    std::tie(I, Fmt) = splitLiteralAndReplacement(Fmt);
    if (!I)
      continue;
    if (I->Type == ReplacementType::Format) {
      if (I->Index == ~0U)
        I->Index = NextAutomaticIndex++;
```
- EN: This section centers on `parseFormatString`, `tie` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `parseFormatString`, `tie` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 161-200

```cpp
#if ENABLE_VALIDATION
      else
        HasExplicitIndex = true;
      NumExpectedArgs = std::max(NumExpectedArgs, I->Index + 1);
#endif
    }

    Replacements.emplace_back(*I);
  }

#if ENABLE_VALIDATION
  if (!Validate)
    return Replacements;

  // Perform additional validation. Verify that the number of arguments matches
  // the number of replacement indices and that there are no holes in the
  // replacement indices.

  // When validation fails, return an array of replacement items that
  // will print an error message as the outout of this formatv() (used when
  // validation is enabled in release mode).
  auto getErrorReplacements = [SavedFmtStr](StringLiteral ErrorMsg) {
    return SmallVector<ReplacementItem, 2>{
        ReplacementItem("Invalid formatv() call: "), ReplacementItem(ErrorMsg),
        ReplacementItem(" for format string: "), ReplacementItem(SavedFmtStr)};
  };

  if (NumExpectedArgs != NumArgs) {
    errs() << formatv("Expected {} Args, but got {} for format string '{}'\n",
                      NumExpectedArgs, NumArgs, SavedFmtStr);
    assert(0 && "Invalid formatv() call");
    return getErrorReplacements("Unexpected number of arguments");
  }

  // Find the number of unique indices seen. All replacement indices
  // are < NumExpectedArgs.
  SmallVector<bool> Indices(NumExpectedArgs);
  unsigned Count = 0;
  for (const ReplacementItem &I : Replacements) {
    if (I.Type != ReplacementType::Format || Indices[I.Index])
```
- EN: This section centers on `assert`, `getErrorReplacements`, `Indices` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `assert`, `getErrorReplacements`, `Indices` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 201-226

```cpp
      continue;
    Indices[I.Index] = true;
    ++Count;
  }

  if (Count != NumExpectedArgs) {
    errs() << formatv(
        "Replacement field indices cannot have holes for format string '{}'\n",
        SavedFmtStr);
    assert(0 && "Invalid format string");
    return getErrorReplacements("Replacement indices have holes");
  }

  // Fail validation if we see both automatic index and explicit index.
  if (NextAutomaticIndex != 0 && HasExplicitIndex) {
    errs() << formatv(
        "Cannot mix automatic and explicit indices for format string '{}'\n",
        SavedFmtStr);
    assert(0 && "Invalid format string");
    return getErrorReplacements("Cannot mix automatic and explicit indices");
  }
#endif // ENABLE_VALIDATION
  return Replacements;
}

void support::detail::format_adapter::anchor() {}
```
- EN: This section centers on `assert`, `getErrorReplacements`, `anchor` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `assert`, `getErrorReplacements`, `anchor` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `translateLocChar`, `consumeFieldLayout`, `parseReplacementItem`, `assert` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Formatting, emission, and outward serialization. / 重点关注格式化、输出与序列化。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/Support/FormatVariadic.h`
- Standard library / 标准库: `optional`
- Other/system headers / 其他或系统头文件: `cassert`
- Related symbols / 相关符号: `translateLocChar`, `consumeFieldLayout`, `parseReplacementItem`, `assert`, `ReplacementItem`
