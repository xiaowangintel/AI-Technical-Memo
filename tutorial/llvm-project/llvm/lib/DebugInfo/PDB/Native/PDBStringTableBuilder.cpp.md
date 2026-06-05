# PDBStringTableBuilder.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/DebugInfo/PDB/Native/PDBStringTableBuilder.cpp`
- Repository: `llvm-project`
- Purpose (EN): The reference implementation doesn't include code for /src/headerblock handling, but it can only read natvis entries lld's PDB files if this hash function truncates the hash to 16 bit.
- Purpose (CN): 该文件位于 LLVM 的 `DebugInfo/PDB/Native` 目录中，主要实现与 `PDBStringTableBuilder` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===- PDBStringTableBuilder.cpp - PDB String Table -------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/DebugInfo/PDB/Native/PDBStringTableBuilder.h"

#include "llvm/ADT/ArrayRef.h"
#include "llvm/DebugInfo/PDB/Native/Hash.h"
#include "llvm/DebugInfo/PDB/Native/RawTypes.h"
#include "llvm/Support/BinaryStreamWriter.h"
#include "llvm/Support/Endian.h"
#include "llvm/Support/TimeProfiler.h"

using namespace llvm;
using namespace llvm::msf;
using namespace llvm::support;
using namespace llvm::support::endian;
using namespace llvm::pdb;

StringTableHashTraits::StringTableHashTraits(PDBStringTableBuilder &Table)
    : Table(&Table) {}

uint32_t StringTableHashTraits::hashLookupKey(StringRef S) const {
  // The reference implementation doesn't include code for /src/headerblock
  // handling, but it can only read natvis entries lld's PDB files if
  // this hash function truncates the hash to 16 bit.
  // PDB/include/misc.h in the reference implementation has a hashSz() function
  // that returns an unsigned short, that seems what's being used for
  // /src/headerblock.
  return static_cast<uint16_t>(Table->getIdForString(S));
}

StringRef StringTableHashTraits::storageKeyToLookupKey(uint32_t Offset) const {
  return Table->getStringForId(Offset);
}

```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 7 direct dependencies, including `llvm/DebugInfo/PDB/Native/PDBStringTableBuilder.h`, `llvm/ADT/ArrayRef.h`, `llvm/DebugInfo/PDB/Native/Hash.h`, `llvm/DebugInfo/PDB/Native/RawTypes.h`.
  CN: 引入了 7 个直接依赖，其中包括 `llvm/DebugInfo/PDB/Native/PDBStringTableBuilder.h`, `llvm/ADT/ArrayRef.h`, `llvm/DebugInfo/PDB/Native/Hash.h`, `llvm/DebugInfo/PDB/Native/RawTypes.h`。
- EN: This section centers on `StringTableHashTraits`, `hashLookupKey`, `storageKeyToLookupKey` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `StringTableHashTraits`, `hashLookupKey`, `storageKeyToLookupKey` 等符号展开，负责查询现有状态并返回计算出的元数据。

### Lines 41-80

```cpp
uint32_t StringTableHashTraits::lookupKeyToStorageKey(StringRef S) {
  return Table->insert(S);
}

uint32_t PDBStringTableBuilder::insert(StringRef S) {
  return Strings.insert(S);
}

uint32_t PDBStringTableBuilder::getIdForString(StringRef S) const {
  return Strings.getIdForString(S);
}

StringRef PDBStringTableBuilder::getStringForId(uint32_t Id) const {
  return Strings.getStringForId(Id);
}

