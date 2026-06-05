# LibraryResolver.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/Orc/TargetProcess/LibraryResolver.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements Library Resolution of Unresolved Symbols.
  - **CN**: 实现 ORC JIT 基础设施，例如 ExecutionSession、JITDylib、物化流程、符号查找以及执行辅助组件。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===- LibraryResolver.cpp - Library Resolution of Unresolved Symbols ---===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Library resolution impl for unresolved symbols
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 12-26
```cpp

#include "llvm/ExecutionEngine/Orc/TargetProcess/LibraryResolver.h"
#include "llvm/ExecutionEngine/Orc/TargetProcess/LibraryScanner.h"

#include "llvm/ADT/StringSet.h"

#include "llvm/BinaryFormat/MachO.h"
#include "llvm/Object/COFF.h"
#include "llvm/Object/ELF.h"
#include "llvm/Object/ELFObjectFile.h"
#include "llvm/Object/MachO.h"
#include "llvm/Object/ObjectFile.h"
#include "llvm/Support/DJB.h"
#include "llvm/Support/Error.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ExecutionEngine/Orc/TargetProcess/LibraryResolver.h`, `llvm/ExecutionEngine/Orc/TargetProcess/LibraryScanner.h`, `llvm/ADT/StringSet.h`, `llvm/BinaryFormat/MachO.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ExecutionEngine/Orc/TargetProcess/LibraryResolver.h`, `llvm/ExecutionEngine/Orc/TargetProcess/LibraryScanner.h`, `llvm/ADT/StringSet.h`, `llvm/BinaryFormat/MachO.h`。

### Lines 27-42
```cpp
#include <mutex>

#define DEBUG_TYPE "orc-resolver"

namespace llvm::orc {

LibraryResolver::LibraryResolver(const LibraryResolver::Setup &S)
    : LibMgr(LibraryManager()),
      LibPathCache(std::make_shared<LibraryPathCache>()),
      LibPathResolver(std::make_shared<PathResolver>(LibPathCache)),
      ScanHelper(S.BasePaths, LibPathCache, LibPathResolver),
      FB(S.FilterBuilder),
      ShouldScanCall(S.ShouldScanCall ? S.ShouldScanCall
                                      : [](StringRef) -> bool { return true; }),
      scanBatchSize(S.ScanBatchSize) {

```
- **EN**: Pulls in the headers needed for this implementation, including `mutex`.
- **CN**: 引入该实现所需的头文件，其中包括 `mutex`。

### Lines 43-54
```cpp
  if (!ScanHelper.hasSearchPath()) {
    LLVM_DEBUG(dbgs() << "Warning: No base paths provided for scanning.\n");
  }
}

std::unique_ptr<LibraryResolutionDriver>
LibraryResolutionDriver::create(const LibraryResolver::Setup &S) {
  auto LR = std::make_unique<LibraryResolver>(S);
  return std::unique_ptr<LibraryResolutionDriver>(
      new LibraryResolutionDriver(std::move(LR)));
}

```
- **EN**: Implements logic around `create`, `make_unique<LibraryResolver>`, `unique_ptr<LibraryResolutionDriver>`, `LibraryResolutionDriver`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `create`, `make_unique<LibraryResolver>`, `unique_ptr<LibraryResolutionDriver>`, `LibraryResolutionDriver` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 55-66
```cpp
void LibraryResolutionDriver::addScanPath(const std::string &Path, PathType K) {
  LR->ScanHelper.addBasePath(Path, K);
}

void LibraryResolutionDriver::markLibraryLoaded(StringRef Path) {
  LR->LibMgr.markLoaded(Path);
}

void LibraryResolutionDriver::markLibraryUnLoaded(StringRef Path) {
  LR->LibMgr.markUnloaded(Path);
}

```
- **EN**: Implements logic around `addScanPath`, `addBasePath`, `markLibraryLoaded`, `markLoaded`, and 2 more symbols.
- **CN**: 围绕 `addScanPath`, `addBasePath`, `markLibraryLoaded`, `markLoaded`, and 2 more symbols 实现具体逻辑。

