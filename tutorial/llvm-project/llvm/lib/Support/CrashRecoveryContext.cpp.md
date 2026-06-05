# CrashRecoveryContext.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/CrashRecoveryContext.cpp`
- Repository: `llvm-project`
- Purpose (EN): When threads are disabled, this links up all active CrashRecoveryContextImpls.
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `CrashRecoveryContext` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-60

```cpp
//===--- CrashRecoveryContext.cpp - Crash Recovery ------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/Support/CrashRecoveryContext.h"
#include "llvm/Config/llvm-config.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/ExitCodes.h"
#include "llvm/Support/Signals.h"
#include "llvm/Support/thread.h"
#include <cassert>
#include <mutex>
#include <setjmp.h>

using namespace llvm;

namespace {

struct CrashRecoveryContextImpl;
static LLVM_THREAD_LOCAL const CrashRecoveryContextImpl *CurrentContext;

struct CrashRecoveryContextImpl {
  // When threads are disabled, this links up all active
  // CrashRecoveryContextImpls.  When threads are enabled there's one thread
  // per CrashRecoveryContext and CurrentContext is a thread-local, so only one
  // CrashRecoveryContextImpl is active per thread and this is always null.
  const CrashRecoveryContextImpl *Next;

  CrashRecoveryContext *CRC;
  ::jmp_buf JumpBuffer;
  volatile unsigned Failed : 1;
  unsigned SwitchedThread : 1;
  unsigned ValidJumpBuffer : 1;

public:
  CrashRecoveryContextImpl(CrashRecoveryContext *CRC) noexcept
      : CRC(CRC), Failed(false), SwitchedThread(false), ValidJumpBuffer(false) {
    Next = CurrentContext;
    CurrentContext = this;
  }
  ~CrashRecoveryContextImpl() {
    if (!SwitchedThread)
      CurrentContext = Next;
  }

  /// Called when the separate crash-recovery thread was finished, to
  /// indicate that we don't need to clear the thread-local CurrentContext.
  void setSwitchedThread() {
#if defined(LLVM_ENABLE_THREADS) && LLVM_ENABLE_THREADS != 0
    SwitchedThread = true;
#endif
  }

  // If the function ran by the CrashRecoveryContext crashes or fails, then
  // 'RetCode' represents the returned error code, as if it was returned by a
  // process. 'Context' represents the signal type on Unix; on Windows, it is
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 9 direct dependencies, including `llvm/Support/CrashRecoveryContext.h`, `llvm/Config/llvm-config.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/ExitCodes.h`.
  CN: 引入了 9 个直接依赖，其中包括 `llvm/Support/CrashRecoveryContext.h`, `llvm/Config/llvm-config.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/ExitCodes.h`。
- EN: This section centers on `CrashRecoveryContextImpl`, `~CrashRecoveryContextImpl`, `setSwitchedThread` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `CrashRecoveryContextImpl`, `~CrashRecoveryContextImpl`, `setSwitchedThread` 等符号展开，负责解析输入并把原始数据转换成结构化形式。

### Lines 61-120

```cpp
  // the ExceptionContext.
  void HandleCrash(int RetCode, uintptr_t Context) {
    // Eliminate the current context entry, to avoid re-entering in case the
    // cleanup code crashes.
    CurrentContext = Next;

    assert(!Failed && "Crash recovery context already failed!");
    Failed = true;

    if (CRC->DumpStackAndCleanupOnFailure)
      sys::CleanupOnSignal(Context);

    CRC->RetCode = RetCode;

    // Jump back to the RunSafely we were called under.
    if (ValidJumpBuffer)
      longjmp(JumpBuffer, 1);

    // Otherwise let the caller decide of the outcome of the crash. Currently
    // this occurs when using SEH on Windows with MSVC or clang-cl.
  }
};

std::mutex &getCrashRecoveryContextMutex() {
  static std::mutex CrashRecoveryContextMutex;
  return CrashRecoveryContextMutex;
}

static bool gCrashRecoveryEnabled = false;

static LLVM_THREAD_LOCAL const CrashRecoveryContext *IsRecoveringFromCrash;

} // namespace

