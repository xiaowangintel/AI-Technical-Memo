# ErrorHandler.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `lld/Common/ErrorHandler.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: ErrorHandler.cpp. It also sits in code that provides shared linker infrastructure used across LLD targets. / 该文件提供各类 LLD 目标共享的链接基础设施。 源码头部说明其职责是：ErrorHandler.cpp。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8

```cpp
//===- ErrorHandler.cpp ---------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 9-21

```cpp
#include "lld/Common/ErrorHandler.h"

#include "lld/Common/CommonLinkerContext.h"
#include "llvm/ADT/Twine.h"
#include "llvm/IR/DiagnosticInfo.h"
#include "llvm/IR/DiagnosticPrinter.h"
#include "llvm/Support/CrashRecoveryContext.h"
#include "llvm/Support/ManagedStatic.h"
#include "llvm/Support/Process.h"
#include "llvm/Support/Program.h"
#include "llvm/Support/raw_ostream.h"
#include <regex>
```

- EN: Pulls in 11 header(s) from local project, LLVM, system dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 11 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 22-30

```cpp
using namespace llvm;
using namespace lld;

static StringRef getSeparator(const Twine &msg) {
  if (StringRef(msg.str()).contains('\n'))
    return "\n";
  return "";
}
```

- EN: Works inside namespace scope `llvm`, `lld` to organize symbols. Declares or implements routines including `getSeparator`. Notable symbols here include `getSeparator`, `llvm`, `lld`.
- CN: 这里位于命名空间 `llvm`, `lld` 中，用于组织符号作用域。这里声明或实现函数，例如 `getSeparator`。这里较值得关注的符号包括 `getSeparator`, `llvm`, `lld`。

### Lines 31-45

```cpp
ErrorHandler::~ErrorHandler() {
  if (cleanupCallback)
    cleanupCallback();
}

void ErrorHandler::initialize(llvm::raw_ostream &stdoutOS,
                              llvm::raw_ostream &stderrOS, bool exitEarly,
                              bool disableOutput) {
  this->stdoutOS = &stdoutOS;
  this->stderrOS = &stderrOS;
  stderrOS.enable_colors(stderrOS.has_colors());
  this->exitEarly = exitEarly;
  this->disableOutput = disableOutput;
}
```

- EN: Declares or implements routines including `ErrorHandler`, `cleanupCallback`. Notable symbols here include `ErrorHandler`, `cleanupCallback`.
- CN: 这里声明或实现函数，例如 `ErrorHandler`, `cleanupCallback`。这里较值得关注的符号包括 `ErrorHandler`, `cleanupCallback`。

### Lines 46-53

```cpp
void ErrorHandler::flushStreams() {
  std::lock_guard<std::mutex> lock(mu);
  outs().flush();
  errs().flush();
}

ErrorHandler &lld::errorHandler() { return context().e; }
```

- EN: Declares or implements routines including `flushStreams`, `lock`, `outs`, `errs`, `errorHandler`. Notable symbols here include `flushStreams`, `lock`, `outs`, `errs`, `errorHandler`.
- CN: 这里声明或实现函数，例如 `flushStreams`, `lock`, `outs`, `errs`, `errorHandler`。这里较值得关注的符号包括 `flushStreams`, `lock`, `outs`, `errs`, `errorHandler`。

### Lines 54-65

```cpp
void lld::error(const Twine &msg) { errorHandler().error(msg); }
void lld::error(const Twine &msg, ErrorTag tag, ArrayRef<StringRef> args) {
  errorHandler().error(msg, tag, args);
}
void lld::fatal(const Twine &msg) { errorHandler().fatal(msg); }
void lld::log(const Twine &msg) { errorHandler().log(msg); }
void lld::message(const Twine &msg, llvm::raw_ostream &s) {
  errorHandler().message(msg, s);
}
void lld::warn(const Twine &msg) { errorHandler().warn(msg); }
uint64_t lld::errorCount() { return errorHandler().errorCount; }
```

- EN: Declares or implements routines including `error`, `errorHandler`, `fatal`, `log`, `message`, and 2 more. Notable symbols here include `error`, `errorHandler`, `fatal`, `log`, `message`, `warn`.
- CN: 这里声明或实现函数，例如 `error`, `errorHandler`, `fatal`, `log`, `message`, and 2 more。这里较值得关注的符号包括 `error`, `errorHandler`, `fatal`, `log`, `message`, `warn`。

### Lines 66-76

```cpp
raw_ostream &lld::outs() {
  ErrorHandler &e = errorHandler();
  return e.outs();
}

