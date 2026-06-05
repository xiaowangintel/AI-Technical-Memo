# ShardingInterfaceImpl.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Linalg/Transforms/ShardingInterfaceImpl.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements Linalg dialect support for rewrite patterns, passes, and IR-to-IR transformation logic, centered on `ShardingInterfaceImpl`.
  - **CN**: 实现 Linalg 方言中围绕 `ShardingInterfaceImpl` 的重写模式、Pass 以及 IR 到 IR 的变换逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10
```cpp
//===- ShardingInterfaceImpl.cpp --------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Linalg/Transforms/ShardingInterfaceImpl.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/Transforms/ShardingInterfaceImpl.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/Transforms/ShardingInterfaceImpl.h`。

### Lines 11-30
```cpp
#include "mlir/Analysis/SliceAnalysis.h"
#include "mlir/Dialect/Affine/IR/AffineOps.h"
#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/Linalg/IR/Linalg.h"
#include "mlir/Dialect/Linalg/IR/LinalgInterfaces.h"
#include "mlir/Dialect/SCF/IR/SCF.h"
#include "mlir/Dialect/Shard/IR/ShardOps.h"
#include "mlir/Dialect/Shard/Interfaces/ShardingInterface.h"
#include "mlir/Dialect/Shard/Interfaces/ShardingInterfaceImpl.h"
#include "mlir/Dialect/Shard/Transforms/Transforms.h"
#include "mlir/Dialect/Tensor/IR/Tensor.h"
#include "mlir/Dialect/Utils/StructuredOpsUtils.h"
#include "mlir/IR/AffineExpr.h"
#include "mlir/IR/DialectRegistry.h"
#include "mlir/IR/IRMapping.h"
#include "mlir/IR/MLIRContext.h"
#include "mlir/IR/OpDefinition.h"
#include "mlir/IR/Operation.h"
#include "mlir/IR/SymbolTable.h"
#include "mlir/IR/Value.h"
```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Analysis/SliceAnalysis.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Linalg/IR/Linalg.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Analysis/SliceAnalysis.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Linalg/IR/Linalg.h`。

### Lines 31-44
```cpp
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/TypeSwitch.h"
#include <numeric>
#include <optional>

