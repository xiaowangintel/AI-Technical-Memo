# Tensor.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/Tensor/IR/Tensor.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR Tensor component.
- **用途（CN）**: 声明 MLIR Tensor 组件相关的 C++ 接口与辅助类型。

## Line-by-Line Analysis / 逐行分析

### Lines 1-27
````cpp
//===- Tensor.h - Tensor dialect --------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_TENSOR_IR_TENSOR_H_
#define MLIR_DIALECT_TENSOR_IR_TENSOR_H_

#include "mlir/Bytecode/BytecodeOpInterface.h"
#include "mlir/Dialect/Utils/ReshapeOpsUtils.h"
#include "mlir/IR/BuiltinTypes.h"
#include "mlir/IR/Dialect.h"
#include "mlir/IR/OpDefinition.h"
#include "mlir/IR/OpImplementation.h"
#include "mlir/Interfaces/CastInterfaces.h"
#include "mlir/Interfaces/ControlFlowInterfaces.h"
#include "mlir/Interfaces/DestinationStyleOpInterface.h"
#include "mlir/Interfaces/InferIntRangeInterface.h"
#include "mlir/Interfaces/InferTypeOpInterface.h"
#include "mlir/Interfaces/ParallelCombiningOpInterface.h"
#include "mlir/Interfaces/ShapedOpInterfaces.h"
#include "mlir/Interfaces/SideEffectInterfaces.h"
#include "mlir/Interfaces/TilingInterface.h"
#include "mlir/Interfaces/ViewLikeInterface.h"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 29-41
````cpp
//===----------------------------------------------------------------------===//
// Tensor Dialect Helpers
//===----------------------------------------------------------------------===//

namespace mlir {

/// Return the list of Range (i.e. offset, size, stride). Each Range
/// entry contains either the dynamic value or a ConstantIndexOp constructed
/// with `b` at location `loc`.
SmallVector<Range, 8> getOrCreateRanges(OffsetSizeAndStrideOpInterface op,
                                        OpBuilder &b, Location loc);

} // namespace mlir
````
- **EN**: This C++ declaration introduces `mlir` and establishes part of the API surface for `Tensor`. Representative entry points here include `getOrCreateRanges`.
- **CN**: 该 C++ 声明引入了 `mlir`，并构成 `Tensor` API 表面的一部分。 这一段可见的代表性接口包括 `getOrCreateRanges`。

### Lines 43-47
````cpp
//===----------------------------------------------------------------------===//
// Tensor Dialect
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Tensor/IR/TensorOpsDialect.h.inc"
````
- **EN**: This block pulls in the MLIR/LLVM/TableGen dependencies required by later declarations and helps establish the compilation boundary of the file.
- **CN**: 该代码块引入后续声明所依赖的 MLIR/LLVM/TableGen 头文件或记录文件，用于建立本文件的编译边界。

### Lines 49-54
````cpp
//===----------------------------------------------------------------------===//
// Tensor Dialect Operations
//===----------------------------------------------------------------------===//

#define GET_OP_CLASSES
#include "mlir/Dialect/Tensor/IR/TensorOps.h.inc"
````
- **EN**: This macro block selects a generated declaration fragment before the corresponding `.inc` file is included.
- **CN**: 该宏代码块在包含对应的 `.inc` 文件之前，选择要展开的生成式声明片段。

### Lines 56-111
````cpp
//===----------------------------------------------------------------------===//
// Tensor Dialect Helpers
//===----------------------------------------------------------------------===//

