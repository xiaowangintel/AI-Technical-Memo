# SourceMgrUtils.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Tools/lsp-server-support/SourceMgrUtils.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements reusable command-line tool support built on top of MLIR libraries.
  - **CN**: 实现构建于 MLIR 库之上的可复用命令行工具支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===--- SourceMgrUtils.cpp - SourceMgr LSP Utils -------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及高层摘要。

### Lines 8-16
```cpp

#include "mlir/Tools/lsp-server-support/SourceMgrUtils.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/Support/Path.h"
#include <optional>

using namespace mlir;
using namespace mlir::lsp;

```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/Tools/lsp-server-support/SourceMgrUtils.h`, `llvm/ADT/StringExtras.h`, `llvm/Support/Path.h`, `optional`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/Tools/lsp-server-support/SourceMgrUtils.h`, `llvm/ADT/StringExtras.h`, `llvm/Support/Path.h`, `optional`。

### Lines 17-23
```cpp
using llvm::lsp::Hover;
using llvm::lsp::Range;
using llvm::lsp::URIForFile;

//===----------------------------------------------------------------------===//
// Utils
//===----------------------------------------------------------------------===//
```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 24-33
```cpp

/// Find the end of a string whose contents start at the given `curPtr`. Returns
/// the position at the end of the string, after a terminal or invalid character
/// (e.g. `"` or `\0`).
static const char *lexLocStringTok(const char *curPtr) {
  while (char c = *curPtr++) {
    // Check for various terminal characters.
    if (StringRef("\"\n\v\f").contains(c))
      return curPtr;

```
- **EN**: Implements logic around `lexLocStringTok`, `StringRef`.
- **CN**: 围绕 `lexLocStringTok`、`StringRef` 实现具体逻辑。

### Lines 34-45
```cpp
    // Check for escape sequences.
    if (c == '\\') {
      // Check a few known escapes and \xx hex digits.
      if (*curPtr == '"' || *curPtr == '\\' || *curPtr == 'n' || *curPtr == 't')
        ++curPtr;
      else if (llvm::isHexDigit(*curPtr) && llvm::isHexDigit(curPtr[1]))
        curPtr += 2;
      else
        return curPtr;
    }
  }

```
- **EN**: Implements logic around `isHexDigit`.
- **CN**: 围绕 `isHexDigit` 实现具体逻辑。

### Lines 46-55
```cpp
  // If we hit this point, we've reached the end of the buffer. Update the end
  // pointer to not point past the buffer.
  return curPtr - 1;
}

SMRange lsp::convertTokenLocToRange(SMLoc loc, StringRef identifierChars) {
  if (!loc.isValid())
    return SMRange();
  const char *curPtr = loc.getPointer();

```
- **EN**: Implements logic around `convertTokenLocToRange`, `isValid`, `SMRange`, `getPointer`.
- **CN**: 围绕 `convertTokenLocToRange`、`isValid`、`SMRange`、`getPointer` 实现具体逻辑。

### Lines 56-66
```cpp
  // Check if this is a string token.
  if (*curPtr == '"') {
    curPtr = lexLocStringTok(curPtr + 1);

    // Otherwise, default to handling an identifier.
  } else {
    // Return if the given character is a valid identifier character.
    auto isIdentifierChar = [=](char c) {
      return isalnum(c) || c == '_' || identifierChars.contains(c);
    };

```
- **EN**: Implements logic around `lexLocStringTok`, `isalnum`.
- **CN**: 围绕 `lexLocStringTok`、`isalnum` 实现具体逻辑。

### Lines 67-73
```cpp
    while (*curPtr && isIdentifierChar(*(++curPtr)))
      continue;
  }

  return SMRange(loc, SMLoc::getFromPointer(curPtr));
}

```
- **EN**: Implements logic around `isIdentifierChar`, `SMRange`.
- **CN**: 围绕 `isIdentifierChar`、`SMRange` 实现具体逻辑。

### Lines 74-87
```cpp
std::optional<std::string>
lsp::extractSourceDocComment(llvm::SourceMgr &sourceMgr, SMLoc loc) {
  // This is a heuristic, and isn't intended to cover every case, but should
  // cover the most common. We essentially look for a comment preceding the
  // line, and if we find one, use that as the documentation.
  if (!loc.isValid())
    return std::nullopt;
  int bufferId = sourceMgr.FindBufferContainingLoc(loc);
  if (bufferId == 0)
    return std::nullopt;
  const char *bufferStart =
      sourceMgr.getMemoryBuffer(bufferId)->getBufferStart();
  StringRef buffer(bufferStart, loc.getPointer() - bufferStart);

```
- **EN**: Implements logic around `extractSourceDocComment`, `isValid`, `FindBufferContainingLoc`, `getMemoryBuffer`, and 1 more symbols.
- **CN**: 围绕 `extractSourceDocComment`、`isValid`、`FindBufferContainingLoc`、`getMemoryBuffer` 等另外 1 个符号 实现具体逻辑。

### Lines 88-97
```cpp
  // Pop the last line from the buffer string.
  auto popLastLine = [&]() -> std::optional<StringRef> {
    size_t newlineOffset = buffer.find_last_of('\n');
    if (newlineOffset == StringRef::npos)
      return std::nullopt;
    StringRef lastLine = buffer.drop_front(newlineOffset).trim();
    buffer = buffer.take_front(newlineOffset);
    return lastLine;
  };

```
- **EN**: Implements logic around `find_last_of`, `drop_front`, `take_front`.
- **CN**: 围绕 `find_last_of`、`drop_front`、`take_front` 实现具体逻辑。

### Lines 98-108
```cpp
  // Try to pop the current line.
  if (!popLastLine())
    return std::nullopt;

  // Try to parse a comment string from the source file.
  SmallVector<StringRef> commentLines;
  while (std::optional<StringRef> line = popLastLine()) {
    // Check for a comment at the beginning of the line.
    if (!line->starts_with("//"))
      break;

```
- **EN**: Implements logic around `popLastLine`, `starts_with`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `popLastLine`、`starts_with` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 109-117
```cpp
    // Extract the document string from the comment.
    commentLines.push_back(line->ltrim('/'));
  }

  if (commentLines.empty())
    return std::nullopt;
  return llvm::join(llvm::reverse(commentLines), "\n");
}

