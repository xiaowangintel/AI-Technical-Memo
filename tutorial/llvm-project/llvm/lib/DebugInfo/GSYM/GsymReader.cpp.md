# GsymReader.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/DebugInfo/GSYM/GsymReader.cpp`
- Repository: `llvm-project`
- Purpose (EN): / Check magic bytes, determine endianness, and return the GSYM version and / endianness.
- Purpose (CN): 该文件位于 LLVM 的 `DebugInfo/GSYM` 目录中，主要实现与 `GsymReader` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-60

```cpp
//===- GsymReader.cpp -----------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/DebugInfo/GSYM/GsymReader.h"

#include <assert.h>
#include <inttypes.h>
#include <stdio.h>
#include <stdlib.h>

#include "llvm/DebugInfo/GSYM/GsymReaderV1.h"
#include "llvm/DebugInfo/GSYM/GsymReaderV2.h"
#include "llvm/DebugInfo/GSYM/Header.h"
#include "llvm/DebugInfo/GSYM/HeaderV2.h"
#include "llvm/DebugInfo/GSYM/InlineInfo.h"
#include "llvm/DebugInfo/GSYM/LineTable.h"
#include "llvm/Support/MemoryBuffer.h"

using namespace llvm;
using namespace gsym;

GsymReader::GsymReader(std::unique_ptr<MemoryBuffer> Buffer,
                       llvm::endianness Endian)
    : MemBuffer(std::move(Buffer)), Endian(Endian),
      AddrInfoOffsetsData(StringRef(), true), FileEntryData(StringRef(), true) {
}

/// Check magic bytes, determine endianness, and return the GSYM version and
/// endianness. If magic bytes are invalid, return error.
static Expected<std::pair<uint16_t, llvm::endianness>>
checkMagicAndDetectVersionEndian(StringRef Bytes) {
  if (Bytes.size() < 6)
    return createStringError(std::errc::invalid_argument,
                             "data too small to be a GSYM file");
  // Detect host endian
  const auto HostEndian = llvm::endianness::native;
  const bool IsHostLittleEndian = (HostEndian == llvm::endianness::little);
  // Read magic bytes using host endian
  GsymDataExtractor Data(Bytes, IsHostLittleEndian);
  uint64_t Offset = 0;
  uint32_t Magic = Data.getU32(&Offset);
  llvm::endianness FileEndian;
  // If magic bytes looks alright, the host and the file have the same
  // endianness, vice versa.
  if (Magic == GSYM_MAGIC) {
    FileEndian = HostEndian;
  } else if (Magic == GSYM_CIGAM) {
    FileEndian =
        IsHostLittleEndian ? llvm::endianness::big : llvm::endianness::little;
    // Re-create GsymDataExtractor with correct endianness to read version.
    Data = GsymDataExtractor(Bytes, !IsHostLittleEndian);
  } else {
    return createStringError(std::errc::invalid_argument,
                             "not a GSYM file (bad magic)");
  }
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 12 direct dependencies, including `llvm/DebugInfo/GSYM/GsymReader.h`, `assert.h`, `inttypes.h`, `stdio.h`.
  CN: 引入了 12 个直接依赖，其中包括 `llvm/DebugInfo/GSYM/GsymReader.h`, `assert.h`, `inttypes.h`, `stdio.h`。
- EN: This section centers on `GsymReader`, `checkMagicAndDetectVersionEndian`, `Data` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `GsymReader`, `checkMagicAndDetectVersionEndian`, `Data` 等符号展开，负责解析输入并把原始数据转换成结构化形式。

### Lines 61-120

```cpp
  // Read version using the correct endian
  uint16_t Version = Data.getU16(&Offset);
  return std::make_pair(Version, FileEndian);
}

llvm::Expected<std::unique_ptr<GsymReader>>
GsymReader::openFile(StringRef Filename) {
  // Open the input file and return an appropriate error if needed.
  ErrorOr<std::unique_ptr<MemoryBuffer>> BuffOrErr =
      MemoryBuffer::getFileOrSTDIN(Filename);
  auto Err = BuffOrErr.getError();
  if (Err)
    return llvm::errorCodeToError(Err);
  return create(BuffOrErr.get());
}

