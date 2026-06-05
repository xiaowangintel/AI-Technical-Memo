# NativeProcessAIX.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/Process/AIX/NativeProcessAIX.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements the LLDB support logic associated with `NativeProcessAIX`.
  - **CN**: 实现与 `NativeProcessAIX` 相关的 LLDB 支持逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```cpp
//===-- NativeProcessAIX.cpp ----------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "NativeProcessAIX.h"
#include "Plugins/Process/POSIX/ProcessPOSIXLog.h"
#include "lldb/Host/Host.h"
#include "lldb/Host/HostInfo.h"
#include "lldb/Host/HostProcess.h"
#include "lldb/Host/ProcessLaunchInfo.h"
#include "lldb/Host/posix/ProcessLauncherPosixFork.h"
#include "lldb/Symbol/ObjectFile.h"
#include "lldb/Utility/Log.h"
#include "lldb/Utility/State.h"
#include "lldb/Utility/Status.h"
#include "llvm/Support/Errno.h"
```
- **EN**: Pulls in the headers needed by this translation unit, including `NativeProcessAIX.h`, `Plugins/Process/POSIX/ProcessPOSIXLog.h`, `lldb/Host/Host.h`, `lldb/Host/HostInfo.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `NativeProcessAIX.h`, `Plugins/Process/POSIX/ProcessPOSIXLog.h`, `lldb/Host/Host.h`, `lldb/Host/HostInfo.h`。

### Lines 21-32
```cpp
#include "llvm/Support/Error.h"
#include <cerrno>
#include <cstdint>
#include <cstring>
#include <sstream>
#include <string>
#include <sys/ptrace.h>
#include <unistd.h>
#define DECLARE_REGISTER_INFOS_PPC64_STRUCT
#include "Plugins/Process/Utility/RegisterInfos_ppc64.h"
#undef DECLARE_REGISTER_INFOS_PPC64_STRUCT

```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/Support/Error.h`, `cerrno`, `cstdint`, `cstring`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/Support/Error.h`, `cerrno`, `cstdint`, `cstring`。

### Lines 33-52
```cpp
using namespace lldb;
using namespace lldb_private;
using namespace lldb_private::process_aix;
using namespace llvm;

static constexpr unsigned k_ptrace_word_size = sizeof(void *);
static_assert(sizeof(long) >= k_ptrace_word_size,
              "Size of long must be larger than ptrace word size");

// Simple helper function to ensure flags are enabled on the given file
// descriptor.
static llvm::Error SetFDFlags(int fd, int flags) {
  int status = fcntl(fd, F_GETFL);
  if (status == -1)
    return errorCodeToError(errnoAsErrorCode());
  if (fcntl(fd, F_SETFL, status | flags) == -1)
    return errorCodeToError(errnoAsErrorCode());
  return Error::success();
}

```
- **EN**: Implements logic around `static_assert`, `SetFDFlags`, `fcntl`, `errorCodeToError`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics; abstracts host OS resources such as files, terminals, sockets, or platforms.
- **CN**: 围绕 `static_assert`, `SetFDFlags`, `fcntl`, `errorCodeToError`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并抽象文件、终端、套接字或平台等主机操作系统资源。

### Lines 53-62
```cpp
NativeProcessAIX::Manager::Manager(MainLoop &mainloop)
    : NativeProcessProtocol::Manager(mainloop) {
  Status status;
  m_sigchld_handle = mainloop.RegisterSignal(
      SIGCHLD, [this](MainLoopBase &) { SigchldHandler(); }, status);
  assert(m_sigchld_handle && status.Success());
}

// Public Static Methods

