# Symbolize.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/DebugInfo/Symbolize/Symbolize.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implementation for LLVM symbolization library.
- Purpose (CN): 该文件位于 LLVM 的 `DebugInfo/Symbolize` 目录中，主要实现与 `Symbolize` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-60

```cpp
//===-- LLVMSymbolize.cpp -------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Implementation for LLVM symbolization library.
//
//===----------------------------------------------------------------------===//

#include "llvm/DebugInfo/Symbolize/Symbolize.h"

#include "llvm/ADT/STLExtras.h"
#include "llvm/DebugInfo/BTF/BTFContext.h"
#include "llvm/DebugInfo/DWARF/DWARFContext.h"
#include "llvm/DebugInfo/GSYM/GsymContext.h"
#include "llvm/DebugInfo/GSYM/GsymReader.h"
#include "llvm/DebugInfo/PDB/PDB.h"
#include "llvm/DebugInfo/PDB/PDBContext.h"
#include "llvm/DebugInfo/Symbolize/SymbolizableObjectFile.h"
#include "llvm/Demangle/Demangle.h"
#include "llvm/Object/Archive.h"
#include "llvm/Object/BuildID.h"
#include "llvm/Object/COFF.h"
#include "llvm/Object/ELFObjectFile.h"
#include "llvm/Object/MachO.h"
#include "llvm/Object/MachOUniversal.h"
#include "llvm/Support/CRC.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/DataExtractor.h"
#include "llvm/Support/Errc.h"
#include "llvm/Support/FileSystem.h"
#include "llvm/Support/MemoryBuffer.h"
#include "llvm/Support/Path.h"
#include <cassert>
#include <cstring>

namespace llvm {
namespace codeview {
union DebugInfo;
}
namespace symbolize {

LLVMSymbolizer::LLVMSymbolizer() = default;

LLVMSymbolizer::LLVMSymbolizer(const Options &Opts)
    : Opts(Opts),
      BIDFetcher(std::make_unique<BuildIDFetcher>(Opts.DebugFileDirectory)) {}

LLVMSymbolizer::~LLVMSymbolizer() = default;

template <typename T>
Expected<DILineInfo>
LLVMSymbolizer::symbolizeCodeCommon(const T &ModuleSpecifier,
                                    object::SectionedAddress ModuleOffset) {

  auto InfoOrErr = getOrCreateModuleInfo(ModuleSpecifier);
  if (!InfoOrErr)
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 25 direct dependencies, including `llvm/DebugInfo/Symbolize/Symbolize.h`, `llvm/ADT/STLExtras.h`, `llvm/DebugInfo/BTF/BTFContext.h`, `llvm/DebugInfo/DWARF/DWARFContext.h`.
  CN: 引入了 25 个直接依赖，其中包括 `llvm/DebugInfo/Symbolize/Symbolize.h`, `llvm/ADT/STLExtras.h`, `llvm/DebugInfo/BTF/BTFContext.h`, `llvm/DebugInfo/DWARF/DWARFContext.h`。
- EN: This section centers on `LLVMSymbolizer`, `symbolizeCodeCommon` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `LLVMSymbolizer`, `symbolizeCodeCommon` 等符号展开，负责实现局部控制流程与状态维护。

### Lines 61-120

```cpp
    return InfoOrErr.takeError();

  SymbolizableModule *Info = *InfoOrErr;

  // A null module means an error has already been reported. Return an empty
  // result.
  if (!Info)
    return DILineInfo();

  // If the user is giving us relative addresses, add the preferred base of the
  // object to the offset before we do the query. It's what DIContext expects.
  if (Opts.RelativeAddresses)
    ModuleOffset.Address += Info->getModulePreferredBase();

  DILineInfo LineInfo = Info->symbolizeCode(
      ModuleOffset,
      DILineInfoSpecifier(Opts.PathStyle, Opts.PrintFunctions,
                          Opts.SkipLineZero),
      Opts.UseSymbolTable);
  if (Opts.Demangle)
    LineInfo.FunctionName = DemangleName(LineInfo.FunctionName, Info);
  return LineInfo;
}

Expected<DILineInfo>
LLVMSymbolizer::symbolizeCode(const ObjectFile &Obj,
                              object::SectionedAddress ModuleOffset) {
  return symbolizeCodeCommon(Obj, ModuleOffset);
}

Expected<DILineInfo>
LLVMSymbolizer::symbolizeCode(StringRef ModuleName,
                              object::SectionedAddress ModuleOffset) {
  return symbolizeCodeCommon(ModuleName, ModuleOffset);
}

Expected<DILineInfo>
LLVMSymbolizer::symbolizeCode(ArrayRef<uint8_t> BuildID,
                              object::SectionedAddress ModuleOffset) {
  return symbolizeCodeCommon(BuildID, ModuleOffset);
}

