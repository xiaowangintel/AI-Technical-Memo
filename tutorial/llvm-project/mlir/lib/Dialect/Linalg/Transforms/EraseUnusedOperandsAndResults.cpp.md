# EraseUnusedOperandsAndResults.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Linalg/Transforms/EraseUnusedOperandsAndResults.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements Linalg dialect support for rewrite patterns, passes, and IR-to-IR transformation logic, centered on `EraseUnusedOperandsAndResults`.
  - **CN**: 实现 Linalg 方言中围绕 `EraseUnusedOperandsAndResults` 的重写模式、Pass 以及 IR 到 IR 的变换逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10
```cpp
//===- EraseUnusedOperandsAndResults.cpp ----------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Linalg/Transforms/Transforms.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/Transforms/Transforms.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/Transforms/Transforms.h`。

### Lines 11-25
```cpp
#include "mlir/Dialect/Linalg/IR/Linalg.h"

using namespace mlir;
using namespace mlir::linalg;

/// Return `true` if the `result` of an operation `genericOp` is dead.
static bool isResultValueDead(linalg::GenericOp genericOp, OpResult result) {
  if (!result.use_empty())
    return false;
  // If out operand not used in payload, we can drop it.
  OpOperand *outputOpOperand =
      genericOp.getDpsInitOperand(result.getResultNumber());
  if (!genericOp.payloadUsesValueFromOperand(outputOpOperand))
    return true;

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/IR/Linalg.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/IR/Linalg.h`。

### Lines 26-38
```cpp
  // The out operand that is part of a payload can be dropped if
  // these conditions are met:
  // - Result from out operand is dead.
  // - User of arg is yield.
  // - outArg data is not being used by other outArgs.

  // Check block arg and cycle from out operand has a single use.
  BlockArgument outputArg =
      genericOp.getRegionOutputArgs()[result.getResultNumber()];
  if (!outputArg.hasOneUse())
    return false;
  Operation *argUserOp = *outputArg.user_begin();

```
- **EN**: Implements logic around `getRegionOutputArgs`, `hasOneUse`, `user_begin`.
- **CN**: 围绕 `getRegionOutputArgs`, `hasOneUse`, `user_begin` 实现具体逻辑。

### Lines 39-51
```cpp
  // Check argUser has no other use.
  if (!argUserOp->use_empty())
    return false;

  // Check that argUser is a yield.
  auto yieldOp = dyn_cast<linalg::YieldOp>(argUserOp);
  if (!yieldOp)
    return false;

  // Check outArg data is not being used by other outArgs.
  if (yieldOp.getOperand(result.getResultNumber()) != outputArg)
    return false;

```
- **EN**: Implements logic around `use_empty`, `YieldOp>`, `getOperand`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `use_empty`, `YieldOp>`, `getOperand` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 52-71
```cpp
  return true;
}

//===---------------------------------------------------------------------===//
// Helper methods for operand deduplication and dead results elimination
//===---------------------------------------------------------------------===//

// Deduplicate input operands, and return the
// - Mapping from operand position in the original op, to operand position in
// the canonicalized op.
// - The preserved input operands list (by reference).
llvm::SmallDenseMap<unsigned, unsigned> static deduplicateInputOperands(
    GenericOp genericOp, SmallVector<OpOperand *> &droppedOpOperands,
    SmallVector<Value> &newInputOperands,
    SmallVector<AffineMap> &newIndexingMaps) {
  llvm::SmallDenseMap<unsigned, unsigned> origToNewPos;
  llvm::SmallDenseMap<std::pair<Value, AffineMap>, unsigned> dedupedInputs;
  for (const auto &en : llvm::enumerate(genericOp.getDpsInputOperands())) {
    OpOperand *inputOpOperand = en.value();
    // Check if operand is dead and if dropping the indexing map makes the
```
- **EN**: Implements logic around `deduplicateInputOperands`, `enumerate`, `value`.
- **CN**: 围绕 `deduplicateInputOperands`, `enumerate`, `value` 实现具体逻辑。

