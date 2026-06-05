# Timing.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Support/Timing.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Facilities to measure and provide statistics on execution time.
  - **CN**: 实现供 MLIR 库与工具共享使用的支持工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===- Timing.cpp - Execution time measurement facilities -----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Facilities to measure and provide statistics on execution time.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及高层摘要。

### Lines 12-26
```cpp

#include "mlir/Support/Timing.h"
#include "mlir/Support/ThreadLocalCache.h"
#include "llvm/ADT/MapVector.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/ADT/StringMap.h"
#include "llvm/ADT/StringSet.h"
#include "llvm/Support/Allocator.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Format.h"
#include "llvm/Support/ManagedStatic.h"
#include "llvm/Support/RWMutex.h"
#include "llvm/Support/Threading.h"
#include "llvm/Support/raw_ostream.h"

```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/Support/Timing.h`, `mlir/Support/ThreadLocalCache.h`, `llvm/ADT/MapVector.h`, `llvm/ADT/Statistic.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/Support/Timing.h`, `mlir/Support/ThreadLocalCache.h`, `llvm/ADT/MapVector.h`, `llvm/ADT/Statistic.h`。

### Lines 27-37
```cpp
#include <chrono>
#include <optional>

using namespace mlir;
using namespace detail;
using DisplayMode = DefaultTimingManager::DisplayMode;
using OutputFormat = DefaultTimingManager::OutputFormat;

constexpr llvm::StringLiteral kTimingDescription =
    "... Execution time report ...";

```
- **EN**: Pulls in the declarations needed by this translation unit, including `chrono`, `optional`.
- **CN**: 引入该编译单元所需的声明，其中包括 `chrono`, `optional`。

### Lines 38-51
```cpp
//===----------------------------------------------------------------------===//
// TimingManager
//===----------------------------------------------------------------------===//

namespace mlir {
namespace detail {
/// Private implementation details of the `TimingManager`.
class TimingManagerImpl {
public:
  // Identifier allocator, map, and mutex for thread safety.
  llvm::BumpPtrAllocator identifierAllocator;
  llvm::StringSet<llvm::BumpPtrAllocator &> identifiers;
  llvm::sys::SmartRWMutex<true> identifierMutex;

```
- **EN**: Introduces declarations for `mlir`, `detail`, `TimingManagerImpl`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `mlir`、`detail`、`TimingManagerImpl` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 52-61
```cpp
  /// A thread local cache of identifiers to reduce lock contention.
  ThreadLocalCache<
      llvm::StringMap<llvm::StringMapEntry<llvm::EmptyStringSetTag> *>>
      localIdentifierCache;

  TimingManagerImpl() : identifiers(identifierAllocator) {}
};
} // namespace detail
} // namespace mlir

```
- **EN**: Implements logic around `TimingManagerImpl`.
- **CN**: 围绕 `TimingManagerImpl` 实现具体逻辑。

### Lines 62-71
```cpp
TimingManager::TimingManager() : impl(std::make_unique<TimingManagerImpl>()) {}

TimingManager::~TimingManager() = default;

/// Get the root timer of this timing manager.
Timer TimingManager::getRootTimer() {
  auto rt = rootTimer();
  return rt ? Timer(*this, *rt) : Timer();
}

```
- **EN**: Implements logic around `TimingManager`, `~TimingManager`, `getRootTimer`, `rootTimer`, and 1 more symbols.
- **CN**: 围绕 `TimingManager`、`~TimingManager`、`getRootTimer`、`rootTimer` 等另外 1 个符号 实现具体逻辑。

### Lines 72-81
```cpp
/// Get the root timer of this timing manager wrapped in a `TimingScope`.
TimingScope TimingManager::getRootScope() {
  return TimingScope(getRootTimer());
}

//===----------------------------------------------------------------------===//
// Identifier uniquing
//===----------------------------------------------------------------------===//

