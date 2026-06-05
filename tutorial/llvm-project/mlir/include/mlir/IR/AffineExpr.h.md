# AffineExpr.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/IR/AffineExpr.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR AffineExpr component. The leading comments describe it as: An affine expression is an affine combination of dimension identifiers and.
- **用途（CN）**: 声明 MLIR AffineExpr 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-23
````cpp
//===- AffineExpr.h - MLIR Affine Expr Class --------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// An affine expression is an affine combination of dimension identifiers and
// symbols, including ceildiv/floordiv/mod by a constant integer.
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_IR_AFFINEEXPR_H
#define MLIR_IR_AFFINEEXPR_H

#include "mlir/IR/Visitors.h"
#include "mlir/Support/LLVM.h"
#include "llvm/ADT/DenseMapInfo.h"
#include "llvm/ADT/Hashing.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/Support/Casting.h"
#include <type_traits>
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 25-70
````cpp
namespace mlir {

class MLIRContext;
class AffineMap;
class IntegerSet;

namespace detail {

struct AffineExprStorage;
struct AffineBinaryOpExprStorage;
struct AffineDimExprStorage;
struct AffineConstantExprStorage;

} // namespace detail

enum class AffineExprKind {
  Add,
  /// RHS of mul is always a constant or a symbolic expression.
  Mul,
  /// RHS of mod is always a constant or a symbolic expression with a positive
  /// value.
  Mod,
  /// RHS of floordiv is always a constant or a symbolic expression.
  FloorDiv,
  /// RHS of ceildiv is always a constant or a symbolic expression.
  CeilDiv,

  /// This is a marker for the last affine binary op. The range of binary
  /// op's is expected to be this element and earlier.
  LAST_AFFINE_BINARY_OP = CeilDiv,

  /// Constant integer.
  Constant,
  /// Dimensional identifier.
  DimId,
  /// Symbolic identifier.
  SymbolId,
};

/// Base type for affine expression.
/// AffineExpr's are immutable value types with intuitive operators to
/// operate on chainable, lightweight compositions.
/// An AffineExpr is an interface to the underlying storage type pointer.
class AffineExpr {
public:
  using ImplType = detail::AffineExprStorage;
````
- **EN**: This C++ declaration introduces `MLIRContext` and establishes part of the API surface for `AffineExpr`.
- **CN**: 该 C++ 声明引入了 `MLIRContext`，并构成 `AffineExpr` API 表面的一部分。

### Lines 71-118
````cpp
  constexpr AffineExpr() {}
  /* implicit */ AffineExpr(const ImplType *expr)
      : expr(const_cast<ImplType *>(expr)) {}

  bool operator==(AffineExpr other) const { return expr == other.expr; }
  bool operator!=(AffineExpr other) const { return !(*this == other); }
  bool operator==(int64_t v) const;
  bool operator!=(int64_t v) const { return !(*this == v); }
  explicit operator bool() const { return expr; }

  bool operator!() const { return expr == nullptr; }

  MLIRContext *getContext() const;

  /// Return the classification for this type.
  AffineExprKind getKind() const;

  void print(raw_ostream &os) const;
  void dump() const;

  /// Returns true if this expression is made out of only symbols and
  /// constants, i.e., it does not involve dimensional identifiers.
  bool isSymbolicOrConstant() const;

  /// Returns true if this is a pure affine expression, i.e., multiplication,
  /// floordiv, ceildiv, and mod is only allowed w.r.t constants.
  bool isPureAffine() const;

  /// Returns the greatest known integral divisor of this affine expression. The
  /// result is always positive.
  int64_t getLargestKnownDivisor() const;

  /// Return true if the affine expression is a multiple of 'factor'.
  bool isMultipleOf(int64_t factor) const;

  /// Return true if the affine expression involves AffineDimExpr `position`.
  bool isFunctionOfDim(unsigned position) const;

  /// Return true if the affine expression involves AffineSymbolExpr `position`.
  bool isFunctionOfSymbol(unsigned position) const;

