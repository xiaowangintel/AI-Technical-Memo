# LoopUtils.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/Affine/LoopUtils.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This header file defines prototypes for various loop transformation utility methods: these are not passes by themselves but are used either by passes, optimization sequences, or in turn by other transformation utilities.
  - **CN**: 该文件位于 `mlir/include/mlir/Dialect/Affine`，围绕 Affine 方言公开 `LoopUtils` 相关的接口、规则或生成式定义。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- LoopUtils.h - Loop transformation utilities --------------*- C++ -*-===//
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
// This header file defines prototypes for various loop transformation utility
// methods: these are not passes by themselves but are used either by passes,
// optimization sequences, or in turn by other transformation utilities.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 14-22
```cpp

#ifndef MLIR_DIALECT_AFFINE_LOOPUTILS_H
#define MLIR_DIALECT_AFFINE_LOOPUTILS_H

#include "mlir/IR/Block.h"
#include "mlir/Support/LLVM.h"
#include "mlir/Transforms/RegionUtils.h"
#include <optional>

```
- **EN**: Pulls in the headers needed by this declaration unit, including `mlir/IR/Block.h`, `mlir/Support/LLVM.h`, `mlir/Transforms/RegionUtils.h`, `optional`.
- **CN**: 引入该声明单元所需的头文件，其中包括 `mlir/IR/Block.h`, `mlir/Support/LLVM.h`, `mlir/Transforms/RegionUtils.h`, `optional`。

### Lines 23-28
```cpp
namespace mlir {
class AffineMap;
class LoopLikeOpInterface;
class OpBuilder;
class Value;
class ValueRange;
```
- **EN**: Introduces declarations for `mlir`, `AffineMap`, `LoopLikeOpInterface`, `OpBuilder`, and 2 more symbols, establishing the public symbols or declarative records used later.
- **CN**: 引入 `mlir`, `AffineMap`, `LoopLikeOpInterface`, `OpBuilder`, and 2 more symbols 等声明，建立后续使用的公共符号或声明式记录。

### Lines 29-34
```cpp

namespace func {
class FuncOp;
} // namespace func

namespace scf {
```
- **EN**: Introduces declarations for `func`, `FuncOp`, `scf`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `func`, `FuncOp`, `scf` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 35-40
```cpp
class ForOp;
class ParallelOp;
} // namespace scf

namespace affine {
class AffineForOp;
```
- **EN**: Introduces declarations for `ForOp`, `ParallelOp`, `scf`, `affine`, and 1 more symbols, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ForOp`, `ParallelOp`, `scf`, `affine`, and 1 more symbols 等声明，建立后续使用的公共符号或声明式记录。

### Lines 41-46
```cpp
struct MemRefRegion;

/// Unrolls this for operation completely if the trip count is known to be
/// constant. Returns failure otherwise.
LogicalResult loopUnrollFull(AffineForOp forOp);

```
- **EN**: Introduces declarations for `MemRefRegion`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `MemRefRegion` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 47-52
```cpp
/// Unrolls this for operation by the specified unroll factor. Returns failure
/// if the loop cannot be unrolled either due to restrictions or due to invalid
/// unroll factors. Requires positive loop bounds and step. If specified,
/// annotates the Ops in each unrolled iteration by applying `annotateFn`.
/// When `cleanUpUnroll` is true, we can ensure the cleanup loop is unrolled
/// regardless of the unroll factor.
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 53-58
```cpp
LogicalResult loopUnrollByFactor(
    AffineForOp forOp, uint64_t unrollFactor,
    function_ref<void(unsigned, Operation *, OpBuilder)> annotateFn = nullptr,
    bool cleanUpUnroll = false);

/// Unrolls this loop by the specified unroll factor or its trip count,
```
- **EN**: Declares APIs or declarative rules around `loopUnrollByFactor`, `function_ref`.
- **CN**: 声明与 `loopUnrollByFactor`, `function_ref` 相关的 API 或声明式规则。

### Lines 59-65
```cpp
/// whichever is lower.
LogicalResult loopUnrollUpToFactor(AffineForOp forOp, uint64_t unrollFactor);

/// Returns true if `loops` is a perfectly nested loop nest, where loops appear
/// in it from outermost to innermost.
[[maybe_unused]] bool isPerfectlyNested(ArrayRef<AffineForOp> loops);

```
- **EN**: Declares APIs or declarative rules around `loopUnrollUpToFactor`, `isPerfectlyNested`.
- **CN**: 声明与 `loopUnrollUpToFactor`, `isPerfectlyNested` 相关的 API 或声明式规则。

