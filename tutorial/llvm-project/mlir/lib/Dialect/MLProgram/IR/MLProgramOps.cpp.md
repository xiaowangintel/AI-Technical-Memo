# MLProgramOps.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/MLProgram/IR/MLProgramOps.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements MLProgram dialect support for dialect IR definitions such as operations, attributes, types, and parsers/printers, centered on `MLProgramOps`.
  - **CN**: 实现 MLProgram 方言中围绕 `MLProgramOps` 的方言 IR 定义，如操作、属性、类型以及解析/打印逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===- MLProgramOps.cpp - MLProgram dialect ops implementation ------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/MLProgram/IR/MLProgram.h"
#include "mlir/IR/Builders.h"
#include "mlir/Interfaces/FunctionImplementation.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/MLProgram/IR/MLProgram.h`, `mlir/IR/Builders.h`, `mlir/Interfaces/FunctionImplementation.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/MLProgram/IR/MLProgram.h`, `mlir/IR/Builders.h`, `mlir/Interfaces/FunctionImplementation.h`。

### Lines 13-22
```cpp
using namespace mlir;
using namespace mlir::ml_program;

//===----------------------------------------------------------------------===//
// Custom asm helpers
//===----------------------------------------------------------------------===//

/// Parse and print an ordering clause for a variadic of consuming tokens
/// and an producing token.
///
```
- **EN**: Introduces declarations for `mlir`, `mlir::ml_program`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `mlir`, `mlir::ml_program` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 23-36
```cpp
/// Syntax:
///   ordering(%0, %1 -> !ml_program.token)
///   ordering(() -> !ml_program.token)
///
/// If both the consuming and producing token are not present on the op, then
/// the clause prints nothing.
static ParseResult parseTokenOrdering(
    OpAsmParser &parser,
    SmallVectorImpl<OpAsmParser::UnresolvedOperand> &consumeTokens,
    Type &produceTokenType) {
  if (failed(parser.parseOptionalKeyword("ordering")) ||
      failed(parser.parseLParen()))
    return success();

```
- **EN**: Implements logic around `parseTokenOrdering`, `failed`, `success`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `parseTokenOrdering`, `failed`, `success` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 37-47
```cpp
  // Parse consuming token list. If there are no consuming tokens, the
  // '()' null list represents this.
  if (succeeded(parser.parseOptionalLParen())) {
    if (failed(parser.parseRParen()))
      return failure();
  } else {
    if (failed(parser.parseOperandList(consumeTokens,
                                       /*requiredOperandCount=*/-1)))
      return failure();
  }

```
- **EN**: Implements logic around `succeeded`, `failed`, `failure`; this block handles textual assembly parsing or printing concerns.
- **CN**: 围绕 `succeeded`, `failed`, `failure` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题。

### Lines 48-59
```cpp
  // Parse producer token.
  if (failed(parser.parseArrow()))
    return failure();
  if (failed(parser.parseType(produceTokenType)))
    return failure();

  if (failed(parser.parseRParen()))
    return failure();

  return success();
}

```
- **EN**: Implements logic around `failed`, `failure`, `success`; this block handles textual assembly parsing or printing concerns.
- **CN**: 围绕 `failed`, `failure`, `success` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题。

