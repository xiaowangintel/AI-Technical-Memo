# NativeSession.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/DebugInfo/PDB/Native/NativeSession.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements NativeSession-related logic for LLVM's DebugInfo/PDB/Native component.
- Purpose (CN): 该文件位于 LLVM 的 `DebugInfo/PDB/Native` 目录中，主要实现与 `NativeSession` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-60

```cpp
//===- NativeSession.cpp - Native implementation of IPDBSession -*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/DebugInfo/PDB/Native/NativeSession.h"

#include "llvm/ADT/SmallString.h"
#include "llvm/BinaryFormat/Magic.h"
#include "llvm/DebugInfo/MSF/MappedBlockStream.h"
#include "llvm/DebugInfo/PDB/IPDBEnumChildren.h"
#include "llvm/DebugInfo/PDB/IPDBSourceFile.h"
#include "llvm/DebugInfo/PDB/Native/DbiModuleDescriptor.h"
#include "llvm/DebugInfo/PDB/Native/DbiModuleList.h"
#include "llvm/DebugInfo/PDB/Native/DbiStream.h"
#include "llvm/DebugInfo/PDB/Native/ISectionContribVisitor.h"
#include "llvm/DebugInfo/PDB/Native/ModuleDebugStream.h"
#include "llvm/DebugInfo/PDB/Native/NativeEnumInjectedSources.h"
#include "llvm/DebugInfo/PDB/Native/NativeExeSymbol.h"
#include "llvm/DebugInfo/PDB/Native/PDBFile.h"
#include "llvm/DebugInfo/PDB/Native/RawConstants.h"
#include "llvm/DebugInfo/PDB/Native/RawError.h"
#include "llvm/DebugInfo/PDB/Native/RawTypes.h"
#include "llvm/DebugInfo/PDB/Native/SymbolCache.h"
#include "llvm/DebugInfo/PDB/PDBSymbol.h"
#include "llvm/DebugInfo/PDB/PDBSymbolCompiland.h"
#include "llvm/DebugInfo/PDB/PDBSymbolExe.h"
#include "llvm/Object/Binary.h"
#include "llvm/Object/COFF.h"
#include "llvm/Support/Allocator.h"
#include "llvm/Support/BinaryByteStream.h"
#include "llvm/Support/BinaryStreamArray.h"
#include "llvm/Support/Error.h"
#include "llvm/Support/ErrorOr.h"
#include "llvm/Support/MemoryBuffer.h"
#include "llvm/Support/Path.h"

#include <cassert>
#include <memory>
#include <utility>

using namespace llvm;
using namespace llvm::msf;
using namespace llvm::pdb;

namespace llvm {
namespace codeview {
union DebugInfo;
}
} // namespace llvm

static DbiStream *getDbiStreamPtr(PDBFile &File) {
  Expected<DbiStream &> DbiS = File.getPDBDbiStream();
  if (DbiS)
    return &DbiS.get();

  consumeError(DbiS.takeError());
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 33 direct dependencies, including `llvm/DebugInfo/PDB/Native/NativeSession.h`, `llvm/ADT/SmallString.h`, `llvm/BinaryFormat/Magic.h`, `llvm/DebugInfo/MSF/MappedBlockStream.h`.
  CN: 引入了 33 个直接依赖，其中包括 `llvm/DebugInfo/PDB/Native/NativeSession.h`, `llvm/ADT/SmallString.h`, `llvm/BinaryFormat/Magic.h`, `llvm/DebugInfo/MSF/MappedBlockStream.h`。
- EN: This section centers on `consumeError` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `consumeError` 等符号展开，负责实现局部控制流程与状态维护。

### Lines 61-120

```cpp
  return nullptr;
}

NativeSession::NativeSession(std::unique_ptr<PDBFile> PdbFile,
                             std::unique_ptr<BumpPtrAllocator> Allocator)
    : Pdb(std::move(PdbFile)), Allocator(std::move(Allocator)),
      Cache(*this, getDbiStreamPtr(*Pdb)), AddrToModuleIndex(IMapAllocator) {}

NativeSession::~NativeSession() = default;

