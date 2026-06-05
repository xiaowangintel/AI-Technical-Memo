# ElementwiseOpFusion.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Linalg/Transforms/ElementwiseOpFusion.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements the linalg dialect Fusion on tensors operations pass.
  - **CN**: 该文件位于 `mlir/lib/Dialect/Linalg/Transforms`，围绕 Linalg 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-32
```cpp
//===- ElementwiseOpFusion.cpp - Implementation of linalg Fusion ---------===///
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the linalg dialect Fusion on tensors operations pass.
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Linalg/Passes.h"

#include "mlir/Dialect/Affine/IR/AffineOps.h"
#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/Arith/Utils/Utils.h"
#include "mlir/Dialect/Linalg/IR/Linalg.h"
#include "mlir/Dialect/Linalg/Transforms/Transforms.h"
#include "mlir/Dialect/SparseTensor/IR/SparseTensor.h"
#include "mlir/Dialect/Tensor/Transforms/Transforms.h"
#include "mlir/IR/AffineExpr.h"
#include "mlir/IR/AffineMap.h"
#include "mlir/IR/Matchers.h"
#include "mlir/IR/PatternMatch.h"
#include "mlir/Support/LLVM.h"
#include "mlir/Transforms/GreedyPatternRewriteDriver.h"
#include "mlir/Transforms/RegionUtils.h"
#include "llvm/ADT/SmallVectorExtras.h"
#include <optional>
#include <utility>

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/Passes.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Arith/Utils/Utils.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/Passes.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Arith/Utils/Utils.h`。

### Lines 33-63
```cpp
namespace mlir {
#define GEN_PASS_DEF_LINALGELEMENTWISEOPFUSIONPASS
#include "mlir/Dialect/Linalg/Passes.h.inc"
} // namespace mlir

using namespace mlir;
using namespace mlir::linalg;

//===---------------------------------------------------------------------===//
// Methods and patterns that fuse elementwise `linalg.generic` operations.
//===---------------------------------------------------------------------===//

/// Append to `fusedOpIndexingMapAttrs` the indexing maps for the operands of
/// the `producer` to use in the fused operation given the indexing map of the
/// result of the producer in the consumer.
static AffineMap getIndexingMapOfProducerOperandsInCoordinatesOfFusedOp(
    OpOperand *producerOpOperand, AffineMap producerResultIndexMap,
    AffineMap fusedConsumerArgIndexMap) {
  // The indexing map in the consumer op (fusedConsumerArgIndexMap) is a map
  // from consumer loop -> consumer arg tensor index/producer result tensor
  // index. The fused loop is same as the consumer loop. For each producer arg
  // the indexing map to be computed is a map from consumer loop -> producer
  // arg tensor index.
  // producerResultIndexMap is a map from producer loop -> tensor index.
  // Compute the inverse to get map from tensor index -> producer loop.
  // The inverse is a map from producer result tensor index -> producer loop.
  AffineMap invProducerResultIndexMap =
      inversePermutation(producerResultIndexMap);
  assert(invProducerResultIndexMap &&
         "expected producer result indexing map to be invertible");

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/Passes.h.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/Passes.h.inc`。

### Lines 64-84
```cpp
  LinalgOp producer = cast<LinalgOp>(producerOpOperand->getOwner());
  // argMap is a map from producer loop -> producer arg tensor index.
  AffineMap argMap = producer.getMatchingIndexingMap(producerOpOperand);

  // Compose argMap with invProducerResultIndexMap to get a map from
  // producer result tensor index -> producer arg tensor index.
  AffineMap t1 = argMap.compose(invProducerResultIndexMap);

  // Compose t1 with fusedConsumerArgIndexMap gives an indexing map from
  // consumer loop/ fused loop -> producer arg tensor index.
  return t1.compose(fusedConsumerArgIndexMap);
}

// Checks if the given operand can be dropped, and the remaining operands
// of the fused producer & consumer after the fusion can still compute the
// bounds of the op.
static bool isOpOperandCanBeDroppedAfterFusedLinalgs(
    GenericOp producer, GenericOp consumer,
    ArrayRef<OpOperand *> opOperandsToIgnore) {
  SmallVector<AffineMap> indexingMaps;

```
- **EN**: Implements logic around `getOwner`, `getMatchingIndexingMap`, `compose`, `isOpOperandCanBeDroppedAfterFusedLinalgs`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getOwner`, `getMatchingIndexingMap`, `compose`, `isOpOperandCanBeDroppedAfterFusedLinalgs` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 85-107
```cpp
  SmallVector<GenericOp> ops = {producer, consumer};
  for (auto &op : ops) {
    for (auto &opOperand : op->getOpOperands()) {
      if (llvm::is_contained(opOperandsToIgnore, &opOperand)) {
        continue;
      }
      indexingMaps.push_back(op.getMatchingIndexingMap(&opOperand));
    }
  }
  if (indexingMaps.empty()) {
    // If there are no indexing maps, the operand can only be dropped
    // if neither op has loops.
    return producer.getNumLoops() == 0 && consumer.getNumLoops() == 0;
  }

  // The concatanation of the remained indexing maps must be invertible, so
  // the bounds of the op can be still computed after dropping the selected
  // operand. inversePermutation returns an empty AffineMap in case the
  // concatanated indexing maps are not invertible.
  return inversePermutation(concatAffineMaps(
             indexingMaps, producer.getContext())) != AffineMap();
}

