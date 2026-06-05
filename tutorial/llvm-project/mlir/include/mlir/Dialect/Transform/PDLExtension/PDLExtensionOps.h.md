# PDLExtensionOps.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/Transform/PDLExtension/PDLExtensionOps.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR PDLExtensionOps component. The leading comments describe it as: PDL constraint callbacks that can be used by the PDL extension of the.
- **用途（CN）**: 声明 MLIR PDLExtensionOps 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
````cpp
//===- PDLExtensionOps.h - PDL extension for Transform dialect --*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_TRANSFORM_PDLEXTENSION_PDLEXTENSIONOPS_H
#define MLIR_DIALECT_TRANSFORM_PDLEXTENSION_PDLEXTENSIONOPS_H

#include "mlir/Bytecode/BytecodeOpInterface.h"
#include "mlir/Dialect/Transform/IR/TransformDialect.h"
#include "mlir/Dialect/Transform/Interfaces/TransformInterfaces.h"
#include "mlir/IR/OpDefinition.h"
#include "mlir/IR/OpImplementation.h"
#include "mlir/IR/SymbolTable.h"
#include "mlir/Interfaces/SideEffectInterfaces.h"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 20-21
````cpp
#define GET_OP_CLASSES
#include "mlir/Dialect/Transform/PDLExtension/PDLExtensionOps.h.inc"
````
- **EN**: This macro block selects a generated declaration fragment before the corresponding `.inc` file is included.
- **CN**: 该宏代码块在包含对应的 `.inc` 文件之前，选择要展开的生成式声明片段。

### Lines 23-50
````cpp
namespace mlir {
namespace transform {
/// PDL constraint callbacks that can be used by the PDL extension of the
/// Transform dialect. These are owned by the Transform dialect and can be
/// populated by extensions.
class PDLMatchHooks : public TransformDialectData<PDLMatchHooks> {
public:
  PDLMatchHooks(MLIRContext *ctx) : TransformDialectData(ctx) {}

  /// Takes ownership of the named PDL constraint function from the given
  /// map and makes them available for use by the operations in the dialect.
  void
  mergeInPDLMatchHooks(llvm::StringMap<PDLConstraintFunction> &&constraintFns);

  /// Returns the named PDL constraint functions available in the dialect
  /// as a map from their name to the function.
  const llvm::StringMap<::mlir::PDLConstraintFunction> &
  getPDLConstraintHooks() const;

private:
  /// A container for PDL constraint function that can be used by
  /// operations in this dialect.
  PDLPatternModule pdlMatchHooks;
};
} // namespace transform
} // namespace mlir

MLIR_DECLARE_EXPLICIT_TYPE_ID(mlir::transform::PDLMatchHooks)
````
- **EN**: This C++ declaration introduces `PDLMatchHooks` and establishes part of the API surface for `PDLExtensionOps`. Representative entry points here include `PDLMatchHooks`, `TransformDialectData`, `mergeInPDLMatchHooks`, `getPDLConstraintHooks`.
- **CN**: 该 C++ 声明引入了 `PDLMatchHooks`，并构成 `PDLExtensionOps` API 表面的一部分。 这一段可见的代表性接口包括 `PDLMatchHooks`, `TransformDialectData`, `mergeInPDLMatchHooks`, `getPDLConstraintHooks`。

### Lines 52-52
````cpp
#endif // MLIR_DIALECT_TRANSFORM_PDLEXTENSION_PDLEXTENSIONOPS_H
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: C++ declaration surface for MLIR infrastructure
  **CN**: MLIR 基础设施的 C++ 声明层

## Dependencies / 依赖关系

- mlir/Bytecode/BytecodeOpInterface.h
- mlir/Dialect/Transform/IR/TransformDialect.h
- mlir/Dialect/Transform/Interfaces/TransformInterfaces.h
- mlir/IR/OpDefinition.h
- mlir/IR/OpImplementation.h
- mlir/IR/SymbolTable.h
- mlir/Interfaces/SideEffectInterfaces.h
- mlir/Dialect/Transform/PDLExtension/PDLExtensionOps.h.inc
- PDLMatchHooks inherits from public TransformDialectData<PDLMatchHooks>
- PDLMatchHooks builds on public TransformDialectData<PDLMatchHooks>