### Lines 66-72
```cpp
/// Get perfectly nested sequence of loops starting at root of loop nest
/// (the first op being another AffineFor, and the second op - a terminator).
/// A loop is perfectly nested iff: the first op in the loop's body is another
/// AffineForOp, and the second op is a terminator).
void getPerfectlyNestedLoops(SmallVectorImpl<AffineForOp> &nestedLoops,
                             AffineForOp root);

```
- **EN**: Declares APIs or declarative rules around `getPerfectlyNestedLoops`.
- **CN**: 声明与 `getPerfectlyNestedLoops` 相关的 API 或声明式规则。

### Lines 73-79
```cpp
/// Unrolls and jams this loop by the specified factor. `forOp` can be a loop
/// with iteration arguments performing supported reductions and its inner loops
/// can have iteration arguments. Returns success if the loop is successfully
/// unroll-jammed.
LogicalResult loopUnrollJamByFactor(AffineForOp forOp,
                                    uint64_t unrollJamFactor);

```
- **EN**: Declares APIs or declarative rules around `loopUnrollJamByFactor`.
- **CN**: 声明与 `loopUnrollJamByFactor` 相关的 API 或声明式规则。

### Lines 80-85
```cpp
/// Unrolls and jams this loop by the specified factor or by the trip count (if
/// constant), whichever is lower.
LogicalResult loopUnrollJamUpToFactor(AffineForOp forOp,
                                      uint64_t unrollJamFactor);

/// Promotes the loop body of a AffineForOp to its containing block if the loop
```
- **EN**: Declares APIs or declarative rules around `loopUnrollJamUpToFactor`.
- **CN**: 声明与 `loopUnrollJamUpToFactor` 相关的 API 或声明式规则。

### Lines 86-92
```cpp
/// was known to have a single iteration.
LogicalResult promoteIfSingleIteration(AffineForOp forOp);

/// Promotes all single iteration AffineForOp's in the Function, i.e., moves
/// their body into the containing Block.
void promoteSingleIterationLoops(func::FuncOp f);

```
- **EN**: Declares APIs or declarative rules around `promoteIfSingleIteration`, `promoteSingleIterationLoops`.
- **CN**: 声明与 `promoteIfSingleIteration`, `promoteSingleIterationLoops` 相关的 API 或声明式规则。

### Lines 93-100
```cpp
/// Skew the operations in an affine.for's body with the specified
/// operation-wise shifts. The shifts are with respect to the original execution
/// order, and are multiplied by the loop 'step' before being applied. If
/// `unrollPrologueEpilogue` is set, fully unroll the prologue and epilogue
/// loops when possible.
LogicalResult affineForOpBodySkew(AffineForOp forOp, ArrayRef<uint64_t> shifts,
                                  bool unrollPrologueEpilogue = false);

```
- **EN**: Declares APIs or declarative rules around `affineForOpBodySkew`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `affineForOpBodySkew` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 101-108
```cpp
/// Tiles the specified band of perfectly nested loops creating tile-space loops
/// and intra-tile loops. A band is a contiguous set of loops. This utility
/// doesn't check for the validity of tiling itself, but just performs it.
LogicalResult
tilePerfectlyNested(MutableArrayRef<AffineForOp> input,
                    ArrayRef<unsigned> tileSizes,
                    SmallVectorImpl<AffineForOp> *tiledNest = nullptr);

```
- **EN**: Declares APIs or declarative rules around `tilePerfectlyNested`.
- **CN**: 声明与 `tilePerfectlyNested` 相关的 API 或声明式规则。

### Lines 109-115
```cpp
/// Tiles the specified band of perfectly nested loops creating tile-space
/// loops and intra-tile loops, using SSA values as tiling parameters. A band
/// is a contiguous set of loops.
LogicalResult tilePerfectlyNestedParametric(
    MutableArrayRef<AffineForOp> input, ArrayRef<Value> tileSizes,
    SmallVectorImpl<AffineForOp> *tiledNest = nullptr);

```
- **EN**: Declares APIs or declarative rules around `tilePerfectlyNestedParametric`.
- **CN**: 声明与 `tilePerfectlyNestedParametric` 相关的 API 或声明式规则。

