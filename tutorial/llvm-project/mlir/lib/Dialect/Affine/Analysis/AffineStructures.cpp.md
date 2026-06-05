# AffineStructures.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Affine/Analysis/AffineStructures.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Structures for affine/polyhedral analysis of affine dialect ops.
  - **CN**: 实现 Affine 方言与仿射循环推理 的分析例程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===- AffineStructures.cpp - MLIR Affine Structures Class-----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Structures for affine/polyhedral analysis of affine dialect ops.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 12-25
```cpp

#include "mlir/Dialect/Affine/Analysis/AffineStructures.h"
#include "mlir/Analysis/Presburger/IntegerRelation.h"
#include "mlir/Analysis/Presburger/Utils.h"
#include "mlir/Dialect/Affine/IR/AffineOps.h"
#include "mlir/Dialect/Affine/IR/AffineValueMap.h"
#include "mlir/Dialect/Utils/StaticValueUtils.h"
#include "mlir/IR/IntegerSet.h"
#include "mlir/Support/LLVM.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"
#include <optional>

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Affine/Analysis/AffineStructures.h`, `mlir/Analysis/Presburger/IntegerRelation.h`, `mlir/Analysis/Presburger/Utils.h`, `mlir/Dialect/Affine/IR/AffineOps.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Affine/Analysis/AffineStructures.h`, `mlir/Analysis/Presburger/IntegerRelation.h`, `mlir/Analysis/Presburger/Utils.h`, `mlir/Dialect/Affine/IR/AffineOps.h`。

### Lines 26-36
```cpp
#define DEBUG_TYPE "affine-structures"

using namespace mlir;
using namespace affine;
using namespace presburger;

LogicalResult
FlatAffineValueConstraints::addInductionVarOrTerminalSymbol(Value val) {
  if (containsVar(val))
    return success();

```
- **EN**: Implements logic around `addInductionVarOrTerminalSymbol`, `containsVar`, `success`; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `addInductionVarOrTerminalSymbol`, `containsVar`, `success` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 37-54
```cpp
  // Caller is expected to fully compose map/operands if necessary.
  if (val.getDefiningOp<affine::AffineApplyOp>() ||
      (!isValidSymbol(val) && !isAffineInductionVar(val))) {
    LLVM_DEBUG(llvm::dbgs()
               << "only valid terminal symbols and affine IVs supported\n");
    return failure();
  }
  // Outer loop IVs could be used in forOp's bounds.
  if (auto loop = getForInductionVarOwner(val)) {
    appendDimVar(val);
    if (failed(this->addAffineForOpDomain(loop))) {
      LLVM_DEBUG(
          loop.emitWarning("failed to add domain info to constraint system"));
      return failure();
    }
    return success();
  }

```
- **EN**: Implements logic around `AffineApplyOp>`, `isValidSymbol`, `dbgs`, `failure`, and 5 more symbols; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `AffineApplyOp>`, `isValidSymbol`, `dbgs`, `failure`, and 5 more symbols 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 55-64
```cpp
  if (auto parallel = getAffineParallelInductionVarOwner(val)) {
    appendDimVar(parallel.getIVs());
    if (failed(this->addAffineParallelOpDomain(parallel))) {
      LLVM_DEBUG(parallel.emitWarning(
          "failed to add domain info to constraint system"));
      return failure();
    }
    return success();
  }

```
- **EN**: Implements logic around `getAffineParallelInductionVarOwner`, `appendDimVar`, `failed`, `emitWarning`, and 2 more symbols.
- **CN**: 围绕 `getAffineParallelInductionVarOwner`, `appendDimVar`, `failed`, `emitWarning`, and 2 more symbols 实现具体逻辑。

