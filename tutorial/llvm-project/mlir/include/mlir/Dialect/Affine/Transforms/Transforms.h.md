# Transforms.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/Affine/Transforms/Transforms.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This header file defines a set of transforms specific for the AffineOps dialect.
  - **CN**: 该文件位于 `mlir/include/mlir/Dialect/Affine/Transforms`，围绕 Affine 方言公开 `Transforms` 相关的接口、规则或生成式定义。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- Transforms.h - Transforms Entrypoints --------------------*- C++ -*-===//
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
// This header file defines a set of transforms specific for the AffineOps
// dialect.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 14-19
```cpp
#ifndef MLIR_DIALECT_AFFINE_TRANSFORMS_TRANSFORMS_H
#define MLIR_DIALECT_AFFINE_TRANSFORMS_TRANSFORMS_H

#include "mlir/Interfaces/ValueBoundsOpInterface.h"
#include "mlir/Support/LLVM.h"

```
- **EN**: Pulls in the headers needed by this declaration unit, including `mlir/Interfaces/ValueBoundsOpInterface.h`, `mlir/Support/LLVM.h`.
- **CN**: 引入该声明单元所需的头文件，其中包括 `mlir/Interfaces/ValueBoundsOpInterface.h`, `mlir/Support/LLVM.h`。

### Lines 20-25
```cpp
namespace mlir {
class AffineMap;
class Location;
class OpBuilder;
class OpFoldResult;
class RewritePatternSet;
```
- **EN**: Introduces declarations for `mlir`, `AffineMap`, `Location`, `OpBuilder`, and 2 more symbols, establishing the public symbols or declarative records used later.
- **CN**: 引入 `mlir`, `AffineMap`, `Location`, `OpBuilder`, and 2 more symbols 等声明，建立后续使用的公共符号或声明式记录。

### Lines 26-32
```cpp
class RewriterBase;
class Value;

namespace presburger {
enum class BoundType;
} // namespace presburger

```
- **EN**: Introduces declarations for `RewriterBase`, `Value`, `presburger`, `BoundType`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `RewriterBase`, `Value`, `presburger`, `BoundType` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 33-38
```cpp
namespace affine {
class AffineApplyOp;
class AffineDelinearizeIndexOp;
class AffineLinearizeIndexOp;
class AffineMaxOp;
class AffineMinOp;
```
- **EN**: Introduces declarations for `affine`, `AffineApplyOp`, `AffineDelinearizeIndexOp`, `AffineLinearizeIndexOp`, and 2 more symbols, establishing the public symbols or declarative records used later.
- **CN**: 引入 `affine`, `AffineApplyOp`, `AffineDelinearizeIndexOp`, `AffineLinearizeIndexOp`, and 2 more symbols 等声明，建立后续使用的公共符号或声明式记录。

### Lines 39-44
```cpp

/// Lowers `affine.delinearize_index` into a sequence of division and remainder
/// operations.
LogicalResult lowerAffineDelinearizeIndexOp(RewriterBase &rewriter,
                                            AffineDelinearizeIndexOp op);

```
- **EN**: Declares APIs or declarative rules around `lowerAffineDelinearizeIndexOp`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `lowerAffineDelinearizeIndexOp` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 45-51
```cpp
/// Lowers `affine.linearize_index` into a sequence of multiplications and
/// additions. Make a best effort to sort the input indices so that
/// the most loop-invariant terms are at the left of the additions
/// to enable loop-invariant code motion.
LogicalResult lowerAffineLinearizeIndexOp(RewriterBase &rewriter,
                                          AffineLinearizeIndexOp op);

```
- **EN**: Declares APIs or declarative rules around `lowerAffineLinearizeIndexOp`; this block uses rewrite-pattern infrastructure to transform operations; checks structural or semantic invariants; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `lowerAffineLinearizeIndexOp` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并检查结构或语义不变式，并协调核心结构化 MLIR 方言之间的行为。

