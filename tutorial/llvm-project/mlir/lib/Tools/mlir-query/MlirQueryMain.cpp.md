# MlirQueryMain.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Tools/mlir-query/MlirQueryMain.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements the general framework of the MLIR query tool. It parses the command line arguments, parses the MLIR file and outputs the query results.
  - **CN**: 实现构建于 MLIR 库之上的可复用命令行工具支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- MlirQueryMain.cpp - MLIR Query main --------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及高层摘要。

### Lines 8-13
```cpp
//
// This file implements the general framework of the MLIR query tool. It
// parses the command line arguments, parses the MLIR file and outputs the query
// results.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 14-23
```cpp

#include "mlir/Tools/mlir-query/MlirQueryMain.h"
#include "mlir/IR/BuiltinOps.h"
#include "mlir/Parser/Parser.h"
#include "mlir/Query/Query.h"
#include "mlir/Query/QuerySession.h"
#include "mlir/Support/FileUtilities.h"
#include "llvm/LineEditor/LineEditor.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/InitLLVM.h"
```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/Tools/mlir-query/MlirQueryMain.h`, `mlir/IR/BuiltinOps.h`, `mlir/Parser/Parser.h`, `mlir/Query/Query.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/Tools/mlir-query/MlirQueryMain.h`, `mlir/IR/BuiltinOps.h`, `mlir/Parser/Parser.h`, `mlir/Query/Query.h`。

### Lines 24-29
```cpp
#include "llvm/Support/Process.h"
#include "llvm/Support/SourceMgr.h"

//===----------------------------------------------------------------------===//
// Query Parser
//===----------------------------------------------------------------------===//
```
- **EN**: Pulls in the declarations needed by this translation unit, including `llvm/Support/Process.h`, `llvm/Support/SourceMgr.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `llvm/Support/Process.h`, `llvm/Support/SourceMgr.h`。

### Lines 30-34
```cpp

