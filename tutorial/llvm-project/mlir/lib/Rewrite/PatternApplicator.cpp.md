# PatternApplicator.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Rewrite/PatternApplicator.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements an applicator that applies pattern rewrites based upon a user defined cost model.
  - **CN**: 实现模式重写、规范化支持以及重写驱动。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- PatternApplicator.cpp - Pattern Application Engine -------*- C++ -*-===//
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
//
// This file implements an applicator that applies pattern rewrites based upon a
// user defined cost model.
//
//===----------------------------------------------------------------------===//

#include "mlir/Rewrite/PatternApplicator.h"
#include "ByteCode.h"
#include "llvm/Support/DebugLog.h"

```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/Rewrite/PatternApplicator.h`, `ByteCode.h`, `llvm/Support/DebugLog.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/Rewrite/PatternApplicator.h`, `ByteCode.h`, `llvm/Support/DebugLog.h`。

### Lines 18-26
```cpp
#ifndef NDEBUG
#include "llvm/ADT/ScopeExit.h"
#endif

#define DEBUG_TYPE "pattern-application"

using namespace mlir;
using namespace mlir::detail;

```
- **EN**: Pulls in the declarations needed by this translation unit, including `llvm/ADT/ScopeExit.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `llvm/ADT/ScopeExit.h`。

### Lines 27-36
```cpp
PatternApplicator::PatternApplicator(
    const FrozenRewritePatternSet &frozenPatternList)
    : frozenPatternList(frozenPatternList) {
  if (const PDLByteCode *bytecode = frozenPatternList.getPDLByteCode()) {
    mutableByteCodeState = std::make_unique<PDLByteCodeMutableState>();
    bytecode->initializeMutableState(*mutableByteCodeState);
  }
}
PatternApplicator::~PatternApplicator() = default;

```
- **EN**: Implements logic around `PatternApplicator`, `frozenPatternList`, `getPDLByteCode`, `make_unique`, and 2 more symbols; this block relies on MLIR pattern rewriting infrastructure.
- **CN**: 围绕 `PatternApplicator`、`frozenPatternList`、`getPDLByteCode`、`make_unique` 等另外 2 个符号 实现具体逻辑；该代码块依赖 MLIR 模式重写基础设施。

### Lines 37-44
```cpp
#ifndef NDEBUG
/// Log a message for a pattern that is impossible to match.
static void logImpossibleToMatch(const Pattern &pattern) {
  LDBG() << "Ignoring pattern '" << pattern.getRootKind()
         << "' because it is impossible to match or cannot lead "
            "to legal IR (by cost model)";
}

```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 45-57
```cpp
/// Log IR after pattern application.
static Operation *getDumpRootOp(Operation *op) {
  Operation *isolatedParent =
      op->getParentWithTrait<mlir::OpTrait::IsIsolatedFromAbove>();
  if (isolatedParent)
    return isolatedParent;
  return op;
}
static void logSucessfulPatternApplication(Operation *op) {
  LDBG(2) << "// *** IR Dump After Pattern Application ***\n" << *op << "\n";
}
#endif

```
- **EN**: Implements logic around `getDumpRootOp`, `IsIsolatedFromAbove>`, `logSucessfulPatternApplication`, `LDBG`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getDumpRootOp`、`IsIsolatedFromAbove>`、`logSucessfulPatternApplication`、`LDBG` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 58-65
```cpp
void PatternApplicator::applyCostModel(CostModel model) {
  // Apply the cost model to the bytecode patterns first, and then the native
  // patterns.
  if (const PDLByteCode *bytecode = frozenPatternList.getPDLByteCode()) {
    for (const auto &it : llvm::enumerate(bytecode->getPatterns()))
      mutableByteCodeState->updatePatternBenefit(it.index(), model(it.value()));
  }

```
- **EN**: Implements logic around `applyCostModel`, `getPDLByteCode`, `enumerate`, `updatePatternBenefit`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `applyCostModel`、`getPDLByteCode`、`enumerate`、`updatePatternBenefit` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 66-79
```cpp
  // Copy over the patterns so that we can sort by benefit based on the cost
  // model. Patterns that are already impossible to match are ignored.
  patterns.clear();
  for (const auto &it : frozenPatternList.getOpSpecificNativePatterns()) {
    for (const RewritePattern *pattern : it.second) {
      if (pattern->getBenefit().isImpossibleToMatch())
        LLVM_DEBUG(logImpossibleToMatch(*pattern));
      else
        patterns[it.first].push_back(pattern);
    }
  }
  anyOpPatterns.clear();
  for (const RewritePattern &pattern :
       frozenPatternList.getMatchAnyOpNativePatterns()) {
```
- **EN**: Implements logic around `clear`, `getOpSpecificNativePatterns`, `getBenefit`, `logImpossibleToMatch`, and 2 more symbols; this block relies on MLIR pattern rewriting infrastructure.
- **CN**: 围绕 `clear`、`getOpSpecificNativePatterns`、`getBenefit`、`logImpossibleToMatch` 等另外 2 个符号 实现具体逻辑；该代码块依赖 MLIR 模式重写基础设施。