```
- **EN**: Implements logic around `getOpOperands`, `is_contained`, `push_back`, `empty`, and 3 more symbols.
- **CN**: 围绕 `getOpOperands`, `is_contained`, `push_back`, `empty`, and 3 more symbols 实现具体逻辑。

### Lines 108-131
```cpp
/// Returns a set of indices of the producer's results which would
/// be preserved after the fusion.
/// * There is a chance that the implementation of the transformation does not
/// agree with the result of this method. This function gives a prediction based
/// on an optimized fusion.
llvm::SmallDenseSet<int> mlir::linalg::getPreservedProducerResults(
    GenericOp producer, GenericOp consumer, OpOperand *fusedOperand) {
  llvm::SmallDenseSet<int> preservedProducerResults;
  llvm::SmallVector<OpOperand *> opOperandsToIgnore;

  // The fusedOperand will be removed during the fusion
  opOperandsToIgnore.emplace_back(fusedOperand);

  for (const auto &producerResult : llvm::enumerate(producer->getResults())) {
    auto *outputOperand = producer.getDpsInitOperand(producerResult.index());
    opOperandsToIgnore.emplace_back(outputOperand);
    if (producer.payloadUsesValueFromOperand(outputOperand) ||
        !isOpOperandCanBeDroppedAfterFusedLinalgs(producer, consumer,
                                                  opOperandsToIgnore) ||
        llvm::any_of(producerResult.value().getUsers(), [&](Operation *user) {
          return user != consumer.getOperation();
        })) {
      preservedProducerResults.insert(producerResult.index());

```
- **EN**: Implements logic around `getPreservedProducerResults`, `emplace_back`, `enumerate`, `getDpsInitOperand`, and 5 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getPreservedProducerResults`, `emplace_back`, `enumerate`, `getDpsInitOperand`, and 5 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 132-150
```cpp
      // In case the operand can't be dropped
      (void)opOperandsToIgnore.pop_back_val();
    }
  }
  return preservedProducerResults;
}

/// Conditions for elementwise fusion of generic operations.
bool mlir::linalg::areElementwiseOpsFusable(OpOperand *fusedOperand) {
  if (!fusedOperand)
    return false;

  auto producer = fusedOperand->get().getDefiningOp<GenericOp>();
  auto consumer = dyn_cast<GenericOp>(fusedOperand->getOwner());

  // Check producer and consumer are generic ops.
  if (!producer || !consumer)
    return false;

```
- **EN**: Implements logic around `pop_back_val`, `areElementwiseOpsFusable`, `get`, `getOwner`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `pop_back_val`, `areElementwiseOpsFusable`, `get`, `getOwner` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 151-173
```cpp
  // Consumer can have mixed semantics, just check operand itself has tensor
  // type. Producer must have full tensor semantics to avoid potential
  // aliasing between producer and consumer memrefs.
  if (!producer.hasPureTensorSemantics() ||
      !isa<RankedTensorType>(fusedOperand->get().getType()))
    return false;

  // Verify that
  // - the producer has all "parallel" iterator type.
  if (producer.getNumParallelLoops() != producer.getNumLoops())
    return false;

  // Only allow fusing the producer of an input operand for now.
  // TODO: allow fusing the producer of an output operand.
  if (!consumer.isDpsInput(fusedOperand))
    return false;

  // Get the consumer index map. The number of results of the consumer index
  // map must match the number of loops of the producer.
  AffineMap consumerIndexMap = consumer.getMatchingIndexingMap(fusedOperand);
  if (consumerIndexMap.getNumResults() != producer.getNumLoops())
    return false;

```
- **EN**: Implements logic around `hasPureTensorSemantics`, `get`, `getNumParallelLoops`, `isDpsInput`, and 2 more symbols; this block checks operation invariants or verification rules; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `hasPureTensorSemantics`, `get`, `getNumParallelLoops`, `isDpsInput`, and 2 more symbols 实现具体逻辑；该代码块检查操作不变式或验证规则，并协调核心结构化 MLIR 方言之间的行为。

### Lines 174-194
```cpp
  // Finally the index_map for the result must be invertible. For now just
  // verify it is a permutation.
  auto producerResult = cast<OpResult>(fusedOperand->get());
  AffineMap producerResultIndexMap =
      producer.getIndexingMapMatchingResult(producerResult);
  if (!producerResultIndexMap.isPermutation())
    return false;

  // Ensure that the fusion does not remove size information required to
  // get the loop bounds. For non-reduction generics, this is trivially the
  // case due to the output operand. For reductions, we need to check that after
  // the fusion, each loop dimension has at least one input that defines it.
  if ((consumer.getNumReductionLoops())) {
    BitVector coveredDims(consumer.getNumLoops(), false);

    auto addToCoveredDims = [&](AffineMap map) {
      for (auto result : map.getResults())
        if (auto dimExpr = dyn_cast<AffineDimExpr>(result))
          coveredDims[dimExpr.getPosition()] = true;
    };

```
- **EN**: Implements logic around `get`, `getIndexingMapMatchingResult`, `isPermutation`, `getNumReductionLoops`, and 3 more symbols; this block checks operation invariants or verification rules.
- **CN**: 围绕 `get`, `getIndexingMapMatchingResult`, `isPermutation`, `getNumReductionLoops`, and 3 more symbols 实现具体逻辑；该代码块检查操作不变式或验证规则。

### Lines 195-213
```cpp
    for (auto pair :
         llvm::zip(consumer->getOperands(), consumer.getIndexingMapsArray())) {
      Value operand = std::get<0>(pair);
      if (operand == fusedOperand->get())
        continue;
      AffineMap operandMap = std::get<1>(pair);
      addToCoveredDims(operandMap);
    }

    for (OpOperand *operand : producer.getDpsInputOperands()) {
      AffineMap newIndexingMap =
          getIndexingMapOfProducerOperandsInCoordinatesOfFusedOp(
              operand, producerResultIndexMap, consumerIndexMap);
      addToCoveredDims(newIndexingMap);
    }
    if (!coveredDims.all())
      return false;
  }

```
- **EN**: Implements logic around `zip`, `get`, `addToCoveredDims`, `getDpsInputOperands`, and 2 more symbols.
- **CN**: 围绕 `zip`, `get`, `addToCoveredDims`, `getDpsInputOperands`, and 2 more symbols 实现具体逻辑。

### Lines 214-231
```cpp
  return true;
}

/// Generate the region of the fused tensor operation. The region of the fused
/// op must be empty.
static void generateFusedElementwiseOpRegion(
    RewriterBase &rewriter, GenericOp fusedOp,
    AffineMap consumerToProducerLoopsMap, OpOperand *fusedOperand,
    unsigned nloops, llvm::SmallDenseSet<int> &preservedProducerResults) {
  auto producer = cast<GenericOp>(fusedOperand->get().getDefiningOp());
  auto consumer = cast<GenericOp>(fusedOperand->getOwner());
  // Build the region of the fused op.
  Block &producerBlock = producer->getRegion(0).front();
  Block &consumerBlock = consumer->getRegion(0).front();
  OpBuilder::InsertionGuard guard(rewriter);
  Block *fusedBlock = rewriter.createBlock(&fusedOp.getRegion());
  IRMapping mapper;

```
- **EN**: Implements logic around `generateFusedElementwiseOpRegion`, `get`, `getOwner`, `getRegion`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `generateFusedElementwiseOpRegion`, `get`, `getOwner`, `getRegion`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 232-258
```cpp
  // 2. Add an index operation for every fused loop dimension and use the
  // `consumerToProducerLoopsMap` to map the producer indices.
  if (producer.hasIndexSemantics()) {
    // Add an index operation for every fused loop dimension.
    unsigned numFusedOpLoops = fusedOp.getNumLoops();
    SmallVector<Value> fusedIndices;
    fusedIndices.reserve(numFusedOpLoops);
    llvm::transform(llvm::seq<uint64_t>(0, numFusedOpLoops),
                    std::back_inserter(fusedIndices), [&](uint64_t dim) {
                      return IndexOp::create(rewriter, producer.getLoc(), dim);
                    });
    for (IndexOp indexOp :
         llvm::make_early_inc_range(producerBlock.getOps<IndexOp>())) {
      Value newIndex = affine::AffineApplyOp::create(
          rewriter, producer.getLoc(),
          consumerToProducerLoopsMap.getSubMap(indexOp.getDim()), fusedIndices);
      mapper.map(indexOp.getResult(), newIndex);
    }
  }
  // TODO: allow fusing the producer of an output operand.
  assert(consumer.isDpsInput(fusedOperand) &&
         "expected producer of input operand");
  // 3. Consumer input operands up to consumerIdx (exclusive).
  for (BlockArgument bbArg : consumerBlock.getArguments().take_front(
           fusedOperand->getOperandNumber())) // input assumption.
    mapper.map(bbArg, fusedBlock->addArgument(bbArg.getType(), bbArg.getLoc()));

```
- **EN**: Implements logic around `hasIndexSemantics`, `getNumLoops`, `reserve`, `transform`, and 9 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `hasIndexSemantics`, `getNumLoops`, `reserve`, `transform`, and 9 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 259-282
```cpp
  // Replacing consumerIdx requires getting the cloned, yielded, value from
  // the (cloned) producer block. This happens in step 9.

  // 4. Splice in producer's input operands.
  for (BlockArgument bbArg :
       producerBlock.getArguments().take_front(producer.getNumDpsInputs()))
    mapper.map(bbArg, fusedBlock->addArgument(bbArg.getType(), bbArg.getLoc()));

  // 5. Remaining consumer's input operands (drop past index `consumerIdx`).
  for (BlockArgument bbArg :
       consumerBlock.getArguments()
           .take_front(consumer.getNumDpsInputs())
           .drop_front(fusedOperand->getOperandNumber() + 1))
    mapper.map(bbArg, fusedBlock->addArgument(bbArg.getType(), bbArg.getLoc()));

  // 6. All of the producer's output operands
  for (const auto &bbArg : llvm::enumerate(
           producerBlock.getArguments().take_back(producer.getNumDpsInits()))) {
    if (!preservedProducerResults.count(bbArg.index()))
      continue;
    mapper.map(bbArg.value(), fusedBlock->addArgument(bbArg.value().getType(),
                                                      bbArg.value().getLoc()));
  }

```
- **EN**: Implements logic around `getArguments`, `map`, `take_front`, `drop_front`, and 3 more symbols.
- **CN**: 围绕 `getArguments`, `map`, `take_front`, `drop_front`, and 3 more symbols 实现具体逻辑。

### Lines 283-301
```cpp
  // 7. All of consumer's output operands.
  for (BlockArgument bbArg :
       consumerBlock.getArguments().take_back(consumer.getNumDpsInits()))
    mapper.map(bbArg, fusedBlock->addArgument(bbArg.getType(), bbArg.getLoc()));

  // 8. Clone all producer operations except for the yield and index operations
  // to the fused operation.
  for (auto &op : producerBlock.without_terminator()) {
    if (!isa<IndexOp>(op))
      rewriter.clone(op, mapper);
  }
  // 9. Now we can map the consumerBlock's `consumerIdx` block argument. Just
  // forward the yield operand.
  auto producerYieldOp = cast<linalg::YieldOp>(producerBlock.getTerminator());
  unsigned producerResultNumber =
      cast<OpResult>(fusedOperand->get()).getResultNumber();
  Value replacement =
      mapper.lookupOrDefault(producerYieldOp.getOperand(producerResultNumber));

```
- **EN**: Implements logic around `getArguments`, `map`, `without_terminator`, `clone`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getArguments`, `map`, `without_terminator`, `clone`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 302-333
```cpp
  // Sanity checks, if replacement is not already in the mapper then it must be
  // produced outside.
  if (replacement == producerYieldOp.getOperand(producerResultNumber)) {
    if (auto bb = dyn_cast<BlockArgument>(replacement))
      assert(bb.getOwner() != &producerBlock &&
             "yielded block argument must have been mapped");
    else
      assert(!producer->isAncestor(replacement.getDefiningOp()) &&
             "yielded value must have been mapped");
  }
  mapper.map(consumerBlock.getArgument(fusedOperand->getOperandNumber()),
             replacement);
  // 10. Clone operations from the consumer to the fused op.
  for (auto &op : consumerBlock.without_terminator())
    rewriter.clone(op, mapper);

  // 11. Include the final yield (which is the remapped values for all the
  // yield)
  auto consumerYieldOp = cast<linalg::YieldOp>(consumerBlock.getTerminator());
  SmallVector<Value> fusedYieldValues;
  fusedYieldValues.reserve(producerYieldOp.getNumOperands() +
                           consumerYieldOp.getNumOperands());
  for (const auto &producerYieldVal :
       llvm::enumerate(producerYieldOp.getOperands())) {
    if (preservedProducerResults.count(producerYieldVal.index()))
      fusedYieldValues.push_back(
          mapper.lookupOrDefault(producerYieldVal.value()));
  }
  for (auto consumerYieldVal : consumerYieldOp.getOperands())
    fusedYieldValues.push_back(mapper.lookupOrDefault(consumerYieldVal));
  YieldOp::create(rewriter, fusedOp.getLoc(), fusedYieldValues);

```
- **EN**: Implements logic around `getOperand`, `assert`, `map`, `without_terminator`, and 10 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getOperand`, `assert`, `map`, `without_terminator`, and 10 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 334-351
```cpp
  // Sanity checks.
  assert(fusedBlock->getNumArguments() == fusedOp.getNumOperands() &&
         "Ill-formed GenericOp region");
}

FailureOr<mlir::linalg::ElementwiseOpFusionResult>
mlir::linalg::fuseElementwiseOps(RewriterBase &rewriter,
                                 OpOperand *fusedOperand) {
  assert(areElementwiseOpsFusable(fusedOperand) &&
         "expected elementwise operation pre-conditions to pass");
  auto producerResult = cast<OpResult>(fusedOperand->get());
  auto producer = cast<GenericOp>(producerResult.getOwner());
  auto consumer = cast<GenericOp>(fusedOperand->getOwner());
  // TODO: allow fusing the producer of an output operand.
  assert(consumer.isDpsInput(fusedOperand) &&
         "expected producer of input operand");
  /// Find the results of the producer that have uses outside of the consumer,
  /// after the fusion.
```
- **EN**: Implements logic around `assert`, `fuseElementwiseOps`, `get`, `getOwner`; this block packages logic as an MLIR pass or pass helper; uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `assert`, `fuseElementwiseOps`, `get`, `getOwner` 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或 Pass 辅助组件，并使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 352-387
```cpp
  llvm::SmallDenseSet<int> preservedProducerResults =
      mlir::linalg::getPreservedProducerResults(producer, consumer,
                                                fusedOperand);

  // Compute the fused operands list and indexing maps.
  SmallVector<Value> fusedInputOperands, fusedOutputOperands;
  SmallVector<Type> fusedResultTypes;
  SmallVector<AffineMap> fusedIndexMaps;
  fusedInputOperands.reserve(producer.getNumDpsInputs() +
                             consumer.getNumDpsInputs());
  fusedOutputOperands.reserve(preservedProducerResults.size() +
                              consumer.getNumDpsInits());
  fusedResultTypes.reserve(preservedProducerResults.size() +
                           consumer.getNumDpsInits());
  fusedIndexMaps.reserve(producer->getNumOperands() +
                         consumer->getNumOperands());
  // In the following, numbering matches that of `generateFusedTensorOpRegion`.
  // 3. Consumer input operands/maps up to consumerIdx (exclusive).
  auto consumerInputs = consumer.getDpsInputOperands();
  auto *it = llvm::find_if(consumerInputs, [&](OpOperand *operand) {
    return operand == fusedOperand;
  });
  assert(it != consumerInputs.end() && "expected to find the consumer operand");
  for (OpOperand *opOperand : llvm::make_range(consumerInputs.begin(), it)) {
    fusedInputOperands.push_back(opOperand->get());
    fusedIndexMaps.push_back(consumer.getMatchingIndexingMap(opOperand));
  }
  // 4. Splice in producer's input operands/maps.
  AffineMap producerResultIndexMap =
      producer.getIndexingMapMatchingResult(producerResult);
  for (OpOperand *opOperand : producer.getDpsInputOperands()) {
    fusedInputOperands.push_back(opOperand->get());
    // Compute indexing maps for the producer args in the fused operation.
    AffineMap map = getIndexingMapOfProducerOperandsInCoordinatesOfFusedOp(
        opOperand, producerResultIndexMap,
        consumer.getMatchingIndexingMap(fusedOperand));
```
- **EN**: Implements logic around `getPreservedProducerResults`, `reserve`, `getNumDpsInputs`, `getNumDpsInits`, and 9 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getPreservedProducerResults`, `reserve`, `getNumDpsInputs`, `getNumDpsInits`, and 9 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 388-410
```cpp
    fusedIndexMaps.push_back(map);
  }
  // 5. Remaining consumer's input operands/maps (drop past index
  // `consumerIdx`).
  for (OpOperand *opOperand :
       llvm::make_range(std::next(it), consumerInputs.end())) {
    fusedInputOperands.push_back(opOperand->get());
    fusedIndexMaps.push_back(consumer.getMatchingIndexingMap(opOperand));
  }

  // 6. Collect all of the producer outputs.
  for (const auto &opOperand : llvm::enumerate(producer.getDpsInitsMutable())) {
    if (!preservedProducerResults.count(opOperand.index()))
      continue;

    fusedOutputOperands.push_back(opOperand.value().get());
    AffineMap map = getIndexingMapOfProducerOperandsInCoordinatesOfFusedOp(
        &opOperand.value(), producerResultIndexMap,
        consumer.getMatchingIndexingMap(fusedOperand));
    fusedIndexMaps.push_back(map);
    fusedResultTypes.push_back(opOperand.value().get().getType());
  }

```
- **EN**: Implements logic around `push_back`, `make_range`, `enumerate`, `count`, and 3 more symbols.
- **CN**: 围绕 `push_back`, `make_range`, `enumerate`, `count`, and 3 more symbols 实现具体逻辑。

### Lines 411-435
```cpp
  // 7. All of consumer's output operands (skip operands: added by the builder).
  for (OpOperand &opOperand : consumer.getDpsInitsMutable()) {
    fusedOutputOperands.push_back(opOperand.get());
    fusedIndexMaps.push_back(consumer.getMatchingIndexingMap(&opOperand));
    Type resultType = opOperand.get().getType();
    if (!isa<MemRefType>(resultType))
      fusedResultTypes.push_back(resultType);
  }

  // Generate the fused op.
  auto fusedOp = GenericOp::create(
      rewriter, consumer.getLoc(), fusedResultTypes, fusedInputOperands,
      fusedOutputOperands, rewriter.getAffineMapArrayAttr(fusedIndexMaps),
      consumer.getIteratorTypes(),
      /*doc=*/nullptr,
      /*library_call=*/nullptr);
  if (!fusedOp.getShapesToLoopsMap()) {
    // Fused op has invalid indexing maps. Typically this means something is off
    // in the input, but going ahead here would result in verification errors.
    // So cleanup and abort.
    rewriter.eraseOp(fusedOp);
    return rewriter.notifyMatchFailure(
        fusedOp, "fused op failed loop bound computation check");
  }

```
- **EN**: Implements logic around `getDpsInitsMutable`, `push_back`, `get`, `create`, and 6 more symbols; this block uses rewrite-pattern infrastructure to transform operations; checks operation invariants or verification rules.
- **CN**: 围绕 `getDpsInitsMutable`, `push_back`, `get`, `create`, and 6 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并检查操作不变式或验证规则。

### Lines 436-462
```cpp
  // Construct an AffineMap from consumer loops to producer loops.
  // consumer loop -> tensor index
  AffineMap consumerResultIndexMap =
      consumer.getMatchingIndexingMap(fusedOperand);
  // tensor index -> producer loop
  AffineMap invProducerResultIndexMap =
      inversePermutation(producerResultIndexMap);
  assert(invProducerResultIndexMap &&
         "expected producer result indexig map to be invertible");
  // consumer loop -> producer loop
  AffineMap consumerToProducerLoopsMap =
      invProducerResultIndexMap.compose(consumerResultIndexMap);

  generateFusedElementwiseOpRegion(
      rewriter, fusedOp, consumerToProducerLoopsMap, fusedOperand,
      consumer.getNumLoops(), preservedProducerResults);
  ElementwiseOpFusionResult result;
  result.fusedOp = fusedOp;
  int resultNum = 0;
  for (auto [index, producerResult] : llvm::enumerate(producer->getResults()))
    if (preservedProducerResults.count(index))
      result.replacements[producerResult] = fusedOp->getResult(resultNum++);
  for (auto consumerResult : consumer->getResults())
    result.replacements[consumerResult] = fusedOp->getResult(resultNum++);
  return result;
}

```
- **EN**: Implements logic around `getMatchingIndexingMap`, `inversePermutation`, `assert`, `compose`, and 6 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getMatchingIndexingMap`, `inversePermutation`, `assert`, `compose`, and 6 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 463-480
```cpp
namespace {
/// Patterns to fuse a generic op, with the producer of its operands.
class FuseElementwiseOps : public OpRewritePattern<GenericOp> {
public:
  FuseElementwiseOps(MLIRContext *context, ControlFusionFn fun,
                     PatternBenefit benefit = 1)
      : OpRewritePattern<GenericOp>(context, benefit),
        controlFn(std::move(fun)) {}

  LogicalResult matchAndRewrite(GenericOp genericOp,
                                PatternRewriter &rewriter) const override {
    // Find the first operand that is defined by another generic op on tensors.
    for (OpOperand &opOperand : genericOp->getOpOperands()) {
      if (!areElementwiseOpsFusable(&opOperand))
        continue;
      if (!controlFn(&opOperand))
        continue;

```
- **EN**: Introduces declarations for `FuseElementwiseOps`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `FuseElementwiseOps` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 481-501
```cpp
      Operation *producer = opOperand.get().getDefiningOp();

      // Find the producer of the operand.
      FailureOr<ElementwiseOpFusionResult> fusionResult =
          fuseElementwiseOps(rewriter, &opOperand);
      if (failed(fusionResult))
        return rewriter.notifyMatchFailure(genericOp, "fusion failed");

      // Perform the fusion.
      for (auto [origVal, replacement] : fusionResult->replacements) {
        rewriter.replaceUsesWithIf(origVal, replacement, [&](OpOperand &use) {
          // Only replace consumer uses.
          return use.get().getDefiningOp() != producer;
        });
      }
      rewriter.eraseOp(genericOp);
      return success();
    }
    return failure();
  }

```
- **EN**: Implements logic around `get`, `fuseElementwiseOps`, `failed`, `notifyMatchFailure`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `get`, `fuseElementwiseOps`, `failed`, `notifyMatchFailure`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 502-519
```cpp
private:
  ControlFusionFn controlFn;
};
} // namespace

//===---------------------------------------------------------------------===//
// Methods and patterns that fuse reshape ops with elementwise operations by
// expanding the dimensionality of the elementwise operations.
//===---------------------------------------------------------------------===//

/// Conditions for folding a structured linalg operation with a reshape op by
/// expanding the iteration space dimensionality for tensor operations. These
/// are preconditions assumed by `foldReshapeByDimExpansion` which implements
/// the following fusion pattern.
///
///  Consider
///
///  %c = linalg.generic ins(%a, %b : memref<?x?x?xf32>, memref<?x?xf32>)
```
- **EN**: Transitions between access-control regions inside a class or struct definition.
- **CN**: 在类或结构体定义内部切换访问控制区域。

### Lines 520-537
```cpp
///         indexing_maps = [affine_map<(d0, d1, d2) -> (d1, d0, d2)>,
///                          affine_map<(d0, d1, d2) -> (d1, d2)>,
///                          affine_map<(d0, d1, d2) -> (d0, d2, d1)>]
///  %d = tensor.expand_shape %c [[0, 1], [2], [3, 4, 5]]
///       : tensor<?x?x?xf32> into tensor<?x?x?x?x?x?xf32>
///
///  The reshape can be folded into the `linalgOp` if its loop dimensionality
///  is increased to match the result (operand) of the tensor.expand_shape.
///  The indexing_map of the fused tensor in the `linalgOp` and the
///  reassociation map helps compute the indexing maps of the modified op.
///  For the above example, based on the reassociation map it
///  can be concluded that
///
///  - The loop used to access the first dimension of the fused tensor is split
///    into two.
///  - The loop used to access the second dimension of the fused tensor is kept
///    as is.
///  - The loop used to access the third dimension of the fused tensor is split
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 538-555
```cpp
///    into three.
///
///  i.e. (e0, e1, e2, e3, e4) is the domain of the indexing map of the modified
///  op, then
///
///   d0 -> e0, e1
///   d1 -> e2, e3, e4
///   d2 -> e5
///
///  substituting this, the structured op can be rewritten as
///
///  %d = linalg.generic ins(%0, %1 : )
///        indexing_maps =
///         [affine_map<(e0, e1, e2, e3, e4, e5) -> (e2, e3, e4, e0, e1, e5)>,
///          affine_map<(e0, e1, e2, e3, e4, e5) -> (e2, e3, e4, e5)>,
///          affine_map<(e0, e1, e2, e3, e4, e5) -> (e0, e1, e5, e2, e3, e4)>]
///
///  Since operands to the linalg generic are now 5D, reshapes can be introduced
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 556-583
```cpp
///  to make it consistent
///
///  %0 = tensor.expand_shape %a [[0, 1, 2], [3, 4], [5]]
///       : tensor<?x?x?xf32> into tensor<?x?x?x?x?x?xf32>
///  %1 = tensor.expand_shape %b [[0, 1, 2], [3]]
///       : tensor<?x?x?xf32> into tensor<?x?x?x?xf32>
///
///  The added reshapes are again expanding patterns, so they will get fused
///  with its producers if possible.
static bool isFusableWithReshapeByDimExpansion(LinalgOp linalgOp,
                                               OpOperand *fusableOpOperand) {
  // Is fusable only if:
  // - All the indexing maps for operands and results are projected
  //   permutations.
  // - The fused tensor is not a scalar.
  SmallVector<utils::IteratorType> iteratorTypes =
      linalgOp.getIteratorTypesArray();
  AffineMap operandMap = linalgOp.getMatchingIndexingMap(fusableOpOperand);
  return linalgOp.hasPureTensorSemantics() &&
         llvm::all_of(linalgOp.getIndexingMaps().getValue(),
                      [](Attribute attr) {
                        return cast<AffineMapAttr>(attr)
                            .getValue()
                            .isProjectedPermutation();
                      }) &&
         operandMap.getNumResults() > 0;
}

```
- **EN**: Implements logic around `isFusableWithReshapeByDimExpansion`, `getIteratorTypesArray`, `getMatchingIndexingMap`, `hasPureTensorSemantics`, and 4 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `isFusableWithReshapeByDimExpansion`, `getIteratorTypesArray`, `getMatchingIndexingMap`, `hasPureTensorSemantics`, and 4 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 584-606
```cpp
namespace {
/// Information needed to expand a generic operation to fold the reshape with
/// it.
class ExpansionInfo {
public:
  // Computes the mapping from original dimensions of the op to the dimensions
  // of the expanded op given the `indexingMap` of the fused operand/result of
  // the generic op, the `reassocationMaps` of the reshape op and the shape of
  // the expanded op.
  LogicalResult compute(LinalgOp linalgOp, OpOperand *fusableOpOperand,
                        ArrayRef<AffineMap> reassociationMaps,
                        ArrayRef<OpFoldResult> expandedShape,
                        PatternRewriter &rewriter);
  unsigned getOrigOpNumDims() const { return reassociation.size(); }
  unsigned getExpandedOpNumDims() const { return expandedOpNumDims; }
  ReassociationIndicesRef getExpandedDims(unsigned i) const {
    return reassociation[i];
  }
  ArrayRef<OpFoldResult> getExpandedShapeOfDim(unsigned i) const {
    return expandedShapeMap[i];
  }
  ArrayRef<OpFoldResult> getOriginalShape() const { return originalLoopExtent; }

```
- **EN**: Introduces declarations for `ExpansionInfo`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ExpansionInfo` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 607-628
```cpp
private:
  /// Reassociation from the dimensions in the original operation to the
  /// dimension of the expanded operation.
  SmallVector<ReassociationIndices> reassociation;
  /// Mapping from extent of loops in the original operation, to the extent of
  /// loops in the expanded operation.
  SmallVector<SmallVector<OpFoldResult>> expandedShapeMap;
  /// Extent of the loop in the original operation.
  SmallVector<OpFoldResult> originalLoopExtent;
  unsigned expandedOpNumDims;
};
} // namespace