### Lines 67-80
```cpp
void LibraryResolutionDriver::resolveSymbols(
    ArrayRef<StringRef> Symbols, LibraryResolver::OnSearchComplete OnCompletion,
    const SearchConfig &Config) {
  LR->searchSymbolsInLibraries(Symbols, std::move(OnCompletion), Config);
}

static bool shouldIgnoreSymbol(const object::SymbolRef &Sym,
                               uint32_t IgnoreFlags) {
  Expected<uint32_t> FlagsOrErr = Sym.getFlags();
  if (!FlagsOrErr) {
    consumeError(FlagsOrErr.takeError());
    return true;
  }

```
- **EN**: Implements logic around `resolveSymbols`, `searchSymbolsInLibraries`, `shouldIgnoreSymbol`, `getFlags`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `resolveSymbols`, `searchSymbolsInLibraries`, `shouldIgnoreSymbol`, `getFlags`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 81-98
```cpp
  uint32_t Flags = *FlagsOrErr;

  using Filter = SymbolEnumeratorOptions;
  if ((IgnoreFlags & Filter::IgnoreUndefined) &&
      (Flags & object::SymbolRef::SF_Undefined))
    return true;
  if ((IgnoreFlags & Filter::IgnoreNonExported) &&
      !(Flags & object::SymbolRef::SF_Exported))
    return true;
  if ((IgnoreFlags & Filter::IgnoreNonGlobal) &&
      !(Flags & object::SymbolRef::SF_Global))
    return true;
  if ((IgnoreFlags & Filter::IgnoreHidden) &&
      (Flags & object::SymbolRef::SF_Hidden))
    return true;
  if ((IgnoreFlags & Filter::IgnoreIndirect) &&
      (Flags & object::SymbolRef::SF_Indirect))
    return true;
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 99-111
```cpp
  if ((IgnoreFlags & Filter::IgnoreWeak) &&
      (Flags & object::SymbolRef::SF_Weak))
    return true;

  return false;
}

bool SymbolEnumerator::enumerateSymbols(object::ObjectFile *Obj,
                                        OnEachSymbolFn OnEach,
                                        const SymbolEnumeratorOptions &Opts) {
  if (!Obj)
    return false;

```
- **EN**: Implements logic around `enumerateSymbols`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `enumerateSymbols` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 112-123
```cpp
  auto processSymbolRange =
      [&](object::ObjectFile::symbol_iterator_range Range) -> EnumerateResult {
    for (const auto &Sym : Range) {
      if (shouldIgnoreSymbol(Sym, Opts.FilterFlags))
        continue;

      auto NameOrErr = Sym.getName();
      if (!NameOrErr) {
        consumeError(NameOrErr.takeError());
        continue;
      }

```
- **EN**: Implements logic around `getName`, `consumeError`.
- **CN**: 围绕 `getName`, `consumeError` 实现具体逻辑。

### Lines 124-134
```cpp
      StringRef Name = *NameOrErr;
      if (Name.empty())
        continue;

      EnumerateResult Res = OnEach(Name);
      if (Res != EnumerateResult::Continue)
        return Res;
    }
    return EnumerateResult::Continue;
  };

```
- **EN**: Implements logic around `OnEach`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `OnEach` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 135-152
```cpp
  EnumerateResult Res = processSymbolRange(Obj->symbols());
  if (Res != EnumerateResult::Continue)
    return Res == EnumerateResult::Stop;

  if (Obj->isELF()) {
    const auto *ElfObj = cast<object::ELFObjectFileBase>(Obj);
    Res = processSymbolRange(ElfObj->getDynamicSymbolIterators());
    if (Res != EnumerateResult::Continue)
      return Res == EnumerateResult::Stop;
  } else if (Obj->isCOFF()) {
    const auto *CoffObj = cast<object::COFFObjectFile>(Obj);
    for (auto I = CoffObj->export_directory_begin(),
              E = CoffObj->export_directory_end();
         I != E; ++I) {
      StringRef Name;
      if (I->getSymbolName(Name))
        continue;
      if (Name.empty())
```
- **EN**: Implements logic around `processSymbolRange`, `ELFObjectFileBase>`, `COFFObjectFile>`, `export_directory_end`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `processSymbolRange`, `ELFObjectFileBase>`, `COFFObjectFile>`, `export_directory_end` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 153-161
```cpp
        continue;

      EnumerateResult Res = OnEach(Name);
      if (Res != EnumerateResult::Continue)
        return Res == EnumerateResult::Stop;
    }
  } else if (Obj->isMachO()) {
  }

```
- **EN**: Implements logic around `OnEach`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `OnEach` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 162-178
```cpp
  return true;
}