### Lines 60-77
```cpp
static void printTokenOrdering(OpAsmPrinter &p, Operation *op,
                               OperandRange consumeTokens,
                               Type produceTokenType) {
  if (consumeTokens.empty() && !produceTokenType)
    return;

  p << " ordering(";
  if (consumeTokens.empty())
    p << "()";
  else
    p.printOperands(consumeTokens);
  if (produceTokenType) {
    p << " -> ";
    p.printType(produceTokenType);
  }
  p << ")";
}

```
- **EN**: Implements logic around `printTokenOrdering`, `empty`, `ordering`, `printOperands`, and 1 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `printTokenOrdering`, `empty`, `ordering`, `printOperands`, and 1 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 78-92
```cpp
/// some.op custom<TypeOrAttr>($type, $attr)
///
/// Uninitialized:
///   some.op : tensor<3xi32>
/// Initialized to narrower type than op:
///   some.op (dense<0> : tensor<3xi32>) : tensor<?xi32>
static ParseResult parseTypedInitialValue(OpAsmParser &parser,
                                          TypeAttr &typeAttr, Attribute &attr) {
  if (succeeded(parser.parseOptionalLParen())) {
    if (failed(parser.parseAttribute(attr)))
      return failure();
    if (failed(parser.parseRParen()))
      return failure();
  }

```
- **EN**: Implements logic around `parseTypedInitialValue`, `succeeded`, `failed`, `failure`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `parseTypedInitialValue`, `succeeded`, `failed`, `failure` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 93-107
```cpp
  Type type;
  if (failed(parser.parseColonType(type)))
    return failure();
  typeAttr = TypeAttr::get(type);
  return success();
}

static void printTypedInitialValue(OpAsmPrinter &p, Operation *op,
                                   TypeAttr type, Attribute attr) {
  if (attr) {
    p << "(";
    p.printAttribute(attr);
    p << ")";
  }

```
- **EN**: Implements logic around `failed`, `failure`, `get`, `success`, and 2 more symbols; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `failed`, `failure`, `get`, `success`, and 2 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 108-127
```cpp
  p << " : ";
  p.printAttribute(type);
}

/// some.op custom<SymbolVisibility>($sym_visibility) $sym_name
/// ->
/// some.op public @foo
/// some.op private @foo
static ParseResult parseSymbolVisibility(OpAsmParser &parser,
                                         StringAttr &symVisibilityAttr) {
  StringRef symVisibility;
  (void)parser.parseOptionalKeyword(&symVisibility,
                                    {"public", "private", "nested"});
  if (symVisibility.empty())
    return parser.emitError(parser.getCurrentLocation())
           << "expected 'public', 'private', or 'nested'";
  if (!symVisibility.empty())
    symVisibilityAttr = parser.getBuilder().getStringAttr(symVisibility);
  return success();
}
```
- **EN**: Implements logic around `printAttribute`, `parseSymbolVisibility`, `parseOptionalKeyword`, `empty`, and 3 more symbols; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `printAttribute`, `parseSymbolVisibility`, `parseOptionalKeyword`, `empty`, and 3 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 128-137
```cpp

static void printSymbolVisibility(OpAsmPrinter &p, Operation *op,
                                  StringAttr symVisibilityAttr) {
  if (!symVisibilityAttr)
    p << "public";
  else
    p << symVisibilityAttr.getValue();
}

//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `printSymbolVisibility`, `getValue`.
- **CN**: 围绕 `printSymbolVisibility`, `getValue` 实现具体逻辑。

### Lines 138-147
```cpp
// TableGen'd op method definitions
//===----------------------------------------------------------------------===//

#define GET_OP_CLASSES
#include "mlir/Dialect/MLProgram/IR/MLProgramOps.cpp.inc"

//===----------------------------------------------------------------------===//
// FuncOp
//===----------------------------------------------------------------------===//

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/MLProgram/IR/MLProgramOps.cpp.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/MLProgram/IR/MLProgramOps.cpp.inc`。

### Lines 148-159
```cpp
ParseResult FuncOp::parse(OpAsmParser &parser, OperationState &result) {
  auto buildFuncType =
      [](Builder &builder, ArrayRef<Type> argTypes, ArrayRef<Type> results,
         function_interface_impl::VariadicFlag,
         std::string &) { return builder.getFunctionType(argTypes, results); };

  return function_interface_impl::parseFunctionOp(
      parser, result, /*allowVariadic=*/false,
      getFunctionTypeAttrName(result.name), buildFuncType,
      getArgAttrsAttrName(result.name), getResAttrsAttrName(result.name));
}

