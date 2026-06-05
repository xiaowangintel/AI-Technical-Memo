# SCFTransformOps.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/SCF/TransformOps/SCFTransformOps.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements SCF dialect support for transform dialect operations and higher-level orchestration helpers, centered on `SCFTransformOps`.
  - **CN**: 实现 SCF 方言中围绕 `SCFTransformOps` 的Transform Dialect 操作及更高层编排辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-26
```cpp
//===- SCFTransformOps.cpp - Implementation of SCF transformation ops -----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/SCF/TransformOps/SCFTransformOps.h"

#include "mlir/Conversion/SCFToControlFlow/SCFToControlFlow.h"
#include "mlir/Dialect/Affine/IR/AffineOps.h"
#include "mlir/Dialect/Affine/LoopUtils.h"
#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/Dialect/SCF/IR/SCF.h"
#include "mlir/Dialect/SCF/Transforms/Patterns.h"
#include "mlir/Dialect/SCF/Transforms/Transforms.h"
#include "mlir/Dialect/SCF/Utils/Utils.h"
#include "mlir/Dialect/Transform/IR/TransformDialect.h"
#include "mlir/Dialect/Transform/Interfaces/TransformInterfaces.h"
#include "mlir/Dialect/Utils/StaticValueUtils.h"
#include "mlir/Dialect/Vector/IR/VectorOps.h"
#include "mlir/IR/BuiltinAttributes.h"
#include "mlir/IR/Dominance.h"
#include "mlir/IR/OpDefinition.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/SCF/TransformOps/SCFTransformOps.h`, `mlir/Conversion/SCFToControlFlow/SCFToControlFlow.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/LoopUtils.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/SCF/TransformOps/SCFTransformOps.h`, `mlir/Conversion/SCFToControlFlow/SCFToControlFlow.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/LoopUtils.h`。

### Lines 27-43
```cpp
using namespace mlir;
using namespace mlir::affine;

//===----------------------------------------------------------------------===//
// Apply...PatternsOp
//===----------------------------------------------------------------------===//

void transform::ApplyForLoopCanonicalizationPatternsOp::populatePatterns(
    RewritePatternSet &patterns) {
  scf::populateSCFForLoopCanonicalizationPatterns(patterns);
}

void transform::ApplySCFStructuralConversionPatternsOp::populatePatterns(
    TypeConverter &typeConverter, RewritePatternSet &patterns) {
  scf::populateSCFStructuralTypeConversions(typeConverter, patterns);
}

```
- **EN**: Introduces declarations for `mlir`, `mlir::affine`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `mlir`, `mlir::affine` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 44-58
```cpp
void transform::ApplySCFStructuralConversionPatternsOp::
    populateConversionTargetRules(const TypeConverter &typeConverter,
                                  ConversionTarget &conversionTarget) {
  scf::populateSCFStructuralTypeConversionTarget(typeConverter,
                                                 conversionTarget);
}

void transform::ApplySCFToControlFlowPatternsOp::populatePatterns(
    TypeConverter &typeConverter, RewritePatternSet &patterns) {
  populateSCFToControlFlowConversionPatterns(patterns);
}

//===----------------------------------------------------------------------===//
// ForallToForOp
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `populateConversionTargetRules`, `populateSCFStructuralTypeConversionTarget`, `populatePatterns`, `populateSCFToControlFlowConversionPatterns`; this block coordinates dialect conversion or lowering decisions; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `populateConversionTargetRules`, `populateSCFStructuralTypeConversionTarget`, `populatePatterns`, `populateSCFToControlFlowConversionPatterns` 实现具体逻辑；该代码块协调方言转换或 lowering 决策，并协调核心结构化 MLIR 方言之间的行为。

### Lines 59-75
```cpp

DiagnosedSilenceableFailure
transform::ForallToForOp::apply(transform::TransformRewriter &rewriter,
                                transform::TransformResults &results,
                                transform::TransformState &state) {
  auto payload = state.getPayloadOps(getTarget());
  if (!llvm::hasSingleElement(payload))
    return emitSilenceableError() << "expected a single payload op";

  auto target = dyn_cast<scf::ForallOp>(*payload.begin());
  if (!target) {
    DiagnosedSilenceableFailure diag =
        emitSilenceableError() << "expected the payload to be scf.forall";
    diag.attachNote((*payload.begin())->getLoc()) << "payload op";
    return diag;
  }

```
- **EN**: Implements logic around `apply`, `getPayloadOps`, `hasSingleElement`, `emitSilenceableError`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `apply`, `getPayloadOps`, `hasSingleElement`, `emitSilenceableError`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 76-91
```cpp
  if (!target.getOutputs().empty()) {
    return emitSilenceableError()
           << "unsupported shared outputs (didn't bufferize?)";
  }

  SmallVector<OpFoldResult> lbs = target.getMixedLowerBound();

  if (getNumResults() != lbs.size()) {
    DiagnosedSilenceableFailure diag =
        emitSilenceableError()
        << "op expects as many results (" << getNumResults()
        << ") as payload has induction variables (" << lbs.size() << ")";
    diag.attachNote(target.getLoc()) << "payload op";
    return diag;
  }

```
- **EN**: Implements logic around `getOutputs`, `emitSilenceableError`, `outputs`, `getMixedLowerBound`, and 4 more symbols.
- **CN**: 围绕 `getOutputs`, `emitSilenceableError`, `outputs`, `getMixedLowerBound`, and 4 more symbols 实现具体逻辑。

### Lines 92-105
```cpp
  SmallVector<Operation *> opResults;
  if (failed(scf::forallToForLoop(rewriter, target, &opResults))) {
    DiagnosedSilenceableFailure diag = emitSilenceableError()
                                       << "failed to convert forall into for";
    return diag;
  }

  for (auto &&[i, res] : llvm::enumerate(opResults)) {
    results.set(cast<OpResult>(getTransformed()[i]), {res});
  }
  return DiagnosedSilenceableFailure::success();
}