raw_ostream &ErrorHandler::outs() {
  if (disableOutput)
    return llvm::nulls();
  return stdoutOS ? *stdoutOS : llvm::outs();
}
```

- EN: Declares or implements routines including `outs`, `errorHandler`. Notable symbols here include `outs`, `errorHandler`.
- CN: 这里声明或实现函数，例如 `outs`, `errorHandler`。这里较值得关注的符号包括 `outs`, `errorHandler`。

### Lines 77-90

```cpp
raw_ostream &ErrorHandler::errs() {
  if (disableOutput)
    return llvm::nulls();
  return stderrOS ? *stderrOS : llvm::errs();
}

void lld::exitLld(int val) {
  if (hasContext()) {
    ErrorHandler &e = errorHandler();
    // Delete any temporary file, while keeping the memory mapping open.
    if (e.outputBuffer)
      e.outputBuffer->discard();
  }
```

- EN: Declares or implements routines including `errs`, `exitLld`, `errorHandler`, `discard`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `errs`, `exitLld`, `errorHandler`, `discard`.
- CN: 这里声明或实现函数，例如 `errs`, `exitLld`, `errorHandler`, `discard`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `errs`, `exitLld`, `errorHandler`, `discard`。

### Lines 91-101

```cpp
  // Re-throw a possible signal or exception once/if it was caught by
  // safeLldMain().
  CrashRecoveryContext::throwIfCrash(val);

  // Dealloc/destroy ManagedStatic variables before calling _exit().
  // In an LTO build, allows us to get the output of -time-passes.
  // Ensures that the thread pool for the parallel algorithms is stopped to
  // avoid intermittent crashes on Windows when exiting.
  if (!CrashRecoveryContext::GetCurrent())
    llvm_shutdown();
```

- EN: Declares or implements routines including `throwIfCrash`, `llvm_shutdown`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `throwIfCrash`, `llvm_shutdown`.
- CN: 这里声明或实现函数，例如 `throwIfCrash`, `llvm_shutdown`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `throwIfCrash`, `llvm_shutdown`。

### Lines 102-110

```cpp
  if (hasContext())
    lld::errorHandler().flushStreams();

  // When running inside safeLldMain(), restore the control flow back to the
  // CrashRecoveryContext. Otherwise simply use _exit(), meanning no cleanup,
  // since we want to avoid further crashes on shutdown.
  llvm::sys::Process::Exit(val, /*NoCleanup=*/true);
}
```

- EN: Declares or implements routines including `errorHandler`, `Exit`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `errorHandler`, `Exit`.
- CN: 这里声明或实现函数，例如 `errorHandler`, `Exit`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `errorHandler`, `Exit`。

### Lines 111-121

```cpp
void lld::diagnosticHandler(const DiagnosticInfo &di) {
  SmallString<128> s;
  raw_svector_ostream os(s);
  DiagnosticPrinterRawOStream dp(os);

  // For an inline asm diagnostic, prepend the module name to get something like
  // "$module <inline asm>:1:5: ".
  if (auto *dism = dyn_cast<DiagnosticInfoSrcMgr>(&di))
    if (dism->isInlineAsmDiag())
      os << dism->getModuleName() << ' ';
```

- EN: Declares or implements routines including `diagnosticHandler`, `os`, `dp`, `getModuleName`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `diagnosticHandler`, `os`, `dp`, `getModuleName`.
- CN: 这里声明或实现函数，例如 `diagnosticHandler`, `os`, `dp`, `getModuleName`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `diagnosticHandler`, `os`, `dp`, `getModuleName`。

### Lines 122-136

```cpp
  di.print(dp);
  switch (di.getSeverity()) {
  case DS_Error:
    error(s);
    break;
  case DS_Warning:
    warn(s);
    break;
  case DS_Remark:
  case DS_Note:
    message(s);
    break;
  }
}
```

- EN: Declares or implements routines including `error`, `warn`, `message`. Notable symbols here include `error`, `warn`, `message`.
- CN: 这里声明或实现函数，例如 `error`, `warn`, `message`。这里较值得关注的符号包括 `error`, `warn`, `message`。

### Lines 137-146

```cpp
void lld::checkError(Error e) {
  handleAllErrors(std::move(e),
                  [&](ErrorInfoBase &eib) { error(eib.message()); });
}

