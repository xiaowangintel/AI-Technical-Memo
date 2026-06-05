# ViewLikeInterface.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Interfaces/ViewLikeInterface.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR ViewLikeInterface component. The leading comments describe it as: This file implements the operation interface for view-like operations.
- **用途（CN）**: 声明 MLIR ViewLikeInterface 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-21
````cpp
//===- ViewLikeInterface.h - View-like operations interface ---------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the operation interface for view-like operations.
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_INTERFACES_VIEWLIKEINTERFACE_H_
#define MLIR_INTERFACES_VIEWLIKEINTERFACE_H_

#include "mlir/Dialect/Utils/StaticValueUtils.h"
#include "mlir/IR/Builders.h"
#include "mlir/IR/BuiltinAttributes.h"
#include "mlir/IR/BuiltinTypes.h"
#include "mlir/IR/OpImplementation.h"
#include "mlir/IR/PatternMatch.h"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 23-44
````cpp
namespace mlir {

class OffsetSizeAndStrideOpInterface;

namespace detail {

LogicalResult verifyOffsetSizeAndStrideOp(OffsetSizeAndStrideOpInterface op);

bool sameOffsetsSizesAndStrides(
    OffsetSizeAndStrideOpInterface a, OffsetSizeAndStrideOpInterface b,
    llvm::function_ref<bool(OpFoldResult, OpFoldResult)> cmp);

/// Helper method to compute the number of dynamic entries of `staticVals`,
/// up to `idx`.
unsigned getNumDynamicEntriesUpToIdx(ArrayRef<int64_t> staticVals,
                                     unsigned idx);

} // namespace detail
} // namespace mlir

/// Include the generated interface declarations.
#include "mlir/Interfaces/ViewLikeInterface.h.inc"
````
- **EN**: This block pulls in the MLIR/LLVM/TableGen dependencies required by later declarations and helps establish the compilation boundary of the file.
- **CN**: 该代码块引入后续声明所依赖的 MLIR/LLVM/TableGen 头文件或记录文件，用于建立本文件的编译边界。

