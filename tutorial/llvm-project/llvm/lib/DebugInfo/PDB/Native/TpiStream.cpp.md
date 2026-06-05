# TpiStream.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/DebugInfo/PDB/Native/TpiStream.cpp`
- Repository: `llvm-project`
- Purpose (EN): The actual type records themselves come from this stream
- Purpose (CN): 该文件位于 LLVM 的 `DebugInfo/PDB/Native` 目录中，主要实现与 `TpiStream` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===- TpiStream.cpp - PDB Type Info (TPI) Stream 2 Access ----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/DebugInfo/PDB/Native/TpiStream.h"

#include "llvm/ADT/iterator_range.h"
#include "llvm/DebugInfo/CodeView/LazyRandomTypeCollection.h"
#include "llvm/DebugInfo/CodeView/RecordName.h"
#include "llvm/DebugInfo/CodeView/TypeRecord.h"
#include "llvm/DebugInfo/CodeView/TypeRecordHelpers.h"
#include "llvm/DebugInfo/MSF/MappedBlockStream.h"
#include "llvm/DebugInfo/PDB/Native/Hash.h"
#include "llvm/DebugInfo/PDB/Native/PDBFile.h"
#include "llvm/DebugInfo/PDB/Native/RawConstants.h"
#include "llvm/DebugInfo/PDB/Native/RawError.h"
#include "llvm/DebugInfo/PDB/Native/RawTypes.h"
#include "llvm/DebugInfo/PDB/Native/TpiHashing.h"
#include "llvm/Support/BinaryStreamReader.h"
#include "llvm/Support/Endian.h"
#include "llvm/Support/Error.h"
#include <cstdint>
#include <vector>

using namespace llvm;
using namespace llvm::codeview;
using namespace llvm::support;
using namespace llvm::msf;
using namespace llvm::pdb;

TpiStream::TpiStream(PDBFile &File, std::unique_ptr<MappedBlockStream> Stream)
    : Pdb(File), Stream(std::move(Stream)) {}

TpiStream::~TpiStream() = default;