void lld::checkError(ErrorHandler &eh, Error e) {
  handleAllErrors(std::move(e),
                  [&](ErrorInfoBase &eib) { eh.error(eib.message()); });
}
```

- EN: Declares or implements routines including `checkError`, `handleAllErrors`. Notable symbols here include `checkError`, `handleAllErrors`.
- CN: 这里声明或实现函数，例如 `checkError`, `handleAllErrors`。这里较值得关注的符号包括 `checkError`, `handleAllErrors`。

### Lines 147-164

```cpp
// This is for --vs-diagnostics.
//
// Normally, lld's error message starts with argv[0]. Therefore, it usually
// looks like this:
//
//   ld.lld: error: ...
//
// This error message style is unfortunately unfriendly to Visual Studio
// IDE. VS interprets the first word of the first line as an error location
// and make it clickable, thus "ld.lld" in the above message would become a
// clickable text. When you click it, VS opens "ld.lld" executable file with
// a binary editor.
//
// As a workaround, we print out an error location instead of "ld.lld" if
// lld is running in VS diagnostics mode. As a result, error message will
// look like this:
//
//   src/foo.c(35): error: ...
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 165-182

```cpp
//
// This function returns an error location string. An error location is
// extracted from an error message using regexps.
std::string ErrorHandler::getLocation(const Twine &msg) {
  if (!vsDiagnostics)
    return std::string(logName);

  static std::regex regexes[] = {
      std::regex(
          R"(^undefined (?:\S+ )?symbol:.*\n)"
          R"(>>> referenced by .+\((\S+):(\d+)\))"),
      std::regex(
          R"(^undefined (?:\S+ )?symbol:.*\n>>> referenced by (\S+):(\d+))"),
      std::regex(R"(^undefined symbol:.*\n>>> referenced by (.*):)"),
      std::regex(
          R"(^duplicate symbol: .*\n>>> defined in (\S+)\n>>> defined in.*)"),
      std::regex(
          R"(^duplicate symbol: .*\n>>> defined at .+\((\S+):(\d+)\))"),
```

- EN: Declares or implements routines including `getLocation`, `regex`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getLocation`, `regex`.
- CN: 这里声明或实现函数，例如 `getLocation`, `regex`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getLocation`, `regex`。

### Lines 183-195

```cpp
      std::regex(R"(^duplicate symbol: .*\n>>> defined at (\S+):(\d+))"),
      std::regex(
          R"(.*\n>>> defined in .*\n>>> referenced by .+\((\S+):(\d+)\))"),
      std::regex(R"(.*\n>>> defined in .*\n>>> referenced by (\S+):(\d+))"),
      std::regex(R"((\S+):(\d+): unclosed quote)"),
  };

  std::string str = msg.str();
  for (std::regex &re : regexes) {
    std::smatch m;
    if (!std::regex_search(str, m, re))
      continue;
```

- EN: Declares or implements routines including `regex`. Notable symbols here include `regex`.
- CN: 这里声明或实现函数，例如 `regex`。这里较值得关注的符号包括 `regex`。

### Lines 196-204

```cpp
    assert(m.size() == 2 || m.size() == 3);
    if (m.size() == 2)
      return m.str(1);
    return m.str(1) + "(" + m.str(2) + ")";
  }

  return std::string(logName);
}
```

- EN: Declares or implements routines including `assert`. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里较值得关注的符号包括 `assert`。

### Lines 205-222

```cpp
void ErrorHandler::reportDiagnostic(StringRef location, Colors c,
                                    StringRef diagKind, const Twine &msg) {
  SmallString<256> buf;
  raw_svector_ostream os(buf);
  os << sep << location << ": ";
  if (!diagKind.empty()) {
    if (errs().colors_enabled()) {
      os.enable_colors(true);
      os << c << diagKind << ": " << Colors::RESET;
    } else {
      os << diagKind << ": ";
    }
  }
  os << msg << '\n';
  errs() << buf;
  // If msg contains a newline, ensure that the next diagnostic is preceded by
  // a blank line separator.
  sep = getSeparator(msg);
```

