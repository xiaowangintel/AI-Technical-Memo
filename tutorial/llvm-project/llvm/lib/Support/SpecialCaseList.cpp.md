# SpecialCaseList.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/SpecialCaseList.cpp`
- Repository: `llvm-project`
- Purpose (EN): This is a utility class for instrumentation passes (like AddressSanitizer or ThreadSanitizer) to avoid instrumenting some functions or global variables, or to instrument some functions or global variables in a specific way, based on a user-supplied list.
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `SpecialCaseList` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-60

```cpp
//===-- SpecialCaseList.cpp - special case list for sanitizers ------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This is a utility class for instrumentation passes (like AddressSanitizer
// or ThreadSanitizer) to avoid instrumenting some functions or global
// variables, or to instrument some functions or global variables in a specific
// way, based on a user-supplied list.
//
//===----------------------------------------------------------------------===//

#include "llvm/Support/SpecialCaseList.h"
#include "llvm/ADT/RadixTree.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/StringMap.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/ADT/iterator_range.h"
#include "llvm/Support/Compiler.h"
#include "llvm/Support/GlobPattern.h"
#include "llvm/Support/LineIterator.h"
#include "llvm/Support/MemoryBuffer.h"
#include "llvm/Support/Regex.h"
#include "llvm/Support/VirtualFileSystem.h"
#include "llvm/Support/raw_ostream.h"
#include <memory>
#include <stdio.h>
#include <string>
#include <system_error>
#include <utility>
#include <variant>
#include <vector>

namespace llvm {

namespace {

// Lagacy v1 matcher.
class RegexMatcher {
public:
  Error insert(StringRef Pattern, unsigned LineNumber);
  unsigned match(StringRef Query) const;

private:
  struct Reg {
    Reg(StringRef Name, unsigned LineNo, Regex &&Rg)
        : Name(Name), LineNo(LineNo), Rg(std::move(Rg)) {}
    StringRef Name;
    unsigned LineNo;
    Regex Rg;
  };

  std::vector<Reg> RegExes;
};

class GlobMatcher {
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 21 direct dependencies, including `llvm/Support/SpecialCaseList.h`, `llvm/ADT/RadixTree.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVector.h`.
  CN: 引入了 21 个直接依赖，其中包括 `llvm/Support/SpecialCaseList.h`, `llvm/ADT/RadixTree.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVector.h`。
- EN: This section centers on `insert`, `match`, `Reg` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `insert`, `match`, `Reg` 等符号展开，负责实现局部控制流程与状态维护。

### Lines 61-120

```cpp
public:
  Error insert(StringRef Pattern, unsigned LineNumber);
  unsigned match(StringRef Query) const;

private:
  struct Glob {
    Glob(StringRef Name, unsigned LineNo, GlobPattern &&Pattern)
        : Name(Name), LineNo(LineNo), Pattern(std::move(Pattern)) {}
    StringRef Name;
    unsigned LineNo;
    GlobPattern Pattern;
  };

  void LazyInit() const;

  std::vector<GlobMatcher::Glob> Globs;

  mutable RadixTree<iterator_range<StringRef::const_iterator>,
                    RadixTree<iterator_range<StringRef::const_reverse_iterator>,
                              SmallVector<int, 1>>>
      PrefixSuffixToGlob;

  mutable RadixTree<iterator_range<StringRef::const_iterator>,
                    SmallVector<int, 1>>
      SubstrToGlob;

  mutable bool Initialized = false;
};

/// Represents a set of patterns and their line numbers
class Matcher {
public:
  Matcher(bool UseGlobs, bool RemoveDotSlash);

  Error insert(StringRef Pattern, unsigned LineNumber);
  unsigned match(StringRef Query) const;

  bool matchAny(StringRef Query) const { return match(Query); }

  std::variant<RegexMatcher, GlobMatcher> M;
  bool RemoveDotSlash;
};

Error RegexMatcher::insert(StringRef Pattern, unsigned LineNumber) {
  if (Pattern.empty())
    return createStringError(errc::invalid_argument,
                             "Supplied regex was blank");

  // Replace * with .*
  auto Regexp = Pattern.str();
  for (size_t pos = 0; (pos = Regexp.find('*', pos)) != std::string::npos;
       pos += strlen(".*")) {
    Regexp.replace(pos, strlen("*"), ".*");
  }

  Regexp = (Twine("^(") + StringRef(Regexp) + ")$").str();

  // Check that the regexp is valid.
  Regex CheckRE(Regexp);
  std::string REError;
```
- EN: This section centers on `insert`, `match`, `Glob` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `insert`, `match`, `Glob` 等符号展开，负责创建并初始化辅助对象或状态。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 121-180

```cpp
  if (!CheckRE.isValid(REError))
    return createStringError(errc::invalid_argument, REError);

  RegExes.emplace_back(Pattern, LineNumber, std::move(CheckRE));
  return Error::success();
}

