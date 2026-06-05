# GlobPattern.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/GlobPattern.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements a glob pattern matcher.
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `GlobPattern` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===-- GlobPattern.cpp - Glob pattern matcher implementation -------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements a glob pattern matcher.
//
//===----------------------------------------------------------------------===//

#include "llvm/Support/GlobPattern.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/Support/Errc.h"

using namespace llvm;

// Expands character ranges and returns a bitmap.
// For example, "a-cf-hz" is expanded to "abcfghz".
static Expected<BitVector> expand(StringRef S, StringRef Original) {
  BitVector BV(256, false);

  // Expand X-Y.
  for (;;) {
    if (S.size() < 3)
      break;

    uint8_t Start = S[0];
    uint8_t End = S[2];

    // If it doesn't start with something like X-Y,
    // consume the first character and proceed.
    if (S[1] != '-') {
      BV[Start] = true;
      S = S.substr(1);
      continue;
    }

    // It must be in the form of X-Y.
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 3 direct dependencies, including `llvm/Support/GlobPattern.h`, `llvm/ADT/StringRef.h`, `llvm/Support/Errc.h`.
  CN: 引入了 3 个直接依赖，其中包括 `llvm/Support/GlobPattern.h`, `llvm/ADT/StringRef.h`, `llvm/Support/Errc.h`。
- EN: This section centers on `expand`, `BV` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `expand`, `BV` 等符号展开，负责实现局部控制流程与状态维护。

### Lines 41-80

```cpp
    // Validate it and then interpret the range.
    if (Start > End)
      return make_error<StringError>("invalid glob pattern: " + Original,
                                     errc::invalid_argument);

    for (int C = Start; C <= End; ++C)
      BV[(uint8_t)C] = true;
    S = S.substr(3);
  }

  for (char C : S)
    BV[(uint8_t)C] = true;
  return BV;
}

// Identify brace expansions in S and return the list of patterns they expand
// into.
static Expected<SmallVector<std::string, 1>>
parseBraceExpansions(StringRef S, std::optional<size_t> MaxSubPatterns) {
  SmallVector<std::string> SubPatterns = {S.str()};
  if (!MaxSubPatterns || !S.contains('{'))
    return std::move(SubPatterns);

  struct BraceExpansion {
    size_t Start;
    size_t Length;
    SmallVector<StringRef, 2> Terms;
  };
  SmallVector<BraceExpansion, 0> BraceExpansions;

  BraceExpansion *CurrentBE = nullptr;
  size_t TermBegin;
  for (size_t I = 0, E = S.size(); I != E; ++I) {
    if (S[I] == '[') {
      I = S.find(']', I + 2);
      if (I == std::string::npos)
        return make_error<StringError>("invalid glob pattern, unmatched '['",
                                       errc::invalid_argument);
    } else if (S[I] == '{') {
      if (CurrentBE)
```
- EN: This section centers on `parseBraceExpansions`, `move` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `parseBraceExpansions`, `move` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 81-120

```cpp
        return make_error<StringError>(
            "nested brace expansions are not supported",
            errc::invalid_argument);
      CurrentBE = &BraceExpansions.emplace_back();
      CurrentBE->Start = I;
      TermBegin = I + 1;
    } else if (S[I] == ',') {
      if (!CurrentBE)
        continue;
      CurrentBE->Terms.push_back(S.substr(TermBegin, I - TermBegin));
      TermBegin = I + 1;
    } else if (S[I] == '}') {
      if (!CurrentBE)
        continue;
      if (CurrentBE->Terms.empty())
        return make_error<StringError>(
            "empty or singleton brace expansions are not supported",
            errc::invalid_argument);
      CurrentBE->Terms.push_back(S.substr(TermBegin, I - TermBegin));
      CurrentBE->Length = I - CurrentBE->Start + 1;
      CurrentBE = nullptr;
    } else if (S[I] == '\\') {
      if (++I == E)
        return make_error<StringError>("invalid glob pattern, stray '\\'",
                                       errc::invalid_argument);
    }
  }
  if (CurrentBE)
    return make_error<StringError>("incomplete brace expansion",
                                   errc::invalid_argument);

  size_t NumSubPatterns = 1;
  for (auto &BE : BraceExpansions) {
    if (NumSubPatterns > std::numeric_limits<size_t>::max() / BE.Terms.size()) {
      NumSubPatterns = std::numeric_limits<size_t>::max();
      break;
    }
    NumSubPatterns *= BE.Terms.size();
  }
  if (NumSubPatterns > *MaxSubPatterns)
```
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 121-160

```cpp
    return make_error<StringError>("too many brace expansions",
                                   errc::invalid_argument);
  // Replace brace expansions in reverse order so that we don't invalidate
  // earlier start indices
  for (auto &BE : reverse(BraceExpansions)) {
    SmallVector<std::string> OrigSubPatterns;
    std::swap(SubPatterns, OrigSubPatterns);
    for (StringRef Term : BE.Terms)
      for (StringRef Orig : OrigSubPatterns)
        SubPatterns.emplace_back(Orig).replace(BE.Start, BE.Length, Term);
  }
  return std::move(SubPatterns);
}

static StringRef maxPlainSubstring(StringRef S) {
  StringRef Best;
  while (!S.empty()) {
    size_t PrefixSize = S.find_first_of("?*[{\\");
    if (PrefixSize == std::string::npos)
      PrefixSize = S.size();

    if (Best.size() < PrefixSize)
      Best = S.take_front(PrefixSize);

    S = S.drop_front(PrefixSize);

    // It's impossible, as the first and last characters of the input string
    // must be Glob special characters, otherwise they would be parts of
    // the prefix or the suffix.
    assert(!S.empty());

    switch (S.front()) {
    case '\\':
      S = S.drop_front(2);
      break;
    case '[': {
      // Drop '[' and the first character which can be ']'.
      S = S.drop_front(2);
      size_t EndBracket = S.find_first_of("]");
      // Should not be possible, SubGlobPattern::create should fail on invalid
```
- EN: This section centers on `swap`, `move`, `maxPlainSubstring` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `swap`, `move`, `maxPlainSubstring` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and iterates over collections, ranges, or records.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并遍历集合、区间或记录。

### Lines 161-200

```cpp
      // pattern before we get here.
      assert(EndBracket != std::string::npos);
      S = S.drop_front(EndBracket + 1);
      break;
    }
    case '{':
      // TODO: implement.
      // Fallback to whatever is best for now.
      return Best;
    default:
      S = S.drop_front(1);
    }
  }

  return Best;
}

Expected<GlobPattern>
GlobPattern::create(StringRef S, std::optional<size_t> MaxSubPatterns) {
  GlobPattern Pat;
  Pat.Pattern = S;

  // Store the prefix that does not contain any metacharacter.
  Pat.PrefixSize = S.find_first_of("?*[{\\");
  if (Pat.PrefixSize == std::string::npos) {
    Pat.PrefixSize = S.size();
    return Pat;
  }
  S = S.substr(Pat.PrefixSize);

  // Just in case we stop on unmatched opening brackets.
  size_t SuffixStart = S.find_last_of("?*[]{}\\");
  assert(SuffixStart != std::string::npos);
  if (S[SuffixStart] == '\\')
    ++SuffixStart;
  if (SuffixStart < S.size())
    ++SuffixStart;
  Pat.SuffixSize = S.size() - SuffixStart;
  S = S.substr(0, SuffixStart);

```
- EN: This section centers on `assert`, `create` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `assert`, `create` 等符号展开，负责创建并初始化辅助对象或状态。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 201-240

```cpp
  SmallVector<std::string, 1> SubPats;
  if (auto Err = parseBraceExpansions(S, MaxSubPatterns).moveInto(SubPats))
    return std::move(Err);
  for (StringRef SubPat : SubPats) {
    auto SubGlobOrErr = SubGlobPattern::create(SubPat);
    if (!SubGlobOrErr)
      return SubGlobOrErr.takeError();
    Pat.SubGlobs.push_back(*SubGlobOrErr);
  }

  return Pat;
}

Expected<GlobPattern::SubGlobPattern>
GlobPattern::SubGlobPattern::create(StringRef S) {
  SubGlobPattern Pat;

  // Parse brackets.
  Pat.Pat.assign(S.begin(), S.end());
  for (size_t I = 0, E = S.size(); I != E; ++I) {
    if (S[I] == '[') {
      // ']' is allowed as the first character of a character class. '[]' is
      // invalid. So, just skip the first character.
      ++I;
      size_t J = S.find(']', I + 1);
      if (J == StringRef::npos)
        return make_error<StringError>("invalid glob pattern, unmatched '['",
                                       errc::invalid_argument);
      StringRef Chars = S.substr(I, J - I);
      bool Invert = S[I] == '^' || S[I] == '!';
      Expected<BitVector> BV =
          Invert ? expand(Chars.substr(1), S) : expand(Chars, S);
      if (!BV)
        return BV.takeError();
      if (Invert)
        BV->flip();
      Pat.Brackets.push_back(Bracket{J + 1, std::move(*BV)});
      I = J;
    } else if (S[I] == '\\') {
      if (++I == E)
```
- EN: This section centers on `create` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `create` 等符号展开，负责创建并初始化辅助对象或状态。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 241-280

```cpp
        return make_error<StringError>("invalid glob pattern, stray '\\'",
                                       errc::invalid_argument);
    }
  }
  return Pat;
}

StringRef GlobPattern::longest_substr() const {
  return maxPlainSubstring(
      Pattern.drop_front(PrefixSize).drop_back(SuffixSize));
}

bool GlobPattern::match(StringRef S) const {
  if (!S.consume_front(prefix()))
    return false;
  if (!S.consume_back(suffix()))
    return false;
  if (SubGlobs.empty() && S.empty())
    return true;
  for (auto &Glob : SubGlobs)
    if (Glob.match(S))
      return true;
  return false;
}

// Factor the pattern into segments split by '*'. The segment is matched
// sequentianlly by finding the first occurrence past the end of the previous
// match.
bool GlobPattern::SubGlobPattern::match(StringRef Str) const {
  const char *P = Pat.data(), *SegmentBegin = nullptr, *S = Str.data(),
             *SavedS = S;
  const char *const PEnd = P + Pat.size(), *const End = S + Str.size();
  size_t B = 0, SavedB = 0;
  while (S != End) {
    if (P == PEnd)
      ;
    else if (*P == '*') {
      // The non-* substring on the left of '*' matches the tail of S. Save the
      // positions to be used by backtracking if we see a mismatch later.
      SegmentBegin = ++P;
```
- EN: This section centers on `longest_substr`, `maxPlainSubstring`, `match` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `longest_substr`, `maxPlainSubstring`, `match` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 281-312

```cpp
      SavedS = S;
      SavedB = B;
      continue;
    } else if (*P == '[') {
      if (Brackets[B].Bytes[uint8_t(*S)]) {
        P = Pat.data() + Brackets[B++].NextOffset;
        ++S;
        continue;
      }
    } else if (*P == '\\') {
      if (*++P == *S) {
        ++P;
        ++S;
        continue;
      }
    } else if (*P == *S || *P == '?') {
      ++P;
      ++S;
      continue;
    }
    if (!SegmentBegin)
      return false;
    // We have seen a '*'. Backtrack to the saved positions. Shift the S
    // position to probe the next starting position in the segment.
    P = SegmentBegin;
    S = ++SavedS;
    B = SavedB;
  }
  // All bytes in Str have been matched. Return true if the rest part of Pat is
  // empty or contains only '*'.
  return getPat().find_first_not_of('*', P - Pat.data()) == std::string::npos;
}
```
- EN: In this range, the code checks conditions and handles edge cases and returns the resulting value to its callers.
  CN: 在这一段中，代码检查条件并处理边界情况，并将结果返回给调用方。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `BraceExpansion`, `expand`, `BV`, `parseBraceExpansions`, `move` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Parsing, decoding, and structural validation. / 重点关注解析、解码与结构校验。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/Support/GlobPattern.h`, `llvm/ADT/StringRef.h`, `llvm/Support/Errc.h`
- Standard library / 标准库: None / 无
- Other/system headers / 其他或系统头文件: None / 无
- Related symbols / 相关符号: `BraceExpansion`, `expand`, `BV`, `parseBraceExpansions`, `move`, `swap`
