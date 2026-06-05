# IRDLOps.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/IRDL/IR/IRDLOps.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file is licensed under the Apache License v2.0 with LLVM Exceptions.
  - **CN**: 实现 IRDL 方言与声明式方言定义支持 的核心 IR、操作、类型、属性或接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- IRDLOps.cpp - IRDL dialect -------------------------------*- C++ -*-===//
//
// This file is licensed under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 8-16
```cpp

#include "mlir/Dialect/IRDL/IR/IRDL.h"
#include "mlir/Dialect/IRDL/IRDLSymbols.h"
#include "mlir/IR/ValueRange.h"
#include <optional>

using namespace mlir;
using namespace mlir::irdl;

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/IRDL/IR/IRDL.h`, `mlir/Dialect/IRDL/IRDLSymbols.h`, `mlir/IR/ValueRange.h`, `optional`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/IRDL/IR/IRDL.h`, `mlir/Dialect/IRDL/IRDLSymbols.h`, `mlir/IR/ValueRange.h`, `optional`。

### Lines 17-30
```cpp
/// Maps given `args` to the index in the `valueToConstr`
static SmallVector<unsigned>
getConstraintIndicesForArgs(mlir::OperandRange args,
                            ArrayRef<Value> valueToConstr) {
  SmallVector<unsigned> constraints;
  for (Value arg : args) {
    for (auto [i, value] : enumerate(valueToConstr)) {
      if (value == arg) {
        constraints.push_back(i);
        break;
      }
    }
  }
  return constraints;
```
- **EN**: Implements logic around `getConstraintIndicesForArgs`, `enumerate`, `push_back`.
- **CN**: 围绕 `getConstraintIndicesForArgs`, `enumerate`, `push_back` 实现具体逻辑。

### Lines 31-40
```cpp
}

std::unique_ptr<Constraint> IsOp::getVerifier(
    ArrayRef<Value> valueToConstr,
    DenseMap<TypeOp, std::unique_ptr<DynamicTypeDefinition>> const &types,
    DenseMap<AttributeOp, std::unique_ptr<DynamicAttrDefinition>> const
        &attrs) {
  return std::make_unique<IsConstraint>(getExpectedAttr());
}

```
- **EN**: Implements logic around `getVerifier`, `make_unique`.
- **CN**: 围绕 `getVerifier`, `make_unique` 实现具体逻辑。

### Lines 41-47
```cpp
std::unique_ptr<Constraint> BaseOp::getVerifier(
    ArrayRef<Value> valueToConstr,
    DenseMap<TypeOp, std::unique_ptr<DynamicTypeDefinition>> const &types,
    DenseMap<AttributeOp, std::unique_ptr<DynamicAttrDefinition>> const
        &attrs) {
  MLIRContext *ctx = getContext();

```
- **EN**: Implements logic around `getVerifier`, `getContext`.
- **CN**: 围绕 `getVerifier`, `getContext` 实现具体逻辑。

### Lines 48-54
```cpp
  // Case where the input is a symbol reference.
  // This corresponds to the case where the base is an IRDL type or attribute.
  if (auto baseRef = getBaseRef()) {
    // The verifier for BaseOp guarantees it is within a dialect.
    Operation *defOp =
        irdl::lookupSymbolNearDialect(getOperation(), baseRef.value());

```
- **EN**: Implements logic around `getBaseRef`, `lookupSymbolNearDialect`; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `getBaseRef`, `lookupSymbolNearDialect` 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 55-62
```cpp
    // Type case.
    if (auto typeOp = dyn_cast<TypeOp>(defOp)) {
      DynamicTypeDefinition *typeDef = types.at(typeOp).get();
      auto name = StringAttr::get(ctx, typeDef->getDialect()->getNamespace() +
                                           "." + typeDef->getName().str());
      return std::make_unique<BaseTypeConstraint>(typeDef->getTypeID(), name);
    }

```
- **EN**: Implements logic around `dyn_cast`, `at`, `get`, `getName`, and 1 more symbols.
- **CN**: 围绕 `dyn_cast`, `at`, `get`, `getName`, and 1 more symbols 实现具体逻辑。

### Lines 63-70
```cpp
    // Attribute case.
    auto attrOp = cast<AttributeOp>(defOp);
    DynamicAttrDefinition *attrDef = attrs.at(attrOp).get();
    auto name = StringAttr::get(ctx, attrDef->getDialect()->getNamespace() +
                                         "." + attrDef->getName().str());
    return std::make_unique<BaseAttrConstraint>(attrDef->getTypeID(), name);
  }

```
- **EN**: Implements logic around `cast`, `at`, `get`, `getName`, and 1 more symbols.
- **CN**: 围绕 `cast`, `at`, `get`, `getName`, and 1 more symbols 实现具体逻辑。

### Lines 71-84
```cpp
  // Case where the input is string literal.
  // This corresponds to the case where the base is a registered type or
  // attribute.
  StringRef baseName = getBaseName().value();

  // Type case.
  if (baseName[0] == '!') {
    auto abstractType = AbstractType::lookup(baseName.drop_front(1), ctx);
    if (!abstractType) {
      emitError() << "no registered type with name " << baseName;
      return nullptr;
    }
    return std::make_unique<BaseTypeConstraint>(abstractType->get().getTypeID(),
                                                abstractType->get().getName());
```
- **EN**: Implements logic around `getBaseName`, `lookup`, `emitError`, `make_unique`, and 1 more symbols.
- **CN**: 围绕 `getBaseName`, `lookup`, `emitError`, `make_unique`, and 1 more symbols 实现具体逻辑。

### Lines 85-95
```cpp
  }

  auto abstractAttr = AbstractAttribute::lookup(baseName.drop_front(1), ctx);
  if (!abstractAttr) {
    emitError() << "no registered attribute with name " << baseName;
    return nullptr;
  }
  return std::make_unique<BaseAttrConstraint>(abstractAttr->get().getTypeID(),
                                              abstractAttr->get().getName());
}

