# Fusion.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Linalg/Transforms/Fusion.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements the linalg dialect Fusion pass.
  - **CN**: 该文件位于 `mlir/lib/Dialect/Linalg/Transforms`，围绕 Linalg 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- Fusion.cpp - Implementation of linalg Fusion -----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-21
```cpp
//
// This file implements the linalg dialect Fusion pass.
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Linalg/IR/Linalg.h"
#include "mlir/Dialect/Linalg/Transforms/Transforms.h"
#include "mlir/Dialect/Linalg/Utils/Utils.h"
#include "mlir/Dialect/MemRef/IR/MemRef.h"
#include "mlir/Dialect/Tensor/IR/Tensor.h"
#include "mlir/Dialect/Tensor/Utils/Utils.h"
#include "mlir/IR/AffineExpr.h"
#include "mlir/IR/AffineMap.h"
#include "mlir/IR/Dominance.h"
```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/Dialect/Linalg/Utils/Utils.h`, `mlir/Dialect/MemRef/IR/MemRef.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/Dialect/Linalg/Utils/Utils.h`, `mlir/Dialect/MemRef/IR/MemRef.h`。

### Lines 22-28
```cpp
#include "mlir/Support/LLVM.h"
#include "llvm/ADT/SmallBitVector.h"
#include "llvm/ADT/SmallVectorExtras.h"
#include "llvm/Support/Debug.h"

#define DEBUG_TYPE "linalg-fusion"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Support/LLVM.h`, `llvm/ADT/SmallBitVector.h`, `llvm/ADT/SmallVectorExtras.h`, `llvm/Support/Debug.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Support/LLVM.h`, `llvm/ADT/SmallBitVector.h`, `llvm/ADT/SmallVectorExtras.h`, `llvm/Support/Debug.h`。

### Lines 29-35
```cpp
using namespace mlir;
using namespace mlir::linalg;

/// Implements a simple high-level fusion pass on linalg structured operations.
///
/// In each block, linalg ops are processed in reverse textual order.
/// Given a linalg op `O`, fusion occurs by:
```
- **EN**: Introduces declarations for `mlir`, `mlir::linalg`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `mlir`, `mlir::linalg` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 36-42
```cpp
///   1. inspecting the linalg ops that write into the views read by `O`. There
///      are 2 cases:
///      a) buffer case: use the SSA value of the views and a simple alias
///         analysis on subview ops to determine producer-consumer dependences;
///      b) tensor case: use SSA use-def chains on extract_slice ops;
///   2. greedily fuse the linalg ops that produce the subview/extract_slice.
///   3. inspect the fused ops and determine whether they have other remaining
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 43-52
```cpp
///      LinalgOp uses. If not, then erase the original producing linalg op.
///
/// More advanced use cases, analyses as well as profitability heuristics are
/// left for future work.

struct ShapeDimension {
  Value shape;
  unsigned dimension;
};

```
- **EN**: Introduces declarations for `ShapeDimension`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ShapeDimension` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 53-66
```cpp
// Given an `op`, returns the first (`shape`, `dimension`) pair that identifies
// the loop range at `loopDepth`. The semantics of the loopToOperandRangesMaps
// guarantees at least one such dimension is found. If multiple candidates exist
// they must agree by construction (i.e. have the same size) and we just return
// the first one.
static ShapeDimension
getShapeDefiningLoopRange(LinalgOp op, unsigned loopDepth,
                          bool fromSubViewOpOnly = false) {
  // Iterate over the inputs and outputs in order.
  // Extract the subranges from the linearized ranges.
  for (OpOperand &opOperand : op->getOpOperands()) {
    // The method `getRangeFromOperandShape` requires using SubViewOp or
    // ExtractSliceOps. If the value isn't defined from there continue.
    // todo: The method should be adapted to get the values from
```
- **EN**: Implements logic around `getShapeDefiningLoopRange`, `getOpOperands`.
- **CN**: 围绕 `getShapeDefiningLoopRange`, `getOpOperands` 实现具体逻辑。