//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `failed`, `emitSilenceableError`, `enumerate`, `set`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates dialect conversion or lowering decisions; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `failed`, `emitSilenceableError`, `enumerate`, `set`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调方言转换或 lowering 决策，并协调核心结构化 MLIR 方言之间的行为。

### Lines 106-124
```cpp
// ForallToForOp
//===----------------------------------------------------------------------===//

DiagnosedSilenceableFailure
transform::ForallToParallelOp::apply(transform::TransformRewriter &rewriter,
                                     transform::TransformResults &results,
                                     transform::TransformState &state) {
  auto payload = state.getPayloadOps(getTarget());
  if (!llvm::hasSingleElement(payload))
    return emitSilenceableError() << "expected a single payload op";

  auto target = dyn_cast<scf::ForallOp>(*payload.begin());
  if (!target) {
    DiagnosedSilenceableFailure diag =
        emitSilenceableError() << "expected the payload to be scf.forall";
    diag.attachNote((*payload.begin())->getLoc()) << "payload op";
    return diag;
  }

```
- **EN**: Implements logic around `apply`, `getPayloadOps`, `hasSingleElement`, `emitSilenceableError`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `apply`, `getPayloadOps`, `hasSingleElement`, `emitSilenceableError`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 125-144
```cpp
  if (!target.getOutputs().empty()) {
    return emitSilenceableError()
           << "unsupported shared outputs (didn't bufferize?)";
  }

  if (getNumResults() != 1) {
    DiagnosedSilenceableFailure diag = emitSilenceableError()
                                       << "op expects one result, given "
                                       << getNumResults();
    diag.attachNote(target.getLoc()) << "payload op";
    return diag;
  }

  scf::ParallelOp opResult;
  if (failed(scf::forallToParallelLoop(rewriter, target, &opResult))) {
    DiagnosedSilenceableFailure diag =
        emitSilenceableError() << "failed to convert forall into parallel";
    return diag;
  }

```
- **EN**: Implements logic around `getOutputs`, `emitSilenceableError`, `outputs`, `getNumResults`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates dialect conversion or lowering decisions; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getOutputs`, `emitSilenceableError`, `outputs`, `getNumResults`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调方言转换或 lowering 决策，并协调核心结构化 MLIR 方言之间的行为。

### Lines 145-159
```cpp
  results.set(cast<OpResult>(getTransformed()[0]), {opResult});
  return DiagnosedSilenceableFailure::success();
}

//===----------------------------------------------------------------------===//
// ParallelForToNestedForOps
//===----------------------------------------------------------------------===//

DiagnosedSilenceableFailure transform::ParallelForToNestedForOps::apply(
    transform::TransformRewriter &rewriter,
    transform::TransformResults &results, transform::TransformState &state) {
  auto payload = state.getPayloadOps(getTarget());
  if (!llvm::hasSingleElement(payload))
    return emitSilenceableError() << "expected a single payload op";

```
- **EN**: Implements logic around `set`, `success`, `apply`, `getPayloadOps`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `set`, `success`, `apply`, `getPayloadOps`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 160-175
```cpp
  auto target = dyn_cast<scf::ParallelOp>(*payload.begin());
  if (!target) {
    DiagnosedSilenceableFailure diag =
        emitSilenceableError() << "expected the payload to be scf.parallel";
    diag.attachNote((*payload.begin())->getLoc()) << "payload op";
    return diag;
  }

  if (getNumResults() != 1) {
    DiagnosedSilenceableFailure diag = emitSilenceableError()
                                       << "op expects one result, given "
                                       << getNumResults();
    diag.attachNote(target.getLoc()) << "payload op";
    return diag;
  }

```
- **EN**: Implements logic around `ParallelOp>`, `emitSilenceableError`, `attachNote`, `getNumResults`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `ParallelOp>`, `emitSilenceableError`, `attachNote`, `getNumResults` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 176-190
```cpp
  FailureOr<scf::LoopNest> loopNest =
      scf::parallelForToNestedFors(rewriter, target);
  if (failed(loopNest)) {
    DiagnosedSilenceableFailure diag =
        emitSilenceableError() << "failed to convert parallel into nested fors";
    return diag;
  }

  results.set(cast<OpResult>(getTransformed()[0]), {loopNest->loops.front()});
  return DiagnosedSilenceableFailure::success();
}

