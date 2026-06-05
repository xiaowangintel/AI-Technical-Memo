# IRPrinting.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Pass/IRPrinting.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements pass-manager infrastructure, pass registration, and pass execution support.
  - **CN**: 实现 Pass 管理器基础设施、Pass 注册以及执行支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-19
```cpp
//===- IRPrinting.cpp -----------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "PassDetail.h"
#include "mlir/IR/SymbolTable.h"
#include "mlir/Pass/PassManager.h"
#include "mlir/Support/FileUtilities.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/Support/FileSystem.h"
#include "llvm/Support/FormatVariadic.h"
#include "llvm/Support/Path.h"
#include "llvm/Support/ToolOutputFile.h"

```
- **EN**: Pulls in the declarations needed by this translation unit, including `PassDetail.h`, `mlir/IR/SymbolTable.h`, `mlir/Pass/PassManager.h`, `mlir/Support/FileUtilities.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `PassDetail.h`, `mlir/IR/SymbolTable.h`, `mlir/Pass/PassManager.h`, `mlir/Support/FileUtilities.h`。

### Lines 20-29
```cpp
using namespace mlir;
using namespace mlir::detail;

namespace {
//===----------------------------------------------------------------------===//
// IRPrinter
//===----------------------------------------------------------------------===//

class IRPrinterInstrumentation : public PassInstrumentation {
public:
```
- **EN**: Introduces declarations for `IRPrinterInstrumentation`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `IRPrinterInstrumentation` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 30-39
```cpp
  IRPrinterInstrumentation(std::unique_ptr<PassManager::IRPrinterConfig> config)
      : config(std::move(config)) {}

private:
  /// Instrumentation hooks.
  void runBeforePass(Pass *pass, Operation *op) override;
  void runAfterPass(Pass *pass, Operation *op) override;
  void runAfterPassFailed(Pass *pass, Operation *op) override;

  /// Configuration to use.
```
- **EN**: Implements logic around `IRPrinterInstrumentation`, `config`, `runBeforePass`, `runAfterPass`, and 1 more symbols; this block coordinates pass execution or analysis state; manipulates core MLIR IR objects.
- **CN**: 围绕 `IRPrinterInstrumentation`、`config`、`runBeforePass`、`runAfterPass` 等另外 1 个符号 实现具体逻辑；该代码块协调 Pass 执行或分析状态，并操作 MLIR 核心 IR 对象。

### Lines 40-54
```cpp
  std::unique_ptr<PassManager::IRPrinterConfig> config;

  /// The following is a set of fingerprints for operations that are currently
  /// being operated on in a pass. This field is only used when the
  /// configuration asked for change detection.
  DenseMap<Pass *, OperationFingerPrint> beforePassFingerPrints;
};
} // namespace

static void printIR(Operation *op, bool printModuleScope, raw_ostream &out,
                    OpPrintingFlags flags) {
  // Check to see if we are not printing at module scope.
  if (!printModuleScope)
    return op->print(out, op->getBlock() ? flags.useLocalScope() : flags);

```
- **EN**: Implements logic around `printIR`, `print`; this block coordinates pass execution or analysis state; manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `printIR`、`print` 实现具体逻辑；该代码块协调 Pass 执行或分析状态，并操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 55-74
```cpp
  // Otherwise, we are printing at module scope.
  // Find the top-level operation.
  auto *topLevelOp = op;
  while (auto *parentOp = topLevelOp->getParentOp())
    topLevelOp = parentOp;
  topLevelOp->print(out, flags);
}

