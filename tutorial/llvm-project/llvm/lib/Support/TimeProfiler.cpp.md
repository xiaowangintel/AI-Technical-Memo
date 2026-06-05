# TimeProfiler.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/TimeProfiler.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements hierarchical time profiler.
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `TimeProfiler` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-60

```cpp
//===-- TimeProfiler.cpp - Hierarchical Time Profiler ---------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements hierarchical time profiler.
//
//===----------------------------------------------------------------------===//

#include "llvm/Support/TimeProfiler.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/STLFunctionalExtras.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/StringMap.h"
#include "llvm/Support/JSON.h"
#include "llvm/Support/Path.h"
#include "llvm/Support/Process.h"
#include "llvm/Support/Threading.h"
#include <algorithm>
#include <cassert>
#include <chrono>
#include <memory>
#include <mutex>
#include <string>
#include <vector>

using namespace llvm;

namespace {

using std::chrono::duration;
using std::chrono::duration_cast;
using std::chrono::microseconds;
using std::chrono::steady_clock;
using std::chrono::system_clock;
using std::chrono::time_point;
using std::chrono::time_point_cast;

struct TimeTraceProfilerInstances {
  std::mutex Lock;
  std::vector<TimeTraceProfiler *> List;
};

TimeTraceProfilerInstances &getTimeTraceProfilerInstances() {
  static TimeTraceProfilerInstances Instances;
  return Instances;
}

} // anonymous namespace

// Per Thread instance
static LLVM_THREAD_LOCAL TimeTraceProfiler *TimeTraceProfilerInstance = nullptr;

TimeTraceProfiler *llvm::getTimeTraceProfilerInstance() {
  return TimeTraceProfilerInstance;
}

```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 16 direct dependencies, including `llvm/Support/TimeProfiler.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/STLFunctionalExtras.h`, `llvm/ADT/SmallVector.h`.
  CN: 引入了 16 个直接依赖，其中包括 `llvm/Support/TimeProfiler.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/STLFunctionalExtras.h`, `llvm/ADT/SmallVector.h`。
- EN: This range defines or extends data types such as `TimeTraceProfilerInstances`.
  CN: 这一段定义或扩展了 `TimeTraceProfilerInstances` 等数据类型。

### Lines 61-120

```cpp
namespace {

using ClockType = steady_clock;
using TimePointType = time_point<ClockType>;
using DurationType = duration<ClockType::rep, ClockType::period>;
using CountAndDurationType = std::pair<size_t, DurationType>;
using NameAndCountAndDurationType =
    std::pair<std::string, CountAndDurationType>;

} // anonymous namespace

/// Represents an open or completed time section entry to be captured.
struct llvm::TimeTraceProfilerEntry {
  const TimePointType Start;
  TimePointType End;
  const std::string Name;
  TimeTraceMetadata Metadata;

  const TimeTraceEventType EventType = TimeTraceEventType::CompleteEvent;
  TimeTraceProfilerEntry(TimePointType &&S, TimePointType &&E, std::string &&N,
                         std::string &&Dt, TimeTraceEventType Et)
      : Start(std::move(S)), End(std::move(E)), Name(std::move(N)), Metadata(),
        EventType(Et) {
    Metadata.Detail = std::move(Dt);
  }

  TimeTraceProfilerEntry(TimePointType &&S, TimePointType &&E, std::string &&N,
                         TimeTraceMetadata &&Mt, TimeTraceEventType Et)
      : Start(std::move(S)), End(std::move(E)), Name(std::move(N)),
        Metadata(std::move(Mt)), EventType(Et) {}

  // Calculate timings for FlameGraph. Cast time points to microsecond precision
  // rather than casting duration. This avoids truncation issues causing inner
  // scopes overruning outer scopes.
  ClockType::rep getFlameGraphStartUs(TimePointType StartTime) const {
    return (time_point_cast<microseconds>(Start) -
            time_point_cast<microseconds>(StartTime))
        .count();
  }

  ClockType::rep getFlameGraphDurUs() const {
    return (time_point_cast<microseconds>(End) -
            time_point_cast<microseconds>(Start))
        .count();
  }
};

// Represents a currently open (in-progress) time trace entry. InstantEvents
// that happen during an open event are associated with the duration of this
// parent event and they are dropped if parent duration is shorter than
// the granularity.
struct InProgressEntry {
  TimeTraceProfilerEntry Event;
  std::vector<TimeTraceProfilerEntry> InstantEvents;

  InProgressEntry(TimePointType S, TimePointType E, std::string N,
                  std::string Dt, TimeTraceEventType Et)
      : Event(std::move(S), std::move(E), std::move(N), std::move(Dt), Et),
        InstantEvents() {}

```
- EN: This section centers on `TimeTraceProfilerEntry`, `getFlameGraphStartUs`, `getFlameGraphDurUs` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `TimeTraceProfilerEntry`, `getFlameGraphStartUs`, `getFlameGraphDurUs` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code updates helper containers and temporary state and returns the resulting value to its callers.
  CN: 在这一段中，代码更新辅助容器和临时状态，并将结果返回给调用方。