//===----------------------------------------------------------------------===//
// LoopOutlineOp
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `parallelForToNestedFors`, `failed`, `emitSilenceableError`, `set`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates dialect conversion or lowering decisions; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `parallelForToNestedFors`, `failed`, `emitSilenceableError`, `set`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调方言转换或 lowering 决策，并协调核心结构化 MLIR 方言之间的行为。

### Lines 191-216
```cpp

/// Wraps the given operation `op` into an `scf.execute_region` operation. Uses
/// the provided rewriter for all operations to remain compatible with the
/// rewriting infra, as opposed to just splicing the op in place.
static scf::ExecuteRegionOp wrapInExecuteRegion(RewriterBase &b,
                                                Operation *op) {
  if (op->getNumRegions() != 1)
    return nullptr;
  OpBuilder::InsertionGuard g(b);
  b.setInsertionPoint(op);
  scf::ExecuteRegionOp executeRegionOp =
      scf::ExecuteRegionOp::create(b, op->getLoc(), op->getResultTypes());
  {
    OpBuilder::InsertionGuard g(b);
    b.setInsertionPointToStart(&executeRegionOp.getRegion().emplaceBlock());
    Operation *clonedOp = b.cloneWithoutRegions(*op);
    Region &clonedRegion = clonedOp->getRegions().front();
    assert(clonedRegion.empty() && "expected empty region");
    b.inlineRegionBefore(op->getRegions().front(), clonedRegion,
                         clonedRegion.end());
    scf::YieldOp::create(b, op->getLoc(), clonedOp->getResults());
  }
  b.replaceOp(op, executeRegionOp.getResults());
  return executeRegionOp;
}

```
- **EN**: Implements logic around `wrapInExecuteRegion`, `getNumRegions`, `g`, `setInsertionPoint`, and 8 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `wrapInExecuteRegion`, `getNumRegions`, `g`, `setInsertionPoint`, and 8 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 217-237
```cpp
DiagnosedSilenceableFailure
transform::LoopOutlineOp::apply(transform::TransformRewriter &rewriter,
                                transform::TransformResults &results,
                                transform::TransformState &state) {
  SmallVector<Operation *> functions;
  SmallVector<Operation *> calls;
  DenseMap<Operation *, SymbolTable> symbolTables;
  for (Operation *target : state.getPayloadOps(getTarget())) {
    Location location = target->getLoc();
    Operation *symbolTableOp = SymbolTable::getNearestSymbolTable(target);
    scf::ExecuteRegionOp exec = wrapInExecuteRegion(rewriter, target);
    if (!exec) {
      DiagnosedSilenceableFailure diag = emitSilenceableError()
                                         << "failed to outline";
      diag.attachNote(target->getLoc()) << "target op";
      return diag;
    }
    func::CallOp call;
    FailureOr<func::FuncOp> outlined = outlineSingleBlockRegion(
        rewriter, location, exec.getRegion(), getFuncName(), &call);

```
- **EN**: Implements logic around `apply`, `getPayloadOps`, `getLoc`, `getNearestSymbolTable`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `apply`, `getPayloadOps`, `getLoc`, `getNearestSymbolTable`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 238-255
```cpp
    if (failed(outlined))
      return emitDefaultDefiniteFailure(target);

    if (symbolTableOp) {
      SymbolTable &symbolTable =
          symbolTables.try_emplace(symbolTableOp, symbolTableOp)
              .first->getSecond();
      symbolTable.insert(*outlined);
      call.setCalleeAttr(FlatSymbolRefAttr::get(*outlined));
    }
    functions.push_back(*outlined);
    calls.push_back(call);
  }
  results.set(cast<OpResult>(getFunction()), functions);
  results.set(cast<OpResult>(getCall()), calls);
  return DiagnosedSilenceableFailure::success();
}

```
- **EN**: Implements logic around `failed`, `emitDefaultDefiniteFailure`, `try_emplace`, `getSecond`, and 5 more symbols.
- **CN**: 围绕 `failed`, `emitDefaultDefiniteFailure`, `try_emplace`, `getSecond`, and 5 more symbols 实现具体逻辑。

