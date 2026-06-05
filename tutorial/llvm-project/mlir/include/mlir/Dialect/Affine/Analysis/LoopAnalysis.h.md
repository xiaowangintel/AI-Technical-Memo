# LoopAnalysis.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/Affine/Analysis/LoopAnalysis.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This header file defines prototypes for methods to analyze loops.
  - **CN**: 该文件位于 `mlir/include/mlir/Dialect/Affine/Analysis`，围绕 Affine 方言公开 `LoopAnalysis` 相关的接口、规则或生成式定义。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- LoopAnalysis.h - loop analysis methods -------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-12
```cpp
//
// This header file defines prototypes for methods to analyze loops.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 13-19
```cpp
#ifndef MLIR_DIALECT_AFFINE_ANALYSIS_LOOPANALYSIS_H
#define MLIR_DIALECT_AFFINE_ANALYSIS_LOOPANALYSIS_H

#include "mlir/Support/LLVM.h"
#include "llvm/ADT/ArrayRef.h"
#include <optional>

```
- **EN**: Pulls in the headers needed by this declaration unit, including `mlir/Support/LLVM.h`, `llvm/ADT/ArrayRef.h`, `optional`.
- **CN**: 引入该声明单元所需的头文件，其中包括 `mlir/Support/LLVM.h`, `llvm/ADT/ArrayRef.h`, `optional`。

### Lines 20-24
```cpp
namespace mlir {
class AffineExpr;
class AffineMap;
class BlockArgument;
class MemRefType;
```
- **EN**: Introduces declarations for `mlir`, `AffineExpr`, `AffineMap`, `BlockArgument`, and 1 more symbols, establishing the public symbols or declarative records used later.
- **CN**: 引入 `mlir`, `AffineExpr`, `AffineMap`, `BlockArgument`, and 1 more symbols 等声明，建立后续使用的公共符号或声明式记录。

### Lines 25-29
```cpp
class Operation;
class Value;

namespace affine {
class AffineForOp;
```
- **EN**: Introduces declarations for `Operation`, `Value`, `affine`, `AffineForOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Operation`, `Value`, `affine`, `AffineForOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 30-34
```cpp
class NestedPattern;

/// Returns the trip count of the loop as an affine map with its corresponding
/// operands if the latter is expressible as an affine expression, and nullptr
/// otherwise. This method always succeeds as long as the lower bound is not a
```
- **EN**: Introduces declarations for `NestedPattern`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `NestedPattern` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 35-40
```cpp
/// multi-result map. The trip count expression is simplified before returning.
/// This method only utilizes map composition to construct lower and upper
/// bounds before computing the trip count expressions
void getTripCountMapAndOperands(AffineForOp forOp, AffineMap *map,
                                SmallVectorImpl<Value> *operands);

```
- **EN**: Declares APIs or declarative rules around `getTripCountMapAndOperands`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `getTripCountMapAndOperands` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 41-45
```cpp
/// Returns the trip count of the loop if it's a constant, std::nullopt
/// otherwise. This uses affine expression analysis and is able to determine
/// constant trip count in non-trivial cases.
std::optional<uint64_t> getConstantTripCount(AffineForOp forOp);

```
- **EN**: Declares APIs or declarative rules around `getConstantTripCount`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `getConstantTripCount` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 46-50
```cpp
/// Returns the greatest known integral divisor of the trip count. Affine
/// expression analysis is used (indirectly through getTripCount), and
/// this method is thus able to determine non-trivial divisors.
uint64_t getLargestDivisorOfTripCount(AffineForOp forOp);

```
- **EN**: Declares APIs or declarative rules around `getLargestDivisorOfTripCount`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `getLargestDivisorOfTripCount` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 51-55
```cpp
/// Checks if an affine read or write operation depends on `forOp`'s IV, i.e.,
/// if the memory access is invariant on `forOp`.
template <typename LoadOrStoreOp>
bool isInvariantAccess(LoadOrStoreOp memOp, AffineForOp forOp);

```
- **EN**: Declares APIs or declarative rules around `isInvariantAccess`; this block checks structural or semantic invariants; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `isInvariantAccess` 相关的 API 或声明式规则；该代码块检查结构或语义不变式，并协调核心结构化 MLIR 方言之间的行为。

### Lines 56-60
```cpp
/// Given an induction variable `iv` of type AffineForOp and `indices` of type
/// IndexType, returns the set of `indices` that are independent of `iv`.
///
/// Prerequisites (inherited from `isAccessInvariant` above):
///   1. `iv` and `indices` of the proper type;
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 61-67
```cpp
///   2. at most one affine.apply is reachable from each index in `indices`;
///
/// Emits a note if it encounters a chain of affine.apply and conservatively
///  those cases.
DenseSet<Value, DenseMapInfo<Value>>
getInvariantAccesses(Value iv, ArrayRef<Value> indices);

```
- **EN**: Declares APIs or declarative rules around `getInvariantAccesses`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `getInvariantAccesses` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 68-72
```cpp
/// Given:
///   1. an induction variable `iv` of type AffineForOp;
///   2. a `memoryOp` of type const LoadOp& or const StoreOp&;
/// determines whether `memoryOp` has a contiguous access along `iv`. Contiguous
/// is defined as either invariant or varying only along a unique MemRef dim.
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 73-77
```cpp
/// Upon success, the unique MemRef dim is written in `memRefDim` (or -1 to
/// convey the memRef access is invariant along `iv`).
///
/// Prerequisites:
///   1. `memRefDim` ~= nullptr;
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 78-82
```cpp
///   2. `iv` of the proper type;
///   3. the MemRef accessed by `memoryOp` has no layout map or at most an
///      identity layout map.
///
/// Currently only supports no layout map or identity layout map in the memref.
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 83-87
```cpp
/// Returns false if the memref has a non-identity layoutMap. This behavior is
/// conservative.
template <typename LoadOrStoreOp>
bool isContiguousAccess(Value iv, LoadOrStoreOp memoryOp, int *memRefDim);

```
- **EN**: Declares APIs or declarative rules around `isContiguousAccess`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `isContiguousAccess` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 88-92
```cpp
using VectorizableLoopFun = std::function<bool(AffineForOp)>;

/// Checks whether the loop is structurally vectorizable; i.e.:
///   1. no conditionals are nested under the loop;
///   2. all nested load/stores are to scalar MemRefs.
```
- **EN**: Declares APIs or declarative rules around `function`.
- **CN**: 声明与 `function` 相关的 API 或声明式规则。

### Lines 93-97
```cpp
/// TODO: relax the no-conditionals restriction
bool isVectorizableLoopBody(AffineForOp loop,
                            NestedPattern &vectorTransferMatcher);

/// Checks whether the loop is structurally vectorizable and that all the LoadOp
```
- **EN**: Declares APIs or declarative rules around `isVectorizableLoopBody`.
- **CN**: 声明与 `isVectorizableLoopBody` 相关的 API 或声明式规则。

### Lines 98-104
```cpp
/// and StoreOp matched have access indexing functions that are either:
///   1. invariant along the loop induction variable created by 'loop';
///   2. varying along at most one memory dimension. If such a unique dimension
///      is found, it is written into `memRefDim`.
bool isVectorizableLoopBody(AffineForOp loop, int *memRefDim,
                            NestedPattern &vectorTransferMatcher);

```
- **EN**: Declares APIs or declarative rules around `isVectorizableLoopBody`; this block checks structural or semantic invariants.
- **CN**: 声明与 `isVectorizableLoopBody` 相关的 API 或声明式规则；该代码块检查结构或语义不变式。

### Lines 105-111
```cpp
/// Checks where SSA dominance would be violated if a for op's body
/// operations are shifted by the specified shifts. This method checks if a
/// 'def' and all its uses have the same shift factor.
// TODO: extend this to check for memory-based dependence violation when we have
// the support.
bool isOpwiseShiftValid(AffineForOp forOp, ArrayRef<uint64_t> shifts);

```
- **EN**: Declares APIs or declarative rules around `isOpwiseShiftValid`.
- **CN**: 声明与 `isOpwiseShiftValid` 相关的 API 或声明式规则。

### Lines 112-116
```cpp
/// Checks whether hyper-rectangular loop tiling of the nest represented by
/// `loops` is valid. The validity condition is from Irigoin and Triolet,
/// which states that two tiles cannot depend on each other. We simplify such
/// condition to just checking whether there is any negative dependence
/// direction, since we have the prior knowledge that the tiling results will be
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 117-121
```cpp
/// hyper-rectangles, which are scheduled in the lexicographically increasing
/// order on the vector of loop indices. This function will return failure when
/// any dependence component is negative along any of `loops`.
bool isTilingValid(ArrayRef<AffineForOp> loops);

```
- **EN**: Declares APIs or declarative rules around `isTilingValid`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `isTilingValid` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 122-126
```cpp
/// Returns true if the affine nest rooted at `root` has a cyclic dependence
/// among its affine memory accesses. The dependence could be through any
/// dependences carried by loops contained in `root` (inclusive of `root`) and
/// those carried by loop bodies (blocks) contained. Dependences carried by
/// loops outer to `root` aren't relevant. This method doesn't consider/account
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 127-132
```cpp
/// for aliases.
bool hasCyclicDependence(AffineForOp root);

} // namespace affine
} // namespace mlir

```
- **EN**: Introduces declarations for `affine`, `mlir`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `affine`, `mlir` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 133-133
```cpp
#endif // MLIR_DIALECT_AFFINE_ANALYSIS_LOOPANALYSIS_H
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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Support/LLVM.h`, `llvm/ADT/ArrayRef.h`
- **Standard-library headers / 标准库头文件**: `<optional>`
- **Subsystem categories / 子系统类别**: support utilities and LLVM interop helpers / 支持工具与 LLVM 互操作辅助逻辑 (1), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (1)
