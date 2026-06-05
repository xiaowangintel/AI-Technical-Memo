# LSPServer.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Tools/mlir-lsp-server/LSPServer.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements reusable command-line tool support built on top of MLIR libraries.
  - **CN**: 实现构建于 MLIR 库之上的可复用命令行工具支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```cpp
//===- LSPServer.cpp - MLIR Language Server -------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "LSPServer.h"
#include "MLIRServer.h"
#include "Protocol.h"
#include "llvm/Support/LSP/Logging.h"
#include "llvm/Support/LSP/Transport.h"
#include <optional>

```
- **EN**: Pulls in the declarations needed by this translation unit, including `LSPServer.h`, `MLIRServer.h`, `Protocol.h`, `llvm/Support/LSP/Logging.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `LSPServer.h`, `MLIRServer.h`, `Protocol.h`, `llvm/Support/LSP/Logging.h`。

### Lines 16-35
```cpp
#define DEBUG_TYPE "mlir-lsp-server"

using namespace mlir;
using namespace mlir::lsp;

using llvm::lsp::Callback;
using llvm::lsp::CodeAction;
using llvm::lsp::CodeActionParams;
using llvm::lsp::CompletionList;
using llvm::lsp::CompletionParams;
using llvm::lsp::DidChangeTextDocumentParams;
using llvm::lsp::DidCloseTextDocumentParams;
using llvm::lsp::DidOpenTextDocumentParams;
using llvm::lsp::DocumentSymbol;
using llvm::lsp::DocumentSymbolParams;
using llvm::lsp::Hover;
using llvm::lsp::InitializedParams;
using llvm::lsp::InitializeParams;
using llvm::lsp::JSONTransport;
using llvm::lsp::Location;
```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 36-47
```cpp
using llvm::lsp::Logger;
using llvm::lsp::MessageHandler;
using llvm::lsp::MLIRConvertBytecodeParams;
using llvm::lsp::MLIRConvertBytecodeResult;
using llvm::lsp::NoParams;
using llvm::lsp::OutgoingNotification;
using llvm::lsp::PublishDiagnosticsParams;
using llvm::lsp::ReferenceParams;
using llvm::lsp::TextDocumentPositionParams;
using llvm::lsp::TextDocumentSyncKind;
using llvm::lsp::URIForFile;

```
- **EN**: Brings selected namespaces, aliases, or helper symbols into the local scope for the implementation below.
- **CN**: 把选定的命名空间、别名或辅助符号引入当前局部作用域，供后续实现使用。

### Lines 48-58
```cpp
//===----------------------------------------------------------------------===//
// LSPServer
//===----------------------------------------------------------------------===//

namespace {
struct LSPServer {
  LSPServer(MLIRServer &server) : server(server) {}

  //===--------------------------------------------------------------------===//
  // Initialization

```
- **EN**: Introduces declarations for `LSPServer`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `LSPServer` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 59-70
```cpp
  void onInitialize(const InitializeParams &params,
                    Callback<llvm::json::Value> reply);
  void onInitialized(const InitializedParams &params);
  void onShutdown(const NoParams &params, Callback<std::nullptr_t> reply);

  //===--------------------------------------------------------------------===//
  // Document Change

  void onDocumentDidOpen(const DidOpenTextDocumentParams &params);
  void onDocumentDidClose(const DidCloseTextDocumentParams &params);
  void onDocumentDidChange(const DidChangeTextDocumentParams &params);

```
- **EN**: Implements logic around `onInitialize`, `onInitialized`, `onShutdown`, `onDocumentDidOpen`, and 2 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `onInitialize`、`onInitialized`、`onShutdown`、`onDocumentDidOpen` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 71-81
```cpp
  //===--------------------------------------------------------------------===//
  // Definitions and References

  void onGoToDefinition(const TextDocumentPositionParams &params,
                        Callback<std::vector<Location>> reply);
  void onReference(const ReferenceParams &params,
                   Callback<std::vector<Location>> reply);

  //===--------------------------------------------------------------------===//
  // Hover

```
- **EN**: Implements logic around `onGoToDefinition`, `onReference`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `onGoToDefinition`、`onReference` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 82-91
```cpp
  void onHover(const TextDocumentPositionParams &params,
               Callback<std::optional<Hover>> reply);