### Lines 121-180

```cpp
  InProgressEntry(TimePointType S, TimePointType E, std::string N,
                  TimeTraceMetadata Mt, TimeTraceEventType Et)
      : Event(std::move(S), std::move(E), std::move(N), std::move(Mt), Et),
        InstantEvents() {}
};

struct llvm::TimeTraceProfiler {
  TimeTraceProfiler(unsigned TimeTraceGranularity = 0, StringRef ProcName = "",
                    bool TimeTraceVerbose = false)
      : BeginningOfTime(system_clock::now()), StartTime(ClockType::now()),
        ProcName(ProcName), Pid(sys::Process::getProcessId()),
        Tid(llvm::get_threadid()), TimeTraceGranularity(TimeTraceGranularity),
        TimeTraceVerbose(TimeTraceVerbose) {
    llvm::get_thread_name(ThreadName);
  }

  TimeTraceProfilerEntry *
  begin(std::string Name, llvm::function_ref<std::string()> Detail,
        TimeTraceEventType EventType = TimeTraceEventType::CompleteEvent) {
    assert(EventType != TimeTraceEventType::InstantEvent &&
           "Instant Events don't have begin and end.");
    Stack.emplace_back(std::make_unique<InProgressEntry>(
        ClockType::now(), TimePointType(), std::move(Name), Detail(),
        EventType));
    return &Stack.back()->Event;
  }

  TimeTraceProfilerEntry *
  begin(std::string Name, llvm::function_ref<TimeTraceMetadata()> Metadata,
        TimeTraceEventType EventType = TimeTraceEventType::CompleteEvent) {
    assert(EventType != TimeTraceEventType::InstantEvent &&
           "Instant Events don't have begin and end.");
    Stack.emplace_back(std::make_unique<InProgressEntry>(
        ClockType::now(), TimePointType(), std::move(Name), Metadata(),
        EventType));
    return &Stack.back()->Event;
  }

  void insert(std::string Name, llvm::function_ref<std::string()> Detail) {
    if (Stack.empty())
      return;

    Stack.back()->InstantEvents.emplace_back(TimeTraceProfilerEntry(
        ClockType::now(), TimePointType(), std::move(Name), Detail(),
        TimeTraceEventType::InstantEvent));
  }

  void end() {
    assert(!Stack.empty() && "Must call begin() first");
    end(Stack.back()->Event);
  }

  void end(TimeTraceProfilerEntry &E) {
    assert(!Stack.empty() && "Must call begin() first");
    E.End = ClockType::now();

    // Calculate duration at full precision for overall counts.
    DurationType Duration = E.End - E.Start;

    const auto *Iter =
```
- EN: This section centers on `InProgressEntry`, `TimeTraceProfiler`, `get_thread_name` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `InProgressEntry`, `TimeTraceProfiler`, `get_thread_name` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 181-240

