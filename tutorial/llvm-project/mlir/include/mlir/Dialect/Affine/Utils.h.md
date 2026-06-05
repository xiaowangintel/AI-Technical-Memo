# Utils.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/Affine/Utils.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This header file declares a set of utilities for the affine dialect ops.
  - **CN**: 该文件位于 `mlir/include/mlir/Dialect/Affine`，围绕 Affine 方言公开 `Utils` 相关的接口、规则或生成式定义。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===- Utils.h - Affine dialect utilities -----------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This header file declares a set of utilities for the affine dialect ops.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 12-21
```cpp

#ifndef MLIR_DIALECT_AFFINE_UTILS_H
#define MLIR_DIALECT_AFFINE_UTILS_H

#include "mlir/Analysis/AliasAnalysis.h"
#include "mlir/Dialect/Affine/Analysis/AffineAnalysis.h"
#include "mlir/Dialect/Affine/IR/AffineOps.h"
#include "mlir/IR/OpDefinition.h"
#include <optional>

```
- **EN**: Pulls in the headers needed by this declaration unit, including `mlir/Analysis/AliasAnalysis.h`, `mlir/Dialect/Affine/Analysis/AffineAnalysis.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/IR/OpDefinition.h`.
- **CN**: 引入该声明单元所需的头文件，其中包括 `mlir/Analysis/AliasAnalysis.h`, `mlir/Dialect/Affine/Analysis/AffineAnalysis.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/IR/OpDefinition.h`。

### Lines 22-29
```cpp
namespace mlir {
class DominanceInfo;
class Operation;
class PostDominanceInfo;
class ImplicitLocOpBuilder;

namespace func {
class FuncOp;
```
- **EN**: Introduces declarations for `mlir`, `DominanceInfo`, `Operation`, `PostDominanceInfo`, and 3 more symbols, establishing the public symbols or declarative records used later.
- **CN**: 引入 `mlir`, `DominanceInfo`, `Operation`, `PostDominanceInfo`, and 3 more symbols 等声明，建立后续使用的公共符号或声明式记录。

### Lines 30-37
```cpp
} // namespace func

namespace memref {
class AllocOp;
class AllocaOp;
class ReinterpretCastOp;
} // namespace memref

```
- **EN**: Introduces declarations for `func`, `memref`, `AllocOp`, `AllocaOp`, and 1 more symbols, establishing the public symbols or declarative records used later.
- **CN**: 引入 `func`, `memref`, `AllocOp`, `AllocaOp`, and 1 more symbols 等声明，建立后续使用的公共符号或声明式记录。

### Lines 38-45
```cpp
namespace affine {
class AffineForOp;
class AffineIfOp;
class AffineParallelOp;

using ReductionLoopMap = DenseMap<Operation *, SmallVector<LoopReduction, 2>>;

/// Replaces a parallel affine.for op with a 1-d affine.parallel op. `forOp`'s
```
- **EN**: Introduces declarations for `affine`, `AffineForOp`, `AffineIfOp`, `AffineParallelOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `affine`, `AffineForOp`, `AffineIfOp`, `AffineParallelOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 46-55
```cpp
/// body is taken by the affine.parallel op and the former is erased.
/// (mlir::isLoopParallel can be used to detect a parallel affine.for op.) The
/// reductions specified in `parallelReductions` are also parallelized.
/// Parallelization will fail in the presence of loop iteration arguments that
/// are not listed in `parallelReductions`. `resOp` if non-null is set to the
/// newly created affine.parallel op.
LogicalResult affineParallelize(AffineForOp forOp,
                                ArrayRef<LoopReduction> parallelReductions = {},
                                AffineParallelOp *resOp = nullptr);

```
- **EN**: Implements logic around `affineParallelize`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `affineParallelize` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 56-63
```cpp
/// Hoists out affine.if/else to as high as possible, i.e., past all invariant
/// affine.fors/parallel's. Returns success if any hoisting happened; folded` is
/// set to true if the op was folded or erased. This hoisting could lead to
/// significant code expansion in some cases.
LogicalResult hoistAffineIfOp(AffineIfOp ifOp, bool *folded = nullptr);