### Lines 72-81
```cpp
    // loops to shape computation invalid.
    if (!genericOp.payloadUsesValueFromOperand(inputOpOperand)) {
      // Add the current operands to the list of potentially droppable
      // operands. If it cannot be dropped, this needs to be popped back.
      droppedOpOperands.push_back(inputOpOperand);
      if (genericOp.canOpOperandsBeDropped(droppedOpOperands))
        continue;
      droppedOpOperands.pop_back();
    }

```
- **EN**: Implements logic around `payloadUsesValueFromOperand`, `push_back`, `canOpOperandsBeDropped`, `pop_back`.
- **CN**: 围绕 `payloadUsesValueFromOperand`, `push_back`, `canOpOperandsBeDropped`, `pop_back` 实现具体逻辑。

### Lines 82-91
```cpp
    // Check if this operand is a duplicate.
    AffineMap indexingMap = genericOp.getMatchingIndexingMap(inputOpOperand);
    auto it =
        dedupedInputs.find(std::make_pair(inputOpOperand->get(), indexingMap));
    if (it != dedupedInputs.end()) {
      origToNewPos[en.index()] = it->second;
      droppedOpOperands.push_back(inputOpOperand);
      continue;
    }

```
- **EN**: Implements logic around `getMatchingIndexingMap`, `find`, `end`, `index`, and 1 more symbols.
- **CN**: 围绕 `getMatchingIndexingMap`, `find`, `end`, `index`, and 1 more symbols 实现具体逻辑。

### Lines 92-101
```cpp
    // This is a preserved argument.
    origToNewPos[en.index()] = newInputOperands.size();
    dedupedInputs[{inputOpOperand->get(), indexingMap}] =
        newInputOperands.size();
    newInputOperands.push_back(inputOpOperand->get());
    newIndexingMaps.push_back(indexingMap);
  }
  return origToNewPos;
}

```
- **EN**: Implements logic around `index`, `get`, `size`, `push_back`.
- **CN**: 围绕 `index`, `get`, `size`, `push_back` 实现具体逻辑。

### Lines 102-121
```cpp
// Deduplicate output operands, and return the
// - Mapping from operand position in the original op, to operand position in
// the canonicalized op.
// - The preserved output operands list (by reference).
llvm::SmallDenseMap<unsigned, unsigned> static deduplicateOutputOperands(
    GenericOp genericOp, SmallVector<OpOperand *> &droppedOpOperands,
    SmallVector<Value> &newOutputOperands,
    SmallVector<AffineMap> &newIndexingMaps, bool removeOutputs) {
  llvm::SmallDenseMap<unsigned, unsigned> origToNewPos;
  llvm::SmallDenseMap<std::tuple<Value, AffineMap, Value>, unsigned>
      dedupedOutpts;
  // If the op doesn't have tensor semantics or outputs should not be removed,
  // keep all the outputs as preserved.
  if (!genericOp.hasPureTensorSemantics() || !removeOutputs) {
    for (const auto &en : llvm::enumerate(genericOp.getDpsInitsMutable())) {
      origToNewPos[en.index()] = newOutputOperands.size();
      newOutputOperands.push_back(en.value().get());
      newIndexingMaps.push_back(genericOp.getMatchingIndexingMap(&en.value()));
    }
    return origToNewPos;
```
- **EN**: Implements logic around `deduplicateOutputOperands`, `hasPureTensorSemantics`, `enumerate`, `index`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `deduplicateOutputOperands`, `hasPureTensorSemantics`, `enumerate`, `index`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 122-141
```cpp
  }
  // Output argument can be dropped if the result has
  // - no users, and
  // - it is not used in the payload, and
  // - the corresponding indexing maps are not needed for loop bound
  //   computation.
  auto yieldOp = cast<YieldOp>(genericOp.getBody()->getTerminator());
  for (const auto &outputOpOperand :
       llvm::enumerate(genericOp.getDpsInitsMutable())) {
    OpResult result = genericOp.getTiedOpResult(&outputOpOperand.value());
    AffineMap indexingMap =
        genericOp.getMatchingIndexingMap(&outputOpOperand.value());
    auto key = std::make_tuple(outputOpOperand.value().get(), indexingMap,
                               yieldOp->getOperand(outputOpOperand.index()));
    if (isResultValueDead(genericOp, result)) {
      // Check if the opoperand can be dropped without affecting loop
      // bound computation. Add the operand to the list of dropped op
      // operand for checking. If it cannot be dropped, need to pop the
      // value back.
      droppedOpOperands.push_back(&outputOpOperand.value());
```
- **EN**: Implements logic around `getBody`, `enumerate`, `getTiedOpResult`, `getMatchingIndexingMap`, and 4 more symbols.
- **CN**: 围绕 `getBody`, `enumerate`, `getTiedOpResult`, `getMatchingIndexingMap`, and 4 more symbols 实现具体逻辑。

