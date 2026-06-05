# LoopPipelining.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/SCF/Transforms/LoopPipelining.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements loop software pipelining.
  - **CN**: 该文件位于 `mlir/lib/Dialect/SCF/Transforms`，围绕 SCF 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24
```cpp
//===- LoopPipelining.cpp - Code to perform loop software pipelining-------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements loop software pipelining
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/SCF/IR/SCF.h"
#include "mlir/Dialect/SCF/Transforms/Patterns.h"
#include "mlir/Dialect/SCF/Transforms/Transforms.h"
#include "mlir/Dialect/SCF/Utils/Utils.h"
#include "mlir/IR/IRMapping.h"
#include "mlir/IR/PatternMatch.h"
#include "mlir/Transforms/RegionUtils.h"
#include "llvm/ADT/MapVector.h"
#include "llvm/Support/DebugLog.h"
#include "llvm/Support/MathExtras.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/SCF/IR/SCF.h`, `mlir/Dialect/SCF/Transforms/Patterns.h`, `mlir/Dialect/SCF/Transforms/Transforms.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/SCF/IR/SCF.h`, `mlir/Dialect/SCF/Transforms/Patterns.h`, `mlir/Dialect/SCF/Transforms/Transforms.h`。

### Lines 25-39
```cpp
#define DEBUG_TYPE "scf-loop-pipelining"

using namespace mlir;
using namespace mlir::scf;

namespace {

/// Helper to keep internal information during pipelining transformation.
struct LoopPipelinerInternal {
  /// Coarse liverange information for ops used across stages.
  struct LiverangeInfo {
    unsigned lastUseStage = 0;
    unsigned defStage = 0;
  };

```
- **EN**: Defines preprocessor-controlled structure, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成式钩子或编译期常量。

### Lines 40-58
```cpp
protected:
  ForOp forOp;
  unsigned maxStage = 0;
  DenseMap<Operation *, unsigned> stages;
  std::vector<Operation *> opOrder;
  Value ub;
  Value lb;
  Value step;
  bool dynamicLoop;
  PipeliningOption::AnnotationlFnType annotateFn = nullptr;
  bool peelEpilogue;
  PipeliningOption::PredicateOpFn predicateFn = nullptr;

  // When peeling the kernel we generate several version of each value for
  // different stage of the prologue. This map tracks the mapping between
  // original Values in the loop and the different versions
  // peeled from the loop.
  DenseMap<Value, llvm::SmallVector<Value>> valueMapping;

```
- **EN**: Transitions between access-control regions inside a class or struct definition.
- **CN**: 在类或结构体定义内部切换访问控制区域。

### Lines 59-72
```cpp
  /// Assign a value to `valueMapping`, this means `val` represents the version
  /// `idx` of `key` in the epilogue.
  void setValueMapping(Value key, Value el, int64_t idx);

  /// Return the defining op of the given value, if the Value is an argument of
  /// the loop return the associated defining op in the loop and its distance to
  /// the Value.
  std::pair<Operation *, int64_t> getDefiningOpAndDistance(Value value);

  /// Return true if the schedule is possible and return false otherwise. A
  /// schedule is correct if all definitions are scheduled before uses.
  bool verifySchedule();

public:
```
- **EN**: Implements logic around `setValueMapping`, `getDefiningOpAndDistance`, `verifySchedule`.
- **CN**: 围绕 `setValueMapping`, `getDefiningOpAndDistance`, `verifySchedule` 实现具体逻辑。

