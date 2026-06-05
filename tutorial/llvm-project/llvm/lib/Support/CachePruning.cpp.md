# CachePruning.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/CachePruning.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements the pruning of a directory based on least recently used.
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `CachePruning` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===-CachePruning.cpp - LLVM Cache Directory Pruning ---------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the pruning of a directory based on least recently used.
//
//===----------------------------------------------------------------------===//

#include "llvm/Support/CachePruning.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/Errc.h"
#include "llvm/Support/Error.h"
#include "llvm/Support/FileSystem.h"
#include "llvm/Support/Path.h"
#include "llvm/Support/WithColor.h"
#include "llvm/Support/raw_ostream.h"

#define DEBUG_TYPE "cache-pruning"

#include <set>
#include <system_error>

using namespace llvm;

namespace {
struct FileInfo {
  sys::TimePoint<> Time;
  uint64_t Size;
  std::string Path;

  /// Used to determine which files to prune first. Also used to determine
  /// set membership, so must take into account all fields.
  bool operator<(const FileInfo &Other) const {
    return std::tie(Time, Other.Size, Path) <
           std::tie(Other.Time, Size, Other.Path);
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 11 direct dependencies, including `llvm/Support/CachePruning.h`, `llvm/ADT/StringRef.h`, `llvm/Support/Debug.h`, `llvm/Support/Errc.h`.
  CN: 引入了 11 个直接依赖，其中包括 `llvm/Support/CachePruning.h`, `llvm/ADT/StringRef.h`, `llvm/Support/Debug.h`, `llvm/Support/Errc.h`。
- EN: This section centers on `tie` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `tie` 等符号展开，负责实现局部控制流程与状态维护。

### Lines 41-80

```cpp
  }
};
} // anonymous namespace

/// Write a new timestamp file with the given path. This is used for the pruning
/// interval option.
static void writeTimestampFile(StringRef TimestampFile) {
  std::error_code EC;
  raw_fd_ostream Out(TimestampFile.str(), EC, sys::fs::OF_None);
}

static Expected<std::chrono::seconds> parseDuration(StringRef Duration) {
  if (Duration.empty())
    return make_error<StringError>("Duration must not be empty",
                                   inconvertibleErrorCode());

  StringRef NumStr = Duration.slice(0, Duration.size()-1);
  uint64_t Num;
  if (NumStr.getAsInteger(0, Num))
    return make_error<StringError>("'" + NumStr + "' not an integer",
                                   inconvertibleErrorCode());

  switch (Duration.back()) {
  case 's':
    return std::chrono::seconds(Num);
  case 'm':
    return std::chrono::minutes(Num);
  case 'h':
    return std::chrono::hours(Num);
  default:
    return make_error<StringError>("'" + Duration +
                                       "' must end with one of 's', 'm' or 'h'",
                                   inconvertibleErrorCode());
  }
}

Expected<CachePruningPolicy>
llvm::parseCachePruningPolicy(StringRef PolicyStr) {
  CachePruningPolicy Policy;
  std::pair<StringRef, StringRef> P = {"", PolicyStr};
```
- EN: This section centers on `writeTimestampFile`, `Out`, `parseDuration` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `writeTimestampFile`, `Out`, `parseDuration` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and checks conditions and handles edge cases.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并检查条件并处理边界情况。

### Lines 81-120

```cpp
  while (!P.second.empty()) {
    P = P.second.split(':');

    StringRef Key, Value;
    std::tie(Key, Value) = P.first.split('=');
    if (Key == "prune_interval") {
      auto DurationOrErr = parseDuration(Value);
      if (!DurationOrErr)
        return DurationOrErr.takeError();
      Policy.Interval = *DurationOrErr;
    } else if (Key == "prune_after") {
      auto DurationOrErr = parseDuration(Value);
      if (!DurationOrErr)
        return DurationOrErr.takeError();
      Policy.Expiration = *DurationOrErr;
    } else if (Key == "cache_size") {
      if (Value.back() != '%')
        return make_error<StringError>("'" + Value + "' must be a percentage",
                                       inconvertibleErrorCode());
      StringRef SizeStr = Value.drop_back();
      uint64_t Size;
      if (SizeStr.getAsInteger(0, Size))
        return make_error<StringError>("'" + SizeStr + "' not an integer",
                                       inconvertibleErrorCode());
      if (Size > 100)
        return make_error<StringError>("'" + SizeStr +
                                           "' must be between 0 and 100",
                                       inconvertibleErrorCode());
      Policy.MaxSizePercentageOfAvailableSpace = Size;
    } else if (Key == "cache_size_bytes") {
      uint64_t Mult = 1;
      switch (tolower(Value.back())) {
      case 'k':
        Mult = 1024;
        Value = Value.drop_back();
        break;
      case 'm':
        Mult = 1024 * 1024;
        Value = Value.drop_back();
        break;
```
- EN: This section centers on `tie` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `tie` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and iterates over collections, ranges, or records.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并遍历集合、区间或记录。

### Lines 121-160

```cpp
      case 'g':
        Mult = 1024 * 1024 * 1024;
        Value = Value.drop_back();
        break;
      }
      uint64_t Size;
      if (Value.getAsInteger(0, Size))
        return make_error<StringError>("'" + Value + "' not an integer",
                                       inconvertibleErrorCode());
      Policy.MaxSizeBytes = Size * Mult;
    } else if (Key == "cache_size_files") {
      if (Value.getAsInteger(0, Policy.MaxSizeFiles))
        return make_error<StringError>("'" + Value + "' not an integer",
                                       inconvertibleErrorCode());
    } else {
      return make_error<StringError>("Unknown key: '" + Key + "'",
                                     inconvertibleErrorCode());
    }
  }

  return Policy;
}

/// Prune the cache of files that haven't been accessed in a long time.
Expected<bool>
llvm::pruneCache(StringRef Path, CachePruningPolicy Policy,
                 const std::vector<std::unique_ptr<MemoryBuffer>> &Files) {
  using namespace std::chrono;

  if (Path.empty())
    return false;

  bool isPathDir;
  if (sys::fs::is_directory(Path, isPathDir))
    return false;

  if (!isPathDir)
    return false;

  Policy.MaxSizePercentageOfAvailableSpace =
```
- EN: This section centers on `inconvertibleErrorCode`, `pruneCache` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `inconvertibleErrorCode`, `pruneCache` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 161-200

```cpp
      std::min(Policy.MaxSizePercentageOfAvailableSpace, 100u);

  if (Policy.Expiration == seconds(0) &&
      Policy.MaxSizePercentageOfAvailableSpace == 0 &&
      Policy.MaxSizeBytes == 0 && Policy.MaxSizeFiles == 0) {
    LLVM_DEBUG(dbgs() << "No pruning settings set, exit early\n");
    // Nothing will be pruned, early exit
    return false;
  }

  // Try to stat() the timestamp file.
  SmallString<128> TimestampFile(Path);
  sys::path::append(TimestampFile, "llvmcache.timestamp");
  sys::fs::file_status FileStatus;
  const auto CurrentTime = system_clock::now();
  if (auto EC = sys::fs::status(TimestampFile, FileStatus)) {
    if (EC == errc::no_such_file_or_directory) {
      // If the timestamp file wasn't there, create one now.
      writeTimestampFile(TimestampFile);
    } else {
      // Unknown error?
      return false;
    }
  } else {
    if (!Policy.Interval)
      return false;
    if (Policy.Interval != seconds(0)) {
      // Check whether the time stamp is older than our pruning interval.
      // If not, do nothing.
      const auto TimeStampModTime = FileStatus.getLastModificationTime();
      auto TimeStampAge = CurrentTime - TimeStampModTime;
      if (TimeStampAge <= *Policy.Interval) {
        LLVM_DEBUG(dbgs() << "Timestamp file too recent ("
                          << duration_cast<seconds>(TimeStampAge).count()
                          << "s old), do not prune.\n");
        return false;
      }
    }
    // Write a new timestamp file so that nobody else attempts to prune.
    // There is a benign race condition here, if two processes happen to
```
- EN: This section centers on `min`, `TimestampFile`, `append` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `min`, `TimestampFile`, `append` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code checks conditions and handles edge cases and returns the resulting value to its callers.
  CN: 在这一段中，代码检查条件并处理边界情况，并将结果返回给调用方。

### Lines 201-240

```cpp
    // notice at the same time that the timestamp is out-of-date.
    writeTimestampFile(TimestampFile);
  }

  // Keep track of files to delete to get below the size limit.
  // Order by time of last use so that recently used files are preserved.
  std::set<FileInfo> FileInfos;
  uint64_t TotalSize = 0;

  // Walk the entire directory cache, looking for unused files.
  std::error_code EC;
  SmallString<128> CachePathNative;
  sys::path::native(Path, CachePathNative);
  // Walk all of the files within this directory.
  for (sys::fs::directory_iterator File(CachePathNative, EC), FileEnd;
       File != FileEnd && !EC; File.increment(EC)) {
    // Ignore filenames not beginning with "llvmcache-" or "Thin-". This
    // includes the timestamp file as well as any files created by the user.
    // This acts as a safeguard against data loss if the user specifies the
    // wrong directory as their cache directory.
    StringRef filename = sys::path::filename(File->path());
    if (!filename.starts_with("llvmcache-") && !filename.starts_with("Thin-"))
      continue;

    // Look at this file. If we can't stat it, there's nothing interesting
    // there.
    ErrorOr<sys::fs::basic_file_status> StatusOrErr = File->status();
    if (!StatusOrErr) {
      LLVM_DEBUG(dbgs() << "Ignore " << File->path() << " (can't stat)\n");
      continue;
    }

    // If the file hasn't been used recently enough, delete it
    const auto FileAccessTime = StatusOrErr->getLastAccessedTime();
    auto FileAge = CurrentTime - FileAccessTime;
    if (Policy.Expiration != seconds(0) && FileAge > Policy.Expiration) {
      LLVM_DEBUG(dbgs() << "Remove " << File->path() << " ("
                        << duration_cast<seconds>(FileAge).count()
                        << "s old)\n");
      sys::fs::remove(File->path());
```
- EN: This section centers on `writeTimestampFile`, `native`, `remove` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `writeTimestampFile`, `native`, `remove` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 241-280

```cpp
      continue;
    }

    // Leave it here for now, but add it to the list of size-based pruning.
    TotalSize += StatusOrErr->getSize();
    FileInfos.insert({FileAccessTime, StatusOrErr->getSize(), File->path()});
  }

  auto FileInfo = FileInfos.begin();
  size_t NumFiles = FileInfos.size();

  auto RemoveCacheFile = [&]() {
    // Remove the file.
    sys::fs::remove(FileInfo->Path);
    // Update size
    TotalSize -= FileInfo->Size;
    NumFiles--;
    LLVM_DEBUG(dbgs() << " - Remove " << FileInfo->Path << " (size "
                      << FileInfo->Size << "), new occupancy is " << TotalSize
                      << "%\n");
    ++FileInfo;
  };

  // files.size() is greater the number of inputs  by one. However, a timestamp
  // file is created and stored in the cache directory if --thinlto-cache-policy
  // option is used. Therefore, files.size() is used as ActualNums.
  const size_t ActualNums = Files.size();
  if (Policy.MaxSizeFiles && ActualNums > Policy.MaxSizeFiles)
    WithColor::warning()
        << "ThinLTO cache pruning happens since the number of created files ("
        << ActualNums << ") exceeds the maximum number of files ("
        << Policy.MaxSizeFiles
        << "); consider adjusting --thinlto-cache-policy\n";

  // Prune for number of files.
  if (Policy.MaxSizeFiles)
    while (NumFiles > Policy.MaxSizeFiles)
      RemoveCacheFile();

  // Prune for size now if needed
```
- EN: This section centers on `remove` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `remove` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 281-320

```cpp
  if (Policy.MaxSizePercentageOfAvailableSpace > 0 || Policy.MaxSizeBytes > 0) {
    auto SpaceInfoOrErr = sys::fs::disk_space(Path);
    if (!SpaceInfoOrErr) {
      auto EC = SpaceInfoOrErr.getError();
      return createStringError(EC,
                               "cannot get available disk space for '%s': '%s'",
                               Path.str().c_str(), EC.message().c_str());
    }
    sys::fs::space_info SpaceInfo = SpaceInfoOrErr.get();
    auto AvailableSpace = TotalSize + SpaceInfo.free;

    if (Policy.MaxSizePercentageOfAvailableSpace == 0)
      Policy.MaxSizePercentageOfAvailableSpace = 100;
    if (Policy.MaxSizeBytes == 0)
      Policy.MaxSizeBytes = AvailableSpace;
    auto TotalSizeTarget = std::min<uint64_t>(
        AvailableSpace * Policy.MaxSizePercentageOfAvailableSpace / 100ull,
        Policy.MaxSizeBytes);

    LLVM_DEBUG(dbgs() << "Occupancy: " << ((100 * TotalSize) / AvailableSpace)
                      << "% target is: "
                      << Policy.MaxSizePercentageOfAvailableSpace << "%, "
                      << Policy.MaxSizeBytes << " bytes\n");

    size_t ActualSizes = 0;
    for (const auto &File : Files)
      if (File)
        ActualSizes += File->getBufferSize();

    if (ActualSizes > TotalSizeTarget)
      WithColor::warning()
          << "ThinLTO cache pruning happens since the total size of the cache "
             "files consumed by the current link job ("
          << ActualSizes << "  bytes) exceeds maximum cache size ("
          << TotalSizeTarget
          << " bytes); consider adjusting --thinlto-cache-policy\n";

    // Remove the oldest accessed files first, till we get below the threshold.
    while (TotalSize > TotalSizeTarget && FileInfo != FileInfos.end())
      RemoveCacheFile();
```
- EN: This section centers on `createStringError` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `createStringError` 等符号展开，负责创建并初始化辅助对象或状态。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 321-323

```cpp
  }
  return true;
}
```
- EN: This range returns the resulting value to its callers.
  CN: 这一段将结果返回给调用方。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `FileInfo`, `tie`, `writeTimestampFile`, `Out`, `parseDuration` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Formatting, emission, and outward serialization. / 重点关注格式化、输出与序列化。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/Support/CachePruning.h`, `llvm/ADT/StringRef.h`, `llvm/Support/Debug.h`, `llvm/Support/Errc.h`, `llvm/Support/Error.h`, `llvm/Support/FileSystem.h`, `llvm/Support/Path.h`, `llvm/Support/WithColor.h`, `llvm/Support/raw_ostream.h`
- Standard library / 标准库: `set`, `system_error`
- Other/system headers / 其他或系统头文件: None / 无
- Related symbols / 相关符号: `FileInfo`, `tie`, `writeTimestampFile`, `Out`, `parseDuration`, `seconds`
