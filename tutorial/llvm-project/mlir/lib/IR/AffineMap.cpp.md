# AffineMap.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/IR/AffineMap.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements MLIR core IR objects, printing/parsing support, and builtin type or attribute behavior.
  - **CN**: 实现 MLIR 核心 IR 对象、打印/解析支持以及内建类型或属性行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-22
```cpp
//===- AffineMap.cpp - MLIR Affine Map Classes ----------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "mlir/IR/AffineMap.h"
#include "AffineMapDetail.h"
#include "mlir/IR/AffineExpr.h"
#include "mlir/IR/Builders.h"
#include "mlir/IR/BuiltinAttributes.h"
#include "mlir/IR/BuiltinTypes.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SmallBitVector.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/Support/MathExtras.h"
#include <numeric>
#include <optional>
#include <type_traits>

```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/IR/AffineMap.h`, `AffineMapDetail.h`, `mlir/IR/AffineExpr.h`, `mlir/IR/Builders.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/IR/AffineMap.h`, `AffineMapDetail.h`, `mlir/IR/AffineExpr.h`, `mlir/IR/Builders.h`。

### Lines 23-36
```cpp
using namespace mlir;

using llvm::divideCeilSigned;
using llvm::divideFloorSigned;
using llvm::mod;

namespace {

// AffineExprConstantFolder evaluates an affine expression using constant
// operands passed in 'operandConsts'. Returns an IntegerAttr attribute
// representing the constant value of the affine expression evaluated on
// constant 'operandConsts', or nullptr if it can't be folded.
class AffineExprConstantFolder {
public:
```
- **EN**: Introduces declarations for `AffineExprConstantFolder`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `AffineExprConstantFolder` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 37-50
```cpp
  AffineExprConstantFolder(unsigned numDims, ArrayRef<Attribute> operandConsts)
      : numDims(numDims), operandConsts(operandConsts) {}

  /// Attempt to constant fold the specified affine expr, or return null on
  /// failure.
  IntegerAttr constantFold(AffineExpr expr) {
    if (auto result = constantFoldImpl(expr))
      return IntegerAttr::get(IndexType::get(expr.getContext()), *result);
    return nullptr;
  }

  bool hasPoison() const { return hasPoison_; }

private:
```
- **EN**: Implements logic around `AffineExprConstantFolder`, `numDims`, `constantFold`, `constantFoldImpl`, and 2 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `AffineExprConstantFolder`、`numDims`、`constantFold`、`constantFoldImpl` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 51-78
```cpp
  std::optional<int64_t> constantFoldImpl(AffineExpr expr) {
    switch (expr.getKind()) {
    case AffineExprKind::Add:
      return constantFoldBinExpr(
          expr, [](int64_t lhs, int64_t rhs) { return lhs + rhs; });
    case AffineExprKind::Mul:
      return constantFoldBinExpr(
          expr, [](int64_t lhs, int64_t rhs) { return lhs * rhs; });
    case AffineExprKind::Mod:
      return constantFoldBinExpr(
          expr, [this](int64_t lhs, int64_t rhs) -> std::optional<int64_t> {
            if (rhs < 1) {
              hasPoison_ = true;
              return std::nullopt;
            }
            return mod(lhs, rhs);
          });
    case AffineExprKind::FloorDiv:
      return constantFoldBinExpr(
          expr, [this](int64_t lhs, int64_t rhs) -> std::optional<int64_t> {
            if (rhs == 0) {
              hasPoison_ = true;
              return std::nullopt;
            }
            return divideFloorSigned(lhs, rhs);
          });
    case AffineExprKind::CeilDiv:
      return constantFoldBinExpr(
```
- **EN**: Implements logic around `constantFoldImpl`, `getKind`, `constantFoldBinExpr`, `mod`, and 1 more symbols.
- **CN**: 围绕 `constantFoldImpl`、`getKind`、`constantFoldBinExpr`、`mod` 等另外 1 个符号 实现具体逻辑。

### Lines 79-102
```cpp
          expr, [this](int64_t lhs, int64_t rhs) -> std::optional<int64_t> {
            if (rhs == 0) {
              hasPoison_ = true;
              return std::nullopt;
            }
            return divideCeilSigned(lhs, rhs);
          });
    case AffineExprKind::Constant:
      return cast<AffineConstantExpr>(expr).getValue();
    case AffineExprKind::DimId:
      if (auto attr = llvm::dyn_cast_or_null<IntegerAttr>(
              operandConsts[cast<AffineDimExpr>(expr).getPosition()]))
        return attr.getInt();
      return std::nullopt;
    case AffineExprKind::SymbolId:
      if (auto attr = llvm::dyn_cast_or_null<IntegerAttr>(
              operandConsts[numDims +
                            cast<AffineSymbolExpr>(expr).getPosition()]))
        return attr.getInt();
      return std::nullopt;
    }
    llvm_unreachable("Unknown AffineExpr");
  }

```
- **EN**: Implements logic around `divideCeilSigned`, `cast`, `dyn_cast_or_null`, `getInt`, and 1 more symbols.
- **CN**: 围绕 `divideCeilSigned`、`cast`、`dyn_cast_or_null`、`getInt` 等另外 1 个符号 实现具体逻辑。

### Lines 103-120
```cpp
  // TODO: Change these to operate on APInts too.
  std::optional<int64_t> constantFoldBinExpr(
      AffineExpr expr,
      llvm::function_ref<std::optional<int64_t>(int64_t, int64_t)> op) {
    auto binOpExpr = cast<AffineBinaryOpExpr>(expr);
    if (auto lhs = constantFoldImpl(binOpExpr.getLHS()))
      if (auto rhs = constantFoldImpl(binOpExpr.getRHS()))
        return op(*lhs, *rhs);
    return std::nullopt;
  }

  // The number of dimension operands in AffineMap containing this expression.
  unsigned numDims;
  // The constant valued operands used to evaluate this AffineExpr.
  ArrayRef<Attribute> operandConsts;
  bool hasPoison_{false};
};

```
- **EN**: Implements logic around `constantFoldBinExpr`, `optional`, `cast`, `constantFoldImpl`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `constantFoldBinExpr`、`optional`、`cast`、`constantFoldImpl` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 121-137
```cpp
} // namespace

/// Returns a single constant result affine map.
AffineMap AffineMap::getConstantMap(int64_t val, MLIRContext *context) {
  return get(/*dimCount=*/0, /*symbolCount=*/0,
             {getAffineConstantExpr(val, context)});
}

/// Returns an identity affine map (d0, ..., dn) -> (dp, ..., dn) on the most
/// minor dimensions.
AffineMap AffineMap::getMinorIdentityMap(unsigned dims, unsigned results,
                                         MLIRContext *context) {
  assert(dims >= results && "Dimension mismatch");
  auto id = AffineMap::getMultiDimIdentityMap(dims, context);
  return AffineMap::get(dims, 0, id.getResults().take_back(results), context);
}