### Lines 116-121
```cpp
/// Performs loop interchange on 'forOpA' and 'forOpB'. Requires that 'forOpA'
/// and 'forOpB' are part of a perfectly nested sequence of loops.
void interchangeLoops(AffineForOp forOpA, AffineForOp forOpB);

/// Checks if the loop interchange permutation 'loopPermMap', of the perfectly
/// nested sequence of loops in 'loops', would violate dependences (loop 'i' in
```
- **EN**: Declares APIs or declarative rules around `interchangeLoops`.
- **CN**: 声明与 `interchangeLoops` 相关的 API 或声明式规则。

### Lines 122-127
```cpp
/// 'loops' is mapped to location 'j = 'loopPermMap[i]' in the interchange).
bool isValidLoopInterchangePermutation(ArrayRef<AffineForOp> loops,
                                       ArrayRef<unsigned> loopPermMap);

/// Performs a loop permutation on a perfectly nested loop nest `inputNest`
/// (where the contained loops appear from outer to inner) as specified by the
```
- **EN**: Declares APIs or declarative rules around `isValidLoopInterchangePermutation`.
- **CN**: 声明与 `isValidLoopInterchangePermutation` 相关的 API 或声明式规则。

### Lines 128-135
```cpp
/// permutation `permMap`: loop 'i' in `inputNest` is mapped to location
/// 'loopPermMap[i]', where positions 0, 1, ... are from the outermost position
/// to inner. Returns the position in `inputNest` of the AffineForOp that
/// becomes the new outermost loop of this nest. This method always succeeds,
/// asserts out on invalid input / specifications.
unsigned permuteLoops(ArrayRef<AffineForOp> inputNest,
                      ArrayRef<unsigned> permMap);

```
- **EN**: Declares APIs or declarative rules around `permuteLoops`.
- **CN**: 声明与 `permuteLoops` 相关的 API 或声明式规则。

### Lines 136-141
```cpp
// Sinks all sequential loops to the innermost levels (while preserving
// relative order among them) and moves all parallel loops to the
// outermost (while again preserving relative order among them).
// Returns AffineForOp of the root of the new loop nest after loop interchanges.
AffineForOp sinkSequentialLoops(AffineForOp forOp);

```
- **EN**: Declares APIs or declarative rules around `sinkSequentialLoops`.
- **CN**: 声明与 `sinkSequentialLoops` 相关的 API 或声明式规则。

### Lines 142-150
```cpp
/// Performs tiling fo imperfectly nested loops (with interchange) by
/// strip-mining the `forOps` by `sizes` and sinking them, in their order of
/// occurrence in `forOps`, under each of the `targets`.
/// Returns the new AffineForOps, one per each of (`forOps`, `targets`) pair,
/// nested immediately under each of `targets`.
SmallVector<SmallVector<AffineForOp, 8>, 8> tile(ArrayRef<AffineForOp> forOps,
                                                 ArrayRef<uint64_t> sizes,
                                                 ArrayRef<AffineForOp> targets);

```
- **EN**: Declares APIs or declarative rules around `tile`.
- **CN**: 声明与 `tile` 相关的 API 或声明式规则。

### Lines 151-157
```cpp
/// Performs tiling (with interchange) by strip-mining the `forOps` by `sizes`
/// and sinking them, in their order of occurrence in `forOps`, under `target`.
/// Returns the new AffineForOps, one per `forOps`, nested immediately under
/// `target`.
SmallVector<AffineForOp, 8> tile(ArrayRef<AffineForOp> forOps,
                                 ArrayRef<uint64_t> sizes, AffineForOp target);

```
- **EN**: Declares APIs or declarative rules around `tile`.
- **CN**: 声明与 `tile` 相关的 API 或声明式规则。

### Lines 158-169
```cpp
/// Explicit copy / DMA generation options for mlir::affineDataCopyGenerate.
struct AffineCopyOptions {
  // True if DMAs should be generated instead of point-wise copies.
  bool generateDma;
  // The slower memory space from which data is to be moved.
  unsigned slowMemorySpace;
  // Memory space of the faster one (typically a scratchpad).
  unsigned fastMemorySpace;
  // Memory space to place tags in: only meaningful for DMAs.
  unsigned tagMemorySpace;
  // Capacity of the fast memory space in bytes.
  uint64_t fastMemCapacityBytes;
```
- **EN**: Introduces declarations for `AffineCopyOptions`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineCopyOptions` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 170-175
```cpp
};

