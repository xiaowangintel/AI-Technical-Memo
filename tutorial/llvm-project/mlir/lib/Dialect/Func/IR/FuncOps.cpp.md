# FuncOps.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Func/IR/FuncOps.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements the core IR, operations, types, attributes, or interfaces for the Func dialect and function-like IR support.
  - **CN**: 实现 Func 方言与类函数 IR 支持 的核心 IR、操作、类型、属性或接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10
```cpp
//===- FuncOps.cpp - Func Dialect Operations ------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Func/IR/FuncOps.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Func/IR/FuncOps.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Func/IR/FuncOps.h`。

### Lines 11-27
```cpp
#include "mlir/Conversion/ConvertToEmitC/ToEmitCInterface.h"
#include "mlir/Conversion/ConvertToLLVM/ToLLVMInterface.h"
#include "mlir/Dialect/Bufferization/IR/BufferizableOpInterface.h"
#include "mlir/IR/BuiltinTypes.h"
#include "mlir/IR/IRMapping.h"
#include "mlir/IR/Matchers.h"
#include "mlir/IR/OpImplementation.h"
#include "mlir/IR/PatternMatch.h"
#include "mlir/IR/TypeUtilities.h"
#include "mlir/IR/Value.h"
#include "mlir/Interfaces/FunctionImplementation.h"
#include "mlir/Transforms/InliningUtils.h"
#include "llvm/ADT/APFloat.h"
#include "llvm/ADT/MapVector.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SmallVectorExtras.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Conversion/ConvertToEmitC/ToEmitCInterface.h`, `mlir/Conversion/ConvertToLLVM/ToLLVMInterface.h`, `mlir/Dialect/Bufferization/IR/BufferizableOpInterface.h`, `mlir/IR/BuiltinTypes.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Conversion/ConvertToEmitC/ToEmitCInterface.h`, `mlir/Conversion/ConvertToLLVM/ToLLVMInterface.h`, `mlir/Dialect/Bufferization/IR/BufferizableOpInterface.h`, `mlir/IR/BuiltinTypes.h`。

### Lines 28-39
```cpp
#include "mlir/Dialect/Func/IR/FuncOpsDialect.cpp.inc"

using namespace mlir;
using namespace mlir::func;

//===----------------------------------------------------------------------===//
// FuncDialect
//===----------------------------------------------------------------------===//

void FuncDialect::initialize() {
  addOperations<
#define GET_OP_LIST
```
- **EN**: Implements logic around `initialize`; this block registers dialect entities or dialect-level hooks.
- **CN**: 围绕 `initialize` 实现具体逻辑；该代码块注册方言实体或方言级钩子。

### Lines 40-49
```cpp
#include "mlir/Dialect/Func/IR/FuncOps.cpp.inc"
      >();
  declarePromisedInterface<ConvertToEmitCPatternInterface, FuncDialect>();
  declarePromisedInterface<DialectInlinerInterface, FuncDialect>();
  declarePromisedInterface<ConvertToLLVMPatternInterface, FuncDialect>();
  declarePromisedInterfaces<bufferization::BufferizableOpInterface, CallOp,
                            FuncOp, ReturnOp>();
}

/// Materialize a single constant operation from a given attribute value with
```
- **EN**: Implements logic around `FuncDialect>`, `ReturnOp>`; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `FuncDialect>`, `ReturnOp>` 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 50-59
```cpp
/// the desired resultant type.
Operation *FuncDialect::materializeConstant(OpBuilder &builder, Attribute value,
                                            Type type, Location loc) {
  if (ConstantOp::isBuildableWith(value, type))
    return ConstantOp::create(builder, loc, type,
                              llvm::cast<FlatSymbolRefAttr>(value));
  return nullptr;
}

//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `materializeConstant`, `isBuildableWith`, `create`, `cast`; this block registers dialect entities or dialect-level hooks.
- **CN**: 围绕 `materializeConstant`, `isBuildableWith`, `create`, `cast` 实现具体逻辑；该代码块注册方言实体或方言级钩子。

### Lines 60-72
```cpp
// CallOp
//===----------------------------------------------------------------------===//