### Lines 67-74
```cpp
    // `ViewInterface`. The interface needs a `getOrCreateRanges` method which
    // currently returns a `linalg.range`. The fix here is to move this op to
    // `std` dialect and add the method to `ViewInterface`.
    if (fromSubViewOpOnly &&
        !isa_and_nonnull<memref::SubViewOp, tensor::ExtractSliceOp>(
            opOperand.get().getDefiningOp()))
      continue;

```
- **EN**: Implements logic around `ExtractSliceOp>`, `get`; this block expresses reusable interface-based behavior; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `ExtractSliceOp>`, `get` 实现具体逻辑；该代码块表达基于接口的可复用行为，并协调核心结构化 MLIR 方言之间的行为。

### Lines 75-88
```cpp
    AffineMap map = op.getMatchingIndexingMap(&opOperand);
    LLVM_DEBUG(llvm::dbgs() << "getShapeDefiningLoopRange I/O idx: "
                            << opOperand.getOperandNumber() << "\n");
    LLVM_DEBUG(llvm::dbgs()
               << "getShapeDefiningLoopRange map: " << map << "\n");
    for (const auto &en : llvm::enumerate(map.getResults())) {
      auto dimExpr = dyn_cast<AffineDimExpr>(en.value());
      if (!dimExpr)
        continue;
      if (loopDepth == cast<AffineDimExpr>(en.value()).getPosition()) {
        LLVM_DEBUG(llvm::dbgs() << "getShapeDefiningLoopRange loopDepth: "
                                << loopDepth << "\n");
        LLVM_DEBUG(llvm::dbgs() << "getShapeDefiningLoopRange shape: "
                                << opOperand.get() << "\n");
```
- **EN**: Implements logic around `getMatchingIndexingMap`, `dbgs`, `getOperandNumber`, `enumerate`, and 2 more symbols.
- **CN**: 围绕 `getMatchingIndexingMap`, `dbgs`, `getOperandNumber`, `enumerate`, and 2 more symbols 实现具体逻辑。

### Lines 89-96
```cpp
        return ShapeDimension{opOperand.get(),
                              static_cast<unsigned>(en.index())};
      }
    }
  }
  llvm_unreachable("Expect to be able to extract a shape defining loop range");
}

```
- **EN**: Implements logic around `get`, `static_cast`.
- **CN**: 围绕 `get`, `static_cast` 实现具体逻辑。

### Lines 97-103
```cpp
static SmallVector<Value> getTiledOperands(LinalgOp producer) {
  return producer->getOperands();
}

/// Fuses the producer by cloning the `producer`. The `fusedLoopsAndRanges`
/// provides the loop range information for the fused loops. The rest are
/// obtained from the producer itself, since they are not tiled + fused.
```
- **EN**: Implements logic around `getTiledOperands`, `getOperands`.
- **CN**: 围绕 `getTiledOperands`, `getOperands` 实现具体逻辑。

### Lines 104-117
```cpp
static LinalgOp fuse(OpBuilder &b, LinalgOp producer,
                     const DenseMap<unsigned, Range> &fusedLoopsAndRanges) {
  SmallVector<OpFoldResult> ivs, tileSizes, sizeBounds;
  SmallVector<Range> loopRanges;
  Location loc = producer.getLoc();

  for (unsigned i = 0, e = producer.getNumLoops(); i < e; ++i) {
    auto shapeDim = getShapeDefiningLoopRange(producer, i);
    OpFoldResult dim =
        createFoldedDimOp(b, loc, shapeDim.shape, shapeDim.dimension);
    sizeBounds.push_back(dim);
    auto it = fusedLoopsAndRanges.find(i);
    if (it != fusedLoopsAndRanges.end()) {
      ivs.push_back(it->second.offset);
```
- **EN**: Implements logic around `fuse`, `getLoc`, `getNumLoops`, `getShapeDefiningLoopRange`, and 4 more symbols.
- **CN**: 围绕 `fuse`, `getLoc`, `getNumLoops`, `getShapeDefiningLoopRange`, and 4 more symbols 实现具体逻辑。

