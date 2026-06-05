# OpStats.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Transforms/OpStats.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements core MLIR transformation passes and transformation utilities.
  - **CN**: 实现核心 MLIR 变换 Pass 与变换辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- OpStats.cpp - Prints stats of operations in module -----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及高层摘要。

### Lines 8-15
```cpp

#include "mlir/Transforms/Passes.h"

#include "mlir/IR/Operation.h"
#include "mlir/IR/OperationSupport.h"
#include "llvm/Support/Format.h"
#include "llvm/Support/raw_ostream.h"

```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/Transforms/Passes.h`, `mlir/IR/Operation.h`, `mlir/IR/OperationSupport.h`, `llvm/Support/Format.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/Transforms/Passes.h`, `mlir/IR/Operation.h`, `mlir/IR/OperationSupport.h`, `llvm/Support/Format.h`。

### Lines 16-20
```cpp
namespace mlir {
#define GEN_PASS_DEF_PRINTOPSTATSPASS
#include "mlir/Transforms/Passes.h.inc"
} // namespace mlir

```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/Transforms/Passes.h.inc`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/Transforms/Passes.h.inc`。

### Lines 21-26
```cpp
using namespace mlir;

namespace {
struct PrintOpStatsPass : public impl::PrintOpStatsPassBase<PrintOpStatsPass> {
  using impl::PrintOpStatsPassBase<PrintOpStatsPass>::PrintOpStatsPassBase;

```
- **EN**: Introduces declarations for `PrintOpStatsPass`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `PrintOpStatsPass` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 27-32
```cpp
  explicit PrintOpStatsPass(raw_ostream &os) : os(&os) {}

  explicit PrintOpStatsPass(raw_ostream &os, bool printAsJSON) : os(&os) {
    this->printAsJSON = printAsJSON;
  }

```
- **EN**: Implements logic around `PrintOpStatsPass`; this block parses or prints textual MLIR representations; implements transformation or simplification logic.
- **CN**: 围绕 `PrintOpStatsPass` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示，并实现变换或简化逻辑。

### Lines 33-38
```cpp
  // Prints the resultant operation statistics post iterating over the module.
  void runOnOperation() override;

  // Print summary of op stats.
  void printSummary();

```
- **EN**: Implements logic around `runOnOperation`, `printSummary`; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations; implements transformation or simplification logic.
- **CN**: 围绕 `runOnOperation`、`printSummary` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示，并实现变换或简化逻辑。

### Lines 39-47
```cpp
  // Print symmary of op stats in JSON.
  void printSummaryInJSON();

private:
  llvm::StringMap<int64_t> opCount;
  raw_ostream *os = &llvm::errs();
};
} // namespace

```
- **EN**: Implements logic around `printSummaryInJSON`, `errs`; this block parses or prints textual MLIR representations; implements transformation or simplification logic.
- **CN**: 围绕 `printSummaryInJSON`、`errs` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示，并实现变换或简化逻辑。

### Lines 48-57
```cpp
void PrintOpStatsPass::runOnOperation() {
  opCount.clear();

  // Compute the operation statistics for the currently visited operation.
  getOperation()->walk(
      [&](Operation *op) { ++opCount[op->getName().getStringRef()]; });
  if (printAsJSON)
    printSummaryInJSON();
  else
    printSummary();
```
- **EN**: Implements logic around `runOnOperation`, `clear`, `getOperation`, `getName`, and 2 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations; implements transformation or simplification logic.
- **CN**: 围绕 `runOnOperation`、`clear`、`getOperation`、`getName` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示，并实现变换或简化逻辑。

### Lines 58-66
```cpp
  markAllAnalysesPreserved();
}

void PrintOpStatsPass::printSummary() {
  *os << "Operations encountered:\n";
  *os << "-----------------------\n";
  SmallVector<StringRef, 64> sorted(opCount.keys());
  llvm::sort(sorted);

```
- **EN**: Implements logic around `markAllAnalysesPreserved`, `printSummary`, `sorted`, `sort`; this block parses or prints textual MLIR representations; implements transformation or simplification logic.
- **CN**: 围绕 `markAllAnalysesPreserved`、`printSummary`、`sorted`、`sort` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示，并实现变换或简化逻辑。

### Lines 67-73
```cpp
  // Split an operation name from its dialect prefix.
  auto splitOperationName = [](StringRef opName) {
    auto splitName = opName.split('.');
    return splitName.second.empty() ? std::make_pair("", splitName.first)
                                    : splitName;
  };

```
- **EN**: Implements logic around `split`, `empty`; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `split`、`empty` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 74-81
```cpp
  // Compute the largest dialect and operation name.
  size_t maxLenOpName = 0, maxLenDialect = 0;
  for (const auto &key : sorted) {
    auto [dialectName, opName] = splitOperationName(key);
    maxLenDialect = std::max(maxLenDialect, dialectName.size());
    maxLenOpName = std::max(maxLenOpName, opName.size());
  }

```
- **EN**: Implements logic around `splitOperationName`, `max`; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `splitOperationName`、`max` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 82-91
```cpp
  for (const auto &key : sorted) {
    auto [dialectName, opName] = splitOperationName(key);

    // Left-align the names (aligning on the dialect) and right-align the count
    // below. The alignment is for readability and does not affect CSV/FileCheck
    // parsing.
    if (dialectName.empty())
      os->indent(maxLenDialect + 3);
    else
      *os << llvm::right_justify(dialectName, maxLenDialect + 2) << '.';
```
- **EN**: Implements logic around `splitOperationName`, `empty`, `indent`, `right_justify`; this block implements transformation or simplification logic.
- **CN**: 围绕 `splitOperationName`、`empty`、`indent`、`right_justify` 实现具体逻辑；该代码块实现变换或简化逻辑。

### Lines 92-98
```cpp

    // Left justify the operation name.
    *os << llvm::left_justify(opName, maxLenOpName) << " , " << opCount[key]
        << '\n';
  }
}

