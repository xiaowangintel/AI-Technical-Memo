# LSPServer.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Tools/tblgen-lsp-server/LSPServer.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements reusable command-line tool support built on top of MLIR libraries.
  - **CN**: 实现构建于 MLIR 库之上的可复用命令行工具支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- LSPServer.cpp - TableGen Language Server ---------------------------===//
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

#include "LSPServer.h"

#include "TableGenServer.h"
#include "llvm/Support/LSP/Logging.h"
#include "llvm/Support/LSP/Protocol.h"
#include "llvm/Support/LSP/Transport.h"
#include <optional>

```
- **EN**: Pulls in the declarations needed by this translation unit, including `LSPServer.h`, `TableGenServer.h`, `llvm/Support/LSP/Logging.h`, `llvm/Support/LSP/Protocol.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `LSPServer.h`, `TableGenServer.h`, `llvm/Support/LSP/Logging.h`, `llvm/Support/LSP/Protocol.h`。

### Lines 17-30
```cpp
using namespace mlir;
using namespace mlir::lsp;

using llvm::lsp::Callback;
using llvm::lsp::DidChangeTextDocumentParams;
using llvm::lsp::DidCloseTextDocumentParams;
using llvm::lsp::DidOpenTextDocumentParams;
using llvm::lsp::DocumentLinkParams;
using llvm::lsp::Hover;
using llvm::lsp::InitializedParams;
using llvm::lsp::InitializeParams;
using llvm::lsp::JSONTransport;
using llvm::lsp::Location;
using llvm::lsp::Logger;
```
- **EN**: Brings selected namespaces, aliases, or helper symbols into the local scope for the implementation below.
- **CN**: 把选定的命名空间、别名或辅助符号引入当前局部作用域，供后续实现使用。

### Lines 31-38
```cpp
using llvm::lsp::MessageHandler;
using llvm::lsp::NoParams;
using llvm::lsp::OutgoingNotification;
using llvm::lsp::PublishDiagnosticsParams;
using llvm::lsp::ReferenceParams;
using llvm::lsp::TextDocumentPositionParams;
using llvm::lsp::TextDocumentSyncKind;

```
- **EN**: Brings selected namespaces, aliases, or helper symbols into the local scope for the implementation below.
- **CN**: 把选定的命名空间、别名或辅助符号引入当前局部作用域，供后续实现使用。

### Lines 39-47
```cpp
//===----------------------------------------------------------------------===//
// LSPServer
//===----------------------------------------------------------------------===//

namespace {
struct LSPServer {
  LSPServer(TableGenServer &server, JSONTransport &transport)
      : server(server), transport(transport) {}

```
- **EN**: Introduces declarations for `LSPServer`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `LSPServer` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 48-55
```cpp
  //===--------------------------------------------------------------------===//
  // Initialization

  void onInitialize(const InitializeParams &params,
                    Callback<llvm::json::Value> reply);
  void onInitialized(const InitializedParams &params);
  void onShutdown(const NoParams &params, Callback<std::nullptr_t> reply);

```
- **EN**: Implements logic around `onInitialize`, `onInitialized`, `onShutdown`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `onInitialize`、`onInitialized`、`onShutdown` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 56-62
```cpp
  //===--------------------------------------------------------------------===//
  // Document Change

  void onDocumentDidOpen(const DidOpenTextDocumentParams &params);
  void onDocumentDidClose(const DidCloseTextDocumentParams &params);
  void onDocumentDidChange(const DidChangeTextDocumentParams &params);

```
- **EN**: Implements logic around `onDocumentDidOpen`, `onDocumentDidClose`, `onDocumentDidChange`.
- **CN**: 围绕 `onDocumentDidOpen`、`onDocumentDidClose`、`onDocumentDidChange` 实现具体逻辑。

### Lines 63-70
```cpp
  //===--------------------------------------------------------------------===//
  // Definitions and References

  void onGoToDefinition(const TextDocumentPositionParams &params,
                        Callback<std::vector<Location>> reply);
  void onReference(const ReferenceParams &params,
                   Callback<std::vector<Location>> reply);

```
- **EN**: Implements logic around `onGoToDefinition`, `onReference`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `onGoToDefinition`、`onReference` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 71-77
```cpp
  //===----------------------------------------------------------------------===//
  // DocumentLink

  void onDocumentLink(const DocumentLinkParams &params,
                      Callback<std::vector<DocumentLink>> reply);

  //===--------------------------------------------------------------------===//
```
- **EN**: Implements logic around `onDocumentLink`.
- **CN**: 围绕 `onDocumentLink` 实现具体逻辑。

### Lines 78-85
```cpp
  // Hover

  void onHover(const TextDocumentPositionParams &params,
               Callback<std::optional<Hover>> reply);

  //===--------------------------------------------------------------------===//
  // Fields
  //===--------------------------------------------------------------------===//
```
- **EN**: Implements logic around `onHover`.
- **CN**: 围绕 `onHover` 实现具体逻辑。

