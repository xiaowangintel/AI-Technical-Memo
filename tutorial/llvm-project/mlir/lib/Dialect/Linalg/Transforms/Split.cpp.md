# Split.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Linalg/Transforms/Split.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements Linalg dialect support for rewrite patterns, passes, and IR-to-IR transformation logic, centered on `Split`.
  - **CN**: 实现 Linalg 方言中围绕 `Split` 的重写模式、Pass 以及 IR 到 IR 的变换逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- Split.cpp - Structured op splitting --------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-17
```cpp

#include "mlir/Dialect/Affine/IR/AffineOps.h"
#include "mlir/Dialect/Linalg/Transforms/Transforms.h"
#include "mlir/Dialect/Utils/StaticValueUtils.h"
#include "mlir/IR/AffineExpr.h"
#include "mlir/IR/Attributes.h"
#include "mlir/IR/BuiltinAttributes.h"
#include "mlir/IR/OpDefinition.h"
#include "mlir/Interfaces/TilingInterface.h"
#include "llvm/ADT/SmallVectorExtras.h"
```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/Dialect/Utils/StaticValueUtils.h`, `mlir/IR/AffineExpr.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/Dialect/Utils/StaticValueUtils.h`, `mlir/IR/AffineExpr.h`。

### Lines 18-24
```cpp

#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SmallVector.h"

using namespace mlir;
using namespace mlir::linalg;

```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVector.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVector.h`。

### Lines 25-29
```cpp
/// Creates a part of the given `op` split along the iteration space `dimension`
/// with the given `size` and an optional `offset` (default 0). Makes slices
/// of operands, using the input operands of the original op and the output
/// operands provided as `resultOperands`. Expects `offsets` and `sizes` to
/// define the shape of the iteration space of the original op. Returns the
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 30-39
```cpp
/// split-out op as well as the output operand values updated with the partial
/// results produced by this op through `results`.
static TilingInterface
createSplitPart(RewriterBase &b, Location loc, TilingInterface op,
                ArrayRef<OpFoldResult> offsets, ArrayRef<OpFoldResult> sizes,
                ValueRange resultOperands, unsigned dimension,
                OpFoldResult size, OpFoldResult offset,
                SmallVectorImpl<Value> &results) {
  // Iteration space of the current part.
  SmallVector<OpFoldResult> sizesCopy = llvm::to_vector(sizes);
```
- **EN**: Implements logic around `createSplitPart`, `to_vector`.
- **CN**: 围绕 `createSplitPart`, `to_vector` 实现具体逻辑。

### Lines 40-47
```cpp
  SmallVector<OpFoldResult> offsetsCopy = llvm::to_vector(offsets);
  sizesCopy[dimension] = size;
  offsetsCopy[dimension] = offset;

  // Create the part as if it were a single tile.
  FailureOr<TilingResult> tilingResult =
      op.getTiledImplementation(b, offsetsCopy, sizesCopy);

```
- **EN**: Implements logic around `to_vector`, `getTiledImplementation`.
- **CN**: 围绕 `to_vector`, `getTiledImplementation` 实现具体逻辑。

### Lines 48-57
```cpp
  // Insert the results back and populate the `results` list.
  for (auto [index, result] : llvm::enumerate(tilingResult->tiledValues)) {
    SmallVector<OpFoldResult> resultOffsets, resultSizes;
    if (failed(op.getResultTilePosition(b, index, offsetsCopy, sizesCopy,
                                        resultOffsets, resultSizes)))
      return nullptr;
    SmallVector<OpFoldResult> resultStrides(resultOffsets.size(),
                                            b.getIndexAttr(1));
    Value inserted = tensor::InsertSliceOp::create(
        b, loc, result, resultOperands[index], resultOffsets, resultSizes,
```
- **EN**: Implements logic around `enumerate`, `failed`, `resultStrides`, `getIndexAttr`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `enumerate`, `failed`, `resultStrides`, `getIndexAttr`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 58-66
```cpp
        resultStrides);
    results.push_back(inserted);
  }
  // TODO: this part can be generalized maybe to not expect a single op.
  assert(tilingResult->tiledOps.size() == 1 &&
         "expected split part to return a single tiled operation");
  return cast<TilingInterface>(tilingResult->tiledOps[0]);
}