```
- **EN**: Implements logic around `Manager`, `RegisterSignal`, `SigchldHandler`, `assert`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `Manager`, `RegisterSignal`, `SigchldHandler`, `assert` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 63-77
```cpp
llvm::Expected<std::unique_ptr<NativeProcessProtocol>>
NativeProcessAIX::Manager::Launch(ProcessLaunchInfo &launch_info,
                                  NativeDelegate &native_delegate) {
  Log *log = GetLog(POSIXLog::Process);

  Status status;
  ::pid_t pid = ProcessLauncherPosixFork()
                    .LaunchProcess(launch_info, status)
                    .GetProcessId();
  LLDB_LOG(log, "pid = {0:x}", pid);
  if (status.Fail()) {
    LLDB_LOG(log, "failed to launch process: {0}", status);
    return status.ToError();
  }

```
- **EN**: Implements logic around `Launch`, `GetLog`, `ProcessLauncherPosixFork`, `LaunchProcess`, and 4 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `Launch`, `GetLog`, `ProcessLauncherPosixFork`, `LaunchProcess`, and 4 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 78-89
```cpp
  // Wait for the child process to trap on its call to execve.
  int wstatus = 0;
  ::pid_t wpid = llvm::sys::RetryAfterSignal(-1, ::waitpid, pid, &wstatus, 0);
  assert(wpid == pid);
  UNUSED_IF_ASSERT_DISABLED(wpid);
  if (!WIFSTOPPED(wstatus)) {
    LLDB_LOG(log, "Could not sync with inferior process: wstatus={1}",
             WaitStatus::Decode(wstatus));
    return llvm::createStringError("could not sync with inferior process");
  }
  LLDB_LOG(log, "inferior started, now in stopped state");

```
- **EN**: Implements logic around `RetryAfterSignal`, `assert`, `UNUSED_IF_ASSERT_DISABLED`, `WIFSTOPPED`, and 3 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `RetryAfterSignal`, `assert`, `UNUSED_IF_ASSERT_DISABLED`, `WIFSTOPPED`, and 3 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 90-99
```cpp
  ProcessInstanceInfo Info;
  if (!Host::GetProcessInfo(pid, Info)) {
    return llvm::make_error<StringError>("Cannot get process architecture",
                                         llvm::inconvertibleErrorCode());
  }

  // Set the architecture to the exe architecture.
  LLDB_LOG(log, "pid = {0}, detected architecture {1}", pid,
           Info.GetArchitecture().GetArchitectureName());

```
- **EN**: Implements logic around `GetProcessInfo`, `make_error`, `inconvertibleErrorCode`, `LLDB_LOG`, and 1 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow; abstracts host OS resources such as files, terminals, sockets, or platforms.
- **CN**: 围绕 `GetProcessInfo`, `make_error`, `inconvertibleErrorCode`, `LLDB_LOG`, and 1 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并抽象文件、终端、套接字或平台等主机操作系统资源。

### Lines 100-110
```cpp
  return std::unique_ptr<NativeProcessAIX>(new NativeProcessAIX(
      pid, launch_info.GetPTY().ReleasePrimaryFileDescriptor(), native_delegate,
      Info.GetArchitecture(), *this, {pid}));
}

llvm::Expected<std::unique_ptr<NativeProcessProtocol>>
NativeProcessAIX::Manager::Attach(
    lldb::pid_t pid, NativeProcessProtocol::NativeDelegate &native_delegate) {
  Log *log = GetLog(POSIXLog::Process);
  LLDB_LOG(log, "pid = {0:x}", pid);

```
- **EN**: Implements logic around `unique_ptr`, `GetPTY`, `GetArchitecture`, `Attach`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `unique_ptr`, `GetPTY`, `GetArchitecture`, `Attach`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 111-123
```cpp
  auto tids_or = NativeProcessAIX::Attach(pid);
  if (!tids_or)
    return tids_or.takeError();

  return std::unique_ptr<NativeProcessAIX>(new NativeProcessAIX(
      pid, -1, native_delegate,
      HostInfo::GetArchitecture(HostInfo::eArchKind64), *this, *tids_or));
}

lldb::addr_t NativeProcessAIX::GetSharedLibraryInfoAddress() {
  return LLDB_INVALID_ADDRESS;
}

