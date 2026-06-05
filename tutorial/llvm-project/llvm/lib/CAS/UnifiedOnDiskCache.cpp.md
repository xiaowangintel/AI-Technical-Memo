# UnifiedOnDiskCache.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/lib/CAS/UnifiedOnDiskCache.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: \file Encapsulates \p OnDiskGraphDB and \p OnDiskKeyValueDB instances within one directory while also restricting storage growth with a scheme of chaining the two most recent directories (primary & upstream), where the primary "faults-in" data from the upstream one. When the primary (most recent) directory exceeds its intended limit a new empty directory becomes the primary one.
  - **CN**: 实现内容寻址存储、磁盘缓存以及相关 schema 支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10
```cpp
//===----------------------------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// Encapsulates \p OnDiskGraphDB and \p OnDiskKeyValueDB instances within one
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件的高层描述。

### Lines 11-20
```cpp
/// directory while also restricting storage growth with a scheme of chaining
/// the two most recent directories (primary & upstream), where the primary
/// "faults-in" data from the upstream one. When the primary (most recent)
/// directory exceeds its intended limit a new empty directory becomes the
/// primary one.
///
/// Within the top-level directory (the path that \p UnifiedOnDiskCache::open
/// receives) there are directories named like this:
///
/// 'v<version>.<x>'
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 21-30
```cpp
/// 'v<version>.<x+1>'
/// 'v<version>.<x+2>'
/// ...
///
/// 'version' is the version integer for this \p UnifiedOnDiskCache's scheme and
/// the part after the dot is an increasing integer. The primary directory is
/// the one with the highest integer and the upstream one is the directory
/// before it. For example, if the sub-directories contained are:
///
/// 'v1.5', 'v1.6', 'v1.7', 'v1.8'
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 31-40
```cpp
///
/// Then the primary one is 'v1.8', the upstream one is 'v1.7', and the rest are
/// unused directories that can be safely deleted at any time and by any
/// process.
///
/// Contained within the top-level directory is a file named "lock" which is
/// used for processes to take shared or exclusive locks for the contents of the
/// top directory. While a \p UnifiedOnDiskCache is open it keeps a shared lock
/// for the top-level directory; when it closes, if the primary sub-directory
/// exceeded its limit, it attempts to get an exclusive lock in order to create
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 41-50
```cpp
/// a new empty primary directory; if it can't get the exclusive lock it gives
/// up and lets the next \p UnifiedOnDiskCache instance that closes to attempt
/// again.
///
/// The downside of this scheme is that while \p UnifiedOnDiskCache is open on a
/// directory, by any process, the storage size in that directory will keep
/// growing unrestricted. But the major benefit is that garbage-collection can
/// be triggered on a directory concurrently, at any time and by any process,
/// without affecting any active readers/writers in the same process or other
/// processes.
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 51-60
```cpp
///
/// The \c UnifiedOnDiskCache also provides validation and recovery on top of
/// the underlying on-disk storage. The low-level storage is designed to remain
/// coherent across regular process crashes, but may be invalid after power loss
/// or similar system failures. \c UnifiedOnDiskCache::validateIfNeeded allows
/// validating the contents once per boot and can recover by marking invalid
/// data for garbage collection.
///
/// The data recovery described above requires exclusive access to the CAS, and
/// it is an error to attempt recovery if the CAS is open in any process/thread.
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 61-80
```cpp
/// In order to maximize backwards compatibility with tools that do not perform
/// validation before opening the CAS, we do not attempt to get exclusive access
/// until recovery is actually performed, meaning as long as the data is valid
/// it will not conflict with concurrent use.
//
//===----------------------------------------------------------------------===//

#include "llvm/CAS/UnifiedOnDiskCache.h"
#include "OnDiskCommon.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/ScopeExit.h"
#include "llvm/ADT/SmallString.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/CAS/OnDiskCASLogger.h"
#include "llvm/CAS/OnDiskGraphDB.h"
#include "llvm/CAS/OnDiskKeyValueDB.h"
#include "llvm/Support/Compiler.h"
#include "llvm/Support/Errc.h"
```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/CAS/UnifiedOnDiskCache.h`, `OnDiskCommon.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/ScopeExit.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/CAS/UnifiedOnDiskCache.h`, `OnDiskCommon.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/ScopeExit.h`。

### Lines 81-90
```cpp
#include "llvm/Support/Error.h"
#include "llvm/Support/FileSystem.h"
#include "llvm/Support/FileUtilities.h"
#include "llvm/Support/IOSandbox.h"
#include "llvm/Support/MemoryBuffer.h"
#include "llvm/Support/Path.h"
#include "llvm/Support/Program.h"
#include "llvm/Support/raw_ostream.h"
#include <optional>

```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/Support/Error.h`, `llvm/Support/FileSystem.h`, `llvm/Support/FileUtilities.h`, `llvm/Support/IOSandbox.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/Support/Error.h`, `llvm/Support/FileSystem.h`, `llvm/Support/FileUtilities.h`, `llvm/Support/IOSandbox.h`。