```
- **EN**: Implements logic around `push_back`, `empty`, `join`.
- **CN**: 围绕 `push_back`、`empty`、`join` 实现具体逻辑。

### Lines 118-125
```cpp
bool lsp::contains(SMRange range, SMLoc loc) {
  return range.Start.getPointer() <= loc.getPointer() &&
         loc.getPointer() < range.End.getPointer();
}

//===----------------------------------------------------------------------===//
// SourceMgrInclude
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `contains`, `getPointer`.
- **CN**: 围绕 `contains`、`getPointer` 实现具体逻辑。

### Lines 126-136
```cpp

Hover SourceMgrInclude::buildHover() const {
  Hover hover(range);
  {
    llvm::raw_string_ostream hoverOS(hover.contents.value);
    hoverOS << "`" << llvm::sys::path::filename(uri.file()) << "`\n***\n"
            << uri.file();
  }
  return hover;
}

```
- **EN**: Implements logic around `buildHover`, `hover`, `hoverOS`, `filename`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `buildHover`、`hover`、`hoverOS`、`filename` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 137-145
```cpp
void lsp::gatherIncludeFiles(llvm::SourceMgr &sourceMgr,
                             SmallVectorImpl<SourceMgrInclude> &includes) {
  for (unsigned i = 1, e = sourceMgr.getNumBuffers(); i < e; ++i) {
    // Check to see if this file was included by the main file.
    SMLoc includeLoc = sourceMgr.getBufferInfo(i + 1).IncludeLoc;
    if (!includeLoc.isValid() || sourceMgr.FindBufferContainingLoc(
                                     includeLoc) != sourceMgr.getMainFileID())
      continue;

```
- **EN**: Implements logic around `gatherIncludeFiles`, `getNumBuffers`, `getBufferInfo`, `isValid`, and 1 more symbols.
- **CN**: 围绕 `gatherIncludeFiles`、`getNumBuffers`、`getBufferInfo`、`isValid` 等另外 1 个符号 实现具体逻辑。

### Lines 146-154
```cpp
    // Try to build a URI for this file path.
    auto *buffer = sourceMgr.getMemoryBuffer(i + 1);
    llvm::SmallString<256> path(buffer->getBufferIdentifier());
    llvm::sys::path::remove_dots(path, /*remove_dot_dot=*/true);

    llvm::Expected<URIForFile> includedFileURI = URIForFile::fromFile(path);
    if (!includedFileURI)
      continue;

```
- **EN**: Implements logic around `getMemoryBuffer`, `path`, `remove_dots`, `fromFile`.
- **CN**: 围绕 `getMemoryBuffer`、`path`、`remove_dots`、`fromFile` 实现具体逻辑。

### Lines 155-164
```cpp
    // Find the end of the include token.
    const char *includeStart = includeLoc.getPointer() - 2;
    while (*(--includeStart) != '\"')
      continue;

    // Push this include.
    SMRange includeRange(SMLoc::getFromPointer(includeStart), includeLoc);
    includes.emplace_back(*includedFileURI, Range(sourceMgr, includeRange));
  }
}
```
- **EN**: Implements logic around `getPointer`, `includeRange`, `emplace_back`.
- **CN**: 围绕 `getPointer`、`includeRange`、`emplace_back` 实现具体逻辑。

## Key Concepts / 关键概念

- **Tooling support / 工具链支持**:
  - **EN**: Builds reusable infrastructure for opt-like tools, translation drivers, and CLIs.
  - **CN**: 为类似 opt 的工具、翻译驱动和命令行程序构建可复用基础设施。
- **SSA and region-based IR / SSA 与基于 Region 的 IR**:
  - **EN**: The file works with MLIR operations, blocks, values, or regions that form MLIR's hierarchical SSA IR.
  - **CN**: 该文件处理 MLIR 操作、块、值或 Region，它们共同构成 MLIR 的层次化 SSA IR。
- **LLVM ADT containers / LLVM ADT 容器**:
  - **EN**: The implementation uses LLVM containers for performance-conscious storage and traversal.
  - **CN**: 该实现使用 LLVM 容器来进行兼顾性能的存储与遍历。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `mlir/Tools/lsp-server-support/SourceMgrUtils.h`, `llvm/ADT/StringExtras.h`, `llvm/Support/Path.h`
- **Standard-library headers / 标准库头文件**: `<optional>`
- **Subsystem categories / 子系统类别**: tooling support declarations / 工具支持声明 (1), LLVM ADT containers and utilities / LLVM ADT 容器与工具 (1), LLVM support-library helpers / LLVM Support 库辅助工具 (1)
