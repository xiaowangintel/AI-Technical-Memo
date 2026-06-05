# OnDiskCommon.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/lib/CAS/OnDiskCommon.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements content-addressable storage, on-disk caches, and related schema support.
  - **CN**: 实现内容寻址存储、磁盘缓存以及相关 schema 支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- OnDiskCommon.cpp ---------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件的高层描述。

### Lines 8-17
```cpp

#include "OnDiskCommon.h"
#include "llvm/Support/Errno.h"
#include "llvm/Support/Error.h"
#include "llvm/Support/FileSystem.h"
#include "llvm/Support/Path.h"
#include "llvm/Support/Process.h"
#include <mutex>
#include <thread>

```
- **EN**: Pulls in the headers needed by this translation unit, including `OnDiskCommon.h`, `llvm/Support/Errno.h`, `llvm/Support/Error.h`, `llvm/Support/FileSystem.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `OnDiskCommon.h`, `llvm/Support/Errno.h`, `llvm/Support/Error.h`, `llvm/Support/FileSystem.h`。

### Lines 18-26
```cpp
#if __has_include(<sys/file.h>)
#include <sys/file.h>
#ifdef LOCK_SH
#define HAVE_FLOCK 1
#else
#define HAVE_FLOCK 0
#endif
#endif

```
- **EN**: Pulls in the headers needed by this translation unit, including `sys/file.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `sys/file.h`。

### Lines 27-34
```cpp
#if __has_include(<fcntl.h>)
#include <fcntl.h>
#endif

#if __has_include(<sys/mount.h>)
#include <sys/mount.h> // statfs
#endif

```
- **EN**: Pulls in the headers needed by this translation unit, including `fcntl.h`, `sys/mount.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `fcntl.h`, `sys/mount.h`。

### Lines 35-42
```cpp
#ifdef __APPLE__
#if __has_include(<sys/sysctl.h>)
#include <sys/sysctl.h>
#endif
#endif

using namespace llvm;

```
- **EN**: Pulls in the headers needed by this translation unit, including `sys/sysctl.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `sys/sysctl.h`。

### Lines 43-54
```cpp
static uint64_t OnDiskCASMaxMappingSize = 0;

Expected<std::optional<uint64_t>> cas::ondisk::getOverriddenMaxMappingSize() {
  static std::once_flag Flag;
  Error Err = Error::success();
  std::call_once(Flag, [&Err] {
    ErrorAsOutParameter EAO(&Err);
    constexpr const char *EnvVar = "LLVM_CAS_MAX_MAPPING_SIZE";
    auto Value = sys::Process::GetEnv(EnvVar);
    if (!Value)
      return;

```
- **EN**: Implements logic around `getOverriddenMaxMappingSize`, `success`, `call_once`, `EAO`, and 1 more symbols; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `getOverriddenMaxMappingSize`, `success`, `call_once`, `EAO`, and 1 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 55-61
```cpp
    uint64_t Size;
    if (StringRef(*Value).getAsInteger(/*auto*/ 0, Size))
      Err = createStringError(inconvertibleErrorCode(),
                              "invalid value for %s: expected integer", EnvVar);
    OnDiskCASMaxMappingSize = Size;
  });

```
- **EN**: Implements logic around `StringRef`, `createStringError`; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `StringRef`, `createStringError` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 62-70
```cpp
  if (Err)
    return std::move(Err);

  if (OnDiskCASMaxMappingSize == 0)
    return std::nullopt;

  return OnDiskCASMaxMappingSize;
}

```
- **EN**: Implements logic around `move`; this block works with hashed storage or cache state.
- **CN**: 围绕 `move` 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 71-84
```cpp
void cas::ondisk::setMaxMappingSize(uint64_t Size) {
  OnDiskCASMaxMappingSize = Size;
}

std::error_code cas::ondisk::lockFileThreadSafe(int FD,
                                                sys::fs::LockKind Kind) {
#if HAVE_FLOCK
  if (sys::RetryAfterSignal(
          -1, flock, FD,
          Kind == sys::fs::LockKind::Exclusive ? LOCK_EX : LOCK_SH) == 0)
    return std::error_code();
  return std::error_code(errno, std::generic_category());
#elif defined(_WIN32)
  // On Windows this implementation is thread-safe.
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、特性开关或编译期常量。

### Lines 85-98
```cpp
  return sys::fs::lockFile(FD, Kind);