unsigned RegexMatcher::match(StringRef Query) const {
  for (const auto &R : reverse(RegExes))
    if (R.Rg.match(Query))
      return R.LineNo;
  return 0;
}

Error GlobMatcher::insert(StringRef Pattern, unsigned LineNumber) {
  if (Pattern.empty())
    return createStringError(errc::invalid_argument, "Supplied glob was blank");

  auto Res = GlobPattern::create(Pattern, /*MaxSubPatterns=*/1024);
  if (auto Err = Res.takeError())
    return Err;
  Globs.emplace_back(Pattern, LineNumber, std::move(Res.get()));
  return Error::success();
}

void GlobMatcher::LazyInit() const {
  if (LLVM_LIKELY(Initialized))
    return;
  Initialized = true;
  for (const auto &[Idx, G] : enumerate(Globs)) {
    StringRef Prefix = G.Pattern.prefix();
    StringRef Suffix = G.Pattern.suffix();

    if (Suffix.empty() && Prefix.empty()) {
      // If both prefix and suffix are empty put into special tree to search by
      // substring in a middle.
      StringRef Substr = G.Pattern.longest_substr();
      if (!Substr.empty()) {
        // But only if substring is not empty. Searching this tree is more
        // expensive.
        auto &V = SubstrToGlob.emplace(Substr).first->second;
        V.emplace_back(Idx);
        continue;
      }
    }

    auto &SToGlob = PrefixSuffixToGlob.emplace(Prefix).first->second;
    auto &V = SToGlob.emplace(reverse(Suffix)).first->second;
    V.emplace_back(Idx);
  }
}

unsigned GlobMatcher::match(StringRef Query) const {
  LazyInit();

  int Best = -1;
  if (!PrefixSuffixToGlob.empty()) {
    for (const auto &[_, SToGlob] : PrefixSuffixToGlob.find_prefixes(Query)) {
      for (const auto &[_, V] : SToGlob.find_prefixes(reverse(Query))) {
        for (int Idx : reverse(V)) {
```
- EN: This section centers on `success`, `match`, `insert` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `success`, `match`, `insert` 等符号展开，负责创建并初始化辅助对象或状态。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 181-240

```cpp
          if (Best > Idx)
            break;
          const GlobMatcher::Glob &G = Globs[Idx];
          if (G.Pattern.match(Query)) {
            Best = Idx;
            // As soon as we find a match in the vector, we can break for this
            // vector, since the globs are already sorted by priority within the
            // prefix group. However, we continue searching other prefix groups
            // in the map, as they may contain a better match overall.
            break;
          }
        }
      }
    }
  }

  if (!SubstrToGlob.empty()) {
    // As we don't know when substring exactly starts, we will try all
    // possibilities. In most cases search will fail on first characters.
    for (StringRef Q = Query; !Q.empty(); Q = Q.drop_front()) {
      for (const auto &[_, V] : SubstrToGlob.find_prefixes(Q)) {
        for (int Idx : reverse(V)) {
          if (Best > Idx)
            break;
          const GlobMatcher::Glob &G = Globs[Idx];
          if (G.Pattern.match(Query)) {
            Best = Idx;
            // As soon as we find a match in the vector, we can break for this
            // vector, since the globs are already sorted by priority within the
            // prefix group. However, we continue searching other prefix groups
            // in the map, as they may contain a better match overall.
            break;
          }
        }
      }
    }
  }
  return Best < 0 ? 0 : Globs[Best].LineNo;
}

Matcher::Matcher(bool UseGlobs, bool RemoveDotSlash)
    : RemoveDotSlash(RemoveDotSlash) {
  if (UseGlobs)
    M.emplace<GlobMatcher>();
  else
    M.emplace<RegexMatcher>();
}

Error Matcher::insert(StringRef Pattern, unsigned LineNumber) {
  return std::visit([&](auto &V) { return V.insert(Pattern, LineNumber); }, M);
}

unsigned Matcher::match(StringRef Query) const {
  if (RemoveDotSlash)
    Query = llvm::sys::path::remove_leading_dotslash(Query);
  return std::visit([&](auto &V) -> unsigned { return V.match(Query); }, M);
}
} // namespace

class SpecialCaseList::Section::SectionImpl {
```
- EN: This section centers on `Matcher`, `insert`, `visit` and walks over records and applies per-item processing.
  CN: 这一段主要围绕 `Matcher`, `insert`, `visit` 等符号展开，负责遍历记录并执行逐项处理。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 241-300

```cpp
public:
  const Matcher *findMatcher(StringRef Prefix, StringRef Category) const;

  using SectionEntries = StringMap<StringMap<Matcher>>;