Error NativeSession::createFromPdb(std::unique_ptr<MemoryBuffer> Buffer,
                                   std::unique_ptr<IPDBSession> &Session) {
  StringRef Path = Buffer->getBufferIdentifier();
  auto Stream = std::make_unique<MemoryBufferByteStream>(
      std::move(Buffer), llvm::endianness::little);

  auto Allocator = std::make_unique<BumpPtrAllocator>();
  auto File = std::make_unique<PDBFile>(Path, std::move(Stream), *Allocator);
  if (auto EC = File->parseFileHeaders())
    return EC;
  if (auto EC = File->parseStreamData())
    return EC;

  Session =
      std::make_unique<NativeSession>(std::move(File), std::move(Allocator));

  return Error::success();
}

static Error validatePdbMagic(StringRef PdbPath) {
  file_magic Magic;
  if (auto EC = identify_magic(PdbPath, Magic))
    return make_error<RawError>(EC);

  if (Magic != file_magic::pdb)
    return make_error<RawError>(
        raw_error_code::invalid_format,
        "The input file did not contain the pdb file magic.");

  return Error::success();
}

static Expected<std::unique_ptr<PDBFile>>
loadPdbFile(StringRef PdbPath, std::unique_ptr<BumpPtrAllocator> &Allocator) {
  ErrorOr<std::unique_ptr<MemoryBuffer>> ErrorOrBuffer =
      MemoryBuffer::getFile(PdbPath, /*IsText=*/false,
                            /*RequiresNullTerminator=*/false);
  if (!ErrorOrBuffer)
    return make_error<RawError>(ErrorOrBuffer.getError());
  std::unique_ptr<llvm::MemoryBuffer> Buffer = std::move(*ErrorOrBuffer);

  PdbPath = Buffer->getBufferIdentifier();
  if (auto EC = validatePdbMagic(PdbPath))
    return std::move(EC);

  auto Stream = std::make_unique<MemoryBufferByteStream>(
      std::move(Buffer), llvm::endianness::little);

  auto File = std::make_unique<PDBFile>(PdbPath, std::move(Stream), *Allocator);
  if (auto EC = File->parseFileHeaders())
```
- EN: This section centers on `NativeSession`, `createFromPdb`, `move` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `NativeSession`, `createFromPdb`, `move` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 121-180

```cpp
    return std::move(EC);

  if (auto EC = File->parseStreamData())
    return std::move(EC);

  return std::move(File);
}

Error NativeSession::createFromPdbPath(StringRef PdbPath,
                                       std::unique_ptr<IPDBSession> &Session) {
  auto Allocator = std::make_unique<BumpPtrAllocator>();
  auto PdbFile = loadPdbFile(PdbPath, Allocator);
  if (!PdbFile)
    return PdbFile.takeError();

  Session = std::make_unique<NativeSession>(std::move(PdbFile.get()),
                                            std::move(Allocator));
  return Error::success();
}

static Expected<std::string> getPdbPathFromExe(StringRef ExePath) {
  Expected<object::OwningBinary<object::Binary>> BinaryFile =
      object::createBinary(ExePath);
  if (!BinaryFile)
    return BinaryFile.takeError();

  const object::COFFObjectFile *ObjFile =
      dyn_cast<object::COFFObjectFile>(BinaryFile->getBinary());
  if (!ObjFile)
    return make_error<RawError>(raw_error_code::invalid_format);

  StringRef PdbPath;
  const llvm::codeview::DebugInfo *PdbInfo = nullptr;
  if (Error E = ObjFile->getDebugPDBInfo(PdbInfo, PdbPath))
    return std::move(E);

  return std::string(PdbPath);
}

Error NativeSession::createFromExe(StringRef ExePath,
                                   std::unique_ptr<IPDBSession> &Session) {
  Expected<std::string> PdbPath = getPdbPathFromExe(ExePath);
  if (!PdbPath)
    return PdbPath.takeError();

  if (auto EC = validatePdbMagic(PdbPath.get()))
    return EC;

  auto Allocator = std::make_unique<BumpPtrAllocator>();
  auto File = loadPdbFile(PdbPath.get(), Allocator);
  if (!File)
    return File.takeError();

  Session = std::make_unique<NativeSession>(std::move(File.get()),
                                            std::move(Allocator));

  return Error::success();
}