```
- **EN**: Implements logic around `lookup`, `emitError`, `make_unique`, `get`.
- **CN**: 围绕 `lookup`, `emitError`, `make_unique`, `get` 实现具体逻辑。

### Lines 96-103
```cpp
std::unique_ptr<Constraint> ParametricOp::getVerifier(
    ArrayRef<Value> valueToConstr,
    DenseMap<TypeOp, std::unique_ptr<DynamicTypeDefinition>> const &types,
    DenseMap<AttributeOp, std::unique_ptr<DynamicAttrDefinition>> const
        &attrs) {
  SmallVector<unsigned> constraints =
      getConstraintIndicesForArgs(getArgs(), valueToConstr);

```
- **EN**: Implements logic around `getVerifier`, `getConstraintIndicesForArgs`.
- **CN**: 围绕 `getVerifier`, `getConstraintIndicesForArgs` 实现具体逻辑。

### Lines 104-112
```cpp
  // Symbol reference case for the base.
  // The verifier for ParametricOp guarantees it is within a dialect.
  SymbolRefAttr symRef = getBaseType();
  Operation *defOp = irdl::lookupSymbolNearDialect(getOperation(), symRef);
  if (!defOp) {
    emitError() << symRef << " does not refer to any existing symbol";
    return nullptr;
  }

```
- **EN**: Implements logic around `getBaseType`, `lookupSymbolNearDialect`, `emitError`; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `getBaseType`, `lookupSymbolNearDialect`, `emitError` 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 113-120
```cpp
  if (auto typeOp = dyn_cast<TypeOp>(defOp))
    return std::make_unique<DynParametricTypeConstraint>(types.at(typeOp).get(),
                                                         constraints);

  if (auto attrOp = dyn_cast<AttributeOp>(defOp))
    return std::make_unique<DynParametricAttrConstraint>(attrs.at(attrOp).get(),
                                                         constraints);

```
- **EN**: Implements logic around `dyn_cast`, `make_unique`.
- **CN**: 围绕 `dyn_cast`, `make_unique` 实现具体逻辑。

### Lines 121-133
```cpp
  llvm_unreachable("verifier should ensure that the referenced operation is "
                   "either a type or an attribute definition");
}