### Lines 142-161
```cpp
      if (genericOp.canOpOperandsBeDropped(droppedOpOperands)) {
        continue;
      }
      droppedOpOperands.pop_back();
    }

    if (!genericOp.payloadUsesValueFromOperand(&outputOpOperand.value())) {
      // The out operand can also be dropped if it is computed redundantly
      // by another result, the conditions for that are
      // - The same operand is used as the out operand
      // - The same indexing map is used
      // - The same yield value is used.
      auto it = dedupedOutpts.find(key);
      if (it != dedupedOutpts.end()) {
        origToNewPos[outputOpOperand.index()] = it->second;
        droppedOpOperands.push_back(&outputOpOperand.value());
        continue;
      }
    }

```
- **EN**: Implements logic around `canOpOperandsBeDropped`, `pop_back`, `payloadUsesValueFromOperand`, `find`, and 3 more symbols.
- **CN**: 围绕 `canOpOperandsBeDropped`, `pop_back`, `payloadUsesValueFromOperand`, `find`, and 3 more symbols 实现具体逻辑。

### Lines 162-181
```cpp
    origToNewPos[outputOpOperand.index()] = newOutputOperands.size();
    dedupedOutpts[key] = newOutputOperands.size();
    newOutputOperands.push_back(outputOpOperand.value().get());
    newIndexingMaps.push_back(
        genericOp.getMatchingIndexingMap(&outputOpOperand.value()));
  }
  return origToNewPos;
}

// Populate the body of the canonicalized operation.
static void populateOpPayload(
    GenericOp genericOp, GenericOp newOp,
    const llvm::SmallDenseMap<unsigned, unsigned> &origInsToNewInsPos,
    const llvm::SmallDenseMap<unsigned, unsigned> &origOutsToNewOutsPos,
    RewriterBase &rewriter) {
  // Merge the body of the original op with the new op.
  Block *newOpBlock = &newOp.getRegion().front();
  assert(newOpBlock->empty() && "expected new op to have an empty payload");
  Block *origOpBlock = &genericOp.getRegion().front();
  SmallVector<Value> replacements(origOpBlock->getNumArguments(), nullptr);
```
- **EN**: Implements logic around `index`, `size`, `push_back`, `getMatchingIndexingMap`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `index`, `size`, `push_back`, `getMatchingIndexingMap`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 182-198
```cpp

  // Replace all arguments in the original op, with arguments from the
  // canonicalized op.
  auto updateReplacements =
      [&](SmallVector<OpOperand *> &origOperands,
          SmallVector<OpOperand *> &newOperands,
          const llvm::SmallDenseMap<unsigned, unsigned> &map) {
        for (const auto &origOperand : llvm::enumerate(origOperands)) {
          auto it = map.find(origOperand.index());
          if (it == map.end())
            continue;
          OpOperand *newOperand = newOperands[it->second];
          replacements[origOperand.value()->getOperandNumber()] =
              newOpBlock->getArgument(newOperand->getOperandNumber());
        }
      };

```
- **EN**: Implements logic around `enumerate`, `find`, `end`, `value`, and 1 more symbols.
- **CN**: 围绕 `enumerate`, `find`, `end`, `value`, and 1 more symbols 实现具体逻辑。