/// Holds parameters to perform n-D vectorization on a single loop nest.
/// For example, for the following loop nest:
```
- **EN**: Declares APIs or declarative rules around `hoistAffineIfOp`; this block checks structural or semantic invariants; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `hoistAffineIfOp` 相关的 API 或声明式规则；该代码块检查结构或语义不变式，并协调核心结构化 MLIR 方言之间的行为。

### Lines 64-71
```cpp
///
/// func @vec2d(%in: memref<64x128x512xf32>, %out: memref<64x128x512xf32>) {
///   affine.for %i0 = 0 to 64 {
///     affine.for %i1 = 0 to 128 {
///       affine.for %i2 = 0 to 512 {
///         %ld = affine.load %in[%i0, %i1, %i2] : memref<64x128x512xf32>
///         affine.store %ld, %out[%i0, %i1, %i2] : memref<64x128x512xf32>
///       }
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 72-79
```cpp
///     }
///   }
///   return
/// }
///
/// and VectorizationStrategy = 'vectorSizes = {8, 4}', 'loopToVectorDim =
/// {{i1->0}, {i2->1}}', SuperVectorizer will generate:
///
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 80-87
```cpp
///  func @vec2d(%arg0: memref<64x128x512xf32>, %arg1: memref<64x128x512xf32>) {
///    affine.for %arg2 = 0 to 64 {
///      affine.for %arg3 = 0 to 128 step 8 {
///        affine.for %arg4 = 0 to 512 step 4 {
///          %cst = arith.constant 0.000000e+00 : f32
///          %0 = vector.transfer_read %arg0[%arg2, %arg3, %arg4], %cst : ...
///          vector.transfer_write %0, %arg1[%arg2, %arg3, %arg4] : ...
///        }
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 88-103
```cpp
///      }
///    }
///    return
///  }
// TODO: Hoist to a VectorizationStrategy.cpp when appropriate.
struct VectorizationStrategy {
  // Vectorization factors to apply to each target vector dimension.
  // Each factor will be applied to a different loop.
  SmallVector<int64_t, 8> vectorSizes;
  // Maps each AffineForOp vectorization candidate with its vector dimension.
  // The candidate will be vectorized using the vectorization factor in
  // 'vectorSizes' for that dimension.
  DenseMap<Operation *, unsigned> loopToVectorDim;
  // Maps loops that implement vectorizable reductions to the corresponding
  // reduction descriptors.
  ReductionLoopMap reductionLoops;
```
- **EN**: Introduces declarations for `VectorizationStrategy`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `VectorizationStrategy` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 104-111
```cpp
};

/// Vectorize affine loops that are children of parentOp (including itself)
void vectorizeChildAffineLoops(Operation *parentOp, bool vectorizeReductions,
                               ArrayRef<int64_t> vectorSizes,
                               ArrayRef<int64_t> fastestVaryingPattern);

/// Replace affine store and load accesses by scalars by forwarding stores to
```
- **EN**: Declares APIs or declarative rules around `vectorizeChildAffineLoops`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `vectorizeChildAffineLoops` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 112-119
```cpp
/// loads and eliminate invariant affine loads; consequently, eliminate dead
/// allocs.
void affineScalarReplace(func::FuncOp f, DominanceInfo &domInfo,
                         PostDominanceInfo &postDomInfo,
                         AliasAnalysis &analysis);

/// Vectorizes affine loops in 'loops' using the n-D vectorization factors in
/// 'vectorSizes'. By default, each vectorization factor is applied
```
- **EN**: Declares APIs or declarative rules around `affineScalarReplace`; this block checks structural or semantic invariants; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `affineScalarReplace` 相关的 API 或声明式规则；该代码块检查结构或语义不变式，并协调核心结构化 MLIR 方言之间的行为。