### Lines 86-93
```cpp

  TableGenServer &server;
  JSONTransport &transport;

  /// An outgoing notification used to send diagnostics to the client when they
  /// are ready to be processed.
  OutgoingNotification<PublishDiagnosticsParams> publishDiagnostics;

```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 94-100
```cpp
  /// Used to indicate that the 'shutdown' request was received from the
  /// Language Server client.
  bool shutdownRequestReceived = false;
};
} // namespace

//===----------------------------------------------------------------------===//
```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 101-114
```cpp
// Initialization
//===----------------------------------------------------------------------===//

void LSPServer::onInitialize(const InitializeParams &params,
                             Callback<llvm::json::Value> reply) {
  // Send a response with the capabilities of this server.
  llvm::json::Object serverCaps{
      {"textDocumentSync",
       llvm::json::Object{
           {"openClose", true},
           {"change", (int)TextDocumentSyncKind::Incremental},
           {"save", true},
       }},
      {"definitionProvider", true},
```
- **EN**: Implements logic around `onInitialize`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `onInitialize` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 115-122
```cpp
      {"referencesProvider", true},
      {"documentLinkProvider",
       llvm::json::Object{
           {"resolveProvider", false},
       }},
      {"hoverProvider", true},
  };

```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 123-134
```cpp
  llvm::json::Object result{
      {{"serverInfo", llvm::json::Object{{"name", "tblgen-lsp-server"},
                                         {"version", "0.0.1"}}},
       {"capabilities", std::move(serverCaps)}}};
  reply(std::move(result));
}
void LSPServer::onInitialized(const InitializedParams &) {}
void LSPServer::onShutdown(const NoParams &, Callback<std::nullptr_t> reply) {
  shutdownRequestReceived = true;
  reply(nullptr);
}

```
- **EN**: Implements logic around `move`, `reply`, `onInitialized`, `onShutdown`; this block processes TableGen records or generates derived code.
- **CN**: 围绕 `move`、`reply`、`onInitialized`、`onShutdown` 实现具体逻辑；该代码块处理 TableGen 记录或生成派生代码。

### Lines 135-144
```cpp
//===----------------------------------------------------------------------===//
// Document Change
//===----------------------------------------------------------------------===//

void LSPServer::onDocumentDidOpen(const DidOpenTextDocumentParams &params) {
  PublishDiagnosticsParams diagParams(params.textDocument.uri,
                                      params.textDocument.version);
  server.addDocument(params.textDocument.uri, params.textDocument.text,
                     params.textDocument.version, diagParams.diagnostics);

```
- **EN**: Implements logic around `onDocumentDidOpen`, `diagParams`, `addDocument`.
- **CN**: 围绕 `onDocumentDidOpen`、`diagParams`、`addDocument` 实现具体逻辑。

### Lines 145-153
```cpp
  // Publish any recorded diagnostics.
  publishDiagnostics(diagParams);
}
void LSPServer::onDocumentDidClose(const DidCloseTextDocumentParams &params) {
  std::optional<int64_t> version =
      server.removeDocument(params.textDocument.uri);
  if (!version)
    return;

```
- **EN**: Implements logic around `publishDiagnostics`, `onDocumentDidClose`, `removeDocument`.
- **CN**: 围绕 `publishDiagnostics`、`onDocumentDidClose`、`removeDocument` 实现具体逻辑。

### Lines 154-165
```cpp
  // Empty out the diagnostics shown for this document. This will clear out
  // anything currently displayed by the client for this document (e.g. in the
  // "Problems" pane of VSCode).
  publishDiagnostics(
      PublishDiagnosticsParams(params.textDocument.uri, *version));
}
void LSPServer::onDocumentDidChange(const DidChangeTextDocumentParams &params) {
  PublishDiagnosticsParams diagParams(params.textDocument.uri,
                                      params.textDocument.version);
  server.updateDocument(params.textDocument.uri, params.contentChanges,
                        params.textDocument.version, diagParams.diagnostics);

```
- **EN**: Implements logic around `publishDiagnostics`, `PublishDiagnosticsParams`, `onDocumentDidChange`, `diagParams`, and 1 more symbols.
- **CN**: 围绕 `publishDiagnostics`、`PublishDiagnosticsParams`、`onDocumentDidChange`、`diagParams` 等另外 1 个符号 实现具体逻辑。

### Lines 166-172
```cpp
  // Publish any recorded diagnostics.
  publishDiagnostics(diagParams);
}

//===----------------------------------------------------------------------===//
// Definitions and References
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `publishDiagnostics`.
- **CN**: 围绕 `publishDiagnostics` 实现具体逻辑。

### Lines 173-180
```cpp

void LSPServer::onGoToDefinition(const TextDocumentPositionParams &params,
                                 Callback<std::vector<Location>> reply) {
  std::vector<Location> locations;
  server.getLocationsOf(params.textDocument.uri, params.position, locations);
  reply(std::move(locations));
}