/// Return an identifier for the specified string.
```
- **EN**: Implements logic around `getRootScope`, `TimingScope`.
- **CN**: 围绕 `getRootScope`、`TimingScope` 实现具体逻辑。

### Lines 82-98
```cpp
TimingIdentifier TimingIdentifier::get(StringRef str, TimingManager &tm) {
  // Check for an existing instance in the local cache.
  auto &impl = *tm.impl;
  auto *&localEntry = (*impl.localIdentifierCache)[str];
  if (localEntry)
    return TimingIdentifier(localEntry);

  // Check for an existing identifier in read-only mode.
  {
    llvm::sys::SmartScopedReader<true> contextLock(impl.identifierMutex);
    auto it = impl.identifiers.find(str);
    if (it != impl.identifiers.end()) {
      localEntry = &*it;
      return TimingIdentifier(localEntry);
    }
  }

```
- **EN**: Implements logic around `get`, `TimingIdentifier`, `contextLock`, `find`, and 1 more symbols.
- **CN**: 围绕 `get`、`TimingIdentifier`、`contextLock`、`find` 等另外 1 个符号 实现具体逻辑。

### Lines 99-108
```cpp
  // Acquire a writer-lock so that we can safely create the new instance.
  llvm::sys::SmartScopedWriter<true> contextLock(impl.identifierMutex);
  auto it = impl.identifiers.insert(str).first;
  localEntry = &*it;
  return TimingIdentifier(localEntry);
}