template <typename T>
Expected<DIInliningInfo> LLVMSymbolizer::symbolizeInlinedCodeCommon(
    const T &ModuleSpecifier, object::SectionedAddress ModuleOffset) {
  auto InfoOrErr = getOrCreateModuleInfo(ModuleSpecifier);
  if (!InfoOrErr)
    return InfoOrErr.takeError();

  SymbolizableModule *Info = *InfoOrErr;

  // A null module means an error has already been reported. Return an empty
  // result.
  if (!Info)
    return DIInliningInfo();

  // If the user is giving us relative addresses, add the preferred base of the
  // object to the offset before we do the query. It's what DIContext expects.
  if (Opts.RelativeAddresses)
    ModuleOffset.Address += Info->getModulePreferredBase();
```
- EN: This section centers on `DILineInfoSpecifier`, `symbolizeCode`, `symbolizeCodeCommon` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `DILineInfoSpecifier`, `symbolizeCode`, `symbolizeCodeCommon` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 121-180

```cpp

  DIInliningInfo InlinedContext = Info->symbolizeInlinedCode(
      ModuleOffset,
      DILineInfoSpecifier(Opts.PathStyle, Opts.PrintFunctions,
                          Opts.SkipLineZero),
      Opts.UseSymbolTable);
  if (Opts.Demangle) {
    for (int i = 0, n = InlinedContext.getNumberOfFrames(); i < n; i++) {
      auto *Frame = InlinedContext.getMutableFrame(i);
      Frame->FunctionName = DemangleName(Frame->FunctionName, Info);
    }
  }
  return InlinedContext;
}

Expected<DIInliningInfo>
LLVMSymbolizer::symbolizeInlinedCode(const ObjectFile &Obj,
                                     object::SectionedAddress ModuleOffset) {
  return symbolizeInlinedCodeCommon(Obj, ModuleOffset);
}

Expected<DIInliningInfo>
LLVMSymbolizer::symbolizeInlinedCode(StringRef ModuleName,
                                     object::SectionedAddress ModuleOffset) {
  return symbolizeInlinedCodeCommon(ModuleName, ModuleOffset);
}

Expected<DIInliningInfo>
LLVMSymbolizer::symbolizeInlinedCode(ArrayRef<uint8_t> BuildID,
                                     object::SectionedAddress ModuleOffset) {
  return symbolizeInlinedCodeCommon(BuildID, ModuleOffset);
}

template <typename T>
Expected<DIGlobal>
LLVMSymbolizer::symbolizeDataCommon(const T &ModuleSpecifier,
                                    object::SectionedAddress ModuleOffset) {

  auto InfoOrErr = getOrCreateModuleInfo(ModuleSpecifier);
  if (!InfoOrErr)
    return InfoOrErr.takeError();

  SymbolizableModule *Info = *InfoOrErr;
  // A null module means an error has already been reported. Return an empty
  // result.
  if (!Info)
    return DIGlobal();

  // If the user is giving us relative addresses, add the preferred base of
  // the object to the offset before we do the query. It's what DIContext
  // expects.
  if (Opts.RelativeAddresses)
    ModuleOffset.Address += Info->getModulePreferredBase();

  DIGlobal Global = Info->symbolizeData(ModuleOffset);
  if (Opts.Demangle)
    Global.Name = DemangleName(Global.Name, Info);
  return Global;
}

```
- EN: This section centers on `DILineInfoSpecifier`, `symbolizeInlinedCode`, `symbolizeInlinedCodeCommon` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `DILineInfoSpecifier`, `symbolizeInlinedCode`, `symbolizeInlinedCodeCommon` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 181-240

```cpp
Expected<DIGlobal>
LLVMSymbolizer::symbolizeData(const ObjectFile &Obj,
                              object::SectionedAddress ModuleOffset) {
  return symbolizeDataCommon(Obj, ModuleOffset);
}

Expected<DIGlobal>
LLVMSymbolizer::symbolizeData(StringRef ModuleName,
                              object::SectionedAddress ModuleOffset) {
  return symbolizeDataCommon(ModuleName, ModuleOffset);
}

Expected<DIGlobal>
LLVMSymbolizer::symbolizeData(ArrayRef<uint8_t> BuildID,
                              object::SectionedAddress ModuleOffset) {
  return symbolizeDataCommon(BuildID, ModuleOffset);
}

template <typename T>
Expected<std::vector<DILocal>>
LLVMSymbolizer::symbolizeFrameCommon(const T &ModuleSpecifier,
                                     object::SectionedAddress ModuleOffset) {
  auto InfoOrErr = getOrCreateModuleInfo(ModuleSpecifier);
  if (!InfoOrErr)
    return InfoOrErr.takeError();

  SymbolizableModule *Info = *InfoOrErr;
  // A null module means an error has already been reported. Return an empty
  // result.
  if (!Info)
    return std::vector<DILocal>();

  // If the user is giving us relative addresses, add the preferred base of
  // the object to the offset before we do the query. It's what DIContext
  // expects.
  if (Opts.RelativeAddresses)
    ModuleOffset.Address += Info->getModulePreferredBase();

  return Info->symbolizeFrame(ModuleOffset);
}

Expected<std::vector<DILocal>>
LLVMSymbolizer::symbolizeFrame(const ObjectFile &Obj,
                               object::SectionedAddress ModuleOffset) {
  return symbolizeFrameCommon(Obj, ModuleOffset);
}

Expected<std::vector<DILocal>>
LLVMSymbolizer::symbolizeFrame(StringRef ModuleName,
                               object::SectionedAddress ModuleOffset) {
  return symbolizeFrameCommon(ModuleName, ModuleOffset);
}

Expected<std::vector<DILocal>>
LLVMSymbolizer::symbolizeFrame(ArrayRef<uint8_t> BuildID,
                               object::SectionedAddress ModuleOffset) {
  return symbolizeFrameCommon(BuildID, ModuleOffset);
}

