# GreedyPatternRewriteDriver.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Transforms/Utils/GreedyPatternRewriteDriver.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements mlir::applyPatternsGreedily.
  - **CN**: 实现核心 MLIR 变换 Pass 与变换辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
//===- GreedyPatternRewriteDriver.cpp - A greedy rewriter -----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements mlir::applyPatternsGreedily.
//
//===----------------------------------------------------------------------===//

#include "mlir/Transforms/GreedyPatternRewriteDriver.h"

```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/Transforms/GreedyPatternRewriteDriver.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/Transforms/GreedyPatternRewriteDriver.h`。

### Lines 15-33
```cpp
#include "mlir/Config/mlir-config.h"
#include "mlir/IR/Action.h"
#include "mlir/IR/Dominance.h"
#include "mlir/IR/Matchers.h"
#include "mlir/IR/Operation.h"
#include "mlir/IR/OperationSupport.h"
#include "mlir/IR/Verifier.h"
#include "mlir/Interfaces/SideEffectInterfaces.h"
#include "mlir/Rewrite/PatternApplicator.h"
#include "mlir/Transforms/CSE.h"
#include "mlir/Transforms/FoldUtils.h"
#include "mlir/Transforms/RegionUtils.h"
#include "llvm/ADT/BitVector.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/ADT/ScopeExit.h"
#include "llvm/Support/DebugLog.h"
#include "llvm/Support/ScopedPrinter.h"
#include "llvm/Support/raw_ostream.h"

```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/Config/mlir-config.h`, `mlir/IR/Action.h`, `mlir/IR/Dominance.h`, `mlir/IR/Matchers.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/Config/mlir-config.h`, `mlir/IR/Action.h`, `mlir/IR/Dominance.h`, `mlir/IR/Matchers.h`。

### Lines 34-47
```cpp
#ifdef MLIR_GREEDY_REWRITE_RANDOMIZER_SEED
#include <random>
#endif // MLIR_GREEDY_REWRITE_RANDOMIZER_SEED

using namespace mlir;

#define DEBUG_TYPE "greedy-rewriter"

namespace {

//===----------------------------------------------------------------------===//
// Debugging Infrastructure
//===----------------------------------------------------------------------===//

```
- **EN**: Pulls in the declarations needed by this translation unit, including `random`.
- **CN**: 引入该编译单元所需的声明，其中包括 `random`。

### Lines 48-61
```cpp
#if MLIR_ENABLE_EXPENSIVE_PATTERN_API_CHECKS
/// A helper struct that performs various "expensive checks" to detect broken
/// rewrite patterns use the rewriter API incorrectly. A rewrite pattern is
/// broken if:
/// * IR does not verify after pattern application / folding.
/// * Pattern returns "failure" but the IR has changed.
/// * Pattern returns "success" but the IR has not changed.
///
/// This struct stores finger prints of ops to determine whether the IR has
/// changed or not.
struct ExpensiveChecks : public RewriterBase::ForwardingListener {
  ExpensiveChecks(RewriterBase::Listener *driver, Operation *topLevel)
      : RewriterBase::ForwardingListener(driver), topLevel(topLevel) {}

```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 62-77
```cpp
  /// Compute finger prints of the given op and its nested ops.
  void computeFingerPrints(Operation *topLevel) {
    this->topLevel = topLevel;
    this->topLevelFingerPrint.emplace(topLevel);
    topLevel->walk([&](Operation *op) {
      fingerprints.try_emplace(op, op, /*includeNested=*/false);
    });
  }

  /// Clear all finger prints.
  void clear() {
    topLevel = nullptr;
    topLevelFingerPrint.reset();
    fingerprints.clear();
  }

```
- **EN**: Implements logic around `computeFingerPrints`, `emplace`, `walk`, `try_emplace`, and 2 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations; implements transformation or simplification logic.
- **CN**: 围绕 `computeFingerPrints`、`emplace`、`walk`、`try_emplace` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示，并实现变换或简化逻辑。

### Lines 78-105
```cpp
  void notifyRewriteSuccess() {
    if (!topLevel)
      return;

    // Make sure that the IR still verifies.
    if (failed(verify(topLevel)))
      llvm::report_fatal_error("IR failed to verify after pattern application");

    // Pattern application success => IR must have changed.
    OperationFingerPrint afterFingerPrint(topLevel);
    if (*topLevelFingerPrint == afterFingerPrint) {
      // Note: Run "mlir-opt -debug" to see which pattern is broken.
      llvm::report_fatal_error(
          "pattern returned success but IR did not change");
    }
    for (const auto &it : fingerprints) {
      // Skip top-level op, its finger print is never invalidated.
      if (it.first == topLevel)
        continue;
      // Note: Finger print computation may crash when an op was erased
      // without notifying the rewriter. (Run with ASAN to see where the op was
      // erased; the op was probably erased directly, bypassing the rewriter
      // API.) Finger print computation does may not crash if a new op was
      // created at the same memory location. (But then the finger print should
      // have changed.)
      if (it.second !=
          OperationFingerPrint(it.first, /*includeNested=*/false)) {
        // Note: Run "mlir-opt -debug" to see which pattern is broken.
```
- **EN**: Implements logic around `notifyRewriteSuccess`, `failed`, `report_fatal_error`, `afterFingerPrint`, and 1 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations; implements transformation or simplification logic.
- **CN**: 围绕 `notifyRewriteSuccess`、`failed`、`report_fatal_error`、`afterFingerPrint` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示，并实现变换或简化逻辑。

### Lines 106-122
```cpp
        llvm::report_fatal_error("operation finger print changed");
      }
    }
  }

  void notifyRewriteFailure() {
    if (!topLevel)
      return;

    // Pattern application failure => IR must not have changed.
    OperationFingerPrint afterFingerPrint(topLevel);
    if (*topLevelFingerPrint != afterFingerPrint) {
      // Note: Run "mlir-opt -debug" to see which pattern is broken.
      llvm::report_fatal_error("pattern returned failure but IR did change");
    }
  }

```
- **EN**: Implements logic around `report_fatal_error`, `notifyRewriteFailure`, `afterFingerPrint`; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations; implements transformation or simplification logic.
- **CN**: 围绕 `report_fatal_error`、`notifyRewriteFailure`、`afterFingerPrint` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示，并实现变换或简化逻辑。

### Lines 123-138
```cpp
  void notifyFoldingSuccess() {
    if (!topLevel)
      return;

    // Make sure that the IR still verifies.
    if (failed(verify(topLevel)))
      llvm::report_fatal_error("IR failed to verify after folding");
  }

protected:
  /// Invalidate the finger print of the given op, i.e., remove it from the map.
  void invalidateFingerPrint(Operation *op) { fingerprints.erase(op); }