//===----------------------------------------------------------------------===//
// Helpers for time record printing
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `contextLock`, `insert`, `TimingIdentifier`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `contextLock`、`insert`、`TimingIdentifier` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 109-122
```cpp

namespace {

class OutputTextStrategy : public OutputStrategy {
public:
  OutputTextStrategy(raw_ostream &os) : OutputStrategy(os) {}

  void printHeader(const TimeRecord &total) override {
    // Figure out how many spaces to description name.
    unsigned padding = (80 - kTimingDescription.size()) / 2;
    os << "===" << std::string(73, '-') << "===\n";
    os.indent(padding) << kTimingDescription << '\n';
    os << "===" << std::string(73, '-') << "===\n";

```
- **EN**: Introduces declarations for `OutputTextStrategy`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `OutputTextStrategy` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 123-140
```cpp
    // Print the total time followed by the section headers.
    os << llvm::format("  Total Execution Time: %.4f seconds\n\n", total.wall);
    if (total.user != total.wall)
      os << "  ----User Time----";
    os << "  ----Wall Time----  ----Name----\n";
  }

  void printFooter() override { os.flush(); }

  void printTime(const TimeRecord &time, const TimeRecord &total) override {
    if (total.user != total.wall) {
      os << llvm::format("  %8.4f (%5.1f%%)", time.user,
                         100.0 * time.user / total.user);
    }
    os << llvm::format("  %8.4f (%5.1f%%)  ", time.wall,
                       100.0 * time.wall / total.wall);
  }

```
- **EN**: Implements logic around `format`, `printFooter`, `printTime`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `format`、`printFooter`、`printTime` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 141-152
```cpp
  void printListEntry(StringRef name, const TimeRecord &time,
                      const TimeRecord &total, bool lastEntry) override {
    printTime(time, total);
    os << name << "\n";
  }

  void printTreeEntry(unsigned indent, StringRef name, const TimeRecord &time,
                      const TimeRecord &total) override {
    printTime(time, total);
    os.indent(indent) << name << "\n";
  }

```
- **EN**: Implements logic around `printListEntry`, `printTime`, `printTreeEntry`, `indent`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `printListEntry`、`printTime`、`printTreeEntry`、`indent` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 153-166
```cpp
  void printTreeEntryEnd(unsigned indent, bool lastEntry) override {}
};

class OutputJsonStrategy : public OutputStrategy {
public:
  OutputJsonStrategy(raw_ostream &os) : OutputStrategy(os) {}

  void printHeader(const TimeRecord &total) override { os << "[" << "\n"; }

  void printFooter() override {
    os << "]" << "\n";
    os.flush();
  }

```
- **EN**: Introduces declarations for `OutputJsonStrategy`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `OutputJsonStrategy` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 167-181
```cpp
  void printTime(const TimeRecord &time, const TimeRecord &total) override {
    if (total.user != total.wall) {
      os << "\"user\": {";
      os << "\"duration\": " << llvm::format("%8.4f", time.user) << ", ";
      os << "\"percentage\": "
         << llvm::format("%5.1f", 100.0 * time.user / total.user);
      os << "}, ";
    }
    os << "\"wall\": {";
    os << "\"duration\": " << llvm::format("%8.4f", time.wall) << ", ";
    os << "\"percentage\": "
       << llvm::format("%5.1f", 100.0 * time.wall / total.wall);
    os << "}";
  }

```
- **EN**: Implements logic around `printTime`, `format`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `printTime`、`format` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 182-192
```cpp
  void printListEntry(StringRef name, const TimeRecord &time,
                      const TimeRecord &total, bool lastEntry) override {
    os << "{";
    printTime(time, total);
    os << ", \"name\": " << "\"" << name << "\"";
    os << "}";
    if (!lastEntry)
      os << ",";
    os << "\n";
  }

```
- **EN**: Implements logic around `printListEntry`, `printTime`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `printListEntry`、`printTime` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 193-209
```cpp
  void printTreeEntry(unsigned indent, StringRef name, const TimeRecord &time,
                      const TimeRecord &total) override {
    os.indent(indent) << "{";
    printTime(time, total);
    os << ", \"name\": " << "\"" << name << "\"";
    os << ", \"passes\": [" << "\n";
  }

  void printTreeEntryEnd(unsigned indent, bool lastEntry) override {
    os.indent(indent) << "{}]";
    os << "}";
    if (!lastEntry)
      os << ",";
    os << "\n";
  }
};

```
- **EN**: Implements logic around `printTreeEntry`, `indent`, `printTime`, `printTreeEntryEnd`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `printTreeEntry`、`indent`、`printTime`、`printTreeEntryEnd` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 210-219
```cpp
} // namespace

//===----------------------------------------------------------------------===//
// Timer Implementation for DefaultTimingManager
//===----------------------------------------------------------------------===//

namespace {

/// A timer used to sample execution time.
///
```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 220-230
```cpp
/// Separately tracks wall time and user time to account for parallel threads of
/// execution. Timers are intended to be started and stopped multiple times.
/// Each start and stop will add to the timer's wall and user time.
class TimerImpl {
public:
  using ChildrenMap = llvm::MapVector<const void *, std::unique_ptr<TimerImpl>>;
  using AsyncChildrenMap = llvm::DenseMap<uint64_t, ChildrenMap>;

  TimerImpl(std::string &&name, std::unique_ptr<OutputStrategy> &output)
      : threadId(llvm::get_threadid()), name(name), output(output) {}

```
- **EN**: Introduces declarations for `TimerImpl`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `TimerImpl` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 231-240
```cpp
  /// Start the timer.
  void start() { startTime = std::chrono::steady_clock::now(); }

  /// Stop the timer.
  void stop() {
    auto newTime = std::chrono::steady_clock::now() - startTime;
    wallTime += newTime;
    userTime += newTime;
  }

```
- **EN**: Implements logic around `start`, `stop`, `now`.
- **CN**: 围绕 `start`、`stop`、`now` 实现具体逻辑。

### Lines 241-254
```cpp
  /// Create a child timer nested within this one. Multiple calls to this
  /// function with the same unique identifier `id` will return the same child
  /// timer.
  ///
  /// This function can be called from other threads, as long as this timer
  /// outlives any uses of the child timer on the other thread.
  TimerImpl *nest(const void *id, function_ref<std::string()> nameBuilder) {
    auto tid = llvm::get_threadid();
    if (tid == threadId)
      return nestTail(children[id], nameBuilder);
    std::unique_lock<std::mutex> lock(asyncMutex);
    return nestTail(asyncChildren[tid][id], nameBuilder);
  }

```
- **EN**: Implements logic around `nest`, `get_threadid`, `nestTail`, `lock`.
- **CN**: 围绕 `nest`、`get_threadid`、`nestTail`、`lock` 实现具体逻辑。

