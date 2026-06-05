# MlirTblgenMain.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Tools/mlir-tblgen/MlirTblgenMain.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Main entry function for mlir-tblgen for when built as standalone binary.
  - **CN**: 实现构建于 MLIR 库之上的可复用命令行工具支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- MlirTblgenMain.cpp - MLIR Tablegen Driver main -----------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及高层摘要。

### Lines 8-14
```cpp
//
// Main entry function for mlir-tblgen for when built as standalone binary.
//
//===----------------------------------------------------------------------===//

#include "mlir/Tools/mlir-tblgen/MlirTblgenMain.h"

```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/Tools/mlir-tblgen/MlirTblgenMain.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/Tools/mlir-tblgen/MlirTblgenMain.h`。

### Lines 15-23
```cpp
#include "mlir/TableGen/GenInfo.h"
#include "mlir/TableGen/GenNameParser.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/InitLLVM.h"
#include "llvm/Support/Signals.h"
#include "llvm/TableGen/Error.h"
#include "llvm/TableGen/Main.h"
#include "llvm/TableGen/Record.h"

```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/TableGen/GenInfo.h`, `mlir/TableGen/GenNameParser.h`, `llvm/Support/CommandLine.h`, `llvm/Support/InitLLVM.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/TableGen/GenInfo.h`, `mlir/TableGen/GenNameParser.h`, `llvm/Support/CommandLine.h`, `llvm/Support/InitLLVM.h`。

### Lines 24-30
```cpp
using namespace mlir;
using namespace llvm;

enum DeprecatedAction { None, Warn, Error };

static DeprecatedAction actionOnDeprecatedValue;

```
- **EN**: Introduces declarations for `DeprecatedAction`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `DeprecatedAction` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 31-40
```cpp
// Returns if there is a use of `deprecatedInit` in `field`.
static bool findUse(const Init *field, const Init *deprecatedInit,
                    llvm::DenseMap<const Init *, bool> &known) {
  if (field == deprecatedInit)
    return true;

  auto it = known.find(field);
  if (it != known.end())
    return it->second;

```
- **EN**: Implements logic around `findUse`, `find`, `end`.
- **CN**: 围绕 `findUse`、`find`、`end` 实现具体逻辑。

### Lines 41-54
```cpp
  auto memoize = [&](bool val) {
    known[field] = val;
    return val;
  };

  if (auto *defInit = dyn_cast<DefInit>(field)) {
    // Only recurse into defs if they are anonymous.
    // Non-anonymous defs are handled by the main loop, with a proper
    // deprecation warning for each. Returning true here, would cause
    // all users of a def to also emit a deprecation warning.
    if (!defInit->getDef()->isAnonymous())
      // Purposefully not memoize as to not include every def use in the map.
      // This is also a trivial case we return false for in constant time.
      return false;
```
- **EN**: Implements logic around `dyn_cast`, `getDef`.
- **CN**: 围绕 `dyn_cast`、`getDef` 实现具体逻辑。

### Lines 55-61
```cpp

    return memoize(
        llvm::any_of(defInit->getDef()->getValues(), [&](const RecordVal &val) {
          return findUse(val.getValue(), deprecatedInit, known);
        }));
  }

```
- **EN**: Implements logic around `memoize`, `any_of`, `findUse`.
- **CN**: 围绕 `memoize`、`any_of`、`findUse` 实现具体逻辑。

### Lines 62-70
```cpp
  if (auto *dagInit = dyn_cast<DagInit>(field)) {
    if (findUse(dagInit->getOperator(), deprecatedInit, known))
      return memoize(true);

    return memoize(llvm::any_of(dagInit->getArgs(), [&](const Init *arg) {
      return findUse(arg, deprecatedInit, known);
    }));
  }

```
- **EN**: Implements logic around `dyn_cast`, `findUse`, `memoize`.
- **CN**: 围绕 `dyn_cast`、`findUse`、`memoize` 实现具体逻辑。