  /// Walk all of the AffineExpr's in this expression in postorder. This allows
  /// a lambda walk function that can either return `void` or a WalkResult. With
  /// a WalkResult, interrupting is supported.
  template <typename FnT, typename RetT = detail::walkResultType<FnT>>
  RetT walk(FnT &&callback) const {
    return walk<RetT>(*this, callback);
  }
````
- **EN**: This block groups callable interfaces such as `AffineExpr`, `expr`, `bool`, `getContext`, indicating how `AffineExpr` is queried or updated.
- **CN**: 该代码块聚合了 `AffineExpr`, `expr`, `bool`, `getContext` 等可调用接口，展示了如何查询或更新 `AffineExpr`。

### Lines 120-164
````cpp
  /// This method substitutes any uses of dimensions and symbols (e.g.
  /// dim#0 with dimReplacements[0]) and returns the modified expression tree.
  /// This is a dense replacement method: a replacement must be specified for
  /// every single dim and symbol.
  AffineExpr replaceDimsAndSymbols(ArrayRef<AffineExpr> dimReplacements,
                                   ArrayRef<AffineExpr> symReplacements) const;

  /// Dim-only version of replaceDimsAndSymbols.
  AffineExpr replaceDims(ArrayRef<AffineExpr> dimReplacements) const;

  /// Symbol-only version of replaceDimsAndSymbols.
  AffineExpr replaceSymbols(ArrayRef<AffineExpr> symReplacements) const;

  /// Sparse replace method. Replace `expr` by `replacement` and return the
  /// modified expression tree.
  AffineExpr replace(AffineExpr expr, AffineExpr replacement) const;

  /// Sparse replace method. If `*this` appears in `map` replaces it by
  /// `map[*this]` and return the modified expression tree. Otherwise traverse
  /// `*this` and apply replace with `map` on its subexpressions.
  AffineExpr replace(const DenseMap<AffineExpr, AffineExpr> &map) const;

  /// Replace dims[offset ... numDims)
  /// by dims[offset + shift ... shift + numDims).
  AffineExpr shiftDims(unsigned numDims, unsigned shift,
                       unsigned offset = 0) const;

  /// Replace symbols[offset ... numSymbols)
  /// by symbols[offset + shift ... shift + numSymbols).
  AffineExpr shiftSymbols(unsigned numSymbols, unsigned shift,
                          unsigned offset = 0) const;

  AffineExpr operator+(int64_t v) const;
  AffineExpr operator+(AffineExpr other) const;
  AffineExpr operator-() const;
  AffineExpr operator-(int64_t v) const;
  AffineExpr operator-(AffineExpr other) const;
  AffineExpr operator*(int64_t v) const;
  AffineExpr operator*(AffineExpr other) const;
  AffineExpr floorDiv(uint64_t v) const;
  AffineExpr floorDiv(AffineExpr other) const;
  AffineExpr ceilDiv(uint64_t v) const;
  AffineExpr ceilDiv(AffineExpr other) const;
  AffineExpr operator%(uint64_t v) const;
  AffineExpr operator%(AffineExpr other) const;
````
- **EN**: This block groups callable interfaces such as `replaceDimsAndSymbols`, `replaceDims`, `replaceSymbols`, `replace`, indicating how `AffineExpr` is queried or updated.
- **CN**: 该代码块聚合了 `replaceDimsAndSymbols`, `replaceDims`, `replaceSymbols`, `replace` 等可调用接口，展示了如何查询或更新 `AffineExpr`。

### Lines 166-213
````cpp
  /// Compose with an AffineMap.
  /// Returns the composition of this AffineExpr with `map`.
  ///
  /// Prerequisites:
  /// `this` and `map` are composable, i.e. that the number of AffineDimExpr of
  /// `this` is smaller than the number of results of `map`. If a result of a
  /// map does not have a corresponding AffineDimExpr, that result simply does
  /// not appear in the produced AffineExpr.
  ///
  /// Example:
  ///   expr: `d0 + d2`
  ///   map:  `(d0, d1, d2)[s0, s1] -> (d0 + s1, d1 + s0, d0 + d1 + d2)`
  ///   returned expr: `d0 * 2 + d1 + d2 + s1`
  AffineExpr compose(AffineMap map) const;

  friend ::llvm::hash_code hash_value(AffineExpr arg);

  /// Methods supporting C API.
  const void *getAsOpaquePointer() const {
    return static_cast<const void *>(expr);
  }
  static AffineExpr getFromOpaquePointer(const void *pointer) {
    return AffineExpr(
        reinterpret_cast<ImplType *>(const_cast<void *>(pointer)));
  }

