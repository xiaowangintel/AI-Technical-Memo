# IntelPTMultiCoreTrace.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/Process/Linux/IntelPTMultiCoreTrace.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements the LLDB support logic associated with `IntelPTMultiCoreTrace`.
  - **CN**: 实现与 `IntelPTMultiCoreTrace` 相关的 LLDB 支持逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- IntelPTMultiCoreTrace.cpp -----------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/LLDB file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/LLDB 标准文件头、许可证声明以及文件的高层说明。

### Lines 8-18
```cpp

#include "IntelPTMultiCoreTrace.h"
#include "Plugins/Process/POSIX/ProcessPOSIXLog.h"
#include "Procfs.h"
#include <optional>

using namespace lldb;
using namespace lldb_private;
using namespace process_linux;
using namespace llvm;

```
- **EN**: Pulls in the headers needed by this translation unit, including `IntelPTMultiCoreTrace.h`, `Plugins/Process/POSIX/ProcessPOSIXLog.h`, `Procfs.h`, `optional`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `IntelPTMultiCoreTrace.h`, `Plugins/Process/POSIX/ProcessPOSIXLog.h`, `Procfs.h`, `optional`。

### Lines 19-26
```cpp
static bool IsTotalBufferLimitReached(ArrayRef<cpu_id_t> cores,
                                      const TraceIntelPTStartRequest &request) {
  uint64_t required = cores.size() * request.ipt_trace_size;
  uint64_t limit = request.process_buffer_size_limit.value_or(
      std::numeric_limits<uint64_t>::max());
  return required > limit;
}

```
- **EN**: Implements logic around `IsTotalBufferLimitReached`, `size`, `value_or`, `max`.
- **CN**: 围绕 `IsTotalBufferLimitReached`, `size`, `value_or`, `max` 实现具体逻辑。

### Lines 27-35
```cpp
static Error IncludePerfEventParanoidMessageInError(Error &&error) {
  return createStringError(
      inconvertibleErrorCode(),
      "%s\nYou might need to rerun as sudo or to set "
      "/proc/sys/kernel/perf_event_paranoid to a value of 0 or -1. You can use "
      "`sudo sysctl -w kernel.perf_event_paranoid=-1` for that.",
      toString(std::move(error)).c_str());
}

```
- **EN**: Implements logic around `IncludePerfEventParanoidMessageInError`, `createStringError`, `inconvertibleErrorCode`, `toString`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `IncludePerfEventParanoidMessageInError`, `createStringError`, `inconvertibleErrorCode`, `toString` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 36-43
```cpp
Expected<std::unique_ptr<IntelPTMultiCoreTrace>>
IntelPTMultiCoreTrace::StartOnAllCores(const TraceIntelPTStartRequest &request,
                                       NativeProcessProtocol &process,
                                       std::optional<int> cgroup_fd) {
  Expected<ArrayRef<cpu_id_t>> cpu_ids = GetAvailableLogicalCoreIDs();
  if (!cpu_ids)
    return cpu_ids.takeError();

```
- **EN**: Implements logic around `StartOnAllCores`, `GetAvailableLogicalCoreIDs`, `takeError`; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `StartOnAllCores`, `GetAvailableLogicalCoreIDs`, `takeError` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 44-52
```cpp
  if (IsTotalBufferLimitReached(*cpu_ids, request))
    return createStringError(
        inconvertibleErrorCode(),
        "The process can't be traced because the process trace size limit "
        "has been reached. Consider retracing with a higher limit.");

  DenseMap<cpu_id_t, std::pair<IntelPTSingleBufferTrace, ContextSwitchTrace>>
      traces;

```
- **EN**: Implements logic around `IsTotalBufferLimitReached`, `createStringError`, `inconvertibleErrorCode`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `IsTotalBufferLimitReached`, `createStringError`, `inconvertibleErrorCode` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 53-59
```cpp
  for (cpu_id_t cpu_id : *cpu_ids) {
    Expected<IntelPTSingleBufferTrace> core_trace =
        IntelPTSingleBufferTrace::Start(request, /*tid=*/std::nullopt, cpu_id,
                                        /*disabled=*/true, cgroup_fd);
    if (!core_trace)
      return IncludePerfEventParanoidMessageInError(core_trace.takeError());

```
- **EN**: Implements logic around `Start`, `IncludePerfEventParanoidMessageInError`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `Start`, `IncludePerfEventParanoidMessageInError` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 60-70
```cpp
    if (Expected<PerfEvent> context_switch_trace =
            CreateContextSwitchTracePerfEvent(cpu_id,
                                              &core_trace->GetPerfEvent())) {
      traces.try_emplace(cpu_id,
                         std::make_pair(std::move(*core_trace),
                                        std::move(*context_switch_trace)));
    } else {
      return context_switch_trace.takeError();
    }
  }

```
- **EN**: Implements logic around `CreateContextSwitchTracePerfEvent`, `GetPerfEvent`, `try_emplace`, `make_pair`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `CreateContextSwitchTracePerfEvent`, `GetPerfEvent`, `try_emplace`, `make_pair`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 71-81
```cpp
  return std::unique_ptr<IntelPTMultiCoreTrace>(
      new IntelPTMultiCoreTrace(std::move(traces), process, (bool)cgroup_fd));
}

void IntelPTMultiCoreTrace::ForEachCore(
    std::function<void(cpu_id_t cpu_id, IntelPTSingleBufferTrace &core_trace)>
        callback) {
  for (auto &it : m_traces_per_core)
    callback(it.first, it.second.first);
}

```
- **EN**: Implements logic around `unique_ptr`, `IntelPTMultiCoreTrace`, `ForEachCore`, `function`, and 1 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `unique_ptr`, `IntelPTMultiCoreTrace`, `ForEachCore`, `function`, and 1 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 82-89
```cpp
void IntelPTMultiCoreTrace::ForEachCore(
    std::function<void(cpu_id_t cpu_id, IntelPTSingleBufferTrace &intelpt_trace,
                       ContextSwitchTrace &context_switch_trace)>
        callback) {
  for (auto &it : m_traces_per_core)
    callback(it.first, it.second.first, it.second.second);
}

```
- **EN**: Implements logic around `ForEachCore`, `function`, `callback`.
- **CN**: 围绕 `ForEachCore`, `function`, `callback` 实现具体逻辑。