static void
installExceptionOrSignalHandlers(bool NeedsPOSIXUtilitySignalHandling);
static void uninstallExceptionOrSignalHandlers();

CrashRecoveryContextCleanup::~CrashRecoveryContextCleanup() = default;

CrashRecoveryContext::CrashRecoveryContext() {
  // On Windows, if abort() was previously triggered (and caught by a previous
  // CrashRecoveryContext) the Windows CRT removes our installed signal handler,
  // so we need to install it again.
  sys::DisableSystemDialogsOnCrash();
}

CrashRecoveryContext::~CrashRecoveryContext() {
  // Reclaim registered resources.
  CrashRecoveryContextCleanup *i = head;
  const CrashRecoveryContext *PC = IsRecoveringFromCrash;
  IsRecoveringFromCrash = this;
  while (i) {
    CrashRecoveryContextCleanup *tmp = i;
    i = tmp->next;
    tmp->cleanupFired = true;
    tmp->recoverResources();
    delete tmp;
  }
  IsRecoveringFromCrash = PC;
```
- EN: This section centers on `HandleCrash`, `assert`, `installExceptionOrSignalHandlers` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `HandleCrash`, `assert`, `installExceptionOrSignalHandlers` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 121-180

```cpp

  CrashRecoveryContextImpl *CRCI = (CrashRecoveryContextImpl *) Impl;
  delete CRCI;
}

bool CrashRecoveryContext::isRecoveringFromCrash() {
  return IsRecoveringFromCrash != nullptr;
}

CrashRecoveryContext *CrashRecoveryContext::GetCurrent() {
  if (!gCrashRecoveryEnabled)
    return nullptr;

  const CrashRecoveryContextImpl *CRCI = CurrentContext;
  if (!CRCI)
    return nullptr;

  return CRCI->CRC;
}

void CrashRecoveryContext::Enable(bool NeedsPOSIXUtilitySignalHandling) {
  std::lock_guard<std::mutex> L(getCrashRecoveryContextMutex());
  // FIXME: Shouldn't this be a refcount or something?
  if (gCrashRecoveryEnabled)
    return;
  gCrashRecoveryEnabled = true;
  installExceptionOrSignalHandlers(NeedsPOSIXUtilitySignalHandling);
}

void CrashRecoveryContext::Disable() {
  std::lock_guard<std::mutex> L(getCrashRecoveryContextMutex());
  if (!gCrashRecoveryEnabled)
    return;
  gCrashRecoveryEnabled = false;
  uninstallExceptionOrSignalHandlers();
}

void CrashRecoveryContext::registerCleanup(CrashRecoveryContextCleanup *cleanup)
{
  if (!cleanup)
    return;
  if (head)
    head->prev = cleanup;
  cleanup->next = head;
  head = cleanup;
}

void
CrashRecoveryContext::unregisterCleanup(CrashRecoveryContextCleanup *cleanup) {
  if (!cleanup)
    return;
  if (cleanup == head) {
    head = cleanup->next;
    if (head)
      head->prev = nullptr;
  }
  else {
    cleanup->prev->next = cleanup->next;
    if (cleanup->next)
      cleanup->next->prev = cleanup->prev;
```
- EN: This section centers on `isRecoveringFromCrash`, `Enable`, `L` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `isRecoveringFromCrash`, `Enable`, `L` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code checks conditions and handles edge cases and returns the resulting value to its callers.
  CN: 在这一段中，代码检查条件并处理边界情况，并将结果返回给调用方。

### Lines 181-240

```cpp
  }
  delete cleanup;
}

#if defined(_MSC_VER)

#include <windows.h> // for GetExceptionInformation

// If _MSC_VER is defined, we must have SEH. Use it if it's available. It's way
// better than VEH. Vectored exception handling catches all exceptions happening
// on the thread with installed exception handlers, so it can interfere with
// internal exception handling of other libraries on that thread. SEH works
// exactly as you would expect normal exception handling to work: it only
// catches exceptions if they would bubble out from the stack frame with __try /
// __except.