LogicalResult ExpansionInfo::compute(LinalgOp linalgOp,
                                     OpOperand *fusableOpOperand,
                                     ArrayRef<AffineMap> reassociationMaps,
                                     ArrayRef<OpFoldResult> expandedShape,
                                     PatternRewriter &rewriter) {
  if (reassociationMaps.empty())
    return failure();
  AffineMap fusedIndexMap = linalgOp.getMatchingIndexingMap(fusableOpOperand);

```
- **EN**: Implements logic around `compute`, `empty`, `failure`, `getMatchingIndexingMap`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `compute`, `empty`, `failure`, `getMatchingIndexingMap` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 629-653
```cpp
  OpBuilder::InsertionGuard g(rewriter);
  rewriter.setInsertionPoint(linalgOp);
  originalLoopExtent = llvm::map_to_vector(
      linalgOp.createLoopRanges(rewriter, linalgOp->getLoc()),
      [](Range r) { return r.size; });

  reassociation.clear();
  expandedShapeMap.clear();
  // Compute the number of dimension in the expanded op that correspond to each
  // dimension of the original op.
  SmallVector<unsigned> numExpandedDims(fusedIndexMap.getNumDims(), 1);
  expandedShapeMap.resize(fusedIndexMap.getNumDims());
  for (const auto &resultExpr : llvm::enumerate(fusedIndexMap.getResults())) {
    unsigned pos = cast<AffineDimExpr>(resultExpr.value()).getPosition();
    AffineMap foldedDims = reassociationMaps[resultExpr.index()];
    numExpandedDims[pos] = foldedDims.getNumResults();
    ArrayRef<OpFoldResult> shape =
        expandedShape.slice(foldedDims.getDimPosition(0), numExpandedDims[pos]);
    expandedShapeMap[pos].assign(shape.begin(), shape.end());
  }
  // The remaining dimensions remain the same.
  for (unsigned i : llvm::seq<unsigned>(0, fusedIndexMap.getNumDims()))
    if (expandedShapeMap[i].empty())
      expandedShapeMap[i] = {originalLoopExtent[i]};

```
- **EN**: Implements logic around `g`, `setInsertionPoint`, `map_to_vector`, `createLoopRanges`, and 11 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `g`, `setInsertionPoint`, `map_to_vector`, `createLoopRanges`, and 11 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 654-684
```cpp
  // Compute reassociation map from the original op to the expanded op.
  unsigned sum = 0;
  reassociation.reserve(fusedIndexMap.getNumDims());
  for (const auto &numFoldedDim : llvm::enumerate(numExpandedDims)) {
    auto seq = llvm::seq<int64_t>(sum, sum + numFoldedDim.value());
    reassociation.emplace_back(seq.begin(), seq.end());
    sum += numFoldedDim.value();
  }
  expandedOpNumDims = sum;
  return success();
}

/// Return the indexing map to use in the expanded op for a given the
/// `indexingMap` of the original operation.
static AffineMap
getIndexingMapInExpandedOp(OpBuilder &builder, AffineMap indexingMap,
                           const ExpansionInfo &expansionInfo) {
  SmallVector<AffineExpr> newExprs;
  for (AffineExpr expr : indexingMap.getResults()) {
    unsigned pos = cast<AffineDimExpr>(expr).getPosition();
    SmallVector<AffineExpr, 4> expandedExprs = llvm::map_to_vector<4>(
        expansionInfo.getExpandedDims(pos), [&](int64_t v) {
          return builder.getAffineDimExpr(static_cast<unsigned>(v));
        });
    newExprs.append(expandedExprs.begin(), expandedExprs.end());
  }
  return AffineMap::get(expansionInfo.getExpandedOpNumDims(),
                        indexingMap.getNumSymbols(), newExprs,
                        builder.getContext());
}

```
- **EN**: Implements logic around `reserve`, `enumerate`, `seq`, `emplace_back`, and 12 more symbols.
- **CN**: 围绕 `reserve`, `enumerate`, `seq`, `emplace_back`, and 12 more symbols 实现具体逻辑。

### Lines 685-703
```cpp
/// Return the shape and type of the operand/result to use in the expanded op
/// given the type in the original op.
static std::tuple<SmallVector<OpFoldResult>, RankedTensorType>
getExpandedShapeAndType(RankedTensorType originalType, AffineMap indexingMap,
                        const ExpansionInfo &expansionInfo) {
  SmallVector<OpFoldResult> expandedShape;
  for (AffineExpr expr : indexingMap.getResults()) {
    unsigned dim = cast<AffineDimExpr>(expr).getPosition();
    ArrayRef<OpFoldResult> dimExpansion =
        expansionInfo.getExpandedShapeOfDim(dim);
    expandedShape.append(dimExpansion.begin(), dimExpansion.end());
  }
  SmallVector<int64_t> expandedStaticShape;
  std::tie(expandedStaticShape, std::ignore) =
      decomposeMixedValues(expandedShape);
  return {expandedShape, RankedTensorType::get(expandedStaticShape,
                                               originalType.getElementType())};
}

```
- **EN**: Implements logic around `getExpandedShapeAndType`, `getResults`, `getPosition`, `getExpandedShapeOfDim`, and 5 more symbols.
- **CN**: 围绕 `getExpandedShapeAndType`, `getResults`, `getPosition`, `getExpandedShapeOfDim`, and 5 more symbols 实现具体逻辑。

### Lines 704-725
```cpp
/// Returns the reassociation maps to use in the `tensor.expand_shape`
/// operation to convert the operands of the original operation to operands of
/// the expanded operation. The same method is used to compute the
/// `tensor.collapse_shape` used to collapse the result of the expanded
/// op to get the value that can replace all uses of the results of the original
/// op.
static SmallVector<ReassociationIndices>
getReassociationForExpansion(AffineMap indexingMap,
                             const ExpansionInfo &expansionInfo) {
  SmallVector<ReassociationIndices> reassociation;
  unsigned numReshapeDims = 0;
  for (AffineExpr expr : indexingMap.getResults()) {
    unsigned dim = cast<AffineDimExpr>(expr).getPosition();
    auto numExpandedDims = expansionInfo.getExpandedDims(dim).size();
    SmallVector<int64_t, 2> indices = llvm::to_vector<2>(
        llvm::seq<int64_t>(numReshapeDims, numReshapeDims + numExpandedDims));
    reassociation.emplace_back(std::move(indices));
    numReshapeDims += numExpandedDims;
  }
  return reassociation;
}

```
- **EN**: Implements logic around `getReassociationForExpansion`, `getResults`, `getPosition`, `getExpandedDims`, and 3 more symbols; this block coordinates dialect conversion or lowering decisions; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getReassociationForExpansion`, `getResults`, `getPosition`, `getExpandedDims`, and 3 more symbols 实现具体逻辑；该代码块协调方言转换或 lowering 决策，并协调核心结构化 MLIR 方言之间的行为。

### Lines 726-745
```cpp
/// Update the body of an expanded linalg operation having index semantics. The
/// indices of the original operation need to be recovered by linearizing the
/// indices of the correspoding dimensions of the expanded operation. For now it
/// is assumed that the shapes of the expanded operation needed for
/// linearization are static.
static void updateExpandedGenericOpRegion(PatternRewriter &rewriter,
                                          Location loc, Region &fusedRegion,
                                          const ExpansionInfo &expansionInfo) {
  // Replace the original indices by the linearization of the expanded indices.
  for (IndexOp indexOp :
       llvm::make_early_inc_range(fusedRegion.front().getOps<IndexOp>())) {
    ArrayRef<int64_t> expandedDims =
        expansionInfo.getExpandedDims(indexOp.getDim());
    assert(!expandedDims.empty() && "expected valid expansion info");

    // Skip index operations that are not affected by the expansion.
    if (expandedDims.size() == 1 &&
        expandedDims.front() == (int64_t)indexOp.getDim())
      continue;

```
- **EN**: Implements logic around `updateExpandedGenericOpRegion`, `make_early_inc_range`, `getExpandedDims`, `assert`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `updateExpandedGenericOpRegion`, `make_early_inc_range`, `getExpandedDims`, `assert`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 746-772
```cpp
    // Linearize the expanded indices of the original index dimension.
    OpBuilder::InsertionGuard guard(rewriter);
    rewriter.setInsertionPointAfter(indexOp);
    ArrayRef<OpFoldResult> expandedDimsShape =
        expansionInfo.getExpandedShapeOfDim(indexOp.getDim()).drop_front();
    SmallVector<Value> expandedIndices;
    expandedIndices.reserve(expandedDims.size() - 1);
    llvm::transform(
        expandedDims.drop_front(), std::back_inserter(expandedIndices),
        [&](int64_t dim) { return IndexOp::create(rewriter, loc, dim); });
    OpFoldResult newIndex =
        IndexOp::create(rewriter, loc, expandedDims.front()).getResult();
    for (auto [expandedShape, expandedIndex] :
         llvm::zip(expandedDimsShape, expandedIndices)) {
      AffineExpr idx, acc, shape;
      bindDims(rewriter.getContext(), idx, acc);
      bindSymbols(rewriter.getContext(), shape);
      newIndex = affine::makeComposedFoldedAffineApply(
          rewriter, indexOp.getLoc(), idx + acc * shape,
          ArrayRef<OpFoldResult>{expandedIndex, newIndex, expandedShape});
    }
    Value newIndexVal =
        getValueOrCreateConstantIndexOp(rewriter, indexOp.getLoc(), newIndex);
    rewriter.replaceOp(indexOp, newIndexVal);
  }
}

```
- **EN**: Implements logic around `guard`, `setInsertionPointAfter`, `getExpandedShapeOfDim`, `reserve`, and 10 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `guard`, `setInsertionPointAfter`, `getExpandedShapeOfDim`, `reserve`, and 10 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 773-804
```cpp
// Create an expanded transpose op.
// the reassociation map is already permuted hence we inverse permute and then
// flatten it. Then we inverse permute it again to get the final expanded
// transpose permutation. For example,
//
// permutation = [2, 0, 1]
// reassociation_map for expansion = [[0, 1], [2], [3, 4, 5]]
//
// inverse permutation = [1, 2, 0]
// applied to reassocation_map and then flattened becomes
// flatened permutation = [2, 3, 4, 5, 0, 1]
// final permuation is the inverse of the flattened permutation.
//
// Becomes
//
// permutation=[4, 5, 0, 1, 2, 3]

static Operation *createExpandedTransposeOp(PatternRewriter &rewriter,
                                            TransposeOp transposeOp,
                                            Value expandedInput, Value output,
                                            ExpansionInfo &expansionInfo) {
  SmallVector<int64_t> newPerm;
  for (int64_t perm : invertPermutationVector(transposeOp.getPermutation())) {
    auto reassoc = expansionInfo.getExpandedDims(perm);
    for (int64_t dim : reassoc) {
      newPerm.push_back(dim);
    }
  }
  return TransposeOp::create(rewriter, transposeOp.getLoc(), expandedInput,
                             output, invertPermutationVector(newPerm));
}

```
- **EN**: Implements logic around `createExpandedTransposeOp`, `invertPermutationVector`, `getExpandedDims`, `push_back`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `createExpandedTransposeOp`, `invertPermutationVector`, `getExpandedDims`, `push_back`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 805-825
```cpp
// Create an expanded generic op.
static Operation *createExpandedGenericOp(
    PatternRewriter &rewriter, LinalgOp linalgOp, TypeRange resultTypes,
    ArrayRef<Value> &expandedOpOperands, ArrayRef<Value> outputs,
    ExpansionInfo &expansionInfo, ArrayRef<AffineMap> expandedOpIndexingMaps) {
  // The iterator types of the expanded op are all parallel.
  SmallVector<utils::IteratorType> iteratorTypes(
      expansionInfo.getExpandedOpNumDims(), utils::IteratorType::parallel);

  for (auto [i, type] : llvm::enumerate(linalgOp.getIteratorTypesArray()))
    for (auto j : expansionInfo.getExpandedDims(i))
      iteratorTypes[j] = type;

  Operation *fused = GenericOp::create(rewriter, linalgOp.getLoc(), resultTypes,
                                       expandedOpOperands, outputs,
                                       expandedOpIndexingMaps, iteratorTypes);

  Region &fusedRegion = fused->getRegion(0);
  Region &originalRegion = linalgOp->getRegion(0);
  rewriter.cloneRegionBefore(originalRegion, fusedRegion, fusedRegion.begin());

```
- **EN**: Implements logic around `createExpandedGenericOp`, `iteratorTypes`, `getExpandedOpNumDims`, `enumerate`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `createExpandedGenericOp`, `iteratorTypes`, `getExpandedOpNumDims`, `enumerate`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 826-861
```cpp
  // Update the index accesses after the expansion.
  updateExpandedGenericOpRegion(rewriter, linalgOp.getLoc(), fusedRegion,
                                expansionInfo);

  return fused;
}

// Create an expanded fused op that retains the name for certain ops
// such as fill, copy and transpose and produce a generic op for
// rest of linalg ops.
static Operation *createExpandedOp(PatternRewriter &rewriter, LinalgOp linalgOp,
                                   TypeRange resultTypes,
                                   ArrayRef<Value> expandedOpOperands,
                                   ArrayRef<Value> outputs,
                                   ArrayRef<AffineMap> expandedOpIndexingMaps,
                                   ExpansionInfo &expansionInfo) {

  return TypeSwitch<Operation *, Operation *>(linalgOp.getOperation())
      .Case([&](TransposeOp transposeOp) {
        return createExpandedTransposeOp(rewriter, transposeOp,
                                         expandedOpOperands[0], outputs[0],
                                         expansionInfo);
      })
      .Case<FillOp, CopyOp>([&](Operation *op) {
        return clone(rewriter, linalgOp, resultTypes,
                     llvm::to_vector(llvm::concat<Value>(
                         llvm::to_vector(expandedOpOperands),
                         llvm::to_vector(outputs))));
      })
      .Default([&](Operation *op) {
        return createExpandedGenericOp(rewriter, linalgOp, resultTypes,
                                       expandedOpOperands, outputs,
                                       expansionInfo, expandedOpIndexingMaps);
      });
}

```
- **EN**: Implements logic around `updateExpandedGenericOpRegion`, `createExpandedOp`, `getOperation`, `Case`, and 6 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `updateExpandedGenericOpRegion`, `createExpandedOp`, `getOperation`, `Case`, and 6 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 862-882
```cpp
/// Implements the fusion of a tensor.collapse_shape or a tensor.expand_shape op
/// and a generic op as explained in `isFusableWithReshapeByExpansion`. Assumes
/// that those conditions have been satisfied.
static std::optional<SmallVector<Value>>
fuseWithReshapeByExpansion(LinalgOp linalgOp, Operation *reshapeOp,
                           OpOperand *fusableOpOperand,
                           PatternRewriter &rewriter) {
  assert(isFusableWithReshapeByDimExpansion(linalgOp, fusableOpOperand) &&
         "preconditions for fuse operation failed");

  Location loc = linalgOp.getLoc();
  SmallVector<OpFoldResult> expandedShape;
  SmallVector<AffineMap, 4> reassociationIndices;
  Value src;
  if (auto expandingReshapeOp = dyn_cast<tensor::ExpandShapeOp>(reshapeOp)) {
    // Try to move the dynamic dimensions in output shape before the `linalgOp`
    // to maintain SSA validity
    if (failed(moveValueDefinitions(
            rewriter, expandingReshapeOp.getOutputShape(), linalgOp)))
      return std::nullopt;

```
- **EN**: Implements logic around `fuseWithReshapeByExpansion`, `assert`, `getLoc`, `ExpandShapeOp>`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `fuseWithReshapeByExpansion`, `assert`, `getLoc`, `ExpandShapeOp>`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 883-902
```cpp
    expandedShape = expandingReshapeOp.getMixedOutputShape();
    reassociationIndices = expandingReshapeOp.getReassociationMaps();
    src = expandingReshapeOp.getSrc();
  } else {
    auto collapsingReshapeOp = dyn_cast<tensor::CollapseShapeOp>(reshapeOp);
    if (!collapsingReshapeOp)
      return std::nullopt;

    expandedShape = tensor::getMixedSizes(
        rewriter, collapsingReshapeOp->getLoc(), collapsingReshapeOp.getSrc());
    reassociationIndices = collapsingReshapeOp.getReassociationMaps();
    src = collapsingReshapeOp.getSrc();
  }

  ExpansionInfo expansionInfo;
  if (failed(expansionInfo.compute(linalgOp, fusableOpOperand,
                                   reassociationIndices, expandedShape,
                                   rewriter)))
    return std::nullopt;

```
- **EN**: Implements logic around `getMixedOutputShape`, `getReassociationMaps`, `getSrc`, `CollapseShapeOp>`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getMixedOutputShape`, `getReassociationMaps`, `getSrc`, `CollapseShapeOp>`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 903-938
```cpp
  SmallVector<AffineMap, 4> expandedOpIndexingMaps =
      llvm::map_to_vector<4>(linalgOp.getIndexingMapsArray(), [&](AffineMap m) {
        return getIndexingMapInExpandedOp(rewriter, m, expansionInfo);
      });

  // Set insertion point to the generic op.
  OpBuilder::InsertionGuard g(rewriter);
  rewriter.setInsertionPoint(linalgOp);

  SmallVector<Value> expandedOpOperands;
  expandedOpOperands.reserve(linalgOp.getNumDpsInputs());
  for (OpOperand *opOperand : linalgOp.getDpsInputOperands()) {
    if (opOperand == fusableOpOperand) {
      expandedOpOperands.push_back(src);
      continue;
    }
    if (auto opOperandType =
            dyn_cast<RankedTensorType>(opOperand->get().getType())) {
      AffineMap indexingMap = linalgOp.getMatchingIndexingMap(opOperand);
      SmallVector<OpFoldResult> expandedOperandShape;
      RankedTensorType expandedOperandType;
      std::tie(expandedOperandShape, expandedOperandType) =
          getExpandedShapeAndType(opOperandType, indexingMap, expansionInfo);
      if (expandedOperandType != opOperand->get().getType()) {
        // Reshape the operand to get the right type.
        SmallVector<ReassociationIndices> reassociation =
            getReassociationForExpansion(indexingMap, expansionInfo);
        if (failed(reshapeLikeShapesAreCompatible(
                [&](const Twine &msg) {
                  return rewriter.notifyMatchFailure(linalgOp, msg);
                },
                opOperandType.getShape(), expandedOperandType.getShape(),
                reassociation,
                /*isExpandingReshape=*/true)))
          return std::nullopt;
        expandedOpOperands.push_back(tensor::ExpandShapeOp::create(
```
- **EN**: Implements logic around `map_to_vector`, `getIndexingMapInExpandedOp`, `g`, `setInsertionPoint`, and 11 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `map_to_vector`, `getIndexingMapInExpandedOp`, `g`, `setInsertionPoint`, and 11 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 939-973
```cpp
            rewriter, loc, expandedOperandType, opOperand->get(), reassociation,
            expandedOperandShape));
        continue;
      }
    }
    expandedOpOperands.push_back(opOperand->get());
  }

  SmallVector<Value> outputs;
  for (OpOperand &opOperand : linalgOp.getDpsInitsMutable()) {
    AffineMap indexingMap = linalgOp.getMatchingIndexingMap(&opOperand);
    auto opOperandType = cast<RankedTensorType>(opOperand.get().getType());
    SmallVector<OpFoldResult> expandedOutputShape;
    RankedTensorType expandedOutputType;
    std::tie(expandedOutputShape, expandedOutputType) =
        getExpandedShapeAndType(opOperandType, indexingMap, expansionInfo);
    if (expandedOutputType != opOperand.get().getType()) {
      SmallVector<ReassociationIndices> reassociation =
          getReassociationForExpansion(indexingMap, expansionInfo);
      if (failed(reshapeLikeShapesAreCompatible(
              [&](const Twine &msg) {
                return rewriter.notifyMatchFailure(linalgOp, msg);
              },
              opOperandType.getShape(), expandedOutputType.getShape(),
              reassociation,
              /*isExpandingReshape=*/true)))
        return std::nullopt;
      outputs.push_back(tensor::ExpandShapeOp::create(
          rewriter, loc, expandedOutputType, opOperand.get(), reassociation,
          expandedOutputShape));
    } else {
      outputs.push_back(opOperand.get());
    }
  }