### Lines 256-283
```cpp
//===----------------------------------------------------------------------===//
// LoopPeelOp
//===----------------------------------------------------------------------===//

DiagnosedSilenceableFailure
transform::LoopPeelOp::applyToOne(transform::TransformRewriter &rewriter,
                                  scf::ForOp target,
                                  transform::ApplyToEachResultList &results,
                                  transform::TransformState &state) {
  scf::ForOp result;
  if (getPeelFront()) {
    LogicalResult status =
        scf::peelForLoopFirstIteration(rewriter, target, result);
    if (failed(status)) {
      DiagnosedSilenceableFailure diag =
          emitSilenceableError() << "failed to peel the first iteration";
      return diag;
    }
  } else {
    LogicalResult status =
        scf::peelForLoopAndSimplifyBounds(rewriter, target, result);
    if (failed(status)) {
      DiagnosedSilenceableFailure diag = emitSilenceableError()
                                         << "failed to peel the last iteration";
      return diag;
    }
  }

```
- **EN**: Implements logic around `applyToOne`, `getPeelFront`, `peelForLoopFirstIteration`, `failed`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `applyToOne`, `getPeelFront`, `peelForLoopFirstIteration`, `failed`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 284-306
```cpp
  results.push_back(target);
  results.push_back(result);

  return DiagnosedSilenceableFailure::success();
}

//===----------------------------------------------------------------------===//
// LoopPipelineOp
//===----------------------------------------------------------------------===//

/// Callback for PipeliningOption. Populates `schedule` with the mapping from an
/// operation to its logical time position given the iteration interval and the
/// read latency. The latter is only relevant for vector transfers.
static void
loopScheduling(scf::ForOp forOp,
               std::vector<std::pair<Operation *, unsigned>> &schedule,
               unsigned iterationInterval, unsigned readLatency) {
  auto getLatency = [&](Operation *op) -> unsigned {
    if (isa<vector::TransferReadOp>(op))
      return readLatency;
    return 1;
  };

```
- **EN**: Implements logic around `push_back`, `success`, `loopScheduling`, `TransferReadOp>`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `push_back`, `success`, `loopScheduling`, `TransferReadOp>` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 307-334
```cpp
  std::optional<int64_t> ubConstant =
      getConstantIntValue(forOp.getUpperBound());
  std::optional<int64_t> lbConstant =
      getConstantIntValue(forOp.getLowerBound());
  DenseMap<Operation *, unsigned> opCycles;
  std::map<unsigned, std::vector<Operation *>> wrappedSchedule;
  for (Operation &op : forOp.getBody()->getOperations()) {
    if (isa<scf::YieldOp>(op))
      continue;
    unsigned earlyCycle = 0;
    for (Value operand : op.getOperands()) {
      Operation *def = operand.getDefiningOp();
      if (!def)
        continue;
      if (ubConstant && lbConstant) {
        unsigned ubInt = ubConstant.value();
        unsigned lbInt = lbConstant.value();
        auto minLatency = std::min(ubInt - lbInt - 1, getLatency(def));
        earlyCycle = std::max(earlyCycle, opCycles[def] + minLatency);
      } else {
        earlyCycle = std::max(earlyCycle, opCycles[def] + getLatency(def));
      }
    }
    opCycles[&op] = earlyCycle;
    wrappedSchedule[earlyCycle % iterationInterval].push_back(&op);
  }
  for (const auto &it : wrappedSchedule) {
    for (Operation *op : it.second) {
```
- **EN**: Implements logic around `getConstantIntValue`, `getBody`, `YieldOp>`, `getOperands`, and 5 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getConstantIntValue`, `getBody`, `YieldOp>`, `getOperands`, and 5 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 335-362
```cpp
      unsigned cycle = opCycles[op];
      schedule.emplace_back(op, cycle / iterationInterval);
    }
  }
}

DiagnosedSilenceableFailure
transform::LoopPipelineOp::applyToOne(transform::TransformRewriter &rewriter,
                                      scf::ForOp target,
                                      transform::ApplyToEachResultList &results,
                                      transform::TransformState &state) {
  scf::PipeliningOption options;
  options.getScheduleFn =
      [this](scf::ForOp forOp,
             std::vector<std::pair<Operation *, unsigned>> &schedule) mutable {
        loopScheduling(forOp, schedule, getIterationInterval(),
                       getReadLatency());
      };
  scf::ForLoopPipeliningPattern pattern(options, target->getContext());
  rewriter.setInsertionPoint(target);
  FailureOr<scf::ForOp> patternResult =
      scf::pipelineForLoop(rewriter, target, options);
  if (succeeded(patternResult)) {
    results.push_back(*patternResult);
    return DiagnosedSilenceableFailure::success();
  }
  return emitDefaultSilenceableFailure(target);
}
```
- **EN**: Implements logic around `emplace_back`, `applyToOne`, `loopScheduling`, `getReadLatency`, and 7 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `emplace_back`, `applyToOne`, `loopScheduling`, `getReadLatency`, and 7 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 363-381
```cpp

//===----------------------------------------------------------------------===//
// LoopPromoteIfOneIterationOp
//===----------------------------------------------------------------------===//

DiagnosedSilenceableFailure transform::LoopPromoteIfOneIterationOp::applyToOne(
    transform::TransformRewriter &rewriter, LoopLikeOpInterface target,
    transform::ApplyToEachResultList &results,
    transform::TransformState &state) {
  (void)target.promoteIfSingleIteration(rewriter);
  return DiagnosedSilenceableFailure::success();
}

