# Utils.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/Transform/Utils/Utils.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR Utils component. The leading comments describe it as: Printer hook for custom directive in assemblyFormat.
- **用途（CN）**: 声明 MLIR Utils 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15
````cpp
//===- Utils.h - Transform dialect utilities --------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_TRANSFORMS_UTILS_UTILS_H
#define MLIR_DIALECT_TRANSFORMS_UTILS_UTILS_H

#include "mlir/IR/OpImplementation.h"
#include "mlir/IR/Value.h"
#include "mlir/IR/ValueRange.h"
#include "mlir/Support/LLVM.h"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 17-66
````cpp
namespace mlir {
class OpAsmPrinter;

namespace transform {
class TransformState;

/// Printer hook for custom directive in assemblyFormat.
///
///   custom<PackedOrDynamicIndexList>($packed, type($packed), $values,
///       type($values), $integers)
///
/// where `values` are variadic Index values, `integers` is an `I64ArrayAttr`
/// and `packed` is a single transform dialect handle who's mapped payload ops
/// have a single Index result and represent the index list. Either `packed`
/// or the other two parameters may be specified.
///
/// This allows idiomatic printing of mixed value and integer attributes in a
/// list or with a single handle. E.g., `[%arg0 : !transform.any_op, 7, 42,
/// %arg42 : !transform.param<i64>]` or just `%h : !transform.any_op`.
void printPackedOrDynamicIndexList(OpAsmPrinter &printer, Operation *op,
                                   Value packed, Type packedType,
                                   OperandRange values, TypeRange valueTypes,
                                   DenseI64ArrayAttr integers);
inline void printPackedOrDynamicIndexList(OpAsmPrinter &printer, Operation *op,
                                          Value packed, OperandRange values,
                                          DenseI64ArrayAttr integers) {
  printPackedOrDynamicIndexList(printer, op, packed, Type(), values,
                                TypeRange{}, integers);
}

/// Parser hook for custom directive in assemblyFormat.
///
///   custom<PackedOrDynamicIndexList>($packed, type($packed), $values,
///       type($values), $integers)
///
/// See `printPackedOrDynamicIndexList` for details.
ParseResult parsePackedOrDynamicIndexList(
    OpAsmParser &parser, std::optional<OpAsmParser::UnresolvedOperand> &packed,
    Type &packedType, SmallVectorImpl<OpAsmParser::UnresolvedOperand> &values,
    SmallVectorImpl<Type> *valueTypes, DenseI64ArrayAttr &integers);
inline ParseResult parsePackedOrDynamicIndexList(
    OpAsmParser &parser, std::optional<OpAsmParser::UnresolvedOperand> &packed,
    SmallVectorImpl<OpAsmParser::UnresolvedOperand> &values,
    DenseI64ArrayAttr &integers) {
  Type packedType;
  return parsePackedOrDynamicIndexList(parser, packed, packedType, values,
                                       nullptr, integers);
}
} // namespace transform
} // namespace mlir
````
- **EN**: This C++ declaration introduces `OpAsmPrinter` and establishes part of the API surface for `Utils`. Representative entry points here include `printPackedOrDynamicIndexList`, `Type`, `parsePackedOrDynamicIndexList`.
- **CN**: 该 C++ 声明引入了 `OpAsmPrinter`，并构成 `Utils` API 表面的一部分。 这一段可见的代表性接口包括 `printPackedOrDynamicIndexList`, `Type`, `parsePackedOrDynamicIndexList`。

### Lines 67-67
````cpp
#endif // MLIR_DIALECT_TRANSFORMS_UTILS_UTILS_H
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: C++ declaration surface for MLIR infrastructure
  **CN**: MLIR 基础设施的 C++ 声明层
- **EN**: Type or attribute abstraction
  **CN**: 类型或属性抽象

## Dependencies / 依赖关系

- mlir/IR/OpImplementation.h
- mlir/IR/Value.h
- mlir/IR/ValueRange.h
- mlir/Support/LLVM.h