static void
installExceptionOrSignalHandlers(bool NeedsPOSIXUtilitySignalHandling) {}
static void uninstallExceptionOrSignalHandlers() {}

// We need this function because the call to GetExceptionInformation() can only
// occur inside the __except evaluation block
static int ExceptionFilter(_EXCEPTION_POINTERS *Except) {
  // Lookup the current thread local recovery object.
  const CrashRecoveryContextImpl *CRCI = CurrentContext;

  if (!CRCI) {
    // Something has gone horribly wrong, so let's just tell everyone
    // to keep searching
    CrashRecoveryContext::Disable();
    return EXCEPTION_CONTINUE_SEARCH;
  }

  int RetCode = (int)Except->ExceptionRecord->ExceptionCode;
  if ((RetCode & 0xF0000000) == 0xE0000000)
    RetCode &= ~0xF0000000; // this crash was generated by sys::Process::Exit

  // Handle the crash
  const_cast<CrashRecoveryContextImpl *>(CRCI)->HandleCrash(
      RetCode, reinterpret_cast<uintptr_t>(Except));

  return EXCEPTION_EXECUTE_HANDLER;
}

#if defined(__clang__) && defined(_M_IX86)
// Work around PR44697.
__attribute__((optnone))
#endif
bool CrashRecoveryContext::RunSafely(function_ref<void()> Fn) {
  if (!gCrashRecoveryEnabled) {
    Fn();
    return true;
  }
  assert(!Impl && "Crash recovery context already initialized!");
  Impl = new CrashRecoveryContextImpl(this);
  __try {
    Fn();
  } __except (ExceptionFilter(GetExceptionInformation())) {
    return false;
  }
```
- EN: Brings in 1 direct dependencies, including `windows.h`.
  CN: 引入了 1 个直接依赖，其中包括 `windows.h`。
- EN: This section centers on `installExceptionOrSignalHandlers`, `uninstallExceptionOrSignalHandlers`, `ExceptionFilter` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `installExceptionOrSignalHandlers`, `uninstallExceptionOrSignalHandlers`, `ExceptionFilter` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 241-300

```cpp
  return true;
}

#else // !_MSC_VER

#if defined(_WIN32)
// This is a non-MSVC compiler, probably mingw gcc or clang without
// -fms-extensions. Use vectored exception handling (VEH).
//
// On Windows, we can make use of vectored exception handling to catch most
// crashing situations.  Note that this does mean we will be alerted of
// exceptions *before* structured exception handling has the opportunity to
// catch it. Unfortunately, this causes problems in practice with other code
// running on threads with LLVM crash recovery contexts, so we would like to
// eventually move away from VEH.
//
// Vectored works on a per-thread basis, which is an advantage over
// SetUnhandledExceptionFilter. SetUnhandledExceptionFilter also doesn't have
// any native support for chaining exception handlers, but VEH allows more than
// one.
//
// The vectored exception handler functionality was added in Windows
// XP, so if support for older versions of Windows is required,
// it will have to be added.

#include "llvm/Support/Windows/WindowsSupport.h"