void transform::LoopPromoteIfOneIterationOp::getEffects(
    SmallVectorImpl<MemoryEffects::EffectInstance> &effects) {
  consumesHandle(getTargetMutable(), effects);
  modifiesPayload(effects);
}

```
- **EN**: Implements logic around `applyToOne`, `promoteIfSingleIteration`, `success`, `getEffects`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `applyToOne`, `promoteIfSingleIteration`, `success`, `getEffects`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 382-399
```cpp
//===----------------------------------------------------------------------===//
// LoopUnrollOp
//===----------------------------------------------------------------------===//

DiagnosedSilenceableFailure
transform::LoopUnrollOp::applyToOne(transform::TransformRewriter &rewriter,
                                    Operation *op,
                                    transform::ApplyToEachResultList &results,
                                    transform::TransformState &state) {
  LogicalResult result(failure());
  if (scf::ForOp scfFor = dyn_cast<scf::ForOp>(op))
    result = loopUnrollByFactor(scfFor, getFactor());
  else if (AffineForOp affineFor = dyn_cast<AffineForOp>(op))
    result = loopUnrollByFactor(affineFor, getFactor());
  else
    return emitSilenceableError()
           << "failed to unroll, incorrect type of payload";

```
- **EN**: Implements logic around `applyToOne`, `result`, `ForOp>`, `loopUnrollByFactor`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `applyToOne`, `result`, `ForOp>`, `loopUnrollByFactor`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 400-422
```cpp
  if (failed(result))
    return emitSilenceableError() << "failed to unroll";

  return DiagnosedSilenceableFailure::success();
}

//===----------------------------------------------------------------------===//
// LoopUnrollAndJamOp
//===----------------------------------------------------------------------===//

DiagnosedSilenceableFailure transform::LoopUnrollAndJamOp::applyToOne(
    transform::TransformRewriter &rewriter, Operation *op,
    transform::ApplyToEachResultList &results,
    transform::TransformState &state) {
  LogicalResult result(failure());
  if (scf::ForOp scfFor = dyn_cast<scf::ForOp>(op))
    result = loopUnrollJamByFactor(scfFor, getFactor());
  else if (AffineForOp affineFor = dyn_cast<AffineForOp>(op))
    result = loopUnrollJamByFactor(affineFor, getFactor());
  else
    return emitSilenceableError()
           << "failed to unroll and jam, incorrect type of payload";

```
- **EN**: Implements logic around `failed`, `emitSilenceableError`, `success`, `applyToOne`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `failed`, `emitSilenceableError`, `success`, `applyToOne`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 423-443
```cpp
  if (failed(result))
    return emitSilenceableError() << "failed to unroll and jam";

  return DiagnosedSilenceableFailure::success();
}

//===----------------------------------------------------------------------===//
// LoopCoalesceOp
//===----------------------------------------------------------------------===//

DiagnosedSilenceableFailure
transform::LoopCoalesceOp::applyToOne(transform::TransformRewriter &rewriter,
                                      Operation *op,
                                      transform::ApplyToEachResultList &results,
                                      transform::TransformState &state) {
  LogicalResult result(failure());
  if (scf::ForOp scfForOp = dyn_cast<scf::ForOp>(op))
    result = coalescePerfectlyNestedSCFForLoops(scfForOp);
  else if (AffineForOp affineForOp = dyn_cast<AffineForOp>(op))
    result = coalescePerfectlyNestedAffineLoops(affineForOp);

```
- **EN**: Implements logic around `failed`, `emitSilenceableError`, `success`, `applyToOne`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `failed`, `emitSilenceableError`, `success`, `applyToOne`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 444-464
```cpp
  results.push_back(op);
  if (failed(result)) {
    DiagnosedSilenceableFailure diag = emitSilenceableError()
                                       << "failed to coalesce";
    return diag;
  }
  return DiagnosedSilenceableFailure::success();
}

//===----------------------------------------------------------------------===//
// LoopCoalesceNestedOp
//===----------------------------------------------------------------------===//
DiagnosedSilenceableFailure transform::LoopCoalesceNestedOp::applyToOne(
    transform::TransformRewriter &rewriter, Operation *op,
    transform::ApplyToEachResultList &results,
    transform::TransformState &state) {
  auto forOp = dyn_cast<scf::ForOp>(op);
  if (!forOp) {
    return emitSilenceableError() << "expected scf.for operation";
  }

```
- **EN**: Implements logic around `push_back`, `failed`, `emitSilenceableError`, `success`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `push_back`, `failed`, `emitSilenceableError`, `success`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 465-481
```cpp
  // Collect nested loops (including imperfectly nested ones)
  SmallVector<scf::ForOp> nestedLoops;
  scf::ForOp currentLoop = forOp;

  while (currentLoop) {
    nestedLoops.push_back(currentLoop);
    Block &body = currentLoop.getRegion().front();

    // Look for the next nested loop
    scf::ForOp nextLoop = nullptr;
    for (Operation &bodyOp : body) {
      if (auto innerFor = dyn_cast<scf::ForOp>(&bodyOp)) {
        nextLoop = innerFor;
        break;
      }
    }

```
- **EN**: Implements logic around `push_back`, `getRegion`, `ForOp>`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `push_back`, `getRegion`, `ForOp>` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 482-498
```cpp
    currentLoop = nextLoop;
  }

  // Need at least 2 loops to coalesce
  if (nestedLoops.size() < 2) {
    return emitSilenceableError() << "need at least 2 nested loops to coalesce";
  }

  // Call coalesceLoops directly
  if (failed(coalesceLoops(rewriter, nestedLoops))) {
    return emitSilenceableError() << "failed to coalesce nested loops";
  }

  results.push_back(nestedLoops.front());
  return DiagnosedSilenceableFailure::success();
}