#else
  return make_error_code(std::errc::no_lock_available);
#endif
}

std::error_code cas::ondisk::unlockFileThreadSafe(int FD) {
#if HAVE_FLOCK
  if (sys::RetryAfterSignal(-1, flock, FD, LOCK_UN) == 0)
    return std::error_code();
  return std::error_code(errno, std::generic_category());
#elif defined(_WIN32)
  // On Windows this implementation is thread-safe.
  return sys::fs::unlockFile(FD);
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、特性开关或编译期常量。

### Lines 99-112
```cpp
#else
  return make_error_code(std::errc::no_lock_available);
#endif
}

std::error_code
cas::ondisk::tryLockFileThreadSafe(int FD, std::chrono::milliseconds Timeout,
                                   sys::fs::LockKind Kind) {
#if HAVE_FLOCK
  auto Start = std::chrono::steady_clock::now();
  auto End = Start + Timeout;
  do {
    if (sys::RetryAfterSignal(
            -1, flock, FD,
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、特性开关或编译期常量。

### Lines 113-126
```cpp
            (Kind == sys::fs::LockKind::Exclusive ? LOCK_EX : LOCK_SH) |
                LOCK_NB) == 0)
      return std::error_code();
    int Error = errno;
    if (Error == EWOULDBLOCK) {
      if (Timeout.count() == 0)
        break;
      // Match sys::fs::tryLockFile, which sleeps for 1 ms per attempt.
      std::this_thread::sleep_for(std::chrono::milliseconds(1));
      continue;
    }
    return std::error_code(Error, std::generic_category());
  } while (std::chrono::steady_clock::now() < End);
  return make_error_code(std::errc::no_lock_available);
```
- **EN**: Implements logic around `error_code`, `count`, `sleep_for`, `now`, and 1 more symbols; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `error_code`, `count`, `sleep_for`, `now`, and 1 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 127-134
```cpp
#elif defined(_WIN32)
  // On Windows this implementation is thread-safe.
  return sys::fs::tryLockFile(FD, Timeout, Kind);
#else
  return make_error_code(std::errc::no_lock_available);
#endif
}