### Lines 255-264
```cpp
  /// Tail-called from `nest()`.
  TimerImpl *nestTail(std::unique_ptr<TimerImpl> &child,
                      function_ref<std::string()> nameBuilder) {
    if (!child)
      child = std::make_unique<TimerImpl>(nameBuilder(), output);
    return child.get();
  }

  /// Finalize this timer and all its children.
  ///
```
- **EN**: Implements logic around `nestTail`, `string`, `make_unique`, `get`.
- **CN**: 围绕 `nestTail`、`string`、`make_unique`、`get` 实现具体逻辑。

### Lines 265-275
```cpp
  /// If this timer has async children, which happens if `nest()` was called
  /// from another thread, this function merges the async childr timers into the
  /// main list of child timers.
  ///
  /// Caution: Call this function only after all nested timers running on other
  /// threads no longer need their timers!
  void finalize() {
    addAsyncUserTime();
    mergeAsyncChildren();
  }

```
- **EN**: Implements logic around `finalize`, `addAsyncUserTime`, `mergeAsyncChildren`.
- **CN**: 围绕 `finalize`、`addAsyncUserTime`、`mergeAsyncChildren` 实现具体逻辑。

### Lines 276-292
```cpp
  /// Add the user time of all async children to this timer's user time. This is
  /// necessary since the user time already contains all regular child timers,
  /// but not the asynchronous ones (by the nesting nature of the timers).
  std::chrono::nanoseconds addAsyncUserTime() {
    auto added = std::chrono::nanoseconds(0);
    for (auto &child : children)
      added += child.second->addAsyncUserTime();
    for (auto &thread : asyncChildren) {
      for (auto &child : thread.second) {
        child.second->addAsyncUserTime();
        added += child.second->userTime;
      }
    }
    userTime += added;
    return added;
  }

```
- **EN**: Implements logic around `addAsyncUserTime`, `nanoseconds`.
- **CN**: 围绕 `addAsyncUserTime`、`nanoseconds` 实现具体逻辑。

### Lines 293-302
```cpp
  /// Ensure that this timer and recursively all its children have their async
  /// children folded into the main map of children.
  void mergeAsyncChildren() {
    for (auto &child : children)
      child.second->mergeAsyncChildren();
    mergeChildren(std::move(asyncChildren));
    assert(asyncChildren.empty());
  }

  /// Merge multiple child timers into this timer.
```
- **EN**: Implements logic around `mergeAsyncChildren`, `mergeChildren`, `assert`.
- **CN**: 围绕 `mergeAsyncChildren`、`mergeChildren`、`assert` 实现具体逻辑。

### Lines 303-318
```cpp
  ///
  /// Children in `other` are added as children to this timer, or, if this timer
  /// already contains a child with the corresponding unique identifier, are
  /// merged into the existing child.
  void mergeChildren(ChildrenMap &&other) {
    if (children.empty()) {
      children = std::move(other);
      for (auto &child : children)
        child.second->mergeAsyncChildren();
    } else {
      for (auto &child : other)
        mergeChild(child.first, std::move(child.second));
      other.clear();
    }
  }

```
- **EN**: Implements logic around `mergeChildren`, `empty`, `move`, `mergeAsyncChildren`, and 2 more symbols.
- **CN**: 围绕 `mergeChildren`、`empty`、`move`、`mergeAsyncChildren` 等另外 2 个符号 实现具体逻辑。

### Lines 319-328
```cpp
  /// See above.
  void mergeChildren(AsyncChildrenMap &&other) {
    for (auto &thread : other) {
      mergeChildren(std::move(thread.second));
    }
    other.clear();
  }

  /// Merge a child timer into this timer for a given unique identifier.
  ///
```
- **EN**: Implements logic around `mergeChildren`, `clear`.
- **CN**: 围绕 `mergeChildren`、`clear` 实现具体逻辑。

### Lines 329-344
```cpp
  /// Moves all child and async child timers of `other` into this timer's child
  /// for the given unique identifier.
  void mergeChild(const void *id, std::unique_ptr<TimerImpl> &&other) {
    auto &into = children[id];
    if (!into) {
      into = std::move(other);
      into->mergeAsyncChildren();
    } else {
      into->wallTime = std::max(into->wallTime, other->wallTime);
      into->userTime += other->userTime;
      into->mergeChildren(std::move(other->children));
      into->mergeChildren(std::move(other->asyncChildren));
      other.reset();
    }
  }

```
- **EN**: Implements logic around `mergeChild`, `move`, `mergeAsyncChildren`, `max`, and 2 more symbols.
- **CN**: 围绕 `mergeChild`、`move`、`mergeAsyncChildren`、`max` 等另外 2 个符号 实现具体逻辑。

