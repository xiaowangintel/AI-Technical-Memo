# QuantUtils.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/Tosa/Utils/QuantUtils.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR QuantUtils component. The leading comments describe it as: Function declarations for TOSA numerical support functions and quantization.
- **用途（CN）**: 声明 MLIR QuantUtils 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
````cpp
//===-- QuantUtils.h - TOSA numerical support declarations ------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Function declarations for TOSA numerical support functions and quantization
// attribute builders
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_TOSA_UTILS_QUANTUTILS_H
#define MLIR_DIALECT_TOSA_UTILS_QUANTUTILS_H

#include "mlir/Dialect/Tosa/IR/TosaOps.h"

#include "mlir/Dialect/Quant/Utils/FakeQuantSupport.h"
#include "mlir/Dialect/Quant/Utils/UniformSupport.h"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 22-23
````cpp
namespace mlir {
namespace tosa {
````
- **EN**: This C++ declaration introduces `mlir` and establishes part of the API surface for `QuantUtils`.
- **CN**: 该 C++ 声明引入了 `mlir`，并构成 `QuantUtils` API 表面的一部分。

### Lines 25-72
````cpp
//===----------------------------------------------------------------------===//
// Utility functions to support quantization handling in Tosa.
//===----------------------------------------------------------------------===//

/// From a scale value, computes multiplier and shift values
/// for 16 or 32-bit scale widths.
bool computeMultiplierAndShift(double scale, int32_t &multiplier,
                               int32_t &shift, int32_t scaleWidth);

// Return a const value for array of IntType vec
template <typename IntType>
Value getConstTensorInt(OpBuilder &builder, Location loc,
                        ArrayRef<IntType> vec) {
  static_assert(
      std::is_same<IntType, int8_t>::value ||
          std::is_same<IntType, int16_t>::value ||
          std::is_same<IntType, int32_t>::value,
      "getConstTensorInt only supports int8_t, int16_t, and int32_t types.");

  int64_t count = vec.size();
  assert(count > 0 && "Vector must not be empty");
  auto element_type = builder.getIntegerType(sizeof(IntType) * 8);
  mlir::RankedTensorType const_type =
      RankedTensorType::get({count}, element_type);
  mlir::DenseElementsAttr const_attr = DenseElementsAttr::get(const_type, vec);
  auto const_op = tosa::ConstOp::create(builder, loc, const_type, const_attr);
  return const_op.getResult();
}

//// Builds ConvOpQuantizationAttr from input and weight.
ConvOpQuantizationAttr buildConvOpQuantizationAttr(OpBuilder &builder,
                                                   Value input, Value weight);

std::pair<Value, Value> createZPsAsConst(OpBuilder &builder, Value input,
                                         Value weight);

//// Builds MatMulOpQuantizationAttr for MatMul operations from A and B.
MatMulOpQuantizationAttr buildMatMulOpQuantizationAttr(OpBuilder &builder,
                                                       Value a, Value b);

//// Builds UnaryOpQuantizationAttr for unary operations from input values.
UnaryOpQuantizationAttr buildUnaryOpQuantizationAttr(OpBuilder &builder,
                                                     Value input,
                                                     Type outputRawType);

//// Builds PadOpQuantizationAttr for pad operations from input values.
PadOpQuantizationAttr buildPadOpQuantizationAttr(OpBuilder &builder,
                                                 Value input);
````
- **EN**: This block groups callable interfaces such as `computeMultiplierAndShift`, `getConstTensorInt`, `static_assert`, `size`, indicating how `QuantUtils` is queried or updated.
- **CN**: 该代码块聚合了 `computeMultiplierAndShift`, `getConstTensorInt`, `static_assert`, `size` 等可调用接口，展示了如何查询或更新 `QuantUtils`。

### Lines 73-93
````cpp
//// construct ConvOp output type with correct bitwidth based on input/weight
/// width.
Type buildConvOpResultTypeInfo(OpBuilder &builder, Type outputType, Value input,
                               Value weight);

/// Builds Tosa quantization attributes from min/max values.
Type buildQTypeFromMinMax(OpBuilder builder, Type inputDType, Attribute minAttr,
                          Attribute maxAttr, IntegerAttr quantBits,
                          int filterQuantDim, bool isSigned,
                          BoolAttr narrowRange);

/// Builds Tosa quantization attributes from min/max values.
TypeAttr buildQTypeAttrFromMinMax(OpBuilder builder, Type inputDType,
                                  Attribute minAttr, Attribute maxAttr,
                                  IntegerAttr quantBits, int filterQuantDim,
                                  bool isSigned, BoolAttr narrowRange);

Type getStorageElementTypeFromQuantized(quant::QuantizedType quantizedType);

} // namespace tosa
} // namespace mlir
````
- **EN**: This block groups callable interfaces such as `buildConvOpResultTypeInfo`, `buildQTypeFromMinMax`, `buildQTypeAttrFromMinMax`, `getStorageElementTypeFromQuantized`, indicating how `QuantUtils` is queried or updated.
- **CN**: 该代码块聚合了 `buildConvOpResultTypeInfo`, `buildQTypeFromMinMax`, `buildQTypeAttrFromMinMax`, `getStorageElementTypeFromQuantized` 等可调用接口，展示了如何查询或更新 `QuantUtils`。

### Lines 96-96
````cpp
#endif // MLIR_DIALECT_TOSA_UTILS_QUANTUTILS_H
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

- mlir/Dialect/Tosa/IR/TosaOps.h
- mlir/Dialect/Quant/Utils/FakeQuantSupport.h
- mlir/Dialect/Quant/Utils/UniformSupport.h