```
- **EN**: Implements logic around `parse`, `getFunctionType`, `parseFunctionOp`, `getFunctionTypeAttrName`, and 1 more symbols; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `parse`, `getFunctionType`, `parseFunctionOp`, `getFunctionTypeAttrName`, and 1 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 160-169
```cpp
void FuncOp::print(OpAsmPrinter &p) {
  function_interface_impl::printFunctionOp(
      p, *this, /*isVariadic=*/false, getFunctionTypeAttrName(),
      getArgAttrsAttrName(), getResAttrsAttrName());
}

//===----------------------------------------------------------------------===//
// GlobalOp
//===----------------------------------------------------------------------===//

```
- **EN**: Implements logic around `print`, `printFunctionOp`, `getFunctionTypeAttrName`, `getArgAttrsAttrName`; this block handles textual assembly parsing or printing concerns.
- **CN**: 围绕 `print`, `printFunctionOp`, `getFunctionTypeAttrName`, `getArgAttrsAttrName` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题。

### Lines 170-179
```cpp
LogicalResult GlobalOp::verify() {
  if (!getIsMutable() && !getValue())
    return emitOpError() << "immutable global must have an initial value";
  return success();
}

//===----------------------------------------------------------------------===//
// GlobalLoadOp
//===----------------------------------------------------------------------===//

```
- **EN**: Implements logic around `verify`, `getIsMutable`, `emitOpError`, `success`; this block checks operation invariants or verification rules.
- **CN**: 围绕 `verify`, `getIsMutable`, `emitOpError`, `success` 实现具体逻辑；该代码块检查操作不变式或验证规则。

### Lines 180-190
```cpp
GlobalOp GlobalLoadOp::getGlobalOp(SymbolTableCollection &symbolTable) {
  for (auto *parent = getOperation()->getParentOp(); parent;
       parent = parent->getParentOp()) {
    if (auto nearest = symbolTable.lookupNearestSymbolFrom<GlobalOp>(
            parent, getGlobalAttr())) {
      return nearest;
    }
  }
  return {};
}

```
- **EN**: Implements logic around `getGlobalOp`, `getOperation`, `getParentOp`, `lookupNearestSymbolFrom`, and 1 more symbols.
- **CN**: 围绕 `getGlobalOp`, `getOperation`, `getParentOp`, `lookupNearestSymbolFrom`, and 1 more symbols 实现具体逻辑。

### Lines 191-202
```cpp
LogicalResult
GlobalLoadOp::verifySymbolUses(SymbolTableCollection &symbolTable) {
  GlobalOp referrent = getGlobalOp(symbolTable);
  if (!referrent)
    return emitOpError() << "undefined global: " << getGlobal();

  if (referrent.getType() != getResult().getType()) {
    return emitOpError() << "cannot load from global typed "
                         << referrent.getType() << " as "
                         << getResult().getType();
  }

```
- **EN**: Implements logic around `verifySymbolUses`, `getGlobalOp`, `emitOpError`, `getType`, and 1 more symbols; this block checks operation invariants or verification rules.
- **CN**: 围绕 `verifySymbolUses`, `getGlobalOp`, `emitOpError`, `getType`, and 1 more symbols 实现具体逻辑；该代码块检查操作不变式或验证规则。

### Lines 203-214
```cpp
  return success();
}

//===----------------------------------------------------------------------===//
// GlobalLoadConstOp
//===----------------------------------------------------------------------===//

GlobalOp GlobalLoadConstOp::getGlobalOp(SymbolTableCollection &symbolTable) {
  return symbolTable.lookupNearestSymbolFrom<GlobalOp>(
      getOperation()->getParentOp(), getGlobalAttr());
}