/// Performs explicit copying for the contiguous sequence of operations in the
/// block iterator range [`begin', `end'), where `end' can't be past the
/// terminator of the block (since additional operations are potentially
/// inserted right before `end`. `copyOptions` provides various parameters, and
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 176-181
```cpp
/// the output argument `copyNests` is the set of all copy nests inserted, each
/// represented by its root affine.for. Since we generate alloc's and dealloc's
/// for all fast buffers (before and after the range of operations resp. or at a
/// hoisted position), all of the fast memory capacity is assumed to be
/// available for processing this block range. When 'filterMemRef' is specified,
/// copies are only generated for the provided MemRef. Returns success if the
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 182-190
```cpp
/// explicit copying succeeded for all memrefs on which affine load/stores were
/// encountered. For memrefs for whose element types a size in bytes can't be
/// computed (`index` type), their capacity is not accounted for and the
/// `fastMemCapacityBytes` copy option would be non-functional in such cases.
LogicalResult affineDataCopyGenerate(Block::iterator begin, Block::iterator end,
                                     const AffineCopyOptions &copyOptions,
                                     std::optional<Value> filterMemRef,
                                     DenseSet<Operation *> &copyNests);

```
- **EN**: Declares APIs or declarative rules around `affineDataCopyGenerate`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `affineDataCopyGenerate` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 191-197
```cpp
/// A convenience version of affineDataCopyGenerate for all ops in the body of
/// an AffineForOp.
LogicalResult affineDataCopyGenerate(AffineForOp forOp,
                                     const AffineCopyOptions &copyOptions,
                                     std::optional<Value> filterMemRef,
                                     DenseSet<Operation *> &copyNests);

```
- **EN**: Declares APIs or declarative rules around `affineDataCopyGenerate`.
- **CN**: 声明与 `affineDataCopyGenerate` 相关的 API 或声明式规则。

### Lines 198-205
```cpp
/// Result for calling generateCopyForMemRegion.
struct CopyGenerateResult {
  // Number of bytes used by alloc.
  uint64_t sizeInBytes;

  // The newly created buffer allocation.
  Operation *alloc;

```
- **EN**: Introduces declarations for `CopyGenerateResult`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `CopyGenerateResult` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 206-211
```cpp
  // Generated loop nest for copying data between the allocated buffer and the
  // original memref.
  Operation *copyNest;
};

/// generateCopyForMemRegion is similar to affineDataCopyGenerate, but works
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 212-217
```cpp
/// with a single memref region. `memrefRegion` is supposed to contain analysis
/// information within analyzedOp. The generated prologue and epilogue always
/// surround `analyzedOp`.
///
/// Note that `analyzedOp` is a single op for API convenience, and the
/// [begin, end) version can be added as needed.
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 218-225
```cpp
///
/// Also note that certain options in `copyOptions` aren't looked at anymore,
/// like slowMemorySpace.
LogicalResult generateCopyForMemRegion(const MemRefRegion &memrefRegion,
                                       Operation *analyzedOp,
                                       const AffineCopyOptions &copyOptions,
                                       CopyGenerateResult &result);

```
- **EN**: Declares APIs or declarative rules around `generateCopyForMemRegion`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `generateCopyForMemRegion` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 226-231
```cpp
/// Replace a perfect nest of "for" loops with a single linearized loop. Assumes
/// `loops` contains a list of perfectly nested loops outermost to innermost
/// that are normalized (step one and lower bound of zero) and with bounds and
/// steps independent of any loop induction variable involved in the nest.
/// Coalescing affine.for loops is not always possible, i.e., the result may not
/// be representable using affine.for.
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 232-237
```cpp
LogicalResult coalesceLoops(MutableArrayRef<AffineForOp> loops);

/// Maps `forOp` for execution on a parallel grid of virtual `processorIds` of
/// size given by `numProcessors`. This is achieved by embedding the SSA values
/// corresponding to `processorIds` and `numProcessors` into the bounds and step
/// of the `forOp`. No check is performed on the legality of the rewrite, it is
```
- **EN**: Declares APIs or declarative rules around `coalesceLoops`.
- **CN**: 声明与 `coalesceLoops` 相关的 API 或声明式规则。