```
- **EN**: Implements logic around `getConstantMap`, `get`, `getAffineConstantExpr`, `getMinorIdentityMap`, and 2 more symbols.
- **CN**: 围绕 `getConstantMap`、`get`、`getAffineConstantExpr`、`getMinorIdentityMap` 等另外 2 个符号 实现具体逻辑。

### Lines 138-156
```cpp
AffineMap AffineMap::getFilteredIdentityMap(
    MLIRContext *ctx, unsigned numDims,
    llvm::function_ref<bool(AffineDimExpr)> keepDimFilter) {
  auto identityMap = getMultiDimIdentityMap(numDims, ctx);

  // Apply filter to results.
  llvm::SmallBitVector dropDimResults(numDims);
  for (auto [idx, resultExpr] : llvm::enumerate(identityMap.getResults()))
    dropDimResults[idx] = !keepDimFilter(cast<AffineDimExpr>(resultExpr));

  return identityMap.dropResults(dropDimResults);
}

bool AffineMap::isMinorIdentity() const {
  return getNumDims() >= getNumResults() &&
         *this ==
             getMinorIdentityMap(getNumDims(), getNumResults(), getContext());
}

```
- **EN**: Implements logic around `getFilteredIdentityMap`, `function_ref`, `getMultiDimIdentityMap`, `dropDimResults`, and 6 more symbols.
- **CN**: 围绕 `getFilteredIdentityMap`、`function_ref`、`getMultiDimIdentityMap`、`dropDimResults` 等另外 6 个符号 实现具体逻辑。

### Lines 157-170
```cpp
SmallVector<unsigned> AffineMap::getBroadcastDims() const {
  SmallVector<unsigned> broadcastedDims;
  for (const auto &[resIdx, expr] : llvm::enumerate(getResults())) {
    if (auto constExpr = dyn_cast<AffineConstantExpr>(expr)) {
      if (constExpr.getValue() != 0)
        continue;
      broadcastedDims.push_back(resIdx);
    }
  }

  return broadcastedDims;
}

/// Returns true if this affine map is a minor identity up to broadcasted
```
- **EN**: Implements logic around `getBroadcastDims`, `enumerate`, `dyn_cast`, `getValue`, and 1 more symbols.
- **CN**: 围绕 `getBroadcastDims`、`enumerate`、`dyn_cast`、`getValue` 等另外 1 个符号 实现具体逻辑。

### Lines 171-198
```cpp
/// dimensions which are indicated by value 0 in the result.
bool AffineMap::isMinorIdentityWithBroadcasting(
    SmallVectorImpl<unsigned> *broadcastedDims) const {
  if (broadcastedDims)
    broadcastedDims->clear();
  if (getNumDims() < getNumResults())
    return false;
  unsigned suffixStart = getNumDims() - getNumResults();
  for (const auto &idxAndExpr : llvm::enumerate(getResults())) {
    unsigned resIdx = idxAndExpr.index();
    AffineExpr expr = idxAndExpr.value();
    if (auto constExpr = dyn_cast<AffineConstantExpr>(expr)) {
      // Each result may be either a constant 0 (broadcasted dimension).
      if (constExpr.getValue() != 0)
        return false;
      if (broadcastedDims)
        broadcastedDims->push_back(resIdx);
    } else if (auto dimExpr = dyn_cast<AffineDimExpr>(expr)) {
      // Or it may be the input dimension corresponding to this result position.
      if (dimExpr.getPosition() != suffixStart + resIdx)
        return false;
    } else {
      return false;
    }
  }
  return true;
}

```
- **EN**: Implements logic around `isMinorIdentityWithBroadcasting`, `clear`, `getNumDims`, `enumerate`, and 6 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `isMinorIdentityWithBroadcasting`、`clear`、`getNumDims`、`enumerate` 等另外 6 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 199-226
```cpp
/// Return true if this affine map can be converted to a minor identity with
/// broadcast by doing a permute. Return a permutation (there may be
/// several) to apply to get to a minor identity with broadcasts.
/// Ex:
///  * (d0, d1, d2) -> (0, d1) maps to minor identity (d1, 0 = d2) with
///  perm = [1, 0] and broadcast d2
///  * (d0, d1, d2) -> (d0, 0) cannot be mapped to a minor identity by
///  permutation + broadcast
///  * (d0, d1, d2, d3) -> (0, d1, d3) maps to minor identity (d1, 0 = d2, d3)
///  with perm = [1, 0, 2] and broadcast d2
///  * (d0, d1) -> (d1, 0, 0, d0) maps to minor identity (d0, d1) with extra
///  leading broadcat dimensions. The map returned would be (0, 0, d0, d1) with
///  perm = [3, 0, 1, 2]
bool AffineMap::isPermutationOfMinorIdentityWithBroadcasting(
    SmallVectorImpl<unsigned> &permutedDims) const {
  unsigned projectionStart =
      getNumResults() < getNumInputs() ? getNumInputs() - getNumResults() : 0;
  permutedDims.clear();
  SmallVector<unsigned> broadcastDims;
  permutedDims.resize(getNumResults(), 0);
  // If there are more results than input dimensions we want the new map to
  // start with broadcast dimensions in order to be a minor identity with
  // broadcasting.
  unsigned leadingBroadcast =
      getNumResults() > getNumInputs() ? getNumResults() - getNumInputs() : 0;
  llvm::SmallBitVector dimFound(std::max(getNumInputs(), getNumResults()),
                                false);
  for (const auto &idxAndExpr : llvm::enumerate(getResults())) {
```
- **EN**: Implements logic around `isPermutationOfMinorIdentityWithBroadcasting`, `getNumResults`, `clear`, `resize`, and 2 more symbols.
- **CN**: 围绕 `isPermutationOfMinorIdentityWithBroadcasting`、`getNumResults`、`clear`、`resize` 等另外 2 个符号 实现具体逻辑。

### Lines 227-254
```cpp
    unsigned resIdx = idxAndExpr.index();
    AffineExpr expr = idxAndExpr.value();
    // Each result may be either a constant 0 (broadcast dimension) or a
    // dimension.
    if (auto constExpr = dyn_cast<AffineConstantExpr>(expr)) {
      if (constExpr.getValue() != 0)
        return false;
      broadcastDims.push_back(resIdx);
    } else if (auto dimExpr = dyn_cast<AffineDimExpr>(expr)) {
      if (dimExpr.getPosition() < projectionStart)
        return false;
      unsigned newPosition =
          dimExpr.getPosition() - projectionStart + leadingBroadcast;
      permutedDims[resIdx] = newPosition;
      dimFound[newPosition] = true;
    } else {
      return false;
    }
  }
  // Find a permuation for the broadcast dimension. Since they are broadcasted
  // any valid permutation is acceptable. We just permute the dim into a slot
  // without an existing dimension.
  unsigned pos = 0;
  for (auto dim : broadcastDims) {
    while (pos < dimFound.size() && dimFound[pos]) {
      pos++;
    }
    permutedDims[dim] = pos++;
```
- **EN**: Implements logic around `index`, `value`, `dyn_cast`, `getValue`, and 3 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `index`、`value`、`dyn_cast`、`getValue` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 255-275
```cpp
  }
  return true;
}

/// Returns an AffineMap representing a permutation.
AffineMap AffineMap::getPermutationMap(ArrayRef<unsigned> permutation,
                                       MLIRContext *context) {
  assert(!permutation.empty() &&
         "Cannot create permutation map from empty permutation vector");
  const auto *m = llvm::max_element(permutation);
  auto permutationMap = getMultiDimMapWithTargets(*m + 1, permutation, context);
  assert(permutationMap.isPermutation() && "Invalid permutation vector");
  return permutationMap;
}
AffineMap AffineMap::getPermutationMap(ArrayRef<int64_t> permutation,
                                       MLIRContext *context) {
  SmallVector<unsigned> perm = llvm::map_to_vector(
      permutation, [](int64_t i) { return static_cast<unsigned>(i); });
  return AffineMap::getPermutationMap(perm, context);
}