### Lines 90-98
```cpp
void IntelPTMultiCoreTrace::ProcessDidStop() {
  ForEachCore([](cpu_id_t cpu_id, IntelPTSingleBufferTrace &core_trace) {
    if (Error err = core_trace.Pause()) {
      LLDB_LOG_ERROR(GetLog(POSIXLog::Trace), std::move(err),
                     "Unable to pause the core trace for core {0}", cpu_id);
    }
  });
}

```
- **EN**: Implements logic around `ProcessDidStop`, `ForEachCore`, `Pause`, `LLDB_LOG_ERROR`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `ProcessDidStop`, `ForEachCore`, `Pause`, `LLDB_LOG_ERROR` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 99-107
```cpp
void IntelPTMultiCoreTrace::ProcessWillResume() {
  ForEachCore([](cpu_id_t cpu_id, IntelPTSingleBufferTrace &core_trace) {
    if (Error err = core_trace.Resume()) {
      LLDB_LOG_ERROR(GetLog(POSIXLog::Trace), std::move(err),
                     "Unable to resume the core trace for core {0}", cpu_id);
    }
  });
}

```
- **EN**: Implements logic around `ProcessWillResume`, `ForEachCore`, `Resume`, `LLDB_LOG_ERROR`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `ProcessWillResume`, `ForEachCore`, `Resume`, `LLDB_LOG_ERROR` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 108-114
```cpp
TraceIntelPTGetStateResponse IntelPTMultiCoreTrace::GetState() {
  TraceIntelPTGetStateResponse state;
  state.using_cgroup_filtering = m_using_cgroup_filtering;

  for (NativeThreadProtocol &thread : m_process.Threads())
    state.traced_threads.push_back(TraceThreadState{thread.GetID(), {}});

```
- **EN**: Implements logic around `GetState`, `Threads`, `push_back`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `GetState`, `Threads`, `push_back` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 115-125
```cpp
  state.cpus.emplace();
  ForEachCore([&](lldb::cpu_id_t cpu_id,
                  const IntelPTSingleBufferTrace &core_trace,
                  const ContextSwitchTrace &context_switch_trace) {
    state.cpus->push_back(
        {cpu_id,
         {{IntelPTDataKinds::kIptTrace, core_trace.GetIptTraceSize()},
          {IntelPTDataKinds::kPerfContextSwitchTrace,
           context_switch_trace.GetEffectiveDataBufferSize()}}});
  });

```
- **EN**: Implements logic around `emplace`, `ForEachCore`, `push_back`, `GetIptTraceSize`, and 1 more symbols.
- **CN**: 围绕 `emplace`, `ForEachCore`, `push_back`, `GetIptTraceSize`, and 1 more symbols 实现具体逻辑。

