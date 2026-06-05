# RemoveDeadValues.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Transforms/RemoveDeadValues.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: The goal of this pass is optimization (reducing runtime) by removing unnecessary instructions. Unlike other passes that rely on local information gathered from patterns to accomplish optimization, this pass uses a full analysis of the IR, specifically, liveness analysis, and is thus more powerful.
  - **CN**: 实现核心 MLIR 变换 Pass 与变换辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-27
```cpp
//===- RemoveDeadValues.cpp - Remove Dead Values --------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// The goal of this pass is optimization (reducing runtime) by removing
// unnecessary instructions. Unlike other passes that rely on local information
// gathered from patterns to accomplish optimization, this pass uses a full
// analysis of the IR, specifically, liveness analysis, and is thus more
// powerful.
//
// Currently, this pass performs the following optimizations:
// (A) Removes function arguments that are not live,
// (B) Removes function return values that are not live across all callers of
// the function,
// (C) Removes unneccesary operands, results, region arguments, and region
// terminator operands of region branch ops, and,
// (D) Removes simple and region branch ops that have all non-live results and
// don't affect memory in any way.
//
// Here, a "simple op" refers to an op that isn't a symbol op, symbol-user op,
// region branch op, branch op, region branch terminator op, or return-like.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及高层摘要。

### Lines 28-55
```cpp

#include "mlir/Analysis/DataFlow/DeadCodeAnalysis.h"
#include "mlir/Analysis/DataFlow/LivenessAnalysis.h"
#include "mlir/Dialect/UB/IR/UBOps.h"
#include "mlir/IR/Builders.h"
#include "mlir/IR/BuiltinAttributes.h"
#include "mlir/IR/Dialect.h"
#include "mlir/IR/Operation.h"
#include "mlir/IR/OperationSupport.h"
#include "mlir/IR/SymbolTable.h"
#include "mlir/IR/Value.h"
#include "mlir/IR/ValueRange.h"
#include "mlir/IR/Visitors.h"
#include "mlir/Interfaces/CallInterfaces.h"
#include "mlir/Interfaces/ControlFlowInterfaces.h"
#include "mlir/Interfaces/FunctionInterfaces.h"
#include "mlir/Interfaces/SideEffectInterfaces.h"
#include "mlir/Pass/Pass.h"
#include "mlir/Support/LLVM.h"
#include "mlir/Transforms/FoldUtils.h"
#include "mlir/Transforms/Passes.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/DebugLog.h"
#include <cassert>
#include <cstddef>
#include <memory>
#include <optional>
```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/Analysis/DataFlow/DeadCodeAnalysis.h`, `mlir/Analysis/DataFlow/LivenessAnalysis.h`, `mlir/Dialect/UB/IR/UBOps.h`, `mlir/IR/Builders.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/Analysis/DataFlow/DeadCodeAnalysis.h`, `mlir/Analysis/DataFlow/LivenessAnalysis.h`, `mlir/Dialect/UB/IR/UBOps.h`, `mlir/IR/Builders.h`。

### Lines 56-70
```cpp
#include <vector>

#define DEBUG_TYPE "remove-dead-values"

namespace mlir {
#define GEN_PASS_DEF_REMOVEDEADVALUESPASS
#include "mlir/Transforms/Passes.h.inc"
} // namespace mlir

using namespace mlir;
using namespace mlir::dataflow;

//===----------------------------------------------------------------------===//
// RemoveDeadValues Pass
//===----------------------------------------------------------------------===//
```
- **EN**: Pulls in the declarations needed by this translation unit, including `vector`, `mlir/Transforms/Passes.h.inc`.
- **CN**: 引入该编译单元所需的声明，其中包括 `vector`, `mlir/Transforms/Passes.h.inc`。

