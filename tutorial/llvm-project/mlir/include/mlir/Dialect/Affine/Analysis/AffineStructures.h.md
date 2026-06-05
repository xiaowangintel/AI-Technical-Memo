# AffineStructures.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/Affine/Analysis/AffineStructures.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Structures for affine/polyhedral analysis of ML functions.
  - **CN**: 该文件位于 `mlir/include/mlir/Dialect/Affine/Analysis`，围绕 Affine 方言公开 `AffineStructures` 相关的接口、规则或生成式定义。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- AffineStructures.h - MLIR Affine Structures Class --------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-15
```cpp
//
// Structures for affine/polyhedral analysis of ML functions.
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_AFFINE_ANALYSIS_AFFINESTRUCTURES_H
#define MLIR_DIALECT_AFFINE_ANALYSIS_AFFINESTRUCTURES_H

```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

### Lines 16-22
```cpp
#include "mlir/Analysis/FlatLinearValueConstraints.h"
#include "mlir/Analysis/Presburger/IntegerRelation.h"
#include "mlir/Analysis/Presburger/Matrix.h"
#include "mlir/IR/AffineExpr.h"
#include "mlir/IR/OpDefinition.h"
#include <optional>

```
- **EN**: Pulls in the headers needed by this declaration unit, including `mlir/Analysis/FlatLinearValueConstraints.h`, `mlir/Analysis/Presburger/IntegerRelation.h`, `mlir/Analysis/Presburger/Matrix.h`, `mlir/IR/AffineExpr.h`.
- **CN**: 引入该声明单元所需的头文件，其中包括 `mlir/Analysis/FlatLinearValueConstraints.h`, `mlir/Analysis/Presburger/IntegerRelation.h`, `mlir/Analysis/Presburger/Matrix.h`, `mlir/IR/AffineExpr.h`。

### Lines 23-28
```cpp
namespace mlir {
class AffineMap;
class IntegerSet;
class MemRefType;
class MLIRContext;
struct MutableAffineMap;
```
- **EN**: Introduces declarations for `mlir`, `AffineMap`, `IntegerSet`, `MemRefType`, and 2 more symbols, establishing the public symbols or declarative records used later.
- **CN**: 引入 `mlir`, `AffineMap`, `IntegerSet`, `MemRefType`, and 2 more symbols 等声明，建立后续使用的公共符号或声明式记录。

### Lines 29-34
```cpp
class Value;

namespace presburger {
class MultiAffineFunction;
} // namespace presburger

```
- **EN**: Introduces declarations for `Value`, `presburger`, `MultiAffineFunction`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Value`, `presburger`, `MultiAffineFunction` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 35-40
```cpp
namespace affine {
class AffineCondition;
class AffineForOp;
class AffineIfOp;
class AffineParallelOp;
class AffineValueMap;
```
- **EN**: Introduces declarations for `affine`, `AffineCondition`, `AffineForOp`, `AffineIfOp`, and 2 more symbols, establishing the public symbols or declarative records used later.
- **CN**: 引入 `affine`, `AffineCondition`, `AffineForOp`, `AffineIfOp`, and 2 more symbols 等声明，建立后续使用的公共符号或声明式记录。

### Lines 41-47
```cpp

/// FlatAffineValueConstraints is an extension of FlatLinearValueConstraints
/// with helper functions for Affine dialect ops.
class FlatAffineValueConstraints : public FlatLinearValueConstraints {
public:
  using FlatLinearValueConstraints::FlatLinearValueConstraints;

```
- **EN**: Introduces declarations for `FlatAffineValueConstraints`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `FlatAffineValueConstraints` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 48-55
```cpp
  /// Return the kind of this object.
  Kind getKind() const override { return Kind::FlatAffineValueConstraints; }