```
- **EN**: Implements logic around `get`, `push_back`, `getDpsInitsMutable`, `getMatchingIndexingMap`, and 6 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `get`, `push_back`, `getDpsInitsMutable`, `getMatchingIndexingMap`, and 6 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 974-999
```cpp
  TypeRange resultTypes = ValueRange(outputs).getTypes();
  Operation *fusedOp =
      createExpandedOp(rewriter, linalgOp, resultTypes, expandedOpOperands,
                       outputs, expandedOpIndexingMaps, expansionInfo);
  // Reshape the result values to their original shape if this is a collapsing
  // reshape folded into its consumer.
  SmallVector<Value> resultVals;
  for (OpResult opResult : linalgOp->getOpResults()) {
    int64_t resultNumber = opResult.getResultNumber();
    if (resultTypes[resultNumber] != opResult.getType()) {
      SmallVector<ReassociationIndices> reassociation =
          getReassociationForExpansion(
              linalgOp.getMatchingIndexingMap(
                  linalgOp.getDpsInitOperand(resultNumber)),
              expansionInfo);
      resultVals.push_back(tensor::CollapseShapeOp::create(
          rewriter, linalgOp.getLoc(), opResult.getType(),
          fusedOp->getResult(resultNumber), reassociation));
    } else {
      resultVals.push_back(fusedOp->getResult(resultNumber));
    }
  }
  // Assuming a single result.
  return resultVals;
}

```
- **EN**: Implements logic around `ValueRange`, `createExpandedOp`, `getOpResults`, `getResultNumber`, and 7 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `ValueRange`, `createExpandedOp`, `getOpResults`, `getResultNumber`, and 7 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1000-1027
```cpp
namespace {

/// Pattern to fuse a tensor.collapse_shape op with its consumer structured op,
/// when the reshape op is collapsing dimensions. The dimensionality of the loop
/// in the consumer is expanded.
class FoldWithProducerReshapeOpByExpansion
    : public OpInterfaceRewritePattern<LinalgOp> {
public:
  FoldWithProducerReshapeOpByExpansion(MLIRContext *context,
                                       ControlFusionFn foldReshapes,
                                       PatternBenefit benefit = 1)
      : OpInterfaceRewritePattern<LinalgOp>(context, benefit),
        controlFoldingReshapes(std::move(foldReshapes)) {}

  LogicalResult matchAndRewrite(LinalgOp linalgOp,
                                PatternRewriter &rewriter) const override {
    for (OpOperand *opOperand : linalgOp.getDpsInputOperands()) {
      tensor::CollapseShapeOp reshapeOp =
          opOperand->get().getDefiningOp<tensor::CollapseShapeOp>();
      if (!reshapeOp)
        continue;
      // Fold only if
      // - The tensor reshape op is folding.
      // - All constraints of fusing with reshape by expansion are met.
      if (!isFusableWithReshapeByDimExpansion(linalgOp, opOperand) ||
          (!controlFoldingReshapes(opOperand)))
        continue;

```
- **EN**: Introduces declarations for `FoldWithProducerReshapeOpByExpansion`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `FoldWithProducerReshapeOpByExpansion` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 1028-1046
```cpp
      std::optional<SmallVector<Value>> replacementValues =
          fuseWithReshapeByExpansion(linalgOp, reshapeOp, opOperand, rewriter);
      if (!replacementValues)
        return failure();
      rewriter.replaceOp(linalgOp, *replacementValues);
      return success();
    }
    return failure();
  }

private:
  ControlFusionFn controlFoldingReshapes;
};

/// Carries information about a padded dimension.
struct PadDimInfo {
  // The resulting shape after padding each dimension.
  SmallVector<int64_t> paddedShape;

```
- **EN**: Introduces declarations for `PadDimInfo`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `PadDimInfo` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 1047-1068
```cpp
  // Low and high padding amounts for each dimension.
  SmallVector<OpFoldResult> lowPad;
  SmallVector<OpFoldResult> highPad;
};

/// Computes the expanded padding information for the given pad operation based
/// on the provided expanded shape and reassociation indices. Returns a list of
/// PadDimInfo containing the low and high padding amounts and the padded
/// size for each dimension, or failure if the expansion is not possible.
static FailureOr<PadDimInfo>
computeExpandedPadding(tensor::PadOp padOp, ArrayRef<int64_t> expandedShape,
                       ArrayRef<ReassociationIndices> reassociations,
                       PatternRewriter &rewriter) {
  // If the padding value depends on the index values of the pad operation,
  // then it may not be valid to expand the dimensions, since it will change
  // the index values on which the padding value depends. This is not currently
  // supported by the pad expansion patterns, but it could be implemented
  // similarly to the expansion of linalg.generic ops with linalg.index ops in
  // the body, as is done in `updateExpandedGenericOpRegion`.
  if (!padOp.getConstantPaddingValue())
    return failure();

```
- **EN**: Implements logic around `computeExpandedPadding`, `getConstantPaddingValue`, `failure`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `computeExpandedPadding`, `getConstantPaddingValue`, `failure` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1069-1094
```cpp
  // Expanded dimensions cannot have padding because the resulting padding may
  // not be representable by a tensor.pad op. There are some special cases where
  // it is possible (like expanding unit dims), but supporting these cases is
  // NYI, so disallow it for now.
  ArrayRef<int64_t> low = padOp.getStaticLow();
  ArrayRef<int64_t> high = padOp.getStaticHigh();
  for (auto [reInd, l, h] : llvm::zip_equal(reassociations, low, high)) {
    if (reInd.size() != 1 && (l != 0 || h != 0))
      return failure();
  }

  SmallVector<OpFoldResult> mixedLowPad(padOp.getMixedLowPad());
  SmallVector<OpFoldResult> mixedHighPad(padOp.getMixedHighPad());
  ArrayRef<int64_t> paddedShape = padOp.getResultType().getShape();
  PadDimInfo padDimInfo;
  padDimInfo.paddedShape.assign(expandedShape);
  padDimInfo.lowPad.assign(expandedShape.size(), rewriter.getIndexAttr(0));
  padDimInfo.highPad.assign(expandedShape.size(), rewriter.getIndexAttr(0));
  for (auto [idx, reInd] : llvm::enumerate(reassociations)) {
    if (reInd.size() == 1) {
      padDimInfo.paddedShape[reInd[0]] = paddedShape[idx];
      padDimInfo.lowPad[reInd[0]] = mixedLowPad[idx];
      padDimInfo.highPad[reInd[0]] = mixedHighPad[idx];
    }
  }

```
- **EN**: Implements logic around `getStaticLow`, `getStaticHigh`, `zip_equal`, `size`, and 6 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getStaticLow`, `getStaticHigh`, `zip_equal`, `size`, and 6 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1095-1113
```cpp
  return padDimInfo;
}

class FoldPadWithProducerReshapeOpByExpansion
    : public OpRewritePattern<tensor::PadOp> {
public:
  FoldPadWithProducerReshapeOpByExpansion(MLIRContext *context,
                                          ControlFusionFn foldReshapes,
                                          PatternBenefit benefit = 1)
      : OpRewritePattern<tensor::PadOp>(context, benefit),
        controlFoldingReshapes(std::move(foldReshapes)) {}

  LogicalResult matchAndRewrite(tensor::PadOp padOp,
                                PatternRewriter &rewriter) const override {
    tensor::CollapseShapeOp reshapeOp =
        padOp.getSource().getDefiningOp<tensor::CollapseShapeOp>();
    if (!reshapeOp)
      return failure();

```
- **EN**: Introduces declarations for `FoldPadWithProducerReshapeOpByExpansion`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `FoldPadWithProducerReshapeOpByExpansion` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 1114-1131
```cpp
    if (!controlFoldingReshapes(&padOp.getSourceMutable())) {
      return rewriter.notifyMatchFailure(padOp,
                                         "fusion blocked by control function");
    }

    RankedTensorType expandedType = reshapeOp.getSrcType();
    SmallVector<ReassociationIndices> reassociations =
        reshapeOp.getReassociationIndices();
    FailureOr<PadDimInfo> maybeExpandedPadding = computeExpandedPadding(
        padOp, expandedType.getShape(), reassociations, rewriter);
    if (failed(maybeExpandedPadding))
      return failure();
    PadDimInfo &expandedPadding = maybeExpandedPadding.value();

    Location loc = padOp->getLoc();
    RankedTensorType expandedPaddedType =
        padOp.getResultType().clone(expandedPadding.paddedShape);

```
- **EN**: Implements logic around `controlFoldingReshapes`, `notifyMatchFailure`, `getSrcType`, `getReassociationIndices`, and 7 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `controlFoldingReshapes`, `notifyMatchFailure`, `getSrcType`, `getReassociationIndices`, and 7 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 1132-1149
```cpp
    auto newPadOp = tensor::PadOp::create(
        rewriter, loc, expandedPaddedType, reshapeOp.getSrc(),
        expandedPadding.lowPad, expandedPadding.highPad,
        padOp.getConstantPaddingValue(), padOp.getNofold());

    rewriter.replaceOpWithNewOp<tensor::CollapseShapeOp>(
        padOp, padOp.getResultType(), newPadOp.getResult(), reassociations);

    return success();
  }

private:
  ControlFusionFn controlFoldingReshapes;
};

class FoldReshapeWithProducerPadOpByExpansion
    : public OpRewritePattern<tensor::ExpandShapeOp> {
public:
```
- **EN**: Introduces declarations for `FoldReshapeWithProducerPadOpByExpansion`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `FoldReshapeWithProducerPadOpByExpansion` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 1150-1175
```cpp
  FoldReshapeWithProducerPadOpByExpansion(MLIRContext *context,
                                          ControlFusionFn foldReshapes,
                                          PatternBenefit benefit = 1)
      : OpRewritePattern<tensor::ExpandShapeOp>(context, benefit),
        controlFoldingReshapes(std::move(foldReshapes)) {}

  LogicalResult matchAndRewrite(tensor::ExpandShapeOp expandOp,
                                PatternRewriter &rewriter) const override {
    tensor::PadOp padOp = expandOp.getSrc().getDefiningOp<tensor::PadOp>();
    if (!padOp)
      return failure();

    if (!controlFoldingReshapes(&expandOp.getSrcMutable())) {
      return rewriter.notifyMatchFailure(expandOp,
                                         "fusion blocked by control function");
    }

    RankedTensorType expandedType = expandOp.getResultType();
    SmallVector<ReassociationIndices> reassociations =
        expandOp.getReassociationIndices();
    FailureOr<PadDimInfo> maybeExpandedPadding = computeExpandedPadding(
        padOp, expandedType.getShape(), reassociations, rewriter);
    if (failed(maybeExpandedPadding))
      return failure();
    PadDimInfo &expandedPadding = maybeExpandedPadding.value();

```
- **EN**: Implements logic around `FoldReshapeWithProducerPadOpByExpansion`, `ExpandShapeOp>`, `controlFoldingReshapes`, `matchAndRewrite`, and 9 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `FoldReshapeWithProducerPadOpByExpansion`, `ExpandShapeOp>`, `controlFoldingReshapes`, `matchAndRewrite`, and 9 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1176-1201
```cpp
    Location loc = expandOp->getLoc();
    SmallVector<OpFoldResult> newExpandedSizes = expandOp.getMixedOutputShape();
    SmallVector<int64_t> newExpandedShape(expandedType.getShape());
    rewriter.setInsertionPointAfterValue(padOp.getSource());
    SmallVector<OpFoldResult> padSrcSizes =
        tensor::getMixedSizes(rewriter, loc, padOp.getSource());
    for (auto [idx, reInd] : llvm::enumerate(reassociations)) {
      // We know that any reassociation with multiple dims is not padded because
      // of the requirements of computeExpandedPadding.
      if (reInd.size() == 1) {
        newExpandedShape[reInd[0]] = padOp.getSourceType().getDimSize(idx);
        newExpandedSizes[reInd[0]] = padSrcSizes[idx];
      }
    }
    RankedTensorType newExpandedType = expandedType.clone(newExpandedShape);
    auto newExpandOp = tensor::ExpandShapeOp::create(
        rewriter, loc, newExpandedType, padOp.getSource(), reassociations,
        newExpandedSizes);
    RankedTensorType expandedPaddedType =
        padOp.getResultType().clone(expandedPadding.paddedShape);
    rewriter.setInsertionPoint(expandOp);
    auto newPadOp = tensor::PadOp::create(
        rewriter, loc, expandedPaddedType, newExpandOp.getResult(),
        expandedPadding.lowPad, expandedPadding.highPad,
        padOp.getConstantPaddingValue(), padOp.getNofold());

```
- **EN**: Implements logic around `getLoc`, `getMixedOutputShape`, `newExpandedShape`, `setInsertionPointAfterValue`, and 11 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getLoc`, `getMixedOutputShape`, `newExpandedShape`, `setInsertionPointAfterValue`, and 11 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1202-1221
```cpp
    rewriter.replaceOp(expandOp, newPadOp.getResult());

    return success();
  }

private:
  ControlFusionFn controlFoldingReshapes;
};

