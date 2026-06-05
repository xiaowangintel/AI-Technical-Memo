# LibraryScanner.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/Orc/TargetProcess/LibraryScanner.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements Provide Library Scanning Implementation.
  - **CN**: 实现 ORC JIT 基础设施，例如 ExecutionSession、JITDylib、物化流程、符号查找以及执行辅助组件。

## Line-by-Line Analysis / 逐行分析

### Lines 1-27
```cpp
//===- LibraryScanner.cpp - Provide Library Scanning Implementation ----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/ExecutionEngine/Orc/TargetProcess/LibraryScanner.h"
#include "llvm/ExecutionEngine/Orc/TargetProcess/LibraryResolver.h"

#include "llvm/ADT/StringExtras.h"
#include "llvm/Object/COFF.h"
#include "llvm/Object/ELF.h"
#include "llvm/Object/ELFObjectFile.h"
#include "llvm/Object/ELFTypes.h"
#include "llvm/Object/MachO.h"
#include "llvm/Object/MachOUniversal.h"
#include "llvm/Object/ObjectFile.h"
#include "llvm/Support/Error.h"
#include "llvm/Support/FileSystem.h"
#include "llvm/Support/MemoryBuffer.h"
#include "llvm/Support/Path.h"
#include "llvm/Support/Program.h"
#include "llvm/TargetParser/Host.h"
#include "llvm/TargetParser/Triple.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ExecutionEngine/Orc/TargetProcess/LibraryScanner.h`, `llvm/ExecutionEngine/Orc/TargetProcess/LibraryResolver.h`, `llvm/ADT/StringExtras.h`, `llvm/Object/COFF.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ExecutionEngine/Orc/TargetProcess/LibraryScanner.h`, `llvm/ExecutionEngine/Orc/TargetProcess/LibraryResolver.h`, `llvm/ADT/StringExtras.h`, `llvm/Object/COFF.h`。

### Lines 28-51
```cpp
#ifdef LLVM_ON_UNIX
#include <sys/stat.h>
#include <unistd.h>
#endif // LLVM_ON_UNIX

#ifdef __APPLE__
#include <sys/stat.h>
#undef LC_LOAD_DYLIB
#undef LC_RPATH
#endif // __APPLE__

#define DEBUG_TYPE "orc-scanner"

namespace llvm::orc {

void handleError(Error Err, StringRef context = "") {
  consumeError(handleErrors(std::move(Err), [&](const ErrorInfoBase &EIB) {
    dbgs() << "LLVM Error";
    if (!context.empty())
      dbgs() << " [" << context << "]";
    dbgs() << ": " << EIB.message() << "\n";
  }));
}

```
- **EN**: Pulls in the headers needed for this implementation, including `sys/stat.h`, `unistd.h`, `sys/stat.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `sys/stat.h`, `unistd.h`, `sys/stat.h`。

### Lines 52-69
```cpp
bool ObjectFileLoader::isArchitectureCompatible(const object::ObjectFile &Obj) {
  static const llvm::Triple HostTriple(llvm::sys::getProcessTriple());

  if (HostTriple.getArch() != Obj.getArch())
    return false;

  if (Obj.getTripleObjectFormat() != HostTriple.getObjectFormat())
    return false;

  return true;
}

Expected<object::OwningBinary<object::ObjectFile>>
ObjectFileLoader::loadObjectFileWithOwnership(StringRef FilePath) {
  LLVM_DEBUG(dbgs() << "ObjectFileLoader: Attempting to open file " << FilePath
                    << "\n";);
  if (auto ObjOrErr = object::ObjectFile::createObjectFile(FilePath)) {

```
- **EN**: Implements logic around `isArchitectureCompatible`, `HostTriple`, `loadObjectFileWithOwnership`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `isArchitectureCompatible`, `HostTriple`, `loadObjectFileWithOwnership` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 70-94
```cpp
    LLVM_DEBUG(dbgs() << "ObjectFileLoader: Detected object file\n";);

    auto OwningBin = std::move(*ObjOrErr);

    if (!isArchitectureCompatible(*OwningBin.getBinary())) {
      LLVM_DEBUG(dbgs() << "ObjectFileLoader: Incompatible architecture: "
                        << FilePath << "\n";);
      return createStringError(inconvertibleErrorCode(),
                               "Incompatible object file: %s",
                               FilePath.str().c_str());
    }

    LLVM_DEBUG(dbgs() << "ObjectFileLoader: Object file is compatible\n";);

    return std::move(OwningBin);
  } else {
#if defined(__APPLE__)
    consumeError(ObjOrErr.takeError());
    auto BinOrErr = object::createBinary(FilePath);
    if (!BinOrErr) {
      LLVM_DEBUG(dbgs() << "ObjectFileLoader: Failed to open file " << FilePath
                        << "\n";);
      return BinOrErr.takeError();
    }

```
- **EN**: Implements logic around `move`, `createStringError`, `str`, `consumeError`, and 2 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `move`, `createStringError`, `str`, `consumeError`, and 2 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 95-119
```cpp
    LLVM_DEBUG(dbgs() << "ObjectFileLoader: Successfully opened file "
                      << FilePath << "\n";);

    auto OwningBin = BinOrErr->takeBinary();
    object::Binary *Bin = OwningBin.first.get();

    if (Bin->isArchive()) {
      LLVM_DEBUG(
          dbgs() << "ObjectFileLoader: File is an archive, not supported: "
                 << FilePath << "\n";);
      return createStringError(std::errc::invalid_argument,
                               "Archive files are not supported: %s",
                               FilePath.str().c_str());
    }

    if (auto *UB = dyn_cast<object::MachOUniversalBinary>(Bin)) {
      LLVM_DEBUG(
          dbgs() << "ObjectFileLoader: Detected Mach-O universal binary: "
                 << FilePath << "\n";);
      for (auto ObjForArch : UB->objects()) {
        auto ObjOrErr = ObjForArch.getAsObjectFile();
        if (!ObjOrErr) {
          LLVM_DEBUG(dbgs() << "ObjectFileLoader: Skipping invalid "
                               "architecture slice\n";);

```
- **EN**: Implements logic around `takeBinary`, `get`, `dbgs`, `createStringError`, and 2 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `takeBinary`, `get`, `dbgs`, `createStringError`, and 2 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 120-150
```cpp
          consumeError(ObjOrErr.takeError());
          continue;
        }

        std::unique_ptr<object::ObjectFile> Obj = std::move(ObjOrErr.get());
        if (isArchitectureCompatible(*Obj)) {
          LLVM_DEBUG(
              dbgs() << "ObjectFileLoader: Found compatible object slice\n";);

          return object::OwningBinary<object::ObjectFile>(
              std::move(Obj), std::move(OwningBin.second));

        } else {
          LLVM_DEBUG(dbgs() << "ObjectFileLoader: Incompatible architecture "
                               "slice skipped\n";);
        }
      }
      LLVM_DEBUG(dbgs() << "ObjectFileLoader: No compatible slices found in "
                           "universal binary\n";);
      return createStringError(inconvertibleErrorCode(),
                               "No compatible object found in fat binary: %s",
                               FilePath.str().c_str());
    }
    return ObjOrErr.takeError();
#else
    LLVM_DEBUG(dbgs() << "ObjectFileLoader: Failed to open file " << FilePath
                      << "\n";);
    return ObjOrErr.takeError();
#endif
  }

```
- **EN**: Implements logic around `consumeError`, `move`, `dbgs`, `ObjectFile>`, and 3 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `consumeError`, `move`, `dbgs`, `ObjectFile>`, and 3 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 151-166
```cpp
  return createStringError(inconvertibleErrorCode(),
                           "Not a compatible object file : %s",
                           FilePath.str().c_str());
}

