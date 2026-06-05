# Signals.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/Signals.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file defines some helpful functions for dealing with the possibility of Unix signals occurring while your program is running.
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `Signals` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===- Signals.cpp - Signal Handling support --------------------*- C++ -*-===//
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

#include "llvm/Support/Signals.h"

#include "DebugOptions.h"

#include "llvm/ADT/StringRef.h"
#include "llvm/Config/llvm-config.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/ErrorOr.h"
#include "llvm/Support/FileSystem.h"
#include "llvm/Support/FileUtilities.h"
#include "llvm/Support/Format.h"
#include "llvm/Support/FormatVariadic.h"
#include "llvm/Support/IOSandbox.h"
#include "llvm/Support/ManagedStatic.h"
#include "llvm/Support/MemoryBuffer.h"
#include "llvm/Support/Path.h"
#include "llvm/Support/Program.h"
#include "llvm/Support/StringSaver.h"
#include "llvm/Support/raw_ostream.h"
#include <array>
#include <cmath>

//===----------------------------------------------------------------------===//
//=== WARNING: Implementation here must contain only TRULY operating system
//===          independent code.
//===----------------------------------------------------------------------===//

```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 19 direct dependencies, including `llvm/Support/Signals.h`, `DebugOptions.h`, `llvm/ADT/StringRef.h`, `llvm/Config/llvm-config.h`.
  CN: 引入了 19 个直接依赖，其中包括 `llvm/Support/Signals.h`, `DebugOptions.h`, `llvm/ADT/StringRef.h`, `llvm/Config/llvm-config.h`。
- EN: This range propagates LLVM-style errors and invariants.
  CN: 这一段传播 LLVM 风格的错误处理并维护不变量。

### Lines 41-80

```cpp
using namespace llvm;

// Use explicit storage to avoid accessing cl::opt in a signal handler.
static bool DisableSymbolicationFlag = false;
static ManagedStatic<std::string> CrashDiagnosticsDirectory;
namespace {
struct CreateDisableSymbolication {
  static void *call() {
    return new cl::opt<bool, true>(
        "disable-symbolication",
        cl::desc("Disable symbolizing crash backtraces."),
        cl::location(DisableSymbolicationFlag), cl::Hidden);
  }
};
struct CreateCrashDiagnosticsDir {
  static void *call() {
    return new cl::opt<std::string, true>(
        "crash-diagnostics-dir", cl::value_desc("directory"),
        cl::desc("Directory for crash diagnostic files."),
        cl::location(*CrashDiagnosticsDirectory), cl::Hidden);
  }
};
} // namespace
void llvm::initSignalsOptions() {
  static ManagedStatic<cl::opt<bool, true>, CreateDisableSymbolication>
      DisableSymbolication;
  static ManagedStatic<cl::opt<std::string, true>, CreateCrashDiagnosticsDir>
      CrashDiagnosticsDir;
  *DisableSymbolication;
  *CrashDiagnosticsDir;
}

constexpr char DisableSymbolizationEnv[] = "LLVM_DISABLE_SYMBOLIZATION";
constexpr char LLVMSymbolizerPathEnv[] = "LLVM_SYMBOLIZER_PATH";
constexpr char EnableSymbolizerMarkupEnv[] = "LLVM_ENABLE_SYMBOLIZER_MARKUP";

// Callbacks to run in signal handler must be lock-free because a signal handler
// could be running as we add new callbacks. We don't add unbounded numbers of
// callbacks, an array is therefore sufficient.
struct CallbackAndCookie {
```
- EN: This section centers on `desc`, `initSignalsOptions` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `desc`, `initSignalsOptions` 等符号展开，负责创建并初始化辅助对象或状态。
- EN: This range returns the resulting value to its callers.
  CN: 这一段将结果返回给调用方。

### Lines 81-120

```cpp
  sys::SignalHandlerCallback Callback;
  void *Cookie;
  enum class Status { Empty, Initializing, Initialized, Executing };
  std::atomic<Status> Flag;
};

