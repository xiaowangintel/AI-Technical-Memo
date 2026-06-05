# StableFunctionMapRecord.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/lib/CGData/StableFunctionMapRecord.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This implements the functionality for the StableFunctionMapRecord class, including methods for serialization and deserialization of stable function maps to and from raw and YAML streams. It also includes utilities for managing function entries and their metadata.
  - **CN**: 实现代码生成剖析数据的数据结构、读取器与写出器。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- StableFunctionMapRecord.cpp ---------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件的高层描述。

### Lines 8-14
```cpp
//
// This implements the functionality for the StableFunctionMapRecord class,
// including methods for serialization and deserialization of stable function
// maps to and from raw and YAML streams. It also includes utilities for
// managing function entries and their metadata.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 15-23
```cpp

#include "llvm/CGData/StableFunctionMapRecord.h"
#include "llvm/Support/EndianStream.h"

#define DEBUG_TYPE "stable-function-map-record"

using namespace llvm;
using namespace llvm::support;

```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/CGData/StableFunctionMapRecord.h`, `llvm/Support/EndianStream.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/CGData/StableFunctionMapRecord.h`, `llvm/Support/EndianStream.h`。

### Lines 24-37
```cpp
LLVM_YAML_IS_SEQUENCE_VECTOR(IndexPairHash)
LLVM_YAML_IS_SEQUENCE_VECTOR(StableFunction)

namespace llvm {
namespace yaml {

template <> struct MappingTraits<IndexPairHash> {
  static void mapping(IO &IO, IndexPairHash &Key) {
    IO.mapRequired("InstIndex", Key.first.first);
    IO.mapRequired("OpndIndex", Key.first.second);
    IO.mapRequired("OpndHash", Key.second);
  }
};

```
- **EN**: Introduces declarations for `llvm`, `yaml`, `MappingTraits`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `llvm`, `yaml`, `MappingTraits` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 38-47
```cpp
template <> struct MappingTraits<StableFunction> {
  static void mapping(IO &IO, StableFunction &Func) {
    IO.mapRequired("Hash", Func.Hash);
    IO.mapRequired("FunctionName", Func.FunctionName);
    IO.mapRequired("ModuleName", Func.ModuleName);
    IO.mapRequired("InstCount", Func.InstCount);
    IO.mapRequired("IndexOperandHashes", Func.IndexOperandHashes);
  }
};

```
- **EN**: Introduces declarations for `MappingTraits`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `MappingTraits` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 48-58
```cpp
} // namespace yaml
} // namespace llvm

// Get a sorted vector of StableFunctionEntry pointers.
static SmallVector<const StableFunctionMap::StableFunctionEntry *>
getStableFunctionEntries(const StableFunctionMap &SFM) {
  SmallVector<const StableFunctionMap::StableFunctionEntry *> FuncEntries;
  for (const auto &P : SFM.getFunctionMap())
    for (auto &Func : P.second.Entries)
      FuncEntries.emplace_back(Func.get());

```
- **EN**: Introduces declarations for `yaml`, `llvm`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `yaml`, `llvm` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 59-68
```cpp
  llvm::stable_sort(
      FuncEntries, [&](auto &A, auto &B) {
        return std::tuple(A->Hash, SFM.getNameForId(A->ModuleNameId),
                          SFM.getNameForId(A->FunctionNameId)) <
               std::tuple(B->Hash, SFM.getNameForId(B->ModuleNameId),
                          SFM.getNameForId(B->FunctionNameId));
      });
  return FuncEntries;
}

```
- **EN**: Implements logic around `stable_sort`, `tuple`, `getNameForId`; this block works with hashed storage or cache state.
- **CN**: 围绕 `stable_sort`, `tuple`, `getNameForId` 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 69-79
```cpp
// Get a sorted vector of IndexOperandHashes.
static IndexOperandHashVecType getStableIndexOperandHashes(
    const StableFunctionMap::StableFunctionEntry *FuncEntry) {
  IndexOperandHashVecType IndexOperandHashes;
  for (auto &[Indices, OpndHash] : *FuncEntry->IndexOperandHashMap)
    IndexOperandHashes.emplace_back(Indices, OpndHash);
  // The indices are unique, so we can just sort by the first.
  llvm::sort(IndexOperandHashes);
  return IndexOperandHashes;
}

```
- **EN**: Implements logic around `getStableIndexOperandHashes`, `emplace_back`, `sort`.
- **CN**: 围绕 `getStableIndexOperandHashes`, `emplace_back`, `sort` 实现具体逻辑。

