# AffineOps.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/Affine/IR/AffineOps.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file defines convenience types for working with Affine operations in the MLIR operation set.
  - **CN**: 该文件位于 `mlir/include/mlir/Dialect/Affine/IR`，围绕 Affine 方言公开 `AffineOps` 相关的接口、规则或生成式定义。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- AffineOps.h - MLIR Affine Operations -------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-13
```cpp
//
// This file defines convenience types for working with Affine operations
// in the MLIR operation set.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 14-24
```cpp
#ifndef MLIR_DIALECT_AFFINE_IR_AFFINEOPS_H
#define MLIR_DIALECT_AFFINE_IR_AFFINEOPS_H

#include "mlir/Dialect/Affine/IR/AffineMemoryOpInterfaces.h"
#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/Utils/StaticValueUtils.h"
#include "mlir/IR/AffineMap.h"
#include "mlir/IR/Builders.h"
#include "mlir/Interfaces/ControlFlowInterfaces.h"
#include "mlir/Interfaces/LoopLikeInterface.h"
namespace mlir {
```
- **EN**: Pulls in the headers needed by this declaration unit, including `mlir/Dialect/Affine/IR/AffineMemoryOpInterfaces.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Utils/StaticValueUtils.h`, `mlir/IR/AffineMap.h`.
- **CN**: 引入该声明单元所需的头文件，其中包括 `mlir/Dialect/Affine/IR/AffineMemoryOpInterfaces.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Utils/StaticValueUtils.h`, `mlir/IR/AffineMap.h`。

### Lines 25-30
```cpp
namespace affine {

class AffineApplyOp;
class AffineBound;
class AffineMaxOp;
class AffineMinOp;
```
- **EN**: Introduces declarations for `affine`, `AffineApplyOp`, `AffineBound`, `AffineMaxOp`, and 1 more symbols, establishing the public symbols or declarative records used later.
- **CN**: 引入 `affine`, `AffineApplyOp`, `AffineBound`, `AffineMaxOp`, and 1 more symbols 等声明，建立后续使用的公共符号或声明式记录。

### Lines 31-36
```cpp
class AffineValueMap;

/// A utility function to check if a value is defined at the top level of an
/// op with trait `AffineScope` or is a region argument for such an op. A value
/// of index type defined at the top level is always a valid symbol for all its
/// uses.
```
- **EN**: Introduces declarations for `AffineValueMap`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineValueMap` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 37-42
```cpp
bool isTopLevelValue(Value value);

/// A utility function to check if a value is defined at the top level of
/// `region` or is an argument of `region`. A value of index type defined at the
/// top level of a `AffineScope` region is always a valid symbol for all
/// uses in that region.
```
- **EN**: Declares APIs or declarative rules around `isTopLevelValue`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `isTopLevelValue` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 43-48
```cpp
bool isTopLevelValue(Value value, Region *region);

/// Returns the closest region enclosing `op` that is held by an operation with
/// trait `AffineScope`; `nullptr` if there is no such region.
Region *getAffineScope(Operation *op);

```
- **EN**: Declares APIs or declarative rules around `isTopLevelValue`, `getAffineScope`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `isTopLevelValue`, `getAffineScope` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 49-55
```cpp
/// Returns the closest region enclosing `op` that is held by a non-affine
/// operation; `nullptr` if there is no such region. This method is meant to
/// be used by affine analysis methods (e.g. dependence analysis) which are
/// only meaningful when performed among/between operations from the same
/// analysis scope.
Region *getAffineAnalysisScope(Operation *op);

```
- **EN**: Declares APIs or declarative rules around `getAffineAnalysisScope`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `getAffineAnalysisScope` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 56-61
```cpp
/// Return the product of `terms`, creating an `affine.apply` if any of them are
/// non-constant values. If any of `terms` is `nullptr`, return `nullptr`.
OpFoldResult computeProduct(Location loc, OpBuilder &builder,
                            ArrayRef<OpFoldResult> terms);

/// Returns true if the given Value can be used as a dimension id in the region
```
- **EN**: Declares APIs or declarative rules around `computeProduct`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `computeProduct` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 62-68
```cpp
/// of the closest surrounding op that has the trait `AffineScope`.
bool isValidDim(Value value);

/// Returns true if the given Value can be used as a dimension id in `region`,
/// i.e., for all its uses in `region`.
bool isValidDim(Value value, Region *region);

```
- **EN**: Declares APIs or declarative rules around `isValidDim`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `isValidDim` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 69-74
```cpp
/// Returns true if the given value can be used as a symbol in the region of the
/// closest surrounding op that has the trait `AffineScope`.
bool isValidSymbol(Value value);

/// Returns true if the given Value can be used as a symbol for `region`, i.e.,
/// for all its uses in `region`.
```
- **EN**: Declares APIs or declarative rules around `isValidSymbol`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `isValidSymbol` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 75-82
```cpp
bool isValidSymbol(Value value, Region *region);

/// Parses dimension and symbol list. `numDims` is set to the number of
/// dimensions in the list parsed.
ParseResult parseDimAndSymbolList(OpAsmParser &parser,
                                  SmallVectorImpl<Value> &operands,
                                  unsigned &numDims);

```
- **EN**: Declares APIs or declarative rules around `isValidSymbol`, `parseDimAndSymbolList`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `isValidSymbol`, `parseDimAndSymbolList` 相关的 API 或声明式规则；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 83-88
```cpp
/// Modifies both `map` and `operands` in-place so as to:
/// 1. drop duplicate operands
/// 2. drop unused dims and symbols from map
/// 3. promote valid symbols to symbolic operands in case they appeared as
///    dimensional operands
/// 4. propagate constant operands and drop them
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 89-96
```cpp
void canonicalizeMapAndOperands(AffineMap *map,
                                SmallVectorImpl<Value> *operands);

/// Canonicalizes an integer set the same way canonicalizeMapAndOperands does
/// for affine maps.
void canonicalizeSetAndOperands(IntegerSet *set,
                                SmallVectorImpl<Value> *operands);

```
- **EN**: Declares APIs or declarative rules around `canonicalizeMapAndOperands`, `canonicalizeSetAndOperands`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `canonicalizeMapAndOperands`, `canonicalizeSetAndOperands` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 97-106
```cpp
/// Returns a composed AffineApplyOp by composing `map` and `operands` with
/// other AffineApplyOps supplying those operands. The operands of the resulting
/// AffineApplyOp do not change the length of  AffineApplyOp chains.
AffineApplyOp makeComposedAffineApply(OpBuilder &b, Location loc, AffineMap map,
                                      ArrayRef<OpFoldResult> operands,
                                      bool composeAffineMin = false);
AffineApplyOp makeComposedAffineApply(OpBuilder &b, Location loc, AffineExpr e,
                                      ArrayRef<OpFoldResult> operands,
                                      bool composeAffineMin = false);

```
- **EN**: Declares APIs or declarative rules around `makeComposedAffineApply`.
- **CN**: 声明与 `makeComposedAffineApply` 相关的 API 或声明式规则。

### Lines 107-116
```cpp
/// Constructs an AffineApplyOp that applies `map` to `operands` after composing
/// the map with the maps of any other AffineApplyOp supplying the operands,
/// then immediately attempts to fold it. If folding results in a constant
/// value, no ops are actually created. The `map` must be a single-result affine
/// map.
OpFoldResult makeComposedFoldedAffineApply(OpBuilder &b, Location loc,
                                           AffineMap map,
                                           ArrayRef<OpFoldResult> operands,
                                           bool composeAffineMin = false);
/// Variant of `makeComposedFoldedAffineApply` that applies to an expression.
```
- **EN**: Declares APIs or declarative rules around `makeComposedFoldedAffineApply`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `makeComposedFoldedAffineApply` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 117-122
```cpp
OpFoldResult makeComposedFoldedAffineApply(OpBuilder &b, Location loc,
                                           AffineExpr expr,
                                           ArrayRef<OpFoldResult> operands,
                                           bool composeAffineMin = false);
/// Variant of `makeComposedFoldedAffineApply` suitable for multi-result maps.
/// Note that this may create as many affine.apply operations as the map has
```
- **EN**: Declares APIs or declarative rules around `makeComposedFoldedAffineApply`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `makeComposedFoldedAffineApply` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 123-128
```cpp
/// results given that affine.apply must be single-result.
SmallVector<OpFoldResult> makeComposedFoldedMultiResultAffineApply(
    OpBuilder &b, Location loc, AffineMap map, ArrayRef<OpFoldResult> operands,
    bool composeAffineMin = false);

/// Returns an AffineMinOp obtained by composing `map` and `operands` with
```
- **EN**: Declares APIs or declarative rules around `makeComposedFoldedMultiResultAffineApply`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `makeComposedFoldedMultiResultAffineApply` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 129-134
```cpp
/// AffineApplyOps supplying those operands.
AffineMinOp makeComposedAffineMin(OpBuilder &b, Location loc, AffineMap map,
                                  ArrayRef<OpFoldResult> operands);

/// Constructs an AffineMinOp that computes a minimum across the results of
/// applying `map` to `operands`, then immediately attempts to fold it. If
```
- **EN**: Declares APIs or declarative rules around `makeComposedAffineMin`.
- **CN**: 声明与 `makeComposedAffineMin` 相关的 API 或声明式规则。

### Lines 135-140
```cpp
/// folding results in a constant value, no ops are actually created.
OpFoldResult makeComposedFoldedAffineMin(OpBuilder &b, Location loc,
                                         AffineMap map,
                                         ArrayRef<OpFoldResult> operands);

/// Constructs an AffineMinOp that computes a maximum across the results of
```
- **EN**: Declares APIs or declarative rules around `makeComposedFoldedAffineMin`.
- **CN**: 声明与 `makeComposedFoldedAffineMin` 相关的 API 或声明式规则。

### Lines 141-146
```cpp
/// applying `map` to `operands`, then immediately attempts to fold it. If
/// folding results in a constant value, no ops are actually created.
OpFoldResult makeComposedFoldedAffineMax(OpBuilder &b, Location loc,
                                         AffineMap map,
                                         ArrayRef<OpFoldResult> operands);

```
- **EN**: Declares APIs or declarative rules around `makeComposedFoldedAffineMax`.
- **CN**: 声明与 `makeComposedFoldedAffineMax` 相关的 API 或声明式规则。

### Lines 147-152
```cpp
/// Given an affine map `map` and its input `operands`, this method composes
/// into `map`, maps of AffineApplyOps whose results are the values in
/// `operands`, iteratively until no more of `operands` are the result of an
/// AffineApplyOp. When this function returns, `map` becomes the composed affine
/// map, and each Value in `operands` is guaranteed to be either a loop IV or a
/// terminal symbol, i.e., a symbol defined at the top level or a block/function
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 153-160
```cpp
/// argument.
void fullyComposeAffineMapAndOperands(AffineMap *map,
                                      SmallVectorImpl<Value> *operands,
                                      bool composeAffineMin = false);

} // namespace affine
} // namespace mlir

```
- **EN**: Introduces declarations for `affine`, `mlir`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `affine`, `mlir` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 161-166
```cpp
#include "mlir/Dialect/Affine/IR/AffineOpsDialect.h.inc"

#define GET_OP_CLASSES
#include "mlir/Dialect/Affine/IR/AffineOps.h.inc"

namespace mlir {
```
- **EN**: Pulls in the headers needed by this declaration unit, including `mlir/Dialect/Affine/IR/AffineOpsDialect.h.inc`, `mlir/Dialect/Affine/IR/AffineOps.h.inc`.
- **CN**: 引入该声明单元所需的头文件，其中包括 `mlir/Dialect/Affine/IR/AffineOpsDialect.h.inc`, `mlir/Dialect/Affine/IR/AffineOps.h.inc`。

### Lines 167-172
```cpp
namespace affine {

/// Returns true if the provided value is the induction variable of an
/// AffineForOp.
bool isAffineForInductionVar(Value val);

```
- **EN**: Introduces declarations for `affine`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `affine` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 173-179
```cpp
/// Returns true if `val` is the induction variable of an AffineParallelOp.
bool isAffineParallelInductionVar(Value val);

/// Returns true if the provided value is the induction variable of an
/// AffineForOp or AffineParallelOp.
bool isAffineInductionVar(Value val);

```
- **EN**: Declares APIs or declarative rules around `isAffineParallelInductionVar`, `isAffineInductionVar`.
- **CN**: 声明与 `isAffineParallelInductionVar`, `isAffineInductionVar` 相关的 API 或声明式规则。

### Lines 180-185
```cpp
/// Returns the loop parent of an induction variable. If the provided value is
/// not an induction variable, then return nullptr.
AffineForOp getForInductionVarOwner(Value val);

/// Returns true if the provided value is among the induction variables of an
/// AffineParallelOp.
```
- **EN**: Declares APIs or declarative rules around `getForInductionVarOwner`.
- **CN**: 声明与 `getForInductionVarOwner` 相关的 API 或声明式规则。

### Lines 186-192
```cpp
AffineParallelOp getAffineParallelInductionVarOwner(Value val);

/// Extracts the induction variables from a list of AffineForOps and places them
/// in the output argument `ivs`.
void extractForInductionVars(ArrayRef<AffineForOp> forInsts,
                             SmallVectorImpl<Value> *ivs);

```
- **EN**: Declares APIs or declarative rules around `getAffineParallelInductionVarOwner`, `extractForInductionVars`.
- **CN**: 声明与 `getAffineParallelInductionVarOwner`, `extractForInductionVars` 相关的 API 或声明式规则。

### Lines 193-198
```cpp
/// Extracts the induction variables from a list of either AffineForOp or
/// AffineParallelOp and places them in the output argument `ivs`.
void extractInductionVars(ArrayRef<Operation *> affineOps,
                          SmallVectorImpl<Value> &ivs);

/// Builds a perfect nest of affine.for loops, i.e., each loop except the
```
- **EN**: Declares APIs or declarative rules around `extractInductionVars`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `extractInductionVars` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 199-210
```cpp
/// innermost one contains only another loop and a terminator. The loops iterate
/// from "lbs" to "ubs" with "steps". The body of the innermost loop is
/// populated by calling "bodyBuilderFn" and providing it with an OpBuilder, a
/// Location and a list of loop induction variables.
void buildAffineLoopNest(OpBuilder &builder, Location loc,
                         ArrayRef<int64_t> lbs, ArrayRef<int64_t> ubs,
                         ArrayRef<int64_t> steps,
                         function_ref<void(OpBuilder &, Location, ValueRange)>
                             bodyBuilderFn = nullptr);
void buildAffineLoopNest(OpBuilder &builder, Location loc, ValueRange lbs,
                         ValueRange ubs, ArrayRef<int64_t> steps,
                         function_ref<void(OpBuilder &, Location, ValueRange)>
```
- **EN**: Declares APIs or declarative rules around `buildAffineLoopNest`, `function_ref`.
- **CN**: 声明与 `buildAffineLoopNest`, `function_ref` 相关的 API 或声明式规则。

### Lines 211-216
```cpp
                             bodyBuilderFn = nullptr);

/// AffineBound represents a lower or upper bound in the for operation.
/// This class does not own the underlying operands. Instead, it refers
/// to the operands stored in the AffineForOp. Its life span should not exceed
/// that of the for operation it refers to.
```
- **EN**: Introduces declarations for `does`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `does` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 217-226
```cpp
class AffineBound {
public:
  AffineForOp getAffineForOp() { return op; }
  AffineMap getMap() { return map; }

  unsigned getNumOperands() { return operands.size(); }
  Value getOperand(unsigned idx) {
    return op.getOperand(operands.getBeginOperandIndex() + idx);
  }

```
- **EN**: Introduces declarations for `AffineBound`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineBound` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 227-233
```cpp
  using operand_iterator = AffineForOp::operand_iterator;
  using operand_range = AffineForOp::operand_range;