```
- **EN**: Implements logic around `getPermutationMap`, `assert`, `max_element`, `getMultiDimMapWithTargets`, and 2 more symbols.
- **CN**: 围绕 `getPermutationMap`、`assert`、`max_element`、`getMultiDimMapWithTargets` 等另外 2 个符号 实现具体逻辑。

### Lines 276-289
```cpp
AffineMap AffineMap::getMultiDimMapWithTargets(unsigned numDims,
                                               ArrayRef<unsigned> targets,
                                               MLIRContext *context) {
  // Inline size chosen empirically based on compilation profiling.
  // Profiled: 3.1M calls, avg=4.1+-3.7. N=8 covers ~86% of cases inline.
  SmallVector<AffineExpr, 8> affExprs;
  for (unsigned t : targets)
    affExprs.push_back(getAffineDimExpr(t, context));
  AffineMap result = AffineMap::get(/*dimCount=*/numDims, /*symbolCount=*/0,
                                    affExprs, context);
  return result;
}

/// Creates an affine map each for each list of AffineExpr's in `exprsList`
```
- **EN**: Implements logic around `getMultiDimMapWithTargets`, `push_back`, `get`.
- **CN**: 围绕 `getMultiDimMapWithTargets`、`push_back`、`get` 实现具体逻辑。

### Lines 290-308
```cpp
/// while inferring the right number of dimensional and symbolic inputs needed
/// based on the maximum dimensional and symbolic identifier appearing in the
/// expressions.
template <typename AffineExprContainer>
static SmallVector<AffineMap, 4>
inferFromExprList(ArrayRef<AffineExprContainer> exprsList,
                  MLIRContext *context) {
  if (exprsList.empty())
    return {};
  int64_t maxDim = -1, maxSym = -1;
  getMaxDimAndSymbol(exprsList, maxDim, maxSym);
  SmallVector<AffineMap, 4> maps;
  maps.reserve(exprsList.size());
  for (const auto &exprs : exprsList)
    maps.push_back(AffineMap::get(/*dimCount=*/maxDim + 1,
                                  /*symbolCount=*/maxSym + 1, exprs, context));
  return maps;
}

```
- **EN**: Implements logic around `inferFromExprList`, `empty`, `getMaxDimAndSymbol`, `reserve`, and 1 more symbols.
- **CN**: 围绕 `inferFromExprList`、`empty`、`getMaxDimAndSymbol`、`reserve` 等另外 1 个符号 实现具体逻辑。

### Lines 309-331
```cpp
SmallVector<AffineMap, 4>
AffineMap::inferFromExprList(ArrayRef<ArrayRef<AffineExpr>> exprsList,
                             MLIRContext *context) {
  return ::inferFromExprList(exprsList, context);
}

SmallVector<AffineMap, 4>
AffineMap::inferFromExprList(ArrayRef<SmallVector<AffineExpr, 4>> exprsList,
                             MLIRContext *context) {
  return ::inferFromExprList(exprsList, context);
}

uint64_t AffineMap::getLargestKnownDivisorOfMapExprs() {
  uint64_t gcd = 0;
  for (AffineExpr resultExpr : getResults()) {
    uint64_t thisGcd = resultExpr.getLargestKnownDivisor();
    gcd = std::gcd(gcd, thisGcd);
  }
  if (gcd == 0)
    gcd = std::numeric_limits<uint64_t>::max();
  return gcd;
}

```
- **EN**: Implements logic around `inferFromExprList`, `getLargestKnownDivisorOfMapExprs`, `getResults`, `getLargestKnownDivisor`, and 2 more symbols.
- **CN**: 围绕 `inferFromExprList`、`getLargestKnownDivisorOfMapExprs`、`getResults`、`getLargestKnownDivisor` 等另外 2 个符号 实现具体逻辑。

### Lines 332-354
```cpp
AffineMap AffineMap::getMultiDimIdentityMap(unsigned numDims,
                                            MLIRContext *context) {
  SmallVector<AffineExpr, 4> dimExprs;
  dimExprs.reserve(numDims);
  for (unsigned i = 0; i < numDims; ++i)
    dimExprs.push_back(mlir::getAffineDimExpr(i, context));
  return get(/*dimCount=*/numDims, /*symbolCount=*/0, dimExprs, context);
}

MLIRContext *AffineMap::getContext() const { return map->context; }

bool AffineMap::isIdentity() const {
  if (getNumDims() != getNumResults())
    return false;
  ArrayRef<AffineExpr> results = getResults();
  for (unsigned i = 0, numDims = getNumDims(); i < numDims; ++i) {
    auto expr = dyn_cast<AffineDimExpr>(results[i]);
    if (!expr || expr.getPosition() != i)
      return false;
  }
  return true;
}

```
- **EN**: Implements logic around `getMultiDimIdentityMap`, `reserve`, `push_back`, `get`, and 6 more symbols.
- **CN**: 围绕 `getMultiDimIdentityMap`、`reserve`、`push_back`、`get` 等另外 6 个符号 实现具体逻辑。

### Lines 355-370
```cpp
bool AffineMap::isSymbolIdentity() const {
  if (getNumSymbols() != getNumResults())
    return false;
  ArrayRef<AffineExpr> results = getResults();
  for (unsigned i = 0, numSymbols = getNumSymbols(); i < numSymbols; ++i) {
    auto expr = dyn_cast<AffineDimExpr>(results[i]);
    if (!expr || expr.getPosition() != i)
      return false;
  }
  return true;
}

bool AffineMap::isEmpty() const {
  return getNumDims() == 0 && getNumSymbols() == 0 && getNumResults() == 0;
}

```
- **EN**: Implements logic around `isSymbolIdentity`, `getNumSymbols`, `getResults`, `dyn_cast`, and 3 more symbols.
- **CN**: 围绕 `isSymbolIdentity`、`getNumSymbols`、`getResults`、`dyn_cast` 等另外 3 个符号 实现具体逻辑。

### Lines 371-391
```cpp
bool AffineMap::isSingleConstant() const {
  return getNumResults() == 1 && isa<AffineConstantExpr>(getResult(0));
}

bool AffineMap::isConstant() const {
  return llvm::all_of(getResults(), llvm::IsaPred<AffineConstantExpr>);
}

int64_t AffineMap::getSingleConstantResult() const {
  assert(isSingleConstant() && "map must have a single constant result");
  return cast<AffineConstantExpr>(getResult(0)).getValue();
}

SmallVector<int64_t> AffineMap::getConstantResults() const {
  assert(isConstant() && "map must have only constant results");
  SmallVector<int64_t> result;
  for (auto expr : getResults())
    result.emplace_back(cast<AffineConstantExpr>(expr).getValue());
  return result;
}