template <class ELFT>
bool isELFSharedLibrary(const object::ELFFile<ELFT> &ELFObj) {
  if (ELFObj.getHeader().e_type != ELF::ET_DYN)
    return false;

  auto PHOrErr = ELFObj.program_headers();
  if (!PHOrErr) {
    consumeError(PHOrErr.takeError());
    return true;
  }

```
- **EN**: Introduces declarations for `ELFT`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `ELFT` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 167-198
```cpp
  for (auto Phdr : *PHOrErr) {
    if (Phdr.p_type == ELF::PT_INTERP)
      return false;
  }

  return true;
}

bool isSharedLibraryObject(object::ObjectFile &Obj) {
  if (Obj.isELF()) {
    if (auto *ELF32LE = dyn_cast<object::ELF32LEObjectFile>(&Obj))
      return isELFSharedLibrary(ELF32LE->getELFFile());
    if (auto *ELF64LE = dyn_cast<object::ELF64LEObjectFile>(&Obj))
      return isELFSharedLibrary(ELF64LE->getELFFile());
    if (auto *ELF32BE = dyn_cast<object::ELF32BEObjectFile>(&Obj))
      return isELFSharedLibrary(ELF32BE->getELFFile());
    if (auto *ELF64BE = dyn_cast<object::ELF64BEObjectFile>(&Obj))
      return isELFSharedLibrary(ELF64BE->getELFFile());
  } else if (Obj.isMachO()) {
    const object::MachOObjectFile *MachO =
        dyn_cast<object::MachOObjectFile>(&Obj);
    if (!MachO) {
      LLVM_DEBUG(dbgs() << "Failed to cast to MachOObjectFile.\n";);
      return false;
    }
    LLVM_DEBUG({
      bool Result =
          MachO->getHeader().filetype == MachO::HeaderFileType::MH_DYLIB;
      dbgs() << "Mach-O filetype: " << MachO->getHeader().filetype
             << " (MH_DYLIB == " << MachO::HeaderFileType::MH_DYLIB
             << "), shared: " << Result << "\n";
    });
```
- **EN**: Implements logic around `isSharedLibraryObject`, `isELFSharedLibrary`, `MachOObjectFile>`, `getHeader`, and 1 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `isSharedLibraryObject`, `isELFSharedLibrary`, `MachOObjectFile>`, `getHeader`, and 1 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 199-216
```cpp

    return MachO->getHeader().filetype == MachO::HeaderFileType::MH_DYLIB;
  } else if (Obj.isCOFF()) {
    const object::COFFObjectFile *coff = dyn_cast<object::COFFObjectFile>(&Obj);
    if (!coff)
      return false;
    return coff->getCharacteristics() & COFF::IMAGE_FILE_DLL;
  } else {
    LLVM_DEBUG(dbgs() << "Binary is not an ObjectFile.\n";);
  }

  return false;
}

bool DylibPathValidator::isSharedLibrary(StringRef Path) const {
  LLVM_DEBUG(dbgs() << "Checking if path is a shared library: " << Path
                    << "\n";);

```
- **EN**: Implements logic around `getHeader`, `COFFObjectFile>`, `getCharacteristics`, `isSharedLibrary`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getHeader`, `COFFObjectFile>`, `getCharacteristics`, `isSharedLibrary` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 217-248
```cpp
  auto FileType = sys::fs::get_file_type(Path, /*Follow*/ true);
  if (FileType != sys::fs::file_type::regular_file) {
    LLVM_DEBUG(dbgs() << "File type is not a regular file for path: " << Path
                      << "\n";);
    return false;
  }

  file_magic MagicCode;
  identify_magic(Path, MagicCode);

  // Skip archives.
  if (MagicCode == file_magic::archive)
    return false;

  // Object file inspection for PE/COFF, ELF, and Mach-O
  bool NeedsObjectInspection =
#if defined(_WIN32)
      (MagicCode == file_magic::pecoff_executable);
#elif defined(__APPLE__)
      (MagicCode == file_magic::macho_universal_binary ||
       MagicCode == file_magic::macho_fixed_virtual_memory_shared_lib ||
       MagicCode == file_magic::macho_dynamically_linked_shared_lib ||
       MagicCode == file_magic::macho_dynamically_linked_shared_lib_stub);
#elif defined(LLVM_ON_UNIX)
#ifdef __CYGWIN__
      (MagicCode == file_magic::pecoff_executable);
#else
      (MagicCode == file_magic::elf_shared_object);
#endif
#else
#error "Unsupported platform."
#endif
```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 249-264
```cpp

  if (!NeedsObjectInspection) {
    LLVM_DEBUG(dbgs() << "Path is not identified as a shared library: " << Path
                      << "\n";);
    return false;
  }

  ObjectFileLoader ObjLoader(Path);
  auto ObjOrErr = ObjLoader.getObjectFile();
  if (!ObjOrErr) {
    consumeError(ObjOrErr.takeError());
    return false;
  }

  bool IsShared = isSharedLibraryObject(ObjOrErr.get());

```
- **EN**: Implements logic around `ObjLoader`, `getObjectFile`, `consumeError`, `isSharedLibraryObject`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `ObjLoader`, `getObjectFile`, `consumeError`, `isSharedLibraryObject` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 265-283
```cpp
  if (IsShared && ObjCache)
    ObjCache->insert(Path, std::move(ObjLoader));

  return IsShared;
}

void DylibSubstitutor::configure(StringRef LoaderPath) {
  SmallString<512> ExecPath(sys::fs::getMainExecutable(nullptr, nullptr));
  sys::path::remove_filename(ExecPath);

  SmallString<512> LoaderDir;
  if (LoaderPath.empty()) {
    LoaderDir = std::move(ExecPath);
  } else {
    LoaderDir = LoaderPath.str();
    if (!sys::fs::is_directory(LoaderPath))
      sys::path::remove_filename(LoaderDir);
  }

```
- **EN**: Implements logic around `insert`, `configure`, `ExecPath`, `remove_filename`, and 2 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `insert`, `configure`, `ExecPath`, `remove_filename`, and 2 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 284-304
```cpp
#ifdef __APPLE__
  Placeholders.push_back({"@loader_path", std::string(LoaderDir)});
  Placeholders.push_back({"@executable_path", std::string(ExecPath)});
#else
  Placeholders.push_back({"$origin", std::string(LoaderDir)});
#endif
}

std::optional<std::string>
SearchPathResolver::resolve(StringRef Stem, const DylibSubstitutor &Subst,
                            DylibPathValidator &Validator) const {
  for (const auto &SP : Paths) {
    std::string Base = Subst.substitute(SP);

    SmallString<512> FullPath(Base);
    if (!PlaceholderPrefix.empty() &&
        Stem.starts_with_insensitive(PlaceholderPrefix))
      FullPath.append(Stem.drop_front(PlaceholderPrefix.size()));
    else
      sys::path::append(FullPath, Stem);

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 305-331
```cpp
    LLVM_DEBUG(dbgs() << "SearchPathResolver::resolve FullPath = " << FullPath
                      << "\n";);

    if (auto Valid = Validator.validate(FullPath.str()))
      return Valid;
  }

  return std::nullopt;
}