  void notifyBlockErased(Block *block) override {
    RewriterBase::ForwardingListener::notifyBlockErased(block);

```
- **EN**: Implements logic around `notifyFoldingSuccess`, `failed`, `report_fatal_error`, `invalidateFingerPrint`, and 1 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations; implements transformation or simplification logic.
- **CN**: 围绕 `notifyFoldingSuccess`、`failed`、`report_fatal_error`、`invalidateFingerPrint` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示，并实现变换或简化逻辑。

### Lines 139-152
```cpp
    // The block structure (number of blocks, types of block arguments, etc.)
    // is part of the fingerprint of the parent op.
    // TODO: The parent op fingerprint should also be invalidated when modifying
    // the block arguments of a block, but we do not have a
    // `notifyBlockModified` callback yet.
    invalidateFingerPrint(block->getParentOp());
  }

  void notifyOperationInserted(Operation *op,
                               OpBuilder::InsertPoint previous) override {
    RewriterBase::ForwardingListener::notifyOperationInserted(op, previous);
    invalidateFingerPrint(op->getParentOp());
  }

```
- **EN**: Implements logic around `invalidateFingerPrint`, `notifyOperationInserted`; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `invalidateFingerPrint`、`notifyOperationInserted` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 153-167
```cpp
  void notifyOperationModified(Operation *op) override {
    RewriterBase::ForwardingListener::notifyOperationModified(op);
    invalidateFingerPrint(op);
  }

  void notifyOperationErased(Operation *op) override {
    RewriterBase::ForwardingListener::notifyOperationErased(op);
    op->walk([this](Operation *op) { invalidateFingerPrint(op); });
  }

  /// Operation finger prints to detect invalid pattern API usage. IR is checked
  /// against these finger prints after pattern application to detect cases
  /// where IR was modified directly, bypassing the rewriter API.
  DenseMap<Operation *, OperationFingerPrint> fingerprints;

```
- **EN**: Implements logic around `notifyOperationModified`, `invalidateFingerPrint`, `notifyOperationErased`, `walk`; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations; implements transformation or simplification logic.
- **CN**: 围绕 `notifyOperationModified`、`invalidateFingerPrint`、`notifyOperationErased`、`walk` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示，并实现变换或简化逻辑。

### Lines 168-189
```cpp
  /// Top-level operation of the current greedy rewrite.
  Operation *topLevel = nullptr;

  /// Finger print of the top-level operation.
  std::optional<OperationFingerPrint> topLevelFingerPrint;
};
#endif // MLIR_ENABLE_EXPENSIVE_PATTERN_API_CHECKS

#ifndef NDEBUG
static Operation *getDumpRootOp(Operation *op) {
  // Dump the parent op so that materialized constants are visible. If the op
  // is a top-level op, dump it directly.
  if (Operation *parentOp = op->getParentOp())
    return parentOp;
  return op;
}
static void logSuccessfulFolding(Operation *op) {
  LDBG() << "// *** IR Dump After Successful Folding ***\n"
         << OpWithFlags(op, OpPrintingFlags().elideLargeElementsAttrs());
}
#endif // NDEBUG

```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 190-203
```cpp
//===----------------------------------------------------------------------===//
// Worklist
//===----------------------------------------------------------------------===//

/// A LIFO worklist of operations with efficient removal and set semantics.
///
/// This class maintains a vector of operations and a mapping of operations to
/// positions in the vector, so that operations can be removed efficiently at
/// random. When an operation is removed, it is replaced with nullptr. Such
/// nullptr are skipped when pop'ing elements.
class Worklist {
public:
  Worklist();

```
- **EN**: Introduces declarations for `Worklist`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `Worklist` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 204-217
```cpp
  /// Clear the worklist.
  void clear();

  /// Return whether the worklist is empty.
  bool empty() const;

  /// Push an operation to the end of the worklist, unless the operation is
  /// already on the worklist.
  void push(Operation *op);

  /// Pop the an operation from the end of the worklist. Only allowed on
  /// non-empty worklists.
  Operation *pop();

```
- **EN**: Implements logic around `clear`, `empty`, `push`, `pop`; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `clear`、`empty`、`push`、`pop` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 218-231
```cpp
  /// Remove an operation from the worklist.
  void remove(Operation *op);

  /// Reverse the worklist.
  void reverse();

protected:
  /// The worklist of operations.
  std::vector<Operation *> list;

  /// A mapping of operations to positions in `list`.
  DenseMap<Operation *, unsigned> map;
};

```
- **EN**: Implements logic around `remove`, `reverse`; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `remove`、`reverse` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 232-252
```cpp
Worklist::Worklist() { list.reserve(64); }

void Worklist::clear() {
  list.clear();
  map.clear();
}

bool Worklist::empty() const {
  // Skip all nullptr.
  return !llvm::any_of(list,
                       [](Operation *op) { return static_cast<bool>(op); });
}

void Worklist::push(Operation *op) {
  assert(op && "cannot push nullptr to worklist");
  // Check to see if the worklist already contains this op.
  if (!map.insert({op, list.size()}).second)
    return;
  list.push_back(op);
}

```
- **EN**: Implements logic around `Worklist`, `clear`, `empty`, `any_of`, and 5 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `Worklist`、`clear`、`empty`、`any_of` 等另外 5 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 253-266
```cpp
Operation *Worklist::pop() {
  assert(!empty() && "cannot pop from empty worklist");
  // Skip and remove all trailing nullptr.
  while (!list.back())
    list.pop_back();
  Operation *op = list.back();
  list.pop_back();
  map.erase(op);
  // Cleanup: Remove all trailing nullptr.
  while (!list.empty() && !list.back())
    list.pop_back();
  return op;
}

```
- **EN**: Implements logic around `pop`, `assert`, `back`, `pop_back`, and 2 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `pop`、`assert`、`back`、`pop_back` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 267-282
```cpp
void Worklist::remove(Operation *op) {
  assert(op && "cannot remove nullptr from worklist");
  auto it = map.find(op);
  if (it != map.end()) {
    assert(list[it->second] == op && "malformed worklist data structure");
    list[it->second] = nullptr;
    map.erase(it);
  }
}

void Worklist::reverse() {
  std::reverse(list.begin(), list.end());
  for (size_t i = 0, e = list.size(); i != e; ++i)
    map[list[i]] = i;
}

```
- **EN**: Implements logic around `remove`, `assert`, `find`, `end`, and 3 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `remove`、`assert`、`find`、`end` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 283-308
```cpp
#ifdef MLIR_GREEDY_REWRITE_RANDOMIZER_SEED
/// A worklist that pops elements at a random position. This worklist is for
/// testing/debugging purposes only. It can be used to ensure that lowering
/// pipelines work correctly regardless of the order in which ops are processed
/// by the GreedyPatternRewriteDriver.
class RandomizedWorklist : public Worklist {
public:
  RandomizedWorklist() : Worklist() {
    generator.seed(MLIR_GREEDY_REWRITE_RANDOMIZER_SEED);
  }