LogicalResult CallOp::verifySymbolUses(SymbolTableCollection &symbolTable) {
  // Check that the callee attribute was specified.
  auto fnAttr = (*this)->getAttrOfType<FlatSymbolRefAttr>("callee");
  if (!fnAttr)
    return emitOpError("requires a 'callee' symbol reference attribute");
  FuncOp fn = symbolTable.lookupNearestSymbolFrom<FuncOp>(*this, fnAttr);
  if (!fn)
    return emitOpError() << "'" << fnAttr.getValue()
                         << "' does not reference a valid function";

```
- **EN**: Implements logic around `verifySymbolUses`, `getAttrOfType`, `emitOpError`, `lookupNearestSymbolFrom`; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `verifySymbolUses`, `getAttrOfType`, `emitOpError`, `lookupNearestSymbolFrom` 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 73-83
```cpp
  // Verify that the operand and result types match the callee.
  auto fnType = fn.getFunctionType();
  if (fnType.getNumInputs() != getNumOperands())
    return emitOpError("incorrect number of operands for callee");

  for (unsigned i = 0, e = fnType.getNumInputs(); i != e; ++i)
    if (getOperand(i).getType() != fnType.getInput(i))
      return emitOpError("operand type mismatch: expected operand type ")
             << fnType.getInput(i) << ", but provided "
             << getOperand(i).getType() << " for operand number " << i;

```
- **EN**: Implements logic around `getFunctionType`, `getNumInputs`, `emitOpError`, `getOperand`, and 1 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `getFunctionType`, `getNumInputs`, `emitOpError`, `getOperand`, and 1 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 84-94
```cpp
  if (fnType.getNumResults() != getNumResults())
    return emitOpError("incorrect number of results for callee");

  for (unsigned i = 0, e = fnType.getNumResults(); i != e; ++i)
    if (getResult(i).getType() != fnType.getResult(i)) {
      auto diag = emitOpError("result type mismatch at index ") << i;
      diag.attachNote() << "      op result types: " << getResultTypes();
      diag.attachNote() << "function result types: " << fnType.getResults();
      return diag;
    }

```
- **EN**: Implements logic around `getNumResults`, `emitOpError`, `getResult`, `attachNote`.
- **CN**: 围绕 `getNumResults`, `emitOpError`, `getResult`, `attachNote` 实现具体逻辑。

### Lines 95-104
```cpp
  return success();
}

FunctionType CallOp::getCalleeType() {
  return FunctionType::get(getContext(), getOperandTypes(), getResultTypes());
}

//===----------------------------------------------------------------------===//
// CallIndirectOp
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `success`, `getCalleeType`, `get`; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `success`, `getCalleeType`, `get` 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 105-120
```cpp

/// Fold indirect calls that have a constant function as the callee operand.
LogicalResult CallIndirectOp::canonicalize(CallIndirectOp indirectCall,
                                           PatternRewriter &rewriter) {
  // Check that the callee is a constant callee.
  SymbolRefAttr calledFn;
  if (!matchPattern(indirectCall.getCallee(), m_Constant(&calledFn)))
    return failure();

  // Replace with a direct call.
  rewriter.replaceOpWithNewOp<CallOp>(indirectCall, calledFn,
                                      indirectCall.getResultTypes(),
                                      indirectCall.getArgOperands());
  return success();
}

```
- **EN**: Implements logic around `canonicalize`, `matchPattern`, `failure`, `replaceOpWithNewOp`, and 3 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; implements verifier, folding, parsing, or printing hooks; works with symbol tables or function-like operations.
- **CN**: 围绕 `canonicalize`, `matchPattern`, `failure`, `replaceOpWithNewOp`, and 3 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并实现验证、折叠、解析或打印钩子，并处理符号表或类函数操作。

