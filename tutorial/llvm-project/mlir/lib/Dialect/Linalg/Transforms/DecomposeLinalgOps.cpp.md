# DecomposeLinalgOps.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Linalg/Transforms/DecomposeLinalgOps.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements Linalg dialect support for rewrite patterns, passes, and IR-to-IR transformation logic, centered on `DecomposeLinalgOps`.
  - **CN**: 实现 Linalg 方言中围绕 `DecomposeLinalgOps` 的重写模式、Pass 以及 IR 到 IR 的变换逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10
```cpp
//===- DecomposeLinalgOps.cpp - Pattern to break up Linalg ops ------------===//
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

### Lines 11-20
```cpp
#include "mlir/Dialect/Affine/IR/AffineOps.h"
#include "mlir/Dialect/Linalg/IR/Linalg.h"
#include "llvm/ADT/SmallVectorExtras.h"
#include <optional>

using namespace mlir;
using namespace mlir::linalg;

namespace {

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `llvm/ADT/SmallVectorExtras.h`, `optional`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `llvm/ADT/SmallVectorExtras.h`, `optional`。

### Lines 21-30
```cpp
/// Pattern to decompose a GenericOp that has more than two statements
/// into one GenericOp with the first statement (i.e. peeled operation), and
/// a second GenericOp with the remaining statements (i.e. residual operations).

/// - The result of the first GenericOp has the same shape as the iteration
///   space of the GenericOp. The body of the op yields as many values as the
///   original op plus all the results of the peeled operation.
/// - The second GenericOp has as many operands as the original operation plus
/// all the results of the first Generic Op. It has the same number of yields as
/// the original op.
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 31-40
```cpp
/// - If the result of the peeled operation was yielded by the original
///   GenericOp the uses of the corresponding results will be replaced with the
///   result of the first GenericOp created.
///
///  Example
///
/// ```mlir
///  %result:2 = linalg.generic ... ins(%arg0, %arg1, %arg2 : ...)
///      outs(%init0, %init1 : ...) {
///    ^bb0(%b0: ... , %b1: ... , %b2: ... , %b3: ..., %b4: ...):
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 41-50
```cpp
///      %0 = <s0> %b0, %b1 : ...
///      %1 = <s1> %0, %b2 : ...
///      linalg.yield %0, %1 : ...
///  } -> (..., ...)
///  return %result#0, %result#1
/// ```
///
/// gets split into
///
/// ```mlir
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 51-60
```cpp
/// %init = tensor.empty ...
/// %op0:3 = linalg.generic ... ins(%arg0, %arg1, %arg2 : ...)
///      outs(%init0, %init1, %init : ...)
///    ^bb0(%b0: ... , %b1: ... , %b2: ... , %b3: ..., %b4: ..., %b5: ...):
///      %0 = <s0> %b0, %b1 : ...
///      linalg.yield %0, %..., %0 : ...
///  } -> (..., ..., ...)
/// %op1:2 = linalg.generic ... ins(%arg0, %arg1, %arg2, %op0#2 : ...)
///      outs(%init0, %init1 : ...) {
///    ^bb0(%b0: ... , %b1: ... , %b2: ... , %b3: ..., %b4: ..., %b5: ...):
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 61-70
```cpp
///      %1 = <s1> %b3, %b2 : ...
///      linalg.yield %..., %1 : ...
///  } -> (..., ...)
///  return %op0#0, %op1#1
/// ```
///
/// After canonicalization this is expected to be
///
/// ```mlir
/// %init = tensor.empty ...
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 71-80
```cpp
/// %op0 = linalg.generic ... ins(%arg0, %arg1, : ...)
///      outs(%init : ...)
///    ^bb0(%b0: ... , %b1: ... , %b2: ...):
///      %0 = <s0> %b0, %b1 : ...
///      linalg.yield %0 : ...
///  } -> ...
/// %op1 = linalg.generic ... ins(%arg2, %op0#2 : ...)
///      outs(%init1 : ...) {
///    ^bb0(%b0: ... , %b1: ... , %b2: ...):
///      %1 = <s1> %b1, %b0 : ...
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 81-90
```cpp
///      linalg.yield %..., %1 : ...
///  } -> ...
///  return %op0, %op1
/// ```
struct DecomposeLinalgOp : public OpRewritePattern<GenericOp> {
  using OpRewritePattern<GenericOp>::OpRewritePattern;