bool SymbolEnumerator::enumerateSymbols(StringRef Path, OnEachSymbolFn OnEach,
                                        const SymbolEnumeratorOptions &Opts) {
  ObjectFileLoader ObjLoader(Path);

  auto ObjOrErr = ObjLoader.getObjectFile();
  if (!ObjOrErr) {
    std::string ErrMsg;
    handleAllErrors(ObjOrErr.takeError(),
                    [&](const ErrorInfoBase &EIB) { ErrMsg = EIB.message(); });
    LLVM_DEBUG(dbgs() << "Failed loading object file: " << Path
                      << "\nError: " << ErrMsg << "\n");
    return false;
  }

```
- **EN**: Implements logic around `enumerateSymbols`, `ObjLoader`, `getObjectFile`, `handleAllErrors`, and 1 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `enumerateSymbols`, `ObjLoader`, `getObjectFile`, `handleAllErrors`, and 1 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 179-191
```cpp
  return SymbolEnumerator::enumerateSymbols(&ObjOrErr.get(), OnEach, Opts);
}

static StringRef GetGnuHashSection(llvm::object::ObjectFile *file) {
  for (auto S : file->sections()) {
    StringRef name = llvm::cantFail(S.getName());
    if (name == ".gnu.hash") {
      return llvm::cantFail(S.getContents());
    }
  }
  return "";
}

```
- **EN**: Implements logic around `enumerateSymbols`, `GetGnuHashSection`, `cantFail`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `enumerateSymbols`, `GetGnuHashSection`, `cantFail` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 192-200
```cpp
/// Bloom filter is a stochastic data structure which can tell us if a symbol
/// name does not exist in a library with 100% certainty. If it tells us it
/// exists this may not be true:
/// https://blogs.oracle.com/solaris/gnu-hash-elf-sections-v2
///
/// ELF has this optimization in the new linkers by default, It is stored in the
/// gnu.hash section of the object file.
///
///\returns true if the symbol may be in the library.
```
- **EN**: Documents the next declaration group or explains a subsystem-specific rule.
- **CN**: 为后续声明分组提供说明，或解释某个子系统的专用规则。

### Lines 201-214
```cpp
static bool MayExistInElfObjectFile(llvm::object::ObjectFile *soFile,
                                    StringRef Sym) {
  assert(soFile->isELF() && "Not ELF");

  uint32_t hash = djbHash(Sym);
  // Compute the platform bitness -- either 64 or 32.
  const unsigned bits = 8 * soFile->getBytesInAddress();

  StringRef contents = GetGnuHashSection(soFile);
  if (contents.size() < 16)
    // We need to search if the library doesn't have .gnu.hash section!
    return true;
  const char *hashContent = contents.data();

```
- **EN**: Implements logic around `MayExistInElfObjectFile`, `assert`, `djbHash`, `getBytesInAddress`, and 2 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `MayExistInElfObjectFile`, `assert`, `djbHash`, `getBytesInAddress`, and 2 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 215-228
```cpp
  // See https://flapenguin.me/2017/05/10/elf-lookup-dt-gnu-hash/ for .gnu.hash
  // table layout.
  uint32_t maskWords = *reinterpret_cast<const uint32_t *>(hashContent + 8);
  uint32_t shift2 = *reinterpret_cast<const uint32_t *>(hashContent + 12);
  uint32_t hash2 = hash >> shift2;
  uint32_t n = (hash / bits) % maskWords;

  const char *bloomfilter = hashContent + 16;
  const char *hash_pos = bloomfilter + n * (bits / 8); // * (Bits / 8)
  uint64_t word = *reinterpret_cast<const uint64_t *>(hash_pos);
  uint64_t bitmask = ((1ULL << (hash % bits)) | (1ULL << (hash2 % bits)));
  return (bitmask & word) == bitmask;
}

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 229-239
```cpp
void LibraryResolver::resolveSymbolsInLibrary(
    LibraryInfo *Lib, SymbolQuery &Query, const SymbolEnumeratorOptions &Opts) {
  LLVM_DEBUG(dbgs() << "Checking unresolved symbols "
                    << " in library : " << Lib->getFileName() << "\n";);

  if (!Query.hasUnresolved()) {
    LLVM_DEBUG(dbgs() << "Skipping library: " << Lib->getFullPath()
                      << " — unresolved symbols exist.\n";);
    return;
  }

```
- **EN**: Implements logic around `resolveSymbolsInLibrary`, `getFileName`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `resolveSymbolsInLibrary`, `getFileName` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 240-248
```cpp
  bool HadAnySym = false;

  // Build candidate vector
  SmallVector<StringRef, 24> CandidateVec;

  Query.getUnresolvedSymbols(CandidateVec, [&](StringRef S) {
    return !Lib->hasFilter() || Lib->mayContain(S);
  });

```
- **EN**: Implements logic around `getUnresolvedSymbols`, `hasFilter`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getUnresolvedSymbols`, `hasFilter` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 249-257
```cpp
  LLVM_DEBUG(dbgs() << "Total candidate symbols : " << CandidateVec.size()
                    << "\n";);
  if (CandidateVec.empty()) {
    LLVM_DEBUG(dbgs() << "No symbol Exist "
                         " in library: "
                      << Lib->getFullPath() << "\n";);
    return;
  }