Error TpiStream::reload() {
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 18 direct dependencies, including `llvm/DebugInfo/PDB/Native/TpiStream.h`, `llvm/ADT/iterator_range.h`, `llvm/DebugInfo/CodeView/LazyRandomTypeCollection.h`, `llvm/DebugInfo/CodeView/RecordName.h`.
  CN: 引入了 18 个直接依赖，其中包括 `llvm/DebugInfo/PDB/Native/TpiStream.h`, `llvm/ADT/iterator_range.h`, `llvm/DebugInfo/CodeView/LazyRandomTypeCollection.h`, `llvm/DebugInfo/CodeView/RecordName.h`。
- EN: This section centers on `TpiStream`, `reload` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `TpiStream`, `reload` 等符号展开，负责实现局部控制流程与状态维护。

### Lines 41-80

```cpp
  BinaryStreamReader Reader(*Stream);

  if (Reader.bytesRemaining() < sizeof(TpiStreamHeader))
    return make_error<RawError>(raw_error_code::corrupt_file,
                                "TPI Stream does not contain a header.");

  if (Reader.readObject(Header))
    return make_error<RawError>(raw_error_code::corrupt_file,
                                "TPI Stream does not contain a header.");

  if (Header->Version != PdbTpiV80)
    return make_error<RawError>(raw_error_code::corrupt_file,
                                "Unsupported TPI Version.");

  if (Header->HeaderSize != sizeof(TpiStreamHeader))
    return make_error<RawError>(raw_error_code::corrupt_file,
                                "Corrupt TPI Header size.");

  if (Header->HashKeySize != sizeof(ulittle32_t))
    return make_error<RawError>(raw_error_code::corrupt_file,
                                "TPI Stream expected 4 byte hash key size.");

  if (Header->NumHashBuckets < MinTpiHashBuckets ||
      Header->NumHashBuckets > MaxTpiHashBuckets)
    return make_error<RawError>(raw_error_code::corrupt_file,
                                "TPI Stream Invalid number of hash buckets.");

  // The actual type records themselves come from this stream
  if (auto EC =
          Reader.readSubstream(TypeRecordsSubstream, Header->TypeRecordBytes))
    return EC;

  BinaryStreamReader RecordReader(TypeRecordsSubstream.StreamData);
  if (auto EC =
          RecordReader.readArray(TypeRecords, TypeRecordsSubstream.size()))
    return EC;

  // Hash indices, hash values, etc come from the hash stream.
  if (Header->HashStreamIndex != kInvalidStreamIndex) {
    auto HS = Pdb.safelyCreateIndexedStream(Header->HashStreamIndex);
```
- EN: This section centers on `Reader`, `RecordReader` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `Reader`, `RecordReader` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 81-120

```cpp
    if (!HS) {
      consumeError(HS.takeError());
      return make_error<RawError>(raw_error_code::corrupt_file,
                                  "Invalid TPI hash stream index.");
    }
    BinaryStreamReader HSR(**HS);

    // There should be a hash value for every type record, or no hashes at all.
    uint32_t NumHashValues =
        Header->HashValueBuffer.Length / sizeof(ulittle32_t);
    if (NumHashValues != getNumTypeRecords() && NumHashValues != 0)
      return make_error<RawError>(
          raw_error_code::corrupt_file,
          "TPI hash count does not match with the number of type records.");
    HSR.setOffset(Header->HashValueBuffer.Off);
    if (auto EC = HSR.readArray(HashValues, NumHashValues))
      return EC;

    HSR.setOffset(Header->IndexOffsetBuffer.Off);
    uint32_t NumTypeIndexOffsets =
        Header->IndexOffsetBuffer.Length / sizeof(TypeIndexOffset);
    if (auto EC = HSR.readArray(TypeIndexOffsets, NumTypeIndexOffsets))
      return EC;

    if (Header->HashAdjBuffer.Length > 0) {
      HSR.setOffset(Header->HashAdjBuffer.Off);
      if (auto EC = HashAdjusters.load(HSR))
        return EC;
    }

    HashStream = std::move(*HS);
  }

  Types = std::make_unique<LazyRandomTypeCollection>(
      TypeRecords, getNumTypeRecords(), getTypeIndexOffsets());
  return Error::success();
}

PdbRaw_TpiVer TpiStream::getTpiVersion() const {
  uint32_t Value = Header->Version;
```
- EN: This section centers on `consumeError`, `HSR`, `getNumTypeRecords` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `consumeError`, `HSR`, `getNumTypeRecords` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 121-160

```cpp
  return static_cast<PdbRaw_TpiVer>(Value);
}

uint32_t TpiStream::TypeIndexBegin() const { return Header->TypeIndexBegin; }

uint32_t TpiStream::TypeIndexEnd() const { return Header->TypeIndexEnd; }

uint32_t TpiStream::getNumTypeRecords() const {
  return TypeIndexEnd() - TypeIndexBegin();
}

uint16_t TpiStream::getTypeHashStreamIndex() const {
  return Header->HashStreamIndex;
}

uint16_t TpiStream::getTypeHashStreamAuxIndex() const {
  return Header->HashAuxStreamIndex;
}

uint32_t TpiStream::getNumHashBuckets() const { return Header->NumHashBuckets; }
uint32_t TpiStream::getHashKeySize() const { return Header->HashKeySize; }

void TpiStream::buildHashMap() {
  if (!HashMap.empty())
    return;
  if (HashValues.empty())
    return;

  HashMap.resize(Header->NumHashBuckets);

  TypeIndex TIB{Header->TypeIndexBegin};
  TypeIndex TIE{Header->TypeIndexEnd};
  while (TIB < TIE) {
    uint32_t HV = HashValues[TIB.toArrayIndex()];
    HashMap[HV].push_back(TIB++);
  }
}

std::vector<TypeIndex> TpiStream::findRecordsByName(StringRef Name) const {
  if (!supportsTypeLookup())
```
- EN: This section centers on `TypeIndexBegin`, `TypeIndexEnd`, `getNumTypeRecords` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `TypeIndexBegin`, `TypeIndexEnd`, `getNumTypeRecords` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 161-200

```cpp
    const_cast<TpiStream*>(this)->buildHashMap();

  uint32_t Bucket = hashStringV1(Name) % Header->NumHashBuckets;
  if (Bucket > HashMap.size())
    return {};

  std::vector<TypeIndex> Result;
  for (TypeIndex TI : HashMap[Bucket]) {
    std::string ThisName = computeTypeName(*Types, TI);
    if (ThisName == Name)
      Result.push_back(TI);
  }
  return Result;
}

bool TpiStream::supportsTypeLookup() const { return !HashMap.empty(); }

Expected<TypeIndex>
TpiStream::findFullDeclForForwardRef(TypeIndex ForwardRefTI) const {
  if (!supportsTypeLookup())
    const_cast<TpiStream*>(this)->buildHashMap();

  CVType F = Types->getType(ForwardRefTI);
  if (!isUdtForwardRef(F))
    return ForwardRefTI;

  Expected<TagRecordHash> ForwardTRH = hashTagRecord(F);
  if (!ForwardTRH)
    return ForwardTRH.takeError();

  uint32_t BucketIdx = ForwardTRH->FullRecordHash % Header->NumHashBuckets;

  for (TypeIndex TI : HashMap[BucketIdx]) {
    CVType CVT = Types->getType(TI);
    if (CVT.kind() != F.kind())
      continue;

    Expected<TagRecordHash> FullTRH = hashTagRecord(CVT);
    if (!FullTRH)
      return FullTRH.takeError();
```
- EN: This section centers on `supportsTypeLookup`, `findFullDeclForForwardRef` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `supportsTypeLookup`, `findFullDeclForForwardRef` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 201-240

```cpp
    if (ForwardTRH->FullRecordHash != FullTRH->FullRecordHash)
      continue;
    TagRecord &ForwardTR = ForwardTRH->getRecord();
    TagRecord &FullTR = FullTRH->getRecord();

    if (!ForwardTR.hasUniqueName()) {
      if (ForwardTR.getName() == FullTR.getName())
        return TI;
      continue;
    }

    if (!FullTR.hasUniqueName())
      continue;
    if (ForwardTR.getUniqueName() == FullTR.getUniqueName())
      return TI;
  }
  return ForwardRefTI;
}

codeview::CVType TpiStream::getType(codeview::TypeIndex Index) {
  assert(!Index.isSimple());
  return Types->getType(Index);
}

codeview::CVType TpiStream::getTypeOrEmpty(codeview::TypeIndex Index) {
  return Types->tryGetType(Index).value_or<CVType>({});
}

std::optional<codeview::CVType>
TpiStream::tryGetType(codeview::TypeIndex Index) {
  return Types->tryGetType(Index);
}

Expected<codeview::CVType>
TpiStream::getTypeOrError(codeview::TypeIndex Index) {
  return Types->getTypeOrError(Index);
}

BinarySubstreamRef TpiStream::getTypeRecordsSubstream() const {
  return TypeRecordsSubstream;
```
- EN: This section centers on `getType`, `assert`, `getTypeOrEmpty` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `getType`, `assert`, `getTypeOrEmpty` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 241-259

```cpp
}

FixedStreamArray<support::ulittle32_t> TpiStream::getHashValues() const {
  return HashValues;
}

FixedStreamArray<TypeIndexOffset> TpiStream::getTypeIndexOffsets() const {
  return TypeIndexOffsets;
}

HashTable<support::ulittle32_t> &TpiStream::getHashAdjusters() {
  return HashAdjusters;
}

CVTypeRange TpiStream::types(bool *HadError) const {
  return make_range(TypeRecords.begin(HadError), TypeRecords.end());
}

Error TpiStream::commit() { return Error::success(); }
```
- EN: This section centers on `getHashValues`, `getTypeIndexOffsets`, `types` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `getHashValues`, `getTypeIndexOffsets`, `types` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code propagates LLVM-style errors and invariants and returns the resulting value to its callers.
  CN: 在这一段中，代码传播 LLVM 风格的错误处理并维护不变量，并将结果返回给调用方。

## Key Concepts / 关键概念
- Domain / 领域: PDB debug information / PDB 调试信息
- Core symbols / 核心符号: `TpiStream`, `reload`, `Reader`, `RecordReader` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Helper routines coordinate local state, control flow, and result construction. / 通过辅助例程协调局部状态、控制流与结果构造。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/DebugInfo/PDB/Native/TpiStream.h`, `llvm/ADT/iterator_range.h`, `llvm/DebugInfo/CodeView/LazyRandomTypeCollection.h`, `llvm/DebugInfo/CodeView/RecordName.h`, `llvm/DebugInfo/CodeView/TypeRecord.h`, `llvm/DebugInfo/CodeView/TypeRecordHelpers.h`, `llvm/DebugInfo/MSF/MappedBlockStream.h`, `llvm/DebugInfo/PDB/Native/Hash.h`, `llvm/DebugInfo/PDB/Native/PDBFile.h`, `llvm/DebugInfo/PDB/Native/RawConstants.h`, `llvm/DebugInfo/PDB/Native/RawError.h`, `llvm/DebugInfo/PDB/Native/RawTypes.h`, `llvm/DebugInfo/PDB/Native/TpiHashing.h`, `llvm/Support/BinaryStreamReader.h`, `llvm/Support/Endian.h`
- Standard library / 标准库: `cstdint`, `vector`
- Other/system headers / 其他或系统头文件: None / 无
- Related symbols / 相关符号: `TpiStream`, `reload`, `Reader`, `RecordReader`, `consumeError`