### Lines 80-93
```cpp
    if (pattern.getBenefit().isImpossibleToMatch())
      LLVM_DEBUG(logImpossibleToMatch(pattern));
    else
      anyOpPatterns.push_back(&pattern);
  }

  // Sort the patterns using the provided cost model.
  llvm::SmallDenseMap<const Pattern *, PatternBenefit> benefits;
  auto cmp = [&benefits](const Pattern *lhs, const Pattern *rhs) {
    return benefits[lhs] > benefits[rhs];
  };
  auto processPatternList = [&](SmallVectorImpl<const RewritePattern *> &list) {
    // Special case for one pattern in the list, which is the most common case.
    if (list.size() == 1) {
```
- **EN**: Implements logic around `getBenefit`, `logImpossibleToMatch`, `push_back`, `size`; this block relies on MLIR pattern rewriting infrastructure.
- **CN**: 围绕 `getBenefit`、`logImpossibleToMatch`、`push_back`、`size` 实现具体逻辑；该代码块依赖 MLIR 模式重写基础设施。

### Lines 94-100
```cpp
      if (model(*list.front()).isImpossibleToMatch()) {
        LLVM_DEBUG(logImpossibleToMatch(*list.front()));
        list.clear();
      }
      return;
    }

```
- **EN**: Implements logic around `model`, `logImpossibleToMatch`, `clear`.
- **CN**: 围绕 `model`、`logImpossibleToMatch`、`clear` 实现具体逻辑。

### Lines 101-114
```cpp
    // Collect the dynamic benefits for the current pattern list.
    benefits.clear();
    for (const Pattern *pat : list)
      benefits.try_emplace(pat, model(*pat));

    // Sort patterns with highest benefit first, and remove those that are
    // impossible to match.
    llvm::stable_sort(list, cmp);
    while (!list.empty() && benefits[list.back()].isImpossibleToMatch()) {
      LLVM_DEBUG(logImpossibleToMatch(*list.back()));
      list.pop_back();
    }
  };
  for (auto &it : patterns)
```
- **EN**: Implements logic around `clear`, `try_emplace`, `stable_sort`, `empty`, and 2 more symbols.
- **CN**: 围绕 `clear`、`try_emplace`、`stable_sort`、`empty` 等另外 2 个符号 实现具体逻辑。

### Lines 115-128
```cpp
    processPatternList(it.second);
  processPatternList(anyOpPatterns);
}

void PatternApplicator::walkAllPatterns(
    function_ref<void(const Pattern &)> walk) {
  for (const auto &it : frozenPatternList.getOpSpecificNativePatterns())
    for (const auto &pattern : it.second)
      walk(*pattern);
  for (const Pattern &it : frozenPatternList.getMatchAnyOpNativePatterns())
    walk(it);
  if (const PDLByteCode *bytecode = frozenPatternList.getPDLByteCode()) {
    for (const Pattern &it : bytecode->getPatterns())
      walk(it);
```
- **EN**: Implements logic around `processPatternList`, `walkAllPatterns`, `function_ref`, `getOpSpecificNativePatterns`, and 4 more symbols.
- **CN**: 围绕 `processPatternList`、`walkAllPatterns`、`function_ref`、`getOpSpecificNativePatterns` 等另外 4 个符号 实现具体逻辑。