### Lines 345-361
```cpp
  /// Dump a human-readable tree representation of the timer and its children.
  /// This is useful for debugging the timing mechanisms and structure of the
  /// timers.
  void dump(raw_ostream &os, unsigned indent = 0, unsigned markThreadId = 0) {
    auto time = getTimeRecord();
    os << std::string(indent * 2, ' ') << name << " [" << threadId << "]"
       << llvm::format("  %7.4f / %7.4f", time.user, time.wall);
    if (threadId != markThreadId && markThreadId != 0)
      os << " (*)";
    os << "\n";
    for (auto &child : children)
      child.second->dump(os, indent + 1, threadId);
    for (auto &thread : asyncChildren)
      for (auto &child : thread.second)
        child.second->dump(os, indent + 1, threadId);
  }

```
- **EN**: Implements logic around `dump`, `getTimeRecord`, `string`, `format`.
- **CN**: 围绕 `dump`、`getTimeRecord`、`string`、`format` 实现具体逻辑。

### Lines 362-371
```cpp
  /// Returns the time for this timer in seconds.
  TimeRecord getTimeRecord() {
    return TimeRecord(
        std::chrono::duration_cast<std::chrono::duration<double>>(wallTime)
            .count(),
        std::chrono::duration_cast<std::chrono::duration<double>>(userTime)
            .count());
  }

  /// Print the timing result in list mode.
```
- **EN**: Implements logic around `getTimeRecord`, `TimeRecord`, `duration`, `count`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `getTimeRecord`、`TimeRecord`、`duration`、`count` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 372-381
```cpp
  void printAsList(TimeRecord total) {
    // Flatten the leaf timers in the tree and merge them by name.
    llvm::StringMap<TimeRecord> mergedTimers;
    std::function<void(TimerImpl *)> addTimer = [&](TimerImpl *timer) {
      mergedTimers[timer->name] += timer->getTimeRecord();
      for (auto &children : timer->children)
        addTimer(children.second.get());
    };
    addTimer(this);

```
- **EN**: Implements logic around `printAsList`, `function`, `getTimeRecord`, `addTimer`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `printAsList`、`function`、`getTimeRecord`、`addTimer` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 382-392
```cpp
    // Sort the timing information by wall time.
    std::vector<std::pair<StringRef, TimeRecord>> timerNameAndTime;
    for (auto &it : mergedTimers)
      timerNameAndTime.emplace_back(it.first(), it.second);
    llvm::array_pod_sort(timerNameAndTime.begin(), timerNameAndTime.end(),
                         [](const std::pair<StringRef, TimeRecord> *lhs,
                            const std::pair<StringRef, TimeRecord> *rhs) {
                           return llvm::array_pod_sort_comparator<double>(
                               &rhs->second.wall, &lhs->second.wall);
                         });

```
- **EN**: Implements logic around `emplace_back`, `array_pod_sort`, `array_pod_sort_comparator`.
- **CN**: 围绕 `emplace_back`、`array_pod_sort`、`array_pod_sort_comparator` 实现具体逻辑。