llvm::Expected<std::unique_ptr<GsymReader>>
GsymReader::copyBuffer(StringRef Bytes) {
  auto MemBuffer = MemoryBuffer::getMemBufferCopy(Bytes, "GSYM bytes");
  return create(MemBuffer);
}

llvm::Expected<std::unique_ptr<GsymReader>>
GsymReader::create(std::unique_ptr<MemoryBuffer> &MemBuffer) {
  if (!MemBuffer)
    return createStringError(std::errc::invalid_argument,
                             "invalid memory buffer");
  Expected<std::pair<uint16_t, llvm::endianness>> VersionEndianOrErr =
      checkMagicAndDetectVersionEndian(MemBuffer->getBuffer());
  if (!VersionEndianOrErr)
    return VersionEndianOrErr.takeError();
  uint16_t Version;
  llvm::endianness Endian;
  std::tie(Version, Endian) = *VersionEndianOrErr;
  std::unique_ptr<GsymReader> GR;
  switch (Version) {
  case Header::getVersion():
    GR.reset(new GsymReaderV1(std::move(MemBuffer), Endian));
    break;
  case HeaderV2::getVersion():
    GR.reset(new GsymReaderV2(std::move(MemBuffer), Endian));
    break;
  default:
    return createStringError(std::errc::invalid_argument,
                             "unsupported GSYM version %u", Version);
  }
  if (auto Err = GR->parse())
    return std::move(Err);
  return std::move(GR);
}

llvm::Error GsymReader::parse() {
  // Step 1: Parse the version-specific header and populate GlobalDataSections.
  if (auto Err = parseHeaderAndGlobalDataEntries())
    return Err;

  // Step 2: Validate that all required sections are present and consistent.
  for (auto Type :
       {GlobalInfoType::AddrOffsets, GlobalInfoType::AddrInfoOffsets,
        GlobalInfoType::StringTable, GlobalInfoType::FileTable,
```
- EN: This section centers on `make_pair`, `openFile`, `getFileOrSTDIN` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `make_pair`, `openFile`, `getFileOrSTDIN` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and iterates over collections, ranges, or records.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并遍历集合、区间或记录。

### Lines 121-180

```cpp
        GlobalInfoType::FunctionInfo})
    if (!GlobalDataSections.count(Type))
      return createStringError(
          std::errc::invalid_argument, "missing required section type %s (%u)",
          getNameForGlobalInfoType(Type).data(), static_cast<uint32_t>(Type));

  if (GlobalDataSections[GlobalInfoType::AddrOffsets].FileSize !=
      static_cast<uint64_t>(getNumAddresses()) * getAddressOffsetSize())
    return createStringError(std::errc::invalid_argument,
                             "AddrOffsets section size mismatch");

  if (GlobalDataSections[GlobalInfoType::AddrInfoOffsets].FileSize !=
      static_cast<uint64_t>(getNumAddresses()) * getAddressInfoOffsetSize())
    return createStringError(std::errc::invalid_argument,
                             "AddrInfoOffsets section size mismatch");

  // Step 3: Parse each global data section.
  llvm::Expected<StringRef> Bytes =
      getRequiredGlobalDataBytes(GlobalInfoType::AddrOffsets);
  if (!Bytes)
    return Bytes.takeError();
  if (auto Err = parseAddrOffsets(*Bytes))
    return Err;

  Bytes = getRequiredGlobalDataBytes(GlobalInfoType::AddrInfoOffsets);
  if (!Bytes)
    return Bytes.takeError();
  if (auto Err = setAddrInfoOffsetsData(*Bytes))
    return Err;

  Bytes = getRequiredGlobalDataBytes(GlobalInfoType::StringTable);
  if (!Bytes)
    return Bytes.takeError();
  if (auto Err = setStringTableData(*Bytes))
    return Err;

  Bytes = getRequiredGlobalDataBytes(GlobalInfoType::FileTable);
  if (!Bytes)
    return Bytes.takeError();
  if (auto Err = setFileTableData(*Bytes))
    return Err;

  return Error::success();
}