llvm::LogicalResult
mlir::mlirQueryMain(int argc, char **argv, MLIRContext &context,
                    const mlir::query::matcher::Registry &matcherRegistry) {

```
- **EN**: Implements logic around `mlirQueryMain`; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `mlirQueryMain` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 35-39
```cpp
  // Override the default '-h' and use the default PrintHelpMessage() which
  // won't print options in categories.
  static llvm::cl::opt<bool> help("h", llvm::cl::desc("Alias for -help"),
                                  llvm::cl::Hidden);

```
- **EN**: Implements logic around `help`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `help` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 40-45
```cpp
  static llvm::cl::OptionCategory mlirQueryCategory("mlir-query options");

  static llvm::cl::list<std::string> commands(
      "c", llvm::cl::desc("Specify command to run"),
      llvm::cl::value_desc("command"), llvm::cl::cat(mlirQueryCategory));

```
- **EN**: Implements logic around `mlirQueryCategory`, `commands`, `desc`, `value_desc`.
- **CN**: 围绕 `mlirQueryCategory`、`commands`、`desc`、`value_desc` 实现具体逻辑。

### Lines 46-55
```cpp
  static llvm::cl::opt<std::string> inputFilename(
      llvm::cl::Positional, llvm::cl::desc("<input file>"), llvm::cl::init("-"),
      llvm::cl::cat(mlirQueryCategory));

  static llvm::cl::opt<bool> noImplicitModule{
      "no-implicit-module",
      llvm::cl::desc(
          "Disable implicit addition of a top-level module op during parsing"),
      llvm::cl::init(false)};

```
- **EN**: Implements logic around `inputFilename`, `desc`, `cat`, `init`.
- **CN**: 围绕 `inputFilename`、`desc`、`cat`、`init` 实现具体逻辑。

### Lines 56-60
```cpp
  static llvm::cl::opt<bool> allowUnregisteredDialects(
      "allow-unregistered-dialect",
      llvm::cl::desc("Allow operation with no registered dialects"),
      llvm::cl::init(false));

```
- **EN**: Implements logic around `allowUnregisteredDialects`, `desc`, `init`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `allowUnregisteredDialects`、`desc`、`init` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 61-66
```cpp
  llvm::cl::HideUnrelatedOptions(mlirQueryCategory);

  llvm::InitLLVM y(argc, argv);

  llvm::cl::ParseCommandLineOptions(argc, argv, "MLIR test case query tool.\n");

```
- **EN**: Implements logic around `HideUnrelatedOptions`, `y`, `ParseCommandLineOptions`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `HideUnrelatedOptions`、`y`、`ParseCommandLineOptions` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 67-71
```cpp
  if (help) {
    llvm::cl::PrintHelpMessage();
    return mlir::success();
  }

```
- **EN**: Implements logic around `PrintHelpMessage`, `success`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `PrintHelpMessage`、`success` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 72-79
```cpp
  // When reading from stdin and the input is a tty, it is often a user mistake
  // and the process "appears to be stuck". Print a message to let the user
  // know!
  if (inputFilename == "-" &&
      llvm::sys::Process::FileDescriptorIsDisplayed(fileno(stdin)))
    llvm::errs() << "(processing input from stdin now, hit ctrl-c/ctrl-d to "
                    "interrupt)\n";

```
- **EN**: Implements logic around `FileDescriptorIsDisplayed`, `errs`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `FileDescriptorIsDisplayed`、`errs` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 80-87
```cpp
  // Set up the input file.
  std::string errorMessage;
  auto file = openInputFile(inputFilename, &errorMessage);
  if (!file) {
    llvm::errs() << errorMessage << "\n";
    return mlir::failure();
  }

```
- **EN**: Implements logic around `openInputFile`, `errs`, `failure`.
- **CN**: 围绕 `openInputFile`、`errs`、`failure` 实现具体逻辑。

### Lines 88-92
```cpp
  auto sourceMgr = llvm::SourceMgr();
  auto bufferId = sourceMgr.AddNewSourceBuffer(std::move(file), SMLoc());

  context.allowUnregisteredDialects(allowUnregisteredDialects);

```
- **EN**: Implements logic around `SourceMgr`, `AddNewSourceBuffer`, `allowUnregisteredDialects`.
- **CN**: 围绕 `SourceMgr`、`AddNewSourceBuffer`、`allowUnregisteredDialects` 实现具体逻辑。

### Lines 93-99
```cpp
  // Parse the input MLIR file.
  OwningOpRef<Operation *> opRef =
      noImplicitModule ? parseSourceFile(sourceMgr, &context)
                       : parseSourceFile<mlir::ModuleOp>(sourceMgr, &context);
  if (!opRef)
    return mlir::failure();

```
- **EN**: Implements logic around `parseSourceFile`, `ModuleOp>`, `failure`; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `parseSourceFile`、`ModuleOp>`、`failure` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 100-109
```cpp
  mlir::query::QuerySession qs(opRef.get(), sourceMgr, bufferId,
                               matcherRegistry);
  if (!commands.empty()) {
    for (auto &command : commands) {
      mlir::query::QueryRef queryRef = mlir::query::parse(command, qs);
      if (mlir::failed(queryRef->run(llvm::outs(), qs)))
        return mlir::failure();
    }
  } else {
    llvm::LineEditor le("mlir-query");
```
- **EN**: Implements logic around `qs`, `empty`, `parse`, `failed`, and 2 more symbols; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `qs`、`empty`、`parse`、`failed` 等另外 2 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 110-119
```cpp
    le.setListCompleter([&qs](llvm::StringRef line, size_t pos) {
      return mlir::query::complete(line, pos, qs);
    });
    while (std::optional<std::string> line = le.readLine()) {
      mlir::query::QueryRef queryRef = mlir::query::parse(*line, qs);
      (void)queryRef->run(llvm::outs(), qs);
      llvm::outs().flush();
      if (qs.terminate)
        break;
    }
```
- **EN**: Implements logic around `setListCompleter`, `complete`, `readLine`, `parse`, and 2 more symbols; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `setListCompleter`、`complete`、`readLine`、`parse` 等另外 2 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 120-123
```cpp
  }

  return mlir::success();
}
```
- **EN**: Implements logic around `success`.
- **CN**: 围绕 `success` 实现具体逻辑。

## Key Concepts / 关键概念

- **Tooling support / 工具链支持**:
  - **EN**: Builds reusable infrastructure for opt-like tools, translation drivers, and CLIs.
  - **CN**: 为类似 opt 的工具、翻译驱动和命令行程序构建可复用基础设施。
- **SSA and region-based IR / SSA 与基于 Region 的 IR**:
  - **EN**: The file works with MLIR operations, blocks, values, or regions that form MLIR's hierarchical SSA IR.
  - **CN**: 该文件处理 MLIR 操作、块、值或 Region，它们共同构成 MLIR 的层次化 SSA IR。
- **Structured diagnostics / 结构化诊断**:
  - **EN**: Failures are surfaced through `LogicalResult`, diagnostics, or related reporting helpers.
  - **CN**: 通过 `LogicalResult`、诊断对象或相关报告工具显式传播失败。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `mlir/Tools/mlir-query/MlirQueryMain.h`, `mlir/IR/BuiltinOps.h`, `mlir/Parser/Parser.h`, `mlir/Query/Query.h`, `mlir/Query/QuerySession.h`, `mlir/Support/FileUtilities.h`, `llvm/LineEditor/LineEditor.h`, `llvm/Support/CommandLine.h`, `llvm/Support/InitLLVM.h`, `llvm/Support/Process.h` ... (+1 more)
- **Subsystem categories / 子系统类别**: LLVM support-library helpers / LLVM Support 库辅助工具 (4), query infrastructure / 查询基础设施 (2), tooling support declarations / 工具支持声明 (1), core MLIR IR abstractions / 核心 MLIR IR 抽象 (1), MLIR parser declarations / MLIR 解析器声明 (1), shared MLIR support helpers / 共享的 MLIR 支持工具 (1), generic LLVM subsystem support / 通用 LLVM 子系统支持 (1)