### Lines 52-57
```cpp
/// Populate patterns that simplify `affine.delinearize_index` /
/// `affine.linearize_index` pairs using value bounds analysis.
void populateSimplifyAffineWithBoundsPatterns(RewritePatternSet &patterns);

/// Populate patterns that expand affine index operations into more fundamental
/// operations (not necessarily restricted to Affine dialect).
```
- **EN**: Declares APIs or declarative rules around `populateSimplifyAffineWithBoundsPatterns`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `populateSimplifyAffineWithBoundsPatterns` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 58-63
```cpp
void populateAffineExpandIndexOpsPatterns(RewritePatternSet &patterns);

/// Populate patterns that expand affine index operations into their equivalent
/// `affine.apply` representations.
void populateAffineExpandIndexOpsAsAffinePatterns(RewritePatternSet &patterns);

```
- **EN**: Declares APIs or declarative rules around `populateAffineExpandIndexOpsPatterns`, `populateAffineExpandIndexOpsAsAffinePatterns`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `populateAffineExpandIndexOpsPatterns`, `populateAffineExpandIndexOpsAsAffinePatterns` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 64-69
```cpp
/// Helper function to rewrite `op`'s affine map and reorder its operands such
/// that they are in increasing order of hoistability (i.e. the least hoistable)
/// operands come first in the operand list.
void reorderOperandsByHoistability(RewriterBase &rewriter, AffineApplyOp op);

/// Split an "affine.apply" operation into smaller ops.
```
- **EN**: Declares APIs or declarative rules around `reorderOperandsByHoistability`; this block uses rewrite-pattern infrastructure to transform operations; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `reorderOperandsByHoistability` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 70-75
```cpp
/// This reassociates a large AffineApplyOp into an ordered list of smaller
/// AffineApplyOps. This can be used right before lowering affine ops to arith
/// to exhibit more opportunities for CSE and LICM.
/// Return the sink AffineApplyOp on success or failure if `op` does not
/// decompose into smaller AffineApplyOps.
/// Note that this can be undone by canonicalization which tries to
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 76-81
```cpp
/// maximally compose chains of AffineApplyOps.
FailureOr<AffineApplyOp> decompose(RewriterBase &rewriter, AffineApplyOp op);

/// Reify a bound for the given variable in terms of SSA values for which
/// `stopCondition` is met.
///
```
- **EN**: Declares APIs or declarative rules around `decompose`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 声明与 `decompose` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作。

### Lines 82-89
```cpp
/// By default, lower/equal bounds are closed and upper bounds are open. If
/// `options.closedUB` is set to "true", upper bounds are also closed.
FailureOr<OpFoldResult>
reifyValueBound(OpBuilder &b, Location loc, presburger::BoundType type,
                const ValueBoundsConstraintSet::Variable &var,
                ValueBoundsConstraintSet::StopConditionFn stopCondition,
                ValueBoundsOptions options = {});

```
- **EN**: Implements logic around `reifyValueBound`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `reifyValueBound` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 90-95
```cpp
/// Reify a bound for the given index-typed value in terms of SSA values for
/// which `stopCondition` is met. If no stop condition is specified, reify in
/// terms of the operands of the owner op.
///
/// By default, lower/equal bounds are closed and upper bounds are open. If
/// `options.closedUB` is set to "true", upper bounds are also closed.
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 96-101
```cpp
///
/// Example:
/// %0 = arith.addi %a, %b : index
/// %1 = arith.addi %0, %c : index
///
/// * If `stopCondition` evaluates to "true" for %0 and %c, "%0 + %c" is an EQ
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 102-111
```cpp
///   bound for %1.
/// * If `stopCondition` evaluates to "true" for %a, %b and %c, "%a + %b + %c"
///   is an EQ bound for %1.
/// * Otherwise, if the owners of %a, %b or %c do not implement the
///   ValueBoundsOpInterface, no bound can be computed.
FailureOr<OpFoldResult> reifyIndexValueBound(
    OpBuilder &b, Location loc, presburger::BoundType type, Value value,
    ValueBoundsConstraintSet::StopConditionFn stopCondition = nullptr,
    ValueBoundsOptions options = {});

```
- **EN**: Implements logic around `reifyIndexValueBound`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `reifyIndexValueBound` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 112-117
```cpp
/// Reify a bound for the specified dimension of the given shaped value in terms
/// of SSA values for which `stopCondition` is met. If no stop condition is
/// specified, reify in terms of the operands of the owner op.
///
/// By default, lower/equal bounds are closed and upper bounds are open. If
/// `options.closedUB` is set to "true", upper bounds are also closed.
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 118-123
```cpp
FailureOr<OpFoldResult> reifyShapedValueDimBound(
    OpBuilder &b, Location loc, presburger::BoundType type, Value value,
    int64_t dim,
    ValueBoundsConstraintSet::StopConditionFn stopCondition = nullptr,
    ValueBoundsOptions options = {});

```
- **EN**: Implements logic around `reifyShapedValueDimBound`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `reifyShapedValueDimBound` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 124-129
```cpp
/// Materialize an already computed bound with Affine dialect ops.
///
/// * `ValueBoundsOpInterface::computeBound` computes bounds but does not
///   create IR. It is dialect independent.
/// * `materializeComputedBound` materializes computed bounds with Affine
///   dialect ops.
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 130-135
```cpp
/// * `reifyIndexValueBound`/`reifyShapedValueDimBound` are a combination of
///   the two functions mentioned above.
OpFoldResult materializeComputedBound(
    OpBuilder &b, Location loc, AffineMap boundMap,
    ArrayRef<std::pair<Value, std::optional<int64_t>>> mapOperands);

