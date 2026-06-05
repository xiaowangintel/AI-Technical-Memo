# Regex.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/Regex.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements a POSIX regular expression matcher.
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `Regex` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===-- Regex.cpp - Regular Expression matcher implementation -------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements a POSIX regular expression matcher.
//
//===----------------------------------------------------------------------===//

#include "llvm/Support/Regex.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/ADT/Twine.h"
#include "regex_impl.h"

#include <cassert>
#include <string>

using namespace llvm;

Regex::Regex() : preg(nullptr), error(REG_BADPAT) {}

Regex::Regex(StringRef regex, RegexFlags Flags) {
  unsigned flags = 0;
  preg = new llvm_regex();
  preg->re_endp = regex.end();
  if (Flags & IgnoreCase)
    flags |= REG_ICASE;
  if (Flags & Newline)
    flags |= REG_NEWLINE;
  if (!(Flags & BasicRegex))
    flags |= REG_EXTENDED;
  error = llvm_regcomp(preg, regex.data(), flags|REG_PEND);
}

Regex::Regex(StringRef regex, unsigned Flags)
    : Regex(regex, static_cast<RegexFlags>(Flags)) {}
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 7 direct dependencies, including `llvm/Support/Regex.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/Twine.h`.
  CN: 引入了 7 个直接依赖，其中包括 `llvm/Support/Regex.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/Twine.h`。
- EN: This section centers on `Regex` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `Regex` 等符号展开，负责实现局部控制流程与状态维护。

### Lines 41-80

```cpp

Regex::Regex(Regex &&regex) {
  preg = regex.preg;
  error = regex.error;
  regex.preg = nullptr;
  regex.error = REG_BADPAT;
}

Regex::~Regex() {
  if (preg) {
    llvm_regfree(preg);
    delete preg;
  }
}

namespace {

/// Utility to convert a regex error code into a human-readable string.
void RegexErrorToString(int error, struct llvm_regex *preg,
                        std::string &Error) {
  size_t len = llvm_regerror(error, preg, nullptr, 0);

  Error.resize(len - 1);
  llvm_regerror(error, preg, &Error[0], len);
}

} // namespace

bool Regex::isValid(std::string &Error) const {
  if (!error)
    return true;

  RegexErrorToString(error, preg, Error);
  return false;
}

/// getNumMatches - In a valid regex, return the number of parenthesized
/// matches it contains.
unsigned Regex::getNumMatches() const {
  return preg->re_nsub;
```
- EN: This section centers on `Regex`, `llvm_regfree`, `RegexErrorToString` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `Regex`, `llvm_regfree`, `RegexErrorToString` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 81-120

```cpp
}

bool Regex::match(StringRef String, SmallVectorImpl<StringRef> *Matches,
                  std::string *Error) const {
  // Reset error, if given.
  if (Error && !Error->empty())
    *Error = "";

  // Check if the regex itself didn't successfully compile.
  if (Error ? !isValid(*Error) : !isValid())
    return false;

  unsigned nmatch = Matches ? preg->re_nsub+1 : 0;

  // Update null string to empty string.
  if (String.data() == nullptr)
    String = "";

  // pmatch needs to have at least one element.
  SmallVector<llvm_regmatch_t, 8> pm;
  pm.resize(nmatch > 0 ? nmatch : 1);
  pm[0].rm_so = 0;
  pm[0].rm_eo = String.size();

  int rc = llvm_regexec(preg, String.data(), nmatch, pm.data(), REG_STARTEND);

  // Failure to match is not an error, it's just a normal return value.
  // Any other error code is considered abnormal, and is logged in the Error.
  if (rc == REG_NOMATCH)
    return false;
  if (rc != 0) {
    if (Error)
      RegexErrorToString(error, preg, *Error);
    return false;
  }

  // There was a match.

  if (Matches) { // match position requested
    Matches->clear();
```
- EN: This section centers on `match` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `match` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 121-160