### Lines 71-84
```cpp

namespace {

// Set of structures below to be filled with operations and arguments to erase.
// This is done to separate analysis and tree modification phases,
// otherwise analysis is operating on half-deleted tree which is incorrect.

struct FunctionToCleanUp {
  FunctionOpInterface funcOp;
  BitVector nonLiveArgs;
  BitVector nonLiveRets;
};

struct ResultsToCleanup {
```
- **EN**: Introduces declarations for `FunctionToCleanUp`, `ResultsToCleanup`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `FunctionToCleanUp`、`ResultsToCleanup` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 85-98
```cpp
  Operation *op;
  BitVector nonLive;
};

struct OperandsToCleanup {
  Operation *op;
  BitVector nonLive;
  // Optional: For CallOpInterface ops, stores the callee function.
  Operation *callee = nullptr;
  // Determines whether the operand should be replaced with a ub.poison result
  // or erased entirely.
  bool replaceWithPoison = false;
};

```
- **EN**: Introduces declarations for `OperandsToCleanup`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `OperandsToCleanup` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 99-118
```cpp
struct BlockArgsToCleanup {
  Block *b;
  BitVector nonLiveArgs;
};

struct SuccessorOperandsToCleanup {
  BranchOpInterface branch;
  unsigned successorIndex;
  BitVector nonLiveOperands;
};

struct RDVFinalCleanupList {
  SmallVector<Operation *> operations;
  SmallVector<FunctionToCleanUp> functions;
  SmallVector<OperandsToCleanup> operands;
  SmallVector<ResultsToCleanup> results;
  SmallVector<BlockArgsToCleanup> blocks;
  SmallVector<SuccessorOperandsToCleanup> successorOperands;
};

```
- **EN**: Introduces declarations for `BlockArgsToCleanup`, `SuccessorOperandsToCleanup`, `RDVFinalCleanupList`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `BlockArgsToCleanup`、`SuccessorOperandsToCleanup`、`RDVFinalCleanupList` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 119-145
```cpp
// Some helper functions...

/// Return true iff at least one value in `values` is live, given the liveness
/// information in `la`.
static bool hasLive(ValueRange values, const DenseSet<Value> &nonLiveSet,
                    RunLivenessAnalysis &la) {
  for (Value value : values) {
    if (nonLiveSet.contains(value)) {
      LDBG() << "Value " << value << " is already marked non-live (dead)";
      continue;
    }

    const Liveness *liveness = la.getLiveness(value);
    if (!liveness) {
      LDBG() << "Value " << value
             << " has no liveness info, conservatively considered live";
      return true;
    }
    if (liveness->isLive) {
      LDBG() << "Value " << value << " is live according to liveness analysis";
      return true;
    }
    LDBG() << "Value " << value << " is dead according to liveness analysis";
  }
  return false;
}

```
- **EN**: Implements logic around `hasLive`, `contains`, `LDBG`, `getLiveness`; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `hasLive`、`contains`、`LDBG`、`getLiveness` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 146-159
```cpp
/// Return a BitVector of size `values.size()` where its i-th bit is 1 iff the
/// i-th value in `values` is live, given the liveness information in `la`.
static BitVector markLives(ValueRange values, const DenseSet<Value> &nonLiveSet,
                           RunLivenessAnalysis &la) {
  BitVector lives(values.size(), true);

  for (auto [index, value] : llvm::enumerate(values)) {
    if (nonLiveSet.contains(value)) {
      lives.reset(index);
      LDBG() << "Value " << value
             << " is already marked non-live (dead) at index " << index;
      continue;
    }

```
- **EN**: Implements logic around `markLives`, `lives`, `enumerate`, `contains`, and 3 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `markLives`、`lives`、`enumerate`、`contains` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 160-181
```cpp
    const Liveness *liveness = la.getLiveness(value);
    // It is important to note that when `liveness` is null, we can't tell if
    // `value` is live or not. So, the safe option is to consider it live. Also,
    // the execution of this pass might create new SSA values when erasing some
    // of the results of an op and we know that these new values are live
    // (because they weren't erased) and also their liveness is null because
    // liveness analysis ran before their creation.
    if (!liveness) {
      LDBG() << "Value " << value << " at index " << index
             << " has no liveness info, conservatively considered live";
      continue;
    }
    if (!liveness->isLive) {
      lives.reset(index);
      LDBG() << "Value " << value << " at index " << index
             << " is dead according to liveness analysis";
    } else {
      LDBG() << "Value " << value << " at index " << index
             << " is live according to liveness analysis";
    }
  }

```
- **EN**: Implements logic around `getLiveness`, `LDBG`, `reset`; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `getLiveness`、`LDBG`、`reset` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 182-198
```cpp
  return lives;
}

/// Collects values marked as "non-live" in the provided range and inserts them
/// into the nonLiveSet. A value is considered "non-live" if the corresponding
/// index in the `nonLive` bit vector is set.
static void collectNonLiveValues(DenseSet<Value> &nonLiveSet, ValueRange range,
                                 const BitVector &nonLive) {
  for (auto [index, result] : llvm::enumerate(range)) {
    if (!nonLive[index])
      continue;
    nonLiveSet.insert(result);
    LDBG() << "Marking value " << result << " as non-live (dead) at index "
           << index;
  }
}

```
- **EN**: Implements logic around `collectNonLiveValues`, `enumerate`, `insert`, `LDBG`; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `collectNonLiveValues`、`enumerate`、`insert`、`LDBG` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 199-212
```cpp
/// Drop the uses of the i-th result of `op` and then erase it iff toErase[i]
/// is 1.
static void dropUsesAndEraseResults(RewriterBase &rewriter, Operation *op,
                                    BitVector toErase) {
  assert(op->getNumResults() == toErase.size() &&
         "expected the number of results in `op` and the size of `toErase` to "
         "be the same");
  for (auto idx : toErase.set_bits())
    op->getResult(idx).dropAllUses();
  rewriter.eraseOpResults(op, toErase);
}

/// Process a simple operation `op` using the liveness analysis `la`.
/// If the operation has no memory effects and none of its results are live:
```
- **EN**: Implements logic around `dropUsesAndEraseResults`, `assert`, `set_bits`, `getResult`, and 1 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `dropUsesAndEraseResults`、`assert`、`set_bits`、`getResult` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 213-240
```cpp
///   1. Add the operation to a list for future removal, and
///   2. Mark all its results as non-live values
///
/// The operation `op` is assumed to be simple. A simple operation is one that
/// is NOT:
///   - Function-like
///   - Call-like
///   - A region branch operation
///   - A branch operation
///   - A region branch terminator
///   - Return-like
static void processSimpleOp(Operation *op, RunLivenessAnalysis &la,
                            DenseSet<Value> &nonLiveSet,
                            RDVFinalCleanupList &cl) {
  // Operations that have dead operands can be erased regardless of their
  // side effects. The liveness analysis would not have marked an SSA value as
  // "dead" if it had a side-effecting user that is reachable.
  bool hasDeadOperand =
      markLives(op->getOperands(), nonLiveSet, la).flip().any();
  if (hasDeadOperand) {
    LDBG() << "Simple op has dead operands, so the op must be dead: "
           << OpWithFlags(op,
                          OpPrintingFlags().skipRegions().printGenericOpForm());
    assert(!hasLive(op->getResults(), nonLiveSet, la) &&
           "expected the op to have no live results");
    cl.operations.push_back(op);
    collectNonLiveValues(nonLiveSet, op->getResults(),
                         BitVector(op->getNumResults(), true));
```
- **EN**: Implements logic around `processSimpleOp`, `markLives`, `LDBG`, `OpWithFlags`, and 5 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations; implements transformation or simplification logic.
- **CN**: 围绕 `processSimpleOp`、`markLives`、`LDBG`、`OpWithFlags` 等另外 5 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示，并实现变换或简化逻辑。