```
- **EN**: Implements logic around `success`, `getGlobalOp`, `lookupNearestSymbolFrom`, `getOperation`.
- **CN**: 围绕 `success`, `getGlobalOp`, `lookupNearestSymbolFrom`, `getOperation` 实现具体逻辑。

### Lines 215-224
```cpp
LogicalResult
GlobalLoadConstOp::verifySymbolUses(SymbolTableCollection &symbolTable) {
  GlobalOp referrent = getGlobalOp(symbolTable);
  if (!referrent)
    return emitOpError() << "undefined global: " << getGlobal();

  if (referrent.getIsMutable())
    return emitOpError() << "cannot load as const from mutable global "
                         << getGlobal();

```
- **EN**: Implements logic around `verifySymbolUses`, `getGlobalOp`, `emitOpError`, `getIsMutable`, and 1 more symbols; this block checks operation invariants or verification rules.
- **CN**: 围绕 `verifySymbolUses`, `getGlobalOp`, `emitOpError`, `getIsMutable`, and 1 more symbols 实现具体逻辑；该代码块检查操作不变式或验证规则。

### Lines 225-235
```cpp
  if (referrent.getType() != getResult().getType())
    return emitOpError() << "cannot load from global typed "
                         << referrent.getType() << " as "
                         << getResult().getType();

  return success();
}

//===----------------------------------------------------------------------===//
// GlobalLoadGraphOp
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `getType`, `emitOpError`, `getResult`, `success`; this block checks operation invariants or verification rules.
- **CN**: 围绕 `getType`, `emitOpError`, `getResult`, `success` 实现具体逻辑；该代码块检查操作不变式或验证规则。

### Lines 236-247
```cpp

GlobalOp GlobalLoadGraphOp::getGlobalOp(SymbolTableCollection &symbolTable) {
  return symbolTable.lookupNearestSymbolFrom<GlobalOp>(
      getOperation()->getParentOp(), getGlobalAttr());
}

LogicalResult
GlobalLoadGraphOp::verifySymbolUses(SymbolTableCollection &symbolTable) {
  GlobalOp referrent = getGlobalOp(symbolTable);
  if (!referrent)
    return emitOpError() << "undefined global: " << getGlobal();

```
- **EN**: Implements logic around `getGlobalOp`, `lookupNearestSymbolFrom`, `getOperation`, `verifySymbolUses`, and 1 more symbols; this block checks operation invariants or verification rules.
- **CN**: 围绕 `getGlobalOp`, `lookupNearestSymbolFrom`, `getOperation`, `verifySymbolUses`, and 1 more symbols 实现具体逻辑；该代码块检查操作不变式或验证规则。

### Lines 248-257
```cpp
  if (referrent.getType() != getResult().getType()) {
    return emitOpError() << "cannot load from global typed "
                         << referrent.getType() << " as "
                         << getResult().getType();
  }

  return success();
}

//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `getType`, `emitOpError`, `getResult`, `success`; this block checks operation invariants or verification rules.
- **CN**: 围绕 `getType`, `emitOpError`, `getResult`, `success` 实现具体逻辑；该代码块检查操作不变式或验证规则。

### Lines 258-271
```cpp
// GlobalStoreOp
//===----------------------------------------------------------------------===//

GlobalOp GlobalStoreOp::getGlobalOp(SymbolTableCollection &symbolTable) {
  for (auto *parent = getOperation()->getParentOp(); parent;) {
    if (auto nearest = symbolTable.lookupNearestSymbolFrom<GlobalOp>(
            parent, getGlobalAttr())) {
      return nearest;
    }
    parent = parent->getParentOp();
  }
  return {};
}

```
- **EN**: Implements logic around `getGlobalOp`, `getOperation`, `lookupNearestSymbolFrom`, `getGlobalAttr`, and 1 more symbols.
- **CN**: 围绕 `getGlobalOp`, `getOperation`, `lookupNearestSymbolFrom`, `getGlobalAttr`, and 1 more symbols 实现具体逻辑。

### Lines 272-282
```cpp
LogicalResult
GlobalStoreOp::verifySymbolUses(SymbolTableCollection &symbolTable) {
  GlobalOp referrent = getGlobalOp(symbolTable);
  if (!referrent)
    return emitOpError() << "undefined global: " << getGlobal();

  if (!referrent.getIsMutable()) {
    return emitOpError() << "cannot store to an immutable global "
                         << getGlobal();
  }

```
- **EN**: Implements logic around `verifySymbolUses`, `getGlobalOp`, `emitOpError`, `getIsMutable`, and 1 more symbols; this block checks operation invariants or verification rules.
- **CN**: 围绕 `verifySymbolUses`, `getGlobalOp`, `emitOpError`, `getIsMutable`, and 1 more symbols 实现具体逻辑；该代码块检查操作不变式或验证规则。

### Lines 283-292
```cpp
  if (referrent.getType() != getValue().getType()) {
    return emitOpError() << "cannot store to a global typed "
                         << referrent.getType() << " from "
                         << getValue().getType();
  }

  return success();
}

