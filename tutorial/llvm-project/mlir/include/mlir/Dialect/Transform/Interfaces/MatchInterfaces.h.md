# MatchInterfaces.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/Transform/Interfaces/MatchInterfaces.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR MatchInterfaces component. The leading comments describe it as: Dispatch `matchOperation` based on Operation* or std::optional<Operation*>.
- **用途（CN）**: 声明 MLIR MatchInterfaces 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17
````cpp
//===- MatchInterfaces.h - Transform Dialect Interfaces ---------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_TRANSFORM_IR_MATCHINTERFACES_H
#define MLIR_DIALECT_TRANSFORM_IR_MATCHINTERFACES_H

#include <optional>
#include <type_traits>

#include "mlir/Dialect/Transform/Interfaces/TransformInterfaces.h"
#include "mlir/IR/OpDefinition.h"
#include "llvm/ADT/STLExtras.h"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 19-69
````cpp
namespace mlir {
namespace transform {
class MatchOpInterface;

namespace detail {
/// Dispatch `matchOperation` based on Operation* or std::optional<Operation*>
/// first operand.
template <typename OpTy>
DiagnosedSilenceableFailure matchOptionalOperation(OpTy op,
                                                   TransformResults &results,
                                                   TransformState &state) {
  if constexpr (std::is_same_v<
                    typename llvm::function_traits<
                        decltype(&OpTy::matchOperation)>::template arg_t<0>,
                    Operation *>) {
    return op.matchOperation(nullptr, results, state);
  } else {
    return op.matchOperation(std::nullopt, results, state);
  }
}
} // namespace detail

template <typename OpTy>
class AtMostOneOpMatcherOpTrait
    : public OpTrait::TraitBase<OpTy, AtMostOneOpMatcherOpTrait> {
  template <typename T>
  using has_get_operand_handle =
      decltype(std::declval<T &>().getOperandHandle());
  template <typename T>
  using has_match_operation_ptr = decltype(std::declval<T &>().matchOperation(
      std::declval<Operation *>(), std::declval<TransformResults &>(),
      std::declval<TransformState &>()));
  template <typename T>
  using has_match_operation_optional =
      decltype(std::declval<T &>().matchOperation(
          std::declval<std::optional<Operation *>>(),
          std::declval<TransformResults &>(),
          std::declval<TransformState &>()));

public:
  static LogicalResult verifyTrait(Operation *op) {
    static_assert(llvm::is_detected<has_get_operand_handle, OpTy>::value,
                  "AtMostOneOpMatcherOpTrait/SingleOpMatcherOpTrait expects "
                  "operation type to have the getOperandHandle() method");
    static_assert(
        llvm::is_detected<has_match_operation_ptr, OpTy>::value ||
            llvm::is_detected<has_match_operation_optional, OpTy>::value,
        "AtMostOneOpMatcherOpTrait/SingleOpMatcherOpTrait expected operation "
        "type to have either the matchOperation(Operation *, TransformResults "
        "&, TransformState &) or the matchOperation(std::optional<Operation*>, "
        "TransformResults &, TransformState &) method");
````
- **EN**: This C++ declaration introduces `MatchOpInterface` and establishes part of the API surface for `MatchInterfaces`. Representative entry points here include `matchOptionalOperation`, `constexpr`, `matchOperation`, `getOperandHandle`.
- **CN**: 该 C++ 声明引入了 `MatchOpInterface`，并构成 `MatchInterfaces` API 表面的一部分。 这一段可见的代表性接口包括 `matchOptionalOperation`, `constexpr`, `matchOperation`, `getOperandHandle`。

### Lines 70-127
````cpp
    // This must be a dynamic assert because interface registration is dynamic.
    assert(
        isa<MatchOpInterface>(op) &&
        "AtMostOneOpMatcherOpTrait/SingleOpMatchOpTrait is only available on "
        "operations with MatchOpInterface");
    Value operandHandle = cast<OpTy>(op).getOperandHandle();
    if (!isa<TransformHandleTypeInterface>(operandHandle.getType())) {
      return op->emitError() << "AtMostOneOpMatcherOpTrait/"
                                "SingleOpMatchOpTrait requires the op handle "
                                "to be of TransformHandleTypeInterface";
    }

    return success();
  }

  DiagnosedSilenceableFailure apply(TransformRewriter &rewriter,
                                    TransformResults &results,
                                    TransformState &state) {
    Value operandHandle = cast<OpTy>(this->getOperation()).getOperandHandle();
    auto payload = state.getPayloadOps(operandHandle);
    if (!llvm::hasNItemsOrLess(payload, 1)) {
      return emitDefiniteFailure(this->getOperation()->getLoc())
             << "AtMostOneOpMatcherOpTrait requires the operand handle to "
                "point to at most one payload op";
    }
    if (payload.empty()) {
      return detail::matchOptionalOperation(cast<OpTy>(this->getOperation()),
                                            results, state);
    }
    return cast<OpTy>(this->getOperation())
        .matchOperation(*payload.begin(), results, state);
  }

  void getEffects(SmallVectorImpl<MemoryEffects::EffectInstance> &effects) {
    onlyReadsHandle(this->getOperation()->getOpOperands(), effects);
    producesHandle(this->getOperation()->getOpResults(), effects);
    onlyReadsPayload(effects);
  }
};

template <typename OpTy>
class SingleOpMatcherOpTrait : public AtMostOneOpMatcherOpTrait<OpTy> {

public:
  DiagnosedSilenceableFailure apply(TransformRewriter &rewriter,
                                    TransformResults &results,
                                    TransformState &state) {
    Value operandHandle = cast<OpTy>(this->getOperation()).getOperandHandle();
    auto payload = state.getPayloadOps(operandHandle);
    if (!llvm::hasSingleElement(payload)) {
      return emitDefiniteFailure(this->getOperation()->getLoc())
             << "SingleOpMatchOpTrait requires the operand handle to point to "
                "a single payload op";
    }
    return static_cast<AtMostOneOpMatcherOpTrait<OpTy> *>(this)->apply(
        rewriter, results, state);
  }
};
````
- **EN**: This C++ declaration introduces `SingleOpMatcherOpTrait` and establishes part of the API surface for `MatchInterfaces`. Representative entry points here include `getOperandHandle`, `getType`, `emitError`, `success`.
- **CN**: 该 C++ 声明引入了 `SingleOpMatcherOpTrait`，并构成 `MatchInterfaces` API 表面的一部分。 这一段可见的代表性接口包括 `getOperandHandle`, `getType`, `emitError`, `success`。

### Lines 129-169
````cpp
template <typename OpTy>
class SingleValueMatcherOpTrait
    : public OpTrait::TraitBase<OpTy, SingleValueMatcherOpTrait> {
public:
  static LogicalResult verifyTrait(Operation *op) {
    // This must be a dynamic assert because interface registration is
    // dynamic.
    assert(isa<MatchOpInterface>(op) &&
           "SingleValueMatchOpTrait is only available on operations with "
           "MatchOpInterface");

    Value operandHandle = cast<OpTy>(op).getOperandHandle();
    if (!isa<TransformValueHandleTypeInterface>(operandHandle.getType())) {
      return op->emitError() << "SingleValueMatchOpTrait requires an operand "
                                "of TransformValueHandleTypeInterface";
    }

    return success();
  }

  DiagnosedSilenceableFailure apply(TransformRewriter &rewriter,
                                    TransformResults &results,
                                    TransformState &state) {
    Value operandHandle = cast<OpTy>(this->getOperation()).getOperandHandle();
    auto payload = state.getPayloadValues(operandHandle);
    if (!llvm::hasSingleElement(payload)) {
      return emitDefiniteFailure(this->getOperation()->getLoc())
             << "SingleValueMatchOpTrait requires the value handle to point "
                "to a single payload value";
    }

    return cast<OpTy>(this->getOperation())
        .matchValue(*payload.begin(), results, state);
  }

  void getEffects(SmallVectorImpl<MemoryEffects::EffectInstance> &effects) {
    onlyReadsHandle(this->getOperation()->getOpOperands(), effects);
    producesHandle(this->getOperation()->getOpResults(), effects);
    onlyReadsPayload(effects);
  }
};
````
- **EN**: This C++ declaration introduces `SingleValueMatcherOpTrait` and establishes part of the API surface for `MatchInterfaces`. Representative entry points here include `verifyTrait`, `getOperandHandle`, `getType`, `emitError`.
- **CN**: 该 C++ 声明引入了 `SingleValueMatcherOpTrait`，并构成 `MatchInterfaces` API 表面的一部分。 这一段可见的代表性接口包括 `verifyTrait`, `getOperandHandle`, `getType`, `emitError`。

### Lines 172-190
````cpp
//===----------------------------------------------------------------------===//
// Printing/parsing for positional specification matchers
//===----------------------------------------------------------------------===//

/// Parses a positional index specification for transform match operations.
/// The following forms are accepted:
///
///  - `all`: sets `isAll` and returns;
///  - comma-separated-integer-list: populates `rawDimList` with the values;
///  - `except` `(` comma-separated-integer-list `)`: populates `rawDimList`
///  with the values and sets `isInverted`.
ParseResult parseTransformMatchDims(OpAsmParser &parser,
                                    DenseI64ArrayAttr &rawDimList,
                                    UnitAttr &isInverted, UnitAttr &isAll);

/// Prints a positional index specification for transform match operations.
void printTransformMatchDims(OpAsmPrinter &printer, Operation *op,
                             DenseI64ArrayAttr rawDimList, UnitAttr isInverted,
                             UnitAttr isAll);
````
- **EN**: This block groups callable interfaces such as `parseTransformMatchDims`, `printTransformMatchDims`, indicating how `MatchInterfaces` is queried or updated.
- **CN**: 该代码块聚合了 `parseTransformMatchDims`, `printTransformMatchDims` 等可调用接口，展示了如何查询或更新 `MatchInterfaces`。

### Lines 192-219
````cpp
//===----------------------------------------------------------------------===//
// Utilities for positional specification matchers
//===----------------------------------------------------------------------===//

/// Checks if the positional specification defined is valid and reports errors
/// otherwise.
LogicalResult verifyTransformMatchDimsOp(Operation *op, ArrayRef<int64_t> raw,
                                         bool inverted, bool all);

/// Populates `result` with the positional identifiers relative to `maxNumber`.
/// If `isAll` is set, the result will contain all numbers from `0` to
/// `maxNumber - 1` inclusive regardless of `rawList`. Otherwise, negative
/// values from `rawList` are  are interpreted as counting backwards from
/// `maxNumber`, i.e., `-1` is interpreted a `maxNumber - 1`, while positive
/// numbers remain as is. If `isInverted` is set, populates `result` with those
/// values from the `0` to `maxNumber - 1` inclusive range that don't appear in
/// `rawList`. If `rawList` contains values that are greater than or equal to
/// `maxNumber` or less than `-maxNumber`, produces a silenceable error at the
/// given location. `maxNumber` must be positive. If `rawList` contains
/// duplicate numbers or numbers that become duplicate after negative value
/// remapping, emits a silenceable error.
DiagnosedSilenceableFailure
expandTargetSpecification(Location loc, bool isAll, bool isInverted,
                          ArrayRef<int64_t> rawList, int64_t maxNumber,
                          SmallVectorImpl<int64_t> &result);

} // namespace transform
} // namespace mlir
````
- **EN**: This block groups callable interfaces such as `verifyTransformMatchDimsOp`, `expandTargetSpecification`, indicating how `MatchInterfaces` is queried or updated.
- **CN**: 该代码块聚合了 `verifyTransformMatchDimsOp`, `expandTargetSpecification` 等可调用接口，展示了如何查询或更新 `MatchInterfaces`。