template <typename T>
```
- EN: This section centers on `symbolizeData`, `symbolizeDataCommon`, `symbolizeFrameCommon` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `symbolizeData`, `symbolizeDataCommon`, `symbolizeFrameCommon` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 241-300

```cpp
Expected<std::vector<DILineInfo>>
LLVMSymbolizer::findSymbolCommon(const T &ModuleSpecifier, StringRef Symbol,
                                 uint64_t Offset) {
  auto InfoOrErr = getOrCreateModuleInfo(ModuleSpecifier);
  if (!InfoOrErr)
    return InfoOrErr.takeError();

  SymbolizableModule *Info = *InfoOrErr;
  std::vector<DILineInfo> Result;

  // A null module means an error has already been reported. Return an empty
  // result.
  if (!Info)
    return Result;

  for (object::SectionedAddress A : Info->findSymbol(Symbol, Offset)) {
    DILineInfo LineInfo = Info->symbolizeCode(
        A, DILineInfoSpecifier(Opts.PathStyle, Opts.PrintFunctions),
        Opts.UseSymbolTable);
    if (LineInfo.FileName != DILineInfo::BadString) {
      if (Opts.Demangle)
        LineInfo.FunctionName = DemangleName(LineInfo.FunctionName, Info);
      Result.push_back(std::move(LineInfo));
    }
  }

  return Result;
}

Expected<std::vector<DILineInfo>>
LLVMSymbolizer::findSymbol(const ObjectFile &Obj, StringRef Symbol,
                           uint64_t Offset) {
  return findSymbolCommon(Obj, Symbol, Offset);
}

Expected<std::vector<DILineInfo>>
LLVMSymbolizer::findSymbol(StringRef ModuleName, StringRef Symbol,
                           uint64_t Offset) {
  return findSymbolCommon(ModuleName, Symbol, Offset);
}

Expected<std::vector<DILineInfo>>
LLVMSymbolizer::findSymbol(ArrayRef<uint8_t> BuildID, StringRef Symbol,
                           uint64_t Offset) {
  return findSymbolCommon(BuildID, Symbol, Offset);
}

void LLVMSymbolizer::flush() {
  ObjectFileCache.clear();
  LRUBinaries.clear();
  CacheSize = 0;
  BinaryForPath.clear();
  ObjectPairForPathArch.clear();
  Modules.clear();
  BuildIDPaths.clear();
}

namespace {

// For Path="/path/to/foo" and Basename="foo" assume that debug info is in
```
- EN: This section centers on `findSymbolCommon`, `DILineInfoSpecifier`, `findSymbol` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `findSymbolCommon`, `DILineInfoSpecifier`, `findSymbol` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 301-360

```cpp
// /path/to/foo.dSYM/Contents/Resources/DWARF/foo.
// For Path="/path/to/bar.dSYM" and Basename="foo" assume that debug info is in
// /path/to/bar.dSYM/Contents/Resources/DWARF/foo.
std::string getDarwinDWARFResourceForPath(const std::string &Path,
                                          const std::string &Basename) {
  SmallString<16> ResourceName = StringRef(Path);
  if (sys::path::extension(Path) != ".dSYM") {
    ResourceName += ".dSYM";
  }
  sys::path::append(ResourceName, "Contents", "Resources", "DWARF");
  sys::path::append(ResourceName, Basename);
  return std::string(ResourceName);
}

bool checkFileCRC(StringRef Path, uint32_t CRCHash) {
  ErrorOr<std::unique_ptr<MemoryBuffer>> MB =
      MemoryBuffer::getFileOrSTDIN(Path);
  if (!MB)
    return false;
  return CRCHash == llvm::crc32(arrayRefFromStringRef(MB.get()->getBuffer()));
}

bool getGNUDebuglinkContents(const ObjectFile *Obj, std::string &DebugName,
                             uint32_t &CRCHash) {
  if (!Obj)
    return false;
  for (const SectionRef &Section : Obj->sections()) {
    StringRef Name;
    consumeError(Section.getName().moveInto(Name));

    Name = Name.substr(Name.find_first_not_of("._"));
    if (Name == "gnu_debuglink") {
      Expected<StringRef> ContentsOrErr = Section.getContents();
      if (!ContentsOrErr) {
        consumeError(ContentsOrErr.takeError());
        return false;
      }
      DataExtractor DE(*ContentsOrErr, Obj->isLittleEndian());
      uint64_t Offset = 0;
      if (const char *DebugNameStr = DE.getCStr(&Offset)) {
        // 4-byte align the offset.
        Offset = (Offset + 3) & ~0x3;
        if (DE.isValidOffsetForDataOfSize(Offset, 4)) {
          DebugName = DebugNameStr;
          CRCHash = DE.getU32(&Offset);
          return true;
        }
      }
      break;
    }
  }
  return false;
}

bool darwinDsymMatchesBinary(const MachOObjectFile *DbgObj,
                             const MachOObjectFile *Obj) {
  ArrayRef<uint8_t> dbg_uuid = DbgObj->getUuid();
  ArrayRef<uint8_t> bin_uuid = Obj->getUuid();
  if (dbg_uuid.empty() || bin_uuid.empty())
    return false;
```
- EN: This section centers on `getDarwinDWARFResourceForPath`, `append`, `string` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `getDarwinDWARFResourceForPath`, `append`, `string` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 361-420

```cpp
  return !memcmp(dbg_uuid.data(), bin_uuid.data(), dbg_uuid.size());
}

} // end anonymous namespace

