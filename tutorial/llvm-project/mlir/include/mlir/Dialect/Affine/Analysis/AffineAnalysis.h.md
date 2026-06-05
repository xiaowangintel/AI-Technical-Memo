# AffineAnalysis.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/Affine/Analysis/AffineAnalysis.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This header file defines prototypes for methods that perform analysis involving affine structures (AffineExprStorage, AffineMap, IntegerSet, etc.) and other IR structures that in turn use these.
  - **CN**: 该文件位于 `mlir/include/mlir/Dialect/Affine/Analysis`，围绕 Affine 方言公开 `AffineAnalysis` 相关的接口、规则或生成式定义。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- AffineAnalysis.h - analyses for affine structures --------*- C++ -*-===//
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
// This header file defines prototypes for methods that perform analysis
// involving affine structures (AffineExprStorage, AffineMap, IntegerSet, etc.)
// and other IR structures that in turn use these.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 14-23
```cpp

#ifndef MLIR_DIALECT_AFFINE_ANALYSIS_AFFINEANALYSIS_H
#define MLIR_DIALECT_AFFINE_ANALYSIS_AFFINEANALYSIS_H

#include "mlir/Analysis/Presburger/IntegerRelation.h"
#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/IR/Value.h"
#include "llvm/ADT/SmallVector.h"
#include <optional>

```
- **EN**: Pulls in the headers needed by this declaration unit, including `mlir/Analysis/Presburger/IntegerRelation.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/IR/Value.h`, `llvm/ADT/SmallVector.h`.
- **CN**: 引入该声明单元所需的头文件，其中包括 `mlir/Analysis/Presburger/IntegerRelation.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/IR/Value.h`, `llvm/ADT/SmallVector.h`。

### Lines 24-29
```cpp
namespace mlir {
class Operation;

namespace affine {
class AffineApplyOp;
class AffineForOp;
```
- **EN**: Introduces declarations for `mlir`, `Operation`, `affine`, `AffineApplyOp`, and 1 more symbols, establishing the public symbols or declarative records used later.
- **CN**: 引入 `mlir`, `Operation`, `affine`, `AffineApplyOp`, and 1 more symbols 等声明，建立后续使用的公共符号或声明式记录。

### Lines 30-35
```cpp
class AffineValueMap;
class FlatAffineRelation;
class FlatAffineValueConstraints;

/// A description of a (parallelizable) reduction in an affine loop.
struct LoopReduction {
```
- **EN**: Introduces declarations for `AffineValueMap`, `FlatAffineRelation`, `FlatAffineValueConstraints`, `LoopReduction`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineValueMap`, `FlatAffineRelation`, `FlatAffineValueConstraints`, `LoopReduction` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 36-41
```cpp
  /// Reduction kind.
  arith::AtomicRMWKind kind;

  /// Position of the iteration argument that acts as accumulator.
  unsigned iterArgPosition;

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 42-49
```cpp
  /// The value being reduced.
  Value value;
};

/// Populate `supportedReductions` with descriptors of the supported reductions.
void getSupportedReductions(
    AffineForOp forOp, SmallVectorImpl<LoopReduction> &supportedReductions);

```
- **EN**: Declares APIs or declarative rules around `getSupportedReductions`.
- **CN**: 声明与 `getSupportedReductions` 相关的 API 或声明式规则。

### Lines 50-56
```cpp
/// Returns true if `forOp' is a parallel loop. If `parallelReductions` is
/// provided, populates it with descriptors of the parallelizable reductions and
/// treats them as not preventing parallelization.
bool isLoopParallel(
    AffineForOp forOp,
    SmallVectorImpl<LoopReduction> *parallelReductions = nullptr);

```
- **EN**: Declares APIs or declarative rules around `isLoopParallel`.
- **CN**: 声明与 `isLoopParallel` 相关的 API 或声明式规则。

### Lines 57-63
```cpp
/// Returns true if `forOp' doesn't have memory dependences preventing
/// parallelization. Memrefs that are allocated inside `forOp` do not impact its
/// dependences and parallelism. This function does not check iter_args (for
/// values other than memref types) and should be used only as a building block
/// for complete parallelism-checking functions.
bool isLoopMemoryParallel(AffineForOp forOp);

