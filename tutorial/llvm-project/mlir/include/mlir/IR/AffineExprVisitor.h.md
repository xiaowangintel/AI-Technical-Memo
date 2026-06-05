# AffineExprVisitor.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/IR/AffineExprVisitor.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR AffineExprVisitor component. The leading comments describe it as: This file defines the AffineExpr visitor class.
- **用途（CN）**: 声明 MLIR AffineExprVisitor 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
````cpp
//===- AffineExprVisitor.h - MLIR AffineExpr Visitor Class ------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the AffineExpr visitor class.
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_IR_AFFINEEXPRVISITOR_H
#define MLIR_IR_AFFINEEXPRVISITOR_H

#include "mlir/IR/AffineExpr.h"
#include "mlir/Support/LLVM.h"
#include "llvm/ADT/ArrayRef.h"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 20-72
````cpp
namespace mlir {

/// Base class for AffineExpr visitors/walkers.
///
/// AffineExpr visitors are used when you want to perform different actions
/// for different kinds of AffineExprs without having to use lots of casts
/// and a big switch instruction.
///
/// To define your own visitor, inherit from this class, specifying your
/// new type for the 'SubClass' template parameter, and "override" visitXXX
/// functions in your class. This class is defined in terms of statically
/// resolved overloading, not virtual functions.
///
/// The visitor is templated on its return type (`RetTy`). With a WalkResult
/// return type, the visitor supports interrupting walks.
///
/// For example, here is a visitor that counts the number of for AffineDimExprs
/// in an AffineExpr.
///
///  /// Declare the class.  Note that we derive from AffineExprVisitor
///  /// instantiated with our new subclasses_ type.
///
///  struct DimExprCounter : public AffineExprVisitor<DimExprCounter> {
///    unsigned numDimExprs;
///    DimExprCounter() : numDimExprs(0) {}
///    void visitDimExpr(AffineDimExpr expr) { ++numDimExprs; }
///  };
///
///  And this class would be used like this:
///    DimExprCounter dec;
///    dec.visit(affineExpr);
///    numDimExprs = dec.numDimExprs;
///
/// AffineExprVisitor provides visit methods for the following binary affine
/// op expressions:
/// AffineBinaryAddOpExpr, AffineBinaryMulOpExpr,
/// AffineBinaryModOpExpr, AffineBinaryFloorDivOpExpr,
/// AffineBinaryCeilDivOpExpr. Note that default implementations of these
/// methods will call the general AffineBinaryOpExpr method.
///
/// In addition, visit methods are provided for the following affine
//  expressions: AffineConstantExpr, AffineDimExpr, and
//  AffineSymbolExpr.
///
/// Note that if you don't implement visitXXX for some affine expression type,
/// the visitXXX method for Instruction superclass will be invoked.
///
/// Note that this class is specifically designed as a template to avoid
/// virtual function call overhead. Defining and using a AffineExprVisitor is
/// just as efficient as having your own switch instruction over the instruction
/// opcode.
template <typename SubClass, typename RetTy>
class AffineExprVisitorBase {
````
- **EN**: This C++ declaration introduces `AffineExprVisitorBase` and establishes part of the API surface for `AffineExprVisitor`.
- **CN**: 该 C++ 声明引入了 `AffineExprVisitorBase`，并构成 `AffineExprVisitor` API 表面的一部分。

### Lines 73-108
````cpp
public:
  // Function to visit an AffineExpr.
  RetTy visit(AffineExpr expr) {
    static_assert(std::is_base_of<AffineExprVisitorBase, SubClass>::value,
                  "Must instantiate with a derived type of AffineExprVisitor");
    auto self = static_cast<SubClass *>(this);
    switch (expr.getKind()) {
    case AffineExprKind::Add: {
      auto binOpExpr = cast<AffineBinaryOpExpr>(expr);
      return self->visitAddExpr(binOpExpr);
    }
    case AffineExprKind::Mul: {
      auto binOpExpr = cast<AffineBinaryOpExpr>(expr);
      return self->visitMulExpr(binOpExpr);
    }
    case AffineExprKind::Mod: {
      auto binOpExpr = cast<AffineBinaryOpExpr>(expr);
      return self->visitModExpr(binOpExpr);
    }
    case AffineExprKind::FloorDiv: {
      auto binOpExpr = cast<AffineBinaryOpExpr>(expr);
      return self->visitFloorDivExpr(binOpExpr);
    }
    case AffineExprKind::CeilDiv: {
      auto binOpExpr = cast<AffineBinaryOpExpr>(expr);
      return self->visitCeilDivExpr(binOpExpr);
    }
    case AffineExprKind::Constant:
      return self->visitConstantExpr(cast<AffineConstantExpr>(expr));
    case AffineExprKind::DimId:
      return self->visitDimExpr(cast<AffineDimExpr>(expr));
    case AffineExprKind::SymbolId:
      return self->visitSymbolExpr(cast<AffineSymbolExpr>(expr));
    }
    llvm_unreachable("Unknown AffineExpr");
  }
````
- **EN**: This block groups callable interfaces such as `visit`, `static_assert`, `getKind`, `visitAddExpr`, indicating how `AffineExprVisitor` is queried or updated.
- **CN**: 该代码块聚合了 `visit`, `static_assert`, `getKind`, `visitAddExpr` 等可调用接口，展示了如何查询或更新 `AffineExprVisitor`。

### Lines 110-144
````cpp
  //===--------------------------------------------------------------------===//
  // Visitation functions... these functions provide default fallbacks in case
  // the user does not specify what to do for a particular instruction type.
  // The default behavior is to generalize the instruction type to its subtype
  // and try visiting the subtype.  All of this should be inlined perfectly,
  // because there are no virtual functions to get in the way.
  //

