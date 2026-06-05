# InitLLVM.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/InitLLVM.cpp`
- Repository: `llvm-project`
- Purpose (EN): AIX has restrictive memory soft-limits out-of-box, so raise them if needed.
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `InitLLVM` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===-- InitLLVM.cpp -----------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/Support/InitLLVM.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/Support/AutoConvert.h"
#include "llvm/Support/Error.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/ManagedStatic.h"
#include "llvm/Support/Signals.h"

#ifdef _WIN32
#include "llvm/Support/Windows/WindowsSupport.h"
#endif

#if defined(HAVE_UNISTD_H)
#include <unistd.h>
#else
#ifndef STDIN_FILENO
#define STDIN_FILENO 0
#endif
#ifndef STDOUT_FILENO
#define STDOUT_FILENO 1
#endif
#ifndef STDERR_FILENO
#define STDERR_FILENO 2
#endif
#endif

static void RaiseLimits() {
#ifdef _AIX
  // AIX has restrictive memory soft-limits out-of-box, so raise them if needed.
  auto RaiseLimit = [](int resource) {
    struct rlimit r;
    getrlimit(resource, &r);
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 9 direct dependencies, including `llvm/Support/InitLLVM.h`, `llvm/ADT/StringRef.h`, `llvm/Support/AutoConvert.h`, `llvm/Support/Error.h`.
  CN: 引入了 9 个直接依赖，其中包括 `llvm/Support/InitLLVM.h`, `llvm/ADT/StringRef.h`, `llvm/Support/AutoConvert.h`, `llvm/Support/Error.h`。
- EN: This section centers on `RaiseLimits`, `getrlimit` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `RaiseLimits`, `getrlimit` 等符号展开，负责查询现有状态并返回计算出的元数据。

### Lines 41-80

```cpp

    // Increase the soft limit to the hard limit, if necessary and
    // possible.
    if (r.rlim_cur != RLIM_INFINITY && r.rlim_cur != r.rlim_max) {
      r.rlim_cur = r.rlim_max;
      setrlimit(resource, &r);
    }
  };

  // Address space size.
  RaiseLimit(RLIMIT_AS);
  // Heap size.
  RaiseLimit(RLIMIT_DATA);
  // Stack size.
  RaiseLimit(RLIMIT_STACK);
#ifdef RLIMIT_RSS
  // Resident set size.
  RaiseLimit(RLIMIT_RSS);
#endif
#endif
}

void CleanupStdHandles(void *Cookie) {
  llvm::raw_ostream *Outs = &llvm::outs(), *Errs = &llvm::errs();
  Outs->flush();
  Errs->flush();
  llvm::restoreStdHandleAutoConversion(STDIN_FILENO);
  llvm::restoreStdHandleAutoConversion(STDOUT_FILENO);
  llvm::restoreStdHandleAutoConversion(STDERR_FILENO);
}

using namespace llvm;
using namespace llvm::sys;

InitLLVM::InitLLVM(int &Argc, const char **&Argv,
                   bool InstallPipeSignalExitHandler,
                   bool NeedsPOSIXUtilitySignalHandling) {
#ifndef NDEBUG
  static std::atomic<bool> Initialized{false};
  assert(!Initialized && "InitLLVM was already initialized!");
```
- EN: This section centers on `setrlimit`, `RaiseLimit`, `CleanupStdHandles` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `setrlimit`, `RaiseLimit`, `CleanupStdHandles` 等符号展开，负责创建并初始化辅助对象或状态。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 81-120

```cpp
  Initialized = true;
#endif

  // Bring stdin/stdout/stderr into a known state.
#ifdef _WIN32
  sys::AddSignalHandler(CleanupStdHandles, nullptr);
#else
  sys::AddSignalHandler(CleanupStdHandles, nullptr,
                        NeedsPOSIXUtilitySignalHandling);
#endif

  if (InstallPipeSignalExitHandler)
    // The pipe signal handler must be installed before any other handlers are
    // registered. This is because the Unix \ref RegisterHandlers function does
    // not perform a sigaction() for SIGPIPE unless a one-shot handler is
    // present, to allow long-lived processes (like lldb) to fully opt-out of
    // llvm's SIGPIPE handling and ignore the signal safely.
    sys::SetOneShotPipeSignalFunction(sys::DefaultOneShotPipeSignalHandler);
  // Initialize the stack printer after installing the one-shot pipe signal
  // handler, so we can perform a sigaction() for SIGPIPE on Unix if requested.
  StackPrinter.emplace(Argc, Argv);
  sys::PrintStackTraceOnErrorSignal(Argv[0]);
  install_out_of_memory_new_handler();
  RaiseLimits();

#ifdef __MVS__

  // We use UTF-8 as the internal character encoding. On z/OS, all external
  // output is encoded in EBCDIC. In order to be able to read all
  // error messages, we turn conversion to EBCDIC on for stderr fd.
  std::string Banner = std::string(Argv[0]) + ": ";
  ExitOnError ExitOnErr(Banner);

  // If turning on conversion for stderr fails then the error message
  // may be garbled. There is no solution to this problem.
  ExitOnErr(errorCodeToError(llvm::enableAutoConversion(STDERR_FILENO)));
  ExitOnErr(errorCodeToError(llvm::enableAutoConversion(STDOUT_FILENO)));
#endif

#ifdef _WIN32
```
- EN: This section centers on `AddSignalHandler`, `PrintStackTraceOnErrorSignal`, `install_out_of_memory_new_handler` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `AddSignalHandler`, `PrintStackTraceOnErrorSignal`, `install_out_of_memory_new_handler` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 121-147

```cpp
  // We use UTF-8 as the internal character encoding. On Windows,
  // arguments passed to main() may not be encoded in UTF-8. In order
  // to reliably detect encoding of command line arguments, we use an
  // Windows API to obtain arguments, convert them to UTF-8, and then
  // write them back to the Argv vector.
  //
  // There's probably other way to do the same thing (e.g. using
  // wmain() instead of main()), but this way seems less intrusive
  // than that.
  std::string Banner = std::string(Argv[0]) + ": ";
  ExitOnError ExitOnErr(Banner);

  ExitOnErr(errorCodeToError(windows::GetCommandLineArguments(Args, Alloc)));

  // GetCommandLineArguments doesn't terminate the vector with a
  // nullptr.  Do it to make it compatible with the real argv.
  Args.push_back(nullptr);

  Argc = Args.size() - 1;
  Argv = Args.data();
#endif
}

InitLLVM::~InitLLVM() {
  CleanupStdHandles(nullptr);
  llvm_shutdown();
}
```
- EN: This section centers on `ExitOnErr`, `CleanupStdHandles`, `llvm_shutdown` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `ExitOnErr`, `CleanupStdHandles`, `llvm_shutdown` 等符号展开，负责实现局部控制流程与状态维护。
- EN: This range propagates LLVM-style errors and invariants.
  CN: 这一段传播 LLVM 风格的错误处理并维护不变量。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `rlimit`, `RaiseLimits`, `getrlimit`, `setrlimit`, `RaiseLimit` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Parsing, decoding, and structural validation. / 重点关注解析、解码与结构校验。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/Support/InitLLVM.h`, `llvm/ADT/StringRef.h`, `llvm/Support/AutoConvert.h`, `llvm/Support/Error.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/ManagedStatic.h`, `llvm/Support/Signals.h`, `llvm/Support/Windows/WindowsSupport.h`
- Standard library / 标准库: `unistd.h`
- Other/system headers / 其他或系统头文件: None / 无
- Related symbols / 相关符号: `rlimit`, `RaiseLimits`, `getrlimit`, `setrlimit`, `RaiseLimit`, `CleanupStdHandles`
