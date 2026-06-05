# FunctionInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/DebugInfo/GSYM/FunctionInfo.cpp`
- Repository: `llvm-project`
- Purpose (EN): / FunctionInfo information type that is used to encode the optional data / that is associated with a FunctionInfo object.
- Purpose (CN): 该文件位于 LLVM 的 `DebugInfo/GSYM` 目录中，主要实现与 `FunctionInfo` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===- FunctionInfo.cpp ---------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/DebugInfo/GSYM/FunctionInfo.h"
#include "llvm/DebugInfo/GSYM/FileWriter.h"
#include "llvm/DebugInfo/GSYM/GsymCreator.h"
#include "llvm/DebugInfo/GSYM/GsymDataExtractor.h"
#include "llvm/DebugInfo/GSYM/GsymReader.h"
#include "llvm/DebugInfo/GSYM/InlineInfo.h"
#include "llvm/DebugInfo/GSYM/LineTable.h"
#include <optional>

using namespace llvm;
using namespace gsym;

/// FunctionInfo information type that is used to encode the optional data
/// that is associated with a FunctionInfo object.
enum InfoType : uint32_t {
  EndOfList = 0u,
  LineTableInfo = 1u,
  InlineInfo = 2u,
  MergedFunctionsInfo = 3u,
  CallSiteInfo = 4u,
};

raw_ostream &llvm::gsym::operator<<(raw_ostream &OS, const FunctionInfo &FI) {
  OS << FI.Range << ": " << "Name=" << HEX32(FI.Name) << '\n';
  if (FI.OptLineTable)
    OS << FI.OptLineTable << '\n';
  if (FI.Inline)
    OS << FI.Inline << '\n';
  if (FI.CallSites)
    OS << *FI.CallSites << '\n';
  return OS;
}
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 8 direct dependencies, including `llvm/DebugInfo/GSYM/FunctionInfo.h`, `llvm/DebugInfo/GSYM/FileWriter.h`, `llvm/DebugInfo/GSYM/GsymCreator.h`, `llvm/DebugInfo/GSYM/GsymDataExtractor.h`.
  CN: 引入了 8 个直接依赖，其中包括 `llvm/DebugInfo/GSYM/FunctionInfo.h`, `llvm/DebugInfo/GSYM/FileWriter.h`, `llvm/DebugInfo/GSYM/GsymCreator.h`, `llvm/DebugInfo/GSYM/GsymDataExtractor.h`。
- EN: This range defines or extends data types such as `InfoType`.
  CN: 这一段定义或扩展了 `InfoType` 等数据类型。

### Lines 41-80

```cpp

llvm::Expected<FunctionInfo> FunctionInfo::decode(GsymDataExtractor &Data,
                                                  uint64_t BaseAddr) {
  FunctionInfo FI;
  uint64_t Offset = 0;
  if (!Data.isValidOffsetForDataOfSize(Offset, 4))
    return createStringError(std::errc::io_error,
        "0x%8.8" PRIx64 ": missing FunctionInfo Size", Offset);
  FI.Range = {BaseAddr, BaseAddr + Data.getU32(&Offset)};
  if (!Data.isValidOffsetForDataOfSize(Offset, 4))
    return createStringError(std::errc::io_error,
        "0x%8.8" PRIx64 ": missing FunctionInfo Name", Offset);
  FI.Name = Data.getStringOffset(&Offset);
  if (FI.Name == 0)
    return createStringError(std::errc::io_error,
                             "0x%8.8" PRIx64
                             ": invalid FunctionInfo Name value 0x%" PRIx64,
                             Offset - Data.getStringOffsetSize(), FI.Name);
  bool Done = false;
  while (!Done) {
    if (!Data.isValidOffsetForDataOfSize(Offset, 4))
      return createStringError(std::errc::io_error,
          "0x%8.8" PRIx64 ": missing FunctionInfo InfoType value", Offset);
    const uint32_t IT = Data.getU32(&Offset);
    if (!Data.isValidOffsetForDataOfSize(Offset, 4))
      return createStringError(std::errc::io_error,
          "0x%8.8" PRIx64 ": missing FunctionInfo InfoType length", Offset);
    const uint32_t InfoLength = Data.getU32(&Offset);
    if (!Data.isValidOffsetForDataOfSize(Offset, InfoLength))
      return createStringError(std::errc::io_error,
          "0x%8.8" PRIx64 ": missing FunctionInfo data for InfoType %u",
          Offset, IT);
    GsymDataExtractor InfoData(Data, Offset, InfoLength);
    switch (IT) {
      case InfoType::EndOfList:
        Done = true;
        break;

      case InfoType::LineTableInfo:
        if (Expected<LineTable> LT = LineTable::decode(InfoData, BaseAddr))
```
- EN: This section centers on `decode`, `InfoData` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `decode`, `InfoData` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and iterates over collections, ranges, or records.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并遍历集合、区间或记录。

