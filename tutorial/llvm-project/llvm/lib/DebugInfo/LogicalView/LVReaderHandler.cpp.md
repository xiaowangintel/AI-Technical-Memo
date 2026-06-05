# LVReaderHandler.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/DebugInfo/LogicalView/LVReaderHandler.cpp`
- Repository: `llvm-project`
- Purpose (EN): This class implements the Reader Handler.
- Purpose (CN): 该文件位于 LLVM 的 `DebugInfo/LogicalView` 目录中，主要实现与 `LVReaderHandler` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===-- LVReaderHandler.cpp -----------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This class implements the Reader Handler.
//
//===----------------------------------------------------------------------===//

#include "llvm/DebugInfo/LogicalView/LVReaderHandler.h"
#include "llvm/DebugInfo/LogicalView/Core/LVCompare.h"
#include "llvm/DebugInfo/LogicalView/Readers/LVCodeViewReader.h"
#include "llvm/DebugInfo/LogicalView/Readers/LVDWARFReader.h"
#include "llvm/DebugInfo/PDB/Native/NativeSession.h"
#include "llvm/DebugInfo/PDB/PDB.h"
#include "llvm/Object/COFF.h"

using namespace llvm;
using namespace llvm::object;
using namespace llvm::pdb;
using namespace llvm::logicalview;

#define DEBUG_TYPE "ReaderHandler"

Error LVReaderHandler::process() {
  if (Error Err = createReaders())
    return Err;
  if (Error Err = printReaders())
    return Err;
  if (Error Err = compareReaders())
    return Err;

  return Error::success();
}