### Lines 121-135
```cpp
//===----------------------------------------------------------------------===//
// ConstantOp
//===----------------------------------------------------------------------===//

LogicalResult ConstantOp::verifySymbolUses(SymbolTableCollection &symbolTable) {
  StringRef fnName = getValue();
  Type type = getType();

  // Try to find the referenced function.
  auto fn = symbolTable.lookupNearestSymbolFrom<FuncOp>(
      this->getOperation(), StringAttr::get(getContext(), fnName));
  if (!fn)
    return emitOpError() << "reference to undefined function '" << fnName
                         << "'";

```
- **EN**: Implements logic around `verifySymbolUses`, `getValue`, `getType`, `lookupNearestSymbolFrom`, and 2 more symbols; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `verifySymbolUses`, `getValue`, `getType`, `lookupNearestSymbolFrom`, and 2 more symbols 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 136-146
```cpp
  // Check that the referenced function has the correct type.
  if (fn.getFunctionType() != type)
    return emitOpError("reference to function with mismatched type");

  return success();
}

OpFoldResult ConstantOp::fold(FoldAdaptor adaptor) {
  return getValueAttr();
}

```
- **EN**: Implements logic around `getFunctionType`, `emitOpError`, `success`, `fold`, and 1 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `getFunctionType`, `emitOpError`, `success`, `fold`, and 1 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 147-156
```cpp
void ConstantOp::getAsmResultNames(
    function_ref<void(Value, StringRef)> setNameFn) {
  setNameFn(getResult(), "f");
}

bool ConstantOp::isBuildableWith(Attribute value, Type type) {
  return llvm::isa<FlatSymbolRefAttr>(value) && llvm::isa<FunctionType>(type);
}

//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `getAsmResultNames`, `function_ref`, `setNameFn`, `isBuildableWith`, and 1 more symbols.
- **CN**: 围绕 `getAsmResultNames`, `function_ref`, `setNameFn`, `isBuildableWith`, and 1 more symbols 实现具体逻辑。

### Lines 157-176
```cpp
// FuncOp
//===----------------------------------------------------------------------===//

FuncOp FuncOp::create(Location location, StringRef name, FunctionType type,
                      ArrayRef<NamedAttribute> attrs) {
  OpBuilder builder(location->getContext());
  OperationState state(location, getOperationName());
  FuncOp::build(builder, state, name, type, attrs);
  return cast<FuncOp>(Operation::create(state));
}
FuncOp FuncOp::create(Location location, StringRef name, FunctionType type,
                      Operation::dialect_attr_range attrs) {
  SmallVector<NamedAttribute, 8> attrRef(attrs);
  return create(location, name, type, llvm::ArrayRef(attrRef));
}
FuncOp FuncOp::create(Location location, StringRef name, FunctionType type,
                      ArrayRef<NamedAttribute> attrs,
                      ArrayRef<DictionaryAttr> argAttrs) {
  FuncOp func = create(location, name, type, attrs);
  func.setAllArgAttrs(argAttrs);
```
- **EN**: Implements logic around `create`, `builder`, `state`, `build`, and 3 more symbols; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `create`, `builder`, `state`, `build`, and 3 more symbols 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 177-188
```cpp
  return func;
}

void FuncOp::build(OpBuilder &builder, OperationState &state, StringRef name,
                   FunctionType type, ArrayRef<NamedAttribute> attrs,
                   ArrayRef<DictionaryAttr> argAttrs) {
  state.addAttribute(SymbolTable::getSymbolAttrName(),
                     builder.getStringAttr(name));
  state.addAttribute(getFunctionTypeAttrName(state.name), TypeAttr::get(type));
  state.attributes.append(attrs.begin(), attrs.end());
  state.addRegion();

```
- **EN**: Implements logic around `build`, `addAttribute`, `getStringAttr`, `append`, and 1 more symbols; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `build`, `addAttribute`, `getStringAttr`, `append`, and 1 more symbols 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 189-202
```cpp
  if (argAttrs.empty())
    return;
  assert(type.getNumInputs() == argAttrs.size());
  call_interface_impl::addArgAndResultAttrs(
      builder, state, argAttrs, /*resultAttrs=*/{},
      getArgAttrsAttrName(state.name), getResAttrsAttrName(state.name));
}