### Lines 221-221
````cpp
#include "mlir/Dialect/Transform/Interfaces/MatchInterfaces.h.inc"
````
- **EN**: This block pulls in the MLIR/LLVM/TableGen dependencies required by later declarations and helps establish the compilation boundary of the file.
- **CN**: 该代码块引入后续声明所依赖的 MLIR/LLVM/TableGen 头文件或记录文件，用于建立本文件的编译边界。

### Lines 223-223
````cpp
#endif // MLIR_DIALECT_TRANSFORM_IR_MATCHINTERFACES_H
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: C++ declaration surface for MLIR infrastructure
  **CN**: MLIR 基础设施的 C++ 声明层
- **EN**: Type or attribute abstraction
  **CN**: 类型或属性抽象

## Dependencies / 依赖关系

- mlir/Dialect/Transform/Interfaces/TransformInterfaces.h
- mlir/IR/OpDefinition.h
- llvm/ADT/STLExtras.h
- mlir/Dialect/Transform/Interfaces/MatchInterfaces.h.inc
- AtMostOneOpMatcherOpTrait inherits from public OpTrait::TraitBase<OpTy, AtMostOneOpMatcherOpTrait>
- SingleOpMatcherOpTrait inherits from public AtMostOneOpMatcherOpTrait<OpTy>
- SingleValueMatcherOpTrait inherits from public OpTrait::TraitBase<OpTy, SingleValueMatcherOpTrait>
- AtMostOneOpMatcherOpTrait builds on public OpTrait::TraitBase<OpTy, AtMostOneOpMatcherOpTrait>
- SingleOpMatcherOpTrait builds on public AtMostOneOpMatcherOpTrait<OpTy>
- SingleValueMatcherOpTrait builds on public OpTrait::TraitBase<OpTy, SingleValueMatcherOpTrait>