### Lines 65-81
```cpp
  // Add top level symbol.
  appendSymbolVar(val);
  // Check if the symbol is a constant.
  if (std::optional<int64_t> constOp = getConstantIntValue(val))
    addBound(BoundType::EQ, val, constOp.value());
  return success();
}

LogicalResult
FlatAffineValueConstraints::addAffineForOpDomain(AffineForOp forOp) {
  unsigned pos;
  // Pre-condition for this method.
  if (!findVar(forOp.getInductionVar(), &pos)) {
    assert(false && "Value not found");
    return failure();
  }

```
- **EN**: Implements logic around `appendSymbolVar`, `getConstantIntValue`, `addBound`, `success`, and 4 more symbols; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `appendSymbolVar`, `getConstantIntValue`, `addBound`, `success`, and 4 more symbols 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 82-101
```cpp
  int64_t step = forOp.getStepAsInt();
  if (step != 1) {
    if (!forOp.hasConstantLowerBound())
      LLVM_DEBUG(forOp.emitWarning("domain conservatively approximated"));
    else {
      // Add constraints for the stride.
      // (iv - lb) % step = 0 can be written as:
      // (iv - lb) - step * q = 0 where q = (iv - lb) / step.
      // Add local variable 'q' and add the above equality.
      // The first constraint is q = (iv - lb) floordiv step
      SmallVector<int64_t, 8> dividend(getNumCols(), 0);
      int64_t lb = forOp.getConstantLowerBound();
      dividend[pos] = 1;
      dividend.back() -= lb;
      unsigned qPos = addLocalFloorDiv(dividend, step);
      // Second constraint: (iv - lb) - step * q = 0.
      SmallVector<int64_t, 8> eq(getNumCols(), 0);
      eq[pos] = 1;
      eq.back() -= lb;
      // For the local var just added above.
```
- **EN**: Implements logic around `getStepAsInt`, `hasConstantLowerBound`, `emitWarning`, `dividend`, and 4 more symbols.
- **CN**: 围绕 `getStepAsInt`, `hasConstantLowerBound`, `emitWarning`, `dividend`, and 4 more symbols 实现具体逻辑。

### Lines 102-115
```cpp
      eq[qPos] = -step;
      addEquality(eq);
    }
  }

  if (forOp.hasConstantLowerBound()) {
    addBound(BoundType::LB, pos, forOp.getConstantLowerBound());
  } else {
    // Non-constant lower bound case.
    if (failed(addBound(BoundType::LB, pos, forOp.getLowerBoundMap(),
                        forOp.getLowerBoundOperands())))
      return failure();
  }

```
- **EN**: Implements logic around `addEquality`, `hasConstantLowerBound`, `addBound`, `failed`, and 2 more symbols.
- **CN**: 围绕 `addEquality`, `hasConstantLowerBound`, `addBound`, `failed`, and 2 more symbols 实现具体逻辑。

### Lines 116-134
```cpp
  if (forOp.hasConstantUpperBound()) {
    addBound(BoundType::UB, pos, forOp.getConstantUpperBound() - 1);
    return success();
  }
  // Non-constant upper bound case.
  return addBound(BoundType::UB, pos, forOp.getUpperBoundMap(),
                  forOp.getUpperBoundOperands());
}

LogicalResult FlatAffineValueConstraints::addAffineParallelOpDomain(
    AffineParallelOp parallelOp) {
  size_t ivPos = 0;
  for (Value iv : parallelOp.getIVs()) {
    unsigned pos;
    if (!findVar(iv, &pos)) {
      assert(false && "variable expected for the IV value");
      return failure();
    }

```
- **EN**: Implements logic around `hasConstantUpperBound`, `addBound`, `success`, `getUpperBoundOperands`, and 5 more symbols.
- **CN**: 围绕 `hasConstantUpperBound`, `addBound`, `success`, `getUpperBoundOperands`, and 5 more symbols 实现具体逻辑。