  /// Pop a random non-empty op from the worklist.
  Operation *pop() {
    Operation *op = nullptr;
    do {
      assert(!list.empty() && "cannot pop from empty worklist");
      int64_t pos = generator() % list.size();
      op = list[pos];
      list.erase(list.begin() + pos);
      for (int64_t i = pos, e = list.size(); i < e; ++i)
        map[list[i]] = i;
      map.erase(op);
    } while (!op);
    return op;
  }

```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 309-322
```cpp
private:
  std::minstd_rand0 generator;
};
#endif // MLIR_GREEDY_REWRITE_RANDOMIZER_SEED

//===----------------------------------------------------------------------===//
// GreedyPatternRewriteDriver
//===----------------------------------------------------------------------===//

/// This is a worklist-driven driver for the PatternMatcher, which repeatedly
/// applies the locally optimal patterns.
///
/// This abstract class manages the worklist and contains helper methods for
/// rewriting ops on the worklist. Derived classes specify how ops are added
```
- **EN**: Transitions between access-control regions inside a class or struct definition.
- **CN**: 在类或结构体定义内部切换访问控制区域。

### Lines 323-336
```cpp
/// to the worklist in the beginning.
class GreedyPatternRewriteDriver : public RewriterBase::Listener {
protected:
  explicit GreedyPatternRewriteDriver(MLIRContext *ctx,
                                      const FrozenRewritePatternSet &patterns,
                                      const GreedyRewriteConfig &config);

  /// Add the given operation to the worklist.
  void addSingleOpToWorklist(Operation *op);

  /// Add the given operation and its ancestors to the worklist.
  void addToWorklist(Operation *op);

  /// Notify the driver that the specified operation may have been modified
```
- **EN**: Introduces declarations for `GreedyPatternRewriteDriver`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `GreedyPatternRewriteDriver` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 337-350
```cpp
  /// in-place. The operation is added to the worklist.
  void notifyOperationModified(Operation *op) override;

  /// Notify the driver that the specified operation was inserted. Update the
  /// worklist as needed: The operation is enqueued depending on scope and
  /// strict mode.
  void notifyOperationInserted(Operation *op,
                               OpBuilder::InsertPoint previous) override;

  /// Notify the driver that the specified operation was removed. Update the
  /// worklist as needed: The operation and its children are removed from the
  /// worklist.
  void notifyOperationErased(Operation *op) override;

```
- **EN**: Implements logic around `notifyOperationModified`, `notifyOperationInserted`, `notifyOperationErased`; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `notifyOperationModified`、`notifyOperationInserted`、`notifyOperationErased` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 351-364
```cpp
  /// Notify the driver that the specified operation was replaced. Update the
  /// worklist as needed: New users are added enqueued.
  void notifyOperationReplaced(Operation *op, ValueRange replacement) override;

  /// Process ops until the worklist is empty or `config.maxNumRewrites` is
  /// reached. Return `true` if any IR was changed.
  bool processWorklist();

  /// The pattern rewriter that is used for making IR modifications and is
  /// passed to rewrite patterns.
  PatternRewriter rewriter;

  /// The worklist for this transformation keeps track of the operations that
  /// need to be (re)visited.
```
- **EN**: Implements logic around `notifyOperationReplaced`, `processWorklist`; this block relies on MLIR pattern rewriting infrastructure; manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `notifyOperationReplaced`、`processWorklist` 实现具体逻辑；该代码块依赖 MLIR 模式重写基础设施，并操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 365-379
```cpp
#ifdef MLIR_GREEDY_REWRITE_RANDOMIZER_SEED
  RandomizedWorklist worklist;
#else
  Worklist worklist;
#endif // MLIR_GREEDY_REWRITE_RANDOMIZER_SEED

  /// Configuration information for how to simplify.
  const GreedyRewriteConfig config;

  /// The list of ops we are restricting our rewrites to. These include the
  /// supplied set of ops as well as new ops created while rewriting those ops
  /// depending on `strictMode`. This set is not maintained when
  /// `config.strictMode` is GreedyRewriteStrictness::AnyOp.
  llvm::SmallDenseSet<Operation *, 4> strictModeFilteredOps;

```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 380-393
```cpp
private:
  /// Look over the provided operands for any defining operations that should
  /// be re-added to the worklist. This function should be called when an
  /// operation is modified or removed, as it may trigger further
  /// simplifications.
  void addOperandsToWorklist(Operation *op);

  /// Notify the driver that the given block was inserted.
  void notifyBlockInserted(Block *block, Region *previous,
                           Region::iterator previousIt) override;

  /// Notify the driver that the given block is about to be removed.
  void notifyBlockErased(Block *block) override;

```
- **EN**: Implements logic around `addOperandsToWorklist`, `notifyBlockInserted`, `notifyBlockErased`; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `addOperandsToWorklist`、`notifyBlockInserted`、`notifyBlockErased` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 394-407
```cpp
  /// For debugging only: Notify the driver of a pattern match failure.
  void
  notifyMatchFailure(Location loc,
                     function_ref<void(Diagnostic &)> reasonCallback) override;

#ifndef NDEBUG
  /// A raw output stream used to prefix the debug log.

  llvm::impl::raw_ldbg_ostream os{(Twine("[") + DEBUG_TYPE + ":1] ").str(),
                                  llvm::dbgs()};
  /// A logger used to emit information during the application process.
  llvm::ScopedPrinter logger{os};
#endif

```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 408-432
```cpp
  /// The low-level pattern applicator.
  PatternApplicator matcher;

#if MLIR_ENABLE_EXPENSIVE_PATTERN_API_CHECKS
  ExpensiveChecks expensiveChecks;
#endif // MLIR_ENABLE_EXPENSIVE_PATTERN_API_CHECKS
};
} // namespace

GreedyPatternRewriteDriver::GreedyPatternRewriteDriver(
    MLIRContext *ctx, const FrozenRewritePatternSet &patterns,
    const GreedyRewriteConfig &config)
    : rewriter(ctx), config(config), matcher(patterns)
#if MLIR_ENABLE_EXPENSIVE_PATTERN_API_CHECKS
      // clang-format off
      , expensiveChecks(
          /*driver=*/this,
          /*topLevel=*/config.getScope() ? config.getScope()->getParentOp()
                                         : nullptr)
// clang-format on
#endif // MLIR_ENABLE_EXPENSIVE_PATTERN_API_CHECKS
{
  // Apply a simple cost model based solely on pattern benefit.
  matcher.applyDefaultCostModel();

```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 433-447
```cpp
  // Set up listener.
#if MLIR_ENABLE_EXPENSIVE_PATTERN_API_CHECKS
  // Send IR notifications to the debug handler. This handler will then forward
  // all notifications to this GreedyPatternRewriteDriver.
  rewriter.setListener(&expensiveChecks);
#else
  rewriter.setListener(this);
#endif // MLIR_ENABLE_EXPENSIVE_PATTERN_API_CHECKS
}

