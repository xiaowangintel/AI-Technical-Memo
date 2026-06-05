# NativeProcessWindows.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/Process/Windows/Common/NativeProcessWindows.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements the LLDB support logic associated with `NativeProcessWindows`.
  - **CN**: 实现与 `NativeProcessWindows` 相关的 LLDB 支持逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- NativeProcessWindows.cpp ------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "lldb/Host/windows/windows.h"
#include <psapi.h>

```
- **EN**: Pulls in the headers needed by this translation unit, including `lldb/Host/windows/windows.h`, `psapi.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `lldb/Host/windows/windows.h`, `psapi.h`。

### Lines 12-31
```cpp
#include "NativeProcessWindows.h"
#include "NativeThreadWindows.h"
#include "lldb/Host/FileSystem.h"
#include "lldb/Host/HostNativeProcessBase.h"
#include "lldb/Host/HostProcess.h"
#include "lldb/Host/ProcessLaunchInfo.h"
#include "lldb/Host/PseudoTerminal.h"
#include "lldb/Host/windows/AutoHandle.h"
#include "lldb/Host/windows/HostThreadWindows.h"
#include "lldb/Host/windows/ProcessLauncherWindows.h"
#include "lldb/Target/MemoryRegionInfo.h"
#include "lldb/Target/Process.h"
#include "lldb/Utility/State.h"
#include "llvm/Support/ConvertUTF.h"
#include "llvm/Support/Errc.h"
#include "llvm/Support/Error.h"
#include "llvm/Support/Format.h"
#include "llvm/Support/Threading.h"
#include "llvm/Support/raw_ostream.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `NativeProcessWindows.h`, `NativeThreadWindows.h`, `lldb/Host/FileSystem.h`, `lldb/Host/HostNativeProcessBase.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `NativeProcessWindows.h`, `NativeThreadWindows.h`, `lldb/Host/FileSystem.h`, `lldb/Host/HostNativeProcessBase.h`。

### Lines 32-41
```cpp
#include "DebuggerThread.h"
#include "ExceptionRecord.h"
#include "ProcessWindowsLog.h"

#include <tlhelp32.h>

#pragma warning(disable : 4005)
#include "winternl.h"
#include <ntstatus.h>

```
- **EN**: Pulls in the headers needed by this translation unit, including `DebuggerThread.h`, `ExceptionRecord.h`, `ProcessWindowsLog.h`, `tlhelp32.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `DebuggerThread.h`, `ExceptionRecord.h`, `ProcessWindowsLog.h`, `tlhelp32.h`。

### Lines 42-61
```cpp
using namespace lldb;
using namespace lldb_private;
using namespace llvm;

