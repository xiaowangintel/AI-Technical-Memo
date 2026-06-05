# LockFileManager.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/LockFileManager.cpp`
- Repository: `llvm-project`
- Purpose (EN): / Attempt to read the lock file with the given name, if it exists.
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `LockFileManager` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===--- LockFileManager.cpp - File-level Locking Utility------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/Support/LockFileManager.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/Config/llvm-config.h" // for LLVM_ON_UNIX
#include "llvm/Support/Errc.h"
#include "llvm/Support/ErrorOr.h"
#include "llvm/Support/ExponentialBackoff.h"
#include "llvm/Support/FileSystem.h"
#include "llvm/Support/IOSandbox.h"
#include "llvm/Support/MemoryBuffer.h"
#include "llvm/Support/Path.h"
#include "llvm/Support/Process.h"
#include "llvm/Support/Signals.h"
#include "llvm/Support/raw_ostream.h"
#include <cerrno>
#include <chrono>
#include <ctime>
#include <memory>
#include <system_error>
#include <tuple>

#ifdef _WIN32
#include <windows.h>
#endif
#if LLVM_ON_UNIX
#include <unistd.h>
#endif

#if defined(__APPLE__) && defined(__ENVIRONMENT_MAC_OS_X_VERSION_MIN_REQUIRED__) && (__ENVIRONMENT_MAC_OS_X_VERSION_MIN_REQUIRED__ > 1050)
#define USE_OSX_GETHOSTUUID 1
#else
#define USE_OSX_GETHOSTUUID 0
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 22 direct dependencies, including `llvm/Support/LockFileManager.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringExtras.h`, `llvm/Config/llvm-config.h`.
  CN: 引入了 22 个直接依赖，其中包括 `llvm/Support/LockFileManager.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringExtras.h`, `llvm/Config/llvm-config.h`。
- EN: In this range, the code propagates LLVM-style errors and invariants and updates helper containers and temporary state.
  CN: 在这一段中，代码传播 LLVM 风格的错误处理并维护不变量，并更新辅助容器和临时状态。

### Lines 41-80

```cpp
#endif

#if USE_OSX_GETHOSTUUID
#include <uuid/uuid.h>
#endif

using namespace llvm;

/// Attempt to read the lock file with the given name, if it exists.
///
/// \param LockFileName The name of the lock file to read.
///
/// \returns The process ID of the process that owns this lock file
std::optional<LockFileManager::OwnedByAnother>
LockFileManager::readLockFile(StringRef LockFileName) {
  // Read the owning host and PID out of the lock file. If it appears that the
  // owning process is dead, the lock file is invalid.
  ErrorOr<std::unique_ptr<MemoryBuffer>> MBOrErr =
      MemoryBuffer::getFile(LockFileName);
  if (!MBOrErr) {
    sys::fs::remove(LockFileName);
    return std::nullopt;
  }
  MemoryBuffer &MB = *MBOrErr.get();

  StringRef Hostname;
  StringRef PIDStr;
  std::tie(Hostname, PIDStr) = getToken(MB.getBuffer(), " ");
  PIDStr = PIDStr.substr(PIDStr.find_first_not_of(' '));
  int PID;
  if (!PIDStr.getAsInteger(10, PID)) {
    OwnedByAnother Owner;
    Owner.OwnerHostName = Hostname;
    Owner.OwnerPID = PID;
    if (processStillExecuting(Owner.OwnerHostName, Owner.OwnerPID))
      return Owner;
  }

  // Delete the lock file. It's invalid anyway.
  sys::fs::remove(LockFileName);
```
- EN: Brings in 1 direct dependencies, including `uuid/uuid.h`.
  CN: 引入了 1 个直接依赖，其中包括 `uuid/uuid.h`。
- EN: This section centers on `readLockFile`, `getFile`, `remove` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `readLockFile`, `getFile`, `remove` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 81-120

```cpp
  return std::nullopt;
}

static std::error_code getHostID(SmallVectorImpl<char> &HostID) {
  HostID.clear();

#if USE_OSX_GETHOSTUUID
  // On OS X, use the more stable hardware UUID instead of hostname.
  struct timespec wait = {1, 0}; // 1 second.
  uuid_t uuid;
  if (gethostuuid(uuid, &wait) != 0)
    return errnoAsErrorCode();

  uuid_string_t UUIDStr;
  uuid_unparse(uuid, UUIDStr);
  StringRef UUIDRef(UUIDStr);
  HostID.append(UUIDRef.begin(), UUIDRef.end());

#elif LLVM_ON_UNIX
  char HostName[256];
  HostName[255] = 0;
  HostName[0] = 0;
  gethostname(HostName, 255);
  StringRef HostNameRef(HostName);
  HostID.append(HostNameRef.begin(), HostNameRef.end());

#else
  StringRef Dummy("localhost");
  HostID.append(Dummy.begin(), Dummy.end());
#endif

  return std::error_code();
}

bool LockFileManager::processStillExecuting(StringRef HostID, int PID) {
#if LLVM_ON_UNIX && !defined(__ANDROID__)
  SmallString<256> StoredHostID;
  if (getHostID(StoredHostID))
    return true; // Conservatively assume it's executing on error.

```
- EN: This section centers on `getHostID`, `uuid_unparse`, `UUIDRef` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `getHostID`, `uuid_unparse`, `UUIDRef` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 121-160