/// Pattern to fold a tensor.expand_shape op with its producer generic op
/// by expanding the dimensionality of the loop in the producer op.
struct FoldReshapeWithGenericOpByExpansion
    : public OpRewritePattern<tensor::ExpandShapeOp> {

  FoldReshapeWithGenericOpByExpansion(MLIRContext *context,
                                      ControlFusionFn foldReshapes,
                                      PatternBenefit benefit = 1)
      : OpRewritePattern<tensor::ExpandShapeOp>(context, benefit),
        controlFoldingReshapes(std::move(foldReshapes)) {}

```
- **EN**: Introduces declarations for `FoldReshapeWithGenericOpByExpansion`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `FoldReshapeWithGenericOpByExpansion` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 1222-1243
```cpp
  LogicalResult matchAndRewrite(tensor::ExpandShapeOp reshapeOp,
                                PatternRewriter &rewriter) const override {
    // Fold only if all constraints of fusing with reshape by expansion are met.
    auto producerResult = dyn_cast<OpResult>(reshapeOp.getSrc());
    if (!producerResult) {
      return rewriter.notifyMatchFailure(reshapeOp,
                                         "source not produced by an operation");
    }

    auto producer = dyn_cast<LinalgOp>(producerResult.getOwner());
    if (!producer) {
      return rewriter.notifyMatchFailure(reshapeOp,
                                         "producer not a generic op");
    }

    if (!isFusableWithReshapeByDimExpansion(
            producer,
            producer.getDpsInitOperand(producerResult.getResultNumber()))) {
      return rewriter.notifyMatchFailure(
          reshapeOp, "failed preconditions of fusion with producer generic op");
    }

```
- **EN**: Implements logic around `matchAndRewrite`, `getSrc`, `notifyMatchFailure`, `getOwner`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; performs canonicalization or folding-oriented reasoning; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `matchAndRewrite`, `getSrc`, `notifyMatchFailure`, `getOwner`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并执行面向规范化或折叠的推理，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1244-1274
```cpp
    if (!controlFoldingReshapes(&reshapeOp.getSrcMutable())) {
      return rewriter.notifyMatchFailure(reshapeOp,
                                         "fusion blocked by control function");
    }

    std::optional<SmallVector<Value>> replacementValues =
        fuseWithReshapeByExpansion(
            producer, reshapeOp,
            producer.getDpsInitOperand(producerResult.getResultNumber()),
            rewriter);
    if (!replacementValues) {
      return rewriter.notifyMatchFailure(reshapeOp,
                                         "fusion by expansion failed");
    }

    // Find the replacement for the reshape op. Since the replacements have the
    // same type as the returns of the original generic op, the consumer reshape
    // op can be replaced by the source of the collapse_shape op that defines
    // the replacement.
    Value reshapeReplacement =
        (*replacementValues)[cast<OpResult>(reshapeOp.getSrc())
                                 .getResultNumber()];
    if (auto collapseOp =
            reshapeReplacement.getDefiningOp<tensor::CollapseShapeOp>()) {
      reshapeReplacement = collapseOp.getSrc();
    }
    rewriter.replaceOp(reshapeOp, reshapeReplacement);
    rewriter.replaceOp(producer, *replacementValues);
    return success();
  }

```
- **EN**: Implements logic around `controlFoldingReshapes`, `notifyMatchFailure`, `fuseWithReshapeByExpansion`, `getDpsInitOperand`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `controlFoldingReshapes`, `notifyMatchFailure`, `fuseWithReshapeByExpansion`, `getDpsInitOperand`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1275-1294
```cpp
private:
  ControlFusionFn controlFoldingReshapes;
};
} // namespace

//===---------------------------------------------------------------------===//
// Methods and patterns to fuse reshape with linalg.generic operations by
// contraction of dimensions.
//===---------------------------------------------------------------------===//

/// For a given list of indices in the range of the `indexingMap` that are
/// folded, return the indices of the corresponding domain. Return
/// `std::nullopt` on failure. Ensures that all the elements of the returned
/// reassociation are distinct.
static ReassociationIndices
getDomainReassociation(AffineMap indexingMap,
                       ReassociationIndicesRef rangeReassociation) {
  assert(indexingMap.isProjectedPermutation() &&
         "expected projected permutation");

```
- **EN**: Implements logic around `getDomainReassociation`, `assert`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getDomainReassociation`, `assert` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1295-1316
```cpp
  ReassociationIndices domainReassociation =
      llvm::map_to_vector<4>(rangeReassociation, [&](int64_t pos) -> int64_t {
        return cast<AffineDimExpr>(indexingMap.getResults()[pos]).getPosition();
      });
  // The projected permutation semantics ensures that there is no repetition of
  // the domain indices.
  return domainReassociation;
}

/// For a given `dimSequence`, check if the sequence is conserved in the
/// `indexingMap`. `indexingMap` is expected to be a projected permutation.
/// Non-existence of the sequence returns true as well.
bool mlir::linalg::isDimSequencePreserved(AffineMap indexingMap,
                                          ReassociationIndicesRef dimSequence) {
  assert(!dimSequence.empty() &&
         "expected non-empty list for dimension sequence");

  // Dimension sequences can only be preserved in projected permutation maps.
  if (!indexingMap.isProjectedPermutation()) {
    return false;
  }

```
- **EN**: Implements logic around `map_to_vector`, `getResults`, `isDimSequencePreserved`, `assert`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `map_to_vector`, `getResults`, `isDimSequencePreserved`, `assert`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1317-1350
```cpp
  llvm::SmallDenseSet<unsigned, 4> sequenceElements;
  sequenceElements.insert_range(dimSequence);

  unsigned dimSequenceStart = dimSequence[0];
  for (const auto &expr : enumerate(indexingMap.getResults())) {
    unsigned dimInMapStart = cast<AffineDimExpr>(expr.value()).getPosition();
    // 1.  Check if this start of the sequence.
    if (dimInMapStart == dimSequenceStart) {
      if (expr.index() + dimSequence.size() > indexingMap.getNumResults())
        return false;
      // 1a. Check if sequence is preserved.
      for (const auto &dimInSequence : enumerate(dimSequence)) {
        unsigned dimInMap =
            cast<AffineDimExpr>(
                indexingMap.getResult(expr.index() + dimInSequence.index()))
                .getPosition();
        if (dimInMap != dimInSequence.value())
          return false;
      }
      // Found the sequence. Projected permutation
      // enforces that all AffineDimExprs in the result are unique, so no
      // further checks are needed.
      return true;
    }
    // 2. If position in the expr (which is of type AffineDimExpr) is part
    // of sequence, return false here. This implies the entire sequence does not
    // exist in the indexing map.
    if (sequenceElements.count(dimInMapStart))
      return false;
  }
  // 3. No element of sequence found. Return true.
  return true;
}

```
- **EN**: Implements logic around `insert_range`, `enumerate`, `value`, `index`, and 3 more symbols.
- **CN**: 围绕 `insert_range`, `enumerate`, `value`, `index`, and 3 more symbols 实现具体逻辑。

### Lines 1351-1386
```cpp
bool mlir::linalg::areDimSequencesPreserved(
    ArrayRef<AffineMap> maps, ArrayRef<ReassociationIndices> dimSequences) {
  return llvm::all_of(maps, [&](AffineMap map) {
    return llvm::all_of(dimSequences, [&](ReassociationIndicesRef dimSequence) {
      return isDimSequencePreserved(map, dimSequence);
    });
  });
}

// Return the list of dimensions of the iteration domain that can be
// collapsed to allow for fusion with the a producer that is an expand_shape
// operation. If all dimensions created by expansion can be collapsed in the
// iteration space then the reshape is defunct.
//
// Example:
//
// ```mlir
// #map = affine_map<(d0, d1) -> (d0, d1)>
// %1 = tensor.expand_shape %0 [[0, 1]] : tensor<?xf32> into tensor<?x4xf32>
// %2 = tensor.empty [..] : tensor<?x4xf32>
// %3 = linalg.generic {
//     indexing_maps = [#map, #map],
//     iterator_types = ["parallel" ,"parallel"]}
//     ins(%1 : tensor<?x4xf32>) outs(%2 : tensor<?x4xf32>) {.. }
// ```
//
// can be fused by collapsing the dimensions of the iteration space.
//
// ```mlir
// #map = affine_map<(d0) -> (d0)>
// %2 = tensor.empty [..] : tensor<?xf32>
// %3 = linalg.generic {
//     indexing_maps = [#map, #map],
//     iterator_types = ["parallel"]}
//     ins(%1 : tensor<?xf32>) outs(%2 : tensor<?xf32>) {.. }
// %4 = tensor.expand_shape %3 [[0, 1]] : tensor<?xf32> into tensor<?x4xf32>
```
- **EN**: Implements logic around `areDimSequencesPreserved`, `all_of`, `isDimSequencePreserved`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `areDimSequencesPreserved`, `all_of`, `isDimSequencePreserved` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1387-1412
```cpp
// ```
//
// In the following example,
//
// ```mlir
// #map0 = affine_map<(d0, d1) -> (d0, d1)>
// #map1 = affine_map<(d0, d1) -> (d1, d0)>
// %1 = tensor.expand_shape %0 [[0, 1]] : tensor<?xf32> into tensor<?x4xf32>
// %2 = tensor.empty [..] : tensor<4x?xf32>
// %2 = linalg.generic {
//     indexing_maps = [#map0, #map1],
//     iterator_types = ["parallel" ,"parallel"]}
//     ins(%1 : tensor<?x4xf32>) outs(%2 : tensor<4x?xf32>) {.. }
// ```
//
// the reshape cannot be fused with the generic op by collapsing the op
// dimensions since the indexing maps will have to contain mods and divs
// to preserve the accesses pattern. When no dimensions of the iteration
// space are collapsable and empty vector is returned.
static SmallVector<ReassociationIndices>
getCollapsableIterationSpaceDims(GenericOp genericOp, OpOperand *fusableOperand,
                                 ArrayRef<ReassociationIndices> reassociation) {
  // Some basic checks for this fusion to be valid.
  if (!genericOp.hasPureTensorSemantics())
    return {};

```
- **EN**: Implements logic around `getCollapsableIterationSpaceDims`, `hasPureTensorSemantics`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getCollapsableIterationSpaceDims`, `hasPureTensorSemantics` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1413-1433
```cpp
  if (!llvm::all_of(genericOp.getIndexingMapsArray(), [](AffineMap map) {
        return map.isProjectedPermutation();
      })) {
    return {};
  }

  // Compute all the loops with the reduction iterator types.
  SmallVector<unsigned> reductionDims;
  genericOp.getReductionDims(reductionDims);

  llvm::SmallDenseSet<unsigned, 4> processedIterationDims;
  AffineMap indexingMap = genericOp.getMatchingIndexingMap(fusableOperand);
  auto iteratorTypes = genericOp.getIteratorTypesArray();
  SmallVector<ReassociationIndices> iterationSpaceReassociation;
  for (ReassociationIndicesRef foldedRangeDims : reassociation) {
    assert(!foldedRangeDims.empty() && "unexpected empty reassociation");

    // Ignore dims that are not folded.
    if (foldedRangeDims.size() == 1)
      continue;

```
- **EN**: Implements logic around `all_of`, `isProjectedPermutation`, `getReductionDims`, `getMatchingIndexingMap`, and 3 more symbols.
- **CN**: 围绕 `all_of`, `isProjectedPermutation`, `getReductionDims`, `getMatchingIndexingMap`, and 3 more symbols 实现具体逻辑。

### Lines 1434-1454
```cpp
    ReassociationIndices foldedIterationSpaceDims =
        getDomainReassociation(indexingMap, foldedRangeDims);

    // Check that the folded iteration dims do not contain already processed
    // dims.
    if (llvm::any_of(foldedIterationSpaceDims, [&](int64_t dim) {
          return processedIterationDims.count(dim);
        }))
      continue;

    // Check that all folded iterator types are all parallel or all reductions.
    utils::IteratorType startIteratorType =
        iteratorTypes[foldedIterationSpaceDims[0]];
    if (!isParallelIterator(startIteratorType) &&
        !isReductionIterator(startIteratorType))
      continue;
    if (llvm::any_of(foldedIterationSpaceDims, [&](int64_t dim) {
          return iteratorTypes[dim] != startIteratorType;
        }))
      continue;

```
- **EN**: Implements logic around `getDomainReassociation`, `any_of`, `count`, `isParallelIterator`, and 1 more symbols.
- **CN**: 围绕 `getDomainReassociation`, `any_of`, `count`, `isParallelIterator`, and 1 more symbols 实现具体逻辑。

### Lines 1455-1485
```cpp
    // If the folded dimensions correspond to a "reduction" iterator type,
    // the folded dimensions need to be "in-order". Strictly speaking this is
    // not necessary, for reductions that are associative and commutative,  but
    // using a more strict definition of reduction for now.
    if (isReductionIterator(startIteratorType)) {
      bool isContiguous = false;
      for (const auto &startDim : llvm::enumerate(reductionDims)) {
        // Move window in `reductionDims` to start of the folded iteration dims.
        if (startDim.value() != foldedIterationSpaceDims[0])
          continue;
        // If sizes doesnt match, trivial not contiguous. This condition should
        // not be hit.
        if (startDim.index() + foldedIterationSpaceDims.size() >
            reductionDims.size())
          break;
        // Check that the contiguity is maintained.
        isContiguous = true;
        for (const auto &foldedDim :
             llvm::enumerate(foldedIterationSpaceDims)) {
          if (reductionDims[foldedDim.index() + startDim.index()] !=
              foldedDim.value()) {
            isContiguous = false;
            break;
          }
        }
        break;
      }
      if (!isContiguous)
        continue;
    }

```
- **EN**: Implements logic around `isReductionIterator`, `enumerate`, `value`, `index`, and 1 more symbols.
- **CN**: 围绕 `isReductionIterator`, `enumerate`, `value`, `index`, and 1 more symbols 实现具体逻辑。

### Lines 1486-1503
```cpp
    // Check that the sequence is preserved in all indexing maps.
    if (llvm::any_of(genericOp.getIndexingMapsArray(),
                     [&](AffineMap indexingMap) {
                       return !isDimSequencePreserved(indexingMap,
                                                      foldedIterationSpaceDims);
                     }))
      continue;

    processedIterationDims.insert_range(foldedIterationSpaceDims);
    iterationSpaceReassociation.emplace_back(
        std::move(foldedIterationSpaceDims));
  }

  return iterationSpaceReassociation;
}

/// Helper class to carry state while collapsing the `linalg.generic` op.
namespace {
```
- **EN**: Introduces declarations for `to`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `to` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 1504-1527
```cpp
class CollapsingInfo {
public:
  LogicalResult initialize(unsigned origNumLoops,
                           ArrayRef<ReassociationIndices> foldedIterationDims) {
    llvm::SmallDenseSet<int64_t, 4> processedDims;
    // Find all the dims that are folded.
    for (ReassociationIndicesRef foldedIterationDim : foldedIterationDims) {
      if (foldedIterationDim.empty())
        continue;
      // If the folded dims contain dims already folded, that's illegal
      // specification. Repetition within a list is also illegal.
      for (auto dim : foldedIterationDim) {
        if (dim >= origNumLoops)
          return failure();
        if (processedDims.count(dim))
          return failure();
        processedDims.insert(dim);
      }
      collapsedOpToOrigOpIterationDim.emplace_back(foldedIterationDim.begin(),
                                                   foldedIterationDim.end());
    }
    if (processedDims.size() > origNumLoops)
      return failure();

```
- **EN**: Introduces declarations for `CollapsingInfo`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `CollapsingInfo` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 1528-1549
```cpp
    // Add all the preserved dims of the original op as single
    // elements to `collapsedOpToOrigOpIterationDim`.
    for (auto dim : llvm::seq<int64_t>(0, origNumLoops)) {
      if (processedDims.count(dim))
        continue;
      collapsedOpToOrigOpIterationDim.emplace_back(ReassociationIndices{dim});
    }

    llvm::sort(collapsedOpToOrigOpIterationDim,
               [&](ReassociationIndicesRef lhs, ReassociationIndicesRef rhs) {
                 return lhs[0] < rhs[0];
               });
    origOpToCollapsedOpIterationDim.resize(origNumLoops);
    for (const auto &foldedDims :
         llvm::enumerate(collapsedOpToOrigOpIterationDim)) {
      for (const auto &dim : enumerate(foldedDims.value()))
        origOpToCollapsedOpIterationDim[dim.value()] =
            std::make_pair<int64_t, unsigned>(foldedDims.index(), dim.index());
    }
    return success();
  }

```
- **EN**: Implements logic around `seq`, `count`, `emplace_back`, `sort`, and 5 more symbols.
- **CN**: 围绕 `seq`, `count`, `emplace_back`, `sort`, and 5 more symbols 实现具体逻辑。

### Lines 1550-1567
```cpp
  /// Return mapping from collapsed loop domain to original loop domain.
  ArrayRef<ReassociationIndices> getCollapsedOpToOrigOpMapping() const {
    return collapsedOpToOrigOpIterationDim;
  }

  /// Return mapping from original loop domain to collapsed loop domain. The
  /// mapping is a pair. First value is the dimension in the collapsed loop that
  /// the original loop is mapped to. Second is the relative position in folded
  /// list of this domain. For example if the original loop domain is 3D, and
  /// the collapsed loop domain is folding all of it, i.e.
  ///
  /// ```
  /// collapsedOpToOrigOpMapping = [[0, 1, 2] [3, 4]]`
  /// ```
  ///
  /// then
  ///
  /// ```
```
- **EN**: Implements logic around `getCollapsedOpToOrigOpMapping`.
- **CN**: 围绕 `getCollapsedOpToOrigOpMapping` 实现具体逻辑。

### Lines 1568-1585
```cpp
  ///  origOpToCollapsedOpMapping[0] = {0, 0};
  ///  origOpToCollapsedOpMapping[1] = {0, 1};
  ///  origOpToCollapsedOpMapping[2] = {0, 2};
  ///  origOpToCollapsedOpMapping[3] = {1, 0};
  ///  origOpToCollapsedOpMapping[4] = {1, 1};
  /// ```
  ///
  ArrayRef<std::pair<int64_t, unsigned>> getOrigOpToCollapsedOpMapping() const {
    return origOpToCollapsedOpIterationDim;
  }

  /// Return the collapsed op iteration domain rank.
  unsigned getCollapsedOpIterationRank() const {
    return collapsedOpToOrigOpIterationDim.size();
  }

private:
  /// Map from the iteration domain index in collapsed op to the iteration
```
- **EN**: Implements logic around `getOrigOpToCollapsedOpMapping`, `getCollapsedOpIterationRank`, `size`.
- **CN**: 围绕 `getOrigOpToCollapsedOpMapping`, `getCollapsedOpIterationRank`, `size` 实现具体逻辑。

### Lines 1586-1612
```cpp
  /// domain indices in the original op.
  SmallVector<ReassociationIndices> collapsedOpToOrigOpIterationDim;

  /// Map from iteration domain index in the original op to the iteration domain
  /// index in the collapsed op.
  SmallVector<std::pair<int64_t, unsigned>> origOpToCollapsedOpIterationDim;
};
} // namespace

