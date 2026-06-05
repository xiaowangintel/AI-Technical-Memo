# GraphWriter.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/GraphWriter.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements misc.
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `GraphWriter` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===- GraphWriter.cpp - Implements GraphWriter support routines ----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements misc. GraphWriter support routines.
//
//===----------------------------------------------------------------------===//

#include "llvm/Support/GraphWriter.h"

#include "DebugOptions.h"

#include "llvm/ADT/SmallString.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/Config/config.h"
#include "llvm/Support/Compiler.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/ErrorOr.h"
#include "llvm/Support/FileSystem.h"
#include "llvm/Support/Path.h"
#include "llvm/Support/Program.h"
#include "llvm/Support/raw_ostream.h"

#ifdef __APPLE__
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/ManagedStatic.h"
#endif

#include <string>
#include <system_error>
#include <vector>

using namespace llvm;

#ifdef __APPLE__
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 18 direct dependencies, including `llvm/Support/GraphWriter.h`, `DebugOptions.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/SmallVector.h`.
  CN: 引入了 18 个直接依赖，其中包括 `llvm/Support/GraphWriter.h`, `DebugOptions.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/SmallVector.h`。
- EN: In this range, the code propagates LLVM-style errors and invariants and updates helper containers and temporary state.
  CN: 在这一段中，代码传播 LLVM 风格的错误处理并维护不变量，并更新辅助容器和临时状态。

### Lines 41-80

```cpp
namespace {
struct CreateViewBackground {
  static void *call() {
    return new cl::opt<bool>("view-background", cl::Hidden,
                             cl::desc("Execute graph viewer in the background. "
                                      "Creates tmp file litter."));
  }
};
} // namespace
static ManagedStatic<cl::opt<bool>, CreateViewBackground> ViewBackground;
void llvm::initGraphWriterOptions() { *ViewBackground; }
#else
void llvm::initGraphWriterOptions() {}
#endif

std::string llvm::DOT::EscapeString(const std::string &Label) {
  std::string Str(Label);
  for (unsigned i = 0; i != Str.length(); ++i)
  switch (Str[i]) {
    case '\n':
      Str.insert(Str.begin()+i, '\\');  // Escape character...
      ++i;
      Str[i] = 'n';
      break;
    case '\t':
      Str.insert(Str.begin()+i, ' ');  // Convert to two spaces
      ++i;
      Str[i] = ' ';
      break;
    case '\\':
      if (i+1 != Str.length())
        switch (Str[i+1]) {
          case 'l': continue; // don't disturb \l
          case '|': case '{': case '}':
            Str.erase(Str.begin()+i); continue;
          default: break;
        }
      [[fallthrough]];
    case '{': case '}':
    case '<': case '>':
```
- EN: This section centers on `desc`, `initGraphWriterOptions`, `EscapeString` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `desc`, `initGraphWriterOptions`, `EscapeString` 等符号展开，负责创建并初始化辅助对象或状态。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and iterates over collections, ranges, or records.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并遍历集合、区间或记录。

### Lines 81-120

```cpp
    case '|': case '"':
      Str.insert(Str.begin()+i, '\\');  // Escape character...
      ++i;  // don't infinite loop
      break;
  }
  return Str;
}

/// Get a color string for this node number. Simply round-robin selects
/// from a reasonable number of colors.
StringRef llvm::DOT::getColorString(unsigned ColorNumber) {
  static const int NumColors = 20;
  static const char* Colors[NumColors] = {
    "aaaaaa", "aa0000", "00aa00", "aa5500", "0055ff", "aa00aa", "00aaaa",
    "555555", "ff5555", "55ff55", "ffff55", "5555ff", "ff55ff", "55ffff",
    "ffaaaa", "aaffaa", "ffffaa", "aaaaff", "ffaaff", "aaffff"};
  return Colors[ColorNumber % NumColors];
}

static std::string replaceIllegalFilenameChars(std::string Filename,
                                               const char ReplacementChar) {
  std::string IllegalChars =
      is_style_windows(sys::path::Style::native) ? "\\/:?\"<>|" : "/";

  for (char IllegalChar : IllegalChars)
    llvm::replace(Filename, IllegalChar, ReplacementChar);

  return Filename;
}

std::string llvm::createGraphFilename(const Twine &Name, int &FD) {
  FD = -1;
  SmallString<128> Filename;

  // Windows can't always handle long paths, so limit the length of the name.
  std::string N = Name.str();
  if (N.size() > 140)
    N.resize(140);

  // Replace illegal characters in graph Filename with '_' if needed
```
- EN: This section centers on `getColorString`, `replaceIllegalFilenameChars`, `createGraphFilename` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `getColorString`, `replaceIllegalFilenameChars`, `createGraphFilename` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 121-160

