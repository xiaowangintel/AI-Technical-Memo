# PDBFileBuilder.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/DebugInfo/PDB/Native/PDBFileBuilder.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements PDBFileBuilder-related logic for LLVM's DebugInfo/PDB/Native component.
- Purpose (CN): 该文件位于 LLVM 的 `DebugInfo/PDB/Native` 目录中，主要实现与 `PDBFileBuilder` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===- PDBFileBuilder.cpp - PDB File Creation -------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/DebugInfo/PDB/Native/PDBFileBuilder.h"
#include "llvm/ADT/SmallString.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/DebugInfo/CodeView/GUID.h"
#include "llvm/DebugInfo/MSF/MSFBuilder.h"
#include "llvm/DebugInfo/MSF/MSFCommon.h"
#include "llvm/DebugInfo/MSF/MappedBlockStream.h"
#include "llvm/DebugInfo/PDB/Native/DbiStreamBuilder.h"
#include "llvm/DebugInfo/PDB/Native/GSIStreamBuilder.h"
#include "llvm/DebugInfo/PDB/Native/InfoStreamBuilder.h"
#include "llvm/DebugInfo/PDB/Native/PDBStringTableBuilder.h"
#include "llvm/DebugInfo/PDB/Native/RawConstants.h"
#include "llvm/DebugInfo/PDB/Native/RawError.h"
#include "llvm/DebugInfo/PDB/Native/RawTypes.h"
#include "llvm/DebugInfo/PDB/Native/TpiStreamBuilder.h"
#include "llvm/Support/BinaryStreamWriter.h"
#include "llvm/Support/CRC.h"
#include "llvm/Support/Path.h"
#include "llvm/Support/TimeProfiler.h"
#include "llvm/Support/xxhash.h"

#include <ctime>

using namespace llvm;
using namespace llvm::codeview;
using namespace llvm::msf;
using namespace llvm::pdb;
using namespace llvm::support;

namespace llvm {
class WritableBinaryStream;
}
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 21 direct dependencies, including `llvm/DebugInfo/PDB/Native/PDBFileBuilder.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/StringExtras.h`, `llvm/DebugInfo/CodeView/GUID.h`.
  CN: 引入了 21 个直接依赖，其中包括 `llvm/DebugInfo/PDB/Native/PDBFileBuilder.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/StringExtras.h`, `llvm/DebugInfo/CodeView/GUID.h`。
- EN: This range defines or extends data types such as `WritableBinaryStream`.
  CN: 这一段定义或扩展了 `WritableBinaryStream` 等数据类型。

### Lines 41-80

```cpp

PDBFileBuilder::PDBFileBuilder(BumpPtrAllocator &Allocator)
    : Allocator(Allocator), InjectedSourceHashTraits(Strings),
      InjectedSourceTable(2) {}

PDBFileBuilder::~PDBFileBuilder() = default;

Error PDBFileBuilder::initialize(uint32_t BlockSize) {
  auto ExpectedMsf = MSFBuilder::create(Allocator, BlockSize);
  if (!ExpectedMsf)
    return ExpectedMsf.takeError();
  Msf = std::make_unique<MSFBuilder>(std::move(*ExpectedMsf));
  return Error::success();
}

MSFBuilder &PDBFileBuilder::getMsfBuilder() { return *Msf; }

InfoStreamBuilder &PDBFileBuilder::getInfoBuilder() {
  if (!Info)
    Info = std::make_unique<InfoStreamBuilder>(*Msf, NamedStreams);
  return *Info;
}

DbiStreamBuilder &PDBFileBuilder::getDbiBuilder() {
  if (!Dbi)
    Dbi = std::make_unique<DbiStreamBuilder>(*Msf);
  return *Dbi;
}

TpiStreamBuilder &PDBFileBuilder::getTpiBuilder() {
  if (!Tpi)
    Tpi = std::make_unique<TpiStreamBuilder>(*Msf, StreamTPI);
  return *Tpi;
}

TpiStreamBuilder &PDBFileBuilder::getIpiBuilder() {
  if (!Ipi)
    Ipi = std::make_unique<TpiStreamBuilder>(*Msf, StreamIPI);
  return *Ipi;
}
```
- EN: This section centers on `PDBFileBuilder`, `initialize`, `success` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `PDBFileBuilder`, `initialize`, `success` 等符号展开，负责创建并初始化辅助对象或状态。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 81-120