### Lines 120-130
```cpp
/// inner-to-outer to the loops of each loop nest. 'fastestVaryingPattern' can
/// be optionally used to provide a different loop vectorization order.
/// If `reductionLoops` is not empty, the given reduction loops may be
/// vectorized along the reduction dimension.
/// TODO: Vectorizing reductions is supported only for 1-D vectorization.
void vectorizeAffineLoops(
    Operation *parentOp,
    llvm::DenseSet<Operation *, DenseMapInfo<Operation *>> &loops,
    ArrayRef<int64_t> vectorSizes, ArrayRef<int64_t> fastestVaryingPattern,
    const ReductionLoopMap &reductionLoops = ReductionLoopMap());

```
- **EN**: Declares APIs or declarative rules around `vectorizeAffineLoops`, `ReductionLoopMap`.
- **CN**: 声明与 `vectorizeAffineLoops`, `ReductionLoopMap` 相关的 API 或声明式规则。

### Lines 131-138
```cpp
/// External utility to vectorize affine loops from a single loop nest using an
/// n-D vectorization strategy (see doc in VectorizationStrategy definition).
/// Loops are provided in a 2D vector container. The first dimension represents
/// the nesting level relative to the loops to be vectorized. The second
/// dimension contains the loops. This means that:
///   a) every loop in 'loops[i]' must have a parent loop in 'loops[i-1]',
///   b) a loop in 'loops[i]' may or may not have a child loop in 'loops[i+1]'.
///
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 139-146
```cpp
/// For example, for the following loop nest:
///
///   func @vec2d(%in0: memref<64x128x512xf32>, %in1: memref<64x128x128xf32>,
///               %out0: memref<64x128x512xf32>,
///               %out1: memref<64x128x128xf32>) {
///     affine.for %i0 = 0 to 64 {
///       affine.for %i1 = 0 to 128 {
///         affine.for %i2 = 0 to 512 {
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 147-154
```cpp
///           %ld = affine.load %in0[%i0, %i1, %i2] : memref<64x128x512xf32>
///           affine.store %ld, %out0[%i0, %i1, %i2] : memref<64x128x512xf32>
///         }
///         affine.for %i3 = 0 to 128 {
///           %ld = affine.load %in1[%i0, %i1, %i3] : memref<64x128x128xf32>
///           affine.store %ld, %out1[%i0, %i1, %i3] : memref<64x128x128xf32>
///         }
///       }
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 155-162
```cpp
///     }
///     return
///   }
///
/// loops = {{%i0}, {%i2, %i3}}, to vectorize the outermost and the two
/// innermost loops;
/// loops = {{%i1}, {%i2, %i3}}, to vectorize the middle and the two innermost
/// loops;
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 163-170
```cpp
/// loops = {{%i2}}, to vectorize only the first innermost loop;
/// loops = {{%i3}}, to vectorize only the second innermost loop;
/// loops = {{%i1}}, to vectorize only the middle loop.
LogicalResult
vectorizeAffineLoopNest(std::vector<SmallVector<AffineForOp, 2>> &loops,
                        const VectorizationStrategy &strategy);

/// Normalize a affine.parallel op so that lower bounds are 0 and steps are 1.
```
- **EN**: Implements logic around `vectorizeAffineLoopNest`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `vectorizeAffineLoopNest` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 171-178
```cpp
/// As currently implemented, this transformation cannot fail and will return
/// early if the op is already in a normalized form.
void normalizeAffineParallel(AffineParallelOp op);

/// Normalize an affine.for op. An affine.for op is normalized by converting the
/// lower bound to zero and loop step to one. The upper bound is set to the trip
/// count of the loop. Original loops must have a lower bound with only a single
/// result. There is no such restriction on upper bounds. Returns success if the
```
- **EN**: Declares APIs or declarative rules around `normalizeAffineParallel`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `normalizeAffineParallel` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 179-186
```cpp
/// loop has been normalized (or is already in the normal form). If
/// `promoteSingleIter` is true, the loop is simply promoted if it has a single
/// iteration.
LogicalResult normalizeAffineFor(AffineForOp op,
                                 bool promoteSingleIter = false);

/// Traverse `e` and return an AffineExpr where all occurrences of `dim` have
/// been replaced by either:
```
- **EN**: Declares APIs or declarative rules around `normalizeAffineFor`.
- **CN**: 声明与 `normalizeAffineFor` 相关的 API 或声明式规则。