  operand_iterator operandBegin() { return operands.begin(); }
  operand_iterator operandEnd() { return operands.end(); }
  operand_range getOperands() { return {operandBegin(), operandEnd()}; }

```
- **EN**: Implements logic around `operandBegin`, `operandEnd`, `getOperands`.
- **CN**: 围绕 `operandBegin`, `operandEnd`, `getOperands` 实现具体逻辑。

### Lines 234-241
```cpp
private:
  // 'affine.for' operation that contains this bound.
  AffineForOp op;
  // Operands of the affine map.
  OperandRange operands;
  // Affine map for this bound.
  AffineMap map;

```
- **EN**: Transitions between access-control regions inside a class or struct definition.
- **CN**: 在类或结构体定义内部切换访问控制区域。

### Lines 242-247
```cpp
  AffineBound(AffineForOp op, OperandRange operands, AffineMap map)
      : op(op), operands(operands), map(map) {}

  friend class AffineForOp;
};

```
- **EN**: Introduces declarations for `AffineForOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AffineForOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 248-251
```cpp
} // namespace affine
} // namespace mlir

#endif
```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

## Key Concepts / 关键概念

- **Dialect declarations / 方言声明**:
  - **EN**: Defines the public include-surface for a dialect, exposing operations, attributes, types, or interfaces.
  - **CN**: 定义方言的公共头文件表面，暴露操作、属性、类型或接口。
- **Dialect IR definitions / 方言 IR 定义**:
  - **EN**: Describes the declarative or C++ interface for dialect operations, attributes, types, and registration helpers.
  - **CN**: 描述方言操作、属性、类型及注册辅助逻辑的声明式或 C++ 接口。
- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Models reusable capabilities that can be queried across dialect boundaries.
  - **CN**: 建模可跨方言查询的可复用能力。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Affine/IR/AffineMemoryOpInterfaces.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Utils/StaticValueUtils.h`, `mlir/IR/AffineMap.h`, `mlir/IR/Builders.h`, `mlir/Interfaces/ControlFlowInterfaces.h`, `mlir/Interfaces/LoopLikeInterface.h`, `mlir/Dialect/Affine/IR/AffineOpsDialect.h.inc`, `mlir/Dialect/Affine/IR/AffineOps.h.inc`
- **Subsystem categories / 子系统类别**: dialect-specific operations, attributes, types, transforms, or interface declarations / 方言相关的操作、属性、类型、变换或接口声明 (5), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (2), cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (2)
- **Generated macros / 生成宏**: `GET_OP_CLASSES`