ParseResult FuncOp::parse(OpAsmParser &parser, OperationState &result) {
  auto buildFuncType =
      [](Builder &builder, ArrayRef<Type> argTypes, ArrayRef<Type> results,
         function_interface_impl::VariadicFlag,
         std::string &) { return builder.getFunctionType(argTypes, results); };

```
- **EN**: Implements logic around `empty`, `assert`, `addArgAndResultAttrs`, `getArgAttrsAttrName`, and 2 more symbols; this block implements verifier, folding, parsing, or printing hooks; works with symbol tables or function-like operations.
- **CN**: 围绕 `empty`, `assert`, `addArgAndResultAttrs`, `getArgAttrsAttrName`, and 2 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并处理符号表或类函数操作。

### Lines 203-214
```cpp
  return function_interface_impl::parseFunctionOp(
      parser, result, /*allowVariadic=*/false,
      getFunctionTypeAttrName(result.name), buildFuncType,
      getArgAttrsAttrName(result.name), getResAttrsAttrName(result.name));
}

void FuncOp::print(OpAsmPrinter &p) {
  function_interface_impl::printFunctionOp(
      p, *this, /*isVariadic=*/false, getFunctionTypeAttrName(),
      getArgAttrsAttrName(), getResAttrsAttrName());
}

```
- **EN**: Implements logic around `parseFunctionOp`, `getFunctionTypeAttrName`, `getArgAttrsAttrName`, `print`, and 1 more symbols; this block implements verifier, folding, parsing, or printing hooks; works with symbol tables or function-like operations.
- **CN**: 围绕 `parseFunctionOp`, `getFunctionTypeAttrName`, `getArgAttrsAttrName`, `print`, and 1 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并处理符号表或类函数操作。

### Lines 215-224
```cpp
/// Clone the internal blocks from this function into dest and all attributes
/// from this function to dest.
void FuncOp::cloneInto(FuncOp dest, IRMapping &mapper) {
  // Add the attributes of this function to dest.
  llvm::MapVector<StringAttr, Attribute> newAttrMap;
  for (const auto &attr : dest->getAttrs())
    newAttrMap.insert({attr.getName(), attr.getValue()});
  for (const auto &attr : (*this)->getAttrs())
    newAttrMap.insert({attr.getName(), attr.getValue()});

```
- **EN**: Implements logic around `cloneInto`, `getAttrs`, `insert`; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `cloneInto`, `getAttrs`, `insert` 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 225-234
```cpp
  auto newAttrs = llvm::map_to_vector(
      newAttrMap, [](std::pair<StringAttr, Attribute> attrPair) {
        return NamedAttribute(attrPair.first, attrPair.second);
      });
  dest->setAttrs(DictionaryAttr::get(getContext(), newAttrs));

  // Clone the body.
  getBody().cloneInto(&dest.getBody(), mapper);
}

