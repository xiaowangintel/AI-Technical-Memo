# LoopFusionUtils.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/Affine/LoopFusionUtils.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This header file defines prototypes for various loop fusion utility methods: these are not passes by themselves but are used either by passes, optimization sequences, or in turn by other transformation utilities.
  - **CN**: 该文件位于 `mlir/include/mlir/Dialect/Affine`，围绕 Affine 方言公开 `LoopFusionUtils` 相关的接口、规则或生成式定义。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- LoopFusionUtils.h - Loop fusion utilities ----------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-13
```cpp
//
// This header file defines prototypes for various loop fusion utility
// methods: these are not passes by themselves but are used either by passes,
// optimization sequences, or in turn by other transformation utilities.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 14-22
```cpp

#ifndef MLIR_DIALECT_AFFINE_LOOPFUSIONUTILS_H
#define MLIR_DIALECT_AFFINE_LOOPFUSIONUTILS_H

#include "mlir/IR/Value.h"
#include "mlir/Support/LLVM.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/ADT/SmallVector.h"

```
- **EN**: Pulls in the headers needed by this declaration unit, including `mlir/IR/Value.h`, `mlir/Support/LLVM.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/SmallVector.h`.
- **CN**: 引入该声明单元所需的头文件，其中包括 `mlir/IR/Value.h`, `mlir/Support/LLVM.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/SmallVector.h`。

### Lines 23-28
```cpp
namespace mlir {
class Operation;

namespace affine {
class AffineForOp;
struct ComputationSliceState;
```
- **EN**: Introduces declarations for `mlir`, `Operation`, `affine`, `AffineForOp`, and 1 more symbols, establishing the public symbols or declarative records used later.
- **CN**: 引入 `mlir`, `Operation`, `affine`, `AffineForOp`, and 1 more symbols 等声明，建立后续使用的公共符号或声明式记录。

### Lines 29-40
```cpp

struct FusionResult {
  enum ResultEnum {
    Success,
    FailPrecondition,     // Failed precondition for fusion. (e.g. same block).
    FailBlockDependence,  // Fusion would violate another dependence in block.
    FailFusionDependence, // Fusion would reverse dependences between loops.
    FailComputationSlice, // Unable to compute src loop computation slice.
    FailIncorrectSlice,   // Slice is computed, but it is incorrect.
  } value;
  FusionResult(ResultEnum v) : value(v) {}
};
```
- **EN**: Introduces declarations for `FusionResult`, `ResultEnum`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `FusionResult`, `ResultEnum` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 41-46
```cpp

/// Describes the fusion strategy to be used in the Affine loop fusion
/// utilities. Currently, it is used to specialized the loop fusion utilities
/// with the assumptions made in the AffineLoopFusion pass for producer-consumer
/// and sibling fusion, while sharing a single implementation. The latter
/// strategies are also limited to scenarios where a single memref is involved
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 47-52
```cpp
/// in the producer-consume or sibling relationship between the candidate
/// loops. We use 'memref' to keep track of such a memref.
// TODO: Generalize utilities so that producer-consumer and sibling fusion
// strategies can be used without the assumptions made in the AffineLoopFusion
// pass.
class FusionStrategy {
```
- **EN**: Introduces declarations for `FusionStrategy`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `FusionStrategy` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 53-64
```cpp
public:
  enum StrategyEnum {
    // Generic loop fusion: Arbitrary loops are considered for fusion. No
    // assumptions about a specific fusion strategy from AffineLoopFusion pass
    // are made.
    // TODO: Generic fusion is not fully implemented by fusion utilities yet.
    // It should only be used for testing.
    Generic,
    // Producer-consumer fusion: Only loops with a producer-consumer
    // memref dependence are considered for fusion. Currently, assumptions from
    // the producer-consumer fusion implementation in AffineLoopFusion pass are
    // made. See pass for specific details.
```
- **EN**: Introduces declarations for `StrategyEnum`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `StrategyEnum` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 65-73
```cpp
    ProducerConsumer,
    // Sibling fusion: Only sibling loops with no producer-consumer memref
    // dependences are considered for fusion. Memref reuse is taken into account
    // for profitability. Currently, assumptions from the sibling fusion
    // implementation in AffineLoopFusion pass are made. See pass for specific
    // details.
    Sibling
  };

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 74-79
```cpp
  /// Construct a generic or producer-consumer fusion strategy.
  FusionStrategy(StrategyEnum strategy) : strategy(strategy) {
    assert(strategy != Sibling &&
           "Sibling fusion strategy requires a specific memref");
  }

```
- **EN**: Implements logic around `FusionStrategy`, `assert`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `FusionStrategy`, `assert` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 80-86
```cpp
  /// Construct a sibling fusion strategy targeting 'memref'. This construct
  /// should only be used for sibling fusion.
  FusionStrategy(Value memref) : strategy(Sibling), memref(memref) {}