static void printIRHeader(raw_ostream &out, StringRef title, Pass *pass,
                          Operation *op, bool printModuleScope,
                          bool failed = false) {
  out << "// -----// IR Dump " << title << " " << pass->getName();
  if (failed)
    out << " Failed";
  out << ": ";
  pass->printAsTextualPipeline(out);
  if (printModuleScope) {
    out << " ('" << op->getName() << "' operation";
    if (auto symbolName =
            op->getAttrOfType<StringAttr>(SymbolTable::getSymbolAttrName()))
```
- **EN**: Implements logic around `getParentOp`, `print`, `printIRHeader`, `getName`, and 2 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `getParentOp`、`print`、`printIRHeader`、`getName` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 75-88
```cpp
      out << ": @" << symbolName.getValue();
    out << ")";
  }
  out << " //----- //\n";
}

/// Instrumentation hooks.
void IRPrinterInstrumentation::runBeforePass(Pass *pass, Operation *op) {
  if (isa<OpToOpPassAdaptor>(pass))
    return;
  // If the config asked to detect changes, record the current fingerprint.
  if (config->shouldPrintAfterOnlyOnChange())
    beforePassFingerPrints.try_emplace(pass, op);

```
- **EN**: Implements logic around `getValue`, `runBeforePass`, `isa`, `shouldPrintAfterOnlyOnChange`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getValue`、`runBeforePass`、`isa`、`shouldPrintAfterOnlyOnChange` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 89-100
```cpp
  config->printBeforeIfEnabled(pass, op, [&](raw_ostream &out) {
    printIRHeader(out, "Before", pass, op, config->shouldPrintAtModuleScope());
    printIR(op, config->shouldPrintAtModuleScope(), out,
            config->getOpPrintingFlags());
    out << "\n\n";
  });
}

void IRPrinterInstrumentation::runAfterPass(Pass *pass, Operation *op) {
  if (isa<OpToOpPassAdaptor>(pass))
    return;

```
- **EN**: Implements logic around `printBeforeIfEnabled`, `printIRHeader`, `printIR`, `getOpPrintingFlags`, and 2 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `printBeforeIfEnabled`、`printIRHeader`、`printIR`、`getOpPrintingFlags` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 101-118
```cpp
  // Check to see if we are only printing on failure.
  if (config->shouldPrintAfterOnlyOnFailure())
    return;

  // If the config asked to detect changes, compare the current fingerprint with
  // the previous.
  if (config->shouldPrintAfterOnlyOnChange()) {
    auto fingerPrintIt = beforePassFingerPrints.find(pass);
    assert(fingerPrintIt != beforePassFingerPrints.end() &&
           "expected valid fingerprint");
    // If the fingerprints are the same, we don't print the IR.
    if (fingerPrintIt->second == OperationFingerPrint(op)) {
      beforePassFingerPrints.erase(fingerPrintIt);
      return;
    }
    beforePassFingerPrints.erase(fingerPrintIt);
  }

```
- **EN**: Implements logic around `shouldPrintAfterOnlyOnFailure`, `shouldPrintAfterOnlyOnChange`, `find`, `assert`, and 2 more symbols; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `shouldPrintAfterOnlyOnFailure`、`shouldPrintAfterOnlyOnChange`、`find`、`assert` 等另外 2 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 119-132
```cpp
  config->printAfterIfEnabled(pass, op, [&](raw_ostream &out) {
    printIRHeader(out, "After", pass, op, config->shouldPrintAtModuleScope());
    printIR(op, config->shouldPrintAtModuleScope(), out,
            config->getOpPrintingFlags());
    out << "\n\n";
  });
}

void IRPrinterInstrumentation::runAfterPassFailed(Pass *pass, Operation *op) {
  if (isa<OpToOpPassAdaptor>(pass))
    return;
  if (config->shouldPrintAfterOnlyOnChange())
    beforePassFingerPrints.erase(pass);

```
- **EN**: Implements logic around `printAfterIfEnabled`, `printIRHeader`, `printIR`, `getOpPrintingFlags`, and 4 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `printAfterIfEnabled`、`printIRHeader`、`printIR`、`getOpPrintingFlags` 等另外 4 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 133-142
```cpp
  config->printAfterIfEnabled(pass, op, [&](raw_ostream &out) {
    printIRHeader(out, "After", pass, op, config->shouldPrintAtModuleScope(),
                  /*failed=*/true);
    printIR(op, config->shouldPrintAtModuleScope(), out,
            config->getOpPrintingFlags());
    out << "\n\n";
  });
}

//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `printAfterIfEnabled`, `printIRHeader`, `printIR`, `getOpPrintingFlags`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `printAfterIfEnabled`、`printIRHeader`、`printIR`、`getOpPrintingFlags` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 143-156
```cpp
// IRPrinterConfig
//===----------------------------------------------------------------------===//

/// Initialize the configuration.
PassManager::IRPrinterConfig::IRPrinterConfig(bool printModuleScope,
                                              bool printAfterOnlyOnChange,
                                              bool printAfterOnlyOnFailure,
                                              OpPrintingFlags opPrintingFlags)
    : printModuleScope(printModuleScope),
      printAfterOnlyOnChange(printAfterOnlyOnChange),
      printAfterOnlyOnFailure(printAfterOnlyOnFailure),
      opPrintingFlags(opPrintingFlags) {}
PassManager::IRPrinterConfig::~IRPrinterConfig() = default;

```
- **EN**: Implements logic around `IRPrinterConfig`, `printModuleScope`, `printAfterOnlyOnChange`, `printAfterOnlyOnFailure`, and 2 more symbols; this block coordinates pass execution or analysis state; parses or prints textual MLIR representations.
- **CN**: 围绕 `IRPrinterConfig`、`printModuleScope`、`printAfterOnlyOnChange`、`printAfterOnlyOnFailure` 等另外 2 个符号 实现具体逻辑；该代码块协调 Pass 执行或分析状态，并解析或打印文本形式的 MLIR 表示。

### Lines 157-166
```cpp
/// A hook that may be overridden by a derived config that checks if the IR
/// of 'operation' should be dumped *before* the pass 'pass' has been
/// executed. If the IR should be dumped, 'printCallback' should be invoked
/// with the stream to dump into.
void PassManager::IRPrinterConfig::printBeforeIfEnabled(
    Pass *pass, Operation *operation, PrintCallbackFn printCallback) {
  // By default, never print.
}

/// A hook that may be overridden by a derived config that checks if the IR
```
- **EN**: Implements logic around `printBeforeIfEnabled`; this block coordinates pass execution or analysis state; manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `printBeforeIfEnabled` 实现具体逻辑；该代码块协调 Pass 执行或分析状态，并操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 167-177
```cpp
/// of 'operation' should be dumped *after* the pass 'pass' has been
/// executed. If the IR should be dumped, 'printCallback' should be invoked
/// with the stream to dump into.
void PassManager::IRPrinterConfig::printAfterIfEnabled(
    Pass *pass, Operation *operation, PrintCallbackFn printCallback) {
  // By default, never print.
}

//===----------------------------------------------------------------------===//
// PassManager
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `printAfterIfEnabled`; this block coordinates pass execution or analysis state; manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `printAfterIfEnabled` 实现具体逻辑；该代码块协调 Pass 执行或分析状态，并操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 178-195
```cpp

namespace {
/// Simple wrapper config that allows for the simpler interface defined above.
struct BasicIRPrinterConfig : public PassManager::IRPrinterConfig {
  BasicIRPrinterConfig(
      std::function<bool(Pass *, Operation *)> shouldPrintBeforePass,
      std::function<bool(Pass *, Operation *)> shouldPrintAfterPass,
      bool printModuleScope, bool printAfterOnlyOnChange,
      bool printAfterOnlyOnFailure, OpPrintingFlags opPrintingFlags,
      raw_ostream &out)
      : IRPrinterConfig(printModuleScope, printAfterOnlyOnChange,
                        printAfterOnlyOnFailure, opPrintingFlags),
        shouldPrintBeforePass(std::move(shouldPrintBeforePass)),
        shouldPrintAfterPass(std::move(shouldPrintAfterPass)), out(out) {
    assert((this->shouldPrintBeforePass || this->shouldPrintAfterPass) &&
           "expected at least one valid filter function");
  }

```
- **EN**: Introduces declarations for `BasicIRPrinterConfig`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `BasicIRPrinterConfig` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 196-207
```cpp
  void printBeforeIfEnabled(Pass *pass, Operation *operation,
                            PrintCallbackFn printCallback) final {
    if (shouldPrintBeforePass && shouldPrintBeforePass(pass, operation))
      printCallback(out);
  }

  void printAfterIfEnabled(Pass *pass, Operation *operation,
                           PrintCallbackFn printCallback) final {
    if (shouldPrintAfterPass && shouldPrintAfterPass(pass, operation))
      printCallback(out);
  }

```
- **EN**: Implements logic around `printBeforeIfEnabled`, `shouldPrintBeforePass`, `printCallback`, `printAfterIfEnabled`, and 1 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `printBeforeIfEnabled`、`shouldPrintBeforePass`、`printCallback`、`printAfterIfEnabled` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 208-217
```cpp
  /// Filter functions for before and after pass execution.
  std::function<bool(Pass *, Operation *)> shouldPrintBeforePass;
  std::function<bool(Pass *, Operation *)> shouldPrintAfterPass;

  /// The stream to output to.
  raw_ostream &out;
};
} // namespace

/// Return pairs of (sanitized op name, symbol name) for `op` and all parent
```
- **EN**: Implements logic around `function`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `function` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 218-228
```cpp
/// operations. Op names are sanitized by replacing periods with underscores.
/// The pairs are returned in order of outer-most to inner-most (ancestors of
/// `op` first, `op` last). This information is used to construct the directory
/// tree for the `FileTreeIRPrinterConfig` below.
/// The counter for `op` will be incremented by this call.
static std::pair<SmallVector<std::pair<std::string, std::string>>, std::string>
getOpAndSymbolNames(Operation *op, StringRef passName,
                    llvm::DenseMap<Operation *, unsigned> &counters) {
  SmallVector<std::pair<std::string, std::string>> pathElements;
  SmallVector<unsigned> countPrefix;

```
- **EN**: Implements logic around `getOpAndSymbolNames`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getOpAndSymbolNames` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 229-239
```cpp
  Operation *iter = op;
  ++counters.try_emplace(op, -1).first->second;
  while (iter) {
    countPrefix.push_back(counters[iter]);
    StringAttr symbolNameAttr =
        iter->getAttrOfType<StringAttr>(SymbolTable::getSymbolAttrName());
    std::string symbolName =
        symbolNameAttr ? symbolNameAttr.str() : "no-symbol-name";
    llvm::replace(symbolName, '/', '_');
    llvm::replace(symbolName, '\\', '_');

```
- **EN**: Implements logic around `try_emplace`, `push_back`, `getAttrOfType`, `str`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `try_emplace`、`push_back`、`getAttrOfType`、`str` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 240-252
```cpp
    std::string opName =
        llvm::join(llvm::split(iter->getName().getStringRef().str(), '.'), "_");
    pathElements.emplace_back(std::move(opName), std::move(symbolName));
    iter = iter->getParentOp();
  }
  // Return in the order of top level (module) down to `op`.
  std::reverse(countPrefix.begin(), countPrefix.end());
  std::reverse(pathElements.begin(), pathElements.end());

  std::string passFileName = llvm::formatv(
      "{0:$[_]}_{1}.mlir",
      llvm::make_range(countPrefix.begin(), countPrefix.end()), passName);

```
- **EN**: Implements logic around `join`, `emplace_back`, `getParentOp`, `reverse`, and 2 more symbols.
- **CN**: 围绕 `join`、`emplace_back`、`getParentOp`、`reverse` 等另外 2 个符号 实现具体逻辑。

### Lines 253-265
```cpp
  return {pathElements, passFileName};
}