static LONG CALLBACK ExceptionHandler(PEXCEPTION_POINTERS ExceptionInfo)
{
  // DBG_PRINTEXCEPTION_WIDE_C is not properly defined on all supported
  // compilers and platforms, so we define it manually.
  constexpr ULONG DbgPrintExceptionWideC = 0x4001000AL;
  switch (ExceptionInfo->ExceptionRecord->ExceptionCode)
  {
  case DBG_PRINTEXCEPTION_C:
  case DbgPrintExceptionWideC:
  case 0x406D1388:  // set debugger thread name
    return EXCEPTION_CONTINUE_EXECUTION;
  }

  // Lookup the current thread local recovery object.
  const CrashRecoveryContextImpl *CRCI = CurrentContext;

  if (!CRCI) {
    // Something has gone horribly wrong, so let's just tell everyone
    // to keep searching
    CrashRecoveryContext::Disable();
    return EXCEPTION_CONTINUE_SEARCH;
  }

  // TODO: We can capture the stack backtrace here and store it on the
  // implementation if we so choose.

  int RetCode = (int)ExceptionInfo->ExceptionRecord->ExceptionCode;
  if ((RetCode & 0xF0000000) == 0xE0000000)
    RetCode &= ~0xF0000000; // this crash was generated by sys::Process::Exit

  // Handle the crash
  const_cast<CrashRecoveryContextImpl *>(CRCI)->HandleCrash(
      RetCode, reinterpret_cast<uintptr_t>(ExceptionInfo));
```
- EN: Brings in 1 direct dependencies, including `llvm/Support/Windows/WindowsSupport.h`.
  CN: 引入了 1 个直接依赖，其中包括 `llvm/Support/Windows/WindowsSupport.h`。
- EN: This section centers on `ExceptionHandler`, `Disable` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `ExceptionHandler`, `Disable` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and checks conditions and handles edge cases.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并检查条件并处理边界情况。

### Lines 301-360

```cpp

  // Note that we don't actually get here because HandleCrash calls
  // longjmp, which means the HandleCrash function never returns.
  llvm_unreachable("Handled the crash, should have longjmp'ed out of here");
}

// Because the Enable and Disable calls are static, it means that
// there may not actually be an Impl available, or even a current
// CrashRecoveryContext at all.  So we make use of a thread-local
// exception table.  The handles contained in here will either be
// non-NULL, valid VEH handles, or NULL.
static LLVM_THREAD_LOCAL const void* sCurrentExceptionHandle;

static void
installExceptionOrSignalHandlers(bool NeedsPOSIXUtilitySignalHandling) {
  // We can set up vectored exception handling now.  We will install our
  // handler as the front of the list, though there's no assurances that
  // it will remain at the front (another call could install itself before
  // our handler).  This 1) isn't likely, and 2) shouldn't cause problems.
  PVOID handle = ::AddVectoredExceptionHandler(1, ExceptionHandler);
  sCurrentExceptionHandle = handle;
}

static void uninstallExceptionOrSignalHandlers() {
  PVOID currentHandle = const_cast<PVOID>(sCurrentExceptionHandle);
  if (currentHandle) {
    // Now we can remove the vectored exception handler from the chain
    ::RemoveVectoredExceptionHandler(currentHandle);

    // Reset the handle in our thread-local set.
    sCurrentExceptionHandle = NULL;
  }
}

#else // !_WIN32

// Generic POSIX implementation.
//
// This implementation relies on synchronous signals being delivered to the
// current thread. We use a thread local object to keep track of the active
// crash recovery context, and install signal handlers to invoke HandleCrash on
// the active object.
//
// This implementation does not attempt to chain signal handlers in any
// reliable fashion -- if we get a signal outside of a crash recovery context we
// simply disable crash recovery and raise the signal again.

#include <signal.h>

static const int Signals[] =
    { SIGABRT, SIGBUS, SIGFPE, SIGILL, SIGSEGV, SIGTRAP };
static const unsigned NumSignals = std::size(Signals);
static struct sigaction PrevActions[NumSignals];

static void CrashRecoverySignalHandler(int Signal) {
  // Lookup the current thread local recovery object.
  const CrashRecoveryContextImpl *CRCI = CurrentContext;

  if (!CRCI) {
    // We didn't find a crash recovery context -- this means either we got a
```
- EN: Brings in 1 direct dependencies, including `signal.h`.
  CN: 引入了 1 个直接依赖，其中包括 `signal.h`。
- EN: This section centers on `llvm_unreachable`, `installExceptionOrSignalHandlers`, `uninstallExceptionOrSignalHandlers` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `llvm_unreachable`, `installExceptionOrSignalHandlers`, `uninstallExceptionOrSignalHandlers` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 361-420

```cpp
    // signal on a thread we didn't expect it on, the application got a signal
    // outside of a crash recovery context, or something else went horribly
    // wrong.
    //
    // Disable crash recovery and raise the signal again. The assumption here is
    // that the enclosing application will terminate soon, and we won't want to
    // attempt crash recovery again.
    //
    // This call of Disable isn't thread safe, but it doesn't actually matter.
    CrashRecoveryContext::Disable();
    raise(Signal);

    // The signal will be thrown once the signal mask is restored.
    return;
  }

  // Unblock the signal we received.
  sigset_t SigMask;
  sigemptyset(&SigMask);
  sigaddset(&SigMask, Signal);
  sigprocmask(SIG_UNBLOCK, &SigMask, nullptr);

  // Return the same error code as if the program crashed, as mentioned in the
  // section "Exit Status for Commands":
  // https://pubs.opengroup.org/onlinepubs/9699919799/xrat/V4_xcu_chap02.html
  int RetCode = 128 + Signal;

  // Don't consider a broken pipe as a crash (see clang/lib/Driver/Driver.cpp)
  if (Signal == SIGPIPE)
    RetCode = EX_IOERR;

  if (CRCI)
    const_cast<CrashRecoveryContextImpl *>(CRCI)->HandleCrash(RetCode, Signal);
}