/// Get the iterator types for the collapsed operation given the original
/// iterator types and collapsed dimensions.
static SmallVector<utils::IteratorType>
getCollapsedOpIteratorTypes(ArrayRef<utils::IteratorType> iteratorTypes,
                            const CollapsingInfo &collapsingInfo) {
  SmallVector<utils::IteratorType> collapsedIteratorTypes;
  for (ReassociationIndicesRef foldedIterDims :
       collapsingInfo.getCollapsedOpToOrigOpMapping()) {
    assert(!foldedIterDims.empty() &&
           "reassociation indices expected to have non-empty sets");
    // Just pick the iterator type of the first folded dim. Pre-condition checks
    // expected to have checked that iterator types of all folded dimensions are
    // the same.
    collapsedIteratorTypes.push_back(iteratorTypes[foldedIterDims[0]]);
  }
  return collapsedIteratorTypes;
}

```
- **EN**: Implements logic around `getCollapsedOpIteratorTypes`, `getCollapsedOpToOrigOpMapping`, `assert`, `push_back`.
- **CN**: 围绕 `getCollapsedOpIteratorTypes`, `getCollapsedOpToOrigOpMapping`, `assert`, `push_back` 实现具体逻辑。

### Lines 1613-1637
```cpp
/// Compute the indexing map in the collapsed op that corresponds to the given
/// `indexingMap` of the original operation.
static AffineMap
getCollapsedOpIndexingMap(AffineMap indexingMap,
                          const CollapsingInfo &collapsingInfo) {
  MLIRContext *context = indexingMap.getContext();
  assert(indexingMap.isProjectedPermutation() &&
         "expected indexing map to be projected permutation");
  SmallVector<AffineExpr> resultExprs;
  auto origOpToCollapsedOpMapping =
      collapsingInfo.getOrigOpToCollapsedOpMapping();
  for (auto expr : indexingMap.getResults()) {
    unsigned dim = cast<AffineDimExpr>(expr).getPosition();
    // If the dim is not the first of the collapsed dim, do nothing.
    if (origOpToCollapsedOpMapping[dim].second != 0)
      continue;
    // The next n-dims are guaranteed to be collapsed. So just use the
    // iteration dimension of the collapsed op.
    resultExprs.push_back(
        getAffineDimExpr(origOpToCollapsedOpMapping[dim].first, context));
  }
  return AffineMap::get(collapsingInfo.getCollapsedOpIterationRank(), 0,
                        resultExprs, context);
}

```
- **EN**: Implements logic around `getCollapsedOpIndexingMap`, `getContext`, `assert`, `getOrigOpToCollapsedOpMapping`, and 5 more symbols.
- **CN**: 围绕 `getCollapsedOpIndexingMap`, `getContext`, `assert`, `getOrigOpToCollapsedOpMapping`, and 5 more symbols 实现具体逻辑。

### Lines 1638-1666
```cpp
/// Return the `reassociation` indices to use to collapse the operand when the
/// iteration space of a generic op is collapsed.
static SmallVector<ReassociationIndices>
getOperandReassociation(AffineMap indexingMap,
                        const CollapsingInfo &collapsingInfo) {
  unsigned counter = 0;
  SmallVector<ReassociationIndices> operandReassociation;
  auto origOpToCollapsedOpMapping =
      collapsingInfo.getOrigOpToCollapsedOpMapping();
  auto collapsedOpToOrigOpMapping =
      collapsingInfo.getCollapsedOpToOrigOpMapping();
  while (counter < indexingMap.getNumResults()) {
    unsigned dim =
        cast<AffineDimExpr>(indexingMap.getResult(counter)).getPosition();
    // This is the start of a collapsed dimensions of the iteration that
    // is gauranteed to be preserved in the indexing map. The number of folded
    // dims is obtained from the collapsed op to original op mapping.
    unsigned numFoldedDims =
        collapsedOpToOrigOpMapping[origOpToCollapsedOpMapping[dim].first]
            .size();
    if (origOpToCollapsedOpMapping[dim].second == 0) {
      auto range = llvm::seq<unsigned>(counter, counter + numFoldedDims);
      operandReassociation.emplace_back(range.begin(), range.end());
    }
    counter += numFoldedDims;
  }
  return operandReassociation;
}

```
- **EN**: Implements logic around `getOperandReassociation`, `getOrigOpToCollapsedOpMapping`, `getCollapsedOpToOrigOpMapping`, `getNumResults`, and 4 more symbols.
- **CN**: 围绕 `getOperandReassociation`, `getOrigOpToCollapsedOpMapping`, `getCollapsedOpToOrigOpMapping`, `getNumResults`, and 4 more symbols 实现具体逻辑。

### Lines 1667-1693
```cpp
/// Get the new value to use for a given `OpOperand` in the collapsed operation.
static Value getCollapsedOpOperand(Location loc, LinalgOp op,
                                   OpOperand *opOperand,
                                   const CollapsingInfo &collapsingInfo,
                                   OpBuilder &builder) {
  AffineMap indexingMap = op.getMatchingIndexingMap(opOperand);
  SmallVector<ReassociationIndices> operandReassociation =
      getOperandReassociation(indexingMap, collapsingInfo);

  // If the number of entries in the reassociation for the operand is same as
  // the number of results of the indexing map, then nothing to do for this
  // operand.
  Value operand = opOperand->get();
  if (operandReassociation.size() == indexingMap.getNumResults())
    return operand;

  // Insert a reshape to collapse the dimensions.
  if (isa<MemRefType>(operand.getType())) {
    return memref::CollapseShapeOp::create(builder, loc, operand,
                                           operandReassociation)
        .getResult();
  }
  return tensor::CollapseShapeOp::create(builder, loc, operand,
                                         operandReassociation)
      .getResult();
}

```
- **EN**: Implements logic around `getCollapsedOpOperand`, `getMatchingIndexingMap`, `getOperandReassociation`, `get`, and 4 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getCollapsedOpOperand`, `getMatchingIndexingMap`, `getOperandReassociation`, `get`, and 4 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1694-1728
```cpp
/// Modify the `linalg.index` operations in the original generic op, to its
/// value in the collapsed operation.
static void generateCollapsedIndexingRegion(
    Location loc, Block *block, const CollapsingInfo &collapsingInfo,
    ArrayRef<OpFoldResult> loopRange, RewriterBase &rewriter) {
  OpBuilder::InsertionGuard g(rewriter);
  rewriter.setInsertionPointToStart(block);

  // Collect all the original index ops.
  auto indexOps = llvm::to_vector(block->getOps<linalg::IndexOp>());

  // For each folded dimension list resolve the original induction variable
  // values in terms of the folded dimension induction variable.
  //   i_{folded} = (i_0 * d1 + i1) * d2 + i2.
  // can be inverted to
  //   i2 = i_{folded} % d2
  //   i1 = (i_{folded} / d2) % d1
  //   i0 = i_{folded} / (d1 * d2)
  llvm::DenseMap<unsigned, Value> indexReplacementVals;
  for (auto foldedDims :
       enumerate(collapsingInfo.getCollapsedOpToOrigOpMapping())) {
    ReassociationIndicesRef foldedDimsRef(foldedDims.value());
    Value newIndexVal =
        linalg::IndexOp::create(rewriter, loc, foldedDims.index());
    for (auto dim : llvm::reverse(foldedDimsRef.drop_front())) {
      Value loopDim =
          getValueOrCreateConstantIndexOp(rewriter, loc, loopRange[dim]);
      indexReplacementVals[dim] =
          rewriter.createOrFold<arith::RemSIOp>(loc, newIndexVal, loopDim);
      newIndexVal =
          rewriter.createOrFold<arith::DivSIOp>(loc, newIndexVal, loopDim);
    }
    indexReplacementVals[foldedDims.value().front()] = newIndexVal;
  }

```
- **EN**: Implements logic around `generateCollapsedIndexingRegion`, `g`, `setInsertionPointToStart`, `to_vector`, and 8 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `generateCollapsedIndexingRegion`, `g`, `setInsertionPointToStart`, `to_vector`, and 8 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1729-1747
```cpp
  for (auto indexOp : indexOps) {
    auto dim = indexOp.getDim();
    rewriter.replaceOp(indexOp, indexReplacementVals[dim]);
  }
}

static void collapseOperandsAndResults(LinalgOp op,
                                       const CollapsingInfo &collapsingInfo,
                                       RewriterBase &rewriter,
                                       SmallVectorImpl<Value> &inputOperands,
                                       SmallVectorImpl<Value> &outputOperands,
                                       SmallVectorImpl<Type> &resultTypes) {
  Location loc = op->getLoc();
  inputOperands =
      llvm::map_to_vector(op.getDpsInputOperands(), [&](OpOperand *opOperand) {
        return getCollapsedOpOperand(loc, op, opOperand, collapsingInfo,
                                     rewriter);
      });

```
- **EN**: Implements logic around `getDim`, `replaceOp`, `collapseOperandsAndResults`, `getLoc`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getDim`, `replaceOp`, `collapseOperandsAndResults`, `getLoc`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 1748-1768
```cpp
  // Get the output operands and result types.
  resultTypes.reserve(op.getNumDpsInits());
  outputOperands.reserve(op.getNumDpsInits());
  for (OpOperand &output : op.getDpsInitsMutable()) {
    Value newOutput =
        getCollapsedOpOperand(loc, op, &output, collapsingInfo, rewriter);
    outputOperands.push_back(newOutput);
    // If the op has "buffer semantics", then the init operands are ranked
    // memrefs and the op has no results.
    if (!op.hasPureBufferSemantics())
      resultTypes.push_back(newOutput.getType());
  }
}

/// Clone a `LinalgOp` to a collapsed version of same name
template <typename OpTy>
static OpTy cloneToCollapsedOp(RewriterBase &rewriter, OpTy origOp,
                               const CollapsingInfo &collapsingInfo) {
  return nullptr;
}

```
- **EN**: Implements logic around `reserve`, `getDpsInitsMutable`, `getCollapsedOpOperand`, `push_back`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `reserve`, `getDpsInitsMutable`, `getCollapsedOpOperand`, `push_back`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 1769-1797
```cpp
/// Collapse any `LinalgOp` that does not require any specialization such as
/// indexing_maps, iterator_types, etc.
template <>
LinalgOp cloneToCollapsedOp<LinalgOp>(RewriterBase &rewriter, LinalgOp origOp,
                                      const CollapsingInfo &collapsingInfo) {
  SmallVector<Value> inputOperands, outputOperands;
  SmallVector<Type> resultTypes;
  collapseOperandsAndResults(origOp, collapsingInfo, rewriter, inputOperands,
                             outputOperands, resultTypes);

  return clone(
      rewriter, origOp, resultTypes,
      llvm::to_vector(llvm::concat<Value>(inputOperands, outputOperands)));
}

/// Collapse a `GenericOp`
template <>
GenericOp cloneToCollapsedOp<GenericOp>(RewriterBase &rewriter,
                                        GenericOp origOp,
                                        const CollapsingInfo &collapsingInfo) {
  SmallVector<Value> inputOperands, outputOperands;
  SmallVector<Type> resultTypes;
  collapseOperandsAndResults(origOp, collapsingInfo, rewriter, inputOperands,
                             outputOperands, resultTypes);
  SmallVector<AffineMap> indexingMaps(
      llvm::map_range(origOp.getIndexingMapsArray(), [&](AffineMap map) {
        return getCollapsedOpIndexingMap(map, collapsingInfo);
      }));

```
- **EN**: Implements logic around `cloneToCollapsedOp`, `collapseOperandsAndResults`, `clone`, `to_vector`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `cloneToCollapsedOp`, `collapseOperandsAndResults`, `clone`, `to_vector`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 1798-1820
```cpp
  SmallVector<utils::IteratorType> iteratorTypes(getCollapsedOpIteratorTypes(
      origOp.getIteratorTypesArray(), collapsingInfo));

  GenericOp collapsedOp = linalg::GenericOp::create(
      rewriter, origOp.getLoc(), resultTypes, inputOperands, outputOperands,
      indexingMaps, iteratorTypes,
      [](OpBuilder &builder, Location loc, ValueRange args) {});
  Block *origOpBlock = &origOp->getRegion(0).front();
  Block *collapsedOpBlock = &collapsedOp->getRegion(0).front();
  rewriter.mergeBlocks(origOpBlock, collapsedOpBlock,
                       collapsedOpBlock->getArguments());
  return collapsedOp;
}

static LinalgOp createCollapsedOp(LinalgOp op,
                                  const CollapsingInfo &collapsingInfo,
                                  RewriterBase &rewriter) {
  if (GenericOp genericOp = dyn_cast<GenericOp>(op.getOperation())) {
    return cloneToCollapsedOp(rewriter, genericOp, collapsingInfo);
  }
  return cloneToCollapsedOp(rewriter, op, collapsingInfo);
}

```
- **EN**: Implements logic around `iteratorTypes`, `getIteratorTypesArray`, `create`, `getLoc`, and 6 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `iteratorTypes`, `getIteratorTypesArray`, `create`, `getLoc`, and 6 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1821-1838
```cpp
/// Implementation of fusion with reshape operation by collapsing dimensions.
FailureOr<CollapseResult> mlir::linalg::collapseOpIterationDims(
    LinalgOp op, ArrayRef<ReassociationIndices> foldedIterationDims,
    RewriterBase &rewriter) {
  // Bail on trivial no-op cases.
  if (op.getNumLoops() <= 1 || foldedIterationDims.empty() ||
      llvm::all_of(foldedIterationDims, [](ReassociationIndicesRef foldedDims) {
        return foldedDims.size() <= 1;
      }))
    return failure();

  CollapsingInfo collapsingInfo;
  if (failed(
          collapsingInfo.initialize(op.getNumLoops(), foldedIterationDims))) {
    return rewriter.notifyMatchFailure(
        op, "illegal to collapse specified dimensions");
  }

```
- **EN**: Implements logic around `collapseOpIterationDims`, `getNumLoops`, `all_of`, `size`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `collapseOpIterationDims`, `getNumLoops`, `all_of`, `size`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1839-1872
```cpp
  bool hasPureBufferSemantics = op.hasPureBufferSemantics();
  if (hasPureBufferSemantics &&
      !llvm::all_of(op->getOpOperands(), [&](OpOperand &opOperand) -> bool {
        MemRefType memRefToCollapse =
            dyn_cast<MemRefType>(opOperand.get().getType());
        if (!memRefToCollapse)
          return true;

        AffineMap indexingMap = op.getMatchingIndexingMap(&opOperand);
        SmallVector<ReassociationIndices> operandReassociation =
            getOperandReassociation(indexingMap, collapsingInfo);
        return memref::CollapseShapeOp::isGuaranteedCollapsible(
            memRefToCollapse, operandReassociation);
      }))
    return rewriter.notifyMatchFailure(op,
                                       "memref is not guaranteed collapsible");

  // Bail on non-canonical ranges.
  SmallVector<Range> loopRanges = op.createLoopRanges(rewriter, op.getLoc());
  auto opFoldIsConstantValue = [](OpFoldResult ofr, int64_t value) {
    if (auto attr = llvm::dyn_cast_if_present<Attribute>(ofr))
      return cast<IntegerAttr>(attr).getInt() == value;
    llvm::APInt actual;
    return matchPattern(cast<Value>(ofr), m_ConstantInt(&actual)) &&
           actual.getSExtValue() == value;
  };
  if (!llvm::all_of(loopRanges, [&](Range range) {
        return opFoldIsConstantValue(range.offset, 0) &&
               opFoldIsConstantValue(range.stride, 1);
      })) {
    return rewriter.notifyMatchFailure(
        op, "expected all loop ranges to have zero start and unit stride");
  }

```
- **EN**: Implements logic around `hasPureBufferSemantics`, `all_of`, `get`, `getMatchingIndexingMap`, and 9 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `hasPureBufferSemantics`, `all_of`, `get`, `getMatchingIndexingMap`, and 9 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1873-1908
```cpp
  LinalgOp collapsedOp = createCollapsedOp(op, collapsingInfo, rewriter);

  Location loc = op->getLoc();
  SmallVector<OpFoldResult> loopBound =
      llvm::map_to_vector(loopRanges, [](Range range) { return range.size; });

  if (collapsedOp.hasIndexSemantics()) {
    // Collect the loop range of the generic op.
    OpBuilder::InsertionGuard g(rewriter);
    rewriter.setInsertionPoint(collapsedOp);
    generateCollapsedIndexingRegion(loc, &collapsedOp->getRegion(0).front(),
                                    collapsingInfo, loopBound, rewriter);
  }

  // Insert expanding reshape for the result to get back the original result
  // type.
  SmallVector<Value> results;
  for (const auto &originalResult : llvm::enumerate(op->getResults())) {
    Value collapsedOpResult = collapsedOp->getResult(originalResult.index());
    auto originalResultType =
        cast<ShapedType>(originalResult.value().getType());
    auto collapsedOpResultType = cast<ShapedType>(collapsedOpResult.getType());
    if (collapsedOpResultType.getRank() != originalResultType.getRank()) {
      AffineMap indexingMap =
          op.getIndexingMapMatchingResult(originalResult.value());
      SmallVector<ReassociationIndices> reassociation =
          getOperandReassociation(indexingMap, collapsingInfo);
      assert(
          indexingMap.isProjectedPermutation() &&
          "Expected indexing map to be a projected permutation for collapsing");
      SmallVector<OpFoldResult> resultShape =
          applyPermutationMap(indexingMap, ArrayRef(loopBound));
      Value result;
      if (isa<MemRefType>(collapsedOpResult.getType())) {
        result = memref::ExpandShapeOp::create(
            rewriter, loc, originalResultType, collapsedOpResult, reassociation,
```
- **EN**: Implements logic around `createCollapsedOp`, `getLoc`, `map_to_vector`, `hasIndexSemantics`, and 14 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `createCollapsedOp`, `getLoc`, `map_to_vector`, `hasIndexSemantics`, and 14 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1909-1926
```cpp
            resultShape);
      } else {
        result = tensor::ExpandShapeOp::create(
            rewriter, loc, originalResultType, collapsedOpResult, reassociation,
            resultShape);
      }
      results.push_back(result);
    } else {
      results.push_back(collapsedOpResult);
    }
  }
  return CollapseResult{results, collapsedOp};
}