### Lines 91-102
```cpp
using namespace llvm;
using namespace llvm::cas;
using namespace llvm::cas::ondisk;

/// FIXME: When the version of \p DBDirPrefix is bumped up we need to figure out
/// how to handle the leftover sub-directories of the previous version, within
/// the \p UnifiedOnDiskCache::collectGarbage function.
static constexpr StringLiteral DBDirPrefix = "v1.";

static constexpr StringLiteral ValidationFilename = "v1.validation";
static constexpr StringLiteral CorruptPrefix = "corrupt.";

```
- **EN**: Introduces declarations for `llvm`, `llvm::cas`, `llvm::cas::ondisk`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `llvm`, `llvm::cas`, `llvm::cas::ondisk` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 103-117
```cpp
ObjectID UnifiedOnDiskCache::getObjectIDFromValue(ArrayRef<char> Value) {
  // little endian encoded.
  assert(Value.size() == sizeof(uint64_t));
  return ObjectID::fromOpaqueData(support::endian::read64le(Value.data()));
}

UnifiedOnDiskCache::ValueBytes
UnifiedOnDiskCache::getValueFromObjectID(ObjectID ID) {
  // little endian encoded.
  UnifiedOnDiskCache::ValueBytes ValBytes;
  static_assert(ValBytes.size() == sizeof(ID.getOpaqueData()));
  support::endian::write64le(ValBytes.data(), ID.getOpaqueData());
  return ValBytes;
}

```
- **EN**: Implements logic around `getObjectIDFromValue`, `assert`, `fromOpaqueData`, `getValueFromObjectID`, and 2 more symbols; this block emits or serializes data to an external representation; works with hashed storage or cache state.
- **CN**: 围绕 `getObjectIDFromValue`, `assert`, `fromOpaqueData`, `getValueFromObjectID`, and 2 more symbols 实现具体逻辑；该代码块把数据输出或序列化为外部表示，并处理基于哈希的存储或缓存状态。