### Lines 393-412
```cpp
    // Print the timing information sequentially.
    for (auto &timeData : timerNameAndTime)
      output->printListEntry(timeData.first, timeData.second, total);
  }

  /// Print the timing result in tree mode.
  void printAsTree(TimeRecord total, unsigned indent = 0) {
    unsigned childIndent = indent;
    if (!hidden) {
      output->printTreeEntry(indent, name, getTimeRecord(), total);
      childIndent += 2;
    }
    for (auto &child : children) {
      child.second->printAsTree(total, childIndent);
    }
    if (!hidden) {
      output->printTreeEntryEnd(indent);
    }
  }

```
- **EN**: Implements logic around `printListEntry`, `printAsTree`, `printTreeEntry`, `printTreeEntryEnd`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `printListEntry`、`printAsTree`、`printTreeEntry`、`printTreeEntryEnd` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 413-428
```cpp
  /// Print the current timing information.
  void print(DisplayMode displayMode) {
    // Print the banner.
    auto total = getTimeRecord();
    output->printHeader(total);

    // Defer to a specialized printer for each display mode.
    switch (displayMode) {
    case DisplayMode::List:
      printAsList(total);
      break;
    case DisplayMode::Tree:
      printAsTree(total);
      break;
    }

```
- **EN**: Implements logic around `print`, `getTimeRecord`, `printHeader`, `printAsList`, and 1 more symbols; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `print`、`getTimeRecord`、`printHeader`、`printAsList` 等另外 1 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 429-438
```cpp
    // Print the top-level time not accounted for by child timers, and the
    // total.
    auto rest = total;
    for (auto &child : children)
      rest -= child.second->getTimeRecord();
    output->printListEntry("Rest", rest, total);
    output->printListEntry("Total", total, total, /*lastEntry=*/true);
    output->printFooter();
  }

```
- **EN**: Implements logic around `getTimeRecord`, `printListEntry`, `printFooter`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `getTimeRecord`、`printListEntry`、`printFooter` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 439-448
```cpp
  /// The last time instant at which the timer was started.
  std::chrono::time_point<std::chrono::steady_clock> startTime;

  /// Accumulated wall time. If multiple threads of execution are merged into
  /// this timer, the wall time will hold the maximum wall time of each thread
  /// of execution.
  std::chrono::nanoseconds wallTime = std::chrono::nanoseconds(0);

  /// Accumulated user time. If multiple threads of execution are merged into
  /// this timer, each thread's user time is added here.
```
- **EN**: Implements logic around `nanoseconds`.
- **CN**: 围绕 `nanoseconds` 实现具体逻辑。

### Lines 449-459
```cpp
  std::chrono::nanoseconds userTime = std::chrono::nanoseconds(0);

  /// The thread on which this timer is running.
  uint64_t threadId;

  /// A descriptive name for this timer.
  std::string name;

  /// Whether to omit this timer from reports and directly show its children.
  bool hidden = false;

```
- **EN**: Implements logic around `nanoseconds`.
- **CN**: 围绕 `nanoseconds` 实现具体逻辑。

### Lines 460-470
```cpp
  /// Child timers on the same thread the timer itself. We keep at most one
  /// timer per unique identifier.
  ChildrenMap children;

  /// Child timers on other threads. We keep at most one timer per unique
  /// identifier.
  AsyncChildrenMap asyncChildren;

  /// Mutex for the async children.
  std::mutex asyncMutex;

```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 471-480
```cpp
  std::unique_ptr<OutputStrategy> &output;
};

} // namespace

//===----------------------------------------------------------------------===//
// DefaultTimingManager
//===----------------------------------------------------------------------===//

namespace mlir {
```
- **EN**: Introduces declarations for `mlir`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `mlir` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 481-491
```cpp
namespace detail {

/// Implementation details of the `DefaultTimingManager`.
class DefaultTimingManagerImpl {
public:
  /// Whether we should do our work or not.
  bool enabled = false;

  /// The configured display mode.
  DisplayMode displayMode = DisplayMode::Tree;

```
- **EN**: Introduces declarations for `detail`, `DefaultTimingManagerImpl`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `detail`、`DefaultTimingManagerImpl` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 492-504
```cpp
  /// The root timer.
  std::unique_ptr<TimerImpl> rootTimer;
};

} // namespace detail
} // namespace mlir

DefaultTimingManager::DefaultTimingManager()
    : impl(std::make_unique<DefaultTimingManagerImpl>()),
      out(std::make_unique<OutputTextStrategy>(llvm::errs())) {
  clear(); // initializes the root timer
}

```
- **EN**: Implements logic around `DefaultTimingManager`, `impl`, `out`, `clear`.
- **CN**: 围绕 `DefaultTimingManager`、`impl`、`out`、`clear` 实现具体逻辑。