### Lines 135-152
```cpp
    AffineMap lowerBound = parallelOp.getLowerBoundMap(ivPos);
    if (lowerBound.isSingleConstant())
      addBound(BoundType::LB, pos, lowerBound.getSingleConstantResult());
    else if (failed(addBound(BoundType::LB, pos, lowerBound,
                             parallelOp.getLowerBoundsOperands())))
      return failure();

    auto upperBound = parallelOp.getUpperBoundMap(ivPos);
    if (upperBound.isSingleConstant())
      addBound(BoundType::UB, pos, upperBound.getSingleConstantResult() - 1);
    else if (failed(addBound(BoundType::UB, pos, upperBound,
                             parallelOp.getUpperBoundsOperands())))
      return failure();
    ++ivPos;
  }
  return success();
}

```
- **EN**: Implements logic around `getLowerBoundMap`, `isSingleConstant`, `addBound`, `failed`, and 5 more symbols; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `getLowerBoundMap`, `isSingleConstant`, `addBound`, `failed`, and 5 more symbols 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 153-165
```cpp
LogicalResult
FlatAffineValueConstraints::addDomainFromSliceMaps(ArrayRef<AffineMap> lbMaps,
                                                   ArrayRef<AffineMap> ubMaps,
                                                   ArrayRef<Value> operands) {
  assert(lbMaps.size() == ubMaps.size());
  assert(lbMaps.size() <= getNumDimVars());

  for (unsigned i = 0, e = lbMaps.size(); i < e; ++i) {
    AffineMap lbMap = lbMaps[i];
    AffineMap ubMap = ubMaps[i];
    assert(!lbMap || lbMap.getNumInputs() == operands.size());
    assert(!ubMap || ubMap.getNumInputs() == operands.size());

```
- **EN**: Implements logic around `addDomainFromSliceMaps`, `assert`, `size`; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `addDomainFromSliceMaps`, `assert`, `size` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 166-181
```cpp
    // Check if this slice is just an equality along this dimension. If so,
    // retrieve the existing loop it equates to and add it to the system.
    if (lbMap && ubMap && lbMap.getNumResults() == 1 &&
        ubMap.getNumResults() == 1 &&
        lbMap.getResult(0) + 1 == ubMap.getResult(0) &&
        // The condition above will be true for maps describing a single
        // iteration (e.g., lbMap.getResult(0) = 0, ubMap.getResult(0) = 1).
        // Make sure we skip those cases by checking that the lb result is not
        // just a constant.
        !isa<AffineConstantExpr>(lbMap.getResult(0))) {
      // Limited support: we expect the lb result to be just a loop dimension.
      // Not supported otherwise for now.
      AffineDimExpr result = dyn_cast<AffineDimExpr>(lbMap.getResult(0));
      if (!result)
        return failure();

```
- **EN**: Implements logic around `getNumResults`, `getResult`, `isa`, `dyn_cast`, and 1 more symbols.
- **CN**: 围绕 `getNumResults`, `getResult`, `isa`, `dyn_cast`, and 1 more symbols 实现具体逻辑。

### Lines 182-191
```cpp
      AffineForOp loop =
          getForInductionVarOwner(operands[result.getPosition()]);
      if (!loop)
        return failure();

      if (failed(addAffineForOpDomain(loop)))
        return failure();
      continue;
    }

```
- **EN**: Implements logic around `getForInductionVarOwner`, `failure`, `failed`.
- **CN**: 围绕 `getForInductionVarOwner`, `failure`, `failed` 实现具体逻辑。

### Lines 192-202
```cpp
    // This slice refers to a loop that doesn't exist in the IR yet. Add its
    // bounds to the system assuming its dimension variable position is the
    // same as the position of the loop in the loop nest.
    if (lbMap && failed(addBound(BoundType::LB, i, lbMap, operands)))
      return failure();
    if (ubMap && failed(addBound(BoundType::UB, i, ubMap, operands)))
      return failure();
  }
  return success();
}

```
- **EN**: Implements logic around `failed`, `failure`, `success`.
- **CN**: 围绕 `failed`, `failure`, `success` 实现具体逻辑。

### Lines 203-212
```cpp
void FlatAffineValueConstraints::addAffineIfOpDomain(AffineIfOp ifOp) {
  IntegerSet set = ifOp.getIntegerSet();
  // Canonicalize set and operands to ensure unique values for
  // FlatAffineValueConstraints below and for early simplification.
  SmallVector<Value> operands(ifOp.getOperands());
  canonicalizeSetAndOperands(&set, &operands);

  // Create the base constraints from the integer set attached to ifOp.
  FlatAffineValueConstraints cst(set, operands);

```
- **EN**: Implements logic around `addAffineIfOpDomain`, `getIntegerSet`, `operands`, `canonicalizeSetAndOperands`, and 1 more symbols.
- **CN**: 围绕 `addAffineIfOpDomain`, `getIntegerSet`, `operands`, `canonicalizeSetAndOperands`, and 1 more symbols 实现具体逻辑。