  static bool classof(const IntegerRelation *cst) {
    return cst->getKind() >= Kind::FlatAffineValueConstraints &&
           cst->getKind() <= Kind::FlatAffineRelation;
  }

```
- **EN**: Implements logic around `getKind`, `classof`.
- **CN**: 围绕 `getKind`, `classof` 实现具体逻辑。

### Lines 56-61
```cpp
  /// Adds constraints (lower and upper bounds) for the specified 'affine.for'
  /// operation's Value using IR information stored in its bound maps. The
  /// right variable is first looked up using `forOp`'s Value. Asserts if the
  /// Value corresponding to the 'affine.for' operation isn't found in the
  /// constraint system. Returns failure for the yet unimplemented/unsupported
  /// cases.  Any new variables that are found in the bound operands of the
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 62-67
```cpp
  /// 'affine.for' operation are added as trailing variables (either
  /// dimensional or symbolic depending on whether the operand is a valid
  /// symbol).
  LogicalResult addAffineForOpDomain(AffineForOp forOp);

  /// Add constraints (lower and upper bounds) for the specified
```
- **EN**: Declares APIs or declarative rules around `addAffineForOpDomain`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `addAffineForOpDomain` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 68-73
```cpp
  /// 'affine.parallel' operation's Value using IR information stored in its
  /// bound maps. Returns failure for the yet unimplemented/unsupported cases.
  /// Asserts if the Value corresponding to the 'affine.parallel' operation
  /// isn't found in the constraint system.
  LogicalResult addAffineParallelOpDomain(AffineParallelOp parallelOp);

```
- **EN**: Declares APIs or declarative rules around `addAffineParallelOpDomain`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `addAffineParallelOpDomain` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 74-79
```cpp
  /// Adds constraints (lower and upper bounds) for each loop in the loop nest
  /// described by the bound maps `lbMaps` and `ubMaps` of a computation slice.
  /// Every pair (`lbMaps[i]`, `ubMaps[i]`) describes the bounds of a loop in
  /// the nest, sorted outer-to-inner. `operands` contains the bound operands
  /// for a single bound map. All the bound maps will use the same bound
  /// operands. Note that some loops described by a computation slice might not
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 80-88
```cpp
  /// exist yet in the IR so the Value attached to those dimension variables
  /// might be empty. For that reason, this method doesn't perform Value
  /// look-ups to retrieve the dimension variable positions. Instead, it
  /// assumes the position of the dim variables in the constraint system is
  /// the same as the position of the loop in the loop nest.
  LogicalResult addDomainFromSliceMaps(ArrayRef<AffineMap> lbMaps,
                                       ArrayRef<AffineMap> ubMaps,
                                       ArrayRef<Value> operands);

```
- **EN**: Declares APIs or declarative rules around `addDomainFromSliceMaps`.
- **CN**: 声明与 `addDomainFromSliceMaps` 相关的 API 或声明式规则。

### Lines 89-94
```cpp
  /// Adds constraints imposed by the `affine.if` operation. These constraints
  /// are collected from the IntegerSet attached to the given `affine.if`
  /// instance argument (`ifOp`). It is asserted that:
  /// 1) The IntegerSet of the given `affine.if` instance should not contain
  /// semi-affine expressions,
  /// 2) The columns of the constraint system created from `ifOp` should match
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 95-100
```cpp
  /// the columns in the current one regarding numbers and values.
  void addAffineIfOpDomain(AffineIfOp ifOp);

  /// Adds a bound for the variable at the specified position with constraints
  /// being drawn from the specified bound map and operands. In case of an
  /// EQ bound, the  bound map is expected to have exactly one result. In case
```
- **EN**: Declares APIs or declarative rules around `addAffineIfOpDomain`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `addAffineIfOpDomain` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 101-106
```cpp
  /// of a LB/UB, the bound map may have more than one result, for each of which
  /// an inequality is added.
  LogicalResult addBound(presburger::BoundType type, unsigned pos,
                         AffineMap boundMap, ValueRange operands);
  using FlatLinearValueConstraints::addBound;

```
- **EN**: Declares APIs or declarative rules around `addBound`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `addBound` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 107-112
```cpp
  /// Add the specified values as a dim or symbol var depending on its nature,
  /// if it already doesn't exist in the system. `val` has to be either a
  /// terminal symbol or a loop IV, i.e., it cannot be the result of an
  /// affine.apply of any symbols or loop IVs. Return failure if the addition
  /// wasn't possible due to the above conditions not being met. This method can
  /// also fail if the addition of the domain of an affine IV fails. The
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 113-118
```cpp
  /// variable is added to the end of the existing dims or symbols. Additional
  /// information on the variable is extracted from the IR and added to the
  /// constraint system.
  LogicalResult addInductionVarOrTerminalSymbol(Value val);