  explicit SectionImpl(bool UseGlobs)
      : SectionMatcher(UseGlobs, /*RemoveDotSlash=*/false) {}

  Matcher SectionMatcher;
  SectionEntries Entries;
};

// TODO: Refactor this to return Expected<...>
std::unique_ptr<SpecialCaseList>
SpecialCaseList::create(const std::vector<std::string> &Paths,
                        llvm::vfs::FileSystem &FS, std::string &Error) {
  std::unique_ptr<SpecialCaseList> SCL(new SpecialCaseList());
  if (SCL->createInternal(Paths, FS, Error))
    return SCL;
  return nullptr;
}

std::unique_ptr<SpecialCaseList> SpecialCaseList::create(const MemoryBuffer *MB,
                                                         std::string &Error) {
  std::unique_ptr<SpecialCaseList> SCL(new SpecialCaseList());
  if (SCL->createInternal(MB, Error))
    return SCL;
  return nullptr;
}

std::unique_ptr<SpecialCaseList>
SpecialCaseList::createOrDie(const std::vector<std::string> &Paths,
                             llvm::vfs::FileSystem &FS) {
  std::string Error;
  if (auto SCL = create(Paths, FS, Error))
    return SCL;
  report_fatal_error(Twine(Error));
}

bool SpecialCaseList::createInternal(const std::vector<std::string> &Paths,
                                     vfs::FileSystem &VFS, std::string &Error) {
  for (size_t i = 0; i < Paths.size(); ++i) {
    const auto &Path = Paths[i];
    ErrorOr<std::unique_ptr<MemoryBuffer>> FileOrErr =
        VFS.getBufferForFile(Path);
    if (std::error_code EC = FileOrErr.getError()) {
      Error = (Twine("can't open file '") + Path + "': " + EC.message()).str();
      return false;
    }
    std::string ParseError;
    if (!parse(i, FileOrErr.get().get(), ParseError)) {
      Error = (Twine("error parsing file '") + Path + "': " + ParseError).str();
      return false;
    }
  }
  return true;
}

bool SpecialCaseList::createInternal(const MemoryBuffer *MB,
                                     std::string &Error) {
```
- EN: This section centers on `SectionImpl`, `create`, `SCL` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `SectionImpl`, `create`, `SCL` 等符号展开，负责创建并初始化辅助对象或状态。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 301-360

```cpp
  if (!parse(0, MB, Error))
    return false;
  return true;
}

Expected<SpecialCaseList::Section *>
SpecialCaseList::addSection(StringRef SectionStr, unsigned FileNo,
                            unsigned LineNo, bool UseGlobs) {
  SectionStr = SectionStr.copy(StrAlloc);
  Sections.emplace_back(SectionStr, FileNo, UseGlobs);
  auto &Section = Sections.back();

  if (auto Err = Section.Impl->SectionMatcher.insert(SectionStr, LineNo)) {
    return createStringError(errc::invalid_argument,
                             "malformed section at line " + Twine(LineNo) +
                                 ": '" + SectionStr +
                                 "': " + toString(std::move(Err)));
  }

  return &Section;
}

bool SpecialCaseList::parse(unsigned FileIdx, const MemoryBuffer *MB,
                            std::string &Error) {
  unsigned long long Version = 2;

  StringRef Header = MB->getBuffer();
  if (Header.consume_front("#!special-case-list-v"))
    consumeUnsignedInteger(Header, 10, Version);

  // In https://reviews.llvm.org/D154014 we added glob support and planned
  // to remove regex support in patterns. We temporarily support the
  // original behavior using regexes if "#!special-case-list-v1" is the
  // first line of the file. For more details, see
  // https://discourse.llvm.org/t/use-glob-instead-of-regex-for-specialcaselists/71666
  bool UseGlobs = Version > 1;

  bool RemoveDotSlash = Version > 2;

  auto ErrOrSection = addSection("*", FileIdx, 1, true);
  if (auto Err = ErrOrSection.takeError()) {
    Error = toString(std::move(Err));
    return false;
  }
  Section::SectionImpl *CurrentImpl = ErrOrSection.get()->Impl.get();

  // This is the current list of prefixes for all existing users matching file
  // path. We may need parametrization in constructor in future.
  constexpr StringRef PathPrefixes[] = {"src", "!src", "mainfile", "source"};

  for (line_iterator LineIt(*MB, /*SkipBlanks=*/true, /*CommentMarker=*/'#');
       !LineIt.is_at_eof(); LineIt++) {
    unsigned LineNo = LineIt.line_number();
    StringRef Line = LineIt->trim();
    if (Line.empty())
      continue;

    // Save section names
    if (Line.starts_with("[")) {
      if (!Line.ends_with("]")) {
```
- EN: This section centers on `addSection`, `createStringError`, `parse` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `addSection`, `createStringError`, `parse` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 361-420

```cpp
        Error =
            ("malformed section header on line " + Twine(LineNo) + ": " + Line)
                .str();
        return false;
      }

      auto ErrOrSection =
          addSection(Line.drop_front().drop_back(), FileIdx, LineNo, UseGlobs);
      if (auto Err = ErrOrSection.takeError()) {
        Error = toString(std::move(Err));
        return false;
      }
      CurrentImpl = ErrOrSection.get()->Impl.get();
      continue;
    }

    // Get our prefix and unparsed glob.
    auto [Prefix, Postfix] = Line.split(":");
    if (Postfix.empty()) {
      // Missing ':' in the line.
      Error = ("malformed line " + Twine(LineNo) + ": '" + Line + "'").str();
      return false;
    }

    auto [Pattern, Category] = Postfix.split("=");
    auto [It, _] = CurrentImpl->Entries[Prefix].try_emplace(
        Category, UseGlobs,
        RemoveDotSlash && llvm::is_contained(PathPrefixes, Prefix));
    Pattern = Pattern.copy(StrAlloc);
    if (auto Err = It->second.insert(Pattern, LineNo)) {
      Error =
          (Twine("malformed ") + (UseGlobs ? "glob" : "regex") + " in line " +
           Twine(LineNo) + ": '" + Pattern + "': " + toString(std::move(Err)))
              .str();
      return false;
    }
  }

  return true;
}