namespace {

/// Pattern to fuse a tensor.expand_shape op with its consumer generic op by
/// contracting dimensions of the loop.
```
- **EN**: Implements logic around `create`, `push_back`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `push_back` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1927-1944
```cpp
class FoldWithProducerReshapeOpByCollapsing
    : public OpRewritePattern<GenericOp> {
public:
  // TODO : support fusion with all linalg ops, not just generic.
  FoldWithProducerReshapeOpByCollapsing(MLIRContext *context,
                                        ControlFusionFn foldReshapes,
                                        PatternBenefit benefit = 1)
      : OpRewritePattern<GenericOp>(context, benefit),
        controlFoldingReshapes(std::move(foldReshapes)) {}

  LogicalResult matchAndRewrite(GenericOp genericOp,
                                PatternRewriter &rewriter) const override {
    for (OpOperand &opOperand : genericOp->getOpOperands()) {
      tensor::ExpandShapeOp reshapeOp =
          opOperand.get().getDefiningOp<tensor::ExpandShapeOp>();
      if (!reshapeOp)
        continue;

```
- **EN**: Introduces declarations for `FoldWithProducerReshapeOpByCollapsing`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `FoldWithProducerReshapeOpByCollapsing` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 1945-1965
```cpp
      SmallVector<ReassociationIndices> collapsableIterationDims =
          getCollapsableIterationSpaceDims(genericOp, &opOperand,
                                           reshapeOp.getReassociationIndices());
      if (collapsableIterationDims.empty() ||
          !controlFoldingReshapes(&opOperand)) {
        continue;
      }

      std::optional<CollapseResult> collapseResult = collapseOpIterationDims(
          genericOp, collapsableIterationDims, rewriter);
      if (!collapseResult) {
        return rewriter.notifyMatchFailure(
            genericOp, "failed to do the fusion by collapsing transformation");
      }

      rewriter.replaceOp(genericOp, collapseResult->results);
      return success();
    }
    return failure();
  }

```
- **EN**: Implements logic around `getCollapsableIterationSpaceDims`, `getReassociationIndices`, `empty`, `controlFoldingReshapes`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getCollapsableIterationSpaceDims`, `getReassociationIndices`, `empty`, `controlFoldingReshapes`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 1966-1990
```cpp
private:
  ControlFusionFn controlFoldingReshapes;
};

/// Pattern to fold a tensor.collapse_shape op with its producer generic op
/// by expanding the dimensionality of the loop in the producer op.
struct FoldReshapeWithGenericOpByCollapsing
    : public OpRewritePattern<tensor::CollapseShapeOp> {

  FoldReshapeWithGenericOpByCollapsing(MLIRContext *context,
                                       ControlFusionFn foldReshapes,
                                       PatternBenefit benefit = 1)
      : OpRewritePattern<tensor::CollapseShapeOp>(context, benefit),
        controlFoldingReshapes(std::move(foldReshapes)) {}

  LogicalResult matchAndRewrite(tensor::CollapseShapeOp reshapeOp,
                                PatternRewriter &rewriter) const override {
    // Fold only if all constraints of fusing with reshape by collapsing are
    // met.
    auto producerResult = dyn_cast<OpResult>(reshapeOp.getSrc());
    if (!producerResult) {
      return rewriter.notifyMatchFailure(reshapeOp,
                                         "source not produced by an operation");
    }

```
- **EN**: Introduces declarations for `FoldReshapeWithGenericOpByCollapsing`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `FoldReshapeWithGenericOpByCollapsing` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 1991-2012
```cpp
    // TODO : support fusion with all linalg producers, not just generic.
    auto producer = dyn_cast<GenericOp>(producerResult.getOwner());
    if (!producer) {
      return rewriter.notifyMatchFailure(reshapeOp,
                                         "producer not a generic op");
    }

    SmallVector<ReassociationIndices> collapsableIterationDims =
        getCollapsableIterationSpaceDims(
            producer,
            producer.getDpsInitOperand(producerResult.getResultNumber()),
            reshapeOp.getReassociationIndices());
    if (collapsableIterationDims.empty()) {
      return rewriter.notifyMatchFailure(
          reshapeOp, "failed preconditions of fusion with producer generic op");
    }

    if (!controlFoldingReshapes(&reshapeOp.getSrcMutable())) {
      return rewriter.notifyMatchFailure(reshapeOp,
                                         "fusion blocked by control function");
    }

```
- **EN**: Implements logic around `getOwner`, `notifyMatchFailure`, `getCollapsableIterationSpaceDims`, `getDpsInitOperand`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getOwner`, `notifyMatchFailure`, `getCollapsableIterationSpaceDims`, `getDpsInitOperand`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 2013-2030
```cpp
    // Set the insertion point after `producer` because there could be uses
    // of `producer` between it and the `tensor.collapse_shape` op.
    rewriter.setInsertionPointAfter(producer);
    std::optional<CollapseResult> collapseResult =
        collapseOpIterationDims(producer, collapsableIterationDims, rewriter);
    if (!collapseResult) {
      return rewriter.notifyMatchFailure(
          producer, "failed to do the fusion by collapsing transformation");
    }

    rewriter.replaceOp(producer, collapseResult->results);
    return success();
  }

private:
  ControlFusionFn controlFoldingReshapes;
};

