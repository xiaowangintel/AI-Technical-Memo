# X86Utils.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/X86/Utils/X86Utils.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR X86Utils component.
- **用途（CN）**: 声明 MLIR X86Utils 组件相关的 C++ 接口与辅助类型。

## Line-by-Line Analysis / 逐行分析

### Lines 1-19
````cpp
//===- X86Utils.h - X86 Utilities -------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_X86_UTILS_X86UTILS_H_
#define MLIR_DIALECT_X86_UTILS_X86UTILS_H_

#include "mlir/Dialect/Vector/IR/VectorOps.h"
#include "mlir/IR/PatternMatch.h"
#include "mlir/IR/Value.h"
#include "mlir/Support/LogicalResult.h"
#include "llvm/ADT/ArrayRef.h"
#include <cstdint>
#include <optional>
#include <string>
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 21-67
````cpp
namespace mlir {
class AffineMap;
class Operation;

namespace x86 {

// Return true if the operation is in VNNI layout.
// Optionally, the check can be constrained to a specific VNNI blocking factor.
bool isInVnniLayout(Operation *op, llvm::ArrayRef<AffineMap> indexingMaps,
                    std::optional<unsigned> blockingFactor = std::nullopt);

// Returns true if two contraction ops form a valid pair for VNNI packing.
// It verifies that both contractions share the appropriate operand, read from
// the same source buffer, and use constant indices that differ by 8 or 16.
bool validatePairVectorContract(vector::ContractionOp contractOp,
                                vector::ContractionOp pairContOp,
                                bool rhsHasMultipleNonUnitDims,
                                int64_t nonUnitDimValue);

// Walks backward from a value to find its originating vector read-like op
// (vector.transfer_read or vector.load), following scf.for iter-args but
// stopping at layout-transforming ops; returns the read op or nullptr.
Operation *traceToVectorReadLikeParentOperation(Value v);

// Recursively traces a value to find a downstream vector write-like op
// (vector.transfer_write or vector.store), crossing scf.for/yield but
// stopping at layout-altering ops. Returns nullptr if no vector writer/store
// ops or there are multiple users.
Operation *traceToVectorWriteLikeUserOperation(Value v);

// Packs the accumulators of two flat BF16 vector.contraction ops into a
// VNNI-packed layout and replaces the original accumulators to enable post-read
// packing transformations.
LogicalResult shuffleAfterReadLikeOp(PatternRewriter &rewriter, Operation *opA,
                                     Operation *opB,
                                     vector::ContractionOp contractA,
                                     vector::ContractionOp contractB,
                                     int64_t nonUnitDimAcc, VectorType accTy);

// Shuffles vectors produced by vector.contraction ops into a flat layout
// before they are written to memory.
LogicalResult shuffleBeforeWriteLikeOp(PatternRewriter &rewriter,
                                       Operation *opA, Operation *opB,
                                       int64_t nonUnitDimAcc, VectorType accTy);

} // namespace x86
} // namespace mlir
````
- **EN**: This C++ declaration introduces `AffineMap` and establishes part of the API surface for `X86Utils`. Representative entry points here include `isInVnniLayout`, `validatePairVectorContract`, `traceToVectorReadLikeParentOperation`, `traceToVectorWriteLikeUserOperation`.
- **CN**: 该 C++ 声明引入了 `AffineMap`，并构成 `X86Utils` API 表面的一部分。 这一段可见的代表性接口包括 `isInVnniLayout`, `validatePairVectorContract`, `traceToVectorReadLikeParentOperation`, `traceToVectorWriteLikeUserOperation`。

### Lines 68-68
````cpp
#endif // MLIR_DIALECT_X86_UTILS_X86UTILS_H_
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: C++ declaration surface for MLIR infrastructure
  **CN**: MLIR 基础设施的 C++ 声明层
- **EN**: Type or attribute abstraction
  **CN**: 类型或属性抽象

## Dependencies / 依赖关系

- mlir/Dialect/Vector/IR/VectorOps.h
- mlir/IR/PatternMatch.h
- mlir/IR/Value.h
- mlir/Support/LogicalResult.h
- llvm/ADT/ArrayRef.h