```cpp
  // Check whether the process is dead. If so, we're done.
  if (StoredHostID == HostID && getsid(PID) == -1 && errno == ESRCH)
    return false;
#endif

  return true;
}

namespace {

/// An RAII helper object ensure that the unique lock file is removed.
///
/// Ensures that if there is an error or a signal before we finish acquiring the
/// lock, the unique file will be removed. And if we successfully take the lock,
/// the signal handler is left in place so that signals while the lock is held
/// will remove the unique lock file. The caller should ensure there is a
/// matching call to sys::DontRemoveFileOnSignal when the lock is released.
class RemoveUniqueLockFileOnSignal {
  StringRef Filename;
  bool RemoveImmediately;
public:
  RemoveUniqueLockFileOnSignal(StringRef Name)
  : Filename(Name), RemoveImmediately(true) {
    sys::RemoveFileOnSignal(Filename, nullptr);
  }

  ~RemoveUniqueLockFileOnSignal() {
    if (!RemoveImmediately) {
      // Leave the signal handler enabled. It will be removed when the lock is
      // released.
      return;
    }
    sys::fs::remove(Filename);
    sys::DontRemoveFileOnSignal(Filename);
  }

  void lockAcquired() { RemoveImmediately = false; }
};

} // end anonymous namespace
```
- EN: This section centers on `RemoveUniqueLockFileOnSignal`, `RemoveFileOnSignal`, `~RemoveUniqueLockFileOnSignal` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `RemoveUniqueLockFileOnSignal`, `RemoveFileOnSignal`, `~RemoveUniqueLockFileOnSignal` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and returns the resulting value to its callers.
  CN: 在这一段中，代码检查条件并处理边界情况，并将结果返回给调用方。

### Lines 161-200

```cpp

LockFileManager::LockFileManager(StringRef FileName)
    : FileName(FileName), Owner(OwnerUnknown{}) {}

Expected<bool> LockFileManager::tryLock() {
  auto BypassSandbox = sys::sandbox::scopedDisable();

  assert(std::holds_alternative<OwnerUnknown>(Owner) &&
         "lock has already been attempted");

  SmallString<128> AbsoluteFileName(FileName);
  if (std::error_code EC = sys::fs::make_absolute(AbsoluteFileName))
    return createStringError(EC, "failed to obtain absolute path for " +
                                     AbsoluteFileName);
  LockFileName = AbsoluteFileName;
  LockFileName += ".lock";

  // If the lock file already exists, don't bother to try to create our own
  // lock file; it won't work anyway. Just figure out who owns this lock file.
  if (auto LockFileOwner = readLockFile(LockFileName)) {
    Owner = std::move(*LockFileOwner);
    return false;
  }

  // Create a lock file that is unique to this instance.
  int UniqueLockFileID;
  {
    SmallString<128> UniqueLockFilePattern = LockFileName;
    UniqueLockFilePattern += "-%%%%%%%%";
    SmallString<128> UniquePath;
    std::error_code EC = sys::fs::createUniqueFile(
        UniqueLockFilePattern, UniqueLockFileID, UniquePath);
    if (EC == errc::no_such_file_or_directory) {
      SmallString<128> Dir = sys::path::parent_path(UniqueLockFilePattern);
      if (!Dir.empty()) {
        if (std::error_code DirEC = sys::fs::create_directories(Dir))
          return createStringError(DirEC,
                                   "failed to create lock directory " + Dir);
      }

```
- EN: This section centers on `tryLock`, `assert`, `AbsoluteFileName` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `tryLock`, `assert`, `AbsoluteFileName` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 201-240

```cpp
      // Retry creating lock file
      EC = sys::fs::createUniqueFile(UniqueLockFilePattern, UniqueLockFileID,
                                     UniquePath);
    }

    if (EC)
      return createStringError(EC,
                               "failed to create unique file " + UniquePath);

    UniqueLockFileName = UniquePath;
  }

  // Clean up the unique file on signal or scope exit.
  RemoveUniqueLockFileOnSignal RemoveUniqueFile(UniqueLockFileName);

  // Write our process ID to our unique lock file.
  {
    SmallString<256> HostID;
    if (auto EC = getHostID(HostID))
      return createStringError(EC, "failed to get host id");

    raw_fd_ostream Out(UniqueLockFileID, /*shouldClose=*/true);
    Out << HostID << ' ' << sys::Process::getProcessId();
    Out.close();

    if (Out.has_error()) {
      // We failed to write out PID, so report the error and fail.
      Error Err = createStringError(Out.error(),
                                    "failed to write to " + UniqueLockFileName);
      // Don't call report_fatal_error.
      Out.clear_error();
      return std::move(Err);
    }
  }

  while (true) {
    // Create a link from the lock file name. If this succeeds, we're done.
    std::error_code EC =
        sys::fs::create_link(UniqueLockFileName, LockFileName);
    if (!EC) {
```
- EN: This section centers on `RemoveUniqueFile`, `Out`, `move` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `RemoveUniqueFile`, `Out`, `move` 等符号展开，负责创建并初始化辅助对象或状态。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 241-280