### Lines 118-129
```cpp
      tileSizes.push_back(it->second.size);
      loopRanges.push_back(it->second);
      LLVM_DEBUG(llvm::dbgs() << "tiled loop#" << i << " with LoopRange "
                              << loopRanges.back() << "\n");
    } else {
      tileSizes.push_back(b.getIndexAttr(0));
      loopRanges.push_back(Range{b.getIndexAttr(0), dim, b.getIndexAttr(1)});
      LLVM_DEBUG(llvm::dbgs() << "full loop#" << i << " with LoopRange "
                              << loopRanges.back() << "\n");
    }
  }

```
- **EN**: Implements logic around `push_back`, `dbgs`, `back`.
- **CN**: 围绕 `push_back`, `dbgs`, `back` 实现具体逻辑。

### Lines 130-137
```cpp
  SmallVector<Value, 8> clonedShapes;
  clonedShapes.reserve(producer->getNumOperands());

  // Compute subranges for all tensor input/output operands.
  clonedShapes.append(makeTiledShapes(
      b, loc, producer, getTiledOperands(producer), ivs, tileSizes, sizeBounds,
      /**omitPartialTileCheck=*/false));

```
- **EN**: Implements logic around `reserve`, `append`, `getTiledOperands`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `reserve`, `append`, `getTiledOperands` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 138-147
```cpp
  // Take result types from the tiled init operands.
  MutableOperandRange producerDpsInits = producer.getDpsInitsMutable();
  SmallVector<Type, 4> resultTypes;
  resultTypes.reserve(producer->getNumResults());
  int64_t firstInitOperandIdx =
      producerDpsInits.getAsOperandRange().getBeginOperandIndex();
  for (int64_t i = 0, e = producer->getNumResults(); i < e; ++i) {
    resultTypes.push_back(clonedShapes[firstInitOperandIdx + i].getType());
  }

```
- **EN**: Implements logic around `getDpsInitsMutable`, `reserve`, `getAsOperandRange`, `getNumResults`, and 1 more symbols.
- **CN**: 围绕 `getDpsInitsMutable`, `reserve`, `getAsOperandRange`, `getNumResults`, and 1 more symbols 实现具体逻辑。

### Lines 148-155
```cpp
  // Clone the producer with new operands and result types.
  LinalgOp clonedOp = clone(b, producer, resultTypes, clonedShapes);

  // Shift all IndexOp results by the tile offset.
  SmallVector<OpFoldResult> allIvs = llvm::map_to_vector(
      loopRanges, [&](Range range) { return range.offset; });
  offsetIndices(b, clonedOp, allIvs);

```
- **EN**: Implements logic around `clone`, `map_to_vector`, `offsetIndices`.
- **CN**: 围绕 `clone`, `map_to_vector`, `offsetIndices` 实现具体逻辑。

### Lines 156-169
```cpp
  return clonedOp;
}

/// Get the loop range for a dimension `dim` based on the `shapedOperand`. It is
/// expected to be defined by a subview op or an extract_slice op.
static Range getRangeFromOperandShape(OpBuilder &b, Location loc,
                                      Value shapedOperand, unsigned dim) {
  Operation *shapeProducingOp = shapedOperand.getDefiningOp();
  if (auto subViewOp = dyn_cast<memref::SubViewOp>(shapeProducingOp))
    return subViewOp.getOrCreateRanges(b, loc)[dim];
  if (auto sliceOp = dyn_cast<tensor::ExtractSliceOp>(shapeProducingOp))
    return sliceOp.getOrCreateRanges(b, loc)[dim];
  llvm_unreachable("SubviewOp or ExtractSliceOp expected");
}
```
- **EN**: Implements logic around `getRangeFromOperandShape`, `getDefiningOp`, `SubViewOp>`, `getOrCreateRanges`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getRangeFromOperandShape`, `getDefiningOp`, `SubViewOp>`, `getOrCreateRanges`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 170-183
```cpp