```
- **EN**: Implements logic around `isSingleConstant`, `getNumResults`, `isConstant`, `all_of`, and 6 more symbols.
- **CN**: 围绕 `isSingleConstant`、`getNumResults`、`isConstant`、`all_of` 等另外 6 个符号 实现具体逻辑。

### Lines 392-412
```cpp
unsigned AffineMap::getNumDims() const {
  assert(map && "uninitialized map storage");
  return map->numDims;
}
unsigned AffineMap::getNumSymbols() const {
  assert(map && "uninitialized map storage");
  return map->numSymbols;
}
unsigned AffineMap::getNumResults() const { return getResults().size(); }
unsigned AffineMap::getNumInputs() const {
  assert(map && "uninitialized map storage");
  return map->numDims + map->numSymbols;
}
ArrayRef<AffineExpr> AffineMap::getResults() const {
  assert(map && "uninitialized map storage");
  return map->results();
}
AffineExpr AffineMap::getResult(unsigned idx) const {
  return getResults()[idx];
}

```
- **EN**: Implements logic around `getNumDims`, `assert`, `getNumSymbols`, `getNumResults`, and 4 more symbols.
- **CN**: 围绕 `getNumDims`、`assert`、`getNumSymbols`、`getNumResults` 等另外 4 个符号 实现具体逻辑。

### Lines 413-428
```cpp
unsigned AffineMap::getDimPosition(unsigned idx) const {
  return cast<AffineDimExpr>(getResult(idx)).getPosition();
}

std::optional<unsigned> AffineMap::getResultPosition(AffineExpr input) const {
  if (!isa<AffineDimExpr>(input))
    return std::nullopt;

  for (unsigned i = 0, numResults = getNumResults(); i < numResults; i++) {
    if (getResult(i) == input)
      return i;
  }

  return std::nullopt;
}

```
- **EN**: Implements logic around `getDimPosition`, `cast`, `getResultPosition`, `isa`, and 2 more symbols.
- **CN**: 围绕 `getDimPosition`、`cast`、`getResultPosition`、`isa` 等另外 2 个符号 实现具体逻辑。

### Lines 429-443
```cpp
/// Folds the results of the application of an affine map on the provided
/// operands to a constant if possible. Returns false if the folding happens,
/// true otherwise.
LogicalResult AffineMap::constantFold(ArrayRef<Attribute> operandConstants,
                                      SmallVectorImpl<Attribute> &results,
                                      bool *hasPoison) const {
  // Attempt partial folding.
  SmallVector<int64_t, 2> integers;
  partialConstantFold(operandConstants, &integers, hasPoison);

  // If all expressions folded to a constant, populate results with attributes
  // containing those constants.
  if (integers.empty())
    return failure();

```
- **EN**: Implements logic around `constantFold`, `partialConstantFold`, `empty`, `failure`; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects.
- **CN**: 围绕 `constantFold`、`partialConstantFold`、`empty`、`failure` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象。

### Lines 444-460
```cpp
  auto range = llvm::map_range(integers, [this](int64_t i) {
    return IntegerAttr::get(IndexType::get(getContext()), i);
  });
  results.append(range.begin(), range.end());
  return success();
}

AffineMap AffineMap::partialConstantFold(ArrayRef<Attribute> operandConstants,
                                         SmallVectorImpl<int64_t> *results,
                                         bool *hasPoison) const {
  assert(getNumInputs() == operandConstants.size());

  // Fold each of the result expressions.
  AffineExprConstantFolder exprFolder(getNumDims(), operandConstants);
  SmallVector<AffineExpr, 4> exprs;
  exprs.reserve(getNumResults());

```
- **EN**: Implements logic around `map_range`, `get`, `append`, `success`, and 4 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `map_range`、`get`、`append`、`success` 等另外 4 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 461-482
```cpp
  for (auto expr : getResults()) {
    auto folded = exprFolder.constantFold(expr);
    if (exprFolder.hasPoison() && hasPoison) {
      *hasPoison = true;
      return {};
    }
    // If did not fold to a constant, keep the original expression, and clear
    // the integer results vector.
    if (folded) {
      exprs.push_back(
          getAffineConstantExpr(folded.getInt(), folded.getContext()));
      if (results)
        results->push_back(folded.getInt());
    } else {
      exprs.push_back(expr);
      if (results) {
        results->clear();
        results = nullptr;
      }
    }
  }

```
- **EN**: Implements logic around `getResults`, `constantFold`, `hasPoison`, `push_back`, and 2 more symbols; this block implements transformation or simplification logic.
- **CN**: 围绕 `getResults`、`constantFold`、`hasPoison`、`push_back` 等另外 2 个符号 实现具体逻辑；该代码块实现变换或简化逻辑。

### Lines 483-496
```cpp
  return get(getNumDims(), getNumSymbols(), exprs, getContext());
}

/// Walk all of the AffineExpr's in this mapping. Each node in an expression
/// tree is visited in postorder.
void AffineMap::walkExprs(llvm::function_ref<void(AffineExpr)> callback) const {
  for (auto expr : getResults())
    expr.walk(callback);
}

/// This method substitutes any uses of dimensions and symbols (e.g.
/// dim#0 with dimReplacements[0]) in subexpressions and returns the modified
/// expression mapping.  Because this can be used to eliminate dims and
/// symbols, the client needs to specify the number of dims and symbols in
```
- **EN**: Implements logic around `get`, `walkExprs`, `getResults`, `walk`.
- **CN**: 围绕 `get`、`walkExprs`、`getResults`、`walk` 实现具体逻辑。

### Lines 497-510
```cpp
/// the result.  The returned map always has the same number of results.
AffineMap AffineMap::replaceDimsAndSymbols(ArrayRef<AffineExpr> dimReplacements,
                                           ArrayRef<AffineExpr> symReplacements,
                                           unsigned numResultDims,
                                           unsigned numResultSyms) const {
  SmallVector<AffineExpr, 8> results;
  results.reserve(getNumResults());
  for (auto expr : getResults())
    results.push_back(
        expr.replaceDimsAndSymbols(dimReplacements, symReplacements));
  return get(numResultDims, numResultSyms, results, getContext());
}

/// Sparse replace method. Apply AffineExpr::replace(`expr`, `replacement`) to
```
- **EN**: Implements logic around `replaceDimsAndSymbols`, `reserve`, `getResults`, `push_back`, and 1 more symbols.
- **CN**: 围绕 `replaceDimsAndSymbols`、`reserve`、`getResults`、`push_back` 等另外 1 个符号 实现具体逻辑。

### Lines 511-524
```cpp
/// each of the results and return a new AffineMap with the new results and
/// with the specified number of dims and symbols.
AffineMap AffineMap::replace(AffineExpr expr, AffineExpr replacement,
                             unsigned numResultDims,
                             unsigned numResultSyms) const {
  SmallVector<AffineExpr, 4> newResults;
  newResults.reserve(getNumResults());
  for (AffineExpr e : getResults())
    newResults.push_back(e.replace(expr, replacement));
  return AffineMap::get(numResultDims, numResultSyms, newResults, getContext());
}

/// Sparse replace method. Apply AffineExpr::replace(`map`) to each of the
/// results and return a new AffineMap with the new results and with the
```
- **EN**: Implements logic around `replace`, `reserve`, `getResults`, `push_back`, and 1 more symbols.
- **CN**: 围绕 `replace`、`reserve`、`getResults`、`push_back` 等另外 1 个符号 实现具体逻辑。

### Lines 525-544
```cpp
/// specified number of dims and symbols.
AffineMap AffineMap::replace(const DenseMap<AffineExpr, AffineExpr> &map,
                             unsigned numResultDims,
                             unsigned numResultSyms) const {
  SmallVector<AffineExpr, 4> newResults;
  newResults.reserve(getNumResults());
  for (AffineExpr e : getResults())
    newResults.push_back(e.replace(map));
  return AffineMap::get(numResultDims, numResultSyms, newResults, getContext());
}