  // Default visit methods. Note that the default op-specific binary op visit
  // methods call the general visitAffineBinaryOpExpr visit method.
  RetTy visitAffineBinaryOpExpr(AffineBinaryOpExpr expr) { return RetTy(); }
  RetTy visitAddExpr(AffineBinaryOpExpr expr) {
    return static_cast<SubClass *>(this)->visitAffineBinaryOpExpr(expr);
  }
  RetTy visitMulExpr(AffineBinaryOpExpr expr) {
    return static_cast<SubClass *>(this)->visitAffineBinaryOpExpr(expr);
  }
  RetTy visitModExpr(AffineBinaryOpExpr expr) {
    return static_cast<SubClass *>(this)->visitAffineBinaryOpExpr(expr);
  }
  RetTy visitFloorDivExpr(AffineBinaryOpExpr expr) {
    return static_cast<SubClass *>(this)->visitAffineBinaryOpExpr(expr);
  }
  RetTy visitCeilDivExpr(AffineBinaryOpExpr expr) {
    return static_cast<SubClass *>(this)->visitAffineBinaryOpExpr(expr);
  }
  RetTy visitConstantExpr(AffineConstantExpr expr) { return RetTy(); }
  RetTy visitDimExpr(AffineDimExpr expr) { return RetTy(); }
  RetTy visitSymbolExpr(AffineSymbolExpr expr) { return RetTy(); }
};

/// See documentation for AffineExprVisitorBase. This visitor supports
/// interrupting walks when a `WalkResult` is used for `RetTy`.
template <typename SubClass, typename RetTy = void>
class AffineExprVisitor : public AffineExprVisitorBase<SubClass, RetTy> {
````
- **EN**: This C++ declaration introduces `AffineExprVisitor` and establishes part of the API surface for `AffineExprVisitor`. Representative entry points here include `visitAffineBinaryOpExpr`, `RetTy`, `visitAddExpr`, `visitMulExpr`.
- **CN**: 该 C++ 声明引入了 `AffineExprVisitor`，并构成 `AffineExprVisitor` API 表面的一部分。 这一段可见的代表性接口包括 `visitAffineBinaryOpExpr`, `RetTy`, `visitAddExpr`, `visitMulExpr`。

### Lines 145-213
````cpp
  //===--------------------------------------------------------------------===//
  // Interface code - This is the public interface of the AffineExprVisitor
  // that you use to visit affine expressions...
public:
  // Function to walk an AffineExpr (in post order).
  RetTy walkPostOrder(AffineExpr expr) {
    static_assert(std::is_base_of<AffineExprVisitor, SubClass>::value,
                  "Must instantiate with a derived type of AffineExprVisitor");
    auto self = static_cast<SubClass *>(this);
    switch (expr.getKind()) {
    case AffineExprKind::Add: {
      auto binOpExpr = cast<AffineBinaryOpExpr>(expr);
      if constexpr (std::is_same<RetTy, WalkResult>::value) {
        if (walkOperandsPostOrder(binOpExpr).wasInterrupted())
          return WalkResult::interrupt();
      } else {
        walkOperandsPostOrder(binOpExpr);
      }
      return self->visitAddExpr(binOpExpr);
    }
    case AffineExprKind::Mul: {
      auto binOpExpr = cast<AffineBinaryOpExpr>(expr);
      if constexpr (std::is_same<RetTy, WalkResult>::value) {
        if (walkOperandsPostOrder(binOpExpr).wasInterrupted())
          return WalkResult::interrupt();
      } else {
        walkOperandsPostOrder(binOpExpr);
      }
      return self->visitMulExpr(binOpExpr);
    }
    case AffineExprKind::Mod: {
      auto binOpExpr = cast<AffineBinaryOpExpr>(expr);
      if constexpr (std::is_same<RetTy, WalkResult>::value) {
        if (walkOperandsPostOrder(binOpExpr).wasInterrupted())
          return WalkResult::interrupt();
      } else {
        walkOperandsPostOrder(binOpExpr);
      }
      return self->visitModExpr(binOpExpr);
    }
    case AffineExprKind::FloorDiv: {
      auto binOpExpr = cast<AffineBinaryOpExpr>(expr);
      if constexpr (std::is_same<RetTy, WalkResult>::value) {
        if (walkOperandsPostOrder(binOpExpr).wasInterrupted())
          return WalkResult::interrupt();
      } else {
        walkOperandsPostOrder(binOpExpr);
      }
      return self->visitFloorDivExpr(binOpExpr);
    }
    case AffineExprKind::CeilDiv: {
      auto binOpExpr = cast<AffineBinaryOpExpr>(expr);
      if constexpr (std::is_same<RetTy, WalkResult>::value) {
        if (walkOperandsPostOrder(binOpExpr).wasInterrupted())
          return WalkResult::interrupt();
      } else {
        walkOperandsPostOrder(binOpExpr);
      }
      return self->visitCeilDivExpr(binOpExpr);
    }
    case AffineExprKind::Constant:
      return self->visitConstantExpr(cast<AffineConstantExpr>(expr));
    case AffineExprKind::DimId:
      return self->visitDimExpr(cast<AffineDimExpr>(expr));
    case AffineExprKind::SymbolId:
      return self->visitSymbolExpr(cast<AffineSymbolExpr>(expr));
    }
    llvm_unreachable("Unknown AffineExpr");
  }
````
- **EN**: This block groups callable interfaces such as `walkPostOrder`, `static_assert`, `getKind`, `constexpr`, indicating how `AffineExprVisitor` is queried or updated.
- **CN**: 该代码块聚合了 `walkPostOrder`, `static_assert`, `getKind`, `constexpr` 等可调用接口，展示了如何查询或更新 `AffineExprVisitor`。

### Lines 214-235
````cpp
private:
  // Walk the operands - each operand is itself walked in post order.
  RetTy walkOperandsPostOrder(AffineBinaryOpExpr expr) {
    if constexpr (std::is_same<RetTy, WalkResult>::value) {
      if (walkPostOrder(expr.getLHS()).wasInterrupted())
        return WalkResult::interrupt();
    } else {
      walkPostOrder(expr.getLHS());
    }
    if constexpr (std::is_same<RetTy, WalkResult>::value) {
      if (walkPostOrder(expr.getRHS()).wasInterrupted())
        return WalkResult::interrupt();
      return WalkResult::advance();
    } else {
      return walkPostOrder(expr.getRHS());
    }
  }
};

template <typename SubClass>
class AffineExprVisitor<SubClass, LogicalResult>
    : public AffineExprVisitorBase<SubClass, LogicalResult> {
````
- **EN**: This C++ declaration introduces `AffineExprVisitor` and establishes part of the API surface for `AffineExprVisitor`. Representative entry points here include `walkOperandsPostOrder`, `constexpr`, `walkPostOrder`, `getLHS`.
- **CN**: 该 C++ 声明引入了 `AffineExprVisitor`，并构成 `AffineExprVisitor` API 表面的一部分。 这一段可见的代表性接口包括 `walkOperandsPostOrder`, `constexpr`, `walkPostOrder`, `getLHS`。

### Lines 237-285
````cpp
  //===--------------------------------------------------------------------===//
  // Interface code - This is the public interface of the AffineExprVisitor
  // that you use to visit affine expressions...
public:
  // Function to walk an AffineExpr (in post order).
  LogicalResult walkPostOrder(AffineExpr expr) {
    static_assert(std::is_base_of<AffineExprVisitor, SubClass>::value,
                  "Must instantiate with a derived type of AffineExprVisitor");
    auto self = static_cast<SubClass *>(this);
    switch (expr.getKind()) {
    case AffineExprKind::Add: {
      auto binOpExpr = cast<AffineBinaryOpExpr>(expr);
      if (failed(walkOperandsPostOrder(binOpExpr)))
        return failure();
      return self->visitAddExpr(binOpExpr);
    }
    case AffineExprKind::Mul: {
      auto binOpExpr = cast<AffineBinaryOpExpr>(expr);
      if (failed(walkOperandsPostOrder(binOpExpr)))
        return failure();
      return self->visitMulExpr(binOpExpr);
    }
    case AffineExprKind::Mod: {
      auto binOpExpr = cast<AffineBinaryOpExpr>(expr);
      if (failed(walkOperandsPostOrder(binOpExpr)))
        return failure();
      return self->visitModExpr(binOpExpr);
    }
    case AffineExprKind::FloorDiv: {
      auto binOpExpr = cast<AffineBinaryOpExpr>(expr);
      if (failed(walkOperandsPostOrder(binOpExpr)))
        return failure();
      return self->visitFloorDivExpr(binOpExpr);
    }
    case AffineExprKind::CeilDiv: {
      auto binOpExpr = cast<AffineBinaryOpExpr>(expr);
      if (failed(walkOperandsPostOrder(binOpExpr)))
        return failure();
      return self->visitCeilDivExpr(binOpExpr);
    }
    case AffineExprKind::Constant:
      return self->visitConstantExpr(cast<AffineConstantExpr>(expr));
    case AffineExprKind::DimId:
      return self->visitDimExpr(cast<AffineDimExpr>(expr));
    case AffineExprKind::SymbolId:
      return self->visitSymbolExpr(cast<AffineSymbolExpr>(expr));
    }
    llvm_unreachable("Unknown AffineExpr");
  }
````
- **EN**: This block groups callable interfaces such as `walkPostOrder`, `static_assert`, `getKind`, `failed`, indicating how `AffineExprVisitor` is queried or updated.
- **CN**: 该代码块聚合了 `walkPostOrder`, `static_assert`, `getKind`, `failed` 等可调用接口，展示了如何查询或更新 `AffineExprVisitor`。

### Lines 286-354
````cpp
private:
  // Walk the operands - each operand is itself walked in post order.
  LogicalResult walkOperandsPostOrder(AffineBinaryOpExpr expr) {
    if (failed(walkPostOrder(expr.getLHS())))
      return failure();
    if (failed(walkPostOrder(expr.getRHS())))
      return failure();
    return success();
  }
};

// This class is used to flatten a pure affine expression (AffineExpr,
// which is in a tree form) into a sum of products (w.r.t constants) when
// possible, and in that process simplifying the expression. For a modulo,
// floordiv, or a ceildiv expression, an additional identifier, called a local
// identifier, is introduced to rewrite the expression as a sum of product
// affine expression. Each local identifier is always and by construction a
// floordiv of a pure add/mul affine function of dimensional, symbolic, and
// other local identifiers, in a non-mutually recursive way. Hence, every local
// identifier can ultimately always be recovered as an affine function of
// dimensional and symbolic identifiers (involving floordiv's); note however
// that by AffineExpr construction, some floordiv combinations are converted to
// mod's. The result of the flattening is a flattened expression and a set of
// constraints involving just the local variables.
//
// d2 + (d0 + d1) floordiv 4  is flattened to d2 + q where 'q' is the local
// variable introduced, with localVarCst containing 4*q <= d0 + d1 <= 4*q + 3.
//
// The simplification performed includes the accumulation of contributions for
// each dimensional and symbolic identifier together, the simplification of
// floordiv/ceildiv/mod expressions and other simplifications that in turn
// happen as a result. A simplification that this flattening naturally performs
// is of simplifying the numerator and denominator of floordiv/ceildiv, and
// folding a modulo expression to a zero, if possible. Three examples are below:
//
// (d0 + 3 * d1) + d0) - 2 * d1) - d0    simplified to     d0 + d1
// (d0 - d0 mod 4 + 4) mod 4             simplified to     0
// (3*d0 + 2*d1 + d0) floordiv 2 + d1    simplified to     2*d0 + 2*d1
//
// The way the flattening works for the second example is as follows: d0 % 4 is
// replaced by d0 - 4*q with q being introduced: the expression then simplifies
// to: (d0 - (d0 - 4q) + 4) = 4q + 4, modulo of which w.r.t 4 simplifies to
// zero. Note that an affine expression may not always be expressible purely as
// a sum of products involving just the original dimensional and symbolic
// identifiers due to the presence of modulo/floordiv/ceildiv expressions that
// may not be eliminated after simplification; in such cases, the final
// expression can be reconstructed by replacing the local identifiers with their
// corresponding explicit form stored in 'localExprs' (note that each of the
// explicit forms itself would have been simplified).
//
// The expression walk method here performs a linear time post order walk that
// performs the above simplifications through visit methods, with partial
// results being stored in 'operandExprStack'. When a parent expr is visited,
// the flattened expressions corresponding to its two operands would already be
// on the stack - the parent expression looks at the two flattened expressions
// and combines the two. It pops off the operand expressions and pushes the
// combined result (although this is done in-place on its LHS operand expr).
// When the walk is completed, the flattened form of the top-level expression
// would be left on the stack.
//
// A flattener can be repeatedly used for multiple affine expressions that bind
// to the same operands, for example, for all result expressions of an
// AffineMap or AffineValueMap. In such cases, using it for multiple expressions
// is more efficient than creating a new flattener for each expression since
// common identical div and mod expressions appearing across different
// expressions are mapped to the same local identifier (same column position in
// 'localVarCst').
class SimpleAffineExprFlattener
    : public AffineExprVisitor<SimpleAffineExprFlattener, LogicalResult> {
````
- **EN**: This C++ declaration introduces `SimpleAffineExprFlattener` and establishes part of the API surface for `AffineExprVisitor`. Representative entry points here include `walkOperandsPostOrder`, `failed`, `walkPostOrder`, `getLHS`.
- **CN**: 该 C++ 声明引入了 `SimpleAffineExprFlattener`，并构成 `AffineExprVisitor` API 表面的一部分。 这一段可见的代表性接口包括 `walkOperandsPostOrder`, `failed`, `walkPostOrder`, `getLHS`。

### Lines 356-407
````cpp
public:
  // Flattened expression layout: [dims, symbols, locals, constant]
  // Stack that holds the LHS and RHS operands while visiting a binary op expr.
  // In the future, consider adding a prepass to determine how big the
  // SmallVector's will be, and linearize this to std::vector<int64_t> to
  // prevent SmallVector moves on re-allocation.
  std::vector<SmallVector<int64_t, 8>> operandExprStack;

  unsigned numDims;
  unsigned numSymbols;

  // Number of newly introduced identifiers to flatten mod/floordiv/ceildiv's.
  unsigned numLocals;

  // AffineExpr's corresponding to the floordiv/ceildiv/mod expressions for
  // which new identifiers were introduced; if the latter do not get canceled
  // out, these expressions can be readily used to reconstruct the AffineExpr
  // (tree) form. Note that these expressions themselves would have been
  // simplified (recursively) by this pass. Eg. d0 + (d0 + 2*d1 + d0) ceildiv 4
  // will be simplified to d0 + q, where q = (d0 + d1) ceildiv 2. (d0 + d1)
  // ceildiv 2 would be the local expression stored for q.
  SmallVector<AffineExpr, 4> localExprs;

  SimpleAffineExprFlattener(unsigned numDims, unsigned numSymbols);

  virtual ~SimpleAffineExprFlattener() = default;

  // Visitor method overrides.
  LogicalResult visitMulExpr(AffineBinaryOpExpr expr);
  LogicalResult visitAddExpr(AffineBinaryOpExpr expr);
  LogicalResult visitDimExpr(AffineDimExpr expr);
  LogicalResult visitSymbolExpr(AffineSymbolExpr expr);
  LogicalResult visitConstantExpr(AffineConstantExpr expr);
  LogicalResult visitCeilDivExpr(AffineBinaryOpExpr expr);
  LogicalResult visitFloorDivExpr(AffineBinaryOpExpr expr);

  //
  // t = expr mod c   <=>  t = expr - c*q and c*q <= expr <= c*q + c - 1
  //
  // A mod expression "expr mod c" is thus flattened by introducing a new local
  // variable q (= expr floordiv c), such that expr mod c is replaced with
  // 'expr - c * q' and c * q <= expr <= c * q + c - 1 are added to localVarCst.
  LogicalResult visitModExpr(AffineBinaryOpExpr expr);

protected:
  // Add a local identifier (needed to flatten a mod, floordiv, ceildiv expr).
  // The local identifier added is always a floordiv of a pure add/mul affine
  // function of other identifiers, coefficients of which are specified in
  // dividend and with respect to a positive constant divisor. localExpr is the
  // simplified tree expression (AffineExpr) corresponding to the quantifier.
  virtual void addLocalFloorDivId(ArrayRef<int64_t> dividend, int64_t divisor,
                                  AffineExpr localExpr);
````
- **EN**: This block groups callable interfaces such as `SimpleAffineExprFlattener`, `visitMulExpr`, `visitAddExpr`, `visitDimExpr`, indicating how `AffineExprVisitor` is queried or updated.
- **CN**: 该代码块聚合了 `SimpleAffineExprFlattener`, `visitMulExpr`, `visitAddExpr`, `visitDimExpr` 等可调用接口，展示了如何查询或更新 `AffineExprVisitor`。

### Lines 408-451
````cpp
  /// Add a local identifier (needed to flatten a mod, floordiv, ceildiv, mul
  /// expr) when the rhs is a symbolic expression. The local identifier added
  /// may be a floordiv, ceildiv, mul or mod of a pure affine/semi-affine
  /// function of other identifiers, coefficients of which are specified in the
  /// lhs of the mod, floordiv, ceildiv or mul expression and with respect to a
  /// symbolic rhs expression. `localExpr` is the simplified tree expression
  /// (AffineExpr) corresponding to the quantifier.
  virtual LogicalResult addLocalIdSemiAffine(ArrayRef<int64_t> lhs,
                                             ArrayRef<int64_t> rhs,
                                             AffineExpr localExpr);

private:
  /// Adds `localExpr`, which may be mod, ceildiv, floordiv or mod expression
  /// representing the affine expression corresponding to the quantifier
  /// introduced as the local variable corresponding to `localExpr`. If the
  /// quantifier is already present, we put the coefficient in the proper index
  /// of `result`, otherwise we add a new local variable and put the coefficient
  /// there.
  LogicalResult addLocalVariableSemiAffine(ArrayRef<int64_t> lhs,
                                           ArrayRef<int64_t> rhs,
                                           AffineExpr localExpr,
                                           SmallVectorImpl<int64_t> &result,
                                           unsigned long resultSize);

  // t = expr floordiv c   <=> t = q, c * q <= expr <= c * q + c - 1
  // A floordiv is thus flattened by introducing a new local variable q, and
  // replacing that expression with 'q' while adding the constraints
  // c * q <= expr <= c * q + c - 1 to localVarCst (done by
  // IntegerRelation::addLocalFloorDiv).
  //
  // A ceildiv is similarly flattened:
  // t = expr ceildiv c   <=> t =  (expr + c - 1) floordiv c
  LogicalResult visitDivExpr(AffineBinaryOpExpr expr, bool isCeil);

  int findLocalId(AffineExpr localExpr);

  inline unsigned getNumCols() const {
    return numDims + numSymbols + numLocals + 1;
  }
  inline unsigned getConstantIndex() const { return getNumCols() - 1; }
  inline unsigned getLocalVarStartIndex() const { return numDims + numSymbols; }
  inline unsigned getSymbolStartIndex() const { return numDims; }
  inline unsigned getDimStartIndex() const { return 0; }
};
````
- **EN**: This block groups callable interfaces such as `addLocalIdSemiAffine`, `addLocalVariableSemiAffine`, `visitDivExpr`, `findLocalId`, indicating how `AffineExprVisitor` is queried or updated.
- **CN**: 该代码块聚合了 `addLocalIdSemiAffine`, `addLocalVariableSemiAffine`, `visitDivExpr`, `findLocalId` 等可调用接口，展示了如何查询或更新 `AffineExprVisitor`。

### Lines 453-453
````cpp
} // namespace mlir
````
- **EN**: This section focuses on } // namespace mlir, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“} // namespace mlir”这一主题，把相关声明与辅助接口组织在一起。

### Lines 456-456
````cpp
#endif // MLIR_IR_AFFINEEXPRVISITOR_H
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: C++ declaration surface for MLIR infrastructure
  **CN**: MLIR 基础设施的 C++ 声明层

## Dependencies / 依赖关系

- mlir/IR/AffineExpr.h
- mlir/Support/LLVM.h
- llvm/ADT/ArrayRef.h
- AffineExprVisitor inherits from public AffineExprVisitorBase<SubClass, RetTy>
- SimpleAffineExprFlattener inherits from public AffineExprVisitor<SimpleAffineExprFlattener, LogicalResult>
- AffineExprVisitor builds on public AffineExprVisitorBase<SubClass, RetTy>
- SimpleAffineExprFlattener builds on public AffineExprVisitor<SimpleAffineExprFlattener, LogicalResult>