```
- **EN**: Declares APIs or declarative rules around `isLoopMemoryParallel`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `isLoopMemoryParallel` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 64-69
```cpp
/// Returns in `affineApplyOps`, the sequence of those AffineApplyOp
/// Operations that are reachable via a search starting from `operands` and
/// ending at those operands that are not the result of an AffineApplyOp.
void getReachableAffineApplyOps(ArrayRef<Value> operands,
                                SmallVectorImpl<Operation *> &affineApplyOps);

```
- **EN**: Declares APIs or declarative rules around `getReachableAffineApplyOps`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `getReachableAffineApplyOps` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 70-75
```cpp
/// Builds a system of constraints with dimensional variables corresponding to
/// the loop IVs of the forOps and AffineIfOp's operands appearing in
/// that order. Bounds of the loop are used to add appropriate inequalities.
/// Constraints from the index sets of AffineIfOp are also added. Any symbols
/// founds in the bound operands are added as symbols in the system. Returns
/// failure for the yet unimplemented cases. `ops` accepts both AffineForOp and
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 76-81
```cpp
/// AffineIfOp.
//  TODO: handle non-unit strides.
LogicalResult getIndexSet(MutableArrayRef<Operation *> ops,
                          FlatAffineValueConstraints *domain);

/// Encapsulates a memref load or store access information.
```
- **EN**: Declares APIs or declarative rules around `getIndexSet`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `getIndexSet` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 82-87
```cpp
struct MemRefAccess {
  Value memref;
  Operation *opInst = nullptr;
  SmallVector<Value, 4> indices;

  /// Constructs a MemRefAccess from an affine read/write operation.
```
- **EN**: Introduces declarations for `MemRefAccess`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `MemRefAccess` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 88-96
```cpp
  explicit MemRefAccess(Operation *memOp);

  MemRefAccess() = default;

  // Returns the rank of the memref associated with this access.
  unsigned getRank() const;
  // Returns true if this access is of a store op.
  bool isStore() const;

```
- **EN**: Declares APIs or declarative rules around `MemRefAccess`, `getRank`, `isStore`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `MemRefAccess`, `getRank`, `isStore` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 97-102
```cpp
  /// Creates an access relation for the access. An access relation maps
  /// elements of an iteration domain to the element(s) of an array domain
  /// accessed by that iteration of the associated statement through some array
  /// reference. For example, given the MLIR code:
  ///
  /// affine.for %i0 = 0 to 10 {
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 103-108
```cpp
  ///   affine.for %i1 = 0 to 10 {
  ///     %a = affine.load %arr[%i0 + %i1, %i0 + 2 * %i1] : memref<100x100xf32>
  ///   }
  /// }
  ///
  /// The access relation, assuming that the memory locations for %arr are
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 109-114
```cpp
  /// represented as %m0, %m1 would be:
  ///
  ///   (%i0, %i1) -> (%m0, %m1)
  ///   %m0 = %i0 + %i1
  ///   %m1 = %i0 + 2 * %i1
  ///   0  <= %i0 < 10
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 115-120
```cpp
  ///   0  <= %i1 < 10
  ///
  /// Returns failure for yet unimplemented/unsupported cases (see docs of
  /// mlir::getIndexSet and mlir::getRelationFromMap for these cases).
  LogicalResult getAccessRelation(presburger::IntegerRelation &accessRel) const;

```
- **EN**: Declares APIs or declarative rules around `getAccessRelation`.
- **CN**: 声明与 `getAccessRelation` 相关的 API 或声明式规则。

### Lines 121-126
```cpp
  /// Populates 'accessMap' with composition of AffineApplyOps reachable from
  /// 'indices'.
  void getAccessMap(AffineValueMap *accessMap) const;

  /// Equal if both affine accesses can be proved to be equivalent at compile
  /// time (considering the memrefs, their respective affine access maps  and
