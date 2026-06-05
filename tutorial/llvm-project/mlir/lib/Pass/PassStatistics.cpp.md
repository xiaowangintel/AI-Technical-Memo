# PassStatistics.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Pass/PassStatistics.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements pass-manager infrastructure, pass registration, and pass execution support.
  - **CN**: 实现 Pass 管理器基础设施、Pass 注册以及执行支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- PassStatistics.cpp -------------------------------------------------===//
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

#include "PassDetail.h"
#include "mlir/Pass/PassManager.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/Support/Format.h"

using namespace mlir;
using namespace mlir::detail;

```
- **EN**: Pulls in the declarations needed by this translation unit, including `PassDetail.h`, `mlir/Pass/PassManager.h`, `llvm/ADT/StringExtras.h`, `llvm/Support/Format.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `PassDetail.h`, `mlir/Pass/PassManager.h`, `llvm/ADT/StringExtras.h`, `llvm/Support/Format.h`。

### Lines 17-27
```cpp
constexpr StringLiteral kPassStatsDescription =
    "... Pass statistics report ...";

namespace {
/// Information pertaining to a specific statistic.
struct Statistic {
  const char *name, *desc;
  uint64_t value;
};
} // namespace

```
- **EN**: Introduces declarations for `Statistic`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `Statistic` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 28-34
```cpp
/// Utility to print a pass entry in the statistics output.
static void printPassEntry(raw_ostream &os, unsigned indent, StringRef pass,
                           MutableArrayRef<Statistic> stats = {}) {
  os.indent(indent) << pass << "\n";
  if (stats.empty())
    return;

```
- **EN**: Implements logic around `printPassEntry`, `indent`, `empty`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `printPassEntry`、`indent`、`empty` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 35-48
```cpp
  // Make sure to sort the statistics by name.
  llvm::array_pod_sort(
      stats.begin(), stats.end(), [](const auto *lhs, const auto *rhs) {
        return StringRef{lhs->name}.compare(StringRef{rhs->name});
      });

  // Collect the largest name and value length from each of the statistics.
  size_t largestName = 0, largestValue = 0;
  for (auto &stat : stats) {
    largestName = std::max(largestName, (size_t)strlen(stat.name));
    largestValue =
        std::max(largestValue, (size_t)llvm::utostr(stat.value).size());
  }

```
- **EN**: Implements logic around `array_pod_sort`, `begin`, `compare`, `max`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `array_pod_sort`、`begin`、`compare`、`max` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 49-56
```cpp
  // Print each of the statistics.
  for (auto &stat : stats) {
    os.indent(indent + 2) << llvm::format("(S) %*u %-*s - %s\n", largestValue,
                                          stat.value, largestName, stat.name,
                                          stat.desc);
  }
}

```
- **EN**: Implements logic around `indent`; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `indent` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 57-63
```cpp
/// Print the statistics results in a list form, where each pass is sorted by
/// name.
static void printResultsAsList(raw_ostream &os, OpPassManager &pm) {
  llvm::StringMap<std::vector<Statistic>> mergedStats;
  std::function<void(Pass *)> addStats = [&](Pass *pass) {
    auto *adaptor = dyn_cast<OpToOpPassAdaptor>(pass);

```
- **EN**: Implements logic around `printResultsAsList`, `function`, `dyn_cast`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `printResultsAsList`、`function`、`dyn_cast` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 64-70
```cpp
    // If this is not an adaptor, add the stats to the list if there are any.
    if (!adaptor) {
#if LLVM_ENABLE_STATS
      auto statistics = pass->getStatistics();
      if (statistics.empty())
        return;

```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 71-82
```cpp
      auto &passEntry = mergedStats[pass->getName()];
      if (passEntry.empty()) {
        for (Pass::Statistic *it : pass->getStatistics())
          passEntry.push_back({it->getName(), it->getDesc(), it->getValue()});
      } else {
        for (auto [idx, statistic] : llvm::enumerate(pass->getStatistics()))
          passEntry[idx].value += statistic->getValue();
      }
#endif
      return;
    }

```
- **EN**: Implements logic around `getName`, `empty`, `getStatistics`, `push_back`, and 2 more symbols; this block coordinates pass execution or analysis state; manipulates core MLIR IR objects.
- **CN**: 围绕 `getName`、`empty`、`getStatistics`、`push_back` 等另外 2 个符号 实现具体逻辑；该代码块协调 Pass 执行或分析状态，并操作 MLIR 核心 IR 对象。

