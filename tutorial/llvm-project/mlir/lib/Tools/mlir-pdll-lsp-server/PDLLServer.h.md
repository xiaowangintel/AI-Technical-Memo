# PDLLServer.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Tools/mlir-pdll-lsp-server/PDLLServer.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Declares reusable command-line tool support built on top of MLIR libraries.
  - **CN**: 声明构建于 MLIR 库之上的可复用命令行工具支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- PDLLServer.h - PDL General Language Server ---------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及高层摘要。

### Lines 8-17
```cpp

#ifndef LIB_MLIR_TOOLS_MLIRPDLLSPSERVER_SERVER_H_
#define LIB_MLIR_TOOLS_MLIRPDLLSPSERVER_SERVER_H_

#include "mlir/Support/LLVM.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/Support/LSP/Protocol.h"
#include <memory>
#include <optional>
#include <string>
```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/Support/LLVM.h`, `llvm/ADT/StringRef.h`, `llvm/Support/LSP/Protocol.h`, `memory`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/Support/LLVM.h`, `llvm/ADT/StringRef.h`, `llvm/Support/LSP/Protocol.h`, `memory`。

### Lines 18-27
```cpp
#include <vector>

namespace mlir {
namespace lsp {
using llvm::lsp::CompletionList;
using llvm::lsp::Diagnostic;
using llvm::lsp::DocumentLink;
using llvm::lsp::DocumentSymbol;
using llvm::lsp::Hover;
using llvm::lsp::InlayHint;
```
- **EN**: Pulls in the declarations needed by this translation unit, including `vector`.
- **CN**: 引入该编译单元所需的声明，其中包括 `vector`。

### Lines 28-34
```cpp
using llvm::lsp::Location;
using llvm::lsp::Position;
using llvm::lsp::Range;
using llvm::lsp::SignatureHelp;
using llvm::lsp::TextDocumentContentChangeEvent;
using llvm::lsp::URIForFile;

```
- **EN**: Brings selected namespaces, aliases, or helper symbols into the local scope for the implementation below.
- **CN**: 把选定的命名空间、别名或辅助符号引入当前局部作用域，供后续实现使用。

### Lines 35-39
```cpp
class CompilationDatabase;
struct PDLLViewOutputResult;
enum class PDLLViewOutputKind;

/// This class implements all of the PDLL related functionality necessary for a
```
- **EN**: Introduces declarations for `CompilationDatabase`, `PDLLViewOutputResult`, `PDLLViewOutputKind`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `CompilationDatabase`、`PDLLViewOutputResult`、`PDLLViewOutputKind` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 40-44
```cpp
/// language server. This class allows for keeping the PDLL specific logic
/// separate from the logic that involves LSP server/client communication.
class PDLLServer {
public:
  struct Options {
```
- **EN**: Introduces declarations for `PDLLServer`, `Options`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `PDLLServer`、`Options` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 45-49
```cpp
    Options(const std::vector<std::string> &compilationDatabases,
            const std::vector<std::string> &extraDirs)
        : compilationDatabases(compilationDatabases), extraDirs(extraDirs) {}

    /// The filenames for databases containing compilation commands for PDLL
```
- **EN**: Implements logic around `Options`, `compilationDatabases`.
- **CN**: 围绕 `Options`、`compilationDatabases` 实现具体逻辑。

### Lines 50-56
```cpp
    /// files passed to the server.
    const std::vector<std::string> &compilationDatabases;

    /// Additional list of include directories to search.
    const std::vector<std::string> &extraDirs;
  };

```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 57-61
```cpp
  PDLLServer(const Options &options);
  ~PDLLServer();

  /// Add the document, with the provided `version`, at the given URI. Any
  /// diagnostics emitted for this document should be added to `diagnostics`.
```
- **EN**: Declares APIs around `PDLLServer`, `~PDLLServer`.
- **CN**: 声明与 `PDLLServer`、`~PDLLServer` 相关的 API。

### Lines 62-66
```cpp
  void addDocument(const URIForFile &uri, StringRef contents, int64_t version,
                   std::vector<Diagnostic> &diagnostics);

  /// Update the document, with the provided `version`, at the given URI. Any
  /// diagnostics emitted for this document should be added to `diagnostics`.
```
- **EN**: Declares APIs around `addDocument`; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 声明与 `addDocument` 相关的 API；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 67-71
```cpp
  void updateDocument(const URIForFile &uri,
                      ArrayRef<TextDocumentContentChangeEvent> changes,
                      int64_t version, std::vector<Diagnostic> &diagnostics);

  /// Remove the document with the given uri. Returns the version of the removed
```
- **EN**: Declares APIs around `updateDocument`; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 声明与 `updateDocument` 相关的 API；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 72-76
```cpp
  /// document, or std::nullopt if the uri did not have a corresponding document
  /// within the server.
  std::optional<int64_t> removeDocument(const URIForFile &uri);

  /// Return the locations of the object pointed at by the given position.
```
- **EN**: Declares APIs around `removeDocument`.
- **CN**: 声明与 `removeDocument` 相关的 API。