static constexpr size_t MaxSignalHandlerCallbacks = 8;

// A global array of CallbackAndCookie may not compile with
// -Werror=global-constructors in c++20 and above
static std::array<CallbackAndCookie, MaxSignalHandlerCallbacks> &
CallBacksToRun() {
  static std::array<CallbackAndCookie, MaxSignalHandlerCallbacks> callbacks;
  return callbacks;
}

// Signal-safe.
void sys::RunSignalHandlers() {
  // Let's not interfere with stack trace symbolication and friends.
  auto BypassSandbox = sandbox::scopedDisable();

  for (CallbackAndCookie &RunMe : CallBacksToRun()) {
    auto Expected = CallbackAndCookie::Status::Initialized;
    auto Desired = CallbackAndCookie::Status::Executing;
    if (!RunMe.Flag.compare_exchange_strong(Expected, Desired))
      continue;
    (*RunMe.Callback)(RunMe.Cookie);
    RunMe.Callback = nullptr;
    RunMe.Cookie = nullptr;
    RunMe.Flag.store(CallbackAndCookie::Status::Empty);
  }
}

// Signal-safe.
static void insertSignalHandler(sys::SignalHandlerCallback FnPtr,
                                void *Cookie) {
  for (CallbackAndCookie &SetMe : CallBacksToRun()) {
    auto Expected = CallbackAndCookie::Status::Empty;
    auto Desired = CallbackAndCookie::Status::Initializing;
    if (!SetMe.Flag.compare_exchange_strong(Expected, Desired))
```
- EN: This section centers on `CallBacksToRun`, `RunSignalHandlers`, `insertSignalHandler` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `CallBacksToRun`, `RunSignalHandlers`, `insertSignalHandler` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 121-160

```cpp
      continue;
    SetMe.Callback = FnPtr;
    SetMe.Cookie = Cookie;
    SetMe.Flag.store(CallbackAndCookie::Status::Initialized);
    return;
  }
  report_fatal_error("too many signal callbacks already registered");
}

static bool findModulesAndOffsets(void **StackTrace, int Depth,
                                  const char **Modules, intptr_t *Offsets,
                                  const char *MainExecutableName,
                                  StringSaver &StrPool);

/// Format a pointer value as hexadecimal. Zero pad it out so its always the
/// same width.
static FormattedNumber format_ptr(void *PC) {
  // Each byte is two hex digits plus 2 for the 0x prefix.
  unsigned PtrWidth = 2 + 2 * sizeof(void *);
  return format_hex((uint64_t)PC, PtrWidth);
}