ObjectFile *LLVMSymbolizer::lookUpDsymFile(const std::string &ExePath,
                                           const MachOObjectFile *MachExeObj,
                                           const std::string &ArchName) {
  // On Darwin we may find DWARF in separate object file in
  // resource directory.
  std::vector<std::string> DsymPaths;
  StringRef Filename = sys::path::filename(ExePath);
  DsymPaths.push_back(
      getDarwinDWARFResourceForPath(ExePath, std::string(Filename)));
  for (const auto &Path : Opts.DsymHints) {
    DsymPaths.push_back(
        getDarwinDWARFResourceForPath(Path, std::string(Filename)));
  }
  for (const auto &Path : DsymPaths) {
    auto DbgObjOrErr = getOrCreateObject(Path, ArchName);
    if (!DbgObjOrErr) {
      // Ignore errors, the file might not exist.
      consumeError(DbgObjOrErr.takeError());
      continue;
    }
    ObjectFile *DbgObj = DbgObjOrErr.get();
    if (!DbgObj)
      continue;
    const MachOObjectFile *MachDbgObj = dyn_cast<const MachOObjectFile>(DbgObj);
    if (!MachDbgObj)
      continue;
    if (darwinDsymMatchesBinary(MachDbgObj, MachExeObj))
      return DbgObj;
  }
  return nullptr;
}

ObjectFile *LLVMSymbolizer::lookUpDebuglinkObject(const std::string &Path,
                                                  const ObjectFile *Obj,
                                                  const std::string &ArchName) {
  std::string DebuglinkName;
  uint32_t CRCHash;
  std::string DebugBinaryPath;
  if (!getGNUDebuglinkContents(Obj, DebuglinkName, CRCHash))
    return nullptr;
  if (!findDebugBinary(Path, DebuglinkName, CRCHash, DebugBinaryPath))
    return nullptr;
  auto DbgObjOrErr = getOrCreateObject(DebugBinaryPath, ArchName);
  if (!DbgObjOrErr) {
    // Ignore errors, the file might not exist.
    consumeError(DbgObjOrErr.takeError());
    return nullptr;
  }
  return DbgObjOrErr.get();
}

ObjectFile *LLVMSymbolizer::lookUpBuildIDObject(const std::string &Path,
                                                const ELFObjectFileBase *Obj,
                                                const std::string &ArchName) {
  auto BuildID = getBuildID(Obj);
```
- EN: This section centers on `getDarwinDWARFResourceForPath`, `consumeError` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `getDarwinDWARFResourceForPath`, `consumeError` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 421-480

```cpp
  if (BuildID.size() < 2)
    return nullptr;
  std::string DebugBinaryPath;
  if (!getOrFindDebugBinary(BuildID, DebugBinaryPath))
    return nullptr;
  auto DbgObjOrErr = getOrCreateObject(DebugBinaryPath, ArchName);
  if (!DbgObjOrErr) {
    consumeError(DbgObjOrErr.takeError());
    return nullptr;
  }
  return DbgObjOrErr.get();
}

bool LLVMSymbolizer::findDebugBinary(const std::string &OrigPath,
                                     const std::string &DebuglinkName,
                                     uint32_t CRCHash, std::string &Result) {
  SmallString<16> OrigDir(OrigPath);
  llvm::sys::path::remove_filename(OrigDir);
  SmallString<16> DebugPath = OrigDir;
  // Try relative/path/to/original_binary/debuglink_name
  llvm::sys::path::append(DebugPath, DebuglinkName);
  if (checkFileCRC(DebugPath, CRCHash)) {
    Result = std::string(DebugPath);
    return true;
  }
  // Try relative/path/to/original_binary/.debug/debuglink_name
  DebugPath = OrigDir;
  llvm::sys::path::append(DebugPath, ".debug", DebuglinkName);
  if (checkFileCRC(DebugPath, CRCHash)) {
    Result = std::string(DebugPath);
    return true;
  }
  // Make the path absolute so that lookups will go to
  // "/usr/lib/debug/full/path/to/debug", not
  // "/usr/lib/debug/to/debug"
  llvm::sys::fs::make_absolute(OrigDir);
  if (!Opts.FallbackDebugPath.empty()) {
    // Try <FallbackDebugPath>/absolute/path/to/original_binary/debuglink_name
    DebugPath = Opts.FallbackDebugPath;
  } else {
#if defined(__NetBSD__)
    // Try /usr/libdata/debug/absolute/path/to/original_binary/debuglink_name
    DebugPath = "/usr/libdata/debug";
#else
    // Try /usr/lib/debug/absolute/path/to/original_binary/debuglink_name
    DebugPath = "/usr/lib/debug";
#endif
  }
  llvm::sys::path::append(DebugPath, llvm::sys::path::relative_path(OrigDir),
                          DebuglinkName);
  if (checkFileCRC(DebugPath, CRCHash)) {
    Result = std::string(DebugPath);
    return true;
  }
  return false;
}

static StringRef getBuildIDStr(ArrayRef<uint8_t> BuildID) {
  return StringRef(reinterpret_cast<const char *>(BuildID.data()),
                   BuildID.size());
```
- EN: This section centers on `consumeError`, `findDebugBinary`, `OrigDir` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `consumeError`, `findDebugBinary`, `OrigDir` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 481-540

```cpp
}

bool LLVMSymbolizer::getOrFindDebugBinary(const ArrayRef<uint8_t> BuildID,
                                          std::string &Result) {
  StringRef BuildIDStr = getBuildIDStr(BuildID);
  auto I = BuildIDPaths.find(BuildIDStr);
  if (I != BuildIDPaths.end()) {
    Result = I->second;
    return true;
  }
  if (!BIDFetcher)
    return false;
  if (std::optional<std::string> Path = BIDFetcher->fetch(BuildID)) {
    Result = *Path;
    auto InsertResult = BuildIDPaths.insert({BuildIDStr, Result});
    assert(InsertResult.second);
    (void)InsertResult;
    return true;
  }

  return false;
}