```cpp
  std::string CleansedName = replaceIllegalFilenameChars(N, '_');

  std::error_code EC =
      sys::fs::createTemporaryFile(CleansedName, "dot", FD, Filename);
  if (EC) {
    errs() << "Error: " << EC.message() << "\n";
    return "";
  }

  errs() << "Writing '" << Filename << "'... ";
  return std::string(Filename);
}

// Execute the graph viewer. Return true if there were errors.
static bool ExecGraphViewer(StringRef ExecPath, std::vector<StringRef> &args,
                            StringRef Filename, bool wait,
                            std::string &ErrMsg) {
  if (wait) {
    if (sys::ExecuteAndWait(ExecPath, args, std::nullopt, {}, 0, 0, &ErrMsg)) {
      errs() << "Error: " << ErrMsg << "\n";
      return true;
    }
    sys::fs::remove(Filename);
    errs() << " done. \n";
  } else {
    sys::ExecuteNoWait(ExecPath, args, std::nullopt, {}, 0, &ErrMsg);
    errs() << "Remember to erase graph file: " << Filename << "\n";
  }
  return false;
}

namespace {

struct GraphSession {
  std::string LogBuffer;

  bool TryFindProgram(StringRef Names, std::string &ProgramPath) {
    raw_string_ostream Log(LogBuffer);
    SmallVector<StringRef, 8> parts;
    Names.split(parts, '|');
```
- EN: This section centers on `createTemporaryFile`, `string`, `ExecGraphViewer` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `createTemporaryFile`, `string`, `ExecGraphViewer` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 161-200

```cpp
    for (auto Name : parts) {
      if (ErrorOr<std::string> P = sys::findProgramByName(Name)) {
        ProgramPath = *P;
        return true;
      }
      Log << "  Tried '" << Name << "'\n";
    }
    return false;
  }
};

} // end anonymous namespace

static const char *getProgramName(GraphProgram::Name program) {
  switch (program) {
  case GraphProgram::DOT:
    return "dot";
  case GraphProgram::FDP:
    return "fdp";
  case GraphProgram::NEATO:
    return "neato";
  case GraphProgram::TWOPI:
    return "twopi";
  case GraphProgram::CIRCO:
    return "circo";
  }
  llvm_unreachable("bad kind");
}

bool llvm::DisplayGraph(StringRef FilenameRef, bool wait,
                        GraphProgram::Name program) {
  std::string Filename = std::string(FilenameRef);
  std::string ErrMsg;
  std::string ViewerPath;
  GraphSession S;

#ifdef __APPLE__
  wait &= !*ViewBackground;
  if (S.TryFindProgram("open", ViewerPath)) {
    std::vector<StringRef> args;
```
- EN: This section centers on `llvm_unreachable`, `DisplayGraph` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `llvm_unreachable`, `DisplayGraph` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and iterates over collections, ranges, or records.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并遍历集合、区间或记录。

### Lines 201-240

```cpp
    args.push_back(ViewerPath);
    if (wait)
      args.push_back("-W");
    args.push_back(Filename);
    errs() << "Trying 'open' program... ";
    if (!ExecGraphViewer(ViewerPath, args, Filename, wait, ErrMsg))
      return false;
  }
#endif
  if (S.TryFindProgram("xdg-open", ViewerPath)) {
    std::vector<StringRef> args;
    args.push_back(ViewerPath);
    args.push_back(Filename);
    errs() << "Trying 'xdg-open' program... ";
    if (!ExecGraphViewer(ViewerPath, args, Filename, wait, ErrMsg))
      return false;
  }

  // Graphviz
  if (S.TryFindProgram("Graphviz", ViewerPath)) {
    std::vector<StringRef> args;
    args.push_back(ViewerPath);
    args.push_back(Filename);

    errs() << "Running 'Graphviz' program... ";
    return ExecGraphViewer(ViewerPath, args, Filename, wait, ErrMsg);
  }

  // xdot
  if (S.TryFindProgram("xdot|xdot.py", ViewerPath)) {
    std::vector<StringRef> args;
    args.push_back(ViewerPath);
    args.push_back(Filename);

    args.push_back("-f");
    args.push_back(getProgramName(program));

    errs() << "Running 'xdot.py' program... ";
    return ExecGraphViewer(ViewerPath, args, Filename, wait, ErrMsg);
  }
```
- EN: This section centers on `ExecGraphViewer` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `ExecGraphViewer` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and updates helper containers and temporary state.
  CN: 在这一段中，代码检查条件并处理边界情况，并更新辅助容器和临时状态。

### Lines 241-280