### Lines 71-83
```cpp
  if (const ListInit *li = dyn_cast<ListInit>(field)) {
    return memoize(llvm::any_of(li->getElements(), [&](const Init *jt) {
      return findUse(jt, deprecatedInit, known);
    }));
  }

  // Purposefully don't use memoize here. There is no need to cache the result
  // for every kind of init (e.g. BitInit or StringInit), which will always
  // return false. Doing so would grow the DenseMap to include almost every Init
  // within the main file.
  return false;
}

```
- **EN**: Implements logic around `dyn_cast`, `memoize`, `findUse`.
- **CN**: 围绕 `dyn_cast`、`memoize`、`findUse` 实现具体逻辑。

### Lines 84-91
```cpp
// Returns if there is a use of `deprecatedInit` in `record`.
static bool findUse(Record &record, const Init *deprecatedInit,
                    llvm::DenseMap<const Init *, bool> &known) {
  return llvm::any_of(record.getValues(), [&](const RecordVal &val) {
    return findUse(val.getValue(), deprecatedInit, known);
  });
}

```
- **EN**: Implements logic around `findUse`, `any_of`.
- **CN**: 围绕 `findUse`、`any_of` 实现具体逻辑。

### Lines 92-101
```cpp
static void warnOfDeprecatedUses(const RecordKeeper &records) {
  // This performs a direct check for any def marked as deprecated and then
  // finds all uses of deprecated def. Deprecated defs are not expected to be
  // either numerous or long lived.
  bool deprecatedDefsFounds = false;
  for (auto &it : records.getDefs()) {
    const RecordVal *r = it.second->getValue("odsDeprecated");
    if (!r || !r->getValue())
      continue;

```
- **EN**: Implements logic around `warnOfDeprecatedUses`, `getDefs`, `getValue`; this block processes TableGen records or generates derived code.
- **CN**: 围绕 `warnOfDeprecatedUses`、`getDefs`、`getValue` 实现具体逻辑；该代码块处理 TableGen 记录或生成派生代码。

### Lines 102-108
```cpp
    llvm::DenseMap<const Init *, bool> hasUse;
    if (auto *si = dyn_cast<StringInit>(r->getValue())) {
      for (auto &jt : records.getDefs()) {
        // Skip anonymous defs.
        if (jt.second->isAnonymous())
          continue;

```
- **EN**: Implements logic around `dyn_cast`, `getDefs`, `isAnonymous`.
- **CN**: 围绕 `dyn_cast`、`getDefs`、`isAnonymous` 实现具体逻辑。

### Lines 109-122
```cpp
        if (findUse(*jt.second, it.second->getDefInit(), hasUse)) {
          PrintWarning(jt.second->getLoc(),
                       "Using deprecated def `" + it.first + "`");
          PrintNote(si->getAsUnquotedString());
          deprecatedDefsFounds = true;
        }
      }
    }
  }
  if (deprecatedDefsFounds &&
      actionOnDeprecatedValue == DeprecatedAction::Error)
    PrintFatalNote("Error'ing out due to deprecated defs");
}

```
- **EN**: Implements logic around `findUse`, `PrintWarning`, `PrintNote`, `PrintFatalNote`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `findUse`、`PrintWarning`、`PrintNote`、`PrintFatalNote` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 123-131
```cpp
// Generator to invoke.
static const mlir::GenInfo *generator;

// TableGenMain requires a function pointer so this function is passed in which
// simply wraps the call to the generator.
static bool mlirTableGenMain(raw_ostream &os, const RecordKeeper &records) {
  if (actionOnDeprecatedValue != DeprecatedAction::None)
    warnOfDeprecatedUses(records);

```
- **EN**: Implements logic around `mlirTableGenMain`, `warnOfDeprecatedUses`; this block processes TableGen records or generates derived code.
- **CN**: 围绕 `mlirTableGenMain`、`warnOfDeprecatedUses` 实现具体逻辑；该代码块处理 TableGen 记录或生成派生代码。