```cpp

PDBStringTableBuilder &PDBFileBuilder::getStringTableBuilder() {
  return Strings;
}

GSIStreamBuilder &PDBFileBuilder::getGsiBuilder() {
  if (!Gsi)
    Gsi = std::make_unique<GSIStreamBuilder>(*Msf);
  return *Gsi;
}

Expected<uint32_t> PDBFileBuilder::allocateNamedStream(StringRef Name,
                                                       uint32_t Size) {
  auto ExpectedStream = Msf->addStream(Size);
  if (ExpectedStream)
    NamedStreams.set(Name, *ExpectedStream);
  return ExpectedStream;
}

Error PDBFileBuilder::addNamedStream(StringRef Name, StringRef Data) {
  Expected<uint32_t> ExpectedIndex = allocateNamedStream(Name, Data.size());
  if (!ExpectedIndex)
    return ExpectedIndex.takeError();
  assert(NamedStreamData.count(*ExpectedIndex) == 0);
  NamedStreamData[*ExpectedIndex] = std::string(Data);
  return Error::success();
}

void PDBFileBuilder::addInjectedSource(StringRef Name,
                                       std::unique_ptr<MemoryBuffer> Buffer) {
  // Stream names must be exact matches, since they get looked up in a hash
  // table and the hash value is dependent on the exact contents of the string.
  // link.exe lowercases a path and converts / to \, so we must do the same.
  SmallString<64> VName;
  sys::path::native(Name.lower(), VName, sys::path::Style::windows_backslash);

  uint32_t NI = getStringTableBuilder().insert(Name);
  uint32_t VNI = getStringTableBuilder().insert(VName);

  InjectedSourceDescriptor Desc;
```
- EN: This section centers on `allocateNamedStream`, `addNamedStream`, `assert` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `allocateNamedStream`, `addNamedStream`, `assert` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 121-160

```cpp
  Desc.Content = std::move(Buffer);
  Desc.NameIndex = NI;
  Desc.VNameIndex = VNI;
  Desc.StreamName = "/src/files/";

  Desc.StreamName += VName;

  InjectedSources.push_back(std::move(Desc));
}

Error PDBFileBuilder::finalizeMsfLayout() {
  llvm::TimeTraceScope timeScope("MSF layout");

  if (Ipi && Ipi->getRecordCount() > 0) {
    // In theory newer PDBs always have an ID stream, but by saying that we're
    // only going to *really* have an ID stream if there is at least one ID
    // record, we leave open the opportunity to test older PDBs such as those
    // that don't have an ID stream.
    auto &Info = getInfoBuilder();
    Info.addFeature(PdbRaw_FeatureSig::VC140);
  }

  uint32_t StringsLen = Strings.calculateSerializedSize();

  Expected<uint32_t> SN = allocateNamedStream("/LinkInfo", 0);
  if (!SN)
    return SN.takeError();

  if (Gsi) {
    if (auto EC = Gsi->finalizeMsfLayout())
      return EC;
    if (Dbi) {
      Dbi->setPublicsStreamIndex(Gsi->getPublicsStreamIndex());
      Dbi->setGlobalsStreamIndex(Gsi->getGlobalsStreamIndex());
      Dbi->setSymbolRecordStreamIndex(Gsi->getRecordStreamIndex());
    }
  }
  if (Tpi) {
    if (auto EC = Tpi->finalizeMsfLayout())
      return EC;
```
- EN: This section centers on `finalizeMsfLayout`, `timeScope` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `finalizeMsfLayout`, `timeScope` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 161-200

```cpp
  }
  if (Dbi) {
    if (auto EC = Dbi->finalizeMsfLayout())
      return EC;
  }
  SN = allocateNamedStream("/names", StringsLen);
  if (!SN)
    return SN.takeError();

  if (Ipi) {
    if (auto EC = Ipi->finalizeMsfLayout())
      return EC;
  }

  // Do this last, since it relies on the named stream map being complete, and
  // that can be updated by previous steps in the finalization.
  if (Info) {
    if (auto EC = Info->finalizeMsfLayout())
      return EC;
  }

  if (!InjectedSources.empty()) {
    for (const auto &IS : InjectedSources) {
      JamCRC CRC(0);
      CRC.update(arrayRefFromStringRef(IS.Content->getBuffer()));

      SrcHeaderBlockEntry Entry;
      ::memset(&Entry, 0, sizeof(SrcHeaderBlockEntry));
      Entry.Size = sizeof(SrcHeaderBlockEntry);
      Entry.FileSize = IS.Content->getBufferSize();
      Entry.FileNI = IS.NameIndex;
      Entry.VFileNI = IS.VNameIndex;
      Entry.ObjNI = 1;
      Entry.IsVirtual = 0;
      Entry.Version =
          static_cast<uint32_t>(PdbRaw_SrcHeaderBlockVer::SrcVerOne);
      Entry.CRC = CRC.getCRC();
      StringRef VName = getStringTableBuilder().getStringForId(IS.VNameIndex);
      InjectedSourceTable.set_as(VName, std::move(Entry),
                                 InjectedSourceHashTraits);
```
- EN: This section centers on `CRC` and performs utility computation and state updates.
  CN: 这一段主要围绕 `CRC` 等符号展开，负责执行工具性计算并更新状态。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 201-240