### Lines 83-90
```cpp
    // Otherwise, recursively add each of the children.
    for (auto &mgr : adaptor->getPassManagers())
      for (Pass &pass : mgr.getPasses())
        addStats(&pass);
  };
  for (Pass &pass : pm.getPasses())
    addStats(&pass);

```
- **EN**: Implements logic around `getPassManagers`, `getPasses`, `addStats`.
- **CN**: 围绕 `getPassManagers`、`getPasses`、`addStats` 实现具体逻辑。

### Lines 91-99
```cpp
  // Sort the statistics by pass name and then by record name.
  auto passAndStatistics =
      llvm::to_vector<16>(llvm::make_pointer_range(mergedStats));
  llvm::array_pod_sort(passAndStatistics.begin(), passAndStatistics.end(),
                       [](const decltype(passAndStatistics)::value_type *lhs,
                          const decltype(passAndStatistics)::value_type *rhs) {
                         return (*lhs)->getKey().compare((*rhs)->getKey());
                       });

```
- **EN**: Implements logic around `to_vector`, `array_pod_sort`, `decltype`, `getKey`.
- **CN**: 围绕 `to_vector`、`array_pod_sort`、`decltype`、`getKey` 实现具体逻辑。

### Lines 100-106
```cpp
  // Print the timing information sequentially.
  for (auto &statData : passAndStatistics)
    printPassEntry(os, /*indent=*/2, statData->first(), statData->second);
}

/// Print the results in pipeline mode that mirrors the internal pass manager
/// structure.
```
- **EN**: Implements logic around `printPassEntry`; this block coordinates pass execution or analysis state; parses or prints textual MLIR representations.
- **CN**: 围绕 `printPassEntry` 实现具体逻辑；该代码块协调 Pass 执行或分析状态，并解析或打印文本形式的 MLIR 表示。

### Lines 107-119
```cpp
static void printResultsAsPipeline(raw_ostream &os, OpPassManager &pm) {
#if LLVM_ENABLE_STATS
  std::function<void(unsigned, Pass *)> printPass = [&](unsigned indent,
                                                        Pass *pass) {
    if (auto *adaptor = dyn_cast<OpToOpPassAdaptor>(pass)) {
      // If this adaptor has more than one internal pipeline, print an entry for
      // it.
      auto mgrs = adaptor->getPassManagers();
      if (mgrs.size() > 1) {
        printPassEntry(os, indent, adaptor->getAdaptorName());
        indent += 2;
      }

```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 120-129
```cpp
      // Print each of the children passes.
      for (OpPassManager &mgr : mgrs) {
        auto name = ("'" + mgr.getOpAnchorName() + "' Pipeline").str();
        printPassEntry(os, indent, name);
        for (Pass &pass : mgr.getPasses())
          printPass(indent + 2, &pass);
      }
      return;
    }

```
- **EN**: Implements logic around `getOpAnchorName`, `printPassEntry`, `getPasses`, `printPass`; this block coordinates pass execution or analysis state; parses or prints textual MLIR representations.
- **CN**: 围绕 `getOpAnchorName`、`printPassEntry`、`getPasses`、`printPass` 实现具体逻辑；该代码块协调 Pass 执行或分析状态，并解析或打印文本形式的 MLIR 表示。

### Lines 130-140
```cpp
    // Otherwise, we print the statistics for this pass.
    std::vector<Statistic> stats;
    for (Pass::Statistic *stat : pass->getStatistics())
      stats.push_back({stat->getName(), stat->getDesc(), stat->getValue()});
    printPassEntry(os, indent, pass->getName(), stats);
  };
  for (Pass &pass : pm.getPasses())
    printPass(/*indent=*/0, &pass);
#endif
}

```
- **EN**: Implements logic around `getStatistics`, `push_back`, `printPassEntry`, `getPasses`, and 1 more symbols; this block coordinates pass execution or analysis state; parses or prints textual MLIR representations.
- **CN**: 围绕 `getStatistics`、`push_back`、`printPassEntry`、`getPasses` 等另外 1 个符号 实现具体逻辑；该代码块协调 Pass 执行或分析状态，并解析或打印文本形式的 MLIR 表示。