  ImplType *getImpl() const { return expr; }

protected:
  ImplType *expr{nullptr};

private:
  /// A trampoline for the templated non-static AffineExpr::walk method to
  /// dispatch lambda `callback`'s of either a void result type or a
  /// WalkResult type. Walk all of the AffineExprs in `e` in postorder. Users
  /// should use the regular (non-static) `walk` method.
  template <typename WalkRetTy>
  static WalkRetTy walk(AffineExpr e,
                        function_ref<WalkRetTy(AffineExpr)> callback);
};

/// Affine binary operation expression. An affine binary operation could be an
/// add, mul, floordiv, ceildiv, or a modulo operation. (Subtraction is
/// represented through a multiply by -1 and add.) These expressions are always
/// constructed in a simplified form. For eg., the LHS and RHS operands can't
/// both be constants. There are additional canonicalizing rules depending on
/// the op type: see checks in the constructor.
class AffineBinaryOpExpr : public AffineExpr {
````
- **EN**: This C++ declaration introduces `AffineBinaryOpExpr` and establishes part of the API surface for `AffineExpr`. Representative entry points here include `compose`, `hash_value`, `getAsOpaquePointer`, `getFromOpaquePointer`.
- **CN**: 该 C++ 声明引入了 `AffineBinaryOpExpr`，并构成 `AffineExpr` API 表面的一部分。 这一段可见的代表性接口包括 `compose`, `hash_value`, `getAsOpaquePointer`, `getFromOpaquePointer`。

### Lines 215-264
````cpp
public:
  using ImplType = detail::AffineBinaryOpExprStorage;
  /* implicit */ AffineBinaryOpExpr(AffineExpr::ImplType *ptr);
  AffineExpr getLHS() const;
  AffineExpr getRHS() const;
};

/// A dimensional identifier appearing in an affine expression.
class AffineDimExpr : public AffineExpr {
public:
  using ImplType = detail::AffineDimExprStorage;
  /* implicit */ AffineDimExpr(AffineExpr::ImplType *ptr);
  unsigned getPosition() const;
};

/// A symbolic identifier appearing in an affine expression.
class AffineSymbolExpr : public AffineExpr {
public:
  using ImplType = detail::AffineDimExprStorage;
  /* implicit */ AffineSymbolExpr(AffineExpr::ImplType *ptr);
  unsigned getPosition() const;
};

/// An integer constant appearing in affine expression.
class AffineConstantExpr : public AffineExpr {
public:
  using ImplType = detail::AffineConstantExprStorage;
  /* implicit */ AffineConstantExpr(AffineExpr::ImplType *ptr = nullptr);
  int64_t getValue() const;
};

/// Make AffineExpr hashable.
inline ::llvm::hash_code hash_value(AffineExpr arg) {
  return ::llvm::hash_value(arg.expr);
}

inline AffineExpr operator+(int64_t val, AffineExpr expr) { return expr + val; }
inline AffineExpr operator*(int64_t val, AffineExpr expr) { return expr * val; }
inline AffineExpr operator-(int64_t val, AffineExpr expr) {
  return expr * (-1) + val;
}

/// These free functions allow clients of the API to not use classes in detail.
AffineExpr getAffineDimExpr(unsigned position, MLIRContext *context);
AffineExpr getAffineSymbolExpr(unsigned position, MLIRContext *context);
AffineExpr getAffineConstantExpr(int64_t constant, MLIRContext *context);
SmallVector<AffineExpr> getAffineConstantExprs(ArrayRef<int64_t> constants,
                                               MLIRContext *context);
AffineExpr getAffineBinaryOpExpr(AffineExprKind kind, AffineExpr lhs,
                                 AffineExpr rhs);
````
- **EN**: This C++ declaration introduces `AffineDimExpr` and establishes part of the API surface for `AffineExpr`. Representative entry points here include `AffineBinaryOpExpr`, `getLHS`, `getRHS`, `AffineDimExpr`.
- **CN**: 该 C++ 声明引入了 `AffineDimExpr`，并构成 `AffineExpr` API 表面的一部分。 这一段可见的代表性接口包括 `AffineBinaryOpExpr`, `getLHS`, `getRHS`, `AffineDimExpr`。

### Lines 265-312
````cpp
/// Constructs an affine expression from a flat ArrayRef. If there are local
/// identifiers (neither dimensional nor symbolic) that appear in the sum of
/// products expression, 'localExprs' is expected to have the AffineExpr
/// for it, and is substituted into. The ArrayRef 'eq' is expected to be in the
/// format [dims, symbols, locals, constant term].
AffineExpr getAffineExprFromFlatForm(ArrayRef<int64_t> flatExprs,
                                     unsigned numDims, unsigned numSymbols,
                                     ArrayRef<AffineExpr> localExprs,
                                     MLIRContext *context);

raw_ostream &operator<<(raw_ostream &os, AffineExpr expr);

/// Simplify an affine expression by flattening and some amount of simple
/// analysis. This has complexity linear in the number of nodes in 'expr'.
/// Returns the simplified expression, which is the same as the input expression
/// if it can't be simplified. When `expr` is semi-affine, a simplified
/// semi-affine expression is constructed in the sorted order of dimension and
/// symbol positions.
AffineExpr simplifyAffineExpr(AffineExpr expr, unsigned numDims,
                              unsigned numSymbols);

namespace detail {
template <int N>
void bindDims(MLIRContext *ctx) {}

template <int N, typename AffineExprTy, typename... AffineExprTy2>
void bindDims(MLIRContext *ctx, AffineExprTy &e, AffineExprTy2 &...exprs) {
  e = getAffineDimExpr(N, ctx);
  bindDims<N + 1, AffineExprTy2 &...>(ctx, exprs...);
}

template <int N>
void bindSymbols(MLIRContext *ctx) {}

template <int N, typename AffineExprTy, typename... AffineExprTy2>
void bindSymbols(MLIRContext *ctx, AffineExprTy &e, AffineExprTy2 &...exprs) {
  e = getAffineSymbolExpr(N, ctx);
  bindSymbols<N + 1, AffineExprTy2 &...>(ctx, exprs...);
}

} // namespace detail

/// Bind a list of AffineExpr references to DimExpr at positions:
///   [0 .. sizeof...(exprs)]
template <typename... AffineExprTy>
void bindDims(MLIRContext *ctx, AffineExprTy &...exprs) {
  detail::bindDims<0>(ctx, exprs...);
}
````
- **EN**: This C++ declaration introduces `detail` and establishes part of the API surface for `AffineExpr`. Representative entry points here include `getAffineExprFromFlatForm`, `simplifyAffineExpr`, `bindDims`, `getAffineDimExpr`.
- **CN**: 该 C++ 声明引入了 `detail`，并构成 `AffineExpr` API 表面的一部分。 这一段可见的代表性接口包括 `getAffineExprFromFlatForm`, `simplifyAffineExpr`, `bindDims`, `getAffineDimExpr`。

### Lines 314-349
````cpp
template <typename AffineExprTy>
void bindDimsList(MLIRContext *ctx, MutableArrayRef<AffineExprTy> exprs) {
  int idx = 0;
  for (AffineExprTy &e : exprs)
    e = getAffineDimExpr(idx++, ctx);
}

/// Bind a list of AffineExpr references to SymbolExpr at positions:
///   [0 .. sizeof...(exprs)]
template <typename... AffineExprTy>
void bindSymbols(MLIRContext *ctx, AffineExprTy &...exprs) {
  detail::bindSymbols<0>(ctx, exprs...);
}

template <typename AffineExprTy>
void bindSymbolsList(MLIRContext *ctx, MutableArrayRef<AffineExprTy> exprs) {
  int idx = 0;
  for (AffineExprTy &e : exprs)
    e = getAffineSymbolExpr(idx++, ctx);
}

/// Get a lower or upper (depending on `isUpper`) bound for `expr` while using
/// the constant lower and upper bounds for its inputs provided in
/// `constLowerBounds` and `constUpperBounds`. Return std::nullopt if such a
/// bound can't be computed. This method only handles simple sum of product
/// expressions (w.r.t constant coefficients) so as to not depend on anything
/// heavyweight in `Analysis`. Expressions of the form: c0*d0 + c1*d1 + c2*s0 +
/// ... + c_n are handled. Expressions involving floordiv, ceildiv, mod or
/// semi-affine ones will lead a none being returned.
std::optional<int64_t>
getBoundForAffineExpr(AffineExpr expr, unsigned numDims, unsigned numSymbols,
                      ArrayRef<std::optional<int64_t>> constLowerBounds,
                      ArrayRef<std::optional<int64_t>> constUpperBounds,
                      bool isUpper);

} // namespace mlir
````
- **EN**: This block groups callable interfaces such as `bindDimsList`, `getAffineDimExpr`, `bindSymbols`, `bindSymbolsList`, indicating how `AffineExpr` is queried or updated.
- **CN**: 该代码块聚合了 `bindDimsList`, `getAffineDimExpr`, `bindSymbols`, `bindSymbolsList` 等可调用接口，展示了如何查询或更新 `AffineExpr`。