### Lines 238-243
```cpp
/// the caller's responsibility to ensure legality.
///
/// Requires that `processorIds` and `numProcessors` have the same size and that
/// for each idx, `processorIds`[idx] takes, at runtime, all values between 0
/// and `numProcessors`[idx] - 1. This corresponds to traditional use cases for:
///   1. GPU (threadIdx, get_local_id(), ...)
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 244-249
```cpp
///   2. MPI (MPI_Comm_rank)
///   3. OpenMP (omp_get_thread_num)
///
/// Example:
/// Assuming a 2-d grid with processorIds = [blockIdx.x, threadIdx.x] and
/// numProcessors = [gridDim.x, blockDim.x], the loop:
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 250-255
```cpp
///
/// ```
///    scf.for %i = %lb to %ub step %step {
///      ...
///    }
/// ```
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 256-261
```cpp
///
/// is rewritten into a version resembling the following pseudo-IR:
///
/// ```
///    scf.for %i = %lb + %step * (threadIdx.x + blockIdx.x * blockDim.x)
///       to %ub step %gridDim.x * blockDim.x * %step {
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 262-267
```cpp
///      ...
///    }
/// ```
void mapLoopToProcessorIds(scf::ForOp forOp, ArrayRef<Value> processorId,
                           ArrayRef<Value> numProcessors);

```
- **EN**: Declares APIs or declarative rules around `mapLoopToProcessorIds`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `mapLoopToProcessorIds` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 268-273
```cpp
/// Gathers all AffineForOps in 'func.func' grouped by loop depth.
void gatherLoops(func::FuncOp func,
                 std::vector<SmallVector<AffineForOp, 2>> &depthToLoops);

/// Creates an AffineForOp while ensuring that the lower and upper bounds are
/// canonicalized, i.e., unused and duplicate operands are removed, any constant
```
- **EN**: Declares APIs or declarative rules around `gatherLoops`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `gatherLoops` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 274-280
```cpp
/// operands propagated/folded in, and duplicate bound maps dropped.
AffineForOp createCanonicalizedAffineForOp(OpBuilder b, Location loc,
                                           ValueRange lbOperands,
                                           AffineMap lbMap,
                                           ValueRange ubOperands,
                                           AffineMap ubMap, int64_t step = 1);

```
- **EN**: Declares APIs or declarative rules around `createCanonicalizedAffineForOp`.
- **CN**: 声明与 `createCanonicalizedAffineForOp` 相关的 API 或声明式规则。

### Lines 281-286
```cpp
/// Separates full tiles from partial tiles for a perfect nest `nest` by
/// generating a conditional guard that selects between the full tile version
/// and the partial tile version using an AffineIfOp. The original loop nest
/// is replaced by this guarded two version form.
///
///    affine.if (cond)
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 287-294
```cpp
///      // full_tile
///    else
///      // partial tile
///
LogicalResult
separateFullTiles(MutableArrayRef<AffineForOp> nest,
                  SmallVectorImpl<AffineForOp> *fullTileNest = nullptr);

```
- **EN**: Declares APIs or declarative rules around `separateFullTiles`.
- **CN**: 声明与 `separateFullTiles` 相关的 API 或声明式规则。

### Lines 295-300
```cpp
/// Walk an affine.for to find a band to coalesce.
LogicalResult coalescePerfectlyNestedAffineLoops(AffineForOp op);

/// Count the number of loops surrounding `operand` such that operand could be
/// hoisted above.
/// Stop counting at the first loop over which the operand cannot be hoisted.
```
- **EN**: Declares APIs or declarative rules around `coalescePerfectlyNestedAffineLoops`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `coalescePerfectlyNestedAffineLoops` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 301-306
```cpp
/// This counts any LoopLikeOpInterface, not just affine.for.
int64_t numEnclosingInvariantLoops(OpOperand &operand);
} // namespace affine
} // namespace mlir

#endif // MLIR_DIALECT_AFFINE_LOOPUTILS_H
```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

## Key Concepts / 关键概念

- **Dialect declarations / 方言声明**:
  - **EN**: Defines the public include-surface for a dialect, exposing operations, attributes, types, or interfaces.
  - **CN**: 定义方言的公共头文件表面，暴露操作、属性、类型或接口。
- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Models reusable capabilities that can be queried across dialect boundaries.
  - **CN**: 建模可跨方言查询的可复用能力。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。
- **Target-specific semantics / 目标相关语义**:
  - **EN**: Encodes rules tied to accelerator, GPU, or binary target environments.
  - **CN**: 编码与加速器、GPU 或二进制目标环境绑定的规则。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/IR/Block.h`, `mlir/Support/LLVM.h`, `mlir/Transforms/RegionUtils.h`
- **Standard-library headers / 标准库头文件**: `<optional>`
- **Subsystem categories / 子系统类别**: MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (1), support utilities and LLVM interop helpers / 支持工具与 LLVM 互操作辅助逻辑 (1), generic transformation and conversion helpers / 通用变换与转换辅助逻辑 (1)