### Lines 118-128
```cpp
Expected<std::optional<ArrayRef<char>>>
UnifiedOnDiskCache::faultInFromUpstreamKV(ArrayRef<uint8_t> Key) {
  assert(UpstreamGraphDB);
  assert(UpstreamKVDB);

  std::optional<ArrayRef<char>> UpstreamValue;
  if (Error E = UpstreamKVDB->get(Key).moveInto(UpstreamValue))
    return std::move(E);
  if (!UpstreamValue)
    return std::nullopt;

```
- **EN**: Implements logic around `faultInFromUpstreamKV`, `assert`, `get`, `move`; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `faultInFromUpstreamKV`, `assert`, `get`, `move` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 129-139
```cpp
  // The value is the \p ObjectID in the context of the upstream
  // \p OnDiskGraphDB instance. Translate it to the context of the primary
  // \p OnDiskGraphDB instance.
  ObjectID UpstreamID = getObjectIDFromValue(*UpstreamValue);
  auto PrimaryID =
      PrimaryGraphDB->getReference(UpstreamGraphDB->getDigest(UpstreamID));
  if (LLVM_UNLIKELY(!PrimaryID))
    return PrimaryID.takeError();
  return PrimaryKVDB->put(Key, getValueFromObjectID(*PrimaryID));
}

```
- **EN**: Implements logic around `getObjectIDFromValue`, `getReference`, `takeError`, `put`; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `getObjectIDFromValue`, `getReference`, `takeError`, `put` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 140-150
```cpp
/// \returns all the 'v<version>.<x>' names of sub-directories, sorted with
/// ascending order of the integer after the dot. Corrupt directories, if
/// included, will come first.
static Expected<SmallVector<std::string, 4>>
getAllDBDirs(StringRef Path, bool IncludeCorrupt = false) {
  struct DBDir {
    uint64_t Order;
    std::string Name;
  };
  SmallVector<DBDir> FoundDBDirs;

```
- **EN**: Introduces declarations for `DBDir`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `DBDir` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 151-170
```cpp
  std::error_code EC;
  for (sys::fs::directory_iterator DirI(Path, EC), DirE; !EC && DirI != DirE;
       DirI.increment(EC)) {
    if (DirI->type() != sys::fs::file_type::directory_file)
      continue;
    StringRef SubDir = sys::path::filename(DirI->path());
    if (IncludeCorrupt && SubDir.starts_with(CorruptPrefix)) {
      FoundDBDirs.push_back({0, std::string(SubDir)});
      continue;
    }
    if (!SubDir.starts_with(DBDirPrefix))
      continue;
    uint64_t Order;
    if (SubDir.substr(DBDirPrefix.size()).getAsInteger(10, Order))
      return createStringError(inconvertibleErrorCode(),
                               "unexpected directory " + DirI->path());
    FoundDBDirs.push_back({Order, std::string(SubDir)});
  }
  if (EC)
    return createFileError(Path, EC);
```
- **EN**: Implements logic around `DirI`, `increment`, `type`, `filename`, and 6 more symbols; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `DirI`, `increment`, `type`, `filename`, and 6 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 171-181
```cpp

  llvm::sort(FoundDBDirs, [](const DBDir &LHS, const DBDir &RHS) -> bool {
    return LHS.Order < RHS.Order;
  });

  SmallVector<std::string, 4> DBDirs;
  for (DBDir &Dir : FoundDBDirs)
    DBDirs.push_back(std::move(Dir.Name));
  return DBDirs;
}

```
- **EN**: Implements logic around `sort`, `push_back`; this block works with hashed storage or cache state.
- **CN**: 围绕 `sort`, `push_back` 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 182-198
```cpp
static Expected<SmallVector<std::string, 4>> getAllGarbageDirs(StringRef Path) {
  auto DBDirs = getAllDBDirs(Path, /*IncludeCorrupt=*/true);
  if (!DBDirs)
    return DBDirs.takeError();

  // FIXME: When the version of \p DBDirPrefix is bumped up we need to figure
  // out how to handle the leftover sub-directories of the previous version.

  for (unsigned Keep = 2; Keep > 0 && !DBDirs->empty(); --Keep) {
    StringRef Back(DBDirs->back());
    if (Back.starts_with(CorruptPrefix))
      break;
    DBDirs->pop_back();
  }
  return *DBDirs;
}

```
- **EN**: Implements logic around `getAllGarbageDirs`, `getAllDBDirs`, `takeError`, `empty`, and 3 more symbols; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `getAllGarbageDirs`, `getAllDBDirs`, `takeError`, `empty`, and 3 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 199-209
```cpp
/// \returns Given a sub-directory named 'v<version>.<x>', it outputs the
/// 'v<version>.<x+1>' name.
static void getNextDBDirName(StringRef DBDir, llvm::raw_ostream &OS) {
  assert(DBDir.starts_with(DBDirPrefix));
  uint64_t Count;
  bool Failed = DBDir.substr(DBDirPrefix.size()).getAsInteger(10, Count);
  assert(!Failed);
  (void)Failed;
  OS << DBDirPrefix << Count + 1;
}

```
- **EN**: Implements logic around `getNextDBDirName`, `assert`, `substr`; this block works with hashed storage or cache state.
- **CN**: 围绕 `getNextDBDirName`, `assert`, `substr` 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 210-223
```cpp
static Error validateOutOfProcess(StringRef LLVMCasBinary, StringRef RootPath,
                                  bool CheckHash) {
  SmallVector<StringRef> Args{LLVMCasBinary, "-cas", RootPath, "-validate"};
  if (CheckHash)
    Args.push_back("-check-hash");

  llvm::SmallString<128> StdErrPath;
  int StdErrFD = -1;
  if (std::error_code EC = sys::fs::createTemporaryFile(
          "llvm-cas-validate-stderr", "txt", StdErrFD, StdErrPath,
          llvm::sys::fs::OF_Text))
    return createStringError(EC, "failed to create temporary file");
  FileRemover OutputRemover(StdErrPath.c_str());

```
- **EN**: Implements logic around `validateOutOfProcess`, `push_back`, `createTemporaryFile`, `createStringError`, and 1 more symbols; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `validateOutOfProcess`, `push_back`, `createTemporaryFile`, `createStringError`, and 1 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 224-234
```cpp
  std::optional<llvm::StringRef> Redirects[] = {
      {""}, // stdin = /dev/null
      {""}, // stdout = /dev/null
      StdErrPath.str(),
  };

  std::string ErrMsg;
  int Result =
      sys::ExecuteAndWait(LLVMCasBinary, Args, /*Env=*/std::nullopt, Redirects,
                          /*SecondsToWait=*/120, /*MemoryLimit=*/0, &ErrMsg);

```
- **EN**: Implements logic around `str`, `ExecuteAndWait`; this block works with hashed storage or cache state.
- **CN**: 围绕 `str`, `ExecuteAndWait` 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 235-253
```cpp
  if (Result == -1)
    return createStringError("failed to exec " + join(Args, " ") + ": " +
                             ErrMsg);
  if (Result != 0) {
    llvm::SmallString<64> Err("cas contents invalid");
    if (!ErrMsg.empty()) {
      Err += ": ";
      Err += ErrMsg;
    }
    auto StdErrBuf = MemoryBuffer::getFile(StdErrPath.c_str());
    if (StdErrBuf && !(*StdErrBuf)->getBuffer().empty()) {
      Err += ": ";
      Err += (*StdErrBuf)->getBuffer();
    }
    return createStringError(Err);
  }
  return Error::success();
}

```
- **EN**: Implements logic around `createStringError`, `Err`, `empty`, `getFile`, and 2 more symbols; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `createStringError`, `Err`, `empty`, `getFile`, and 2 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 254-272
```cpp
Error UnifiedOnDiskCache::validateActionCache() const {
  return getKeyValueDB().validate();
}

static Error validateInProcess(StringRef RootPath, StringRef HashName,
                               unsigned HashByteSize, bool CheckHash,
                               OnDiskGraphDB::HashingFuncT HashFn) {
  std::shared_ptr<UnifiedOnDiskCache> UniDB;
  if (Error E = UnifiedOnDiskCache::open(RootPath, std::nullopt, HashName,
                                         HashByteSize)
                    .moveInto(UniDB))
    return E;
  if (Error E = UniDB->getGraphDB().validate(CheckHash, HashFn))
    return E;
  if (Error E = UniDB->validateActionCache())
    return E;
  return Error::success();
}

```
- **EN**: Implements logic around `validateActionCache`, `getKeyValueDB`, `validateInProcess`, `open`, and 3 more symbols; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `validateActionCache`, `getKeyValueDB`, `validateInProcess`, `open`, and 3 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 273-287
```cpp
Expected<ValidationResult> UnifiedOnDiskCache::validateIfNeeded(
    StringRef RootPath, StringRef HashName, unsigned HashByteSize,
    bool CheckHash, OnDiskGraphDB::HashingFuncT HashFn, bool AllowRecovery,
    bool ForceValidation, std::optional<StringRef> LLVMCasBinaryPath) {
  if (std::error_code EC = sys::fs::create_directories(RootPath))
    return createFileError(RootPath, EC);

  SmallString<256> PathBuf(RootPath);
  sys::path::append(PathBuf, ValidationFilename);
  int FD = -1;
  if (std::error_code EC = sys::fs::openFileForReadWrite(
          PathBuf, FD, sys::fs::CD_OpenAlways, sys::fs::OF_None))
    return createFileError(PathBuf, EC);
  assert(FD != -1);

```
- **EN**: Implements logic around `validateIfNeeded`, `create_directories`, `createFileError`, `PathBuf`, and 3 more symbols; this block works with hashed storage or cache state.
- **CN**: 围绕 `validateIfNeeded`, `create_directories`, `createFileError`, `PathBuf`, and 3 more symbols 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 288-301
```cpp
  sys::fs::file_t File = sys::fs::convertFDToNativeFile(FD);
  llvm::scope_exit CloseFile([&]() { sys::fs::closeFile(File); });

  if (std::error_code EC = lockFileThreadSafe(FD, sys::fs::LockKind::Exclusive))
    return createFileError(PathBuf, EC);
  llvm::scope_exit UnlockFD([&]() { unlockFileThreadSafe(FD); });

  std::shared_ptr<ondisk::OnDiskCASLogger> Logger;
#ifndef _WIN32
  if (Error E =
          ondisk::OnDiskCASLogger::openIfEnabled(RootPath).moveInto(Logger))
    return std::move(E);
#endif

```
- **EN**: Defines preprocessor-controlled structure, feature gates, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、特性开关或编译期常量。