namespace lldb_private {

NativeProcessWindows::NativeProcessWindows(ProcessLaunchInfo &launch_info,
                                           NativeDelegate &delegate,
                                           llvm::Error &E)
    : NativeProcessProtocol(
          LLDB_INVALID_PROCESS_ID,
          PseudoTerminal::invalid_fd, // TODO: Implement on Windows
          delegate),
      ProcessDebugger(), m_arch(launch_info.GetArchitecture()) {
  ErrorAsOutParameter EOut(&E);
  DebugDelegateSP delegate_sp(new NativeDebugDelegate(*this));
  E = LaunchProcess(launch_info, delegate_sp).ToError();
  if (E)
    return;

```
- **EN**: Introduces declarations for `lldb_private`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `lldb_private` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 62-76
```cpp
  SetID(GetDebuggedProcessId());
}

NativeProcessWindows::NativeProcessWindows(lldb::pid_t pid, int terminal_fd,
                                           NativeDelegate &delegate,
                                           llvm::Error &E)
    : NativeProcessProtocol(pid, terminal_fd, delegate), ProcessDebugger() {
  ErrorAsOutParameter EOut(&E);
  DebugDelegateSP delegate_sp(new NativeDebugDelegate(*this));
  ProcessAttachInfo attach_info;
  attach_info.SetProcessID(pid);
  E = AttachProcess(pid, attach_info, delegate_sp).ToError();
  if (E)
    return;

```
- **EN**: Implements logic around `SetID`, `NativeProcessWindows`, `NativeProcessProtocol`, `EOut`, and 3 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `SetID`, `NativeProcessWindows`, `NativeProcessProtocol`, `EOut`, and 3 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 77-87
```cpp
  SetID(GetDebuggedProcessId());

  ProcessInstanceInfo info;
  if (!Host::GetProcessInfo(pid, info)) {
    E = createStringError(inconvertibleErrorCode(),
                          "Cannot get process information");
    return;
  }
  m_arch = info.GetArchitecture();
}

```
- **EN**: Implements logic around `SetID`, `GetProcessInfo`, `createStringError`, `GetArchitecture`; this block coordinates debugger runtime objects, events, or asynchronous control flow; abstracts host OS resources such as files, terminals, sockets, or platforms.
- **CN**: 围绕 `SetID`, `GetProcessInfo`, `createStringError`, `GetArchitecture` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并抽象文件、终端、套接字或平台等主机操作系统资源。

### Lines 88-98
```cpp
Status NativeProcessWindows::Resume(const ResumeActionList &resume_actions) {
  Log *log = GetLog(WindowsLog::Process);
  Status error;
  llvm::sys::ScopedLock lock(m_mutex);

  StateType state = GetState();
  if (state == eStateStopped || state == eStateCrashed) {
    LLDB_LOG(log, "process {0} is in state {1}.  Resuming...",
             GetDebuggedProcessId(), state);
    LLDB_LOG(log, "resuming {0} threads.", m_threads.size());

```
- **EN**: Implements logic around `Resume`, `GetLog`, `lock`, `GetState`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `Resume`, `GetLog`, `lock`, `GetState`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 99-118
```cpp
    bool failed = false;
    for (uint32_t i = 0; i < m_threads.size(); ++i) {
      auto thread = static_cast<NativeThreadWindows *>(m_threads[i].get());
      const ResumeAction *const action =
          resume_actions.GetActionForThread(thread->GetID(), true);
      if (action == nullptr)
        continue;

      switch (action->state) {
      case eStateRunning:
      case eStateStepping: {
        Status result = thread->DoResume(action->state);
        if (result.Fail()) {
          failed = true;
          LLDB_LOG(log,
                   "Trying to resume thread at index {0}, but failed with "
                   "error {1}.",
                   i, result);
        }
        break;
```
- **EN**: Implements logic around `size`, `get`, `GetActionForThread`, `DoResume`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `size`, `get`, `GetActionForThread`, `DoResume`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 119-132
```cpp
      }
      case eStateSuspended:
      case eStateStopped:
        break;

      default:
        return Status::FromErrorStringWithFormat(
            "NativeProcessWindows::%s (): unexpected state %s specified "
            "for pid %" PRIu64 ", tid %" PRIu64,
            __FUNCTION__, StateAsCString(action->state), GetID(),
            thread->GetID());
      }
    }

```
- **EN**: Implements logic around `FromErrorStringWithFormat`, `s`, `StateAsCString`, `GetID`; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `FromErrorStringWithFormat`, `s`, `StateAsCString`, `GetID` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 133-152
```cpp
    if (failed) {
      error = Status::FromErrorString("NativeProcessWindows::DoResume failed");
    } else {
      SetState(eStateRunning);
    }

    // Resume the debug loop.
    ExceptionRecordSP active_exception =
        m_session_data->m_debugger->GetActiveException().lock();
    if (active_exception) {
      // Resume the process and continue processing debug events.  Mask the
      // exception so that from the process's view, there is no indication that
      // anything happened.
      m_session_data->m_debugger->ContinueAsyncException(
          ExceptionResult::MaskException);
    }
  } else {
    LLDB_LOG(log, "error: process {0} is in state {1}.  Returning...",
             GetDebuggedProcessId(), GetState());
  }
```
- **EN**: Implements logic around `FromErrorString`, `SetState`, `GetActiveException`, `ContinueAsyncException`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `FromErrorString`, `SetState`, `GetActiveException`, `ContinueAsyncException`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 153-162
```cpp

  return error;
}

NativeThreadWindows *
NativeProcessWindows::GetThreadByID(lldb::tid_t thread_id) {
  return static_cast<NativeThreadWindows *>(
      NativeProcessProtocol::GetThreadByID(thread_id));
}

```
- **EN**: Implements logic around `GetThreadByID`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `GetThreadByID` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 163-182
```cpp
Status NativeProcessWindows::Halt() {
  bool caused_stop = false;
  StateType state = GetState();
  if (state != eStateStopped)
    return HaltProcess(caused_stop);
  return Status();
}

Status NativeProcessWindows::Detach() {
  Status error;
  Log *log = GetLog(WindowsLog::Process);
  StateType state = GetState();
  if (state != eStateExited && state != eStateDetached) {
    error = DetachProcess();
    if (error.Success())
      SetState(eStateDetached);
    else
      LLDB_LOG(log, "Detaching process error: {0}", error);
  } else {
    error = Status::FromErrorStringWithFormatv(
```
- **EN**: Implements logic around `Halt`, `GetState`, `HaltProcess`, `Status`, and 7 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `Halt`, `GetState`, `HaltProcess`, `Status`, and 7 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 183-197
```cpp
        "error: process {0} in state = {1}, but "
        "cannot detach it in this state.",
        GetID(), state);
    LLDB_LOG(log, "error: {0}", error);
  }
  return error;
}

Status NativeProcessWindows::Signal(int signo) {
  Status error;
  error = Status::FromErrorString(
      "Windows does not support sending signals to processes");
  return error;
}

```
- **EN**: Implements logic around `GetID`, `LLDB_LOG`, `Signal`, `FromErrorString`; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `GetID`, `LLDB_LOG`, `Signal`, `FromErrorString` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 198-208
```cpp
Status NativeProcessWindows::Interrupt() { return Halt(); }

Status NativeProcessWindows::Kill() {
  StateType state = GetState();
  return DestroyProcess(state);
}

Status NativeProcessWindows::IgnoreSignals(llvm::ArrayRef<int> signals) {
  return Status();
}

```
- **EN**: Implements logic around `Interrupt`, `Kill`, `GetState`, `DestroyProcess`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `Interrupt`, `Kill`, `GetState`, `DestroyProcess`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 209-218
```cpp
Status NativeProcessWindows::GetMemoryRegionInfo(lldb::addr_t load_addr,
                                                 MemoryRegionInfo &range_info) {
  return ProcessDebugger::GetMemoryRegionInfo(load_addr, range_info);
}

Status NativeProcessWindows::ReadMemory(lldb::addr_t addr, void *buf,
                                        size_t size, size_t &bytes_read) {
  return ProcessDebugger::ReadMemory(addr, buf, size, bytes_read);
}

```
- **EN**: Implements logic around `GetMemoryRegionInfo`, `ReadMemory`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `GetMemoryRegionInfo`, `ReadMemory` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 219-232
```cpp
Status NativeProcessWindows::WriteMemory(lldb::addr_t addr, const void *buf,
                                         size_t size, size_t &bytes_written) {
  return ProcessDebugger::WriteMemory(addr, buf, size, bytes_written);
}

llvm::Expected<lldb::addr_t>
NativeProcessWindows::AllocateMemory(size_t size, uint32_t permissions) {
  lldb::addr_t addr;
  Status ST = ProcessDebugger::AllocateMemory(size, permissions, addr);
  if (ST.Success())
    return addr;
  return ST.ToError();
}

```
- **EN**: Implements logic around `WriteMemory`, `AllocateMemory`, `Success`, `ToError`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `WriteMemory`, `AllocateMemory`, `Success`, `ToError` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 233-252
```cpp
llvm::Error NativeProcessWindows::DeallocateMemory(lldb::addr_t addr) {
  return ProcessDebugger::DeallocateMemory(addr).ToError();
}

lldb::addr_t NativeProcessWindows::GetSharedLibraryInfoAddress() { return 0; }

bool NativeProcessWindows::IsAlive() const {
  StateType state = GetState();
  switch (state) {
  case eStateCrashed:
  case eStateDetached:
  case eStateExited:
  case eStateInvalid:
  case eStateUnloaded:
    return false;
  default:
    return true;
  }
}

```
- **EN**: Implements logic around `DeallocateMemory`, `GetSharedLibraryInfoAddress`, `IsAlive`, `GetState`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `DeallocateMemory`, `GetSharedLibraryInfoAddress`, `IsAlive`, `GetState` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 253-262
```cpp
void NativeProcessWindows::SetStopReasonForThread(NativeThreadWindows &thread,
                                                  lldb::StopReason reason,
                                                  std::string description) {
  SetCurrentThreadID(thread.GetID());

  ThreadStopInfo stop_info;
  stop_info.reason = reason;
  // No signal support on Windows but required to provide a 'valid' signum.
  stop_info.signo = SIGTRAP;

```
- **EN**: Implements logic around `SetStopReasonForThread`, `SetCurrentThreadID`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `SetStopReasonForThread`, `SetCurrentThreadID` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 263-277
```cpp
  if (reason == StopReason::eStopReasonException) {
    stop_info.details.exception.type = 0;
    stop_info.details.exception.data_count = 0;
  }

  thread.SetStopReason(stop_info, description);
}

void NativeProcessWindows::StopThread(lldb::tid_t thread_id,
                                      lldb::StopReason reason,
                                      std::string description) {
  NativeThreadWindows *thread = GetThreadByID(thread_id);
  if (!thread)
    return;

```
- **EN**: Implements logic around `SetStopReason`, `StopThread`, `GetThreadByID`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `SetStopReason`, `StopThread`, `GetThreadByID` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 278-288
```cpp
  for (uint32_t i = 0; i < m_threads.size(); ++i) {
    auto t = static_cast<NativeThreadWindows *>(m_threads[i].get());
    Status error = t->DoStop();
    if (error.Fail())
      exit(1);
  }
  SetStopReasonForThread(*thread, reason, description);
}

size_t NativeProcessWindows::UpdateThreads() { return m_threads.size(); }

```
- **EN**: Implements logic around `size`, `get`, `DoStop`, `Fail`, and 3 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `size`, `get`, `DoStop`, `Fail`, and 3 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 289-300
```cpp
llvm::ErrorOr<std::unique_ptr<llvm::MemoryBuffer>>
NativeProcessWindows::GetAuxvData() const {
  // Not available on this target.
  return llvm::errc::not_supported;
}

llvm::Expected<llvm::ArrayRef<uint8_t>>
NativeProcessWindows::GetSoftwareBreakpointTrapOpcode(size_t size_hint) {
  static const uint8_t g_aarch64_opcode[] = {0x00, 0x00, 0x3e,
                                             0xd4};     // brk #0xf000
  static const uint8_t g_thumb_opcode[] = {0xfe, 0xde}; // udf #0xfe

```
- **EN**: Implements logic around `GetAuxvData`, `GetSoftwareBreakpointTrapOpcode`; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `GetAuxvData`, `GetSoftwareBreakpointTrapOpcode` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 301-313
```cpp
  switch (GetArchitecture().GetMachine()) {
  case llvm::Triple::aarch64:
    return llvm::ArrayRef(g_aarch64_opcode);

  case llvm::Triple::arm:
  case llvm::Triple::thumb:
    return llvm::ArrayRef(g_thumb_opcode);

  default:
    return NativeProcessProtocol::GetSoftwareBreakpointTrapOpcode(size_hint);
  }
}

```
- **EN**: Implements logic around `GetArchitecture`, `ArrayRef`, `GetSoftwareBreakpointTrapOpcode`.
- **CN**: 围绕 `GetArchitecture`, `ArrayRef`, `GetSoftwareBreakpointTrapOpcode` 实现具体逻辑。

### Lines 314-326
```cpp
size_t NativeProcessWindows::GetSoftwareBreakpointPCOffset() {
  // Windows always reports an incremented PC after a breakpoint is hit,
  // even on ARM.
  return cantFail(GetSoftwareBreakpointTrapOpcode(0)).size();
}

bool NativeProcessWindows::FindSoftwareBreakpoint(lldb::addr_t addr) {
  auto it = m_software_breakpoints.find(addr);
  if (it == m_software_breakpoints.end())
    return false;
  return true;
}

```
- **EN**: Implements logic around `GetSoftwareBreakpointPCOffset`, `cantFail`, `FindSoftwareBreakpoint`, `find`, and 1 more symbols; this block tracks breakpoint state, stop conditions, or hit-processing policy.
- **CN**: 围绕 `GetSoftwareBreakpointPCOffset`, `cantFail`, `FindSoftwareBreakpoint`, `find`, and 1 more symbols 实现具体逻辑；该代码块跟踪断点状态、停机条件或命中处理策略。

### Lines 327-340
```cpp
Status NativeProcessWindows::SetBreakpoint(lldb::addr_t addr, uint32_t size,
                                           bool hardware) {
  if (hardware)
    return SetHardwareBreakpoint(addr, size);
  return SetSoftwareBreakpoint(addr, size);
}

Status NativeProcessWindows::RemoveBreakpoint(lldb::addr_t addr,
                                              bool hardware) {
  if (hardware)
    return RemoveHardwareBreakpoint(addr);
  return RemoveSoftwareBreakpoint(addr);
}

```
- **EN**: Implements logic around `SetBreakpoint`, `SetHardwareBreakpoint`, `SetSoftwareBreakpoint`, `RemoveBreakpoint`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `SetBreakpoint`, `SetHardwareBreakpoint`, `SetSoftwareBreakpoint`, `RemoveBreakpoint`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 341-356
```cpp
Status NativeProcessWindows::CacheLoadedModules() {
  Status error;
  if (!m_loaded_modules.empty())
    return Status();

  // Retrieve loaded modules by a Target/Module free implemenation.
  AutoHandle snapshot(CreateToolhelp32Snapshot(TH32CS_SNAPMODULE, GetID()));
  if (snapshot.IsValid()) {
    MODULEENTRY32W me;
    me.dwSize = sizeof(MODULEENTRY32W);
    if (Module32FirstW(snapshot.get(), &me)) {
      do {
        std::string path;
        if (!llvm::convertWideToUTF8(me.szExePath, path))
          continue;

```
- **EN**: Implements logic around `CacheLoadedModules`, `empty`, `Status`, `snapshot`, and 3 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow; maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `CacheLoadedModules`, `empty`, `Status`, `snapshot`, and 3 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流，并将可执行状态映射回模块、符号、节区或地址。

### Lines 357-366
```cpp
        FileSpec file_spec(path);
        FileSystem::Instance().Resolve(file_spec);
        m_loaded_modules[file_spec] = (addr_t)me.modBaseAddr;
      } while (Module32Next(snapshot.get(), &me));
    }

    if (!m_loaded_modules.empty())
      return Status();
  }

```
- **EN**: Implements logic around `file_spec`, `Instance`, `Module32Next`, `empty`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `file_spec`, `Instance`, `Module32Next`, `empty`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 367-376
```cpp
  error = Status(::GetLastError(), lldb::ErrorType::eErrorTypeWin32);
  return error;
}

Status NativeProcessWindows::GetLoadedModuleFileSpec(const char *module_path,
                                                     FileSpec &file_spec) {
  Status error = CacheLoadedModules();
  if (error.Fail())
    return error;

```
- **EN**: Implements logic around `Status`, `GetLoadedModuleFileSpec`, `CacheLoadedModules`, `Fail`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `Status`, `GetLoadedModuleFileSpec`, `CacheLoadedModules`, `Fail` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 377-389
```cpp
  FileSpec module_file_spec(module_path);
  FileSystem::Instance().Resolve(module_file_spec);
  for (auto &it : m_loaded_modules) {
    if (it.first == module_file_spec) {
      file_spec = it.first;
      return Status();
    }
  }
  return Status::FromErrorStringWithFormat(
      "Module (%s) not found in process %" PRIu64 "!",
      module_file_spec.GetPath().c_str(), GetID());
}

```
- **EN**: Implements logic around `module_file_spec`, `Instance`, `Status`, `FromErrorStringWithFormat`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow; maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `module_file_spec`, `Instance`, `Status`, `FromErrorStringWithFormat`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流，并将可执行状态映射回模块、符号、节区或地址。

### Lines 390-409
```cpp
Status
NativeProcessWindows::GetFileLoadAddress(const llvm::StringRef &file_name,
                                         lldb::addr_t &load_addr) {
  Status error = CacheLoadedModules();
  if (error.Fail())
    return error;

  load_addr = LLDB_INVALID_ADDRESS;
  FileSpec file_spec(file_name);
  FileSystem::Instance().Resolve(file_spec);
  for (auto &it : m_loaded_modules) {
    if (it.first == file_spec) {
      load_addr = it.second;
      return Status();
    }
  }
  return Status::FromErrorStringWithFormat(
      "Can't get loaded address of file (%s) in process %" PRIu64 "!",
      file_spec.GetPath().c_str(), GetID());
}
```
- **EN**: Implements logic around `GetFileLoadAddress`, `CacheLoadedModules`, `Fail`, `file_spec`, and 5 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow; maps executable state back to modules, symbols, sections, or addresses; abstracts host OS resources such as files, terminals, sockets, or platforms.
- **CN**: 围绕 `GetFileLoadAddress`, `CacheLoadedModules`, `Fail`, `file_spec`, and 5 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流，并将可执行状态映射回模块、符号、节区或地址，并抽象文件、终端、套接字或平台等主机操作系统资源。

### Lines 410-420
```cpp

void NativeProcessWindows::OnExitProcess(uint32_t exit_code) {
  Log *log = GetLog(WindowsLog::Process);
  LLDB_LOG(log, "Process {0} exited with code {1}", GetID(), exit_code);

  ProcessDebugger::OnExitProcess(exit_code);

  // No signal involved.  It is just an exit event.
  WaitStatus wait_status(WaitStatus::Exit, exit_code);
  SetExitStatus(wait_status, true);

```
- **EN**: Implements logic around `OnExitProcess`, `GetLog`, `LLDB_LOG`, `wait_status`, and 1 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `OnExitProcess`, `GetLog`, `LLDB_LOG`, `wait_status`, and 1 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 421-434
```cpp
  // Notify the native delegate.
  SetState(eStateExited, true);
}

void NativeProcessWindows::OnDebuggerConnected(lldb::addr_t image_base) {
  Log *log = GetLog(WindowsLog::Process);
  LLDB_LOG(log, "Debugger connected to process {0}. Image base = {1:x}",
           GetDebuggedProcessId(), image_base);

  // This is the earliest chance we can resolve the process ID and
  // architecture if we don't know them yet.
  if (GetID() == LLDB_INVALID_PROCESS_ID)
    SetID(GetDebuggedProcessId());

```
- **EN**: Implements logic around `SetState`, `OnDebuggerConnected`, `GetLog`, `LLDB_LOG`, and 3 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `SetState`, `OnDebuggerConnected`, `GetLog`, `LLDB_LOG`, and 3 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 435-444
```cpp
  if (GetArchitecture().GetMachine() == llvm::Triple::UnknownArch) {
    ProcessInstanceInfo process_info;
    if (!Host::GetProcessInfo(GetDebuggedProcessId(), process_info)) {
      LLDB_LOG(log, "Cannot get process information during debugger connecting "
                    "to process");
      return;
    }
    SetArchitecture(process_info.GetArchitecture());
  }

```
- **EN**: Implements logic around `GetArchitecture`, `GetProcessInfo`, `LLDB_LOG`, `SetArchitecture`; this block coordinates debugger runtime objects, events, or asynchronous control flow; abstracts host OS resources such as files, terminals, sockets, or platforms.
- **CN**: 围绕 `GetArchitecture`, `GetProcessInfo`, `LLDB_LOG`, `SetArchitecture` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并抽象文件、终端、套接字或平台等主机操作系统资源。

### Lines 445-456
```cpp
  // The very first one shall always be the main thread.
  assert(m_threads.empty());
  m_threads.push_back(std::make_unique<NativeThreadWindows>(
      *this, m_session_data->m_debugger->GetMainThread()));
}

ExceptionResult
NativeProcessWindows::OnDebugException(bool first_chance,
                                       const ExceptionRecord &record) {
  Log *log = GetLog(WindowsLog::Exception);
  llvm::sys::ScopedLock lock(m_mutex);

```
- **EN**: Implements logic around `assert`, `push_back`, `OnDebugException`, `GetLog`, and 1 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `assert`, `push_back`, `OnDebugException`, `GetLog`, and 1 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 457-476
```cpp
  // Let the debugger establish the internal status.
  ProcessDebugger::OnDebugException(first_chance, record);

  static bool initial_stop = false;
  if (!first_chance) {
    SetState(eStateStopped, false);
  }

  switch (record.GetExceptionCode()) {
  case DWORD(STATUS_SINGLE_STEP):
  case STATUS_WX86_SINGLE_STEP: {
#ifndef __aarch64__
    uint32_t wp_id = LLDB_INVALID_INDEX32;
    if (NativeThreadWindows *thread = GetThreadByID(record.GetThreadID())) {
      NativeRegisterContextWindows &reg_ctx = thread->GetRegisterContext();
      Status error =
          reg_ctx.GetWatchpointHitIndex(wp_id, record.GetExceptionAddress());
      if (error.Fail())
        LLDB_LOG(log,
                 "received error while checking for watchpoint hits, pid = "
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or include-guard state.
- **CN**: 定义受预处理器控制的结构、特性开关或头文件保护状态。

### Lines 477-491
```cpp
                 "{0}, error = {1}",
                 thread->GetID(), error);
      if (wp_id != LLDB_INVALID_INDEX32) {
        addr_t wp_addr = reg_ctx.GetWatchpointAddress(wp_id);
        addr_t wp_hit_addr = reg_ctx.GetWatchpointHitAddress(wp_id);
        std::string desc =
            formatv("{0} {1} {2}", wp_addr, wp_id, wp_hit_addr).str();
        StopThread(record.GetThreadID(), StopReason::eStopReasonWatchpoint,
                   desc);
      }
    }
    if (wp_id == LLDB_INVALID_INDEX32)
#endif
      StopThread(record.GetThreadID(), StopReason::eStopReasonTrace);

```
- **EN**: Defines preprocessor-controlled structure, feature gates, or include-guard state.
- **CN**: 定义受预处理器控制的结构、特性开关或头文件保护状态。

### Lines 492-504
```cpp
    SetState(eStateStopped, true);

    // Continue the debugger.
    return ExceptionResult::MaskException;
  }
  case DWORD(STATUS_BREAKPOINT):
  case STATUS_WX86_BREAKPOINT: {
    if (NativeThreadWindows *stop_thread =
            GetThreadByID(record.GetThreadID())) {
      auto &reg_ctx = stop_thread->GetRegisterContext();
      const auto exception_addr = record.GetExceptionAddress();
      const auto thread_id = record.GetThreadID();

```
- **EN**: Implements logic around `SetState`, `DWORD`, `GetThreadByID`, `GetRegisterContext`, and 2 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `SetState`, `DWORD`, `GetThreadByID`, `GetRegisterContext`, and 2 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 505-524
```cpp
      if (FindSoftwareBreakpoint(exception_addr)) {
        LLDB_LOG(log, "Hit non-loader breakpoint at address {0:x}.",
                 exception_addr);
        StopThread(thread_id, StopReason::eStopReasonBreakpoint);
        // The current PC is AFTER the BP opcode, on all architectures.
        reg_ctx.SetPC(reg_ctx.GetPC() - GetSoftwareBreakpointPCOffset());
        SetState(eStateStopped, true);
        return ExceptionResult::MaskException;
      } else {
        // This block of code will only be entered in case of a hardware
        // watchpoint or breakpoint hit on AArch64. However, we only handle
        // hardware watchpoints below as breakpoints are not yet supported.
        const std::vector<ULONG_PTR> &args = record.GetExceptionArguments();
        // Check that the ExceptionInformation array of EXCEPTION_RECORD
        // contains at least two elements: the first is a read-write flag
        // indicating the type of data access operation (read or write) while
        // the second contains the virtual address of the accessed data.
        if (args.size() >= 2) {
          uint32_t hw_id = LLDB_INVALID_INDEX32;
          Status error = reg_ctx.GetWatchpointHitIndex(hw_id, args[1]);
```
- **EN**: Implements logic around `FindSoftwareBreakpoint`, `LLDB_LOG`, `StopThread`, `SetPC`, and 4 more symbols; this block propagates recoverable errors, status objects, or diagnostics; tracks breakpoint state, stop conditions, or hit-processing policy; maps executable state back to modules, symbols, sections, or addresses; defines user-visible settings, options, or policy flags.
- **CN**: 围绕 `FindSoftwareBreakpoint`, `LLDB_LOG`, `StopThread`, `SetPC`, and 4 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并跟踪断点状态、停机条件或命中处理策略，并将可执行状态映射回模块、符号、节区或地址，并定义用户可见的设置、选项或策略标志。

### Lines 525-543
```cpp
          if (error.Fail())
            LLDB_LOG(log,
                     "received error while checking for watchpoint hits, pid = "
                     "{0}, error = {1}",
                     thread_id, error);

          if (hw_id != LLDB_INVALID_INDEX32) {
            std::string desc =
                formatv("{0} {1} {2}", reg_ctx.GetWatchpointAddress(hw_id),
                        hw_id, exception_addr)
                    .str();
            StopThread(thread_id, StopReason::eStopReasonWatchpoint, desc);
            SetState(eStateStopped, true);
            return ExceptionResult::MaskException;
          }
        }
      }
    }

```
- **EN**: Implements logic around `Fail`, `LLDB_LOG`, `formatv`, `str`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics; tracks breakpoint state, stop conditions, or hit-processing policy.
- **CN**: 围绕 `Fail`, `LLDB_LOG`, `formatv`, `str`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并跟踪断点状态、停机条件或命中处理策略。

### Lines 544-555
```cpp
    if (!initial_stop) {
      initial_stop = true;
      LLDB_LOG(log,
               "Hit loader breakpoint at address {0:x}, setting initial stop "
               "event.",
               record.GetExceptionAddress());

      // We are required to report the reason for the first stop after
      // launching or being attached.
      if (NativeThreadWindows *thread = GetThreadByID(record.GetThreadID()))
        SetStopReasonForThread(*thread, StopReason::eStopReasonBreakpoint);

```
- **EN**: Implements logic around `LLDB_LOG`, `GetExceptionAddress`, `GetThreadByID`, `SetStopReasonForThread`; this block tracks breakpoint state, stop conditions, or hit-processing policy; coordinates debugger runtime objects, events, or asynchronous control flow; maps executable state back to modules, symbols, sections, or addresses; defines user-visible settings, options, or policy flags.
- **CN**: 围绕 `LLDB_LOG`, `GetExceptionAddress`, `GetThreadByID`, `SetStopReasonForThread` 实现具体逻辑；该代码块跟踪断点状态、停机条件或命中处理策略，并协调调试器运行时对象、事件或异步控制流，并将可执行状态映射回模块、符号、节区或地址，并定义用户可见的设置、选项或策略标志。

### Lines 556-575
```cpp
      // Do not notify the native delegate (e.g. llgs) since at this moment
      // the program hasn't returned from Manager::Launch() and the delegate
      // might not have an valid native process to operate on.
      SetState(eStateStopped, false);

      // Hit the initial stop. Continue the application.
      return ExceptionResult::BreakInDebugger;
    }

    // Any remaining STATUS_BREAKPOINT is a breakpoint instruction in the
    // program's own code (e.g. `__debugbreak()` or `__builtin_debugtrap()`).
    // Stop the debugger and let the user decide what to do.
    std::string desc =
        formatv("Exception {0:x8} encountered at address {1:x8}",
                record.GetExceptionCode(), record.GetExceptionAddress())
            .str();
    StopThread(record.GetThreadID(), StopReason::eStopReasonException,
               std::move(desc));
    SetState(eStateStopped, true);

```
- **EN**: Implements logic around `SetState`, `formatv`, `GetExceptionCode`, `str`, and 2 more symbols; this block tracks breakpoint state, stop conditions, or hit-processing policy; coordinates debugger runtime objects, events, or asynchronous control flow; maps executable state back to modules, symbols, sections, or addresses; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `SetState`, `formatv`, `GetExceptionCode`, `str`, and 2 more symbols 实现具体逻辑；该代码块跟踪断点状态、停机条件或命中处理策略，并协调调试器运行时对象、事件或异步控制流，并将可执行状态映射回模块、符号、节区或地址，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 576-593
```cpp
    return ExceptionResult::MaskException;
  }
  default: {
    LLDB_LOG(log,
             "Debugger thread reported exception {0:x} at address {1:x} "
             "(first_chance={2})",
             record.GetExceptionCode(), record.GetExceptionAddress(),
             first_chance);

    std::string desc;
    llvm::raw_string_ostream desc_stream(desc);
    desc_stream << "Exception "
                << llvm::format_hex(record.GetExceptionCode(), 8)
                << " encountered at address "
                << llvm::format_hex(record.GetExceptionAddress(), 8);
    StopThread(record.GetThreadID(), StopReason::eStopReasonException,
               desc.c_str());

```
- **EN**: Implements logic around `LLDB_LOG`, `GetExceptionCode`, `desc_stream`, `format_hex`, and 2 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow; maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `LLDB_LOG`, `GetExceptionCode`, `desc_stream`, `format_hex`, and 2 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并将可执行状态映射回模块、符号、节区或地址。

### Lines 594-605
```cpp
    SetState(eStateStopped, true);

    // For non-breakpoints, give the application a chance to handle the
    // exception first.
    if (first_chance)
      return ExceptionResult::SendToApplication;
    else
      return ExceptionResult::BreakInDebugger;
  }
  }
}

```
- **EN**: Implements logic around `SetState`.
- **CN**: 围绕 `SetState` 实现具体逻辑。

### Lines 606-616
```cpp
void NativeProcessWindows::OnCreateThread(const HostThread &new_thread) {
  llvm::sys::ScopedLock lock(m_mutex);

  auto thread = std::make_unique<NativeThreadWindows>(*this, new_thread);
  thread->GetRegisterContext().ClearAllHardwareWatchpoints();
  for (const auto &pair : GetWatchpointMap()) {
    const NativeWatchpoint &wp = pair.second;
    thread->SetWatchpoint(wp.m_addr, wp.m_size, wp.m_watch_flags,
                          wp.m_hardware);
  }

```
- **EN**: Implements logic around `OnCreateThread`, `lock`, `make_unique`, `GetRegisterContext`, and 2 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `OnCreateThread`, `lock`, `make_unique`, `GetRegisterContext`, and 2 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 617-626
```cpp
  m_threads.push_back(std::move(thread));
}

void NativeProcessWindows::OnExitThread(lldb::tid_t thread_id,
                                        uint32_t exit_code) {
  llvm::sys::ScopedLock lock(m_mutex);
  NativeThreadWindows *thread = GetThreadByID(thread_id);
  if (!thread)
    return;

```
- **EN**: Implements logic around `push_back`, `OnExitThread`, `lock`, `GetThreadByID`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `push_back`, `OnExitThread`, `lock`, `GetThreadByID` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 627-642
```cpp
  for (auto t = m_threads.begin(); t != m_threads.end();) {
    if ((*t)->GetID() == thread_id) {
      t = m_threads.erase(t);
    } else {
      ++t;
    }
  }
}

void NativeProcessWindows::OnLoadDll(const ModuleSpec &module_spec,
                                     lldb::addr_t module_addr) {
  // Simply invalidate the cached loaded modules.
  if (!m_loaded_modules.empty())
    m_loaded_modules.clear();
}

```
- **EN**: Implements logic around `begin`, `GetID`, `erase`, `OnLoadDll`, and 2 more symbols.
- **CN**: 围绕 `begin`, `GetID`, `erase`, `OnLoadDll`, and 2 more symbols 实现具体逻辑。

### Lines 643-659
```cpp
void NativeProcessWindows::OnUnloadDll(lldb::addr_t module_addr) {
  if (!m_loaded_modules.empty())
    m_loaded_modules.clear();
}

llvm::Expected<std::unique_ptr<NativeProcessProtocol>>
NativeProcessWindows::Manager::Launch(
    ProcessLaunchInfo &launch_info,
    NativeProcessProtocol::NativeDelegate &native_delegate) {
  Error E = Error::success();
  auto process_up = std::unique_ptr<NativeProcessWindows>(
      new NativeProcessWindows(launch_info, native_delegate, E));
  if (E)
    return std::move(E);
  return std::move(process_up);
}

```
- **EN**: Implements logic around `OnUnloadDll`, `empty`, `clear`, `Launch`, and 4 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `OnUnloadDll`, `empty`, `clear`, `Launch`, and 4 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 660-671
```cpp
llvm::Expected<std::unique_ptr<NativeProcessProtocol>>
NativeProcessWindows::Manager::Attach(
    lldb::pid_t pid, NativeProcessProtocol::NativeDelegate &native_delegate) {
  Error E = Error::success();
  // Set pty primary fd invalid since it is not available.
  auto process_up = std::unique_ptr<NativeProcessWindows>(
      new NativeProcessWindows(pid, -1, native_delegate, E));
  if (E)
    return std::move(E);
  return std::move(process_up);
}
} // namespace lldb_private
```
- **EN**: Implements logic around `Attach`, `success`, `unique_ptr`, `NativeProcessWindows`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics; abstracts host OS resources such as files, terminals, sockets, or platforms.
- **CN**: 围绕 `Attach`, `success`, `unique_ptr`, `NativeProcessWindows`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并抽象文件、终端、套接字或平台等主机操作系统资源。

## Key Concepts / 关键概念

- **LLDB architecture / LLDB 架构**:
  - **EN**: Explains how LLDB organizes debugger APIs, runtime objects, and platform-specific helpers.
  - **CN**: 说明 LLDB 如何组织调试器 API、运行时对象以及平台相关辅助组件。
- **Stop control / 停机控制**:
  - **EN**: Represents stop reasons, conditions, callbacks, and breakpoint resolution state.
  - **CN**: 表示停机原因、条件、回调以及断点解析状态。
- **Symbolication / 符号化**:
  - **EN**: Relates raw addresses back to modules, sections, symbols, and source locations.
  - **CN**: 将原始地址关联回模块、节区、符号与源码位置。
- **Runtime control / 运行时控制**:
  - **EN**: Models debugger ownership over targets, processes, threads, and event delivery.
  - **CN**: 建模调试器对目标、进程、线程以及事件投递的控制。
- **OS integration / 操作系统集成**:
  - **EN**: Bridges LLDB abstractions to host files, sockets, terminals, and native process facilities.
  - **CN**: 将 LLDB 抽象连接到主机文件、套接字、终端与本地进程设施。

## Dependencies / 依赖关系

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `lldb/Host/windows/windows.h`, `NativeProcessWindows.h`, `NativeThreadWindows.h`, `lldb/Host/FileSystem.h`, `lldb/Host/HostNativeProcessBase.h`, `lldb/Host/HostProcess.h`, `lldb/Host/ProcessLaunchInfo.h`, `lldb/Host/PseudoTerminal.h`, `lldb/Host/windows/AutoHandle.h`, `lldb/Host/windows/HostThreadWindows.h` ... (+14 more)
- **Standard-library headers / 标准库头文件**: `<psapi.h>`, `<tlhelp32.h>`, `<ntstatus.h>`
- **Subsystem categories / 子系统类别**: host OS abstraction helpers / 主机操作系统抽象辅助组件 (9), LLVM support-library helpers / LLVM Support 库辅助组件 (6), target, process, and thread control / 目标、进程与线程控制 (2), shared LLDB utility classes / 共享 LLDB 工具类 (1)