### Lines 126-133
```cpp
  return state;
}

bool IntelPTMultiCoreTrace::TracesThread(lldb::tid_t tid) const {
  // All the process' threads are being traced automatically.
  return (bool)m_process.GetThreadByID(tid);
}

```
- **EN**: Implements logic around `TracesThread`, `GetThreadByID`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `TracesThread`, `GetThreadByID` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 134-142
```cpp
llvm::Error IntelPTMultiCoreTrace::TraceStart(lldb::tid_t tid) {
  // All the process' threads are being traced automatically.
  if (!TracesThread(tid))
    return createStringError(
        inconvertibleErrorCode(),
        "Thread %" PRIu64 " is not part of the target process", tid);
  return Error::success();
}

```
- **EN**: Implements logic around `TraceStart`, `TracesThread`, `createStringError`, `inconvertibleErrorCode`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `TraceStart`, `TracesThread`, `createStringError`, `inconvertibleErrorCode`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 143-156
```cpp
Error IntelPTMultiCoreTrace::TraceStop(lldb::tid_t tid) {
  return createStringError(inconvertibleErrorCode(),
                           "Can't stop tracing an individual thread when "
                           "per-cpu process tracing is enabled.");
}

Expected<std::optional<std::vector<uint8_t>>>
IntelPTMultiCoreTrace::TryGetBinaryData(
    const TraceGetBinaryDataRequest &request) {
  if (!request.cpu_id)
    return std::nullopt;
  auto it = m_traces_per_core.find(*request.cpu_id);
  if (it == m_traces_per_core.end())
    return createStringError(
```
- **EN**: Implements logic around `TraceStop`, `createStringError`, `TryGetBinaryData`, `find`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `TraceStop`, `createStringError`, `TryGetBinaryData`, `find`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 157-165
```cpp
        inconvertibleErrorCode(),
        formatv("Core {0} is not being traced", *request.cpu_id));

  if (request.kind == IntelPTDataKinds::kIptTrace)
    return it->second.first.GetIptTrace();
  if (request.kind == IntelPTDataKinds::kPerfContextSwitchTrace)
    return it->second.second.GetReadOnlyDataBuffer();
  return std::nullopt;
}
```
- **EN**: Implements logic around `inconvertibleErrorCode`, `formatv`, `GetIptTrace`, `GetReadOnlyDataBuffer`.
- **CN**: 围绕 `inconvertibleErrorCode`, `formatv`, `GetIptTrace`, `GetReadOnlyDataBuffer` 实现具体逻辑。

## Key Concepts / 关键概念

- **LLDB architecture / LLDB 架构**:
  - **EN**: Explains how LLDB organizes debugger APIs, runtime objects, and platform-specific helpers.
  - **CN**: 说明 LLDB 如何组织调试器 API、运行时对象以及平台相关辅助组件。
- **Runtime control / 运行时控制**:
  - **EN**: Models debugger ownership over targets, processes, threads, and event delivery.
  - **CN**: 建模调试器对目标、进程、线程以及事件投递的控制。
- **OS integration / 操作系统集成**:
  - **EN**: Bridges LLDB abstractions to host files, sockets, terminals, and native process facilities.
  - **CN**: 将 LLDB 抽象连接到主机文件、套接字、终端与本地进程设施。

## Dependencies / 依赖关系

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `IntelPTMultiCoreTrace.h`, `Plugins/Process/POSIX/ProcessPOSIXLog.h`, `Procfs.h`
- **Standard-library headers / 标准库头文件**: `<optional>`