### Lines 302-311
```cpp
  SmallString<8> Bytes;
  if (Error E = sys::fs::readNativeFileToEOF(File, Bytes))
    return createFileError(PathBuf, std::move(E));

  uint64_t ValidationBootTime = 0;
  if (!Bytes.empty() &&
      StringRef(Bytes).trim().getAsInteger(10, ValidationBootTime))
    return createFileError(PathBuf, errc::illegal_byte_sequence,
                           "expected integer");

```
- **EN**: Implements logic around `readNativeFileToEOF`, `createFileError`, `empty`, `StringRef`; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `readNativeFileToEOF`, `createFileError`, `empty`, `StringRef` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 312-329
```cpp
  static uint64_t BootTime = 0;
  if (BootTime == 0)
    if (Error E = getBootTime().moveInto(BootTime))
      return std::move(E);

  bool Recovered = false;
  bool Skipped = false;
  std::string LogValidationError;

  llvm::scope_exit Log([&] {
    if (!Logger)
      return;
    Logger->logUnifiedOnDiskCacheValidateIfNeeded(
        RootPath, BootTime, ValidationBootTime, CheckHash, AllowRecovery,
        ForceValidation, LLVMCasBinaryPath, LogValidationError, Skipped,
        Recovered);
  });

```
- **EN**: Implements logic around `getBootTime`, `move`, `Log`, `logUnifiedOnDiskCacheValidateIfNeeded`; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `getBootTime`, `move`, `Log`, `logUnifiedOnDiskCacheValidateIfNeeded` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 330-349
```cpp
  if (ValidationBootTime == BootTime && !ForceValidation) {
    Skipped = true;
    return ValidationResult::Skipped;
  }

  // Validate!
  bool NeedsRecovery = false;
  Error E = LLVMCasBinaryPath
                ? validateOutOfProcess(*LLVMCasBinaryPath, RootPath, CheckHash)
                : validateInProcess(RootPath, HashName, HashByteSize, CheckHash,
                                    HashFn);
  if (E) {
    if (Logger)
      LogValidationError = toStringWithoutConsuming(E);
    if (AllowRecovery) {
      consumeError(std::move(E));
      NeedsRecovery = true;
    } else {
      return std::move(E);
    }
```
- **EN**: Implements logic around `validateOutOfProcess`, `validateInProcess`, `toStringWithoutConsuming`, `consumeError`, and 1 more symbols; this block propagates recoverable errors through LLVM error utilities; parses or classifies structured input; works with hashed storage or cache state.
- **CN**: 围绕 `validateOutOfProcess`, `validateInProcess`, `toStringWithoutConsuming`, `consumeError`, and 1 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并解析或分类结构化输入，并处理基于哈希的存储或缓存状态。