```cpp
    }

    uint32_t SrcHeaderBlockSize =
        sizeof(SrcHeaderBlockHeader) +
        InjectedSourceTable.calculateSerializedLength();
    SN = allocateNamedStream("/src/headerblock", SrcHeaderBlockSize);
    if (!SN)
      return SN.takeError();
    for (const auto &IS : InjectedSources) {
      SN = allocateNamedStream(IS.StreamName, IS.Content->getBufferSize());
      if (!SN)
        return SN.takeError();
    }
  }

  // Do this last, since it relies on the named stream map being complete, and
  // that can be updated by previous steps in the finalization.
  if (Info) {
    if (auto EC = Info->finalizeMsfLayout())
      return EC;
  }

  return Error::success();
}

Expected<uint32_t> PDBFileBuilder::getNamedStreamIndex(StringRef Name) const {
  uint32_t SN = 0;
  if (!NamedStreams.get(Name, SN))
    return llvm::make_error<pdb::RawError>(raw_error_code::no_stream);
  return SN;
}

void PDBFileBuilder::commitSrcHeaderBlock(WritableBinaryStream &MsfBuffer,
                                          const msf::MSFLayout &Layout) {
  assert(!InjectedSourceTable.empty());

  uint32_t SN = cantFail(getNamedStreamIndex("/src/headerblock"));
  auto Stream = WritableMappedBlockStream::createIndexedStream(
      Layout, MsfBuffer, SN, Allocator);
  BinaryStreamWriter Writer(*Stream);
```
- EN: This section centers on `success`, `getNamedStreamIndex`, `commitSrcHeaderBlock` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `success`, `getNamedStreamIndex`, `commitSrcHeaderBlock` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 241-280

```cpp

  SrcHeaderBlockHeader Header;
  ::memset(&Header, 0, sizeof(Header));
  Header.Version = static_cast<uint32_t>(PdbRaw_SrcHeaderBlockVer::SrcVerOne);
  Header.Size = Writer.bytesRemaining();

  cantFail(Writer.writeObject(Header));
  cantFail(InjectedSourceTable.commit(Writer));

  assert(Writer.bytesRemaining() == 0);
}

void PDBFileBuilder::commitInjectedSources(WritableBinaryStream &MsfBuffer,
                                           const msf::MSFLayout &Layout) {
  if (InjectedSourceTable.empty())
    return;

  llvm::TimeTraceScope timeScope("Commit injected sources");
  commitSrcHeaderBlock(MsfBuffer, Layout);

  for (const auto &IS : InjectedSources) {
    uint32_t SN = cantFail(getNamedStreamIndex(IS.StreamName));

    auto SourceStream = WritableMappedBlockStream::createIndexedStream(
        Layout, MsfBuffer, SN, Allocator);
    BinaryStreamWriter SourceWriter(*SourceStream);
    assert(SourceWriter.bytesRemaining() == IS.Content->getBufferSize());
    cantFail(SourceWriter.writeBytes(
        arrayRefFromStringRef(IS.Content->getBuffer())));
  }
}

Error PDBFileBuilder::commit(StringRef Filename, codeview::GUID *Guid) {
  assert(!Filename.empty());
  if (auto EC = finalizeMsfLayout())
    return EC;

  MSFLayout Layout;
  Expected<FileBufferByteStream> ExpectedMsfBuffer =
      Msf->commit(Filename, Layout);
```
- EN: This section centers on `cantFail`, `assert`, `commitInjectedSources` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `cantFail`, `assert`, `commitInjectedSources` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 281-320