### Lines 187-194
```cpp
///  - `min` if `positivePath` is true when we reach an occurrence of `dim`
///  - `max` if `positivePath` is true when we reach an occurrence of `dim`
/// `positivePath` is negated each time we hit a multiplicative or divisive
/// binary op with a constant negative coefficient.
AffineExpr substWithMin(AffineExpr e, AffineExpr dim, AffineExpr min,
                        AffineExpr max, bool positivePath = true);

/// Replaces all "dereferencing" uses of `oldMemRef` with `newMemRef` while
```
- **EN**: Declares APIs or declarative rules around `substWithMin`.
- **CN**: 声明与 `substWithMin` 相关的 API 或声明式规则。

### Lines 195-202
```cpp
/// optionally remapping the old memref's indices using the supplied affine map,
/// `indexRemap`. The new memref could be of a different shape or rank.
/// `extraIndices` provides any additional access indices to be added to the
/// start.
///
/// `indexRemap` remaps indices of the old memref access to a new set of indices
/// that are used to index the memref. Additional input operands to indexRemap
/// can be optionally provided in `extraOperands`, and they occupy the start
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 203-210
```cpp
/// of its input list. `indexRemap`'s dimensional inputs are expected to
/// correspond to memref's indices, and its symbolic inputs if any should be
/// provided in `symbolOperands`.
//
/// If `userFilterFn` is specified, restrict replacement to only those users
/// that pass the specified filter (i.e., the filter returns true).
///
/// 'allowNonDereferencingOps', if set, allows replacement of non-dereferencing
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 211-218
```cpp
/// uses of a memref without any requirement for access index rewrites as long
/// as the user operation has the MemRefsNormalizable trait. The default value
/// of this flag is false.
///
/// 'replaceInDeallocOp', if set, lets DeallocOp, a non-dereferencing user, to
/// also be a candidate for replacement. The default value of this flag is
/// false.
///
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 219-234
```cpp
/// Returns true on success and false if the replacement is not possible,
/// whenever a memref is used as an operand in a non-dereferencing context and
/// 'allowNonDereferencingOps' is false, except for dealloc's on the memref
/// which are left untouched. See comments at function definition for an
/// example.
//
//  Ex: to replace load %A[%i, %j] with load %Abuf[%t mod 2, %ii - %i, %j]:
//  The SSA value corresponding to '%t mod 2' should be in 'extraIndices', and
//  index remap will perform (%i, %j) -> (%ii - %i, %j), i.e., indexRemap = (d0,
//  d1, d2) -> (d0 - d1, d2), and %ii will be the extra operand. Without any
//  extra operands, note that 'indexRemap' would just be applied to existing
//  indices (%i, %j).
//
//  TODO: allow extraIndices to be added at any position.
LogicalResult replaceAllMemRefUsesWith(
    Value oldMemRef, Value newMemRef, ArrayRef<Value> extraIndices = {},
```
- **EN**: Implements logic around `replaceAllMemRefUsesWith`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `replaceAllMemRefUsesWith` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 235-242
```cpp
    AffineMap indexRemap = AffineMap(), ArrayRef<Value> extraOperands = {},
    ArrayRef<Value> symbolOperands = {},
    llvm::function_ref<bool(Operation *)> userFilterFn = nullptr,
    bool allowNonDereferencingOps = false, bool replaceInDeallocOp = false);