AffineMap
AffineMap::replace(const DenseMap<AffineExpr, AffineExpr> &map) const {
  SmallVector<AffineExpr, 4> newResults;
  newResults.reserve(getNumResults());
  for (AffineExpr e : getResults())
    newResults.push_back(e.replace(map));
  return AffineMap::inferFromExprList(newResults, getContext()).front();
}

```
- **EN**: Implements logic around `replace`, `reserve`, `getResults`, `push_back`, and 2 more symbols.
- **CN**: 围绕 `replace`、`reserve`、`getResults`、`push_back` 等另外 2 个符号 实现具体逻辑。

### Lines 545-572
```cpp
AffineMap AffineMap::dropResults(const llvm::SmallBitVector &positions) const {
  auto exprs = llvm::to_vector<4>(getResults());
  // TODO: this is a pretty terrible API .. is there anything better?
  for (auto pos = positions.find_last(); pos != -1;
       pos = positions.find_prev(pos))
    exprs.erase(exprs.begin() + pos);
  return AffineMap::get(getNumDims(), getNumSymbols(), exprs, getContext());
}

AffineMap AffineMap::compose(AffineMap map) const {
  assert(getNumDims() == map.getNumResults() && "Number of results mismatch");
  // Prepare `map` by concatenating the symbols and rewriting its exprs.
  unsigned numDims = map.getNumDims();
  unsigned numSymbolsThisMap = getNumSymbols();
  unsigned numSymbols = numSymbolsThisMap + map.getNumSymbols();
  SmallVector<AffineExpr, 8> newDims(numDims);
  for (unsigned idx = 0; idx < numDims; ++idx) {
    newDims[idx] = getAffineDimExpr(idx, getContext());
  }
  SmallVector<AffineExpr, 8> newSymbols(numSymbols - numSymbolsThisMap);
  for (unsigned idx = numSymbolsThisMap; idx < numSymbols; ++idx) {
    newSymbols[idx - numSymbolsThisMap] =
        getAffineSymbolExpr(idx, getContext());
  }
  auto newMap =
      map.replaceDimsAndSymbols(newDims, newSymbols, numDims, numSymbols);
  SmallVector<AffineExpr, 8> exprs;
  exprs.reserve(getResults().size());
```
- **EN**: Implements logic around `dropResults`, `to_vector`, `find_last`, `find_prev`, and 12 more symbols.
- **CN**: 围绕 `dropResults`、`to_vector`、`find_last`、`find_prev` 等另外 12 个符号 实现具体逻辑。

### Lines 573-592
```cpp
  for (auto expr : getResults())
    exprs.push_back(expr.compose(newMap));
  return AffineMap::get(numDims, numSymbols, exprs, map.getContext());
}

// Inline size chosen empirically based on compilation profiling.
// Profiled: 43.5M calls, avg=3.1+-2.3. N=8 covers ~98% of cases inline.
SmallVector<int64_t, 8> AffineMap::compose(ArrayRef<int64_t> values) const {
  assert(getNumSymbols() == 0 && "Expected symbol-less map");
  SmallVector<AffineExpr, 8> exprs;
  MLIRContext *ctx = getContext();
  for (int64_t value : values)
    exprs.push_back(getAffineConstantExpr(value, ctx));
  SmallVector<int64_t, 8> res;
  res.reserve(getNumResults());
  for (auto e : getResults())
    res.push_back(cast<AffineConstantExpr>(e.replaceDims(exprs)).getValue());
  return res;
}

```
- **EN**: Implements logic around `getResults`, `push_back`, `get`, `compose`, and 3 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getResults`、`push_back`、`get`、`compose` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 593-606
```cpp
size_t AffineMap::getNumOfZeroResults() const {
  size_t res = 0;
  for (auto expr : getResults()) {
    auto constExpr = dyn_cast<AffineConstantExpr>(expr);
    if (constExpr && constExpr.getValue() == 0)
      res++;
  }

  return res;
}

AffineMap AffineMap::dropZeroResults() {
  SmallVector<AffineExpr> newExprs;

```
- **EN**: Implements logic around `getNumOfZeroResults`, `getResults`, `dyn_cast`, `getValue`, and 1 more symbols.
- **CN**: 围绕 `getNumOfZeroResults`、`getResults`、`dyn_cast`、`getValue` 等另外 1 个符号 实现具体逻辑。

### Lines 607-623
```cpp
  for (auto expr : getResults()) {
    auto constExpr = dyn_cast<AffineConstantExpr>(expr);
    if (!constExpr || constExpr.getValue() != 0)
      newExprs.push_back(expr);
  }
  return AffineMap::get(getNumDims(), getNumSymbols(), newExprs, getContext());
}

bool AffineMap::isProjectedPermutation(bool allowZeroInResults) const {
  if (getNumSymbols() > 0)
    return false;

  // Having more results than inputs means that results have duplicated dims or
  // zeros that can't be mapped to input dims.
  if (getNumResults() > getNumInputs())
    return false;

```
- **EN**: Implements logic around `getResults`, `dyn_cast`, `getValue`, `push_back`, and 4 more symbols.
- **CN**: 围绕 `getResults`、`dyn_cast`、`getValue`、`push_back` 等另外 4 个符号 实现具体逻辑。

### Lines 624-640
```cpp
  SmallVector<bool, 8> seen(getNumInputs(), false);
  // A projected permutation can have, at most, only one instance of each input
  // dimension in the result expressions. Zeros are allowed as long as the
  // number of result expressions is lower or equal than the number of input
  // expressions.
  for (auto expr : getResults()) {
    if (auto dim = dyn_cast<AffineDimExpr>(expr)) {
      if (seen[dim.getPosition()])
        return false;
      seen[dim.getPosition()] = true;
    } else {
      auto constExpr = dyn_cast<AffineConstantExpr>(expr);
      if (!allowZeroInResults || !constExpr || constExpr.getValue() != 0)
        return false;
    }
  }

```
- **EN**: Implements logic around `seen`, `getResults`, `dyn_cast`, `getPosition`, and 1 more symbols.
- **CN**: 围绕 `seen`、`getResults`、`dyn_cast`、`getPosition` 等另外 1 个符号 实现具体逻辑。

### Lines 641-658
```cpp
  // Results are either dims or zeros and zeros can be mapped to input dims.
  return true;
}

bool AffineMap::isPermutation() const {
  if (getNumDims() != getNumResults())
    return false;
  return isProjectedPermutation();
}

AffineMap AffineMap::getSubMap(ArrayRef<unsigned> resultPos) const {
  SmallVector<AffineExpr, 4> exprs;
  exprs.reserve(resultPos.size());
  for (auto idx : resultPos)
    exprs.push_back(getResult(idx));
  return AffineMap::get(getNumDims(), getNumSymbols(), exprs, getContext());
}

```
- **EN**: Implements logic around `isPermutation`, `getNumDims`, `isProjectedPermutation`, `getSubMap`, and 3 more symbols.
- **CN**: 围绕 `isPermutation`、`getNumDims`、`isProjectedPermutation`、`getSubMap` 等另外 3 个符号 实现具体逻辑。