//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `getType`, `emitOpError`, `getValue`, `success`; this block checks operation invariants or verification rules.
- **CN**: 围绕 `getType`, `emitOpError`, `getValue`, `success` 实现具体逻辑；该代码块检查操作不变式或验证规则。

### Lines 293-306
```cpp
// GlobalStoreGraphOp
//===----------------------------------------------------------------------===//

GlobalOp GlobalStoreGraphOp::getGlobalOp(SymbolTableCollection &symbolTable) {
  return symbolTable.lookupNearestSymbolFrom<GlobalOp>(
      getOperation()->getParentOp(), getGlobalAttr());
}

LogicalResult
GlobalStoreGraphOp::verifySymbolUses(SymbolTableCollection &symbolTable) {
  GlobalOp referrent = getGlobalOp(symbolTable);
  if (!referrent)
    return emitOpError() << "undefined global: " << getGlobal();

```
- **EN**: Implements logic around `getGlobalOp`, `lookupNearestSymbolFrom`, `getOperation`, `verifySymbolUses`, and 1 more symbols; this block checks operation invariants or verification rules.
- **CN**: 围绕 `getGlobalOp`, `lookupNearestSymbolFrom`, `getOperation`, `verifySymbolUses`, and 1 more symbols 实现具体逻辑；该代码块检查操作不变式或验证规则。

### Lines 307-317
```cpp
  if (!referrent.getIsMutable()) {
    return emitOpError() << "cannot store to an immutable global "
                         << getGlobal();
  }

  if (referrent.getType() != getValue().getType()) {
    return emitOpError() << "cannot store to a global typed "
                         << referrent.getType() << " from "
                         << getValue().getType();
  }

```
- **EN**: Implements logic around `getIsMutable`, `emitOpError`, `getGlobal`, `getType`, and 1 more symbols; this block checks operation invariants or verification rules.
- **CN**: 围绕 `getIsMutable`, `emitOpError`, `getGlobal`, `getType`, and 1 more symbols 实现具体逻辑；该代码块检查操作不变式或验证规则。

### Lines 318-330
```cpp
  return success();
}

//===----------------------------------------------------------------------===//
// SubgraphOp
//===----------------------------------------------------------------------===//

ParseResult SubgraphOp::parse(OpAsmParser &parser, OperationState &result) {
  auto buildFuncType =
      [](Builder &builder, ArrayRef<Type> argTypes, ArrayRef<Type> results,
         function_interface_impl::VariadicFlag,
         std::string &) { return builder.getFunctionType(argTypes, results); };

```
- **EN**: Implements logic around `success`, `parse`, `getFunctionType`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `success`, `parse`, `getFunctionType` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 331-342
```cpp
  return function_interface_impl::parseFunctionOp(
      parser, result, /*allowVariadic=*/false,
      getFunctionTypeAttrName(result.name), buildFuncType,
      getArgAttrsAttrName(result.name), getResAttrsAttrName(result.name));
}

void SubgraphOp::print(OpAsmPrinter &p) {
  function_interface_impl::printFunctionOp(
      p, *this, /*isVariadic=*/false, getFunctionTypeAttrName(),
      getArgAttrsAttrName(), getResAttrsAttrName());
}

```
- **EN**: Implements logic around `parseFunctionOp`, `getFunctionTypeAttrName`, `getArgAttrsAttrName`, `print`, and 1 more symbols; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `parseFunctionOp`, `getFunctionTypeAttrName`, `getArgAttrsAttrName`, `print`, and 1 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 343-356
```cpp
//===----------------------------------------------------------------------===//
// OutputOp
//===----------------------------------------------------------------------===//

LogicalResult OutputOp::verify() {
  auto function = cast<SubgraphOp>((*this)->getParentOp());

  // The operand number and types must match the function signature.
  const auto &results = function.getFunctionType().getResults();
  if (getNumOperands() != results.size())
    return emitOpError("has ")
           << getNumOperands() << " operands, but enclosing function (@"
           << function.getName() << ") outputs " << results.size();

```
- **EN**: Implements logic around `verify`, `getParentOp`, `getFunctionType`, `getNumOperands`, and 2 more symbols; this block checks operation invariants or verification rules; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `verify`, `getParentOp`, `getFunctionType`, `getNumOperands`, and 2 more symbols 实现具体逻辑；该代码块检查操作不变式或验证规则，并处理方言 IR 实体，如操作、类型或属性。