```
- **EN**: Implements logic around `Attach`, `takeError`, `unique_ptr`, `GetArchitecture`, and 1 more symbols.
- **CN**: 围绕 `Attach`, `takeError`, `unique_ptr`, `GetArchitecture`, and 1 more symbols 实现具体逻辑。

### Lines 124-133
```cpp
static std::optional<std::pair<lldb::pid_t, WaitStatus>> WaitPid() {
  Log *log = GetLog(POSIXLog::Process);

  int status;
  ::pid_t wait_pid =
      llvm::sys::RetryAfterSignal(-1, ::waitpid, -1, &status, WNOHANG);

  if (wait_pid == 0)
    return std::nullopt;

```
- **EN**: Implements logic around `WaitPid`, `GetLog`, `RetryAfterSignal`; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `WaitPid`, `GetLog`, `RetryAfterSignal` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 134-146
```cpp
  if (wait_pid == -1) {
    Status error(errno, eErrorTypePOSIX);
    LLDB_LOG(log, "waitpid(-1, &status, _) failed: {0}", error);
    return std::nullopt;
  }

  WaitStatus wait_status = WaitStatus::Decode(status);

  LLDB_LOG(log, "waitpid(-1, &status, _) = {0}, status = {1}", wait_pid,
           wait_status);
  return std::make_pair(wait_pid, wait_status);
}

```
- **EN**: Implements logic around `error`, `LLDB_LOG`, `Decode`, `make_pair`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `error`, `LLDB_LOG`, `Decode`, `make_pair` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 147-156
```cpp
void NativeProcessAIX::Manager::SigchldHandler() {
  while (true) {
    auto wait_result = WaitPid();
    if (!wait_result)
      return;
  }
}

void NativeProcessAIX::Manager::CollectThread(::pid_t tid) {}

```
- **EN**: Implements logic around `SigchldHandler`, `WaitPid`, `CollectThread`.
- **CN**: 围绕 `SigchldHandler`, `WaitPid`, `CollectThread` 实现具体逻辑。

### Lines 157-168
```cpp
// Public Instance Methods

NativeProcessAIX::NativeProcessAIX(::pid_t pid, int terminal_fd,
                                   NativeDelegate &delegate,
                                   const ArchSpec &arch, Manager &manager,
                                   llvm::ArrayRef<::pid_t> tids)
    : NativeProcessProtocol(pid, terminal_fd, delegate), m_manager(manager),
      m_arch(arch) {
  manager.AddProcess(*this);
  if (m_terminal_fd != -1)
    cantFail(SetFDFlags(m_terminal_fd, O_NONBLOCK));

```
- **EN**: Implements logic around `NativeProcessAIX`, `NativeProcessProtocol`, `m_arch`, `AddProcess`, and 1 more symbols; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `NativeProcessAIX`, `NativeProcessProtocol`, `m_arch`, `AddProcess`, and 1 more symbols 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 169-179
```cpp
  // Let our process instance know the thread has stopped.
  SetCurrentThreadID(tids[0]);
  SetState(StateType::eStateStopped, false);
}

llvm::Expected<std::vector<::pid_t>> NativeProcessAIX::Attach(::pid_t pid) {
  Log *log = GetLog(POSIXLog::Process);
  Status status;
  if (llvm::Error err = PtraceWrapper(PT_ATTACH, pid).takeError())
    return err;

```
- **EN**: Implements logic around `SetCurrentThreadID`, `SetState`, `Attach`, `GetLog`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `SetCurrentThreadID`, `SetState`, `Attach`, `GetLog`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 180-189
```cpp
  int wpid = llvm::sys::RetryAfterSignal(-1, ::waitpid, pid, nullptr, WNOHANG);
  if (wpid <= 0)
    return llvm::errorCodeToError(errnoAsErrorCode());
  LLDB_LOG(log, "adding pid = {0}", pid);

  return std::vector<::pid_t>{pid};
}

bool NativeProcessAIX::SupportHardwareSingleStepping() const { return false; }

```
- **EN**: Implements logic around `RetryAfterSignal`, `errorCodeToError`, `LLDB_LOG`, `SupportHardwareSingleStepping`.
- **CN**: 围绕 `RetryAfterSignal`, `errorCodeToError`, `LLDB_LOG`, `SupportHardwareSingleStepping` 实现具体逻辑。

### Lines 190-199
```cpp
Status NativeProcessAIX::Resume(const ResumeActionList &resume_actions) {
  return Status("unsupported");
}