```
- **EN**: Implements logic around `setInsertionPointAfter`, `collapseOpIterationDims`, `notifyMatchFailure`, `replaceOp`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `setInsertionPointAfter`, `collapseOpIterationDims`, `notifyMatchFailure`, `replaceOp`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 2031-2060
```cpp
/// Computes the collapsed padding information for the given pad operation based
/// on the provided collapsed shape and reassociation indices. Returns a
/// PadDimInfo containing the low and high padding amounts and the collapsed
/// shape for each dimension, or failure if the collapse is not possible.
static FailureOr<PadDimInfo>
computeCollapsedPadding(tensor::PadOp padOp,
                        ArrayRef<ReassociationIndices> reassociations,
                        PatternRewriter &rewriter) {
  // If the padding value depends on the index values of the pad operation,
  // then it may not be valid to collapse the dimensions, since it will change
  // the index values on which the padding value depends. This is not currently
  // supported by the pad collapsing patterns, but it could be implemented
  // similarly to the collapsing of linalg.generic ops with linalg.index ops in
  // the body, as is done in `generateCollapsedIndexingRegion`.
  if (!padOp.getConstantPaddingValue())
    return failure();

  // Collapsed dimensions cannot have padding because this can produce strided
  // padding that isn't representable by a tensor.pad op. There are some special
  // cases where it is possible (like collapsing unit dims), but supporting
  // these cases is NYI, so disallow it for now.
  ArrayRef<int64_t> low = padOp.getStaticLow();
  ArrayRef<int64_t> high = padOp.getStaticHigh();
  for (auto [idx, reInd] : llvm::enumerate(reassociations)) {
    for (int64_t dim : reInd) {
      if ((low[dim] != 0 || high[dim] != 0) && reInd.size() != 1)
        return failure();
    }
  }

```
- **EN**: Implements logic around `computeCollapsedPadding`, `getConstantPaddingValue`, `failure`, `getStaticLow`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `computeCollapsedPadding`, `getConstantPaddingValue`, `failure`, `getStaticLow`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 2061-2083
```cpp
  // Initialize padding values for collapsed tensors with zeros
  ArrayRef<int64_t> expandedPaddedShape = padOp.getType().getShape();
  PadDimInfo padDimInfo;
  padDimInfo.lowPad.assign(reassociations.size(), rewriter.getIndexAttr(0));
  padDimInfo.highPad.assign(reassociations.size(), rewriter.getIndexAttr(0));

  // Update padding for dimensions that are not being collapsed, and compute
  // the collapsed padded shape.
  SmallVector<OpFoldResult> mixedLowPad(padOp.getMixedLowPad());
  SmallVector<OpFoldResult> mixedHighPad(padOp.getMixedHighPad());
  for (auto [idx, reInd] : llvm::enumerate(reassociations)) {
    if (reInd.size() == 1) {
      padDimInfo.lowPad[idx] = mixedLowPad[reInd[0]];
      padDimInfo.highPad[idx] = mixedHighPad[reInd[0]];
    }
    SaturatedInteger collapsedSize = SaturatedInteger::wrap(1);
    for (int64_t dim : reInd) {
      collapsedSize =
          collapsedSize * SaturatedInteger::wrap(expandedPaddedShape[dim]);
    }
    padDimInfo.paddedShape.push_back(collapsedSize.asInteger());
  }

```
- **EN**: Implements logic around `getType`, `assign`, `mixedLowPad`, `mixedHighPad`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getType`, `assign`, `mixedLowPad`, `mixedHighPad`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 2084-2102
```cpp
  return padDimInfo;
}

class FoldPadWithProducerReshapeOpByCollapsing
    : public OpRewritePattern<tensor::PadOp> {
public:
  FoldPadWithProducerReshapeOpByCollapsing(MLIRContext *context,
                                           ControlFusionFn foldReshapes,
                                           PatternBenefit benefit = 1)
      : OpRewritePattern<tensor::PadOp>(context, benefit),
        controlFoldingReshapes(std::move(foldReshapes)) {}

  LogicalResult matchAndRewrite(tensor::PadOp padOp,
                                PatternRewriter &rewriter) const override {
    tensor::ExpandShapeOp reshapeOp =
        padOp.getSource().getDefiningOp<tensor::ExpandShapeOp>();
    if (!reshapeOp)
      return failure();

```
- **EN**: Introduces declarations for `FoldPadWithProducerReshapeOpByCollapsing`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `FoldPadWithProducerReshapeOpByCollapsing` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 2103-2130
```cpp
    if (!controlFoldingReshapes(&padOp.getSourceMutable())) {
      return rewriter.notifyMatchFailure(padOp,
                                         "fusion blocked by control function");
    }

    SmallVector<ReassociationIndices> reassociations =
        reshapeOp.getReassociationIndices();
    FailureOr<PadDimInfo> maybeCollapsedPadding =
        computeCollapsedPadding(padOp, reassociations, rewriter);
    if (failed(maybeCollapsedPadding))
      return failure();
    PadDimInfo &collapsedPadding = maybeCollapsedPadding.value();

    SmallVector<OpFoldResult> expandedPaddedSizes =
        reshapeOp.getMixedOutputShape();
    AffineExpr d0, d1, d2;
    bindDims(rewriter.getContext(), d0, d1, d2);
    auto addMap = AffineMap::get(3, 0, {d0 + d1 + d2});
    Location loc = reshapeOp->getLoc();
    for (auto [reInd, l, h] :
         llvm::zip_equal(reassociations, collapsedPadding.lowPad,
                         collapsedPadding.highPad)) {
      if (reInd.size() == 1) {
        expandedPaddedSizes[reInd[0]] = affine::makeComposedFoldedAffineApply(
            rewriter, loc, addMap, {l, h, expandedPaddedSizes[reInd[0]]});
      }
    }

```
- **EN**: Implements logic around `controlFoldingReshapes`, `notifyMatchFailure`, `getReassociationIndices`, `computeCollapsedPadding`, and 10 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `controlFoldingReshapes`, `notifyMatchFailure`, `getReassociationIndices`, `computeCollapsedPadding`, and 10 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 2131-2148
```cpp
    RankedTensorType collapsedPaddedType =
        padOp.getType().clone(collapsedPadding.paddedShape);
    auto newPadOp = tensor::PadOp::create(
        rewriter, loc, collapsedPaddedType, reshapeOp.getSrc(),
        collapsedPadding.lowPad, collapsedPadding.highPad,
        padOp.getConstantPaddingValue(), padOp.getNofold());

    rewriter.replaceOpWithNewOp<tensor::ExpandShapeOp>(
        padOp, padOp.getResultType(), newPadOp.getResult(), reassociations,
        expandedPaddedSizes);

    return success();
  }

private:
  ControlFusionFn controlFoldingReshapes;
};

```
- **EN**: Implements logic around `getType`, `create`, `getSrc`, `getConstantPaddingValue`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getType`, `create`, `getSrc`, `getConstantPaddingValue`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 2149-2168
```cpp
class FoldReshapeWithProducerPadOpByCollapsing
    : public OpRewritePattern<tensor::CollapseShapeOp> {
public:
  FoldReshapeWithProducerPadOpByCollapsing(MLIRContext *context,
                                           ControlFusionFn foldReshapes,
                                           PatternBenefit benefit = 1)
      : OpRewritePattern<tensor::CollapseShapeOp>(context, benefit),
        controlFoldingReshapes(std::move(foldReshapes)) {}

  LogicalResult matchAndRewrite(tensor::CollapseShapeOp reshapeOp,
                                PatternRewriter &rewriter) const override {
    tensor::PadOp padOp = reshapeOp.getSrc().getDefiningOp<tensor::PadOp>();
    if (!padOp)
      return failure();

    if (!controlFoldingReshapes(&reshapeOp.getSrcMutable())) {
      return rewriter.notifyMatchFailure(padOp,
                                         "fusion blocked by control function");
    }

```
- **EN**: Introduces declarations for `FoldReshapeWithProducerPadOpByCollapsing`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `FoldReshapeWithProducerPadOpByCollapsing` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 2169-2186
```cpp
    SmallVector<ReassociationIndices> reassociations =
        reshapeOp.getReassociationIndices();
    RankedTensorType collapsedPaddedType = reshapeOp.getResultType();
    FailureOr<PadDimInfo> maybeCollapsedPadding =
        computeCollapsedPadding(padOp, reassociations, rewriter);
    if (failed(maybeCollapsedPadding))
      return failure();
    PadDimInfo &collapsedPadding = maybeCollapsedPadding.value();

    Location loc = reshapeOp->getLoc();
    auto newCollapseOp = tensor::CollapseShapeOp::create(
        rewriter, loc, padOp.getSource(), reassociations);

    auto newPadOp = tensor::PadOp::create(
        rewriter, loc, collapsedPaddedType, newCollapseOp.getResult(),
        collapsedPadding.lowPad, collapsedPadding.highPad,
        padOp.getConstantPaddingValue(), padOp.getNofold());

```
- **EN**: Implements logic around `getReassociationIndices`, `getResultType`, `computeCollapsedPadding`, `failed`, and 7 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getReassociationIndices`, `getResultType`, `computeCollapsedPadding`, `failed`, and 7 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 2187-2204
```cpp
    rewriter.replaceOp(reshapeOp, newPadOp.getResult());
    return success();
  }

private:
  ControlFusionFn controlFoldingReshapes;
};

/// Pattern to collapse dimensions.
template <typename LinalgType>
class CollapseLinalgDimensions : public OpRewritePattern<LinalgType> {
public:
  CollapseLinalgDimensions(MLIRContext *context,
                           GetCollapsableDimensionsFn collapseDimensions,
                           PatternBenefit benefit = 1)
      : OpRewritePattern<LinalgType>(context, benefit),
        controlCollapseDimension(std::move(collapseDimensions)) {}

```
- **EN**: Introduces declarations for `CollapseLinalgDimensions`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `CollapseLinalgDimensions` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 2205-2227
```cpp
  LogicalResult matchAndRewrite(LinalgType op,
                                PatternRewriter &rewriter) const override {
    SmallVector<ReassociationIndices> collapsableIterationDims =
        controlCollapseDimension(op);
    if (collapsableIterationDims.empty())
      return failure();

    // Check if the specified list of dimensions to collapse is a valid list.
    if (!areDimSequencesPreserved(op.getIndexingMapsArray(),
                                  collapsableIterationDims)) {
      return rewriter.notifyMatchFailure(
          op, "specified dimensions cannot be collapsed");
    }

    std::optional<CollapseResult> collapseResult =
        collapseOpIterationDims(op, collapsableIterationDims, rewriter);
    if (!collapseResult) {
      return rewriter.notifyMatchFailure(op, "failed to collapse dimensions");
    }
    rewriter.replaceOp(op, collapseResult->results);
    return success();
  }

```
- **EN**: Implements logic around `matchAndRewrite`, `controlCollapseDimension`, `empty`, `failure`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `matchAndRewrite`, `controlCollapseDimension`, `empty`, `failure`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 2228-2245
```cpp
private:
  GetCollapsableDimensionsFn controlCollapseDimension;
};

} // namespace

//===---------------------------------------------------------------------===//
// Methods and patterns that fuse constants with linalg.generic operations.
//===---------------------------------------------------------------------===//

namespace {
/// Pattern to fold a generic op with a splat constant/scalar constant. Does not
/// handle cases where the constant is not single-valued.
class FoldScalarOrSplatConstant : public OpRewritePattern<GenericOp> {
public:
  FoldScalarOrSplatConstant(MLIRContext *context, PatternBenefit benefit = 1)
      : OpRewritePattern<GenericOp>(context, benefit) {}

```
- **EN**: Introduces declarations for `FoldScalarOrSplatConstant`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `FoldScalarOrSplatConstant` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 2246-2279
```cpp
  LogicalResult matchAndRewrite(GenericOp genericOp,
                                PatternRewriter &rewriter) const override {
    if (!genericOp.hasPureTensorSemantics())
      return failure();
    for (OpOperand *opOperand : genericOp.getDpsInputOperands()) {
      Operation *def = opOperand->get().getDefiningOp();
      TypedAttr constantAttr;
      auto isScalarOrSplatConstantOp = [&constantAttr](Operation *def) -> bool {
        {
          DenseElementsAttr splatAttr;
          if (matchPattern(def, m_Constant<DenseElementsAttr>(&splatAttr)) &&
              splatAttr.isSplat() &&
              splatAttr.getType().getElementType().isIntOrFloat()) {
            constantAttr = splatAttr.getSplatValue<TypedAttr>();
            return true;
          }
        }
        {
          IntegerAttr intAttr;
          if (matchPattern(def, m_Constant<IntegerAttr>(&intAttr))) {
            constantAttr = intAttr;
            return true;
          }
        }
        {
          FloatAttr floatAttr;
          if (matchPattern(def, m_Constant<FloatAttr>(&floatAttr))) {
            constantAttr = floatAttr;
            return true;
          }
        }
        return false;
      };

```
- **EN**: Implements logic around `matchAndRewrite`, `hasPureTensorSemantics`, `failure`, `getDpsInputOperands`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `matchAndRewrite`, `hasPureTensorSemantics`, `failure`, `getDpsInputOperands`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 2280-2305
```cpp
      auto resultValue = dyn_cast<OpResult>(opOperand->get());
      if (!def || !resultValue || !isScalarOrSplatConstantOp(def))
        continue;

      // The operands and the indexing_maps of the fused operation the same as
      // the operands and indexing_maps of the generic operations with the
      // values at the constant index dropped.
      SmallVector<AffineMap> fusedIndexMaps;
      SmallVector<Value> fusedOperands;
      SmallVector<Location> fusedLocs{genericOp.getLoc()};
      fusedIndexMaps.reserve(genericOp->getNumOperands());
      fusedOperands.reserve(genericOp.getNumDpsInputs());
      fusedLocs.reserve(fusedLocs.size() + genericOp.getNumDpsInputs());
      for (OpOperand *inputOperand : genericOp.getDpsInputOperands()) {
        if (inputOperand == opOperand)
          continue;
        Value inputValue = inputOperand->get();
        fusedIndexMaps.push_back(
            genericOp.getMatchingIndexingMap(inputOperand));
        fusedOperands.push_back(inputValue);
        fusedLocs.push_back(inputValue.getLoc());
      }
      for (OpOperand &outputOperand : genericOp.getDpsInitsMutable())
        fusedIndexMaps.push_back(
            genericOp.getMatchingIndexingMap(&outputOperand));

```
- **EN**: Implements logic around `get`, `isScalarOrSplatConstantOp`, `getLoc`, `reserve`, and 4 more symbols.
- **CN**: 围绕 `get`, `isScalarOrSplatConstantOp`, `getLoc`, `reserve`, and 4 more symbols 实现具体逻辑。

### Lines 2306-2327
```cpp
      // Check if the operation shapes to loops map is computable.
      if (!inversePermutation(
              concatAffineMaps(fusedIndexMaps, rewriter.getContext()))) {
        return rewriter.notifyMatchFailure(
            genericOp, "fused op loop bound computation failed");
      }

      // Create a constant scalar value from the splat constant.
      Value scalarConstant =
          arith::ConstantOp::create(rewriter, def->getLoc(), constantAttr);

      SmallVector<Value> outputOperands = genericOp.getOutputs();
      auto fusedOp =
          GenericOp::create(rewriter, rewriter.getFusedLoc(fusedLocs),
                            genericOp->getResultTypes(),
                            /*inputs=*/fusedOperands,
                            /*outputs=*/outputOperands,
                            rewriter.getAffineMapArrayAttr(fusedIndexMaps),
                            genericOp.getIteratorTypes(),
                            /*doc=*/nullptr,
                            /*library_call=*/nullptr);

```
- **EN**: Implements logic around `inversePermutation`, `concatAffineMaps`, `notifyMatchFailure`, `create`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `inversePermutation`, `concatAffineMaps`, `notifyMatchFailure`, `create`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 2328-2346
```cpp
      // Map the block argument corresponding to the replaced argument with the
      // scalar constant.
      Region &region = genericOp->getRegion(0);
      Block &entryBlock = *region.begin();
      IRMapping mapping;
      mapping.map(entryBlock.getArgument(opOperand->getOperandNumber()),
                  scalarConstant);
      Region &fusedRegion = fusedOp->getRegion(0);
      rewriter.cloneRegionBefore(region, fusedRegion, fusedRegion.begin(),
                                 mapping);
      rewriter.replaceOp(genericOp, fusedOp->getResults());
      return success();
    }
    return failure();
  }
};

} // namespace

```
- **EN**: Implements logic around `getRegion`, `begin`, `map`, `cloneRegionBefore`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getRegion`, `begin`, `map`, `cloneRegionBefore`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 2347-2370
```cpp
//===---------------------------------------------------------------------===//
// Miscellaneous patterns that help fusion.
//===---------------------------------------------------------------------===//

namespace {
/// Forces `outs` operands of linalg operations to use `tensor.empty` if the
/// value of the `outs` operand is not used within the op.  This is only
/// implemented for `linalg.generic` operations for now, but should hold for all
/// linalg structured ops.
struct RemoveOutsDependency : public OpRewritePattern<GenericOp> {
  using OpRewritePattern<GenericOp>::OpRewritePattern;

  LogicalResult matchAndRewrite(GenericOp op,
                                PatternRewriter &rewriter) const override {
    rewriter.startOpModification(op);
    bool modifiedOutput = false;
    Location loc = op.getLoc();
    for (OpOperand &opOperand : op.getDpsInitsMutable()) {
      if (!op.payloadUsesValueFromOperand(&opOperand)) {
        Value operandVal = opOperand.get();
        auto operandType = dyn_cast<RankedTensorType>(operandVal.getType());
        if (!operandType)
          continue;

```
- **EN**: Introduces declarations for `RemoveOutsDependency`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `RemoveOutsDependency` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 2371-2395
```cpp
        // If outs is sparse, leave it to the sparsifier.
        if (sparse_tensor::getSparseTensorEncoding(operandVal.getType()))
          continue;

        // If outs is already an `empty` operation, nothing to do.
        auto definingOp = operandVal.getDefiningOp<tensor::EmptyOp>();
        if (definingOp)
          continue;
        modifiedOutput = true;
        SmallVector<OpFoldResult> mixedSizes =
            tensor::getMixedSizes(rewriter, loc, operandVal);
        Value emptyTensor = tensor::EmptyOp::create(
            rewriter, loc, mixedSizes, operandType.getElementType());
        op->setOperand(opOperand.getOperandNumber(), emptyTensor);
      }
    }
    if (!modifiedOutput) {
      rewriter.cancelOpModification(op);
      return failure();
    }
    rewriter.finalizeOpModification(op);
    return success();
  }
};

```
- **EN**: Implements logic around `getSparseTensorEncoding`, `EmptyOp>`, `getMixedSizes`, `create`, and 6 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getSparseTensorEncoding`, `EmptyOp>`, `getMixedSizes`, `create`, and 6 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 2396-2426
```cpp
/// Fold linalg.fill into linalg.generic
struct FoldFillWithGenericOp : public OpRewritePattern<GenericOp> {
  using OpRewritePattern<GenericOp>::OpRewritePattern;

  LogicalResult matchAndRewrite(GenericOp genericOp,
                                PatternRewriter &rewriter) const override {
    if (!genericOp.hasPureTensorSemantics())
      return failure();
    bool fillFound = false;
    Block &payload = genericOp.getRegion().front();
    for (OpOperand *opOperand : genericOp.getDpsInputOperands()) {
      if (!genericOp.payloadUsesValueFromOperand(opOperand))
        continue;
      FillOp fillOp = opOperand->get().getDefiningOp<FillOp>();
      if (!fillOp)
        continue;
      fillFound = true;
      Value fillVal = fillOp.value();
      auto resultType =
          cast<RankedTensorType>(fillOp.result().getType()).getElementType();
      Value convertedVal =
          convertScalarToDtype(rewriter, fillOp.getLoc(), fillVal, resultType,
                               /*isUnsignedCast =*/false);
      rewriter.replaceAllUsesWith(
          payload.getArgument(opOperand->getOperandNumber()), convertedVal);
    }
    return success(fillFound);
  }
};
} // namespace

```
- **EN**: Introduces declarations for `FoldFillWithGenericOp`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `FoldFillWithGenericOp` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 2427-2452
```cpp
void mlir::linalg::populateFoldReshapeOpsByExpansionPatterns(
    RewritePatternSet &patterns,
    const ControlFusionFn &controlFoldingReshapes) {
  patterns.add<FoldReshapeWithGenericOpByExpansion>(patterns.getContext(),
                                                    controlFoldingReshapes);
  patterns.add<FoldPadWithProducerReshapeOpByExpansion>(patterns.getContext(),
                                                        controlFoldingReshapes);
  patterns.add<FoldReshapeWithProducerPadOpByExpansion>(patterns.getContext(),
                                                        controlFoldingReshapes);
  patterns.add<FoldWithProducerReshapeOpByExpansion>(patterns.getContext(),
                                                     controlFoldingReshapes);
}

void mlir::linalg::populateFoldReshapeOpsByCollapsingPatterns(
    RewritePatternSet &patterns,
    const ControlFusionFn &controlFoldingReshapes) {
  patterns.add<FoldWithProducerReshapeOpByCollapsing>(patterns.getContext(),
                                                      controlFoldingReshapes);
  patterns.add<FoldPadWithProducerReshapeOpByCollapsing>(
      patterns.getContext(), controlFoldingReshapes);
  patterns.add<FoldReshapeWithProducerPadOpByCollapsing>(
      patterns.getContext(), controlFoldingReshapes);
  patterns.add<FoldReshapeWithGenericOpByCollapsing>(patterns.getContext(),
                                                     controlFoldingReshapes);
}

```
- **EN**: Implements logic around `populateFoldReshapeOpsByExpansionPatterns`, `add`, `populateFoldReshapeOpsByCollapsingPatterns`, `getContext`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `populateFoldReshapeOpsByExpansionPatterns`, `add`, `populateFoldReshapeOpsByCollapsingPatterns`, `getContext` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 2453-2471
```cpp
void mlir::linalg::populateElementwiseOpsFusionPatterns(
    RewritePatternSet &patterns,
    const ControlFusionFn &controlElementwiseOpsFusion) {
  auto *context = patterns.getContext();
  patterns.add<FuseElementwiseOps>(context, controlElementwiseOpsFusion);
  patterns.add<FoldFillWithGenericOp, FoldScalarOrSplatConstant,
               RemoveOutsDependency>(context);
  // Add the patterns that clean up dead operands and results.
  populateEraseUnusedOperandsAndResultsPatterns(patterns);
}

void mlir::linalg::populateCollapseDimensions(
    RewritePatternSet &patterns,
    const GetCollapsableDimensionsFn &controlCollapseDimensions) {
  patterns.add<CollapseLinalgDimensions<linalg::GenericOp>,
               CollapseLinalgDimensions<linalg::CopyOp>>(
      patterns.getContext(), controlCollapseDimensions);
}

```
- **EN**: Implements logic around `populateElementwiseOpsFusionPatterns`, `getContext`, `add`, `RemoveOutsDependency>`, and 3 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `populateElementwiseOpsFusionPatterns`, `getContext`, `add`, `RemoveOutsDependency>`, and 3 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 2472-2493
```cpp
//===---------------------------------------------------------------------===//
// Passes
//===---------------------------------------------------------------------===//

namespace {

/// Pass that fuses generic ops on tensors. Used only for testing.
// TODO(ravishankarm): This pass is to be deprecated. The efficacy of the
// patterns added here heavily depends on the cost function used. Having an
// opinionated pass of this form is not recommended. Deprecate this pass in
// favor of test passes that check the functionality of each of the patterns
// added here individually.
struct LinalgElementwiseOpFusionPass
    : public impl::LinalgElementwiseOpFusionPassBase<
          LinalgElementwiseOpFusionPass> {
  using impl::LinalgElementwiseOpFusionPassBase<
      LinalgElementwiseOpFusionPass>::LinalgElementwiseOpFusionPassBase;
  void runOnOperation() override {
    Operation *op = getOperation();
    MLIRContext *context = op->getContext();
    RewritePatternSet patterns(context);

```
- **EN**: Introduces declarations for `LinalgElementwiseOpFusionPass`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `LinalgElementwiseOpFusionPass` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 2494-2512
```cpp
    // Add folding with reshape by expansion patterns.
    ControlFusionFn defaultControlFn = [](OpOperand *fusedOperand) {
      Operation *producer = fusedOperand->get().getDefiningOp();
      return producer && producer->hasOneUse();
    };

    // Add elementwise op fusion patterns.
    populateElementwiseOpsFusionPatterns(patterns, defaultControlFn);
    populateFoldReshapeOpsByExpansionPatterns(patterns, defaultControlFn);
    tensor::populateBubbleUpExpandShapePatterns(patterns);

    // General canonicalization patterns.
    affine::AffineApplyOp::getCanonicalizationPatterns(patterns, context);
    GenericOp::getCanonicalizationPatterns(patterns, context);
    tensor::ExpandShapeOp::getCanonicalizationPatterns(patterns, context);
    tensor::CollapseShapeOp::getCanonicalizationPatterns(patterns, context);
    context->getLoadedDialect<LinalgDialect>()->getCanonicalizationPatterns(
        patterns);

```
- **EN**: Implements logic around `get`, `hasOneUse`, `populateElementwiseOpsFusionPatterns`, `populateFoldReshapeOpsByExpansionPatterns`, and 3 more symbols; this block performs canonicalization or folding-oriented reasoning; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `get`, `hasOneUse`, `populateElementwiseOpsFusionPatterns`, `populateFoldReshapeOpsByExpansionPatterns`, and 3 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理，并协调核心结构化 MLIR 方言之间的行为。

### Lines 2513-2522
```cpp
    // Add constant folding patterns.
    populateConstantFoldLinalgOperations(patterns, defaultControlFn);

    // Use TopDownTraversal for compile time reasons.
    (void)applyPatternsGreedily(op, std::move(patterns),
                                GreedyRewriteConfig().setUseTopDownTraversal());
  }
};

} // namespace
```
- **EN**: Implements logic around `populateConstantFoldLinalgOperations`, `applyPatternsGreedily`, `GreedyRewriteConfig`.
- **CN**: 围绕 `populateConstantFoldLinalgOperations`, `applyPatternsGreedily`, `GreedyRewriteConfig` 实现具体逻辑。

## Key Concepts / 关键概念

- **Rewrite and lowering / 重写与 lowering**:
  - **EN**: Uses MLIR pattern infrastructure or passes to canonicalize, legalize, or lower operations.
  - **CN**: 使用 MLIR 模式基础设施或 Pass 对操作进行规范化、合法化或 lowering。
- **Pattern rewriting / 模式重写**:
  - **EN**: Uses MLIR rewrite patterns to match operations and replace them with improved forms.
  - **CN**: 使用 MLIR 重写模式匹配操作，并将其替换为更合适的形式。
- **Pass infrastructure / Pass 基础设施**:
  - **EN**: Wraps transformations as reusable passes that can be scheduled in MLIR pipelines.
  - **CN**: 将变换封装为可在 MLIR 流水线中调度的可复用 Pass。
- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Models reusable capabilities that can be queried across dialect boundaries.
  - **CN**: 建模可跨方言查询的可复用能力。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Linalg/Passes.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Arith/Utils/Utils.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/Dialect/SparseTensor/IR/SparseTensor.h`, `mlir/Dialect/Tensor/Transforms/Transforms.h`, `mlir/IR/AffineExpr.h`, `mlir/IR/AffineMap.h` ... (+7 more)
- **Standard-library headers / 标准库头文件**: `<optional>`, `<utility>`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (9), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (4), generic transformation and conversion helpers / 通用变换与转换辅助逻辑 (2), support utilities and LLVM interop helpers / 支持工具与 LLVM 互操作辅助逻辑 (1), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (1)
