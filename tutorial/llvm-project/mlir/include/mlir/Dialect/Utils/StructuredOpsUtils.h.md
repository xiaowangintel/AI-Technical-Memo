# StructuredOpsUtils.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/Utils/StructuredOpsUtils.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR StructuredOpsUtils component. The leading comments describe it as: This header file define utilities that operate on builtin types and are.
- **用途（CN）**: 声明 MLIR StructuredOpsUtils 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-27
````cpp
//===- StructuredOpsUtils.h - Utilities used by structured ops --*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This header file define utilities that operate on builtin types and are
// useful across multiple dialects that use structured ops abstractions. These
// abstractions consist of define custom operations that encode and transport
// information about their semantics (e.g. type of iterators like parallel,
// reduction, etc..) as attributes.
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_UTILS_STRUCTUREDOPSUTILS_H
#define MLIR_DIALECT_UTILS_STRUCTUREDOPSUTILS_H

#include "mlir/IR/AffineMap.h"
#include "mlir/IR/BuiltinAttributes.h"
#include "mlir/IR/Location.h"
#include "mlir/IR/TypeRange.h"
#include "mlir/Support/LLVM.h"

// Pull in all enum type definitions and utility function declarations.
#include "mlir/Dialect/Utils/DialectUtilsEnums.h.inc"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 29-74
````cpp
namespace mlir {

class OpBuilder;
class RewriterBase;

/// Tests whether the given maps describe a row major matmul. The test is
/// permutation-invariant. Note that this only checks the affine maps from an
/// operation, so does not perform any checks on the math being performed within
/// the reduction.
bool isRowMajorMatmul(ArrayAttr indexingMaps);

/// Tests whether the given maps describe a column major matmul. The test is
/// permutation-invariant. Note that this only checks the affine maps from an
/// operation, so does not perform any checks on the math being performed within
/// the reduction.
bool isColumnMajorMatmul(ArrayAttr indexingMaps);

/// Tests whether the given maps describe a row major batch matmul. The test is
/// permutation-invariant. Note that this only checks the affine maps from an
/// operation, so does not perform any checks on the math being performed within
/// the reduction.
bool isRowMajorBatchMatmul(ArrayAttr indexingMaps);

/// Tests whether the given maps describe a vector matrix multiplication. The
/// test is permutation-invariant. Note that this only checks the affine maps
/// from an operation, so does not perform any checks on the math being
/// performed within the reduction.
bool isVecmat(ArrayAttr indexingMaps);

/// Tests whether the given maps describe a batch vector matrix multiplication.
/// The test is permutation-invariant. Note that this only checks the affine
/// maps from an operation, so does not perform any checks on the math being
/// performed within the reduction.
bool isBatchVecmat(ArrayAttr indexingMaps);

/// Tests whether the given maps describe a matrix vector multiplication. The
/// test is permutation-invariant. Note that this only checks the affine maps
/// from an operation, so does not perform any checks on the math being
/// performed within the reduction.
bool isMatvec(ArrayAttr indexingMaps);

/// Tests whether the given maps describe a batch matrix vector multiplication.
/// The test is permutation-invariant. Note that this only checks the affine
/// maps from an operation, so does not perform any checks on the math being
/// performed within the reduction.
bool isBatchMatvec(ArrayAttr indexingMaps);
````
- **EN**: This C++ declaration introduces `OpBuilder` and establishes part of the API surface for `StructuredOpsUtils`. Representative entry points here include `isRowMajorMatmul`, `isColumnMajorMatmul`, `isRowMajorBatchMatmul`, `isVecmat`.
- **CN**: 该 C++ 声明引入了 `OpBuilder`，并构成 `StructuredOpsUtils` API 表面的一部分。 这一段可见的代表性接口包括 `isRowMajorMatmul`, `isColumnMajorMatmul`, `isRowMajorBatchMatmul`, `isVecmat`。

### Lines 75-120
````cpp
/// Return positions in `iteratorTypes` that match `iteratorTypeName`.
inline void findPositionsOfType(ArrayRef<utils::IteratorType> iteratorTypes,
                                utils::IteratorType iteratorTypeName,
                                SmallVectorImpl<unsigned> &res) {
  for (const auto &en : llvm::enumerate(iteratorTypes)) {
    if (en.value() == iteratorTypeName)
      res.push_back(en.index());
  }
}

/// Helper StructuredGenerator class to manipulate and rewrite ops with
/// `StructuredOpInterface`. This is templated for now because VectorOps do not
/// yet implement the StructuredOpInterface itself.
template <typename StructuredOpInterface, typename IteratorTypeT>
class StructuredGenerator {
public:
  using MapList = ArrayRef<ArrayRef<AffineExpr>>;

