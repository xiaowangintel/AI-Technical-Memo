# FormattedStream.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/FormattedStream.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains the implementation of formatted_raw_ostream.
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `FormattedStream` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===-- llvm/Support/FormattedStream.cpp - Formatted streams ----*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the implementation of formatted_raw_ostream.
//
//===----------------------------------------------------------------------===//

#include "llvm/Support/FormattedStream.h"
#include "llvm/Support/ConvertUTF.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/Unicode.h"
#include "llvm/Support/raw_ostream.h"
#include <algorithm>

using namespace llvm;

/// UpdatePosition - Examine the given char sequence and figure out which
/// column we end up in after output, and how many line breaks are contained.
/// This assumes that the input string is well-formed UTF-8, and takes into
/// account Unicode characters which render as multiple columns wide.
void formatted_raw_ostream::UpdatePosition(const char *Ptr, size_t Size) {
  unsigned &Column = Position.first;
  unsigned &Line = Position.second;

  auto ProcessUTF8CodePoint = [&Line, &Column](StringRef CP) {
    int Width = sys::unicode::columnWidthUTF8(CP);
    if (Width != sys::unicode::ErrorNonPrintableCharacter)
      Column += Width;

    // The only special whitespace characters we care about are single-byte.
    if (CP.size() > 1)
      return;

    switch (CP[0]) {
    case '\n':
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 6 direct dependencies, including `llvm/Support/FormattedStream.h`, `llvm/Support/ConvertUTF.h`, `llvm/Support/Debug.h`, `llvm/Support/Unicode.h`.
  CN: 引入了 6 个直接依赖，其中包括 `llvm/Support/FormattedStream.h`, `llvm/Support/ConvertUTF.h`, `llvm/Support/Debug.h`, `llvm/Support/Unicode.h`。
- EN: This section centers on `UpdatePosition` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `UpdatePosition` 等符号展开，负责实现局部控制流程与状态维护。

### Lines 41-80

```cpp
      Line += 1;
      [[fallthrough]];
    case '\r':
      Column = 0;
      break;
    case '\t':
      // Assumes tab stop = 8 characters.
      Column += (8 - (Column & 0x7)) & 0x7;
      break;
    }
  };

  // If we have a partial UTF-8 sequence from the previous buffer, check that
  // first.
  if (PartialUTF8Char.size()) {
    size_t BytesFromBuffer =
        getNumBytesForUTF8(PartialUTF8Char[0]) - PartialUTF8Char.size();
    if (Size < BytesFromBuffer) {
      // If we still don't have enough bytes for a complete code point, just
      // append what we have.
      PartialUTF8Char.append(StringRef(Ptr, Size));
      return;
    } else {
      // The first few bytes from the buffer will complete the code point.
      // Concatenate them and process their effect on the line and column
      // numbers.
      PartialUTF8Char.append(StringRef(Ptr, BytesFromBuffer));
      ProcessUTF8CodePoint(PartialUTF8Char);
      PartialUTF8Char.clear();
      Ptr += BytesFromBuffer;
      Size -= BytesFromBuffer;
    }
  }

  // Now scan the rest of the buffer.
  unsigned NumBytes;
  for (const char *End = Ptr + Size; Ptr < End; Ptr += NumBytes) {
    // Fast path for printable ASCII characters without special handling.
    if (*Ptr >= 0x20 && *Ptr <= 0x7e) {
      NumBytes = 1;
```
- EN: This section centers on `getNumBytesForUTF8`, `ProcessUTF8CodePoint` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `getNumBytesForUTF8`, `ProcessUTF8CodePoint` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 81-120

```cpp
      ++Column;
      continue;
    }

    NumBytes = getNumBytesForUTF8(*Ptr);

    // The buffer might end part way through a UTF-8 code unit sequence for a
    // Unicode scalar value if it got flushed. If this happens, we can't know
    // the display width until we see the rest of the code point. Stash the
    // bytes we do have, so that we can reconstruct the whole code point later,
    // even if the buffer is being flushed.
    if ((unsigned)(End - Ptr) < NumBytes) {
      PartialUTF8Char = StringRef(Ptr, End - Ptr);
      return;
    }

    ProcessUTF8CodePoint(StringRef(Ptr, NumBytes));
  }
}

/// ComputePosition - Examine the current output and update line and column
/// counts.
void formatted_raw_ostream::ComputePosition(const char *Ptr, size_t Size) {
  if (DisableScan)
    return;

  // If our previous scan pointer is inside the buffer, assume we already
  // scanned those bytes. This depends on raw_ostream to not change our buffer
  // in unexpected ways.
  if (Ptr <= Scanned && Scanned <= Ptr + Size)
    // Scan all characters added since our last scan to determine the new
    // column.
    UpdatePosition(Scanned, Size - (Scanned - Ptr));
  else
    UpdatePosition(Ptr, Size);

  // Update the scanning pointer.
  Scanned = Ptr + Size;
}

```
- EN: This section centers on `ProcessUTF8CodePoint`, `ComputePosition`, `UpdatePosition` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `ProcessUTF8CodePoint`, `ComputePosition`, `UpdatePosition` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and returns the resulting value to its callers.
  CN: 在这一段中，代码检查条件并处理边界情况，并将结果返回给调用方。

### Lines 121-160

```cpp
/// PadToColumn - Align the output to some column number.
///
/// \param NewCol - The column to move to.
///
formatted_raw_ostream &formatted_raw_ostream::PadToColumn(unsigned NewCol) {
  // Figure out what's in the buffer and add it to the column count.
  ComputePosition(getBufferStart(), GetNumBytesInBuffer());

  // Output spaces until we reach the desired column.
  indent(std::max(int(NewCol - getColumn()), 1));
  return *this;
}

void formatted_raw_ostream::write_impl(const char *Ptr, size_t Size) {
  // Figure out what's in the buffer and add it to the column count.
  ComputePosition(Ptr, Size);

  // Write the data to the underlying stream (which is unbuffered, so
  // the data will be immediately written out).
  TheStream->write(Ptr, Size);

  // Reset the scanning pointer.
  Scanned = nullptr;
}

/// fouts() - This returns a reference to a formatted_raw_ostream for
/// standard output.  Use it like: fouts() << "foo" << "bar";
formatted_raw_ostream &llvm::fouts() {
  static formatted_raw_ostream S(outs());
  return S;
}

/// ferrs() - This returns a reference to a formatted_raw_ostream for
/// standard error.  Use it like: ferrs() << "foo" << "bar";
formatted_raw_ostream &llvm::ferrs() {
  static formatted_raw_ostream S(errs());
  return S;
}

/// fdbgs() - This returns a reference to a formatted_raw_ostream for
```
- EN: This section centers on `ComputePosition`, `indent`, `write_impl` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `ComputePosition`, `indent`, `write_impl` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: This range returns the resulting value to its callers.
  CN: 这一段将结果返回给调用方。

### Lines 161-165

```cpp
/// the debug stream.  Use it like: fdbgs() << "foo" << "bar";
formatted_raw_ostream &llvm::fdbgs() {
  static formatted_raw_ostream S(dbgs());
  return S;
}
```
- EN: This section centers on `S` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `S` 等符号展开，负责实现局部控制流程与状态维护。
- EN: This range returns the resulting value to its callers.
  CN: 这一段将结果返回给调用方。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `UpdatePosition`, `getNumBytesForUTF8`, `ProcessUTF8CodePoint`, `ComputePosition` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Formatting, emission, and outward serialization. / 重点关注格式化、输出与序列化。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/Support/FormattedStream.h`, `llvm/Support/ConvertUTF.h`, `llvm/Support/Debug.h`, `llvm/Support/Unicode.h`, `llvm/Support/raw_ostream.h`
- Standard library / 标准库: `algorithm`
- Other/system headers / 其他或系统头文件: None / 无
- Related symbols / 相关符号: `UpdatePosition`, `getNumBytesForUTF8`, `ProcessUTF8CodePoint`, `ComputePosition`, `indent`