static LogicalResult createDirectoryOrPrintErr(llvm::StringRef dirPath) {
  if (std::error_code ec =
          llvm::sys::fs::create_directory(dirPath, /*IgnoreExisting=*/true)) {
    llvm::errs() << "Error while creating directory " << dirPath << ": "
                 << ec.message() << "\n";
    return failure();
  }
  return success();
}

```
- **EN**: Implements logic around `createDirectoryOrPrintErr`, `create_directory`, `errs`, `message`, and 2 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `createDirectoryOrPrintErr`、`create_directory`、`errs`、`message` 等另外 2 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 266-278
```cpp
/// Creates  directories (if required) and opens an output file for the
/// FileTreeIRPrinterConfig.
static std::unique_ptr<llvm::ToolOutputFile>
createTreePrinterOutputPath(Operation *op, llvm::StringRef passArgument,
                            llvm::StringRef rootDir,
                            llvm::DenseMap<Operation *, unsigned> &counters) {
  // Create the path. We will create a tree rooted at the given 'rootDir'
  // directory. The root directory will contain folders with the names of
  // modules. Sub-directories within those folders mirror the nesting
  // structure of the pass manager, using symbol names for directory names.
  auto [opAndSymbolNames, fileName] =
      getOpAndSymbolNames(op, passArgument, counters);

```
- **EN**: Implements logic around `createTreePrinterOutputPath`, `getOpAndSymbolNames`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `createTreePrinterOutputPath`、`getOpAndSymbolNames` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 279-290
```cpp
  // Create all the directories, starting at the root. Abort early if we fail to
  // create any directory.
  llvm::SmallString<128> path(rootDir);
  if (failed(createDirectoryOrPrintErr(path)))
    return nullptr;

  for (const auto &[opName, symbolName] : opAndSymbolNames) {
    llvm::sys::path::append(path, opName + "_" + symbolName);
    if (failed(createDirectoryOrPrintErr(path)))
      return nullptr;
  }

```
- **EN**: Implements logic around `path`, `failed`, `append`.
- **CN**: 围绕 `path`、`failed`、`append` 实现具体逻辑。

### Lines 291-302
```cpp
  // Open output file.
  llvm::sys::path::append(path, fileName);
  std::string error;
  std::unique_ptr<llvm::ToolOutputFile> file = openOutputFile(path, &error);
  if (!file) {
    llvm::errs() << "Error opening output file " << path << ": " << error
                 << "\n";
    return nullptr;
  }
  return file;
}

