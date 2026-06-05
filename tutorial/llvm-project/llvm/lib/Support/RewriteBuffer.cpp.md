# RewriteBuffer.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/RewriteBuffer.cpp`
- Repository: `llvm-project`
- Purpose (EN): Walk RewriteRope chunks efficiently using MoveToNextPiece() instead of the character iterator.
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `RewriteBuffer` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
//===- RewriteBuffer.h - Buffer rewriting interface -----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/ADT/RewriteBuffer.h"
#include "llvm/Support/raw_ostream.h"

using namespace llvm;

raw_ostream &RewriteBuffer::write(raw_ostream &Stream) const {
  // Walk RewriteRope chunks efficiently using MoveToNextPiece() instead of the
  // character iterator.
  for (RopePieceBTreeIterator I = begin(), E = end(); I != E;
       I.MoveToNextPiece())
    Stream << I.piece();
  return Stream;
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 2 direct dependencies, including `llvm/ADT/RewriteBuffer.h`, `llvm/Support/raw_ostream.h`.
  CN: 引入了 2 个直接依赖，其中包括 `llvm/ADT/RewriteBuffer.h`, `llvm/Support/raw_ostream.h`。
- EN: In this range, the code iterates over collections, ranges, or records and returns the resulting value to its callers.
  CN: 在这一段中，代码遍历集合、区间或记录，并将结果返回给调用方。

### Lines 21-40

```cpp
}

/// Return true if this character is non-new-line whitespace:
/// ' ', '\\t', '\\f', '\\v', '\\r'.
static inline bool isWhitespaceExceptNL(unsigned char c) {
  return c == ' ' || c == '\t' || c == '\f' || c == '\v' || c == '\r';
}

void RewriteBuffer::RemoveText(unsigned OrigOffset, unsigned Size,
                               bool removeLineIfEmpty) {
  // Nothing to remove, exit early.
  if (Size == 0)
    return;

  unsigned RealOffset = getMappedOffset(OrigOffset, true);
  assert(RealOffset + Size <= Buffer.size() && "Invalid location");

  // Remove the dead characters.
  Buffer.erase(RealOffset, Size);

```
- EN: This section centers on `isWhitespaceExceptNL`, `RemoveText`, `assert` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `isWhitespaceExceptNL`, `RemoveText`, `assert` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 41-60

```cpp
  // Add a delta so that future changes are offset correctly.
  AddReplaceDelta(OrigOffset, -Size);

  if (removeLineIfEmpty) {
    // Find the line that the remove occurred and if it is completely empty
    // remove the line as well.

    iterator curLineStart = begin();
    unsigned curLineStartOffs = 0;
    iterator posI = begin();
    for (unsigned i = 0; i != RealOffset; ++i) {
      if (*posI == '\n') {
        curLineStart = posI;
        ++curLineStart;
        curLineStartOffs = i + 1;
      }
      ++posI;
    }

    unsigned lineSize = 0;
```
- EN: This section centers on `AddReplaceDelta` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `AddReplaceDelta` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 61-80

```cpp
    posI = curLineStart;
    while (posI != end() && isWhitespaceExceptNL(*posI)) {
      ++posI;
      ++lineSize;
    }
    if (posI != end() && *posI == '\n') {
      Buffer.erase(curLineStartOffs, lineSize + 1 /* + '\n'*/);
      // FIXME: Here, the offset of the start of the line is supposed to be
      // expressed in terms of the original input not the "real" rewrite
      // buffer.  How do we compute that reliably?  It might be tempting to use
      // curLineStartOffs + OrigOffset - RealOffset, but that assumes the
      // difference between the original and real offset is the same at the
      // removed text and at the start of the line, but that's not true if
      // edits were previously made earlier on the line.  This bug is also
      // documented by a FIXME on the definition of
      // clang::Rewriter::RewriteOptions::RemoveLineIfEmpty.  A reproducer for
      // the implementation below is the test RemoveLineIfEmpty in
      // clang/unittests/Rewrite/RewriteBufferTest.cpp.
      AddReplaceDelta(curLineStartOffs, -(lineSize + 1 /* + '\n'*/));
    }
```
- EN: This section centers on `AddReplaceDelta` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `AddReplaceDelta` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 81-100

```cpp
  }
}

void RewriteBuffer::InsertText(unsigned OrigOffset, StringRef Str,
                               bool InsertAfter) {
  // Nothing to insert, exit early.
  if (Str.empty())
    return;

  unsigned RealOffset = getMappedOffset(OrigOffset, InsertAfter);
  Buffer.insert(RealOffset, Str.begin(), Str.end());

  // Add a delta so that future changes are offset correctly.
  AddInsertDelta(OrigOffset, Str.size());
}

/// ReplaceText - This method replaces a range of characters in the input
/// buffer with a new string.  This is effectively a combined "remove+insert"
/// operation.
void RewriteBuffer::ReplaceText(unsigned OrigOffset, unsigned OrigLength,
```
- EN: This section centers on `InsertText`, `AddInsertDelta` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `InsertText`, `AddInsertDelta` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and returns the resulting value to its callers.
  CN: 在这一段中，代码检查条件并处理边界情况，并将结果返回给调用方。

### Lines 101-107

```cpp
                                StringRef NewStr) {
  unsigned RealOffset = getMappedOffset(OrigOffset, true);
  Buffer.erase(RealOffset, OrigLength);
  Buffer.insert(RealOffset, NewStr.begin(), NewStr.end());
  if (OrigLength != NewStr.size())
    AddReplaceDelta(OrigOffset, NewStr.size() - OrigLength);
}
```
- EN: This range checks conditions and handles edge cases.
  CN: 这一段检查条件并处理边界情况。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `isWhitespaceExceptNL`, `RemoveText`, `assert`, `AddReplaceDelta` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Formatting, emission, and outward serialization. / 重点关注格式化、输出与序列化。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/ADT/RewriteBuffer.h`, `llvm/Support/raw_ostream.h`
- Standard library / 标准库: None / 无
- Other/system headers / 其他或系统头文件: None / 无
- Related symbols / 相关符号: `isWhitespaceExceptNL`, `RemoveText`, `assert`, `AddReplaceDelta`, `InsertText`