### Lines 129-142
```cpp
  }
}

LogicalResult PatternApplicator::matchAndRewrite(
    Operation *op, PatternRewriter &rewriter,
    function_ref<bool(const Pattern &)> canApply,
    function_ref<void(const Pattern &)> onFailure,
    function_ref<LogicalResult(const Pattern &)> onSuccess) {
  // Before checking native patterns, first match against the bytecode. This
  // won't automatically perform any rewrites so there is no need to worry about
  // conflicts.
  SmallVector<PDLByteCode::MatchResult, 4> pdlMatches;
  const PDLByteCode *bytecode = frozenPatternList.getPDLByteCode();
  if (bytecode)
```
- **EN**: Implements logic around `matchAndRewrite`, `function_ref`, `getPDLByteCode`; this block makes success/failure or diagnostics explicit through MLIR result utilities; relies on MLIR pattern rewriting infrastructure; manipulates core MLIR IR objects.
- **CN**: 围绕 `matchAndRewrite`、`function_ref`、`getPDLByteCode` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并依赖 MLIR 模式重写基础设施，并操作 MLIR 核心 IR 对象。

### Lines 143-150
```cpp
    bytecode->match(op, rewriter, pdlMatches, *mutableByteCodeState);

  // Check to see if there are patterns matching this specific operation type.
  MutableArrayRef<const RewritePattern *> opPatterns;
  auto patternIt = patterns.find(op->getName());
  if (patternIt != patterns.end())
    opPatterns = patternIt->second;

```
- **EN**: Implements logic around `match`, `find`, `end`; this block relies on MLIR pattern rewriting infrastructure; manipulates core MLIR IR objects.
- **CN**: 围绕 `match`、`find`、`end` 实现具体逻辑；该代码块依赖 MLIR 模式重写基础设施，并操作 MLIR 核心 IR 对象。

### Lines 151-161
```cpp
  // Process the patterns for that match the specific operation type, and any
  // operation type in an interleaved fashion.
  unsigned opIt = 0, opE = opPatterns.size();
  unsigned anyIt = 0, anyE = anyOpPatterns.size();
  unsigned pdlIt = 0, pdlE = pdlMatches.size();
  LogicalResult result = failure();
  do {
    // Find the next pattern with the highest benefit.
    const Pattern *bestPattern = nullptr;
    unsigned *bestPatternIt = &opIt;

```
- **EN**: Implements logic around `size`, `failure`; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects.
- **CN**: 围绕 `size`、`failure` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象。

### Lines 162-172
```cpp
    /// Operation specific patterns.
    if (opIt < opE)
      bestPattern = opPatterns[opIt];
    /// Operation agnostic patterns.
    if (anyIt < anyE &&
        (!bestPattern ||
         bestPattern->getBenefit() < anyOpPatterns[anyIt]->getBenefit())) {
      bestPatternIt = &anyIt;
      bestPattern = anyOpPatterns[anyIt];
    }

```
- **EN**: Implements logic around `getBenefit`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getBenefit` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 173-181
```cpp
    const PDLByteCode::MatchResult *pdlMatch = nullptr;
    /// PDL patterns.
    if (pdlIt < pdlE && (!bestPattern || bestPattern->getBenefit() <
                                             pdlMatches[pdlIt].benefit)) {
      bestPatternIt = &pdlIt;
      pdlMatch = &pdlMatches[pdlIt];
      bestPattern = pdlMatch->pattern;
    }

```
- **EN**: Implements logic around `getBenefit`.
- **CN**: 围绕 `getBenefit` 实现具体逻辑。

### Lines 182-188
```cpp
    if (!bestPattern)
      break;

    // Update the pattern iterator on failure so that this pattern isn't
    // attempted again.
    ++(*bestPatternIt);