### Lines 73-86
```cpp
  /// Initalize the information for the given `op`, return true if it
  /// satisfies the pre-condition to apply pipelining.
  bool initializeLoopInfo(ForOp op, const PipeliningOption &options);
  /// Emits the prologue, this creates `maxStage - 1` part which will contain
  /// operations from stages [0; i], where i is the part index.
  LogicalResult emitPrologue(RewriterBase &rewriter);
  /// Gather liverange information for Values that are used in a different stage
  /// than its definition.
  llvm::MapVector<Value, LiverangeInfo> analyzeCrossStageValues();
  scf::ForOp createKernelLoop(
      const llvm::MapVector<Value, LiverangeInfo> &crossStageValues,
      RewriterBase &rewriter,
      llvm::DenseMap<std::pair<Value, unsigned>, unsigned> &loopArgMap);
  /// Emits the pipelined kernel. This clones loop operations following user
```
- **EN**: Implements logic around `initializeLoopInfo`, `emitPrologue`, `analyzeCrossStageValues`, `createKernelLoop`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `initializeLoopInfo`, `emitPrologue`, `analyzeCrossStageValues`, `createKernelLoop` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 87-106
```cpp
  /// order and remaps operands defined in a different stage as their use.
  LogicalResult createKernel(
      scf::ForOp newForOp,
      const llvm::MapVector<Value, LiverangeInfo> &crossStageValues,
      const llvm::DenseMap<std::pair<Value, unsigned>, unsigned> &loopArgMap,
      RewriterBase &rewriter);
  /// Emits the epilogue, this creates `maxStage - 1` part which will contain
  /// operations from stages [i; maxStage], where i is the part index.
  LogicalResult emitEpilogue(RewriterBase &rewriter,
                             llvm::SmallVector<Value> &returnValues);
};

bool LoopPipelinerInternal::initializeLoopInfo(
    ForOp op, const PipeliningOption &options) {
  LDBG() << "Start initializeLoopInfo";
  forOp = op;
  ub = forOp.getUpperBound();
  lb = forOp.getLowerBound();
  step = forOp.getStep();

```
- **EN**: Implements logic around `createKernel`, `emitEpilogue`, `initializeLoopInfo`, `LDBG`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `createKernel`, `emitEpilogue`, `initializeLoopInfo`, `LDBG`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 107-120
```cpp
  std::vector<std::pair<Operation *, unsigned>> schedule;
  options.getScheduleFn(forOp, schedule);
  if (schedule.empty()) {
    LDBG() << "--empty schedule -> BAIL";
    return false;
  }

  opOrder.reserve(schedule.size());
  for (auto &opSchedule : schedule) {
    maxStage = std::max(maxStage, opSchedule.second);
    stages[opSchedule.first] = opSchedule.second;
    opOrder.push_back(opSchedule.first);
  }

```
- **EN**: Implements logic around `getScheduleFn`, `empty`, `LDBG`, `reserve`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getScheduleFn`, `empty`, `LDBG`, `reserve`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 121-148
```cpp
  dynamicLoop = true;
  auto upperBoundCst = getConstantIntValue(ub);
  auto lowerBoundCst = getConstantIntValue(lb);
  auto stepCst = getConstantIntValue(step);
  if (!upperBoundCst || !lowerBoundCst || !stepCst) {
    if (!options.supportDynamicLoops) {
      LDBG() << "--dynamic loop not supported -> BAIL";
      return false;
    }
  } else {
    int64_t ubImm = upperBoundCst.value();
    int64_t lbImm = lowerBoundCst.value();
    int64_t stepImm = stepCst.value();
    if (stepImm <= 0) {
      LDBG() << "--invalid loop step -> BAIL";
      return false;
    }
    int64_t numIteration = llvm::divideCeilSigned(ubImm - lbImm, stepImm);
    if (numIteration >= maxStage) {
      dynamicLoop = false;
    } else if (!options.supportDynamicLoops) {
      LDBG() << "--fewer loop iterations than pipeline stages -> BAIL";
      return false;
    }
  }
  peelEpilogue = options.peelEpilogue;
  predicateFn = options.predicateFn;
  if ((!peelEpilogue || dynamicLoop) && predicateFn == nullptr) {
```
- **EN**: Implements logic around `getConstantIntValue`, `LDBG`, `value`, `divideCeilSigned`.
- **CN**: 围绕 `getConstantIntValue`, `LDBG`, `value`, `divideCeilSigned` 实现具体逻辑。

### Lines 149-166
```cpp
    LDBG() << "--no epilogue or predicate set -> BAIL";
    return false;
  }

  // All operations need to have a stage.
  for (Operation &op : forOp.getBody()->without_terminator()) {
    if (!stages.contains(&op)) {
      op.emitOpError("not assigned a pipeline stage");
      LDBG() << "--op not assigned a pipeline stage: " << op << " -> BAIL";
      return false;
    }
  }

  if (!verifySchedule()) {
    LDBG() << "--invalid schedule: " << op << " -> BAIL";
    return false;
  }

```
- **EN**: Implements logic around `LDBG`, `getBody`, `contains`, `emitOpError`, and 1 more symbols; this block checks operation invariants or verification rules.
- **CN**: 围绕 `LDBG`, `getBody`, `contains`, `emitOpError`, and 1 more symbols 实现具体逻辑；该代码块检查操作不变式或验证规则。

### Lines 167-187
```cpp
  // Currently, we do not support assigning stages to ops in nested regions. The
  // block of all operations assigned a stage should be the single `scf.for`
  // body block.
  for (const auto &[op, stageNum] : stages) {
    (void)stageNum;
    if (op == forOp.getBody()->getTerminator()) {
      op->emitError("terminator should not be assigned a stage");
      LDBG() << "--terminator should not be assigned stage: " << *op
             << " -> BAIL";
      return false;
    }
    if (op->getBlock() != forOp.getBody()) {
      op->emitOpError("the owning Block of all operations assigned a stage "
                      "should be the loop body block");
      LDBG() << "--the owning Block of all operations assigned a stage "
                "should be the loop body block: "
             << *op << " -> BAIL";
      return false;
    }
  }

```
- **EN**: Implements logic around `getBody`, `emitError`, `LDBG`, `getBlock`, and 1 more symbols; this block checks operation invariants or verification rules; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getBody`, `emitError`, `LDBG`, `getBlock`, and 1 more symbols 实现具体逻辑；该代码块检查操作不变式或验证规则，并协调核心结构化 MLIR 方言之间的行为。

### Lines 188-205
```cpp
  // Support only loop-carried dependencies with a distance of one iteration or
  // those defined outside of the loop. This means that any dependency within a
  // loop should either be on the immediately preceding iteration, the current
  // iteration, or on variables whose values are set before entering the loop.
  if (llvm::any_of(forOp.getBody()->getTerminator()->getOperands(),
                   [this](Value operand) {
                     Operation *def = operand.getDefiningOp();
                     return !def ||
                            (!stages.contains(def) && forOp->isAncestor(def));
                   })) {
    LDBG() << "--only support loop carried dependency with a distance of 1 or "
              "defined outside of the loop -> BAIL";
    return false;
  }
  annotateFn = options.annotateFn;
  return true;
}

```
- **EN**: Implements logic around `any_of`, `getDefiningOp`, `contains`, `LDBG`.
- **CN**: 围绕 `any_of`, `getDefiningOp`, `contains`, `LDBG` 实现具体逻辑。