```
- **EN**: Implements logic around `map_to_vector`, `NamedAttribute`, `setAttrs`, `getBody`.
- **CN**: 围绕 `map_to_vector`, `NamedAttribute`, `setAttrs`, `getBody` 实现具体逻辑。

### Lines 235-249
```cpp
/// Create a deep copy of this function and all of its blocks, remapping
/// any operands that use values outside of the function using the map that is
/// provided (leaving them alone if no entry is present). Replaces references
/// to cloned sub-values with the corresponding value that is copied, and adds
/// those mappings to the mapper.
FuncOp FuncOp::clone(IRMapping &mapper) {
  // Create the new function.
  FuncOp newFunc = cast<FuncOp>(getOperation()->cloneWithoutRegions());

  // If the function has a body, then the user might be deleting arguments to
  // the function by specifying them in the mapper. If so, we don't add the
  // argument to the input type vector.
  if (!isExternal()) {
    FunctionType oldType = getFunctionType();

```
- **EN**: Implements logic around `clone`, `cast`, `isExternal`, `getFunctionType`; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `clone`, `cast`, `isExternal`, `getFunctionType` 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 250-262
```cpp
    unsigned oldNumArgs = oldType.getNumInputs();
    SmallVector<Type, 4> newInputs;
    newInputs.reserve(oldNumArgs);
    for (unsigned i = 0; i != oldNumArgs; ++i)
      if (!mapper.contains(getArgument(i)))
        newInputs.push_back(oldType.getInput(i));

    /// If any of the arguments were dropped, update the type and drop any
    /// necessary argument attributes.
    if (newInputs.size() != oldNumArgs) {
      newFunc.setType(FunctionType::get(oldType.getContext(), newInputs,
                                        oldType.getResults()));

```
- **EN**: Implements logic around `getNumInputs`, `reserve`, `contains`, `push_back`, and 3 more symbols.
- **CN**: 围绕 `getNumInputs`, `reserve`, `contains`, `push_back`, and 3 more symbols 实现具体逻辑。

### Lines 263-273
```cpp
      if (ArrayAttr argAttrs = getAllArgAttrs()) {
        SmallVector<Attribute> newArgAttrs;
        newArgAttrs.reserve(newInputs.size());
        for (unsigned i = 0; i != oldNumArgs; ++i)
          if (!mapper.contains(getArgument(i)))
            newArgAttrs.push_back(argAttrs[i]);
        newFunc.setAllArgAttrs(newArgAttrs);
      }
    }
  }

```
- **EN**: Implements logic around `getAllArgAttrs`, `reserve`, `contains`, `push_back`, and 1 more symbols.
- **CN**: 围绕 `getAllArgAttrs`, `reserve`, `contains`, `push_back`, and 1 more symbols 实现具体逻辑。

### Lines 274-283
```cpp
  /// Clone the current function into the new one and return it.
  cloneInto(newFunc, mapper);
  return newFunc;
}
FuncOp FuncOp::clone() {
  IRMapping mapper;
  return clone(mapper);
}

//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `cloneInto`, `clone`; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `cloneInto`, `clone` 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 284-303
```cpp
// ReturnOp
//===----------------------------------------------------------------------===//

LogicalResult FuncOp::verifyRegions() {
  // External declarations have no body to check.
  if (isDeclaration())
    return success();
  // Hoist the result types once; they are the same for every return site.
  auto resultTypes = getFunctionType().getResults();
  for (Block &block : getBody()) {
    if (block.empty())
      continue;
    // Check func.return or other return-like terminators ops (e.g.
    // llvm.return, test.return).
    auto returnOp = dyn_cast<RegionBranchTerminatorOpInterface>(&block.back());
    if (!returnOp)
      continue;
    auto operands =
        returnOp.getMutableSuccessorOperands(RegionSuccessor::parent());
    if (operands.size() != resultTypes.size())
```
- **EN**: Implements logic around `verifyRegions`, `isDeclaration`, `success`, `getFunctionType`, and 5 more symbols; this block manipulates MLIR regions, blocks, or control-flow edges; works with symbol tables or function-like operations.
- **CN**: 围绕 `verifyRegions`, `isDeclaration`, `success`, `getFunctionType`, and 5 more symbols 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边，并处理符号表或类函数操作。