```cpp
        llvm::find_if(Stack, [&](const std::unique_ptr<InProgressEntry> &Val) {
          return &Val->Event == &E;
        });
    assert(Iter != Stack.end() && "Event not in the Stack");

    // Only include sections longer or equal to TimeTraceGranularity msec.
    if (duration_cast<microseconds>(Duration).count() >= TimeTraceGranularity) {
      Entries.emplace_back(E);
      for (auto &IE : Iter->get()->InstantEvents) {
        Entries.emplace_back(IE);
      }
    }

    // Track total time taken by each "name", but only the topmost levels of
    // them; e.g. if there's a template instantiation that instantiates other
    // templates from within, we only want to add the topmost one. "topmost"
    // happens to be the ones that don't have any currently open entries above
    // itself.
    if (llvm::none_of(llvm::drop_begin(llvm::reverse(Stack)),
                      [&](const std::unique_ptr<InProgressEntry> &Val) {
                        return Val->Event.Name == E.Name;
                      })) {
      auto &CountAndTotal = CountAndTotalPerName[E.Name];
      CountAndTotal.first++;
      CountAndTotal.second += Duration;
    };

    Stack.erase(Iter);
  }

  // Write events from this TimeTraceProfilerInstance and
  // ThreadTimeTraceProfilerInstances.
  void write(raw_pwrite_stream &OS) {
    // Acquire Mutex as reading ThreadTimeTraceProfilerInstances.
    auto &Instances = getTimeTraceProfilerInstances();
    std::lock_guard<std::mutex> Lock(Instances.Lock);
    assert(Stack.empty() &&
           "All profiler sections should be ended when calling write");
    assert(llvm::all_of(Instances.List,
                        [](const auto &TTP) { return TTP->Stack.empty(); }) &&
           "All profiler sections should be ended when calling write");

    json::OStream J(OS);
    J.objectBegin();
    J.attributeBegin("traceEvents");
    J.arrayBegin();

    // Emit all events for the main flame graph.
    auto writeEvent = [&](const auto &E, uint64_t Tid) {
      auto StartUs = E.getFlameGraphStartUs(StartTime);
      auto DurUs = E.getFlameGraphDurUs();

      J.object([&] {
        J.attribute("pid", Pid);
        J.attribute("tid", int64_t(Tid));
        J.attribute("ts", StartUs);
        if (E.EventType == TimeTraceEventType::AsyncEvent) {
          J.attribute("cat", E.Name);
          J.attribute("ph", "b");
          J.attribute("id", 0);
```
- EN: This section centers on `find_if`, `assert`, `write` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `find_if`, `assert`, `write` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 241-300

```cpp
        } else if (E.EventType == TimeTraceEventType::CompleteEvent) {
          J.attribute("ph", "X");
          J.attribute("dur", DurUs);
        } else { // instant event
          assert(E.EventType == TimeTraceEventType::InstantEvent &&
                 "InstantEvent expected");
          J.attribute("ph", "i");
        }
        J.attribute("name", E.Name);
        if (!E.Metadata.isEmpty()) {
          J.attributeObject("args", [&] {
            if (!E.Metadata.Detail.empty())
              J.attribute("detail", E.Metadata.Detail);
            if (!E.Metadata.File.empty())
              J.attribute("file", E.Metadata.File);
            if (E.Metadata.Line > 0)
              J.attribute("line", E.Metadata.Line);
          });
        }
      });

      if (E.EventType == TimeTraceEventType::AsyncEvent) {
        J.object([&] {
          J.attribute("pid", Pid);
          J.attribute("tid", int64_t(Tid));
          J.attribute("ts", StartUs + DurUs);
          J.attribute("cat", E.Name);
          J.attribute("ph", "e");
          J.attribute("id", 0);
          J.attribute("name", E.Name);
        });
      }
    };
    for (const TimeTraceProfilerEntry &E : Entries)
      writeEvent(E, this->Tid);
    for (const TimeTraceProfiler *TTP : Instances.List)
      for (const TimeTraceProfilerEntry &E : TTP->Entries)
        writeEvent(E, TTP->Tid);

    // Emit totals by section name as additional "thread" events, sorted from
    // longest one.
    // Find highest used thread id.
    uint64_t MaxTid = this->Tid;
    for (const TimeTraceProfiler *TTP : Instances.List)
      MaxTid = std::max(MaxTid, TTP->Tid);

    // Combine all CountAndTotalPerName from threads into one.
    StringMap<CountAndDurationType> AllCountAndTotalPerName;
    auto combineStat = [&](const auto &Stat) {
      StringRef Key = Stat.getKey();
      auto Value = Stat.getValue();
      auto &CountAndTotal = AllCountAndTotalPerName[Key];
      CountAndTotal.first += Value.first;
      CountAndTotal.second += Value.second;
    };
    for (const auto &Stat : CountAndTotalPerName)
      combineStat(Stat);
    for (const TimeTraceProfiler *TTP : Instances.List)
      for (const auto &Stat : TTP->CountAndTotalPerName)
        combineStat(Stat);
```
- EN: This section centers on `assert` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `assert` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 301-360