### Lines 80-89
```cpp
void StableFunctionMapRecord::serialize(
    raw_ostream &OS, std::vector<CGDataPatchItem> &PatchItems) const {
  serialize(OS, FunctionMap.get(), PatchItems);
}

void StableFunctionMapRecord::serialize(
    raw_ostream &OS, const StableFunctionMap *FunctionMap,
    std::vector<CGDataPatchItem> &PatchItems) {
  support::endian::Writer Writer(OS, endianness::little);

```
- **EN**: Implements logic around `serialize`, `Writer`; this block emits or serializes data to an external representation.
- **CN**: 围绕 `serialize`, `Writer` 实现具体逻辑；该代码块把数据输出或序列化为外部表示。

### Lines 90-103
```cpp
  // Write Names.
  ArrayRef<std::string> Names = FunctionMap->getNames();
  Writer.write<uint32_t>(Names.size());
  // Remember the position, write back the total size of Names, so we can skip
  // reading them if needed.
  const uint64_t NamesByteSizeOffset = Writer.OS.tell();
  Writer.write<uint64_t>(0);
  for (auto &Name : Names)
    Writer.OS << Name << '\0';
  // Align current position to 4 bytes.
  uint32_t Padding = offsetToAlignment(Writer.OS.tell(), Align(4));
  for (uint32_t I = 0; I < Padding; ++I)
    Writer.OS << '\0';
  const auto NamesByteSize =
```
- **EN**: Implements logic around `getNames`, `write`, `tell`, `offsetToAlignment`; this block emits or serializes data to an external representation.
- **CN**: 围绕 `getNames`, `write`, `tell`, `offsetToAlignment` 实现具体逻辑；该代码块把数据输出或序列化为外部表示。

### Lines 104-117
```cpp
      Writer.OS.tell() - NamesByteSizeOffset - sizeof(NamesByteSizeOffset);
  PatchItems.emplace_back(NamesByteSizeOffset, &NamesByteSize, 1);

  // Write StableFunctionEntries whose pointers are sorted.
  auto FuncEntries = getStableFunctionEntries(*FunctionMap);
  Writer.write<uint32_t>(FuncEntries.size());
  for (const auto *FuncRef : FuncEntries)
    Writer.write<stable_hash>(FuncRef->Hash);
  std::vector<uint64_t> IndexOperandHashesOffsets;
  IndexOperandHashesOffsets.reserve(FuncEntries.size());
  for (const auto *FuncRef : FuncEntries) {
    Writer.write<uint32_t>(FuncRef->FunctionNameId);
    Writer.write<uint32_t>(FuncRef->ModuleNameId);
    Writer.write<uint32_t>(FuncRef->InstCount);
```
- **EN**: Implements logic around `tell`, `emplace_back`, `getStableFunctionEntries`, `write`, and 1 more symbols; this block emits or serializes data to an external representation; works with hashed storage or cache state.
- **CN**: 围绕 `tell`, `emplace_back`, `getStableFunctionEntries`, `write`, and 1 more symbols 实现具体逻辑；该代码块把数据输出或序列化为外部表示，并处理基于哈希的存储或缓存状态。

### Lines 118-131
```cpp
    const uint64_t Offset = Writer.OS.tell();
    IndexOperandHashesOffsets.push_back(Offset);
    Writer.write<uint64_t>(0);
  }
  const uint64_t IndexOperandHashesByteSizeOffset = Writer.OS.tell();
  Writer.write<uint64_t>(0);
  for (size_t I = 0; I < FuncEntries.size(); ++I) {
    const uint64_t Offset = Writer.OS.tell() - IndexOperandHashesOffsets[I];
    PatchItems.emplace_back(IndexOperandHashesOffsets[I], &Offset, 1);
    // Emit IndexOperandHashes sorted from IndexOperandHashMap.
    const auto *FuncRef = FuncEntries[I];
    IndexOperandHashVecType IndexOperandHashes =
        getStableIndexOperandHashes(FuncRef);
    Writer.write<uint32_t>(IndexOperandHashes.size());
```
- **EN**: Implements logic around `tell`, `push_back`, `write`, `size`, and 2 more symbols; this block emits or serializes data to an external representation.
- **CN**: 围绕 `tell`, `push_back`, `write`, `size`, and 2 more symbols 实现具体逻辑；该代码块把数据输出或序列化为外部表示。