```
- **EN**: Implements logic around `tryLockFile`, `make_error_code`; this block works with hashed storage or cache state.
- **CN**: 围绕 `tryLockFile`, `make_error_code` 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 135-148
```cpp
Expected<size_t> cas::ondisk::preallocateFileTail(int FD, size_t CurrentSize,
                                                  size_t NewSize) {
  auto CreateError = [&](std::error_code EC) -> Expected<size_t> {
    if (EC == std::errc::not_supported)
      // Ignore ENOTSUP in case the filesystem cannot preallocate.
      return NewSize;
#if defined(HAVE_POSIX_FALLOCATE)
    if (EC == std::errc::invalid_argument && CurrentSize < NewSize && // len > 0
        NewSize < std::numeric_limits<off_t>::max()) // 0 <= offset, len < max
      // Prior to 2024, POSIX required EINVAL for cases that should be ENOTSUP,
      // so handle it the same as above if it is not one of the other ways to
      // get EINVAL.
      return NewSize;
#endif
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、特性开关或编译期常量。

### Lines 149-162
```cpp
    return createStringError(EC,
                             "failed to allocate to CAS file: " + EC.message());
  };
#if defined(HAVE_POSIX_FALLOCATE)
  // Note: posix_fallocate returns its error directly, not via errno.
  int Err;
  do {
    Err = posix_fallocate(FD, CurrentSize, NewSize - CurrentSize);
  } while (Err == EINTR);
  if (Err)
    return CreateError(std::error_code(Err, std::generic_category()));
  return NewSize;
#elif defined(__APPLE__)
  fstore_t FAlloc;
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、特性开关或编译期常量。

### Lines 163-176
```cpp
  FAlloc.fst_flags = F_ALLOCATEALL;
#if defined(F_ALLOCATEPERSIST) &&                                              \
    defined(__ENVIRONMENT_MAC_OS_X_VERSION_MIN_REQUIRED__) &&                  \
    __ENVIRONMENT_MAC_OS_X_VERSION_MIN_REQUIRED__ >= 130000
  // F_ALLOCATEPERSIST is introduced in macOS 13.
  FAlloc.fst_flags |= F_ALLOCATEPERSIST;
#endif
  FAlloc.fst_posmode = F_PEOFPOSMODE;
  FAlloc.fst_offset = 0;
  FAlloc.fst_length = NewSize - CurrentSize;
  FAlloc.fst_bytesalloc = 0;
  if (sys::RetryAfterSignal(-1, ::fcntl, FD, F_PREALLOCATE, &FAlloc) == -1)
    return CreateError(errnoAsErrorCode());
  assert(CurrentSize + FAlloc.fst_bytesalloc >= NewSize);
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、特性开关或编译期常量。

### Lines 177-183
```cpp
  return CurrentSize + FAlloc.fst_bytesalloc;
#else
  (void)CreateError; // Silence unused variable.
  return NewSize;    // Pretend it worked.
#endif
}

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 184-190
```cpp
bool cas::ondisk::useSmallMappingSize(const Twine &P) {
  // Add exceptions to use small database file here.
#if defined(__APPLE__) && __has_include(<sys/mount.h>)
  // macOS tmpfs does not support sparse tails.
  SmallString<128> PathStorage;
  StringRef Path = P.toNullTerminatedStringRef(PathStorage);
  struct statfs StatFS;
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、特性开关或编译期常量。

### Lines 191-200
```cpp
  if (statfs(Path.data(), &StatFS) != 0)
    return false;

  if (strcmp(StatFS.f_fstypename, "tmpfs") == 0)
    return true;
#endif
  // Default to use regular database file.
  return false;
}

```
- **EN**: Implements logic around `statfs`, `strcmp`; this block works with hashed storage or cache state.
- **CN**: 围绕 `statfs`, `strcmp` 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 201-214
```cpp
Expected<uint64_t> cas::ondisk::getBootTime() {
#ifdef __APPLE__
#if __has_include(<sys/sysctl.h>) && defined(KERN_BOOTTIME)
  struct timeval TV;
  size_t TVLen = sizeof(TV);
  int KernBoot[2] = {CTL_KERN, KERN_BOOTTIME};
  if (sysctl(KernBoot, 2, &TV, &TVLen, nullptr, 0) < 0)
    return createStringError(llvm::errnoAsErrorCode(),
                             "failed to get boottime");
  if (TVLen != sizeof(TV))
    return createStringError("sysctl kern.boottime unexpected format");
  return TV.tv_sec;
#else
  return 0;
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、特性开关或编译期常量。

### Lines 215-227
```cpp
#endif
#elif defined(__linux__)
  // Use the mtime for /proc, which is recreated during system boot.
  // We could also read /proc/stat and search for 'btime'.
  sys::fs::file_status Status;
  if (std::error_code EC = sys::fs::status("/proc", Status))
    return createFileError("/proc", EC);
  return Status.getLastModificationTime().time_since_epoch().count();
#else
  return 0;
#endif
}

```
- **EN**: Implements logic around `status`, `createFileError`, `getLastModificationTime`; this block works with hashed storage or cache state.
- **CN**: 围绕 `status`, `createFileError`, `getLastModificationTime` 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 228-241
```cpp
Expected<StringRef>
cas::ondisk::UniqueTempFile::createAndCopyFrom(StringRef ParentPath,
                                               StringRef CopyFromPath) {
  // \c clonefile requires that the destination path doesn't exist. We create
  // a "placeholder" temporary file, then modify its path a bit and use that
  // for \c clonefile to write to.
  // FIXME: Instead of creating a dummy file, add a new file system API for
  // copying to a unique path that can loop while checking EEXIST.
  SmallString<256> UniqueTmpPath;
  SmallString<256> Model;
  Model += ParentPath;
  sys::path::append(Model, "%%%%%%%.tmp");
  if (std::error_code EC = sys::fs::createUniqueFile(Model, UniqueTmpPath))
    return createFileError(Model, EC);
```
- **EN**: Implements logic around `createAndCopyFrom`, `append`, `createUniqueFile`, `createFileError`; this block emits or serializes data to an external representation; works with hashed storage or cache state.
- **CN**: 围绕 `createAndCopyFrom`, `append`, `createUniqueFile`, `createFileError` 实现具体逻辑；该代码块把数据输出或序列化为外部表示，并处理基于哈希的存储或缓存状态。

### Lines 242-250
```cpp
  TmpPath = std::move(UniqueTmpPath);
  TmpPath += ".tmp"; // modify so that there's no file at that path.
  // \c copy_file will use \c clonefile when applicable.
  if (std::error_code EC = sys::fs::copy_file(CopyFromPath, TmpPath))
    return createFileError(TmpPath, EC);

  return TmpPath;
}

```
- **EN**: Implements logic around `move`, `copy_file`, `createFileError`; this block works with hashed storage or cache state.
- **CN**: 围绕 `move`, `copy_file`, `createFileError` 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 251-257
```cpp
Error cas::ondisk::UniqueTempFile::renameTo(StringRef RenameToPath) {
  if (std::error_code EC = sys::fs::rename(TmpPath, RenameToPath))
    return createFileError(RenameToPath, EC);
  TmpPath.clear();
  return Error::success();
}

```
- **EN**: Implements logic around `renameTo`, `rename`, `createFileError`, `clear`, and 1 more symbols; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `renameTo`, `rename`, `createFileError`, `clear`, and 1 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 258-263
```cpp
cas::ondisk::UniqueTempFile::~UniqueTempFile() {
  if (!TmpPath.empty())
    sys::fs::remove(TmpPath);
  if (!UniqueTmpPath.empty())
    sys::fs::remove(UniqueTmpPath);
}
```
- **EN**: Implements logic around `~UniqueTempFile`, `empty`, `remove`; this block works with hashed storage or cache state.
- **CN**: 围绕 `~UniqueTempFile`, `empty`, `remove` 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

## Key Concepts / 关键概念

- **Content-addressable storage / 内容寻址存储**:
  - **EN**: Represents objects by hash and manages in-memory or on-disk persistence.
  - **CN**: 通过哈希表示对象，并管理内存或磁盘持久化。
- **Structured error handling / 结构化错误处理**:
  - **EN**: Uses `Expected`, `Error`, or related helpers to make failures explicit.
  - **CN**: 使用 `Expected`、`Error` 或相关辅助工具显式表示失败。
- **Concurrency or parallel work / 并发或并行工作**:
  - **EN**: Coordinates tasks that may execute concurrently or partition work.
  - **CN**: 协调可能并发执行或分片处理的任务。

## Dependencies / 依赖关系

- **Direct LLVM/local includes / 直接的 LLVM/本地包含**: `OnDiskCommon.h`, `llvm/Support/Errno.h`, `llvm/Support/Error.h`, `llvm/Support/FileSystem.h`, `llvm/Support/Path.h`, `llvm/Support/Process.h`
- **Standard-library headers / 标准库头文件**: `<mutex>`, `<thread>`, `<sys/file.h>`, `<fcntl.h>`, `<sys/mount.h>`, `<sys/sysctl.h>`
- **Subsystem categories / 子系统类别**: support-library helpers / Support 库辅助功能 (5)
