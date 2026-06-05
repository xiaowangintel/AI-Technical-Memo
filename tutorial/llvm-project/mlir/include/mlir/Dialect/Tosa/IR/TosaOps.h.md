# TosaOps.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/Tosa/IR/TosaOps.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR TosaOps component. The leading comments describe it as: This file declares the TOSA Dialect in MLIR.
- **用途（CN）**: 声明 MLIR TosaOps 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-26
````cpp
//===-- TosaOps.h - TOSA dialect operation definitions ----------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file declares the TOSA Dialect in MLIR.
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_TOSA_IR_TOSAOPS_H
#define MLIR_DIALECT_TOSA_IR_TOSAOPS_H

#include "mlir/Bytecode/BytecodeOpInterface.h"
#include "mlir/Dialect/Quant/IR/QuantTypes.h"
#include "mlir/Dialect/Traits.h"
#include "mlir/IR/Matchers.h"
#include "mlir/IR/OpDefinition.h"
#include "mlir/IR/OpImplementation.h"
#include "mlir/IR/TypeUtilities.h"
#include "mlir/Interfaces/InferTypeOpInterface.h"
#include "mlir/Interfaces/LoopLikeInterface.h"
#include "mlir/Interfaces/SideEffectInterfaces.h"
#include "mlir/Interfaces/VectorInterfaces.h"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 28-34
````cpp
//===----------------------------------------------------------------------===//
// TOSA dialect and structs includes.
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Tosa/IR/TosaEnums.h.inc"
#include "mlir/Dialect/Tosa/IR/TosaOpsDialect.h.inc"
#include "mlir/Transforms/DialectConversion.h"
````
- **EN**: This block pulls in the MLIR/LLVM/TableGen dependencies required by later declarations and helps establish the compilation boundary of the file.
- **CN**: 该代码块引入后续声明所依赖的 MLIR/LLVM/TableGen 头文件或记录文件，用于建立本文件的编译边界。

### Lines 36-40
````cpp
//===----------------------------------------------------------------------===//
// TOSA operation validation includes.
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Tosa/IR/TosaAvailability.h.inc"
````
- **EN**: This block pulls in the MLIR/LLVM/TableGen dependencies required by later declarations and helps establish the compilation boundary of the file.
- **CN**: 该代码块引入后续声明所依赖的 MLIR/LLVM/TableGen 头文件或记录文件，用于建立本文件的编译边界。

