# Query.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Query/Query.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements MLIR query facilities, matcher infrastructure, or command-line querying behavior.
  - **CN**: 实现 MLIR 查询能力、匹配器基础设施或命令行查询行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===---- Query.cpp - -----------------------------------------------------===//
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

#include "mlir/Query/Query.h"
#include "QueryParser.h"
#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/IR/IRMapping.h"
#include "mlir/IR/Verifier.h"
#include "mlir/Query/Matcher/MatchFinder.h"
#include "mlir/Query/QuerySession.h"
#include "llvm/Support/SourceMgr.h"
#include "llvm/Support/raw_ostream.h"
```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/Query/Query.h`, `QueryParser.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/IR/IRMapping.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/Query/Query.h`, `QueryParser.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/IR/IRMapping.h`。

### Lines 18-24
```cpp

namespace mlir::query {

QueryRef parse(llvm::StringRef line, const QuerySession &qs) {
  return QueryParser::parse(line, qs);
}

```
- **EN**: Introduces declarations for `mlir::query`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `mlir::query` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 25-29
```cpp
std::vector<llvm::LineEditor::Completion>
complete(llvm::StringRef line, size_t pos, const QuerySession &qs) {
  return QueryParser::complete(line, pos, qs);
}

```
- **EN**: Implements logic around `complete`.
- **CN**: 围绕 `complete` 实现具体逻辑。

### Lines 30-37
```cpp
// TODO: Extract into a helper function that can be reused outside query
// context.
static Operation *extractFunction(std::vector<Operation *> &ops,
                                  MLIRContext *context,
                                  llvm::StringRef functionName) {
  context->loadDialect<func::FuncDialect>();
  OpBuilder builder(context);

```
- **EN**: Implements logic around `extractFunction`, `FuncDialect>`, `builder`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `extractFunction`、`FuncDialect>`、`builder` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 38-42
```cpp
  // Collect data for function creation
  std::vector<Operation *> slice;
  std::vector<Value> values;
  std::vector<Type> outputTypes;

```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 43-47
```cpp
  for (auto *op : ops) {
    // Return op's operands are propagated, but the op itself isn't needed.
    if (!isa<func::ReturnOp>(op))
      slice.push_back(op);

```
- **EN**: Implements logic around `ReturnOp>`, `push_back`.
- **CN**: 围绕 `ReturnOp>`、`push_back` 实现具体逻辑。

### Lines 48-54
```cpp
    // All results are returned by the extracted function.
    llvm::append_range(outputTypes, op->getResults().getTypes());

    // Track all values that need to be taken as input to function.
    llvm::append_range(values, op->getOperands());
  }

```
- **EN**: Implements logic around `append_range`.
- **CN**: 围绕 `append_range` 实现具体逻辑。

### Lines 55-60
```cpp
  // Create the function
  FunctionType funcType =
      builder.getFunctionType(TypeRange(ValueRange(values)), outputTypes);
  auto loc = builder.getUnknownLoc();
  func::FuncOp funcOp = func::FuncOp::create(loc, functionName, funcType);

```
- **EN**: Implements logic around `getFunctionType`, `getUnknownLoc`, `create`.
- **CN**: 围绕 `getFunctionType`、`getUnknownLoc`、`create` 实现具体逻辑。

### Lines 61-67
```cpp
  builder.setInsertionPointToEnd(funcOp.addEntryBlock());

  // Map original values to function arguments
  IRMapping mapper;
  for (const auto &arg : llvm::enumerate(values))
    mapper.map(arg.value(), funcOp.getArgument(arg.index()));

```
- **EN**: Implements logic around `setInsertionPointToEnd`, `enumerate`, `map`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `setInsertionPointToEnd`、`enumerate`、`map` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 68-77
```cpp
  // Clone operations and build function body
  std::vector<Operation *> clonedOps;
  std::vector<Value> clonedVals;
  // TODO: Handle extraction of operations with compute payloads defined via
  // regions.
  for (Operation *slicedOp : slice) {
    Operation *clonedOp =
        clonedOps.emplace_back(builder.clone(*slicedOp, mapper));
    clonedVals.insert(clonedVals.end(), clonedOp->result_begin(),
                      clonedOp->result_end());
```
- **EN**: Implements logic around `emplace_back`, `insert`, `result_end`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `emplace_back`、`insert`、`result_end` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 78-87
```cpp
  }
  // Add return operation
  func::ReturnOp::create(builder, loc, clonedVals);

  // Remove unused function arguments
  size_t currentIndex = 0;
  while (currentIndex < funcOp.getNumArguments()) {
    // Erase if possible.
    if (funcOp.getArgument(currentIndex).use_empty())
      if (succeeded(funcOp.eraseArgument(currentIndex)))
```
- **EN**: Implements logic around `create`, `getNumArguments`, `getArgument`, `succeeded`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `create`、`getNumArguments`、`getArgument`、`succeeded` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 88-94
```cpp
        continue;
    ++currentIndex;
  }

  return funcOp;
}

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 95-101
```cpp
Query::~Query() = default;