  struct IteratorType {
    IteratorType(IteratorTypeT iter) : iter(iter) {}
    bool isOfType(IteratorTypeT expectedIter) const {
      return expectedIter == iter;
    }
    IteratorTypeT iter;
  };
  struct Par : public IteratorType {
    Par() : IteratorType(IteratorTypeT::parallel) {}
  };
  struct Red : public IteratorType {
    Red() : IteratorType(IteratorTypeT::reduction) {}
  };

  StructuredGenerator(RewriterBase &rewriter, StructuredOpInterface op)
      : rewriter(rewriter), ctx(op.getContext()), loc(op.getLoc()),
        iterators(op.getIteratorTypesArray()), maps(op.getIndexingMapsArray()),
        op(op) {}

  bool iters(ArrayRef<IteratorType> its) {
    if (its.size() != iterators.size())
      return false;
    for (int i = 0, e = its.size(); i != e; ++i) {
      if (!its[i].isOfType(iterators[i]))
        return false;
    }
    return true;
  }
````
- **EN**: This C++ declaration introduces `StructuredGenerator` and establishes part of the API surface for `StructuredOpsUtils`. Representative entry points here include `findPositionsOfType`, `enumerate`, `value`, `push_back`.
- **CN**: 该 C++ 声明引入了 `StructuredGenerator`，并构成 `StructuredOpsUtils` API 表面的一部分。 这一段可见的代表性接口包括 `findPositionsOfType`, `enumerate`, `value`, `push_back`。

### Lines 122-160
````cpp
  bool layout(MapList l) {
    auto infer = [&](MapList m) {
      return AffineMap::inferFromExprList(m, ctx);
    };
    return maps == infer(l);
  }

protected:
  RewriterBase &rewriter;
  MLIRContext *ctx;
  Location loc;
  SmallVector<IteratorTypeT> iterators;
  SmallVector<AffineMap, 4> maps;
  Operation *op;
};

// Clone the current operation with the operands. This is used to abstract away
// the optional underlying region creation.
// Note: this is a true builder that notifies the OpBuilder listener.
Operation *clone(OpBuilder &b, Operation *op, TypeRange newResultTypes,
                 ValueRange newOperands);
template <typename OpT>
OpT clone(OpBuilder &b, OpT op, TypeRange newResultTypes,
          ValueRange newOperands) {
  return cast<OpT>(clone(b, op.getOperation(), newResultTypes, newOperands));
}

// Clone the current operation with the operands but leave the regions empty.
// Note: this is a true builder that notifies the OpBuilder listener.
Operation *cloneWithoutRegions(OpBuilder &b, Operation *op,
                               TypeRange newResultTypes,
                               ValueRange newOperands);

// Get the list of attributes associated with the op, ignoring
// those with the provided name.
SmallVector<NamedAttribute>
getPrunedAttributeList(Operation *op, ArrayRef<StringRef> elidedAttrs);

} // namespace mlir
````
- **EN**: This block groups callable interfaces such as `layout`, `inferFromExprList`, `infer`, `clone`, indicating how `StructuredOpsUtils` is queried or updated.
- **CN**: 该代码块聚合了 `layout`, `inferFromExprList`, `infer`, `clone` 等可调用接口，展示了如何查询或更新 `StructuredOpsUtils`。

### Lines 163-163
````cpp
#endif // MLIR_DIALECT_UTILS_STRUCTUREDOPSUTILS_H
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: C++ declaration surface for MLIR infrastructure
  **CN**: MLIR 基础设施的 C++ 声明层
- **EN**: Iterator-based traversal APIs
  **CN**: 基于迭代器的遍历接口
- **EN**: Region/block ownership and nesting
  **CN**: Region/Block 的所有权与嵌套关系
- **EN**: Type or attribute abstraction
  **CN**: 类型或属性抽象

## Dependencies / 依赖关系

- mlir/IR/AffineMap.h
- mlir/IR/BuiltinAttributes.h
- mlir/IR/Location.h
- mlir/IR/TypeRange.h
- mlir/Support/LLVM.h
- mlir/Dialect/Utils/DialectUtilsEnums.h.inc