namespace mlir {
namespace tensor {

/// Returns true if `target` is a ranked tensor type that preserves static
/// information available in the `source` ranked tensor type.
bool preservesStaticInformation(Type source, Type target);

/// Determines whether tensor::CastOp casts to a more dynamic version of the
/// source tensor. This is useful to fold a tensor.cast into a consuming op and
/// implement canonicalization patterns for ops in different dialects that may
/// consume the results of tensor.cast operations. Such foldable tensor.cast
/// operations are typically inserted as `extract_slice` ops and are
/// canonicalized, to preserve the type compatibility of their uses.
///
/// Returns true when all conditions are met:
/// 1. source and result are ranked tensors with same element type and rank.
/// 2. the tensor type has more static information than the result
///
/// Example:
/// ```mlir
///   %1 = tensor.cast %0 : tensor<8x16xf32> to tensor<?x?xf32>
///   %2 = consumer %1 ... : tensor<?x?xf32> ...
/// ```
///
/// folds into:
///
/// ```mlir
///   %2 = consumer %0 ... : tensor<8x16xf32> ...
/// ```
bool canFoldIntoConsumerOp(CastOp castOp);

/// Determines whether the tensor::CastOp casts to a more static version of the
/// source tensor. This is useful to fold into a producing op and implement
/// canonicaliation patterns with the `tensor.cast` op as the root, but producer
/// being from different dialects. Returns true when all conditions are met:
/// 1. source and result and ranked tensors with same element type and rank.
/// 2. the result type has more static information than the source.
///
/// Example:
/// ```mlir
///   %1 = producer ... : tensor<?x?xf32>
///   %2 = tensor.cast %1 : tensor<?x?xf32> to tensor<8x16xf32>
/// ```
///
/// can be canonicalized to :
///
/// ```mlir
///   %2 = producer ... : tensor<8x16xf32>
/// ```
/// Not all ops might be canonicalizable this way, but for those that can be,
/// this method provides a check that it is worth doing the canonicalization.
bool canFoldIntoProducerOp(CastOp castOp);
````
- **EN**: This C++ declaration introduces `mlir` and establishes part of the API surface for `Tensor`. Representative entry points here include `preservesStaticInformation`, `canFoldIntoConsumerOp`, `canFoldIntoProducerOp`.
- **CN**: 该 C++ 声明引入了 `mlir`，并构成 `Tensor` API 表面的一部分。 这一段可见的代表性接口包括 `preservesStaticInformation`, `canFoldIntoConsumerOp`, `canFoldIntoProducerOp`。

### Lines 112-155
````cpp
/// Return true if any of the operands of `op` is a CastOp that can be folded
/// into its consumer, i.e. `op`. This is effectively a convenience wrapper for
/// `canFoldIntoProducerOp`.
bool hasFoldableTensorCastOperand(Operation *op);

/// Assuming that `op` contains at least one operand that is a foldable CastOp
/// (i.e. `hasFoldableTensorCastOperand` returns true), calculate the updated
/// operands.
SmallVector<Value>
getUpdatedOperandsAfterCastOpFolding(DestinationStyleOpInterface op,
                                     SmallVector<Type> &newResTy);

/// Performs folding of any operand of `op` if it comes from a tensor::CastOp
/// that can be folded.
LogicalResult foldTensorCast(Operation *op);

/// Return the dimension of the given tensor value.
OpFoldResult getMixedSize(OpBuilder &builder, Location loc, Value value,
                          int64_t dim);

/// Return the dimensions of the given tensor value.
SmallVector<OpFoldResult> getMixedSizes(OpBuilder &builder, Location loc,
                                        Value value);

/// Create a rank-reducing ExtractSliceOp @[0 .. 0] with strides [1 .. 1] and
/// appropriate sizes (i.e. `tensor.getSizes()`) to reduce the rank of `tensor`
/// to that of `targetType`.
Value createCanonicalRankReducingExtractSliceOp(OpBuilder &b, Location loc,
                                                Value tensor,
                                                RankedTensorType targetType);

/// Create a rank-reducing InsertSliceOp @[0 .. 0] with strides [1 .. 1] and
/// appropriate sizes (i.e. `dest.getSizes()`). The result is a new tensor with
/// rank increased to that of `dest`, obtained by inserting `tensor` into `dest`
/// at the canonical [0 .. 0] position.
Value createCanonicalRankReducingInsertSliceOp(OpBuilder &b, Location loc,
                                               Value tensor, Value dest);

/// This is a helper function for DestinationStyleOpInterface. If there is a
/// destination operand for the given OpResult, return that operand. Otherwise,
/// return an empty tensor (`tensor.empty`) with the shape of the OpResult.
/// Dynamic dimensions are queried via ReifyRankedShapedTypeOpInterface.
FailureOr<Value> getOrCreateDestination(OpBuilder &b, Location loc,
                                        OpResult opResult);
````
- **EN**: This block groups callable interfaces such as `hasFoldableTensorCastOperand`, `getUpdatedOperandsAfterCastOpFolding`, `foldTensorCast`, `getMixedSize`, indicating how `Tensor` is queried or updated.
- **CN**: 该代码块聚合了 `hasFoldableTensorCastOperand`, `getUpdatedOperandsAfterCastOpFolding`, `foldTensorCast`, `getMixedSize` 等可调用接口，展示了如何查询或更新 `Tensor`。

### Lines 157-183
````cpp
/// This is a helper function for DestinationStyleOpInterface. Get or create
/// destinations for every tensor OpResult of the given op.
LogicalResult getOrCreateDestinations(OpBuilder &b, Location loc, Operation *op,
                                      SmallVector<Value> &result);

/// Tests if types are the same when ignoring encoding on ranked tensors.
bool isSameTypeWithoutEncoding(Type tp1, Type tp2);

/// Function to control the folding of constant and extract slice.
using ControlConstantExtractSliceFusionFn = std::function<bool(ExtractSliceOp)>;

/// Patterns to fold the extract slice op with its constant operand.
void populateFoldConstantExtractSlicePatterns(
    RewritePatternSet &patterns,
    const ControlConstantExtractSliceFusionFn &controlFn =
        [](ExtractSliceOp op) {
          // Disable by default because the folding can generate a large
          // constant tensor, which would affect the compile time and storage.
          return false;
        });

/// Patterns to fold extracts of a collapse_shaped tensor to an extract of the
/// source tensor.
void populateFoldCollapseExtractPatterns(RewritePatternSet &patterns);

} // namespace tensor
} // namespace mlir
````
- **EN**: This block groups callable interfaces such as `getOrCreateDestinations`, `isSameTypeWithoutEncoding`, `bool`, `populateFoldConstantExtractSlicePatterns`, indicating how `Tensor` is queried or updated.
- **CN**: 该代码块聚合了 `getOrCreateDestinations`, `isSameTypeWithoutEncoding`, `bool`, `populateFoldConstantExtractSlicePatterns` 等可调用接口，展示了如何查询或更新 `Tensor`。

