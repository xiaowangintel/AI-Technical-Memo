# LayoutUtils.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/SPIRV/Utils/LayoutUtils.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR LayoutUtils component. The leading comments describe it as: This file defines utilities used to get alignment and layout information for.
- **用途（CN）**: 声明 MLIR LayoutUtils 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17
````cpp
//===-- LayoutUtils.h - Vulkan Layout Util functions ------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines utilities used to get alignment and layout information for
// types in SPIR-V dialect.
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_SPIRV_UTILS_LAYOUTUTILS_H_
#define MLIR_DIALECT_SPIRV_UTILS_LAYOUTUTILS_H_

#include <cstdint>
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 19-66
````cpp
namespace mlir {
class Type;
class VectorType;

namespace spirv {
class ArrayType;
class RuntimeArrayType;
class StructType;
class MatrixType;
} // namespace spirv

/// According to the Vulkan spec "15.6.4. Offset and Stride Assignment":
/// "There are different alignment requirements depending on the specific
/// resources and on the features enabled on the device."
///
/// There are 3 types of alignment: scalar, base, extended.
/// See the spec for details.
///
/// Note: Even if scalar alignment is supported, it is generally more
/// performant to use the base alignment. So here the calculation is based on
/// base alignment.
///
/// The memory layout must obey the following rules:
/// 1. The Offset decoration of any member must be a multiple of its alignment.
/// 2. Any ArrayStride or MatrixStride decoration must be a multiple of the
/// alignment of the array or matrix as defined above.
///
/// According to the SPIR-V spec:
/// "The ArrayStride, MatrixStride, and Offset decorations must be large
/// enough to hold the size of the objects they affect (that is, specifying
/// overlap is invalid)."
class VulkanLayoutUtils {
public:
  using Size = uint64_t;

  /// Returns a new StructType with layout decoration.
  static spirv::StructType decorateType(spirv::StructType structType);

  /// Checks whether a type is legal in terms of Vulkan layout info
  /// decoration. A type is dynamically illegal if it's a composite type in the
  /// StorageBuffer, PhysicalStorageBuffer, Uniform, and PushConstant Storage
  /// Classes without layout information.
  static bool isLegalType(Type type);

private:
  /// Returns a new type with layout decoration. Assigns the type size in bytes
  /// to the `size`. Assigns the type alignment in bytes to the `alignment`.
  static Type decorateType(Type type, Size &size, Size &alignment);
````
- **EN**: This C++ declaration introduces `Type` and establishes part of the API surface for `LayoutUtils`. Representative entry points here include `decorateType`, `isLegalType`.
- **CN**: 该 C++ 声明引入了 `Type`，并构成 `LayoutUtils` API 表面的一部分。 这一段可见的代表性接口包括 `decorateType`, `isLegalType`。

### Lines 67-80
````cpp
  static Type decorateType(VectorType vectorType, Size &size, Size &alignment);
  static Type decorateType(spirv::ArrayType arrayType, Size &size,
                           Size &alignment);
  static Type decorateType(spirv::MatrixType matrixType, Size &size,
                           Size &alignment);
  static Type decorateType(spirv::RuntimeArrayType arrayType, Size &alignment);
  static spirv::StructType decorateType(spirv::StructType structType,
                                        Size &size, Size &alignment);

  /// Calculates the alignment for the given scalar type.
  static Size getScalarTypeAlignment(Type scalarType);
};

} // namespace mlir
````
- **EN**: This block groups callable interfaces such as `decorateType`, `getScalarTypeAlignment`, indicating how `LayoutUtils` is queried or updated.
- **CN**: 该代码块聚合了 `decorateType`, `getScalarTypeAlignment` 等可调用接口，展示了如何查询或更新 `LayoutUtils`。

### Lines 83-83
````cpp
#endif // MLIR_DIALECT_SPIRV_UTILS_LAYOUTUTILS_H_
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: C++ declaration surface for MLIR infrastructure
  **CN**: MLIR 基础设施的 C++ 声明层
- **EN**: Type or attribute abstraction
  **CN**: 类型或属性抽象

## Dependencies / 依赖关系

- Primarily relies on nearby MLIR declarations surrounding LayoutUtils.h.