### Lines 199-209
```cpp
  SmallVector<OpOperand *> origInputOperands = genericOp.getDpsInputOperands();
  SmallVector<OpOperand *> newInputOperands = newOp.getDpsInputOperands();
  updateReplacements(origInputOperands, newInputOperands, origInsToNewInsPos);

  SmallVector<OpOperand *> origOutputOperands =
      llvm::to_vector(llvm::make_pointer_range(genericOp.getDpsInitsMutable()));
  SmallVector<OpOperand *> newOutputOperands =
      llvm::to_vector(llvm::make_pointer_range(newOp.getDpsInitsMutable()));
  updateReplacements(origOutputOperands, newOutputOperands,
                     origOutsToNewOutsPos);

```
- **EN**: Implements logic around `getDpsInputOperands`, `updateReplacements`, `to_vector`.
- **CN**: 围绕 `getDpsInputOperands`, `updateReplacements`, `to_vector` 实现具体逻辑。

### Lines 210-226
```cpp
  // Drop the unused yield args.
  if (newOp.getNumDpsInits() != genericOp.getNumDpsInits()) {
    OpBuilder::InsertionGuard g(rewriter);
    YieldOp origYieldOp = cast<YieldOp>(origOpBlock->getTerminator());
    rewriter.setInsertionPoint(origYieldOp);

    SmallVector<Value> newYieldVals(newOp.getNumDpsInits(), nullptr);
    for (const auto &yieldOpOperands :
         llvm::enumerate(origYieldOp.getValues())) {
      auto it = origOutsToNewOutsPos.find(yieldOpOperands.index());
      if (it == origOutsToNewOutsPos.end())
        continue;
      newYieldVals[it->second] = yieldOpOperands.value();
    }
    rewriter.replaceOpWithNewOp<YieldOp>(origYieldOp, newYieldVals);
  }

```
- **EN**: Implements logic around `getNumDpsInits`, `g`, `getTerminator`, `setInsertionPoint`, and 6 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getNumDpsInits`, `g`, `getTerminator`, `setInsertionPoint`, and 6 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 227-236
```cpp
  rewriter.mergeBlocks(origOpBlock, newOpBlock, replacements);
}