### Lines 241-260
```cpp
    return;
  }

  if (!isMemoryEffectFree(op) || hasLive(op->getResults(), nonLiveSet, la)) {
    LDBG() << "Simple op is not memory effect free or has live results, "
              "preserving it: "
           << OpWithFlags(op,
                          OpPrintingFlags().skipRegions().printGenericOpForm());
    return;
  }

  LDBG()
      << "Simple op has all dead results and is memory effect free, scheduling "
         "for removal: "
      << OpWithFlags(op, OpPrintingFlags().skipRegions().printGenericOpForm());
  cl.operations.push_back(op);
  collectNonLiveValues(nonLiveSet, op->getResults(),
                       BitVector(op->getNumResults(), true));
}

```
- **EN**: Implements logic around `isMemoryEffectFree`, `LDBG`, `OpWithFlags`, `OpPrintingFlags`, and 3 more symbols; this block parses or prints textual MLIR representations; implements transformation or simplification logic.
- **CN**: 围绕 `isMemoryEffectFree`、`LDBG`、`OpWithFlags`、`OpPrintingFlags` 等另外 3 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示，并实现变换或简化逻辑。

### Lines 261-288
```cpp
/// Process a function-like operation `funcOp` using the liveness analysis `la`
/// and the IR in `module`. If it is not public or external:
///   (1) Adding its non-live arguments to a list for future removal.
///   (2) Marking their corresponding operands in its callers for removal.
///   (3) Identifying and enqueueing unnecessary terminator operands
///       (return values that are non-live across all callers) for removal.
///   (4) Enqueueing the non-live arguments and return values for removal.
///   (5) Collecting the uses of these return values in its callers for future
///       removal.
///   (6) Marking all its results as non-live values.
static void processFuncOp(FunctionOpInterface funcOp, Operation *module,
                          RunLivenessAnalysis &la, DenseSet<Value> &nonLiveSet,
                          RDVFinalCleanupList &cl) {
  LDBG() << "Processing function op: "
         << OpWithFlags(funcOp,
                        OpPrintingFlags().skipRegions().printGenericOpForm());
  if (funcOp.isPublic() || funcOp.isExternal()) {
    LDBG() << "Function is public or external, skipping: "
           << funcOp.getOperation()->getName();
    return;
  }
  SymbolTable::UseRange uses = *funcOp.getSymbolUses(module);
  if (llvm::any_of(uses, [](SymbolTable::SymbolUse use) {
        return !isa<CallOpInterface>(use.getUser());
      })) {
    // If a non-call operation references the function (e.g. spirv.EntryPoint),
    // we cannot safely remove arguments or return values since we don't know
    // what the user expects. Skip this function entirely.
```
- **EN**: Implements logic around `processFuncOp`, `LDBG`, `OpWithFlags`, `OpPrintingFlags`, and 5 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations; connects IR to external target or serialization formats; implements transformation or simplification logic.
- **CN**: 围绕 `processFuncOp`、`LDBG`、`OpWithFlags`、`OpPrintingFlags` 等另外 5 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示，并把 IR 连接到外部目标或序列化格式，并实现变换或简化逻辑。

### Lines 289-313
```cpp
    return;
  }
  // Get the list of unnecessary (non-live) arguments in `nonLiveArgs`.
  SmallVector<Value> arguments(funcOp.getArguments());
  BitVector nonLiveArgs = markLives(arguments, nonLiveSet, la);
  nonLiveArgs = nonLiveArgs.flip();

  // Do (1).
  for (auto [index, arg] : llvm::enumerate(arguments))
    if (arg && nonLiveArgs[index])
      nonLiveSet.insert(arg);

  // Do (2). (Skip creating generic operand cleanup entries for call ops.
  // Call arguments will be removed in the call-site specific segment-aware
  // cleanup, avoiding generic eraseOperands bitvector mechanics.)
  for (SymbolTable::SymbolUse use : uses) {
    Operation *callOp = use.getUser();
    // Push an empty operand cleanup entry so that call-site specific logic in
    // cleanUpDeadVals runs (it keys off CallOpInterface). The BitVector is
    // intentionally all false to avoid generic erasure.
    // Store the funcOp as the callee to avoid expensive symbol lookup later.
    cl.operands.push_back({callOp, BitVector(callOp->getNumOperands(), false),
                           funcOp.getOperation()});
  }

```
- **EN**: Implements logic around `arguments`, `markLives`, `flip`, `enumerate`, and 4 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `arguments`、`markLives`、`flip`、`enumerate` 等另外 4 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 314-341
```cpp
  // Do (3).
  // Get the list of unnecessary terminator operands (return values that are
  // non-live across all callers) in `nonLiveRets`. There is a very important
  // subtlety here. Unnecessary terminator operands are NOT the operands of the
  // terminator that are non-live. Instead, these are the return values of the
  // callers such that a given return value is non-live across all callers. Such
  // corresponding operands in the terminator could be live. An example to
  // demonstrate this:
  //  func.func private @f(%arg0: memref<i32>) -> (i32, i32) {
  //    %c0_i32 = arith.constant 0 : i32
  //    %0 = arith.addi %c0_i32, %c0_i32 : i32
  //    memref.store %0, %arg0[] : memref<i32>
  //    return %c0_i32, %0 : i32, i32
  //  }
  //  func.func @main(%arg0: i32, %arg1: memref<i32>) -> (i32) {
  //    %1:2 = call @f(%arg1) : (memref<i32>) -> i32
  //    return %1#0 : i32
  //  }
  // Here, we can see that %1#1 is never used. It is non-live. Thus, @f doesn't
  // need to return %0. But, %0 is live. And, still, we want to stop it from
  // being returned, in order to optimize our IR. So, this demonstrates how we
  // can make our optimization strong by even removing a live return value (%0),
  // since it forwards only to non-live value(s) (%1#1).
  size_t numReturns = funcOp.getNumResults();
  BitVector nonLiveRets(numReturns, true);
  for (SymbolTable::SymbolUse use : uses) {
    Operation *callOp = use.getUser();
    assert(isa<CallOpInterface>(callOp) && "expected a call-like user");
```
- **EN**: Implements logic around `getNumResults`, `nonLiveRets`, `getUser`, `assert`; this block manipulates core MLIR IR objects; bridges MLIR-produced code with runtime entry points; implements transformation or simplification logic.
- **CN**: 围绕 `getNumResults`、`nonLiveRets`、`getUser`、`assert` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并在 MLIR 生成代码与运行时入口点之间建立桥接，并实现变换或简化逻辑。