### Lines 141-150
```cpp
static void printStatistics(OpPassManager &pm, PassDisplayMode displayMode) {
  auto os = llvm::CreateInfoOutputFile();

  // Print the stats header.
  *os << "===" << std::string(73, '-') << "===\n";
  // Figure out how many spaces for the description name.
  unsigned padding = (80 - kPassStatsDescription.size()) / 2;
  os->indent(padding) << kPassStatsDescription << '\n';
  *os << "===" << std::string(73, '-') << "===\n";

```
- **EN**: Implements logic around `printStatistics`, `CreateInfoOutputFile`, `string`, `size`, and 1 more symbols; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `printStatistics`、`CreateInfoOutputFile`、`string`、`size` 等另外 1 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 151-163
```cpp
  // Defer to a specialized printer for each display mode.
  switch (displayMode) {
  case PassDisplayMode::List:
    printResultsAsList(*os, pm);
    break;
  case PassDisplayMode::Pipeline:
    printResultsAsPipeline(*os, pm);
    break;
  }
  *os << "\n";
  os->flush();
}

```
- **EN**: Implements logic around `printResultsAsList`, `printResultsAsPipeline`, `flush`; this block coordinates pass execution or analysis state; parses or prints textual MLIR representations.
- **CN**: 围绕 `printResultsAsList`、`printResultsAsPipeline`、`flush` 实现具体逻辑；该代码块协调 Pass 执行或分析状态，并解析或打印文本形式的 MLIR 表示。

### Lines 164-177
```cpp
//===----------------------------------------------------------------------===//
// PassStatistics
//===----------------------------------------------------------------------===//

Pass::Statistic::Statistic(Pass *owner, const char *name,
                           const char *description)
    : llvm::Statistic{/*DebugType=*/"", name, description} {
#if LLVM_ENABLE_STATS
  // Always set the 'initialized' bit to true so that this statistic isn't
  // placed in the static registry.
  // TODO: This is sort of hack as `llvm::Statistic`s can't be setup to avoid
  // automatic registration with the global registry. We should either add
  // support for this in LLVM, or just write our own statistics classes.
  Initialized = true;
```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 178-188
```cpp
#endif

  // Register this statistic with the parent.
  owner->statistics.push_back(this);
}

auto Pass::Statistic::operator=(unsigned value) -> Statistic & {
  llvm::Statistic::operator=(value);
  return *this;
}

```
- **EN**: Implements logic around `push_back`; this block coordinates pass execution or analysis state; manipulates core MLIR IR objects.
- **CN**: 围绕 `push_back` 实现具体逻辑；该代码块协调 Pass 执行或分析状态，并操作 MLIR 核心 IR 对象。

### Lines 189-196
```cpp
//===----------------------------------------------------------------------===//
// PassManager
//===----------------------------------------------------------------------===//

/// Merge the pass statistics of this class into 'other'.
void OpPassManager::mergeStatisticsInto(OpPassManager &other) {
  auto passes = getPasses(), otherPasses = other.getPasses();

```
- **EN**: Implements logic around `mergeStatisticsInto`, `getPasses`; this block coordinates pass execution or analysis state.
- **CN**: 围绕 `mergeStatisticsInto`、`getPasses` 实现具体逻辑；该代码块协调 Pass 执行或分析状态。

### Lines 197-210
```cpp
  for (auto passPair : llvm::zip(passes, otherPasses)) {
    Pass &pass = std::get<0>(passPair), &otherPass = std::get<1>(passPair);

    // If this is an adaptor, then recursively merge the pass managers.
    if (auto *adaptorPass = dyn_cast<OpToOpPassAdaptor>(&pass)) {
      auto *otherAdaptorPass = cast<OpToOpPassAdaptor>(&otherPass);
      for (auto mgrs : llvm::zip(adaptorPass->getPassManagers(),
                                 otherAdaptorPass->getPassManagers()))
        std::get<0>(mgrs).mergeStatisticsInto(std::get<1>(mgrs));
      continue;
    }
    // Otherwise, merge the statistics for the current pass.
    assert(pass.statistics.size() == otherPass.statistics.size());
    for (unsigned i = 0, e = pass.statistics.size(); i != e; ++i) {
```
- **EN**: Implements logic around `zip`, `get`, `dyn_cast`, `cast`, and 3 more symbols.
- **CN**: 围绕 `zip`、`get`、`dyn_cast`、`cast` 等另外 3 个符号 实现具体逻辑。

### Lines 211-218
```cpp
      assert(pass.statistics[i]->getName() ==
             StringRef(otherPass.statistics[i]->getName()));
      *otherPass.statistics[i] += *pass.statistics[i];
      *pass.statistics[i] = 0;
    }
  }
}

```
- **EN**: Implements logic around `assert`, `StringRef`.
- **CN**: 围绕 `assert`、`StringRef` 实现具体逻辑。