FailureOr<linalg::GenericOp>
mlir::linalg::deduplicateOperandsAndRemoveDeadResults(
    RewriterBase &rewriter, linalg::GenericOp genericOp, bool removeOutputs) {
  // Create a map from argument position in the original op to the argument
  // position in the new op. If the argument is dropped it wont have an entry.
  SmallVector<OpOperand *> droppedOpOperands;

```
- **EN**: Implements logic around `mergeBlocks`, `deduplicateOperandsAndRemoveDeadResults`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `mergeBlocks`, `deduplicateOperandsAndRemoveDeadResults` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 237-250
```cpp
  // Information needed to build the new op.
  SmallVector<Value> newInputOperands, newOutputOperands;
  SmallVector<AffineMap> newIndexingMaps;

  // Gather information about duplicate input operands.
  llvm::SmallDenseMap<unsigned, unsigned> origInsToNewInsPos =
      deduplicateInputOperands(genericOp, droppedOpOperands, newInputOperands,
                               newIndexingMaps);

  // Gather information about the dropped outputs.
  llvm::SmallDenseMap<unsigned, unsigned> origOutsToNewOutsPos =
      deduplicateOutputOperands(genericOp, droppedOpOperands, newOutputOperands,
                                newIndexingMaps, removeOutputs);

```
- **EN**: Implements logic around `deduplicateInputOperands`, `deduplicateOutputOperands`.
- **CN**: 围绕 `deduplicateInputOperands`, `deduplicateOutputOperands` 实现具体逻辑。

### Lines 251-270
```cpp
  // Check if there is any change to operands.
  if (newInputOperands.size() + newOutputOperands.size() ==
      genericOp->getNumOperands())
    return genericOp;

  // Create the new op with the body being empty.
  Location loc = genericOp.getLoc();
  SmallVector<Type> newResultTypes;
  for (Value v : newOutputOperands)
    if (isa<TensorType>(v.getType()))
      newResultTypes.push_back(v.getType());
  auto newOp = GenericOp::create(
      rewriter, loc, newResultTypes, newInputOperands, newOutputOperands,
      rewriter.getAffineMapArrayAttr(newIndexingMaps),
      genericOp.getIteratorTypes(), genericOp.getDocAttr(),
      genericOp.getLibraryCallAttr(),
      [](OpBuilder & /*builder*/, Location /*loc*/, ValueRange /*args*/) {
        return;
      });
  // Copy over unknown attributes. They might be load bearing for some flow.
```
- **EN**: Implements logic around `size`, `getNumOperands`, `getLoc`, `getType`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `size`, `getNumOperands`, `getLoc`, `getType`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 271-290
```cpp
  ArrayRef<StringRef> odsAttrs = genericOp.getAttributeNames();
  for (NamedAttribute kv : genericOp->getAttrs())
    if (!llvm::is_contained(odsAttrs, kv.getName().getValue()))
      newOp->setAttr(kv.getName(), kv.getValue());

  // Fix up the payload of the canonicalized operation.
  populateOpPayload(genericOp, newOp, origInsToNewInsPos, origOutsToNewOutsPos,
                    rewriter);

  // Replace all live uses of the op.
  SmallVector<Value> replacementsVals(genericOp->getNumResults(), nullptr);
  for (const auto &result : llvm::enumerate(genericOp.getResults())) {
    auto it = origOutsToNewOutsPos.find(result.index());
    if (it == origOutsToNewOutsPos.end())
      continue;
    replacementsVals[result.index()] = newOp.getResult(it->second);
  }
  rewriter.replaceOp(genericOp, replacementsVals);
  return newOp;
}
```
- **EN**: Implements logic around `getAttributeNames`, `getAttrs`, `is_contained`, `setAttr`, and 7 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getAttributeNames`, `getAttrs`, `is_contained`, `setAttr`, and 7 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 291-310
```cpp

namespace {

struct DeduplicateAndRemoveDeadOperandsAndResults
    : public OpRewritePattern<GenericOp> {
  DeduplicateAndRemoveDeadOperandsAndResults(MLIRContext *ctx,
                                             bool removeOutputs)
      : OpRewritePattern<GenericOp>(ctx), removeOutputs(removeOutputs) {}

  LogicalResult matchAndRewrite(GenericOp genericOp,
                                PatternRewriter &rewriter) const override {
    FailureOr<GenericOp> newOp = deduplicateOperandsAndRemoveDeadResults(
        rewriter, genericOp, removeOutputs);
    if (failed(newOp) || newOp.value() == genericOp) {
      return rewriter.notifyMatchFailure(
          genericOp, "failed to dedup operands/remove dead results");
    }
    return success();
  }

```
- **EN**: Introduces declarations for `DeduplicateAndRemoveDeadOperandsAndResults`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `DeduplicateAndRemoveDeadOperandsAndResults` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 311-320
```cpp
private:
  /// If unset, outputs are not modified by this pattern.
  bool removeOutputs;
};

/// Remove unused cycles.
/// We can remove unused cycle within a payload of generic region
/// if these conditions are met:
/// - Result from out operand is dead.
/// - Block arg from out operand has a single use in the %cycle
```
- **EN**: Transitions between access-control regions inside a class or struct definition.
- **CN**: 在类或结构体定义内部切换访问控制区域。