### Lines 342-356
```cpp
    BitVector liveCallRets = markLives(callOp->getResults(), nonLiveSet, la);
    nonLiveRets &= liveCallRets.flip();
  }

  // Note that in the absence of control flow ops forcing the control to go from
  // the entry (first) block to the other blocks, the control never reaches any
  // block other than the entry block, because every block has a terminator.
  for (Block &block : funcOp.getBlocks()) {
    Operation *returnOp = block.getTerminator();
    if (!returnOp->hasTrait<OpTrait::ReturnLike>())
      continue;
    if (returnOp && returnOp->getNumOperands() == numReturns)
      cl.operands.push_back({returnOp, nonLiveRets});
  }

```
- **EN**: Implements logic around `markLives`, `flip`, `getBlocks`, `getTerminator`, and 3 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `markLives`、`flip`、`getBlocks`、`getTerminator` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 357-370
```cpp
  // Do (4).
  cl.functions.push_back({funcOp, nonLiveArgs, nonLiveRets});

  // Do (5) and (6).
  if (numReturns == 0)
    return;
  for (SymbolTable::SymbolUse use : uses) {
    Operation *callOp = use.getUser();
    assert(isa<CallOpInterface>(callOp) && "expected a call-like user");
    cl.results.push_back({callOp, nonLiveRets});
    collectNonLiveValues(nonLiveSet, callOp->getResults(), nonLiveRets);
  }
}

```
- **EN**: Implements logic around `push_back`, `getUser`, `assert`, `collectNonLiveValues`; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `push_back`、`getUser`、`assert`、`collectNonLiveValues` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 371-384
```cpp
/// Process a region branch operation `regionBranchOp` using the liveness
/// information in `la`. The processing involves two scenarios:
///
/// Scenario 1: If the operation has no memory effects and none of its results
/// are live:
///   1.1. Enqueue all its uses for deletion.
///   1.2. Enqueue the branch itself for deletion.
///
/// Scenario 2: Otherwise:
///   2.1. Find all operands that are forwarded to only dead region successor
///        inputs. I.e., forwarded to block arguments / op results that we do
///        not want to keep.
///   2.2. Also find operands who's values are dead (i.e., are scheduled for
///        erasure) due to other operations.
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 385-408
```cpp
///   2.3. Enqueue all such operands for replacement with ub.poison.
///
/// Note: In scenario 2, block arguments and op results are not removed.
/// However, the IR is simplified such that canonicalization patterns can
/// remove them later.
static void processRegionBranchOp(RegionBranchOpInterface regionBranchOp,
                                  RunLivenessAnalysis &la,
                                  DenseSet<Value> &nonLiveSet,
                                  RDVFinalCleanupList &cl) {
  LDBG() << "Processing region branch op: "
         << OpWithFlags(regionBranchOp,
                        OpPrintingFlags().skipRegions().printGenericOpForm());

  // Scenario 1. This is the only case where the entire `regionBranchOp`
  // is removed. It will not happen in any other scenario. Note that in this
  // case, a non-forwarded operand of `regionBranchOp` could be live/non-live.
  // It could never be live because of this op but its liveness could have been
  // attributed to something else.
  if (isMemoryEffectFree(regionBranchOp.getOperation()) &&
      !hasLive(regionBranchOp->getResults(), nonLiveSet, la)) {
    cl.operations.push_back(regionBranchOp.getOperation());
    return;
  }

```
- **EN**: Implements logic around `processRegionBranchOp`, `LDBG`, `OpWithFlags`, `OpPrintingFlags`, and 3 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations; implements transformation or simplification logic.
- **CN**: 围绕 `processRegionBranchOp`、`LDBG`、`OpWithFlags`、`OpPrintingFlags` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示，并实现变换或简化逻辑。

### Lines 409-430
```cpp
  // Mapping from operands to forwarded successor inputs. An operand can be
  // forwarded to multiple successors.
  //
  // Example:
  //
  // %0 = scf.while : () -> i32 {
  //   scf.condition(...) %forwarded_value : i32
  // } do {
  // ^bb0(%arg0: i32):
  //   scf.yield
  // }
  // // No uses of %0.
  //
  // In the above example, %forwarded_value is forwarded to %arg0 and %0. Both
  // %arg0 and %0 are dead, so %forwarded_value can be replaced with a
  // ub.poison result.
  //
  // operandToSuccessorInputs[%forwarded_value] = {%arg0, %0}
  //
  RegionBranchSuccessorMapping operandToSuccessorInputs;
  regionBranchOp.getSuccessorOperandInputMapping(operandToSuccessorInputs);

```
- **EN**: Implements logic around `getSuccessorOperandInputMapping`; this block implements transformation or simplification logic.
- **CN**: 围绕 `getSuccessorOperandInputMapping` 实现具体逻辑；该代码块实现变换或简化逻辑。

### Lines 431-447
```cpp
  DenseMap<Operation *, BitVector> deadOperandsPerOp;
  for (auto [opOperand, successorInputs] : operandToSuccessorInputs) {
    // Helper function to mark the operand as dead, to be replaced with a
    // ub.poison result.
    auto markOperandDead = [&opOperand = opOperand, &deadOperandsPerOp]() {
      // Create an entry in `deadOperandsPerOp` (initialized to "false", i.e.,
      // no "dead" op operands) if it's the first time that we are seeing an op
      // operand for this op. Otherwise, just take the existing bit vector from
      // the map.
      BitVector &deadOperands =
          deadOperandsPerOp
              .try_emplace(opOperand->getOwner(),
                           opOperand->getOwner()->getNumOperands(), false)
              .first->second;
      deadOperands.set(opOperand->getOperandNumber());
    };

```
- **EN**: Implements logic around `try_emplace`, `getOwner`, `set`; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `try_emplace`、`getOwner`、`set` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 448-465
```cpp
    // The operand value is scheduled for removal. Mark it as dead.
    if (!hasLive(opOperand->get(), nonLiveSet, la)) {
      markOperandDead();
      continue;
    }

    // If one of the successor inputs is live, the respective operand must be
    // kept. Otherwise, ub.poison can be passed as operand.
    if (!hasLive(successorInputs, nonLiveSet, la))
      markOperandDead();
  }

  for (auto [op, deadOperands] : deadOperandsPerOp) {
    cl.operands.push_back(
        {op, deadOperands, nullptr, /*replaceWithPoison=*/true});
  }
}