llvm::Error GsymReader::parseGlobalDataEntries(uint64_t Offset) {
  if (getVersion() < HeaderV2::getVersion())
    return createStringError(std::errc::invalid_argument,
                             "GlobalData section not supported in GSYM V1");

  const StringRef Buf = MemBuffer->getBuffer();
  const uint64_t BufSize = Buf.size();
  GsymDataExtractor Data(Buf, isLittleEndian());
  while (Offset + sizeof(GlobalData) <= BufSize) {
    auto GDOrErr = GlobalData::decode(Data, Offset);
    if (!GDOrErr)
      return GDOrErr.takeError();
    const GlobalData &GD = *GDOrErr;

    if (GD.Type == GlobalInfoType::EndOfList)
```
- EN: This section centers on `getRequiredGlobalDataBytes`, `success`, `parseGlobalDataEntries` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `getRequiredGlobalDataBytes`, `success`, `parseGlobalDataEntries` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 181-240

```cpp
      return Error::success();

    if (GD.FileSize == 0)
      return createStringError(std::errc::invalid_argument,
                               "GlobalData section type %u has zero size",
                               static_cast<uint32_t>(GD.Type));

    if (GD.FileOffset + GD.FileSize > BufSize)
      return createStringError(
          std::errc::invalid_argument,
          "GlobalData section type %u extends beyond "
          "buffer (offset=%" PRIu64 ", size=%" PRIu64 ", bufsize=%" PRIu64 ")",
          static_cast<uint32_t>(GD.Type), GD.FileOffset, GD.FileSize, BufSize);

    GlobalDataSections[GD.Type] = GD;
  }
  return createStringError(std::errc::invalid_argument,
                           "GlobalData array not terminated by EndOfList");
}

llvm::Error GsymReader::parseAddrOffsets(StringRef Bytes) {
  const uint8_t AddrOffSize = getAddressOffsetSize();
  const uint32_t NumAddrs = getNumAddresses();
  const size_t TotalBytes = NumAddrs * AddrOffSize;
  if (Bytes.size() < TotalBytes)
    return createStringError(std::errc::invalid_argument,
                             "failed to read address table");

  // Parse the non-swap case
  if (Endian == llvm::endianness::native) {
    AddrOffsets = ArrayRef<uint8_t>(
        reinterpret_cast<const uint8_t *>(Bytes.data()), TotalBytes);
    return Error::success();
  }

  // Parse the swap case
  GsymDataExtractor Data(Bytes, isLittleEndian());
  uint64_t Offset = 0;
  SwappedAddrOffsets.resize(TotalBytes);
  switch (AddrOffSize) {
  case 1:
    if (!Data.getU8(&Offset, SwappedAddrOffsets.data(), NumAddrs))
      return createStringError(std::errc::invalid_argument,
                               "failed to read address table");
    break;
  case 2:
    if (!Data.getU16(&Offset,
                     reinterpret_cast<uint16_t *>(SwappedAddrOffsets.data()),
                     NumAddrs))
      return createStringError(std::errc::invalid_argument,
                               "failed to read address table");
    break;
  case 4:
    if (!Data.getU32(&Offset,
                     reinterpret_cast<uint32_t *>(SwappedAddrOffsets.data()),
                     NumAddrs))
      return createStringError(std::errc::invalid_argument,
                               "failed to read address table");
    break;
  case 8:
```
- EN: This section centers on `success`, `createStringError`, `parseAddrOffsets` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `success`, `createStringError`, `parseAddrOffsets` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and checks conditions and handles edge cases.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并检查条件并处理边界情况。

### Lines 241-300

```cpp
    if (!Data.getU64(&Offset,
                     reinterpret_cast<uint64_t *>(SwappedAddrOffsets.data()),
                     NumAddrs))
      return createStringError(std::errc::invalid_argument,
                               "failed to read address table");
    break;
  }
  AddrOffsets = ArrayRef<uint8_t>(SwappedAddrOffsets);
  return Error::success();
}

llvm::Error GsymReader::setAddrInfoOffsetsData(StringRef Bytes) {
  AddrInfoOffsetsData = GsymDataExtractor(Bytes, isLittleEndian());
  return Error::success();
}

llvm::Error GsymReader::setStringTableData(StringRef Bytes) {
  StrTab.Data = Bytes;
  return Error::success();
}