  /// Returns the fusion strategy.
  StrategyEnum getStrategy() const { return strategy; };

```
- **EN**: Implements logic around `FusionStrategy`, `getStrategy`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `FusionStrategy`, `getStrategy` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 87-92
```cpp
  /// Returns the memref attached to this sibling fusion strategy.
  Value getSiblingFusionMemRef() const {
    assert(strategy == Sibling && "Memref is only valid for sibling fusion");
    return memref;
  }

```
- **EN**: Implements logic around `getSiblingFusionMemRef`, `assert`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getSiblingFusionMemRef`, `assert` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 93-98
```cpp
private:
  /// Fusion strategy.
  StrategyEnum strategy;

  /// Target memref for this fusion transformation. Only used for sibling
  /// fusion.
```
- **EN**: Transitions between access-control regions inside a class or struct definition.
- **CN**: 在类或结构体定义内部切换访问控制区域。

### Lines 99-104
```cpp
  Value memref;
};

/// Checks the feasibility of fusing the loop nest rooted at 'srcForOp' into the
/// loop nest rooted at 'dstForOp' at 'dstLoopDepth'. Returns FusionResult
/// 'Success' if fusion of the src/dst loop nests is feasible (i.e. they are
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 105-113
```cpp
/// in the same block and dependences would not be violated). Otherwise
/// returns a FusionResult explaining why fusion is not feasible.
/// NOTE: This function is not feature complete and should only be used in
/// testing.
FusionResult
canFuseLoops(AffineForOp srcForOp, AffineForOp dstForOp, unsigned dstLoopDepth,
             ComputationSliceState *srcSlice,
             FusionStrategy fusionStrategy = FusionStrategy::Generic);

```
- **EN**: Declares APIs or declarative rules around `canFuseLoops`.
- **CN**: 声明与 `canFuseLoops` 相关的 API 或声明式规则。

### Lines 114-121
```cpp
/// Fuses 'srcForOp' into 'dstForOp' with destination loop block insertion
/// point and source slice loop bounds specified in 'srcSlice'.
/// `isInnermostSiblingInsertionFusion` enables cleanup of `srcForOp that is a
/// single-iteration reduction loop being sibling-fused into a 'dstForOp'.
void fuseLoops(AffineForOp srcForOp, AffineForOp dstForOp,
               const ComputationSliceState &srcSlice,
               bool isInnermostSiblingInsertionFusion = false);

```
- **EN**: Declares APIs or declarative rules around `fuseLoops`.
- **CN**: 声明与 `fuseLoops` 相关的 API 或声明式规则。

### Lines 122-128
```cpp
/// LoopNestStats aggregates various per-loop statistics (eg. loop trip count
/// and operation count) for a loop nest up until (and including) the innermost
/// loop body.
struct LoopNestStats {
  /// Map from AffineForOp to immediate child AffineForOps in its loop body.
  DenseMap<Operation *, SmallVector<AffineForOp, 2>> loopMap;
  /// Map from AffineForOp to count of operations in its loop body.
```
- **EN**: Introduces declarations for `LoopNestStats`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `LoopNestStats` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 129-134
```cpp
  DenseMap<Operation *, uint64_t> opCountMap;
  /// Map from AffineForOp to its constant trip count.
  DenseMap<Operation *, uint64_t> tripCountMap;
};

/// Collect loop nest statistics (eg. loop trip count and operation count)
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 135-140
```cpp
/// in 'stats' for loop nest rooted at 'forOp'. Returns true on success,
/// returns false otherwise.
// TODO: Consider moving this to LoopUtils.
bool getLoopNestStats(AffineForOp forOp, LoopNestStats *stats);

/// Computes the total cost of the loop nest rooted at 'forOp' using 'stats'.
```
- **EN**: Declares APIs or declarative rules around `getLoopNestStats`.
- **CN**: 声明与 `getLoopNestStats` 相关的 API 或声明式规则。

### Lines 141-146
```cpp
/// Currently, the total cost is computed by counting the total operation
/// instance count (i.e. total number of operations in the loop body * loop
/// trip count) for the entire loop nest.
int64_t getComputeCost(AffineForOp forOp, LoopNestStats &stats);

/// Computes and returns in 'computeCost', the total compute cost of fusing the
```
- **EN**: Declares APIs or declarative rules around `getComputeCost`.
- **CN**: 声明与 `getComputeCost` 相关的 API 或声明式规则。

### Lines 147-156
```cpp
/// 'slice' of the loop nest rooted at 'srcForOp' into 'dstForOp'. Currently,
/// the total cost is computed by counting the total operation instance count
/// (i.e. total number of operations in the loop body * loop trip count) for
/// the entire loop nest.
/// Returns true on success, failure otherwise (e.g. non-constant trip counts).
bool getFusionComputeCost(AffineForOp srcForOp, LoopNestStats &srcStats,
                          AffineForOp dstForOp, LoopNestStats &dstStats,
                          const ComputationSliceState &slice,
                          int64_t *computeCost);

```
- **EN**: Declares APIs or declarative rules around `getFusionComputeCost`.
- **CN**: 声明与 `getFusionComputeCost` 相关的 API 或声明式规则。

### Lines 157-163
```cpp
/// Returns in 'producerConsumerMemrefs' the memrefs involved in a
/// producer-consumer dependence between write ops in 'srcOps' and read ops in
/// 'dstOps'.
void gatherProducerConsumerMemrefs(ArrayRef<Operation *> srcOps,
                                   ArrayRef<Operation *> dstOps,
                                   DenseSet<Value> &producerConsumerMemrefs);

```
- **EN**: Declares APIs or declarative rules around `gatherProducerConsumerMemrefs`.
- **CN**: 声明与 `gatherProducerConsumerMemrefs` 相关的 API 或声明式规则。

### Lines 164-167
```cpp
} // namespace affine
} // namespace mlir

#endif // MLIR_DIALECT_AFFINE_LOOPFUSIONUTILS_H
```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

## Key Concepts / 关键概念

- **Dialect declarations / 方言声明**:
  - **EN**: Defines the public include-surface for a dialect, exposing operations, attributes, types, or interfaces.
  - **CN**: 定义方言的公共头文件表面，暴露操作、属性、类型或接口。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/IR/Value.h`, `mlir/Support/LLVM.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/SmallVector.h`
- **Subsystem categories / 子系统类别**: LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (2), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (1), support utilities and LLVM interop helpers / 支持工具与 LLVM 互操作辅助逻辑 (1)