```cpp

  enum ViewerKind {
    VK_None,
    VK_OSXOpen,
    VK_XDGOpen,
    VK_Ghostview,
    VK_CmdStart
  };
  ViewerKind Viewer = VK_None;
#ifdef __APPLE__
  if (!Viewer && S.TryFindProgram("open", ViewerPath))
    Viewer = VK_OSXOpen;
#endif
  if (!Viewer && S.TryFindProgram("gv", ViewerPath))
    Viewer = VK_Ghostview;
  if (!Viewer && S.TryFindProgram("xdg-open", ViewerPath))
    Viewer = VK_XDGOpen;
#ifdef _WIN32
  if (!Viewer && S.TryFindProgram("cmd", ViewerPath)) {
    Viewer = VK_CmdStart;
  }
#endif

  // PostScript or PDF graph generator + PostScript/PDF viewer
  std::string GeneratorPath;
  if (Viewer &&
      (S.TryFindProgram(getProgramName(program), GeneratorPath) ||
       S.TryFindProgram("dot|fdp|neato|twopi|circo", GeneratorPath))) {
    std::string OutputFilename =
        Filename + (Viewer == VK_CmdStart ? ".pdf" : ".ps");

    std::vector<StringRef> args;
    args.push_back(GeneratorPath);
    if (Viewer == VK_CmdStart)
      args.push_back("-Tpdf");
    else
      args.push_back("-Tps");
    args.push_back("-Nfontname=Courier");
    args.push_back("-Gsize=7.5,10");
    args.push_back(Filename);
```
- EN: This range defines or extends data types such as `ViewerKind`.
  CN: 这一段定义或扩展了 `ViewerKind` 等数据类型。
- EN: In this range, the code checks conditions and handles edge cases and updates helper containers and temporary state.
  CN: 在这一段中，代码检查条件并处理边界情况，并更新辅助容器和临时状态。

### Lines 281-320

```cpp
    args.push_back("-o");
    args.push_back(OutputFilename);

    errs() << "Running '" << GeneratorPath << "' program... ";

    if (ExecGraphViewer(GeneratorPath, args, Filename, true, ErrMsg))
      return true;

    // The lifetime of StartArg must include the call of ExecGraphViewer
    // because the args are passed as vector of char*.
    std::string StartArg;

    args.clear();
    args.push_back(ViewerPath);
    switch (Viewer) {
    case VK_OSXOpen:
      args.push_back("-W");
      args.push_back(OutputFilename);
      break;
    case VK_XDGOpen:
      wait = false;
      args.push_back(OutputFilename);
      break;
    case VK_Ghostview:
      args.push_back("--spartan");
      args.push_back(OutputFilename);
      break;
    case VK_CmdStart:
      args.push_back("/S");
      args.push_back("/C");
      StartArg =
          (StringRef("start ") + (wait ? "/WAIT " : "") + OutputFilename).str();
      args.push_back(StartArg);
      break;
    case VK_None:
      llvm_unreachable("Invalid viewer");
    }

    ErrMsg.clear();
    return ExecGraphViewer(ViewerPath, args, OutputFilename, wait, ErrMsg);
```
- EN: This section centers on `llvm_unreachable`, `ExecGraphViewer` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `llvm_unreachable`, `ExecGraphViewer` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and checks conditions and handles edge cases.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并检查条件并处理边界情况。

### Lines 321-340

```cpp
  }

  // dotty
  if (S.TryFindProgram("dotty", ViewerPath)) {
    std::vector<StringRef> args;
    args.push_back(ViewerPath);
    args.push_back(Filename);

// Dotty spawns another app and doesn't wait until it returns
#ifdef _WIN32
    wait = false;
#endif
    errs() << "Running 'dotty' program... ";
    return ExecGraphViewer(ViewerPath, args, Filename, wait, ErrMsg);
  }

  errs() << "Error: Couldn't find a usable graph viewer program:\n";
  errs() << S.LogBuffer << "\n";
  return true;
}
```
- EN: This section centers on `ExecGraphViewer` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `ExecGraphViewer` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `CreateViewBackground`, `GraphSession`, `ViewerKind`, `desc`, `initGraphWriterOptions`, `EscapeString`, `Str` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Helper routines coordinate local state, control flow, and result construction. / 通过辅助例程协调局部状态、控制流与结果构造。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/Support/GraphWriter.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringRef.h`, `llvm/Config/config.h`, `llvm/Support/Compiler.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/ErrorOr.h`, `llvm/Support/FileSystem.h`, `llvm/Support/Path.h`, `llvm/Support/Program.h`, `llvm/Support/raw_ostream.h`, `llvm/Support/CommandLine.h`, `llvm/Support/ManagedStatic.h`
- Standard library / 标准库: `string`, `system_error`, `vector`
- Other/system headers / 其他或系统头文件: `DebugOptions.h`
- Related symbols / 相关符号: `CreateViewBackground`, `GraphSession`, `ViewerKind`, `desc`, `initGraphWriterOptions`, `EscapeString`, `Str`, `getColorString`