```cpp

    std::vector<NameAndCountAndDurationType> SortedTotals;
    SortedTotals.reserve(AllCountAndTotalPerName.size());
    for (const auto &Total : AllCountAndTotalPerName)
      SortedTotals.emplace_back(std::string(Total.getKey()), Total.getValue());

    llvm::sort(SortedTotals, [](const NameAndCountAndDurationType &A,
                                const NameAndCountAndDurationType &B) {
      return A.second.second > B.second.second;
    });

    // Report totals on separate threads of tracing file.
    uint64_t TotalTid = MaxTid + 1;
    for (const NameAndCountAndDurationType &Total : SortedTotals) {
      auto DurUs = duration_cast<microseconds>(Total.second.second).count();
      auto Count = AllCountAndTotalPerName[Total.first].first;

      J.object([&] {
        J.attribute("pid", Pid);
        J.attribute("tid", int64_t(TotalTid));
        J.attribute("ph", "X");
        J.attribute("ts", 0);
        J.attribute("dur", DurUs);
        J.attribute("name", "Total " + Total.first);
        J.attributeObject("args", [&] {
          J.attribute("count", int64_t(Count));
          J.attribute("avg ms", int64_t(DurUs / Count / 1000));
        });
      });

      ++TotalTid;
    }

    auto writeMetadataEvent = [&](const char *Name, uint64_t Tid,
                                  StringRef arg) {
      J.object([&] {
        J.attribute("cat", "");
        J.attribute("pid", Pid);
        J.attribute("tid", int64_t(Tid));
        J.attribute("ts", 0);
        J.attribute("ph", "M");
        J.attribute("name", Name);
        J.attributeObject("args", [&] { J.attribute("name", arg); });
      });
    };

    writeMetadataEvent("process_name", Tid, ProcName);
    writeMetadataEvent("thread_name", Tid, ThreadName);
    for (const TimeTraceProfiler *TTP : Instances.List)
      writeMetadataEvent("thread_name", TTP->Tid, TTP->ThreadName);

    J.arrayEnd();
    J.attributeEnd();

    // Emit the absolute time when this TimeProfiler started.
    // This can be used to combine the profiling data from
    // multiple processes and preserve actual time intervals.
    J.attribute("beginningOfTime",
                time_point_cast<microseconds>(BeginningOfTime)
                    .time_since_epoch()
```
- EN: This section centers on `sort`, `writeMetadataEvent` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `sort`, `writeMetadataEvent` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code iterates over collections, ranges, or records and updates helper containers and temporary state.
  CN: 在这一段中，代码遍历集合、区间或记录，并更新辅助容器和临时状态。

### Lines 361-420

```cpp
                    .count());

    J.objectEnd();
  }

  SmallVector<std::unique_ptr<InProgressEntry>, 16> Stack;
  SmallVector<TimeTraceProfilerEntry, 128> Entries;
  StringMap<CountAndDurationType> CountAndTotalPerName;
  // System clock time when the session was begun.
  const time_point<system_clock> BeginningOfTime;
  // Profiling clock time when the session was begun.
  const TimePointType StartTime;
  const std::string ProcName;
  const sys::Process::Pid Pid;
  SmallString<0> ThreadName;
  const uint64_t Tid;

  // Minimum time granularity (in microseconds)
  const unsigned TimeTraceGranularity;

  // Make time trace capture verbose event details (e.g. source filenames). This
  // can increase the size of the output by 2-3 times.
  const bool TimeTraceVerbose;
};

bool llvm::isTimeTraceVerbose() {
  return getTimeTraceProfilerInstance() &&
         getTimeTraceProfilerInstance()->TimeTraceVerbose;
}

void llvm::timeTraceProfilerInitialize(unsigned TimeTraceGranularity,
                                       StringRef ProcName,
                                       bool TimeTraceVerbose) {
  assert(TimeTraceProfilerInstance == nullptr &&
         "Profiler should not be initialized");
  TimeTraceProfilerInstance = new TimeTraceProfiler(
      TimeTraceGranularity, llvm::sys::path::filename(ProcName),
      TimeTraceVerbose);
}

// Removes all TimeTraceProfilerInstances.
// Called from main thread.
void llvm::timeTraceProfilerCleanup() {
  delete TimeTraceProfilerInstance;
  TimeTraceProfilerInstance = nullptr;

  auto &Instances = getTimeTraceProfilerInstances();
  std::lock_guard<std::mutex> Lock(Instances.Lock);
  for (auto *TTP : Instances.List)
    delete TTP;
  Instances.List.clear();
}

// Finish TimeTraceProfilerInstance on a worker thread.
// This doesn't remove the instance, just moves the pointer to global vector.
void llvm::timeTraceProfilerFinishThread() {
  auto &Instances = getTimeTraceProfilerInstances();
  std::lock_guard<std::mutex> Lock(Instances.Lock);
  Instances.List.push_back(TimeTraceProfilerInstance);
  TimeTraceProfilerInstance = nullptr;
```
- EN: This section centers on `isTimeTraceVerbose`, `getTimeTraceProfilerInstance`, `timeTraceProfilerInitialize` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `isTimeTraceVerbose`, `getTimeTraceProfilerInstance`, `timeTraceProfilerInitialize` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code iterates over collections, ranges, or records and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码遍历集合、区间或记录，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 421-480