bool GreedyPatternRewriteDriver::processWorklist() {
#ifndef NDEBUG
  const char *logLineComment =
      "//===-------------------------------------------===//\n";

```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 448-461
```cpp
  /// A utility function to log a process result for the given reason.
  auto logResult = [&](StringRef result, const llvm::Twine &msg = {}) {
    logger.unindent();
    logger.startLine() << "} -> " << result;
    if (!msg.isTriviallyEmpty())
      logger.getOStream() << " : " << msg;
    logger.getOStream() << "\n";
  };
  auto logResultWithLine = [&](StringRef result, const llvm::Twine &msg = {}) {
    logResult(result, msg);
    logger.startLine() << logLineComment;
  };
#endif

```
- **EN**: Implements logic around `unindent`, `startLine`, `isTriviallyEmpty`, `getOStream`, and 1 more symbols; this block implements transformation or simplification logic.
- **CN**: 围绕 `unindent`、`startLine`、`isTriviallyEmpty`、`getOStream` 等另外 1 个符号 实现具体逻辑；该代码块实现变换或简化逻辑。

### Lines 462-475
```cpp
  bool changed = false;
  int64_t numRewrites = 0;
  while (!worklist.empty() &&
         (numRewrites < config.getMaxNumRewrites() ||
          config.getMaxNumRewrites() == GreedyRewriteConfig::kNoLimit)) {
    auto *op = worklist.pop();

    LLVM_DEBUG({
      logger.getOStream() << "\n";
      logger.startLine() << logLineComment;
      logger.startLine() << "Processing operation : '" << op->getName() << "'("
                         << op << ") {\n";
      logger.indent();

```
- **EN**: Implements logic around `empty`, `getMaxNumRewrites`, `pop`, `getOStream`, and 2 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `empty`、`getMaxNumRewrites`、`pop`、`getOStream` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 476-489
```cpp
      // If the operation has no regions, just print it here.
      if (op->getNumRegions() == 0) {
        op->print(
            logger.startLine(),
            OpPrintingFlags().printGenericOpForm().elideLargeElementsAttrs());
        logger.getOStream() << "\n\n";
      }
    });

    // If the operation is trivially dead - remove it.
    if (isOpTriviallyDead(op)) {
      rewriter.eraseOp(op);
      changed = true;

```
- **EN**: Implements logic around `getNumRegions`, `print`, `startLine`, `OpPrintingFlags`, and 3 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations; implements transformation or simplification logic.
- **CN**: 围绕 `getNumRegions`、`print`、`startLine`、`OpPrintingFlags` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示，并实现变换或简化逻辑。

### Lines 490-515
```cpp
      LLVM_DEBUG(logResultWithLine("success", "operation is trivially dead"));
      continue;
    }

    // Try to fold this op. Do not fold constant ops. That would lead to an
    // infinite folding loop, as every constant op would be folded to an
    // Attribute and then immediately be rematerialized as a constant op, which
    // is then put on the worklist.
    if (config.isFoldingEnabled() && !op->hasTrait<OpTrait::ConstantLike>()) {
      SmallVector<OpFoldResult> foldResults;
      if (succeeded(op->fold(foldResults))) {
        LLVM_DEBUG(logResultWithLine("success", "operation was folded"));
#ifndef NDEBUG
        Operation *dumpRootOp = getDumpRootOp(op);
#endif // NDEBUG
        if (foldResults.empty()) {
          // Op was modified in-place.
          notifyOperationModified(op);
          changed = true;
          LLVM_DEBUG(logSuccessfulFolding(dumpRootOp));
#if MLIR_ENABLE_EXPENSIVE_PATTERN_API_CHECKS
          expensiveChecks.notifyFoldingSuccess();
#endif // MLIR_ENABLE_EXPENSIVE_PATTERN_API_CHECKS
          continue;
        }

```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 516-534
```cpp
        // Op results can be replaced with `foldResults`.
        assert(foldResults.size() == op->getNumResults() &&
               "folder produced incorrect number of results");
        OpBuilder::InsertionGuard g(rewriter);
        rewriter.setInsertionPoint(op);
        SmallVector<Value> replacements;
        bool materializationSucceeded = true;
        for (auto [ofr, resultType] :
             llvm::zip_equal(foldResults, op->getResultTypes())) {
          if (auto value = dyn_cast<Value>(ofr)) {
            assert(value.getType() == resultType &&
                   "folder produced value of incorrect type");
            replacements.push_back(value);
            continue;
          }
          // Materialize Attributes as SSA values.
          Operation *constOp = op->getDialect()->materializeConstant(
              rewriter, cast<Attribute>(ofr), resultType, op->getLoc());

```
- **EN**: Implements logic around `assert`, `g`, `setInsertionPoint`, `zip_equal`, and 4 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `assert`、`g`、`setInsertionPoint`、`zip_equal` 等另外 4 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 535-548
```cpp
          if (!constOp) {
            // If materialization fails, cleanup any operations generated for
            // the previous results.
            llvm::SmallDenseSet<Operation *> replacementOps;
            for (Value replacement : replacements) {
              assert(replacement.use_empty() &&
                     "folder reused existing op for one result but constant "
                     "materialization failed for another result");
              replacementOps.insert(replacement.getDefiningOp());
            }
            for (Operation *op : replacementOps) {
              rewriter.eraseOp(op);
            }

```
- **EN**: Implements logic around `assert`, `insert`, `eraseOp`; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `assert`、`insert`、`eraseOp` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 549-571
```cpp
            materializationSucceeded = false;
            break;
          }

          assert(constOp->hasTrait<OpTrait::ConstantLike>() &&
                 "materializeConstant produced op that is not a ConstantLike");
          assert(constOp->getResultTypes()[0] == resultType &&
                 "materializeConstant produced incorrect result type");
          replacements.push_back(constOp->getResult(0));
        }

        if (materializationSucceeded) {
          rewriter.replaceOp(op, replacements);
          changed = true;
          LLVM_DEBUG(logSuccessfulFolding(dumpRootOp));
#if MLIR_ENABLE_EXPENSIVE_PATTERN_API_CHECKS
          expensiveChecks.notifyFoldingSuccess();
#endif // MLIR_ENABLE_EXPENSIVE_PATTERN_API_CHECKS
          continue;
        }
      }
    }

```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 572-599
```cpp
    // Try to match one of the patterns. The rewriter is automatically
    // notified of any necessary changes, so there is nothing else to do
    // here.
    auto canApplyCallback = [&](const Pattern &pattern) {
      LLVM_DEBUG({
        logger.getOStream() << "\n";
        logger.startLine() << "* Pattern " << pattern.getDebugName() << " : '"
                           << op->getName() << " -> (";
        llvm::interleaveComma(pattern.getGeneratedOps(), logger.getOStream());
        logger.getOStream() << ")' {\n";
        logger.indent();
      });
      if (RewriterBase::Listener *listener = config.getListener())
        listener->notifyPatternBegin(pattern, op);
      return true;
    };
    function_ref<bool(const Pattern &)> canApply = canApplyCallback;
    auto onFailureCallback = [&](const Pattern &pattern) {
      LLVM_DEBUG(logResult("failure", "pattern failed to match"));
      if (RewriterBase::Listener *listener = config.getListener())
        listener->notifyPatternEnd(pattern, failure());
    };
    function_ref<void(const Pattern &)> onFailure = onFailureCallback;
    auto onSuccessCallback = [&](const Pattern &pattern) {
      LLVM_DEBUG(logResult("success", "pattern applied successfully"));
      if (RewriterBase::Listener *listener = config.getListener())
        listener->notifyPatternEnd(pattern, success());
      return success();
```
- **EN**: Implements logic around `getOStream`, `startLine`, `getName`, `interleaveComma`, and 7 more symbols; this block implements transformation or simplification logic.
- **CN**: 围绕 `getOStream`、`startLine`、`getName`、`interleaveComma` 等另外 7 个符号 实现具体逻辑；该代码块实现变换或简化逻辑。

### Lines 600-619
```cpp
    };
    function_ref<LogicalResult(const Pattern &)> onSuccess = onSuccessCallback;

#ifdef NDEBUG
    // Optimization: PatternApplicator callbacks are not needed when running in
    // optimized mode and without a listener.
    if (!config.getListener()) {
      canApply = nullptr;
      onFailure = nullptr;
      onSuccess = nullptr;
    }
#endif // NDEBUG

#if MLIR_ENABLE_EXPENSIVE_PATTERN_API_CHECKS
    if (config.getScope()) {
      expensiveChecks.computeFingerPrints(config.getScope()->getParentOp());
    }
    llvm::scope_exit clearFingerprints([&]() { expensiveChecks.clear(); });
#endif // MLIR_ENABLE_EXPENSIVE_PATTERN_API_CHECKS

```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 620-637
```cpp
    LogicalResult matchResult =
        matcher.matchAndRewrite(op, rewriter, canApply, onFailure, onSuccess);

    if (succeeded(matchResult)) {
      LLVM_DEBUG(logResultWithLine("success", "at least one pattern matched"));
#if MLIR_ENABLE_EXPENSIVE_PATTERN_API_CHECKS
      expensiveChecks.notifyRewriteSuccess();
#endif // MLIR_ENABLE_EXPENSIVE_PATTERN_API_CHECKS
      changed = true;
      ++numRewrites;
    } else {
      LLVM_DEBUG(logResultWithLine("failure", "all patterns failed to match"));
#if MLIR_ENABLE_EXPENSIVE_PATTERN_API_CHECKS
      expensiveChecks.notifyRewriteFailure();
#endif // MLIR_ENABLE_EXPENSIVE_PATTERN_API_CHECKS
    }
  }

```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 638-659
```cpp
  return changed;
}