/// Reads a file \p Filename written by llvm-symbolizer containing function
/// names and source locations for the addresses in \p AddressList and returns
/// the strings in a vector of pairs, where the first pair element is the index
/// of the corresponding entry in AddressList and the second is the symbolized
/// frame, in a format based on the sanitizer stack trace printer, with the
/// exception that it does not write out frame numbers (i.e. "#2 " for the
/// third address), as it is not assumed that \p AddressList corresponds to a
/// single stack trace.
/// There may be multiple returned entries for a single \p AddressList entry if
/// that frame address corresponds to one or more inlined frames; in this case,
/// all frames for an address will appear contiguously and in-order.
std::optional<SmallVector<std::pair<unsigned, std::string>, 0>>
collectAddressSymbols(void **AddressList, unsigned AddressCount,
                      const char *MainExecutableName,
                      const std::string &LLVMSymbolizerPath) {
  BumpPtrAllocator Allocator;
  StringSaver StrPool(Allocator);
  SmallVector<const char *, 0> Modules(AddressCount, nullptr);
```
- EN: This section centers on `report_fatal_error`, `findModulesAndOffsets`, `format_ptr` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `report_fatal_error`, `findModulesAndOffsets`, `format_ptr` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code updates helper containers and temporary state and returns the resulting value to its callers.
  CN: 在这一段中，代码更新辅助容器和临时状态，并将结果返回给调用方。

### Lines 161-200

```cpp
  SmallVector<intptr_t, 0> Offsets(AddressCount, 0);
  if (!findModulesAndOffsets(AddressList, AddressCount, Modules.data(),
                             Offsets.data(), MainExecutableName, StrPool))
    return {};
  int InputFD;
  SmallString<32> InputFile, OutputFile;
  sys::fs::createTemporaryFile("symbolizer-input", "", InputFD, InputFile);
  sys::fs::createTemporaryFile("symbolizer-output", "", OutputFile);
  FileRemover InputRemover(InputFile.c_str());
  FileRemover OutputRemover(OutputFile.c_str());

  {
    raw_fd_ostream Input(InputFD, true);
    for (unsigned AddrIdx = 0; AddrIdx < AddressCount; AddrIdx++) {
      if (Modules[AddrIdx])
        Input << Modules[AddrIdx] << " " << (void *)Offsets[AddrIdx] << "\n";
    }
  }

  std::optional<StringRef> Redirects[] = {InputFile.str(), OutputFile.str(),
                                          StringRef("")};
  StringRef Args[] = {"llvm-symbolizer", "--functions=linkage", "--inlining",
#ifdef _WIN32
                      // Pass --relative-address on Windows so that we don't
                      // have to add ImageBase from PE file.
                      // FIXME: Make this the default for llvm-symbolizer.
                      "--relative-address",
#endif
                      "--demangle"};
  int RunResult =
      sys::ExecuteAndWait(LLVMSymbolizerPath, Args, std::nullopt, Redirects);
  if (RunResult != 0)
    return {};

  SmallVector<std::pair<unsigned, std::string>, 0> Result;
  auto OutputBuf = MemoryBuffer::getFile(OutputFile.c_str());
  if (!OutputBuf)
    return {};
  StringRef Output = OutputBuf.get()->getBuffer();
  SmallVector<StringRef, 32> Lines;
```
- EN: This section centers on `Offsets`, `createTemporaryFile`, `InputRemover` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `Offsets`, `createTemporaryFile`, `InputRemover` 等符号展开，负责创建并初始化辅助对象或状态。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 201-240

```cpp
  Output.split(Lines, "\n");
  auto *CurLine = Lines.begin();
  // Lines contains the output from llvm-symbolizer, which should contain for
  // each address with a module in order of appearance, one or more lines
  // containing the function name and line associated with that address,
  // followed by an empty line.
  // For each address, adds an output entry for every real or inlined frame at
  // that address. For addresses without known modules, we have a single entry
  // containing just the formatted address; for all other output entries, we
  // output the function entry if it is known, and either the line number if it
  // is known or the module+address offset otherwise.
  for (unsigned AddrIdx = 0; AddrIdx < AddressCount; AddrIdx++) {
    if (!Modules[AddrIdx]) {
      auto &SymbolizedFrame = Result.emplace_back(std::make_pair(AddrIdx, ""));
      raw_string_ostream OS(SymbolizedFrame.second);
      OS << format_ptr(AddressList[AddrIdx]);
      continue;
    }
    // Read pairs of lines (function name and file/line info) until we
    // encounter empty line.
    for (;;) {
      if (CurLine == Lines.end())
        return {};
      StringRef FunctionName = *CurLine++;
      if (FunctionName.empty())
        break;
      auto &SymbolizedFrame = Result.emplace_back(std::make_pair(AddrIdx, ""));
      raw_string_ostream OS(SymbolizedFrame.second);
      OS << format_ptr(AddressList[AddrIdx]) << ' ';
      if (!FunctionName.starts_with("??"))
        OS << FunctionName << ' ';
      if (CurLine == Lines.end())
        return {};
      StringRef FileLineInfo = *CurLine++;
      if (!FileLineInfo.starts_with("??")) {
        OS << FileLineInfo;
      } else {
        OS << "(" << Modules[AddrIdx] << '+' << format_hex(Offsets[AddrIdx], 0)
           << ")";
      }
```
- EN: This section centers on `OS`, `format_ptr` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `OS`, `format_ptr` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 241-280

```cpp
    }
  }
  return Result;
}