### Lines 81-120

```cpp
          FI.OptLineTable = std::move(LT.get());
        else
          return LT.takeError();
        break;

      case InfoType::InlineInfo:
        if (Expected<InlineInfo> II = InlineInfo::decode(InfoData, BaseAddr))
          FI.Inline = std::move(II.get());
        else
          return II.takeError();
        break;

      case InfoType::MergedFunctionsInfo:
        if (Expected<MergedFunctionsInfo> MI =
                MergedFunctionsInfo::decode(InfoData, BaseAddr))
          FI.MergedFunctions = std::move(MI.get());
        else
          return MI.takeError();
        break;

      case InfoType::CallSiteInfo:
        if (Expected<llvm::gsym::CallSiteInfoCollection> CI =
                llvm::gsym::CallSiteInfoCollection::decode(InfoData))
          FI.CallSites = std::move(CI.get());
        else
          return CI.takeError();
        break;

      default:
        return createStringError(std::errc::io_error,
                                 "0x%8.8" PRIx64 ": unsupported InfoType %u",
                                 Offset-8, IT);
    }
    Offset += InfoLength;
  }
  return std::move(FI);
}

uint64_t FunctionInfo::cacheEncoding(GsymCreator &GC) {
  EncodingCache.clear();
```
- EN: This section centers on `createStringError`, `move`, `cacheEncoding` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `createStringError`, `move`, `cacheEncoding` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 121-160

```cpp
  if (!isValid())
    return 0;
  raw_svector_ostream OutStrm(EncodingCache);
  FileWriter FW(OutStrm, llvm::endianness::native);
  FW.setStringOffsetSize(GC.getStringOffsetSize());
  llvm::Expected<uint64_t> Result = encode(FW);
  if (!Result) {
    EncodingCache.clear();
    consumeError(Result.takeError());
    return 0;
  }
  return EncodingCache.size();
}

llvm::Expected<uint64_t> FunctionInfo::encode(FileWriter &Out,
                                              bool NoPadding) const {
  if (!isValid())
    return createStringError(std::errc::invalid_argument,
        "attempted to encode invalid FunctionInfo object");
  // Align FunctionInfo data to a 4 byte alignment, if padding is allowed
  if (NoPadding == false)
    Out.alignTo(4);
  const uint64_t FuncInfoOffset = Out.tell();
  // Check if we have already encoded this function info into EncodingCache.
  // This will be non empty when creating segmented GSYM files as we need to
  // precompute exactly how big FunctionInfo objects encode into so we can
  // accurately make segments of a specific size.
  if (!EncodingCache.empty() &&
      llvm::endianness::native == Out.getByteOrder()) {
    // We already encoded this object, just write out the bytes.
    Out.writeData(llvm::ArrayRef<uint8_t>((const uint8_t *)EncodingCache.data(),
                                          EncodingCache.size()));
    return FuncInfoOffset;
  }
  // Write the size in bytes of this function as a uint32_t. This can be zero
  // if we just have a symbol from a symbol table and that symbol has no size.
  Out.writeU32(size());
  // Write the name of this function as a string table offset.
  Out.writeStringOffset(Name);

```
- EN: This section centers on `OutStrm`, `FW`, `consumeError` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `OutStrm`, `FW`, `consumeError` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 161-200

