# LVSupport.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/DebugInfo/LogicalView/Core/LVSupport.cpp`
- Repository: `llvm-project`
- Purpose (EN): This implements the supporting functions.
- Purpose (CN): 该文件位于 LLVM 的 `DebugInfo/LogicalView/Core` 目录中，主要实现与 `LVSupport` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===-- LVSupport.cpp -----------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This implements the supporting functions.
//
//===----------------------------------------------------------------------===//

#include "llvm/DebugInfo/LogicalView/Core/LVSupport.h"
#include "llvm/Support/FormatVariadic.h"

using namespace llvm;
using namespace llvm::logicalview;

#define DEBUG_TYPE "Support"

namespace {
// Unique string pool instance used by all logical readers.
LVStringPool StringPool;
} // namespace
LVStringPool &llvm::logicalview::getStringPool() { return StringPool; }

// Perform the following transformations to the given 'Path':
// - all characters to lowercase.
// - '\\' into '/' (Platform independent).
// - '//' into '/'
std::string llvm::logicalview::transformPath(StringRef Path) {
  std::string Name(Path);
  llvm::transform(Name, Name.begin(), tolower);
  llvm::replace(Name, '\\', '/');

  // Remove all duplicate slashes.
  size_t Pos = 0;
  while ((Pos = Name.find("//", Pos)) != std::string::npos)
    Name.erase(Pos, 1);

```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 2 direct dependencies, including `llvm/DebugInfo/LogicalView/Core/LVSupport.h`, `llvm/Support/FormatVariadic.h`.
  CN: 引入了 2 个直接依赖，其中包括 `llvm/DebugInfo/LogicalView/Core/LVSupport.h`, `llvm/Support/FormatVariadic.h`。
- EN: This section centers on `transformPath`, `Name`, `transform` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `transformPath`, `Name`, `transform` 等符号展开，负责实现局部控制流程与状态维护。

### Lines 41-80

```cpp
  return Name;
}

// Convert the given 'Path' to lowercase and change any matching character
// from 'CharSet' into '_'.
// The characters in 'CharSet' are:
//   '/', '\', '<', '>', '.', ':', '%', '*', '?', '|', '"', ' '.
std::string llvm::logicalview::flattenedFilePath(StringRef Path) {
  std::string Name(Path);
  llvm::transform(Name, Name.begin(), tolower);

  const char *CharSet = "/\\<>.:%*?|\" ";
  char *Input = Name.data();
  while (Input && *Input) {
    Input = strpbrk(Input, CharSet);
    if (Input)
      *Input++ = '_';
  };
  return Name;
}

using LexicalEntry = std::pair<size_t, size_t>;
using LexicalIndexes = SmallVector<LexicalEntry, 10>;

static LexicalIndexes getAllLexicalIndexes(StringRef Name) {
  if (Name.empty())
    return {};

  size_t AngleCount = 0;
  size_t ColonSeen = 0;
  size_t Current = 0;

  LexicalIndexes Indexes;

#ifndef NDEBUG
  auto PrintLexicalEntry = [&]() {
    LexicalEntry Entry = Indexes.back();
    llvm::dbgs() << formatv(
        "'{0}:{1}', '{2}'\n", Entry.first, Entry.second,
        Name.substr(Entry.first, Entry.second - Entry.first + 1));
```
- EN: This section centers on `flattenedFilePath`, `Name`, `transform` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `flattenedFilePath`, `Name`, `transform` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 81-120

```cpp
  };
#endif

  size_t Length = Name.size();
  for (size_t Index = 0; Index < Length; ++Index) {
    LLVM_DEBUG({
      llvm::dbgs() << formatv("Index: '{0}', Char: '{1}'\n", Index,
                              Name[Index]);
    });
    switch (Name[Index]) {
    case '<':
      ++AngleCount;
      break;
    case '>':
      --AngleCount;
      break;
    case ':':
      ++ColonSeen;
      break;
    }
    if (ColonSeen == 2) {
      if (!AngleCount) {
        Indexes.push_back(LexicalEntry(Current, Index - 2));
        Current = Index + 1;
        LLVM_DEBUG({ PrintLexicalEntry(); });
      }
      ColonSeen = 0;
      continue;
    }
  }

  // Store last component.
  Indexes.push_back(LexicalEntry(Current, Length - 1));
  LLVM_DEBUG({ PrintLexicalEntry(); });
  return Indexes;
}

LVLexicalComponent llvm::logicalview::getInnerComponent(StringRef Name) {
  if (Name.empty())
    return {};
```
- EN: This section centers on `getInnerComponent` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `getInnerComponent` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and iterates over collections, ranges, or records.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并遍历集合、区间或记录。

### Lines 121-160

```cpp

  LexicalIndexes Indexes = getAllLexicalIndexes(Name);
  if (Indexes.size() == 1)
    return std::make_tuple(StringRef(), Name);

  LexicalEntry BeginEntry = Indexes.front();
  LexicalEntry EndEntry = Indexes[Indexes.size() - 2];
  StringRef Outer =
      Name.substr(BeginEntry.first, EndEntry.second - BeginEntry.first + 1);

  LexicalEntry LastEntry = Indexes.back();
  StringRef Inner =
      Name.substr(LastEntry.first, LastEntry.second - LastEntry.first + 1);

  return std::make_tuple(Outer, Inner);
}

LVStringRefs llvm::logicalview::getAllLexicalComponents(StringRef Name) {
  if (Name.empty())
    return {};

  LexicalIndexes Indexes = getAllLexicalIndexes(Name);
  LVStringRefs Components;
  for (const LexicalEntry &Entry : Indexes)
    Components.push_back(
        Name.substr(Entry.first, Entry.second - Entry.first + 1));

  return Components;
}

std::string llvm::logicalview::getScopedName(const LVStringRefs &Components,
                                             StringRef BaseName) {
  if (Components.empty())
    return {};
  std::string Name(BaseName);
  raw_string_ostream Stream(Name);
  if (BaseName.size())
    Stream << "::";
  Stream << Components[0];
  for (LVStringRefs::size_type Index = 1; Index < Components.size(); ++Index)
```
- EN: This section centers on `make_tuple`, `getAllLexicalComponents`, `getScopedName` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `make_tuple`, `getAllLexicalComponents`, `getScopedName` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 161-163

```cpp
    Stream << "::" << Components[Index];
  return Name;
}
```
- EN: This range returns the resulting value to its callers.
  CN: 这一段将结果返回给调用方。

## Key Concepts / 关键概念
- Domain / 领域: debug information infrastructure / 调试信息基础设施
- Core symbols / 核心符号: `transformPath`, `Name`, `transform`, `replace` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Helper routines coordinate local state, control flow, and result construction. / 通过辅助例程协调局部状态、控制流与结果构造。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/DebugInfo/LogicalView/Core/LVSupport.h`, `llvm/Support/FormatVariadic.h`
- Standard library / 标准库: None / 无
- Other/system headers / 其他或系统头文件: None / 无
- Related symbols / 相关符号: `transformPath`, `Name`, `transform`, `replace`, `flattenedFilePath`