```
- **EN**: Implements logic around `append`, `openOutputFile`, `errs`.
- **CN**: 围绕 `append`、`openOutputFile`、`errs` 实现具体逻辑。

### Lines 303-322
```cpp
namespace {
/// A configuration that prints the IR before/after each pass to a set of files
/// in the specified directory. The files are organized into subdirectories that
/// mirror the nesting structure of the IR.
struct FileTreeIRPrinterConfig : public PassManager::IRPrinterConfig {
  FileTreeIRPrinterConfig(
      std::function<bool(Pass *, Operation *)> shouldPrintBeforePass,
      std::function<bool(Pass *, Operation *)> shouldPrintAfterPass,
      bool printModuleScope, bool printAfterOnlyOnChange,
      bool printAfterOnlyOnFailure, OpPrintingFlags opPrintingFlags,
      llvm::StringRef treeDir)
      : IRPrinterConfig(printModuleScope, printAfterOnlyOnChange,
                        printAfterOnlyOnFailure, opPrintingFlags),
        shouldPrintBeforePass(std::move(shouldPrintBeforePass)),
        shouldPrintAfterPass(std::move(shouldPrintAfterPass)),
        treeDir(treeDir) {
    assert((this->shouldPrintBeforePass || this->shouldPrintAfterPass) &&
           "expected at least one valid filter function");
  }

```
- **EN**: Introduces declarations for `FileTreeIRPrinterConfig`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `FileTreeIRPrinterConfig` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 323-334
```cpp
  void printBeforeIfEnabled(Pass *pass, Operation *operation,
                            PrintCallbackFn printCallback) final {
    if (!shouldPrintBeforePass || !shouldPrintBeforePass(pass, operation))
      return;
    std::unique_ptr<llvm::ToolOutputFile> file = createTreePrinterOutputPath(
        operation, pass->getArgument(), treeDir, counters);
    if (!file)
      return;
    printCallback(file->os());
    file->keep();
  }

```
- **EN**: Implements logic around `printBeforeIfEnabled`, `shouldPrintBeforePass`, `createTreePrinterOutputPath`, `getArgument`, and 2 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `printBeforeIfEnabled`、`shouldPrintBeforePass`、`createTreePrinterOutputPath`、`getArgument` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 335-346
```cpp
  void printAfterIfEnabled(Pass *pass, Operation *operation,
                           PrintCallbackFn printCallback) final {
    if (!shouldPrintAfterPass || !shouldPrintAfterPass(pass, operation))
      return;
    std::unique_ptr<llvm::ToolOutputFile> file = createTreePrinterOutputPath(
        operation, pass->getArgument(), treeDir, counters);
    if (!file)
      return;
    printCallback(file->os());
    file->keep();
  }

```
- **EN**: Implements logic around `printAfterIfEnabled`, `shouldPrintAfterPass`, `createTreePrinterOutputPath`, `getArgument`, and 2 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `printAfterIfEnabled`、`shouldPrintAfterPass`、`createTreePrinterOutputPath`、`getArgument` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 347-358
```cpp
  /// Filter functions for before and after pass execution.
  std::function<bool(Pass *, Operation *)> shouldPrintBeforePass;
  std::function<bool(Pass *, Operation *)> shouldPrintAfterPass;