Status NativeProcessAIX::Halt() { return Status("unsupported"); }

Status NativeProcessAIX::Detach() { return Status("unsupported"); }

Status NativeProcessAIX::Signal(int signo) { return Status("unsupported"); }

```
- **EN**: Implements logic around `Resume`, `Status`, `Halt`, `Detach`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `Resume`, `Status`, `Halt`, `Detach`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 200-219
```cpp
Status NativeProcessAIX::Interrupt() { return Status("unsupported"); }

Status NativeProcessAIX::Kill() {

  Log *log = GetLog(POSIXLog::Process);
  LLDB_LOG(log, "pid {0}", GetID());

  Status error;

  switch (m_state) {
  case StateType::eStateInvalid:
  case StateType::eStateExited:
  case StateType::eStateCrashed:
  case StateType::eStateDetached:
  case StateType::eStateUnloaded:
    // Nothing to do - the process is already dead.
    LLDB_LOG(log, "ignored for PID {0} due to current state: {1}", GetID(),
             m_state);
    return error;

```
- **EN**: Implements logic around `Interrupt`, `Kill`, `GetLog`, `LLDB_LOG`; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `Interrupt`, `Kill`, `GetLog`, `LLDB_LOG` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 220-230
```cpp
  case StateType::eStateConnected:
  case StateType::eStateAttaching:
  case StateType::eStateLaunching:
  case StateType::eStateStopped:
  case StateType::eStateRunning:
  case StateType::eStateStepping:
  case StateType::eStateSuspended:
    // We can try to kill a process in these states.
    break;
  }

```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

### Lines 231-240
```cpp
  llvm::Expected<int> result =
      PtraceWrapper(PT_KILL, GetID(), nullptr, nullptr, 0);
  if (!result) {
    std::string error_string = std::string("Kill failed for process. error: ") +
                               llvm::toString(result.takeError());
    error.FromErrorString(error_string.c_str());
  }
  return error;
}

```
- **EN**: Implements logic around `PtraceWrapper`, `string`, `toString`, `FromErrorString`; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `PtraceWrapper`, `string`, `toString`, `FromErrorString` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 241-250
```cpp
Status NativeProcessAIX::ReadMemory(lldb::addr_t addr, void *buf, size_t size,
                                    size_t &bytes_read) {
  return Status("unsupported");
}

Status NativeProcessAIX::WriteMemory(lldb::addr_t addr, const void *buf,
                                     size_t size, size_t &bytes_written) {
  return Status("unsupported");
}

```
- **EN**: Implements logic around `ReadMemory`, `Status`, `WriteMemory`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `ReadMemory`, `Status`, `WriteMemory` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 251-262
```cpp
size_t NativeProcessAIX::UpdateThreads() {
  // The NativeProcessAIX monitoring threads are always up to date with
  // respect to thread state and they keep the thread list populated properly.
  // All this method needs to do is return the thread count.
  return m_threads.size();
}

Status NativeProcessAIX::GetFileLoadAddress(const llvm::StringRef &file_name,
                                            lldb::addr_t &load_addr) {
  return Status("unsupported");
}

```
- **EN**: Implements logic around `UpdateThreads`, `size`, `GetFileLoadAddress`, `Status`; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `UpdateThreads`, `size`, `GetFileLoadAddress`, `Status` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 263-274
```cpp
Status NativeProcessAIX::GetLoadedModuleFileSpec(const char *module_path,
                                                 FileSpec &file_spec) {
  return Status("unsupported");
}

Status NativeProcessAIX::SetBreakpoint(lldb::addr_t addr, uint32_t size,
                                       bool hardware) {
  if (hardware)
    return SetHardwareBreakpoint(addr, size);
  return SetSoftwareBreakpoint(addr, size);
}

```
- **EN**: Implements logic around `GetLoadedModuleFileSpec`, `Status`, `SetBreakpoint`, `SetHardwareBreakpoint`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `GetLoadedModuleFileSpec`, `Status`, `SetBreakpoint`, `SetHardwareBreakpoint`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 275-284
```cpp
Status NativeProcessAIX::RemoveBreakpoint(lldb::addr_t addr, bool hardware) {
  if (hardware)
    return RemoveHardwareBreakpoint(addr);
  return NativeProcessProtocol::RemoveBreakpoint(addr);
}