void GreedyPatternRewriteDriver::addToWorklist(Operation *op) {
  assert(op && "expected valid op");
  // Gather potential ancestors while looking for a "scope" parent region.
  SmallVector<Operation *, 8> ancestors;
  Region *region = nullptr;
  do {
    ancestors.push_back(op);
    region = op->getParentRegion();
    if (config.getScope() == region) {
      // Scope (can be `nullptr`) was reached. Stop traveral and enqueue ops.
      for (Operation *op : ancestors)
        addSingleOpToWorklist(op);
      return;
    }
    if (region == nullptr)
      return;
  } while ((op = region->getParentOp()));
}

```
- **EN**: Implements logic around `addToWorklist`, `assert`, `push_back`, `getParentRegion`, and 3 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `addToWorklist`、`assert`、`push_back`、`getParentRegion` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 660-676
```cpp
void GreedyPatternRewriteDriver::addSingleOpToWorklist(Operation *op) {
  if (config.getStrictness() == GreedyRewriteStrictness::AnyOp ||
      strictModeFilteredOps.contains(op))
    worklist.push(op);
}

void GreedyPatternRewriteDriver::notifyBlockInserted(
    Block *block, Region *previous, Region::iterator previousIt) {
  if (RewriterBase::Listener *listener = config.getListener())
    listener->notifyBlockInserted(block, previous, previousIt);
}

void GreedyPatternRewriteDriver::notifyBlockErased(Block *block) {
  if (RewriterBase::Listener *listener = config.getListener())
    listener->notifyBlockErased(block);
}

```
- **EN**: Implements logic around `addSingleOpToWorklist`, `getStrictness`, `contains`, `push`, and 3 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `addSingleOpToWorklist`、`getStrictness`、`contains`、`push` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 677-699
```cpp
void GreedyPatternRewriteDriver::notifyOperationInserted(
    Operation *op, OpBuilder::InsertPoint previous) {
  LLVM_DEBUG({
    logger.startLine() << "** Insert  : '" << op->getName() << "'(" << op
                       << ")\n";
  });
  if (RewriterBase::Listener *listener = config.getListener())
    listener->notifyOperationInserted(op, previous);
  if (config.getStrictness() == GreedyRewriteStrictness::ExistingAndNewOps)
    strictModeFilteredOps.insert(op);
  addToWorklist(op);
}

void GreedyPatternRewriteDriver::notifyOperationModified(Operation *op) {
  LLVM_DEBUG({
    logger.startLine() << "** Modified: '" << op->getName() << "'(" << op
                       << ")\n";
  });
  if (RewriterBase::Listener *listener = config.getListener())
    listener->notifyOperationModified(op);
  addToWorklist(op);
}

```
- **EN**: Implements logic around `notifyOperationInserted`, `startLine`, `getListener`, `getStrictness`, and 3 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `notifyOperationInserted`、`startLine`、`getListener`、`getStrictness` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 700-713
```cpp
void GreedyPatternRewriteDriver::addOperandsToWorklist(Operation *op) {
  for (Value operand : op->getOperands()) {
    // If this operand currently has at most 2 users, add its defining op to the
    // worklist. Indeed, after the op is deleted, then the operand will have at
    // most 1 user left. If it has 0 users left, it can be deleted too,
    // and if it has 1 user left, there may be further canonicalization
    // opportunities.
    if (!operand)
      continue;

    auto *defOp = operand.getDefiningOp();
    if (!defOp)
      continue;

```
- **EN**: Implements logic around `addOperandsToWorklist`, `getOperands`, `getDefiningOp`; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `addOperandsToWorklist`、`getOperands`、`getDefiningOp` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 714-728
```cpp
    Operation *otherUser = nullptr;
    bool hasMoreThanTwoUses = false;
    for (auto *user : operand.getUsers()) {
      if (user == op || user == otherUser)
        continue;
      if (!otherUser) {
        otherUser = user;
        continue;
      }
      hasMoreThanTwoUses = true;
      break;
    }
    if (hasMoreThanTwoUses)
      continue;

```
- **EN**: Implements logic around `getUsers`; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `getUsers` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 729-749
```cpp
    addToWorklist(defOp);
  }
}