### Lines 352-400
````cpp
namespace llvm {

// AffineExpr hash just like pointers
template <>
struct DenseMapInfo<mlir::AffineExpr> {
  static mlir::AffineExpr getEmptyKey() {
    auto *pointer = llvm::DenseMapInfo<void *>::getEmptyKey();
    return mlir::AffineExpr(static_cast<mlir::AffineExpr::ImplType *>(pointer));
  }
  static mlir::AffineExpr getTombstoneKey() {
    auto *pointer = llvm::DenseMapInfo<void *>::getTombstoneKey();
    return mlir::AffineExpr(static_cast<mlir::AffineExpr::ImplType *>(pointer));
  }
  static unsigned getHashValue(mlir::AffineExpr val) {
    return mlir::hash_value(val);
  }
  static bool isEqual(mlir::AffineExpr LHS, mlir::AffineExpr RHS) {
    return LHS == RHS;
  }
};

/// Add support for llvm style casts. We provide a cast between To and From if
/// From is mlir::AffineExpr or derives from it.
template <typename To, typename From>
struct CastInfo<To, From,
                std::enable_if_t<std::is_same_v<mlir::AffineExpr,
                                                std::remove_const_t<From>> ||
                                 std::is_base_of_v<mlir::AffineExpr, From>>>
    : NullableValueCastFailed<To>,
      DefaultDoCastIfPossible<To, From, CastInfo<To, From>> {

  static inline bool isPossible(mlir::AffineExpr expr) {
    /// Return a constant true instead of a dynamic true when casting to self or
    /// up the hierarchy.
    if constexpr (std::is_base_of_v<To, From>) {
      return true;
    } else {
      if constexpr (std::is_same_v<To, ::mlir::AffineBinaryOpExpr>)
        return expr.getKind() <= ::mlir::AffineExprKind::LAST_AFFINE_BINARY_OP;
      if constexpr (std::is_same_v<To, ::mlir::AffineDimExpr>)
        return expr.getKind() == ::mlir::AffineExprKind::DimId;
      if constexpr (std::is_same_v<To, ::mlir::AffineSymbolExpr>)
        return expr.getKind() == ::mlir::AffineExprKind::SymbolId;
      if constexpr (std::is_same_v<To, ::mlir::AffineConstantExpr>)
        return expr.getKind() == ::mlir::AffineExprKind::Constant;
    }
  }
  static inline To doCast(mlir::AffineExpr expr) { return To(expr.getImpl()); }
};
````
- **EN**: This C++ declaration introduces `DenseMapInfo` and establishes part of the API surface for `AffineExpr`. Representative entry points here include `getEmptyKey`, `AffineExpr`, `getTombstoneKey`, `getHashValue`.
- **CN**: 该 C++ 声明引入了 `DenseMapInfo`，并构成 `AffineExpr` API 表面的一部分。 这一段可见的代表性接口包括 `getEmptyKey`, `AffineExpr`, `getTombstoneKey`, `getHashValue`。

### Lines 401-401
````cpp
} // namespace llvm
````
- **EN**: This section focuses on } // namespace llvm, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“} // namespace llvm”这一主题，把相关声明与辅助接口组织在一起。

### Lines 404-404
````cpp
#endif // MLIR_IR_AFFINEEXPR_H
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: C++ declaration surface for MLIR infrastructure
  **CN**: MLIR 基础设施的 C++ 声明层
- **EN**: Type or attribute abstraction
  **CN**: 类型或属性抽象

## Dependencies / 依赖关系

- mlir/IR/Visitors.h
- mlir/Support/LLVM.h
- llvm/ADT/DenseMapInfo.h
- llvm/ADT/Hashing.h
- llvm/ADT/SmallVector.h
- llvm/Support/Casting.h
- AffineBinaryOpExpr inherits from public AffineExpr
- AffineDimExpr inherits from public AffineExpr
- AffineSymbolExpr inherits from public AffineExpr
- AffineConstantExpr inherits from public AffineExpr
- AffineBinaryOpExpr builds on public AffineExpr
- AffineDimExpr builds on public AffineExpr
- AffineSymbolExpr builds on public AffineExpr
- AffineConstantExpr builds on public AffineExpr
