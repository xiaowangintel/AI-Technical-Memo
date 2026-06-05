# GlobalsStream.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/DebugInfo/PDB/Native/GlobalsStream.cpp`
- Repository: `llvm-project`
- Purpose (EN): The on-disk structores used in this file are based on the reference implementation which is available at https://github.
- Purpose (CN): 该文件位于 LLVM 的 `DebugInfo/PDB/Native` 目录中，主要实现与 `GlobalsStream` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===- GlobalsStream.cpp - PDB Index of Symbols by Name ---------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// The on-disk structores used in this file are based on the reference
// implementation which is available at
// https://github.com/Microsoft/microsoft-pdb/blob/master/PDB/dbi/gsi.h
//
// When you are reading the reference source code, you'd find the
// information below useful.
//
//  - ppdb1->m_fMinimalDbgInfo seems to be always true.
//  - SMALLBUCKETS macro is defined.
//
//===----------------------------------------------------------------------===//

#include "llvm/DebugInfo/PDB/Native/GlobalsStream.h"

#include "llvm/DebugInfo/CodeView/RecordName.h"
#include "llvm/DebugInfo/MSF/MappedBlockStream.h"
#include "llvm/DebugInfo/PDB/Native/Hash.h"
#include "llvm/DebugInfo/PDB/Native/RawError.h"
#include "llvm/DebugInfo/PDB/Native/SymbolStream.h"
#include "llvm/Support/BinaryStreamReader.h"
#include "llvm/Support/Error.h"

using namespace llvm;
using namespace llvm::msf;
using namespace llvm::pdb;

GlobalsStream::GlobalsStream(std::unique_ptr<MappedBlockStream> Stream)
    : Stream(std::move(Stream)) {}

GlobalsStream::~GlobalsStream() = default;