```
- **EN**: Implements logic around `push_back`, `assert`.
- **CN**: 围绕 `push_back`, `assert` 实现具体逻辑。

### Lines 67-72
```cpp
std::pair<TilingInterface, TilingInterface>
linalg::splitOp(RewriterBase &rewriter, TilingInterface op, unsigned dimension,
                OpFoldResult splitPoint) {
  // Compute the iteration space.
  SmallVector<Range> iterationSpace = op.getIterationDomain(rewriter);

```
- **EN**: Implements logic around `splitOp`, `getIterationDomain`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `splitOp`, `getIterationDomain` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 73-81
```cpp
  // Bail out on dimension overflow.
  if (dimension >= iterationSpace.size())
    return std::make_pair(op, TilingInterface());

  SmallVector<OpFoldResult> offsets = llvm::map_to_vector(
      iterationSpace, [](const Range &range) { return range.offset; });
  SmallVector<OpFoldResult> sizes = llvm::map_to_vector(
      iterationSpace, [](const Range &range) { return range.size; });

```
- **EN**: Implements logic around `size`, `make_pair`, `map_to_vector`.
- **CN**: 围绕 `size`, `make_pair`, `map_to_vector` 实现具体逻辑。

### Lines 82-91
```cpp
  // Adjust the split point so that it doesn't overflow the size.
  AffineExpr d0, d1, d2;
  bindDims(rewriter.getContext(), d0, d1, d2);
  OpFoldResult minSplitPoint = affine::makeComposedFoldedAffineMin(
      rewriter, op.getLoc(),
      AffineMap::inferFromExprList(ArrayRef<AffineExpr>{d0, d1 + d2},
                                   rewriter.getContext())
          .front(),
      {splitPoint, offsets[dimension], sizes[dimension]});

```
- **EN**: Implements logic around `bindDims`, `makeComposedFoldedAffineMin`, `getLoc`, `inferFromExprList`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `bindDims`, `makeComposedFoldedAffineMin`, `getLoc`, `inferFromExprList`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 92-101
```cpp
  // Compute the size of the second part. Return early if the second part would
  // have an empty iteration space.
  OpFoldResult remainingSize = affine::makeComposedFoldedAffineApply(
      rewriter, op.getLoc(), d0 + d1 - d2,
      {iterationSpace[dimension].offset, iterationSpace[dimension].size,
       minSplitPoint});
  if (auto attr = llvm::dyn_cast_if_present<Attribute>(remainingSize)) {
    if (cast<IntegerAttr>(attr).getValue().isZero())
      return {op, TilingInterface()};
  }
```
- **EN**: Implements logic around `makeComposedFoldedAffineApply`, `getLoc`, `dyn_cast_if_present`, `getValue`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `makeComposedFoldedAffineApply`, `getLoc`, `dyn_cast_if_present`, `getValue`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 102-109
```cpp

  // Compute destination tensors.
  SmallVector<Value> destinationTensors;
  LogicalResult destStatus = tensor::getOrCreateDestinations(
      rewriter, op.getLoc(), op, destinationTensors);
  (void)destStatus;
  assert(succeeded(destStatus) && "failed to get destination tensors");

```
- **EN**: Implements logic around `getOrCreateDestinations`, `getLoc`, `assert`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getOrCreateDestinations`, `getLoc`, `assert` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 110-115
```cpp
  // Create the first part.
  SmallVector<Value> firstResults;
  TilingInterface firstPart = createSplitPart(
      rewriter, op.getLoc(), op, offsets, sizes, destinationTensors, dimension,
      minSplitPoint, iterationSpace[dimension].offset, firstResults);

```
- **EN**: Implements logic around `createSplitPart`, `getLoc`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `createSplitPart`, `getLoc` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 116-125
```cpp
  // Need to pretend that the original op now takes as operands firstResults,
  // otherwise tiling interface implementation will take the wrong value to
  // produce data tiles.
  rewriter.modifyOpInPlace(op, [&]() {
    unsigned numTotalOperands = op->getNumOperands();
    unsigned numOutputOperands = firstResults.size();
    op->setOperands(numTotalOperands - numOutputOperands, numOutputOperands,
                    firstResults);
  });

```
- **EN**: Implements logic around `modifyOpInPlace`, `getNumOperands`, `size`, `setOperands`; this block uses rewrite-pattern infrastructure to transform operations; expresses reusable interface-based behavior.
- **CN**: 围绕 `modifyOpInPlace`, `getNumOperands`, `size`, `setOperands` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并表达基于接口的可复用行为。

### Lines 126-133
```cpp
  // Create the second part.
  OpFoldResult totalOffset = affine::makeComposedFoldedAffineApply(
      rewriter, op.getLoc(), d0 + d1, {offsets[dimension], minSplitPoint});
  SmallVector<Value> secondResults;
  TilingInterface secondPart =
      createSplitPart(rewriter, op.getLoc(), op, offsets, sizes, firstResults,
                      dimension, remainingSize, totalOffset, secondResults);

```
- **EN**: Implements logic around `makeComposedFoldedAffineApply`, `getLoc`, `createSplitPart`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `makeComposedFoldedAffineApply`, `getLoc`, `createSplitPart` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 134-141
```cpp
  // Propagate any errors in part creation.
  if (!firstPart || !secondPart)
    return {TilingInterface(), TilingInterface()};

  // Replace the original op with the results of the two newly created ops.
  rewriter.replaceOp(op, secondResults);
  return {firstPart, secondPart};
}
```
- **EN**: Implements logic around `TilingInterface`, `replaceOp`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `TilingInterface`, `replaceOp` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

## Key Concepts / 关键概念

- **Rewrite and lowering / 重写与 lowering**:
  - **EN**: Uses MLIR pattern infrastructure or passes to canonicalize, legalize, or lower operations.
  - **CN**: 使用 MLIR 模式基础设施或 Pass 对操作进行规范化、合法化或 lowering。
- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Models reusable capabilities that can be queried across dialect boundaries.
  - **CN**: 建模可跨方言查询的可复用能力。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/Dialect/Utils/StaticValueUtils.h`, `mlir/IR/AffineExpr.h`, `mlir/IR/Attributes.h`, `mlir/IR/BuiltinAttributes.h`, `mlir/IR/OpDefinition.h`, `mlir/Interfaces/TilingInterface.h`, `llvm/ADT/SmallVectorExtras.h`, `llvm/ADT/STLExtras.h` ... (+1 more)
- **Subsystem categories / 子系统类别**: MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (4), dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (3), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (3), cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (1)
