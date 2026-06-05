# LSPServer.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Tools/mlir-pdll-lsp-server/LSPServer.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements reusable command-line tool support built on top of MLIR libraries.
  - **CN**: 实现构建于 MLIR 库之上的可复用命令行工具支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10
```cpp
//===- LSPServer.cpp - PDLL Language Server -------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "LSPServer.h"

```
- **EN**: Pulls in the declarations needed by this translation unit, including `LSPServer.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `LSPServer.h`。

### Lines 11-22
```cpp
#include "PDLLServer.h"
#include "Protocol.h"
#include "llvm/Support/LSP/Logging.h"
#include "llvm/Support/LSP/Protocol.h"
#include "llvm/Support/LSP/Transport.h"
#include <optional>

#define DEBUG_TYPE "pdll-lsp-server"

using namespace mlir;
using namespace mlir::lsp;

```
- **EN**: Pulls in the declarations needed by this translation unit, including `PDLLServer.h`, `Protocol.h`, `llvm/Support/LSP/Logging.h`, `llvm/Support/LSP/Protocol.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `PDLLServer.h`, `Protocol.h`, `llvm/Support/LSP/Logging.h`, `llvm/Support/LSP/Protocol.h`。

### Lines 23-42
```cpp
using llvm::lsp::Callback;
using llvm::lsp::CompletionList;
using llvm::lsp::CompletionParams;
using llvm::lsp::DidChangeTextDocumentParams;
using llvm::lsp::DidCloseTextDocumentParams;
using llvm::lsp::DidOpenTextDocumentParams;
using llvm::lsp::DocumentLinkParams;
using llvm::lsp::DocumentSymbol;
using llvm::lsp::DocumentSymbolParams;
using llvm::lsp::Hover;
using llvm::lsp::InitializedParams;
using llvm::lsp::InitializeParams;
using llvm::lsp::InlayHintsParams;
using llvm::lsp::JSONTransport;
using llvm::lsp::Location;
using llvm::lsp::Logger;
using llvm::lsp::MessageHandler;
using llvm::lsp::NoParams;
using llvm::lsp::OutgoingNotification;
using llvm::lsp::PublishDiagnosticsParams;
```
- **EN**: Brings selected namespaces, aliases, or helper symbols into the local scope for the implementation below.
- **CN**: 把选定的命名空间、别名或辅助符号引入当前局部作用域，供后续实现使用。

### Lines 43-52
```cpp
using llvm::lsp::ReferenceParams;
using llvm::lsp::TextDocumentPositionParams;
using llvm::lsp::TextDocumentSyncKind;

//===----------------------------------------------------------------------===//
// LSPServer
//===----------------------------------------------------------------------===//

namespace {
struct LSPServer {
```
- **EN**: Introduces declarations for `LSPServer`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `LSPServer` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 53-63
```cpp
  LSPServer(PDLLServer &server, JSONTransport &transport)
      : server(server), transport(transport) {}

  //===--------------------------------------------------------------------===//
  // Initialization

  void onInitialize(const InitializeParams &params,
                    Callback<llvm::json::Value> reply);
  void onInitialized(const InitializedParams &params);
  void onShutdown(const NoParams &params, Callback<std::nullptr_t> reply);

```
- **EN**: Implements logic around `LSPServer`, `server`, `onInitialize`, `onInitialized`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `LSPServer`、`server`、`onInitialize`、`onInitialized` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 64-73
```cpp
  //===--------------------------------------------------------------------===//
  // Document Change

  void onDocumentDidOpen(const DidOpenTextDocumentParams &params);
  void onDocumentDidClose(const DidCloseTextDocumentParams &params);
  void onDocumentDidChange(const DidChangeTextDocumentParams &params);

  //===--------------------------------------------------------------------===//
  // Definitions and References

```
- **EN**: Implements logic around `onDocumentDidOpen`, `onDocumentDidClose`, `onDocumentDidChange`.
- **CN**: 围绕 `onDocumentDidOpen`、`onDocumentDidClose`、`onDocumentDidChange` 实现具体逻辑。