### Lines 42-89
````cpp
namespace mlir {
class PatternRewriter;

namespace tosa {

ParseResult parseVariableOpTypeOrInitialValue(OpAsmParser &parser,
                                              DenseElementsAttr &varShapeAttr,
                                              TypeAttr &typeAttr,
                                              Attribute &initialValueAttr);
void printVariableOpTypeOrInitialValue(OpAsmPrinter &p, Operation *op,
                                       DenseElementsAttr varShapeAttr,
                                       TypeAttr typeAttr,
                                       Attribute initialValueAttr);

#include "mlir/Dialect/Tosa/IR/TosaInterfaces.h.inc"

} // namespace tosa

namespace OpTrait {
namespace tosa {

// This trait verifies if the element type amoung operands and result
// of multiplication match tosa specification.
template <typename ConcreteType>
class MulOperandsAndResultElementType
    : public TraitBase<ConcreteType, MulOperandsAndResultElementType> {
public:
  static LogicalResult verifyTrait(Operation *op) {
    // Check we have a single result.
    if (failed(impl::verifyOneResult(op)))
      return failure();
    Type resElemType = getElementTypeOrSelf(op->getResult(0));

    // Check we have lhs and rhs.
    if (failed(impl::verifyAtLeastNOperands(op, 2)))
      return failure();

    Type lhsElemType = getElementTypeOrSelf(op->getOperand(0));
    Type rhsElemType = getElementTypeOrSelf(op->getOperand(1));

    // Check that for i32 a shift has been explicitly provided.
    if (lhsElemType.isInteger(32) && failed(impl::verifyNOperands(op, 3)))
      return failure();

    // Verify operands type match (ignoring the shift parameter which will
    // always be i8).
    if (lhsElemType != rhsElemType)
      return op->emitOpError("requires the same element type for all operands");
````
- **EN**: This block pulls in the MLIR/LLVM/TableGen dependencies required by later declarations and helps establish the compilation boundary of the file.
- **CN**: 该代码块引入后续声明所依赖的 MLIR/LLVM/TableGen 头文件或记录文件，用于建立本文件的编译边界。

### Lines 90-134
````cpp
    // Though the spec requires the element type of result to be i32, a more
    // relaxed way is provided at dialect level for easier cooperating with
    // other dialects.
    if (auto resIntType = dyn_cast<IntegerType>(resElemType)) {
      auto lhsIntType = cast<IntegerType>(lhsElemType);
      if (lhsIntType.getWidth() > resIntType.getWidth())
        return op->emitOpError("invalid data type size for operands or result");
    } else {
      // In cases of floating point type or quant types, op requires the same
      // element type for all operands and result (excluding shift).
      if (resElemType != lhsElemType)
        return op->emitOpError(
            "requires the same element type for all operands and results");
    }

    return llvm::success();
  }
};

/// This class indicates that an op is tosa-elementwise (permits broadcasting,
/// unlike Elementwise trait).
template <typename ConcreteType>
class TosaElementwiseOperator
    : public TraitBase<ConcreteType, TosaElementwiseOperator> {};

LogicalResult verifyTosaResolvableShapeOperands(Operation *op);
/// This class verifies that tosa shape operands are compile time resolvable
template <typename ConcreteType>
class TosaResolvableShapeOperands
    : public TraitBase<ConcreteType, TosaResolvableShapeOperands> {
public:
  static LogicalResult verifyTrait(Operation *op) {
    return verifyTosaResolvableShapeOperands(op);
  }
};

/// This class indicates that op operates on tosa shape types
template <typename ConcreteType>
class TosaShapeOperator : public TraitBase<ConcreteType, TosaShapeOperator> {};

LogicalResult verifyTosaShapeOperatorWithSameRanks(Operation *op);
/// This class indicates that op operates on tosa shape types
template <typename ConcreteType>
class TosaShapeOperatorWithSameRanks
    : public TraitBase<ConcreteType, TosaShapeOperatorWithSameRanks> {
````
- **EN**: This C++ declaration introduces `TosaElementwiseOperator` and establishes part of the API surface for `TosaOps`. Representative entry points here include `getWidth`, `emitOpError`, `success`, `verifyTosaResolvableShapeOperands`.
- **CN**: 该 C++ 声明引入了 `TosaElementwiseOperator`，并构成 `TosaOps` API 表面的一部分。 这一段可见的代表性接口包括 `getWidth`, `emitOpError`, `success`, `verifyTosaResolvableShapeOperands`。

### Lines 136-157
````cpp
public:
  static LogicalResult verifyTrait(Operation *op) {
    return verifyTosaShapeOperatorWithSameRanks(op);
  }
};

} // namespace tosa
} // namespace OpTrait

namespace tosa {

bool isa_tosa_shape_type(mlir::Type t);

/// Represents a dimension in the shape of a tensor that can be inferred
/// based on the other provided dimensions. For example, in a reshape
/// operation, -1 can be used to indicate a size that is the remainder
/// of the other dimensions.
constexpr int64_t kInferableDimSize = -1;

} // namespace tosa

} // namespace mlir
````
- **EN**: This C++ declaration introduces `tosa` and establishes part of the API surface for `TosaOps`. Representative entry points here include `verifyTrait`, `verifyTosaShapeOperatorWithSameRanks`, `isa_tosa_shape_type`.
- **CN**: 该 C++ 声明引入了 `tosa`，并构成 `TosaOps` API 表面的一部分。 这一段可见的代表性接口包括 `verifyTrait`, `verifyTosaShapeOperatorWithSameRanks`, `isa_tosa_shape_type`。