  //===--------------------------------------------------------------------===//
  // Document Symbols

  void onDocumentSymbol(const DocumentSymbolParams &params,
                        Callback<std::vector<DocumentSymbol>> reply);

  //===--------------------------------------------------------------------===//
```
- **EN**: Implements logic around `onHover`, `onDocumentSymbol`.
- **CN**: 围绕 `onHover`、`onDocumentSymbol` 实现具体逻辑。

### Lines 92-102
```cpp
  // Code Completion

  void onCompletion(const CompletionParams &params,
                    Callback<CompletionList> reply);

  //===--------------------------------------------------------------------===//
  // Code Action

  void onCodeAction(const CodeActionParams &params,
                    Callback<llvm::json::Value> reply);

```
- **EN**: Implements logic around `onCompletion`, `onCodeAction`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `onCompletion`、`onCodeAction` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 103-113
```cpp
  //===--------------------------------------------------------------------===//
  // Bytecode

  void onConvertFromBytecode(const MLIRConvertBytecodeParams &params,
                             Callback<MLIRConvertBytecodeResult> reply);
  void onConvertToBytecode(const MLIRConvertBytecodeParams &params,
                           Callback<MLIRConvertBytecodeResult> reply);

  //===--------------------------------------------------------------------===//
  // Fields
  //===--------------------------------------------------------------------===//
```
- **EN**: Implements logic around `onConvertFromBytecode`, `onConvertToBytecode`.
- **CN**: 围绕 `onConvertFromBytecode`、`onConvertToBytecode` 实现具体逻辑。

### Lines 114-126
```cpp

  MLIRServer &server;

  /// An outgoing notification used to send diagnostics to the client when they
  /// are ready to be processed.
  OutgoingNotification<PublishDiagnosticsParams> publishDiagnostics;

  /// Used to indicate that the 'shutdown' request was received from the
  /// Language Server client.
  bool shutdownRequestReceived = false;
};
} // namespace

```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 127-143
```cpp
//===----------------------------------------------------------------------===//
// Initialization
//===----------------------------------------------------------------------===//

