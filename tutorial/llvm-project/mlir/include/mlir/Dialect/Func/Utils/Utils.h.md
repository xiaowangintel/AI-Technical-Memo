# Utils.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/Func/Utils/Utils.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This header file defines prototypes for various transformation utilities for the Func dialect. These are not passes by themselves but are used either by passes, optimization sequences, or in turn by other transformation utilities.
  - **CN**: 该文件位于 `mlir/include/mlir/Dialect/Func/Utils`，围绕 Func 方言公开 `Utils` 相关的接口、规则或生成式定义。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- Utils.h - General Func transformation utilities ----*- C++ -*-------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-14
```cpp
//
// This header file defines prototypes for various transformation utilities for
// the Func dialect. These are not passes by themselves but are used
// either by passes, optimization sequences, or in turn by other transformation
// utilities.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 15-22
```cpp

#ifndef MLIR_DIALECT_FUNC_UTILS_H
#define MLIR_DIALECT_FUNC_UTILS_H

#include "mlir/IR/PatternMatch.h"
#include "llvm/ADT/ArrayRef.h"
#include <string>

```
- **EN**: Pulls in the headers needed by this declaration unit, including `mlir/IR/PatternMatch.h`, `llvm/ADT/ArrayRef.h`, `string`.
- **CN**: 引入该声明单元所需的头文件，其中包括 `mlir/IR/PatternMatch.h`, `llvm/ADT/ArrayRef.h`, `string`。

### Lines 23-27
```cpp
namespace mlir {

class ModuleOp;

namespace func {
```
- **EN**: Introduces declarations for `mlir`, `ModuleOp`, `func`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `mlir`, `ModuleOp`, `func` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 28-32
```cpp

class FuncOp;
class CallOp;

/// Creates a new function operation with the same name as the original
```
- **EN**: Introduces declarations for `FuncOp`, `CallOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `FuncOp`, `CallOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 33-37
```cpp
/// function operation, but with the arguments mapped according to
/// the `oldArgToNewArg` and `oldResToNewRes`.
/// The `funcOp` operation must have exactly one block.
/// Returns the new function operation or failure if `funcOp` doesn't
/// have exactly one block.
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 38-43
```cpp
/// Note: the method asserts that the `oldArgToNewArg` and `oldResToNewRes`
/// maps the whole function arguments and results.
mlir::FailureOr<mlir::func::FuncOp> replaceFuncWithNewMapping(
    mlir::RewriterBase &rewriter, mlir::func::FuncOp funcOp,
    ArrayRef<int> oldArgIdxToNewArgIdx, ArrayRef<int> oldResIdxToNewResIdx);
/// Creates a new call operation with the values as the original
```
- **EN**: Declares APIs or declarative rules around `replaceFuncWithNewMapping`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 声明与 `replaceFuncWithNewMapping` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作。

### Lines 44-51
```cpp
/// call operation, but with the arguments mapped according to
/// the `oldArgToNewArg` and `oldResToNewRes`.
/// Note: the method asserts that the `oldArgToNewArg` and `oldResToNewRes`
/// maps the whole call operation arguments and results.
mlir::func::CallOp replaceCallOpWithNewMapping(
    mlir::RewriterBase &rewriter, mlir::func::CallOp callOp,
    ArrayRef<int> oldArgIdxToNewArgIdx, ArrayRef<int> oldResIdxToNewResIdx);

```
- **EN**: Declares APIs or declarative rules around `replaceCallOpWithNewMapping`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 声明与 `replaceCallOpWithNewMapping` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作。

### Lines 52-56
```cpp
/// This utility function examines all call operations within the given
/// `moduleOp` that target the specified `funcOp`. It identifies duplicate
/// operands in the call operations, creates mappings to deduplicate them, and
/// then applies the transformation to both the function and its call sites. For
/// now, it only supports one call operation for the function operation. The
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 57-62
```cpp
/// function returns a pair containing the new funcOp and the new callOp. Note:
/// after the transformation, the original funcOp and callOp will be erased.
mlir::FailureOr<std::pair<mlir::func::FuncOp, mlir::func::CallOp>>
deduplicateArgsOfFuncOp(mlir::RewriterBase &rewriter, mlir::func::FuncOp funcOp,
                        mlir::ModuleOp moduleOp);

```
- **EN**: Declares APIs or declarative rules around `deduplicateArgsOfFuncOp`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 声明与 `deduplicateArgsOfFuncOp` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作。

### Lines 63-69
```cpp
/// Look up a FuncOp with signature `resultTypes`(`paramTypes`)` and name
/// `name`. Return a failure if the FuncOp is found but with a different
/// signature.
FailureOr<FuncOp> lookupFnDecl(SymbolOpInterface symTable, StringRef name,
                               FunctionType funcT,
                               SymbolTableCollection *symbolTables = nullptr);

```
- **EN**: Declares APIs or declarative rules around `lookupFnDecl`.
- **CN**: 声明与 `lookupFnDecl` 相关的 API 或声明式规则。

### Lines 70-76
```cpp
/// Create a FuncOp decl and insert it into `symTable` operation. If
/// `symbolTables` is provided, then the decl will be inserted into the
/// SymbolTableCollection.
FuncOp createFnDecl(OpBuilder &b, SymbolOpInterface symTable, StringRef name,
                    FunctionType funcT, bool setPrivate,
                    SymbolTableCollection *symbolTables = nullptr);

```
- **EN**: Declares APIs or declarative rules around `createFnDecl`.
- **CN**: 声明与 `createFnDecl` 相关的 API 或声明式规则。

### Lines 77-85
```cpp
/// Helper function to look up or create the symbol for a runtime library
/// function with the given parameter types. Returns an int64_t, unless a
/// different result type is specified.
FailureOr<FuncOp>
lookupOrCreateFnDecl(OpBuilder &b, SymbolOpInterface symTable, StringRef name,
                     TypeRange paramTypes,
                     SymbolTableCollection *symbolTables = nullptr,
                     Type resultType = {});

```
- **EN**: Implements logic around `lookupOrCreateFnDecl`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `lookupOrCreateFnDecl` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 86-89
```cpp
} // namespace func
} // namespace mlir

#endif // MLIR_DIALECT_FUNC_UTILS_H
```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

## Key Concepts / 关键概念

- **Dialect declarations / 方言声明**:
  - **EN**: Defines the public include-surface for a dialect, exposing operations, attributes, types, or interfaces.
  - **CN**: 定义方言的公共头文件表面，暴露操作、属性、类型或接口。
- **Dialect utilities / 方言工具**:
  - **EN**: Packages helper declarations that keep larger dialect components factored and reusable.
  - **CN**: 封装辅助声明，使较大的方言组件保持解耦并可复用。
- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Models reusable capabilities that can be queried across dialect boundaries.
  - **CN**: 建模可跨方言查询的可复用能力。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/IR/PatternMatch.h`, `llvm/ADT/ArrayRef.h`
- **Standard-library headers / 标准库头文件**: `<string>`
- **Subsystem categories / 子系统类别**: MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (1), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (1)
