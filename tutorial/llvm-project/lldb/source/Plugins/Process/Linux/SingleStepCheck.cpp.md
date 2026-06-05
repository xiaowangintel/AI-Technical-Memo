# SingleStepCheck.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/Process/Linux/SingleStepCheck.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements the LLDB support logic associated with `SingleStepCheck`.
  - **CN**: 实现与 `SingleStepCheck` 相关的 LLDB 支持逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- SingleStepCheck.cpp -----------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/LLDB file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/LLDB 标准文件头、许可证声明以及文件的高层说明。

### Lines 8-15
```cpp

#include "SingleStepCheck.h"

#include <csignal>
#include <sched.h>
#include <sys/wait.h>
#include <unistd.h>

```
- **EN**: Pulls in the headers needed by this translation unit, including `SingleStepCheck.h`, `csignal`, `sched.h`, `sys/wait.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `SingleStepCheck.h`, `csignal`, `sched.h`, `sys/wait.h`。

### Lines 16-24
```cpp
#include "NativeProcessLinux.h"

#include "llvm/Support/Compiler.h"
#include "llvm/Support/Errno.h"

#include "Plugins/Process/POSIX/ProcessPOSIXLog.h"
#include "lldb/Host/linux/Ptrace.h"
#include "lldb/Utility/Status.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `NativeProcessLinux.h`, `llvm/Support/Compiler.h`, `llvm/Support/Errno.h`, `Plugins/Process/POSIX/ProcessPOSIXLog.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `NativeProcessLinux.h`, `llvm/Support/Compiler.h`, `llvm/Support/Errno.h`, `Plugins/Process/POSIX/ProcessPOSIXLog.h`。

### Lines 25-31
```cpp
using namespace lldb;
using namespace lldb_private;
using namespace lldb_private::process_linux;

#if defined(__arm64__) || defined(__aarch64__)
namespace {

```
- **EN**: Defines preprocessor-controlled structure, feature gates, or include-guard state.
- **CN**: 定义受预处理器控制的结构、特性开关或头文件保护状态。

### Lines 32-40
```cpp
[[noreturn]] void Child() {
  if (ptrace(PTRACE_TRACEME, 0, nullptr, nullptr) == -1)
    _exit(1);

  // We just do an endless loop SIGSTOPPING ourselves until killed. The tracer
  // will fiddle with our cpu affinities and monitor the behaviour.
  for (;;) {
    raise(SIGSTOP);

```
- **EN**: Implements logic around `Child`, `ptrace`, `_exit`, `raise`.
- **CN**: 围绕 `Child`, `ptrace`, `_exit`, `raise` 实现具体逻辑。

### Lines 41-49
```cpp
    // Generate a bunch of instructions here, so that a single-step does not
    // land in the raise() accidentally. If single-stepping works, we will be
    // spinning in this loop. If it doesn't, we'll land in the raise() call
    // above.
    for (volatile unsigned i = 0; i < CPU_SETSIZE; ++i)
      ;
  }
}

```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

### Lines 50-61
```cpp
struct ChildDeleter {
  ::pid_t pid;

  ~ChildDeleter() {
    int status;
    // Kill the child.
    kill(pid, SIGKILL);
    // Pick up the remains.
    llvm::sys::RetryAfterSignal(-1, waitpid, pid, &status, __WALL);
  }
};

```
- **EN**: Introduces declarations for `ChildDeleter`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ChildDeleter` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 62-75
```cpp
bool WorkaroundNeeded() {
  // We shall spawn a child, and use it to verify the debug capabilities of the
  // cpu. We shall iterate through the cpus, bind the child to each one in
  // turn, and verify that single-stepping works on that cpu. A workaround is
  // needed if we find at least one broken cpu.

  Log *log = GetLog(POSIXLog::Thread);
  ::pid_t child_pid = fork();
  if (child_pid == -1) {
    LLDB_LOG(log, "failed to fork(): {0}", Status(errno, eErrorTypePOSIX));
    return false;
  }
  if (child_pid == 0)
    Child();
```
- **EN**: Implements logic around `WorkaroundNeeded`, `GetLog`, `fork`, `LLDB_LOG`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `WorkaroundNeeded`, `GetLog`, `fork`, `LLDB_LOG`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 76-85
```cpp

  ChildDeleter child_deleter{child_pid};
  cpu_set_t available_cpus;
  if (sched_getaffinity(child_pid, sizeof available_cpus, &available_cpus) ==
      -1) {
    LLDB_LOG(log, "failed to get available cpus: {0}",
             Status(errno, eErrorTypePOSIX));
    return false;
  }

```
- **EN**: Implements logic around `sched_getaffinity`, `LLDB_LOG`, `Status`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `sched_getaffinity`, `LLDB_LOG`, `Status` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 86-94
```cpp
  int status;
  ::pid_t wpid = llvm::sys::RetryAfterSignal(-1, waitpid,
      child_pid, &status, __WALL);
  if (wpid != child_pid || !WIFSTOPPED(status)) {
    LLDB_LOG(log, "waitpid() failed (status = {0:x}): {1}", status,
             Status(errno, eErrorTypePOSIX));
    return false;
  }

```
- **EN**: Implements logic around `RetryAfterSignal`, `WIFSTOPPED`, `LLDB_LOG`, `Status`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `RetryAfterSignal`, `WIFSTOPPED`, `LLDB_LOG`, `Status` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 95-108
```cpp
  unsigned cpu;
  for (cpu = 0; cpu < CPU_SETSIZE; ++cpu) {
    if (!CPU_ISSET(cpu, &available_cpus))
      continue;

    cpu_set_t cpus;
    CPU_ZERO(&cpus);
    CPU_SET(cpu, &cpus);
    if (sched_setaffinity(child_pid, sizeof cpus, &cpus) == -1) {
      LLDB_LOG(log, "failed to switch to cpu {0}: {1}", cpu,
               Status(errno, eErrorTypePOSIX));
      continue;
    }

```
- **EN**: Implements logic around `CPU_ISSET`, `CPU_ZERO`, `CPU_SET`, `sched_setaffinity`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `CPU_ISSET`, `CPU_ZERO`, `CPU_SET`, `sched_setaffinity`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 109-116
```cpp
    int status;
    Status error =
        NativeProcessLinux::PtraceWrapper(PTRACE_SINGLESTEP, child_pid);
    if (error.Fail()) {
      LLDB_LOG(log, "single step failed: {0}", error);
      break;
    }

```
- **EN**: Implements logic around `PtraceWrapper`, `Fail`, `LLDB_LOG`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `PtraceWrapper`, `Fail`, `LLDB_LOG` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 117-130
```cpp
    wpid = llvm::sys::RetryAfterSignal(-1, waitpid,
        child_pid, &status, __WALL);
    if (wpid != child_pid || !WIFSTOPPED(status)) {
      LLDB_LOG(log, "waitpid() failed (status = {0:x}): {1}", status,
               Status(errno, eErrorTypePOSIX));
      break;
    }
    if (WSTOPSIG(status) != SIGTRAP) {
      LLDB_LOG(log, "single stepping on cpu {0} failed with status {1:x}", cpu,
               status);
      break;
    }
  }

```
- **EN**: Implements logic around `RetryAfterSignal`, `WIFSTOPPED`, `LLDB_LOG`, `Status`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `RetryAfterSignal`, `WIFSTOPPED`, `LLDB_LOG`, `Status`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 131-140
```cpp
  // cpu is either the index of the first broken cpu, or CPU_SETSIZE.
  if (cpu == 0) {
    LLDB_LOG(log,
             "SINGLE STEPPING ON FIRST CPU IS NOT WORKING. DEBUGGING "
             "LIKELY TO BE UNRELIABLE.");
    // No point in trying to fiddle with the affinities, just give it our best
    // shot and see how it goes.
    return false;
  }

```
- **EN**: Implements logic around `LLDB_LOG`.
- **CN**: 围绕 `LLDB_LOG` 实现具体逻辑。

### Lines 141-148
```cpp
  return cpu != CPU_SETSIZE;
}

} // end anonymous namespace