  /// Adds slice lower bounds represented by lower bounds in `lbMaps` and upper
```
- **EN**: Declares APIs or declarative rules around `addInductionVarOrTerminalSymbol`.
- **CN**: 声明与 `addInductionVarOrTerminalSymbol` 相关的 API 或声明式规则。

### Lines 119-128
```cpp
  /// bounds in `ubMaps` to each variable in the constraint system which has
  /// a value in `values`. Note that both lower/upper bounds share the same
  /// operand list `operands`.
  /// This function assumes `values.size` == `lbMaps.size` == `ubMaps.size`.
  /// Note that both lower/upper bounds use operands from `operands`.
  LogicalResult addSliceBounds(ArrayRef<Value> values,
                               ArrayRef<AffineMap> lbMaps,
                               ArrayRef<AffineMap> ubMaps,
                               ArrayRef<Value> operands);

```
- **EN**: Declares APIs or declarative rules around `addSliceBounds`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `addSliceBounds` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 129-134
```cpp
  /// Changes all symbol variables which are loop IVs to dim variables.
  void convertLoopIVSymbolsToDims();

  /// Returns the bound for the variable at `pos` from the inequality at
  /// `ineqPos` as a 1-d affine value map (affine map + operands). The returned
  /// affine value map can either be a lower bound or an upper bound depending
```
- **EN**: Declares APIs or declarative rules around `convertLoopIVSymbolsToDims`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `convertLoopIVSymbolsToDims` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 135-140
```cpp
  /// on the sign of atIneq(ineqPos, pos). Asserts if the row at `ineqPos` does
  /// not involve the `pos`th variable.
  void getIneqAsAffineValueMap(unsigned pos, unsigned ineqPos,
                               AffineValueMap &vmap,
                               MLIRContext *context) const;

```
- **EN**: Declares APIs or declarative rules around `getIneqAsAffineValueMap`.
- **CN**: 声明与 `getIneqAsAffineValueMap` 相关的 API 或声明式规则。

### Lines 141-146
```cpp
  /// Composes the affine value map with this FlatAffineValueConstrains, adding
  /// the results of the map as dimensions at the front
  /// [0, vMap->getNumResults()) and with the dimensions set to the equalities
  /// specified by the value map.
  ///
  /// Returns failure if the composition fails (when vMap is a semi-affine map).
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 147-153
```cpp
  /// The vMap's operand Value's are used to look up the right positions in
  /// the FlatAffineValueConstraints with which to associate. Every operand of
  /// vMap should have a matching dim/symbol column in this constraint system
  /// (with the same associated Value).
  LogicalResult composeMap(const AffineValueMap *vMap);
};

```
- **EN**: Declares APIs or declarative rules around `composeMap`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `composeMap` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 154-159
```cpp
/// A FlatAffineRelation represents a set of ordered pairs (domain -> range)
/// where "domain" and "range" are tuples of variables. The relation is
/// represented as a FlatAffineValueConstraints with separation of dimension
/// variables into domain and  range. The variables are stored as:
/// [domainVars, rangeVars, symbolVars, localVars, constant].
///
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 160-165
```cpp
/// Deprecated: use IntegerRelation and store SSA Values in the PresburgerSpace
/// of the relation using PresburgerSpace::identifiers. Note that
/// FlatAffineRelation::numDomainDims and FlatAffineRelation::numRangeDims are
/// independent of numDomain and numRange of the relation's space. In
/// particular, operations such as FlatAffineRelation::compose do not ensure
/// consistency between numDomainDims/numRangeDims and numDomain/numRange which
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 166-177
```cpp
/// may lead to unexpected behaviour.
class FlatAffineRelation : public FlatAffineValueConstraints {
public:
  FlatAffineRelation(unsigned numReservedInequalities,
                     unsigned numReservedEqualities, unsigned numReservedCols,
                     unsigned numDomainDims, unsigned numRangeDims,
                     unsigned numSymbols, unsigned numLocals,
                     ArrayRef<std::optional<Value>> valArgs = {})
      : FlatAffineValueConstraints(
            numReservedInequalities, numReservedEqualities, numReservedCols,
            numDomainDims + numRangeDims, numSymbols, numLocals, valArgs),
        numDomainDims(numDomainDims), numRangeDims(numRangeDims) {}
```
- **EN**: Introduces declarations for `FlatAffineRelation`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `FlatAffineRelation` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 178-184
```cpp

  FlatAffineRelation(unsigned numDomainDims = 0, unsigned numRangeDims = 0,
                     unsigned numSymbols = 0, unsigned numLocals = 0)
      : FlatAffineValueConstraints(numDomainDims + numRangeDims, numSymbols,
                                   numLocals),
        numDomainDims(numDomainDims), numRangeDims(numRangeDims) {}

```
- **EN**: Implements logic around `FlatAffineRelation`, `FlatAffineValueConstraints`, `numDomainDims`.
- **CN**: 围绕 `FlatAffineRelation`, `FlatAffineValueConstraints`, `numDomainDims` 实现具体逻辑。

### Lines 185-194
```cpp
  FlatAffineRelation(unsigned numDomainDims, unsigned numRangeDims,
                     FlatAffineValueConstraints &fac)
      : FlatAffineValueConstraints(fac), numDomainDims(numDomainDims),
        numRangeDims(numRangeDims) {}

  FlatAffineRelation(unsigned numDomainDims, unsigned numRangeDims,
                     IntegerPolyhedron &fac)
      : FlatAffineValueConstraints(fac), numDomainDims(numDomainDims),
        numRangeDims(numRangeDims) {}

```
- **EN**: Implements logic around `FlatAffineRelation`, `FlatAffineValueConstraints`, `numRangeDims`.
- **CN**: 围绕 `FlatAffineRelation`, `FlatAffineValueConstraints`, `numRangeDims` 实现具体逻辑。

### Lines 195-201
```cpp
  /// Return the kind of this object.
  Kind getKind() const override { return Kind::FlatAffineRelation; }

  static bool classof(const IntegerRelation *cst) {
    return cst->getKind() == Kind::FlatAffineRelation;
  }

```
- **EN**: Implements logic around `getKind`, `classof`.
- **CN**: 围绕 `getKind`, `classof` 实现具体逻辑。

### Lines 202-207
```cpp
  /// Returns a set corresponding to the domain/range of the affine relation.
  FlatAffineValueConstraints getDomainSet() const;
  FlatAffineValueConstraints getRangeSet() const;

  /// Returns the number of variables corresponding to domain/range of
  /// relation.
```
- **EN**: Declares APIs or declarative rules around `getDomainSet`, `getRangeSet`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `getDomainSet`, `getRangeSet` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 208-213
```cpp
  inline unsigned getNumDomainDims() const { return numDomainDims; }
  inline unsigned getNumRangeDims() const { return numRangeDims; }

  /// Given affine relation `other: (domainOther -> rangeOther)`, this operation
  /// takes the composition of `other` on `this: (domainThis -> rangeThis)`.
  /// The resulting relation represents tuples of the form: `domainOther ->
```
- **EN**: Implements logic around `getNumDomainDims`, `getNumRangeDims`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getNumDomainDims`, `getNumRangeDims` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 214-220
```cpp
  /// rangeThis`.
  void compose(const FlatAffineRelation &other);

