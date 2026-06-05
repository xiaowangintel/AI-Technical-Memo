# VectorDistribution.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/Vector/Transforms/VectorDistribution.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR VectorDistribution component. The leading comments describe it as: Lamdba function to let users allocate memory needed for the lowering of.
- **用途（CN）**: 声明 MLIR VectorDistribution 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13
````cpp
//===- VectorDistribution.h - Vector distribution patterns --*- C++------*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_VECTOR_TRANSFORMS_VECTORDISTRIBUTION_H_
#define MLIR_DIALECT_VECTOR_TRANSFORMS_VECTORDISTRIBUTION_H_

#include "mlir/Dialect/GPU/IR/GPUDialect.h"
#include "mlir/Dialect/Vector/IR/VectorOps.h"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 15-73
````cpp
namespace mlir {
class RewritePatternSet;
namespace vector {

struct WarpExecuteOnLane0LoweringOptions {
  /// Lamdba function to let users allocate memory needed for the lowering of
  /// WarpExecuteOnLane0Op.
  /// The function needs to return an allocation that the lowering can use as
  /// temporary memory. The allocation needs to match the shape of the type (the
  /// type may be VectorType or a scalar) and be availble for the current warp.
  /// If there are several warps running in parallel the allocation needs to be
  /// split so that each warp has its own allocation.
  using WarpAllocationFn = std::function<Value(
      Location, OpBuilder &, gpu::WarpExecuteOnLane0Op, Type)>;
  WarpAllocationFn warpAllocationFn = nullptr;

  /// Lamdba function to let user emit operation to synchronize all the thread
  /// within a warp. After this operation all the threads can see any memory
  /// written before the operation.
  using WarpSynchronizationFn =
      std::function<void(Location, OpBuilder &, gpu::WarpExecuteOnLane0Op)>;
  WarpSynchronizationFn warpSynchronizationFn = nullptr;
};

void populateWarpExecuteOnLane0OpToScfForPattern(
    RewritePatternSet &patterns,
    const WarpExecuteOnLane0LoweringOptions &options,
    PatternBenefit benefit = 1);

using DistributionMapFn = std::function<AffineMap(Value)>;

/// Distribute transfer_write ops based on the affine map returned by
/// `distributionMapFn`. Writes of size more than `maxNumElementToExtract`
/// will not be distributed (it should be less than the warp size).
///
/// Example:
/// ```
/// %0 = gpu.warp_execute_on_lane_0(%id){
///   ...
///   vector.transfer_write %v, %A[%c0] : vector<32xf32>, memref<128xf32>
///   gpu.yield
/// }
/// ```
/// To
/// ```
/// %r:3 = gpu.warp_execute_on_lane_0(%id) -> (vector<1xf32>) {
///   ...
///   gpu.yield %v : vector<32xf32>
/// }
/// vector.transfer_write %v, %A[%id] : vector<1xf32>, memref<128xf32>
///
/// When applied at the same time as the vector propagation patterns,
/// distribution of `vector.transfer_write` is expected to have the highest
/// priority (pattern benefit). By making propagation of `vector.transfer_read`
/// be the lowest priority pattern, it will be the last vector operation to
/// distribute, meaning writes should propagate first.
void populateDistributeTransferWriteOpPatterns(
    RewritePatternSet &patterns, const DistributionMapFn &distributionMapFn,
    unsigned maxNumElementsToExtract, PatternBenefit benefit = 2);
````
- **EN**: This C++ declaration introduces `RewritePatternSet` and establishes part of the API surface for `VectorDistribution`. Representative entry points here include `Value`, `void`, `populateWarpExecuteOnLane0OpToScfForPattern`, `AffineMap`.
- **CN**: 该 C++ 声明引入了 `RewritePatternSet`，并构成 `VectorDistribution` API 表面的一部分。 这一段可见的代表性接口包括 `Value`, `void`, `populateWarpExecuteOnLane0OpToScfForPattern`, `AffineMap`。

### Lines 74-113
````cpp
/// Move scalar operations with no dependency on the warp op outside of the
/// region.
void moveScalarUniformCode(gpu::WarpExecuteOnLane0Op op);

/// Lambda signature to compute a warp shuffle of a given value of a given lane
/// within a given warp size.
using WarpShuffleFromIdxFn =
    std::function<Value(Location, OpBuilder &b, Value, Value, int64_t)>;

/// Collect patterns to propagate warp distribution. `distributionMapFn` is used
/// to decide how a value should be distributed when this cannot be inferred
/// from its uses.
///
/// The separate control over the `vector.transfer_read` op pattern benefit
/// is given to ensure the order of reads/writes before and after distribution
/// is consistent. As noted above, writes are expected to have the highest
/// priority for distribution, but are only ever distributed if adjacent to the
/// yield. By making reads the lowest priority pattern, it will be the last
/// vector operation to distribute, meaning writes should propagate first. This
/// is relatively brittle when ops fail to distribute, but that is a limitation
/// of these propagation patterns when there is a dependency not modeled by SSA.
void populatePropagateWarpVectorDistributionPatterns(
    RewritePatternSet &pattern, const DistributionMapFn &distributionMapFn,
    const WarpShuffleFromIdxFn &warpShuffleFromIdxFn,
    PatternBenefit benefit = 1, PatternBenefit readBenefit = 0);

/// Lambda signature to compute a reduction of a distributed value for the given
/// reduction kind and size.
using DistributedReductionFn =
    std::function<Value(Location, OpBuilder &, Value, CombiningKind, uint32_t)>;

/// Collect patterns to distribute vector reduction ops using given lamdba to
/// distribute reduction op.
void populateDistributeReduction(
    RewritePatternSet &pattern,
    const DistributedReductionFn &distributedReductionFn,
    PatternBenefit benefit = 1);

} // namespace vector
} // namespace mlir
````
- **EN**: This block groups callable interfaces such as `moveScalarUniformCode`, `Value`, `populatePropagateWarpVectorDistributionPatterns`, `populateDistributeReduction`, indicating how `VectorDistribution` is queried or updated.
- **CN**: 该代码块聚合了 `moveScalarUniformCode`, `Value`, `populatePropagateWarpVectorDistributionPatterns`, `populateDistributeReduction` 等可调用接口，展示了如何查询或更新 `VectorDistribution`。

### Lines 115-115
````cpp
#endif // MLIR_DIALECT_VECTOR_TRANSFORMS_VECTORDISTRIBUTION_H_
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: C++ declaration surface for MLIR infrastructure
  **CN**: MLIR 基础设施的 C++ 声明层
- **EN**: Type or attribute abstraction
  **CN**: 类型或属性抽象

## Dependencies / 依赖关系

- mlir/Dialect/GPU/IR/GPUDialect.h
- mlir/Dialect/Vector/IR/VectorOps.h