std::string LLVMSymbolizer::lookUpGsymFile(const std::string &Path) {
  if (Opts.DisableGsym)
    return {};

  auto CheckGsymFile = [](const llvm::StringRef &GsymPath) {
    sys::fs::file_status Status;
    std::error_code EC = llvm::sys::fs::status(GsymPath, Status);
    return !EC && !llvm::sys::fs::is_directory(Status);
  };

  // First, look beside the binary file
  if (const auto GsymPath = Path + ".gsym"; CheckGsymFile(GsymPath))
    return GsymPath;

  // Then, look in the directories specified by GsymFileDirectory

  for (const auto &Directory : Opts.GsymFileDirectory) {
    SmallString<16> GsymPath = llvm::StringRef{Directory};
    llvm::sys::path::append(GsymPath,
                            llvm::sys::path::filename(Path) + ".gsym");

    if (CheckGsymFile(GsymPath))
      return static_cast<std::string>(GsymPath);
  }

  return {};
}

Expected<LLVMSymbolizer::ObjectPair>
LLVMSymbolizer::getOrCreateObjectPair(const std::string &Path,
                                      const std::string &ArchName) {
  auto I = ObjectPairForPathArch.find(std::make_pair(Path, ArchName));
  if (I != ObjectPairForPathArch.end()) {
    recordAccess(BinaryForPath.find(Path)->second);
    return I->second;
  }

```
- EN: This section centers on `getOrFindDebugBinary`, `assert`, `lookUpGsymFile` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `getOrFindDebugBinary`, `assert`, `lookUpGsymFile` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 541-600

```cpp
  auto ObjOrErr = getOrCreateObject(Path, ArchName);
  if (!ObjOrErr) {
    ObjectPairForPathArch.emplace(std::make_pair(Path, ArchName),
                                  ObjectPair(nullptr, nullptr));
    return ObjOrErr.takeError();
  }

  ObjectFile *Obj = ObjOrErr.get();
  assert(Obj != nullptr);
  ObjectFile *DbgObj = nullptr;

  if (auto MachObj = dyn_cast<const MachOObjectFile>(Obj))
    DbgObj = lookUpDsymFile(Path, MachObj, ArchName);
  else if (auto ELFObj = dyn_cast<const ELFObjectFileBase>(Obj))
    DbgObj = lookUpBuildIDObject(Path, ELFObj, ArchName);
  if (!DbgObj)
    DbgObj = lookUpDebuglinkObject(Path, Obj, ArchName);
  if (!DbgObj)
    DbgObj = Obj;
  ObjectPair Res = std::make_pair(Obj, DbgObj);
  auto Pair =
      ObjectPairForPathArch.emplace(std::make_pair(Path, ArchName), Res);
  std::string FullDbgObjKey;
  auto It = ObjectToArchivePath.find(DbgObj);
  if (It != ObjectToArchivePath.end()) {
    StringRef ArchivePath = It->second;
    StringRef MemberName = sys::path::filename(DbgObj->getFileName());
    FullDbgObjKey = (ArchivePath + "(" + MemberName + ")").str();
  } else {
    FullDbgObjKey = DbgObj->getFileName().str();
  }
  BinaryForPath.find(FullDbgObjKey)
      ->second.pushEvictor(
          [this, I = Pair.first]() { ObjectPairForPathArch.erase(I); });
  return Res;
}

Expected<object::Binary *>
LLVMSymbolizer::loadOrGetBinary(const std::string &ArchivePathKey,
                                std::optional<StringRef> FullPathKey) {
  // If no separate cache key is provided, use the archive path itself.
  std::string FullPathKeyStr =
      FullPathKey ? FullPathKey->str() : ArchivePathKey;
  auto Pair = BinaryForPath.emplace(FullPathKeyStr, OwningBinary<Binary>());
  if (!Pair.second) {
    recordAccess(Pair.first->second);
    return Pair.first->second->getBinary();
  }

  Expected<OwningBinary<Binary>> BinOrErr = createBinary(ArchivePathKey);
  if (!BinOrErr)
    return BinOrErr.takeError();

  CachedBinary &CachedBin = Pair.first->second;
  CachedBin = std::move(*BinOrErr);
  CachedBin.pushEvictor([this, I = Pair.first]() { BinaryForPath.erase(I); });
  LRUBinaries.push_back(CachedBin);
  CacheSize += CachedBin.size();
  return CachedBin->getBinary();
}
```
- EN: This section centers on `ObjectPair`, `assert`, `loadOrGetBinary` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `ObjectPair`, `assert`, `loadOrGetBinary` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 601-660

```cpp

Expected<ObjectFile *> LLVMSymbolizer::findOrCacheObject(
    const ContainerCacheKey &Key,
    llvm::function_ref<Expected<std::unique_ptr<ObjectFile>>()> Loader,
    const std::string &PathForBinaryCache) {
  auto It = ObjectFileCache.find(Key);
  if (It != ObjectFileCache.end())
    return It->second.get();

  Expected<std::unique_ptr<ObjectFile>> ObjOrErr = Loader();
  if (!ObjOrErr) {
    ObjectFileCache.emplace(Key, std::unique_ptr<ObjectFile>());
    return ObjOrErr.takeError();
  }

  ObjectFile *Res = ObjOrErr->get();
  auto NewEntry = ObjectFileCache.emplace(Key, std::move(*ObjOrErr));
  auto CacheIter = BinaryForPath.find(PathForBinaryCache);
  if (CacheIter != BinaryForPath.end())
    CacheIter->second.pushEvictor(
        [this, Iter = NewEntry.first]() { ObjectFileCache.erase(Iter); });
  return Res;
}