ErrorOr<std::string> getLLVMSymbolizerPath(StringRef Argv0 = {}) {
  ErrorOr<std::string> LLVMSymbolizerPathOrErr = std::error_code();
  if (const char *Path = getenv(LLVMSymbolizerPathEnv)) {
    LLVMSymbolizerPathOrErr = sys::findProgramByName(Path);
  } else if (!Argv0.empty()) {
    StringRef Parent = llvm::sys::path::parent_path(Argv0);
    if (!Parent.empty())
      LLVMSymbolizerPathOrErr =
          sys::findProgramByName("llvm-symbolizer", Parent);
  }
  if (!LLVMSymbolizerPathOrErr)
    LLVMSymbolizerPathOrErr = sys::findProgramByName("llvm-symbolizer");
  return LLVMSymbolizerPathOrErr;
}

/// Helper that launches llvm-symbolizer and symbolizes a backtrace.
LLVM_ATTRIBUTE_USED
static bool printSymbolizedStackTrace(StringRef Argv0, void **StackTrace,
                                      int Depth, llvm::raw_ostream &OS) {
  if (DisableSymbolicationFlag || getenv(DisableSymbolizationEnv))
    return false;

  // Don't recursively invoke the llvm-symbolizer binary.
  if (Argv0.contains("llvm-symbolizer"))
    return false;

  // FIXME: Subtract necessary number from StackTrace entries to turn return
  // addresses into actual instruction addresses.
  // Use llvm-symbolizer tool to symbolize the stack traces. First look for it
  // alongside our binary, then in $PATH.
  ErrorOr<std::string> LLVMSymbolizerPathOrErr = getLLVMSymbolizerPath(Argv0);
  if (!LLVMSymbolizerPathOrErr)
    return false;
  const std::string &LLVMSymbolizerPath = *LLVMSymbolizerPathOrErr;

```
- EN: This section centers on `printSymbolizedStackTrace` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `printSymbolizedStackTrace` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 281-320

```cpp
  // If we don't know argv0 or the address of main() at this point, try
  // to guess it anyway (it's possible on some platforms).
  std::string MainExecutableName =
      sys::fs::exists(Argv0) ? std::string(Argv0)
                             : sys::fs::getMainExecutable(nullptr, nullptr);

  auto SymbolizedAddressesOpt = collectAddressSymbols(
      StackTrace, Depth, MainExecutableName.c_str(), LLVMSymbolizerPath);
  if (!SymbolizedAddressesOpt)
    return false;
  for (unsigned FrameNo = 0; FrameNo < SymbolizedAddressesOpt->size();
       ++FrameNo) {
    OS << right_justify(formatv("#{0}", FrameNo).str(), std::log10(Depth) + 2)
       << ' ' << (*SymbolizedAddressesOpt)[FrameNo].second << '\n';
  }
  return true;
}

