# MLIRServer.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Tools/mlir-lsp-server/MLIRServer.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Declares reusable command-line tool support built on top of MLIR libraries.
  - **CN**: 声明构建于 MLIR 库之上的可复用命令行工具支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- MLIRServer.h - MLIR General Language Server --------------*- C++ -*-===//
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

#ifndef LIB_MLIR_TOOLS_MLIRLSPSERVER_SERVER_H_
#define LIB_MLIR_TOOLS_MLIRLSPSERVER_SERVER_H_

#include "Protocol.h"
#include "mlir/Support/LLVM.h"
#include "mlir/Tools/mlir-lsp-server/MlirLspRegistryFunction.h"
#include "llvm/Support/Error.h"
#include <memory>
#include <optional>
```
- **EN**: Pulls in the declarations needed by this translation unit, including `Protocol.h`, `mlir/Support/LLVM.h`, `mlir/Tools/mlir-lsp-server/MlirLspRegistryFunction.h`, `llvm/Support/Error.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `Protocol.h`, `mlir/Support/LLVM.h`, `mlir/Tools/mlir-lsp-server/MlirLspRegistryFunction.h`, `llvm/Support/Error.h`。

### Lines 18-22
```cpp

namespace mlir {
class DialectRegistry;

namespace lsp {
```
- **EN**: Introduces declarations for `mlir`, `DialectRegistry`, `lsp`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `mlir`、`DialectRegistry`、`lsp` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 23-32
```cpp
using llvm::lsp::CodeAction;
using llvm::lsp::CodeActionContext;
using llvm::lsp::CompletionList;
using llvm::lsp::Diagnostic;
using llvm::lsp::DocumentSymbol;
using llvm::lsp::Hover;
using llvm::lsp::Location;
using llvm::lsp::MLIRConvertBytecodeResult;
using llvm::lsp::Position;
using llvm::lsp::Range;
```
- **EN**: Brings selected namespaces, aliases, or helper symbols into the local scope for the implementation below.
- **CN**: 把选定的命名空间、别名或辅助符号引入当前局部作用域，供后续实现使用。

### Lines 33-37
```cpp
using llvm::lsp::URIForFile;

/// This class implements all of the MLIR related functionality necessary for a
/// language server. This class allows for keeping the MLIR specific logic
/// separate from the logic that involves LSP server/client communication.
```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 38-43
```cpp
class MLIRServer {
public:
  /// Construct a new server with the given dialect registry function.
  MLIRServer(DialectRegistryFn registry_fn);
  ~MLIRServer();

```
- **EN**: Introduces declarations for `MLIRServer`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `MLIRServer` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 44-50
```cpp
  /// Add or update the document, with the provided `version`, at the given URI.
  /// Any diagnostics emitted for this document should be added to
  /// `diagnostics`.
  void addOrUpdateDocument(const URIForFile &uri, StringRef contents,
                           int64_t version,
                           std::vector<Diagnostic> &diagnostics);

```
- **EN**: Declares APIs around `addOrUpdateDocument`; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 声明与 `addOrUpdateDocument` 相关的 API；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 51-55
```cpp
  /// Remove the document with the given uri. Returns the version of the removed
  /// document, or std::nullopt if the uri did not have a corresponding document
  /// within the server.
  std::optional<int64_t> removeDocument(const URIForFile &uri);

```
- **EN**: Declares APIs around `removeDocument`.
- **CN**: 声明与 `removeDocument` 相关的 API。

### Lines 56-60
```cpp
  /// Return the locations of the object pointed at by the given position.
  void getLocationsOf(const URIForFile &uri, const Position &defPos,
                      std::vector<Location> &locations);

  /// Find all references of the object pointed at by the given position.
```
- **EN**: Declares APIs around `getLocationsOf`; this block manipulates core MLIR IR objects.
- **CN**: 声明与 `getLocationsOf` 相关的 API；该代码块操作 MLIR 核心 IR 对象。

### Lines 61-65
```cpp
  void findReferencesOf(const URIForFile &uri, const Position &pos,
                        std::vector<Location> &references);

  /// Find a hover description for the given hover position, or std::nullopt if
  /// one couldn't be found.
```
- **EN**: Declares APIs around `findReferencesOf`; this block manipulates core MLIR IR objects.
- **CN**: 声明与 `findReferencesOf` 相关的 API；该代码块操作 MLIR 核心 IR 对象。

### Lines 66-72
```cpp
  std::optional<Hover> findHover(const URIForFile &uri,
                                 const Position &hoverPos);

  /// Find all of the document symbols within the given file.
  void findDocumentSymbols(const URIForFile &uri,
                           std::vector<DocumentSymbol> &symbols);

```
- **EN**: Declares APIs around `findHover`, `findDocumentSymbols`.
- **CN**: 声明与 `findHover`、`findDocumentSymbols` 相关的 API。

### Lines 73-77
```cpp
  /// Get the code completion list for the position within the given file.
  CompletionList getCodeCompletion(const URIForFile &uri,
                                   const Position &completePos);

  /// Get the set of code actions within the file.
```
- **EN**: Declares APIs around `getCodeCompletion`.
- **CN**: 声明与 `getCodeCompletion` 相关的 API。

### Lines 78-82
```cpp
  void getCodeActions(const URIForFile &uri, const Range &pos,
                      const CodeActionContext &context,
                      std::vector<CodeAction> &actions);

  /// Convert the given bytecode file to the textual format.
```
- **EN**: Declares APIs around `getCodeActions`.
- **CN**: 声明与 `getCodeActions` 相关的 API。

### Lines 83-89
```cpp
  llvm::Expected<MLIRConvertBytecodeResult>
  convertFromBytecode(const URIForFile &uri);

  /// Convert the given textual file to the bytecode format.
  llvm::Expected<MLIRConvertBytecodeResult>
  convertToBytecode(const URIForFile &uri);

```
- **EN**: Declares APIs around `convertFromBytecode`, `convertToBytecode`.
- **CN**: 声明与 `convertFromBytecode`、`convertToBytecode` 相关的 API。

### Lines 90-94
```cpp
  /// Set the workspace root for the server.
  void setWorkspaceRoot(StringRef root);

private:
  struct Impl;
```
- **EN**: Introduces declarations for `Impl`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `Impl` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 95-101
```cpp

  std::unique_ptr<Impl> impl;
};

} // namespace lsp
} // namespace mlir

```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 102-102
```cpp
#endif // LIB_MLIR_TOOLS_MLIRLSPSERVER_SERVER_H_
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
- **Bytecode support / Bytecode 支持**:
  - **EN**: The file reads, writes, or describes compact MLIR bytecode forms.
  - **CN**: 该文件读取、写出或描述紧凑的 MLIR bytecode 形式。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `Protocol.h`, `mlir/Support/LLVM.h`, `mlir/Tools/mlir-lsp-server/MlirLspRegistryFunction.h`, `llvm/Support/Error.h`
- **Standard-library headers / 标准库头文件**: `<memory>`, `<optional>`
- **Subsystem categories / 子系统类别**: shared MLIR support helpers / 共享的 MLIR 支持工具 (1), tooling support declarations / 工具支持声明 (1), LLVM support-library helpers / LLVM Support 库辅助工具 (1)