Expected<ObjectFile *> LLVMSymbolizer::getOrCreateObjectFromArchive(
    StringRef ArchivePath, StringRef MemberName, StringRef ArchName,
    StringRef FullPath) {
  Expected<object::Binary *> BinOrErr =
      loadOrGetBinary(ArchivePath.str(), FullPath);
  if (!BinOrErr)
    return BinOrErr.takeError();
  object::Binary *Bin = *BinOrErr;

  object::Archive *Archive = dyn_cast_if_present<object::Archive>(Bin);
  if (!Archive)
    return createStringError(std::errc::invalid_argument,
                             "'%s' is not a valid archive",
                             ArchivePath.str().c_str());

  Error Err = Error::success();
  for (auto &Child : Archive->children(Err, /*SkipInternal=*/true)) {
    Expected<StringRef> NameOrErr = Child.getName();
    if (!NameOrErr) {
      // TODO: Report this as a warning to the client. Consider adding a
      // callback mechanism to report warning-level issues.
      consumeError(NameOrErr.takeError());
      continue;
    }
    if (*NameOrErr == MemberName) {
      Expected<std::unique_ptr<object::Binary>> MemberOrErr =
          Child.getAsBinary();
      if (!MemberOrErr) {
        // TODO: Report this as a warning to the client. Consider adding a
        // callback mechanism to report warning-level issues.
        consumeError(MemberOrErr.takeError());
        continue;
      }

      std::unique_ptr<object::Binary> Binary = std::move(*MemberOrErr);
      if (auto *Obj = dyn_cast<object::ObjectFile>(Binary.get())) {
```
- EN: This section centers on `findOrCacheObject`, `getOrCreateObjectFromArchive`, `loadOrGetBinary` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `findOrCacheObject`, `getOrCreateObjectFromArchive`, `loadOrGetBinary` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 661-720

```cpp
        ObjectToArchivePath[Obj] = ArchivePath.str();
        Triple::ArchType ObjArch = Obj->makeTriple().getArch();
        Triple RequestedTriple;
        RequestedTriple.setArch(Triple::getArchTypeForLLVMName(ArchName));
        if (ObjArch != RequestedTriple.getArch())
          continue;

        ContainerCacheKey CacheKey{ArchivePath.str(), MemberName.str(),
                                   ArchName.str()};
        Expected<ObjectFile *> Res = findOrCacheObject(
            CacheKey,
            [O = std::unique_ptr<ObjectFile>(
                 Obj)]() mutable -> Expected<std::unique_ptr<ObjectFile>> {
              return std::move(O);
            },
            ArchivePath.str());
        Binary.release();
        return Res;
      }
    }
  }
  if (Err)
    return std::move(Err);
  return createStringError(std::errc::invalid_argument,
                           "no matching member '%s' with arch '%s' in '%s'",
                           MemberName.str().c_str(), ArchName.str().c_str(),
                           ArchivePath.str().c_str());
}

Expected<ObjectFile *>
LLVMSymbolizer::getOrCreateObject(const std::string &Path,
                                  const std::string &ArchName) {
  // First check for archive(member) format - more efficient to check closing
  // paren first.
  if (!Path.empty() && Path.back() == ')') {
    size_t OpenParen = Path.rfind('(', Path.size() - 1);
    if (OpenParen != std::string::npos) {
      StringRef ArchivePath = StringRef(Path).substr(0, OpenParen);
      StringRef MemberName =
          StringRef(Path).substr(OpenParen + 1, Path.size() - OpenParen - 2);
      return getOrCreateObjectFromArchive(ArchivePath, MemberName, ArchName,
                                          Path);
    }
  }

  Expected<object::Binary *> BinOrErr = loadOrGetBinary(Path);
  if (!BinOrErr)
    return BinOrErr.takeError();
  object::Binary *Bin = *BinOrErr;

  if (MachOUniversalBinary *UB = dyn_cast_or_null<MachOUniversalBinary>(Bin)) {
    ContainerCacheKey CacheKey{Path, "", ArchName};
    return findOrCacheObject(
        CacheKey,
        [UB, ArchName]() -> Expected<std::unique_ptr<ObjectFile>> {
          return UB->getMachOObjectForArch(ArchName);
        },
        Path);
  }
  if (Bin->isObject()) {
```
- EN: This section centers on `move`, `createStringError`, `getOrCreateObject` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `move`, `createStringError`, `getOrCreateObject` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 721-780

```cpp
    return cast<ObjectFile>(Bin);
  }
  return errorCodeToError(object_error::arch_not_found);
}

Expected<SymbolizableModule *>
LLVMSymbolizer::createModuleInfo(const ObjectFile *Obj,
                                 std::unique_ptr<DIContext> Context,
                                 StringRef ModuleName) {
  auto InfoOrErr = SymbolizableObjectFile::create(Obj, std::move(Context),
                                                  Opts.UntagAddresses);
  std::unique_ptr<SymbolizableModule> SymMod;
  if (InfoOrErr)
    SymMod = std::move(*InfoOrErr);
  auto InsertResult = Modules.insert(
      std::make_pair(std::string(ModuleName), std::move(SymMod)));
  assert(InsertResult.second);
  if (!InfoOrErr)
    return InfoOrErr.takeError();
  return InsertResult.first->second.get();
}

