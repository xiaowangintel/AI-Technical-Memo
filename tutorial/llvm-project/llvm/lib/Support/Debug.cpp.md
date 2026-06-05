# Debug.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/Debug.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements a handy way of adding debugging information to your code, without it being enabled all of the time, and without having to add command line options to enable it.
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `Debug` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===-- Debug.cpp - An easy way to add debug output to your code ----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements a handy way of adding debugging information to your
// code, without it being enabled all of the time, and without having to add
// command line options to enable it.
//
// In particular, just wrap your code with the LLVM_DEBUG() macro, and it will
// be enabled automatically if you specify '-debug' on the command-line.
// Alternatively, you can also use the SET_DEBUG_TYPE("foo") macro to specify
// that your debug code belongs to class "foo".  Then, on the command line, you
// can specify '-debug-only=foo' to enable JUST the debug information for the
// foo class.
//
// When compiling without assertions, the -debug-* options and all code in
// LLVM_DEBUG() statements disappears, so it does not affect the runtime of the
// code.
//
//===----------------------------------------------------------------------===//

#include "llvm/Support/Debug.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/ManagedStatic.h"
#include "llvm/Support/Signals.h"
#include "llvm/Support/circular_raw_ostream.h"
#include "llvm/Support/raw_ostream.h"
#include <utility>

#include "DebugOptions.h"

#undef isCurrentDebugType
#undef setCurrentDebugType
#undef setCurrentDebugTypes

```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 9 direct dependencies, including `llvm/Support/Debug.h`, `llvm/ADT/StringExtras.h`, `llvm/Support/CommandLine.h`, `llvm/Support/ManagedStatic.h`.
  CN: 引入了 9 个直接依赖，其中包括 `llvm/Support/Debug.h`, `llvm/ADT/StringExtras.h`, `llvm/Support/CommandLine.h`, `llvm/Support/ManagedStatic.h`。
- EN: This range propagates LLVM-style errors and invariants.
  CN: 这一段传播 LLVM 风格的错误处理并维护不变量。

### Lines 41-80

```cpp
using namespace llvm;

/// Parse a debug type string into a pair of the debug type and the debug level.
/// The expected format is "type[:level]", where the level is an optional
/// integer.
static std::pair<std::string, std::optional<int>>
parseDebugType(StringRef DbgType) {
  std::optional<int> Level;
  size_t ColonPos = DbgType.find(':');
  if (ColonPos != StringRef::npos) {
    StringRef LevelStr = DbgType.substr(ColonPos + 1);
    DbgType = DbgType.take_front(ColonPos);
    if (LevelStr.empty())
      Level = 0;
    else {
      int parsedLevel;
      if (to_integer(LevelStr, parsedLevel, 10))
        Level = parsedLevel;
    }
  }
  return std::make_pair(DbgType.str(), Level);
}