namespace mlir::linalg {

using GridAxis = shard::GridAxis;
using ReductionKind = shard::ReductionKind;
using Sharding = shard::Sharding;
using ShardingArray = shard::ShardingArray;
using GridOp = shard::GridOp;

```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/TypeSwitch.h`, `numeric`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/TypeSwitch.h`, `numeric`。

### Lines 45-64
```cpp
// Returns the corresponding grid reduction kind for the given arith op.
static ReductionKind getReductionKind(Operation *op) {
  return llvm::TypeSwitch<Operation *, ReductionKind>(op)
      // Floating-point operations.
      .Case([](arith::AddFOp op) { return ReductionKind::Sum; })
      .Case([](arith::MulFOp op) { return ReductionKind::Product; })
      // TODO: handle maxnumf and minnumf.
      .Case([](arith::MaximumFOp op) { return ReductionKind::Max; })
      .Case([](arith::MinimumFOp op) { return ReductionKind::Min; })
      // Integer operations.
      .Case([](arith::AddIOp op) { return ReductionKind::Sum; })
      .Case([](arith::OrIOp op) { return ReductionKind::BitwiseOr; })
      .Case([](arith::XOrIOp op) { return ReductionKind::BitwiseXor; })
      .Case([](arith::AndIOp op) { return ReductionKind::BitwiseAnd; })
      // TODO: handle signless, signed and unsigned types properly.
      // It is assumed that the element type of the collective operands and
      // result drive the meaning of the reduction kind, whether it is signed
      // or unsigned.
      // The reduction op inside the linalg op may have different result type
      // from the element type of the linalg op's result.
```
- **EN**: Implements logic around `getReductionKind`, `ReductionKind>`, `Case`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getReductionKind`, `ReductionKind>`, `Case` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 65-75
```cpp
      // Also signed and unsigned Arith dialect ops may accept signed, unsigned
      // or signless operands.
      // Maybe expand the reduction kinds.
      .Case([](arith::MaxUIOp op) { return ReductionKind::Max; })
      .Case([](arith::MinUIOp op) { return ReductionKind::Min; })
      .Case([](arith::MaxSIOp op) { return ReductionKind::Max; })
      .Case([](arith::MinSIOp op) { return ReductionKind::Min; })
      .Case([](arith::MulIOp op) { return ReductionKind::Product; })
      .Default([](Operation *op) { return ReductionKind::Generic; });
}

```
- **EN**: Implements logic around `Case`, `Default`.
- **CN**: 围绕 `Case`, `Default` 实现具体逻辑。

### Lines 76-85
```cpp
static std::optional<Operation *> getCombinerOp(LinalgOp op) {
  SmallVector<Operation *> combinerOps;
  Value reducedValue = matchReduction(op.getRegionOutputArgs(), 0, combinerOps);
  if (!reducedValue || combinerOps.size() != 1) {
    return std::nullopt;
  }

  return combinerOps[0];
}

```
- **EN**: Implements logic around `getCombinerOp`, `matchReduction`, `size`.
- **CN**: 围绕 `getCombinerOp`, `matchReduction`, `size` 实现具体逻辑。

### Lines 86-99
```cpp
static ReductionKind getReductionKindOfLinalgOp(LinalgOp op) {
  std::optional<Operation *> reductionOp = getCombinerOp(op);
  if (!reductionOp) {
    return ReductionKind::Generic;
  }
  [[maybe_unused]] Type resultElementType =
      llvm::cast<RankedTensorType>(op->getResult(0).getType()).getElementType();
  // TODO: handle case when result type of the reduction op does not match the
  // element type of the result tensor.
  // Would it makes sense at all?
  assert(resultElementType == reductionOp.value()->getResult(0).getType());
  return getReductionKind(reductionOp.value());
}

```
- **EN**: Implements logic around `getReductionKindOfLinalgOp`, `getCombinerOp`, `getResult`, `assert`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getReductionKindOfLinalgOp`, `getCombinerOp`, `getResult`, `assert`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 100-114
```cpp
static GridOp getGrid(Operation *op, ArrayRef<Sharding> operandShardings,
                      ArrayRef<Sharding> resultShardings,
                      SymbolTableCollection &symbolTable) {
  for (const Sharding &sharding : operandShardings) {
    if (sharding) {
      return shard::getGrid(op, sharding.getGridAttr(), symbolTable);
    }
  }

  for (const Sharding &sharding : resultShardings) {
    if (sharding) {
      return shard::getGrid(op, sharding.getGridAttr(), symbolTable);
    }
  }

```
- **EN**: Implements logic around `getGrid`.
- **CN**: 围绕 `getGrid` 实现具体逻辑。

### Lines 115-134
```cpp
  assert(false);
  return nullptr;
}

// Choose the operand based on the current process index along the reduction
// grid axes.
// We need to use the initial value only once to avoid including it in the
// reduction multiple times.
// In each process group only the leading process with linear index 0 would use
// the original operand.
// The other processes would use the reduction operation neutral tensor.
static Value createDestinationPassingStyleInitOperand(
    LinalgOp op, int operandNumber, Value partitionedOperand,
    ArrayRef<GridAxis> reductionGridAxes, GridOp gridOp,
    ImplicitLocOpBuilder &builder) {
  Value processLinearIndexInReductionGroup = shard::createProcessLinearIndex(
      builder, gridOp.getSymName(), reductionGridAxes);
  Value zero = arith::ConstantIndexOp::create(builder, 0);
  Value isLeadProcess = arith::CmpIOp::create(
      builder, builder.getI1Type(), arith::CmpIPredicate::eq,
```
- **EN**: Implements logic around `assert`, `createDestinationPassingStyleInitOperand`, `createProcessLinearIndex`, `getSymName`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `assert`, `createDestinationPassingStyleInitOperand`, `createProcessLinearIndex`, `getSymName`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 135-144
```cpp
      processLinearIndexInReductionGroup, zero);
  scf::IfOp ifOp = scf::IfOp::create(builder, partitionedOperand.getType(),
                                     isLeadProcess, true, true);
  // Then block.
  {
    OpBuilder::InsertionGuard insertionGuard(builder);
    builder.setInsertionPointToEnd(&ifOp.getThenRegion().front());
    scf::YieldOp::create(builder, partitionedOperand);
  }

```
- **EN**: Implements logic around `create`, `insertionGuard`, `setInsertionPointToEnd`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `insertionGuard`, `setInsertionPointToEnd` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 145-157
```cpp
  // Else block.
  {
    OpBuilder::InsertionGuard insertionGuard(builder);
    builder.setInsertionPointToEnd(&ifOp.getElseRegion().front());
    SmallVector<OpFoldResult> shape =
        tensor::getMixedSizes(builder, builder.getLoc(), partitionedOperand);

    SmallVector<Operation *> combinerOps;
    matchReduction(op.getRegionOutputArgs(), operandNumber, combinerOps);
    assert(combinerOps.size() == 1);
    std::optional<TypedAttr> neutralEl =
        arith::getNeutralElement(combinerOps[0]);

```
- **EN**: Implements logic around `insertionGuard`, `setInsertionPointToEnd`, `getMixedSizes`, `matchReduction`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `insertionGuard`, `setInsertionPointToEnd`, `getMixedSizes`, `matchReduction`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 158-169
```cpp
    Value init = tensor::EmptyOp::create(builder, op.getLoc(), shape,
                                         neutralEl.value().getType());
    Value constant =
        arith::ConstantOp::create(builder, op.getLoc(), neutralEl.value());
    Value fill = linalg::FillOp::create(builder, op.getLoc(), constant, init)
                     .getResult(0);

    scf::YieldOp::create(builder, fill);
  }
  return ifOp.getResult(0);
}

```
- **EN**: Implements logic around `create`, `value`, `getResult`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `value`, `getResult` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 170-187
```cpp
// Create the DPS init operands for the partitioned Linalg op.
// Return all the new partitioned operands.
static SmallVector<Value> createDestinationPassingStyleInitOperands(
    LinalgOp op, GridOp gridOp, ArrayRef<Value> partitionedOperands,
    ArrayRef<GridAxis> reductionGridAxes, IRMapping &partitionMap,
    ImplicitLocOpBuilder &builder) {
  // TODO: add support for multiple destination passing style initial value
  // operands.
  assert(op.getNumDpsInits() == 1 && "Multiple initial values not supported.");
  SmallVector<Value> newOperands = llvm::to_vector(partitionedOperands);
  auto operandIdx = op.getDpsInitOperand(0)->getOperandNumber();
  Value partitionedInitOperand =
      partitionMap.lookup(op->getOperands()[operandIdx]);
  newOperands[operandIdx] = createDestinationPassingStyleInitOperand(
      op, 0, partitionedInitOperand, reductionGridAxes, gridOp, builder);
  return newOperands;
}

```
- **EN**: Implements logic around `createDestinationPassingStyleInitOperands`, `assert`, `to_vector`, `getDpsInitOperand`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `createDestinationPassingStyleInitOperands`, `assert`, `to_vector`, `getDpsInitOperand`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 188-203
```cpp
static void createAllReduceForResultsWithoutPartialShardings(
    LinalgOp unshardedOp, ArrayRef<GridAxis> opReductionGridAxes,
    ArrayRef<Sharding> resultShardings, IRMapping &partitionMap,
    ImplicitLocOpBuilder &builder) {
  ReductionKind reductionKind = getReductionKindOfLinalgOp(unshardedOp);
  for (auto [unshardedLinalgOpResult, resultSharding] :
       llvm::zip_equal(unshardedOp->getResults(), resultShardings)) {
    Value partitionedLinalgOpResult =
        partitionMap.lookup(unshardedLinalgOpResult);
    Value reducedValue = shard::AllReduceOp::create(
        builder, partitionedLinalgOpResult, resultSharding.getGrid(),
        opReductionGridAxes, reductionKind);
    partitionMap.map(unshardedLinalgOpResult, reducedValue);
  }
}

```
- **EN**: Implements logic around `createAllReduceForResultsWithoutPartialShardings`, `getReductionKindOfLinalgOp`, `zip_equal`, `lookup`, and 3 more symbols.
- **CN**: 围绕 `createAllReduceForResultsWithoutPartialShardings`, `getReductionKindOfLinalgOp`, `zip_equal`, `lookup`, and 3 more symbols 实现具体逻辑。

### Lines 204-223
```cpp
static void partitionLinalgOpWithShardedReduction(
    LinalgOp op, ArrayRef<Value> partitionedOperands,
    ArrayRef<Sharding> operandShardings, ArrayRef<Sharding> resultShardings,
    ArrayRef<utils::IteratorType> loopIteratorTypes,
    ArrayRef<SmallVector<GridAxis>> gridAxisAssignmentForLoopIterators,
    IRMapping &partitionMap, SymbolTableCollection &symbolTable,
    ImplicitLocOpBuilder &builder) {
  GridOp grid = getGrid(op, operandShardings, resultShardings, symbolTable);
  SmallVector<GridAxis> reductionGridAxes = shard::getReductionGridAxes(
      loopIteratorTypes, gridAxisAssignmentForLoopIterators);
  SmallVector<Value> partitionedLinalgOpOperands =
      createDestinationPassingStyleInitOperands(op, grid, partitionedOperands,
                                                reductionGridAxes, partitionMap,
                                                builder);
  // We must not change the operand mappings of the original partitionMap as
  // they are the mappings for the whole partition blob and may be used by
  // others.
  IRMapping internalPartitionMap;
  for (auto [unshardedOperand, partitionedOperand] :
       llvm::zip_equal(op->getOperands(), partitionedLinalgOpOperands)) {
```
- **EN**: Implements logic around `partitionLinalgOpWithShardedReduction`, `getGrid`, `getReductionGridAxes`, `createDestinationPassingStyleInitOperands`, and 1 more symbols.
- **CN**: 围绕 `partitionLinalgOpWithShardedReduction`, `getGrid`, `getReductionGridAxes`, `createDestinationPassingStyleInitOperands`, and 1 more symbols 实现具体逻辑。

### Lines 224-237
```cpp
    internalPartitionMap.map(unshardedOperand, partitionedOperand);
  }
  partitionTriviallyShardableOperation(
      *op, partitionedLinalgOpOperands, operandShardings, resultShardings,
      internalPartitionMap, symbolTable, builder);
  for (Value result : op->getResults()) {
    partitionMap.map(result, internalPartitionMap.lookup(result));
  }

  // Handle partial shardings.
  createAllReduceForResultsWithoutPartialShardings(
      op, reductionGridAxes, resultShardings, partitionMap, builder);
}

```
- **EN**: Implements logic around `map`, `partitionTriviallyShardableOperation`, `getResults`, `createAllReduceForResultsWithoutPartialShardings`.
- **CN**: 围绕 `map`, `partitionTriviallyShardableOperation`, `getResults`, `createAllReduceForResultsWithoutPartialShardings` 实现具体逻辑。

### Lines 238-250
```cpp
namespace {

// ShardingInterface for ops that implement LinalgStructuredInterface.
// The supported ops are only those where the indexing maps are projected
// permutations.
template <typename Op>
struct StructuredOpShardingInterface
    : public shard::ShardingInterface::ExternalModel<
          StructuredOpShardingInterface<Op>, Op> {
  SmallVector<utils::IteratorType> getLoopIteratorTypes(Operation *op) const {
    return llvm::cast<LinalgOp>(op).getIteratorTypesArray();
  }

```
- **EN**: Introduces declarations for `StructuredOpShardingInterface`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `StructuredOpShardingInterface` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 251-260
```cpp
  SmallVector<AffineMap> getIndexingMaps(Operation *op) const {
    LinalgOp linalgOp = llvm::cast<LinalgOp>(op);
    SmallVector<AffineMap> res = linalgOp.getIndexingMapsArray();

    // Results must have the same indexing as destination passing style initial
    // operands.
    for (int64_t i = 0; i < linalgOp.getNumDpsInits(); ++i) {
      res.push_back(res[linalgOp.getDpsInitOperand(i)->getOperandNumber()]);
    }

```
- **EN**: Implements logic around `getIndexingMaps`, `getIndexingMapsArray`, `getNumDpsInits`, `push_back`.
- **CN**: 围绕 `getIndexingMaps`, `getIndexingMapsArray`, `getNumDpsInits`, `push_back` 实现具体逻辑。

### Lines 261-276
```cpp
    return res;
  }

  SmallVector<ReductionKind>
  getReductionLoopIteratorKinds(Operation *op) const {
    LinalgOp linalgOp = llvm::cast<LinalgOp>(op);
    SmallVector<utils::IteratorType> iteratorTypes =
        linalgOp.getIteratorTypesArray();
    unsigned reductionItersCount = llvm::accumulate(
        iteratorTypes, 0u, [](unsigned count, utils::IteratorType iter) {
          return count + (iter == utils::IteratorType::reduction);
        });
    shard::ReductionKind reductionKind = getReductionKindOfLinalgOp(linalgOp);
    return SmallVector<ReductionKind>(reductionItersCount, reductionKind);
  }

```
- **EN**: Implements logic around `getReductionLoopIteratorKinds`, `getIteratorTypesArray`, `accumulate`, `getReductionKindOfLinalgOp`, and 1 more symbols.
- **CN**: 围绕 `getReductionLoopIteratorKinds`, `getIteratorTypesArray`, `accumulate`, `getReductionKindOfLinalgOp`, and 1 more symbols 实现具体逻辑。

### Lines 277-295
```cpp
  LogicalResult partition(Operation *op, ArrayRef<Value> partitionedOperands,
                          ArrayRef<Sharding> operandShardings,
                          ArrayRef<Sharding> resultShardings,
                          IRMapping &partitionMap,
                          SymbolTableCollection &symbolTable,
                          OpBuilder &builder) const {
    LinalgOp linalgOp = llvm::cast<LinalgOp>(op);

    SmallVector<AffineMap> indexingMaps = linalgOp.getIndexingMapsArray();
    bool allIndexingMapsAreProjectedPermutation =
        llvm::all_of(indexingMaps, [](AffineMap map) {
          return map.isProjectedPermutation();
        });
    if (!allIndexingMapsAreProjectedPermutation) {
      // TODO: handle non-projected permutations.
      return op->emitOpError()
             << "supports indexing maps that are only projected permutation.";
    }

```
- **EN**: Implements logic around `partition`, `getIndexingMapsArray`, `all_of`, `isProjectedPermutation`, and 1 more symbols; this block checks operation invariants or verification rules.
- **CN**: 围绕 `partition`, `getIndexingMapsArray`, `all_of`, `isProjectedPermutation`, and 1 more symbols 实现具体逻辑；该代码块检查操作不变式或验证规则。

### Lines 296-313
```cpp
    SmallVector<utils::IteratorType> loopIteratorTypes =
        linalgOp.getIteratorTypesArray();
    ShardingArray gridAxisAssignmentForLoopIterators =
        getGridAxisAssignmentForLoopIterators(operandShardings, resultShardings,
                                              loopIteratorTypes, indexingMaps);
    if (shard::isAtLeastOneReductionIteratorSharded(
            loopIteratorTypes, gridAxisAssignmentForLoopIterators)) {
      ImplicitLocOpBuilder implicitLocBuilder(op->getLoc(), builder);
      partitionLinalgOpWithShardedReduction(
          linalgOp, partitionedOperands, operandShardings, resultShardings,
          loopIteratorTypes, gridAxisAssignmentForLoopIterators, partitionMap,
          symbolTable, implicitLocBuilder);
    } else {
      partitionTriviallyShardableOperation(*op, partitionedOperands,
                                           operandShardings, resultShardings,
                                           partitionMap, symbolTable, builder);
    }

```
- **EN**: Implements logic around `getIteratorTypesArray`, `getGridAxisAssignmentForLoopIterators`, `isAtLeastOneReductionIteratorSharded`, `implicitLocBuilder`, and 2 more symbols.
- **CN**: 围绕 `getIteratorTypesArray`, `getGridAxisAssignmentForLoopIterators`, `isAtLeastOneReductionIteratorSharded`, `implicitLocBuilder`, and 2 more symbols 实现具体逻辑。

### Lines 314-324
```cpp
    return success();
  }
};

} // namespace

template <typename OpType>
static void registerOne(MLIRContext *ctx) {
  OpType::template attachInterface<StructuredOpShardingInterface<OpType>>(*ctx);
}

```
- **EN**: Implements logic around `success`, `registerOne`, `attachInterface`.
- **CN**: 围绕 `success`, `registerOne`, `attachInterface` 实现具体逻辑。

### Lines 325-339
```cpp
/// Variadic helper function.
template <typename... OpTypes>
static void registerAll(MLIRContext *ctx) {
  (registerOne<OpTypes>(ctx), ...);
}

void registerShardingInterfaceExternalModels(DialectRegistry &registry) {
  registry.addExtension(+[](MLIRContext *ctx, LinalgDialect *dialect) {
    DialectRegistry registry;
    registry.insert<affine::AffineDialect, arith::ArithDialect, scf::SCFDialect,
                    tensor::TensorDialect>();
    ctx->appendDialectRegistry(registry);
    for (StringRef name : registry.getRegisteredDialectNames())
      ctx->getOrLoadDialect(name);

```
- **EN**: Implements logic around `registerAll`, `registerOne`, `registerShardingInterfaceExternalModels`, `addExtension`, and 4 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `registerAll`, `registerOne`, `registerShardingInterfaceExternalModels`, `addExtension`, and 4 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 340-348
```cpp
    registerOne<linalg::GenericOp>(ctx);
    registerAll<
#define GET_OP_LIST
#include "mlir/Dialect/Linalg/IR/LinalgStructuredOps.cpp.inc"
        >(ctx);
  });
}

} // namespace mlir::linalg
```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/IR/LinalgStructuredOps.cpp.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/IR/LinalgStructuredOps.cpp.inc`。

## Key Concepts / 关键概念

- **Rewrite and lowering / 重写与 lowering**:
  - **EN**: Uses MLIR pattern infrastructure or passes to canonicalize, legalize, or lower operations.
  - **CN**: 使用 MLIR 模式基础设施或 Pass 对操作进行规范化、合法化或 lowering。
- **Pass infrastructure / Pass 基础设施**:
  - **EN**: Wraps transformations as reusable passes that can be scheduled in MLIR pipelines.
  - **CN**: 将变换封装为可在 MLIR 流水线中调度的可复用 Pass。
- **Operation verification / 操作验证**:
  - **EN**: Checks structural and semantic invariants so malformed IR is rejected early.
  - **CN**: 检查结构与语义不变式，以便尽早拒绝非法 IR。
- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Models reusable capabilities that can be queried across dialect boundaries.
  - **CN**: 建模可跨方言查询的可复用能力。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Linalg/Transforms/ShardingInterfaceImpl.h`, `mlir/Analysis/SliceAnalysis.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/IR/LinalgInterfaces.h`, `mlir/Dialect/SCF/IR/SCF.h`, `mlir/Dialect/Shard/IR/ShardOps.h`, `mlir/Dialect/Shard/Interfaces/ShardingInterface.h`, `mlir/Dialect/Shard/Interfaces/ShardingInterfaceImpl.h` ... (+15 more)
- **Standard-library headers / 标准库头文件**: `<numeric>`, `<optional>`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (13), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (8), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (3)
- **Generated macros / 生成宏**: `GET_OP_LIST`
