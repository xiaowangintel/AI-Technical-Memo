# XeGPUUtils.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/XeGPU/Utils/XeGPUUtils.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR XeGPUUtils component.
- **用途（CN）**: 声明 MLIR XeGPUUtils 组件相关的 C++ 接口与辅助类型。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
````cpp
//===- XeGPUUtils.h - Vector Utilities --------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_XEGPU_UTILS_XEGPUUTILS_H_
#define MLIR_DIALECT_XEGPU_UTILS_XEGPUUTILS_H_

#include "mlir/Dialect/XeGPU/IR/XeGPU.h"
#include "mlir/IR/BuiltinTypes.h"
#include "mlir/IR/OpDefinition.h"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 15-63
````cpp
namespace mlir {

class VectorType;
class OpOperand;
class OpResult;
class OpBuilder;
class ValueRange;
class TypeConverter;
class OpFoldResult;

namespace xegpu {
class DistributeLayoutAttr;
class LayoutAttr;
class TensorDescType;

namespace uArch {
struct uArch;
} // namespace uArch
} // namespace xegpu

namespace xegpu {

/// Flatten a set of ValueRange into a single SmallVector<Value>
SmallVector<Value> flattenValues(ArrayRef<ValueRange> values);

/// If tensor descriptor has a layout attribute it is used in SIMT mode.
/// In this mode, the distributed vector shape is determined as follows:
/// Definitions:
///        lane_data_size = lane_data[0] × lane_data[1]
///        subgroup_size = lane_layout[0] × lane_layout[1]
///        distribution_unit_size = subgroup_size × lane_data_size
///
/// Case 1: Regular loads/stores.
/// The following conditions must be met:
///        * tensor_desc[0] == lane_layout[0]
/// Distributed vector is a 1D vector with shape:
///        [chunk_size]
///
/// Case 2: Block loads/stores
/// Additional definitions:
///        tensor_size = tensor_desc[0] * .. * tensor_desc[r-1] * array_length
///        n_distribution_units = tensor_size / distribution_unit_size
///        fragment_size = n_distribution_units * lane_data_size
/// Given above definitions, the following conditions must be met:
///        * tensor_desc[0] % (lane_layout[0] × lane_data[0]) == 0
///        * tensor_desc[1] % (lane_layout[1] × lane_data[1]) == 0
/// Distributed vector is a 1D vector with shape:
///        [fragment_size]
FailureOr<VectorType> getDistributedVectorType(xegpu::TensorDescType tdescTy);
````
- **EN**: This C++ declaration introduces `VectorType` and establishes part of the API surface for `XeGPUUtils`. Representative entry points here include `flattenValues`, `getDistributedVectorType`.
- **CN**: 该 C++ 声明引入了 `VectorType`，并构成 `XeGPUUtils` API 表面的一部分。 这一段可见的代表性接口包括 `flattenValues`, `getDistributedVectorType`。

### Lines 64-110
````cpp
/// Helper to get the distributed vector type for a given vector type according
/// to a given LayoutAttr.
FailureOr<VectorType> getDistributedVectorType(VectorType originalType,
                                               LayoutAttr layout);

/// Helper function to get distributed vector type for a source vector type
/// according to the lane_layout. We simply divide each dimension of tensor
/// descriptor shape by corresponding lane_layout dimension. If
/// array_length > 1, that is appended to the front of the distributed shape.
///
/// Examples:
/// | original vector shape | lane_layout | distributed vector shape |
/// |-----------------------|-------------|--------------------------|
/// | 32x16                 | [1, 16]     | 32x1                     |
/// | 32x16                 | [2, 8]      | 16x2                     |
/// | 2x32x16               | [1, 16]     | 2x32x1                   |
FailureOr<VectorType>
getDistVecTypeBasedOnLaneLayout(DistributeLayoutAttr layout,
                                VectorType originalType);

/// Extract a set of small vectors from a value with a given shape using
/// vector.extract_stride_slice
SmallVector<Value> extractVectorsWithShapeFromValue(OpBuilder &builder,
                                                    Location loc, Value value,
                                                    ArrayRef<int64_t> shape);

/// Create a vector of shape from a set of values using
/// vector.insert_stride_slice.
Value createVectorWithShapeFromValues(OpBuilder &builder, Location loc,
                                      ValueRange values,
                                      ArrayRef<int64_t> shape);

/// Do type conversion for SCF structural ops, e.g., scf.for using SCF structure
/// type convertion patterns. Since VectorType cannot carry the layout
/// attribute, which is needed to guide the type conversion for XeGPU, they are
/// first converted into RankedTensorType, where the layout attribute can be
/// attached. And then upstream SCF structural type conversion patterns are
/// applied with the provided converter.
/// TODO: This is a temporary solution. We should refactor it when context-aware
/// type conversion is available.
void doSCFStructuralTypeConversionWithTensorType(Operation *op,
                                                 TypeConverter converter);

/// Retrieves the chip string from the XeVM target attribute of the parent
/// GPU module operation. Returns the chip identifier if found, or nullopt
/// if no GPU module parent or XeVM target attribute exists.
std::optional<std::string> getChipStr(Operation *op);
````
- **EN**: This block groups callable interfaces such as `getDistributedVectorType`, `getDistVecTypeBasedOnLaneLayout`, `extractVectorsWithShapeFromValue`, `createVectorWithShapeFromValues`, indicating how `XeGPUUtils` is queried or updated.
- **CN**: 该代码块聚合了 `getDistributedVectorType`, `getDistVecTypeBasedOnLaneLayout`, `extractVectorsWithShapeFromValue`, `createVectorWithShapeFromValues` 等可调用接口，展示了如何查询或更新 `XeGPUUtils`。

### Lines 112-156
````cpp
/// Generates element-wise addition ops of two arrays with same length.
SmallVector<OpFoldResult> addElementwise(OpBuilder &builder, Location loc,
                                         ArrayRef<OpFoldResult> lhs,
                                         ArrayRef<OpFoldResult> rhs);

/// Generates element-wise addition ops of two arrays with automatic alignment.
/// When the input arrays have different sizes, the shorter array is
/// right-aligned with the longer array, and the unmatched leading elements from
/// the longer array are preserved unchanged. This is commonly used for offset
/// computation where higher-dimensional offsets need to be added to
/// lower-dimensional adjustments.
///
/// Example:
///   lhs = [l1, l2, l3], rhs = [r1, r2]
///   Result: [11, l2+r1, l3+r2]
SmallVector<OpFoldResult> addWithRightAligned(OpBuilder &builder, Location loc,
                                              ArrayRef<OpFoldResult> lhs,
                                              ArrayRef<OpFoldResult> rhs);

/// Given an `input` value representing per-lane data, this function returns the
/// result after performing a reduction on the input over all lanes (number of
/// lanes given by `size`). This uses butterfly shuffles to perform the
/// reduction in a log2(size) number of steps.
/// NOTE: Implementation taken from TestVectorTransforms.cpp
Value subgroupReduction(Location loc, OpBuilder &builder, Value input,
                        vector::CombiningKind kind, uint32_t size);

/// Given a `src` and an `acc` argumments from a vector::MultiDimReductionOp,
/// lower to a set of vector::ReductionOp ops over 1D slices extracted from
/// `src`. The reduction is performed along `reductionDim`. The result is a
/// vector with the same shape as `acc`.
/// TODO: Only 2D to 1D reduction is supported for now.
Value lowerToVectorReductions(TypedValue<VectorType> src,
                              TypedValue<VectorType> acc,
                              vector::CombiningKind kind, int64_t reductionDim,
                              Location loc, PatternRewriter &rewriter);

/// Creates a constant filled with the neutral (identity) value for the
/// given reduction kind. For example: 0 for ADD/OR/XOR, 1 for MUL/AND,
/// max/min signed/unsigned int for MINSI/MINUI/MAXSI/MAXUI, and +/-infinity
/// for float min/max operations. If \p type is a VectorType, returns a splat
/// vector constant; otherwise returns a scalar constant. Returns nullptr if
/// the element type is incompatible with the requested reduction kind.
Value createReductionNeutralValue(OpBuilder &builder, Location loc, Type type,
                                  vector::CombiningKind kind);
````
- **EN**: This block groups callable interfaces such as `addElementwise`, `addWithRightAligned`, `subgroupReduction`, `lowerToVectorReductions`, indicating how `XeGPUUtils` is queried or updated.
- **CN**: 该代码块聚合了 `addElementwise`, `addWithRightAligned`, `subgroupReduction`, `lowerToVectorReductions` 等可调用接口，展示了如何查询或更新 `XeGPUUtils`。

### Lines 158-203
````cpp
/// Lowers cross-lane reductions to shuffle operations on a 2D vector.
/// Extracts slices along the reduction dimension, performs subgroup reductions
/// with shuffles across reductionSize work-items, and inserts the results back
/// into an accumulator vector.
Value lowerCrossLaneReductionToShuffles(TypedValue<VectorType> src,
                                        TypedValue<VectorType> acc,
                                        vector::CombiningKind kind,
                                        int64_t reductionDim,
                                        int64_t reductionSize, Location loc,
                                        PatternRewriter &rewriter);

/// Helper Function to find a proper instruction multiple for the user-supplied
/// sg-level data shape (diven by `dim`). `candidates` are uArch allowed shapes.
/// `candidateMultiples` are uArch multiples of such shapes (i.e. block count or
/// array length).
template <typename T>
int getLargestDivisor(T dim, ArrayRef<T> candidates,
                      ArrayRef<T> candidateMultiples = {});

/// Retrieves the DistributeLayoutAttr associated with a given Value. For
/// TensorDescType values, the DistributeLayoutAttr is extracted from the
/// TensorDescType itself. For other values, it is obtained from the attributes
/// of the defining operation. Returns nullptr if no DistributeLayoutAttr is
/// found.
DistributeLayoutAttr getDistributeLayoutAttr(const Value value);

/// Retrieves the DistributeLayoutAttr associated with a given OpOperand. It
/// will first check the operand_layout_{id} of the owner operation. If not
/// found, it will check the operand itself and its defining op.
DistributeLayoutAttr getDistributeLayoutAttr(const OpOperand &opr);

/// [to-be-deprecated] Sets the DistributeLayoutAttr for a given OpResult
/// user should use setAnchorLayout instead
void setDistributeLayoutAttr(const OpResult &Result,
                             const DistributeLayoutAttr layout);

/// [to-be-deprecated] Sets the DistributeLayoutAttr for a given OpOperand
/// user should use setAnchorLayout instead
void setDistributeLayoutAttr(const OpOperand &opr,
                             const DistributeLayoutAttr layout);

/// Return the attribute name for the OpOperand to attach DistributeLayoutAttr
std::string getTemporaryLayoutName(const OpOperand &operand);

/// Return the attribute name for the OpResult to attach DistributeLayoutAttr
std::string getTemporaryLayoutName(const OpResult result);
````
- **EN**: This block groups callable interfaces such as `lowerCrossLaneReductionToShuffles`, `getLargestDivisor`, `getDistributeLayoutAttr`, `setDistributeLayoutAttr`, indicating how `XeGPUUtils` is queried or updated.
- **CN**: 该代码块聚合了 `lowerCrossLaneReductionToShuffles`, `getLargestDivisor`, `getDistributeLayoutAttr`, `setDistributeLayoutAttr` 等可调用接口，展示了如何查询或更新 `XeGPUUtils`。

### Lines 205-238
````cpp
/// get and set distribute layout attribute for non-anchor operations
/// (and offsets/masks of load/store ops before we get rid of their temp attrs)
template <typename T,
          typename = std::enable_if_t<std::is_same_v<T, OpOperand> ||
                                      std::is_same_v<T, OpResult>>>
DistributeLayoutAttr getTemporaryLayout(const T &operandOrResult);

template <typename T,
          typename = std::enable_if_t<std::is_same_v<T, OpOperand> ||
                                      std::is_same_v<T, OpResult>>>
void setTemporaryLayout(const T &operandOrResult,
                        const DistributeLayoutAttr layout);

/// Helper function to check if the layout is packed. Layout is packed if it is
/// 2D and lane_data[0] != 1 (data packed from col dimension).
/// TODO: Move to target info.
bool requirePacked(const DistributeLayoutAttr layout);

/// Helper function to check if the layout requires a transpose effect.
bool requireTranspose(const DistributeLayoutAttr layout,
                      const uArch::uArch *uArch);

// Check if dst shape is an expansion of src shape by inserting unit dimensions.
bool matchUnitDimExpansion(ArrayRef<int64_t> src, ArrayRef<int64_t> dst,
                           SmallVector<int64_t> &expandedUnitDims);

// Checks if dst shape is an expansion of src shape where each dimension in src
// is split into one or more consecutive dimensions in dst
bool matchSplitDimExpansion(ArrayRef<int64_t> src, ArrayRef<int64_t> dst,
                            SmallVector<SmallVector<int64_t>> &splitDimGroups);

} // namespace xegpu

} // namespace mlir
````
- **EN**: This block groups callable interfaces such as `getTemporaryLayout`, `setTemporaryLayout`, `requirePacked`, `requireTranspose`, indicating how `XeGPUUtils` is queried or updated.
- **CN**: 该代码块聚合了 `getTemporaryLayout`, `setTemporaryLayout`, `requirePacked`, `requireTranspose` 等可调用接口，展示了如何查询或更新 `XeGPUUtils`。

### Lines 241-241
````cpp
#endif // MLIR_DIALECT_XEGPU_UTILS_XEGPUUTILS_H_
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

- mlir/Dialect/XeGPU/IR/XeGPU.h
- mlir/IR/BuiltinTypes.h
- mlir/IR/OpDefinition.h
