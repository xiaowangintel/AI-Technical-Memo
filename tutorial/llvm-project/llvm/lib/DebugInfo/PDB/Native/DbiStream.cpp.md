# DbiStream.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/DebugInfo/PDB/Native/DbiStream.cpp`
- Repository: `llvm-project`
- Purpose (EN): Require at least version 7, which should be present in all PDBs produced in the last decade and allows us to avoid having to special case all kinds of complicated arcane formats.
- Purpose (CN): 该文件位于 LLVM 的 `DebugInfo/PDB/Native` 目录中，主要实现与 `DbiStream` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===- DbiStream.cpp - PDB Dbi Stream (Stream 3) Access -------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/DebugInfo/PDB/Native/DbiStream.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/DebugInfo/MSF/MappedBlockStream.h"
#include "llvm/DebugInfo/PDB/Native/ISectionContribVisitor.h"
#include "llvm/DebugInfo/PDB/Native/PDBFile.h"
#include "llvm/DebugInfo/PDB/Native/RawConstants.h"
#include "llvm/DebugInfo/PDB/Native/RawError.h"
#include "llvm/DebugInfo/PDB/Native/RawTypes.h"
#include "llvm/DebugInfo/PDB/PDBTypes.h"
#include "llvm/Object/COFF.h"
#include "llvm/Support/BinaryStreamArray.h"
#include "llvm/Support/BinaryStreamReader.h"
#include "llvm/Support/Error.h"
#include <cstddef>
#include <cstdint>

using namespace llvm;
using namespace llvm::codeview;
using namespace llvm::msf;
using namespace llvm::pdb;
using namespace llvm::support;