Expected<std::string>
```
- EN: This section centers on `move`, `createFromPdbPath`, `success` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `move`, `createFromPdbPath`, `success` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 181-240

```cpp
NativeSession::searchForPdb(const PdbSearchOptions &Opts) {
  Expected<std::string> PathOrErr = getPdbPathFromExe(Opts.ExePath);
  if (!PathOrErr)
    return PathOrErr.takeError();
  StringRef PathFromExe = PathOrErr.get();
  sys::path::Style Style = PathFromExe.starts_with("/")
                               ? sys::path::Style::posix
                               : sys::path::Style::windows;
  StringRef PdbName = sys::path::filename(PathFromExe, Style);

  // Check if pdb exists in the executable directory.
  SmallString<128> PdbPath = StringRef(Opts.ExePath);
  sys::path::remove_filename(PdbPath);
  sys::path::append(PdbPath, PdbName);

  auto Allocator = std::make_unique<BumpPtrAllocator>();

  if (auto File = loadPdbFile(PdbPath, Allocator))
    return std::string(PdbPath);
  else
    consumeError(File.takeError());

  // Check path that was in the executable.
  if (auto File = loadPdbFile(PathFromExe, Allocator))
    return std::string(PathFromExe);
  else
    return File.takeError();

  return make_error<RawError>("PDB not found");
}

uint64_t NativeSession::getLoadAddress() const { return LoadAddress; }

bool NativeSession::setLoadAddress(uint64_t Address) {
  LoadAddress = Address;
  return true;
}

std::unique_ptr<PDBSymbolExe> NativeSession::getGlobalScope() {
  return PDBSymbol::createAs<PDBSymbolExe>(*this, getNativeGlobalScope());
}

std::unique_ptr<PDBSymbol>
NativeSession::getSymbolById(SymIndexId SymbolId) const {
  return Cache.getSymbolById(SymbolId);
}

bool NativeSession::addressForVA(uint64_t VA, uint32_t &Section,
                                 uint32_t &Offset) const {
  uint32_t RVA = VA - getLoadAddress();
  return addressForRVA(RVA, Section, Offset);
}

bool NativeSession::addressForRVA(uint32_t RVA, uint32_t &Section,
                                  uint32_t &Offset) const {
  Section = 0;
  Offset = 0;

  auto Dbi = Pdb->getPDBDbiStream();
  if (!Dbi)
```
- EN: This section centers on `searchForPdb`, `remove_filename`, `append` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `searchForPdb`, `remove_filename`, `append` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 241-300

```cpp
    return false;

  if ((int32_t)RVA < 0)
    return true;

  Offset = RVA;
  for (; Section < Dbi->getSectionHeaders().size(); ++Section) {
    auto &Sec = Dbi->getSectionHeaders()[Section];
    if (RVA < Sec.VirtualAddress)
      return true;
    Offset = RVA - Sec.VirtualAddress;
  }
  return true;
}

std::unique_ptr<PDBSymbol>
NativeSession::findSymbolByAddress(uint64_t Address, PDB_SymType Type) {
  if (AddrToModuleIndex.empty())
    parseSectionContribs();

  return Cache.findSymbolByVA(Address, Type);
}

std::unique_ptr<PDBSymbol> NativeSession::findSymbolByRVA(uint32_t RVA,
                                                          PDB_SymType Type) {
  return findSymbolByAddress(getLoadAddress() + RVA, Type);
}

std::unique_ptr<PDBSymbol>
NativeSession::findSymbolBySectOffset(uint32_t Sect, uint32_t Offset,
                                      PDB_SymType Type) {
  return findSymbolByAddress(getVAFromSectOffset(Sect, Offset), Type);
}

std::unique_ptr<IPDBEnumLineNumbers>
NativeSession::findLineNumbers(const PDBSymbolCompiland &Compiland,
                               const IPDBSourceFile &File) const {
  return nullptr;
}

std::unique_ptr<IPDBEnumLineNumbers>
NativeSession::findLineNumbersByAddress(uint64_t Address,
                                        uint32_t Length) const {
  return Cache.findLineNumbersByVA(Address, Length);
}

std::unique_ptr<IPDBEnumLineNumbers>
NativeSession::findLineNumbersByRVA(uint32_t RVA, uint32_t Length) const {
  return Cache.findLineNumbersByVA(getLoadAddress() + RVA, Length);
}

std::unique_ptr<IPDBEnumLineNumbers>
NativeSession::findLineNumbersBySectOffset(uint32_t Section, uint32_t Offset,
                                           uint32_t Length) const {
  uint64_t VA = getVAFromSectOffset(Section, Offset);
  return Cache.findLineNumbersByVA(VA, Length);
}