llvm::Error NativeProcessAIX::Detach(lldb::tid_t tid) {
  return PtraceWrapper(PT_DETACH, tid).takeError();
}

```
- **EN**: Implements logic around `RemoveBreakpoint`, `RemoveHardwareBreakpoint`, `Detach`, `PtraceWrapper`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `RemoveBreakpoint`, `RemoveHardwareBreakpoint`, `Detach`, `PtraceWrapper` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 285-301
```cpp
template <typename GPR_T, typename PTSPRS_T>
int GetSPRs(int req, lldb::tid_t tid, GPR_T *gpr) {
  PTSPRS_T sprs;

  int ret = ptrace64(req, tid, reinterpret_cast<long long>(&sprs), 0, 0);

  if (ret != -1) {
    gpr->cr = sprs.pt_cr;
    gpr->msr = sprs.pt_msr;
    gpr->xer = sprs.pt_xer;
    gpr->lr = sprs.pt_lr;
    gpr->ctr = sprs.pt_ctr;
    gpr->pc = sprs.pt_iar;
  }
  return ret;
}

```
- **EN**: Implements logic around `GetSPRs`, `ptrace64`.
- **CN**: 围绕 `GetSPRs`, `ptrace64` 实现具体逻辑。

### Lines 302-312
```cpp
template <typename GPR_T, typename PTSPRS_T>
int SetSPRs(int req, lldb::tid_t tid, GPR_T *gpr) {
  PTSPRS_T sprs;

  sprs.pt_cr = gpr->cr;
  sprs.pt_msr = gpr->msr;
  sprs.pt_xer = gpr->xer;
  sprs.pt_lr = gpr->lr;
  sprs.pt_ctr = gpr->ctr;
  sprs.pt_iar = gpr->pc;

```
- **EN**: Implements logic around `SetSPRs`.
- **CN**: 围绕 `SetSPRs` 实现具体逻辑。

### Lines 313-329
```cpp
  return ptrace64(req, tid, reinterpret_cast<long long>(&sprs), 0, 0);
}