// Even though LLVM might be built with NDEBUG, define symbols that the code
// built without NDEBUG can depend on via the llvm/Support/Debug.h header.
namespace llvm {
/// Exported boolean set by the -debug option.
bool DebugFlag = false;

/// The current debug type and an optional debug level.
/// The debug level is the verbosity of the debug output.
/// 0 is a special level that acts as an opt-out for this specific debug type.
/// If provided, the debug output is enabled only if the user specified a level
/// at least as high as the provided level.
static ManagedStatic<std::vector<std::pair<std::string, std::optional<int>>>>
    CurrentDebugType;

/// Return true if the specified string is the debug type
/// specified on the command line, or if none was specified on the command line
/// with the -debug-only=X option.
```
- EN: This section centers on `parseDebugType`, `make_pair` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `parseDebugType`, `make_pair` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code checks conditions and handles edge cases and updates helper containers and temporary state.
  CN: 在这一段中，代码检查条件并处理边界情况，并更新辅助容器和临时状态。

### Lines 81-120

```cpp
bool isCurrentDebugType(const char *DebugType, int Level) {
  if (CurrentDebugType->empty())
    return true;
  // Track if there is at least one debug type with a level, this is used
  // to allow to opt-out of some DebugType and leaving all the others enabled.
  bool HasEnabledDebugType = false;
  // See if DebugType is in list. Note: do not use find() as that forces us to
  // unnecessarily create an std::string instance.
  for (auto &D : *CurrentDebugType) {
    HasEnabledDebugType =
        HasEnabledDebugType || (!D.second.has_value() || D.second.value() > 0);
    if (D.first != DebugType)
      continue;
    if (!D.second.has_value())
      return true;
    return D.second >= Level;
  }
  return !HasEnabledDebugType;
}

/// Set the current debug type, as if the -debug-only=X
/// option were specified.  Note that DebugFlag also needs to be set to true for
/// debug output to be produced.
///
void setCurrentDebugTypes(const char **Types, unsigned Count);

void setCurrentDebugType(const char *Type) {
  setCurrentDebugTypes(&Type, 1);
}

void setCurrentDebugTypes(const char **Types, unsigned Count) {
  CurrentDebugType->clear();
  CurrentDebugType->reserve(Count);
  for (const char *Type : ArrayRef(Types, Count))
    CurrentDebugType->push_back(parseDebugType(Type));
}

} // namespace llvm

// All Debug.h functionality is a no-op in NDEBUG mode.
```
- EN: This section centers on `isCurrentDebugType`, `setCurrentDebugTypes`, `setCurrentDebugType` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `isCurrentDebugType`, `setCurrentDebugTypes`, `setCurrentDebugType` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 121-160

```cpp
#ifndef NDEBUG

namespace {
struct CreateDebug {
  static void *call() {
    return new cl::opt<bool, true>("debug", cl::desc("Enable debug output"),
                                   cl::Hidden, cl::location(DebugFlag));
  }
};

// -debug-buffer-size - Buffer the last N characters of debug output
//until program termination.
struct CreateDebugBufferSize {
  static void *call() {
    return new cl::opt<unsigned>(
        "debug-buffer-size",
        cl::desc("Buffer the last N characters of debug output "
                 "until program termination. "
                 "[default 0 -- immediate print-out]"),
        cl::Hidden, cl::init(0));
  }
};
} // namespace

// -debug - Command line option to enable the DEBUG statements in the passes.
// This flag may only be enabled in debug builds.
static ManagedStatic<cl::opt<bool, true>, CreateDebug> Debug;
static ManagedStatic<cl::opt<unsigned>, CreateDebugBufferSize> DebugBufferSize;

namespace {

struct DebugOnlyOpt {
  void operator=(const std::string &Val) const {
    if (Val.empty())
      return;
    DebugFlag = true;
    SmallVector<StringRef, 8> DbgTypes;
    StringRef(Val).split(DbgTypes, ',', -1, false);
    for (auto DbgType : DbgTypes)
      CurrentDebugType->push_back(parseDebugType(DbgType));
```
- EN: This section centers on `location`, `desc`, `StringRef` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `location`, `desc`, `StringRef` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 161-200

```cpp
  }
};
} // namespace

static DebugOnlyOpt DebugOnlyOptLoc;

namespace {
struct CreateDebugOnly {
  static void *call() {
    return new cl::opt<DebugOnlyOpt, true, cl::parser<std::string>>(
        "debug-only",
        cl::desc(
            "Enable a specific type of debug output (comma separated list "
            "of types using the format \"type[:level]\", where the level "
            "is an optional integer. The level can be set to 1, 2, 3, etc. to "
            "control the verbosity of the output. Setting a debug-type level "
            "to zero acts as an opt-out for this specific debug-type without "
            "affecting the others."),
        cl::Hidden, cl::value_desc("debug string"),
        cl::location(DebugOnlyOptLoc), cl::ValueRequired);
  }
};
} // namespace

static ManagedStatic<cl::opt<DebugOnlyOpt, true, cl::parser<std::string>>,
                     CreateDebugOnly>
    DebugOnly;

