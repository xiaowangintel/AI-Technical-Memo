# PrettyStackTrace.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/PrettyStackTrace.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file defines some helpful functions for dealing with the possibility of Unix signals occurring while your program is running.
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `PrettyStackTrace` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===- PrettyStackTrace.cpp - Pretty Crash Handling -----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines some helpful functions for dealing with the possibility of
// Unix signals occurring while your program is running.
//
//===----------------------------------------------------------------------===//

#include "llvm/Support/PrettyStackTrace.h"
#include "llvm-c/ErrorHandling.h"
#include "llvm/Config/config.h"
#include "llvm/Support/Compiler.h"
#include "llvm/Support/SaveAndRestore.h"
#include "llvm/Support/Signals.h"
#include "llvm/Support/Watchdog.h"
#include "llvm/Support/raw_ostream.h"

#ifdef __APPLE__
#include "llvm/ADT/SmallString.h"
#endif

#include <atomic>
#include <cassert>
#include <cstdarg>
#include <cstdio>
#include <cstring>
#include <tuple>

#ifdef HAVE_CRASHREPORTERCLIENT_H
#include <CrashReporterClient.h>
#endif

using namespace llvm;

static const char *BugReportMsg =
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 16 direct dependencies, including `llvm/Support/PrettyStackTrace.h`, `llvm-c/ErrorHandling.h`, `llvm/Config/config.h`, `llvm/Support/Compiler.h`.
  CN: 引入了 16 个直接依赖，其中包括 `llvm/Support/PrettyStackTrace.h`, `llvm-c/ErrorHandling.h`, `llvm/Config/config.h`, `llvm/Support/Compiler.h`。
- EN: This range propagates LLVM-style errors and invariants.
  CN: 这一段传播 LLVM 风格的错误处理并维护不变量。

### Lines 41-80

```cpp
    "PLEASE submit a bug report to " BUG_REPORT_URL
    " and include the crash backtrace and instructions to reproduce the bug.\n";

// If backtrace support is not enabled, compile out support for pretty stack
// traces.  This has the secondary effect of not requiring thread local storage
// when backtrace support is disabled.
#if ENABLE_BACKTRACES

// We need a thread local pointer to manage the stack of our stack trace
// objects, but we *really* cannot tolerate destructors running and do not want
// to pay any overhead of synchronizing. As a consequence, we use a raw
// thread-local variable.
static LLVM_THREAD_LOCAL PrettyStackTraceEntry *PrettyStackTraceHead = nullptr;

// The use of 'volatile' here is to ensure that any particular thread always
// reloads the value of the counter. The 'std::atomic' allows us to specify that
// this variable is accessed in an unsychronized way (it's not actually
// synchronizing). This does technically mean that the value may not appear to
// be the same across threads running simultaneously on different CPUs, but in
// practice the worst that will happen is that we won't print a stack trace when
// we could have.
//
// This is initialized to 1 because 0 is used as a sentinel for "not enabled on
// the current thread". If the user happens to overflow an 'unsigned' with
// SIGINFO requests, it's possible that some threads will stop responding to it,
// but the program won't crash.
static volatile std::atomic<unsigned> GlobalSigInfoGenerationCounter = 1;
static LLVM_THREAD_LOCAL unsigned ThreadLocalSigInfoGenerationCounter = 0;

namespace llvm {
PrettyStackTraceEntry *ReverseStackTrace(PrettyStackTraceEntry *Head) {
  PrettyStackTraceEntry *Prev = nullptr;
  while (Head)
    std::tie(Prev, Head, Head->NextEntry) =
        std::make_tuple(Head, Head->NextEntry, Prev);
  return Prev;
}
} // namespace llvm

static void PrintStack(raw_ostream &OS) {
```
- EN: This section centers on `PrintStack` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `PrintStack` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code iterates over collections, ranges, or records and returns the resulting value to its callers.
  CN: 在这一段中，代码遍历集合、区间或记录，并将结果返回给调用方。

### Lines 81-120

