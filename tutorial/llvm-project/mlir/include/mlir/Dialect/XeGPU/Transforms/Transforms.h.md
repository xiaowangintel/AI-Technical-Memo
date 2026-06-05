# Transforms.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/XeGPU/Transforms/Transforms.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR Transforms component.
- **用途（CN）**: 声明 MLIR Transforms 组件相关的 C++ 接口与辅助类型。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
````cpp
//===- Transforms.h - XeGPU Dialect transformations -------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_XEGPU_TRANSFORMS_TRANSFORMS_H
#define MLIR_DIALECT_XEGPU_TRANSFORMS_TRANSFORMS_H

#include "mlir/IR/Builders.h"
#include "mlir/IR/Operation.h"
#include "mlir/Transforms/DialectConversion.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/Support/LogicalResult.h"

#include <functional>
#include <optional>
#include <utility>
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 22-84
````cpp
namespace mlir {
class RewritePatternSet;

namespace xegpu {

/// Options to control the XeGPU unrolling. Its main purpose is to
/// provide a way to customize the native shape of the operation.
struct UnrollOptions {
  /// Callback function that indicates whether vector unrolling should be
  /// attempted on the operation.
  using FilterConstraintFnType = std::function<LogicalResult(Operation *op)>;
  FilterConstraintFnType filterConstraint = nullptr;
  UnrollOptions &setFilterConstraint(FilterConstraintFnType constraint) {
    filterConstraint = std::move(constraint);
    return *this;
  }

  /// Function that computes the target shape for unrolling. It returns an
  /// optional vector of integers representing the shape. If it returns
  /// `std::nullopt`, unrolling is aborted for the given operation.
  using NativeShapeFnType =
      std::function<std::optional<SmallVector<int64_t>>(Operation *op)>;
  NativeShapeFnType nativeShape = nullptr;
  UnrollOptions &setNativeShapeFn(NativeShapeFnType fn) {
    nativeShape = std::move(fn);
    return *this;
  }

  /// Function that converts a ShapedType (TensorDescType or VectorType)
  /// into the unrolled type based on the tileShape. It returns a vector of
  /// types representing the unrolled types for simplicity. When
  /// `returnSingleType` is true, it returns a vector containing only one single
  /// unrolled type.
  using UnrolledTypeFnType = std::function<SmallVector<Type>(
      ShapedType type, ArrayRef<int64_t> tileShape, bool returnSingleType)>;
  UnrolledTypeFnType getUnrolledTypes = nullptr;
  UnrollOptions &setUnrolledTypesFn(UnrolledTypeFnType fn) {
    getUnrolledTypes = std::move(fn);
    return *this;
  }
};

/// Appends patterns for optimizing block load operations into `patterns`.
void populateXeGPUPeepHoleOptimizerPatterns(RewritePatternSet &patterns);
/// Appends patterns for array length optimization into `patterns`.
void populateXeGPUArrayLengthOptimizationPatterns(RewritePatternSet &patterns);
/// Appends patterns for XeGPU SIMT distribution into `patterns`.
void populateXeGPUSubgroupDistributePatterns(RewritePatternSet &patterns);
/// Appends patterns for moving function body into gpu.warp_execute_on_lane0 op.
void populateXeGPUMoveFuncBodyToWarpOpPatterns(RewritePatternSet &patterns);
/// Appends patterns for XeGPU workgroup to subgroup distribution into
/// `patterns`.
void populateXeGPUWgToSgDistributePatterns(RewritePatternSet &patterns);
/// Define only the type conversions needed for XeGPU subgroup to workitem
/// distribution.
void populateXeGPUSgToWiDistributeTypeConversions(TypeConverter &typeConverter);
/// Defines type conversions and legality for XeGPU subgroup to workitem
/// distribution and appends the required conversion patterns into `patterns`.
/// Appends patterns for XeGPU subgroup to workitem distribution into
/// `patterns`.
void populateXeGPUSgToWiDistributeTypeConversionAndLegality(
    TypeConverter &typeConverter, RewritePatternSet &patterns,
    ConversionTarget &target);
````
- **EN**: This C++ declaration introduces `RewritePatternSet` and establishes part of the API surface for `Transforms`. Representative entry points here include `LogicalResult`, `setFilterConstraint`, `move`, `setNativeShapeFn`.
- **CN**: 该 C++ 声明引入了 `RewritePatternSet`，并构成 `Transforms` API 表面的一部分。 这一段可见的代表性接口包括 `LogicalResult`, `setFilterConstraint`, `move`, `setNativeShapeFn`。

### Lines 85-106
````cpp
/// Collect a set of patterns to unroll xegpu operations to a smaller shapes.
/// Users can control whether an operation to be unrolled or not, as well as
/// its target shape via `options` structure. (via setting filterConstraint
/// and nativeShape respectively, both of them are function refs taking `op` as
/// input).
/// An `op` is unrolled to the `targetShape` as follows, for each of its
/// operands:
///   1. the unrolled type `unrolledType` and number of unrolled instances
///   `numUnrolledInstances` are computed from the `targetShape`.
///   2. pack each operand. ExtractStridedSlice are created to break-up the
///   vector operands. And BuiltinUnrealizedCastOp are created to break-up
///    the TensorDesc operands.
///   3. the original op is cloned `numUnrolledInstances` times, once for each
///   result.
///   4. unpack the results. InsertStridedSlice are inserted for VectorType
///   result, and BuiltinUnrealizedCastOp are inserted for TensorDescType result
///   to re-assemble the slices into the original shape.
void populateXeGPUUnrollPatterns(RewritePatternSet &patterns,
                                 const UnrollOptions &options);

} // namespace xegpu
} // namespace mlir
````
- **EN**: This block groups callable interfaces such as `populateXeGPUUnrollPatterns`, indicating how `Transforms` is queried or updated.
- **CN**: 该代码块聚合了 `populateXeGPUUnrollPatterns` 等可调用接口，展示了如何查询或更新 `Transforms`。

### Lines 109-109
````cpp
#endif // MLIR_DIALECT_XEGPU_TRANSFORMS_TRANSFORMS_H
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

- mlir/IR/Builders.h
- mlir/IR/Operation.h
- mlir/Transforms/DialectConversion.h
- llvm/ADT/SmallVector.h
- llvm/Support/LogicalResult.h