### Lines 74-84
```cpp
  void onGoToDefinition(const TextDocumentPositionParams &params,
                        Callback<std::vector<Location>> reply);
  void onReference(const ReferenceParams &params,
                   Callback<std::vector<Location>> reply);

  //===----------------------------------------------------------------------===//
  // DocumentLink

  void onDocumentLink(const DocumentLinkParams &params,
                      Callback<std::vector<DocumentLink>> reply);

```
- **EN**: Implements logic around `onGoToDefinition`, `onReference`, `onDocumentLink`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `onGoToDefinition`、`onReference`、`onDocumentLink` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 85-96
```cpp
  //===--------------------------------------------------------------------===//
  // Hover

  void onHover(const TextDocumentPositionParams &params,
               Callback<std::optional<Hover>> reply);

  //===--------------------------------------------------------------------===//
  // Document Symbols

  void onDocumentSymbol(const DocumentSymbolParams &params,
                        Callback<std::vector<DocumentSymbol>> reply);

```
- **EN**: Implements logic around `onHover`, `onDocumentSymbol`.
- **CN**: 围绕 `onHover`、`onDocumentSymbol` 实现具体逻辑。

### Lines 97-108
```cpp
  //===--------------------------------------------------------------------===//
  // Code Completion

  void onCompletion(const CompletionParams &params,
                    Callback<CompletionList> reply);

  //===--------------------------------------------------------------------===//
  // Signature Help

  void onSignatureHelp(const TextDocumentPositionParams &params,
                       Callback<SignatureHelp> reply);

```
- **EN**: Implements logic around `onCompletion`, `onSignatureHelp`.
- **CN**: 围绕 `onCompletion`、`onSignatureHelp` 实现具体逻辑。

### Lines 109-120
```cpp
  //===--------------------------------------------------------------------===//
  // Inlay Hints

  void onInlayHint(const InlayHintsParams &params,
                   Callback<std::vector<InlayHint>> reply);

  //===--------------------------------------------------------------------===//
  // PDLL View Output

  void onPDLLViewOutput(const PDLLViewOutputParams &params,
                        Callback<std::optional<PDLLViewOutputResult>> reply);

```
- **EN**: Implements logic around `onInlayHint`, `onPDLLViewOutput`.
- **CN**: 围绕 `onInlayHint`、`onPDLLViewOutput` 实现具体逻辑。