/// Performs the same replacement as the other version above but only for the
/// dereferencing uses of `oldMemRef` in `op`, except in cases where
/// 'allowNonDereferencingOps' is set to true where we replace the
```
- **EN**: Implements logic around `AffineMap`, `function_ref`.
- **CN**: 围绕 `AffineMap`, `function_ref` 实现具体逻辑。

### Lines 243-251
```cpp
/// non-dereferencing uses as well.
LogicalResult replaceAllMemRefUsesWith(Value oldMemRef, Value newMemRef,
                                       Operation *op,
                                       ArrayRef<Value> extraIndices = {},
                                       AffineMap indexRemap = AffineMap(),
                                       ArrayRef<Value> extraOperands = {},
                                       ArrayRef<Value> symbolOperands = {},
                                       bool allowNonDereferencingOps = false);

```
- **EN**: Implements logic around `replaceAllMemRefUsesWith`, `AffineMap`.
- **CN**: 围绕 `replaceAllMemRefUsesWith`, `AffineMap` 实现具体逻辑。

### Lines 252-262
```cpp
/// Rewrites the memref defined by alloc or reinterpret_cast op to have an
/// identity layout map and updates all its indexing uses. Returns failure if
/// any of its uses escape (while leaving the IR in a valid state).
template <typename AllocLikeOp>
LogicalResult normalizeMemRef(AllocLikeOp op);
extern template LogicalResult
normalizeMemRef<memref::AllocaOp>(memref::AllocaOp op);
extern template LogicalResult
normalizeMemRef<memref::AllocOp>(memref::AllocOp op);
LogicalResult normalizeMemRef(memref::ReinterpretCastOp op);

```
- **EN**: Declares APIs or declarative rules around `normalizeMemRef`, `AllocaOp>`, `AllocOp>`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `normalizeMemRef`, `AllocaOp>`, `AllocOp>` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 263-270
```cpp
/// Normalizes `memrefType` so that the affine layout map of the memref is
/// transformed to an identity map with a new shape being computed for the
/// normalized memref type and returns it. The old memref type is simplify
/// returned if the normalization failed.
MemRefType normalizeMemRefType(MemRefType memrefType);

/// Given an operation, inserts one or more single result affine apply
/// operations, results of which are exclusively used by this operation.
```
- **EN**: Declares APIs or declarative rules around `normalizeMemRefType`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `normalizeMemRefType` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 271-278
```cpp
/// The operands of these newly created affine apply ops are
/// guaranteed to be loop iterators or terminal symbols of a function.
///
/// Before
///
/// affine.for %i = 0 to #map(%N)
///   %idx = affine.apply (d0) -> (d0 mod 2) (%i)
///   send %A[%idx], ...
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 279-286
```cpp
///   %v = "compute"(%idx, ...)
///
/// After
///
/// affine.for %i = 0 to #map(%N)
///   %idx = affine.apply (d0) -> (d0 mod 2) (%i)
///   send %A[%idx], ...
///   %idx_ = affine.apply (d0) -> (d0 mod 2) (%i)
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 287-294
```cpp
///   %v = "compute"(%idx_, ...)