```
- **EN**: Implements logic around `getFullPath`.
- **CN**: 围绕 `getFullPath` 实现具体逻辑。

### Lines 258-270
```cpp
  bool BuildingFilter = !Lib->hasFilter();

  ObjectFileLoader ObjLoader(Lib->getFullPath());
  auto ObjOrErr = ObjLoader.getObjectFile();
  if (!ObjOrErr) {
    std::string ErrMsg;
    handleAllErrors(ObjOrErr.takeError(),
                    [&](const ErrorInfoBase &EIB) { ErrMsg = EIB.message(); });
    LLVM_DEBUG(dbgs() << "Failed loading object file: " << Lib->getFullPath()
                      << "\nError: " << ErrMsg << "\n");
    return;
  }

```
- **EN**: Implements logic around `hasFilter`, `ObjLoader`, `getObjectFile`, `handleAllErrors`, and 1 more symbols.
- **CN**: 围绕 `hasFilter`, `ObjLoader`, `getObjectFile`, `handleAllErrors`, and 1 more symbols 实现具体逻辑。

### Lines 271-279
```cpp
  object::ObjectFile *Obj = &ObjOrErr.get();
  if (BuildingFilter && Obj->isELF()) {

    erase_if(CandidateVec,
             [&](StringRef C) { return !MayExistInElfObjectFile(Obj, C); });
    if (CandidateVec.empty())
      return;
  }

```
- **EN**: Implements logic around `get`, `erase_if`, `MayExistInElfObjectFile`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `get`, `erase_if`, `MayExistInElfObjectFile` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 280-291
```cpp
  SmallVector<StringRef, 256> SymbolVec;

  LLVM_DEBUG(dbgs() << "Enumerating symbols in library: " << Lib->getFullPath()
                    << "\n";);

  SymbolEnumerator::enumerateSymbols(
      Obj,
      [&](StringRef S) {
        // Collect symbols if we're building a filter
        if (BuildingFilter)
          SymbolVec.push_back(S);

```
- **EN**: Implements logic around `enumerateSymbols`, `push_back`.
- **CN**: 围绕 `enumerateSymbols`, `push_back` 实现具体逻辑。

### Lines 292-303
```cpp
        // auto It = std::lower_bound(CandidateVec.begin(),
        // CandidateVec.end(), S);
        auto It = std::find(CandidateVec.begin(), CandidateVec.end(), S);
        if (It != CandidateVec.end() && *It == S) {
          // Resolve and remove from CandidateVec
          LLVM_DEBUG(dbgs() << "Symbol '" << S << "' resolved in library: "
                            << Lib->getFullPath() << "\n";);
          Query.resolve(S, Lib->getFullPath());
          HadAnySym = true;
          *It = CandidateVec.back();
          CandidateVec.pop_back();

```
- **EN**: Implements logic around `find`, `getFullPath`, `resolve`, `back`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `find`, `getFullPath`, `resolve`, `back`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 304-312
```cpp
          // Stop — if nothing remains, stop enumeration
          if (!BuildingFilter && CandidateVec.empty()) {
            return EnumerateResult::Stop;
          }
          // Also stop if SymbolQuery has no more unresolved symbols
          if (!BuildingFilter && !Query.hasUnresolved())
            return EnumerateResult::Stop;
        }

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 313-325
```cpp
        return EnumerateResult::Continue;
      },
      Opts);

  if (BuildingFilter) {
    LLVM_DEBUG(dbgs() << "Building filter for library: " << Lib->getFullPath()
                      << "\n";);
    if (SymbolVec.empty()) {
      LLVM_DEBUG(dbgs() << "  Skip : No symbols found in : "
                        << Lib->getFullPath() << "\n";);
      return;
    }

```
- **EN**: Implements logic around `getFullPath`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getFullPath` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 326-337
```cpp
    Lib->ensureFilterBuilt(FB, SymbolVec);
    LLVM_DEBUG({
      dbgs() << "DiscoveredSymbols : " << SymbolVec.size() << "\n";
      for (const auto &S : SymbolVec)
        dbgs() << "DiscoveredSymbols : " << S << "\n";
    });
  }

  if (HadAnySym && Lib->getState() != LibState::Loaded)
    Lib->setState(LibState::Queried);
}