/// Fuses the producer into the loop immediately enclosing the consumer.
/// This is achieved by "recomputing" the producer at the time it
/// is needed just before the consumer.
static LinalgOp fuse(OpBuilder &b, LinalgOp producerOp, AffineMap producerMap,
                     OpOperand &consumerOpOperand) {
  LLVM_DEBUG(llvm::dbgs() << "Producer map: " << producerMap << "\n");
  DenseMap<unsigned, Range> fusedLoopsAndRanges;
  Value shapedOperand = consumerOpOperand.get();
  for (const auto &en : llvm::enumerate(producerMap.getResults())) {
    unsigned posInProducerLoop = cast<AffineDimExpr>(en.value()).getPosition();
    fusedLoopsAndRanges[posInProducerLoop] = getRangeFromOperandShape(
        b, consumerOpOperand.getOwner()->getLoc(), shapedOperand, en.index());
  }
```
- **EN**: Implements logic around `fuse`, `dbgs`, `get`, `enumerate`, and 3 more symbols.
- **CN**: 围绕 `fuse`, `dbgs`, `get`, `enumerate`, and 3 more symbols 实现具体逻辑。

### Lines 184-196
```cpp
  return fuse(b, producerOp, fusedLoopsAndRanges);
}

/// Walk back use-def chain through scf::For yields.
/// Sets `producer` and `outputIndex` if it finds a producer LinalgOp

// TODO(ravishankarm, ntv): This can be moved into the dependence graphs
// dependence tracking since the dependence tracking is similar to what is done
// w.r.t to buffers.
static void getProducerOfTensor(Value tensor, OpResult &opResult) {
  if (!isa<RankedTensorType>(tensor.getType()))
    return;

```
- **EN**: Implements logic around `fuse`, `getProducerOfTensor`, `getType`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `fuse`, `getProducerOfTensor`, `getType` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 197-210
```cpp
  while (true) {
    LLVM_DEBUG(llvm::dbgs() << "\ngetProducerOfTensor: " << tensor);
    if (auto linalgOp = tensor.getDefiningOp<LinalgOp>()) {
      opResult = cast<OpResult>(tensor);
      return;
    }
    if (auto sliceOp = tensor.getDefiningOp<tensor::ExtractSliceOp>()) {
      tensor = sliceOp.getSource();
      continue;
    }
    if (auto blockArg = dyn_cast<BlockArgument>(tensor)) {
      if (auto forOp = blockArg.getDefiningOp<scf::ForOp>()) {
        tensor = forOp.getInitArgs()[blockArg.getArgNumber()];
        continue;
```
- **EN**: Implements logic around `dbgs`, `getDefiningOp`, `ExtractSliceOp>`, `getSource`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `dbgs`, `getDefiningOp`, `ExtractSliceOp>`, `getSource`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 211-224
```cpp
      }
    }
    return;
  }
}

FailureOr<FusionInfo>
mlir::linalg::fuseProducerOfTensor(OpBuilder &b, OpOperand &consumerOpOperand) {
  Value inputTensor = consumerOpOperand.get();
  OpResult producerOpResult;
  getProducerOfTensor(inputTensor, producerOpResult);
  if (!producerOpResult) {
    LLVM_DEBUG(llvm::dbgs() << "\nUnable to find producer");
    return failure();
```
- **EN**: Implements logic around `fuseProducerOfTensor`, `get`, `getProducerOfTensor`, `dbgs`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `fuseProducerOfTensor`, `get`, `getProducerOfTensor`, `dbgs`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 225-235
```cpp
  }
  return fuseProducerOfTensor(b, producerOpResult, consumerOpOperand);
}