- EN: Declares or implements routines including `os`, `errs`, `getSeparator`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `os`, `errs`, `getSeparator`.
- CN: 这里声明或实现函数，例如 `os`, `errs`, `getSeparator`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `os`, `errs`, `getSeparator`。

### Lines 223-231

```cpp
}

void ErrorHandler::log(const Twine &msg) {
  if (!verbose || disableOutput)
    return;
  std::lock_guard<std::mutex> lock(mu);
  reportDiagnostic(logName, Colors::RESET, "", msg);
}
```

- EN: Declares or implements routines including `log`, `lock`, `reportDiagnostic`. Notable symbols here include `log`, `lock`, `reportDiagnostic`.
- CN: 这里声明或实现函数，例如 `log`, `lock`, `reportDiagnostic`。这里较值得关注的符号包括 `log`, `lock`, `reportDiagnostic`。

### Lines 232-239

```cpp
void ErrorHandler::message(const Twine &msg, llvm::raw_ostream &s) {
  if (disableOutput)
    return;
  std::lock_guard<std::mutex> lock(mu);
  s << msg << "\n";
  s.flush();
}
```

- EN: Declares or implements routines including `message`, `lock`. Notable symbols here include `message`, `lock`.
- CN: 这里声明或实现函数，例如 `message`, `lock`。这里较值得关注的符号包括 `message`, `lock`。

### Lines 240-248

```cpp
void ErrorHandler::warn(const Twine &msg) {
  if (fatalWarnings) {
    error(msg);
    return;
  }

  if (suppressWarnings)
    return;
```

- EN: Declares or implements routines including `warn`, `error`. Notable symbols here include `warn`, `error`.
- CN: 这里声明或实现函数，例如 `warn`, `error`。这里较值得关注的符号包括 `warn`, `error`。

### Lines 249-262

```cpp
  std::lock_guard<std::mutex> lock(mu);
  reportDiagnostic(getLocation(msg), Colors::MAGENTA, "warning", msg);
}

void ErrorHandler::error(const Twine &msg) {
  // If Visual Studio-style error message mode is enabled,
  // this particular error is printed out as two errors.
  if (vsDiagnostics) {
    static std::regex re(R"(^(duplicate symbol: .*))"
                         R"((\n>>> defined at \S+:\d+.*\n>>>.*))"
                         R"((\n>>> defined at \S+:\d+.*\n>>>.*))");
    std::string str = msg.str();
    std::smatch m;
```

- EN: Declares or implements routines including `lock`, `reportDiagnostic`, `error`, `re`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `lock`, `reportDiagnostic`, `error`, `re`.
- CN: 这里声明或实现函数，例如 `lock`, `reportDiagnostic`, `error`, `re`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `lock`, `reportDiagnostic`, `error`, `re`。

### Lines 263-273

```cpp
    if (std::regex_match(str, m, re)) {
      error(m.str(1) + m.str(2));
      error(m.str(1) + m.str(3));
      return;
    }
  }

  bool exit = false;
  {
    std::lock_guard<std::mutex> lock(mu);
```

- EN: Declares or implements routines including `error`, `lock`. Notable symbols here include `error`, `lock`.
- CN: 这里声明或实现函数，例如 `error`, `lock`。这里较值得关注的符号包括 `error`, `lock`。

### Lines 274-283

```cpp
    if (errorLimit == 0 || errorCount < errorLimit) {
      reportDiagnostic(getLocation(msg), Colors::RED, "error", msg);
    } else if (errorCount == errorLimit) {
      reportDiagnostic(logName, Colors::RED, "error", errorLimitExceededMsg);
      exit = exitEarly;
    }

    ++errorCount;
  }
```

- EN: Declares or implements routines including `reportDiagnostic`, `if`. Notable symbols here include `reportDiagnostic`, `if`.
- CN: 这里声明或实现函数，例如 `reportDiagnostic`, `if`。这里较值得关注的符号包括 `reportDiagnostic`, `if`。

### Lines 284-301

