# TableGenLspServerMain.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Tools/tblgen-lsp-server/TableGenLspServerMain.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements reusable command-line tool support built on top of MLIR libraries.
  - **CN**: 实现构建于 MLIR 库之上的可复用命令行工具支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- TableGenLspServerMain.cpp - TableGen Language Server main ----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及高层摘要。

### Lines 8-16
```cpp

#include "mlir/Tools/tblgen-lsp-server/TableGenLspServerMain.h"
#include "LSPServer.h"
#include "TableGenServer.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/LSP/Logging.h"
#include "llvm/Support/LSP/Transport.h"
#include "llvm/Support/Program.h"

```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/Tools/tblgen-lsp-server/TableGenLspServerMain.h`, `LSPServer.h`, `TableGenServer.h`, `llvm/Support/CommandLine.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/Tools/tblgen-lsp-server/TableGenLspServerMain.h`, `LSPServer.h`, `TableGenServer.h`, `llvm/Support/CommandLine.h`。

### Lines 17-23
```cpp
using namespace mlir;
using namespace mlir::lsp;

using llvm::lsp::JSONStreamStyle;
using llvm::lsp::JSONTransport;
using llvm::lsp::Logger;

```
- **EN**: Brings selected namespaces, aliases, or helper symbols into the local scope for the implementation below.
- **CN**: 把选定的命名空间、别名或辅助符号引入当前局部作用域，供后续实现使用。

### Lines 24-33
```cpp
LogicalResult mlir::TableGenLspServerMain(int argc, char **argv) {
  llvm::cl::opt<JSONStreamStyle> inputStyle{
      "input-style",
      llvm::cl::desc("Input JSON stream encoding"),
      llvm::cl::values(clEnumValN(JSONStreamStyle::Standard, "standard",
                                  "usual LSP protocol"),
                       clEnumValN(JSONStreamStyle::Delimited, "delimited",
                                  "messages delimited by `// -----` lines, "
                                  "with // comment support")),
      llvm::cl::init(JSONStreamStyle::Standard),
```
- **EN**: Implements logic around `TableGenLspServerMain`, `desc`, `values`, `clEnumValN`, and 1 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `TableGenLspServerMain`、`desc`、`values`、`clEnumValN` 等另外 1 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 34-43
```cpp
      llvm::cl::Hidden,
  };
  llvm::cl::opt<bool> litTest{
      "lit-test",
      llvm::cl::desc(
          "Abbreviation for -input-style=delimited -pretty -log=verbose. "
          "Intended to simplify lit tests"),
      llvm::cl::init(false),
  };
  llvm::cl::opt<Logger::Level> logLevel{
```
- **EN**: Implements logic around `desc`, `init`.
- **CN**: 围绕 `desc`、`init` 实现具体逻辑。

### Lines 44-53
```cpp
      "log",
      llvm::cl::desc("Verbosity of log messages written to stderr"),
      llvm::cl::values(
          clEnumValN(Logger::Level::Error, "error", "Error messages only"),
          clEnumValN(Logger::Level::Info, "info",
                     "High level execution tracing"),
          clEnumValN(Logger::Level::Debug, "verbose", "Low level details")),
      llvm::cl::init(Logger::Level::Info),
  };
  llvm::cl::opt<bool> prettyPrint{
```
- **EN**: Implements logic around `desc`, `values`, `clEnumValN`, `init`.
- **CN**: 围绕 `desc`、`values`、`clEnumValN`、`init` 实现具体逻辑。

### Lines 54-63
```cpp
      "pretty",
      llvm::cl::desc("Pretty-print JSON output"),
      llvm::cl::init(false),
  };
  llvm::cl::list<std::string> extraIncludeDirs(
      "tablegen-extra-dir", llvm::cl::desc("Extra directory of include files"),
      llvm::cl::value_desc("directory"), llvm::cl::Prefix);
  llvm::cl::list<std::string> compilationDatabases(
      "tablegen-compilation-database",
      llvm::cl::desc("Compilation YAML databases containing additional "
```
- **EN**: Implements logic around `desc`, `init`, `extraIncludeDirs`, `value_desc`, and 1 more symbols; this block parses or prints textual MLIR representations; processes TableGen records or generates derived code.
- **CN**: 围绕 `desc`、`init`、`extraIncludeDirs`、`value_desc` 等另外 1 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示，并处理 TableGen 记录或生成派生代码。

### Lines 64-73
```cpp
                     "compilation information for .td files"));

  llvm::cl::ParseCommandLineOptions(argc, argv, "TableGen LSP Language Server");

  if (litTest) {
    inputStyle = JSONStreamStyle::Delimited;
    logLevel = Logger::Level::Debug;
    prettyPrint = true;
  }

```
- **EN**: Implements logic around `ParseCommandLineOptions`; this block parses or prints textual MLIR representations; processes TableGen records or generates derived code.
- **CN**: 围绕 `ParseCommandLineOptions` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示，并处理 TableGen 记录或生成派生代码。

### Lines 74-80
```cpp
  // Configure the logger.
  Logger::setLogLevel(logLevel);

  // Configure the transport used for communication.
  llvm::sys::ChangeStdinToBinary();
  JSONTransport transport(stdin, llvm::outs(), inputStyle, prettyPrint);

```
- **EN**: Implements logic around `setLogLevel`, `ChangeStdinToBinary`, `transport`.
- **CN**: 围绕 `setLogLevel`、`ChangeStdinToBinary`、`transport` 实现具体逻辑。

### Lines 81-85
```cpp
  // Configure the servers and start the main language server.
  TableGenServer::Options options(compilationDatabases, extraIncludeDirs);
  TableGenServer server(options);
  return runTableGenLSPServer(server, transport);
}
```
- **EN**: Implements logic around `options`, `server`, `runTableGenLSPServer`.
- **CN**: 围绕 `options`、`server`、`runTableGenLSPServer` 实现具体逻辑。

## Key Concepts / 关键概念

- **Tooling support / 工具链支持**:
  - **EN**: Builds reusable infrastructure for opt-like tools, translation drivers, and CLIs.
  - **CN**: 为类似 opt 的工具、翻译驱动和命令行程序构建可复用基础设施。
- **Structured diagnostics / 结构化诊断**:
  - **EN**: Failures are surfaced through `LogicalResult`, diagnostics, or related reporting helpers.
  - **CN**: 通过 `LogicalResult`、诊断对象或相关报告工具显式传播失败。
- **TableGen-driven generation / TableGen 驱动生成**:
  - **EN**: Declarative records are converted into generated MLIR declarations or implementation fragments.
  - **CN**: 把声明式记录转换为生成的 MLIR 声明或实现片段。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `mlir/Tools/tblgen-lsp-server/TableGenLspServerMain.h`, `LSPServer.h`, `TableGenServer.h`, `llvm/Support/CommandLine.h`, `llvm/Support/LSP/Logging.h`, `llvm/Support/LSP/Transport.h`, `llvm/Support/Program.h`
- **Subsystem categories / 子系统类别**: LLVM support-library helpers / LLVM Support 库辅助工具 (4), tooling support declarations / 工具支持声明 (1)
