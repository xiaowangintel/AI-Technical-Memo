# InlineInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/DebugInfo/GSYM/InlineInfo.cpp`
- Repository: `llvm-project`
- Purpose (EN): If this is the top level that represents the concrete function, there will be no name and we shoud clear the inline stack.
- Purpose (CN): 该文件位于 LLVM 的 `DebugInfo/GSYM` 目录中，主要实现与 `InlineInfo` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===- InlineInfo.cpp -------------------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/DebugInfo/GSYM/InlineInfo.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/DebugInfo/GSYM/FileEntry.h"
#include "llvm/DebugInfo/GSYM/FileWriter.h"
#include "llvm/DebugInfo/GSYM/GsymDataExtractor.h"
#include "llvm/DebugInfo/GSYM/GsymReader.h"
#include "llvm/Support/InterleavedRange.h"
#include <inttypes.h>

using namespace llvm;
using namespace gsym;


raw_ostream &llvm::gsym::operator<<(raw_ostream &OS, const InlineInfo &II) {
  if (!II.isValid())
    return OS;
  OS << interleaved(II.Ranges, " ");
  OS << " Name = " << HEX32(II.Name) << ", CallFile = " << II.CallFile
     << ", CallLine = " << II.CallFile << '\n';
  for (const auto &Child : II.Children)
    OS << Child;
  return OS;
}