### Lines 206-233
```cpp
/// Find operands of all the nested operations within `op`.
static SetVector<Value> getNestedOperands(Operation *op) {
  SetVector<Value> operands;
  op->walk([&](Operation *nestedOp) {
    operands.insert_range(nestedOp->getOperands());
  });
  return operands;
}

/// Compute unrolled cycles of each op (consumer) and verify that each op is
/// scheduled after its operands (producers) while adjusting for the distance
/// between producer and consumer.
bool LoopPipelinerInternal::verifySchedule() {
  int64_t numCylesPerIter = opOrder.size();
  // Pre-compute the unrolled cycle of each op.
  DenseMap<Operation *, int64_t> unrolledCyles;
  for (int64_t cycle = 0; cycle < numCylesPerIter; cycle++) {
    Operation *def = opOrder[cycle];
    auto it = stages.find(def);
    assert(it != stages.end());
    int64_t stage = it->second;
    unrolledCyles[def] = cycle + stage * numCylesPerIter;
  }
  for (Operation *consumer : opOrder) {
    int64_t consumerCycle = unrolledCyles[consumer];
    for (Value operand : getNestedOperands(consumer)) {
      auto [producer, distance] = getDefiningOpAndDistance(operand);
      if (!producer)
```
- **EN**: Implements logic around `getNestedOperands`, `walk`, `insert_range`, `verifySchedule`, and 4 more symbols; this block checks operation invariants or verification rules.
- **CN**: 围绕 `getNestedOperands`, `walk`, `insert_range`, `verifySchedule`, and 4 more symbols 实现具体逻辑；该代码块检查操作不变式或验证规则。

### Lines 234-248
```cpp
        continue;
      auto it = unrolledCyles.find(producer);
      // Skip producer coming from outside the loop.
      if (it == unrolledCyles.end())
        continue;
      int64_t producerCycle = it->second;
      if (consumerCycle < producerCycle - numCylesPerIter * distance) {
        consumer->emitError("operation scheduled before its operands");
        return false;
      }
    }
  }
  return true;
}

```
- **EN**: Implements logic around `find`, `end`, `emitError`.
- **CN**: 围绕 `find`, `end`, `emitError` 实现具体逻辑。

### Lines 249-267
```cpp
/// Clone `op` and call `callback` on the cloned op's oeprands as well as any
/// operands of nested ops that:
/// 1) aren't defined within the new op or
/// 2) are block arguments.
static Operation *
cloneAndUpdateOperands(RewriterBase &rewriter, Operation *op,
                       function_ref<void(OpOperand *newOperand)> callback) {
  Operation *clone = rewriter.clone(*op);
  clone->walk<WalkOrder::PreOrder>([&](Operation *nested) {
    // 'clone' itself will be visited first.
    for (OpOperand &operand : nested->getOpOperands()) {
      Operation *def = operand.get().getDefiningOp();
      if ((def && !clone->isAncestor(def)) || isa<BlockArgument>(operand.get()))
        callback(&operand);
    }
  });
  return clone;
}

```
- **EN**: Implements logic around `cloneAndUpdateOperands`, `function_ref`, `clone`, `PreOrder>`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `cloneAndUpdateOperands`, `function_ref`, `clone`, `PreOrder>`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 268-290
```cpp
LogicalResult LoopPipelinerInternal::emitPrologue(RewriterBase &rewriter) {
  // Initialize the iteration argument to the loop initial values.
  for (auto [arg, operand] :
       llvm::zip(forOp.getRegionIterArgs(), forOp.getInitsMutable())) {
    setValueMapping(arg, operand.get(), 0);
  }
  auto yield = cast<scf::YieldOp>(forOp.getBody()->getTerminator());
  Location loc = forOp.getLoc();
  SmallVector<Value> predicates(maxStage);
  for (int64_t i = 0; i < maxStage; i++) {
    if (dynamicLoop) {
      Type t = ub.getType();
      // pred = ub > lb + (i * step)
      Value iv = arith::AddIOp::create(
          rewriter, loc, lb,
          arith::MulIOp::create(
              rewriter, loc, step,
              arith::ConstantOp::create(rewriter, loc,
                                        rewriter.getIntegerAttr(t, i))));
      predicates[i] = arith::CmpIOp::create(rewriter, loc,
                                            arith::CmpIPredicate::slt, iv, ub);
    }

```
- **EN**: Implements logic around `emitPrologue`, `zip`, `setValueMapping`, `YieldOp>`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `emitPrologue`, `zip`, `setValueMapping`, `YieldOp>`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 291-318
```cpp
    // special handling for induction variable as the increment is implicit.
    // iv = lb + i * step
    Type t = lb.getType();
    Value iv = arith::AddIOp::create(
        rewriter, loc, lb,
        arith::MulIOp::create(
            rewriter, loc, step,
            arith::ConstantOp::create(rewriter, loc,
                                      rewriter.getIntegerAttr(t, i))));
    setValueMapping(forOp.getInductionVar(), iv, i);
    for (Operation *op : opOrder) {
      if (stages[op] > i)
        continue;
      Operation *newOp =
          cloneAndUpdateOperands(rewriter, op, [&](OpOperand *newOperand) {
            auto it = valueMapping.find(newOperand->get());
            if (it != valueMapping.end()) {
              Value replacement = it->second[i - stages[op]];
              newOperand->set(replacement);
            }
          });
      int predicateIdx = i - stages[op];
      if (predicates[predicateIdx]) {
        OpBuilder::InsertionGuard insertGuard(rewriter);
        newOp = predicateFn(rewriter, newOp, predicates[predicateIdx]);
        if (newOp == nullptr)
          return failure();
      }
```
- **EN**: Implements logic around `getType`, `create`, `getIntegerAttr`, `setValueMapping`, and 7 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getType`, `create`, `getIntegerAttr`, `setValueMapping`, and 7 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 319-346
```cpp
      if (annotateFn)
        annotateFn(newOp, PipeliningOption::PipelinerPart::Prologue, i);
      for (unsigned destId : llvm::seq(unsigned(0), op->getNumResults())) {
        Value source = newOp->getResult(destId);
        // If the value is a loop carried dependency update the loop argument
        for (OpOperand &operand : yield->getOpOperands()) {
          if (operand.get() != op->getResult(destId))
            continue;
          if (predicates[predicateIdx] &&
              !forOp.getResult(operand.getOperandNumber()).use_empty()) {
            // If the value is used outside the loop, we need to make sure we
            // return the correct version of it.
            Value prevValue = valueMapping
                [forOp.getRegionIterArgs()[operand.getOperandNumber()]]
                [i - stages[op]];
            source = arith::SelectOp::create(
                rewriter, loc, predicates[predicateIdx], source, prevValue);
          }
          setValueMapping(forOp.getRegionIterArgs()[operand.getOperandNumber()],
                          source, i - stages[op] + 1);
        }
        setValueMapping(op->getResult(destId), newOp->getResult(destId),
                        i - stages[op]);
      }
    }
  }
  return success();
}
```
- **EN**: Implements logic around `annotateFn`, `seq`, `getResult`, `getOpOperands`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `annotateFn`, `seq`, `getResult`, `getOpOperands`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 347-367
```cpp