std::unique_ptr<Constraint> AnyOfOp::getVerifier(
    ArrayRef<Value> valueToConstr,
    DenseMap<TypeOp, std::unique_ptr<DynamicTypeDefinition>> const &types,
    DenseMap<AttributeOp, std::unique_ptr<DynamicAttrDefinition>> const
        &attrs) {
  return std::make_unique<AnyOfConstraint>(
      getConstraintIndicesForArgs(getArgs(), valueToConstr));
}

```
- **EN**: Implements logic around `llvm_unreachable`, `getVerifier`, `make_unique`, `getConstraintIndicesForArgs`.
- **CN**: 围绕 `llvm_unreachable`, `getVerifier`, `make_unique`, `getConstraintIndicesForArgs` 实现具体逻辑。

### Lines 134-142
```cpp
std::unique_ptr<Constraint> AllOfOp::getVerifier(
    ArrayRef<Value> valueToConstr,
    DenseMap<TypeOp, std::unique_ptr<DynamicTypeDefinition>> const &types,
    DenseMap<AttributeOp, std::unique_ptr<DynamicAttrDefinition>> const
        &attrs) {
  return std::make_unique<AllOfConstraint>(
      getConstraintIndicesForArgs(getArgs(), valueToConstr));
}

```
- **EN**: Implements logic around `getVerifier`, `make_unique`, `getConstraintIndicesForArgs`.
- **CN**: 围绕 `getVerifier`, `make_unique`, `getConstraintIndicesForArgs` 实现具体逻辑。

### Lines 143-150
```cpp
std::unique_ptr<Constraint> AnyOp::getVerifier(
    ArrayRef<Value> valueToConstr,
    DenseMap<TypeOp, std::unique_ptr<DynamicTypeDefinition>> const &types,
    DenseMap<AttributeOp, std::unique_ptr<DynamicAttrDefinition>> const
        &attrs) {
  return std::make_unique<AnyAttributeConstraint>();
}

```
- **EN**: Implements logic around `getVerifier`, `make_unique`.
- **CN**: 围绕 `getVerifier`, `make_unique` 实现具体逻辑。

### Lines 151-161
```cpp
std::unique_ptr<RegionConstraint> RegionOp::getVerifier(
    ArrayRef<Value> valueToConstr,
    DenseMap<TypeOp, std::unique_ptr<DynamicTypeDefinition>> const &types,
    DenseMap<AttributeOp, std::unique_ptr<DynamicAttrDefinition>> const
        &attrs) {
  return std::make_unique<RegionConstraint>(
      getConstrainedArguments() ? std::optional{getConstraintIndicesForArgs(
                                      getEntryBlockArgs(), valueToConstr)}
                                : std::nullopt,
      getNumberOfBlocks());
}
```
- **EN**: Implements logic around `getVerifier`, `make_unique`, `getConstrainedArguments`, `getEntryBlockArgs`, and 1 more symbols.
- **CN**: 围绕 `getVerifier`, `make_unique`, `getConstrainedArguments`, `getEntryBlockArgs`, and 1 more symbols 实现具体逻辑。

## Key Concepts / 关键概念

- **Dialect IR modeling / 方言 IR 建模**:
  - **EN**: Defines operations, attributes, types, verifiers, parsers, and printers for a dialect.
  - **CN**: 定义方言的操作、属性、类型、验证器、解析器与打印器。
- **Declarative dialect definitions / 声明式方言定义**:
  - **EN**: Encodes dialect structure and constraints as IR that can define other IR.
  - **CN**: 把方言结构与约束编码成可定义其他 IR 的 IR。
- **Type/attribute storage / 类型/属性存储**:
  - **EN**: Uses uniqued storage and parser/printer hooks for extensible compile-time metadata.
  - **CN**: 使用唯一化存储以及解析/打印钩子来表示可扩展的编译期元数据。
- **Region-based control flow / 基于 Region 的控制流**:
  - **EN**: Represents nested blocks and successors as first-class IR structure.
  - **CN**: 把嵌套 block 与后继边表示为一等 IR 结构。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/IRDL/IR/IRDL.h`, `mlir/Dialect/IRDL/IRDLSymbols.h`, `mlir/IR/ValueRange.h`
- **Standard-library headers / 标准库头文件**: `<optional>`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (2), MLIR IR core abstractions / MLIR IR 核心抽象 (1)
