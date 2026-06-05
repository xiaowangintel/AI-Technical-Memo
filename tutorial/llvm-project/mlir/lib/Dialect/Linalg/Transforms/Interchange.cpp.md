# Interchange.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Linalg/Transforms/Interchange.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements the linalg interchange transformation.
  - **CN**: 该文件位于 `mlir/lib/Dialect/Linalg/Transforms`，围绕 Linalg 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- Interchange.cpp - Linalg interchange transformation ----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-12
```cpp
//
// This file implements the linalg interchange transformation.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 13-22
```cpp
#include "mlir/Dialect/Affine/IR/AffineOps.h"
#include "mlir/Dialect/Linalg/IR/Linalg.h"
#include "mlir/Dialect/Linalg/Transforms/Transforms.h"
#include "mlir/Dialect/Linalg/Utils/Utils.h"
#include "mlir/Dialect/Utils/IndexingUtils.h"
#include "mlir/Dialect/Utils/StructuredOpsUtils.h"
#include "mlir/Dialect/Vector/IR/VectorOps.h"
#include "mlir/IR/AffineExpr.h"
#include "mlir/IR/PatternMatch.h"
#include "mlir/Support/LLVM.h"
```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/Dialect/Linalg/Utils/Utils.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/Dialect/Linalg/Utils/Utils.h`。

### Lines 23-29
```cpp
#include "llvm/ADT/ScopeExit.h"

#define DEBUG_TYPE "linalg-interchange"

using namespace mlir;
using namespace mlir::linalg;

```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/ADT/ScopeExit.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/ADT/ScopeExit.h`。

### Lines 30-39
```cpp
static LogicalResult
interchangeGenericOpPrecondition(GenericOp genericOp,
                                 ArrayRef<unsigned> interchangeVector) {
  // Interchange vector must be non-empty and match the number of loops.
  if (interchangeVector.empty() ||
      genericOp.getNumLoops() != interchangeVector.size())
    return failure();
  // Permutation map must be invertible.
  if (!inversePermutation(AffineMap::getPermutationMap(interchangeVector,
                                                       genericOp.getContext())))
```
- **EN**: Implements logic around `interchangeGenericOpPrecondition`, `empty`, `getNumLoops`, `failure`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `interchangeGenericOpPrecondition`, `empty`, `getNumLoops`, `failure`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 40-49
```cpp
    return failure();
  return success();
}

FailureOr<GenericOp>
mlir::linalg::interchangeGenericOp(RewriterBase &rewriter, GenericOp genericOp,
                                   ArrayRef<unsigned> interchangeVector) {
  if (failed(interchangeGenericOpPrecondition(genericOp, interchangeVector)))
    return rewriter.notifyMatchFailure(genericOp, "preconditions not met");

```
- **EN**: Implements logic around `failure`, `success`, `interchangeGenericOp`, `failed`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `failure`, `success`, `interchangeGenericOp`, `failed`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 50-56
```cpp
  // 1. Compute the inverse permutation map, it must be non-null since the
  // preconditions are satisfied.
  MLIRContext *context = genericOp.getContext();
  AffineMap permutationMap = inversePermutation(
      AffineMap::getPermutationMap(interchangeVector, context));
  assert(permutationMap && "unexpected null map");

```
- **EN**: Implements logic around `getContext`, `inversePermutation`, `getPermutationMap`, `assert`.
- **CN**: 围绕 `getContext`, `inversePermutation`, `getPermutationMap`, `assert` 实现具体逻辑。

### Lines 57-66
```cpp
  // Start a guarded inplace update.
  rewriter.startOpModification(genericOp);
  llvm::scope_exit guard([&]() { rewriter.finalizeOpModification(genericOp); });

  // 2. Compute the interchanged indexing maps.
  SmallVector<AffineMap> newIndexingMaps;
  for (OpOperand &opOperand : genericOp->getOpOperands()) {
    AffineMap m = genericOp.getMatchingIndexingMap(&opOperand);
    if (!permutationMap.isEmpty())
      m = m.compose(permutationMap);
```
- **EN**: Implements logic around `startOpModification`, `guard`, `getOpOperands`, `getMatchingIndexingMap`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `startOpModification`, `guard`, `getOpOperands`, `getMatchingIndexingMap`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 67-71
```cpp
    newIndexingMaps.push_back(m);
  }
  genericOp.setIndexingMapsAttr(
      rewriter.getAffineMapArrayAttr(newIndexingMaps));

```
- **EN**: Implements logic around `push_back`, `setIndexingMapsAttr`, `getAffineMapArrayAttr`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `push_back`, `setIndexingMapsAttr`, `getAffineMapArrayAttr` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 72-79
```cpp
  // 3. Compute the interchanged iterator types.
  ArrayRef<Attribute> itTypes = genericOp.getIteratorTypes().getValue();
  SmallVector<Attribute> itTypesVector;
  llvm::append_range(itTypesVector, itTypes);
  SmallVector<int64_t> permutation(interchangeVector);
  applyPermutationToVector(itTypesVector, permutation);
  genericOp.setIteratorTypesAttr(rewriter.getArrayAttr(itTypesVector));

```
- **EN**: Implements logic around `getIteratorTypes`, `append_range`, `permutation`, `applyPermutationToVector`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getIteratorTypes`, `append_range`, `permutation`, `applyPermutationToVector`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 80-89
```cpp
  // 4. Transform the index operations by applying the permutation map.
  if (genericOp.hasIndexSemantics()) {
    OpBuilder::InsertionGuard guard(rewriter);
    for (IndexOp indexOp :
         llvm::make_early_inc_range(genericOp.getBody()->getOps<IndexOp>())) {
      rewriter.setInsertionPoint(indexOp);
      SmallVector<Value> allIndices;
      allIndices.reserve(genericOp.getNumLoops());
      llvm::transform(llvm::seq<uint64_t>(0, genericOp.getNumLoops()),
                      std::back_inserter(allIndices), [&](uint64_t dim) {
```
- **EN**: Implements logic around `hasIndexSemantics`, `guard`, `make_early_inc_range`, `setInsertionPoint`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `hasIndexSemantics`, `guard`, `make_early_inc_range`, `setInsertionPoint`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 90-97
```cpp
                        return IndexOp::create(rewriter, indexOp->getLoc(),
                                               dim);
                      });
      rewriter.replaceOpWithNewOp<affine::AffineApplyOp>(
          indexOp, permutationMap.getSubMap(indexOp.getDim()), allIndices);
    }
  }

```
- **EN**: Implements logic around `create`, `AffineApplyOp>`, `getSubMap`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `AffineApplyOp>`, `getSubMap` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 98-99
```cpp
  return genericOp;
}
```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

## Key Concepts / 关键概念

- **Rewrite and lowering / 重写与 lowering**:
  - **EN**: Uses MLIR pattern infrastructure or passes to canonicalize, legalize, or lower operations.
  - **CN**: 使用 MLIR 模式基础设施或 Pass 对操作进行规范化、合法化或 lowering。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/Dialect/Linalg/Utils/Utils.h`, `mlir/Dialect/Utils/IndexingUtils.h`, `mlir/Dialect/Utils/StructuredOpsUtils.h`, `mlir/Dialect/Vector/IR/VectorOps.h`, `mlir/IR/AffineExpr.h`, `mlir/IR/PatternMatch.h`, `mlir/Support/LLVM.h` ... (+1 more)
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (7), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (2), support utilities and LLVM interop helpers / 支持工具与 LLVM 互操作辅助逻辑 (1), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (1)