llvm::MapVector<Value, LoopPipelinerInternal::LiverangeInfo>
LoopPipelinerInternal::analyzeCrossStageValues() {
  llvm::MapVector<Value, LoopPipelinerInternal::LiverangeInfo> crossStageValues;
  for (Operation *op : opOrder) {
    unsigned stage = stages[op];

    auto analyzeOperand = [&](OpOperand &operand) {
      auto [def, distance] = getDefiningOpAndDistance(operand.get());
      if (!def)
        return;
      auto defStage = stages.find(def);
      if (defStage == stages.end() || defStage->second == stage ||
          defStage->second == stage + distance)
        return;
      assert(stage > defStage->second);
      LiverangeInfo &info = crossStageValues[operand.get()];
      info.defStage = defStage->second;
      info.lastUseStage = std::max(info.lastUseStage, stage);
    };

```
- **EN**: Implements logic around `analyzeCrossStageValues`, `getDefiningOpAndDistance`, `find`, `end`, and 3 more symbols.
- **CN**: 围绕 `analyzeCrossStageValues`, `getDefiningOpAndDistance`, `find`, `end`, and 3 more symbols 实现具体逻辑。

### Lines 368-395
```cpp
    for (OpOperand &operand : op->getOpOperands())
      analyzeOperand(operand);
    visitUsedValuesDefinedAbove(op->getRegions(), [&](OpOperand *operand) {
      analyzeOperand(*operand);
    });
  }
  return crossStageValues;
}

std::pair<Operation *, int64_t>
LoopPipelinerInternal::getDefiningOpAndDistance(Value value) {
  int64_t distance = 0;
  if (auto arg = dyn_cast<BlockArgument>(value)) {
    if (arg.getOwner() != forOp.getBody())
      return {nullptr, 0};
    // Ignore induction variable.
    if (arg.getArgNumber() == 0)
      return {nullptr, 0};
    distance++;
    value =
        forOp.getBody()->getTerminator()->getOperand(arg.getArgNumber() - 1);
  }
  Operation *def = value.getDefiningOp();
  if (!def)
    return {nullptr, 0};
  return {def, distance};
}

```
- **EN**: Implements logic around `getOpOperands`, `analyzeOperand`, `visitUsedValuesDefinedAbove`, `getDefiningOpAndDistance`, and 4 more symbols.
- **CN**: 围绕 `getOpOperands`, `analyzeOperand`, `visitUsedValuesDefinedAbove`, `getDefiningOpAndDistance`, and 4 more symbols 实现具体逻辑。

### Lines 396-423
```cpp
scf::ForOp LoopPipelinerInternal::createKernelLoop(
    const llvm::MapVector<Value, LoopPipelinerInternal::LiverangeInfo>
        &crossStageValues,
    RewriterBase &rewriter,
    llvm::DenseMap<std::pair<Value, unsigned>, unsigned> &loopArgMap) {
  // Creates the list of initial values associated to values used across
  // stages. The initial values come from the prologue created above.
  // Keep track of the kernel argument associated to each version of the
  // values passed to the kernel.
  llvm::SmallVector<Value> newLoopArg;
  // For existing loop argument initialize them with the right version from the
  // prologue.
  for (const auto &retVal :
       llvm::enumerate(forOp.getBody()->getTerminator()->getOperands())) {
    Operation *def = retVal.value().getDefiningOp();
    assert(def && "Only support loop carried dependencies of distance of 1 or "
                  "outside the loop");
    auto defStage = stages.find(def);
    if (defStage != stages.end()) {
      Value valueVersion =
          valueMapping[forOp.getRegionIterArgs()[retVal.index()]]
                      [maxStage - defStage->second];
      assert(valueVersion);
      newLoopArg.push_back(valueVersion);
    } else {
      newLoopArg.push_back(forOp.getInitArgs()[retVal.index()]);
    }
  }
```
- **EN**: Implements logic around `createKernelLoop`, `enumerate`, `value`, `assert`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `createKernelLoop`, `enumerate`, `value`, `assert`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 424-437
```cpp
  for (auto escape : crossStageValues) {
    LiverangeInfo &info = escape.second;
    Value value = escape.first;
    for (unsigned stageIdx = 0; stageIdx < info.lastUseStage - info.defStage;
         stageIdx++) {
      Value valueVersion =
          valueMapping[value][maxStage - info.lastUseStage + stageIdx];
      assert(valueVersion);
      newLoopArg.push_back(valueVersion);
      loopArgMap[std::make_pair(value, info.lastUseStage - info.defStage -
                                           stageIdx)] = newLoopArg.size() - 1;
    }
  }

```
- **EN**: Implements logic around `assert`, `push_back`, `make_pair`, `size`.
- **CN**: 围绕 `assert`, `push_back`, `make_pair`, `size` 实现具体逻辑。

### Lines 438-461
```cpp
  // Create the new kernel loop. When we peel the epilgue we need to peel
  // `numStages - 1` iterations. Then we adjust the upper bound to remove those
  // iterations.
  Value newUb = forOp.getUpperBound();
  if (peelEpilogue) {
    Type t = ub.getType();
    Location loc = forOp.getLoc();
    // newUb = ub - maxStage * step
    Value maxStageValue = arith::ConstantOp::create(
        rewriter, loc, rewriter.getIntegerAttr(t, maxStage));
    Value maxStageByStep =
        arith::MulIOp::create(rewriter, loc, step, maxStageValue);
    newUb = arith::SubIOp::create(rewriter, loc, ub, maxStageByStep);
  }
  auto newForOp =
      scf::ForOp::create(rewriter, forOp.getLoc(), forOp.getLowerBound(), newUb,
                         forOp.getStep(), newLoopArg);
  // When there are no iter args, the loop body terminator will be created.
  // Since we always create it below, remove the terminator if it was created.
  if (!newForOp.getBody()->empty())
    rewriter.eraseOp(newForOp.getBody()->getTerminator());
  return newForOp;
}