### Lines 121-131
```cpp
  //===--------------------------------------------------------------------===//
  // Fields
  //===--------------------------------------------------------------------===//

  PDLLServer &server;
  JSONTransport &transport;

  /// An outgoing notification used to send diagnostics to the client when they
  /// are ready to be processed.
  OutgoingNotification<PublishDiagnosticsParams> publishDiagnostics;

```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 132-141
```cpp
  /// Used to indicate that the 'shutdown' request was received from the
  /// Language Server client.
  bool shutdownRequestReceived = false;
};
} // namespace

//===----------------------------------------------------------------------===//
// Initialization
//===----------------------------------------------------------------------===//

```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 142-161
```cpp
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
      {"completionProvider",
       llvm::json::Object{
           {"allCommitCharacters",
            {"\t", "(", ")", "[", "]", "{",  "}", "<", ">",
             ":",  ";", ",", "+", "-", "/",  "*", "%", "^",
             "&",  "#", "?", ".", "=", "\"", "'", "|"}},
           {"resolveProvider", false},
           {"triggerCharacters",
            {".", ">", "(", "{", ",", "<", ":", "[", " ", "\"", "/"}},
       }},
```
- **EN**: Implements logic around `onInitialize`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `onInitialize` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 162-176
```cpp
      {"signatureHelpProvider",
       llvm::json::Object{
           {"triggerCharacters", {"(", ","}},
       }},
      {"definitionProvider", true},
      {"referencesProvider", true},
      {"documentLinkProvider",
       llvm::json::Object{
           {"resolveProvider", false},
       }},
      {"hoverProvider", true},
      {"documentSymbolProvider", true},
      {"inlayHintProvider", true},
  };

```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 177-188
```cpp
  llvm::json::Object result{
      {{"serverInfo", llvm::json::Object{{"name", "mlir-pdll-lsp-server"},
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
- **EN**: Implements logic around `move`, `reply`, `onInitialized`, `onShutdown`.
- **CN**: 围绕 `move`、`reply`、`onInitialized`、`onShutdown` 实现具体逻辑。

### Lines 189-198
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

### Lines 199-218
```cpp
  // Publish any recorded diagnostics.
  publishDiagnostics(diagParams);
}
void LSPServer::onDocumentDidClose(const DidCloseTextDocumentParams &params) {
  std::optional<int64_t> version =
      server.removeDocument(params.textDocument.uri);
  if (!version)
    return;

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
- **EN**: Implements logic around `publishDiagnostics`, `onDocumentDidClose`, `removeDocument`, `PublishDiagnosticsParams`, and 3 more symbols.
- **CN**: 围绕 `publishDiagnostics`、`onDocumentDidClose`、`removeDocument`、`PublishDiagnosticsParams` 等另外 3 个符号 实现具体逻辑。

### Lines 219-234
```cpp

  // Publish any recorded diagnostics.
  publishDiagnostics(diagParams);
}

//===----------------------------------------------------------------------===//
// Definitions and References
//===----------------------------------------------------------------------===//

void LSPServer::onGoToDefinition(const TextDocumentPositionParams &params,
                                 Callback<std::vector<Location>> reply) {
  std::vector<Location> locations;
  server.getLocationsOf(params.textDocument.uri, params.position, locations);
  reply(std::move(locations));
}

```
- **EN**: Implements logic around `publishDiagnostics`, `onGoToDefinition`, `getLocationsOf`, `reply`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `publishDiagnostics`、`onGoToDefinition`、`getLocationsOf`、`reply` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 235-244
```cpp
void LSPServer::onReference(const ReferenceParams &params,
                            Callback<std::vector<Location>> reply) {
  std::vector<Location> locations;
  server.findReferencesOf(params.textDocument.uri, params.position, locations);
  reply(std::move(locations));
}

//===----------------------------------------------------------------------===//
// DocumentLink
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `onReference`, `findReferencesOf`, `reply`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `onReference`、`findReferencesOf`、`reply` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 245-255
```cpp

void LSPServer::onDocumentLink(const DocumentLinkParams &params,
                               Callback<std::vector<DocumentLink>> reply) {
  std::vector<DocumentLink> links;
  server.getDocumentLinks(params.textDocument.uri, links);
  reply(std::move(links));
}

//===----------------------------------------------------------------------===//
// Hover
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `onDocumentLink`, `getDocumentLinks`, `reply`.
- **CN**: 围绕 `onDocumentLink`、`getDocumentLinks`、`reply` 实现具体逻辑。

### Lines 256-265
```cpp

void LSPServer::onHover(const TextDocumentPositionParams &params,
                        Callback<std::optional<Hover>> reply) {
  reply(server.findHover(params.textDocument.uri, params.position));
}

//===----------------------------------------------------------------------===//
// Document Symbols
//===----------------------------------------------------------------------===//

```
- **EN**: Implements logic around `onHover`, `reply`.
- **CN**: 围绕 `onHover`、`reply` 实现具体逻辑。

### Lines 266-275
```cpp
void LSPServer::onDocumentSymbol(const DocumentSymbolParams &params,
                                 Callback<std::vector<DocumentSymbol>> reply) {
  std::vector<DocumentSymbol> symbols;
  server.findDocumentSymbols(params.textDocument.uri, symbols);
  reply(std::move(symbols));
}

//===----------------------------------------------------------------------===//
// Code Completion
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `onDocumentSymbol`, `findDocumentSymbols`, `reply`.
- **CN**: 围绕 `onDocumentSymbol`、`findDocumentSymbols`、`reply` 实现具体逻辑。

### Lines 276-285
```cpp

void LSPServer::onCompletion(const CompletionParams &params,
                             Callback<CompletionList> reply) {
  reply(server.getCodeCompletion(params.textDocument.uri, params.position));
}

//===----------------------------------------------------------------------===//
// Signature Help
//===----------------------------------------------------------------------===//

```
- **EN**: Implements logic around `onCompletion`, `reply`.
- **CN**: 围绕 `onCompletion`、`reply` 实现具体逻辑。

### Lines 286-301
```cpp
void LSPServer::onSignatureHelp(const TextDocumentPositionParams &params,
                                Callback<SignatureHelp> reply) {
  reply(server.getSignatureHelp(params.textDocument.uri, params.position));
}

//===----------------------------------------------------------------------===//
// Inlay Hints
//===----------------------------------------------------------------------===//

void LSPServer::onInlayHint(const InlayHintsParams &params,
                            Callback<std::vector<InlayHint>> reply) {
  std::vector<InlayHint> hints;
  server.getInlayHints(params.textDocument.uri, params.range, hints);
  reply(std::move(hints));
}

```
- **EN**: Implements logic around `onSignatureHelp`, `reply`, `onInlayHint`, `getInlayHints`.
- **CN**: 围绕 `onSignatureHelp`、`reply`、`onInlayHint`、`getInlayHints` 实现具体逻辑。

### Lines 302-311
```cpp
//===----------------------------------------------------------------------===//
// PDLL ViewOutput
//===----------------------------------------------------------------------===//

void LSPServer::onPDLLViewOutput(
    const PDLLViewOutputParams &params,
    Callback<std::optional<PDLLViewOutputResult>> reply) {
  reply(server.getPDLLViewOutput(params.uri, params.kind));
}

```
- **EN**: Implements logic around `onPDLLViewOutput`, `reply`.
- **CN**: 围绕 `onPDLLViewOutput`、`reply` 实现具体逻辑。

### Lines 312-326
```cpp
//===----------------------------------------------------------------------===//
// Entry Point
//===----------------------------------------------------------------------===//

LogicalResult mlir::lsp::runPdllLSPServer(PDLLServer &server,
                                          JSONTransport &transport) {
  LSPServer lspServer(server, transport);
  MessageHandler messageHandler(transport);

  // Initialization
  messageHandler.method("initialize", &lspServer, &LSPServer::onInitialize);
  messageHandler.notification("initialized", &lspServer,
                              &LSPServer::onInitialized);
  messageHandler.method("shutdown", &lspServer, &LSPServer::onShutdown);

```
- **EN**: Implements logic around `runPdllLSPServer`, `lspServer`, `messageHandler`, `method`, and 1 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `runPdllLSPServer`、`lspServer`、`messageHandler`、`method` 等另外 1 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 327-340
```cpp
  // Document Changes
  messageHandler.notification("textDocument/didOpen", &lspServer,
                              &LSPServer::onDocumentDidOpen);
  messageHandler.notification("textDocument/didClose", &lspServer,
                              &LSPServer::onDocumentDidClose);
  messageHandler.notification("textDocument/didChange", &lspServer,
                              &LSPServer::onDocumentDidChange);

  // Definitions and References
  messageHandler.method("textDocument/definition", &lspServer,
                        &LSPServer::onGoToDefinition);
  messageHandler.method("textDocument/references", &lspServer,
                        &LSPServer::onReference);

```
- **EN**: Implements logic around `notification`, `method`.
- **CN**: 围绕 `notification`、`method` 实现具体逻辑。

### Lines 341-351
```cpp
  // Document Link
  messageHandler.method("textDocument/documentLink", &lspServer,
                        &LSPServer::onDocumentLink);

  // Hover
  messageHandler.method("textDocument/hover", &lspServer, &LSPServer::onHover);

  // Document Symbols
  messageHandler.method("textDocument/documentSymbol", &lspServer,
                        &LSPServer::onDocumentSymbol);

```
- **EN**: Implements logic around `method`.
- **CN**: 围绕 `method` 实现具体逻辑。

### Lines 352-363
```cpp
  // Code Completion
  messageHandler.method("textDocument/completion", &lspServer,
                        &LSPServer::onCompletion);

  // Signature Help
  messageHandler.method("textDocument/signatureHelp", &lspServer,
                        &LSPServer::onSignatureHelp);

  // Inlay Hints
  messageHandler.method("textDocument/inlayHint", &lspServer,
                        &LSPServer::onInlayHint);

```
- **EN**: Implements logic around `method`.
- **CN**: 围绕 `method` 实现具体逻辑。

### Lines 364-380
```cpp
  // PDLL ViewOutput
  messageHandler.method("pdll/viewOutput", &lspServer,
                        &LSPServer::onPDLLViewOutput);

  // Diagnostics
  lspServer.publishDiagnostics =
      messageHandler.outgoingNotification<PublishDiagnosticsParams>(
          "textDocument/publishDiagnostics");

  // Run the main loop of the transport.
  if (llvm::Error error = transport.run(messageHandler)) {
    Logger::error("Transport error: {0}", error);
    llvm::consumeError(std::move(error));
    return failure();
  }
  return success(lspServer.shutdownRequestReceived);
}
```
- **EN**: Implements logic around `method`, `outgoingNotification`, `run`, `error`, and 3 more symbols.
- **CN**: 围绕 `method`、`outgoingNotification`、`run`、`error` 等另外 3 个符号 实现具体逻辑。

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

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `LSPServer.h`, `PDLLServer.h`, `Protocol.h`, `llvm/Support/LSP/Logging.h`, `llvm/Support/LSP/Protocol.h`, `llvm/Support/LSP/Transport.h`
- **Standard-library headers / 标准库头文件**: `<optional>`
- **Subsystem categories / 子系统类别**: LLVM support-library helpers / LLVM Support 库辅助工具 (3)