```
- **EN**: Implements logic around `onGoToDefinition`, `getLocationsOf`, `reply`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `onGoToDefinition`、`getLocationsOf`、`reply` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 181-187
```cpp
void LSPServer::onReference(const ReferenceParams &params,
                            Callback<std::vector<Location>> reply) {
  std::vector<Location> locations;
  server.findReferencesOf(params.textDocument.uri, params.position, locations);
  reply(std::move(locations));
}

```
- **EN**: Implements logic around `onReference`, `findReferencesOf`, `reply`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `onReference`、`findReferencesOf`、`reply` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 188-198
```cpp
//===----------------------------------------------------------------------===//
// DocumentLink
//===----------------------------------------------------------------------===//

void LSPServer::onDocumentLink(const DocumentLinkParams &params,
                               Callback<std::vector<DocumentLink>> reply) {
  std::vector<DocumentLink> links;
  server.getDocumentLinks(params.textDocument.uri, links);
  reply(std::move(links));
}

```
- **EN**: Implements logic around `onDocumentLink`, `getDocumentLinks`, `reply`.
- **CN**: 围绕 `onDocumentLink`、`getDocumentLinks`、`reply` 实现具体逻辑。

### Lines 199-207
```cpp
//===----------------------------------------------------------------------===//
// Hover
//===----------------------------------------------------------------------===//

void LSPServer::onHover(const TextDocumentPositionParams &params,
                        Callback<std::optional<Hover>> reply) {
  reply(server.findHover(params.textDocument.uri, params.position));
}

```
- **EN**: Implements logic around `onHover`, `reply`.
- **CN**: 围绕 `onHover`、`reply` 实现具体逻辑。

### Lines 208-216
```cpp
//===----------------------------------------------------------------------===//
// Entry Point
//===----------------------------------------------------------------------===//

LogicalResult mlir::lsp::runTableGenLSPServer(TableGenServer &server,
                                              JSONTransport &transport) {
  LSPServer lspServer(server, transport);
  MessageHandler messageHandler(transport);

```
- **EN**: Implements logic around `runTableGenLSPServer`, `lspServer`, `messageHandler`; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `runTableGenLSPServer`、`lspServer`、`messageHandler` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 217-230
```cpp
  // Initialization
  messageHandler.method("initialize", &lspServer, &LSPServer::onInitialize);
  messageHandler.notification("initialized", &lspServer,
                              &LSPServer::onInitialized);
  messageHandler.method("shutdown", &lspServer, &LSPServer::onShutdown);

  // Document Changes
  messageHandler.notification("textDocument/didOpen", &lspServer,
                              &LSPServer::onDocumentDidOpen);
  messageHandler.notification("textDocument/didClose", &lspServer,
                              &LSPServer::onDocumentDidClose);
  messageHandler.notification("textDocument/didChange", &lspServer,
                              &LSPServer::onDocumentDidChange);

```
- **EN**: Implements logic around `method`, `notification`.
- **CN**: 围绕 `method`、`notification` 实现具体逻辑。

### Lines 231-240
```cpp
  // Definitions and References
  messageHandler.method("textDocument/definition", &lspServer,
                        &LSPServer::onGoToDefinition);
  messageHandler.method("textDocument/references", &lspServer,
                        &LSPServer::onReference);

  // Document Link
  messageHandler.method("textDocument/documentLink", &lspServer,
                        &LSPServer::onDocumentLink);

```
- **EN**: Implements logic around `method`.
- **CN**: 围绕 `method` 实现具体逻辑。

### Lines 241-248
```cpp
  // Hover
  messageHandler.method("textDocument/hover", &lspServer, &LSPServer::onHover);

  // Diagnostics
  lspServer.publishDiagnostics =
      messageHandler.outgoingNotification<PublishDiagnosticsParams>(
          "textDocument/publishDiagnostics");

```
- **EN**: Implements logic around `method`, `outgoingNotification`.
- **CN**: 围绕 `method`、`outgoingNotification` 实现具体逻辑。

### Lines 249-256
```cpp
  // Run the main loop of the transport.
  if (llvm::Error error = transport.run(messageHandler)) {
    Logger::error("Transport error: {0}", error);
    llvm::consumeError(std::move(error));
    return failure();
  }
  return success(lspServer.shutdownRequestReceived);
}
```
- **EN**: Implements logic around `run`, `error`, `consumeError`, `failure`, and 1 more symbols.
- **CN**: 围绕 `run`、`error`、`consumeError`、`failure` 等另外 1 个符号 实现具体逻辑。

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
- **TableGen-driven generation / TableGen 驱动生成**:
  - **EN**: Declarative records are converted into generated MLIR declarations or implementation fragments.
  - **CN**: 把声明式记录转换为生成的 MLIR 声明或实现片段。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `LSPServer.h`, `TableGenServer.h`, `llvm/Support/LSP/Logging.h`, `llvm/Support/LSP/Protocol.h`, `llvm/Support/LSP/Transport.h`
- **Standard-library headers / 标准库头文件**: `<optional>`
- **Subsystem categories / 子系统类别**: LLVM support-library helpers / LLVM Support 库辅助工具 (3)