static void
installExceptionOrSignalHandlers(bool NeedsPOSIXUtilitySignalHandling) {
  // Setup the signal handler.
  struct sigaction Handler;
  Handler.sa_handler = CrashRecoverySignalHandler;
  Handler.sa_flags = 0;
  sigemptyset(&Handler.sa_mask);

  for (unsigned i = 0; i != NumSignals; ++i) {
    if (NeedsPOSIXUtilitySignalHandling) {
      // Don't install the new handler if the signal disposition is SIG_IGN.
      struct sigaction act;
      if (sigaction(Signals[i], NULL, &act) == 0 && act.sa_handler != SIG_IGN)
        sigaction(Signals[i], &Handler, &PrevActions[i]);
    } else {
      sigaction(Signals[i], &Handler, &PrevActions[i]);
    }
  }
}

static void uninstallExceptionOrSignalHandlers() {
  // Restore the previous signal handlers.
  for (unsigned i = 0; i != NumSignals; ++i)
    sigaction(Signals[i], &PrevActions[i], nullptr);
}
```
- EN: This section centers on `Disable`, `raise`, `sigemptyset` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `Disable`, `raise`, `sigemptyset` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 421-480

```cpp

#endif // !_WIN32

bool CrashRecoveryContext::RunSafely(function_ref<void()> Fn) {
  // If crash recovery is disabled, do nothing.
  if (gCrashRecoveryEnabled) {
    assert(!Impl && "Crash recovery context already initialized!");
    CrashRecoveryContextImpl *CRCI = new CrashRecoveryContextImpl(this);
    Impl = CRCI;

    CRCI->ValidJumpBuffer = true;
    if (setjmp(CRCI->JumpBuffer) != 0) {
      return false;
    }
  }

  Fn();
  return true;
}

#endif // !_MSC_VER

[[noreturn]] void CrashRecoveryContext::HandleExit(int RetCode) {
#if defined(_WIN32)
  // Since the exception code is actually of NTSTATUS type, we use the
  // Microsoft-recommended 0xE prefix, to signify that this is a user error.
  // This value is a combination of the customer field (bit 29) and severity
  // field (bits 30-31) in the NTSTATUS specification.
  ::RaiseException(0xE0000000 | RetCode, 0, 0, NULL);
#else
  // On Unix we don't need to raise an exception, we go directly to
  // HandleCrash(), then longjmp will unwind the stack for us.
  CrashRecoveryContextImpl *CRCI = (CrashRecoveryContextImpl *)Impl;
  assert(CRCI && "Crash recovery context never initialized!");
  CRCI->HandleCrash(RetCode, 0 /*no sig num*/);
#endif
  llvm_unreachable("Most likely setjmp wasn't called!");
}

bool CrashRecoveryContext::isCrash(int RetCode) {
#if defined(_WIN32)
  // On Windows, the code is interpreted as NTSTATUS. The two high bits
  // represent the severity. Values starting with 0x80000000 are reserved for
  // "warnings"; values of 0xC0000000 and up are for "errors". In practice, both
  // are interpreted as a non-continuable signal.
  unsigned Code = ((unsigned)RetCode & 0xF0000000) >> 28;
  if (Code != 0xC && Code != 8)
    return false;
#else
  // On Unix, signals are represented by return codes of 128 or higher.
  // Exit code 128 is a reserved value and should not be raised as a signal.
  if (RetCode <= 128)
    return false;
#endif
  return true;
}