template <typename ContribType>
static Error loadSectionContribs(FixedStreamArray<ContribType> &Output,
                                 BinaryStreamReader &Reader) {
  if (Reader.bytesRemaining() % sizeof(ContribType) != 0)
    return make_error<RawError>(
        raw_error_code::corrupt_file,
        "Invalid number of bytes of section contributions");

  uint32_t Count = Reader.bytesRemaining() / sizeof(ContribType);
  if (auto EC = Reader.readArray(Output, Count))
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 15 direct dependencies, including `llvm/DebugInfo/PDB/Native/DbiStream.h`, `llvm/ADT/StringRef.h`, `llvm/DebugInfo/MSF/MappedBlockStream.h`, `llvm/DebugInfo/PDB/Native/ISectionContribVisitor.h`.
  CN: 引入了 15 个直接依赖，其中包括 `llvm/DebugInfo/PDB/Native/DbiStream.h`, `llvm/ADT/StringRef.h`, `llvm/DebugInfo/MSF/MappedBlockStream.h`, `llvm/DebugInfo/PDB/Native/ISectionContribVisitor.h`。
- EN: This section centers on `loadSectionContribs` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `loadSectionContribs` 等符号展开，负责实现局部控制流程与状态维护。

### Lines 41-80

```cpp
    return EC;
  return Error::success();
}

DbiStream::DbiStream(std::unique_ptr<BinaryStream> Stream)
    : Stream(std::move(Stream)), Header(nullptr) {}

DbiStream::~DbiStream() = default;

Error DbiStream::reload(PDBFile *Pdb) {
  BinaryStreamReader Reader(*Stream);

  if (Stream->getLength() < sizeof(DbiStreamHeader))
    return make_error<RawError>(raw_error_code::corrupt_file,
                                "DBI Stream does not contain a header.");
  if (auto EC = Reader.readObject(Header))
    return make_error<RawError>(raw_error_code::corrupt_file,
                                "DBI Stream does not contain a header.");

  if (Header->VersionSignature != -1)
    return make_error<RawError>(raw_error_code::corrupt_file,
                                "Invalid DBI version signature.");

  // Require at least version 7, which should be present in all PDBs
  // produced in the last decade and allows us to avoid having to
  // special case all kinds of complicated arcane formats.
  if (Header->VersionHeader < PdbDbiV70)
    return make_error<RawError>(raw_error_code::feature_unsupported,
                                "Unsupported DBI version.");

  if (Stream->getLength() !=
      sizeof(DbiStreamHeader) + Header->ModiSubstreamSize +
          Header->SecContrSubstreamSize + Header->SectionMapSize +
          Header->FileInfoSize + Header->TypeServerSize +
          Header->OptionalDbgHdrSize + Header->ECSubstreamSize)
    return make_error<RawError>(raw_error_code::corrupt_file,
                                "DBI Length does not equal sum of substreams.");

  // Only certain substreams are guaranteed to be aligned.  Validate
  // them here.
```
- EN: This section centers on `success`, `DbiStream`, `reload` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `success`, `DbiStream`, `reload` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 81-120

```cpp
  if (Header->ModiSubstreamSize % sizeof(uint32_t) != 0)
    return make_error<RawError>(raw_error_code::corrupt_file,
                                "DBI MODI substream not aligned.");
  if (Header->SecContrSubstreamSize % sizeof(uint32_t) != 0)
    return make_error<RawError>(
        raw_error_code::corrupt_file,
        "DBI section contribution substream not aligned.");
  if (Header->SectionMapSize % sizeof(uint32_t) != 0)
    return make_error<RawError>(raw_error_code::corrupt_file,
                                "DBI section map substream not aligned.");
  if (Header->FileInfoSize % sizeof(uint32_t) != 0)
    return make_error<RawError>(raw_error_code::corrupt_file,
                                "DBI file info substream not aligned.");
  if (Header->TypeServerSize % sizeof(uint32_t) != 0)
    return make_error<RawError>(raw_error_code::corrupt_file,
                                "DBI type server substream not aligned.");

  if (auto EC = Reader.readSubstream(ModiSubstream, Header->ModiSubstreamSize))
    return EC;

  if (auto EC = Reader.readSubstream(SecContrSubstream,
                                     Header->SecContrSubstreamSize))
    return EC;
  if (auto EC = Reader.readSubstream(SecMapSubstream, Header->SectionMapSize))
    return EC;
  if (auto EC = Reader.readSubstream(FileInfoSubstream, Header->FileInfoSize))
    return EC;
  if (auto EC =
          Reader.readSubstream(TypeServerMapSubstream, Header->TypeServerSize))
    return EC;
  if (auto EC = Reader.readSubstream(ECSubstream, Header->ECSubstreamSize))
    return EC;
  if (auto EC = Reader.readArray(
          DbgStreams, Header->OptionalDbgHdrSize / sizeof(ulittle16_t)))
    return EC;

  if (auto EC = Modules.initialize(ModiSubstream.StreamData,
                                   FileInfoSubstream.StreamData))
    return EC;

```
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 121-160

```cpp
  if (auto EC = initializeSectionContributionData())
    return EC;
  if (auto EC = initializeSectionHeadersData(Pdb))
    return EC;
  if (auto EC = initializeSectionMapData())
    return EC;
  if (auto EC = initializeOldFpoRecords(Pdb))
    return EC;
  if (auto EC = initializeNewFpoRecords(Pdb))
     return EC;

  if (Reader.bytesRemaining() > 0)
    return make_error<RawError>(raw_error_code::corrupt_file,
                                "Found unexpected bytes in DBI Stream.");

  if (!ECSubstream.empty()) {
    BinaryStreamReader ECReader(ECSubstream.StreamData);
    if (auto EC = ECNames.reload(ECReader))
      return EC;
  }

  return Error::success();
}

PdbRaw_DbiVer DbiStream::getDbiVersion() const {
  uint32_t Value = Header->VersionHeader;
  return static_cast<PdbRaw_DbiVer>(Value);
}

uint32_t DbiStream::getAge() const { return Header->Age; }

uint16_t DbiStream::getPublicSymbolStreamIndex() const {
  return Header->PublicSymbolStreamIndex;
}

uint16_t DbiStream::getGlobalSymbolStreamIndex() const {
  return Header->GlobalSymbolStreamIndex;
}

uint16_t DbiStream::getFlags() const { return Header->Flags; }
```
- EN: This section centers on `ECReader`, `success`, `getDbiVersion` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `ECReader`, `success`, `getDbiVersion` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 161-200

```cpp

bool DbiStream::isIncrementallyLinked() const {
  return (Header->Flags & DbiFlags::FlagIncrementalMask) != 0;
}

bool DbiStream::hasCTypes() const {
  return (Header->Flags & DbiFlags::FlagHasCTypesMask) != 0;
}

bool DbiStream::isStripped() const {
  return (Header->Flags & DbiFlags::FlagStrippedMask) != 0;
}

uint16_t DbiStream::getBuildNumber() const { return Header->BuildNumber; }

uint16_t DbiStream::getBuildMajorVersion() const {
  return (Header->BuildNumber & DbiBuildNo::BuildMajorMask) >>
         DbiBuildNo::BuildMajorShift;
}

uint16_t DbiStream::getBuildMinorVersion() const {
  return (Header->BuildNumber & DbiBuildNo::BuildMinorMask) >>
         DbiBuildNo::BuildMinorShift;
}

uint16_t DbiStream::getPdbDllRbld() const { return Header->PdbDllRbld; }

uint32_t DbiStream::getPdbDllVersion() const { return Header->PdbDllVersion; }

uint32_t DbiStream::getSymRecordStreamIndex() const {
  return Header->SymRecordStreamIndex;
}

PDB_Machine DbiStream::getMachineType() const {
  uint16_t Machine = Header->MachineType;
  return static_cast<PDB_Machine>(Machine);
}

FixedStreamArray<object::coff_section> DbiStream::getSectionHeaders() const {
  return SectionHeaders;
```
- EN: This section centers on `isIncrementallyLinked`, `hasCTypes`, `isStripped` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `isIncrementallyLinked`, `hasCTypes`, `isStripped` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: This range returns the resulting value to its callers.
  CN: 这一段将结果返回给调用方。

### Lines 201-240

```cpp
}

bool DbiStream::hasOldFpoRecords() const { return OldFpoStream != nullptr; }

FixedStreamArray<object::FpoData> DbiStream::getOldFpoRecords() const {
  return OldFpoRecords;
}

bool DbiStream::hasNewFpoRecords() const { return NewFpoStream != nullptr; }

const DebugFrameDataSubsectionRef &DbiStream::getNewFpoRecords() const {
  return NewFpoRecords;
}

const DbiModuleList &DbiStream::modules() const { return Modules; }

FixedStreamArray<SecMapEntry> DbiStream::getSectionMap() const {
  return SectionMap;
}

void DbiStream::visitSectionContributions(
    ISectionContribVisitor &Visitor) const {
  if (!SectionContribs.empty()) {
    assert(SectionContribVersion == DbiSecContribVer60);
    for (auto &SC : SectionContribs)
      Visitor.visit(SC);
  } else if (!SectionContribs2.empty()) {
    assert(SectionContribVersion == DbiSecContribV2);
    for (auto &SC : SectionContribs2)
      Visitor.visit(SC);
  }
}

Expected<StringRef> DbiStream::getECName(uint32_t NI) const {
  return ECNames.getStringForID(NI);
}

Error DbiStream::initializeSectionContributionData() {
  if (SecContrSubstream.empty())
    return Error::success();
```
- EN: This section centers on `hasOldFpoRecords`, `getOldFpoRecords`, `hasNewFpoRecords` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `hasOldFpoRecords`, `getOldFpoRecords`, `hasNewFpoRecords` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 241-280

```cpp

  BinaryStreamReader SCReader(SecContrSubstream.StreamData);
  if (auto EC = SCReader.readEnum(SectionContribVersion))
    return EC;

  if (SectionContribVersion == DbiSecContribVer60)
    return loadSectionContribs<SectionContrib>(SectionContribs, SCReader);
  if (SectionContribVersion == DbiSecContribV2)
    return loadSectionContribs<SectionContrib2>(SectionContribs2, SCReader);

  return make_error<RawError>(raw_error_code::feature_unsupported,
                              "Unsupported DBI Section Contribution version");
}

// Initializes this->SectionHeaders.
Error DbiStream::initializeSectionHeadersData(PDBFile *Pdb) {
  Expected<std::unique_ptr<msf::MappedBlockStream>> ExpectedStream =
      createIndexedStreamForHeaderType(Pdb, DbgHeaderType::SectionHdr);
  if (auto EC = ExpectedStream.takeError())
    return EC;

  auto &SHS = *ExpectedStream;
  if (!SHS)
    return Error::success();

  size_t StreamLen = SHS->getLength();
  if (StreamLen % sizeof(object::coff_section))
    return make_error<RawError>(raw_error_code::corrupt_file,
                                "Corrupted section header stream.");

  size_t NumSections = StreamLen / sizeof(object::coff_section);
  BinaryStreamReader Reader(*SHS);
  if (auto EC = Reader.readArray(SectionHeaders, NumSections))
    return make_error<RawError>(raw_error_code::corrupt_file,
                                "Could not read a bitmap.");

  SectionHeaderStream = std::move(SHS);
  return Error::success();
}

```
- EN: This section centers on `SCReader`, `initializeSectionHeadersData`, `createIndexedStreamForHeaderType` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `SCReader`, `initializeSectionHeadersData`, `createIndexedStreamForHeaderType` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 281-320

```cpp
// Initializes this->Fpos.
Error DbiStream::initializeOldFpoRecords(PDBFile *Pdb) {
  Expected<std::unique_ptr<msf::MappedBlockStream>> ExpectedStream =
      createIndexedStreamForHeaderType(Pdb, DbgHeaderType::FPO);
  if (auto EC = ExpectedStream.takeError())
    return EC;

  auto &FS = *ExpectedStream;
  if (!FS)
    return Error::success();

  size_t StreamLen = FS->getLength();
  if (StreamLen % sizeof(object::FpoData))
    return make_error<RawError>(raw_error_code::corrupt_file,
                                "Corrupted Old FPO stream.");

  size_t NumRecords = StreamLen / sizeof(object::FpoData);
  BinaryStreamReader Reader(*FS);
  if (auto EC = Reader.readArray(OldFpoRecords, NumRecords))
    return make_error<RawError>(raw_error_code::corrupt_file,
                                "Corrupted Old FPO stream.");
  OldFpoStream = std::move(FS);
  return Error::success();
}

Error DbiStream::initializeNewFpoRecords(PDBFile *Pdb) {
  Expected<std::unique_ptr<msf::MappedBlockStream>> ExpectedStream =
      createIndexedStreamForHeaderType(Pdb, DbgHeaderType::NewFPO);
  if (auto EC = ExpectedStream.takeError())
    return EC;

  auto &FS = *ExpectedStream;
  if (!FS)
    return Error::success();

  if (auto EC = NewFpoRecords.initialize(*FS))
    return EC;

  NewFpoStream = std::move(FS);
  return Error::success();
```
- EN: This section centers on `initializeOldFpoRecords`, `createIndexedStreamForHeaderType`, `Reader` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `initializeOldFpoRecords`, `createIndexedStreamForHeaderType`, `Reader` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 321-360

```cpp
}

Expected<std::unique_ptr<msf::MappedBlockStream>>
DbiStream::createIndexedStreamForHeaderType(PDBFile *Pdb,
                                            DbgHeaderType Type) const {
  if (!Pdb)
    return nullptr;

  if (DbgStreams.empty())
    return nullptr;

  uint32_t StreamNum = getDebugStreamIndex(Type);

  // This means there is no such stream.
  if (StreamNum == kInvalidStreamIndex)
    return nullptr;

  return Pdb->safelyCreateIndexedStream(StreamNum);
}

BinarySubstreamRef DbiStream::getSectionContributionData() const {
  return SecContrSubstream;
}

BinarySubstreamRef DbiStream::getSecMapSubstreamData() const {
  return SecMapSubstream;
}

BinarySubstreamRef DbiStream::getModiSubstreamData() const {
  return ModiSubstream;
}

BinarySubstreamRef DbiStream::getFileInfoSubstreamData() const {
  return FileInfoSubstream;
}

BinarySubstreamRef DbiStream::getTypeServerMapSubstreamData() const {
  return TypeServerMapSubstream;
}

```
- EN: This section centers on `createIndexedStreamForHeaderType`, `getSectionContributionData`, `getSecMapSubstreamData` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `createIndexedStreamForHeaderType`, `getSectionContributionData`, `getSecMapSubstreamData` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 361-381

```cpp
BinarySubstreamRef DbiStream::getECSubstreamData() const { return ECSubstream; }

Error DbiStream::initializeSectionMapData() {
  if (SecMapSubstream.empty())
    return Error::success();

  BinaryStreamReader SMReader(SecMapSubstream.StreamData);
  const SecMapHeader *Header;
  if (auto EC = SMReader.readObject(Header))
    return EC;
  if (auto EC = SMReader.readArray(SectionMap, Header->SecCount))
    return EC;
  return Error::success();
}

uint32_t DbiStream::getDebugStreamIndex(DbgHeaderType Type) const {
  uint16_t T = static_cast<uint16_t>(Type);
  if (T >= DbgStreams.size())
    return kInvalidStreamIndex;
  return DbgStreams[T];
}
```
- EN: This section centers on `getECSubstreamData`, `initializeSectionMapData`, `SMReader` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `getECSubstreamData`, `initializeSectionMapData`, `SMReader` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

## Key Concepts / 关键概念
- Domain / 领域: PDB debug information / PDB 调试信息
- Core symbols / 核心符号: `loadSectionContribs`, `success`, `DbiStream`, `reload` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Parsing, decoding, and structural validation. / 重点关注解析、解码与结构校验。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/DebugInfo/PDB/Native/DbiStream.h`, `llvm/ADT/StringRef.h`, `llvm/DebugInfo/MSF/MappedBlockStream.h`, `llvm/DebugInfo/PDB/Native/ISectionContribVisitor.h`, `llvm/DebugInfo/PDB/Native/PDBFile.h`, `llvm/DebugInfo/PDB/Native/RawConstants.h`, `llvm/DebugInfo/PDB/Native/RawError.h`, `llvm/DebugInfo/PDB/Native/RawTypes.h`, `llvm/DebugInfo/PDB/PDBTypes.h`, `llvm/Object/COFF.h`, `llvm/Support/BinaryStreamArray.h`, `llvm/Support/BinaryStreamReader.h`, `llvm/Support/Error.h`
- Standard library / 标准库: `cstddef`, `cstdint`
- Other/system headers / 其他或系统头文件: None / 无
- Related symbols / 相关符号: `loadSectionContribs`, `success`, `DbiStream`, `reload`, `Reader`