```
- **EN**: Implements logic around `getUpperBound`, `getType`, `getLoc`, `create`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getUpperBound`, `getType`, `getLoc`, `create`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 462-489
```cpp
LogicalResult LoopPipelinerInternal::createKernel(
    scf::ForOp newForOp,
    const llvm::MapVector<Value, LoopPipelinerInternal::LiverangeInfo>
        &crossStageValues,
    const llvm::DenseMap<std::pair<Value, unsigned>, unsigned> &loopArgMap,
    RewriterBase &rewriter) {
  valueMapping.clear();

  // Create the kernel, we clone instruction based on the order given by
  // user and remap operands coming from a previous stages.
  rewriter.setInsertionPoint(newForOp.getBody(), newForOp.getBody()->begin());
  IRMapping mapping;
  mapping.map(forOp.getInductionVar(), newForOp.getInductionVar());
  for (const auto &arg : llvm::enumerate(forOp.getRegionIterArgs())) {
    mapping.map(arg.value(), newForOp.getRegionIterArgs()[arg.index()]);
  }
  SmallVector<Value> predicates(maxStage + 1, nullptr);
  if (!peelEpilogue) {
    // Create a predicate for each stage except the last stage.
    Location loc = newForOp.getLoc();
    Type t = ub.getType();
    for (unsigned i = 0; i < maxStage; i++) {
      // c = ub - (maxStage - i) * step
      Value c = arith::SubIOp::create(
          rewriter, loc, ub,
          arith::MulIOp::create(
              rewriter, loc, step,
              arith::ConstantOp::create(
```
- **EN**: Implements logic around `createKernel`, `clear`, `setInsertionPoint`, `map`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `createKernel`, `clear`, `setInsertionPoint`, `map`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 490-515
```cpp
                  rewriter, loc,
                  rewriter.getIntegerAttr(t, int64_t(maxStage - i)))));

      Value pred = arith::CmpIOp::create(rewriter, newForOp.getLoc(),
                                         arith::CmpIPredicate::slt,
                                         newForOp.getInductionVar(), c);
      predicates[i] = pred;
    }
  }
  for (Operation *op : opOrder) {
    int64_t useStage = stages[op];
    auto *newOp = rewriter.clone(*op, mapping);
    SmallVector<OpOperand *> operands;
    // Collect all the operands for the cloned op and its nested ops.
    op->walk([&operands](Operation *nestedOp) {
      for (OpOperand &operand : nestedOp->getOpOperands()) {
        operands.push_back(&operand);
      }
    });
    for (OpOperand *operand : operands) {
      Operation *nestedNewOp = mapping.lookup(operand->getOwner());
      // Special case for the induction variable uses. We replace it with a
      // version incremented based on the stage where it is used.
      if (operand->get() == forOp.getInductionVar()) {
        rewriter.setInsertionPoint(newOp);

```
- **EN**: Implements logic around `getIntegerAttr`, `create`, `getInductionVar`, `clone`, and 6 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getIntegerAttr`, `create`, `getInductionVar`, `clone`, and 6 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 516-543
```cpp
        // offset = (maxStage - stages[op]) * step
        Type t = step.getType();
        Value offset = arith::MulIOp::create(
            rewriter, forOp.getLoc(), step,
            arith::ConstantOp::create(
                rewriter, forOp.getLoc(),
                rewriter.getIntegerAttr(t, maxStage - stages[op])));
        Value iv = arith::AddIOp::create(rewriter, forOp.getLoc(),
                                         newForOp.getInductionVar(), offset);
        nestedNewOp->setOperand(operand->getOperandNumber(), iv);
        rewriter.setInsertionPointAfter(newOp);
        continue;
      }
      Value source = operand->get();
      auto arg = dyn_cast<BlockArgument>(source);
      if (arg && arg.getOwner() == forOp.getBody()) {
        Value ret = forOp.getBody()->getTerminator()->getOperand(
            arg.getArgNumber() - 1);
        Operation *dep = ret.getDefiningOp();
        if (!dep)
          continue;
        auto stageDep = stages.find(dep);
        if (stageDep == stages.end() || stageDep->second == useStage)
          continue;
        // If the value is a loop carried value coming from stage N + 1 remap,
        // it will become a direct use.
        if (stageDep->second == useStage + 1) {
          nestedNewOp->setOperand(operand->getOperandNumber(),
```
- **EN**: Implements logic around `getType`, `create`, `getLoc`, `getIntegerAttr`, and 10 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getType`, `create`, `getLoc`, `getIntegerAttr`, and 10 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 544-564
```cpp
                                  mapping.lookupOrDefault(ret));
          continue;
        }
        source = ret;
      }
      // For operands defined in a previous stage we need to remap it to use
      // the correct region argument. We look for the right version of the
      // Value based on the stage where it is used.
      Operation *def = source.getDefiningOp();
      if (!def)
        continue;
      auto stageDef = stages.find(def);
      if (stageDef == stages.end() || stageDef->second == useStage)
        continue;
      auto remap = loopArgMap.find(
          std::make_pair(operand->get(), useStage - stageDef->second));
      assert(remap != loopArgMap.end());
      nestedNewOp->setOperand(operand->getOperandNumber(),
                              newForOp.getRegionIterArgs()[remap->second]);
    }

```
- **EN**: Implements logic around `lookupOrDefault`, `getDefiningOp`, `find`, `end`, and 4 more symbols.
- **CN**: 围绕 `lookupOrDefault`, `getDefiningOp`, `find`, `end`, and 4 more symbols 实现具体逻辑。

### Lines 565-592
```cpp
    if (predicates[useStage]) {
      OpBuilder::InsertionGuard insertGuard(rewriter);
      newOp = predicateFn(rewriter, newOp, predicates[useStage]);
      if (!newOp)
        return failure();
      // Remap the results to the new predicated one.
      for (auto values : llvm::zip(op->getResults(), newOp->getResults()))
        mapping.map(std::get<0>(values), std::get<1>(values));
    }
    if (annotateFn)
      annotateFn(newOp, PipeliningOption::PipelinerPart::Kernel, 0);
  }

  // Collect the Values that need to be returned by the forOp. For each
  // value we need to have `LastUseStage - DefStage` number of versions
  // returned.
  // We create a mapping between original values and the associated loop
  // returned values that will be needed by the epilogue.
  llvm::SmallVector<Value> yieldOperands;
  for (OpOperand &yieldOperand :
       forOp.getBody()->getTerminator()->getOpOperands()) {
    Value source = mapping.lookupOrDefault(yieldOperand.get());
    // When we don't peel the epilogue and the yield value is used outside the
    // loop we need to make sure we return the version from numStages -
    // defStage.
    if (!peelEpilogue &&
        !forOp.getResult(yieldOperand.getOperandNumber()).use_empty()) {
      Operation *def = getDefiningOpAndDistance(yieldOperand.get()).first;
```
- **EN**: Implements logic around `insertGuard`, `predicateFn`, `failure`, `zip`, and 6 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `insertGuard`, `predicateFn`, `failure`, `zip`, and 6 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 593-606
```cpp
      if (def) {
        auto defStage = stages.find(def);
        if (defStage != stages.end() && defStage->second < maxStage) {
          Value pred = predicates[defStage->second];
          source = arith::SelectOp::create(
              rewriter, pred.getLoc(), pred, source,
              newForOp.getBody()
                  ->getArguments()[yieldOperand.getOperandNumber() + 1]);
        }
      }
    }
    yieldOperands.push_back(source);
  }

```
- **EN**: Implements logic around `find`, `end`, `create`, `getLoc`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `find`, `end`, `create`, `getLoc`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 607-634
```cpp
  for (auto &it : crossStageValues) {
    int64_t version = maxStage - it.second.lastUseStage + 1;
    unsigned numVersionReturned = it.second.lastUseStage - it.second.defStage;
    // add the original version to yield ops.
    // If there is a live range spanning across more than 2 stages we need to
    // add extra arg.
    for (unsigned i = 1; i < numVersionReturned; i++) {
      setValueMapping(it.first, newForOp->getResult(yieldOperands.size()),
                      version++);
      yieldOperands.push_back(
          newForOp.getBody()->getArguments()[yieldOperands.size() + 1 +
                                             newForOp.getNumInductionVars()]);
    }
    setValueMapping(it.first, newForOp->getResult(yieldOperands.size()),
                    version++);
    yieldOperands.push_back(mapping.lookupOrDefault(it.first));
  }
  // Map the yield operand to the forOp returned value.
  for (const auto &retVal :
       llvm::enumerate(forOp.getBody()->getTerminator()->getOperands())) {
    Operation *def = retVal.value().getDefiningOp();
    assert(def && "Only support loop carried dependencies of distance of 1 or "
                  "defined outside the loop");
    auto defStage = stages.find(def);
    if (defStage == stages.end()) {
      for (unsigned int stage = 1; stage <= maxStage; stage++)
        setValueMapping(forOp.getRegionIterArgs()[retVal.index()],
                        retVal.value(), stage);
```
- **EN**: Implements logic around `setValueMapping`, `push_back`, `getBody`, `getNumInductionVars`, and 5 more symbols.
- **CN**: 围绕 `setValueMapping`, `push_back`, `getBody`, `getNumInductionVars`, and 5 more symbols 实现具体逻辑。

### Lines 635-650
```cpp
    } else if (defStage->second > 0) {
      setValueMapping(forOp.getRegionIterArgs()[retVal.index()],
                      newForOp->getResult(retVal.index()),
                      maxStage - defStage->second + 1);
    }
  }
  scf::YieldOp::create(rewriter, forOp.getLoc(), yieldOperands);
  return success();
}

LogicalResult
LoopPipelinerInternal::emitEpilogue(RewriterBase &rewriter,
                                    llvm::SmallVector<Value> &returnValues) {
  Location loc = forOp.getLoc();
  Type t = lb.getType();

```
- **EN**: Implements logic around `setValueMapping`, `getResult`, `create`, `success`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `setValueMapping`, `getResult`, `create`, `success`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 651-668
```cpp
  // Emit different versions of the induction variable. They will be
  // removed by dead code if not used.

  auto createConst = [&](int v) {
    return arith::ConstantOp::create(rewriter, loc,
                                     rewriter.getIntegerAttr(t, v));
  };

  // total_iterations = cdiv(range_diff, step);
  // - range_diff = ub - lb
  // - total_iterations = (range_diff + step + (step < 0 ? 1 : -1)) / step
  Value zero = createConst(0);
  Value one = createConst(1);
  Value stepLessZero = arith::CmpIOp::create(
      rewriter, loc, arith::CmpIPredicate::slt, step, zero);
  Value stepDecr = arith::SelectOp::create(rewriter, loc, stepLessZero, one,
                                           createConst(-1));

```
- **EN**: Implements logic around `create`, `getIntegerAttr`, `createConst`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `create`, `getIntegerAttr`, `createConst` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 669-682
```cpp
  Value rangeDiff = arith::SubIOp::create(rewriter, loc, ub, lb);
  Value rangeIncrStep = arith::AddIOp::create(rewriter, loc, rangeDiff, step);
  Value rangeDecr =
      arith::AddIOp::create(rewriter, loc, rangeIncrStep, stepDecr);
  Value totalIterations =
      arith::DivSIOp::create(rewriter, loc, rangeDecr, step);

  // If total_iters < max_stage, start the epilogue at zero to match the
  // ramp-up in the prologue.
  // start_iter = max(0, total_iters - max_stage)
  Value iterI = arith::SubIOp::create(rewriter, loc, totalIterations,
                                      createConst(maxStage));
  iterI = arith::MaxSIOp::create(rewriter, loc, zero, iterI);

```
- **EN**: Implements logic around `create`, `createConst`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `create`, `createConst` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 683-704
```cpp
  // Capture predicates for dynamic loops.
  SmallVector<Value> predicates(maxStage + 1);

  for (int64_t i = 1; i <= maxStage; i++) {
    // newLastIter = lb + step * iterI
    Value newlastIter = arith::AddIOp::create(
        rewriter, loc, lb, arith::MulIOp::create(rewriter, loc, step, iterI));

    setValueMapping(forOp.getInductionVar(), newlastIter, i);

    // increment to next iterI
    iterI = arith::AddIOp::create(rewriter, loc, iterI, one);

    if (dynamicLoop) {
      // Disable stages when `i` is greater than total_iters.
      // pred = total_iters >= i
      predicates[i] =
          arith::CmpIOp::create(rewriter, loc, arith::CmpIPredicate::sge,
                                totalIterations, createConst(i));
    }
  }

```
- **EN**: Implements logic around `predicates`, `create`, `setValueMapping`, `createConst`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `predicates`, `create`, `setValueMapping`, `createConst` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 705-730
```cpp
  // Emit `maxStage - 1` epilogue part that includes operations from stages
  // [i; maxStage].
  for (int64_t i = 1; i <= maxStage; i++) {
    SmallVector<std::pair<Value, unsigned>> returnMap(returnValues.size());
    for (Operation *op : opOrder) {
      if (stages[op] < i)
        continue;
      unsigned currentVersion = maxStage - stages[op] + i;
      unsigned nextVersion = currentVersion + 1;
      Operation *newOp =
          cloneAndUpdateOperands(rewriter, op, [&](OpOperand *newOperand) {
            auto it = valueMapping.find(newOperand->get());
            if (it != valueMapping.end()) {
              Value replacement = it->second[currentVersion];
              newOperand->set(replacement);
            }
          });
      if (dynamicLoop) {
        OpBuilder::InsertionGuard insertGuard(rewriter);
        newOp = predicateFn(rewriter, newOp, predicates[currentVersion]);
        if (!newOp)
          return failure();
      }
      if (annotateFn)
        annotateFn(newOp, PipeliningOption::PipelinerPart::Epilogue, i - 1);

```
- **EN**: Implements logic around `returnMap`, `cloneAndUpdateOperands`, `find`, `end`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `returnMap`, `cloneAndUpdateOperands`, `find`, `end`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 731-758
```cpp
      for (auto [opRes, newRes] :
           llvm::zip(op->getResults(), newOp->getResults())) {
        setValueMapping(opRes, newRes, currentVersion);
        // If the value is a loop carried dependency update the loop argument
        // mapping and keep track of the last version to replace the original
        // forOp uses.
        for (OpOperand &operand :
             forOp.getBody()->getTerminator()->getOpOperands()) {
          if (operand.get() != opRes)
            continue;
          // If the version is greater than maxStage it means it maps to the
          // original forOp returned value.
          unsigned ri = operand.getOperandNumber();
          returnValues[ri] = newRes;
          Value mapVal = forOp.getRegionIterArgs()[ri];
          returnMap[ri] = std::make_pair(mapVal, currentVersion);
          if (nextVersion <= maxStage)
            setValueMapping(mapVal, newRes, nextVersion);
        }
      }
    }
    if (dynamicLoop) {
      // Select return values from this stage (live outs) based on predication.
      // If the stage is valid select the peeled value, else use previous stage
      // value.
      for (auto pair : llvm::enumerate(returnValues)) {
        unsigned ri = pair.index();
        auto [mapVal, currentVersion] = returnMap[ri];
```
- **EN**: Implements logic around `zip`, `setValueMapping`, `getBody`, `get`, and 5 more symbols.
- **CN**: 围绕 `zip`, `setValueMapping`, `getBody`, `get`, and 5 more symbols 实现具体逻辑。

### Lines 759-774
```cpp
        if (mapVal) {
          unsigned nextVersion = currentVersion + 1;
          Value pred = predicates[currentVersion];
          Value prevValue = valueMapping[mapVal][currentVersion];
          auto selOp = arith::SelectOp::create(rewriter, loc, pred,
                                               pair.value(), prevValue);
          returnValues[ri] = selOp;
          if (nextVersion <= maxStage)
            setValueMapping(mapVal, selOp, nextVersion);
        }
      }
    }
  }
  return success();
}

```
- **EN**: Implements logic around `create`, `value`, `setValueMapping`, `success`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `create`, `value`, `setValueMapping`, `success` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 775-788
```cpp
void LoopPipelinerInternal::setValueMapping(Value key, Value el, int64_t idx) {
  auto it = valueMapping.find(key);
  // If the value is not in the map yet add a vector big enough to store all
  // versions.
  if (it == valueMapping.end())
    it =
        valueMapping
            .insert(std::make_pair(key, llvm::SmallVector<Value>(maxStage + 1)))
            .first;
  it->second[idx] = el;
}

} // namespace

```
- **EN**: Implements logic around `setValueMapping`, `find`, `end`, `insert`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `setValueMapping`, `find`, `end`, `insert` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 789-802
```cpp
FailureOr<ForOp> mlir::scf::pipelineForLoop(RewriterBase &rewriter, ForOp forOp,
                                            const PipeliningOption &options,
                                            bool *modifiedIR) {
  if (modifiedIR)
    *modifiedIR = false;

  // TODO: Add support for unsigned loops.
  if (forOp.getUnsignedCmp())
    return failure();

  LoopPipelinerInternal pipeliner;
  if (!pipeliner.initializeLoopInfo(forOp, options))
    return failure();

```
- **EN**: Implements logic around `pipelineForLoop`, `getUnsignedCmp`, `failure`, `initializeLoopInfo`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `pipelineForLoop`, `getUnsignedCmp`, `failure`, `initializeLoopInfo` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 803-816
```cpp
  if (modifiedIR)
    *modifiedIR = true;

  // 1. Emit prologue.
  if (failed(pipeliner.emitPrologue(rewriter)))
    return failure();

  // 2. Track values used across stages. When a value cross stages it will
  // need to be passed as loop iteration arguments.
  // We first collect the values that are used in a different stage than where
  // they are defined.
  llvm::MapVector<Value, LoopPipelinerInternal::LiverangeInfo>
      crossStageValues = pipeliner.analyzeCrossStageValues();

```
- **EN**: Implements logic around `failed`, `failure`, `analyzeCrossStageValues`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `failed`, `failure`, `analyzeCrossStageValues` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 817-843
```cpp
  // Mapping between original loop values used cross stage and the block
  // arguments associated after pipelining. A Value may map to several
  // arguments if its liverange spans across more than 2 stages.
  llvm::DenseMap<std::pair<Value, unsigned>, unsigned> loopArgMap;
  // 3. Create the new kernel loop and return the block arguments mapping.
  ForOp newForOp =
      pipeliner.createKernelLoop(crossStageValues, rewriter, loopArgMap);
  // Create the kernel block, order ops based on user choice and remap
  // operands.
  if (failed(pipeliner.createKernel(newForOp, crossStageValues, loopArgMap,
                                    rewriter)))
    return failure();

  llvm::SmallVector<Value> returnValues =
      newForOp.getResults().take_front(forOp->getNumResults());
  if (options.peelEpilogue) {
    // 4. Emit the epilogue after the new forOp.
    rewriter.setInsertionPointAfter(newForOp);
    if (failed(pipeliner.emitEpilogue(rewriter, returnValues)))
      return failure();
  }
  // 5. Erase the original loop and replace the uses with the epilogue output.
  if (forOp->getNumResults() > 0)
    rewriter.replaceOp(forOp, returnValues);
  else
    rewriter.eraseOp(forOp);

```
- **EN**: Implements logic around `createKernelLoop`, `failed`, `failure`, `getResults`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `createKernelLoop`, `failed`, `failure`, `getResults`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 844-850
```cpp
  return newForOp;
}

void mlir::scf::populateSCFLoopPipeliningPatterns(
    RewritePatternSet &patterns, const PipeliningOption &options) {
  patterns.add<ForLoopPipeliningPattern>(options, patterns.getContext());
}
```
- **EN**: Implements logic around `populateSCFLoopPipeliningPatterns`, `add`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `populateSCFLoopPipeliningPatterns`, `add` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

## Key Concepts / 关键概念

- **Rewrite and lowering / 重写与 lowering**:
  - **EN**: Uses MLIR pattern infrastructure or passes to canonicalize, legalize, or lower operations.
  - **CN**: 使用 MLIR 模式基础设施或 Pass 对操作进行规范化、合法化或 lowering。
- **Pattern rewriting / 模式重写**:
  - **EN**: Uses MLIR rewrite patterns to match operations and replace them with improved forms.
  - **CN**: 使用 MLIR 重写模式匹配操作，并将其替换为更合适的形式。
- **Operation verification / 操作验证**:
  - **EN**: Checks structural and semantic invariants so malformed IR is rejected early.
  - **CN**: 检查结构与语义不变式，以便尽早拒绝非法 IR。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/SCF/IR/SCF.h`, `mlir/Dialect/SCF/Transforms/Patterns.h`, `mlir/Dialect/SCF/Transforms/Transforms.h`, `mlir/Dialect/SCF/Utils/Utils.h`, `mlir/IR/IRMapping.h`, `mlir/IR/PatternMatch.h`, `mlir/Transforms/RegionUtils.h`, `llvm/ADT/MapVector.h`, `llvm/Support/DebugLog.h` ... (+1 more)
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (5), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (2), LLVM support-library facilities / LLVM Support 库设施 (2), generic transformation and conversion helpers / 通用变换与转换辅助逻辑 (1), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (1)