#if LLVM_ENABLE_DEBUGLOC_TRACKING_ORIGIN
void sys::symbolizeAddresses(AddressSet &Addresses,
                             SymbolizedAddressMap &SymbolizedAddresses) {
  assert(!DisableSymbolicationFlag && !getenv(DisableSymbolizationEnv) &&
         "Debugify origin stacktraces require symbolization to be enabled.");

  // This function deals with temporary files for the purposes of symbolization
  // only, not formal compiler output.
  auto BypassSandbox = sys::sandbox::scopedDisable();

  // Convert Set of Addresses to ordered list.
  SmallVector<void *, 0> AddressList(Addresses.begin(), Addresses.end());
  if (AddressList.empty())
    return;
  llvm::sort(AddressList);

  // Use llvm-symbolizer tool to symbolize the stack traces. First look for it
  // alongside our binary, then in $PATH.
  ErrorOr<std::string> LLVMSymbolizerPathOrErr = getLLVMSymbolizerPath();
  if (!LLVMSymbolizerPathOrErr)
    report_fatal_error("Debugify origin stacktraces require llvm-symbolizer");
  const std::string &LLVMSymbolizerPath = *LLVMSymbolizerPathOrErr;
```
- EN: This section centers on `exists`, `symbolizeAddresses`, `assert` and performs utility computation and state updates.
  CN: 这一段主要围绕 `exists`, `symbolizeAddresses`, `assert` 等符号展开，负责执行工具性计算并更新状态。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 321-360

```cpp

  // Try to guess the main executable name, since we don't have argv0 available
  // here.
  std::string MainExecutableName = sys::fs::getMainExecutable(nullptr, nullptr);

  auto SymbolizedAddressesOpt =
      collectAddressSymbols(AddressList.begin(), AddressList.size(),
                            MainExecutableName.c_str(), LLVMSymbolizerPath);
  if (!SymbolizedAddressesOpt)
    return;
  for (auto SymbolizedFrame : *SymbolizedAddressesOpt) {
    SmallVector<std::string, 0> &SymbolizedAddrs =
        SymbolizedAddresses[AddressList[SymbolizedFrame.first]];
    SymbolizedAddrs.push_back(SymbolizedFrame.second);
  }
  return;
}
#endif

static bool printMarkupContext(raw_ostream &OS, const char *MainExecutableName);

LLVM_ATTRIBUTE_USED
static bool printMarkupStackTrace(StringRef Argv0, void **StackTrace, int Depth,
                                  raw_ostream &OS) {
  const char *Env = getenv(EnableSymbolizerMarkupEnv);
  if (!Env || !*Env)
    return false;

  std::string MainExecutableName =
      sys::fs::exists(Argv0) ? std::string(Argv0)
                             : sys::fs::getMainExecutable(nullptr, nullptr);
  if (!printMarkupContext(OS, MainExecutableName.c_str()))
    return false;
  for (int I = 0; I < Depth; I++)
    OS << format("{{{bt:%d:%#016x}}}\n", I, StackTrace[I]);
  return true;
}

// Include the platform-specific parts of this class.
#ifdef LLVM_ON_UNIX
```
- EN: This section centers on `collectAddressSymbols`, `printMarkupContext`, `printMarkupStackTrace` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `collectAddressSymbols`, `printMarkupContext`, `printMarkupStackTrace` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 361-365

```cpp
#include "Unix/Signals.inc"
#endif
#ifdef _WIN32
#include "Windows/Signals.inc"
#endif
```
- EN: Brings in 2 direct dependencies, including `Unix/Signals.inc`, `Windows/Signals.inc`.
  CN: 引入了 2 个直接依赖，其中包括 `Unix/Signals.inc`, `Windows/Signals.inc`。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `CreateDisableSymbolication`, `CreateCrashDiagnosticsDir`, `CallbackAndCookie`, `desc`, `initSignalsOptions`, `CallBacksToRun`, `RunSignalHandlers` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Parsing, decoding, and structural validation. / 重点关注解析、解码与结构校验。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/Support/Signals.h`, `llvm/ADT/StringRef.h`, `llvm/Config/llvm-config.h`, `llvm/Support/CommandLine.h`, `llvm/Support/ErrorOr.h`, `llvm/Support/FileSystem.h`, `llvm/Support/FileUtilities.h`, `llvm/Support/Format.h`, `llvm/Support/FormatVariadic.h`, `llvm/Support/IOSandbox.h`, `llvm/Support/ManagedStatic.h`, `llvm/Support/MemoryBuffer.h`, `llvm/Support/Path.h`, `llvm/Support/Program.h`, `llvm/Support/StringSaver.h`
- Standard library / 标准库: `array`, `cmath`
- Other/system headers / 其他或系统头文件: `DebugOptions.h`, `Unix/Signals.inc`, `Windows/Signals.inc`
- Related symbols / 相关符号: `CreateDisableSymbolication`, `CreateCrashDiagnosticsDir`, `CallbackAndCookie`, `Status`, `desc`, `initSignalsOptions`, `CallBacksToRun`, `RunSignalHandlers`, `insertSignalHandler`