llvm::Error GsymReader::setFileTableData(StringRef Bytes) {
  const uint8_t StrpSize = getStringOffsetSize();
  GsymDataExtractor Data(Bytes, isLittleEndian(), StrpSize);
  uint64_t Offset = 0;
  uint32_t NumFiles = Data.getU32(&Offset);
  uint64_t EntriesSize =
      static_cast<uint64_t>(NumFiles) * FileEntry::getEncodedSize(StrpSize);
  if (Bytes.size() < Offset + EntriesSize)
    return createStringError(std::errc::invalid_argument,
                             "FileTable section too small for %u files",
                             NumFiles);
  FileEntryData = GsymDataExtractor(Data, Offset, EntriesSize);
  return Error::success();
}

std::optional<GlobalData> GsymReader::getGlobalData(GlobalInfoType Type) const {
  auto It = GlobalDataSections.find(Type);
  if (It == GlobalDataSections.end())
    return std::nullopt;
  return It->second;
}

llvm::Expected<StringRef>
GsymReader::getRequiredGlobalDataBytes(GlobalInfoType Type) const {
  if (auto Data = getOptionalGlobalDataBytes(Type))
    return *Data;
  const char *TypeName = getNameForGlobalInfoType(Type).data();
  std::optional<GlobalData> GD = getGlobalData(Type);
  // We have a GlobalData entry but didn't get any bytes — the file may be
  // truncated.
  if (GD)
    return createStringError(
        std::errc::invalid_argument,
        "missing bytes for %s, GSYM file might be truncated", TypeName);
  return createStringError(std::errc::invalid_argument,
                           "missing required section type %s", TypeName);
}

std::optional<StringRef>
```
- EN: This section centers on `success`, `setAddrInfoOffsetsData`, `setStringTableData` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `success`, `setAddrInfoOffsetsData`, `setStringTableData` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 301-360

```cpp
GsymReader::getOptionalGlobalDataBytes(GlobalInfoType Type) const {
  std::optional<GlobalData> GD = getGlobalData(Type);
  if (!GD)
    return std::nullopt;
  StringRef Buf = MemBuffer->getBuffer();
  if (GD->FileSize == 0 || GD->FileOffset + GD->FileSize > Buf.size())
    return std::nullopt;
  return Buf.substr(GD->FileOffset, GD->FileSize);
}

std::optional<uint64_t> GsymReader::getAddress(size_t Index) const {
  switch (getAddressOffsetSize()) {
  case 1: return addressForIndex<uint8_t>(Index);
  case 2: return addressForIndex<uint16_t>(Index);
  case 4: return addressForIndex<uint32_t>(Index);
  case 8: return addressForIndex<uint64_t>(Index);
  default:
    llvm_unreachable("unsupported address offset size");
  }
  return std::nullopt;
}

std::optional<uint64_t> GsymReader::getAddressInfoOffset(size_t Index) const {
  if (Index >= getNumAddresses())
    return std::nullopt;
  const uint8_t AddrInfoOffsetSize = getAddressInfoOffsetSize();
  uint64_t Offset = Index * AddrInfoOffsetSize;
  uint64_t AddrInfoOffset =
      AddrInfoOffsetsData.getUnsigned(&Offset, AddrInfoOffsetSize);
  // V1 stores absolute file offsets in AddrInfoOffsets, so no base offset is
  // needed. V2+ stores offsets relative to the FunctionInfo section start.
  if (getVersion() != Header::getVersion())
    AddrInfoOffset +=
        GlobalDataSections.at(GlobalInfoType::FunctionInfo).FileOffset;
  return AddrInfoOffset;
}

Expected<uint64_t> GsymReader::getAddressIndex(const uint64_t Addr) const {
  const uint64_t BaseAddr = getBaseAddress();
  if (Addr >= BaseAddr) {
    const uint64_t AddrOffset = Addr - BaseAddr;
    std::optional<uint64_t> AddrOffsetIndex;
    switch (getAddressOffsetSize()) {
    case 1:
      AddrOffsetIndex = getAddressOffsetIndex<uint8_t>(AddrOffset);
      break;
    case 2:
      AddrOffsetIndex = getAddressOffsetIndex<uint16_t>(AddrOffset);
      break;
    case 4:
      AddrOffsetIndex = getAddressOffsetIndex<uint32_t>(AddrOffset);
      break;
    case 8:
      AddrOffsetIndex = getAddressOffsetIndex<uint64_t>(AddrOffset);
      break;
    default:
      return createStringError(std::errc::invalid_argument,
                               "unsupported address offset size %u",
                               getAddressOffsetSize());
    }
```
- EN: This section centers on `getOptionalGlobalDataBytes`, `getAddress`, `llvm_unreachable` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `getOptionalGlobalDataBytes`, `getAddress`, `llvm_unreachable` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and checks conditions and handles edge cases.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并检查条件并处理边界情况。

### Lines 361-420

```cpp
    if (AddrOffsetIndex)
      return *AddrOffsetIndex;
  }
  return createStringError(std::errc::invalid_argument,
                           "address 0x%" PRIx64 " is not in GSYM", Addr);
}