bool CrashRecoveryContext::throwIfCrash(int RetCode) {
  if (!isCrash(RetCode))
    return false;
```
- EN: This section centers on `RunSafely`, `assert`, `Fn` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `RunSafely`, `assert`, `Fn` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 481-540

```cpp
#if defined(_WIN32)
  ::RaiseException(RetCode, 0, 0, NULL);
#else
  llvm::sys::unregisterHandlers();
  raise(RetCode - 128);
#endif
  return true;
}

// FIXME: Portability.
static void setThreadBackgroundPriority() {
#ifdef __APPLE__
  setpriority(PRIO_DARWIN_THREAD, 0, PRIO_DARWIN_BG);
#endif
}

static bool hasThreadBackgroundPriority() {
#ifdef __APPLE__
  return getpriority(PRIO_DARWIN_THREAD, 0) == 1;
#else
  return false;
#endif
}

namespace {
struct RunSafelyOnThreadInfo {
  function_ref<void()> Fn;
  CrashRecoveryContext *CRC;
  bool UseBackgroundPriority;
  bool Result;
};
} // namespace

static void RunSafelyOnThread_Dispatch(void *UserData) {
  RunSafelyOnThreadInfo *Info =
    reinterpret_cast<RunSafelyOnThreadInfo*>(UserData);

  if (Info->UseBackgroundPriority)
    setThreadBackgroundPriority();

  Info->Result = Info->CRC->RunSafely(Info->Fn);
}
bool CrashRecoveryContext::RunSafelyOnThread(function_ref<void()> Fn,
                                             unsigned RequestedStackSize) {
  bool UseBackgroundPriority = hasThreadBackgroundPriority();
  RunSafelyOnThreadInfo Info = { Fn, this, UseBackgroundPriority, false };
  llvm::thread Thread(RequestedStackSize == 0
                          ? std::nullopt
                          : std::optional<unsigned>(RequestedStackSize),
                      RunSafelyOnThread_Dispatch, &Info);
  Thread.join();

  if (CrashRecoveryContextImpl *CRC = (CrashRecoveryContextImpl *)Impl)
    CRC->setSwitchedThread();
  return Info.Result;
}

bool CrashRecoveryContext::RunSafelyOnNewStack(function_ref<void()> Fn,
                                               unsigned RequestedStackSize) {
  return RunSafelyOnThread(Fn, RequestedStackSize);
```
- EN: This section centers on `unregisterHandlers`, `raise`, `setThreadBackgroundPriority` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `unregisterHandlers`, `raise`, `setThreadBackgroundPriority` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and checks conditions and handles edge cases.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并检查条件并处理边界情况。

### Lines 541-541

```cpp
}
```
- EN: This range contains supporting statements, comments, or structural glue code.
  CN: 这一段主要包含辅助语句、注释或结构性胶水代码。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `CrashRecoveryContextImpl`, `sigaction`, `RunSafelyOnThreadInfo`, `~CrashRecoveryContextImpl`, `setSwitchedThread`, `HandleCrash` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Helper routines coordinate local state, control flow, and result construction. / 通过辅助例程协调局部状态、控制流与结果构造。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/Support/CrashRecoveryContext.h`, `llvm/Config/llvm-config.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/ExitCodes.h`, `llvm/Support/Signals.h`, `llvm/Support/thread.h`, `llvm/Support/Windows/WindowsSupport.h`
- Standard library / 标准库: `mutex`, `setjmp.h`
- Other/system headers / 其他或系统头文件: `cassert`, `windows.h`, `signal.h`
- Related symbols / 相关符号: `CrashRecoveryContextImpl`, `sigaction`, `RunSafelyOnThreadInfo`, `~CrashRecoveryContextImpl`, `setSwitchedThread`, `HandleCrash`, `assert`