### Lines 77-83
```cpp
  void getLocationsOf(const URIForFile &uri, const Position &defPos,
                      std::vector<Location> &locations);

  /// Find all references of the object pointed at by the given position.
  void findReferencesOf(const URIForFile &uri, const Position &pos,
                        std::vector<Location> &references);

```
- **EN**: Declares APIs around `getLocationsOf`, `findReferencesOf`; this block manipulates core MLIR IR objects.
- **CN**: 声明与 `getLocationsOf`、`findReferencesOf` 相关的 API；该代码块操作 MLIR 核心 IR 对象。

### Lines 84-88
```cpp
  /// Return the document links referenced by the given file.
  void getDocumentLinks(const URIForFile &uri,
                        std::vector<DocumentLink> &documentLinks);

  /// Find a hover description for the given hover position, or std::nullopt if
```
- **EN**: Declares APIs around `getDocumentLinks`.
- **CN**: 声明与 `getDocumentLinks` 相关的 API。

### Lines 89-93
```cpp
  /// one couldn't be found.
  std::optional<Hover> findHover(const URIForFile &uri,
                                 const Position &hoverPos);

  /// Find all of the document symbols within the given file.
```
- **EN**: Declares APIs around `findHover`.
- **CN**: 声明与 `findHover` 相关的 API。

### Lines 94-100
```cpp
  void findDocumentSymbols(const URIForFile &uri,
                           std::vector<DocumentSymbol> &symbols);

  /// Get the code completion list for the position within the given file.
  CompletionList getCodeCompletion(const URIForFile &uri,
                                   const Position &completePos);

```
- **EN**: Declares APIs around `findDocumentSymbols`, `getCodeCompletion`.
- **CN**: 声明与 `findDocumentSymbols`、`getCodeCompletion` 相关的 API。

### Lines 101-105
```cpp
  /// Get the signature help for the position within the given file.
  SignatureHelp getSignatureHelp(const URIForFile &uri,
                                 const Position &helpPos);

  /// Get the inlay hints for the range within the given file.
```
- **EN**: Declares APIs around `getSignatureHelp`.
- **CN**: 声明与 `getSignatureHelp` 相关的 API。

### Lines 106-110
```cpp
  void getInlayHints(const URIForFile &uri, const Range &range,
                     std::vector<InlayHint> &inlayHints);

  /// Get the output of the given PDLL file, or std::nullopt if there is no
  /// valid output.
```
- **EN**: Declares APIs around `getInlayHints`.
- **CN**: 声明与 `getInlayHints` 相关的 API。

### Lines 111-115
```cpp
  std::optional<PDLLViewOutputResult>
  getPDLLViewOutput(const URIForFile &uri, PDLLViewOutputKind kind);

private:
  struct Impl;
```
- **EN**: Introduces declarations for `Impl`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `Impl` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 116-121
```cpp
  std::unique_ptr<Impl> impl;
};

} // namespace lsp
} // namespace mlir

```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 122-122
```cpp
#endif // LIB_MLIR_TOOLS_MLIRPDLLSPSERVER_SERVER_H_
```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

## Key Concepts / 关键概念

- **Tooling support / 工具链支持**:
  - **EN**: Builds reusable infrastructure for opt-like tools, translation drivers, and CLIs.
  - **CN**: 为类似 opt 的工具、翻译驱动和命令行程序构建可复用基础设施。
- **Structured diagnostics / 结构化诊断**:
  - **EN**: Failures are surfaced through `LogicalResult`, diagnostics, or related reporting helpers.
  - **CN**: 通过 `LogicalResult`、诊断对象或相关报告工具显式传播失败。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `mlir/Support/LLVM.h`, `llvm/ADT/StringRef.h`, `llvm/Support/LSP/Protocol.h`
- **Standard-library headers / 标准库头文件**: `<memory>`, `<optional>`, `<string>`, `<vector>`
- **Subsystem categories / 子系统类别**: shared MLIR support helpers / 共享的 MLIR 支持工具 (1), LLVM ADT containers and utilities / LLVM ADT 容器与工具 (1), LLVM support-library helpers / LLVM Support 库辅助工具 (1)