  /// Directory that should be used as the root of the file tree.
  std::string treeDir;

  /// Counters used for labeling the prefix. Every op which could be targeted by
  /// a pass gets its own counter.
  llvm::DenseMap<Operation *, unsigned> counters;
};

```
- **EN**: Implements logic around `function`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `function` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 359-371
```cpp
} // namespace

/// Add an instrumentation to print the IR before and after pass execution,
/// using the provided configuration.
void PassManager::enableIRPrinting(std::unique_ptr<IRPrinterConfig> config) {
  if (config->shouldPrintAtModuleScope() &&
      getContext()->isMultithreadingEnabled())
    llvm::report_fatal_error("IR printing can't be setup on a pass-manager "
                             "without disabling multi-threading first.");
  addInstrumentation(
      std::make_unique<IRPrinterInstrumentation>(std::move(config)));
}

```
- **EN**: Implements logic around `enableIRPrinting`, `shouldPrintAtModuleScope`, `getContext`, `report_fatal_error`, and 2 more symbols; this block coordinates pass execution or analysis state; parses or prints textual MLIR representations.
- **CN**: 围绕 `enableIRPrinting`、`shouldPrintAtModuleScope`、`getContext`、`report_fatal_error` 等另外 2 个符号 实现具体逻辑；该代码块协调 Pass 执行或分析状态，并解析或打印文本形式的 MLIR 表示。

### Lines 372-384
```cpp
/// Add an instrumentation to print the IR before and after pass execution.
void PassManager::enableIRPrinting(
    std::function<bool(Pass *, Operation *)> shouldPrintBeforePass,
    std::function<bool(Pass *, Operation *)> shouldPrintAfterPass,
    bool printModuleScope, bool printAfterOnlyOnChange,
    bool printAfterOnlyOnFailure, raw_ostream &out,
    OpPrintingFlags opPrintingFlags) {
  enableIRPrinting(std::make_unique<BasicIRPrinterConfig>(
      std::move(shouldPrintBeforePass), std::move(shouldPrintAfterPass),
      printModuleScope, printAfterOnlyOnChange, printAfterOnlyOnFailure,
      opPrintingFlags, out));
}