### Lines 159-160
````cpp
#define GET_ATTRDEF_CLASSES
#include "mlir/Dialect/Tosa/IR/TosaAttributes.h.inc"
````
- **EN**: This macro block selects a generated declaration fragment before the corresponding `.inc` file is included.
- **CN**: 该宏代码块在包含对应的 `.inc` 文件之前，选择要展开的生成式声明片段。

### Lines 162-163
````cpp
#define GET_TYPEDEF_CLASSES
#include "mlir/Dialect/Tosa/IR/TosaOpsTypesBase.h.inc"
````
- **EN**: This macro block selects a generated declaration fragment before the corresponding `.inc` file is included.
- **CN**: 该宏代码块在包含对应的 `.inc` 文件之前，选择要展开的生成式声明片段。

### Lines 165-166
````cpp
#define GET_OP_CLASSES
#include "mlir/Dialect/Tosa/IR/TosaOps.h.inc"
````
- **EN**: This macro block selects a generated declaration fragment before the corresponding `.inc` file is included.
- **CN**: 该宏代码块在包含对应的 `.inc` 文件之前，选择要展开的生成式声明片段。

### Lines 168-186
````cpp
namespace mlir {
namespace tosa {

// Create a rank-1 const tensor for zero point of the source tensor.
std::optional<Value> createZeroPointTensor(OpBuilder &builder, Location loc,
                                           Type srcElemType, int64_t zp = 0);

// Create a pad-const const tensor with value of `val` of required data-type
Value createPadConstTensor(OpBuilder &builder, Location loc, Value src,
                           int32_t val = 0);

// returns type of variable op
RankedTensorType getVariableType(VariableOp variableOp);

// Returns the bitwidth of a TOSA tensor element type
unsigned getBitWidth(Type type);

} // namespace tosa
} // namespace mlir
````
- **EN**: This C++ declaration introduces `mlir` and establishes part of the API surface for `TosaOps`. Representative entry points here include `createZeroPointTensor`, `createPadConstTensor`, `getVariableType`, `getBitWidth`.
- **CN**: 该 C++ 声明引入了 `mlir`，并构成 `TosaOps` API 表面的一部分。 这一段可见的代表性接口包括 `createZeroPointTensor`, `createPadConstTensor`, `getVariableType`, `getBitWidth`。

### Lines 188-188
````cpp
#endif // MLIR_DIALECT_TOSA_IR_TOSAOPS_H
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

- mlir/Bytecode/BytecodeOpInterface.h
- mlir/Dialect/Quant/IR/QuantTypes.h
- mlir/Dialect/Traits.h
- mlir/IR/Matchers.h
- mlir/IR/OpDefinition.h
- mlir/IR/OpImplementation.h
- mlir/IR/TypeUtilities.h
- mlir/Interfaces/InferTypeOpInterface.h
- mlir/Interfaces/LoopLikeInterface.h
- mlir/Interfaces/SideEffectInterfaces.h
- mlir/Interfaces/VectorInterfaces.h
- mlir/Dialect/Tosa/IR/TosaEnums.h.inc
- mlir/Dialect/Tosa/IR/TosaOpsDialect.h.inc
- mlir/Transforms/DialectConversion.h
- mlir/Dialect/Tosa/IR/TosaAvailability.h.inc
- mlir/Dialect/Tosa/IR/TosaInterfaces.h.inc
- mlir/Dialect/Tosa/IR/TosaAttributes.h.inc
- mlir/Dialect/Tosa/IR/TosaOpsTypesBase.h.inc
- mlir/Dialect/Tosa/IR/TosaOps.h.inc
- MulOperandsAndResultElementType inherits from public TraitBase<ConcreteType, MulOperandsAndResultElementType>