```
- **EN**: Declares APIs or declarative rules around `materializeComputedBound`.
- **CN**: 声明与 `materializeComputedBound` 相关的 API 或声明式规则。

### Lines 136-141
```cpp
/// This transform tries to simplify the affine min operation `op`, by finding a
/// common lower bound for a set of expressions in the affine map results. It
/// returns whether the transform updated `op`'s affine map.
///
/// In concrete terms, given an operation like:
/// `affine.min affine_map<(d0)[s0, s1] -> (d0, s1, s0, 128)>(%i)[%s0, %s1]`
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 142-147
```cpp
/// If `d0 < 128` and `128 < s1 < s0`, the transform will update `op` to:
/// `affine.min affine_map<(d0)[s0, s1] -> (d0, 128)>(%i)[%s0, %s1]`.
bool simplifyAffineMinOp(RewriterBase &rewriter, AffineMinOp op);

/// This transform tries to simplify the affine max operation `op`, by finding a
/// common upper bound for a set of expressions in the affine map results. It
```
- **EN**: Declares APIs or declarative rules around `simplifyAffineMinOp`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `simplifyAffineMinOp` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 148-153
```cpp
/// returns whether the transform updated `op`'s affine map.
///
/// In concrete terms, given an operation like:
/// `affine.max affine_map<(d0)[s0, s1] -> (d0, s1, s0, 128)>(%i)[%s0, %s1]`
/// If `d0 > 128` and `s0 > s1 > 128`, the transform will update `op` to:
/// `affine.max affine_map<(d0)[s0, s1] -> (d0, s0)>(%i)[%s0, %s1]`.
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 154-159
```cpp
bool simplifyAffineMaxOp(RewriterBase &rewriter, AffineMaxOp op);

/// This transform applies `simplifyAffineMinOp` and `simplifyAffineMaxOp` to
/// all the `affine.min` or `affine.max` operations in `ops`. After
/// simplification, it invokes the `affine.min/max` canonicalization patterns on
/// `ops`.
```
- **EN**: Declares APIs or declarative rules around `simplifyAffineMaxOp`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `simplifyAffineMaxOp` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 160-169
```cpp
///
/// This transform returns failure if the greedy pattern rewriter failed to
/// converge during canonicalization, otherwise it returns success. If provided,
/// `modified` is set to `true` if the IR was modified in any way.
LogicalResult simplifyAffineMinMaxOps(RewriterBase &rewriter,
                                      ArrayRef<Operation *> ops,
                                      bool *modified = nullptr);
} // namespace affine
} // namespace mlir

```
- **EN**: Introduces declarations for `affine`, `mlir`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `affine`, `mlir` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 170-170
```cpp
#endif // MLIR_DIALECT_AFFINE_TRANSFORMS_TRANSFORMS_H
```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

## Key Concepts / 关键概念

- **Dialect declarations / 方言声明**:
  - **EN**: Defines the public include-surface for a dialect, exposing operations, attributes, types, or interfaces.
  - **CN**: 定义方言的公共头文件表面，暴露操作、属性、类型或接口。
- **Rewrite and lowering support / 重写与 lowering 支持**:
  - **EN**: Collects patterns or pass declarations used to canonicalize, legalize, or lower IR.
  - **CN**: 汇集用于规范化、合法化或 lowering IR 的模式或 Pass 声明。
- **Pattern rewriting / 模式重写**:
  - **EN**: Uses MLIR rewrite patterns to match operations and replace them with improved forms.
  - **CN**: 使用 MLIR 重写模式匹配操作，并将其替换为更合适的形式。
- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Models reusable capabilities that can be queried across dialect boundaries.
  - **CN**: 建模可跨方言查询的可复用能力。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Interfaces/ValueBoundsOpInterface.h`, `mlir/Support/LLVM.h`
- **Subsystem categories / 子系统类别**: cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (1), support utilities and LLVM interop helpers / 支持工具与 LLVM 互操作辅助逻辑 (1)