```
- **EN**: Implements logic around `enableIRPrinting`, `function`, `move`; this block coordinates pass execution or analysis state; manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `enableIRPrinting`、`function`、`move` 实现具体逻辑；该代码块协调 Pass 执行或分析状态，并操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 385-396
```cpp
/// Add an instrumentation to print the IR before and after pass execution.
void PassManager::enableIRPrintingToFileTree(
    std::function<bool(Pass *, Operation *)> shouldPrintBeforePass,
    std::function<bool(Pass *, Operation *)> shouldPrintAfterPass,
    bool printModuleScope, bool printAfterOnlyOnChange,
    bool printAfterOnlyOnFailure, StringRef printTreeDir,
    OpPrintingFlags opPrintingFlags) {
  enableIRPrinting(std::make_unique<FileTreeIRPrinterConfig>(
      std::move(shouldPrintBeforePass), std::move(shouldPrintAfterPass),
      printModuleScope, printAfterOnlyOnChange, printAfterOnlyOnFailure,
      opPrintingFlags, printTreeDir));
}
```
- **EN**: Implements logic around `enableIRPrintingToFileTree`, `function`, `enableIRPrinting`, `move`; this block coordinates pass execution or analysis state; manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `enableIRPrintingToFileTree`、`function`、`enableIRPrinting`、`move` 实现具体逻辑；该代码块协调 Pass 执行或分析状态，并操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

## Key Concepts / 关键概念

- **Pass infrastructure / Pass 基础设施**:
  - **EN**: Coordinates pass pipelines, analyses, instrumentation, and nested operation traversal.
  - **CN**: 协调 Pass 流水线、分析、插桩以及嵌套操作遍历。
- **SSA and region-based IR / SSA 与基于 Region 的 IR**:
  - **EN**: The file works with MLIR operations, blocks, values, or regions that form MLIR's hierarchical SSA IR.
  - **CN**: 该文件处理 MLIR 操作、块、值或 Region，它们共同构成 MLIR 的层次化 SSA IR。
- **Structured diagnostics / 结构化诊断**:
  - **EN**: Failures are surfaced through `LogicalResult`, diagnostics, or related reporting helpers.
  - **CN**: 通过 `LogicalResult`、诊断对象或相关报告工具显式传播失败。
- **Pass pipelines / Pass 流水线**:
  - **EN**: The file participates in pass registration, scheduling, or execution orchestration.
  - **CN**: 该文件参与 Pass 的注册、调度或执行编排。
- **Runtime interop / 运行时互操作**:
  - **EN**: The code exposes or consumes ABI-level hooks used by generated code at runtime.
  - **CN**: 代码暴露或消费运行时由生成代码使用的 ABI 级钩子。
- **Streaming output / 流式输出**:
  - **EN**: Text, diagnostics, or generated content are emitted through LLVM stream APIs.
  - **CN**: 通过 LLVM 流 API 输出文本、诊断或生成内容。
- **LLVM ADT containers / LLVM ADT 容器**:
  - **EN**: The implementation uses LLVM containers for performance-conscious storage and traversal.
  - **CN**: 该实现使用 LLVM 容器来进行兼顾性能的存储与遍历。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `PassDetail.h`, `mlir/IR/SymbolTable.h`, `mlir/Pass/PassManager.h`, `mlir/Support/FileUtilities.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/StringExtras.h`, `llvm/Support/FileSystem.h`, `llvm/Support/FormatVariadic.h`, `llvm/Support/Path.h`, `llvm/Support/ToolOutputFile.h`
- **Subsystem categories / 子系统类别**: LLVM support-library helpers / LLVM Support 库辅助工具 (4), LLVM ADT containers and utilities / LLVM ADT 容器与工具 (2), core MLIR IR abstractions / 核心 MLIR IR 抽象 (1), pass-manager infrastructure / Pass 管理器基础设施 (1), shared MLIR support helpers / 共享的 MLIR 支持工具 (1)