void llvm::initDebugOptions() {
  *Debug;
  *DebugBufferSize;
  *DebugOnly;
}

static void printDebugLogImpl() {
  // This is a bit sneaky.  Since this is under #ifndef NDEBUG, we
  // know that debug mode is enabled and dbgs() really is a
  // circular_raw_ostream.  If NDEBUG is defined, then dbgs() ==
  // errs() but this will never be invoked.
  llvm::circular_raw_ostream &dbgout =
```
- EN: This section centers on `desc`, `initDebugOptions`, `printDebugLogImpl` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `desc`, `initDebugOptions`, `printDebugLogImpl` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: This range returns the resulting value to its callers.
  CN: 这一段将结果返回给调用方。

### Lines 201-240

```cpp
      static_cast<circular_raw_ostream &>(llvm::dbgs());
  dbgout.flushBufferWithBanner();
}

// Signal handlers - dump debug output on termination.
static void debug_user_sig_handler(void *Cookie) { printDebugLogImpl(); }

/// dbgs - Return a circular-buffered debug stream.
raw_ostream &llvm::dbgs() {
  // Do one-time initialization in a thread-safe way.
  static struct dbgstream {
    circular_raw_ostream strm;

    dbgstream()
        : strm(errs(), "*** Debug Log Output ***\n",
               (!EnableDebugBuffering || !DebugFlag) ? 0 : *DebugBufferSize) {
      if (EnableDebugBuffering && DebugFlag && *DebugBufferSize != 0)
        // TODO: Add a handler for SIGUSER1-type signals so the user can
        // force a debug dump.
        sys::AddSignalHandler(&debug_user_sig_handler, nullptr);
      // Otherwise we've already set the debug stream buffer size to
      // zero, disabling buffering so it will output directly to errs().
    }
  } thestrm;

  return thestrm.strm;
}

void llvm::printDebugLog() {
  if (EnableDebugBuffering && DebugFlag && *DebugBufferSize != 0)
    printDebugLogImpl();
}

#else
// Avoid "has no symbols" warning.
namespace llvm {
  /// dbgs - Return errs().
  raw_ostream &dbgs() {
    return errs();
  }
```
- EN: This section centers on `debug_user_sig_handler`, `dbgstream`, `printDebugLog` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `debug_user_sig_handler`, `dbgstream`, `printDebugLog` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code checks conditions and handles edge cases and returns the resulting value to its callers.
  CN: 在这一段中，代码检查条件并处理边界情况，并将结果返回给调用方。

### Lines 241-249

```cpp
}
void llvm::initDebugOptions() {}

void llvm::printDebugLog() {}
#endif

/// EnableDebugBuffering - Turn on signal handler installation.
///
bool llvm::EnableDebugBuffering = false;
```
- EN: This section centers on `initDebugOptions`, `printDebugLog` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `initDebugOptions`, `printDebugLog` 等符号展开，负责创建并初始化辅助对象或状态。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `CreateDebug`, `CreateDebugBufferSize`, `DebugOnlyOpt`, `parseDebugType`, `make_pair`, `isCurrentDebugType`, `setCurrentDebugTypes` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Parsing, decoding, and structural validation. / 重点关注解析、解码与结构校验。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/Support/Debug.h`, `llvm/ADT/StringExtras.h`, `llvm/Support/CommandLine.h`, `llvm/Support/ManagedStatic.h`, `llvm/Support/Signals.h`, `llvm/Support/circular_raw_ostream.h`, `llvm/Support/raw_ostream.h`
- Standard library / 标准库: `utility`
- Other/system headers / 其他或系统头文件: `DebugOptions.h`
- Related symbols / 相关符号: `CreateDebug`, `CreateDebugBufferSize`, `DebugOnlyOpt`, `CreateDebugOnly`, `dbgstream`, `parseDebugType`, `make_pair`, `isCurrentDebugType`, `setCurrentDebugTypes`, `setCurrentDebugType`