### Lines 213-232
```cpp
  // Merge the constraints from ifOp to the current domain. We need first merge
  // and align the IDs from both constraints, and then append the constraints
  // from the ifOp into the current one.
  mergeAndAlignVarsWithOther(0, &cst);
  append(cst);
}

LogicalResult FlatAffineValueConstraints::addBound(BoundType type, unsigned pos,
                                                   AffineMap boundMap,
                                                   ValueRange boundOperands) {
  // Fully compose map and operands; canonicalize and simplify so that we
  // transitively get to terminal symbols or loop IVs.
  auto map = boundMap;
  SmallVector<Value, 4> operands(boundOperands.begin(), boundOperands.end());
  fullyComposeAffineMapAndOperands(&map, &operands);
  map = simplifyAffineMap(map);
  canonicalizeMapAndOperands(&map, &operands);
  for (Value operand : operands) {
    if (failed(addInductionVarOrTerminalSymbol(operand)))
      return failure();
```
- **EN**: Implements logic around `mergeAndAlignVarsWithOther`, `append`, `addBound`, `operands`, and 5 more symbols; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `mergeAndAlignVarsWithOther`, `append`, `addBound`, `operands`, and 5 more symbols 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 233-251
```cpp
  }
  return addBound(type, pos, computeAlignedMap(map, operands));
}

// Adds slice lower bounds represented by lower bounds in 'lbMaps' and upper
// bounds in 'ubMaps' to each value in `values' that appears in the constraint
// system. Note that both lower/upper bounds share the same operand list
// 'operands'.
// This function assumes 'values.size' == 'lbMaps.size' == 'ubMaps.size', and
// skips any null AffineMaps in 'lbMaps' or 'ubMaps'.
// Note that both lower/upper bounds use operands from 'operands'.
// Returns failure for unimplemented cases such as semi-affine expressions or
// expressions with mod/floordiv.
LogicalResult FlatAffineValueConstraints::addSliceBounds(
    ArrayRef<Value> values, ArrayRef<AffineMap> lbMaps,
    ArrayRef<AffineMap> ubMaps, ArrayRef<Value> operands) {
  assert(values.size() == lbMaps.size());
  assert(lbMaps.size() == ubMaps.size());

```
- **EN**: Implements logic around `addBound`, `addSliceBounds`, `assert`; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `addBound`, `addSliceBounds`, `assert` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 252-261
```cpp
  for (unsigned i = 0, e = lbMaps.size(); i < e; ++i) {
    unsigned pos;
    if (!findVar(values[i], &pos))
      continue;

    AffineMap lbMap = lbMaps[i];
    AffineMap ubMap = ubMaps[i];
    assert(!lbMap || lbMap.getNumInputs() == operands.size());
    assert(!ubMap || ubMap.getNumInputs() == operands.size());

```
- **EN**: Implements logic around `size`, `findVar`, `assert`; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `size`, `findVar`, `assert` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 262-281
```cpp
    // Check if this slice is just an equality along this dimension.
    if (lbMap && ubMap && lbMap.getNumResults() == 1 &&
        ubMap.getNumResults() == 1 &&
        lbMap.getResult(0) + 1 == ubMap.getResult(0)) {
      if (failed(addBound(BoundType::EQ, pos, lbMap, operands)))
        return failure();
      continue;
    }

    // If lower or upper bound maps are null or provide no results, it implies
    // that the source loop was not at all sliced, and the entire loop will be a
    // part of the slice.
    if (lbMap && lbMap.getNumResults() != 0 && ubMap &&
        ubMap.getNumResults() != 0) {
      if (failed(addBound(BoundType::LB, pos, lbMap, operands)))
        return failure();
      if (failed(addBound(BoundType::UB, pos, ubMap, operands)))
        return failure();
    } else {
      auto loop = getForInductionVarOwner(values[i]);
```
- **EN**: Implements logic around `getNumResults`, `getResult`, `failed`, `failure`, and 1 more symbols.
- **CN**: 围绕 `getNumResults`, `getResult`, `failed`, `failure`, and 1 more symbols 实现具体逻辑。

### Lines 282-294
```cpp
      if (failed(this->addAffineForOpDomain(loop)))
        return failure();
    }
  }
  return success();
}

LogicalResult
FlatAffineValueConstraints::composeMap(const AffineValueMap *vMap) {
  return composeMatchingMap(
      computeAlignedMap(vMap->getAffineMap(), vMap->getOperands()));
}

```
- **EN**: Implements logic around `failed`, `failure`, `success`, `composeMap`, and 2 more symbols.
- **CN**: 围绕 `failed`, `failure`, `success`, `composeMap`, and 2 more symbols 实现具体逻辑。

### Lines 295-304
```cpp
// Turn a symbol into a dimension.
static void turnSymbolIntoDim(FlatAffineValueConstraints *cst, Value value) {
  unsigned pos;
  if (cst->findVar(value, &pos) && pos >= cst->getNumDimVars() &&
      pos < cst->getNumDimAndSymbolVars()) {
    cst->swapVar(pos, cst->getNumDimVars());
    cst->setDimSymbolSeparation(cst->getNumSymbolVars() - 1);
  }
}