  /// Swap domain and range of the relation.
  /// `(domain -> range)` is converted to `(range -> domain)`.
  void inverse();

```
- **EN**: Declares APIs or declarative rules around `compose`, `inverse`.
- **CN**: 声明与 `compose`, `inverse` 相关的 API 或声明式规则。

### Lines 221-226
```cpp
  /// Insert `num` variables of the specified kind after the `pos` variable
  /// of that kind. The coefficient columns corresponding to the added
  /// variables are initialized to zero.
  void insertDomainVar(unsigned pos, unsigned num = 1);
  void insertRangeVar(unsigned pos, unsigned num = 1);

```
- **EN**: Declares APIs or declarative rules around `insertDomainVar`, `insertRangeVar`.
- **CN**: 声明与 `insertDomainVar`, `insertRangeVar` 相关的 API 或声明式规则。

### Lines 227-232
```cpp
  /// Append `num` variables of the specified kind after the last variable
  /// of that kind. The coefficient columns corresponding to the added
  /// variables are initialized to zero.
  void appendDomainVar(unsigned num = 1);
  void appendRangeVar(unsigned num = 1);

```
- **EN**: Declares APIs or declarative rules around `appendDomainVar`, `appendRangeVar`.
- **CN**: 声明与 `appendDomainVar`, `appendRangeVar` 相关的 API 或声明式规则。

### Lines 233-239
```cpp
  /// Removes variables in the column range [varStart, varLimit), and copies any
  /// remaining valid data into place, updates member variables, and resizes
  /// arrays as needed.
  void removeVarRange(VarKind kind, unsigned varStart,
                      unsigned varLimit) override;
  using IntegerRelation::removeVarRange;

```
- **EN**: Declares APIs or declarative rules around `removeVarRange`.
- **CN**: 声明与 `removeVarRange` 相关的 API 或声明式规则。

### Lines 240-247
```cpp
protected:
  // Number of dimension variables corresponding to domain variables.
  unsigned numDomainDims;