### Lines 46-95
````cpp
namespace mlir {

/// Result for slice bounds verification;
struct SliceBoundsVerificationResult {
  /// If set to "true", the slice bounds verification was successful.
  bool isValid;
  /// An error message that can be printed during op verification.
  std::string errorMessage;
};

/// Verify that the offsets/sizes/strides-style access into the given shape
/// is in-bounds. Only static values are verified. If `generateErrorMessage`
/// is set to "true", an error message is produced that can be printed by the
///  op verifier.
SliceBoundsVerificationResult
verifyInBoundsSlice(ArrayRef<int64_t> shape, ArrayRef<int64_t> staticOffsets,
                    ArrayRef<int64_t> staticSizes,
                    ArrayRef<int64_t> staticStrides,
                    bool generateErrorMessage = false);
SliceBoundsVerificationResult verifyInBoundsSlice(
    ArrayRef<int64_t> shape, ArrayRef<OpFoldResult> mixedOffsets,
    ArrayRef<OpFoldResult> mixedSizes, ArrayRef<OpFoldResult> mixedStrides,
    bool generateErrorMessage = false);

/// Pattern to rewrite dynamic offsets/sizes/strides of view/slice-like ops as
/// constant arguments. This pattern assumes that the op has a suitable builder
/// that takes a result type, a "source" operand and mixed offsets, sizes and
/// strides.
///
/// `OpType` is the type of op to which this pattern is applied. `ResultTypeFn`
/// returns the new result type of the op, based on the new offsets, sizes and
/// strides. `CastOpFunc` is used to generate a cast op if the result type of
/// the op has changed.
template <typename OpType, typename ResultTypeFn, typename CastOpFunc>
class OpWithOffsetSizesAndStridesConstantArgumentFolder final
    : public OpRewritePattern<OpType> {
public:
  using OpRewritePattern<OpType>::OpRewritePattern;

  LogicalResult matchAndRewrite(OpType op,
                                PatternRewriter &rewriter) const override {
    SmallVector<OpFoldResult> mixedOffsets(op.getMixedOffsets());
    SmallVector<OpFoldResult> mixedSizes(op.getMixedSizes());
    SmallVector<OpFoldResult> mixedStrides(op.getMixedStrides());

    // No constant operands were folded, just return;
    if (failed(foldDynamicIndexList(mixedOffsets, /*onlyNonNegative=*/true)) &&
        failed(foldDynamicIndexList(mixedSizes, /*onlyNonNegative=*/true)) &&
        failed(foldDynamicIndexList(mixedStrides)))
      return failure();
````
- **EN**: This C++ declaration introduces `SliceBoundsVerificationResult` and establishes part of the API surface for `ViewLikeInterface`. Representative entry points here include `verifyInBoundsSlice`, `matchAndRewrite`, `mixedOffsets`, `getMixedOffsets`.
- **CN**: 该 C++ 声明引入了 `SliceBoundsVerificationResult`，并构成 `ViewLikeInterface` API 表面的一部分。 这一段可见的代表性接口包括 `verifyInBoundsSlice`, `matchAndRewrite`, `mixedOffsets`, `getMixedOffsets`。

### Lines 96-174
````cpp
    // Pattern does not apply if the produced op would not verify.
    SliceBoundsVerificationResult sliceResult = verifyInBoundsSlice(
        cast<ShapedType>(op.getSource().getType()).getShape(), mixedOffsets,
        mixedSizes, mixedStrides);
    if (!sliceResult.isValid)
      return failure();

    // Compute the new result type.
    auto resultType =
        ResultTypeFn()(op, mixedOffsets, mixedSizes, mixedStrides);
    if (!resultType)
      return failure();

    // Create the new op in canonical form.
    auto newOp =
        OpType::create(rewriter, op.getLoc(), resultType, op.getSource(),
                       mixedOffsets, mixedSizes, mixedStrides);
    CastOpFunc()(rewriter, op, newOp);

    return success();
  }
};

/// Printer hooks for custom directive in assemblyFormat.
///
///   custom<DynamicIndexList>($values, $integers)
///   custom<DynamicIndexList>($values, $integers, type($values))
///
/// where `values` is of ODS type `Variadic<*>` and `integers` is of ODS type
/// `I64ArrayAttr`. Print a list where each element is either:
///    1. the static integer value in `integers`, if it's not `kDynamic` or,
///    2. the next value in `values`, otherwise.
///
/// If `valueTypes` is provided, the corresponding type of each dynamic value is
/// printed. Otherwise, the type is not printed. Each type must match the type
/// of the corresponding value in `values`. `valueTypes` is redundant for
/// printing as we can retrieve the types from the actual `values`. However,
/// `valueTypes` is needed for parsing and we must keep the API symmetric for
/// parsing and printing. The type for integer elements is `i64` by default and
/// never printed.
///
/// Integer indices can also be scalable in the context of scalable vectors,
/// denoted by square brackets (e.g., "[2, [4], 8]"). For each value in
/// `integers`, the corresponding `bool` in `scalableFlags` encodes whether it's
/// a scalable index. If `scalableFlags` is empty then assume that all indices
/// are non-scalable.
///
/// Examples:
///
///   * Input: `integers = [kDynamic, 7, 42, kDynamic]`,
///            `values = [%arg0, %arg42]` and
///            `valueTypes = [index, index]`
///     prints:
///       `[%arg0 : index, 7, 42, %arg42 : i32]`
///
///   * Input: `integers = [kDynamic, 7, 42, kDynamic]`,
///            `values = [%arg0, %arg42]` and
///            `valueTypes = []`
///     prints:
///       `[%arg0, 7, 42, %arg42]`
///
///   * Input: `integers = [2, 4, 8]`,
///            `values = []` and
///            `scalableFlags = [false, true, false]`
///     prints:
///       `[2, [4], 8]`
///
void printDynamicIndexList(
    OpAsmPrinter &printer, Operation *op, OperandRange values,
    ArrayRef<int64_t> integers, ArrayRef<bool> scalableFlags,
    TypeRange valueTypes = TypeRange(),
    AsmParser::Delimiter delimiter = AsmParser::Delimiter::Square);
inline void printDynamicIndexList(
    OpAsmPrinter &printer, Operation *op, OperandRange values,
    ArrayRef<int64_t> integers, TypeRange valueTypes = TypeRange(),
    AsmParser::Delimiter delimiter = AsmParser::Delimiter::Square) {
  return printDynamicIndexList(printer, op, values, integers,
                               /*scalableFlags=*/{}, valueTypes, delimiter);
}
````
- **EN**: This block groups callable interfaces such as `verifyInBoundsSlice`, `getSource`, `getType`, `getShape`, indicating how `ViewLikeInterface` is queried or updated.
- **CN**: 该代码块聚合了 `verifyInBoundsSlice`, `getSource`, `getType`, `getShape` 等可调用接口，展示了如何查询或更新 `ViewLikeInterface`。

### Lines 176-223
````cpp
/// Parser hooks for custom directive in assemblyFormat.
///
///   custom<DynamicIndexList>($values, $integers)
///   custom<DynamicIndexList>($values, $integers, type($values))
///
/// where `values` is of ODS type `Variadic<*>` and `integers` is of ODS
/// type `I64ArrayAttr`. Parse a mixed list where each element is either a
/// static integer or an SSA value. Fill `integers` with the integer ArrayAttr,
/// where `kDynamic` encodes the position of SSA values. Add the parsed SSA
/// values to `values` in-order.
///
/// If `valueTypes` is provided, fill it with the types corresponding to each
/// value in `values`. Otherwise, the caller must handle the types and parsing
/// will fail if the type of the value is found (e.g., `[%arg0 : index, 3, %arg1
/// : index]`).
///
/// Integer indices can also be scalable in the context of scalable vectors,
/// denoted by square brackets (e.g., "[2, [4], 8]"). For each value in
/// `integers`, the corresponding `bool` in `scalableFlags` encodes whether it's
/// a scalable index.
///
/// Examples:
///
///   * After parsing "[%arg0 : index, 7, 42, %arg42 : i32]":
///       1. `result` is filled with `[kDynamic, 7, 42, kDynamic]`
///       2. `values` is filled with "[%arg0, %arg1]".
///       3. `scalableFlags` is filled with `[false, true, false]`.
///
///   * After parsing `[2, [4], 8]`:
///       1. `result` is filled with `[2, 4, 8]`
///       2. `values` is empty.
///       3. `scalableFlags` is filled with `[false, true, false]`.
///
ParseResult parseDynamicIndexList(
    OpAsmParser &parser,
    SmallVectorImpl<OpAsmParser::UnresolvedOperand> &values,
    DenseI64ArrayAttr &integers, DenseBoolArrayAttr &scalableFlags,
    SmallVectorImpl<Type> *valueTypes = nullptr,
    AsmParser::Delimiter delimiter = AsmParser::Delimiter::Square);
inline ParseResult parseDynamicIndexList(
    OpAsmParser &parser,
    SmallVectorImpl<OpAsmParser::UnresolvedOperand> &values,
    DenseI64ArrayAttr &integers, SmallVectorImpl<Type> *valueTypes = nullptr,
    AsmParser::Delimiter delimiter = AsmParser::Delimiter::Square) {
  DenseBoolArrayAttr scalableFlags;
  return parseDynamicIndexList(parser, values, integers, scalableFlags,
                               valueTypes, delimiter);
}
````
- **EN**: This block groups callable interfaces such as `parseDynamicIndexList`, indicating how `ViewLikeInterface` is queried or updated.
- **CN**: 该代码块聚合了 `parseDynamicIndexList` 等可调用接口，展示了如何查询或更新 `ViewLikeInterface`。

### Lines 225-248
````cpp
/// Verify that a the `values` has as many elements as the number of entries in
/// `attr` for which `isDynamic` evaluates to true.
LogicalResult verifyListOfOperandsOrIntegers(Operation *op, StringRef name,
                                             unsigned expectedNumElements,
                                             ArrayRef<int64_t> attr,
                                             ValueRange values);

namespace OpTrait {
/// This trai indicates that pointer-like objects (such as memrefs) returned
/// from this operation will never alias with each other. This provides a
/// guarantee to optimization passes that accesses through different results
/// of this operation can be safely reordered, as they will never reference
/// overlapping memory locations.
///
/// Operations with this trait take multiple pointer-like operands
/// and return the same operands with additional non-aliasing guarantees.
/// If the access to the results of this operation aliases at runtime, the
/// behavior of such access is undefined.
template <typename ConcreteType>
class DistinctObjectsTrait
    : public TraitBase<ConcreteType, DistinctObjectsTrait> {};
} // namespace OpTrait

} // namespace mlir
````
- **EN**: This C++ declaration introduces `DistinctObjectsTrait` and establishes part of the API surface for `ViewLikeInterface`. Representative entry points here include `verifyListOfOperandsOrIntegers`.
- **CN**: 该 C++ 声明引入了 `DistinctObjectsTrait`，并构成 `ViewLikeInterface` API 表面的一部分。 这一段可见的代表性接口包括 `verifyListOfOperandsOrIntegers`。

### Lines 251-251
````cpp
#endif // MLIR_INTERFACES_VIEWLIKEINTERFACE_H_
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

- mlir/Dialect/Utils/StaticValueUtils.h
- mlir/IR/Builders.h
- mlir/IR/BuiltinAttributes.h
- mlir/IR/BuiltinTypes.h
- mlir/IR/OpImplementation.h
- mlir/IR/PatternMatch.h
- mlir/Interfaces/ViewLikeInterface.h.inc
- DistinctObjectsTrait inherits from public TraitBase<ConcreteType, DistinctObjectsTrait>
- DistinctObjectsTrait builds on public TraitBase<ConcreteType, DistinctObjectsTrait>