  LogicalResult matchAndRewrite(GenericOp genericOp,
                                PatternRewriter &rewriter) const override;

```
- **EN**: Introduces declarations for `DecomposeLinalgOp`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `DecomposeLinalgOp` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 91-104
```cpp
private:
  /// Helper method to create a generic op for the peeled scalar operation. The
  /// created op has an empty region.
  GenericOp createPeeledGenericOp(GenericOp genericOp,
                                  PatternRewriter &rewriter) const;

  /// Helper method to create a generic op for the residual scalar operation.
  /// The created op has the same region as the original op.
  GenericOp createResidualGenericOp(GenericOp genericOp,
                                    GenericOp peeledGenericOp,
                                    PatternRewriter &rewriter) const;
};
} // namespace

```
- **EN**: Implements logic around `createPeeledGenericOp`, `createResidualGenericOp`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `createPeeledGenericOp`, `createResidualGenericOp` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 105-118
```cpp
/// Helper method to compute the range of a generic op.
static SmallVector<OpFoldResult> getGenericOpLoopRange(OpBuilder &b,
                                                       GenericOp op) {
  OpBuilder::InsertionGuard g(b);
  b.setInsertionPoint(op);
  Location loc = op.getLoc();
  auto allShapesSizes =
      cast<LinalgOp>(op.getOperation()).createFlatListOfOperandDims(b, loc);
  AffineMap map = op.getShapesToLoopsMap();
  IRRewriter rewriter(b);
  return affine::makeComposedFoldedMultiResultAffineApply(rewriter, loc, map,
                                                          allShapesSizes);
}

```
- **EN**: Implements logic around `getGenericOpLoopRange`, `g`, `setInsertionPoint`, `getLoc`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getGenericOpLoopRange`, `g`, `setInsertionPoint`, `getLoc`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 119-132
```cpp
/// Get zero value for an element type.
static Value getZero(OpBuilder &b, Location loc, Type elementType) {
  assert(elementType.isIntOrIndexOrFloat() &&
         "expected scalar type while computing zero value");
  if (isa<IntegerType>(elementType))
    return arith::ConstantIntOp::create(b, loc, elementType, 0);
  if (elementType.isIndex())
    return arith::ConstantIndexOp::create(b, loc, 0);
  // Assume float.
  auto floatType = cast<FloatType>(elementType);
  return arith::ConstantFloatOp::create(
      b, loc, floatType, APFloat::getZero(floatType.getFloatSemantics()));
}

```
- **EN**: Implements logic around `getZero`, `assert`, `create`, `isIndex`.
- **CN**: 围绕 `getZero`, `assert`, `create`, `isIndex` 实现具体逻辑。