```
- **EN**: Implements logic around `hasLive`, `markOperandDead`, `push_back`; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `hasLive`、`markOperandDead`、`push_back` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 466-482
```cpp
/// Steps to process a `BranchOpInterface` operation:
///
/// When a non-forwarded operand is dead (e.g., the condition value of a
/// conditional branch op), the entire operation is dead.
///
/// Otherwise, iterate through each successor block of `branchOp`.
/// (1) For each successor block, gather all operands from all successors.
/// (2) Fetch their associated liveness analysis data and collect for future
///     removal.
/// (3) Identify and collect the dead operands from the successor block
///     as well as their corresponding arguments.

static void processBranchOp(BranchOpInterface branchOp, RunLivenessAnalysis &la,
                            DenseSet<Value> &nonLiveSet,
                            RDVFinalCleanupList &cl) {
  LDBG() << "Processing branch op: " << *branchOp;

```
- **EN**: Implements logic around `processBranchOp`, `LDBG`; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `processBranchOp`、`LDBG` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 483-498
```cpp
  // Check for dead non-forwarded operands.
  BitVector deadNonForwardedOperands =
      markLives(branchOp->getOperands(), nonLiveSet, la).flip();
  unsigned numSuccessors = branchOp->getNumSuccessors();
  for (unsigned succIdx = 0; succIdx < numSuccessors; ++succIdx) {
    SuccessorOperands successorOperands =
        branchOp.getSuccessorOperands(succIdx);
    // Remove all non-forwarded operands from the bit vector.
    for (OpOperand &opOperand : successorOperands.getMutableForwardedOperands())
      deadNonForwardedOperands[opOperand.getOperandNumber()] = false;
  }
  if (deadNonForwardedOperands.any()) {
    cl.operations.push_back(branchOp.getOperation());
    return;
  }

```
- **EN**: Implements logic around `markLives`, `getNumSuccessors`, `getSuccessorOperands`, `getMutableForwardedOperands`, and 3 more symbols; this block implements transformation or simplification logic.
- **CN**: 围绕 `markLives`、`getNumSuccessors`、`getSuccessorOperands`、`getMutableForwardedOperands` 等另外 3 个符号 实现具体逻辑；该代码块实现变换或简化逻辑。

### Lines 499-516
```cpp
  for (unsigned succIdx = 0; succIdx < numSuccessors; ++succIdx) {
    Block *successorBlock = branchOp->getSuccessor(succIdx);

    // Do (1)
    SuccessorOperands successorOperands =
        branchOp.getSuccessorOperands(succIdx);
    SmallVector<Value> operandValues;
    for (unsigned operandIdx = 0; operandIdx < successorOperands.size();
         ++operandIdx) {
      operandValues.push_back(successorOperands[operandIdx]);
    }

    // Do (2)
    BitVector successorNonLive =
        markLives(operandValues, nonLiveSet, la).flip();
    collectNonLiveValues(nonLiveSet, successorBlock->getArguments(),
                         successorNonLive);

```
- **EN**: Implements logic around `getSuccessor`, `getSuccessorOperands`, `size`, `push_back`, and 2 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `getSuccessor`、`getSuccessorOperands`、`size`、`push_back` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 517-533
```cpp
    // Do (3)
    cl.blocks.push_back({successorBlock, successorNonLive});
    cl.successorOperands.push_back({branchOp, succIdx, successorNonLive});
  }
}

/// Create ub.poison ops for the given values. If a value has no uses, return
/// an "empty" value.
static SmallVector<Value> createPoisonedValues(OpBuilder &b,
                                               ValueRange values) {
  return llvm::map_to_vector(values, [&](Value value) {
    if (value.use_empty())
      return Value();
    return ub::PoisonOp::create(b, value.getLoc(), value.getType()).getResult();
  });
}

```
- **EN**: Implements logic around `push_back`, `createPoisonedValues`, `map_to_vector`, `use_empty`, and 2 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `push_back`、`createPoisonedValues`、`map_to_vector`、`use_empty` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 534-549
```cpp
namespace {
/// A listener that keeps track of ub.poison ops.
struct TrackingListener : public RewriterBase::Listener {
  void notifyOperationErased(Operation *op) override {
    if (auto poisonOp = dyn_cast<ub::PoisonOp>(op))
      poisonOps.erase(poisonOp);
  }
  void notifyOperationInserted(Operation *op,
                               OpBuilder::InsertPoint previous) override {
    if (auto poisonOp = dyn_cast<ub::PoisonOp>(op))
      poisonOps.insert(poisonOp);
  }
  DenseSet<ub::PoisonOp> poisonOps;
};
} // namespace