Expected<SymbolizableModule *>
LLVMSymbolizer::getOrCreateModuleInfo(StringRef ModuleName) {
  StringRef BinaryName = ModuleName;
  StringRef ArchName = Opts.DefaultArch;
  size_t ColonPos = ModuleName.find_last_of(':');
  // Verify that substring after colon form a valid arch name.
  if (ColonPos != std::string::npos) {
    StringRef ArchStr = ModuleName.substr(ColonPos + 1);
    if (Triple(ArchStr).getArch() != Triple::UnknownArch) {
      BinaryName = ModuleName.substr(0, ColonPos);
      ArchName = ArchStr;
    }
  }

  auto I = Modules.find(ModuleName);
  if (I != Modules.end()) {
    recordAccess(BinaryForPath.find(BinaryName)->second);
    return I->second.get();
  }

  auto ObjectsOrErr =
      getOrCreateObjectPair(std::string{BinaryName}, std::string{ArchName});
  if (!ObjectsOrErr) {
    // Failed to find valid object file.
    Modules.emplace(ModuleName, std::unique_ptr<SymbolizableModule>());
    return ObjectsOrErr.takeError();
  }
  ObjectPair Objects = ObjectsOrErr.get();

  std::unique_ptr<DIContext> Context;
  // If this is a COFF object containing PDB info and not containing DWARF
  // section, use a PDBContext to symbolize. Otherwise, use DWARF.
  // Create a DIContext to symbolize as follows:
  // - If there is a GSYM file, create a GsymContext.
  // - Otherwise, if this is a COFF object containing PDB info, create a
  // PDBContext.
  // - Otherwise, create a DWARFContext.
  const auto GsymFile = lookUpGsymFile(BinaryName.str());
```
- EN: This section centers on `errorCodeToError`, `createModuleInfo`, `make_pair` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `errorCodeToError`, `createModuleInfo`, `make_pair` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 781-840

```cpp
  if (!GsymFile.empty()) {
    auto ReaderOrErr = gsym::GsymReader::openFile(GsymFile);

    if (ReaderOrErr)
      Context = std::make_unique<gsym::GsymContext>(std::move(*ReaderOrErr));
  }
  if (!Context) {
    if (auto CoffObject = dyn_cast<COFFObjectFile>(Objects.first)) {
      const codeview::DebugInfo *DebugInfo;
      StringRef PDBFileName;
      auto EC = CoffObject->getDebugPDBInfo(DebugInfo, PDBFileName);
      // Use DWARF if there're DWARF sections.
      bool HasDwarf = llvm::any_of(
          Objects.first->sections(), [](SectionRef Section) -> bool {
            if (Expected<StringRef> SectionName = Section.getName())
              return SectionName.get() == ".debug_info";
            return false;
          });
      if (!EC && !HasDwarf && DebugInfo != nullptr && !PDBFileName.empty()) {
        using namespace pdb;
        std::unique_ptr<IPDBSession> Session;

        PDB_ReaderType ReaderType =
            Opts.UseDIA ? PDB_ReaderType::DIA : PDB_ReaderType::Native;
        if (auto Err = loadDataForEXE(ReaderType, Objects.first->getFileName(),
                                      Session)) {
          Modules.emplace(ModuleName, std::unique_ptr<SymbolizableModule>());
          // Return along the PDB filename to provide more context
          return createFileError(PDBFileName, std::move(Err));
        }
        Context.reset(new PDBContext(*CoffObject, std::move(Session)));
      }
    }
  }
  if (!Context)
    Context = DWARFContext::create(
        *Objects.second, DWARFContext::ProcessDebugRelocations::Process,
        nullptr, Opts.DWPName);
  auto ModuleOrErr =
      createModuleInfo(Objects.first, std::move(Context), ModuleName);
  if (ModuleOrErr) {
    auto I = Modules.find(ModuleName);
    BinaryForPath.find(BinaryName)->second.pushEvictor([this, I]() {
      Modules.erase(I);
    });
  }
  return ModuleOrErr;
}

// For BPF programs .BTF.ext section contains line numbers information,
// use it if regular DWARF is not available (e.g. for stripped binary).
static bool useBTFContext(const ObjectFile &Obj) {
  return Obj.makeTriple().isBPF() && !Obj.hasDebugInfo() &&
         BTFParser::hasBTFSections(Obj);
}

Expected<SymbolizableModule *>
LLVMSymbolizer::getOrCreateModuleInfo(const ObjectFile &Obj) {
  StringRef ObjName = Obj.getFileName();
  auto I = Modules.find(ObjName);
```
- EN: This section centers on `createFileError`, `createModuleInfo`, `useBTFContext` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `createFileError`, `createModuleInfo`, `useBTFContext` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 841-900

```cpp
  if (I != Modules.end())
    return I->second.get();

  std::unique_ptr<DIContext> Context;
  if (useBTFContext(Obj))
    Context = BTFContext::create(Obj);
  else
    Context = DWARFContext::create(Obj);
  // FIXME: handle COFF object with PDB info to use PDBContext
  return createModuleInfo(&Obj, std::move(Context), ObjName);
}

Expected<SymbolizableModule *>
LLVMSymbolizer::getOrCreateModuleInfo(ArrayRef<uint8_t> BuildID) {
  std::string Path;
  if (!getOrFindDebugBinary(BuildID, Path)) {
    return createStringError(errc::no_such_file_or_directory,
                             "could not find build ID");
  }
  return getOrCreateModuleInfo(Path);
}