### Lines 350-369
```cpp
  }

  if (NeedsRecovery) {
    sys::path::remove_filename(PathBuf);
    sys::path::append(PathBuf, "lock");

    int LockFD = -1;
    if (std::error_code EC = sys::fs::openFileForReadWrite(
            PathBuf, LockFD, sys::fs::CD_OpenAlways, sys::fs::OF_None))
      return createFileError(PathBuf, EC);
    sys::fs::file_t LockFile = sys::fs::convertFDToNativeFile(LockFD);
    llvm::scope_exit CloseLock([&]() { sys::fs::closeFile(LockFile); });
    if (std::error_code EC = tryLockFileThreadSafe(LockFD)) {
      if (EC == std::errc::no_lock_available)
        return createFileError(
            PathBuf, EC,
            "CAS validation requires exclusive access but CAS was in use");
      return createFileError(PathBuf, EC);
    }
    llvm::scope_exit UnlockFD([&]() { unlockFileThreadSafe(LockFD); });
```
- **EN**: Implements logic around `remove_filename`, `append`, `openFileForReadWrite`, `createFileError`, and 4 more symbols; this block works with hashed storage or cache state.
- **CN**: 围绕 `remove_filename`, `append`, `openFileForReadWrite`, `createFileError`, and 4 more symbols 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 370-389
```cpp

    auto DBDirs = getAllDBDirs(RootPath);
    if (!DBDirs)
      return DBDirs.takeError();

    for (StringRef DBDir : *DBDirs) {
      sys::path::remove_filename(PathBuf);
      sys::path::append(PathBuf, DBDir);
      std::error_code EC;
      int Attempt = 0, MaxAttempts = 100;
      SmallString<128> GCPath;
      for (; Attempt < MaxAttempts; ++Attempt) {
        GCPath.assign(RootPath);
        sys::path::append(GCPath, CorruptPrefix + std::to_string(Attempt) +
                                      "." + DBDir);
        EC = sys::fs::rename(PathBuf, GCPath);
        // Darwin uses ENOTEMPTY. Linux may return either ENOTEMPTY or EEXIST.
        if (EC != errc::directory_not_empty && EC != errc::file_exists)
          break;
      }
```
- **EN**: Implements logic around `getAllDBDirs`, `takeError`, `remove_filename`, `append`, and 2 more symbols; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `getAllDBDirs`, `takeError`, `remove_filename`, `append`, and 2 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 390-400
```cpp
      if (Attempt == MaxAttempts)
        return createStringError(
            EC, "rename " + PathBuf +
                    " failed: too many CAS directories awaiting pruning");
      if (EC)
        return createStringError(EC, "rename " + PathBuf + " to " + GCPath +
                                         " failed: " + EC.message());
    }
    Recovered = true;
  }

```
- **EN**: Implements logic around `createStringError`, `message`; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `createStringError`, `message` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 401-413
```cpp
  if (ValidationBootTime != BootTime) {
    // Fix filename in case we have error to report.
    sys::path::remove_filename(PathBuf);
    sys::path::append(PathBuf, ValidationFilename);
    if (std::error_code EC = sys::fs::resize_file(FD, 0))
      return createFileError(PathBuf, EC);
    raw_fd_ostream OS(FD, /*shouldClose=*/false);
    OS.seek(0); // resize does not reset position
    OS << BootTime << '\n';
    if (OS.has_error())
      return createFileError(PathBuf, OS.error());
  }

```
- **EN**: Implements logic around `remove_filename`, `append`, `resize_file`, `createFileError`, and 3 more symbols; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `remove_filename`, `append`, `resize_file`, `createFileError`, and 3 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 414-425
```cpp
  return NeedsRecovery ? ValidationResult::Recovered : ValidationResult::Valid;
}

Expected<std::unique_ptr<UnifiedOnDiskCache>>
UnifiedOnDiskCache::open(StringRef RootPath, std::optional<uint64_t> SizeLimit,
                         StringRef HashName, unsigned HashByteSize,
                         OnDiskGraphDB::FaultInPolicy FaultInPolicy) {
  auto BypassSandbox = sys::sandbox::scopedDisable();

  if (std::error_code EC = sys::fs::create_directories(RootPath))
    return createFileError(RootPath, EC);

```
- **EN**: Implements logic around `open`, `scopedDisable`, `create_directories`, `createFileError`; this block works with hashed storage or cache state.
- **CN**: 围绕 `open`, `scopedDisable`, `create_directories`, `createFileError` 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 426-440
```cpp
  SmallString<256> PathBuf(RootPath);
  sys::path::append(PathBuf, "lock");
  int LockFD = -1;
  if (std::error_code EC = sys::fs::openFileForReadWrite(
          PathBuf, LockFD, sys::fs::CD_OpenAlways, sys::fs::OF_None))
    return createFileError(PathBuf, EC);
  assert(LockFD != -1);
  // Locking the directory using shared lock, which will prevent other processes
  // from creating a new chain (essentially while a \p UnifiedOnDiskCache
  // instance holds a shared lock the storage for the primary directory will
  // grow unrestricted).
  if (std::error_code EC =
          lockFileThreadSafe(LockFD, sys::fs::LockKind::Shared))
    return createFileError(PathBuf, EC);

```
- **EN**: Implements logic around `PathBuf`, `append`, `openFileForReadWrite`, `createFileError`, and 2 more symbols; this block works with hashed storage or cache state.
- **CN**: 围绕 `PathBuf`, `append`, `openFileForReadWrite`, `createFileError`, and 2 more symbols 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 441-453
```cpp
  auto DBDirs = getAllDBDirs(RootPath);
  if (!DBDirs)
    return DBDirs.takeError();
  if (DBDirs->empty())
    DBDirs->push_back((Twine(DBDirPrefix) + "1").str());

  std::shared_ptr<ondisk::OnDiskCASLogger> Logger;
#ifndef _WIN32
  if (Error E =
          ondisk::OnDiskCASLogger::openIfEnabled(RootPath).moveInto(Logger))
    return std::move(E);
#endif

```
- **EN**: Defines preprocessor-controlled structure, feature gates, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、特性开关或编译期常量。