### Lines 659-679
```cpp
AffineMap AffineMap::getSliceMap(unsigned start, unsigned length) const {
  return AffineMap::get(getNumDims(), getNumSymbols(),
                        getResults().slice(start, length), getContext());
}

AffineMap AffineMap::getMajorSubMap(unsigned numResults) const {
  if (numResults == 0)
    return AffineMap();
  if (numResults > getNumResults())
    return *this;
  return getSliceMap(0, numResults);
}

AffineMap AffineMap::getMinorSubMap(unsigned numResults) const {
  if (numResults == 0)
    return AffineMap();
  if (numResults > getNumResults())
    return *this;
  return getSliceMap(getNumResults() - numResults, numResults);
}

```
- **EN**: Implements logic around `getSliceMap`, `get`, `getResults`, `getMajorSubMap`, and 3 more symbols.
- **CN**: 围绕 `getSliceMap`、`get`、`getResults`、`getMajorSubMap` 等另外 3 个符号 实现具体逻辑。

### Lines 680-707
```cpp
/// Implementation detail to compress multiple affine maps with a compressionFun
/// that is expected to be either compressUnusedDims or compressUnusedSymbols.
/// The implementation keeps track of num dims and symbols across the different
/// affine maps.
static SmallVector<AffineMap> compressUnusedListImpl(
    ArrayRef<AffineMap> maps,
    llvm::function_ref<AffineMap(AffineMap)> compressionFun) {
  if (maps.empty())
    return SmallVector<AffineMap>();
  SmallVector<AffineExpr> allExprs;
  allExprs.reserve(maps.size() * maps.front().getNumResults());
  unsigned numDims = maps.front().getNumDims(),
           numSymbols = maps.front().getNumSymbols();
  for (auto m : maps) {
    assert(numDims == m.getNumDims() && numSymbols == m.getNumSymbols() &&
           "expected maps with same num dims and symbols");
    llvm::append_range(allExprs, m.getResults());
  }
  AffineMap unifiedMap = compressionFun(
      AffineMap::get(numDims, numSymbols, allExprs, maps.front().getContext()));
  unsigned unifiedNumDims = unifiedMap.getNumDims(),
           unifiedNumSymbols = unifiedMap.getNumSymbols();
  ArrayRef<AffineExpr> unifiedResults = unifiedMap.getResults();
  SmallVector<AffineMap> res;
  res.reserve(maps.size());
  for (auto m : maps) {
    res.push_back(AffineMap::get(unifiedNumDims, unifiedNumSymbols,
                                 unifiedResults.take_front(m.getNumResults()),
```
- **EN**: Implements logic around `compressUnusedListImpl`, `function_ref`, `empty`, `SmallVector`, and 11 more symbols.
- **CN**: 围绕 `compressUnusedListImpl`、`function_ref`、`empty`、`SmallVector` 等另外 11 个符号 实现具体逻辑。

### Lines 708-722
```cpp
                                 m.getContext()));
    unifiedResults = unifiedResults.drop_front(m.getNumResults());
  }
  return res;
}

AffineMap mlir::compressDims(AffineMap map,
                             const llvm::SmallBitVector &unusedDims) {
  return projectDims(map, unusedDims, /*compressDimsFlag=*/true);
}

AffineMap mlir::compressUnusedDims(AffineMap map) {
  return compressDims(map, getUnusedDimsBitVector({map}));
}

```
- **EN**: Implements logic around `getContext`, `drop_front`, `compressDims`, `projectDims`, and 1 more symbols.
- **CN**: 围绕 `getContext`、`drop_front`、`compressDims`、`projectDims` 等另外 1 个符号 实现具体逻辑。

### Lines 723-736
```cpp
SmallVector<AffineMap> mlir::compressUnusedDims(ArrayRef<AffineMap> maps) {
  return compressUnusedListImpl(
      maps, [](AffineMap m) { return compressUnusedDims(m); });
}

AffineMap mlir::compressSymbols(AffineMap map,
                                const llvm::SmallBitVector &unusedSymbols) {
  return projectSymbols(map, unusedSymbols, /*compressSymbolsFlag=*/true);
}

AffineMap mlir::compressUnusedSymbols(AffineMap map) {
  return compressSymbols(map, getUnusedSymbolsBitVector({map}));
}

```
- **EN**: Implements logic around `compressUnusedDims`, `compressUnusedListImpl`, `compressSymbols`, `projectSymbols`, and 1 more symbols.
- **CN**: 围绕 `compressUnusedDims`、`compressUnusedListImpl`、`compressSymbols`、`projectSymbols` 等另外 1 个符号 实现具体逻辑。

### Lines 737-764
```cpp
SmallVector<AffineMap> mlir::compressUnusedSymbols(ArrayRef<AffineMap> maps) {
  return compressUnusedListImpl(
      maps, [](AffineMap m) { return compressUnusedSymbols(m); });
}

AffineMap mlir::foldAttributesIntoMap(Builder &b, AffineMap map,
                                      ArrayRef<OpFoldResult> operands,
                                      SmallVector<Value> &remainingValues) {
  SmallVector<AffineExpr> dimReplacements, symReplacements;
  int64_t numDims = 0;
  for (int64_t i = 0; i < map.getNumDims(); ++i) {
    if (auto attr = dyn_cast<Attribute>(operands[i])) {
      dimReplacements.push_back(
          b.getAffineConstantExpr(cast<IntegerAttr>(attr).getInt()));
    } else {
      dimReplacements.push_back(b.getAffineDimExpr(numDims++));
      remainingValues.push_back(cast<Value>(operands[i]));
    }
  }
  int64_t numSymbols = 0;
  for (int64_t i = 0; i < map.getNumSymbols(); ++i) {
    if (auto attr = dyn_cast<Attribute>(operands[i + map.getNumDims()])) {
      symReplacements.push_back(
          b.getAffineConstantExpr(cast<IntegerAttr>(attr).getInt()));
    } else {
      symReplacements.push_back(b.getAffineSymbolExpr(numSymbols++));
      remainingValues.push_back(cast<Value>(operands[i + map.getNumDims()]));
    }
```
- **EN**: Implements logic around `compressUnusedSymbols`, `compressUnusedListImpl`, `foldAttributesIntoMap`, `getNumDims`, and 4 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `compressUnusedSymbols`、`compressUnusedListImpl`、`foldAttributesIntoMap`、`getNumDims` 等另外 4 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 765-779
```cpp
  }
  return map.replaceDimsAndSymbols(dimReplacements, symReplacements, numDims,
                                   numSymbols);
}

AffineMap mlir::simplifyAffineMap(AffineMap map) {
  SmallVector<AffineExpr, 8> exprs;
  for (auto e : map.getResults()) {
    exprs.push_back(
        simplifyAffineExpr(e, map.getNumDims(), map.getNumSymbols()));
  }
  return AffineMap::get(map.getNumDims(), map.getNumSymbols(), exprs,
                        map.getContext());
}

```
- **EN**: Implements logic around `replaceDimsAndSymbols`, `simplifyAffineMap`, `getResults`, `push_back`, and 3 more symbols.
- **CN**: 围绕 `replaceDimsAndSymbols`、`simplifyAffineMap`、`getResults`、`push_back` 等另外 3 个符号 实现具体逻辑。