```
- **EN**: Implements logic around `left_justify`; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `left_justify` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 99-104
```cpp
void PrintOpStatsPass::printSummaryInJSON() {
  SmallVector<StringRef, 64> sorted(opCount.keys());
  llvm::sort(sorted);

  *os << "{\n";

```
- **EN**: Implements logic around `printSummaryInJSON`, `sorted`, `sort`; this block parses or prints textual MLIR representations; implements transformation or simplification logic.
- **CN**: 围绕 `printSummaryInJSON`、`sorted`、`sort` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示，并实现变换或简化逻辑。

### Lines 105-114
```cpp
  for (unsigned i = 0, e = sorted.size(); i != e; ++i) {
    const auto &key = sorted[i];
    *os << "  \"" << key << "\" : " << opCount[key];
    if (i != e - 1)
      *os << ",\n";
    else
      *os << "\n";
  }
  *os << "}\n";
}
```
- **EN**: Implements logic around `size`; this block implements transformation or simplification logic.
- **CN**: 围绕 `size` 实现具体逻辑；该代码块实现变换或简化逻辑。

### Lines 115-119
```cpp

std::unique_ptr<Pass> mlir::createPrintOpStatsPass(raw_ostream &os) {
  return std::make_unique<PrintOpStatsPass>(os);
}

```
- **EN**: Implements logic around `createPrintOpStatsPass`, `make_unique`; this block parses or prints textual MLIR representations; implements transformation or simplification logic.
- **CN**: 围绕 `createPrintOpStatsPass`、`make_unique` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示，并实现变换或简化逻辑。

### Lines 120-123
```cpp
std::unique_ptr<Pass> mlir::createPrintOpStatsPass(raw_ostream &os,
                                                   bool printAsJSON) {
  return std::make_unique<PrintOpStatsPass>(os, printAsJSON);
}
```
- **EN**: Implements logic around `createPrintOpStatsPass`, `make_unique`; this block parses or prints textual MLIR representations; implements transformation or simplification logic.
- **CN**: 围绕 `createPrintOpStatsPass`、`make_unique` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示，并实现变换或简化逻辑。

## Key Concepts / 关键概念

- **IR transformation passes / IR 变换 Pass**:
  - **EN**: Applies canonicalization, dataflow-aware simplification, or structural rewrites to MLIR.
  - **CN**: 对 MLIR 应用规范化、数据流感知简化或结构性重写。
- **SSA and region-based IR / SSA 与基于 Region 的 IR**:
  - **EN**: The file works with MLIR operations, blocks, values, or regions that form MLIR's hierarchical SSA IR.
  - **CN**: 该文件处理 MLIR 操作、块、值或 Region，它们共同构成 MLIR 的层次化 SSA IR。
- **Streaming output / 流式输出**:
  - **EN**: Text, diagnostics, or generated content are emitted through LLVM stream APIs.
  - **CN**: 通过 LLVM 流 API 输出文本、诊断或生成内容。
- **LLVM ADT containers / LLVM ADT 容器**:
  - **EN**: The implementation uses LLVM containers for performance-conscious storage and traversal.
  - **CN**: 该实现使用 LLVM 容器来进行兼顾性能的存储与遍历。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `mlir/Transforms/Passes.h`, `mlir/IR/Operation.h`, `mlir/IR/OperationSupport.h`, `llvm/Support/Format.h`, `llvm/Support/raw_ostream.h`, `mlir/Transforms/Passes.h.inc`
- **Subsystem categories / 子系统类别**: core transformation utilities / 核心变换工具 (2), core MLIR IR abstractions / 核心 MLIR IR 抽象 (2), LLVM support-library helpers / LLVM Support 库辅助工具 (2)