```cpp
  if (OptLineTable) {
    Out.writeU32(InfoType::LineTableInfo);
    // Write a uint32_t length as zero for now, we will fix this up after
    // writing the LineTable out with the number of bytes that were written.
    Out.writeU32(0);
    const auto StartOffset = Out.tell();
    llvm::Error err = OptLineTable->encode(Out, Range.start());
    if (err)
      return std::move(err);
    const auto Length = Out.tell() - StartOffset;
    if (Length > UINT32_MAX)
        return createStringError(std::errc::invalid_argument,
            "LineTable length is greater than UINT32_MAX");
    // Fixup the size of the LineTable data with the correct size.
    Out.fixup32(static_cast<uint32_t>(Length), StartOffset - 4);
  }

  // Write out the inline function info if we have any and if it is valid.
  if (Inline) {
    Out.writeU32(InfoType::InlineInfo);
    // Write a uint32_t length as zero for now, we will fix this up after
    // writing the LineTable out with the number of bytes that were written.
    Out.writeU32(0);
    const auto StartOffset = Out.tell();
    llvm::Error err = Inline->encode(Out, Range.start());
    if (err)
      return std::move(err);
    const auto Length = Out.tell() - StartOffset;
    if (Length > UINT32_MAX)
        return createStringError(std::errc::invalid_argument,
            "InlineInfo length is greater than UINT32_MAX");
    // Fixup the size of the InlineInfo data with the correct size.
    Out.fixup32(static_cast<uint32_t>(Length), StartOffset - 4);
  }

  // Write out the merged functions info if we have any and if it is valid.
  if (MergedFunctions) {
    Out.writeU32(InfoType::MergedFunctionsInfo);
    // Write a uint32_t length as zero for now, we will fix this up after
    // writing the LineTable out with the number of bytes that were written.
```
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 201-240

```cpp
    Out.writeU32(0);
    const auto StartOffset = Out.tell();
    llvm::Error err = MergedFunctions->encode(Out);
    if (err)
      return std::move(err);
    const auto Length = Out.tell() - StartOffset;
    if (Length > UINT32_MAX)
      return createStringError(
          std::errc::invalid_argument,
          "MergedFunctionsInfo length is greater than UINT32_MAX");
    // Fixup the size of the MergedFunctionsInfo data with the correct size.
    Out.fixup32(static_cast<uint32_t>(Length), StartOffset - 4);
  }

  // Write out the call sites if we have any and if they are valid.
  if (CallSites) {
    Out.writeU32(InfoType::CallSiteInfo);
    // Write a uint32_t length as zero for now, we will fix this up after
    // writing the CallSites out with the number of bytes that were written.
    Out.writeU32(0);
    const auto StartOffset = Out.tell();
    Error Err = CallSites->encode(Out);
    if (Err)
      return std::move(Err);
    const auto Length = Out.tell() - StartOffset;
    if (Length > UINT32_MAX)
      return createStringError(std::errc::invalid_argument,
                               "CallSites length is greater than UINT32_MAX");
    // Fixup the size of the CallSites data with the correct size.
    Out.fixup32(static_cast<uint32_t>(Length), StartOffset - 4);
  }

  // Terminate the data chunks with an end of list with zero size.
  Out.writeU32(InfoType::EndOfList);
  Out.writeU32(0);
  return FuncInfoOffset;
}

llvm::Expected<LookupResult>
FunctionInfo::lookup(GsymDataExtractor &Data, const GsymReader &GR,
```
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 241-280

```cpp
                     uint64_t FuncAddr, uint64_t Addr,
                     std::optional<GsymDataExtractor> *MergedFuncsData) {
  LookupResult LR;
  LR.LookupAddr = Addr;
  uint64_t Offset = 0;
  LR.FuncRange = {FuncAddr, FuncAddr + Data.getU32(&Offset)};
  gsym_strp_t NameOffset = Data.getStringOffset(&Offset);
  // The "lookup" functions doesn't report errors as accurately as the "decode"
  // function as it is meant to be fast. For more accurage errors we could call
  // "decode".
  if (!Data.isValidOffset(Offset))
    return createStringError(std::errc::io_error,
                              "FunctionInfo data is truncated");
  // This function will be called with the result of a binary search of the
  // address table, we must still make sure the address does not fall into a
  // gap between functions are after the last function.
  if (LR.FuncRange.size() > 0 && !LR.FuncRange.contains(Addr))
    return createStringError(std::errc::io_error,
        "address 0x%" PRIx64 " is not in GSYM", Addr);

  if (NameOffset == 0)
    return createStringError(std::errc::io_error,
                             "0x%8.8" PRIx64
                             ": invalid FunctionInfo Name value 0x0",
                             Offset - Data.getStringOffsetSize());
  LR.FuncName = GR.getString(NameOffset);
  bool Done = false;
  std::optional<LineEntry> LineEntry;
  std::optional<GsymDataExtractor> InlineInfoData;
  while (!Done) {
    if (!Data.isValidOffsetForDataOfSize(Offset, 8))
      return createStringError(std::errc::io_error,
                               "FunctionInfo data is truncated");
    const uint32_t IT = Data.getU32(&Offset);
    const uint32_t InfoLength = Data.getU32(&Offset);
    const StringRef InfoBytes = Data.getData().substr(Offset, InfoLength);
    if (InfoLength != InfoBytes.size())
      return createStringError(std::errc::io_error,
                               "FunctionInfo data is truncated");
    GsymDataExtractor InfoData(Data, Offset, InfoLength);
```
- EN: This section centers on `InfoData` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `InfoData` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 281-320