llvm::Expected<GsymDataExtractor>
GsymReader::getFunctionInfoDataForAddress(uint64_t Addr,
                                          uint64_t &FuncStartAddr) const {
  Expected<uint64_t> ExpectedAddrIdx = getAddressIndex(Addr);
  if (!ExpectedAddrIdx)
    return ExpectedAddrIdx.takeError();
  const uint64_t FirstAddrIdx = *ExpectedAddrIdx;
  // The AddrIdx is the first index of the function info entries that match
  // \a Addr. We need to iterate over all function info objects that start with
  // the same address until we find a range that contains \a Addr.
  std::optional<uint64_t> FirstFuncStartAddr;
  const size_t NumAddresses = getNumAddresses();
  for (uint64_t AddrIdx = FirstAddrIdx; AddrIdx < NumAddresses; ++AddrIdx) {
    auto ExpextedData = getFunctionInfoDataAtIndex(AddrIdx, FuncStartAddr);
    // If there was an error, return the error.
    if (!ExpextedData)
      return ExpextedData;

    // Remember the first function start address if it hasn't already been set.
    // If it is already valid, check to see if it matches the first function
    // start address and only continue if it matches.
    if (FirstFuncStartAddr.has_value()) {
      if (*FirstFuncStartAddr != FuncStartAddr)
        break; // Done with consecutive function entries with same address.
    } else {
      FirstFuncStartAddr = FuncStartAddr;
    }
    // Make sure the current function address ranges contains \a Addr.
    // Some symbols on Darwin don't have valid sizes, so if we run into a
    // symbol with zero size, then we have found a match for our address.

    // The first thing the encoding of a FunctionInfo object is the function
    // size.
    uint64_t Offset = 0;
    uint32_t FuncSize = ExpextedData->getU32(&Offset);
    if (FuncSize == 0 ||
        AddressRange(FuncStartAddr, FuncStartAddr + FuncSize).contains(Addr))
      return ExpextedData;
  }
  return createStringError(std::errc::invalid_argument,
                           "address 0x%" PRIx64 " is not in GSYM", Addr);
}

llvm::Expected<GsymDataExtractor>
GsymReader::getFunctionInfoDataAtIndex(uint64_t AddrIdx,
                                       uint64_t &FuncStartAddr) const {
  const std::optional<uint64_t> AddrInfoOffset = getAddressInfoOffset(AddrIdx);
  if (AddrInfoOffset == std::nullopt)
    return createStringError(std::errc::invalid_argument,
                             "invalid address index %" PRIu64, AddrIdx);
  assert((Endian == endianness::big || Endian == endianness::little) &&
         "Endian must be either big or little");
  StringRef Bytes = MemBuffer->getBuffer().substr(*AddrInfoOffset);
```
- EN: This section centers on `createStringError`, `getFunctionInfoDataForAddress`, `getFunctionInfoDataAtIndex` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `createStringError`, `getFunctionInfoDataForAddress`, `getFunctionInfoDataAtIndex` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 421-480

```cpp
  if (Bytes.empty())
    return createStringError(std::errc::invalid_argument,
                             "invalid address info offset 0x%" PRIx64,
                             *AddrInfoOffset);
  std::optional<uint64_t> OptFuncStartAddr = getAddress(AddrIdx);
  if (!OptFuncStartAddr)
    return createStringError(std::errc::invalid_argument,
                             "failed to extract address[%" PRIu64 "]", AddrIdx);
  FuncStartAddr = *OptFuncStartAddr;
  GsymDataExtractor Data(Bytes, isLittleEndian(), getStringOffsetSize());
  return Data;
}