```
- **EN**: Implements logic around `turnSymbolIntoDim`, `findVar`, `getNumDimAndSymbolVars`, `swapVar`, and 1 more symbols; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `turnSymbolIntoDim`, `findVar`, `getNumDimAndSymbolVars`, `swapVar`, and 1 more symbols 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 305-318
```cpp
// Changes all symbol variables which are loop IVs to dim variables.
void FlatAffineValueConstraints::convertLoopIVSymbolsToDims() {
  // Gather all symbols which are loop IVs.
  SmallVector<Value, 4> loopIVs;
  for (unsigned i = getNumDimVars(), e = getNumDimAndSymbolVars(); i < e; i++) {
    if (hasValue(i) && getForInductionVarOwner(getValue(i)))
      loopIVs.push_back(getValue(i));
  }
  // Turn each symbol in 'loopIVs' into a dim variable.
  for (auto iv : loopIVs) {
    turnSymbolIntoDim(this, iv);
  }
}

```
- **EN**: Implements logic around `convertLoopIVSymbolsToDims`, `getNumDimVars`, `hasValue`, `push_back`, and 1 more symbols; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `convertLoopIVSymbolsToDims`, `getNumDimVars`, `hasValue`, `push_back`, and 1 more symbols 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 319-334
```cpp
void FlatAffineValueConstraints::getIneqAsAffineValueMap(
    unsigned pos, unsigned ineqPos, AffineValueMap &vmap,
    MLIRContext *context) const {
  unsigned numDims = getNumDimVars();
  unsigned numSyms = getNumSymbolVars();

  assert(pos < numDims && "invalid position");
  assert(ineqPos < getNumInequalities() && "invalid inequality position");

  // Get expressions for local vars.
  SmallVector<AffineExpr, 8> memo(getNumVars(), AffineExpr());
  if (failed(computeLocalVars(memo, context)))
    assert(false &&
           "one or more local exprs do not have an explicit representation");
  auto localExprs = ArrayRef<AffineExpr>(memo).take_back(getNumLocalVars());

```
- **EN**: Implements logic around `getIneqAsAffineValueMap`, `getNumDimVars`, `getNumSymbolVars`, `assert`, and 3 more symbols; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `getIneqAsAffineValueMap`, `getNumDimVars`, `getNumSymbolVars`, `assert`, and 3 more symbols 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 335-349
```cpp
  // Compute the AffineExpr lower/upper bound for this inequality.
  SmallVector<int64_t, 8> inequality = getInequality64(ineqPos);
  SmallVector<int64_t, 8> bound;
  bound.reserve(getNumCols() - 1);
  // Everything other than the coefficient at `pos`.
  bound.append(inequality.begin(), inequality.begin() + pos);
  bound.append(inequality.begin() + pos + 1, inequality.end());

  if (inequality[pos] > 0)
    // Lower bound.
    llvm::transform(bound, bound.begin(), std::negate<int64_t>());
  else
    // Upper bound (which is exclusive).
    bound.back() += 1;

```
- **EN**: Implements logic around `getInequality64`, `reserve`, `append`, `transform`, and 1 more symbols; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `getInequality64`, `reserve`, `append`, `transform`, and 1 more symbols 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 350-362
```cpp
  // Convert to AffineExpr (tree) form.
  auto boundExpr = getAffineExprFromFlatForm(bound, numDims - 1, numSyms,
                                             localExprs, context);

  // Get the values to bind to this affine expr (all dims and symbols).
  SmallVector<Value, 4> operands;
  getValues(0, pos, &operands);
  SmallVector<Value, 4> trailingOperands;
  getValues(pos + 1, getNumDimAndSymbolVars(), &trailingOperands);
  operands.append(trailingOperands.begin(), trailingOperands.end());
  vmap.reset(AffineMap::get(numDims - 1, numSyms, boundExpr), operands);
}

```
- **EN**: Implements logic around `getAffineExprFromFlatForm`, `getValues`, `append`, `reset`; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `getAffineExprFromFlatForm`, `getValues`, `append`, `reset` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 363-377
```cpp
FlatAffineValueConstraints FlatAffineRelation::getDomainSet() const {
  FlatAffineValueConstraints domain = *this;
  // Convert all range variables to local variables.
  domain.convertToLocal(VarKind::SetDim, getNumDomainDims(),
                        getNumDomainDims() + getNumRangeDims());
  return domain;
}

FlatAffineValueConstraints FlatAffineRelation::getRangeSet() const {
  FlatAffineValueConstraints range = *this;
  // Convert all domain variables to local variables.
  range.convertToLocal(VarKind::SetDim, 0, getNumDomainDims());
  return range;
}