```
- **EN**: Declares APIs or declarative rules around `getAccessMap`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `getAccessMap` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 127-133
```cpp
  /// operands). The equality of access functions + operands is checked by
  /// subtracting fully composed value maps, and then simplifying the difference
  /// using the expression flattener. This does not account for aliasing of
  /// memrefs.
  bool operator==(const MemRefAccess &rhs) const;
  bool operator!=(const MemRefAccess &rhs) const { return !(*this == rhs); }

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 134-143
```cpp
  explicit operator bool() const { return !!memref; }
};

// DependenceComponent contains state about the direction of a dependence as an
// interval [lb, ub] for an AffineForOp.
// Distance vectors components are represented by the interval [lb, ub] with
// lb == ub.
// Direction vectors components are represented by the interval [lb, ub] with
// lb < ub. Note that ub/lb == None means unbounded.
struct DependenceComponent {
```
- **EN**: Introduces declarations for `DependenceComponent`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `DependenceComponent` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 144-152
```cpp
  // The AffineForOp Operation associated with this dependence component.
  Operation *op = nullptr;
  // The lower bound of the dependence distance.
  std::optional<int64_t> lb;
  // The upper bound of the dependence distance (inclusive).
  std::optional<int64_t> ub;
  DependenceComponent() : lb(std::nullopt), ub(std::nullopt) {}
};

```
- **EN**: Implements logic around `DependenceComponent`.
- **CN**: 围绕 `DependenceComponent` 实现具体逻辑。

### Lines 153-158
```cpp
/// Checks whether two accesses to the same memref access the same element.
/// Each access is specified using the MemRefAccess structure, which contains
/// the operation, indices and memref associated with the access. Returns
/// 'NoDependence' if it can be determined conclusively that the accesses do not
/// access the same memref element. If 'allowRAR' is true, will consider
/// read-after-read dependences (typically used by applications trying to
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 159-164
```cpp
/// optimize input reuse).
// TODO: Wrap 'dependenceConstraints' and 'dependenceComponents' into a single
// struct.
// TODO: Make 'dependenceConstraints' optional arg.
struct DependenceResult {
  enum ResultEnum {
```
- **EN**: Introduces declarations for `DependenceResult`, `ResultEnum`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `DependenceResult`, `ResultEnum` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 165-171
```cpp
    HasDependence, // A dependence exists between 'srcAccess' and 'dstAccess'.
    NoDependence,  // No dependence exists between 'srcAccess' and 'dstAccess'.
    Failure,       // Dependence check failed due to unsupported cases.
  } value;
  DependenceResult(ResultEnum v) : value(v) {}
};

```
- **EN**: Implements logic around `DependenceResult`.
- **CN**: 围绕 `DependenceResult` 实现具体逻辑。

### Lines 172-178
```cpp
DependenceResult checkMemrefAccessDependence(
    const MemRefAccess &srcAccess, const MemRefAccess &dstAccess,
    unsigned loopDepth,
    FlatAffineValueConstraints *dependenceConstraints = nullptr,
    SmallVector<DependenceComponent, 2> *dependenceComponents = nullptr,
    bool allowRAR = false);

```
- **EN**: Declares APIs or declarative rules around `checkMemrefAccessDependence`.
- **CN**: 声明与 `checkMemrefAccessDependence` 相关的 API 或声明式规则。

### Lines 179-184
```cpp
/// Utility function that returns true if the provided DependenceResult
/// corresponds to a dependence result.
inline bool hasDependence(DependenceResult result) {
  return result.value == DependenceResult::HasDependence;
}

```
- **EN**: Implements logic around `hasDependence`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `hasDependence` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 185-190
```cpp
/// Returns true if the provided DependenceResult corresponds to the absence of
/// a dependence.
inline bool noDependence(DependenceResult result) {
  return result.value == DependenceResult::NoDependence;
}

```
- **EN**: Implements logic around `noDependence`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `noDependence` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 191-197
```cpp
/// Returns in 'depCompsVec', dependence components for dependences between all
/// load and store ops in loop nest rooted at 'forOp', at loop depths in range
/// [1, maxLoopDepth].
void getDependenceComponents(
    AffineForOp forOp, unsigned maxLoopDepth,
    std::vector<SmallVector<DependenceComponent, 2>> *depCompsVec);

```
- **EN**: Declares APIs or declarative rules around `getDependenceComponents`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `getDependenceComponents` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 198-201
```cpp
} // namespace affine
} // namespace mlir

#endif // MLIR_DIALECT_AFFINE_ANALYSIS_AFFINEANALYSIS_H
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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Analysis/Presburger/IntegerRelation.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/IR/Value.h`, `llvm/ADT/SmallVector.h`
- **Standard-library headers / 标准库头文件**: `<optional>`
- **Subsystem categories / 子系统类别**: dialect-specific operations, attributes, types, transforms, or interface declarations / 方言相关的操作、属性、类型、变换或接口声明 (1), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (1), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (1)