### Lines 505-517
```cpp
DefaultTimingManager::~DefaultTimingManager() { print(); }

/// Enable or disable execution time sampling.
void DefaultTimingManager::setEnabled(bool enabled) { impl->enabled = enabled; }

/// Return whether execution time sampling is enabled.
bool DefaultTimingManager::isEnabled() const { return impl->enabled; }

/// Change the display mode.
void DefaultTimingManager::setDisplayMode(DisplayMode displayMode) {
  impl->displayMode = displayMode;
}

```
- **EN**: Implements logic around `~DefaultTimingManager`, `setEnabled`, `isEnabled`, `setDisplayMode`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `~DefaultTimingManager`、`setEnabled`、`isEnabled`、`setDisplayMode` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 518-527
```cpp
/// Return the current display mode;
DefaultTimingManager::DisplayMode DefaultTimingManager::getDisplayMode() const {
  return impl->displayMode;
}

/// Change the stream where the output will be printed to.
void DefaultTimingManager::setOutput(std::unique_ptr<OutputStrategy> output) {
  out = std::move(output);
}

```
- **EN**: Implements logic around `getDisplayMode`, `setOutput`, `move`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `getDisplayMode`、`setOutput`、`move` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 528-537
```cpp
/// Print and clear the timing results.
void DefaultTimingManager::print() {
  if (impl->enabled) {
    impl->rootTimer->finalize();
    impl->rootTimer->print(impl->displayMode);
  }
  clear();
}

/// Clear the timing results.
```
- **EN**: Implements logic around `print`, `finalize`, `clear`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `print`、`finalize`、`clear` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 538-547
```cpp
void DefaultTimingManager::clear() {
  impl->rootTimer = std::make_unique<TimerImpl>("root", out);
  impl->rootTimer->hidden = true;
}

/// Debug print the timer data structures to an output stream.
void DefaultTimingManager::dumpTimers(raw_ostream &os) {
  impl->rootTimer->dump(os);
}

```
- **EN**: Implements logic around `clear`, `make_unique`, `dumpTimers`, `dump`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `clear`、`make_unique`、`dumpTimers`、`dump` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 548-559
```cpp
/// Debug print the timers as a list.
void DefaultTimingManager::dumpAsList(raw_ostream &os) {
  impl->rootTimer->finalize();
  impl->rootTimer->print(DisplayMode::List);
}

/// Debug print the timers as a tree.
void DefaultTimingManager::dumpAsTree(raw_ostream &os) {
  impl->rootTimer->finalize();
  impl->rootTimer->print(DisplayMode::Tree);
}

```
- **EN**: Implements logic around `dumpAsList`, `finalize`, `print`, `dumpAsTree`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `dumpAsList`、`finalize`、`print`、`dumpAsTree` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 560-569
```cpp
std::optional<void *> DefaultTimingManager::rootTimer() {
  if (impl->enabled)
    return impl->rootTimer.get();
  return std::nullopt;
}

void DefaultTimingManager::startTimer(void *handle) {
  static_cast<TimerImpl *>(handle)->start();
}

```
- **EN**: Implements logic around `rootTimer`, `get`, `startTimer`, `start`.
- **CN**: 围绕 `rootTimer`、`get`、`startTimer`、`start` 实现具体逻辑。

### Lines 570-582
```cpp
void DefaultTimingManager::stopTimer(void *handle) {
  static_cast<TimerImpl *>(handle)->stop();
}

void *DefaultTimingManager::nestTimer(void *handle, const void *id,
                                      function_ref<std::string()> nameBuilder) {
  return static_cast<TimerImpl *>(handle)->nest(id, nameBuilder);
}

void DefaultTimingManager::hideTimer(void *handle) {
  static_cast<TimerImpl *>(handle)->hidden = true;
}

```
- **EN**: Implements logic around `stopTimer`, `stop`, `nestTimer`, `string`, and 2 more symbols.
- **CN**: 围绕 `stopTimer`、`stop`、`nestTimer`、`string` 等另外 2 个符号 实现具体逻辑。