### Lines 454-473
```cpp
  /// If there is only one directory open databases on it. If there are 2 or
  /// more directories, get the most recent directories and chain them, with the
  /// most recent being the primary one. The remaining directories are unused
  /// data than can be garbage-collected.
  auto UniDB = std::unique_ptr<UnifiedOnDiskCache>(new UnifiedOnDiskCache());
  std::unique_ptr<OnDiskGraphDB> UpstreamGraphDB;
  std::unique_ptr<OnDiskKeyValueDB> UpstreamKVDB;
  if (DBDirs->size() > 1) {
    StringRef UpstreamDir = *(DBDirs->end() - 2);
    PathBuf = RootPath;
    sys::path::append(PathBuf, UpstreamDir);
    if (Error E =
            OnDiskGraphDB::open(PathBuf, HashName, HashByteSize,
                                /*UpstreamDB=*/nullptr, Logger, FaultInPolicy)
                .moveInto(UpstreamGraphDB))
      return std::move(E);
    if (Error E = OnDiskKeyValueDB::open(PathBuf, HashName, HashByteSize,
                                         /*ValueName=*/"objectid",
                                         /*ValueSize=*/sizeof(uint64_t),
                                         /*UnifiedCache=*/nullptr, Logger)
```
- **EN**: Implements logic around `unique_ptr`, `size`, `end`, `append`, and 3 more symbols; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `unique_ptr`, `size`, `end`, `append`, and 3 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 474-493
```cpp
                      .moveInto(UpstreamKVDB))
      return std::move(E);
  }

  StringRef PrimaryDir = *(DBDirs->end() - 1);
  PathBuf = RootPath;
  sys::path::append(PathBuf, PrimaryDir);
  std::unique_ptr<OnDiskGraphDB> PrimaryGraphDB;
  if (Error E =
          OnDiskGraphDB::open(PathBuf, HashName, HashByteSize,
                              UpstreamGraphDB.get(), Logger, FaultInPolicy)
              .moveInto(PrimaryGraphDB))
    return std::move(E);
  std::unique_ptr<OnDiskKeyValueDB> PrimaryKVDB;
  // \p UnifiedOnDiskCache does manual chaining for key-value requests,
  // including an extra translation step of the value during fault-in.
  if (Error E = OnDiskKeyValueDB::open(PathBuf, HashName, HashByteSize,
                                       /*ValueName=*/"objectid",
                                       /*ValueSize=*/sizeof(uint64_t),
                                       UniDB.get(), Logger)
```
- **EN**: Implements logic around `moveInto`, `move`, `end`, `append`, and 2 more symbols; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `moveInto`, `move`, `end`, `append`, and 2 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 494-507
```cpp
                    .moveInto(PrimaryKVDB))
    return std::move(E);

  UniDB->RootPath = RootPath;
  UniDB->SizeLimit = SizeLimit.value_or(0);
  UniDB->LockFD = LockFD;
  UniDB->NeedsGarbageCollection = DBDirs->size() > 2;
  UniDB->PrimaryDBDir = PrimaryDir;
  UniDB->UpstreamGraphDB = std::move(UpstreamGraphDB);
  UniDB->PrimaryGraphDB = std::move(PrimaryGraphDB);
  UniDB->UpstreamKVDB = std::move(UpstreamKVDB);
  UniDB->PrimaryKVDB = std::move(PrimaryKVDB);
  UniDB->Logger = std::move(Logger);

```
- **EN**: Implements logic around `moveInto`, `move`, `value_or`, `size`; this block works with hashed storage or cache state.
- **CN**: 围绕 `moveInto`, `move`, `value_or`, `size` 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 508-523
```cpp
  return std::move(UniDB);
}

void UnifiedOnDiskCache::setSizeLimit(std::optional<uint64_t> SizeLimit) {
  this->SizeLimit = SizeLimit.value_or(0);
}

uint64_t UnifiedOnDiskCache::getStorageSize() const {
  uint64_t TotalSize = getPrimaryStorageSize();
  if (UpstreamGraphDB)
    TotalSize += UpstreamGraphDB->getStorageSize();
  if (UpstreamKVDB)
    TotalSize += UpstreamKVDB->getStorageSize();
  return TotalSize;
}

```
- **EN**: Implements logic around `move`, `setSizeLimit`, `value_or`, `getStorageSize`, and 1 more symbols; this block works with hashed storage or cache state.
- **CN**: 围绕 `move`, `setSizeLimit`, `value_or`, `getStorageSize`, and 1 more symbols 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 524-539
```cpp
uint64_t UnifiedOnDiskCache::getPrimaryStorageSize() const {
  return PrimaryGraphDB->getStorageSize() + PrimaryKVDB->getStorageSize();
}

bool UnifiedOnDiskCache::hasExceededSizeLimit() const {
  uint64_t CurSizeLimit = SizeLimit;
  if (!CurSizeLimit)
    return false;

  // If the hard limit is beyond 85%, declare above limit and request clean up.
  unsigned CurrentPercent =
      std::max(PrimaryGraphDB->getHardStorageLimitUtilization(),
               PrimaryKVDB->getHardStorageLimitUtilization());
  if (CurrentPercent > 85)
    return true;

```
- **EN**: Implements logic around `getPrimaryStorageSize`, `getStorageSize`, `hasExceededSizeLimit`, `max`, and 1 more symbols; this block works with hashed storage or cache state; handles HTTP protocol state or streaming.
- **CN**: 围绕 `getPrimaryStorageSize`, `getStorageSize`, `hasExceededSizeLimit`, `max`, and 1 more symbols 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态，并处理 HTTP 协议状态或流式传输。