```cpp
  if (exit)
    exitLld(1);
}

void ErrorHandler::error(const Twine &msg, ErrorTag tag,
                         ArrayRef<StringRef> args) {
  if (errorHandlingScript.empty() || disableOutput) {
    error(msg);
    return;
  }
  SmallVector<StringRef, 4> scriptArgs;
  scriptArgs.push_back(errorHandlingScript);
  switch (tag) {
  case ErrorTag::LibNotFound:
    scriptArgs.push_back("missing-lib");
    break;
  case ErrorTag::SymbolNotFound:
    scriptArgs.push_back("undefined-symbol");
```

- EN: Declares or implements routines including `exitLld`, `error`. Notable symbols here include `exitLld`, `error`.
- CN: 这里声明或实现函数，例如 `exitLld`, `error`。这里较值得关注的符号包括 `exitLld`, `error`。

### Lines 302-316

```cpp
    break;
  }
  scriptArgs.insert(scriptArgs.end(), args.begin(), args.end());
  int res = llvm::sys::ExecuteAndWait(errorHandlingScript, scriptArgs);
  if (res == 0) {
    return error(msg);
  } else {
    // Temporarily disable error limit to make sure the two calls to error(...)
    // only count as one.
    uint64_t currentErrorLimit = errorLimit;
    errorLimit = 0;
    error(msg);
    errorLimit = currentErrorLimit;
    --errorCount;
```

- EN: Declares or implements routines including `ExecuteAndWait`, `error`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `ExecuteAndWait`, `error`.
- CN: 这里声明或实现函数，例如 `ExecuteAndWait`, `error`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `ExecuteAndWait`, `error`。

### Lines 317-332

```cpp
    switch (res) {
    case -1:
      error("error handling script '" + errorHandlingScript +
            "' failed to execute");
      break;
    case -2:
      error("error handling script '" + errorHandlingScript +
            "' crashed or timeout");
      break;
    default:
      error("error handling script '" + errorHandlingScript +
            "' exited with code " + Twine(res));
    }
  }
}
```

- EN: Declares or implements routines including `Twine`. Notable symbols here include `Twine`.
- CN: 这里声明或实现函数，例如 `Twine`。这里较值得关注的符号包括 `Twine`。

### Lines 333-350

```cpp
void ErrorHandler::fatal(const Twine &msg) {
  error(msg);
  exitLld(1);
}

SyncStream::~SyncStream() {
  switch (level) {
  case DiagLevel::None:
    break;
  case DiagLevel::Log:
    e.log(buf);
    break;
  case DiagLevel::Msg:
    e.message(buf, e.outs());
    break;
  case DiagLevel::Warn:
    e.warn(buf);
    break;
```

- EN: Declares or implements routines including `fatal`, `error`, `exitLld`, `SyncStream`. Notable symbols here include `fatal`, `error`, `exitLld`, `SyncStream`.
- CN: 这里声明或实现函数，例如 `fatal`, `error`, `exitLld`, `SyncStream`。这里较值得关注的符号包括 `fatal`, `error`, `exitLld`, `SyncStream`。

### Lines 351-358

```cpp
  case DiagLevel::Err:
    e.error(buf);
    break;
  case DiagLevel::Fatal:
    e.fatal(buf);
    break;
  }
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

## Key Concepts / 关键概念

- `getSeparator`: function or method entry point / 函数或方法入口
- `ErrorHandler`: function or method entry point / 函数或方法入口
- `cleanupCallback`: function or method entry point / 函数或方法入口
- `flushStreams`: function or method entry point / 函数或方法入口
- `lock`: function or method entry point / 函数或方法入口
- `llvm`: namespace scope / 命名空间作用域
- `lld`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `lld/Common/ErrorHandler.h`, `lld/Common/CommonLinkerContext.h`
- LLVM headers / LLVM 头文件: `llvm/ADT/Twine.h`, `llvm/IR/DiagnosticInfo.h`, `llvm/IR/DiagnosticPrinter.h`, `llvm/Support/CrashRecoveryContext.h`, `llvm/Support/ManagedStatic.h`, `llvm/Support/Process.h`, `llvm/Support/Program.h`, `llvm/Support/raw_ostream.h`
- System headers / 系统头文件: `regex`
- Directory context / 目录上下文: `lld/Common` neighbors usually cooperate with this file to provide the surrounding subsystem / `lld/Common` 下的相邻文件通常与本文件协作组成对应子系统