```cpp

    for (unsigned i = 0; i != nmatch; ++i) {
      if (pm[i].rm_so == -1) {
        // this group didn't match
        Matches->push_back(StringRef());
        continue;
      }
      assert(pm[i].rm_eo >= pm[i].rm_so);
      Matches->push_back(StringRef(String.data()+pm[i].rm_so,
                                   pm[i].rm_eo-pm[i].rm_so));
    }
  }

  return true;
}

std::string Regex::sub(StringRef Repl, StringRef String,
                       std::string *Error) const {
  SmallVector<StringRef, 8> Matches;

  // Return the input if there was no match.
  if (!match(String, &Matches, Error))
    return std::string(String);

  // Otherwise splice in the replacement string, starting with the prefix before
  // the match.
  std::string Res(String.begin(), Matches[0].begin());

  // Then the replacement string, honoring possible substitutions.
  while (!Repl.empty()) {
    // Skip to the next escape.
    std::pair<StringRef, StringRef> Split = Repl.split('\\');

    // Add the skipped substring.
    Res += Split.first;

    // Check for termination and trailing backslash.
    if (Split.second.empty()) {
      if (Repl.size() != Split.first.size() &&
          Error && Error->empty())
```
- EN: This section centers on `assert`, `sub`, `Res` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `assert`, `sub`, `Res` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 161-200

```cpp
        *Error = "replacement string contained trailing backslash";
      break;
    }

    // Otherwise update the replacement string and interpret escapes.
    Repl = Split.second;

    // FIXME: We should have a StringExtras function for mapping C99 escapes.
    switch (Repl[0]) {

      // Backreference with the "\g<ref>" syntax
    case 'g':
      if (Repl.size() >= 4 && Repl[1] == '<') {
        size_t End = Repl.find('>');
        StringRef Ref = Repl.slice(2, End);
        unsigned RefValue;
        if (End != StringRef::npos && !Ref.getAsInteger(10, RefValue)) {
          Repl = Repl.substr(End + 1);
          if (RefValue < Matches.size())
            Res += Matches[RefValue];
          else if (Error && Error->empty())
            *Error =
                ("invalid backreference string 'g<" + Twine(Ref) + ">'").str();
          break;
        }
      }
      [[fallthrough]];

      // Treat all unrecognized characters as self-quoting.
    default:
      Res += Repl[0];
      Repl = Repl.substr(1);
      break;

      // Single character escapes.
    case 't':
      Res += '\t';
      Repl = Repl.substr(1);
      break;
    case 'n':
```
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and checks conditions and handles edge cases.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并检查条件并处理边界情况。

### Lines 201-240

```cpp
      Res += '\n';
      Repl = Repl.substr(1);
      break;

      // Decimal escapes are backreferences.
    case '0': case '1': case '2': case '3': case '4':
    case '5': case '6': case '7': case '8': case '9': {
      // Extract the backreference number.
      StringRef Ref = Repl.slice(0, Repl.find_first_not_of("0123456789"));
      Repl = Repl.substr(Ref.size());

      unsigned RefValue;
      if (!Ref.getAsInteger(10, RefValue) &&
          RefValue < Matches.size())
        Res += Matches[RefValue];
      else if (Error && Error->empty())
        *Error = ("invalid backreference string '" + Twine(Ref) + "'").str();
      break;
    }
    }
  }

  // And finally the suffix.
  Res += StringRef(Matches[0].end(), String.end() - Matches[0].end());

  return Res;
}

// These are the special characters matched in functions like "p_ere_exp".
static const char RegexMetachars[] = "()^$|*+?.[]\\{}";

bool Regex::isLiteralERE(StringRef Str) {
  // Check for regex metacharacters.  This list was derived from our regex
  // implementation in regcomp.c and double checked against the POSIX extended
  // regular expression specification.
  return Str.find_first_of(RegexMetachars) == StringRef::npos;
}

std::string Regex::escape(StringRef String) {
  std::string RegexStr;
```
- EN: This section centers on `isLiteralERE`, `escape` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `isLiteralERE`, `escape` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 241-248

```cpp
  for (char C : String) {
    if (strchr(RegexMetachars, C))
      RegexStr += '\\';
    RegexStr += C;
  }

  return RegexStr;
}
```
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `llvm_regex`, `Regex`, `llvm_regfree`, `RegexErrorToString`, `llvm_regerror` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Helper routines coordinate local state, control flow, and result construction. / 通过辅助例程协调局部状态、控制流与结果构造。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/Support/Regex.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/Twine.h`
- Standard library / 标准库: `string`
- Other/system headers / 其他或系统头文件: `regex_impl.h`, `cassert`
- Related symbols / 相关符号: `llvm_regex`, `Regex`, `llvm_regfree`, `RegexErrorToString`, `llvm_regerror`, `isValid`