```cpp
  // Print out the stack in reverse order. To avoid recursion (which is likely
  // to fail if we crashed due to stack overflow), we do an up-front pass to
  // reverse the stack, then print it, then reverse it again.
  unsigned ID = 0;
  SaveAndRestore<PrettyStackTraceEntry *> SavedStack{PrettyStackTraceHead,
                                                     nullptr};
  PrettyStackTraceEntry *ReversedStack = ReverseStackTrace(SavedStack.get());
  for (const PrettyStackTraceEntry *Entry = ReversedStack; Entry;
       Entry = Entry->getNextEntry()) {
    OS << ID++ << ".\t";
    sys::Watchdog W(5);
    Entry->print(OS);
  }
  llvm::ReverseStackTrace(ReversedStack);
}

/// Print the current stack trace to the specified stream.
///
/// Marked NOINLINE so it can be called from debuggers.
LLVM_ATTRIBUTE_NOINLINE
static void PrintCurStackTrace(raw_ostream &OS) {
  // Don't print an empty trace.
  if (!PrettyStackTraceHead) return;

  // If there are pretty stack frames registered, walk and emit them.
  OS << "Stack dump:\n";

  PrintStack(OS);
  OS.flush();
}

// Integrate with crash reporter libraries.
#if defined (__APPLE__) && defined(HAVE_CRASHREPORTERCLIENT_H)
//  If any clients of llvm try to link to libCrashReporterClient.a themselves,
//  only one crash info struct will be used.
extern "C" {
#ifdef CRASHREPORTER_ANNOTATIONS_INITIALIZER
// Should be available in CRASHREPORTER_ANNOTATIONS_VERSION > 5
CRASHREPORTER_ANNOTATIONS_INITIALIZER()
#else
```
- EN: This section centers on `W`, `ReverseStackTrace`, `PrintCurStackTrace` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `W`, `ReverseStackTrace`, `PrintCurStackTrace` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 121-160

```cpp
// Older CrashReporter annotations layouts
CRASH_REPORTER_CLIENT_HIDDEN
struct crashreporter_annotations_t gCRAnnotations
    __attribute__((section("__DATA," CRASHREPORTER_ANNOTATIONS_SECTION))) = {
        CRASHREPORTER_ANNOTATIONS_VERSION,
        0,
        0,
        0,
        0,
        0,
        0,
#if CRASHREPORTER_ANNOTATIONS_VERSION > 4
        0
#endif // CRASHREPORTER_ANNOTATIONS_VERSION > 4
};
#endif // CRASHREPORTER_ANNOTATIONS_INITIALIZER
} // extern "C"
#elif defined(__APPLE__) && HAVE_CRASHREPORTER_INFO
extern "C" const char *__crashreporter_info__
    __attribute__((visibility("hidden"))) = 0;
asm(".desc ___crashreporter_info__, 0x10");
#endif

[[maybe_unused]] static void setCrashLogMessage(const char *msg);
static void setCrashLogMessage(const char *msg) {
#ifdef HAVE_CRASHREPORTERCLIENT_H
  (void)CRSetCrashLogMessage(msg);
#elif HAVE_CRASHREPORTER_INFO
  __crashreporter_info__ = msg;
#endif
  // Don't reorder subsequent operations: whatever comes after might crash and
  // we want the system crash handling to see the message we just set.
  std::atomic_signal_fence(std::memory_order_seq_cst);
}

#ifdef __APPLE__
using CrashHandlerString = SmallString<2048>;
using CrashHandlerStringStorage = std::byte[sizeof(CrashHandlerString)];
alignas(CrashHandlerString) static CrashHandlerStringStorage
    crashHandlerStringStorage;
```
- EN: This section centers on `asm`, `setCrashLogMessage`, `atomic_signal_fence` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `asm`, `setCrashLogMessage`, `atomic_signal_fence` 等符号展开，负责实现局部控制流程与状态维护。

### Lines 161-200