### Lines 583-602
```cpp
//===----------------------------------------------------------------------===//
// DefaultTimingManager Command Line Options
//===----------------------------------------------------------------------===//

namespace {
struct DefaultTimingManagerOptions {
  llvm::cl::opt<bool> timing{"mlir-timing",
                             llvm::cl::desc("Display execution times"),
                             llvm::cl::init(false)};
  llvm::cl::opt<DisplayMode> displayMode{
      "mlir-timing-display", llvm::cl::desc("Display method for timing data"),
      llvm::cl::init(DisplayMode::Tree),
      llvm::cl::values(
          clEnumValN(DisplayMode::List, "list",
                     "display the results in a list sorted by total time"),
          clEnumValN(DisplayMode::Tree, "tree",
                     "display the results ina with a nested tree view"))};
  llvm::cl::opt<OutputFormat> outputFormat{
      "mlir-output-format", llvm::cl::desc("Output format for timing data"),
      llvm::cl::init(OutputFormat::Text),
```
- **EN**: Introduces declarations for `DefaultTimingManagerOptions`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `DefaultTimingManagerOptions` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 603-616
```cpp
      llvm::cl::values(clEnumValN(OutputFormat::Text, "text",
                                  "display the results in text format"),
                       clEnumValN(OutputFormat::Json, "json",
                                  "display the results in JSON format"))};
};
} // namespace

static llvm::ManagedStatic<DefaultTimingManagerOptions> options;

void mlir::registerDefaultTimingManagerCLOptions() {
  // Make sure that the options struct has been constructed.
  *options;
}

```
- **EN**: Implements logic around `values`, `clEnumValN`, `registerDefaultTimingManagerCLOptions`.
- **CN**: 围绕 `values`、`clEnumValN`、`registerDefaultTimingManagerCLOptions` 实现具体逻辑。

### Lines 617-635
```cpp
void mlir::applyDefaultTimingManagerCLOptions(DefaultTimingManager &tm) {
  if (!options.isConstructed())
    return;
  tm.setEnabled(options->timing);
  tm.setDisplayMode(options->displayMode);
  tm.setOutput(createOutputStrategy(options->outputFormat, llvm::errs()));
}

std::unique_ptr<OutputStrategy>
mlir::createOutputStrategy(DefaultTimingManager::OutputFormat fmt,
                           raw_ostream &os) {
  switch (fmt) {
  case OutputFormat::Text:
    return std::make_unique<OutputTextStrategy>(os);
  case OutputFormat::Json:
    return std::make_unique<OutputJsonStrategy>(os);
  }
  llvm_unreachable("Invalid output format");
}
```
- **EN**: Implements logic around `applyDefaultTimingManagerCLOptions`, `isConstructed`, `setEnabled`, `setDisplayMode`, and 4 more symbols.
- **CN**: 围绕 `applyDefaultTimingManagerCLOptions`、`isConstructed`、`setEnabled`、`setDisplayMode` 等另外 4 个符号 实现具体逻辑。

## Key Concepts / 关键概念

- **Shared support utilities / 共享支持工具**:
  - **EN**: Provides reusable helpers that are intentionally lower level than dialect-specific logic.
  - **CN**: 提供刻意保持在方言逻辑之下层级的可复用辅助能力。
- **Streaming output / 流式输出**:
  - **EN**: Text, diagnostics, or generated content are emitted through LLVM stream APIs.
  - **CN**: 通过 LLVM 流 API 输出文本、诊断或生成内容。
- **LLVM ADT containers / LLVM ADT 容器**:
  - **EN**: The implementation uses LLVM containers for performance-conscious storage and traversal.
  - **CN**: 该实现使用 LLVM 容器来进行兼顾性能的存储与遍历。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `mlir/Support/Timing.h`, `mlir/Support/ThreadLocalCache.h`, `llvm/ADT/MapVector.h`, `llvm/ADT/Statistic.h`, `llvm/ADT/StringMap.h`, `llvm/ADT/StringSet.h`, `llvm/Support/Allocator.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Format.h`, `llvm/Support/ManagedStatic.h` ... (+3 more)
- **Standard-library headers / 标准库头文件**: `<chrono>`, `<optional>`
- **Subsystem categories / 子系统类别**: LLVM support-library helpers / LLVM Support 库辅助工具 (7), LLVM ADT containers and utilities / LLVM ADT 容器与工具 (4), shared MLIR support helpers / 共享的 MLIR 支持工具 (2)