```cpp
      RemoveUniqueFile.lockAcquired();
      Owner = OwnedByUs{};
      return true;
    }

    if (EC != errc::file_exists)
      return createStringError(EC, "failed to create link " + LockFileName +
                                       " to " + UniqueLockFileName);

    // Someone else managed to create the lock file first. Read the process ID
    // from the lock file.
    if (auto LockFileOwner = readLockFile(LockFileName)) {
      Owner = std::move(*LockFileOwner);
      return false;
    }

    if (!sys::fs::exists(LockFileName)) {
      // The previous owner released the lock file before we could read it.
      // Try to get ownership again.
      continue;
    }

    // There is a lock file that nobody owns; try to clean it up and get
    // ownership.
    if ((EC = sys::fs::remove(LockFileName)))
      return createStringError(EC, "failed to remove lockfile " +
                                       UniqueLockFileName);
  }
}

LockFileManager::~LockFileManager() {
  auto BypassSandbox = sys::sandbox::scopedDisable();

  if (!std::holds_alternative<OwnedByUs>(Owner))
    return;

  // Since we own the lock, remove the lock file and our own unique lock file.
  sys::fs::remove(LockFileName);
  sys::fs::remove(UniqueLockFileName);
  // The unique file is now gone, so remove it from the signal handler. This
```
- EN: This section centers on `remove` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `remove` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 281-320

```cpp
  // matches a sys::RemoveFileOnSignal() in LockFileManager().
  sys::DontRemoveFileOnSignal(UniqueLockFileName);
}

WaitForUnlockResult
LockFileManager::waitForUnlockFor(std::chrono::seconds MaxSeconds) {
  auto BypassSandbox = sys::sandbox::scopedDisable();

  auto *LockFileOwner = std::get_if<OwnedByAnother>(&Owner);
  assert(LockFileOwner &&
         "waiting for lock to be unlocked without knowing the owner");

  // Since we don't yet have an event-based method to wait for the lock file,
  // use randomized exponential backoff, similar to Ethernet collision
  // algorithm. This improves performance on machines with high core counts
  // when the file lock is heavily contended by multiple clang processes
  using namespace std::chrono_literals;
  ExponentialBackoff Backoff(MaxSeconds, 10ms, 500ms);

  // Wait first as this is only called when the lock is known to be held.
  while (Backoff.waitForNextAttempt()) {
    // FIXME: implement event-based waiting
    if (sys::fs::access(LockFileName.c_str(), sys::fs::AccessMode::Exist) ==
        errc::no_such_file_or_directory)
      return WaitForUnlockResult::Success;

    // If the process owning the lock died without cleaning up, just bail out.
    if (!processStillExecuting(LockFileOwner->OwnerHostName,
                               LockFileOwner->OwnerPID))
      return WaitForUnlockResult::OwnerDied;
  }

  // Give up.
  return WaitForUnlockResult::Timeout;
}

std::error_code LockFileManager::unsafeUnlock() {
  auto BypassSandbox = sys::sandbox::scopedDisable();

  return sys::fs::remove(LockFileName);
```
- EN: This section centers on `DontRemoveFileOnSignal`, `waitForUnlockFor`, `assert` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `DontRemoveFileOnSignal`, `waitForUnlockFor`, `assert` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 321-321

```cpp
}
```
- EN: This range contains supporting statements, comments, or structural glue code.
  CN: 这一段主要包含辅助语句、注释或结构性胶水代码。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `timespec`, `RemoveUniqueLockFileOnSignal`, `readLockFile`, `getFile`, `remove`, `tie` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Parsing, decoding, and structural validation. / 重点关注解析、解码与结构校验。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/Support/LockFileManager.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringExtras.h`, `llvm/Config/llvm-config.h`, `llvm/Support/Errc.h`, `llvm/Support/ErrorOr.h`, `llvm/Support/ExponentialBackoff.h`, `llvm/Support/FileSystem.h`, `llvm/Support/IOSandbox.h`, `llvm/Support/MemoryBuffer.h`, `llvm/Support/Path.h`, `llvm/Support/Process.h`, `llvm/Support/Signals.h`, `llvm/Support/raw_ostream.h`
- Standard library / 标准库: `chrono`, `memory`, `system_error`, `tuple`, `unistd.h`
- Other/system headers / 其他或系统头文件: `cerrno`, `ctime`, `windows.h`, `uuid/uuid.h`
- Related symbols / 相关符号: `timespec`, `RemoveUniqueLockFileOnSignal`, `readLockFile`, `getFile`, `remove`, `tie`, `getHostID`