std::unique_ptr<SingleStepWorkaround> SingleStepWorkaround::Get(::pid_t tid) {
  Log *log = GetLog(POSIXLog::Thread);

```
- **EN**: Implements logic around `Get`, `GetLog`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `Get`, `GetLog` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 149-162
```cpp
  static bool workaround_needed = WorkaroundNeeded();
  if (!workaround_needed) {
    LLDB_LOG(log, "workaround for thread {0} not needed", tid);
    return nullptr;
  }

  cpu_set_t original_set;
  if (sched_getaffinity(tid, sizeof original_set, &original_set) != 0) {
    // This should really not fail. But, just in case...
    LLDB_LOG(log, "Unable to get cpu affinity for thread {0}: {1}", tid,
             Status(errno, eErrorTypePOSIX));
    return nullptr;
  }

```
- **EN**: Implements logic around `WorkaroundNeeded`, `LLDB_LOG`, `sched_getaffinity`, `Status`; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `WorkaroundNeeded`, `LLDB_LOG`, `sched_getaffinity`, `Status` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 163-173
```cpp
  cpu_set_t set;
  CPU_ZERO(&set);
  CPU_SET(0, &set);
  if (sched_setaffinity(tid, sizeof set, &set) != 0) {
    // This may fail in very locked down systems, if the thread is not allowed
    // to run on cpu 0. If that happens, only thing we can do is it log it and
    // continue...
    LLDB_LOG(log, "Unable to set cpu affinity for thread {0}: {1}", tid,
             Status(errno, eErrorTypePOSIX));
  }

```
- **EN**: Implements logic around `CPU_ZERO`, `CPU_SET`, `sched_setaffinity`, `LLDB_LOG`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `CPU_ZERO`, `CPU_SET`, `sched_setaffinity`, `LLDB_LOG`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 174-186
```cpp
  LLDB_LOG(log, "workaround for thread {0} prepared", tid);
  return std::make_unique<SingleStepWorkaround>(tid, original_set);
}

SingleStepWorkaround::~SingleStepWorkaround() {
  Log *log = GetLog(POSIXLog::Thread);
  LLDB_LOG(log, "Removing workaround");
  if (sched_setaffinity(m_tid, sizeof m_original_set, &m_original_set) != 0) {
    LLDB_LOG(log, "Unable to reset cpu affinity for thread {0}: {1}", m_tid,
             Status(errno, eErrorTypePOSIX));
  }
}
#endif
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or include-guard state.
- **CN**: 定义受预处理器控制的结构、特性开关或头文件保护状态。

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

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `SingleStepCheck.h`, `NativeProcessLinux.h`, `llvm/Support/Compiler.h`, `llvm/Support/Errno.h`, `Plugins/Process/POSIX/ProcessPOSIXLog.h`, `lldb/Host/linux/Ptrace.h`, `lldb/Utility/Status.h`
- **Standard-library headers / 标准库头文件**: `<csignal>`, `<sched.h>`, `<sys/wait.h>`, `<unistd.h>`
- **Subsystem categories / 子系统类别**: LLVM support-library helpers / LLVM Support 库辅助组件 (2), host OS abstraction helpers / 主机操作系统抽象辅助组件 (1), shared LLDB utility classes / 共享 LLDB 工具类 (1)