static uint32_t computeBucketCount(uint32_t NumStrings) {
  // This is a precomputed list of Buckets given the specified number of
  // strings.  Matching the reference algorithm exactly is not strictly
  // necessary for correctness, but it helps when comparing LLD's PDBs with
  // Microsoft's PDBs so as to eliminate superfluous differences.
  // The reference implementation does (in nmt.h, NMT::grow()):
  //   unsigned StringCount = 0;
  //   unsigned BucketCount = 1;
  //   fn insert() {
  //     ++StringCount;
  //     if (BucketCount * 3 / 4 < StringCount)
  //       BucketCount = BucketCount * 3 / 2 + 1;
  //   }
  // This list contains all StringCount, BucketCount pairs where BucketCount was
  // just incremented.  It ends before the first BucketCount entry where
  // BucketCount * 3 would overflow a 32-bit unsigned int.
  static const std::pair<uint32_t, uint32_t> StringsToBuckets[] = {
      {0, 1},
      {1, 2},
      {2, 4},
      {4, 7},
      {6, 11},
      {9, 17},
      {13, 26},
```
- EN: This section centers on `lookupKeyToStorageKey`, `insert`, `getIdForString` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `lookupKeyToStorageKey`, `insert`, `getIdForString` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and returns the resulting value to its callers.
  CN: 在这一段中，代码检查条件并处理边界情况，并将结果返回给调用方。

### Lines 81-120

```cpp
      {20, 40},
      {31, 61},
      {46, 92},
      {70, 139},
      {105, 209},
      {157, 314},
      {236, 472},
      {355, 709},
      {532, 1064},
      {799, 1597},
      {1198, 2396},
      {1798, 3595},
      {2697, 5393},
      {4045, 8090},
      {6068, 12136},
      {9103, 18205},
      {13654, 27308},
      {20482, 40963},
      {30723, 61445},
      {46084, 92168},
      {69127, 138253},
      {103690, 207380},
      {155536, 311071},
      {233304, 466607},
      {349956, 699911},
      {524934, 1049867},
      {787401, 1574801},
      {1181101, 2362202},
      {1771652, 3543304},
      {2657479, 5314957},
      {3986218, 7972436},
      {5979328, 11958655},
      {8968992, 17937983},
      {13453488, 26906975},
      {20180232, 40360463},
      {30270348, 60540695},
      {45405522, 90811043},
      {68108283, 136216565},
      {102162424, 204324848},
      {153243637, 306487273},
```
- EN: Most of this range is a static table or dense initializer that feeds later lookup logic.
  CN: 这一段大多是静态表或密集初始化数据，供后续查找逻辑使用。

### Lines 121-160

```cpp
      {229865455, 459730910},
      {344798183, 689596366},
      {517197275, 1034394550},
      {775795913, 1551591826},
      {1163693870, 2327387740}};
  const auto *Entry = llvm::lower_bound(
      StringsToBuckets, std::make_pair(NumStrings, 0U), llvm::less_first());
  assert(Entry != std::end(StringsToBuckets));
  return Entry->second;
}

uint32_t PDBStringTableBuilder::calculateHashTableSize() const {
  uint32_t Size = sizeof(uint32_t); // Hash table begins with 4-byte size field.
  Size += sizeof(uint32_t) * computeBucketCount(Strings.size());

  return Size;
}

uint32_t PDBStringTableBuilder::calculateSerializedSize() const {
  uint32_t Size = 0;
  Size += sizeof(PDBStringTableHeader);
  Size += Strings.calculateSerializedSize();
  Size += calculateHashTableSize();
  Size += sizeof(uint32_t); // The /names stream ends with the string count.
  return Size;
}

void PDBStringTableBuilder::setStrings(
    const codeview::DebugStringTableSubsection &Strings) {
  this->Strings = Strings;
}

Error PDBStringTableBuilder::writeHeader(BinaryStreamWriter &Writer) const {
  // Write a header
  PDBStringTableHeader H;
  H.Signature = PDBStringTableSignature;
  H.HashVersion = 1;
  H.ByteSize = Strings.calculateSerializedSize();
  if (auto EC = Writer.writeObject(H))
    return EC;
```
- EN: This section centers on `make_pair`, `assert`, `calculateHashTableSize` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `make_pair`, `assert`, `calculateHashTableSize` 等符号展开，负责创建并初始化辅助对象或状态。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 161-200

```cpp
  assert(Writer.bytesRemaining() == 0);
  return Error::success();
}