### Lines 780-807
```cpp
AffineMap mlir::removeDuplicateExprs(AffineMap map) {
  auto results = map.getResults();
  SmallVector<AffineExpr, 4> uniqueExprs(results);
  uniqueExprs.erase(llvm::unique(uniqueExprs), uniqueExprs.end());
  return AffineMap::get(map.getNumDims(), map.getNumSymbols(), uniqueExprs,
                        map.getContext());
}

AffineMap mlir::inversePermutation(AffineMap map) {
  if (map.isEmpty())
    return map;
  assert(map.getNumSymbols() == 0 && "expected map without symbols");
  SmallVector<AffineExpr, 4> exprs(map.getNumDims());
  for (const auto &en : llvm::enumerate(map.getResults())) {
    auto expr = en.value();
    // Skip non-permutations.
    if (auto d = dyn_cast<AffineDimExpr>(expr)) {
      if (exprs[d.getPosition()])
        continue;
      exprs[d.getPosition()] = getAffineDimExpr(en.index(), d.getContext());
    }
  }
  SmallVector<AffineExpr, 4> seenExprs;
  seenExprs.reserve(map.getNumDims());
  for (auto expr : exprs)
    if (expr)
      seenExprs.push_back(expr);
  if (seenExprs.size() != map.getNumInputs())
```
- **EN**: Implements logic around `removeDuplicateExprs`, `getResults`, `uniqueExprs`, `erase`, and 13 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `removeDuplicateExprs`、`getResults`、`uniqueExprs`、`erase` 等另外 13 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 808-826
```cpp
    return AffineMap();
  return AffineMap::get(map.getNumResults(), 0, seenExprs, map.getContext());
}

AffineMap mlir::inverseAndBroadcastProjectedPermutation(AffineMap map) {
  assert(map.isProjectedPermutation(/*allowZeroInResults=*/true));
  MLIRContext *context = map.getContext();
  AffineExpr zero = mlir::getAffineConstantExpr(0, context);
  // Start with all the results as 0.
  SmallVector<AffineExpr, 4> exprs(map.getNumInputs(), zero);
  for (unsigned i : llvm::seq(unsigned(0), map.getNumResults())) {
    // Skip zeros from input map. 'exprs' is already initialized to zero.
    if (auto constExpr = dyn_cast<AffineConstantExpr>(map.getResult(i))) {
      assert(constExpr.getValue() == 0 &&
             "Unexpected constant in projected permutation");
      (void)constExpr;
      continue;
    }

```
- **EN**: Implements logic around `AffineMap`, `get`, `inverseAndBroadcastProjectedPermutation`, `assert`, and 5 more symbols.
- **CN**: 围绕 `AffineMap`、`get`、`inverseAndBroadcastProjectedPermutation`、`assert` 等另外 5 个符号 实现具体逻辑。

### Lines 827-845
```cpp
    // Reverse each dimension existing in the original map result.
    exprs[map.getDimPosition(i)] = getAffineDimExpr(i, context);
  }
  return AffineMap::get(map.getNumResults(), /*symbolCount=*/0, exprs, context);
}

AffineMap mlir::concatAffineMaps(ArrayRef<AffineMap> maps,
                                 MLIRContext *context) {
  if (maps.empty())
    return AffineMap::get(context);
  unsigned numResults = 0, numDims = 0, numSymbols = 0;
  for (auto m : maps)
    numResults += m.getNumResults();
  SmallVector<AffineExpr, 8> results;
  results.reserve(numResults);
  for (auto m : maps) {
    for (auto res : m.getResults())
      results.push_back(res.shiftSymbols(m.getNumSymbols(), numSymbols));

```
- **EN**: Implements logic around `getDimPosition`, `get`, `concatAffineMaps`, `empty`, and 4 more symbols.
- **CN**: 围绕 `getDimPosition`、`get`、`concatAffineMaps`、`empty` 等另外 4 个符号 实现具体逻辑。

### Lines 846-863
```cpp
    numSymbols += m.getNumSymbols();
    numDims = std::max(m.getNumDims(), numDims);
  }
  return AffineMap::get(numDims, numSymbols, results, context);
}

/// Common implementation to project out dimensions or symbols from an affine
/// map based on the template type.
/// Additionally, if 'compress' is true, the projected out dimensions or symbols
/// are also dropped from the resulting map.
template <typename AffineDimOrSymExpr>
static AffineMap projectCommonImpl(AffineMap map,
                                   const llvm::SmallBitVector &toProject,
                                   bool compress) {
  static_assert(llvm::is_one_of<AffineDimOrSymExpr, AffineDimExpr,
                                AffineSymbolExpr>::value,
                "expected AffineDimExpr or AffineSymbolExpr");

```
- **EN**: Implements logic around `getNumSymbols`, `max`, `get`, `projectCommonImpl`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getNumSymbols`、`max`、`get`、`projectCommonImpl` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 864-882
```cpp
  constexpr bool isDim = std::is_same<AffineDimOrSymExpr, AffineDimExpr>::value;
  int64_t numDimOrSym = (isDim) ? map.getNumDims() : map.getNumSymbols();
  SmallVector<AffineExpr> replacements;
  replacements.reserve(numDimOrSym);

  auto createNewDimOrSym = (isDim) ? getAffineDimExpr : getAffineSymbolExpr;

  using replace_fn_ty =
      std::function<AffineExpr(AffineExpr, ArrayRef<AffineExpr>)>;
  replace_fn_ty replaceDims = [](AffineExpr e,
                                 ArrayRef<AffineExpr> replacements) {
    return e.replaceDims(replacements);
  };
  replace_fn_ty replaceSymbols = [](AffineExpr e,
                                    ArrayRef<AffineExpr> replacements) {
    return e.replaceSymbols(replacements);
  };
  replace_fn_ty replaceNewDimOrSym = (isDim) ? replaceDims : replaceSymbols;

```
- **EN**: Implements logic around `getNumDims`, `reserve`, `function`, `replaceDims`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getNumDims`、`reserve`、`function`、`replaceDims` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 883-897
```cpp
  MLIRContext *context = map.getContext();
  int64_t newNumDimOrSym = 0;
  for (unsigned dimOrSym = 0; dimOrSym < numDimOrSym; ++dimOrSym) {
    if (toProject.test(dimOrSym)) {
      replacements.push_back(getAffineConstantExpr(0, context));
      continue;
    }
    int64_t newPos = compress ? newNumDimOrSym++ : dimOrSym;
    replacements.push_back(createNewDimOrSym(newPos, context));
  }
  SmallVector<AffineExpr> resultExprs;
  resultExprs.reserve(map.getNumResults());
  for (auto e : map.getResults())
    resultExprs.push_back(replaceNewDimOrSym(e, replacements));

```
- **EN**: Implements logic around `getContext`, `test`, `push_back`, `reserve`, and 1 more symbols.
- **CN**: 围绕 `getContext`、`test`、`push_back`、`reserve` 等另外 1 个符号 实现具体逻辑。