namespace {

// Undo these various manglings for Win32 extern "C" functions:
// cdecl       - _foo
// stdcall     - _foo@12
// fastcall    - @foo@12
// vectorcall  - foo@@12
// These are all different linkage names for 'foo'.
StringRef demanglePE32ExternCFunc(StringRef SymbolName) {
  char Front = SymbolName.empty() ? '\0' : SymbolName[0];

  // Remove any '@[0-9]+' suffix.
  bool HasAtNumSuffix = false;
  if (Front != '?') {
    size_t AtPos = SymbolName.rfind('@');
    if (AtPos != StringRef::npos &&
        all_of(drop_begin(SymbolName, AtPos + 1), isDigit)) {
      SymbolName = SymbolName.substr(0, AtPos);
      HasAtNumSuffix = true;
    }
  }

  // Remove any ending '@' for vectorcall.
  bool IsVectorCall = false;
  if (HasAtNumSuffix && SymbolName.ends_with("@")) {
    SymbolName = SymbolName.drop_back();
    IsVectorCall = true;
  }

  // If not vectorcall, remove any '_' or '@' prefix.
  if (!IsVectorCall && (Front == '_' || Front == '@'))
    SymbolName = SymbolName.drop_front();

  return SymbolName;
}

} // end anonymous namespace

```
- EN: This section centers on `createModuleInfo`, `getOrCreateModuleInfo`, `createStringError` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `createModuleInfo`, `getOrCreateModuleInfo`, `createStringError` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 901-960

```cpp
std::string
LLVMSymbolizer::DemangleName(StringRef Name,
                             const SymbolizableModule *DbiModuleDescriptor) {
  std::string Result;
  if (nonMicrosoftDemangle(Name, Result))
    return Result;

  if (Name.starts_with('?')) {
    // Only do MSVC C++ demangling on symbols starting with '?'.
    int status = 0;
    char *DemangledName = microsoftDemangle(
        Name, nullptr, &status,
        MSDemangleFlags(MSDF_NoAccessSpecifier | MSDF_NoCallingConvention |
                        MSDF_NoMemberType | MSDF_NoReturnType));
    if (status != 0)
      return std::string{Name};
    Result = DemangledName;
    free(DemangledName);
    return Result;
  }

  if (DbiModuleDescriptor && DbiModuleDescriptor->isWin32Module()) {
    std::string DemangledCName(demanglePE32ExternCFunc(Name));
    // On i386 Windows, the C name mangling for different calling conventions
    // may also be applied on top of the Itanium or Rust name mangling.
    if (nonMicrosoftDemangle(DemangledCName, Result))
      return Result;
    return DemangledCName;
  }
  return std::string{Name};
}

void LLVMSymbolizer::recordAccess(CachedBinary &Bin) {
  if (Bin->getBinary())
    LRUBinaries.splice(LRUBinaries.end(), LRUBinaries, Bin.getIterator());
}

void LLVMSymbolizer::pruneCache() {
  // Evict the LRU binary until the max cache size is reached or there's <= 1
  // item in the cache. The MRU binary is always kept to avoid thrashing if it's
  // larger than the cache size.
  while (CacheSize > Opts.MaxCacheSize && !LRUBinaries.empty() &&
         std::next(LRUBinaries.begin()) != LRUBinaries.end()) {
    CachedBinary &Bin = LRUBinaries.front();
    CacheSize -= Bin.size();
    LRUBinaries.pop_front();
    Bin.evict();
  }
}

void CachedBinary::pushEvictor(std::function<void()> NewEvictor) {
  if (Evictor) {
    this->Evictor = [OldEvictor = std::move(this->Evictor),
                     NewEvictor = std::move(NewEvictor)]() {
      NewEvictor();
      OldEvictor();
    };
  } else {
    this->Evictor = std::move(NewEvictor);
  }
```
- EN: This section centers on `DemangleName`, `MSDemangleFlags`, `free` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `DemangleName`, `MSDemangleFlags`, `free` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 961-964

```cpp
}

} // namespace symbolize
} // namespace llvm
```
- EN: This range contains supporting statements, comments, or structural glue code.
  CN: 这一段主要包含辅助语句、注释或结构性胶水代码。

## Key Concepts / 关键概念
- Domain / 领域: debug information infrastructure / 调试信息基础设施
- Core symbols / 核心符号: `LLVMSymbolizer`, `symbolizeCodeCommon`, `DILineInfoSpecifier`, `symbolizeCode` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Formatting, emission, and outward serialization. / 重点关注格式化、输出与序列化。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/DebugInfo/Symbolize/Symbolize.h`, `llvm/ADT/STLExtras.h`, `llvm/DebugInfo/BTF/BTFContext.h`, `llvm/DebugInfo/DWARF/DWARFContext.h`, `llvm/DebugInfo/GSYM/GsymContext.h`, `llvm/DebugInfo/GSYM/GsymReader.h`, `llvm/DebugInfo/PDB/PDB.h`, `llvm/DebugInfo/PDB/PDBContext.h`, `llvm/DebugInfo/Symbolize/SymbolizableObjectFile.h`, `llvm/Demangle/Demangle.h`, `llvm/Object/Archive.h`, `llvm/Object/BuildID.h`, `llvm/Object/COFF.h`, `llvm/Object/ELFObjectFile.h`, `llvm/Object/MachO.h`
- Standard library / 标准库: `cstring`
- Other/system headers / 其他或系统头文件: `cassert`
- Related symbols / 相关符号: `LLVMSymbolizer`, `symbolizeCodeCommon`, `DILineInfoSpecifier`, `symbolizeCode`, `symbolizeInlinedCodeCommon`