void GreedyPatternRewriteDriver::notifyOperationErased(Operation *op) {
  LLVM_DEBUG({
    logger.startLine() << "** Erase   : '" << op->getName() << "'(" << op
                       << ")\n";
  });

#ifndef NDEBUG
  // Only ops that are within the configured scope are added to the worklist of
  // the greedy pattern rewriter. Moreover, the parent op of the scope region is
  // the part of the IR that is taken into account for the "expensive checks".
  // A greedy pattern rewrite is not allowed to erase the parent op of the scope
  // region, as that would break the worklist handling and the expensive checks.
  if (Region *scope = config.getScope(); scope->getParentOp() == op)
    llvm_unreachable(
        "scope region must not be erased during greedy pattern rewrite");
#endif // NDEBUG

```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 750-769
```cpp
  if (RewriterBase::Listener *listener = config.getListener())
    listener->notifyOperationErased(op);

  addOperandsToWorklist(op);
  worklist.remove(op);

  if (config.getStrictness() != GreedyRewriteStrictness::AnyOp)
    strictModeFilteredOps.erase(op);
}

void GreedyPatternRewriteDriver::notifyOperationReplaced(
    Operation *op, ValueRange replacement) {
  LLVM_DEBUG({
    logger.startLine() << "** Replace : '" << op->getName() << "'(" << op
                       << ")\n";
  });
  if (RewriterBase::Listener *listener = config.getListener())
    listener->notifyOperationReplaced(op, replacement);
}

```
- **EN**: Implements logic around `getListener`, `notifyOperationErased`, `addOperandsToWorklist`, `remove`, and 4 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `getListener`、`notifyOperationErased`、`addOperandsToWorklist`、`remove` 等另外 4 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 770-783
```cpp
void GreedyPatternRewriteDriver::notifyMatchFailure(
    Location loc, function_ref<void(Diagnostic &)> reasonCallback) {
  LLVM_DEBUG({
    Diagnostic diag(loc, DiagnosticSeverity::Remark);
    reasonCallback(diag);
    logger.startLine() << "** Match Failure : " << diag.str() << "\n";
  });
  if (RewriterBase::Listener *listener = config.getListener())
    listener->notifyMatchFailure(loc, reasonCallback);
}

//===----------------------------------------------------------------------===//
// RegionPatternRewriteDriver
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `notifyMatchFailure`, `function_ref`, `diag`, `reasonCallback`, and 2 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `notifyMatchFailure`、`function_ref`、`diag`、`reasonCallback` 等另外 2 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 784-798
```cpp

namespace {
/// This driver simplfies all ops in a region. If a scope is set in the
/// config, the provided region must be within that scope.
class RegionPatternRewriteDriver : public GreedyPatternRewriteDriver {
public:
  explicit RegionPatternRewriteDriver(MLIRContext *ctx,
                                      const FrozenRewritePatternSet &patterns,
                                      const GreedyRewriteConfig &config,
                                      Region &regions);

  /// Simplify ops inside `region` and simplify the region itself. Return
  /// success if the transformation converged.
  LogicalResult simplify(bool *changed) &&;

```
- **EN**: Introduces declarations for `RegionPatternRewriteDriver`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `RegionPatternRewriteDriver` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 799-823
```cpp
private:
  /// The region that is simplified.
  Region &region;
};
} // namespace

RegionPatternRewriteDriver::RegionPatternRewriteDriver(
    MLIRContext *ctx, const FrozenRewritePatternSet &patterns,
    const GreedyRewriteConfig &config, Region &region)
    : GreedyPatternRewriteDriver(ctx, patterns, config), region(region) {
  // Populate strict mode ops.
  if (config.getStrictness() != GreedyRewriteStrictness::AnyOp) {
    region.walk([&](Operation *op) { strictModeFilteredOps.insert(op); });
  }
#ifndef NDEBUG
  // Verify that the region is within the configured scope (if any).
  if (Region *scope = config.getScope()) {
    Region *r = &region;
    while (r && r != scope)
      r = r->getParentRegion();
    assert(r && "provided region is not within the config scope");
  }
#endif
}

```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 824-837
```cpp
namespace {
class GreedyPatternRewriteIteration
    : public tracing::ActionImpl<GreedyPatternRewriteIteration> {
public:
  MLIR_DEFINE_EXPLICIT_INTERNAL_INLINE_TYPE_ID(GreedyPatternRewriteIteration)
  GreedyPatternRewriteIteration(ArrayRef<IRUnit> units, int64_t iteration)
      : tracing::ActionImpl<GreedyPatternRewriteIteration>(units),
        iteration(iteration) {}
  static constexpr StringLiteral tag = "GreedyPatternRewriteIteration";
  void print(raw_ostream &os) const override {
    os << "GreedyPatternRewriteIteration(" << iteration << ")";
  }

private:
```
- **EN**: Introduces declarations for `GreedyPatternRewriteIteration`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `GreedyPatternRewriteIteration` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 838-851
```cpp
  int64_t iteration = 0;
};
} // namespace

LogicalResult RegionPatternRewriteDriver::simplify(bool *changed) && {
  bool continueRewrites = false;
  int64_t iteration = 0;
  MLIRContext *ctx = rewriter.getContext();
  do {
    // Check if the iteration limit was reached.
    if (++iteration > config.getMaxIterations() &&
        config.getMaxIterations() != GreedyRewriteConfig::kNoLimit)
      break;

```
- **EN**: Implements logic around `simplify`, `getContext`, `getMaxIterations`; this block makes success/failure or diagnostics explicit through MLIR result utilities; implements transformation or simplification logic.
- **CN**: 围绕 `simplify`、`getContext`、`getMaxIterations` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并实现变换或简化逻辑。