```
- **EN**: Implements logic around `getDomainSet`, `convertToLocal`, `getNumDomainDims`, `getRangeSet`.
- **CN**: 围绕 `getDomainSet`, `convertToLocal`, `getNumDomainDims`, `getRangeSet` 实现具体逻辑。

### Lines 378-397
```cpp
void FlatAffineRelation::compose(const FlatAffineRelation &other) {
  assert(getNumDomainDims() == other.getNumRangeDims() &&
         "Domain of this and range of other do not match");
  assert(space.getDomainSpace().isAligned(other.getSpace().getRangeSpace()) &&
         "Values of domain of this and range of other do not match");

  FlatAffineRelation rel = other;

  // Convert `rel` from
  //    [otherDomain] -> [otherRange]
  // to
  //    [otherDomain] -> [otherRange thisRange]
  // and `this` from
  //    [thisDomain] -> [thisRange]
  // to
  //    [otherDomain thisDomain] -> [thisRange].
  unsigned removeDims = rel.getNumRangeDims();
  insertDomainVar(0, rel.getNumDomainDims());
  rel.appendRangeVar(getNumRangeDims());

```
- **EN**: Implements logic around `compose`, `assert`, `getNumRangeDims`, `insertDomainVar`, and 1 more symbols.
- **CN**: 围绕 `compose`, `assert`, `getNumRangeDims`, `insertDomainVar`, and 1 more symbols 实现具体逻辑。

### Lines 398-412
```cpp
  // Merge symbol and local variables.
  mergeSymbolVars(rel);
  mergeLocalVars(rel);

  // Convert `rel` from [otherDomain] -> [otherRange thisRange] to
  // [otherDomain] -> [thisRange] by converting first otherRange range vars
  // to local vars.
  rel.convertToLocal(VarKind::SetDim, rel.getNumDomainDims(),
                     rel.getNumDomainDims() + removeDims);
  // Convert `this` from [otherDomain thisDomain] -> [thisRange] to
  // [otherDomain] -> [thisRange] by converting last thisDomain domain vars
  // to local vars.
  convertToLocal(VarKind::SetDim, getNumDomainDims() - removeDims,
                 getNumDomainDims());

```
- **EN**: Implements logic around `mergeSymbolVars`, `mergeLocalVars`, `convertToLocal`, `getNumDomainDims`; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `mergeSymbolVars`, `mergeLocalVars`, `convertToLocal`, `getNumDomainDims` 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 413-426
```cpp
  auto thisMaybeValues = getMaybeValues(VarKind::SetDim);
  auto relMaybeValues = rel.getMaybeValues(VarKind::SetDim);

  // Add and match domain of `rel` to domain of `this`.
  for (unsigned i = 0, e = rel.getNumDomainDims(); i < e; ++i)
    if (relMaybeValues[i].has_value())
      setValue(i, *relMaybeValues[i]);
  // Add and match range of `this` to range of `rel`.
  for (unsigned i = 0, e = getNumRangeDims(); i < e; ++i) {
    unsigned rangeIdx = rel.getNumDomainDims() + i;
    if (thisMaybeValues[rangeIdx].has_value())
      rel.setValue(rangeIdx, *thisMaybeValues[rangeIdx]);
  }

```
- **EN**: Implements logic around `getMaybeValues`, `getNumDomainDims`, `has_value`, `setValue`, and 1 more symbols.
- **CN**: 围绕 `getMaybeValues`, `getNumDomainDims`, `has_value`, `setValue`, and 1 more symbols 实现具体逻辑。

### Lines 427-446
```cpp
  // Append `this` to `rel` and simplify constraints.
  rel.append(*this);
  rel.removeRedundantLocalVars();

  *this = rel;
}

void FlatAffineRelation::inverse() {
  unsigned oldDomain = getNumDomainDims();
  unsigned oldRange = getNumRangeDims();
  // Add new range vars.
  appendRangeVar(oldDomain);
  // Swap new vars with domain.
  for (unsigned i = 0; i < oldDomain; ++i)
    swapVar(i, oldDomain + oldRange + i);
  // Remove the swapped domain.
  removeVarRange(0, oldDomain);
  // Set domain and range as inverse.
  numDomainDims = oldRange;
  numRangeDims = oldDomain;
```
- **EN**: Implements logic around `append`, `removeRedundantLocalVars`, `inverse`, `getNumDomainDims`, and 4 more symbols.
- **CN**: 围绕 `append`, `removeRedundantLocalVars`, `inverse`, `getNumDomainDims`, and 4 more symbols 实现具体逻辑。

### Lines 447-462
```cpp
}