### Lines 357-367
```cpp
  for (unsigned i = 0, e = results.size(); i != e; ++i)
    if (getOperand(i).getType() != results[i])
      return emitError() << "type of output operand " << i << " ("
                         << getOperand(i).getType()
                         << ") doesn't match function result type ("
                         << results[i] << ")"
                         << " in function @" << function.getName();

  return success();
}

```
- **EN**: Implements logic around `size`, `getOperand`, `emitError`, `type`, and 2 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `size`, `getOperand`, `emitError`, `type`, and 2 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 368-381
```cpp
//===----------------------------------------------------------------------===//
// ReturnOp
//===----------------------------------------------------------------------===//

LogicalResult ReturnOp::verify() {
  auto function = cast<FuncOp>((*this)->getParentOp());

  // The operand number and types must match the function signature.
  const auto &results = function.getFunctionType().getResults();
  if (getNumOperands() != results.size())
    return emitOpError("has ")
           << getNumOperands() << " operands, but enclosing function (@"
           << function.getName() << ") returns " << results.size();

```
- **EN**: Implements logic around `verify`, `getParentOp`, `getFunctionType`, `getNumOperands`, and 2 more symbols; this block checks operation invariants or verification rules; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `verify`, `getParentOp`, `getFunctionType`, `getNumOperands`, and 2 more symbols 实现具体逻辑；该代码块检查操作不变式或验证规则，并处理方言 IR 实体，如操作、类型或属性。

### Lines 382-391
```cpp
  for (unsigned i = 0, e = results.size(); i != e; ++i)
    if (getOperand(i).getType() != results[i])
      return emitError() << "type of return operand " << i << " ("
                         << getOperand(i).getType()
                         << ") doesn't match function result type ("
                         << results[i] << ")"
                         << " in function @" << function.getName();

  return success();
}
```
- **EN**: Implements logic around `size`, `getOperand`, `emitError`, `type`, and 2 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `size`, `getOperand`, `emitError`, `type`, and 2 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

## Key Concepts / 关键概念

- **Dialect IR definitions / 方言 IR 定义**:
  - **EN**: Defines or implements dialect operations, attributes, types, traits, and registration hooks.
  - **CN**: 定义或实现方言操作、属性、类型、trait 以及注册钩子。
- **Textual assembly syntax / 文本汇编语法**:
  - **EN**: Implements dialect-specific parsing and printing hooks for MLIR assembly.
  - **CN**: 实现 MLIR 汇编的方言专用解析与打印钩子。
- **Operation verification / 操作验证**:
  - **EN**: Checks structural and semantic invariants so malformed IR is rejected early.
  - **CN**: 检查结构与语义不变式，以便尽早拒绝非法 IR。
- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Models reusable capabilities that can be queried across dialect boundaries.
  - **CN**: 建模可跨方言查询的可复用能力。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/MLProgram/IR/MLProgram.h`, `mlir/IR/Builders.h`, `mlir/Interfaces/FunctionImplementation.h`, `mlir/Dialect/MLProgram/IR/MLProgramOps.cpp.inc`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (2), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (1), cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (1)
- **Generated macros / 生成宏**: `GET_OP_CLASSES`