std::optional<std::string>
DylibResolverImpl::tryWithExtensions(StringRef LibStem) const {
  LLVM_DEBUG(dbgs() << "tryWithExtensions: baseName = " << LibStem << "\n";);
  SmallVector<SmallString<256>, 8> Candidates;

  // Add extensions by platform
#if defined(__APPLE__)
  Candidates.emplace_back(LibStem);
  Candidates.back() += ".dylib";
#elif defined(_WIN32)
  Candidates.emplace_back(LibStem);
  Candidates.back() += ".dll";
#else
  Candidates.emplace_back(LibStem);
  Candidates.back() += ".so";
#endif

```
- **EN**: Implements logic around `tryWithExtensions`, `emplace_back`, `back`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `tryWithExtensions`, `emplace_back`, `back` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 332-349
```cpp
  // Optionally try "lib" prefix if not already there
  StringRef FileName = sys::path::filename(LibStem);
  StringRef Base = sys::path::parent_path(LibStem);
  if (!FileName.starts_with("lib")) {
    SmallString<256> WithPrefix(Base);
    if (!WithPrefix.empty())
      sys::path::append(WithPrefix, ""); // ensure separator if needed
    WithPrefix += "lib";
    WithPrefix += FileName;

#if defined(__APPLE__)
    WithPrefix += ".dylib";
#elif defined(_WIN32)
    WithPrefix += ".dll";
#else
    WithPrefix += ".so";
#endif

```
- **EN**: Implements logic around `filename`, `parent_path`, `WithPrefix`, `append`.
- **CN**: 围绕 `filename`, `parent_path`, `WithPrefix`, `append` 实现具体逻辑。

### Lines 350-369
```cpp
    Candidates.push_back(std::move(WithPrefix));
  }

  LLVM_DEBUG({
    dbgs() << "  Candidates to try:\n";
    for (const auto &C : Candidates)
      dbgs() << "    " << C << "\n";
  });

  // Try all variants using tryAllPaths
  for (const auto &Name : Candidates) {

    LLVM_DEBUG(dbgs() << "  Trying candidate: " << Name << "\n";);

    for (const auto &R : Resolvers) {
      if (auto Res = R.resolve(Name, Substitutor, Validator))
        return Res;
    }
  }

```
- **EN**: Implements logic around `push_back`, `dbgs`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `push_back`, `dbgs` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 370-389
```cpp
  LLVM_DEBUG(dbgs() << "  -> No candidate Resolved.\n";);

  return std::nullopt;
}