void FlatAffineRelation::insertDomainVar(unsigned pos, unsigned num) {
  assert(pos <= getNumDomainDims() &&
         "Var cannot be inserted at invalid position");
  insertDimVar(pos, num);
  numDomainDims += num;
}

void FlatAffineRelation::insertRangeVar(unsigned pos, unsigned num) {
  assert(pos <= getNumRangeDims() &&
         "Var cannot be inserted at invalid position");
  insertDimVar(getNumDomainDims() + pos, num);
  numRangeDims += num;
}

```
- **EN**: Implements logic around `insertDomainVar`, `assert`, `insertDimVar`, `insertRangeVar`.
- **CN**: 围绕 `insertDomainVar`, `assert`, `insertDimVar`, `insertRangeVar` 实现具体逻辑。

### Lines 463-472
```cpp
void FlatAffineRelation::appendDomainVar(unsigned num) {
  insertDimVar(getNumDomainDims(), num);
  numDomainDims += num;
}

void FlatAffineRelation::appendRangeVar(unsigned num) {
  insertDimVar(getNumDimVars(), num);
  numRangeDims += num;
}

```
- **EN**: Implements logic around `appendDomainVar`, `insertDimVar`, `appendRangeVar`.
- **CN**: 围绕 `appendDomainVar`, `insertDimVar`, `appendRangeVar` 实现具体逻辑。

### Lines 473-484
```cpp
void FlatAffineRelation::removeVarRange(VarKind kind, unsigned varStart,
                                        unsigned varLimit) {
  assert(varLimit <= getNumVarKind(kind));
  if (varStart >= varLimit)
    return;

  FlatAffineValueConstraints::removeVarRange(kind, varStart, varLimit);

  // If kind is not SetDim, domain and range don't need to be updated.
  if (kind != VarKind::SetDim)
    return;

```
- **EN**: Implements logic around `removeVarRange`, `assert`.
- **CN**: 围绕 `removeVarRange`, `assert` 实现具体逻辑。

### Lines 485-497
```cpp
  // Compute number of domain and range variables to remove. This is done by
  // intersecting the range of domain/range vars with range of vars to remove.
  unsigned intersectDomainLHS = std::min(varLimit, getNumDomainDims());
  unsigned intersectDomainRHS = varStart;
  unsigned intersectRangeLHS = std::min(varLimit, getNumDimVars());
  unsigned intersectRangeRHS = std::max(varStart, getNumDomainDims());

  if (intersectDomainLHS > intersectDomainRHS)
    numDomainDims -= intersectDomainLHS - intersectDomainRHS;
  if (intersectRangeLHS > intersectRangeRHS)
    numRangeDims -= intersectRangeLHS - intersectRangeRHS;
}

```
- **EN**: Implements logic around `min`, `max`.
- **CN**: 围绕 `min`, `max` 实现具体逻辑。

### Lines 498-510
```cpp
LogicalResult mlir::affine::getRelationFromMap(AffineMap &map,
                                               IntegerRelation &rel) {
  // Get flattened affine expressions.
  std::vector<SmallVector<int64_t, 8>> flatExprs;
  FlatAffineValueConstraints localVarCst;
  if (failed(getFlattenedAffineExprs(map, &flatExprs, &localVarCst)))
    return failure();

  const unsigned oldDimNum = localVarCst.getNumDimVars();
  const unsigned oldCols = localVarCst.getNumCols();
  const unsigned numRangeVars = map.getNumResults();
  const unsigned numDomainVars = map.getNumDims();

```
- **EN**: Implements logic around `getRelationFromMap`, `failed`, `failure`, `getNumDimVars`, and 3 more symbols; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `getRelationFromMap`, `failed`, `failure`, `getNumDimVars`, and 3 more symbols 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 511-530
```cpp
  // Add range as the new expressions.
  localVarCst.appendDimVar(numRangeVars);

  // Add identifiers to the local constraints as getFlattenedAffineExprs creates
  // a FlatLinearConstraints with no identifiers.
  for (unsigned i = 0, e = localVarCst.getNumDimAndSymbolVars(); i < e; ++i)
    localVarCst.setValue(i, Value());

  // Add equalities between source and range.
  SmallVector<int64_t, 8> eq(localVarCst.getNumCols());
  for (unsigned i = 0, e = map.getNumResults(); i < e; ++i) {
    // Zero fill.
    llvm::fill(eq, 0);
    // Fill equality.
    for (unsigned j = 0, f = oldDimNum; j < f; ++j)
      eq[j] = flatExprs[i][j];
    for (unsigned j = oldDimNum, f = oldCols; j < f; ++j)
      eq[j + numRangeVars] = flatExprs[i][j];
    // Set this dimension to -1 to equate lhs and rhs and add equality.
    eq[numDomainVars + i] = -1;
```
- **EN**: Implements logic around `appendDimVar`, `getNumDimAndSymbolVars`, `setValue`, `eq`, and 2 more symbols.
- **CN**: 围绕 `appendDimVar`, `getNumDimAndSymbolVars`, `setValue`, `eq`, and 2 more symbols 实现具体逻辑。

### Lines 531-540
```cpp
    localVarCst.addEquality(eq);
  }

  rel = localVarCst;
  return success();
}