### Lines 304-317
```cpp
      return returnOp->emitOpError("has ")
             << operands.size() << " operands, but enclosing function (@"
             << getName() << ") returns " << resultTypes.size();

    for (auto [i, opType] : llvm::enumerate(llvm::zip(operands, resultTypes))) {
      auto [operand, resTy] = opType;
      if (operand.get().getType() != resTy)
        return returnOp->emitError() << "type of return operand " << i << " ("
                                     << operand.get().getType()
                                     << ") doesn't match function result type ("
                                     << resTy << ") in function @" << getName();
    }
  }

```
- **EN**: Implements logic around `emitOpError`, `size`, `getName`, `enumerate`, and 3 more symbols.
- **CN**: 围绕 `emitOpError`, `size`, `getName`, `enumerate`, and 3 more symbols 实现具体逻辑。

### Lines 318-326
```cpp
  return success();
}

//===----------------------------------------------------------------------===//
// TableGen'd op method definitions
//===----------------------------------------------------------------------===//

#define GET_OP_CLASSES
#include "mlir/Dialect/Func/IR/FuncOps.cpp.inc"
```
- **EN**: Implements logic around `success`.
- **CN**: 围绕 `success` 实现具体逻辑。

## Key Concepts / 关键概念

- **Dialect IR modeling / 方言 IR 建模**:
  - **EN**: Defines operations, attributes, types, verifiers, parsers, and printers for a dialect.
  - **CN**: 定义方言的操作、属性、类型、验证器、解析器与打印器。
- **Function-like regions / 类函数 Region**:
  - **EN**: Provides function operations, callable interfaces, and symbol-aware bodies.
  - **CN**: 提供函数操作、可调用接口以及符号感知的函数体。
- **Dialect registration / 方言注册**:
  - **EN**: Registers operations, types, attributes, or interfaces into an MLIR dialect object.
  - **CN**: 把操作、类型、属性或接口注册到 MLIR 方言对象中。
- **Assembly format hooks / 汇编格式钩子**:
  - **EN**: Custom parsers and printers define how operations or attributes appear in MLIR assembly.
  - **CN**: 自定义解析器和打印器定义操作或属性在 MLIR 汇编中的呈现方式。
- **Tensor/buffer boundary / 张量/缓冲区边界**:
  - **EN**: Tracks how abstract tensor values are converted into explicit memory effects and memref-based IR.
  - **CN**: 跟踪抽象张量值如何转换成显式内存效应与基于 memref 的 IR。
- **Interface dispatch / 接口分派**:
  - **EN**: Attaches shared behavior to different ops or types through MLIR interfaces.
  - **CN**: 通过 MLIR 接口把共享行为附着到不同操作或类型上。
- **Type/attribute storage / 类型/属性存储**:
  - **EN**: Uses uniqued storage and parser/printer hooks for extensible compile-time metadata.
  - **CN**: 使用唯一化存储以及解析/打印钩子来表示可扩展的编译期元数据。
- **Region-based control flow / 基于 Region 的控制流**:
  - **EN**: Represents nested blocks and successors as first-class IR structure.
  - **CN**: 把嵌套 block 与后继边表示为一等 IR 结构。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Conversion/ConvertToEmitC/ToEmitCInterface.h`, `mlir/Conversion/ConvertToLLVM/ToLLVMInterface.h`, `mlir/Dialect/Bufferization/IR/BufferizableOpInterface.h`, `mlir/IR/BuiltinTypes.h`, `mlir/IR/IRMapping.h`, `mlir/IR/Matchers.h`, `mlir/IR/OpImplementation.h`, `mlir/IR/PatternMatch.h`, `mlir/IR/TypeUtilities.h` ... (+9 more)
- **Subsystem categories / 子系统类别**: MLIR IR core abstractions / MLIR IR 核心抽象 (7), other MLIR dialect declarations / 其他 MLIR 方言声明 (4), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (4), dialect conversion infrastructure / 方言转换基础设施 (2), MLIR interface declarations / MLIR 接口声明 (1), common transformation and pattern-rewrite helpers / 通用变换与模式重写辅助工具 (1)
- **Generated macros / 生成宏**: `GET_OP_LIST`, `GET_OP_CLASSES`