### Lines 540-552
```cpp
  // We allow each of the directories in the chain to reach up to half the
  // intended size limit. Check whether the primary directory has exceeded half
  // the limit or not, in order to decide whether we need to start a new chain.
  //
  // We could check the size limit against the sum of sizes of both the primary
  // and upstream directories but then if the upstream is significantly larger
  // than the intended limit, it would trigger a new chain to be created before
  // the primary has reached its own limit. Essentially in such situation we
  // prefer reclaiming the storage later in order to have more consistent cache
  // hits behavior.
  return (CurSizeLimit / 2) < getPrimaryStorageSize();
}

```
- **EN**: Implements logic around `getPrimaryStorageSize`; this block works with hashed storage or cache state.
- **CN**: 围绕 `getPrimaryStorageSize` 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 553-564
```cpp
Error UnifiedOnDiskCache::close(bool CheckSizeLimit) {
  auto BypassSandbox = sys::sandbox::scopedDisable();

  if (LockFD == -1)
    return Error::success(); // already closed.
  llvm::scope_exit CloseLock([&]() {
    assert(LockFD >= 0);
    sys::fs::file_t LockFile = sys::fs::convertFDToNativeFile(LockFD);
    sys::fs::closeFile(LockFile);
    LockFD = -1;
  });

```
- **EN**: Implements logic around `close`, `scopedDisable`, `success`, `CloseLock`, and 3 more symbols; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `close`, `scopedDisable`, `success`, `CloseLock`, and 3 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 565-575
```cpp
  bool ExceededSizeLimit = CheckSizeLimit ? hasExceededSizeLimit() : false;
  UpstreamKVDB.reset();
  PrimaryKVDB.reset();
  UpstreamGraphDB.reset();
  PrimaryGraphDB.reset();
  if (std::error_code EC = unlockFileThreadSafe(LockFD))
    return createFileError(RootPath, EC);

  if (!ExceededSizeLimit)
    return Error::success();

```
- **EN**: Implements logic around `hasExceededSizeLimit`, `reset`, `unlockFileThreadSafe`, `createFileError`, and 1 more symbols; this block works with hashed storage or cache state.
- **CN**: 围绕 `hasExceededSizeLimit`, `reset`, `unlockFileThreadSafe`, `createFileError`, and 1 more symbols 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 576-587
```cpp
  // The primary directory exceeded its intended size limit. Try to get an
  // exclusive lock in order to create a new primary directory for next time
  // this \p UnifiedOnDiskCache path is opened.

  if (std::error_code EC = tryLockFileThreadSafe(
          LockFD, std::chrono::milliseconds(0), sys::fs::LockKind::Exclusive)) {
    if (EC == errc::no_lock_available)
      return Error::success(); // couldn't get exclusive lock, give up.
    return createFileError(RootPath, EC);
  }
  llvm::scope_exit UnlockFile([&]() { unlockFileThreadSafe(LockFD); });

```
- **EN**: Implements logic around `tryLockFileThreadSafe`, `milliseconds`, `success`, `createFileError`, and 1 more symbols; this block works with hashed storage or cache state.
- **CN**: 围绕 `tryLockFileThreadSafe`, `milliseconds`, `success`, `createFileError`, and 1 more symbols 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 588-601
```cpp
  // Managed to get an exclusive lock which means there are no other open
  // \p UnifiedOnDiskCache instances for the same path, so we can safely start a
  // new primary directory. To start a new primary directory we just have to
  // create a new empty directory with the next consecutive index; since this is
  // an atomic operation we will leave the top-level directory in a consistent
  // state even if the process dies during this code-path.

  SmallString<256> PathBuf(RootPath);
  raw_svector_ostream OS(PathBuf);
  OS << sys::path::get_separator();
  getNextDBDirName(PrimaryDBDir, OS);
  if (std::error_code EC = sys::fs::create_directory(PathBuf))
    return createFileError(PathBuf, EC);

```
- **EN**: Implements logic around `PathBuf`, `OS`, `get_separator`, `getNextDBDirName`, and 2 more symbols; this block works with hashed storage or cache state.
- **CN**: 围绕 `PathBuf`, `OS`, `get_separator`, `getNextDBDirName`, and 2 more symbols 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 602-615
```cpp
  NeedsGarbageCollection = true;
  return Error::success();
}

UnifiedOnDiskCache::UnifiedOnDiskCache() = default;

UnifiedOnDiskCache::~UnifiedOnDiskCache() { consumeError(close()); }

Error UnifiedOnDiskCache::collectGarbage(StringRef Path,
                                         ondisk::OnDiskCASLogger *Logger) {
  auto DBDirs = getAllGarbageDirs(Path);
  if (!DBDirs)
    return DBDirs.takeError();

```
- **EN**: Implements logic around `success`, `UnifiedOnDiskCache`, `~UnifiedOnDiskCache`, `collectGarbage`, and 2 more symbols; this block propagates recoverable errors through LLVM error utilities; parses or classifies structured input; works with hashed storage or cache state.
- **CN**: 围绕 `success`, `UnifiedOnDiskCache`, `~UnifiedOnDiskCache`, `collectGarbage`, and 2 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并解析或分类结构化输入，并处理基于哈希的存储或缓存状态。