```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 189-202
```cpp
    // Check that the pattern can be applied.
    if (canApply && !canApply(*bestPattern))
      continue;

    // Try to match and rewrite this pattern. The patterns are sorted by
    // benefit, so if we match we can immediately rewrite. For PDL patterns, the
    // match has already been performed, we just need to rewrite.
    bool matched = false;
    op->getContext()->executeAction<ApplyPatternAction>(
        [&]() {
          rewriter.setInsertionPoint(op);
#ifndef NDEBUG
          // Operation `op` may be invalidated after applying the rewrite
          // pattern.
```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 203-213
```cpp
          Operation *dumpRootOp = getDumpRootOp(op);
#endif
          if (pdlMatch) {
            result =
                bytecode->rewrite(rewriter, *pdlMatch, *mutableByteCodeState);
          } else {
            LDBG() << "Trying to match \"" << bestPattern->getDebugName()
                   << "\"";
            const auto *pattern =
                static_cast<const RewritePattern *>(bestPattern);

```
- **EN**: Implements logic around `getDumpRootOp`, `rewrite`, `LDBG`; this block relies on MLIR pattern rewriting infrastructure; manipulates core MLIR IR objects.
- **CN**: 围绕 `getDumpRootOp`、`rewrite`、`LDBG` 实现具体逻辑；该代码块依赖 MLIR 模式重写基础设施，并操作 MLIR 核心 IR 对象。

### Lines 214-227
```cpp
#ifndef NDEBUG
            OpBuilder::Listener *oldListener = rewriter.getListener();
            auto loggingListener =
                std::make_unique<RewriterBase::PatternLoggingListener>(
                    oldListener, pattern->getDebugName());
            rewriter.setListener(loggingListener.get());
            llvm::scope_exit resetListenerCallback(
                [&] { rewriter.setListener(oldListener); });
#endif
            result = pattern->matchAndRewrite(op, rewriter);
            LDBG() << " -> matchAndRewrite "
                   << (succeeded(result) ? "successful" : "failed");
          }

```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 228-236
```cpp
          // Process the result of the pattern application.
          if (succeeded(result) && onSuccess && failed(onSuccess(*bestPattern)))
            result = failure();
          if (succeeded(result)) {
            LLVM_DEBUG(logSucessfulPatternApplication(dumpRootOp));
            matched = true;
            return;
          }

```
- **EN**: Implements logic around `succeeded`, `failure`, `logSucessfulPatternApplication`.
- **CN**: 围绕 `succeeded`、`failure`、`logSucessfulPatternApplication` 实现具体逻辑。

### Lines 237-245
```cpp
          // Perform any necessary cleanups.
          if (onFailure)
            onFailure(*bestPattern);
        },
        {op}, *bestPattern);
    if (matched)
      break;
  } while (true);

```
- **EN**: Implements logic around `onFailure`.
- **CN**: 围绕 `onFailure` 实现具体逻辑。

### Lines 246-249
```cpp
  if (mutableByteCodeState)
    mutableByteCodeState->cleanupAfterMatchAndRewrite();
  return result;
}
```
- **EN**: Implements logic around `cleanupAfterMatchAndRewrite`.
- **CN**: 围绕 `cleanupAfterMatchAndRewrite` 实现具体逻辑。

## Key Concepts / 关键概念

- **Pattern-driven rewriting / 基于模式的重写**:
  - **EN**: Uses declarative or imperative rewrite patterns to transform IR.
  - **CN**: 使用声明式或命令式重写模式对 IR 进行变换。
- **SSA and region-based IR / SSA 与基于 Region 的 IR**:
  - **EN**: The file works with MLIR operations, blocks, values, or regions that form MLIR's hierarchical SSA IR.
  - **CN**: 该文件处理 MLIR 操作、块、值或 Region，它们共同构成 MLIR 的层次化 SSA IR。
- **Structured diagnostics / 结构化诊断**:
  - **EN**: Failures are surfaced through `LogicalResult`, diagnostics, or related reporting helpers.
  - **CN**: 通过 `LogicalResult`、诊断对象或相关报告工具显式传播失败。
- **Pattern rewriting / 模式重写**:
  - **EN**: The implementation uses rewrite patterns to match and transform IR.
  - **CN**: 该实现使用重写模式来匹配并变换 IR。
- **Bytecode support / Bytecode 支持**:
  - **EN**: The file reads, writes, or describes compact MLIR bytecode forms.
  - **CN**: 该文件读取、写出或描述紧凑的 MLIR bytecode 形式。
- **LLVM ADT containers / LLVM ADT 容器**:
  - **EN**: The implementation uses LLVM containers for performance-conscious storage and traversal.
  - **CN**: 该实现使用 LLVM 容器来进行兼顾性能的存储与遍历。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `mlir/Rewrite/PatternApplicator.h`, `ByteCode.h`, `llvm/Support/DebugLog.h`, `llvm/ADT/ScopeExit.h`
- **Subsystem categories / 子系统类别**: pattern rewrite support / 模式重写支持 (1), LLVM support-library helpers / LLVM Support 库辅助工具 (1), LLVM ADT containers and utilities / LLVM ADT 容器与工具 (1)