```
- **EN**: Implements logic around `ensureFilterBuilt`, `dbgs`, `setState`.
- **CN**: 围绕 `ensureFilterBuilt`, `dbgs`, `setState` 实现具体逻辑。

### Lines 338-347
```cpp
void LibraryResolver::searchSymbolsInLibraries(ArrayRef<StringRef> SymbolList,
                                               OnSearchComplete OnComplete,
                                               const SearchConfig &Config) {
  SymbolQuery Q(SymbolList);

  using LibraryType = PathType;
  auto tryResolveFrom = [&](LibState S, LibraryType K) {
    LLVM_DEBUG(dbgs() << "Trying resolve from state=" << static_cast<int>(S)
                      << " type=" << static_cast<int>(K) << "\n";);

```
- **EN**: Implements logic around `searchSymbolsInLibraries`, `Q`, `static_cast<int>`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `searchSymbolsInLibraries`, `Q`, `static_cast<int>` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 348-357
```cpp
    LibraryCursor Cur = LibMgr.getCursor(K, S);
    while (!Q.allResolved()) {
      const LibraryInfo *Lib = Cur.nextValidLib();
      // Cursor not valid?
      if (!Lib) {
        if (!scanForNewLibraries(K, Cur))
          break;  // nothing new was added
        continue; // Try to resolve next library
      }

```
- **EN**: Implements logic around `getCursor`, `nextValidLib`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `getCursor`, `nextValidLib` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 358-371
```cpp
      // can use Async here?
      resolveSymbolsInLibrary(const_cast<LibraryInfo *>(Lib), Q,
                              Config.Options);
      if (Q.allResolved())
        break;
    }
  };

  for (const auto &[St, Ty] : Config.Policy.Plan) {
    tryResolveFrom(St, Ty);
    if (Q.allResolved())
      break;
  }

```
- **EN**: Implements logic around `resolveSymbolsInLibrary`, `tryResolveFrom`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `resolveSymbolsInLibrary`, `tryResolveFrom` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 372-382
```cpp
  // done:
  LLVM_DEBUG({
    dbgs() << "Search complete.\n";
    for (const auto &r : Q.getAllResults())
      dbgs() << "Resolved Symbol:" << r->Name << " -> " << r->ResolvedLibPath
             << "\n";
  });

  OnComplete(Q);
}

```
- **EN**: Implements logic around `dbgs`, `OnComplete`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `dbgs`, `OnComplete` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 383-391
```cpp
bool LibraryResolver::scanForNewLibraries(PathType K, LibraryCursor &Cur) {
  while (ScanHelper.leftToScan(K)) {
    scanLibrariesIfNeeded(K, scanBatchSize);

    // Check if scanning added new libraries
    if (Cur.hasMoreValidLib())
      return true;
  }

```
- **EN**: Implements logic around `scanForNewLibraries`, `scanLibrariesIfNeeded`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `scanForNewLibraries`, `scanLibrariesIfNeeded` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 392-402
```cpp
  // No new libraries were added
  return false;
}

bool LibraryResolver::scanLibrariesIfNeeded(PathType PK, size_t BatchSize) {
  LLVM_DEBUG(dbgs() << "LibraryResolver::scanLibrariesIfNeeded: Scanning for "
                    << (PK == PathType::User ? "User" : "System")
                    << " libraries\n";);
  if (!ScanHelper.leftToScan(PK))
    return false;

```
- **EN**: Implements logic around `scanLibrariesIfNeeded`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `scanLibrariesIfNeeded` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 403-407
```cpp
  LibraryScanner Scanner(ScanHelper, LibMgr, ShouldScanCall);
  Scanner.scanNext(PK, BatchSize);
  return true;
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

- **Direct includes / 直接包含**: `llvm/ExecutionEngine/Orc/TargetProcess/LibraryResolver.h`, `llvm/ExecutionEngine/Orc/TargetProcess/LibraryScanner.h`, `llvm/ADT/StringSet.h`, `llvm/BinaryFormat/MachO.h`, `llvm/Object/COFF.h`, `llvm/Object/ELF.h`, `llvm/Object/ELFObjectFile.h`, `llvm/Object/MachO.h`, `llvm/Object/ObjectFile.h`, `llvm/Support/DJB.h`, `llvm/Support/Error.h`, `mutex`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine, Object, Support, BinaryFormat