LogicalResult mlir::affine::getRelationFromMap(const AffineValueMap &map,
                                               IntegerRelation &rel) {

```
- **EN**: Implements logic around `addEquality`, `success`, `getRelationFromMap`; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `addEquality`, `success`, `getRelationFromMap` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 541-554
```cpp
  AffineMap affineMap = map.getAffineMap();
  if (failed(getRelationFromMap(affineMap, rel)))
    return failure();

  // Set identifiers for domain and symbol variables.
  for (unsigned i = 0, e = affineMap.getNumDims(); i < e; ++i)
    rel.setId(VarKind::SetDim, i, Identifier(map.getOperand(i)));

  const unsigned mapNumResults = affineMap.getNumResults();
  for (unsigned i = 0, e = rel.getNumSymbolVars(); i < e; ++i)
    rel.setId(
        VarKind::Symbol, i,
        Identifier(map.getOperand(rel.getNumDimVars() + i - mapNumResults)));

```
- **EN**: Implements logic around `getAffineMap`, `failed`, `failure`, `getNumDims`, and 4 more symbols; this block performs affine reasoning or shape/bounds manipulation; works with symbol tables or function-like operations.
- **CN**: 围绕 `getAffineMap`, `failed`, `failure`, `getNumDims`, and 4 more symbols 实现具体逻辑；该代码块执行仿射推理或形状/边界处理，并处理符号表或类函数操作。

### Lines 555-556
```cpp
  return success();
}
```
- **EN**: Implements logic around `success`.
- **CN**: 围绕 `success` 实现具体逻辑。

## Key Concepts / 关键概念

- **Static analysis / 静态分析**:
  - **EN**: Collects facts, dependence information, or structural properties that other passes consume.
  - **CN**: 收集供其他 pass 使用的事实、依赖信息或结构属性。
- **Affine reasoning / 仿射推理**:
  - **EN**: Works with affine maps, bounds, dependence reasoning, and loop structure.
  - **CN**: 处理仿射映射、边界、依赖推理与循环结构。
- **Type/attribute storage / 类型/属性存储**:
  - **EN**: Uses uniqued storage and parser/printer hooks for extensible compile-time metadata.
  - **CN**: 使用唯一化存储以及解析/打印钩子来表示可扩展的编译期元数据。
- **LLVM interop / LLVM 互操作**:
  - **EN**: Bridges MLIR concepts to LLVM-compatible data structures, intrinsics, or codegen expectations.
  - **CN**: 把 MLIR 概念桥接到 LLVM 兼容的数据结构、intrinsic 或代码生成预期。
- **Streaming output / 流式输出**:
  - **EN**: Writes diagnostics or assembly-like textual forms through LLVM/MLIR stream APIs.
  - **CN**: 通过 LLVM/MLIR 流式 API 输出诊断或类汇编文本。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Affine/Analysis/AffineStructures.h`, `mlir/Analysis/Presburger/IntegerRelation.h`, `mlir/Analysis/Presburger/Utils.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/IR/AffineValueMap.h`, `mlir/Dialect/Utils/StaticValueUtils.h`, `mlir/IR/IntegerSet.h`, `mlir/Support/LLVM.h`, `llvm/ADT/SmallVector.h`, `llvm/Support/Debug.h` ... (+1 more)
- **Standard-library headers / 标准库头文件**: `<optional>`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (4), MLIR analysis interfaces / MLIR 分析接口 (2), LLVM support-library helpers / LLVM Support 库辅助功能 (2), MLIR IR core abstractions / MLIR IR 核心抽象 (1), MLIR support-library helpers / MLIR Support 库辅助功能 (1), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (1)