std::unique_ptr<IPDBEnumSourceFiles>
NativeSession::findSourceFiles(const PDBSymbolCompiland *Compiland,
```
- EN: This section centers on `findSymbolByAddress`, `findSymbolByRVA`, `findSymbolBySectOffset` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `findSymbolByAddress`, `findSymbolByRVA`, `findSymbolBySectOffset` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 301-360

```cpp
                               StringRef Pattern,
                               PDB_NameSearchFlags Flags) const {
  return nullptr;
}

std::unique_ptr<IPDBSourceFile>
NativeSession::findOneSourceFile(const PDBSymbolCompiland *Compiland,
                                 StringRef Pattern,
                                 PDB_NameSearchFlags Flags) const {
  return nullptr;
}

std::unique_ptr<IPDBEnumChildren<PDBSymbolCompiland>>
NativeSession::findCompilandsForSourceFile(StringRef Pattern,
                                           PDB_NameSearchFlags Flags) const {
  return nullptr;
}

std::unique_ptr<PDBSymbolCompiland>
NativeSession::findOneCompilandForSourceFile(StringRef Pattern,
                                             PDB_NameSearchFlags Flags) const {
  return nullptr;
}

std::unique_ptr<IPDBEnumSourceFiles> NativeSession::getAllSourceFiles() const {
  return nullptr;
}

std::unique_ptr<IPDBEnumSourceFiles> NativeSession::getSourceFilesForCompiland(
    const PDBSymbolCompiland &Compiland) const {
  return nullptr;
}

std::unique_ptr<IPDBSourceFile>
NativeSession::getSourceFileById(uint32_t FileId) const {
  return Cache.getSourceFileById(FileId);
}

std::unique_ptr<IPDBEnumDataStreams> NativeSession::getDebugStreams() const {
  return nullptr;
}

std::unique_ptr<IPDBEnumTables> NativeSession::getEnumTables() const {
  return nullptr;
}

std::unique_ptr<IPDBEnumInjectedSources>
NativeSession::getInjectedSources() const {
  auto ISS = Pdb->getInjectedSourceStream();
  if (!ISS) {
    consumeError(ISS.takeError());
    return nullptr;
  }
  auto Strings = Pdb->getStringTable();
  if (!Strings) {
    consumeError(Strings.takeError());
    return nullptr;
  }
  return std::make_unique<NativeEnumInjectedSources>(*Pdb, *ISS, *Strings);
}
```
- EN: This section centers on `findOneSourceFile`, `findCompilandsForSourceFile`, `findOneCompilandForSourceFile` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `findOneSourceFile`, `findCompilandsForSourceFile`, `findOneCompilandForSourceFile` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 361-420

```cpp

std::unique_ptr<IPDBEnumSectionContribs>
NativeSession::getSectionContribs() const {
  return nullptr;
}

std::unique_ptr<IPDBEnumFrameData>
NativeSession::getFrameData() const {
  return nullptr;
}

void NativeSession::initializeExeSymbol() {
  if (ExeSymbol == 0)
    ExeSymbol = Cache.createSymbol<NativeExeSymbol>();
}

NativeExeSymbol &NativeSession::getNativeGlobalScope() const {
  const_cast<NativeSession &>(*this).initializeExeSymbol();

  return Cache.getNativeSymbolById<NativeExeSymbol>(ExeSymbol);
}

uint32_t NativeSession::getRVAFromSectOffset(uint32_t Section,
                                             uint32_t Offset) const {
  if (Section <= 0)
    return 0;

  auto Dbi = getDbiStreamPtr(*Pdb);
  if (!Dbi)
    return 0;

  uint32_t MaxSection = Dbi->getSectionHeaders().size();
  if (Section > MaxSection + 1)
    Section = MaxSection + 1;
  auto &Sec = Dbi->getSectionHeaders()[Section - 1];
  return Sec.VirtualAddress + Offset;
}

uint64_t NativeSession::getVAFromSectOffset(uint32_t Section,
                                            uint32_t Offset) const {
  return LoadAddress + getRVAFromSectOffset(Section, Offset);
}

bool NativeSession::moduleIndexForVA(uint64_t VA, uint16_t &ModuleIndex) const {
  ModuleIndex = 0;
  auto Iter = AddrToModuleIndex.find(VA);
  if (Iter.valid() && !IMap::KeyTraits::startLess(VA, Iter.start())) {
    ModuleIndex = Iter.value();
    return true;
  }
  return false;
}

bool NativeSession::moduleIndexForSectOffset(uint32_t Sect, uint32_t Offset,
                                             uint16_t &ModuleIndex) const {
  return moduleIndexForVA(getVAFromSectOffset(Sect, Offset), ModuleIndex);
}