static bool getInlineStackHelper(const InlineInfo &II, uint64_t Addr,
    std::vector<const InlineInfo *> &InlineStack) {
  if (II.Ranges.contains(Addr)) {
    // If this is the top level that represents the concrete function,
    // there will be no name and we shoud clear the inline stack. Otherwise
    // we have found an inline call stack that we need to insert.
    if (II.Name != 0)
      InlineStack.insert(InlineStack.begin(), &II);
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 8 direct dependencies, including `llvm/DebugInfo/GSYM/InlineInfo.h`, `llvm/ADT/StringExtras.h`, `llvm/DebugInfo/GSYM/FileEntry.h`, `llvm/DebugInfo/GSYM/FileWriter.h`.
  CN: 引入了 8 个直接依赖，其中包括 `llvm/DebugInfo/GSYM/InlineInfo.h`, `llvm/ADT/StringExtras.h`, `llvm/DebugInfo/GSYM/FileEntry.h`, `llvm/DebugInfo/GSYM/FileWriter.h`。
- EN: This section centers on `interleaved`, `getInlineStackHelper` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `interleaved`, `getInlineStackHelper` 等符号展开，负责查询现有状态并返回计算出的元数据。

### Lines 41-80

```cpp
    for (const auto &Child : II.Children) {
      if (::getInlineStackHelper(Child, Addr, InlineStack))
        break;
    }
    return !InlineStack.empty();
  }
  return false;
}

std::optional<InlineInfo::InlineArray>
InlineInfo::getInlineStack(uint64_t Addr) const {
  InlineArray Result;
  if (getInlineStackHelper(*this, Addr, Result))
    return Result;
  return std::nullopt;
}

/// Skip an InlineInfo object in the specified data at the specified offset.
///
/// Used during the InlineInfo::lookup() call to quickly skip child InlineInfo
/// objects where the addres ranges isn't contained in the InlineInfo object
/// or its children. This avoids allocations by not appending child InlineInfo
/// objects to the InlineInfo::Children array.
///
/// \param Data The binary stream to read the data from.
///
/// \param Offset The byte offset within \a Data.
///
/// \param SkippedRanges If true, address ranges have already been skipped.

static bool skip(GsymDataExtractor &Data, uint64_t &Offset,
                 bool SkippedRanges) {
  if (!SkippedRanges) {
    if (skipRanges(Data, Offset) == 0)
      return false;
  }
  bool HasChildren = Data.getU8(&Offset) != 0;
  Data.getStringOffset(&Offset); // Skip Inline.Name.
  Data.getULEB128(&Offset); // Skip Inline.CallFile.
  Data.getULEB128(&Offset); // Skip Inline.CallLine.
```
- EN: This section centers on `getInlineStack`, `skip` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `getInlineStack`, `skip` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 81-120

```cpp
  if (HasChildren) {
    while (skip(Data, Offset, false /* SkippedRanges */))
      /* Do nothing */;
  }
  // We skipped a valid InlineInfo.
  return true;
}

/// A Lookup helper functions.
///
/// Used during the InlineInfo::lookup() call to quickly only parse an
/// InlineInfo object if the address falls within this object. This avoids
/// allocations by not appending child InlineInfo objects to the
/// InlineInfo::Children array and also skips any InlineInfo objects that do
/// not contain the address we are looking up.
///
/// \param Data The binary stream to read the data from.
///
/// \param Offset The byte offset within \a Data.
///
/// \param BaseAddr The address that the relative address range offsets are
///                 relative to.

static bool lookup(const GsymReader &GR, GsymDataExtractor &Data,
                   uint64_t &Offset, uint64_t BaseAddr, uint64_t Addr,
                   SourceLocations &SrcLocs, llvm::Error &Err) {
  InlineInfo Inline;
  decodeRanges(Inline.Ranges, Data, BaseAddr, Offset);
  if (Inline.Ranges.empty())
    return true;
  // Check if the address is contained within the inline information, and if
  // not, quickly skip this InlineInfo object and all its children.
  if (!Inline.Ranges.contains(Addr)) {
    skip(Data, Offset, true /* SkippedRanges */);
    return false;
  }

  // The address range is contained within this InlineInfo, add the source
  // location for this InlineInfo and any children that contain the address.
  bool HasChildren = Data.getU8(&Offset) != 0;
```
- EN: This section centers on `lookup`, `decodeRanges`, `skip` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `lookup`, `decodeRanges`, `skip` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 121-160

```cpp
  Inline.Name = Data.getStringOffset(&Offset);
  Inline.CallFile = (uint32_t)Data.getULEB128(&Offset);
  Inline.CallLine = (uint32_t)Data.getULEB128(&Offset);
  if (HasChildren) {
    // Child address ranges are encoded relative to the first address in the
    // parent InlineInfo object.
    const auto ChildBaseAddr = Inline.Ranges[0].start();
    bool Done = false;
    while (!Done)
      Done = lookup(GR, Data, Offset, ChildBaseAddr, Addr, SrcLocs, Err);
  }

  std::optional<FileEntry> CallFile = GR.getFile(Inline.CallFile);
  if (!CallFile) {
    Err = createStringError(std::errc::invalid_argument,
                            "failed to extract file[%" PRIu32 "]",
                            Inline.CallFile);
    return false;
  }

  if (CallFile->Dir || CallFile->Base) {
    SourceLocation SrcLoc;
    SrcLoc.Name = SrcLocs.back().Name;
    SrcLoc.Offset = SrcLocs.back().Offset;
    SrcLoc.Dir = GR.getString(CallFile->Dir);
    SrcLoc.Base = GR.getString(CallFile->Base);
    SrcLoc.Line = Inline.CallLine;
    SrcLocs.back().Name = GR.getString(Inline.Name);
    SrcLocs.back().Offset = Addr - Inline.Ranges[0].start();
    SrcLocs.push_back(SrcLoc);
  }
  return true;
}

llvm::Error InlineInfo::lookup(const GsymReader &GR, GsymDataExtractor &Data,
                               uint64_t BaseAddr, uint64_t Addr,
                               SourceLocations &SrcLocs) {
  // Call our recursive helper function starting at offset zero.
  uint64_t Offset = 0;
  llvm::Error Err = Error::success();
```
- EN: This section centers on `lookup` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `lookup` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 161-200

```cpp
  ::lookup(GR, Data, Offset, BaseAddr, Addr, SrcLocs, Err);
  return Err;
}

/// Decode an InlineInfo in Data at the specified offset.
///
/// A local helper function to decode InlineInfo objects. This function is
/// called recursively when parsing child InlineInfo objects.
///
/// \param Data The data extractor to decode from.
/// \param Offset The offset within \a Data to decode from.
/// \param BaseAddr The base address to use when decoding address ranges.
/// \returns An InlineInfo or an error describing the issue that was
/// encountered during decoding.
static llvm::Expected<InlineInfo> decode(GsymDataExtractor &Data,
                                         uint64_t &Offset, uint64_t BaseAddr) {
  InlineInfo Inline;
  if (!Data.isValidOffset(Offset))
    return createStringError(std::errc::io_error,
        "0x%8.8" PRIx64 ": missing InlineInfo address ranges data", Offset);
  decodeRanges(Inline.Ranges, Data, BaseAddr, Offset);
  if (Inline.Ranges.empty())
    return Inline;
  if (!Data.isValidOffsetForDataOfSize(Offset, 1))
    return createStringError(std::errc::io_error,
        "0x%8.8" PRIx64 ": missing InlineInfo uint8_t indicating children",
        Offset);
  bool HasChildren = Data.getU8(&Offset) != 0;
  if (!Data.isValidOffsetForDataOfSize(Offset, Data.getStringOffsetSize()))
    return createStringError(std::errc::io_error,
                             "0x%8.8" PRIx64 ": missing InlineInfo name",
                             Offset);
  Inline.Name = Data.getStringOffset(&Offset);
  if (!Data.isValidOffset(Offset))
    return createStringError(std::errc::io_error,
        "0x%8.8" PRIx64 ": missing ULEB128 for InlineInfo call file", Offset);
  Inline.CallFile = (uint32_t)Data.getULEB128(&Offset);
  if (!Data.isValidOffset(Offset))
    return createStringError(std::errc::io_error,
        "0x%8.8" PRIx64 ": missing ULEB128 for InlineInfo call line", Offset);
```
- EN: This section centers on `decode`, `decodeRanges` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `decode`, `decodeRanges` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 201-240

```cpp
  Inline.CallLine = (uint32_t)Data.getULEB128(&Offset);
  if (HasChildren) {
    // Child address ranges are encoded relative to the first address in the
    // parent InlineInfo object.
    const auto ChildBaseAddr = Inline.Ranges[0].start();
    while (true) {
      llvm::Expected<InlineInfo> Child = decode(Data, Offset, ChildBaseAddr);
      if (!Child)
        return Child.takeError();
      // InlineInfo with empty Ranges termintes a child sibling chain.
      if (Child.get().Ranges.empty())
        break;
      Inline.Children.emplace_back(std::move(*Child));
    }
  }
  return Inline;
}

llvm::Expected<InlineInfo> InlineInfo::decode(GsymDataExtractor &Data,
                                              uint64_t BaseAddr) {
  uint64_t Offset = 0;
  return ::decode(Data, Offset, BaseAddr);
}

llvm::Error InlineInfo::encode(FileWriter &O, uint64_t BaseAddr) const {
  // Users must verify the InlineInfo is valid prior to calling this funtion.
  // We don't want to emit any InlineInfo objects if they are not valid since
  // it will waste space in the GSYM file.
  if (!isValid())
    return createStringError(std::errc::invalid_argument,
                             "attempted to encode invalid InlineInfo object");
  encodeRanges(Ranges, O, BaseAddr);
  bool HasChildren = !Children.empty();
  O.writeU8(HasChildren);
  O.writeStringOffset(Name);
  O.writeULEB(CallFile);
  O.writeULEB(CallLine);
  if (HasChildren) {
    // Child address ranges are encoded as relative to the first
    // address in the Ranges for this object. This keeps the offsets
```
- EN: This section centers on `decode`, `encode`, `encodeRanges` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `decode`, `encode`, `encodeRanges` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 241-273

```cpp
    // small and allows for efficient encoding using ULEB offsets.
    const uint64_t ChildBaseAddr = Ranges[0].start();
    for (const auto &Child : Children) {
      // Make sure all child address ranges are contained in the parent address
      // ranges.
      for (const auto &ChildRange: Child.Ranges) {
        if (!Ranges.contains(ChildRange))
          return createStringError(std::errc::invalid_argument,
                                   "child range not contained in parent");
      }
      llvm::Error Err = Child.encode(O, ChildBaseAddr);
      if (Err)
        return Err;
    }

    // Terminate child sibling chain by emitting a zero. This zero will cause
    // the decodeAll() function above to return false and stop the decoding
    // of child InlineInfo objects that are siblings.
    O.writeULEB(0);
  }
  return Error::success();
}

static uint64_t GetTotalNumChildren(const InlineInfo &II) {
  uint64_t NumChildren = II.Children.size();
  for (const auto &Child : II.Children)
    NumChildren += GetTotalNumChildren(Child);
  return NumChildren;
}

bool InlineInfo::operator<(const InlineInfo &RHS) const {
  return GetTotalNumChildren(*this) < GetTotalNumChildren(RHS);
}
```
- EN: This section centers on `success`, `GetTotalNumChildren` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `success`, `GetTotalNumChildren` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

## Key Concepts / 关键概念
- Domain / 领域: debug information infrastructure / 调试信息基础设施
- Core symbols / 核心符号: `interleaved`, `getInlineStackHelper`, `getInlineStack`, `skip` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Parsing, decoding, and structural validation. / 重点关注解析、解码与结构校验。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/DebugInfo/GSYM/InlineInfo.h`, `llvm/ADT/StringExtras.h`, `llvm/DebugInfo/GSYM/FileEntry.h`, `llvm/DebugInfo/GSYM/FileWriter.h`, `llvm/DebugInfo/GSYM/GsymDataExtractor.h`, `llvm/DebugInfo/GSYM/GsymReader.h`, `llvm/Support/InterleavedRange.h`
- Standard library / 标准库: None / 无
- Other/system headers / 其他或系统头文件: `inttypes.h`
- Related symbols / 相关符号: `interleaved`, `getInlineStackHelper`, `getInlineStack`, `skip`, `lookup`