/// This allows the application of different transformations on send and
/// compute (for eg. different shifts/delays)
///
/// Fills `sliceOps` with the list of affine.apply operations.
/// In the following cases, `sliceOps` remains empty:
///   1. If none of opInst's operands were the result of an affine.apply
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 295-302
```cpp
///      (i.e., there was no affine computation slice to create).
///   2. If all the affine.apply op's supplying operands to this opInst did not
///      have any uses other than those in this opInst.
void createAffineComputationSlice(Operation *opInst,
                                  SmallVectorImpl<AffineApplyOp> *sliceOps);

/// Emit code that computes the given affine expression using standard
/// arithmetic operations applied to the provided dimension and symbol values.
```
- **EN**: Declares APIs or declarative rules around `createAffineComputationSlice`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `createAffineComputationSlice` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 303-312
```cpp
Value expandAffineExpr(OpBuilder &builder, Location loc, AffineExpr expr,
                       ValueRange dimValues, ValueRange symbolValues);

/// Create a sequence of operations that implement the `affineMap` applied to
/// the given `operands` (as it it were an AffineApplyOp).
std::optional<SmallVector<Value, 8>> expandAffineMap(OpBuilder &builder,
                                                     Location loc,
                                                     AffineMap affineMap,
                                                     ValueRange operands);

```
- **EN**: Declares APIs or declarative rules around `expandAffineExpr`, `expandAffineMap`.
- **CN**: 声明与 `expandAffineExpr`, `expandAffineMap` 相关的 API 或声明式规则。

### Lines 313-321
```cpp
/// Holds the result of (div a, b)  and (mod a, b).
struct DivModValue {
  Value quotient;
  Value remainder;
};

/// Create IR to calculate (div lhs, rhs) and (mod lhs, rhs).
DivModValue getDivMod(OpBuilder &b, Location loc, Value lhs, Value rhs);

```
- **EN**: Introduces declarations for `DivModValue`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `DivModValue` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 322-331
```cpp
/// Generate the IR to delinearize `linearIndex` given the `basis` and return
/// the multi-index. `hasOuterBound` indicates whether `basis` has an entry
/// given the size of the first multi-index result - if it is true, the function
/// will return `basis.size()` values, otherwise, it will return `basis.size() +
/// 1`.
FailureOr<SmallVector<Value>> delinearizeIndex(OpBuilder &b, Location loc,
                                               Value linearIndex,
                                               ArrayRef<Value> basis,
                                               bool hasOuterBound = true);

```
- **EN**: Declares APIs or declarative rules around `delinearizeIndex`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `delinearizeIndex` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 332-343
```cpp
FailureOr<SmallVector<Value>> delinearizeIndex(OpBuilder &b, Location loc,
                                               Value linearIndex,
                                               ArrayRef<OpFoldResult> basis,
                                               bool hasOuterBound = true);

// Generate IR that extracts the linear index from a multi-index according to
// a basis/shape. The basis may contain either `multiIndex.size()` or
// `multiIndex.size() - 1` elements.
OpFoldResult linearizeIndex(ArrayRef<OpFoldResult> multiIndex,
                            ArrayRef<OpFoldResult> basis,
                            ImplicitLocOpBuilder &builder);

```
- **EN**: Declares APIs or declarative rules around `delinearizeIndex`, `linearizeIndex`.
- **CN**: 声明与 `delinearizeIndex`, `linearizeIndex` 相关的 API 或声明式规则。

### Lines 344-351
```cpp
OpFoldResult linearizeIndex(OpBuilder &builder, Location loc,
                            ArrayRef<OpFoldResult> multiIndex,
                            ArrayRef<OpFoldResult> basis);

/// Ensure that all operations that could be executed after `start`
/// (noninclusive) and prior to `memOp` (e.g. on a control flow/op path
/// between the operations) do not have the potential memory effect
/// `EffectType` on `memOp`. `memOp`  is an operation that reads or writes to
```
- **EN**: Declares APIs or declarative rules around `linearizeIndex`.
- **CN**: 声明与 `linearizeIndex` 相关的 API 或声明式规则。

### Lines 352-359
```cpp
/// a memref. For example, if `EffectType` is MemoryEffects::Write, this method
/// will check if there is no write to the memory between `start` and `memOp`
/// that would change the read within `memOp`.
template <typename EffectType, typename T>
bool hasNoInterveningEffect(Operation *start, T memOp,
                            llvm::function_ref<bool(Value, Value)> mayAlias);