LogicalResult InvalidQuery::run(llvm::raw_ostream &os, QuerySession &qs) const {
  os << errStr << "\n";
  return mlir::failure();
}

```
- **EN**: Implements logic around `~Query`, `run`, `failure`; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `~Query`、`run`、`failure` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 102-111
```cpp
LogicalResult NoOpQuery::run(llvm::raw_ostream &os, QuerySession &qs) const {
  return mlir::success();
}

LogicalResult HelpQuery::run(llvm::raw_ostream &os, QuerySession &qs) const {
  os << "Available commands:\n\n"
        "  match MATCHER, m MATCHER      "
        "Match the mlir against the given matcher.\n"
        "  quit                              "
        "Terminates the query session.\n\n";
```
- **EN**: Implements logic around `run`, `success`; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `run`、`success` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 112-119
```cpp
  return mlir::success();
}

LogicalResult QuitQuery::run(llvm::raw_ostream &os, QuerySession &qs) const {
  qs.terminate = true;
  return mlir::success();
}

```
- **EN**: Implements logic around `success`, `run`; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `success`、`run` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 120-124
```cpp
LogicalResult MatchQuery::run(llvm::raw_ostream &os, QuerySession &qs) const {
  Operation *rootOp = qs.getRootOp();
  int matchCount = 0;
  matcher::MatchFinder finder;

```
- **EN**: Implements logic around `run`, `getRootOp`; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects.
- **CN**: 围绕 `run`、`getRootOp` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象。

### Lines 125-134
```cpp
  StringRef functionName = matcher.getFunctionName();
  auto matches = finder.collectMatches(rootOp, std::move(matcher));

  // An extract call is recognized by considering if the matcher has a name.
  // TODO: Consider making the extract more explicit.
  if (!functionName.empty()) {
    std::vector<Operation *> flattenedMatches =
        finder.flattenMatchedOps(matches);
    Operation *function =
        extractFunction(flattenedMatches, rootOp->getContext(), functionName);
```
- **EN**: Implements logic around `getFunctionName`, `collectMatches`, `empty`, `flattenMatchedOps`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getFunctionName`、`collectMatches`、`empty`、`flattenMatchedOps` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 135-141
```cpp
    if (failed(verify(function)))
      return mlir::failure();
    os << "\n" << *function << "\n\n";
    function->erase();
    return mlir::success();
  }

```
- **EN**: Implements logic around `failed`, `failure`, `erase`, `success`.
- **CN**: 围绕 `failed`、`failure`、`erase`、`success` 实现具体逻辑。

### Lines 142-151
```cpp
  os << "\n";
  for (auto &results : matches) {
    os << "Match #" << ++matchCount << ":\n\n";
    for (Operation *op : results.matchedOps) {
      if (op == results.rootOp) {
        finder.printMatch(os, qs, op, "root");
      } else {
        finder.printMatch(os, qs, op);
      }
    }
```
- **EN**: Implements logic around `printMatch`; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `printMatch` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 152-156
```cpp
  }
  os << matchCount << (matchCount == 1 ? " match.\n\n" : " matches.\n\n");
  return mlir::success();
}

```
- **EN**: Implements logic around `success`.
- **CN**: 围绕 `success` 实现具体逻辑。

### Lines 157-157
```cpp
} // namespace mlir::query
```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

## Key Concepts / 关键概念

- **IR querying / IR 查询**:
  - **EN**: Supports querying or filtering IR according to structural or semantic predicates.
  - **CN**: 支持依据结构或语义谓词对 IR 进行查询或过滤。
- **SSA and region-based IR / SSA 与基于 Region 的 IR**:
  - **EN**: The file works with MLIR operations, blocks, values, or regions that form MLIR's hierarchical SSA IR.
  - **CN**: 该文件处理 MLIR 操作、块、值或 Region，它们共同构成 MLIR 的层次化 SSA IR。
- **Structured diagnostics / 结构化诊断**:
  - **EN**: Failures are surfaced through `LogicalResult`, diagnostics, or related reporting helpers.
  - **CN**: 通过 `LogicalResult`、诊断对象或相关报告工具显式传播失败。
- **Streaming output / 流式输出**:
  - **EN**: Text, diagnostics, or generated content are emitted through LLVM stream APIs.
  - **CN**: 通过 LLVM 流 API 输出文本、诊断或生成内容。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `mlir/Query/Query.h`, `QueryParser.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/IR/IRMapping.h`, `mlir/IR/Verifier.h`, `mlir/Query/Matcher/MatchFinder.h`, `mlir/Query/QuerySession.h`, `llvm/Support/SourceMgr.h`, `llvm/Support/raw_ostream.h`
- **Subsystem categories / 子系统类别**: query infrastructure / 查询基础设施 (3), core MLIR IR abstractions / 核心 MLIR IR 抽象 (2), LLVM support-library helpers / LLVM Support 库辅助工具 (2)