### Lines 133-142
```cpp
GenericOp
DecomposeLinalgOp::createPeeledGenericOp(GenericOp genericOp,
                                         PatternRewriter &rewriter) const {
  Block *body = genericOp.getBody();
  Operation *peeledScalarOperation = &(*body->begin());
  SmallVector<AffineMap> peeledGenericOpIndexingMaps =
      genericOp.getIndexingMapsArray();

  /// Compute the loop ranges for operation. This is the shape of the result of
  /// the generic op for the peeled operation.
```
- **EN**: Implements logic around `createPeeledGenericOp`, `getBody`, `begin`, `getIndexingMapsArray`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `createPeeledGenericOp`, `getBody`, `begin`, `getIndexingMapsArray` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 143-152
```cpp
  Location loc = genericOp.getLoc();
  SmallVector<OpFoldResult> domain = getGenericOpLoopRange(rewriter, genericOp);
  SmallVector<Value> newInitValues;
  SmallVector<Type> newResultTypes;

  // Add as many new results as the number of results of the peeled scalar op.
  for (auto scalarOpResult : peeledScalarOperation->getResults()) {
    // If the result is yielded by the original op, use the operand, indexing
    // map and result type that correspond to the yielded value.

```
- **EN**: Implements logic around `getLoc`, `getGenericOpLoopRange`, `getResults`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getLoc`, `getGenericOpLoopRange`, `getResults` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 153-172
```cpp
    std::optional<unsigned> resultNumber;
    for (auto *user : scalarOpResult.getUsers()) {
      if (auto yieldOp = dyn_cast<YieldOp>(user)) {
        // Find the first use of the `scalarOpResult` in the yield op.
        for (OpOperand &yieldOperand : yieldOp->getOpOperands()) {
          if (yieldOperand.get() == scalarOpResult) {
            resultNumber = yieldOperand.getOperandNumber();
            break;
          }
        }
        assert(resultNumber && "unable to find use of a value in its user");
        break;
      }
    }
    if (resultNumber) {
      newInitValues.push_back(
          genericOp.getDpsInitOperand(*resultNumber)->get());
      OpResult result = cast<OpResult>(genericOp.getResult(*resultNumber));
      newResultTypes.push_back(result.getType());
      peeledGenericOpIndexingMaps.push_back(
```
- **EN**: Implements logic around `getUsers`, `getOpOperands`, `get`, `getOperandNumber`, and 4 more symbols.
- **CN**: 围绕 `getUsers`, `getOpOperands`, `get`, `getOperandNumber`, and 4 more symbols 实现具体逻辑。

### Lines 173-185
```cpp
          genericOp.getIndexingMapMatchingResult(result));
      continue;
    }

    // Fall back path, use an `init_tensor` and identity indexing map.
    AffineMap indexingMap = rewriter.getMultiDimIdentityMap(domain.size());
    Value emptyTensor = tensor::EmptyOp::create(rewriter, loc, domain,
                                                scalarOpResult.getType());
    newInitValues.push_back(emptyTensor);
    newResultTypes.push_back(emptyTensor.getType());
    peeledGenericOpIndexingMaps.push_back(indexingMap);
  }

