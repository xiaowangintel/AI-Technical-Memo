# LoweringPatterns.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/Vector/Transforms/LoweringPatterns.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR LoweringPatterns component. The leading comments describe it as: Lowering pattern populate functions.
- **用途（CN）**: 声明 MLIR LoweringPatterns 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13
````cpp
//===- LoweringPatterns.h - Vector rewrite patterns --------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_VECTOR_TRANSFORMS_LOWERINGPATTERNS_H
#define MLIR_DIALECT_VECTOR_TRANSFORMS_LOWERINGPATTERNS_H

#include "mlir/Dialect/Vector/Transforms/VectorRewritePatterns.h"
#include "mlir/Dialect/Vector/Transforms/VectorTransforms.h"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 15-18
````cpp
namespace mlir {
class RewritePatternSet;

namespace vector {
````
- **EN**: This C++ declaration introduces `RewritePatternSet` and establishes part of the API surface for `LoweringPatterns`.
- **CN**: 该 C++ 声明引入了 `RewritePatternSet`，并构成 `LoweringPatterns` API 表面的一部分。

### Lines 20-71
````cpp
//===----------------------------------------------------------------------===//
// Lowering pattern populate functions
//===----------------------------------------------------------------------===//

/// Populate the pattern set with the following patterns:
///
/// [OuterProductOpLowering]
/// Progressively lower a `vector.outerproduct` to linearized
/// `vector.extract` + `vector.fma` + `vector.insert`.
///
/// [ContractionOpLowering]
/// Progressive lowering of ContractionOp.
/// One:
///   %x = vector.contract with at least one free/batch dimension
/// is replaced by:
///   %a = vector.contract with one less free/batch dimension
///   %b = vector.contract with one less free/batch dimension
///
/// [ContractionOpToMatmulOpLowering]
/// Progressively lower a `vector.contract` with row-major matmul semantics to
/// linearized `vector.shape_cast` + `vector.matmul` on the way to
/// `llvm.matrix.multiply`.
///
/// [ContractionOpToDotLowering]
/// Progressively lower a `vector.contract` with row-major matmul semantics to
/// linearized `vector.extract` + `vector.reduction` + `vector.insert`.
///
/// [ContractionOpToOuterProductOpLowering]
/// Progressively lower a `vector.contract` with row-major matmul semantics to
/// linearized `vector.extract` + `vector.outerproduct` + `vector.insert`.
void populateVectorContractLoweringPatterns(
    RewritePatternSet &patterns,
    VectorContractLowering vectorContractLoweringOption,
    PatternBenefit benefit = 1, bool disableOuterProductLowering = false);

/// Populate the pattern set with the following patterns:
///
/// [OuterProductOpLowering]
/// Progressively lower a `vector.outerproduct` to linearized
/// `vector.extract` + `vector.fma` + `vector.insert`.
void populateVectorOuterProductLoweringPatterns(RewritePatternSet &patterns,
                                                PatternBenefit benefit = 1);

/// Populate the pattern set with the following patterns:
///
/// [InnerOuterDimReductionConversion]
/// Rewrites vector.multi_reduction such that all reduction dimensions are
/// either innermost or outermost, by adding the proper vector.transpose
/// operations.
void populateVectorMultiReductionReorderPatterns(
    RewritePatternSet &patterns, VectorMultiReductionLowering options,
    PatternBenefit benefit = 1);
````
- **EN**: This block groups callable interfaces such as `populateVectorContractLoweringPatterns`, `populateVectorOuterProductLoweringPatterns`, `populateVectorMultiReductionReorderPatterns`, indicating how `LoweringPatterns` is queried or updated.
- **CN**: 该代码块聚合了 `populateVectorContractLoweringPatterns`, `populateVectorOuterProductLoweringPatterns`, `populateVectorMultiReductionReorderPatterns` 等可调用接口，展示了如何查询或更新 `LoweringPatterns`。

### Lines 72-119
````cpp
/// Populate the pattern set with the following patterns:
///
/// [ReduceMultiDimReductionRank]
/// Once in innermost or outermost reduction
/// form, rewrites n-D vector.multi_reduction into 2-D vector.multi_reduction,
/// by introducing vector.shape_cast ops to collapse + multi-reduce + expand
/// back.
void populateVectorMultiReductionFlatteningPatterns(
    RewritePatternSet &patterns, VectorMultiReductionLowering options,
    PatternBenefit benefit = 1);

/// Populate the pattern set with the following patterns:
///
/// [OneDimMultiReductionToReduction]
/// Converts 1-D vector.multi_reduction to vector.reduction.
///
/// [TwoDimMultiReductionToElementWise]
/// Once in 2-D vector.multi_reduction form, with an **outermost** reduction
/// dimension, unroll the outer dimension to obtain a sequence of 1-D vector
/// ops. This also has an opportunity for tree-reduction (in the future).
///
/// [TwoDimMultiReductionToReduction]
/// Once in 2-D vector.multi_reduction form, with an **innermost** reduction
/// dimension, unroll the outer dimension to obtain a sequence of extract +
/// vector.reduction + insert. This can further lower to horizontal reduction
/// ops.
void populateVectorMultiReductionUnrollingPatterns(
    RewritePatternSet &patterns, VectorMultiReductionLowering options,
    PatternBenefit benefit = 1);

/// Populate the pattern set with the following patterns:
///
/// [TransferReadToVectorLoadLowering]
/// Progressive lowering of BroadcastOp to ExtractOp + InsertOp + lower-D
/// BroadcastOp until dim 1.
void populateVectorBroadcastLoweringPatterns(RewritePatternSet &patterns,
                                             PatternBenefit benefit = 1);

/// Populate the pattern set with the following patterns:
///
/// [CreateMaskOp]
/// Progressive lowering of CreateMaskOp to lower-D CreateMaskOp until dim 1.
///
/// [ConstantMaskOp]
/// Progressive lowering of ConstantMaskOp to lower-D ConstantMaskOp until
/// dim 1.
void populateVectorMaskOpLoweringPatterns(RewritePatternSet &patterns,
                                          PatternBenefit benefit = 1);
````
- **EN**: This block groups callable interfaces such as `populateVectorMultiReductionFlatteningPatterns`, `populateVectorMultiReductionUnrollingPatterns`, `populateVectorBroadcastLoweringPatterns`, `populateVectorMaskOpLoweringPatterns`, indicating how `LoweringPatterns` is queried or updated.
- **CN**: 该代码块聚合了 `populateVectorMultiReductionFlatteningPatterns`, `populateVectorMultiReductionUnrollingPatterns`, `populateVectorBroadcastLoweringPatterns`, `populateVectorMaskOpLoweringPatterns` 等可调用接口，展示了如何查询或更新 `LoweringPatterns`。

### Lines 121-175
````cpp
/// Collects patterns that lower scalar vector transfer ops to memref loads and
/// stores when beneficial. If `allowMultipleUses` is set to true, the patterns
/// are applied to vector transfer reads with any number of uses. Otherwise,
/// only vector transfer reads with a single use will be lowered.
void populateScalarVectorTransferLoweringPatterns(RewritePatternSet &patterns,
                                                  PatternBenefit benefit,
                                                  bool allowMultipleUses);

/// Populate the pattern set with the following patterns:
///
/// [ShapeCastOp2DDownCastRewritePattern]
/// ShapeOp 2D -> 1D downcast serves the purpose of flattening 2-D to 1-D
/// vectors progressively.
///
/// [ShapeCastOp2DUpCastRewritePattern]
/// ShapeOp 1D -> 2D upcast serves the purpose of unflattening 2-D from 1-D
/// vectors progressively.
///
/// [ShapeCastOpRewritePattern]
/// Reference lowering to fully unrolled sequences of single element ExtractOp +
/// InsertOp. Note that applying this pattern can almost always be considered a
/// performance bug.
void populateVectorShapeCastLoweringPatterns(RewritePatternSet &patterns,
                                             PatternBenefit benefit = 1);

/// Populate the pattern set with the following patterns:
///
/// [TransposeOpLowering]
///
/// [TransposeOp2DToShuffleLowering]
///
void populateVectorTransposeLoweringPatterns(
    RewritePatternSet &patterns,
    VectorTransposeLowering vectorTransposeLowering,
    PatternBenefit benefit = 1);

/// Populate the pattern set with the following patterns:
///
/// [TransferReadToVectorLoadLowering]
/// Progressive lowering of transfer_read.This pattern supports lowering of
/// `vector.transfer_read` to a combination of `vector.load` and
/// `vector.broadcast`
///
/// [TransferWriteToVectorStoreLowering]
/// Progressive lowering of transfer_write. This pattern supports lowering of
/// `vector.transfer_write` to `vector.store`
///
/// These patterns lower transfer ops to simpler ops like `vector.load`,
/// `vector.store` and `vector.broadcast`. Only transfers with a transfer rank
/// of a most `maxTransferRank` are lowered. This is useful when combined with
/// VectorToSCF, which reduces the rank of vector transfer ops.
void populateVectorTransferLoweringPatterns(
    RewritePatternSet &patterns,
    std::optional<unsigned> maxTransferRank = std::nullopt,
    PatternBenefit benefit = 1);
````
- **EN**: This block groups callable interfaces such as `populateScalarVectorTransferLoweringPatterns`, `populateVectorShapeCastLoweringPatterns`, `populateVectorTransposeLoweringPatterns`, `populateVectorTransferLoweringPatterns`, indicating how `LoweringPatterns` is queried or updated.
- **CN**: 该代码块聚合了 `populateScalarVectorTransferLoweringPatterns`, `populateVectorShapeCastLoweringPatterns`, `populateVectorTransposeLoweringPatterns`, `populateVectorTransferLoweringPatterns` 等可调用接口，展示了如何查询或更新 `LoweringPatterns`。

### Lines 177-230
````cpp
/// Collect a set of transfer read/write lowering patterns that simplify the
/// permutation map (e.g., converting it to a minor identity map) by inserting
/// broadcasts and transposes. More specifically:
///
/// [TransferReadPermutationLowering]
/// Lower transfer_read op with permutation into a transfer_read with a
/// permutation map composed of leading zeros followed by a minor identity +
/// vector.transpose op.
/// Ex:
///     vector.transfer_read ...
///         permutation_map: (d0, d1, d2) -> (0, d1)
/// into:
///     %v = vector.transfer_read ...
///         permutation_map: (d0, d1, d2) -> (d1, 0)
///     vector.transpose %v, [1, 0]
///
///     vector.transfer_read ...
///         permutation_map: (d0, d1, d2, d3) -> (0, 0, 0, d1, d3)
/// into:
///     %v = vector.transfer_read ...
///         permutation_map: (d0, d1, d2, d3) -> (0, 0, d1, 0, d3)
///     vector.transpose %v, [0, 1, 3, 2, 4]
/// Note that an alternative is to transform it to linalg.transpose +
/// vector.transfer_read to do the transpose in memory instead.
///
/// [TransferWritePermutationLowering]
/// Lower transfer_write op with permutation into a transfer_write with a
/// minor identity permutation map. (transfer_write ops cannot have broadcasts.)
/// Ex:
///     vector.transfer_write %v ...
///         permutation_map: (d0, d1, d2) -> (d2, d0, d1)
/// into:
///     %tmp = vector.transpose %v, [2, 0, 1]
///     vector.transfer_write %tmp ...
///         permutation_map: (d0, d1, d2) -> (d0, d1, d2)
///
///     vector.transfer_write %v ...
///         permutation_map: (d0, d1, d2, d3) -> (d3, d2)
/// into:
///     %tmp = vector.transpose %v, [1, 0]
///     %v = vector.transfer_write %tmp ...
///         permutation_map: (d0, d1, d2, d3) -> (d2, d3)
///
/// [TransferOpReduceRank]
/// Lower transfer_read op with broadcast in the leading dimensions into
/// transfer_read of lower rank + vector.broadcast.
/// Ex: vector.transfer_read ...
///         permutation_map: (d0, d1, d2, d3) -> (0, d1, 0, d3)
/// into:
///     %v = vector.transfer_read ...
///         permutation_map: (d0, d1, d2, d3) -> (d1, 0, d3)
///     vector.broadcast %v
void populateVectorTransferPermutationMapLoweringPatterns(
    RewritePatternSet &patterns, PatternBenefit benefit = 1);
````
- **EN**: This block groups callable interfaces such as `populateVectorTransferPermutationMapLoweringPatterns`, indicating how `LoweringPatterns` is queried or updated.
- **CN**: 该代码块聚合了 `populateVectorTransferPermutationMapLoweringPatterns` 等可调用接口，展示了如何查询或更新 `LoweringPatterns`。

### Lines 232-278
````cpp
/// Populate the pattern set with the following patterns:
///
/// [ScanToArithOps]
/// Convert vector.scan op into arith ops and vector.insert_strided_slice /
/// vector.extract_strided_slice.
void populateVectorScanLoweringPatterns(RewritePatternSet &patterns,
                                        PatternBenefit benefit = 1);

/// Populate the pattern set with the following patterns:
///
/// [StepToArithConstantOp]
/// Convert vector.step op into arith ops if not using scalable vectors
void populateVectorStepLoweringPatterns(RewritePatternSet &patterns,
                                        PatternBenefit benefit = 1);

/// Populate the pattern set with the following patterns:
///
/// [UnrollGather]
/// Unrolls 2 or more dimensional `vector.gather` ops by unrolling the
/// outermost dimension.
void populateVectorGatherLoweringPatterns(RewritePatternSet &patterns,
                                          PatternBenefit benefit = 1);

/// Populate the pattern set with the following patterns:
///
/// [Gather1DToConditionalLoads]
/// Turns 1-d `vector.gather` into a scalarized sequence of `vector.loads` or
/// `tensor.extract`s. To avoid out-of-bounds memory accesses, these
/// loads/extracts are made conditional using `scf.if` ops.
void populateVectorGatherToConditionalLoadPatterns(RewritePatternSet &patterns,
                                                   PatternBenefit benefit = 1);

/// Populates instances of `MaskOpRewritePattern` to lower masked operations
/// with `vector.mask`. Patterns should rewrite the `vector.mask` operation and
/// not its nested `MaskableOpInterface`.
void populateVectorMaskLoweringPatternsForSideEffectingOps(
    RewritePatternSet &patterns);

/// Populate the pattern set with the following patterns:
///
/// [VectorMaskedLoadOpConverter]
/// Turns vector.maskedload to scf.if + memref.load
///
/// [VectorMaskedStoreOpConverter]
/// Turns vector.maskedstore to scf.if + memref.store
void populateVectorMaskedLoadStoreEmulationPatterns(RewritePatternSet &patterns,
                                                    PatternBenefit benefit = 1);
````
- **EN**: This block groups callable interfaces such as `populateVectorScanLoweringPatterns`, `populateVectorStepLoweringPatterns`, `populateVectorGatherLoweringPatterns`, `populateVectorGatherToConditionalLoadPatterns`, indicating how `LoweringPatterns` is queried or updated.
- **CN**: 该代码块聚合了 `populateVectorScanLoweringPatterns`, `populateVectorStepLoweringPatterns`, `populateVectorGatherLoweringPatterns`, `populateVectorGatherToConditionalLoadPatterns` 等可调用接口，展示了如何查询或更新 `LoweringPatterns`。

### Lines 280-326
````cpp
/// Populate the pattern set with the following patterns:
///
/// [UnrollInterleaveOp]
/// A one-shot unrolling of InterleaveOp to (one or more) ExtractOp +
/// InterleaveOp (of `targetRank`) + InsertOp.
void populateVectorInterleaveLoweringPatterns(RewritePatternSet &patterns,
                                              int64_t targetRank = 1,
                                              PatternBenefit benefit = 1);

void populateVectorInterleaveToShufflePatterns(RewritePatternSet &patterns,
                                               PatternBenefit benefit = 1);

void populateVectorDeinterleaveToShufflePatterns(RewritePatternSet &patterns,
                                                 PatternBenefit benefit = 1);

/// Populates the pattern set with the following patterns:
///
/// [UnrollBitCastOp]
/// A one-shot unrolling of BitCastOp to (one or more) ExtractOp +
/// BitCastOp (of `targetRank`) + InsertOp.
void populateVectorBitCastLoweringPatterns(RewritePatternSet &patterns,
                                           int64_t targetRank = 1,
                                           PatternBenefit benefit = 1);

void populateVectorShuffleLoweringPatterns(RewritePatternSet &patterns,
                                           PatternBenefit benefit = 1);

/// Populates a pattern that rank-reduces n-D FMAs into (n-1)-D FMAs where
/// n > 1.
void populateVectorRankReducingFMAPattern(RewritePatternSet &patterns);

/// Populate patterns to rewrite sequences of `vector.to_elements` +
/// `vector.from_elements` operations into a tree of `vector.shuffle`
/// operations.
void populateVectorToFromElementsToShuffleTreePatterns(
    RewritePatternSet &patterns, PatternBenefit benefit = 1);

/// Populate the pattern set with the following patterns:
///
/// [ContractionOpToMatmulOpLowering]
/// Lowers `vector.contract` to `llvm.intr.matrix.multiply`.
///
/// Given the high benefit, this will be prioriotised over other
/// contract-lowering patterns. As such, the convert-vector-to-llvm pass will
/// only run this registration conditionally.
void populateVectorContractToMatrixMultiply(RewritePatternSet &patterns,
                                            PatternBenefit benefit = 100);
````
- **EN**: This block groups callable interfaces such as `populateVectorInterleaveLoweringPatterns`, `populateVectorInterleaveToShufflePatterns`, `populateVectorDeinterleaveToShufflePatterns`, `populateVectorBitCastLoweringPatterns`, indicating how `LoweringPatterns` is queried or updated.
- **CN**: 该代码块聚合了 `populateVectorInterleaveLoweringPatterns`, `populateVectorInterleaveToShufflePatterns`, `populateVectorDeinterleaveToShufflePatterns`, `populateVectorBitCastLoweringPatterns` 等可调用接口，展示了如何查询或更新 `LoweringPatterns`。

### Lines 328-340
````cpp
/// Populate the pattern set with the following patterns:
///
/// [TransposeOpLowering]
/// Lowers `vector.transpose` to `llvm.intr.matrix.flat_transpose`.
///
/// Given the high benefit, this will be prioriotised over other
/// transpose-lowering patterns. As such, the convert-vector-to-llvm pass will
/// only run this registration conditionally.
void populateVectorTransposeToFlatTranspose(RewritePatternSet &patterns,
                                            PatternBenefit benefit = 100);

} // namespace vector
} // namespace mlir
````
- **EN**: This block groups callable interfaces such as `populateVectorTransposeToFlatTranspose`, indicating how `LoweringPatterns` is queried or updated.
- **CN**: 该代码块聚合了 `populateVectorTransposeToFlatTranspose` 等可调用接口，展示了如何查询或更新 `LoweringPatterns`。

### Lines 343-343
````cpp
#endif // MLIR_DIALECT_VECTOR_TRANSFORMS_LOWERINGPATTERNS_H
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: C++ declaration surface for MLIR infrastructure
  **CN**: MLIR 基础设施的 C++ 声明层
- **EN**: IR construction and mutation helpers
  **CN**: IR 构造与变更辅助接口

## Dependencies / 依赖关系

- mlir/Dialect/Vector/Transforms/VectorRewritePatterns.h
- mlir/Dialect/Vector/Transforms/VectorTransforms.h