Error LVReaderHandler::createReader(StringRef Filename, LVReaders &Readers,
                                    PdbOrObj &Input, StringRef FileFormatName,
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 7 direct dependencies, including `llvm/DebugInfo/LogicalView/LVReaderHandler.h`, `llvm/DebugInfo/LogicalView/Core/LVCompare.h`, `llvm/DebugInfo/LogicalView/Readers/LVCodeViewReader.h`, `llvm/DebugInfo/LogicalView/Readers/LVDWARFReader.h`.
  CN: 引入了 7 个直接依赖，其中包括 `llvm/DebugInfo/LogicalView/LVReaderHandler.h`, `llvm/DebugInfo/LogicalView/Core/LVCompare.h`, `llvm/DebugInfo/LogicalView/Readers/LVCodeViewReader.h`, `llvm/DebugInfo/LogicalView/Readers/LVDWARFReader.h`。
- EN: This section centers on `process`, `success` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `process`, `success` 等符号展开，负责实现局部控制流程与状态维护。

### Lines 41-80

```cpp
                                    StringRef ExePath) {
  auto CreateOneReader = [&]() -> std::unique_ptr<LVReader> {
    if (isa<ObjectFile *>(Input)) {
      ObjectFile &Obj = *cast<ObjectFile *>(Input);
      if (Obj.isCOFF()) {
        COFFObjectFile *COFF = cast<COFFObjectFile>(&Obj);
        return std::make_unique<LVCodeViewReader>(Filename, FileFormatName,
                                                  *COFF, W, ExePath);
      }
      if (Obj.isELF() || Obj.isMachO() || Obj.isWasm())
        return std::make_unique<LVDWARFReader>(Filename, FileFormatName, Obj,
                                               W);
    }
    if (isa<PDBFile *>(Input)) {
      PDBFile &Pdb = *cast<PDBFile *>(Input);
      return std::make_unique<LVCodeViewReader>(Filename, FileFormatName, Pdb,
                                                W, ExePath);
    }
    return nullptr;
  };

  std::unique_ptr<LVReader> ReaderObj = CreateOneReader();
  if (!ReaderObj)
    return createStringError(errc::invalid_argument,
                             "unable to create reader for: '%s'",
                             Filename.str().c_str());

  LVReader *Reader = ReaderObj.get();
  Readers.emplace_back(std::move(ReaderObj));
  return Reader->doLoad();
}

Error LVReaderHandler::handleArchive(LVReaders &Readers, StringRef Filename,
                                     Archive &Arch) {
  Error Err = Error::success();
  for (const Archive::Child &Child : Arch.children(Err)) {
    Expected<MemoryBufferRef> BuffOrErr = Child.getMemoryBufferRef();
    if (Error Err = BuffOrErr.takeError())
      return createStringError(errorToErrorCode(std::move(Err)), "%s",
                               Filename.str().c_str());
```
- EN: This section centers on `handleArchive` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `handleArchive` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 81-120

```cpp
    Expected<StringRef> NameOrErr = Child.getName();
    if (Error Err = NameOrErr.takeError())
      return createStringError(errorToErrorCode(std::move(Err)), "%s",
                               Filename.str().c_str());
    std::string Name = (Filename + "(" + NameOrErr.get() + ")").str();
    if (Error Err = handleBuffer(Readers, Name, BuffOrErr.get()))
      return createStringError(errorToErrorCode(std::move(Err)), "%s",
                               Filename.str().c_str());
  }

  if (Err)
    return createStringError(errorToErrorCode(std::move(Err)), "%s",
                             Filename.str().c_str());
  return Error::success();
}

// Search for a matching executable image for the given PDB path.
static std::string searchForExe(const StringRef Path,
                                const StringRef Extension) {
  SmallString<128> ExePath(Path);
  llvm::sys::path::replace_extension(ExePath, Extension);

  std::unique_ptr<IPDBSession> Session;
  if (Error Err = loadDataForEXE(PDB_ReaderType::Native, ExePath, Session)) {
    consumeError(std::move(Err));
    return {};
  }
  // We have a candidate for the executable image.
  Expected<std::string> PdbPathOrErr = NativeSession::searchForPdb({ExePath});
  if (!PdbPathOrErr) {
    consumeError(PdbPathOrErr.takeError());
    return {};
  }
  // Convert any Windows backslashes into forward slashes to get the path.
  std::string ConvertedPath = sys::path::convert_to_slash(
      PdbPathOrErr.get(), sys::path::Style::windows);
  if (ConvertedPath == Path)
    return std::string(ExePath);

  return {};
```
- EN: This section centers on `success`, `searchForExe`, `ExePath` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `success`, `searchForExe`, `ExePath` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 121-160

```cpp
}

// Search for a matching object image for the given PDB path.
static std::string searchForObj(const StringRef Path,
                                const StringRef Extension) {
  SmallString<128> ObjPath(Path);
  llvm::sys::path::replace_extension(ObjPath, Extension);
  if (llvm::sys::fs::exists(ObjPath)) {
    ErrorOr<std::unique_ptr<MemoryBuffer>> BuffOrErr =
        MemoryBuffer::getFileOrSTDIN(ObjPath);
    if (!BuffOrErr)
      return {};
    return std::string(ObjPath);
  }

  return {};
}

Error LVReaderHandler::handleBuffer(LVReaders &Readers, StringRef Filename,
                                    MemoryBufferRef Buffer, StringRef ExePath) {
  // As PDB does not support the Binary interface, at this point we can check
  // if the buffer corresponds to a PDB or PE file.
  file_magic FileMagic = identify_magic(Buffer.getBuffer());
  if (FileMagic == file_magic::pdb) {
    if (!ExePath.empty())
      return handleObject(Readers, Filename, Buffer.getBuffer(), ExePath);

    // Search in the directory derived from the given 'Filename' for a
    // matching object file (.o, .obj, .lib) or a matching executable file
    // (.exe/.dll) and try to create the reader based on the matched file.
    // If no matching file is found then we load the original PDB file.
    std::vector<StringRef> ExecutableExtensions = {"exe", "dll"};
    for (StringRef Extension : ExecutableExtensions) {
      std::string ExecutableImage = searchForExe(Filename, Extension);
      if (ExecutableImage.empty())
        continue;
      if (Error Err = handleObject(Readers, Filename, Buffer.getBuffer(),
                                   ExecutableImage)) {
        consumeError(std::move(Err));
        continue;
```
- EN: This section centers on `searchForObj`, `ObjPath`, `replace_extension` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `searchForObj`, `ObjPath`, `replace_extension` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 161-200

```cpp
      }
      return Error::success();
    }

    std::vector<StringRef> ObjectExtensions = {"o", "obj", "lib"};
    for (StringRef Extension : ObjectExtensions) {
      std::string ObjectImage = searchForObj(Filename, Extension);
      if (ObjectImage.empty())
        continue;
      if (Error Err = handleFile(Readers, ObjectImage)) {
        consumeError(std::move(Err));
        continue;
      }
      return Error::success();
    }

    // No matching executable/object image was found. Load the given PDB.
    return handleObject(Readers, Filename, Buffer.getBuffer(), ExePath);
  }
  if (FileMagic == file_magic::pecoff_executable) {
    // If we have a valid executable, try to find a matching PDB file.
    Expected<std::string> PdbPath = NativeSession::searchForPdb({Filename});
    if (errorToErrorCode(PdbPath.takeError())) {
      return createStringError(
          errc::not_supported,
          "Binary object format in '%s' does not have debug info.",
          Filename.str().c_str());
    }
    // Process the matching PDB file and pass the executable filename.
    return handleFile(Readers, PdbPath.get(), Filename);
  }

  Expected<std::unique_ptr<Binary>> BinOrErr = createBinary(Buffer);
  if (errorToErrorCode(BinOrErr.takeError())) {
    return createStringError(errc::not_supported,
                             "Binary object format in '%s' is not supported.",
                             Filename.str().c_str());
  }
  return handleObject(Readers, Filename, *BinOrErr.get());
}
```
- EN: This section centers on `success`, `consumeError`, `handleObject` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `success`, `consumeError`, `handleObject` 等符号展开，负责创建并初始化辅助对象或状态。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 201-240

```cpp

Error LVReaderHandler::handleFile(LVReaders &Readers, StringRef Filename,
                                  StringRef ExePath) {
  // Convert any Windows backslashes into forward slashes to get the path.
  std::string ConvertedPath =
      sys::path::convert_to_slash(Filename, sys::path::Style::windows);
  ErrorOr<std::unique_ptr<MemoryBuffer>> BuffOrErr =
      MemoryBuffer::getFileOrSTDIN(ConvertedPath);
  if (BuffOrErr.getError()) {
    return createStringError(errc::bad_file_descriptor,
                             "File '%s' does not exist.",
                             ConvertedPath.c_str());
  }
  std::unique_ptr<MemoryBuffer> Buffer = std::move(BuffOrErr.get());
  return handleBuffer(Readers, ConvertedPath, *Buffer, ExePath);
}

Error LVReaderHandler::handleMach(LVReaders &Readers, StringRef Filename,
                                  MachOUniversalBinary &Mach) {
  for (const MachOUniversalBinary::ObjectForArch &ObjForArch : Mach.objects()) {
    std::string ObjName = (Twine(Filename) + Twine("(") +
                           Twine(ObjForArch.getArchFlagName()) + Twine(")"))
                              .str();
    if (Expected<std::unique_ptr<MachOObjectFile>> MachOOrErr =
            ObjForArch.getAsObjectFile()) {
      MachOObjectFile &Obj = **MachOOrErr;
      PdbOrObj Input = &Obj;
      if (Error Err =
              createReader(Filename, Readers, Input, Obj.getFileFormatName()))
        return Err;
      continue;
    } else
      consumeError(MachOOrErr.takeError());
    if (Expected<std::unique_ptr<Archive>> ArchiveOrErr =
            ObjForArch.getAsArchive()) {
      if (Error Err = handleArchive(Readers, ObjName, *ArchiveOrErr.get()))
        return Err;
      continue;
    } else
      consumeError(ArchiveOrErr.takeError());
```
- EN: This section centers on `handleFile`, `convert_to_slash`, `getFileOrSTDIN` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `handleFile`, `convert_to_slash`, `getFileOrSTDIN` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 241-280

```cpp
  }
  return Error::success();
}

Error LVReaderHandler::handleObject(LVReaders &Readers, StringRef Filename,
                                    Binary &Binary) {
  if (PdbOrObj Input = dyn_cast<ObjectFile>(&Binary))
    return createReader(Filename, Readers, Input,
                        cast<ObjectFile *>(Input)->getFileFormatName());

  if (MachOUniversalBinary *Fat = dyn_cast<MachOUniversalBinary>(&Binary))
    return handleMach(Readers, Filename, *Fat);

  if (Archive *Arch = dyn_cast<Archive>(&Binary))
    return handleArchive(Readers, Filename, *Arch);

  return createStringError(errc::not_supported,
                           "Binary object format in '%s' is not supported.",
                           Filename.str().c_str());
}

Error LVReaderHandler::handleObject(LVReaders &Readers, StringRef Filename,
                                    StringRef Buffer, StringRef ExePath) {
  std::unique_ptr<IPDBSession> Session;
  if (Error Err = loadDataForPDB(PDB_ReaderType::Native, Filename, Session))
    return createStringError(errorToErrorCode(std::move(Err)), "%s",
                             Filename.str().c_str());

  std::unique_ptr<NativeSession> PdbSession;
  PdbSession.reset(static_cast<NativeSession *>(Session.release()));
  PdbOrObj Input = &PdbSession->getPDBFile();
  StringRef FileFormatName;
  size_t Pos = Buffer.find_first_of("\r\n");
  if (Pos)
    FileFormatName = Buffer.substr(0, Pos - 1);
  return createReader(Filename, Readers, Input, FileFormatName, ExePath);
}

Error LVReaderHandler::createReaders() {
  LLVM_DEBUG(dbgs() << "createReaders\n");
```
- EN: This section centers on `success`, `handleObject`, `createStringError` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `success`, `handleObject`, `createStringError` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 281-320

```cpp
  for (std::string &Object : Objects) {
    LVReaders Readers;
    if (Error Err = createReader(Object, Readers))
      return Err;
    TheReaders.insert(TheReaders.end(),
                      std::make_move_iterator(Readers.begin()),
                      std::make_move_iterator(Readers.end()));
  }

  return Error::success();
}

Error LVReaderHandler::printReaders() {
  LLVM_DEBUG(dbgs() << "printReaders\n");
  if (options().getPrintExecute())
    for (const std::unique_ptr<LVReader> &Reader : TheReaders)
      if (Error Err = Reader->doPrint())
        return Err;

  return Error::success();
}

Error LVReaderHandler::compareReaders() {
  LLVM_DEBUG(dbgs() << "compareReaders\n");
  size_t ReadersCount = TheReaders.size();
  if (options().getCompareExecute() && ReadersCount >= 2) {
    // If we have more than 2 readers, compare them by pairs.
    size_t ViewPairs = ReadersCount / 2;
    LVCompare Compare(OS);
    for (size_t Pair = 0, Index = 0; Pair < ViewPairs; ++Pair) {
      if (Error Err = Compare.execute(TheReaders[Index].get(),
                                      TheReaders[Index + 1].get()))
        return Err;
      Index += 2;
    }
  }

  return Error::success();
}

```
- EN: This section centers on `make_move_iterator`, `success`, `printReaders` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `make_move_iterator`, `success`, `printReaders` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 321-321

```cpp
void LVReaderHandler::print(raw_ostream &OS) const { OS << "ReaderHandler\n"; }
```
- EN: This section centers on `print` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `print` 等符号展开，负责为下游使用方写出、格式化或序列化结果。

## Key Concepts / 关键概念
- Domain / 领域: debug information infrastructure / 调试信息基础设施
- Core symbols / 核心符号: `implements`, `process`, `success`, `createReader`, `handleArchive` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Formatting, emission, and outward serialization. / 重点关注格式化、输出与序列化。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/DebugInfo/LogicalView/LVReaderHandler.h`, `llvm/DebugInfo/LogicalView/Core/LVCompare.h`, `llvm/DebugInfo/LogicalView/Readers/LVCodeViewReader.h`, `llvm/DebugInfo/LogicalView/Readers/LVDWARFReader.h`, `llvm/DebugInfo/PDB/Native/NativeSession.h`, `llvm/DebugInfo/PDB/PDB.h`, `llvm/Object/COFF.h`
- Standard library / 标准库: None / 无
- Other/system headers / 其他或系统头文件: None / 无
- Related symbols / 相关符号: `implements`, `process`, `success`, `createReader`, `handleArchive`, `searchForExe`