```cpp
#endif

/// This callback is run if a fatal signal is delivered to the process, it
/// prints the pretty stack trace.
static void CrashHandler(void *) {
  errs() << BugReportMsg ;

#ifndef __APPLE__
  // On non-apple systems, just emit the crash stack trace to stderr.
  PrintCurStackTrace(errs());
#else
  // Emit the crash stack trace to a SmallString, put it where the system crash
  // handling will find it, and also send it to stderr.
  //
  // The SmallString is fairly large in the hope that we don't allocate (we're
  // handling a fatal signal, something is already pretty wrong, allocation
  // might not work). Further, we don't use a magic static in case that's also
  // borked. We leak any allocation that does occur because the program is about
  // to die anyways. This is technically racy if we were handling two fatal
  // signals, however if we're in that situation a race is the least of our
  // worries.
  auto &crashHandlerString =
      *new (&crashHandlerStringStorage) CrashHandlerString;

  // If we crash while trying to print the stack trace, we still want the system
  // crash handling to have some partial information. That'll work out as long
  // as the SmallString doesn't allocate. If it does allocate then the system
  // crash handling will see some garbage because the inline buffer now contains
  // a pointer.
  setCrashLogMessage(crashHandlerString.c_str());

  {
    raw_svector_ostream Stream(crashHandlerString);
    PrintCurStackTrace(Stream);
  }

  if (!crashHandlerString.empty()) {
    setCrashLogMessage(crashHandlerString.c_str());
    errs() << crashHandlerString.str();
  } else {
```
- EN: This section centers on `CrashHandler`, `PrintCurStackTrace`, `setCrashLogMessage` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `CrashHandler`, `PrintCurStackTrace`, `setCrashLogMessage` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: This range checks conditions and handles edge cases.
  CN: 这一段检查条件并处理边界情况。

### Lines 201-240

```cpp
    setCrashLogMessage("No crash information.");
  }
#endif
}

static void printForSigInfoIfNeeded() {
  unsigned CurrentSigInfoGeneration =
      GlobalSigInfoGenerationCounter.load(std::memory_order_relaxed);
  if (ThreadLocalSigInfoGenerationCounter == 0 ||
      ThreadLocalSigInfoGenerationCounter == CurrentSigInfoGeneration) {
    return;
  }

  PrintCurStackTrace(errs());
  ThreadLocalSigInfoGenerationCounter = CurrentSigInfoGeneration;
}

#endif // ENABLE_BACKTRACES

void llvm::setBugReportMsg(const char *Msg) {
  BugReportMsg = Msg;
}

const char *llvm::getBugReportMsg() {
  return BugReportMsg;
}

PrettyStackTraceEntry::PrettyStackTraceEntry() {
#if ENABLE_BACKTRACES
  // Handle SIGINFO first, because we haven't finished constructing yet.
  printForSigInfoIfNeeded();
  // Link ourselves.
  NextEntry = PrettyStackTraceHead;
  PrettyStackTraceHead = this;
#endif
}

PrettyStackTraceEntry::~PrettyStackTraceEntry() {
#if ENABLE_BACKTRACES
  assert(PrettyStackTraceHead == this &&
```
- EN: This section centers on `setCrashLogMessage`, `printForSigInfoIfNeeded`, `PrintCurStackTrace` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `setCrashLogMessage`, `printForSigInfoIfNeeded`, `PrintCurStackTrace` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 241-280

```cpp
         "Pretty stack trace entry destruction is out of order");
  PrettyStackTraceHead = NextEntry;
  // Handle SIGINFO first, because we already started destructing.
  printForSigInfoIfNeeded();
#endif
}

void PrettyStackTraceString::print(raw_ostream &OS) const { OS << Str << "\n"; }

PrettyStackTraceFormat::PrettyStackTraceFormat(const char *Format, ...) {
  va_list AP;
  va_start(AP, Format);
  const int SizeOrError = vsnprintf(nullptr, 0, Format, AP);
  va_end(AP);
  if (SizeOrError < 0) {
    return;
  }

  const int Size = SizeOrError + 1; // '\0'
  Str.resize(Size);
  va_start(AP, Format);
  vsnprintf(Str.data(), Size, Format, AP);
  va_end(AP);
}

void PrettyStackTraceFormat::print(raw_ostream &OS) const { OS << Str << "\n"; }

void PrettyStackTraceProgram::print(raw_ostream &OS) const {
  OS << "Program arguments: ";
  // Print the argument list.
  for (int I = 0; I < ArgC; ++I) {
    const bool HaveSpace = ::strchr(ArgV[I], ' ');
    if (I)
      OS << ' ';
    if (HaveSpace)
      OS << '"';
    OS.write_escaped(ArgV[I]);
    if (HaveSpace)
      OS << '"';
  }
```
- EN: This section centers on `printForSigInfoIfNeeded`, `print`, `PrettyStackTraceFormat` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `printForSigInfoIfNeeded`, `print`, `PrettyStackTraceFormat` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 281-320