```cpp
  if (!ExpectedMsfBuffer)
    return ExpectedMsfBuffer.takeError();
  FileBufferByteStream Buffer = std::move(*ExpectedMsfBuffer);

  auto ExpectedSN = getNamedStreamIndex("/names");
  if (!ExpectedSN)
    return ExpectedSN.takeError();

  auto NS = WritableMappedBlockStream::createIndexedStream(
      Layout, Buffer, *ExpectedSN, Allocator);
  BinaryStreamWriter NSWriter(*NS);
  if (auto EC = Strings.commit(NSWriter))
    return EC;

  {
    llvm::TimeTraceScope timeScope("Named stream data");
    for (const auto &NSE : NamedStreamData) {
      if (NSE.second.empty())
        continue;

      auto NS = WritableMappedBlockStream::createIndexedStream(
          Layout, Buffer, NSE.first, Allocator);
      BinaryStreamWriter NSW(*NS);
      if (auto EC = NSW.writeBytes(arrayRefFromStringRef(NSE.second)))
        return EC;
    }
  }

  if (Info) {
    if (auto EC = Info->commit(Layout, Buffer))
      return EC;
  }

  if (Dbi) {
    if (auto EC = Dbi->commit(Layout, Buffer))
      return EC;
  }

  if (Tpi) {
    if (auto EC = Tpi->commit(Layout, Buffer))
```
- EN: This section centers on `NSWriter`, `timeScope`, `NSW` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `NSWriter`, `timeScope`, `NSW` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 321-360

```cpp
      return EC;
  }

  if (Ipi) {
    if (auto EC = Ipi->commit(Layout, Buffer))
      return EC;
  }

  if (Gsi) {
    if (auto EC = Gsi->commit(Layout, Buffer))
      return EC;
  }

  auto InfoStreamBlocks = Layout.StreamMap[StreamPDB];
  assert(!InfoStreamBlocks.empty());
  uint64_t InfoStreamFileOffset =
      blockToOffset(InfoStreamBlocks.front(), Layout.SB->BlockSize);
  InfoStreamHeader *H = reinterpret_cast<InfoStreamHeader *>(
      Buffer.getBufferStart() + InfoStreamFileOffset);

  commitInjectedSources(Buffer, Layout);

  // Set the build id at the very end, after every other byte of the PDB
  // has been written.
  if (Info->hashPDBContentsToGUID()) {
    llvm::TimeTraceScope timeScope("Compute build ID");

    // Compute a hash of all sections of the output file.
    uint64_t Digest =
        xxh3_64bits({Buffer.getBufferStart(), Buffer.getBufferEnd()});

    H->Age = 1;

    memcpy(H->Guid.Guid, &Digest, 8);
    // xxhash only gives us 8 bytes, so put some fixed data in the other half.
    memcpy(H->Guid.Guid + 8, "LLD PDB.", 8);

    // Put the hash in the Signature field too.
    H->Signature = static_cast<uint32_t>(Digest);

```
- EN: This section centers on `assert`, `blockToOffset`, `commitInjectedSources` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `assert`, `blockToOffset`, `commitInjectedSources` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 361-371

```cpp
    // Return GUID to caller.
    memcpy(Guid, H->Guid.Guid, 16);
  } else {
    H->Age = Info->getAge();
    H->Guid = Info->getGuid();
    std::optional<uint32_t> Sig = Info->getSignature();
    H->Signature = Sig ? *Sig : time(nullptr);
  }

  return Buffer.commit();
}
```
- EN: This section centers on `memcpy` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `memcpy` 等符号展开，负责实现局部控制流程与状态维护。
- EN: This range returns the resulting value to its callers.
  CN: 这一段将结果返回给调用方。

## Key Concepts / 关键概念
- Domain / 领域: PDB debug information / PDB 调试信息
- Core symbols / 核心符号: `WritableBinaryStream`, `PDBFileBuilder`, `initialize`, `success`, `allocateNamedStream` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Helper routines coordinate local state, control flow, and result construction. / 通过辅助例程协调局部状态、控制流与结果构造。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/DebugInfo/PDB/Native/PDBFileBuilder.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/StringExtras.h`, `llvm/DebugInfo/CodeView/GUID.h`, `llvm/DebugInfo/MSF/MSFBuilder.h`, `llvm/DebugInfo/MSF/MSFCommon.h`, `llvm/DebugInfo/MSF/MappedBlockStream.h`, `llvm/DebugInfo/PDB/Native/DbiStreamBuilder.h`, `llvm/DebugInfo/PDB/Native/GSIStreamBuilder.h`, `llvm/DebugInfo/PDB/Native/InfoStreamBuilder.h`, `llvm/DebugInfo/PDB/Native/PDBStringTableBuilder.h`, `llvm/DebugInfo/PDB/Native/RawConstants.h`, `llvm/DebugInfo/PDB/Native/RawError.h`, `llvm/DebugInfo/PDB/Native/RawTypes.h`, `llvm/DebugInfo/PDB/Native/TpiStreamBuilder.h`
- Standard library / 标准库: None / 无
- Other/system headers / 其他或系统头文件: `ctime`
- Related symbols / 相关符号: `WritableBinaryStream`, `PDBFileBuilder`, `initialize`, `success`, `allocateNamedStream`, `addNamedStream`