### Lines 132-138
```cpp
  if (!generator) {
    os << records;
    return false;
  }
  return generator->invoke(records, os);
}

```
- **EN**: Implements logic around `invoke`.
- **CN**: 围绕 `invoke` 实现具体逻辑。

### Lines 139-150
```cpp
int mlir::MlirTblgenMain(int argc, char **argv) {

  llvm::InitLLVM y(argc, argv);

  llvm::cl::opt<DeprecatedAction, true> actionOnDeprecated(
      "on-deprecated", llvm::cl::desc("Action to perform on deprecated def"),
      llvm::cl::values(
          clEnumValN(DeprecatedAction::None, "none", "No action"),
          clEnumValN(DeprecatedAction::Warn, "warn", "Warn on use"),
          clEnumValN(DeprecatedAction::Error, "error", "Error on use")),
      cl::location(actionOnDeprecatedValue), llvm::cl::init(Warn));

```
- **EN**: Implements logic around `MlirTblgenMain`, `y`, `actionOnDeprecated`, `desc`, and 3 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `MlirTblgenMain`、`y`、`actionOnDeprecated`、`desc` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 151-164
```cpp
  llvm::cl::opt<const mlir::GenInfo *, true, mlir::GenNameParser> generator(
      "", llvm::cl::desc("Generator to run"), cl::location(::generator));

  cl::ParseCommandLineOptions(argc, argv);

  return TableGenMain(
      argv[0], [](TableGenOutputFiles &OutFiles, const RecordKeeper &RK) {
        std::string S;
        raw_string_ostream OS(S);
        bool Res = mlirTableGenMain(OS, RK);
        OutFiles = {S, {}};
        return Res;
      });
}
```
- **EN**: Implements logic around `generator`, `desc`, `ParseCommandLineOptions`, `TableGenMain`, and 2 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations; processes TableGen records or generates derived code.
- **CN**: 围绕 `generator`、`desc`、`ParseCommandLineOptions`、`TableGenMain` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示，并处理 TableGen 记录或生成派生代码。

## Key Concepts / 关键概念

- **Tooling support / 工具链支持**:
  - **EN**: Builds reusable infrastructure for opt-like tools, translation drivers, and CLIs.
  - **CN**: 为类似 opt 的工具、翻译驱动和命令行程序构建可复用基础设施。
- **TableGen-driven generation / TableGen 驱动生成**:
  - **EN**: Declarative records are converted into generated MLIR declarations or implementation fragments.
  - **CN**: 把声明式记录转换为生成的 MLIR 声明或实现片段。
- **Streaming output / 流式输出**:
  - **EN**: Text, diagnostics, or generated content are emitted through LLVM stream APIs.
  - **CN**: 通过 LLVM 流 API 输出文本、诊断或生成内容。
- **LLVM ADT containers / LLVM ADT 容器**:
  - **EN**: The implementation uses LLVM containers for performance-conscious storage and traversal.
  - **CN**: 该实现使用 LLVM 容器来进行兼顾性能的存储与遍历。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `mlir/Tools/mlir-tblgen/MlirTblgenMain.h`, `mlir/TableGen/GenInfo.h`, `mlir/TableGen/GenNameParser.h`, `llvm/Support/CommandLine.h`, `llvm/Support/InitLLVM.h`, `llvm/Support/Signals.h`, `llvm/TableGen/Error.h`, `llvm/TableGen/Main.h`, `llvm/TableGen/Record.h`
- **Subsystem categories / 子系统类别**: LLVM support-library helpers / LLVM Support 库辅助工具 (3), LLVM TableGen infrastructure / LLVM TableGen 基础设施 (3), TableGen backend support / TableGen 后端支持 (2), tooling support declarations / 工具支持声明 (1)