```cpp
  OS << '\n';
}

#if ENABLE_BACKTRACES
static bool RegisterCrashPrinter() {
  sys::AddSignalHandler(CrashHandler, nullptr);
  return false;
}
#endif

void llvm::EnablePrettyStackTrace() {
#if ENABLE_BACKTRACES
  // The first time this is called, we register the crash printer.
  static bool HandlerRegistered = RegisterCrashPrinter();
  (void)HandlerRegistered;
#endif
}

void llvm::EnablePrettyStackTraceOnSigInfoForThisThread(bool ShouldEnable) {
#if ENABLE_BACKTRACES
  if (!ShouldEnable) {
    ThreadLocalSigInfoGenerationCounter = 0;
    return;
  }

  // The first time this is called, we register the SIGINFO handler.
  static bool HandlerRegistered = []{
    sys::SetInfoSignalFunction([]{
      GlobalSigInfoGenerationCounter.fetch_add(1, std::memory_order_relaxed);
    });
    return false;
  }();
  (void)HandlerRegistered;

  // Next, enable it for the current thread.
  ThreadLocalSigInfoGenerationCounter =
      GlobalSigInfoGenerationCounter.load(std::memory_order_relaxed);
#endif
}

```
- EN: This section centers on `RegisterCrashPrinter`, `AddSignalHandler`, `EnablePrettyStackTrace` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `RegisterCrashPrinter`, `AddSignalHandler`, `EnablePrettyStackTrace` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code checks conditions and handles edge cases and returns the resulting value to its callers.
  CN: 在这一段中，代码检查条件并处理边界情况，并将结果返回给调用方。

### Lines 321-338

```cpp
const void *llvm::SavePrettyStackState() {
#if ENABLE_BACKTRACES
  return PrettyStackTraceHead;
#else
  return nullptr;
#endif
}

void llvm::RestorePrettyStackState(const void *Top) {
#if ENABLE_BACKTRACES
  PrettyStackTraceHead =
      static_cast<PrettyStackTraceEntry *>(const_cast<void *>(Top));
#endif
}

void LLVMEnablePrettyStackTrace() {
  EnablePrettyStackTrace();
}
```
- EN: This section centers on `RestorePrettyStackState`, `LLVMEnablePrettyStackTrace`, `EnablePrettyStackTrace` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `RestorePrettyStackState`, `LLVMEnablePrettyStackTrace`, `EnablePrettyStackTrace` 等符号展开，负责实现局部控制流程与状态维护。
- EN: This range returns the resulting value to its callers.
  CN: 这一段将结果返回给调用方。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `will`, `crashreporter_annotations_t`, `PrintStack`, `W`, `ReverseStackTrace`, `PrintCurStackTrace` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Formatting, emission, and outward serialization. / 重点关注格式化、输出与序列化。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/Support/PrettyStackTrace.h`, `llvm/Config/config.h`, `llvm/Support/Compiler.h`, `llvm/Support/SaveAndRestore.h`, `llvm/Support/Signals.h`, `llvm/Support/Watchdog.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/SmallString.h`
- Standard library / 标准库: `atomic`, `cstdio`, `cstring`, `tuple`
- Other/system headers / 其他或系统头文件: `llvm-c/ErrorHandling.h`, `cassert`, `cstdarg`, `CrashReporterClient.h`
- Related symbols / 相关符号: `will`, `crashreporter_annotations_t`, `PrintStack`, `W`, `ReverseStackTrace`, `PrintCurStackTrace`, `asm`