struct AffineValueExpr {
```
- **EN**: Introduces declarations for `AffineValueExpr`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineValueExpr` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 360-374
```cpp
  explicit AffineValueExpr(AffineExpr e) : e(e) {}
  AffineValueExpr bind(Value v) {
    this->v = v;
    return *this;
  }
  AffineValueExpr bind(OpFoldResult v) {
    this->v = v;
    return *this;
  }
  operator AffineExpr() const { return e; }
  operator OpFoldResult() const { return v; }
  AffineExpr e;
  OpFoldResult v;
};

```
- **EN**: Implements logic around `AffineValueExpr`, `bind`, `AffineExpr`, `OpFoldResult`.
- **CN**: 围绕 `AffineValueExpr`, `bind`, `AffineExpr`, `OpFoldResult` 实现具体逻辑。

### Lines 375-390
```cpp
/// Helper struct to build simple AffineValueExprs with minimal type inference
/// support.
struct AffineBuilder {
  AffineBuilder(OpBuilder &b, Location loc) : b(b), loc(loc) {}
  OpFoldResult add(AffineValueExpr lhs, AffineValueExpr rhs) {
    return makeComposedFoldedAffineApply(b, loc, {lhs.e + rhs.e}, {lhs, rhs});
  }
  OpFoldResult sub(AffineValueExpr lhs, AffineValueExpr rhs) {
    return makeComposedFoldedAffineApply(b, loc, {lhs.e - rhs.e}, {lhs, rhs});
  }
  OpFoldResult mul(AffineValueExpr lhs, AffineValueExpr rhs) {
    return makeComposedFoldedAffineApply(b, loc, {lhs.e * rhs.e}, {lhs, rhs});
  }
  OpFoldResult floor(AffineValueExpr lhs, AffineValueExpr rhs) {
    return makeComposedFoldedAffineApply(b, loc, {lhs.e.floorDiv(rhs.e)},
                                         {lhs, rhs});
```
- **EN**: Introduces declarations for `to`, `AffineBuilder`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `to`, `AffineBuilder` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 391-406
```cpp
  }
  OpFoldResult ceil(AffineValueExpr lhs, AffineValueExpr rhs) {
    return makeComposedFoldedAffineApply(b, loc, {lhs.e.ceilDiv(rhs.e)},
                                         {lhs, rhs});
  }
  OpFoldResult min(ArrayRef<OpFoldResult> vals) {
    return makeComposedFoldedAffineMin(
        b, loc, AffineMap::getMultiDimIdentityMap(vals.size(), b.getContext()),
        vals);
  }
  OpFoldResult max(ArrayRef<OpFoldResult> vals) {
    return makeComposedFoldedAffineMax(
        b, loc, AffineMap::getMultiDimIdentityMap(vals.size(), b.getContext()),
        vals);
  }

```
- **EN**: Implements logic around `ceil`, `makeComposedFoldedAffineApply`, `min`, `makeComposedFoldedAffineMin`, and 3 more symbols.
- **CN**: 围绕 `ceil`, `makeComposedFoldedAffineApply`, `min`, `makeComposedFoldedAffineMin`, and 3 more symbols 实现具体逻辑。

### Lines 407-414
```cpp
private:
  OpBuilder &b;
  Location loc;
};

} // namespace affine
} // namespace mlir

```
- **EN**: Introduces declarations for `affine`, `mlir`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `affine`, `mlir` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 415-415
```cpp
#endif // MLIR_DIALECT_AFFINE_UTILS_H
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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Analysis/AliasAnalysis.h`, `mlir/Dialect/Affine/Analysis/AffineAnalysis.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/IR/OpDefinition.h`
- **Standard-library headers / 标准库头文件**: `<optional>`
- **Subsystem categories / 子系统类别**: dialect-specific operations, attributes, types, transforms, or interface declarations / 方言相关的操作、属性、类型、变换或接口声明 (2), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (1)