### Lines 219-227
```cpp
/// Prepare the statistics of passes within the given pass manager for
/// consumption(e.g. dumping).
static void prepareStatistics(OpPassManager &pm) {
  for (Pass &pass : pm.getPasses()) {
    OpToOpPassAdaptor *adaptor = dyn_cast<OpToOpPassAdaptor>(&pass);
    if (!adaptor)
      continue;
    MutableArrayRef<OpPassManager> nestedPms = adaptor->getPassManagers();

```
- **EN**: Implements logic around `prepareStatistics`, `getPasses`, `dyn_cast`, `getPassManagers`.
- **CN**: 围绕 `prepareStatistics`、`getPasses`、`dyn_cast`、`getPassManagers` 实现具体逻辑。

### Lines 228-236
```cpp
    // Merge the statistics from the async pass managers into the main nested
    // pass managers.  Prepare recursively before merging.
    for (auto &asyncPM : adaptor->getParallelPassManagers()) {
      for (unsigned i = 0, e = asyncPM.size(); i != e; ++i) {
        prepareStatistics(asyncPM[i]);
        asyncPM[i].mergeStatisticsInto(nestedPms[i]);
      }
    }

```
- **EN**: Implements logic around `getParallelPassManagers`, `size`, `prepareStatistics`, `mergeStatisticsInto`.
- **CN**: 围绕 `getParallelPassManagers`、`size`、`prepareStatistics`、`mergeStatisticsInto` 实现具体逻辑。

### Lines 237-243
```cpp
    // Prepare the statistics of each of the nested passes.
    for (OpPassManager &nestedPM : nestedPms)
      prepareStatistics(nestedPM);
  }
}

/// Dump the statistics of the passes within this pass manager.
```
- **EN**: Implements logic around `prepareStatistics`.
- **CN**: 围绕 `prepareStatistics` 实现具体逻辑。

### Lines 244-252
```cpp
void PassManager::dumpStatistics() {
  prepareStatistics(*this);
  printStatistics(*this, *passStatisticsMode);
}

/// Dump the statistics for each pass after running.
void PassManager::enableStatistics(PassDisplayMode displayMode) {
  passStatisticsMode = displayMode;
}
```
- **EN**: Implements logic around `dumpStatistics`, `prepareStatistics`, `printStatistics`, `enableStatistics`; this block coordinates pass execution or analysis state; parses or prints textual MLIR representations.
- **CN**: 围绕 `dumpStatistics`、`prepareStatistics`、`printStatistics`、`enableStatistics` 实现具体逻辑；该代码块协调 Pass 执行或分析状态，并解析或打印文本形式的 MLIR 表示。

## Key Concepts / 关键概念

- **Pass infrastructure / Pass 基础设施**:
  - **EN**: Coordinates pass pipelines, analyses, instrumentation, and nested operation traversal.
  - **CN**: 协调 Pass 流水线、分析、插桩以及嵌套操作遍历。
- **SSA and region-based IR / SSA 与基于 Region 的 IR**:
  - **EN**: The file works with MLIR operations, blocks, values, or regions that form MLIR's hierarchical SSA IR.
  - **CN**: 该文件处理 MLIR 操作、块、值或 Region，它们共同构成 MLIR 的层次化 SSA IR。
- **Pass pipelines / Pass 流水线**:
  - **EN**: The file participates in pass registration, scheduling, or execution orchestration.
  - **CN**: 该文件参与 Pass 的注册、调度或执行编排。
- **Streaming output / 流式输出**:
  - **EN**: Text, diagnostics, or generated content are emitted through LLVM stream APIs.
  - **CN**: 通过 LLVM 流 API 输出文本、诊断或生成内容。
- **LLVM ADT containers / LLVM ADT 容器**:
  - **EN**: The implementation uses LLVM containers for performance-conscious storage and traversal.
  - **CN**: 该实现使用 LLVM 容器来进行兼顾性能的存储与遍历。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `PassDetail.h`, `mlir/Pass/PassManager.h`, `llvm/ADT/StringExtras.h`, `llvm/Support/Format.h`
- **Subsystem categories / 子系统类别**: pass-manager infrastructure / Pass 管理器基础设施 (1), LLVM ADT containers and utilities / LLVM ADT 容器与工具 (1), LLVM support-library helpers / LLVM Support 库辅助工具 (1)