### Lines 852-867
```cpp
    // New iteration: start with an empty worklist.
    worklist.clear();

    // `OperationFolder` CSE's constant ops (and may move them into parents
    // regions to enable more aggressive CSE'ing).
    OperationFolder folder(ctx, this);
    auto insertKnownConstant = [&](Operation *op) {
      // Check for existing constants when populating the worklist. This avoids
      // accidentally reversing the constant order during processing.
      Attribute constValue;
      if (matchPattern(op, m_Constant(&constValue)))
        if (!folder.insertKnownConstant(op, constValue))
          return true;
      return false;
    };

```
- **EN**: Implements logic around `clear`, `folder`, `matchPattern`, `insertKnownConstant`; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `clear`、`folder`、`matchPattern`、`insertKnownConstant` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 868-883
```cpp
    if (!config.getUseTopDownTraversal()) {
      // Add operations to the worklist in postorder.
      region.walk([&](Operation *op) {
        if (!config.isConstantCSEEnabled() || !insertKnownConstant(op))
          addToWorklist(op);
      });
    } else {
      // Add all nested operations to the worklist in preorder.
      region.walk<WalkOrder::PreOrder>([&](Operation *op) {
        if (!config.isConstantCSEEnabled() || !insertKnownConstant(op)) {
          addToWorklist(op);
          return WalkResult::advance();
        }
        return WalkResult::skip();
      });

```
- **EN**: Implements logic around `getUseTopDownTraversal`, `walk`, `isConstantCSEEnabled`, `addToWorklist`, and 3 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `getUseTopDownTraversal`、`walk`、`isConstantCSEEnabled`、`addToWorklist` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 884-900
```cpp
      // Reverse the list so our pop-back loop processes them in-order.
      worklist.reverse();
    }

    ctx->executeAction<GreedyPatternRewriteIteration>(
        [&] {
          continueRewrites = false;

          // Erase unreachable blocks
          // Operations like:
          //   %add = arith.addi %add, %add : i64
          // are legal in unreachable code. Unfortunately many patterns would be
          // unsafe to apply on such IR and can lead to crashes or infinite
          // loops.
          continueRewrites |=
              succeeded(eraseUnreachableBlocks(rewriter, region));

```
- **EN**: Implements logic around `reverse`, `executeAction`, `succeeded`; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `reverse`、`executeAction`、`succeeded` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 901-925
```cpp
          continueRewrites |= processWorklist();

          // After applying patterns, make sure that the CFG of each of the
          // regions is kept up to date.
          if (config.getRegionSimplificationLevel() !=
              GreedySimplifyRegionLevel::Disabled) {
            continueRewrites |= succeeded(simplifyRegions(
                rewriter, region,
                /*mergeBlocks=*/config.getRegionSimplificationLevel() ==
                    GreedySimplifyRegionLevel::Aggressive));
          }

          // Optionally run full CSE. If CSE changes the IR we iterate again so
          // that patterns can fire on the deduplicated operations.
          if (config.isCSEBetweenIterationsEnabled()) {
            DominanceInfo domInfo;
            bool cseChanged = false;
            eliminateCommonSubExpressions(rewriter, domInfo, region,
                                          &cseChanged);
            continueRewrites |= cseChanged;
          }
        },
        {&region}, iteration);
  } while (continueRewrites);

```
- **EN**: Implements logic around `processWorklist`, `getRegionSimplificationLevel`, `succeeded`, `isCSEBetweenIterationsEnabled`, and 1 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `processWorklist`、`getRegionSimplificationLevel`、`succeeded`、`isCSEBetweenIterationsEnabled` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 926-942
```cpp
  if (changed)
    *changed = iteration > 1;

  // Whether the rewrite converges, i.e. wasn't changed in the last iteration.
  return success(!continueRewrites);
}

LogicalResult
mlir::applyPatternsGreedily(Region &region,
                            const FrozenRewritePatternSet &patterns,
                            GreedyRewriteConfig config, bool *changed) {
  // The top-level operation must be known to be isolated from above to
  // prevent performing canonicalizations on operations defined at or above
  // the region containing 'op'.
  assert(region.getParentOp()->hasTrait<OpTrait::IsIsolatedFromAbove>() &&
         "patterns can only be applied to operations IsolatedFromAbove");

```
- **EN**: Implements logic around `success`, `applyPatternsGreedily`, `assert`; this block makes success/failure or diagnostics explicit through MLIR result utilities; relies on MLIR pattern rewriting infrastructure; manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `success`、`applyPatternsGreedily`、`assert` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并依赖 MLIR 模式重写基础设施，并操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 943-962
```cpp
  // Set scope if not specified.
  if (!config.getScope())
    config.setScope(&region);

#if MLIR_ENABLE_EXPENSIVE_PATTERN_API_CHECKS
  if (failed(verify(config.getScope()->getParentOp())))
    llvm::report_fatal_error(
        "greedy pattern rewriter input IR failed to verify");
#endif // MLIR_ENABLE_EXPENSIVE_PATTERN_API_CHECKS

  // Start the pattern driver.
  RegionPatternRewriteDriver driver(region.getContext(), patterns, config,
                                    region);
  LogicalResult converged = std::move(driver).simplify(changed);
  if (failed(converged))
    LDBG() << "The pattern rewrite did not converge after scanning "
           << config.getMaxIterations() << " times";
  return converged;
}

```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 963-976
```cpp
//===----------------------------------------------------------------------===//
// MultiOpPatternRewriteDriver
//===----------------------------------------------------------------------===//

namespace {
/// This driver simplfies a list of ops.
class MultiOpPatternRewriteDriver : public GreedyPatternRewriteDriver {
public:
  explicit MultiOpPatternRewriteDriver(
      MLIRContext *ctx, const FrozenRewritePatternSet &patterns,
      const GreedyRewriteConfig &config, ArrayRef<Operation *> ops,
      llvm::SmallDenseSet<Operation *, 4> *survivingOps = nullptr);

  /// Simplify `ops`. Return `success` if the transformation converged.
```
- **EN**: Introduces declarations for `MultiOpPatternRewriteDriver`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `MultiOpPatternRewriteDriver` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 977-992
```cpp
  LogicalResult simplify(ArrayRef<Operation *> ops, bool *changed = nullptr) &&;

private:
  void notifyOperationErased(Operation *op) override {
    GreedyPatternRewriteDriver::notifyOperationErased(op);
    if (survivingOps)
      survivingOps->erase(op);
  }

  /// An optional set of ops that survived the rewrite. This set is populated
  /// at the beginning of `simplifyLocally` with the inititally provided list
  /// of ops.
  llvm::SmallDenseSet<Operation *, 4> *const survivingOps = nullptr;
};
} // namespace

```
- **EN**: Implements logic around `simplify`, `notifyOperationErased`, `erase`; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `simplify`、`notifyOperationErased`、`erase` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 993-1007
```cpp
MultiOpPatternRewriteDriver::MultiOpPatternRewriteDriver(
    MLIRContext *ctx, const FrozenRewritePatternSet &patterns,
    const GreedyRewriteConfig &config, ArrayRef<Operation *> ops,
    llvm::SmallDenseSet<Operation *, 4> *survivingOps)
    : GreedyPatternRewriteDriver(ctx, patterns, config),
      survivingOps(survivingOps) {
  if (config.getStrictness() != GreedyRewriteStrictness::AnyOp)
    strictModeFilteredOps.insert_range(ops);

  if (survivingOps) {
    survivingOps->clear();
    survivingOps->insert_range(ops);
  }
}

```
- **EN**: Implements logic around `MultiOpPatternRewriteDriver`, `GreedyPatternRewriteDriver`, `survivingOps`, `getStrictness`, and 2 more symbols; this block relies on MLIR pattern rewriting infrastructure; manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `MultiOpPatternRewriteDriver`、`GreedyPatternRewriteDriver`、`survivingOps`、`getStrictness` 等另外 2 个符号 实现具体逻辑；该代码块依赖 MLIR 模式重写基础设施，并操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 1008-1021
```cpp
LogicalResult MultiOpPatternRewriteDriver::simplify(ArrayRef<Operation *> ops,
                                                    bool *changed) && {
  // Populate the initial worklist.
  for (Operation *op : ops)
    addSingleOpToWorklist(op);

  // Process ops on the worklist.
  bool result = processWorklist();
  if (changed)
    *changed = result;

  return success(worklist.empty());
}

```
- **EN**: Implements logic around `simplify`, `addSingleOpToWorklist`, `processWorklist`, `success`; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `simplify`、`addSingleOpToWorklist`、`processWorklist`、`success` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 1022-1049
```cpp
/// Find the region that is the closest common ancestor of all given ops.
///
/// Note: This function returns `nullptr` if there is a top-level op among the
/// given list of ops.
static Region *findCommonAncestor(ArrayRef<Operation *> ops) {
  assert(!ops.empty() && "expected at least one op");
  // Fast path in case there is only one op.
  if (ops.size() == 1)
    return ops.front()->getParentRegion();

  Region *region = ops.front()->getParentRegion();
  ops = ops.drop_front();
  int sz = ops.size();
  llvm::BitVector remainingOps(sz, true);
  while (region) {
    int pos = -1;
    // Iterate over all remaining ops.
    while ((pos = remainingOps.find_first_in(pos + 1, sz)) != -1) {
      // Is this op contained in `region`?
      if (region->findAncestorOpInRegion(*ops[pos]))
        remainingOps.reset(pos);
    }
    if (remainingOps.none())
      break;
    region = region->getParentRegion();
  }
  return region;
}
```
- **EN**: Implements logic around `findCommonAncestor`, `assert`, `size`, `front`, and 7 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `findCommonAncestor`、`assert`、`size`、`front` 等另外 7 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 1050-1076
```cpp

LogicalResult mlir::applyOpPatternsGreedily(
    ArrayRef<Operation *> ops, const FrozenRewritePatternSet &patterns,
    GreedyRewriteConfig config, bool *changed, bool *allErased) {
  if (ops.empty()) {
    if (changed)
      *changed = false;
    if (allErased)
      *allErased = true;
    return success();
  }

  // Determine scope of rewrite.
  if (!config.getScope()) {
    // Compute scope if none was provided. The scope will remain `nullptr` if
    // there is a top-level op among `ops`.
    config.setScope(findCommonAncestor(ops));
  } else {
    // If a scope was provided, make sure that all ops are in scope.
#ifndef NDEBUG
    bool allOpsInScope = llvm::all_of(ops, [&](Operation *op) {
      return static_cast<bool>(config.getScope()->findAncestorOpInRegion(*op));
    });
    assert(allOpsInScope && "ops must be within the specified scope");
#endif // NDEBUG
  }

```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 1077-1095
```cpp
#if MLIR_ENABLE_EXPENSIVE_PATTERN_API_CHECKS
  if (config.getScope() && failed(verify(config.getScope()->getParentOp())))
    llvm::report_fatal_error(
        "greedy pattern rewriter input IR failed to verify");
#endif // MLIR_ENABLE_EXPENSIVE_PATTERN_API_CHECKS

  // Start the pattern driver.
  llvm::SmallDenseSet<Operation *, 4> surviving;
  MultiOpPatternRewriteDriver driver(ops.front()->getContext(), patterns,
                                     config, ops,
                                     allErased ? &surviving : nullptr);
  LogicalResult converged = std::move(driver).simplify(ops, changed);
  if (allErased)
    *allErased = surviving.empty();
  if (failed(converged))
    LDBG() << "The pattern rewrite did not converge after "
           << config.getMaxNumRewrites() << " rewrites";
  return converged;
}
```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