```cpp
    switch (IT) {
      case InfoType::EndOfList:
        Done = true;
        break;

      case InfoType::LineTableInfo:
        if (auto ExpectedLE = LineTable::lookup(InfoData, FuncAddr, Addr))
          LineEntry = ExpectedLE.get();
        else
          return ExpectedLE.takeError();
        break;

      case InfoType::MergedFunctionsInfo:
        // Store the merged functions data for later parsing, if needed.
        if (MergedFuncsData)
          *MergedFuncsData = InfoData;
        break;

      case InfoType::InlineInfo:
        // We will parse the inline info after our line table, but only if
        // we have a line entry.
        InlineInfoData = InfoData;
        break;

      case InfoType::CallSiteInfo:
        if (auto CSIC = CallSiteInfoCollection::decode(InfoData)) {
          // Find matching call site based on relative offset
          for (const auto &CS : CSIC->CallSites) {
            // Check if the call site matches the lookup address
            if (CS.ReturnOffset == Addr - FuncAddr) {
              // Get regex patterns
              for (gsym_strp_t RegexOffset : CS.MatchRegex) {
                LR.CallSiteFuncRegex.push_back(GR.getString(RegexOffset));
              }
              break;
            }
          }
        } else {
          return CSIC.takeError();
        }
```
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and iterates over collections, ranges, or records.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并遍历集合、区间或记录。

### Lines 321-360

```cpp
        break;

      default:
        break;
    }
    Offset += InfoLength;
  }

  if (!LineEntry) {
    // We don't have a valid line entry for our address, fill in our source
    // location as best we can and return.
    SourceLocation SrcLoc;
    SrcLoc.Name = LR.FuncName;
    SrcLoc.Offset = Addr - FuncAddr;
    LR.Locations.push_back(SrcLoc);
    return LR;
  }

  std::optional<FileEntry> LineEntryFile = GR.getFile(LineEntry->File);
  if (!LineEntryFile)
    return createStringError(std::errc::invalid_argument,
                              "failed to extract file[%" PRIu32 "]",
                              LineEntry->File);

  SourceLocation SrcLoc;
  SrcLoc.Name = LR.FuncName;
  SrcLoc.Offset = Addr - FuncAddr;
  SrcLoc.Dir = GR.getString(LineEntryFile->Dir);
  SrcLoc.Base = GR.getString(LineEntryFile->Base);
  SrcLoc.Line = LineEntry->Line;
  LR.Locations.push_back(SrcLoc);
  // If we don't have inline information, we are done.
  if (!InlineInfoData)
    return LR;
  // We have inline information. Try to augment the lookup result with this
  // data.
  llvm::Error Err = InlineInfo::lookup(GR, *InlineInfoData, FuncAddr, Addr,
                                       LR.Locations);
  if (Err)
    return std::move(Err);
```
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 361-362

```cpp
  return LR;
}
```
- EN: This range returns the resulting value to its callers.
  CN: 这一段将结果返回给调用方。

## Key Concepts / 关键概念
- Domain / 领域: debug information infrastructure / 调试信息基础设施
- Core symbols / 核心符号: `InfoType`, `decode`, `InfoData`, `createStringError`, `move` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Parsing, decoding, and structural validation. / 重点关注解析、解码与结构校验。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/DebugInfo/GSYM/FunctionInfo.h`, `llvm/DebugInfo/GSYM/FileWriter.h`, `llvm/DebugInfo/GSYM/GsymCreator.h`, `llvm/DebugInfo/GSYM/GsymDataExtractor.h`, `llvm/DebugInfo/GSYM/GsymReader.h`, `llvm/DebugInfo/GSYM/InlineInfo.h`, `llvm/DebugInfo/GSYM/LineTable.h`
- Standard library / 标准库: `optional`
- Other/system headers / 其他或系统头文件: None / 无
- Related symbols / 相关符号: `InfoType`, `decode`, `InfoData`, `createStringError`, `move`, `cacheEncoding`