  // Number of dimension variables corresponding to range variables.
  unsigned numRangeDims;
};

```
- **EN**: Transitions between access-control regions inside a class or struct definition.
- **CN**: 在类或结构体定义内部切换访问控制区域。

### Lines 248-253
```cpp
/// Builds a relation from the given AffineMap/AffineValueMap `map`, containing
/// all pairs of the form `operands -> result` that satisfy `map`. `rel` is set
/// to the relation built. For example, give the AffineMap:
///
///   (d0, d1)[s0] -> (d0 + s0, d0 - s0)
///
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 254-259
```cpp
/// the resulting relation formed is:
///
///   (d0, d1) -> (r1, r2)
///   [d0  d1  r1  r2  s0  const]
///    1   0   -1   0  1     0     = 0
///    0   1    0  -1  -1    0     = 0
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 260-268
```cpp
///
/// For AffineValueMap, the domain and symbols have Value set corresponding to
/// the Value in `map`. Returns failure if the AffineMap could not be flattened
/// (i.e., semi-affine is not yet handled).
LogicalResult getRelationFromMap(AffineMap &map,
                                 presburger::IntegerRelation &rel);
LogicalResult getRelationFromMap(const AffineValueMap &map,
                                 presburger::IntegerRelation &rel);

```
- **EN**: Declares APIs or declarative rules around `getRelationFromMap`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `getRelationFromMap` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 269-272
```cpp
} // namespace affine
} // namespace mlir

#endif // MLIR_DIALECT_AFFINE_ANALYSIS_AFFINESTRUCTURES_H
```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

## Key Concepts / 关键概念

- **Dialect declarations / 方言声明**:
  - **EN**: Defines the public include-surface for a dialect, exposing operations, attributes, types, or interfaces.
  - **CN**: 定义方言的公共头文件表面，暴露操作、属性、类型或接口。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Analysis/FlatLinearValueConstraints.h`, `mlir/Analysis/Presburger/IntegerRelation.h`, `mlir/Analysis/Presburger/Matrix.h`, `mlir/IR/AffineExpr.h`, `mlir/IR/OpDefinition.h`
- **Standard-library headers / 标准库头文件**: `<optional>`
- **Subsystem categories / 子系统类别**: MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (2)