```
- **EN**: Implements logic around `getIndexingMapMatchingResult`, `getMultiDimIdentityMap`, `create`, `getType`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getIndexingMapMatchingResult`, `getMultiDimIdentityMap`, `create`, `getType`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 186-198
```cpp
  /// Create the peeled generic op with an empty body.
  SmallVector<Value> outsOperands = genericOp.getOutputs();
  outsOperands.append(newInitValues.begin(), newInitValues.end());
  SmallVector<Type> resultTypes = llvm::to_vector(genericOp.getResultTypes());
  resultTypes.append(newResultTypes.begin(), newResultTypes.end());
  auto indexingMapAttr =
      rewriter.getAffineMapArrayAttr(peeledGenericOpIndexingMaps);
  return GenericOp::create(
      rewriter, loc, resultTypes, genericOp.getInputs(), outsOperands,
      indexingMapAttr, genericOp.getIteratorTypes(), /*doc=*/nullptr,
      /*libraryCall=*/nullptr, [](OpBuilder, Location, ValueRange) {});
}

```
- **EN**: Implements logic around `getOutputs`, `append`, `to_vector`, `getAffineMapArrayAttr`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getOutputs`, `append`, `to_vector`, `getAffineMapArrayAttr`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 199-213
```cpp
GenericOp
DecomposeLinalgOp::createResidualGenericOp(GenericOp genericOp,
                                           GenericOp peeledGenericOp,
                                           PatternRewriter &rewriter) const {
  /// Append all results from the peeledGenericOps as `ins` operand for the
  /// residual generic op.
  SmallVector<Value> residualGenericOpOperands = genericOp.getInputs();
  unsigned origNumResults = genericOp.getNumResults();
  unsigned peeledGenericOpNumResults = peeledGenericOp.getNumResults();
  SmallVector<Value> extraIns;
  for (auto resultNum :
       llvm::seq<unsigned>(origNumResults, peeledGenericOpNumResults))
    extraIns.push_back(peeledGenericOp->getResult(resultNum));
  residualGenericOpOperands.append(extraIns);

```
- **EN**: Implements logic around `createResidualGenericOp`, `getInputs`, `getNumResults`, `seq`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `createResidualGenericOp`, `getInputs`, `getNumResults`, `seq`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 214-228
```cpp
  /// Add indexing maps for the newly added operands. Use the same map
  /// as those used for the new results of the peeledGenericOp.
  auto indexingMaps = llvm::map_to_vector(
      genericOp.getDpsInputOperands(), [&](OpOperand *operand) {
        return genericOp.getMatchingIndexingMap(operand);
      });
  for (auto resultNum :
       llvm::seq<unsigned>(origNumResults, peeledGenericOpNumResults)) {
    OpResult result = cast<OpResult>(peeledGenericOp.getResult(resultNum));
    indexingMaps.push_back(
        peeledGenericOp.getIndexingMapMatchingResult(result));
  }
  for (OpOperand &outOperand : genericOp.getDpsInitsMutable())
    indexingMaps.push_back(genericOp.getMatchingIndexingMap(&outOperand));

```
- **EN**: Implements logic around `map_to_vector`, `getDpsInputOperands`, `getMatchingIndexingMap`, `seq`, and 4 more symbols.
- **CN**: 围绕 `map_to_vector`, `getDpsInputOperands`, `getMatchingIndexingMap`, `seq`, and 4 more symbols 实现具体逻辑。

### Lines 229-240
```cpp
  auto indexingMapAttr = rewriter.getAffineMapArrayAttr(indexingMaps);
  return GenericOp::create(
      rewriter, genericOp->getLoc(), genericOp->getResultTypes(),
      residualGenericOpOperands, genericOp.getOutputs(), indexingMapAttr,
      genericOp.getIteratorTypes(), /*doc=*/nullptr, /*libraryCall=*/nullptr,
      [](OpBuilder, Location, ValueRange) {});
}