### Lines 132-144
```cpp
    for (auto &IndexOperandHash : IndexOperandHashes) {
      Writer.write<uint32_t>(IndexOperandHash.first.first);
      Writer.write<uint32_t>(IndexOperandHash.first.second);
      Writer.write<stable_hash>(IndexOperandHash.second);
    }
  }
  // Write the total size of IndexOperandHashes.
  const uint64_t IndexOperandHashesByteSize =
      Writer.OS.tell() - IndexOperandHashesByteSizeOffset - sizeof(uint64_t);
  PatchItems.emplace_back(IndexOperandHashesByteSizeOffset,
                          &IndexOperandHashesByteSize, 1);
}

```
- **EN**: Implements logic around `write`, `tell`, `emplace_back`; this block emits or serializes data to an external representation.
- **CN**: 围绕 `write`, `tell`, `emplace_back` 实现具体逻辑；该代码块把数据输出或序列化为外部表示。

### Lines 145-158
```cpp
void StableFunctionMapRecord::deserializeEntry(const unsigned char *Ptr,
                                               stable_hash Hash,
                                               StableFunctionMap *FunctionMap) {
  auto FunctionNameId =
      endian::readNext<uint32_t, endianness::little, unaligned>(Ptr);
  if (FunctionMap->ReadStableFunctionMapNames)
    assert(FunctionMap->getNameForId(FunctionNameId) &&
           "FunctionNameId out of range");
  auto ModuleNameId =
      endian::readNext<uint32_t, endianness::little, unaligned>(Ptr);
  if (FunctionMap->ReadStableFunctionMapNames)
    assert(FunctionMap->getNameForId(ModuleNameId) &&
           "ModuleNameId out of range");
  auto InstCount =
```
- **EN**: Implements logic around `deserializeEntry`, `unaligned>`, `assert`; this block works with hashed storage or cache state.
- **CN**: 围绕 `deserializeEntry`, `unaligned>`, `assert` 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 159-172
```cpp
      endian::readNext<uint32_t, endianness::little, unaligned>(Ptr);

  // Read IndexOperandHashes to build IndexOperandHashMap
  auto CurrentPosition = reinterpret_cast<uintptr_t>(Ptr);
  auto IndexOperandHashesOffset =
      endian::readNext<uint64_t, endianness::little, unaligned>(Ptr);
  auto *IndexOperandHashesPtr = reinterpret_cast<const unsigned char *>(
      CurrentPosition + IndexOperandHashesOffset);
  auto NumIndexOperandHashes =
      endian::readNext<uint32_t, endianness::little, unaligned>(
          IndexOperandHashesPtr);
  auto IndexOperandHashMap = std::make_unique<IndexOperandHashMapType>();
  for (unsigned J = 0; J < NumIndexOperandHashes; ++J) {
    auto InstIndex = endian::readNext<uint32_t, endianness::little, unaligned>(
```
- **EN**: Implements logic around `unaligned>`, `reinterpret_cast`, `make_unique`.
- **CN**: 围绕 `unaligned>`, `reinterpret_cast`, `make_unique` 实现具体逻辑。

### Lines 173-180
```cpp
        IndexOperandHashesPtr);
    auto OpndIndex = endian::readNext<uint32_t, endianness::little, unaligned>(
        IndexOperandHashesPtr);
    auto OpndHash =
        endian::readNext<stable_hash, endianness::little, unaligned>(
            IndexOperandHashesPtr);
    assert(InstIndex < InstCount && "InstIndex out of range");

```
- **EN**: Implements logic around `unaligned>`, `assert`.
- **CN**: 围绕 `unaligned>`, `assert` 实现具体逻辑。