```
- **EN**: Implements logic around `size`, `emitSilenceableError`, `failed`, `push_back`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `size`, `emitSilenceableError`, `failed`, `push_back`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 499-514
```cpp
//===----------------------------------------------------------------------===//
// TakeAssumedBranchOp
//===----------------------------------------------------------------------===//
/// Replaces the given op with the contents of the given single-block region,
/// using the operands of the block terminator to replace operation results.
static void replaceOpWithRegion(RewriterBase &rewriter, Operation *op,
                                Region &region) {
  assert(region.hasOneBlock() && "expected single-block region");
  Block *block = &region.front();
  Operation *terminator = block->getTerminator();
  ValueRange results = terminator->getOperands();
  rewriter.inlineBlockBefore(block, op, /*blockArgs=*/{});
  rewriter.replaceOp(op, results);
  rewriter.eraseOp(terminator);
}

```
- **EN**: Implements logic around `replaceOpWithRegion`, `assert`, `front`, `getTerminator`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `replaceOpWithRegion`, `assert`, `front`, `getTerminator`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 515-530
```cpp
DiagnosedSilenceableFailure transform::TakeAssumedBranchOp::applyToOne(
    transform::TransformRewriter &rewriter, scf::IfOp ifOp,
    transform::ApplyToEachResultList &results,
    transform::TransformState &state) {
  rewriter.setInsertionPoint(ifOp);
  Region &region =
      getTakeElseBranch() ? ifOp.getElseRegion() : ifOp.getThenRegion();
  if (!region.hasOneBlock()) {
    return emitDefiniteFailure()
           << "requires an scf.if op with a single-block "
           << ((getTakeElseBranch()) ? "`else`" : "`then`") << " region";
  }
  replaceOpWithRegion(rewriter, ifOp, region);
  return DiagnosedSilenceableFailure::success();
}

```
- **EN**: Implements logic around `applyToOne`, `setInsertionPoint`, `getTakeElseBranch`, `hasOneBlock`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `applyToOne`, `setInsertionPoint`, `getTakeElseBranch`, `hasOneBlock`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 531-544
```cpp
void transform::TakeAssumedBranchOp::getEffects(
    SmallVectorImpl<MemoryEffects::EffectInstance> &effects) {
  onlyReadsHandle(getTargetMutable(), effects);
  modifiesPayload(effects);
}

//===----------------------------------------------------------------------===//
// LoopFuseSiblingOp
//===----------------------------------------------------------------------===//