llvm::Expected<FunctionInfo> GsymReader::getFunctionInfo(uint64_t Addr) const {
  uint64_t FuncStartAddr = 0;
  if (auto ExpectedData = getFunctionInfoDataForAddress(Addr, FuncStartAddr))
    return FunctionInfo::decode(*ExpectedData, FuncStartAddr);
  else
    return ExpectedData.takeError();
}

llvm::Expected<FunctionInfo>
GsymReader::getFunctionInfoAtIndex(uint64_t Idx) const {
  uint64_t FuncStartAddr = 0;
  if (auto ExpectedData = getFunctionInfoDataAtIndex(Idx, FuncStartAddr))
    return FunctionInfo::decode(*ExpectedData, FuncStartAddr);
  else
    return ExpectedData.takeError();
}

llvm::Expected<LookupResult> GsymReader::lookup(
    uint64_t Addr,
    std::optional<GsymDataExtractor> *MergedFunctionsData) const {
  uint64_t FuncStartAddr = 0;
  if (auto ExpectedData = getFunctionInfoDataForAddress(Addr, FuncStartAddr))
    return FunctionInfo::lookup(*ExpectedData, *this, FuncStartAddr, Addr,
                                MergedFunctionsData);
  else
    return ExpectedData.takeError();
}

llvm::Expected<std::vector<LookupResult>>
GsymReader::lookupAll(uint64_t Addr) const {
  std::vector<LookupResult> Results;
  std::optional<GsymDataExtractor> MergedFunctionsData;

  // First perform a lookup to get the primary function info result.
  auto MainResult = lookup(Addr, &MergedFunctionsData);
  if (!MainResult)
    return MainResult.takeError();

  // Add the main result as the first entry.
  Results.push_back(std::move(*MainResult));

  // Now process any merged functions data that was found during the lookup.
  if (MergedFunctionsData) {
    // Get data extractors for each merged function.
    auto ExpectedMergedFuncExtractors =
        MergedFunctionsInfo::getFuncsDataExtractors(*MergedFunctionsData);
    if (!ExpectedMergedFuncExtractors)
```
- EN: This section centers on `Data`, `getFunctionInfo`, `getFunctionInfoAtIndex` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `Data`, `getFunctionInfo`, `getFunctionInfoAtIndex` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 481-540

```cpp
      return ExpectedMergedFuncExtractors.takeError();

    // Process each merged function data.
    for (GsymDataExtractor &MergedData : *ExpectedMergedFuncExtractors) {
      if (auto FI = FunctionInfo::lookup(MergedData, *this,
                                         MainResult->FuncRange.start(), Addr)) {
        Results.push_back(std::move(*FI));
      } else {
        return FI.takeError();
      }
    }
  }

  return Results;
}

void GsymReader::dump(raw_ostream &OS, const FunctionInfo &FI,
                      uint32_t Indent) {
  OS.indent(Indent);
  OS << FI.Range << " \"" << getString(FI.Name) << "\"\n";
  if (FI.OptLineTable)
    dump(OS, *FI.OptLineTable, Indent);
  if (FI.Inline)
    dump(OS, *FI.Inline, Indent);

  if (FI.CallSites)
    dump(OS, *FI.CallSites, Indent);

  if (FI.MergedFunctions) {
    assert(Indent == 0 && "MergedFunctionsInfo should only exist at top level");
    dump(OS, *FI.MergedFunctions);
  }
}

void GsymReader::dump(raw_ostream &OS, const MergedFunctionsInfo &MFI) {
  for (uint32_t inx = 0; inx < MFI.MergedFunctions.size(); inx++) {
    OS << "++ Merged FunctionInfos[" << inx << "]:\n";
    dump(OS, MFI.MergedFunctions[inx], 4);
  }
}