### Lines 181-188
```cpp
    IndexOperandHashMap->try_emplace({InstIndex, OpndIndex}, OpndHash);
  }

  // Insert a new StableFunctionEntry into the map.
  auto FuncEntry = std::make_unique<StableFunctionMap::StableFunctionEntry>(
      Hash, FunctionNameId, ModuleNameId, InstCount,
      std::move(IndexOperandHashMap));

```
- **EN**: Implements logic around `try_emplace`, `StableFunctionEntry>`, `move`; this block works with hashed storage or cache state.
- **CN**: 围绕 `try_emplace`, `StableFunctionEntry>`, `move` 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 189-202
```cpp
  FunctionMap->insert(std::move(FuncEntry));
}

void StableFunctionMapRecord::deserialize(const unsigned char *&Ptr,
                                          bool Lazy) {
  // Assert that Ptr is 4-byte aligned
  assert(((uintptr_t)Ptr % 4) == 0);
  // Read Names.
  auto NumNames =
      endian::readNext<uint32_t, endianness::little, unaligned>(Ptr);
  // Early exit if there is no name.
  if (NumNames == 0)
    return;
  const auto NamesByteSize =
```
- **EN**: Implements logic around `insert`, `deserialize`, `assert`, `unaligned>`.
- **CN**: 围绕 `insert`, `deserialize`, `assert`, `unaligned>` 实现具体逻辑。

### Lines 203-216
```cpp
      endian::readNext<uint64_t, endianness::little, unaligned>(Ptr);
  const auto NamesOffset = reinterpret_cast<uintptr_t>(Ptr);
  if (FunctionMap->ReadStableFunctionMapNames) {
    for (unsigned I = 0; I < NumNames; ++I) {
      StringRef Name(reinterpret_cast<const char *>(Ptr));
      Ptr += Name.size() + 1;
      FunctionMap->getIdOrCreateForName(Name);
    }
    // Align Ptr to 4 bytes.
    Ptr = reinterpret_cast<const uint8_t *>(alignAddr(Ptr, Align(4)));
    assert(reinterpret_cast<uintptr_t>(Ptr) - NamesOffset == NamesByteSize &&
           "NamesByteSize does not match the actual size of names");
  } else {
    // skip reading Names by advancing the pointer.
```
- **EN**: Implements logic around `unaligned>`, `reinterpret_cast`, `Name`, `size`, and 3 more symbols.
- **CN**: 围绕 `unaligned>`, `reinterpret_cast`, `Name`, `size`, and 3 more symbols 实现具体逻辑。

### Lines 217-230
```cpp
    Ptr = reinterpret_cast<const uint8_t *>(NamesOffset + NamesByteSize);
  }

  // Read StableFunctionEntries.
  auto NumFuncs =
      endian::readNext<uint32_t, endianness::little, unaligned>(Ptr);
  auto FixedSizeFieldsOffset =
      reinterpret_cast<uintptr_t>(Ptr) + NumFuncs * sizeof(stable_hash);
  constexpr uint32_t FixedSizeFieldsSizePerEntry =
      // FunctionNameId
      sizeof(uint32_t) +
      // ModuleNameId
      sizeof(uint32_t) +
      // InstCount
```
- **EN**: Implements logic around `unaligned>`, `reinterpret_cast`.
- **CN**: 围绕 `unaligned>`, `reinterpret_cast` 实现具体逻辑。

### Lines 231-244
```cpp
      sizeof(uint32_t) +
      // Relative offset to IndexOperandHashes
      sizeof(uint64_t);
  for (unsigned I = 0; I < NumFuncs; ++I) {
    auto Hash =
        endian::readNext<stable_hash, endianness::little, unaligned>(Ptr);
    if (Lazy) {
      auto It = FunctionMap->HashToFuncs.try_emplace(Hash).first;
      StableFunctionMap::EntryStorage &Storage = It->second;
      Storage.Offsets.push_back(FixedSizeFieldsOffset);
    } else {
      deserializeEntry(
          reinterpret_cast<const unsigned char *>(FixedSizeFieldsOffset), Hash,
          FunctionMap.get());
```
- **EN**: Implements logic around `unaligned>`, `try_emplace`, `push_back`, `deserializeEntry`, and 1 more symbols; this block works with hashed storage or cache state.
- **CN**: 围绕 `unaligned>`, `try_emplace`, `push_back`, `deserializeEntry`, and 1 more symbols 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 245-257
```cpp
    }
    FixedSizeFieldsOffset += FixedSizeFieldsSizePerEntry;
  }

  // Update Ptr to the end of the serialized map to meet the expectation of
  // CodeGenDataReader.
  Ptr = reinterpret_cast<const unsigned char *>(FixedSizeFieldsOffset);
  auto IndexOperandHashesByteSize =
      endian::readNext<uint64_t, endianness::little, unaligned>(Ptr);
  Ptr = reinterpret_cast<const unsigned char *>(
      reinterpret_cast<uintptr_t>(Ptr) + IndexOperandHashesByteSize);
}

```
- **EN**: Implements logic around `unaligned>`, `reinterpret_cast`; this block emits or serializes data to an external representation.
- **CN**: 围绕 `unaligned>`, `reinterpret_cast` 实现具体逻辑；该代码块把数据输出或序列化为外部表示。