```
- **EN**: Introduces declarations for `TrackingListener`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `TrackingListener` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 550-577
```cpp
/// Removes dead values collected in RDVFinalCleanupList.
/// To be run once when all dead values have been collected.
static void cleanUpDeadVals(MLIRContext *ctx, RDVFinalCleanupList &list) {
  LDBG() << "Starting cleanup of dead values...";

  // New ub.poison ops may be inserted during cleanup. Some of these ops may no
  // longer be needed after the cleanup. A tracking listener keeps track of all
  // new ub.poison ops, so that they can be removed again after the cleanup.
  TrackingListener listener;
  IRRewriter rewriter(ctx, &listener);

  // 1. Blocks, We must remove the block arguments and successor operands before
  // deleting the operation, as they may reside in the region operation.
  LDBG() << "Cleaning up " << list.blocks.size() << " block argument lists";
  for (auto &b : list.blocks) {
    // blocks that are accessed via multiple codepaths processed once
    if (b.b->getNumArguments() != b.nonLiveArgs.size())
      continue;
    LDBG_OS([&](raw_ostream &os) {
      os << "Erasing non-live arguments [";
      llvm::interleaveComma(b.nonLiveArgs.set_bits(), os);
      os << "] from block #" << b.b->computeBlockNumber() << " in region #"
         << b.b->getParent()->getRegionNumber() << " of operation "
         << OpWithFlags(b.b->getParent()->getParentOp(),
                        OpPrintingFlags().skipRegions().printGenericOpForm());
    });
    // Note: Iterate from the end to make sure that that indices of not yet
    // processes arguments do not change.
```
- **EN**: Implements logic around `cleanUpDeadVals`, `LDBG`, `rewriter`, `getNumArguments`, and 6 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations; implements transformation or simplification logic.
- **CN**: 围绕 `cleanUpDeadVals`、`LDBG`、`rewriter`、`getNumArguments` 等另外 6 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示，并实现变换或简化逻辑。

### Lines 578-605
```cpp
    for (int i = b.nonLiveArgs.size() - 1; i >= 0; --i) {
      if (!b.nonLiveArgs[i])
        continue;
      b.b->getArgument(i).dropAllUses();
      b.b->eraseArgument(i);
    }
  }

  // 2. Successor Operands
  LDBG() << "Cleaning up " << list.successorOperands.size()
         << " successor operand lists";
  for (auto &op : list.successorOperands) {
    SuccessorOperands successorOperands =
        op.branch.getSuccessorOperands(op.successorIndex);
    // blocks that are accessed via multiple codepaths processed once
    if (successorOperands.size() != op.nonLiveOperands.size())
      continue;
    LDBG_OS([&](raw_ostream &os) {
      os << "Erasing non-live successor operands [";
      llvm::interleaveComma(op.nonLiveOperands.set_bits(), os);
      os << "] from successor " << op.successorIndex << " of branch: "
         << OpWithFlags(op.branch.getOperation(),
                        OpPrintingFlags().skipRegions().printGenericOpForm());
    });
    // it iterates backwards because erase invalidates all successor indexes
    for (int i = successorOperands.size() - 1; i >= 0; --i) {
      if (!op.nonLiveOperands[i])
        continue;
```
- **EN**: Implements logic around `size`, `getArgument`, `eraseArgument`, `LDBG`, and 5 more symbols; this block parses or prints textual MLIR representations; implements transformation or simplification logic.
- **CN**: 围绕 `size`、`getArgument`、`eraseArgument`、`LDBG` 等另外 5 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示，并实现变换或简化逻辑。

### Lines 606-633
```cpp
      successorOperands.erase(i);
    }
  }

  // 3. Functions
  LDBG() << "Cleaning up " << list.functions.size() << " functions";
  // Record which function arguments were erased so we can shrink call-site
  // argument segments for CallOpInterface operations (e.g. ops using
  // AttrSizedOperandSegments) in the next phase.
  DenseMap<Operation *, BitVector> erasedFuncArgs;
  for (auto &f : list.functions) {
    LDBG() << "Cleaning up function: " << f.funcOp.getName() << " ("
           << f.funcOp.getOperation() << ")";
    LDBG_OS([&](raw_ostream &os) {
      os << "  Erasing non-live arguments [";
      llvm::interleaveComma(f.nonLiveArgs.set_bits(), os);
      os << "]\n";
      os << "  Erasing non-live return values [";
      llvm::interleaveComma(f.nonLiveRets.set_bits(), os);
      os << "]";
    });
    // Drop all uses of the dead arguments.
    for (auto deadIdx : f.nonLiveArgs.set_bits())
      f.funcOp.getArgument(deadIdx).dropAllUses();
    // Some functions may not allow erasing arguments or results. These calls
    // return failure in such cases without modifying the function, so it's okay
    // to proceed.
    if (succeeded(f.funcOp.eraseArguments(f.nonLiveArgs))) {
```
- **EN**: Implements logic around `erase`, `LDBG`, `getOperation`, `LDBG_OS`, and 4 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `erase`、`LDBG`、`getOperation`、`LDBG_OS` 等另外 4 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 634-661
```cpp
      // Record only if we actually erased something.
      if (f.nonLiveArgs.any())
        erasedFuncArgs.try_emplace(f.funcOp.getOperation(), f.nonLiveArgs);
    } else {
      LDBG() << "Failed to erase arguments for function: "
             << f.funcOp.getName();
    }
    (void)f.funcOp.eraseResults(f.nonLiveRets);
  }

  // 4. Operands
  LDBG() << "Cleaning up " << list.operands.size() << " operand lists";
  for (OperandsToCleanup &o : list.operands) {
    // Handle call-specific cleanup only when we have a cached callee reference.
    // This avoids expensive symbol lookup and is defensive against future
    // changes.
    bool handledAsCall = false;
    if (o.callee && isa<CallOpInterface>(o.op)) {
      auto call = cast<CallOpInterface>(o.op);
      auto it = erasedFuncArgs.find(o.callee);
      if (it != erasedFuncArgs.end()) {
        const BitVector &deadArgIdxs = it->second;
        MutableOperandRange args = call.getArgOperandsMutable();
        // First, erase the call arguments corresponding to erased callee
        // args. We iterate backwards to preserve indices.
        for (unsigned argIdx : llvm::reverse(deadArgIdxs.set_bits()))
          args.erase(argIdx);
        // If this operand cleanup entry also has a generic nonLive bitvector,
```
- **EN**: Implements logic around `any`, `try_emplace`, `LDBG`, `getName`, and 8 more symbols; this block implements transformation or simplification logic.
- **CN**: 围绕 `any`、`try_emplace`、`LDBG`、`getName` 等另外 8 个符号 实现具体逻辑；该代码块实现变换或简化逻辑。

### Lines 662-689
```cpp
        // clear bits for call arguments we already erased above to avoid
        // double-erasing (which could impact other segments of ops with
        // AttrSizedOperandSegments).
        if (o.nonLive.any()) {
          // Map the argument logical index to the operand number(s) recorded.
          int operandOffset = call.getArgOperands().getBeginOperandIndex();
          for (int argIdx : deadArgIdxs.set_bits()) {
            int operandNumber = operandOffset + argIdx;
            if (operandNumber < static_cast<int>(o.nonLive.size()))
              o.nonLive.reset(operandNumber);
          }
        }
        handledAsCall = true;
      }
    }
    // Perform generic operand erasure for:
    // - Non-call operations
    // - Call operations without cached callee (where handledAsCall is false)
    // But skip call operations that were already handled via segment-aware path
    if (!handledAsCall && o.nonLive.any()) {
      LDBG_OS([&](raw_ostream &os) {
        os << "Erasing non-live operands [";
        llvm::interleaveComma(o.nonLive.set_bits(), os);
        os << "] from operation: "
           << OpWithFlags(o.op,
                          OpPrintingFlags().skipRegions().printGenericOpForm());
      });
      if (o.replaceWithPoison) {
```
- **EN**: Implements logic around `any`, `getArgOperands`, `set_bits`, `static_cast`, and 5 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations; implements transformation or simplification logic.
- **CN**: 围绕 `any`、`getArgOperands`、`set_bits`、`static_cast` 等另外 5 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示，并实现变换或简化逻辑。

### Lines 690-714
```cpp
        rewriter.setInsertionPoint(o.op);
        for (auto deadIdx : o.nonLive.set_bits()) {
          o.op->setOperand(
              deadIdx, createPoisonedValues(rewriter, o.op->getOperand(deadIdx))
                           .front());
        }
      } else {
        o.op->eraseOperands(o.nonLive);
      }
    }
  }

  // 5. Results
  LDBG() << "Cleaning up " << list.results.size() << " result lists";
  for (auto &r : list.results) {
    LDBG_OS([&](raw_ostream &os) {
      os << "Erasing non-live results [";
      llvm::interleaveComma(r.nonLive.set_bits(), os);
      os << "] from operation: "
         << OpWithFlags(r.op,
                        OpPrintingFlags().skipRegions().printGenericOpForm());
    });
    dropUsesAndEraseResults(rewriter, r.op, r.nonLive);
  }

```
- **EN**: Implements logic around `setInsertionPoint`, `set_bits`, `setOperand`, `createPoisonedValues`, and 8 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations; implements transformation or simplification logic.
- **CN**: 围绕 `setInsertionPoint`、`set_bits`、`setOperand`、`createPoisonedValues` 等另外 8 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示，并实现变换或简化逻辑。

### Lines 715-742
```cpp
  // 6. Operations
  LDBG() << "Cleaning up " << list.operations.size() << " operations";
  for (Operation *op : list.operations) {
    LDBG() << "Erasing operation: "
           << OpWithFlags(op,
                          OpPrintingFlags().skipRegions().printGenericOpForm());
    rewriter.setInsertionPoint(op);
    if (op->hasTrait<OpTrait::IsTerminator>()) {
      // When erasing a terminator, insert an unreachable op in its place.
      ub::UnreachableOp::create(rewriter, op->getLoc());
    }

    // Before erasing the operation, replace all result values with live-uses by
    // ub.poison values. This is important to maintain IR validity. For example,
    // if we have an op with one of its results used by another op, erasing the
    // op without replacing its corresponding result would leave us with a
    // dangling operand in the user op. By replacing the result with a ub.poison
    // value, we ensure that the user op still has a valid operand, even though
    // it's a poison value which will be cleaned up later if it can be cleaned
    // up. This keeps the IR valid for further simplification and
    // canonicalization.
    auto opResults = op->getResults();
    for (Value opResult : opResults) {
      // Early continue for the case where the op result has no uses. No need to
      // create a poison op here.
      if (opResult.use_empty())
        continue;

```
- **EN**: Implements logic around `LDBG`, `OpWithFlags`, `OpPrintingFlags`, `setInsertionPoint`, and 4 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations; implements transformation or simplification logic.
- **CN**: 围绕 `LDBG`、`OpWithFlags`、`OpPrintingFlags`、`setInsertionPoint` 等另外 4 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示，并实现变换或简化逻辑。

### Lines 743-757
```cpp
      rewriter.setInsertionPoint(op);
      Value poisonedValue = createPoisonedValues(rewriter, opResult).front();
      rewriter.replaceAllUsesWith(opResult, poisonedValue);
    }

    op->dropAllUses();
    rewriter.eraseOp(op);
  }

  // 7. Remove all dead poison ops.
  for (ub::PoisonOp poisonOp : listener.poisonOps) {
    if (poisonOp.use_empty())
      poisonOp.erase();
  }

```
- **EN**: Implements logic around `setInsertionPoint`, `createPoisonedValues`, `replaceAllUsesWith`, `dropAllUses`, and 3 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `setInsertionPoint`、`createPoisonedValues`、`replaceAllUsesWith`、`dropAllUses` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 758-772
```cpp
  LDBG() << "Finished cleanup of dead values";
}

