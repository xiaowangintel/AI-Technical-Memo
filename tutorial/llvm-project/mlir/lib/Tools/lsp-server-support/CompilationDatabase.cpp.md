# CompilationDatabase.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Tools/lsp-server-support/CompilationDatabase.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements reusable command-line tool support built on top of MLIR libraries.
  - **CN**: 实现构建于 MLIR 库之上的可复用命令行工具支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- CompilationDatabase.cpp - LSP Compilation Database -----------------===//
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

#include "mlir/Tools/lsp-server-support/CompilationDatabase.h"
#include "mlir/Support/FileUtilities.h"
#include "llvm/ADT/SetVector.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/Support/LSP/Logging.h"
#include "llvm/Support/LSP/Protocol.h"
#include "llvm/Support/YAMLTraits.h"

```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/Tools/lsp-server-support/CompilationDatabase.h`, `mlir/Support/FileUtilities.h`, `llvm/ADT/SetVector.h`, `llvm/ADT/StringRef.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/Tools/lsp-server-support/CompilationDatabase.h`, `mlir/Support/FileUtilities.h`, `llvm/ADT/SetVector.h`, `llvm/ADT/StringRef.h`。

### Lines 17-21
```cpp
using namespace mlir;
using namespace mlir::lsp;
using llvm::lsp::Logger;

//===----------------------------------------------------------------------===//
```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 22-26
```cpp
// YamlFileInfo
//===----------------------------------------------------------------------===//

namespace {
struct YamlFileInfo {
```
- **EN**: Introduces declarations for `YamlFileInfo`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `YamlFileInfo` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 27-33
```cpp
  /// The absolute path to the file.
  std::string filename;
  /// The include directories available for the file.
  std::vector<std::string> includeDirs;
};
} // namespace

```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 34-39
```cpp
//===----------------------------------------------------------------------===//
// CompilationDatabase
//===----------------------------------------------------------------------===//

LLVM_YAML_IS_DOCUMENT_LIST_VECTOR(YamlFileInfo)

```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 40-48
```cpp
namespace llvm {
namespace yaml {
template <>
struct MappingTraits<YamlFileInfo> {
  static void mapping(IO &io, YamlFileInfo &info) {
    // Parse the filename and normalize it to the form we will expect from
    // incoming URIs.
    io.mapRequired("filepath", info.filename);

```
- **EN**: Introduces declarations for `llvm`, `yaml`, `MappingTraits`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `llvm`、`yaml`、`MappingTraits` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 49-53
```cpp
    // Normalize the filename to avoid incompatability with incoming URIs.
    if (Expected<lsp::URIForFile> uri =
            lsp::URIForFile::fromFile(info.filename))
      info.filename = uri->file().str();

```
- **EN**: Implements logic around `fromFile`, `file`.
- **CN**: 围绕 `fromFile`、`file` 实现具体逻辑。

### Lines 54-63
```cpp
    // Parse the includes from the yaml stream. These are in the form of a
    // semi-colon delimited list.
    std::string combinedIncludes;
    io.mapRequired("includes", combinedIncludes);
    for (StringRef include : llvm::split(combinedIncludes, ";")) {
      if (!include.empty())
        info.includeDirs.push_back(include.str());
    }
  }
};
```
- **EN**: Implements logic around `mapRequired`, `split`, `empty`, `push_back`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `mapRequired`、`split`、`empty`、`push_back` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 64-71
```cpp
} // end namespace yaml
} // end namespace llvm

CompilationDatabase::CompilationDatabase(ArrayRef<std::string> databases) {
  for (StringRef filename : databases)
    loadDatabase(filename);
}

```
- **EN**: Implements logic around `CompilationDatabase`, `loadDatabase`.
- **CN**: 围绕 `CompilationDatabase`、`loadDatabase` 实现具体逻辑。

### Lines 72-77
```cpp
const CompilationDatabase::FileInfo &
CompilationDatabase::getFileInfo(StringRef filename) const {
  auto it = files.find(filename);
  return it == files.end() ? defaultFileInfo : it->second;
}

```
- **EN**: Implements logic around `getFileInfo`, `find`, `end`.
- **CN**: 围绕 `getFileInfo`、`find`、`end` 实现具体逻辑。

### Lines 78-87
```cpp
void CompilationDatabase::loadDatabase(StringRef filename) {
  if (filename.empty())
    return;

  // Set up the input file.
  std::string errorMessage;
  std::unique_ptr<llvm::MemoryBuffer> inputFile =
      openInputFile(filename, &errorMessage);
  if (!inputFile) {
    Logger::error("Failed to open compilation database: {0}", errorMessage);
```
- **EN**: Implements logic around `loadDatabase`, `empty`, `openInputFile`, `error`.
- **CN**: 围绕 `loadDatabase`、`empty`、`openInputFile`、`error` 实现具体逻辑。

### Lines 88-95
```cpp
    return;
  }
  llvm::yaml::Input yaml(inputFile->getBuffer());

  // Parse the yaml description and add any new files to the database.
  std::vector<YamlFileInfo> parsedFiles;
  yaml >> parsedFiles;

```
- **EN**: Implements logic around `yaml`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `yaml` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 96-105
```cpp
  SetVector<StringRef> knownIncludes;
  for (auto &file : parsedFiles) {
    auto it = files.try_emplace(file.filename, std::move(file.includeDirs));

    // If we encounter a duplicate file, log a warning and ignore it.
    if (!it.second) {
      Logger::info("Duplicate file in compilation database: {0}",
                   file.filename);
      continue;
    }
```
- **EN**: Implements logic around `try_emplace`, `info`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `try_emplace`、`info` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 106-110
```cpp

    // Track the includes for the file.
    knownIncludes.insert_range(it.first->second.includeDirs);
  }

```
- **EN**: Implements logic around `insert_range`.
- **CN**: 围绕 `insert_range` 实现具体逻辑。

### Lines 111-118
```cpp
  // Add all of the known includes to the default file info. We don't know any
  // information about how to treat these files, but these may be project files
  // that we just don't yet have information for. In these cases, providing some
  // heuristic information provides a better user experience, and generally
  // shouldn't lead to any negative side effects.
  for (StringRef include : knownIncludes)
    defaultFileInfo.includeDirs.push_back(include.str());
}
```
- **EN**: Implements logic around `push_back`.
- **CN**: 围绕 `push_back` 实现具体逻辑。

## Key Concepts / 关键概念

- **Tooling support / 工具链支持**:
  - **EN**: Builds reusable infrastructure for opt-like tools, translation drivers, and CLIs.
  - **CN**: 为类似 opt 的工具、翻译驱动和命令行程序构建可复用基础设施。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `mlir/Tools/lsp-server-support/CompilationDatabase.h`, `mlir/Support/FileUtilities.h`, `llvm/ADT/SetVector.h`, `llvm/ADT/StringRef.h`, `llvm/Support/LSP/Logging.h`, `llvm/Support/LSP/Protocol.h`, `llvm/Support/YAMLTraits.h`
- **Subsystem categories / 子系统类别**: LLVM support-library helpers / LLVM Support 库辅助工具 (3), LLVM ADT containers and utilities / LLVM ADT 容器与工具 (2), tooling support declarations / 工具支持声明 (1), shared MLIR support helpers / 共享的 MLIR 支持工具 (1)