```cpp
}

void llvm::timeTraceProfilerWrite(raw_pwrite_stream &OS) {
  assert(TimeTraceProfilerInstance != nullptr &&
         "Profiler object can't be null");
  TimeTraceProfilerInstance->write(OS);
}

Error llvm::timeTraceProfilerWrite(StringRef PreferredFileName,
                                   StringRef FallbackFileName) {
  assert(TimeTraceProfilerInstance != nullptr &&
         "Profiler object can't be null");

  std::string Path = PreferredFileName.str();
  if (Path.empty()) {
    Path = FallbackFileName == "-" ? "out" : FallbackFileName.str();
    Path += ".time-trace";
  }

  std::error_code EC;
  raw_fd_ostream OS(Path, EC, sys::fs::OF_TextWithCRLF);
  if (EC)
    return createStringError(EC, "Could not open " + Path);

  timeTraceProfilerWrite(OS);
  return Error::success();
}

TimeTraceProfilerEntry *llvm::timeTraceProfilerBegin(StringRef Name,
                                                     StringRef Detail) {
  if (TimeTraceProfilerInstance != nullptr)
    return TimeTraceProfilerInstance->begin(
        std::string(Name), [&]() { return std::string(Detail); },
        TimeTraceEventType::CompleteEvent);
  return nullptr;
}

TimeTraceProfilerEntry *
llvm::timeTraceProfilerBegin(StringRef Name,
                             llvm::function_ref<std::string()> Detail) {
  if (TimeTraceProfilerInstance != nullptr)
    return TimeTraceProfilerInstance->begin(std::string(Name), Detail,
                                            TimeTraceEventType::CompleteEvent);
  return nullptr;
}

TimeTraceProfilerEntry *
llvm::timeTraceProfilerBegin(StringRef Name,
                             llvm::function_ref<TimeTraceMetadata()> Metadata) {
  if (TimeTraceProfilerInstance != nullptr)
    return TimeTraceProfilerInstance->begin(std::string(Name), Metadata,
                                            TimeTraceEventType::CompleteEvent);
  return nullptr;
}

TimeTraceProfilerEntry *llvm::timeTraceAsyncProfilerBegin(StringRef Name,
                                                          StringRef Detail) {
  if (TimeTraceProfilerInstance != nullptr)
    return TimeTraceProfilerInstance->begin(
        std::string(Name), [&]() { return std::string(Detail); },
```
- EN: This section centers on `timeTraceProfilerWrite`, `assert`, `OS` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `timeTraceProfilerWrite`, `assert`, `OS` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 481-499

```cpp
        TimeTraceEventType::AsyncEvent);
  return nullptr;
}

void llvm::timeTraceAddInstantEvent(StringRef Name,
                                    llvm::function_ref<std::string()> Detail) {
  if (TimeTraceProfilerInstance != nullptr)
    TimeTraceProfilerInstance->insert(std::string(Name), Detail);
}

void llvm::timeTraceProfilerEnd() {
  if (TimeTraceProfilerInstance != nullptr)
    TimeTraceProfilerInstance->end();
}

void llvm::timeTraceProfilerEnd(TimeTraceProfilerEntry *E) {
  if (TimeTraceProfilerInstance != nullptr)
    TimeTraceProfilerInstance->end(*E);
}
```
- EN: This section centers on `timeTraceAddInstantEvent`, `timeTraceProfilerEnd` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `timeTraceAddInstantEvent`, `timeTraceProfilerEnd` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and returns the resulting value to its callers.
  CN: 在这一段中，代码检查条件并处理边界情况，并将结果返回给调用方。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `TimeTraceProfilerInstances`, `llvm`, `InProgressEntry`, `TimeTraceProfilerEntry`, `getFlameGraphStartUs`, `getFlameGraphDurUs` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Formatting, emission, and outward serialization. / 重点关注格式化、输出与序列化。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/Support/TimeProfiler.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/STLFunctionalExtras.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringMap.h`, `llvm/Support/JSON.h`, `llvm/Support/Path.h`, `llvm/Support/Process.h`, `llvm/Support/Threading.h`
- Standard library / 标准库: `algorithm`, `chrono`, `memory`, `mutex`, `string`, `vector`
- Other/system headers / 其他或系统头文件: `cassert`
- Related symbols / 相关符号: `TimeTraceProfilerInstances`, `llvm`, `InProgressEntry`, `TimeTraceProfilerEntry`, `getFlameGraphStartUs`, `getFlameGraphDurUs`, `TimeTraceProfiler`