std::optional<std::string>
DylibResolverImpl::resolve(StringRef LibStem, bool VariateLibStem) const {
  LLVM_DEBUG(dbgs() << "Resolving library stem: " << LibStem << "\n";);

  // If it is an absolute path, don't try iterate over the paths.
  if (sys::path::is_absolute(LibStem)) {
    LLVM_DEBUG(dbgs() << "  -> Absolute path detected.\n";);
    return Validator.validate(LibStem);
  }

  if (!LibStem.starts_with_insensitive("@rpath")) {
    if (auto norm = Validator.validate(Substitutor.substitute(LibStem))) {
      LLVM_DEBUG(dbgs() << "  -> Resolved after substitution: " << *norm
                        << "\n";);

```
- **EN**: Implements logic around `resolve`, `validate`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `resolve`, `validate` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 390-408
```cpp
      return norm;
    }
  }

  for (const auto &R : Resolvers) {
    LLVM_DEBUG(dbgs() << "  -> Resolving via search path ... \n";);
    if (auto Result = R.resolve(LibStem, Substitutor, Validator)) {
      LLVM_DEBUG(dbgs() << "  -> Resolved via search path: " << *Result
                        << "\n";);

      return Result;
    }
  }

  // Expand libStem with paths, extensions, etc.
  // std::string foundName;
  if (VariateLibStem) {
    LLVM_DEBUG(dbgs() << "  -> Trying with extensions...\n";);

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 409-426
```cpp
    if (auto Norm = tryWithExtensions(LibStem)) {
      LLVM_DEBUG(dbgs() << "  -> Resolved via tryWithExtensions: " << *Norm
                        << "\n";);
      return Norm;
    }
  }

  LLVM_DEBUG(dbgs() << "  -> Could not resolve: " << LibStem << "\n";);

  return std::nullopt;
}

#ifndef _WIN32
mode_t PathResolver::lstatCached(StringRef Path) {
  // If already cached - retun cached result
  if (auto Cache = LibPathCache->read_lstat(Path))
    return *Cache;

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 427-450
```cpp
  // Not cached: perform lstat and store
  struct stat buf{};
  mode_t st_mode = (lstat(Path.str().c_str(), &buf) == -1) ? 0 : buf.st_mode;

  LibPathCache->insert_lstat(Path, st_mode);

  return st_mode;
}

std::optional<std::string> PathResolver::readlinkCached(StringRef Path) {
  // If already cached - retun cached result
  if (auto Cache = LibPathCache->read_link(Path))
    return Cache;

  // If result not in cache - call system function and cache result
  SmallString<128> Buf;
  if (!sys::fs::readlink(Path, Buf)) {
    std::string s(Buf.str());
    LibPathCache->insert_link(Path, s);
    return s;
  }
  return std::nullopt;
}

```
- **EN**: Introduces declarations for `stat`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `stat` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 451-470
```cpp
void createComponent(StringRef Path, StringRef BasePath, bool BaseIsResolved,
                     SmallVector<StringRef, 16> &Component) {
  StringRef Separator = sys::path::get_separator();
  if (!BaseIsResolved) {
    if (Path[0] == '~' &&
        (Path.size() == 1 || sys::path::is_separator(Path[1]))) {
      static SmallString<128> HomeP;
      if (HomeP.str().empty())
        sys::path::home_directory(HomeP);
      StringRef(HomeP).split(Component, Separator, /*MaxSplit*/ -1,
                             /*KeepEmpty*/ false);
    } else if (BasePath.empty()) {
      static SmallString<256> CurrentPath;
      if (CurrentPath.str().empty())
        sys::fs::current_path(CurrentPath);
      StringRef(CurrentPath)
          .split(Component, Separator, /*MaxSplit*/ -1, /*KeepEmpty*/ false);
    } else {
      BasePath.split(Component, Separator, /*MaxSplit*/ -1,
                     /*KeepEmpty*/ false);
```
- **EN**: Implements logic around `createComponent`, `get_separator`, `size`, `home_directory`, and 3 more symbols; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `createComponent`, `get_separator`, `size`, `home_directory`, and 3 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 471-495
```cpp
    }
  }

  Path.split(Component, Separator, /*MaxSplit*/ -1, /*KeepEmpty*/ false);
}

void normalizePathSegments(SmallVector<StringRef, 16> &PathParts) {
  SmallVector<StringRef, 16> NormalizedPath;
  for (auto &Part : PathParts) {
    if (Part == ".") {
      continue;
    } else if (Part == "..") {
      if (!NormalizedPath.empty() && NormalizedPath.back() != "..") {
        NormalizedPath.pop_back();
      } else {
        NormalizedPath.push_back("..");
      }
    } else {
      NormalizedPath.push_back(Part);
    }
  }
  PathParts.swap(NormalizedPath);
}
#endif

```
- **EN**: Implements logic around `split`, `normalizePathSegments`, `pop_back`, `push_back`, and 1 more symbols.
- **CN**: 围绕 `split`, `normalizePathSegments`, `pop_back`, `push_back`, and 1 more symbols 实现具体逻辑。

### Lines 496-515
```cpp
std::optional<std::string> PathResolver::realpathCached(StringRef Path,
                                                        std::error_code &EC,
                                                        StringRef Base,
                                                        bool BaseIsResolved,
                                                        long SymLoopLevel) {
  EC.clear();

  if (Path.empty()) {
    EC = std::make_error_code(std::errc::no_such_file_or_directory);
    LLVM_DEBUG(dbgs() << "PathResolver::realpathCached: Empty path\n";);

    return std::nullopt;
  }

  if (SymLoopLevel <= 0) {
    EC = std::make_error_code(std::errc::too_many_symbolic_link_levels);
    LLVM_DEBUG(
        dbgs() << "PathResolver::realpathCached: Too many Symlink levels: "
               << Path << "\n";);

```
- **EN**: Implements logic around `realpathCached`, `clear`, `make_error_code`, `dbgs`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `realpathCached`, `clear`, `make_error_code`, `dbgs` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 516-537
```cpp
    return std::nullopt;
  }

  // If already cached - retun cached result
  bool isRelative = sys::path::is_relative(Path);
  if (!isRelative) {
    if (auto Cached = LibPathCache->read_realpath(Path)) {
      EC = Cached->ErrnoCode;
      if (EC) {
        LLVM_DEBUG(dbgs() << "PathResolver::realpathCached: Cached (error) for "
                          << Path << "\n";);
      } else {
        LLVM_DEBUG(
            dbgs() << "PathResolver::realpathCached: Cached (success) for "
                   << Path << " => " << Cached->canonicalPath << "\n";);
      }
      return Cached->canonicalPath.empty()
                 ? std::nullopt
                 : std::make_optional(Cached->canonicalPath);
    }
  }

```
- **EN**: Implements logic around `is_relative`, `dbgs`, `empty`, `make_optional`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `is_relative`, `dbgs`, `empty`, `make_optional` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 538-557
```cpp
  LLVM_DEBUG(dbgs() << "PathResolver::realpathCached: Resolving path: " << Path
                    << "\n";);

  // If result not in cache - call system function and cache result

  StringRef Separator(sys::path::get_separator());
  SmallString<256> Resolved(Separator);
#ifndef _WIN32
  SmallVector<StringRef, 16> Components;

  if (isRelative) {
    if (BaseIsResolved) {
      Resolved.assign(Base);
      LLVM_DEBUG(dbgs() << "  Using Resolved base: " << Base << "\n";);
    }
    createComponent(Path, Base, BaseIsResolved, Components);
  } else {
    Path.split(Components, Separator, /*MaxSplit*/ -1, /*KeepEmpty*/ false);
  }

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 558-579
```cpp
  normalizePathSegments(Components);
  LLVM_DEBUG({
    for (auto &C : Components)
      dbgs() << " " << C << " ";

    dbgs() << "\n";
  });

  // Handle path list items
  for (const auto &Component : Components) {
    if (Component == ".")
      continue;
    if (Component == "..") {
      // collapse "a/b/../c" to "a/c"
      size_t S = Resolved.rfind(Separator);
      if (S != llvm::StringRef::npos)
        Resolved.resize(S);
      if (Resolved.empty())
        Resolved = Separator;
      continue;
    }

```
- **EN**: Implements logic around `normalizePathSegments`, `dbgs`, `rfind`, `resize`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `normalizePathSegments`, `dbgs`, `rfind`, `resize` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 580-596
```cpp
    size_t oldSize = Resolved.size();
    sys::path::append(Resolved, Component);
    const char *ResolvedPath = Resolved.c_str();
    LLVM_DEBUG(dbgs() << "  Processing Component: " << Component << " => "
                      << ResolvedPath << "\n";);
    mode_t st_mode = lstatCached(ResolvedPath);

    if (S_ISLNK(st_mode)) {
      LLVM_DEBUG(dbgs() << "    Found symlink: " << ResolvedPath << "\n";);

      auto SymlinkOpt = readlinkCached(ResolvedPath);
      if (!SymlinkOpt) {
        EC = std::make_error_code(std::errc::no_such_file_or_directory);
        LibPathCache->insert_realpath(Path, LibraryPathCache::PathInfo{"", EC});
        LLVM_DEBUG(dbgs() << "    Failed to read symlink: " << ResolvedPath
                          << "\n";);

```
- **EN**: Implements logic around `size`, `append`, `c_str`, `lstatCached`, and 3 more symbols; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `size`, `append`, `c_str`, `lstatCached`, and 3 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 597-616
```cpp
        return std::nullopt;
      }

      StringRef Symlink = *SymlinkOpt;
      LLVM_DEBUG(dbgs() << "    Symlink points to: " << Symlink << "\n";);

      std::string resolvedBase = "";
      if (sys::path::is_relative(Symlink)) {
        Resolved.resize(oldSize);
        resolvedBase = Resolved.str().str();
      }

      auto RealSymlink =
          realpathCached(Symlink, EC, resolvedBase,
                         /*BaseIsResolved=*/true, SymLoopLevel - 1);
      if (!RealSymlink) {
        LibPathCache->insert_realpath(Path, LibraryPathCache::PathInfo{"", EC});
        LLVM_DEBUG(dbgs() << "    Failed to resolve symlink target: " << Symlink
                          << "\n";);

```
- **EN**: Implements logic around `resize`, `str`, `realpathCached`, `insert_realpath`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `resize`, `str`, `realpathCached`, `insert_realpath` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 617-635
```cpp
        return std::nullopt;
      }

      Resolved.assign(*RealSymlink);
      LLVM_DEBUG(dbgs() << "    Symlink Resolved to: " << Resolved << "\n";);

    } else if (st_mode == 0) {
      EC = std::make_error_code(std::errc::no_such_file_or_directory);
      LibPathCache->insert_realpath(Path, LibraryPathCache::PathInfo{"", EC});
      LLVM_DEBUG(dbgs() << "    Component does not exist: " << ResolvedPath
                        << "\n";);

      return std::nullopt;
    }
  }
#else
  EC = sys::fs::real_path(Path, Resolved); // Windows fallback
#endif

```
- **EN**: Implements logic around `assign`, `make_error_code`, `insert_realpath`, `real_path`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `assign`, `make_error_code`, `insert_realpath`, `real_path` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 636-667
```cpp
  std::string Canonical = Resolved.str().str();
  {
    LibPathCache->insert_realpath(Path, LibraryPathCache::PathInfo{
                                            Canonical,
                                            std::error_code() // success
                                        });
  }
  LLVM_DEBUG(dbgs() << "PathResolver::realpathCached: Final Resolved: " << Path
                    << " => " << Canonical << "\n";);
  return Canonical;
}

void LibraryScanHelper::addBasePath(const std::string &Path, PathType K) {
  std::error_code EC;
  std::string Canon = resolveCanonical(Path, EC);
  if (EC) {
    LLVM_DEBUG(
        dbgs()
            << "LibraryScanHelper::addBasePath: Failed to canonicalize path: "
            << Path << "\n";);
    return;
  }
  std::unique_lock<std::shared_mutex> Lock(Mtx);
  if (LibSearchPaths.count(Canon)) {
    LLVM_DEBUG(dbgs() << "LibraryScanHelper::addBasePath: Already added: "
                      << Canon << "\n";);
    return;
  }
  K = K == PathType::Unknown ? classifyKind(Canon) : K;
  LibSearchPaths[Canon] = std::make_unique<LibrarySearchPath>(Canon, K);
  auto &SP = LibSearchPaths[Canon];

```
- **EN**: Implements logic around `str`, `insert_realpath`, `error_code`, `addBasePath`, and 5 more symbols; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `str`, `insert_realpath`, `error_code`, `addBasePath`, and 5 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 668-683
```cpp
  if (K == PathType::User) {
    LLVM_DEBUG(dbgs() << "LibraryScanHelper::addBasePath: Added User path: "
                      << Canon << "\n";);
    UnscannedUsr.push_back(StringRef(SP->BasePath));
  } else {
    LLVM_DEBUG(dbgs() << "LibraryScanHelper::addBasePath: Added System path: "
                      << Canon << "\n";);
    UnscannedSys.push_back(StringRef(SP->BasePath));
  }
}

void LibraryScanHelper::getNextBatch(
    PathType K, size_t BatchSize,
    SmallVectorImpl<const LibrarySearchPath *> &Result) {
  auto &Queue = (K == PathType::User) ? UnscannedUsr : UnscannedSys;

```
- **EN**: Implements logic around `push_back`, `getNextBatch`.
- **CN**: 围绕 `push_back`, `getNextBatch` 实现具体逻辑。

### Lines 684-699
```cpp
  std::unique_lock<std::shared_mutex> Lock(Mtx);

  while (!Queue.empty() && (BatchSize == 0 || Result.size() < BatchSize)) {
    StringRef Base = Queue.front();
    auto It = LibSearchPaths.find(Base);
    if (It != LibSearchPaths.end()) {
      auto &SP = It->second;
      ScanState Expected = ScanState::NotScanned;
      if (SP->State.compare_exchange_strong(Expected, ScanState::Scanning)) {
        Result.push_back(SP.get());
      }
    }
    Queue.pop_front();
  }
}

```
- **EN**: Implements logic around `Lock`, `front`, `find`, `push_back`, and 1 more symbols.
- **CN**: 围绕 `Lock`, `front`, `find`, `push_back`, and 1 more symbols 实现具体逻辑。

### Lines 700-719
```cpp
bool LibraryScanHelper::isTrackedBasePath(StringRef Path) const {
  std::error_code EC;
  std::string Canon = resolveCanonical(Path, EC);
  if (EC)
    return false;

  std::shared_lock<std::shared_mutex> Lock(Mtx);
  return LibSearchPaths.count(Canon) > 0;
}

bool LibraryScanHelper::leftToScan(PathType K) const {
  std::shared_lock<std::shared_mutex> Lock(Mtx);
  for (const auto &KV : LibSearchPaths) {
    const auto &SP = KV.second;
    if (SP->Kind == K && SP->State == ScanState::NotScanned)
      return true;
  }
  return false;
}

```
- **EN**: Implements logic around `isTrackedBasePath`, `resolveCanonical`, `Lock`, `count`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `isTrackedBasePath`, `resolveCanonical`, `Lock`, `count`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 720-740
```cpp
void LibraryScanHelper::resetToScan() {
  std::shared_lock<std::shared_mutex> Lock(Mtx);

  for (auto &[_, SP] : LibSearchPaths) {
    ScanState Expected = ScanState::Scanned;

    if (!SP->State.compare_exchange_strong(Expected, ScanState::NotScanned))
      continue;

    auto &TargetList =
        (SP->Kind == PathType::User) ? UnscannedUsr : UnscannedSys;
    TargetList.emplace_back(SP->BasePath);
  }
}

std::string LibraryScanHelper::resolveCanonical(StringRef Path,
                                                std::error_code &EC) const {
  auto Canon = LibPathResolver->resolve(Path, EC);
  return EC ? Path.str() : *Canon;
}

```
- **EN**: Implements logic around `resetToScan`, `Lock`, `emplace_back`, `resolveCanonical`, and 2 more symbols; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `resetToScan`, `Lock`, `emplace_back`, `resolveCanonical`, and 2 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 741-759
```cpp
PathType LibraryScanHelper::classifyKind(StringRef Path) const {
  // Detect home directory
  const char *Home = getenv("HOME");
  if (Home && Path.starts_with(Home))
    return PathType::User;

  static const std::array<std::string, 5> UserPrefixes = {
      "/usr/local",    // often used by users for manual installs
      "/opt/homebrew", // common on macOS
      "/opt/local",    // MacPorts
      "/home",         // Linux home dirs
      "/Users",        // macOS user dirs
  };

  for (const auto &Prefix : UserPrefixes) {
    if (Path.starts_with(Prefix))
      return PathType::User;
  }

```
- **EN**: Implements logic around `classifyKind`, `getenv`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `classifyKind`, `getenv` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 760-784
```cpp
  return PathType::System;
}

Expected<LibraryDepsInfo> parseMachODeps(const object::MachOObjectFile &Obj) {
  LibraryDepsInfo Libdeps;
  LLVM_DEBUG(dbgs() << "Parsing Mach-O dependencies...\n";);
  for (const auto &Command : Obj.load_commands()) {
    switch (Command.C.cmd) {
    case MachO::LC_LOAD_DYLIB: {
      MachO::dylib_command dylibCmd = Obj.getDylibIDLoadCommand(Command);
      const char *name = Command.Ptr + dylibCmd.dylib.name;
      Libdeps.addDep(name);
      LLVM_DEBUG(dbgs() << "  Found LC_LOAD_DYLIB: " << name << "\n";);
    } break;
    case MachO::LC_LOAD_WEAK_DYLIB:
    case MachO::LC_REEXPORT_DYLIB:
    case MachO::LC_LOAD_UPWARD_DYLIB:
    case MachO::LC_LAZY_LOAD_DYLIB:
      break;
    case MachO::LC_RPATH: {
      // Extract RPATH
      MachO::rpath_command rpathCmd = Obj.getRpathCommand(Command);
      const char *rpath = Command.Ptr + rpathCmd.path;
      LLVM_DEBUG(dbgs() << "  Found LC_RPATH: " << rpath << "\n";);

```
- **EN**: Implements logic around `parseMachODeps`, `getDylibIDLoadCommand`, `addDep`, `getRpathCommand`; this block uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `parseMachODeps`, `getDylibIDLoadCommand`, `addDep`, `getRpathCommand` 实现具体逻辑；这一段使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 785-800
```cpp
      SmallVector<StringRef, 4> RawPaths;
      SplitString(StringRef(rpath), RawPaths,
                  sys::EnvPathSeparator == ':' ? ":" : ";");

      for (const auto &raw : RawPaths) {
        Libdeps.addRPath(raw.str()); // Convert to std::string
        LLVM_DEBUG(dbgs() << "    Parsed RPATH entry: " << raw << "\n";);
      }
      break;
    }
    }
  }

  return Expected<LibraryDepsInfo>(std::move(Libdeps));
}

```
- **EN**: Implements logic around `SplitString`, `addRPath`, `Expected<LibraryDepsInfo>`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `SplitString`, `addRPath`, `Expected<LibraryDepsInfo>` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 801-820
```cpp
template <class ELFT>
static Expected<StringRef> getDynamicStrTab(const object::ELFFile<ELFT> &Elf) {
  auto DynamicEntriesOrError = Elf.dynamicEntries();
  if (!DynamicEntriesOrError)
    return DynamicEntriesOrError.takeError();

  for (const typename ELFT::Dyn &Dyn : *DynamicEntriesOrError) {
    if (Dyn.d_tag == ELF::DT_STRTAB) {
      auto MappedAddrOrError = Elf.toMappedAddr(Dyn.getPtr());
      if (!MappedAddrOrError)
        return MappedAddrOrError.takeError();
      return StringRef(reinterpret_cast<const char *>(*MappedAddrOrError));
    }
  }

  // If the dynamic segment is not present, we fall back on the sections.
  auto SectionsOrError = Elf.sections();
  if (!SectionsOrError)
    return SectionsOrError.takeError();

```
- **EN**: Introduces declarations for `ELFT`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `ELFT` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 821-836
```cpp
  for (const typename ELFT::Shdr &Sec : *SectionsOrError) {
    if (Sec.sh_type == ELF::SHT_DYNSYM)
      return Elf.getStringTableForSymtab(Sec);
  }

  return make_error<StringError>("dynamic string table not found",
                                 inconvertibleErrorCode());
}

template <typename ELFT>
Expected<LibraryDepsInfo> parseELF(const object::ELFFile<ELFT> &Elf) {
  LibraryDepsInfo Deps;
  Expected<StringRef> StrTabOrErr = getDynamicStrTab(Elf);
  if (!StrTabOrErr)
    return StrTabOrErr.takeError();

```
- **EN**: Implements logic around `getStringTableForSymtab`, `make_error<StringError>`, `inconvertibleErrorCode`, `parseELF`, and 2 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getStringTableForSymtab`, `make_error<StringError>`, `inconvertibleErrorCode`, `parseELF`, and 2 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 837-868
```cpp
  const char *Data = StrTabOrErr->data();

  auto DynamicEntriesOrError = Elf.dynamicEntries();
  if (!DynamicEntriesOrError) {
    return DynamicEntriesOrError.takeError();
  }

  for (const typename ELFT::Dyn &Dyn : *DynamicEntriesOrError) {
    switch (Dyn.d_tag) {
    case ELF::DT_NEEDED:
      Deps.addDep(Data + Dyn.d_un.d_val);
      break;
    case ELF::DT_RPATH: {
      SmallVector<StringRef, 4> RawPaths;
      SplitString(Data + Dyn.d_un.d_val, RawPaths,
                  sys::EnvPathSeparator == ':' ? ":" : ";");
      for (const auto &raw : RawPaths)
        Deps.addRPath(raw.str());
      break;
    }
    case ELF::DT_RUNPATH: {
      SmallVector<StringRef, 4> RawPaths;
      SplitString(Data + Dyn.d_un.d_val, RawPaths,
                  sys::EnvPathSeparator == ':' ? ":" : ";");
      for (const auto &raw : RawPaths)
        Deps.addRunPath(raw.str());
      break;
    }
    case ELF::DT_FLAGS_1:
      // Check if this is not a pie executable.
      if (Dyn.d_un.d_val & ELF::DF_1_PIE)
        Deps.isPIE = true;
```
- **EN**: Implements logic around `data`, `dynamicEntries`, `takeError`, `addDep`, and 3 more symbols; this block uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `data`, `dynamicEntries`, `takeError`, `addDep`, and 3 more symbols 实现具体逻辑；这一段使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 869-891
```cpp
      break;
      // (Dyn.d_tag == ELF::DT_NULL) continue;
      // (Dyn.d_tag == ELF::DT_AUXILIARY || Dyn.d_tag == ELF::DT_FILTER)
    default:
      break;
    }
  }

  return Expected<LibraryDepsInfo>(std::move(Deps));
}

Expected<LibraryDepsInfo> parseELFDeps(const object::ELFObjectFileBase &Obj) {
  using namespace object;
  LLVM_DEBUG(dbgs() << "parseELFDeps: Detected ELF object\n";);
  if (const auto *ELF = dyn_cast<ELF32LEObjectFile>(&Obj))
    return parseELF(ELF->getELFFile());
  else if (const auto *ELF = dyn_cast<ELF32BEObjectFile>(&Obj))
    return parseELF(ELF->getELFFile());
  else if (const auto *ELF = dyn_cast<ELF64LEObjectFile>(&Obj))
    return parseELF(ELF->getELFFile());
  else if (const auto *ELF = dyn_cast<ELF64BEObjectFile>(&Obj))
    return parseELF(ELF->getELFFile());

```
- **EN**: Introduces declarations for `object`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `object` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 892-911
```cpp
  LLVM_DEBUG(dbgs() << "parseELFDeps: Unknown ELF format\n";);
  return createStringError(std::errc::not_supported, "Unknown ELF format");
}

Expected<LibraryDepsInfo> parseDependencies(StringRef FilePath,
                                            object::ObjectFile *Obj) {

  if (auto *elfObj = dyn_cast<object::ELFObjectFileBase>(Obj)) {
    LLVM_DEBUG(dbgs() << "extractDeps: File " << FilePath
                      << " is an ELF object\n";);

    return parseELFDeps(*elfObj);
  }

  if (auto *macho = dyn_cast<object::MachOObjectFile>(Obj)) {
    LLVM_DEBUG(dbgs() << "extractDeps: File " << FilePath
                      << " is a Mach-O object\n";);
    return parseMachODeps(*macho);
  }

```
- **EN**: Implements logic around `createStringError`, `parseDependencies`, `parseELFDeps`, `parseMachODeps`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `createStringError`, `parseDependencies`, `parseELFDeps`, `parseMachODeps` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 912-934
```cpp
  if (Obj->isCOFF()) {
    // TODO: COFF support
    return LibraryDepsInfo();
  }

  LLVM_DEBUG(dbgs() << "extractDeps: Unsupported binary format for file "
                    << FilePath << "\n";);
  return createStringError(inconvertibleErrorCode(),
                           "Unsupported binary format: %s",
                           FilePath.str().c_str());
}

Expected<LibraryDepsInfo> LibraryScanner::extractDeps(StringRef FilePath) {
  LLVM_DEBUG(dbgs() << "extractDeps: Attempting to open file " << FilePath
                    << "\n";);
  //  check cache first
  if (auto Cached = ObjCache.take(FilePath)) {
    auto ObjOrErr = Cached->getObjectFile();
    if (!ObjOrErr)
      return ObjOrErr.takeError();
    return parseDependencies(FilePath, &*ObjOrErr);
  }

```
- **EN**: Implements logic around `LibraryDepsInfo`, `createStringError`, `str`, `extractDeps`, and 3 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `LibraryDepsInfo`, `createStringError`, `str`, `extractDeps`, and 3 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 935-955
```cpp
  // fall back to normal loading
  ObjectFileLoader ObjLoader(FilePath);
  auto ObjOrErr = ObjLoader.getObjectFile();
  if (!ObjOrErr) {
    LLVM_DEBUG(dbgs() << "extractDeps: Failed to open " << FilePath << "\n";);
    return ObjOrErr.takeError();
  }

  return parseDependencies(FilePath, &*ObjOrErr);
}

bool LibraryScanner::shouldScan(StringRef FilePath, bool IsResolvingDep) {
  LLVM_DEBUG(dbgs() << "[shouldScan] Checking: " << FilePath << "\n";);

  LibraryPathCache &Cache = ScanHelper.getCache();
  // [1] Skip if we've already seen this path (via cache)
  if (Cache.hasSeen(FilePath)) {
    LLVM_DEBUG(dbgs() << "  -> Skipped: already seen.\n";);
    return false;
  }

```
- **EN**: Implements logic around `ObjLoader`, `getObjectFile`, `takeError`, `parseDependencies`, and 2 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `ObjLoader`, `getObjectFile`, `takeError`, `parseDependencies`, and 2 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 956-976
```cpp
  // [2] Already tracked in LibraryManager?
  /*if (LibMgr.hasLibrary(FilePath)) {
    LLVM_DEBUG(dbgs() << "  -> Skipped: already tracked by LibraryManager.\n";);
    return false;
  }*/

  // [3] Skip if it's not a shared library.
  if (!IsResolvingDep && !Validator.isSharedLibrary(FilePath)) {
    LLVM_DEBUG(dbgs() << "  -> Skipped: not a shared library.\n";);
    return false;
  }

  // Mark seen this path
  Cache.markSeen(FilePath.str());

  // [4] Run user-defined hook (default: always true)
  if (!ShouldScanCall(FilePath)) {
    LLVM_DEBUG(dbgs() << "  -> Skipped: user-defined hook rejected.\n";);
    return false;
  }

```
- **EN**: Implements logic around `markSeen`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `markSeen` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 977-996
```cpp
  LLVM_DEBUG(dbgs() << "  -> Accepted: ready to scan " << FilePath << "\n";);
  return true;
}

void LibraryScanner::handleLibrary(StringRef FilePath, PathType K, int level) {
  LLVM_DEBUG(dbgs() << "LibraryScanner::handleLibrary: Scanning: " << FilePath
                    << ", level=" << level << "\n";);
  if (!shouldScan(FilePath, level > 0)) {
    LLVM_DEBUG(dbgs() << "  Skipped (shouldScan returned false): " << FilePath
                      << "\n";);
    return;
  }

  auto DepsOrErr = extractDeps(FilePath);
  if (!DepsOrErr) {
    LLVM_DEBUG(dbgs() << "  Failed to extract deps for: " << FilePath << "\n";);
    handleError(DepsOrErr.takeError());
    return;
  }

```
- **EN**: Implements logic around `handleLibrary`, `extractDeps`, `handleError`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `handleLibrary`, `extractDeps`, `handleError` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 997-1014
```cpp
  LibraryDepsInfo &Deps = *DepsOrErr;

  LLVM_DEBUG({
    dbgs() << "    Found deps : \n";
    for (const auto &dep : Deps.deps)
      dbgs() << "        : " << dep << "\n";
    dbgs() << "    Found @rpath : " << Deps.rpath.size() << "\n";
    for (const auto &r : Deps.rpath)
      dbgs() << "     : " << r << "\n";
    dbgs() << "    Found @runpath : \n";
    for (const auto &r : Deps.runPath)
      dbgs() << "     : " << r << "\n";
  });

  if (Deps.isPIE && level == 0) {
    LLVM_DEBUG(dbgs() << "  Skipped PIE executable at top level: " << FilePath
                      << "\n";);

```
- **EN**: Implements logic around `dbgs`.
- **CN**: 围绕 `dbgs` 实现具体逻辑。

### Lines 1015-1031
```cpp
    return;
  }

  bool Added = LibMgr.addLibrary(FilePath.str(), K);
  if (!Added) {
    LLVM_DEBUG(dbgs() << "  Already added: " << FilePath << "\n";);
    return;
  }

  // Heuristic 1: No RPATH/RUNPATH, skip deps
  if (Deps.rpath.empty() && Deps.runPath.empty()) {
    LLVM_DEBUG(
        dbgs() << "LibraryScanner::handleLibrary: Skipping deps (Heuristic1): "
               << FilePath << "\n";);
    return;
  }

```
- **EN**: Implements logic around `addLibrary`, `dbgs`.
- **CN**: 围绕 `addLibrary`, `dbgs` 实现具体逻辑。

### Lines 1032-1048
```cpp
  // Heuristic 2: All RPATH and RUNPATH already tracked
  auto allTracked = [&](const auto &Paths) {
    LLVM_DEBUG(dbgs() << "   Checking : " << Paths.size() << "\n";);
    return std::all_of(Paths.begin(), Paths.end(), [&](StringRef P) {
      LLVM_DEBUG(dbgs() << "      Checking isTrackedBasePath : " << P << "\n";);
      return ScanHelper.isTrackedBasePath(
          DylibResolver::resolvelinkerFlag(P, FilePath));
    });
  };

  if (allTracked(Deps.rpath) && allTracked(Deps.runPath)) {
    LLVM_DEBUG(
        dbgs() << "LibraryScanner::handleLibrary: Skipping deps (Heuristic2): "
               << FilePath << "\n";);
    return;
  }

```
- **EN**: Implements logic around `all_of`, `isTrackedBasePath`, `resolvelinkerFlag`, `dbgs`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `all_of`, `isTrackedBasePath`, `resolvelinkerFlag`, `dbgs` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 1049-1066
```cpp
  DylibResolver Resolver(Validator);
  Resolver.configure(FilePath,
                     {{Deps.rpath, SearchPathType::RPath},
                      {ScanHelper.getSearchPaths(), SearchPathType::UsrOrSys},
                      {Deps.runPath, SearchPathType::RunPath}});
  for (StringRef Dep : Deps.deps) {
    LLVM_DEBUG(dbgs() << "  Resolving dep: " << Dep << "\n";);
    auto DepFullOpt = Resolver.resolve(Dep);
    if (!DepFullOpt) {
      LLVM_DEBUG(dbgs() << "    Failed to resolve dep: " << Dep << "\n";);
      continue;
    }
    LLVM_DEBUG(dbgs() << "    Resolved dep to: " << *DepFullOpt << "\n";);

    handleLibrary(*DepFullOpt, K, level + 1);
  }
}

```
- **EN**: Implements logic around `Resolver`, `configure`, `getSearchPaths`, `resolve`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `Resolver`, `configure`, `getSearchPaths`, `resolve`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 1067-1086
```cpp
void LibraryScanner::scanBaseDir(LibrarySearchPath *SP) {
  if (!sys::fs::is_directory(SP->BasePath) || SP->BasePath.empty()) {
    LLVM_DEBUG(
        dbgs() << "LibraryScanner::scanBaseDir: Invalid or empty basePath: "
               << SP->BasePath << "\n";);
    return;
  }

  LLVM_DEBUG(dbgs() << "LibraryScanner::scanBaseDir: Scanning directory: "
                    << SP->BasePath << "\n";);
  std::error_code EC;

  SP->State.store(ScanState::Scanning);

  for (sys::fs::directory_iterator It(SP->BasePath, EC), end; It != end && !EC;
       It.increment(EC)) {
    auto Entry = *It;
    if (!Entry.status())
      continue;

```
- **EN**: Implements logic around `scanBaseDir`, `dbgs`, `store`, `increment`.
- **CN**: 围绕 `scanBaseDir`, `dbgs`, `store`, `increment` 实现具体逻辑。

### Lines 1087-1104
```cpp
    auto Status = *Entry.status();
    if (sys::fs::is_regular_file(Status) || sys::fs::is_symlink_file(Status)) {
      LLVM_DEBUG(dbgs() << "  Found file: " << Entry.path() << "\n";);

      std::string FinalPath;
      bool IsSymlink = sys::fs::is_symlink_file(Status);

      // Resolve symlink
      if (IsSymlink) {
        LLVM_DEBUG(dbgs() << "    Symlink → resolving...\n");

        auto CanonicalOpt = ScanHelper.resolve(Entry.path(), EC);
        if (EC || !CanonicalOpt) {
          LLVM_DEBUG(dbgs() << "    -> Skipped: resolve failed (EC="
                            << EC.message() << ")\n");
          continue;
        }

```
- **EN**: Implements logic around `status`, `is_symlink_file`, `resolve`, `message`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `status`, `is_symlink_file`, `resolve`, `message` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 1105-1123
```cpp
        FinalPath = std::move(*CanonicalOpt);

        LLVM_DEBUG(dbgs() << "    Canonical: " << FinalPath << "\n");

      } else {
        // make absolute
        SmallString<256> Abs(Entry.path());
        sys::fs::make_absolute(Abs);
        FinalPath = Abs.str().str();

        LLVM_DEBUG(dbgs() << "    Regular: absolute = " << FinalPath << "\n");
      }

      // Check if it's a directory — skip directories
      if (sys::fs::is_directory(Status)) {
        LLVM_DEBUG(dbgs() << "  -> Skipped: path is a directory.\n";);
        continue;
      }

```
- **EN**: Implements logic around `move`, `Abs`, `make_absolute`, `str`.
- **CN**: 围绕 `move`, `Abs`, `make_absolute`, `str` 实现具体逻辑。

### Lines 1124-1145
```cpp
      // async support ?
      handleLibrary(FinalPath, SP->Kind);
    }
  }

  SP->State.store(ScanState::Scanned);
}

void LibraryScanner::scanNext(PathType K, size_t BatchSize) {
  LLVM_DEBUG(dbgs() << "LibraryScanner::scanNext: Scanning next batch of size "
                    << BatchSize << " for kind "
                    << (K == PathType::User ? "User" : "System") << "\n";);

  SmallVector<const LibrarySearchPath *> SearchPaths;
  ScanHelper.getNextBatch(K, BatchSize, SearchPaths);
  for (const auto *SP : SearchPaths) {
    LLVM_DEBUG(dbgs() << "  Scanning unit with basePath: " << SP->BasePath
                      << "\n";);
    scanBaseDir(const_cast<LibrarySearchPath *>(SP));
  }
}
} // end namespace llvm::orc
```
- **EN**: Introduces declarations for `llvm::orc`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm::orc` 等声明，定义本文件后续使用的数据结构或接口。

## Key Concepts / 关键概念

- **ORC JIT / ORC JIT**:
  - **EN**: Coordinates JITDylibs, symbol materialization, execution sessions, and asynchronous compilation flows
  - **CN**: 协调 JITDylib、符号物化、ExecutionSession 与异步编译流程
- **Debug/unwind metadata / 调试与展开元数据**:
  - **EN**: Emits or manages metadata needed for debugging and stack unwinding
  - **CN**: 输出或管理调试与栈展开所需的元数据

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/ExecutionEngine/Orc/TargetProcess/LibraryScanner.h`, `llvm/ExecutionEngine/Orc/TargetProcess/LibraryResolver.h`, `llvm/ADT/StringExtras.h`, `llvm/Object/COFF.h`, `llvm/Object/ELF.h`, `llvm/Object/ELFObjectFile.h`, `llvm/Object/ELFTypes.h`, `llvm/Object/MachO.h`, `llvm/Object/MachOUniversal.h`, `llvm/Object/ObjectFile.h`, `llvm/Support/Error.h`, `llvm/Support/FileSystem.h` ... (+7 more)
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine, Object, Support, Target/TargetParser