#ifndef NDEBUG
void NativeSession::checkSymbolRange(uint64_t Start, uint64_t Stop) const {
```
- EN: This section centers on `getSectionContribs`, `getFrameData`, `initializeExeSymbol` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `getSectionContribs`, `getFrameData`, `initializeExeSymbol` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and returns the resulting value to its callers.
  CN: 在这一段中，代码检查条件并处理边界情况，并将结果返回给调用方。

### Lines 421-478

```cpp
  auto Iter = AddrToModuleIndex.find(Start);
  assert(Iter.valid() && !IMap::KeyTraits::startLess(Start, Iter.start()) &&
         !IMap::KeyTraits::stopLess(Iter.stop(), Stop - 1) &&
         "Symbol range is not within a single contiguous module range");
}
#endif

void NativeSession::parseSectionContribs() {
  auto Dbi = Pdb->getPDBDbiStream();
  if (!Dbi)
    return;

  class Visitor : public ISectionContribVisitor {
    NativeSession &Session;
    IMap &AddrMap;

  public:
    Visitor(NativeSession &Session, IMap &AddrMap)
        : Session(Session), AddrMap(AddrMap) {}
    void visit(const SectionContrib &C) override {
      if (C.Size == 0)
        return;

      uint64_t VA = Session.getVAFromSectOffset(C.ISect, C.Off);
      uint64_t End = VA + C.Size;

      // Ignore overlapping sections based on the assumption that a valid
      // PDB file should not have overlaps.
      if (!AddrMap.overlaps(VA, End))
        AddrMap.insert(VA, End, C.Imod);
    }
    void visit(const SectionContrib2 &C) override { visit(C.Base); }
  };

  Visitor V(*this, AddrToModuleIndex);
  Dbi->visitSectionContributions(V);
}

Expected<ModuleDebugStreamRef>
NativeSession::getModuleDebugStream(uint32_t Index) const {
  auto *Dbi = getDbiStreamPtr(*Pdb);
  assert(Dbi && "Dbi stream not present");

  DbiModuleDescriptor Modi = Dbi->modules().getModuleDescriptor(Index);

  uint16_t ModiStream = Modi.getModuleStreamIndex();
  if (ModiStream == kInvalidStreamIndex)
    return make_error<RawError>("Module stream not present");

  std::unique_ptr<msf::MappedBlockStream> ModStreamData =
      Pdb->createIndexedStream(ModiStream);

  ModuleDebugStreamRef ModS(Modi, std::move(ModStreamData));
  if (auto EC = ModS.reload())
    return std::move(EC);

  return std::move(ModS);
}
```
- EN: This section centers on `assert`, `parseSectionContribs`, `Visitor` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `assert`, `parseSectionContribs`, `Visitor` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

## Key Concepts / 关键概念
- Domain / 领域: PDB debug information / PDB 调试信息
- Core symbols / 核心符号: `Visitor`, `consumeError`, `NativeSession`, `createFromPdb`, `move` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Helper routines coordinate local state, control flow, and result construction. / 通过辅助例程协调局部状态、控制流与结果构造。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/DebugInfo/PDB/Native/NativeSession.h`, `llvm/ADT/SmallString.h`, `llvm/BinaryFormat/Magic.h`, `llvm/DebugInfo/MSF/MappedBlockStream.h`, `llvm/DebugInfo/PDB/IPDBEnumChildren.h`, `llvm/DebugInfo/PDB/IPDBSourceFile.h`, `llvm/DebugInfo/PDB/Native/DbiModuleDescriptor.h`, `llvm/DebugInfo/PDB/Native/DbiModuleList.h`, `llvm/DebugInfo/PDB/Native/DbiStream.h`, `llvm/DebugInfo/PDB/Native/ISectionContribVisitor.h`, `llvm/DebugInfo/PDB/Native/ModuleDebugStream.h`, `llvm/DebugInfo/PDB/Native/NativeEnumInjectedSources.h`, `llvm/DebugInfo/PDB/Native/NativeExeSymbol.h`, `llvm/DebugInfo/PDB/Native/PDBFile.h`, `llvm/DebugInfo/PDB/Native/RawConstants.h`
- Standard library / 标准库: `memory`, `utility`
- Other/system headers / 其他或系统头文件: `cassert`
- Related symbols / 相关符号: `Visitor`, `consumeError`, `NativeSession`, `createFromPdb`, `move`, `success`