void LSPServer::onInitialize(const InitializeParams &params,
                             Callback<llvm::json::Value> reply) {
  // Configure the workspace root if it was provided.
  if (params.rootUri) {
    llvm::Expected<URIForFile> rootURI = URIForFile::fromURI(*params.rootUri);
    if (rootURI)
      server.setWorkspaceRoot(rootURI->file());
    else
      consumeError(rootURI.takeError());
  } else if (params.rootPath) {
    server.setWorkspaceRoot(*params.rootPath);
  }

```
- **EN**: Implements logic around `onInitialize`, `fromURI`, `setWorkspaceRoot`, `consumeError`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `onInitialize`、`fromURI`、`setWorkspaceRoot`、`consumeError` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 144-163
```cpp
  // Send a response with the capabilities of this server.
  llvm::json::Object serverCaps{
      {"textDocumentSync",
       llvm::json::Object{
           {"openClose", true},
           {"change", (int)TextDocumentSyncKind::Full},
           {"save", true},
       }},
      {"completionProvider",
       llvm::json::Object{
           {"allCommitCharacters",
            {
                "\t",
                ";",
                ",",
                ".",
                "=",
            }},
           {"resolveProvider", false},
           {"triggerCharacters",
```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 164-175
```cpp
            {".", "%", "^", "!", "#", "(", ",", "<", ":", "[", " ", "\"", "/"}},
       }},
      {"definitionProvider", true},
      {"referencesProvider", true},
      {"hoverProvider", true},

      // For now we only support documenting symbols when the client supports
      // hierarchical symbols.
      {"documentSymbolProvider",
       params.capabilities.hierarchicalDocumentSymbol},
  };

```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 176-185
```cpp
  // Per LSP, codeActionProvider can be either boolean or CodeActionOptions.
  // CodeActionOptions is only valid if the client supports action literal
  // via textDocument.codeAction.codeActionLiteralSupport.
  serverCaps["codeActionProvider"] =
      params.capabilities.codeActionStructure
          ? llvm::json::Object{{"codeActionKinds",
                                {CodeAction::kQuickFix, CodeAction::kRefactor,
                                 CodeAction::kInfo}}}
          : llvm::json::Value(true);

```
- **EN**: Implements logic around `Value`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `Value` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 186-197
```cpp
  llvm::json::Object result{
      {{"serverInfo",
        llvm::json::Object{{"name", "mlir-lsp-server"}, {"version", "0.0.0"}}},
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

### Lines 198-208
```cpp
//===----------------------------------------------------------------------===//
// Document Change
//===----------------------------------------------------------------------===//

void LSPServer::onDocumentDidOpen(const DidOpenTextDocumentParams &params) {
  PublishDiagnosticsParams diagParams(params.textDocument.uri,
                                      params.textDocument.version);
  server.addOrUpdateDocument(params.textDocument.uri, params.textDocument.text,
                             params.textDocument.version,
                             diagParams.diagnostics);

```
- **EN**: Implements logic around `onDocumentDidOpen`, `diagParams`, `addOrUpdateDocument`.
- **CN**: 围绕 `onDocumentDidOpen`、`diagParams`、`addOrUpdateDocument` 实现具体逻辑。

### Lines 209-228
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
  // TODO: We currently only support full document updates, we should refactor
  // to avoid this.
  if (params.contentChanges.size() != 1)
    return;
```
- **EN**: Implements logic around `publishDiagnostics`, `onDocumentDidClose`, `removeDocument`, `PublishDiagnosticsParams`, and 2 more symbols.
- **CN**: 围绕 `publishDiagnostics`、`onDocumentDidClose`、`removeDocument`、`PublishDiagnosticsParams` 等另外 2 个符号 实现具体逻辑。

### Lines 229-238
```cpp
  PublishDiagnosticsParams diagParams(params.textDocument.uri,
                                      params.textDocument.version);
  server.addOrUpdateDocument(
      params.textDocument.uri, params.contentChanges.front().text,
      params.textDocument.version, diagParams.diagnostics);

  // Publish any recorded diagnostics.
  publishDiagnostics(diagParams);
}

```
- **EN**: Implements logic around `diagParams`, `addOrUpdateDocument`, `front`, `publishDiagnostics`.
- **CN**: 围绕 `diagParams`、`addOrUpdateDocument`、`front`、`publishDiagnostics` 实现具体逻辑。

### Lines 239-249
```cpp
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
- **EN**: Implements logic around `onGoToDefinition`, `getLocationsOf`, `reply`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `onGoToDefinition`、`getLocationsOf`、`reply` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 250-259
```cpp
void LSPServer::onReference(const ReferenceParams &params,
                            Callback<std::vector<Location>> reply) {
  std::vector<Location> locations;
  server.findReferencesOf(params.textDocument.uri, params.position, locations);
  reply(std::move(locations));
}

//===----------------------------------------------------------------------===//
// Hover
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `onReference`, `findReferencesOf`, `reply`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `onReference`、`findReferencesOf`、`reply` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 260-269
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

### Lines 270-279
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

### Lines 280-289
```cpp

void LSPServer::onCompletion(const CompletionParams &params,
                             Callback<CompletionList> reply) {
  reply(server.getCodeCompletion(params.textDocument.uri, params.position));
}

//===----------------------------------------------------------------------===//
// Code Action
//===----------------------------------------------------------------------===//

```
- **EN**: Implements logic around `onCompletion`, `reply`.
- **CN**: 围绕 `onCompletion`、`reply` 实现具体逻辑。

### Lines 290-303
```cpp
void LSPServer::onCodeAction(const CodeActionParams &params,
                             Callback<llvm::json::Value> reply) {
  URIForFile uri = params.textDocument.uri;

  // Check whether a particular CodeActionKind is included in the response.
  auto isKindAllowed = [only(params.context.only)](StringRef kind) {
    if (only.empty())
      return true;
    return llvm::any_of(only, [&](StringRef base) {
      return kind.consume_front(base) &&
             (kind.empty() || kind.starts_with("."));
    });
  };

```
- **EN**: Implements logic around `onCodeAction`, `only`, `empty`, `any_of`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `onCodeAction`、`only`、`empty`、`any_of` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 304-313
```cpp
  // We provide a code action for fixes on the specified diagnostics.
  std::vector<CodeAction> actions;
  if (isKindAllowed(CodeAction::kQuickFix))
    server.getCodeActions(uri, params.range.start, params.context, actions);
  reply(std::move(actions));
}

//===----------------------------------------------------------------------===//
// Bytecode
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `isKindAllowed`, `getCodeActions`, `reply`.
- **CN**: 围绕 `isKindAllowed`、`getCodeActions`、`reply` 实现具体逻辑。

### Lines 314-325
```cpp

void LSPServer::onConvertFromBytecode(
    const MLIRConvertBytecodeParams &params,
    Callback<MLIRConvertBytecodeResult> reply) {
  reply(server.convertFromBytecode(params.uri));
}

void LSPServer::onConvertToBytecode(const MLIRConvertBytecodeParams &params,
                                    Callback<MLIRConvertBytecodeResult> reply) {
  reply(server.convertToBytecode(params.uri));
}

```
- **EN**: Implements logic around `onConvertFromBytecode`, `reply`, `onConvertToBytecode`.
- **CN**: 围绕 `onConvertFromBytecode`、`reply`、`onConvertToBytecode` 实现具体逻辑。

### Lines 326-340
```cpp
//===----------------------------------------------------------------------===//
// Entry point
//===----------------------------------------------------------------------===//

LogicalResult lsp::runMlirLSPServer(MLIRServer &server,
                                    JSONTransport &transport) {
  LSPServer lspServer(server);
  MessageHandler messageHandler(transport);

  // Initialization
  messageHandler.method("initialize", &lspServer, &LSPServer::onInitialize);
  messageHandler.notification("initialized", &lspServer,
                              &LSPServer::onInitialized);
  messageHandler.method("shutdown", &lspServer, &LSPServer::onShutdown);

```
- **EN**: Implements logic around `runMlirLSPServer`, `lspServer`, `messageHandler`, `method`, and 1 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `runMlirLSPServer`、`lspServer`、`messageHandler`、`method` 等另外 1 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 341-354
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

### Lines 355-365
```cpp
  // Hover
  messageHandler.method("textDocument/hover", &lspServer, &LSPServer::onHover);

  // Document Symbols
  messageHandler.method("textDocument/documentSymbol", &lspServer,
                        &LSPServer::onDocumentSymbol);

  // Code Completion
  messageHandler.method("textDocument/completion", &lspServer,
                        &LSPServer::onCompletion);

```
- **EN**: Implements logic around `method`.
- **CN**: 围绕 `method` 实现具体逻辑。

### Lines 366-375
```cpp
  // Code Action
  messageHandler.method("textDocument/codeAction", &lspServer,
                        &LSPServer::onCodeAction);

  // Bytecode
  messageHandler.method("mlir/convertFromBytecode", &lspServer,
                        &LSPServer::onConvertFromBytecode);
  messageHandler.method("mlir/convertToBytecode", &lspServer,
                        &LSPServer::onConvertToBytecode);

```
- **EN**: Implements logic around `method`.
- **CN**: 围绕 `method` 实现具体逻辑。

### Lines 376-391
```cpp
  // Diagnostics
  lspServer.publishDiagnostics =
      messageHandler.outgoingNotification<PublishDiagnosticsParams>(
          "textDocument/publishDiagnostics");

  // Run the main loop of the transport.
  LogicalResult result = success();
  if (llvm::Error error = transport.run(messageHandler)) {
    Logger::error("Transport error: {0}", error);
    llvm::consumeError(std::move(error));
    result = failure();
  } else {
    result = success(lspServer.shutdownRequestReceived);
  }
  return result;
}
```
- **EN**: Implements logic around `outgoingNotification`, `success`, `run`, `error`, and 2 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `outgoingNotification`、`success`、`run`、`error` 等另外 2 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

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
- **Bytecode support / Bytecode 支持**:
  - **EN**: The file reads, writes, or describes compact MLIR bytecode forms.
  - **CN**: 该文件读取、写出或描述紧凑的 MLIR bytecode 形式。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `LSPServer.h`, `MLIRServer.h`, `Protocol.h`, `llvm/Support/LSP/Logging.h`, `llvm/Support/LSP/Transport.h`
- **Standard-library headers / 标准库头文件**: `<optional>`
- **Subsystem categories / 子系统类别**: LLVM support-library helpers / LLVM Support 库辅助工具 (2)