LogicalResult
DecomposeLinalgOp::matchAndRewrite(GenericOp genericOp,
                                   PatternRewriter &rewriter) const {
  /// For now only match on operations where the iterator types are all parallel
```
- **EN**: Implements logic around `getAffineMapArrayAttr`, `create`, `getLoc`, `getOutputs`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getAffineMapArrayAttr`, `create`, `getLoc`, `getOutputs`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 241-253
```cpp
  if (genericOp.getNumParallelLoops() != genericOp.getNumLoops()) {
    return rewriter.notifyMatchFailure(genericOp,
                                       "unhandled decomposition of operation "
                                       "with non-parallel iterator types");
  }
  // TODO: this could be generalized to handle `linalg.generic` with buffer
  // operands too but requires allocation for intermediates. Punt on this for
  // now.
  if (!genericOp.hasPureTensorSemantics()) {
    return rewriter.notifyMatchFailure(
        genericOp, "only operations with tensor semantics are handled");
  }

```
- **EN**: Implements logic around `getNumParallelLoops`, `notifyMatchFailure`, `hasPureTensorSemantics`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getNumParallelLoops`, `notifyMatchFailure`, `hasPureTensorSemantics` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 254-268
```cpp
  if (llvm::any_of(genericOp.getDpsInitsMutable(), [&](OpOperand &outOperand) {
        return !genericOp.getMatchingIndexingMap(&outOperand).isPermutation();
      })) {
    return rewriter.notifyMatchFailure(
        genericOp, "unhandled decomposition of generic op with out operand not "
                   "accessed using a permutation");
  }

  /// If the op has only a single statement (apart from the yield), do nothing.
  Block *body = genericOp.getBody();
  if (body->getOperations().size() <= 2) {
    return rewriter.notifyMatchFailure(genericOp,
                                       "operation has less than 3 statements");
  }

```
- **EN**: Implements logic around `any_of`, `getMatchingIndexingMap`, `notifyMatchFailure`, `getBody`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `any_of`, `getMatchingIndexingMap`, `notifyMatchFailure`, `getBody`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 269-280
```cpp
  /// Check that the peeled statement has a scalar element type.
  if (llvm::any_of(body->getOperations().begin()->getResultTypes(),
                   [](Type t) { return !t.isIntOrIndexOrFloat(); })) {
    return rewriter.notifyMatchFailure(
        &(*body->getOperations().begin()),
        "expected return type to be only int, index or float");
  }

  GenericOp peeledGenericOp = createPeeledGenericOp(genericOp, rewriter);
  GenericOp residualGenericOp =
      createResidualGenericOp(genericOp, peeledGenericOp, rewriter);

```
- **EN**: Implements logic around `any_of`, `isIntOrIndexOrFloat`, `notifyMatchFailure`, `getOperations`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `any_of`, `isIntOrIndexOrFloat`, `notifyMatchFailure`, `getOperations`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 281-291
```cpp
  /// Move the first statement of the original operation into the body of the
  /// generic op for the peeled operation.
  Block *peeledGenericOpBody = peeledGenericOp.getBody();
  Block *residualGenericOpBody = residualGenericOp.getBody();
  assert(peeledGenericOpBody->empty() && residualGenericOpBody->empty() &&
         "expected split generic ops to have empty region");
  peeledGenericOpBody->getOperations().splice(
      peeledGenericOpBody->begin(), body->getOperations(), body->begin());
  residualGenericOpBody->getOperations().splice(residualGenericOpBody->begin(),
                                                body->getOperations());

```
- **EN**: Implements logic around `getBody`, `assert`, `getOperations`, `begin`.
- **CN**: 围绕 `getBody`, `assert`, `getOperations`, `begin` 实现具体逻辑。

### Lines 292-311
```cpp
  Operation *peeledScalarOperation = &(*peeledGenericOpBody->begin());
  auto *yieldOp = residualGenericOpBody->getTerminator();
  {
    // Yield all the result of the peeled scalar operation.
    OpBuilder::InsertionGuard g(rewriter);
    rewriter.setInsertionPointToEnd(peeledGenericOpBody);
    SmallVector<Value> yieldedVals;
    for (auto origYield : yieldOp->getOperands()) {
      if (origYield.getDefiningOp() == peeledScalarOperation) {
        yieldedVals.push_back(origYield);
      } else {
        // Do not materialize any new ops inside of the decomposed LinalgOp,
        // as that would trigger another application of the rewrite pattern
        // (infinite loop).
        OpBuilder::InsertionGuard g(rewriter);
        rewriter.setInsertionPoint(peeledGenericOp);
        yieldedVals.push_back(
            getZero(rewriter, genericOp.getLoc(), origYield.getType()));
      }
    }
```
- **EN**: Implements logic around `begin`, `getTerminator`, `g`, `setInsertionPointToEnd`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `begin`, `getTerminator`, `g`, `setInsertionPointToEnd`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 312-329
```cpp
    yieldedVals.append(
        llvm::map_to_vector(peeledScalarOperation->getResults(),
                            [](OpResult opr) -> Value { return opr; }));
    YieldOp::create(rewriter, genericOp.getLoc(), yieldedVals);
  }

  /// In the split operations, replace block arguments uses that refer to
  /// original operation to the block arguments of the newly created operation.
  unsigned origNumInputs = genericOp.getNumDpsInputs();
  for (const auto &inputBlockArg :
       llvm::enumerate(genericOp.getBody()->getArguments())) {
    Value residualOpReplacementArg =
        residualGenericOpBody->getArgument(inputBlockArg.index());
    rewriter.replaceUsesWithIf(
        inputBlockArg.value(), residualOpReplacementArg, [&](OpOperand &use) {
          return use.getOwner()->getBlock() == residualGenericOpBody;
        });

```
- **EN**: Implements logic around `append`, `map_to_vector`, `create`, `getNumDpsInputs`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `append`, `map_to_vector`, `create`, `getNumDpsInputs`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 330-339
```cpp
    Value peeledOpReplacementArg =
        peeledGenericOpBody->getArgument(inputBlockArg.index());
    rewriter.replaceUsesWithIf(
        inputBlockArg.value(), peeledOpReplacementArg, [&](OpOperand &use) {
          return use.getOwner()->getBlock() == peeledGenericOpBody;
        });
  }

  /// Before fixing up the residual operation, track what values are yielded. If
  /// any of those are from the peeled scalar operation, the uses of the
```
- **EN**: Implements logic around `getArgument`, `replaceUsesWithIf`, `value`, `getOwner`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getArgument`, `replaceUsesWithIf`, `value`, `getOwner` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 340-350
```cpp
  /// corresponding result have to be remapped to result of the generic op for
  /// the peeled operation.
  SmallVector<Value> replacements;
  for (const auto &yieldValue : llvm::enumerate(yieldOp->getOperands())) {
    OpResult opr = dyn_cast<OpResult>(yieldValue.value());
    if (!opr || opr.getOwner() != peeledScalarOperation)
      replacements.push_back(residualGenericOp.getResult(yieldValue.index()));
    else
      replacements.push_back(peeledGenericOp->getResult(yieldValue.index()));
  }

```
- **EN**: Implements logic around `enumerate`, `value`, `getOwner`, `push_back`.
- **CN**: 围绕 `enumerate`, `value`, `getOwner`, `push_back` 实现具体逻辑。

### Lines 351-366
```cpp
  /// Update all uses of the peeled scalar operation results in the residual op
  /// to the newly added arguments.
  {
    SmallVector<Value> scalarReplacements;
    unsigned peeledScalarOpNumResults = peeledScalarOperation->getNumResults();
    scalarReplacements.reserve(peeledScalarOpNumResults);
    for (auto num : llvm::seq<unsigned>(0, peeledScalarOpNumResults))
      scalarReplacements.push_back(
          residualGenericOpBody->getArgument(num + origNumInputs));
    bool allUsesReplaced = false;
    rewriter.replaceOpUsesWithinBlock(peeledScalarOperation, scalarReplacements,
                                      residualGenericOpBody, &allUsesReplaced);
    assert(!allUsesReplaced &&
           "peeled scalar operation is erased when it wasnt expected to be");
  }

```
- **EN**: Implements logic around `getNumResults`, `reserve`, `seq`, `push_back`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getNumResults`, `reserve`, `seq`, `push_back`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 367-378
```cpp
  // Replace the original operation
  rewriter.replaceOp(genericOp, replacements);
  return success();
}

void mlir::linalg::populateDecomposeLinalgOpsPattern(
    RewritePatternSet &patterns, bool removeDeadArgsAndResults) {
  patterns.insert<DecomposeLinalgOp>(patterns.getContext());
  // Add the patterns to clean up the dead operands and results.
  if (removeDeadArgsAndResults)
    populateEraseUnusedOperandsAndResultsPatterns(patterns);
}
```
- **EN**: Implements logic around `replaceOp`, `success`, `populateDecomposeLinalgOpsPattern`, `insert`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `replaceOp`, `success`, `populateDecomposeLinalgOpsPattern`, `insert`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `llvm/ADT/SmallVectorExtras.h`
- **Standard-library headers / 标准库头文件**: `<optional>`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (3), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (1)