### Lines 186-186
````cpp
#endif // MLIR_DIALECT_TENSOR_IR_TENSOR_H_
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: C++ declaration surface for MLIR infrastructure
  **CN**: MLIR 基础设施的 C++ 声明层
- **EN**: IR construction and mutation helpers
  **CN**: IR 构造与变更辅助接口
- **EN**: Type or attribute abstraction
  **CN**: 类型或属性抽象

## Dependencies / 依赖关系

- mlir/Bytecode/BytecodeOpInterface.h
- mlir/Dialect/Utils/ReshapeOpsUtils.h
- mlir/IR/BuiltinTypes.h
- mlir/IR/Dialect.h
- mlir/IR/OpDefinition.h
- mlir/IR/OpImplementation.h
- mlir/Interfaces/CastInterfaces.h
- mlir/Interfaces/ControlFlowInterfaces.h
- mlir/Interfaces/DestinationStyleOpInterface.h
- mlir/Interfaces/InferIntRangeInterface.h
- mlir/Interfaces/InferTypeOpInterface.h
- mlir/Interfaces/ParallelCombiningOpInterface.h
- mlir/Interfaces/ShapedOpInterfaces.h
- mlir/Interfaces/SideEffectInterfaces.h
- mlir/Interfaces/TilingInterface.h
- mlir/Interfaces/ViewLikeInterface.h
- mlir/Dialect/Tensor/IR/TensorOpsDialect.h.inc
- mlir/Dialect/Tensor/IR/TensorOps.h.inc