FailureOr<FusionInfo>
mlir::linalg::fuseProducerOfTensor(OpBuilder &b, OpResult producerOpResult,
                                   OpOperand &consumerOpOperand) {
  auto producerOp = dyn_cast<LinalgOp>(producerOpResult.getOwner());
  if (!producerOp)
    return failure();

```
- **EN**: Implements logic around `fuseProducerOfTensor`, `getOwner`, `failure`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `fuseProducerOfTensor`, `getOwner`, `failure` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 236-249
```cpp
  LinalgOp consumerOp = dyn_cast<LinalgOp>(consumerOpOperand.getOwner());
  if (!consumerOp)
    return failure();

  Value inputTensor = consumerOpOperand.get();

  // Must be an extract_slice op to guarantee there are loops we can fuse into.
  auto sliceOp = inputTensor.getDefiningOp<tensor::ExtractSliceOp>();
  if (!sliceOp) {
    LLVM_DEBUG(llvm::dbgs()
               << "\nNot fusable, not an extract_slice op: " << inputTensor);
    return failure();
  }

```
- **EN**: Implements logic around `getOwner`, `failure`, `get`, `ExtractSliceOp>`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getOwner`, `failure`, `get`, `ExtractSliceOp>`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 250-263
```cpp
  // If producer is already in the same block as consumer, we are done.
  if (consumerOpOperand.get().getParentBlock() ==
      producerOpResult.getParentBlock())
    return failure();

  // Insert fused `producer` just before `consumer`.
  OpBuilder::InsertionGuard g(b);
  b.setInsertionPoint(consumerOp);
  LLVM_DEBUG(llvm::dbgs() << "Fuse into consumer: " << *consumerOp << "\n");
  OpOperand *opOperand =
      producerOp.getDpsInitOperand(producerOpResult.getResultNumber());
  LinalgOp fusedProducer =
      fuse(b, producerOp, producerOp.getMatchingIndexingMap(opOperand),
           consumerOpOperand);
```
- **EN**: Implements logic around `get`, `getParentBlock`, `failure`, `g`, and 4 more symbols.
- **CN**: 围绕 `get`, `getParentBlock`, `failure`, `g`, and 4 more symbols 实现具体逻辑。

### Lines 264-273
```cpp

  // Replace use.
  Value def = fusedProducer->getResult(producerOpResult.getResultNumber());
  Type consumerType = consumerOpOperand.get().getType();
  // Check if rank-reduction occurred as part of the extract_slice. If yes,
  // collapse the dropped dimensions.
  if (cast<ShapedType>(consumerType).getRank() !=
      cast<ShapedType>(def.getType()).getRank()) {
    llvm::SmallBitVector droppedDims = sliceOp.getDroppedDims();
    def =
```
- **EN**: Implements logic around `getResult`, `get`, `getRank`, `getType`, and 1 more symbols.
- **CN**: 围绕 `getResult`, `get`, `getRank`, `getType`, and 1 more symbols 实现具体逻辑。

### Lines 274-281
```cpp
        tensor::dropGivenUnitDims(b, fusedProducer.getLoc(), def, droppedDims);
  }
  // Canonicalizations are not guaranteed to have happened before constructing
  // `fusedProducer`. In the tensor case this can result in temporary type
  // mismatches. Insert a `tensor.cast` op to propagate the transformation
  // invariant that types are compatible.
  if (consumerType != def.getType())
    def = tensor::CastOp::create(b, fusedProducer.getLoc(), consumerType, def);
```
- **EN**: Implements logic around `dropGivenUnitDims`, `getType`, `create`; this block checks operation invariants or verification rules; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `dropGivenUnitDims`, `getType`, `create` 实现具体逻辑；该代码块检查操作不变式或验证规则，并协调核心结构化 MLIR 方言之间的行为。

### Lines 282-284
```cpp
  consumerOpOperand.set(def);
  return FusionInfo{cast<LinalgOp>(producerOpResult.getOwner()), fusedProducer};
}
```
- **EN**: Implements logic around `set`, `getOwner`.
- **CN**: 围绕 `set`, `getOwner` 实现具体逻辑。

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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/Dialect/Linalg/Utils/Utils.h`, `mlir/Dialect/MemRef/IR/MemRef.h`, `mlir/Dialect/Tensor/IR/Tensor.h`, `mlir/Dialect/Tensor/Utils/Utils.h`, `mlir/IR/AffineExpr.h`, `mlir/IR/AffineMap.h`, `mlir/IR/Dominance.h`, `mlir/Support/LLVM.h` ... (+3 more)
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (6), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (3), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (2), support utilities and LLVM interop helpers / 支持工具与 LLVM 互操作辅助逻辑 (1), LLVM support-library facilities / LLVM Support 库设施 (1)