Error PDBStringTableBuilder::writeStrings(BinaryStreamWriter &Writer) const {
  if (auto EC = Strings.commit(Writer))
    return EC;

  assert(Writer.bytesRemaining() == 0);
  return Error::success();
}

Error PDBStringTableBuilder::writeHashTable(BinaryStreamWriter &Writer) const {
  // Write a hash table.
  uint32_t BucketCount = computeBucketCount(Strings.size());
  if (auto EC = Writer.writeInteger(BucketCount))
    return EC;
  std::vector<ulittle32_t> Buckets(BucketCount);

  for (const auto &Pair : Strings) {
    StringRef S = Pair.getKey();
    uint32_t Offset = Pair.getValue();
    uint32_t Hash = hashStringV1(S);

    for (uint32_t I = 0; I != BucketCount; ++I) {
      uint32_t Slot = (Hash + I) % BucketCount;
      if (Buckets[Slot] != 0)
        continue;
      Buckets[Slot] = Offset;
      break;
    }
  }

  if (auto EC = Writer.writeArray(ArrayRef<ulittle32_t>(Buckets)))
    return EC;

  assert(Writer.bytesRemaining() == 0);
  return Error::success();
}

```
- EN: This section centers on `assert`, `success`, `writeStrings` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `assert`, `success`, `writeStrings` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 201-230

```cpp
Error PDBStringTableBuilder::writeEpilogue(BinaryStreamWriter &Writer) const {
  if (auto EC = Writer.writeInteger<uint32_t>(Strings.size()))
    return EC;
  assert(Writer.bytesRemaining() == 0);
  return Error::success();
}

Error PDBStringTableBuilder::commit(BinaryStreamWriter &Writer) const {
  llvm::TimeTraceScope timeScope("Commit strings table");
  BinaryStreamWriter SectionWriter;

  std::tie(SectionWriter, Writer) = Writer.split(sizeof(PDBStringTableHeader));
  if (auto EC = writeHeader(SectionWriter))
    return EC;

  std::tie(SectionWriter, Writer) =
      Writer.split(Strings.calculateSerializedSize());
  if (auto EC = writeStrings(SectionWriter))
    return EC;

  std::tie(SectionWriter, Writer) = Writer.split(calculateHashTableSize());
  if (auto EC = writeHashTable(SectionWriter))
    return EC;

  std::tie(SectionWriter, Writer) = Writer.split(sizeof(uint32_t));
  if (auto EC = writeEpilogue(SectionWriter))
    return EC;

  return Error::success();
}
```
- EN: This section centers on `writeEpilogue`, `assert`, `success` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `writeEpilogue`, `assert`, `success` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

## Key Concepts / 关键概念
- Domain / 领域: PDB debug information / PDB 调试信息
- Core symbols / 核心符号: `StringTableHashTraits`, `hashLookupKey`, `storageKeyToLookupKey`, `lookupKeyToStorageKey` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Parsing, decoding, and structural validation. / 重点关注解析、解码与结构校验。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/DebugInfo/PDB/Native/PDBStringTableBuilder.h`, `llvm/ADT/ArrayRef.h`, `llvm/DebugInfo/PDB/Native/Hash.h`, `llvm/DebugInfo/PDB/Native/RawTypes.h`, `llvm/Support/BinaryStreamWriter.h`, `llvm/Support/Endian.h`, `llvm/Support/TimeProfiler.h`
- Standard library / 标准库: None / 无
- Other/system headers / 其他或系统头文件: None / 无
- Related symbols / 相关符号: `StringTableHashTraits`, `hashLookupKey`, `storageKeyToLookupKey`, `lookupKeyToStorageKey`, `insert`