Error GlobalsStream::reload() {
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 8 direct dependencies, including `llvm/DebugInfo/PDB/Native/GlobalsStream.h`, `llvm/DebugInfo/CodeView/RecordName.h`, `llvm/DebugInfo/MSF/MappedBlockStream.h`, `llvm/DebugInfo/PDB/Native/Hash.h`.
  CN: 引入了 8 个直接依赖，其中包括 `llvm/DebugInfo/PDB/Native/GlobalsStream.h`, `llvm/DebugInfo/CodeView/RecordName.h`, `llvm/DebugInfo/MSF/MappedBlockStream.h`, `llvm/DebugInfo/PDB/Native/Hash.h`。
- EN: This section centers on `GlobalsStream`, `reload` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `GlobalsStream`, `reload` 等符号展开，负责实现局部控制流程与状态维护。

### Lines 41-80

```cpp
  BinaryStreamReader Reader(*Stream);
  if (auto E = GlobalsTable.read(Reader))
    return E;
  return Error::success();
}

std::vector<std::pair<uint32_t, codeview::CVSymbol>>
GlobalsStream::findRecordsByName(StringRef Name,
                                 const SymbolStream &Symbols) const {
  std::vector<std::pair<uint32_t, codeview::CVSymbol>> Result;

  // Hash the name to figure out which bucket this goes into.
  size_t ExpandedBucketIndex = hashStringV1(Name) % IPHR_HASH;
  int32_t CompressedBucketIndex = GlobalsTable.BucketMap[ExpandedBucketIndex];
  if (CompressedBucketIndex == -1)
    return Result;

  uint32_t LastBucketIndex = GlobalsTable.HashBuckets.size() - 1;
  uint32_t StartRecordIndex =
      GlobalsTable.HashBuckets[CompressedBucketIndex] / 12;
  uint32_t EndRecordIndex = 0;
  if (LLVM_LIKELY(uint32_t(CompressedBucketIndex) < LastBucketIndex)) {
    EndRecordIndex = GlobalsTable.HashBuckets[CompressedBucketIndex + 1];
  } else {
    // If this is the last bucket, it consists of all hash records until the end
    // of the HashRecords array.
    EndRecordIndex = GlobalsTable.HashRecords.size() * 12;
  }

  EndRecordIndex /= 12;

  assert(EndRecordIndex <= GlobalsTable.HashRecords.size());
  while (StartRecordIndex < EndRecordIndex) {
    PSHashRecord PSH = GlobalsTable.HashRecords[StartRecordIndex];
    uint32_t Off = PSH.Off - 1;
    codeview::CVSymbol Record = Symbols.readRecord(Off);
    if (codeview::getSymbolName(Record) == Name)
      Result.push_back(std::make_pair(Off, std::move(Record)));
    ++StartRecordIndex;
  }
```
- EN: This section centers on `Reader`, `success`, `findRecordsByName` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `Reader`, `success`, `findRecordsByName` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 81-120

```cpp
  return Result;
}

static Error checkHashHdrVersion(const GSIHashHeader *HashHdr) {
  if (HashHdr->VerHdr != GSIHashHeader::HdrVersion)
    return make_error<RawError>(
        raw_error_code::feature_unsupported,
        "Encountered unsupported globals stream version.");

  return Error::success();
}

static Error readGSIHashHeader(const GSIHashHeader *&HashHdr,
                               BinaryStreamReader &Reader) {
  if (Reader.readObject(HashHdr))
    return make_error<RawError>(raw_error_code::corrupt_file,
                                "Stream does not contain a GSIHashHeader.");

  if (HashHdr->VerSignature != GSIHashHeader::HdrSignature)
    return make_error<RawError>(
        raw_error_code::feature_unsupported,
        "GSIHashHeader signature (0xffffffff) not found.");

  return Error::success();
}

static Error readGSIHashRecords(FixedStreamArray<PSHashRecord> &HashRecords,
                                const GSIHashHeader *HashHdr,
                                BinaryStreamReader &Reader) {
  if (auto EC = checkHashHdrVersion(HashHdr))
    return EC;

  // HashHdr->HrSize specifies the number of bytes of PSHashRecords we have.
  // Verify that we can read them all.
  if (HashHdr->HrSize % sizeof(PSHashRecord))
    return make_error<RawError>(raw_error_code::corrupt_file,
                                "Invalid HR array size.");
  uint32_t NumHashRecords = HashHdr->HrSize / sizeof(PSHashRecord);
  if (auto EC = Reader.readArray(HashRecords, NumHashRecords))
    return joinErrors(std::move(EC),
```
- EN: This section centers on `checkHashHdrVersion`, `success`, `readGSIHashHeader` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `checkHashHdrVersion`, `success`, `readGSIHashHeader` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 121-160

```cpp
                      make_error<RawError>(raw_error_code::corrupt_file,
                                           "Error reading hash records."));

  return Error::success();
}

static Error
readGSIHashBuckets(FixedStreamArray<support::ulittle32_t> &HashBuckets,
                   FixedStreamArray<support::ulittle32_t> &HashBitmap,
                   const GSIHashHeader *HashHdr,
                   MutableArrayRef<int32_t> BucketMap,
                   BinaryStreamReader &Reader) {
  if (auto EC = checkHashHdrVersion(HashHdr))
    return EC;

  // Before the actual hash buckets, there is a bitmap of length determined by
  // IPHR_HASH.
  size_t BitmapSizeInBits = alignTo(IPHR_HASH + 1, 32);
  uint32_t NumBitmapEntries = BitmapSizeInBits / 32;
  if (auto EC = Reader.readArray(HashBitmap, NumBitmapEntries))
    return joinErrors(std::move(EC),
                      make_error<RawError>(raw_error_code::corrupt_file,
                                           "Could not read a bitmap."));
  uint32_t CompressedBucketIdx = 0;
  for (uint32_t I = 0; I <= IPHR_HASH; ++I) {
    uint8_t WordIdx = I / 32;
    uint8_t BitIdx = I % 32;
    bool IsSet = HashBitmap[WordIdx] & (1U << BitIdx);
    if (IsSet) {
      BucketMap[I] = CompressedBucketIdx++;
    } else {
      BucketMap[I] = -1;
    }
  }

  uint32_t NumBuckets = 0;
  for (uint32_t B : HashBitmap)
    NumBuckets += llvm::popcount(B);

  // Hash buckets follow.
```
- EN: This section centers on `success`, `readGSIHashBuckets` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `success`, `readGSIHashBuckets` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 161-179

```cpp
  if (auto EC = Reader.readArray(HashBuckets, NumBuckets))
    return joinErrors(std::move(EC),
                      make_error<RawError>(raw_error_code::corrupt_file,
                                           "Hash buckets corrupted."));

  return Error::success();
}

Error GSIHashTable::read(BinaryStreamReader &Reader) {
  if (auto EC = readGSIHashHeader(HashHdr, Reader))
    return EC;
  if (auto EC = readGSIHashRecords(HashRecords, HashHdr, Reader))
    return EC;
  if (HashHdr->HrSize > 0)
    if (auto EC = readGSIHashBuckets(HashBuckets, HashBitmap, HashHdr,
                                     BucketMap, Reader))
      return EC;
  return Error::success();
}
```
- EN: This section centers on `success`, `read` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `success`, `read` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

## Key Concepts / 关键概念
- Domain / 领域: PDB debug information / PDB 调试信息
- Core symbols / 核心符号: `GlobalsStream`, `reload`, `Reader`, `success` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Parsing, decoding, and structural validation. / 重点关注解析、解码与结构校验。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/DebugInfo/PDB/Native/GlobalsStream.h`, `llvm/DebugInfo/CodeView/RecordName.h`, `llvm/DebugInfo/MSF/MappedBlockStream.h`, `llvm/DebugInfo/PDB/Native/Hash.h`, `llvm/DebugInfo/PDB/Native/RawError.h`, `llvm/DebugInfo/PDB/Native/SymbolStream.h`, `llvm/Support/BinaryStreamReader.h`, `llvm/Support/Error.h`
- Standard library / 标准库: None / 无
- Other/system headers / 其他或系统头文件: None / 无
- Related symbols / 相关符号: `GlobalsStream`, `reload`, `Reader`, `success`, `findRecordsByName`