/// Check if `target` and `source` are siblings, in the context that `target`
/// is being fused into `source`.
///
/// This is a simple check that just checks if both operations are in the same
```
- **EN**: Implements logic around `getEffects`, `onlyReadsHandle`, `modifiesPayload`.
- **CN**: 围绕 `getEffects`, `onlyReadsHandle`, `modifiesPayload` 实现具体逻辑。

### Lines 545-558
```cpp
/// block and some checks to ensure that the fused IR does not violate
/// dominance.
static DiagnosedSilenceableFailure isOpSibling(Operation *target,
                                               Operation *source) {
  // Check if both operations are same.
  if (target == source)
    return emitSilenceableFailure(source)
           << "target and source need to be different loops";

  // Check if both operations are in the same block.
  if (target->getBlock() != source->getBlock())
    return emitSilenceableFailure(source)
           << "target and source are not in the same block";

```
- **EN**: Implements logic around `isOpSibling`, `emitSilenceableFailure`, `getBlock`.
- **CN**: 围绕 `isOpSibling`, `emitSilenceableFailure`, `getBlock` 实现具体逻辑。

### Lines 559-574
```cpp
  // Check if fusion will violate dominance.
  DominanceInfo domInfo(source);
  if (target->isBeforeInBlock(source)) {
    // Since `target` is before `source`, all users of results of `target`
    // need to be dominated by `source`.
    for (Operation *user : target->getUsers()) {
      if (!domInfo.properlyDominates(source, user, /*enclosingOpOk=*/false)) {
        return emitSilenceableFailure(target)
               << "user of results of target should be properly dominated by "
                  "source";
      }
    }
  } else {
    // Since `target` is after `source`, all values used by `target` need
    // to dominate `source`.

```
- **EN**: Implements logic around `domInfo`, `isBeforeInBlock`, `getUsers`, `properlyDominates`, and 1 more symbols.
- **CN**: 围绕 `domInfo`, `isBeforeInBlock`, `getUsers`, `properlyDominates`, and 1 more symbols 实现具体逻辑。

### Lines 575-589
```cpp
    // Check if operands of `target` are dominated by `source`.
    for (Value operand : target->getOperands()) {
      Operation *operandOp = operand.getDefiningOp();
      // Operands without defining operations are block arguments. When `target`
      // and `source` occur in the same block, these operands dominate `source`.
      if (!operandOp)
        continue;

      // Operand's defining operation should properly dominate `source`.
      if (!domInfo.properlyDominates(operandOp, source,
                                     /*enclosingOpOk=*/false))
        return emitSilenceableFailure(target)
               << "operands of target should be properly dominated by source";
    }

```
- **EN**: Implements logic around `getOperands`, `getDefiningOp`, `properlyDominates`, `emitSilenceableFailure`.
- **CN**: 围绕 `getOperands`, `getDefiningOp`, `properlyDominates`, `emitSilenceableFailure` 实现具体逻辑。

### Lines 590-603
```cpp
    // Check if values used by `target` are dominated by `source`.
    bool failed = false;
    OpOperand *failedValue = nullptr;
    visitUsedValuesDefinedAbove(target->getRegions(), [&](OpOperand *operand) {
      Operation *operandOp = operand->get().getDefiningOp();
      if (operandOp && !domInfo.properlyDominates(operandOp, source,
                                                  /*enclosingOpOk=*/false)) {
        // `operand` is not an argument of an enclosing block and the defining
        // op of `operand` is outside `target` but does not dominate `source`.
        failed = true;
        failedValue = operand;
      }
    });

```
- **EN**: Implements logic around `visitUsedValuesDefinedAbove`, `get`, `properlyDominates`.
- **CN**: 围绕 `visitUsedValuesDefinedAbove`, `get`, `properlyDominates` 实现具体逻辑。

### Lines 604-617
```cpp
    if (failed)
      return emitSilenceableFailure(failedValue->getOwner())
             << "values used inside regions of target should be properly "
                "dominated by source";
  }

  return DiagnosedSilenceableFailure::success();
}

/// Check if `target` scf.forall can be fused into `source` scf.forall.
///
/// This simply checks if both loops have the same bounds, steps and mapping.
/// No attempt is made at checking that the side effects of `target` and
/// `source` are independent of each other.
```
- **EN**: Implements logic around `emitSilenceableFailure`, `success`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `emitSilenceableFailure`, `success` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 618-631
```cpp
static bool isForallWithIdenticalConfiguration(Operation *target,
                                               Operation *source) {
  auto targetOp = dyn_cast<scf::ForallOp>(target);
  auto sourceOp = dyn_cast<scf::ForallOp>(source);
  if (!targetOp || !sourceOp)
    return false;

  return targetOp.getMixedLowerBound() == sourceOp.getMixedLowerBound() &&
         targetOp.getMixedUpperBound() == sourceOp.getMixedUpperBound() &&
         targetOp.getMixedStep() == sourceOp.getMixedStep() &&
         targetOp.getMapping() == sourceOp.getMapping();
}

/// Check if `target` scf.for can be fused into `source` scf.for.
```
- **EN**: Implements logic around `isForallWithIdenticalConfiguration`, `ForallOp>`, `getMixedLowerBound`, `getMixedUpperBound`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `isForallWithIdenticalConfiguration`, `ForallOp>`, `getMixedLowerBound`, `getMixedUpperBound`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 632-647
```cpp
///
/// This simply checks if both loops have the same bounds and steps. No attempt
/// is made at checking that the side effects of `target` and `source` are
/// independent of each other.
static bool isForWithIdenticalConfiguration(Operation *target,
                                            Operation *source) {
  auto targetOp = dyn_cast<scf::ForOp>(target);
  auto sourceOp = dyn_cast<scf::ForOp>(source);
  if (!targetOp || !sourceOp)
    return false;

  return targetOp.getLowerBound() == sourceOp.getLowerBound() &&
         targetOp.getUpperBound() == sourceOp.getUpperBound() &&
         targetOp.getStep() == sourceOp.getStep();
}