llvm::Expected<int> NativeProcessAIX::PtraceWrapper(int req, lldb::pid_t pid,
                                                    void *addr, void *data,
                                                    size_t data_size) {
  int ret = 0;
  Log *log = GetLog(POSIXLog::Ptrace);
  // PTT_* requests require a thread ID (TID).
  // Each entry under /proc/<pid>/lwp/ represents a thread,
  // and the directory name corresponds to its thread ID (TID).
  // A process may contain multiple threads.
  // TODO: With multi-threading support, iterate over all entries to enumerate
  // available TIDs and retrieve the target debugging thread ID.
  llvm::SmallString<128> proc_lwp_dir;
  llvm::sys::path::append(proc_lwp_dir, "/proc/", std::to_string(pid), "/lwp/");

```
- **EN**: Implements logic around `ptrace64`, `PtraceWrapper`, `GetLog`, `append`; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `ptrace64`, `PtraceWrapper`, `GetLog`, `append` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 330-340
```cpp
  lldb::tid_t tid = 0;
  std::error_code ec;
  bool result;
  if (!llvm::sys::fs::is_directory(proc_lwp_dir, result) && result) {
    for (sys::fs::directory_iterator it(proc_lwp_dir, ec), end;
         it != end && !ec; it.increment(ec)) {
      llvm::StringRef name = llvm::sys::path::filename(it->path());

      if (name == "." || name == "..")
        continue;

```
- **EN**: Implements logic around `is_directory`, `it`, `increment`, `filename`.
- **CN**: 围绕 `is_directory`, `it`, `increment`, `filename` 实现具体逻辑。

### Lines 341-359
```cpp
      if (!name.getAsInteger(10, tid)) {
        break;
      }
    }
  }

  switch (req) {
  // On AIX, ptrace exposes differently. GPRs and SPRs are handled via separate
  // requests: PTT_READ_GPRS reads only GPRs & PTT_READ_SPRS is required to
  // fetch SPRs. Similarly, writes are also split across:
  // PTT_WRITE_GPRS & PTT_WRITE_SPRS.
  case PTT_READ_GPRS:
    if (data_size == sizeof(GPR_PPC)) // 32bit SPRs read
      ret = GetSPRs<GPR_PPC, ptsprs>(PTT_READ_SPRS, tid,
                                     static_cast<GPR_PPC *>(data));
    else if (data_size == sizeof(GPR_PPC64)) // 64bit SPRs read
      ret = GetSPRs<GPR_PPC64, ptxsprs>(PTT_READ_SPRS, tid,
                                        static_cast<GPR_PPC64 *>(data));

```
- **EN**: Implements logic around `getAsInteger`, `ptsprs>`, `ptxsprs>`.
- **CN**: 围绕 `getAsInteger`, `ptsprs>`, `ptxsprs>` 实现具体逻辑。

### Lines 360-372
```cpp
    if (ret != -1)
      ret = ptrace64(req, tid, reinterpret_cast<long long>(data), 0,
                     0); // read GPRs
    break;

  case PTT_WRITE_GPRS:
    if (data_size == sizeof(GPR_PPC)) // 32bit SPRs write
      ret = SetSPRs<GPR_PPC, ptsprs>(PTT_WRITE_SPRS, tid,
                                     static_cast<GPR_PPC *>(data));
    else if (data_size == sizeof(GPR_PPC64)) // 64bit SPRS write
      ret = SetSPRs<GPR_PPC64, ptxsprs>(PTT_WRITE_SPRS, tid,
                                        static_cast<GPR_PPC64 *>(data));

```
- **EN**: Implements logic around `ptrace64`, `ptsprs>`, `ptxsprs>`.
- **CN**: 围绕 `ptrace64`, `ptsprs>`, `ptxsprs>` 实现具体逻辑。

### Lines 373-385
```cpp
    if (ret != -1)
      ret = ptrace64(req, tid, reinterpret_cast<long long>(data), 0,
                     0); // write GPRs
    break;
  case PT_ATTACH:
  case PT_DETACH:
  case PT_KILL:
    ret = ptrace64(req, pid, 0, 0, nullptr);
    break;
  default:
    llvm_unreachable("PT_ request not supported yet.");
  }

```
- **EN**: Implements logic around `ptrace64`, `llvm_unreachable`.
- **CN**: 围绕 `ptrace64`, `llvm_unreachable` 实现具体逻辑。

### Lines 386-394
```cpp
  LLDB_LOG(log, "ptrace({0}, {1}, {2}, {3}, {4})={5:x}", req, pid, addr, data,
           data_size, ret);

  if (ret == -1) {
    LLDB_LOG(log, "ptrace() failed");
    return llvm::errorCodeToError(errnoAsErrorCode());
  }
  return ret;
}
```
- **EN**: Implements logic around `LLDB_LOG`, `errorCodeToError`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `LLDB_LOG`, `errorCodeToError` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

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

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `NativeProcessAIX.h`, `Plugins/Process/POSIX/ProcessPOSIXLog.h`, `lldb/Host/Host.h`, `lldb/Host/HostInfo.h`, `lldb/Host/HostProcess.h`, `lldb/Host/ProcessLaunchInfo.h`, `lldb/Host/posix/ProcessLauncherPosixFork.h`, `lldb/Symbol/ObjectFile.h`, `lldb/Utility/Log.h`, `lldb/Utility/State.h` ... (+4 more)
- **Standard-library headers / 标准库头文件**: `<cerrno>`, `<cstdint>`, `<cstring>`, `<sstream>`, `<string>`, `<sys/ptrace.h>`, `<unistd.h>`
- **Subsystem categories / 子系统类别**: host OS abstraction helpers / 主机操作系统抽象辅助组件 (5), shared LLDB utility classes / 共享 LLDB 工具类 (3), LLVM support-library helpers / LLVM Support 库辅助组件 (2), symbol and debug-info abstractions / 符号与调试信息抽象 (1)