SpecialCaseList::~SpecialCaseList() = default;

bool SpecialCaseList::inSection(StringRef Section, StringRef Prefix,
                                StringRef Query, StringRef Category) const {
  auto [FileIdx, LineNo] = inSectionBlame(Section, Prefix, Query, Category);
  return LineNo;
}

std::pair<unsigned, unsigned>
SpecialCaseList::inSectionBlame(StringRef Section, StringRef Prefix,
                                StringRef Query, StringRef Category) const {
  for (const auto &S : reverse(Sections)) {
    if (S.Impl->SectionMatcher.matchAny(Section)) {
      unsigned Blame = S.getLastMatch(Prefix, Query, Category);
      if (Blame)
        return {S.FileIdx, Blame};
    }
  }
  return NotFound;
```
- EN: This section centers on `addSection`, `is_contained`, `Twine` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `addSection`, `is_contained`, `Twine` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 421-461

```cpp
}

SpecialCaseList::Section::Section(StringRef Str, unsigned FileIdx,
                                  bool UseGlobs)
    : Name(Str), FileIdx(FileIdx),
      Impl(std::make_unique<SectionImpl>(UseGlobs)) {}

SpecialCaseList::Section::Section(Section &&) = default;

SpecialCaseList::Section::~Section() = default;

bool SpecialCaseList::Section::matchName(StringRef Name) const {
  return Impl->SectionMatcher.matchAny(Name);
}

const Matcher *
SpecialCaseList::Section::SectionImpl::findMatcher(StringRef Prefix,
                                                   StringRef Category) const {
  SectionEntries::const_iterator I = Entries.find(Prefix);
  if (I == Entries.end())
    return nullptr;
  StringMap<Matcher>::const_iterator II = I->second.find(Category);
  if (II == I->second.end())
    return nullptr;

  return &II->second;
}

unsigned SpecialCaseList::Section::getLastMatch(StringRef Prefix,
                                                StringRef Query,
                                                StringRef Category) const {
  if (const Matcher *M = Impl->findMatcher(Prefix, Category))
    return M->match(Query);
  return 0;
}

bool SpecialCaseList::Section::hasPrefix(StringRef Prefix) const {
  return Impl->Entries.contains(Prefix);
}

} // namespace llvm
```
- EN: This section centers on `Section`, `matchName`, `findMatcher` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `Section`, `matchName`, `findMatcher` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and updates helper containers and temporary state.
  CN: 在这一段中，代码检查条件并处理边界情况，并更新辅助容器和临时状态。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `for`, `RegexMatcher`, `Reg`, `insert`, `match`, `Glob` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Parsing, decoding, and structural validation. / 重点关注解析、解码与结构校验。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/Support/SpecialCaseList.h`, `llvm/ADT/RadixTree.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringMap.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/iterator_range.h`, `llvm/Support/Compiler.h`, `llvm/Support/GlobPattern.h`, `llvm/Support/LineIterator.h`, `llvm/Support/MemoryBuffer.h`, `llvm/Support/Regex.h`, `llvm/Support/VirtualFileSystem.h`, `llvm/Support/raw_ostream.h`
- Standard library / 标准库: `memory`, `stdio.h`, `string`, `system_error`, `utility`, `variant`, `vector`
- Other/system headers / 其他或系统头文件: None / 无
- Related symbols / 相关符号: `for`, `RegexMatcher`, `Reg`, `GlobMatcher`, `Glob`, `insert`, `match`, `LazyInit`