void GsymReader::dump(raw_ostream &OS, const CallSiteInfo &CSI) {
  OS << HEX16(CSI.ReturnOffset);

  std::string Flags;
  auto addFlag = [&](const char *Flag) {
    if (!Flags.empty())
      Flags += " | ";
    Flags += Flag;
  };

  if (CSI.Flags == CallSiteInfo::Flags::None)
    Flags = "None";
  else {
    if (CSI.Flags & CallSiteInfo::Flags::InternalCall)
      addFlag("InternalCall");

    if (CSI.Flags & CallSiteInfo::Flags::ExternalCall)
      addFlag("ExternalCall");
  }
```
- EN: This section centers on `dump`, `assert`, `HEX16` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `dump`, `assert`, `HEX16` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 541-600

```cpp
  OS << " Flags[" << Flags << "]";

  if (!CSI.MatchRegex.empty()) {
    OS << " MatchRegex[";
    for (uint32_t i = 0; i < CSI.MatchRegex.size(); ++i) {
      if (i > 0)
        OS << ";";
      OS << getString(CSI.MatchRegex[i]);
    }
    OS << "]";
  }
}

void GsymReader::dump(raw_ostream &OS, const CallSiteInfoCollection &CSIC,
                      uint32_t Indent) {
  OS.indent(Indent);
  OS << "CallSites (by relative return offset):\n";
  for (const auto &CS : CSIC.CallSites) {
    OS.indent(Indent);
    OS << "  ";
    dump(OS, CS);
    OS << "\n";
  }
}

void GsymReader::dump(raw_ostream &OS, const LineTable &LT, uint32_t Indent) {
  OS.indent(Indent);
  OS << "LineTable:\n";
  for (auto &LE : LT) {
    OS.indent(Indent);
    OS << "  " << HEX64(LE.Addr) << ' ';
    if (LE.File)
      dump(OS, getFile(LE.File));
    OS << ':' << LE.Line << '\n';
  }
}

void GsymReader::dump(raw_ostream &OS, const InlineInfo &II, uint32_t Indent) {
  if (Indent == 0)
    OS << "InlineInfo:\n";
  else
    OS.indent(Indent);
  OS << II.Ranges << ' ' << getString(II.Name);
  if (II.CallFile != 0) {
    if (auto File = getFile(II.CallFile)) {
      OS << " called from ";
      dump(OS, File);
      OS << ':' << II.CallLine;
    }
  }
  OS << '\n';
  for (const auto &ChildII : II.Children)
    dump(OS, ChildII, Indent + 2);
}

void GsymReader::dump(raw_ostream &OS, std::optional<FileEntry> FE) {
  if (FE) {
    // IF we have the file from index 0, then don't print anything
    if (FE->Dir == 0 && FE->Base == 0)
      return;
```
- EN: This section centers on `getString`, `dump` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `getString`, `dump` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 601-617

```cpp
    StringRef Dir = getString(FE->Dir);
    StringRef Base = getString(FE->Base);
    if (!Dir.empty()) {
      OS << Dir;
      if (Dir.contains('\\') && !Dir.contains('/'))
        OS << '\\';
      else
        OS << '/';
    }
    if (!Base.empty()) {
      OS << Base;
    }
    if (!Dir.empty() || !Base.empty())
      return;
  }
  OS << "<invalid-file>";
}
```
- EN: In this range, the code checks conditions and handles edge cases and returns the resulting value to its callers.
  CN: 在这一段中，代码检查条件并处理边界情况，并将结果返回给调用方。

## Key Concepts / 关键概念
- Domain / 领域: debug information infrastructure / 调试信息基础设施
- Core symbols / 核心符号: `GsymReader`, `checkMagicAndDetectVersionEndian`, `Data`, `createStringError` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Parsing, decoding, and structural validation. / 重点关注解析、解码与结构校验。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/DebugInfo/GSYM/GsymReader.h`, `llvm/DebugInfo/GSYM/GsymReaderV1.h`, `llvm/DebugInfo/GSYM/GsymReaderV2.h`, `llvm/DebugInfo/GSYM/Header.h`, `llvm/DebugInfo/GSYM/HeaderV2.h`, `llvm/DebugInfo/GSYM/InlineInfo.h`, `llvm/DebugInfo/GSYM/LineTable.h`, `llvm/Support/MemoryBuffer.h`
- Standard library / 标准库: `assert.h`, `stdio.h`, `stdlib.h`
- Other/system headers / 其他或系统头文件: `inttypes.h`
- Related symbols / 相关符号: `GsymReader`, `checkMagicAndDetectVersionEndian`, `Data`, `createStringError`, `make_pair`