struct RemoveDeadValues
    : public impl::RemoveDeadValuesPassBase<RemoveDeadValues> {
  using impl::RemoveDeadValuesPassBase<
      RemoveDeadValues>::RemoveDeadValuesPassBase;
  void runOnOperation() override;
};
} // namespace

void RemoveDeadValues::runOnOperation() {
  auto &la = getAnalysis<RunLivenessAnalysis>();
  Operation *module = getOperation();

```
- **EN**: Introduces declarations for `RemoveDeadValues`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `RemoveDeadValues` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 773-798
```cpp
  // Tracks values eligible for erasure - complements liveness analysis to
  // identify "droppable" values.
  DenseSet<Value> deadVals;

  // Maintains a list of Ops, values, branches, etc., slated for cleanup at the
  // end of this pass.
  RDVFinalCleanupList finalCleanupList;

  module->walk([&](Operation *op) {
    if (auto funcOp = dyn_cast<FunctionOpInterface>(op)) {
      processFuncOp(funcOp, module, la, deadVals, finalCleanupList);
    } else if (auto regionBranchOp = dyn_cast<RegionBranchOpInterface>(op)) {
      processRegionBranchOp(regionBranchOp, la, deadVals, finalCleanupList);
    } else if (auto branchOp = dyn_cast<BranchOpInterface>(op)) {
      processBranchOp(branchOp, la, deadVals, finalCleanupList);
    } else if (op->hasTrait<::mlir::OpTrait::IsTerminator>()) {
      // Nothing to do here because this is a terminator op and it should be
      // honored with respect to its parent
    } else if (isa<CallOpInterface>(op)) {
      // Nothing to do because this op is associated with a function op and gets
      // cleaned when the latter is cleaned.
    } else {
      processSimpleOp(op, la, deadVals, finalCleanupList);
    }
  });

```
- **EN**: Implements logic around `walk`, `dyn_cast`, `processFuncOp`, `processRegionBranchOp`, and 4 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `walk`、`dyn_cast`、`processFuncOp`、`processRegionBranchOp` 等另外 4 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 799-822
```cpp
  MLIRContext *context = module->getContext();
  cleanUpDeadVals(context, finalCleanupList);

  if (!canonicalize)
    return;

  // Canonicalize all region branch ops.
  SmallVector<Operation *> opsToCanonicalize;
  module->walk([&](RegionBranchOpInterface regionBranchOp) {
    opsToCanonicalize.push_back(regionBranchOp.getOperation());
  });
  // Collect all canonicalization patterns for region branch ops.
  RewritePatternSet owningPatterns(context);
  DenseSet<RegisteredOperationName> populatedPatterns;
  for (Operation *op : opsToCanonicalize)
    if (std::optional<RegisteredOperationName> info = op->getRegisteredInfo())
      if (populatedPatterns.insert(*info).second)
        info->getCanonicalizationPatterns(owningPatterns, context);
  if (failed(applyOpPatternsGreedily(opsToCanonicalize,
                                     std::move(owningPatterns)))) {
    module->emitError("greedy pattern rewrite failed to converge");
    signalPassFailure();
  }
}
```
- **EN**: Implements logic around `getContext`, `cleanUpDeadVals`, `walk`, `push_back`, and 8 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `getContext`、`cleanUpDeadVals`、`walk`、`push_back` 等另外 8 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

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
- **Runtime interop / 运行时互操作**:
  - **EN**: The code exposes or consumes ABI-level hooks used by generated code at runtime.
  - **CN**: 代码暴露或消费运行时由生成代码使用的 ABI 级钩子。
- **Target or format bridging / 目标或格式桥接**:
  - **EN**: IR is translated to, from, or alongside external target representations.
  - **CN**: 该文件把 IR 翻译到外部目标表示、从外部目标表示恢复，或与之协同工作。
- **Streaming output / 流式输出**:
  - **EN**: Text, diagnostics, or generated content are emitted through LLVM stream APIs.
  - **CN**: 通过 LLVM 流 API 输出文本、诊断或生成内容。
- **LLVM ADT containers / LLVM ADT 容器**:
  - **EN**: The implementation uses LLVM containers for performance-conscious storage and traversal.
  - **CN**: 该实现使用 LLVM 容器来进行兼顾性能的存储与遍历。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `mlir/Analysis/DataFlow/DeadCodeAnalysis.h`, `mlir/Analysis/DataFlow/LivenessAnalysis.h`, `mlir/Dialect/UB/IR/UBOps.h`, `mlir/IR/Builders.h`, `mlir/IR/BuiltinAttributes.h`, `mlir/IR/Dialect.h`, `mlir/IR/Operation.h`, `mlir/IR/OperationSupport.h`, `mlir/IR/SymbolTable.h`, `mlir/IR/Value.h` ... (+14 more)
- **Standard-library headers / 标准库头文件**: `<cassert>`, `<cstddef>`, `<memory>`, `<optional>`, `<vector>`
- **Subsystem categories / 子系统类别**: core MLIR IR abstractions / 核心 MLIR IR 抽象 (9), MLIR interface declarations / MLIR 接口声明 (4), core transformation utilities / 核心变换工具 (3), MLIR analysis interfaces / MLIR 分析接口 (2), LLVM support-library helpers / LLVM Support 库辅助工具 (2), pass-manager infrastructure / Pass 管理器基础设施 (1), shared MLIR support helpers / 共享的 MLIR 支持工具 (1), LLVM ADT containers and utilities / LLVM ADT 容器与工具 (1)