### Lines 258-269
```cpp
void StableFunctionMapRecord::deserialize(const unsigned char *&Ptr) {
  deserialize(Ptr, /*Lazy=*/false);
}

void StableFunctionMapRecord::lazyDeserialize(
    std::shared_ptr<MemoryBuffer> Buffer, uint64_t Offset) {
  const auto *Ptr = reinterpret_cast<const unsigned char *>(
      reinterpret_cast<uintptr_t>(Buffer->getBufferStart()) + Offset);
  deserialize(Ptr, /*Lazy=*/true);
  FunctionMap->Buffer = std::move(Buffer);
}

```
- **EN**: Implements logic around `deserialize`, `lazyDeserialize`, `reinterpret_cast`, `move`.
- **CN**: 围绕 `deserialize`, `lazyDeserialize`, `reinterpret_cast`, `move` 实现具体逻辑。

### Lines 270-280
```cpp
void StableFunctionMapRecord::serializeYAML(yaml::Output &YOS) const {
  auto FuncEntries = getStableFunctionEntries(*FunctionMap);
  SmallVector<StableFunction> Functions;
  for (const auto *FuncEntry : FuncEntries) {
    auto IndexOperandHashes = getStableIndexOperandHashes(FuncEntry);
    Functions.emplace_back(
        FuncEntry->Hash, *FunctionMap->getNameForId(FuncEntry->FunctionNameId),
        *FunctionMap->getNameForId(FuncEntry->ModuleNameId),
        FuncEntry->InstCount, std::move(IndexOperandHashes));
  }

```
- **EN**: Implements logic around `serializeYAML`, `getStableFunctionEntries`, `getStableIndexOperandHashes`, `emplace_back`, and 2 more symbols; this block emits or serializes data to an external representation; works with hashed storage or cache state.
- **CN**: 围绕 `serializeYAML`, `getStableFunctionEntries`, `getStableIndexOperandHashes`, `emplace_back`, and 2 more symbols 实现具体逻辑；该代码块把数据输出或序列化为外部表示，并处理基于哈希的存储或缓存状态。

### Lines 281-290
```cpp
  YOS << Functions;
}

void StableFunctionMapRecord::deserializeYAML(yaml::Input &YIS) {
  std::vector<StableFunction> Funcs;
  YIS >> Funcs;
  for (auto &Func : Funcs)
    FunctionMap->insert(Func);
  YIS.nextDocument();
}
```
- **EN**: Implements logic around `deserializeYAML`, `insert`, `nextDocument`.
- **CN**: 围绕 `deserializeYAML`, `insert`, `nextDocument` 实现具体逻辑。

## Key Concepts / 关键概念

- **CodeGen profiling data / 代码生成剖析数据**:
  - **EN**: Stores or transfers profile-like data consumed by code-generation workflows.
  - **CN**: 存储或传输代码生成流程消费的剖析类数据。
- **LLVM container usage / LLVM 容器使用**:
  - **EN**: Relies on LLVM ADT containers for performance-conscious in-memory data management.
  - **CN**: 依赖 LLVM ADT 容器来进行注重性能的内存数据管理。
- **Streaming output / 流式输出**:
  - **EN**: Writes diagnostics, serialized data, or textual representations through LLVM stream APIs.
  - **CN**: 通过 LLVM 流式 API 输出诊断、序列化数据或文本表示。
- **YAML bridging / YAML 桥接**:
  - **EN**: Converts LLVM-internal structures to or from YAML representations.
  - **CN**: 在 LLVM 内部结构与 YAML 表示之间进行转换。

## Dependencies / 依赖关系

- **Direct LLVM/local includes / 直接的 LLVM/本地包含**: `llvm/CGData/StableFunctionMapRecord.h`, `llvm/Support/EndianStream.h`
- **Subsystem categories / 子系统类别**: support-library helpers / Support 库辅助功能 (1)
