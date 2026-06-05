# Transforms.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/Tensor/Transforms/Transforms.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR Transforms component. The leading comments describe it as: Patterns.
- **用途（CN）**: 声明 MLIR Transforms 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
````cpp
//===- Transforms.h - Tensor Transformation Patterns ------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_TENSOR_TRANSFORMS_TRANSFORMS_H
#define MLIR_DIALECT_TENSOR_TRANSFORMS_TRANSFORMS_H

#include "mlir/Dialect/Tensor/IR/Tensor.h"
#include "mlir/IR/PatternMatch.h"
#include "mlir/Interfaces/ViewLikeInterface.h"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 16-20
````cpp
namespace mlir {

struct TilingResult;

namespace tensor {
````
- **EN**: This C++ declaration introduces `TilingResult` and establishes part of the API surface for `Transforms`.
- **CN**: 该 C++ 声明引入了 `TilingResult`，并构成 `Transforms` API 表面的一部分。

### Lines 22-43
````cpp
//===----------------------------------------------------------------------===//
// Patterns
//===----------------------------------------------------------------------===//

/// Method to swap an `tensor.extract_slice` with its producer when the
/// producer implements the `TilingInterface`. The pattern itself does not
/// provide a mechanism to control where the application happens. With use of
/// transform dialect that control is done within the transform dialect. Other
/// use cases can inherit from this pattern and add necessary controls.
FailureOr<TilingResult> replaceExtractSliceWithTiledProducer(
    OpBuilder &builder, tensor::ExtractSliceOp sliceOp, OpResult producerOp);

/// Method to swap `tensor.insert_slice`s with their consumers when the
/// consumer implements the `TilingInterface`. The size of `sliceOps` and
/// `consumerOperands` is expected to be the same. Every entry in
/// `consumerOperands` represents a use of the the corresponding
/// entry in `sliceOps` in the consumer. All entries of `consumerOperands` is
/// expected to be uses in the same consumer.
FailureOr<TilingResult>
replaceInsertSlicesWithTiledConsumer(OpBuilder &builder,
                                     ArrayRef<tensor::InsertSliceOp> sliceOps,
                                     ArrayRef<OpOperand *> consumerOperands);
````
- **EN**: This block groups callable interfaces such as `replaceExtractSliceWithTiledProducer`, `replaceInsertSlicesWithTiledConsumer`, indicating how `Transforms` is queried or updated.
- **CN**: 该代码块聚合了 `replaceExtractSliceWithTiledProducer`, `replaceInsertSlicesWithTiledConsumer` 等可调用接口，展示了如何查询或更新 `Transforms`。

### Lines 45-90
````cpp
//===----------------------------------------------------------------------===//
// Populate functions.
//===----------------------------------------------------------------------===//

/// Appends patterns for folding tensor subset ops into consumer load/store
/// ops into `patterns`. (This includes patterns for folding tensor subset ops
/// into vector transfer ops.)
void populateFoldTensorSubsetOpPatterns(RewritePatternSet &patterns);

/// Appends patterns for folding tensor subset ops into vector transfer ops.
void populateFoldTensorSubsetIntoVectorTransferPatterns(
    RewritePatternSet &patterns);

/// Collects patterns to merge consecutive tensor.insert_slice/extract_slice
/// into one. These patterns are in this separate entry point because the
/// bufferization is sensitive to IR structure, particularly those
/// tensor.extract_slice and tensor.insert_slice ops for creating the slices.
void populateMergeConsecutiveInsertExtractSlicePatterns(
    RewritePatternSet &patterns);

/// Appends patterns that are used to bubble up tensor.extract slice op above
/// its producer. When used as cleanup patterns of tile and fuse, enables fusing
/// the producer with the consumer even if the producer does not implement the
/// tiling interface.
void populateBubbleUpExtractSliceOpPatterns(RewritePatternSet &patterns);

/// Populates `patterns` with patterns that drop redundant tensor.insert_slice
/// rank expansions.
void populateDropRedundantInsertSliceRankExpansionPatterns(
    RewritePatternSet &patterns);

/// Populates `patterns` with patterns that fold `tensor.expand_shape` and
/// `tensor.collapse_shape` into other ops.
void populateReassociativeReshapeFoldingPatterns(RewritePatternSet &patterns);

/// Populates `patterns` with patterns that bubble up `tensor.expand_shape`
/// through `tensor.collapse_shape` ops.
void populateBubbleUpExpandShapePatterns(RewritePatternSet &patterns);

/// Populates `patterns` with patterns that fold tensor.empty with its
/// consumers.
///
/// If `singleUseOnly` is set to "true", only tensor.empty ops with a single
/// use are folded.
void populateFoldTensorEmptyPatterns(RewritePatternSet &patterns,
                                     bool foldSingleUseOnly = false);
````
- **EN**: This block groups callable interfaces such as `populateFoldTensorSubsetOpPatterns`, `populateFoldTensorSubsetIntoVectorTransferPatterns`, `populateMergeConsecutiveInsertExtractSlicePatterns`, `populateBubbleUpExtractSliceOpPatterns`, indicating how `Transforms` is queried or updated.
- **CN**: 该代码块聚合了 `populateFoldTensorSubsetOpPatterns`, `populateFoldTensorSubsetIntoVectorTransferPatterns`, `populateMergeConsecutiveInsertExtractSlicePatterns`, `populateBubbleUpExtractSliceOpPatterns` 等可调用接口，展示了如何查询或更新 `Transforms`。

### Lines 91-103
````cpp
/// Populates `patterns` with patterns that decompose `tensor.concat` into
/// `tensor.empty` of a tensor of the concatenated size, followed by a chain
/// of `tensor.insert_slice` operations on the inputs. This is intended to be
/// used as a fallback tensor -> tensor lowering that decomposes concat such
/// that it can be bufferized into a sequence of copies.
void populateDecomposeTensorConcatPatterns(RewritePatternSet &patterns);

using ControlFoldFn = std::function<bool(OpOperand *)>;

/// Populates `patterns` with patterns that replace tensor ops (such as
/// tensor.generate) with constants when possible.
void populateRewriteAsConstantPatterns(RewritePatternSet &patterns,
                                       const ControlFoldFn &controlFn);
````
- **EN**: This block groups callable interfaces such as `populateDecomposeTensorConcatPatterns`, `bool`, `populateRewriteAsConstantPatterns`, indicating how `Transforms` is queried or updated.
- **CN**: 该代码块聚合了 `populateDecomposeTensorConcatPatterns`, `bool`, `populateRewriteAsConstantPatterns` 等可调用接口，展示了如何查询或更新 `Transforms`。

### Lines 106-155
````cpp
//===----------------------------------------------------------------------===//
// Transform helpers
//===----------------------------------------------------------------------===//

/// Build a new tensor::PadOp with low/high padding that is independent of all
/// given independencies. If the op is already independent of all
/// independencies, the same PadOp result is returned.
///
/// Failure indicates the no suitable upper bound for low/high padding could be
/// found.
///
/// Example:
/// scf.for %iv = %lb to %ub step %step {
///   %high = affine.apply affine_map<(d0)[s0] -> (s0 - d0)> (%i)[%ub]
///   %p = tensor.pad %t low[5] high[%high] ...
///   ...
/// }
///
/// The function builds IR such as:
/// %high_new = affine.apply affine_map<()[s0, s1] -> (-s0 + s1)> ()[%lb, %ub]
/// %p_hoistable = tensor.pad %t low[5] high[%high_new]
/// %dim = tensor.dim %t, %c0
/// %size = affine.apply affine_map<(d0)[s0, s1] -> (-d0 + s0 + s1 + 5)>
///     (%iv)[%ub, %dim]
/// %slice = tensor.extract_slice %p_hoistable [0] [%size] [1]
///
/// The slice is returned.
FailureOr<Value> buildIndependentOp(OpBuilder &b, tensor::PadOp padOp,
                                    ValueRange independencies);

/// Build a new tensor::EmptyOp who's dynamic sizes are independent of all
/// given independencies. If the op is already independent of all
/// independencies, the same EmptyOp result is returned.
///
/// Failure indicates the no suitable upper bound for the dynamic sizes could be
/// found.
FailureOr<Value> buildIndependentOp(OpBuilder &b, tensor::EmptyOp emptyOp,
                                    ValueRange independencies);

/// Computes the offsets, sizes, and strides needed to build a collapsed
/// `sliceOp`. The dimensions to collapse are specified by `reassociation`.
///
/// This fails when the specified collapse cannot be represented by a valid
/// ExtractSliceOp.
LogicalResult
getCollapsedExtractSliceInfo(OpBuilder &b, tensor::ExtractSliceOp sliceOp,
                             ArrayRef<ReassociationIndices> reassociation,
                             SmallVectorImpl<OpFoldResult> &collapsedOffsets,
                             SmallVectorImpl<OpFoldResult> &collapsedSizes,
                             SmallVectorImpl<OpFoldResult> &collapsedStrides);
````
- **EN**: This block groups callable interfaces such as `buildIndependentOp`, `getCollapsedExtractSliceInfo`, indicating how `Transforms` is queried or updated.
- **CN**: 该代码块聚合了 `buildIndependentOp`, `getCollapsedExtractSliceInfo` 等可调用接口，展示了如何查询或更新 `Transforms`。

### Lines 156-171
````cpp
/// Computes the offsets, sizes, and strides needed to build an expanded
/// `sliceOp`. The dimensions to expand are specified by `reassociation` and
/// the shape of `expandedValue`.
///
/// This fails when the specified expansion cannot be represented by a valid
/// ExtractSliceOp.
LogicalResult
getExpandedExtractSliceInfo(OpBuilder &b, tensor::ExtractSliceOp sliceOp,
                            ArrayRef<ReassociationIndices> reassociation,
                            Value expandedValue,
                            SmallVectorImpl<OpFoldResult> &expandedOffsets,
                            SmallVectorImpl<OpFoldResult> &expandedSizes,
                            SmallVectorImpl<OpFoldResult> &expandedStrides);

} // namespace tensor
} // namespace mlir
````
- **EN**: This block groups callable interfaces such as `getExpandedExtractSliceInfo`, indicating how `Transforms` is queried or updated.
- **CN**: 该代码块聚合了 `getExpandedExtractSliceInfo` 等可调用接口，展示了如何查询或更新 `Transforms`。

### Lines 174-174
````cpp
#endif // MLIR_DIALECT_TENSOR_TRANSFORMS_TRANSFORMS_H
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: C++ declaration surface for MLIR infrastructure
  **CN**: MLIR 基础设施的 C++ 声明层
- **EN**: IR construction and mutation helpers
  **CN**: IR 构造与变更辅助接口

## Dependencies / 依赖关系

- mlir/Dialect/Tensor/IR/Tensor.h
- mlir/IR/PatternMatch.h
- mlir/Interfaces/ViewLikeInterface.h