## Key Concepts / 关键概念

- **IR transformation passes / IR 变换 Pass**:
  - **EN**: Applies canonicalization, dataflow-aware simplification, or structural rewrites to MLIR.
  - **CN**: 对 MLIR 应用规范化、数据流感知简化或结构性重写。
- **SSA and region-based IR / SSA 与基于 Region 的 IR**:
  - **EN**: The file works with MLIR operations, blocks, values, or regions that form MLIR's hierarchical SSA IR.
  - **CN**: 该文件处理 MLIR 操作、块、值或 Region，它们共同构成 MLIR 的层次化 SSA IR。
- **Structured diagnostics / 结构化诊断**:
  - **EN**: Failures are surfaced through `LogicalResult`, diagnostics, or related reporting helpers.
  - **CN**: 通过 `LogicalResult`、诊断对象或相关报告工具显式传播失败。
- **Pattern rewriting / 模式重写**:
  - **EN**: The implementation uses rewrite patterns to match and transform IR.
  - **CN**: 该实现使用重写模式来匹配并变换 IR。
- **Streaming output / 流式输出**:
  - **EN**: Text, diagnostics, or generated content are emitted through LLVM stream APIs.
  - **CN**: 通过 LLVM 流 API 输出文本、诊断或生成内容。
- **LLVM ADT containers / LLVM ADT 容器**:
  - **EN**: The implementation uses LLVM containers for performance-conscious storage and traversal.
  - **CN**: 该实现使用 LLVM 容器来进行兼顾性能的存储与遍历。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `mlir/Transforms/GreedyPatternRewriteDriver.h`, `mlir/Config/mlir-config.h`, `mlir/IR/Action.h`, `mlir/IR/Dominance.h`, `mlir/IR/Matchers.h`, `mlir/IR/Operation.h`, `mlir/IR/OperationSupport.h`, `mlir/IR/Verifier.h`, `mlir/Interfaces/SideEffectInterfaces.h`, `mlir/Rewrite/PatternApplicator.h` ... (+9 more)
- **Standard-library headers / 标准库头文件**: `<random>`
- **Subsystem categories / 子系统类别**: core MLIR IR abstractions / 核心 MLIR IR 抽象 (6), core transformation utilities / 核心变换工具 (4), LLVM ADT containers and utilities / LLVM ADT 容器与工具 (3), LLVM support-library helpers / LLVM Support 库辅助工具 (3), MLIR interface declarations / MLIR 接口声明 (1), pattern rewrite support / 模式重写支持 (1)