### Lines 321-332
```cpp
/// instruction.
/// - Cycle has a single use and it is in yield.
struct RemoveUnusedCycleInGenericOp : public OpRewritePattern<GenericOp> {
  using OpRewritePattern<GenericOp>::OpRewritePattern;

  LogicalResult matchAndRewrite(GenericOp genericOp,
                                PatternRewriter &rewriter) const override {

    // If the op doesnt have tensor semantics, preserve the outputs as is.
    if (!genericOp.hasPureTensorSemantics())
      return failure();

```
- **EN**: Introduces declarations for `RemoveUnusedCycleInGenericOp`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `RemoveUnusedCycleInGenericOp` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 333-342
```cpp
    bool hasRemovedCycles = false;
    // Iterate over output operands and remove any unused cycles.
    for (const auto &outputOpOperand :
         llvm::enumerate(genericOp.getDpsInits())) {

      // Check that result from out operand is dead.
      Value result = genericOp.getResult(outputOpOperand.index());
      if (!result.use_empty())
        continue;

```
- **EN**: Implements logic around `enumerate`, `getResult`, `use_empty`.
- **CN**: 围绕 `enumerate`, `getResult`, `use_empty` 实现具体逻辑。

### Lines 343-353
```cpp
      // Check that outputArg has one use in cycle.
      BlockArgument outputArg =
          genericOp.getRegionOutputArgs()[outputOpOperand.index()];
      if (!outputArg.hasOneUse())
        continue;

      // Check cycle has at most one use.
      Operation *cycleOp = *outputArg.user_begin();
      if (!cycleOp->hasOneUse())
        continue;

```
- **EN**: Implements logic around `getRegionOutputArgs`, `hasOneUse`, `user_begin`.
- **CN**: 围绕 `getRegionOutputArgs`, `hasOneUse`, `user_begin` 实现具体逻辑。

### Lines 354-363
```cpp
      // Check that the cycleUser is a yield.
      Operation *cycleUserOp = *cycleOp->user_begin();
      if (!isa<linalg::YieldOp>(cycleUserOp))
        continue;

      // Check that argIndex matches yieldIndex, else data is being used.
      if (cycleUserOp->getOperand(outputOpOperand.index()) !=
          cycleOp->getResult(0))
        continue;

```
- **EN**: Implements logic around `user_begin`, `YieldOp>`, `getOperand`, `getResult`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `user_begin`, `YieldOp>`, `getOperand`, `getResult` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 364-374
```cpp
      // Directly replace the cycle with the blockArg such that
      // Deduplicate pattern can eliminate it along with unused yield.
      rewriter.replaceOp(cycleOp, outputArg);
      rewriter.modifyOpInPlace(genericOp, [] {});
      hasRemovedCycles = true;
    }

    if (hasRemovedCycles) {
      return success();
    }

```
- **EN**: Implements logic around `replaceOp`, `modifyOpInPlace`, `success`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `replaceOp`, `modifyOpInPlace`, `success` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 375-384
```cpp
    return failure();
  }
};

/// Fold uses of duplicate inputs in the body of a linalg.generic. E.g.:
/// ```
/// linalg.generic ins(%a, %b, %a, %b) outs(%a)
/// ^bb0(%in0, %in1, %in2, %in3, %out1)
/// ```
/// Assuming that all %a and %b have the same index map:
```
- **EN**: Implements logic around `failure`; this block performs canonicalization or folding-oriented reasoning; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `failure` 实现具体逻辑；该代码块执行面向规范化或折叠的推理，并协调核心结构化 MLIR 方言之间的行为。

### Lines 385-404
```cpp
/// * All uses of %in0 and %in2 are replaced with %out1
/// * All uses of %in1 are replaced with %in3
/// This pattern can enable additional canonicalizations: In the above example,
/// %in0, %in1 and %in3 have no uses anymore and their corresponding operands
/// can be folded away. This pattern does not modify uses of output block args.
struct FoldDuplicateInputBbArgs : public OpRewritePattern<GenericOp> {
  using OpRewritePattern<GenericOp>::OpRewritePattern;