### Lines 898-916
```cpp
  int64_t numDims = (compress && isDim) ? newNumDimOrSym : map.getNumDims();
  int64_t numSyms = (compress && !isDim) ? newNumDimOrSym : map.getNumSymbols();
  return AffineMap::get(numDims, numSyms, resultExprs, context);
}

AffineMap mlir::projectDims(AffineMap map,
                            const llvm::SmallBitVector &projectedDimensions,
                            bool compressDimsFlag) {
  return projectCommonImpl<AffineDimExpr>(map, projectedDimensions,
                                          compressDimsFlag);
}

AffineMap mlir::projectSymbols(AffineMap map,
                               const llvm::SmallBitVector &projectedSymbols,
                               bool compressSymbolsFlag) {
  return projectCommonImpl<AffineSymbolExpr>(map, projectedSymbols,
                                             compressSymbolsFlag);
}

```
- **EN**: Implements logic around `getNumDims`, `getNumSymbols`, `get`, `projectDims`, and 2 more symbols.
- **CN**: 围绕 `getNumDims`、`getNumSymbols`、`get`、`projectDims` 等另外 2 个符号 实现具体逻辑。

### Lines 917-938
```cpp
AffineMap mlir::getProjectedMap(AffineMap map,
                                const llvm::SmallBitVector &projectedDimensions,
                                bool compressDimsFlag,
                                bool compressSymbolsFlag) {
  map = projectDims(map, projectedDimensions, compressDimsFlag);
  if (compressSymbolsFlag)
    map = compressUnusedSymbols(map);
  return map;
}

llvm::SmallBitVector mlir::getUnusedDimsBitVector(ArrayRef<AffineMap> maps) {
  unsigned numDims = maps[0].getNumDims();
  llvm::SmallBitVector numDimsBitVector(numDims, true);
  for (AffineMap m : maps) {
    for (unsigned i = 0; i < numDims; ++i) {
      if (m.isFunctionOfDim(i))
        numDimsBitVector.reset(i);
    }
  }
  return numDimsBitVector;
}

```
- **EN**: Implements logic around `getProjectedMap`, `projectDims`, `compressUnusedSymbols`, `getUnusedDimsBitVector`, and 4 more symbols.
- **CN**: 围绕 `getProjectedMap`、`projectDims`、`compressUnusedSymbols`、`getUnusedDimsBitVector` 等另外 4 个符号 实现具体逻辑。

### Lines 939-958
```cpp
llvm::SmallBitVector mlir::getUnusedSymbolsBitVector(ArrayRef<AffineMap> maps) {
  unsigned numSymbols = maps[0].getNumSymbols();
  llvm::SmallBitVector numSymbolsBitVector(numSymbols, true);
  for (AffineMap m : maps) {
    for (unsigned i = 0; i < numSymbols; ++i) {
      if (m.isFunctionOfSymbol(i))
        numSymbolsBitVector.reset(i);
    }
  }
  return numSymbolsBitVector;
}

AffineMap
mlir::expandDimsToRank(AffineMap map, int64_t rank,
                       const llvm::SmallBitVector &projectedDimensions) {
  auto id = AffineMap::getMultiDimIdentityMap(rank, map.getContext());
  AffineMap proj = id.dropResults(projectedDimensions);
  return map.compose(proj);
}

```
- **EN**: Implements logic around `getUnusedSymbolsBitVector`, `getNumSymbols`, `numSymbolsBitVector`, `isFunctionOfSymbol`, and 5 more symbols.
- **CN**: 围绕 `getUnusedSymbolsBitVector`、`getNumSymbols`、`numSymbolsBitVector`、`isFunctionOfSymbol` 等另外 5 个符号 实现具体逻辑。

### Lines 959-974
```cpp
//===----------------------------------------------------------------------===//
// MutableAffineMap.
//===----------------------------------------------------------------------===//

MutableAffineMap::MutableAffineMap(AffineMap map)
    : results(map.getResults()), numDims(map.getNumDims()),
      numSymbols(map.getNumSymbols()), context(map.getContext()) {}

void MutableAffineMap::reset(AffineMap map) {
  results.clear();
  numDims = map.getNumDims();
  numSymbols = map.getNumSymbols();
  context = map.getContext();
  llvm::append_range(results, map.getResults());
}

```
- **EN**: Implements logic around `MutableAffineMap`, `results`, `numSymbols`, `reset`, and 5 more symbols.
- **CN**: 围绕 `MutableAffineMap`、`results`、`numSymbols`、`reset` 等另外 5 个符号 实现具体逻辑。

### Lines 975-988
```cpp
bool MutableAffineMap::isMultipleOf(unsigned idx, int64_t factor) const {
  return results[idx].isMultipleOf(factor);
}

// Simplifies the result affine expressions of this map. The expressions
// have to be pure for the simplification implemented.
void MutableAffineMap::simplify() {
  // Simplify each of the results if possible.
  // TODO: functional-style map
  for (unsigned i = 0, e = getNumResults(); i < e; i++) {
    results[i] = simplifyAffineExpr(getResult(i), numDims, numSymbols);
  }
}

```
- **EN**: Implements logic around `isMultipleOf`, `simplify`, `getNumResults`, `simplifyAffineExpr`.
- **CN**: 围绕 `isMultipleOf`、`simplify`、`getNumResults`、`simplifyAffineExpr` 实现具体逻辑。

### Lines 989-991
```cpp
AffineMap MutableAffineMap::getAffineMap() const {
  return AffineMap::get(numDims, numSymbols, results, context);
}
```
- **EN**: Implements logic around `getAffineMap`, `get`.
- **CN**: 围绕 `getAffineMap`、`get` 实现具体逻辑。

## Key Concepts / 关键概念

- **Core IR modeling / 核心 IR 建模**:
  - **EN**: Defines operations, regions, blocks, attributes, types, and the assembly form that ties them together.
  - **CN**: 定义操作、区域、块、属性、类型，以及将它们串联起来的汇编表示。
- **SSA and region-based IR / SSA 与基于 Region 的 IR**:
  - **EN**: The file works with MLIR operations, blocks, values, or regions that form MLIR's hierarchical SSA IR.
  - **CN**: 该文件处理 MLIR 操作、块、值或 Region，它们共同构成 MLIR 的层次化 SSA IR。
- **Structured diagnostics / 结构化诊断**:
  - **EN**: Failures are surfaced through `LogicalResult`, diagnostics, or related reporting helpers.
  - **CN**: 通过 `LogicalResult`、诊断对象或相关报告工具显式传播失败。
- **LLVM ADT containers / LLVM ADT 容器**:
  - **EN**: The implementation uses LLVM containers for performance-conscious storage and traversal.
  - **CN**: 该实现使用 LLVM 容器来进行兼顾性能的存储与遍历。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `mlir/IR/AffineMap.h`, `AffineMapDetail.h`, `mlir/IR/AffineExpr.h`, `mlir/IR/Builders.h`, `mlir/IR/BuiltinAttributes.h`, `mlir/IR/BuiltinTypes.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallBitVector.h`, `llvm/ADT/SmallVector.h`, `llvm/Support/MathExtras.h`
- **Standard-library headers / 标准库头文件**: `<numeric>`, `<optional>`, `<type_traits>`
- **Subsystem categories / 子系统类别**: core MLIR IR abstractions / 核心 MLIR IR 抽象 (5), LLVM ADT containers and utilities / LLVM ADT 容器与工具 (3), LLVM support-library helpers / LLVM Support 库辅助工具 (1)