```
- **EN**: Implements logic around `isForWithIdenticalConfiguration`, `ForOp>`, `getLowerBound`, `getUpperBound`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `isForWithIdenticalConfiguration`, `ForOp>`, `getLowerBound`, `getUpperBound`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 648-662
```cpp
DiagnosedSilenceableFailure
transform::LoopFuseSiblingOp::apply(transform::TransformRewriter &rewriter,
                                    transform::TransformResults &results,
                                    transform::TransformState &state) {
  auto targetOps = state.getPayloadOps(getTarget());
  auto sourceOps = state.getPayloadOps(getSource());

  if (!llvm::hasSingleElement(targetOps) ||
      !llvm::hasSingleElement(sourceOps)) {
    return emitDefiniteFailure()
           << "requires exactly one target handle (got "
           << llvm::range_size(targetOps) << ") and exactly one "
           << "source handle (got " << llvm::range_size(sourceOps) << ")";
  }

```
- **EN**: Implements logic around `apply`, `getPayloadOps`, `hasSingleElement`, `emitDefiniteFailure`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `apply`, `getPayloadOps`, `hasSingleElement`, `emitDefiniteFailure`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 663-683
```cpp
  Operation *target = *targetOps.begin();
  Operation *source = *sourceOps.begin();

  // Check if the target and source are siblings.
  DiagnosedSilenceableFailure diag = isOpSibling(target, source);
  if (!diag.succeeded())
    return diag;

  Operation *fusedLoop;
  /// TODO: Support fusion for loop-like ops besides scf.for and scf.forall.
  if (isForWithIdenticalConfiguration(target, source)) {
    fusedLoop = fuseIndependentSiblingForLoops(
        cast<scf::ForOp>(target), cast<scf::ForOp>(source), rewriter);
  } else if (isForallWithIdenticalConfiguration(target, source)) {
    fusedLoop = fuseIndependentSiblingForallLoops(
        cast<scf::ForallOp>(target), cast<scf::ForallOp>(source), rewriter);
  } else {
    return emitSilenceableFailure(target->getLoc())
           << "operations cannot be fused";
  }

```
- **EN**: Implements logic around `begin`, `isOpSibling`, `succeeded`, `isForWithIdenticalConfiguration`, and 6 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `begin`, `isOpSibling`, `succeeded`, `isForWithIdenticalConfiguration`, and 6 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 684-698
```cpp
  assert(fusedLoop && "failed to fuse operations");

  results.set(cast<OpResult>(getFusedLoop()), {fusedLoop});
  return DiagnosedSilenceableFailure::success();
}

//===----------------------------------------------------------------------===//
// Transform op registration
//===----------------------------------------------------------------------===//

namespace {
class SCFTransformDialectExtension
    : public transform::TransformDialectExtension<
          SCFTransformDialectExtension> {
public:
```
- **EN**: Introduces declarations for `SCFTransformDialectExtension`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `SCFTransformDialectExtension` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 699-714
```cpp
  MLIR_DEFINE_EXPLICIT_INTERNAL_INLINE_TYPE_ID(SCFTransformDialectExtension)

  using Base::Base;

  void init() {
    declareGeneratedDialect<affine::AffineDialect>();
    declareGeneratedDialect<func::FuncDialect>();

    registerTransformOps<
#define GET_OP_LIST
#include "mlir/Dialect/SCF/TransformOps/SCFTransformOps.cpp.inc"
        >();
  }
};
} // namespace

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/SCF/TransformOps/SCFTransformOps.cpp.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/SCF/TransformOps/SCFTransformOps.cpp.inc`。

### Lines 715-720
```cpp
#define GET_OP_CLASSES
#include "mlir/Dialect/SCF/TransformOps/SCFTransformOps.cpp.inc"

void mlir::scf::registerTransformDialectExtension(DialectRegistry &registry) {
  registry.addExtensions<SCFTransformDialectExtension>();
}
```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/SCF/TransformOps/SCFTransformOps.cpp.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/SCF/TransformOps/SCFTransformOps.cpp.inc`。

## Key Concepts / 关键概念

- **Transform dialect orchestration / Transform Dialect 编排**:
  - **EN**: Represents transform operations that drive structured rewrites over payload IR.
  - **CN**: 表示驱动负载 IR 结构化重写的 Transform Dialect 操作。
- **Pattern rewriting / 模式重写**:
  - **EN**: Uses MLIR rewrite patterns to match operations and replace them with improved forms.
  - **CN**: 使用 MLIR 重写模式匹配操作，并将其替换为更合适的形式。
- **Dialect conversion pipeline / 方言转换流水线**:
  - **EN**: Coordinates legality checks, type adaptation, and target-specific lowering.
  - **CN**: 协调合法性检查、类型适配以及目标相关 lowering。
- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Models reusable capabilities that can be queried across dialect boundaries.
  - **CN**: 建模可跨方言查询的可复用能力。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/SCF/TransformOps/SCFTransformOps.h`, `mlir/Conversion/SCFToControlFlow/SCFToControlFlow.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/LoopUtils.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/SCF/IR/SCF.h`, `mlir/Dialect/SCF/Transforms/Patterns.h`, `mlir/Dialect/SCF/Transforms/Transforms.h`, `mlir/Dialect/SCF/Utils/Utils.h`, `mlir/Dialect/Transform/IR/TransformDialect.h` ... (+7 more)
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (13), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (3)
- **Generated macros / 生成宏**: `GET_OP_LIST`, `GET_OP_CLASSES`