  LogicalResult matchAndRewrite(GenericOp genericOp,
                                PatternRewriter &rewriter) const override {
    // Find replacement bbArgs for all input bbArg.
    DenseMap<int, int> replacements;
    for (int i = 0; i < genericOp.getNumDpsInputs(); ++i) {
      // Skip bbArgs that have no uses.
      if (genericOp.getBody()->getArgument(i).getUses().empty())
        continue;
      // Find replacement bbArg. This can be an input or an output bbArg.
      for (int j = genericOp->getNumOperands() - 1; j > i; --j) {
        if (genericOp->getOperand(i) == genericOp->getOperand(j) &&
            genericOp.getIndexingMapsArray()[i] ==
```
- **EN**: Introduces declarations for `FoldDuplicateInputBbArgs`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `FoldDuplicateInputBbArgs` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 405-415
```cpp
                genericOp.getIndexingMapsArray()[j]) {
          replacements[i] = j;
          break;
        }
      }
    }

    // Stop here if no replacements were found.
    if (replacements.empty())
      return failure();

```
- **EN**: Implements logic around `getIndexingMapsArray`, `empty`, `failure`.
- **CN**: 围绕 `getIndexingMapsArray`, `empty`, `failure` 实现具体逻辑。

### Lines 416-428
```cpp
    // Rewrite the op.
    rewriter.modifyOpInPlace(genericOp, [&]() {
      for (auto [before, after] : replacements) {
        BlockArgument bbArg = genericOp.getBody()->getArgument(before);
        BlockArgument replacement = genericOp.getBody()->getArgument(after);
        rewriter.replaceAllUsesWith(bbArg, replacement);
      }
    });

    return success();
  }
};

```
- **EN**: Implements logic around `modifyOpInPlace`, `getBody`, `replaceAllUsesWith`, `success`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `modifyOpInPlace`, `getBody`, `replaceAllUsesWith`, `success` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 429-443
```cpp
} // namespace

void mlir::linalg::populateEraseUnusedOperandsAndResultsPatterns(
    RewritePatternSet &patterns) {
  patterns.insert<DeduplicateAndRemoveDeadOperandsAndResults>(
      patterns.getContext(), /*removeOutputs=*/true);
  patterns.insert<RemoveUnusedCycleInGenericOp>(patterns.getContext());
}

void mlir::linalg::populateEraseUnnecessaryInputsPatterns(
    RewritePatternSet &patterns) {
  patterns.insert<DeduplicateAndRemoveDeadOperandsAndResults>(
      patterns.getContext(), /*removeOutputs=*/false);
  patterns.insert<FoldDuplicateInputBbArgs>(patterns.getContext());
}
```
- **EN**: Implements logic around `populateEraseUnusedOperandsAndResultsPatterns`, `insert`, `getContext`, `populateEraseUnnecessaryInputsPatterns`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `populateEraseUnusedOperandsAndResultsPatterns`, `insert`, `getContext`, `populateEraseUnnecessaryInputsPatterns` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

## Key Concepts / 关键概念

- **Rewrite and lowering / 重写与 lowering**:
  - **EN**: Uses MLIR pattern infrastructure or passes to canonicalize, legalize, or lower operations.
  - **CN**: 使用 MLIR 模式基础设施或 Pass 对操作进行规范化、合法化或 lowering。
- **Pattern rewriting / 模式重写**:
  - **EN**: Uses MLIR rewrite patterns to match operations and replace them with improved forms.
  - **CN**: 使用 MLIR 重写模式匹配操作，并将其替换为更合适的形式。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/Dialect/Linalg/IR/Linalg.h`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (2)