### Lines 616-627
```cpp
  SmallString<256> PathBuf(Path);
  for (StringRef UnusedSubDir : *DBDirs) {
    sys::path::append(PathBuf, UnusedSubDir);
    if (Logger)
      Logger->logUnifiedOnDiskCacheCollectGarbage(PathBuf);
    if (std::error_code EC = sys::fs::remove_directories(PathBuf))
      return createFileError(PathBuf, EC);
    sys::path::remove_filename(PathBuf);
  }
  return Error::success();
}

```
- **EN**: Implements logic around `PathBuf`, `append`, `logUnifiedOnDiskCacheCollectGarbage`, `remove_directories`, and 3 more symbols; this block works with hashed storage or cache state.
- **CN**: 围绕 `PathBuf`, `append`, `logUnifiedOnDiskCacheCollectGarbage`, `remove_directories`, and 3 more symbols 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 628-630
```cpp
Error UnifiedOnDiskCache::collectGarbage() {
  return collectGarbage(RootPath, Logger.get());
}
```
- **EN**: Implements logic around `collectGarbage`; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `collectGarbage` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

## Key Concepts / 关键概念

- **Content-addressable storage / 内容寻址存储**:
  - **EN**: Represents objects by hash and manages in-memory or on-disk persistence.
  - **CN**: 通过哈希表示对象，并管理内存或磁盘持久化。
- **Structured error handling / 结构化错误处理**:
  - **EN**: Uses `Expected`, `Error`, or related helpers to make failures explicit.
  - **CN**: 使用 `Expected`、`Error` 或相关辅助工具显式表示失败。
- **LLVM container usage / LLVM 容器使用**:
  - **EN**: Relies on LLVM ADT containers for performance-conscious in-memory data management.
  - **CN**: 依赖 LLVM ADT 容器来进行注重性能的内存数据管理。
- **Streaming output / 流式输出**:
  - **EN**: Writes diagnostics, serialized data, or textual representations through LLVM stream APIs.
  - **CN**: 通过 LLVM 流式 API 输出诊断、序列化数据或文本表示。
- **Concurrency or parallel work / 并发或并行工作**:
  - **EN**: Coordinates tasks that may execute concurrently or partition work.
  - **CN**: 协调可能并发执行或分片处理的任务。

## Dependencies / 依赖关系

- **Direct LLVM/local includes / 直接的 LLVM/本地包含**: `llvm/CAS/UnifiedOnDiskCache.h`, `OnDiskCommon.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/ScopeExit.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringExtras.h`, `llvm/ADT/StringRef.h`, `llvm/CAS/OnDiskCASLogger.h`, `llvm/CAS/OnDiskGraphDB.h` ... (+11 more)
- **Standard-library headers / 标准库头文件**: `<optional>`
- **Subsystem categories / 子系统类别**: support-library helpers / Support 库辅助功能 (10), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (6), content-addressable storage interfaces / 内容寻址存储接口 (4)
