# InferIntRangeCommon.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Interfaces/Utils/InferIntRangeCommon.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file contains implementations of range inference for operations that are common to both the `arith` and `index` dialects to facilitate reuse.
  - **CN**: 实现 MLIR 接口或生成的接口适配胶水代码。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```cpp
//===- InferIntRangeCommon.cpp - Inference for common ops ------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains implementations of range inference for operations that are
// common to both the `arith` and `index` dialects to facilitate reuse.
//
//===----------------------------------------------------------------------===//

#include "mlir/Interfaces/Utils/InferIntRangeCommon.h"

```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/Interfaces/Utils/InferIntRangeCommon.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/Interfaces/Utils/InferIntRangeCommon.h`。

### Lines 16-29
```cpp
#include "mlir/IR/AffineExpr.h"
#include "mlir/Interfaces/InferIntRangeInterface.h"
#include "mlir/Interfaces/ShapedOpInterfaces.h"

#include "llvm/ADT/ArrayRef.h"
#include "llvm/ADT/STLExtras.h"

#include "llvm/Support/Debug.h"

#include <iterator>
#include <optional>

using namespace mlir;

```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/IR/AffineExpr.h`, `mlir/Interfaces/InferIntRangeInterface.h`, `mlir/Interfaces/ShapedOpInterfaces.h`, `llvm/ADT/ArrayRef.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/IR/AffineExpr.h`, `mlir/Interfaces/InferIntRangeInterface.h`, `mlir/Interfaces/ShapedOpInterfaces.h`, `llvm/ADT/ArrayRef.h`。

### Lines 30-43
```cpp
#define DEBUG_TYPE "int-range-analysis"

//===----------------------------------------------------------------------===//
// General utilities
//===----------------------------------------------------------------------===//

/// Function that evaluates the result of doing something on arithmetic
/// constants and returns std::nullopt on overflow.
using ConstArithFn =
    function_ref<std::optional<APInt>(const APInt &, const APInt &)>;
using ConstArithStdFn =
    std::function<std::optional<APInt>(const APInt &, const APInt &)>;

/// Compute op(minLeft, minRight) and op(maxLeft, maxRight) if possible,
```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 44-57
```cpp
/// If either computation overflows, make the result unbounded.
static ConstantIntRanges computeBoundsBy(ConstArithFn op, const APInt &minLeft,
                                         const APInt &minRight,
                                         const APInt &maxLeft,
                                         const APInt &maxRight, bool isSigned) {
  std::optional<APInt> maybeMin = op(minLeft, minRight);
  std::optional<APInt> maybeMax = op(maxLeft, maxRight);
  if (maybeMin && maybeMax)
    return ConstantIntRanges::range(*maybeMin, *maybeMax, isSigned);
  return ConstantIntRanges::maxRange(minLeft.getBitWidth());
}

/// Compute the minimum and maximum of `(op(l, r) for l in lhs for r in rhs)`,
/// ignoring unbounded values. Returns the maximal range if `op` overflows.
```
- **EN**: Implements logic around `computeBoundsBy`, `op`, `range`, `maxRange`.
- **CN**: 围绕 `computeBoundsBy`、`op`、`range`、`maxRange` 实现具体逻辑。

### Lines 58-77
```cpp
static ConstantIntRanges minMaxBy(ConstArithFn op, ArrayRef<APInt> lhs,
                                  ArrayRef<APInt> rhs, bool isSigned) {
  unsigned width = lhs[0].getBitWidth();
  APInt min =
      isSigned ? APInt::getSignedMaxValue(width) : APInt::getMaxValue(width);
  APInt max =
      isSigned ? APInt::getSignedMinValue(width) : APInt::getZero(width);
  for (const APInt &left : lhs) {
    for (const APInt &right : rhs) {
      std::optional<APInt> maybeThisResult = op(left, right);
      if (!maybeThisResult)
        return ConstantIntRanges::maxRange(width);
      APInt result = std::move(*maybeThisResult);
      min = (isSigned ? result.slt(min) : result.ult(min)) ? result : min;
      max = (isSigned ? result.sgt(max) : result.ugt(max)) ? result : max;
    }
  }
  return ConstantIntRanges::range(min, max, isSigned);
}

```
- **EN**: Implements logic around `minMaxBy`, `getBitWidth`, `getSignedMaxValue`, `getSignedMinValue`, and 6 more symbols.
- **CN**: 围绕 `minMaxBy`、`getBitWidth`、`getSignedMaxValue`、`getSignedMinValue` 等另外 6 个符号 实现具体逻辑。

### Lines 78-97
```cpp
//===----------------------------------------------------------------------===//
// Ext, trunc, index op handling
//===----------------------------------------------------------------------===//

ConstantIntRanges
mlir::intrange::inferIndexOp(const InferRangeFn &inferFn,
                             ArrayRef<ConstantIntRanges> argRanges,
                             intrange::CmpMode mode) {
  ConstantIntRanges sixtyFour = inferFn(argRanges);
  SmallVector<ConstantIntRanges, 2> truncated;
  llvm::transform(argRanges, std::back_inserter(truncated),
                  [](const ConstantIntRanges &range) {
                    return truncRange(range, /*destWidth=*/indexMinWidth);
                  });
  ConstantIntRanges thirtyTwo = inferFn(truncated);
  ConstantIntRanges thirtyTwoAsSixtyFour =
      extRange(thirtyTwo, /*destWidth=*/indexMaxWidth);
  ConstantIntRanges sixtyFourAsThirtyTwo =
      truncRange(sixtyFour, /*destWidth=*/indexMinWidth);

```
- **EN**: Implements logic around `inferIndexOp`, `inferFn`, `transform`, `truncRange`, and 1 more symbols.
- **CN**: 围绕 `inferIndexOp`、`inferFn`、`transform`、`truncRange` 等另外 1 个符号 实现具体逻辑。

### Lines 98-120
```cpp
  LLVM_DEBUG(llvm::dbgs() << "Index handling: 64-bit result = " << sixtyFour
                          << " 32-bit = " << thirtyTwo << "\n");
  bool truncEqual = false;
  switch (mode) {
  case intrange::CmpMode::Both:
    truncEqual = (thirtyTwo == sixtyFourAsThirtyTwo);
    break;
  case intrange::CmpMode::Signed:
    truncEqual = (thirtyTwo.smin() == sixtyFourAsThirtyTwo.smin() &&
                  thirtyTwo.smax() == sixtyFourAsThirtyTwo.smax());
    break;
  case intrange::CmpMode::Unsigned:
    truncEqual = (thirtyTwo.umin() == sixtyFourAsThirtyTwo.umin() &&
                  thirtyTwo.umax() == sixtyFourAsThirtyTwo.umax());
    break;
  }
  if (truncEqual)
    // Returing the 64-bit result preserves more information.
    return sixtyFour;
  ConstantIntRanges merged = sixtyFour.rangeUnion(thirtyTwoAsSixtyFour);
  return merged;
}

```
- **EN**: Implements logic around `dbgs`, `smin`, `smax`, `umin`, and 2 more symbols.
- **CN**: 围绕 `dbgs`、`smin`、`smax`、`umin` 等另外 2 个符号 实现具体逻辑。

### Lines 121-136
```cpp
ConstantIntRanges mlir::intrange::extRange(const ConstantIntRanges &range,
                                           unsigned int destWidth) {
  APInt umin = range.umin().zext(destWidth);
  APInt umax = range.umax().zext(destWidth);
  APInt smin = range.smin().sext(destWidth);
  APInt smax = range.smax().sext(destWidth);
  return {umin, umax, smin, smax};
}

ConstantIntRanges mlir::intrange::extUIRange(const ConstantIntRanges &range,
                                             unsigned destWidth) {
  APInt umin = range.umin().zext(destWidth);
  APInt umax = range.umax().zext(destWidth);
  return ConstantIntRanges::fromUnsigned(umin, umax);
}

```
- **EN**: Implements logic around `extRange`, `umin`, `umax`, `smin`, and 3 more symbols.
- **CN**: 围绕 `extRange`、`umin`、`umax`、`smin` 等另外 3 个符号 实现具体逻辑。

### Lines 137-156
```cpp
ConstantIntRanges mlir::intrange::extSIRange(const ConstantIntRanges &range,
                                             unsigned destWidth) {
  APInt smin = range.smin().sext(destWidth);
  APInt smax = range.smax().sext(destWidth);
  return ConstantIntRanges::fromSigned(smin, smax);
}

ConstantIntRanges mlir::intrange::truncRange(const ConstantIntRanges &range,
                                             unsigned int destWidth) {
  // If you truncate the first four bytes in [0xaaaabbbb, 0xccccbbbb],
  // the range of the resulting value is not contiguous ind includes 0.
  // Ex. If you truncate [256, 258] from i16 to i8, you validly get [0, 2],
  // but you can't truncate [255, 257] similarly.
  bool hasUnsignedRollover =
      range.umin().lshr(destWidth) != range.umax().lshr(destWidth);
  APInt umin = hasUnsignedRollover ? APInt::getZero(destWidth)
                                   : range.umin().trunc(destWidth);
  APInt umax = hasUnsignedRollover ? APInt::getMaxValue(destWidth)
                                   : range.umax().trunc(destWidth);

```
- **EN**: Implements logic around `extSIRange`, `smin`, `smax`, `fromSigned`, and 5 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `extSIRange`、`smin`、`smax`、`fromSigned` 等另外 5 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 157-179
```cpp
  // Signed post-truncation rollover will not occur when either:
  // - The high parts of the min and max, plus the sign bit, are the same
  // - The high halves + sign bit of the min and max are either all 1s or all 0s
  //  and you won't create a [positive, negative] range by truncating.
  // For example, you can truncate the ranges [256, 258]_i16 to [0, 2]_i8
  // but not [255, 257]_i16 to a range of i8s. You can also truncate
  // [-256, -256]_i16 to [-2, 0]_i8, but not [-257, -255]_i16.
  // You can also truncate [-130, 0]_i16 to i8 because -130_i16 (0xff7e)
  // will truncate to 0x7e, which is greater than 0
  APInt sminHighPart = range.smin().ashr(destWidth - 1);
  APInt smaxHighPart = range.smax().ashr(destWidth - 1);
  bool hasSignedOverflow =
      (sminHighPart != smaxHighPart) &&
      !(sminHighPart.isAllOnes() &&
        (smaxHighPart.isAllOnes() || smaxHighPart.isZero())) &&
      !(sminHighPart.isZero() && smaxHighPart.isZero());
  APInt smin = hasSignedOverflow ? APInt::getSignedMinValue(destWidth)
                                 : range.smin().trunc(destWidth);
  APInt smax = hasSignedOverflow ? APInt::getSignedMaxValue(destWidth)
                                 : range.smax().trunc(destWidth);
  return {umin, umax, smin, smax};
}

```
- **EN**: Implements logic around `smin`, `smax`, `isAllOnes`, `isZero`, and 2 more symbols.
- **CN**: 围绕 `smin`、`smax`、`isAllOnes`、`isZero` 等另外 2 个符号 实现具体逻辑。

### Lines 180-205
```cpp
//===----------------------------------------------------------------------===//
// Addition
//===----------------------------------------------------------------------===//

ConstantIntRanges
mlir::intrange::inferAdd(ArrayRef<ConstantIntRanges> argRanges,
                         OverflowFlags ovfFlags) {
  const ConstantIntRanges &lhs = argRanges[0], &rhs = argRanges[1];

  ConstArithStdFn uadd = [=](const APInt &a,
                             const APInt &b) -> std::optional<APInt> {
    bool overflowed = false;
    APInt result = any(ovfFlags & OverflowFlags::Nuw)
                       ? a.uadd_sat(b)
                       : a.uadd_ov(b, overflowed);
    return overflowed ? std::optional<APInt>() : result;
  };
  ConstArithStdFn sadd = [=](const APInt &a,
                             const APInt &b) -> std::optional<APInt> {
    bool overflowed = false;
    APInt result = any(ovfFlags & OverflowFlags::Nsw)
                       ? a.sadd_sat(b)
                       : a.sadd_ov(b, overflowed);
    return overflowed ? std::optional<APInt>() : result;
  };

```
- **EN**: Implements logic around `inferAdd`, `any`, `uadd_sat`, `uadd_ov`, and 3 more symbols.
- **CN**: 围绕 `inferAdd`、`any`、`uadd_sat`、`uadd_ov` 等另外 3 个符号 实现具体逻辑。

### Lines 206-221
```cpp
  ConstantIntRanges urange = computeBoundsBy(
      uadd, lhs.umin(), rhs.umin(), lhs.umax(), rhs.umax(), /*isSigned=*/false);
  ConstantIntRanges srange = computeBoundsBy(
      sadd, lhs.smin(), rhs.smin(), lhs.smax(), rhs.smax(), /*isSigned=*/true);
  return urange.intersection(srange);
}

//===----------------------------------------------------------------------===//
// Subtraction
//===----------------------------------------------------------------------===//

ConstantIntRanges
mlir::intrange::inferSub(ArrayRef<ConstantIntRanges> argRanges,
                         OverflowFlags ovfFlags) {
  const ConstantIntRanges &lhs = argRanges[0], &rhs = argRanges[1];

```
- **EN**: Implements logic around `computeBoundsBy`, `umin`, `smin`, `intersection`, and 1 more symbols.
- **CN**: 围绕 `computeBoundsBy`、`umin`、`smin`、`intersection` 等另外 1 个符号 实现具体逻辑。

### Lines 222-244
```cpp
  ConstArithStdFn usub = [=](const APInt &a,
                             const APInt &b) -> std::optional<APInt> {
    bool overflowed = false;
    APInt result = any(ovfFlags & OverflowFlags::Nuw)
                       ? a.usub_sat(b)
                       : a.usub_ov(b, overflowed);
    return overflowed ? std::optional<APInt>() : result;
  };
  ConstArithStdFn ssub = [=](const APInt &a,
                             const APInt &b) -> std::optional<APInt> {
    bool overflowed = false;
    APInt result = any(ovfFlags & OverflowFlags::Nsw)
                       ? a.ssub_sat(b)
                       : a.ssub_ov(b, overflowed);
    return overflowed ? std::optional<APInt>() : result;
  };
  ConstantIntRanges urange = computeBoundsBy(
      usub, lhs.umin(), rhs.umax(), lhs.umax(), rhs.umin(), /*isSigned=*/false);
  ConstantIntRanges srange = computeBoundsBy(
      ssub, lhs.smin(), rhs.smax(), lhs.smax(), rhs.smin(), /*isSigned=*/true);
  return urange.intersection(srange);
}

```
- **EN**: Implements logic around `any`, `usub_sat`, `usub_ov`, `optional`, and 6 more symbols.
- **CN**: 围绕 `any`、`usub_sat`、`usub_ov`、`optional` 等另外 6 个符号 实现具体逻辑。

### Lines 245-270
```cpp
//===----------------------------------------------------------------------===//
// Multiplication
//===----------------------------------------------------------------------===//

ConstantIntRanges
mlir::intrange::inferMul(ArrayRef<ConstantIntRanges> argRanges,
                         OverflowFlags ovfFlags) {
  const ConstantIntRanges &lhs = argRanges[0], &rhs = argRanges[1];

  ConstArithStdFn umul = [=](const APInt &a,
                             const APInt &b) -> std::optional<APInt> {
    bool overflowed = false;
    APInt result = any(ovfFlags & OverflowFlags::Nuw)
                       ? a.umul_sat(b)
                       : a.umul_ov(b, overflowed);
    return overflowed ? std::optional<APInt>() : result;
  };
  ConstArithStdFn smul = [=](const APInt &a,
                             const APInt &b) -> std::optional<APInt> {
    bool overflowed = false;
    APInt result = any(ovfFlags & OverflowFlags::Nsw)
                       ? a.smul_sat(b)
                       : a.smul_ov(b, overflowed);
    return overflowed ? std::optional<APInt>() : result;
  };

```
- **EN**: Implements logic around `inferMul`, `any`, `umul_sat`, `umul_ov`, and 3 more symbols.
- **CN**: 围绕 `inferMul`、`any`、`umul_sat`、`umul_ov` 等另外 3 个符号 实现具体逻辑。

### Lines 271-284
```cpp
  ConstantIntRanges urange =
      minMaxBy(umul, {lhs.umin(), lhs.umax()}, {rhs.umin(), rhs.umax()},
               /*isSigned=*/false);
  ConstantIntRanges srange =
      minMaxBy(smul, {lhs.smin(), lhs.smax()}, {rhs.smin(), rhs.smax()},
               /*isSigned=*/true);
  return urange.intersection(srange);
}

//===----------------------------------------------------------------------===//
// DivU, CeilDivU (Unsigned division)
//===----------------------------------------------------------------------===//

/// Fix up division results (ex. for ceiling and floor), returning an APInt
```
- **EN**: Implements logic around `minMaxBy`, `intersection`.
- **CN**: 围绕 `minMaxBy`、`intersection` 实现具体逻辑。

### Lines 285-302
```cpp
/// if there has been no overflow
using DivisionFixupFn = function_ref<std::optional<APInt>(
    const APInt &lhs, const APInt &rhs, const APInt &result)>;

static ConstantIntRanges inferDivURange(const ConstantIntRanges &lhs,
                                        const ConstantIntRanges &rhs,
                                        DivisionFixupFn fixup) {
  const APInt &lhsMin = lhs.umin(), &lhsMax = lhs.umax(), &rhsMin = rhs.umin(),
              &rhsMax = rhs.umax();
  if (!rhsMin.isZero() && !rhsMax.isZero()) {
    auto udiv = [&fixup](const APInt &a,
                         const APInt &b) -> std::optional<APInt> {
      return fixup(a, b, a.udiv(b));
    };
    return minMaxBy(udiv, {lhsMin, lhsMax}, {rhsMin, rhsMax},
                    /*isSigned=*/false);
  }

```
- **EN**: Implements logic around `optional`, `inferDivURange`, `umin`, `umax`, and 3 more symbols.
- **CN**: 围绕 `optional`、`inferDivURange`、`umin`、`umax` 等另外 3 个符号 实现具体逻辑。

### Lines 303-318
```cpp
  APInt umin = APInt::getZero(rhsMin.getBitWidth());
  if (lhsMin.uge(rhsMax) && !rhsMax.isZero())
    umin = lhsMin.udiv(rhsMax);

  // X u/ Y u<= X.
  const APInt &umax = lhsMax;
  return ConstantIntRanges::fromUnsigned(umin, umax);
}

ConstantIntRanges
mlir::intrange::inferDivU(ArrayRef<ConstantIntRanges> argRanges) {
  return inferDivURange(argRanges[0], argRanges[1],
                        [](const APInt &lhs, const APInt &rhs,
                           const APInt &result) { return result; });
}

```
- **EN**: Implements logic around `getZero`, `uge`, `udiv`, `fromUnsigned`, and 2 more symbols.
- **CN**: 围绕 `getZero`、`uge`、`udiv`、`fromUnsigned` 等另外 2 个符号 实现具体逻辑。

### Lines 319-335
```cpp
ConstantIntRanges
mlir::intrange::inferCeilDivU(ArrayRef<ConstantIntRanges> argRanges) {
  const ConstantIntRanges &lhs = argRanges[0], &rhs = argRanges[1];

  auto ceilDivUIFix = [](const APInt &lhs, const APInt &rhs,
                         const APInt &result) -> std::optional<APInt> {
    if (!lhs.urem(rhs).isZero()) {
      bool overflowed = false;
      APInt corrected =
          result.uadd_ov(APInt(result.getBitWidth(), 1), overflowed);
      return overflowed ? std::optional<APInt>() : corrected;
    }
    return result;
  };
  return inferDivURange(lhs, rhs, ceilDivUIFix);
}

```
- **EN**: Implements logic around `inferCeilDivU`, `urem`, `uadd_ov`, `optional`, and 1 more symbols.
- **CN**: 围绕 `inferCeilDivU`、`urem`、`uadd_ov`、`optional` 等另外 1 个符号 实现具体逻辑。

### Lines 336-359
```cpp
//===----------------------------------------------------------------------===//
// DivS, CeilDivS, FloorDivS (Signed division)
//===----------------------------------------------------------------------===//

static ConstantIntRanges inferDivSRange(const ConstantIntRanges &lhs,
                                        const ConstantIntRanges &rhs,
                                        DivisionFixupFn fixup) {
  const APInt &lhsMin = lhs.smin(), &lhsMax = lhs.smax(), &rhsMin = rhs.smin(),
              &rhsMax = rhs.smax();
  bool canDivide = rhsMin.isStrictlyPositive() || rhsMax.isNegative();

  if (canDivide) {
    auto sdiv = [&fixup](const APInt &a,
                         const APInt &b) -> std::optional<APInt> {
      bool overflowed = false;
      APInt result = a.sdiv_ov(b, overflowed);
      return overflowed ? std::optional<APInt>() : fixup(a, b, result);
    };
    return minMaxBy(sdiv, {lhsMin, lhsMax}, {rhsMin, rhsMax},
                    /*isSigned=*/true);
  }
  return ConstantIntRanges::maxRange(rhsMin.getBitWidth());
}

```
- **EN**: Implements logic around `inferDivSRange`, `smin`, `smax`, `isStrictlyPositive`, and 4 more symbols.
- **CN**: 围绕 `inferDivSRange`、`smin`、`smax`、`isStrictlyPositive` 等另外 4 个符号 实现具体逻辑。

### Lines 360-387
```cpp
ConstantIntRanges
mlir::intrange::inferDivS(ArrayRef<ConstantIntRanges> argRanges) {
  return inferDivSRange(argRanges[0], argRanges[1],
                        [](const APInt &lhs, const APInt &rhs,
                           const APInt &result) { return result; });
}

ConstantIntRanges
mlir::intrange::inferCeilDivS(ArrayRef<ConstantIntRanges> argRanges) {
  const ConstantIntRanges &lhs = argRanges[0], &rhs = argRanges[1];

  auto ceilDivSIFix = [](const APInt &lhs, const APInt &rhs,
                         const APInt &result) -> std::optional<APInt> {
    if (!lhs.srem(rhs).isZero() && lhs.isNonNegative() == rhs.isNonNegative()) {
      bool overflowed = false;
      APInt corrected =
          result.sadd_ov(APInt(result.getBitWidth(), 1), overflowed);
      return overflowed ? std::optional<APInt>() : corrected;
    }
    // Special case where the usual implementation of ceilDiv causes
    // INT_MIN / [positive number] to be positive. This doesn't match the
    // definition of signed ceiling division mathematically, but it prevents
    // inconsistent constant-folding results. This arises because (-int_min) is
    // still negative, so -(-int_min / b) is -(int_min / b), which is
    // positive See #115293.
    if (lhs.isMinSignedValue() && rhs.sgt(1)) {
      return -result;
    }
```
- **EN**: Implements logic around `inferDivS`, `inferDivSRange`, `inferCeilDivS`, `srem`, and 3 more symbols.
- **CN**: 围绕 `inferDivS`、`inferDivSRange`、`inferCeilDivS`、`srem` 等另外 3 个符号 实现具体逻辑。

### Lines 388-404
```cpp
    return result;
  };
  ConstantIntRanges result = inferDivSRange(lhs, rhs, ceilDivSIFix);
  if (lhs.smin().isMinSignedValue() && lhs.smax().sgt(lhs.smin())) {
    // If lhs range includes INT_MIN and lhs is not a single value, we can
    // suddenly wrap to positive val, skipping entire negative range, add
    // [INT_MIN + 1, smax()] range to the result to handle this.
    auto newLhs = ConstantIntRanges::fromSigned(lhs.smin() + 1, lhs.smax());
    result = result.rangeUnion(inferDivSRange(newLhs, rhs, ceilDivSIFix));
  }
  return result;
}

ConstantIntRanges
mlir::intrange::inferFloorDivS(ArrayRef<ConstantIntRanges> argRanges) {
  const ConstantIntRanges &lhs = argRanges[0], &rhs = argRanges[1];

```
- **EN**: Implements logic around `inferDivSRange`, `smin`, `fromSigned`, `rangeUnion`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `inferDivSRange`、`smin`、`fromSigned`、`rangeUnion` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 405-418
```cpp
  auto floorDivSIFix = [](const APInt &lhs, const APInt &rhs,
                          const APInt &result) -> std::optional<APInt> {
    if (!lhs.srem(rhs).isZero() && lhs.isNonNegative() != rhs.isNonNegative()) {
      bool overflowed = false;
      APInt corrected =
          result.ssub_ov(APInt(result.getBitWidth(), 1), overflowed);
      return overflowed ? std::optional<APInt>() : corrected;
    }
    return result;
  };
  return inferDivSRange(lhs, rhs, floorDivSIFix);
}

//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `srem`, `ssub_ov`, `optional`, `inferDivSRange`.
- **CN**: 围绕 `srem`、`ssub_ov`、`optional`、`inferDivSRange` 实现具体逻辑。

### Lines 419-446
```cpp
// Signed remainder (RemS)
//===----------------------------------------------------------------------===//

ConstantIntRanges
mlir::intrange::inferRemS(ArrayRef<ConstantIntRanges> argRanges) {
  const ConstantIntRanges &lhs = argRanges[0], &rhs = argRanges[1];
  const APInt &lhsMin = lhs.smin(), &lhsMax = lhs.smax(), &rhsMin = rhs.smin(),
              &rhsMax = rhs.smax();

  unsigned width = rhsMax.getBitWidth();
  APInt smin = APInt::getSignedMinValue(width);
  APInt smax = APInt::getSignedMaxValue(width);
  // No bounds if zero could be a divisor.
  bool canBound = (rhsMin.isStrictlyPositive() || rhsMax.isNegative());
  if (canBound) {
    APInt maxDivisor = rhsMin.isStrictlyPositive() ? rhsMax : rhsMin.abs();
    bool canNegativeDividend = lhsMin.isNegative();
    bool canPositiveDividend = lhsMax.isStrictlyPositive();
    APInt zero = APInt::getZero(maxDivisor.getBitWidth());
    APInt maxPositiveResult = maxDivisor - 1;
    APInt minNegativeResult = -maxPositiveResult;
    smin = canNegativeDividend ? minNegativeResult : zero;
    smax = canPositiveDividend ? maxPositiveResult : zero;
    // Special case: sweeping out a contiguous range in N/[modulus].
    if (rhsMin == rhsMax) {
      if ((lhsMax - lhsMin).ult(maxDivisor)) {
        APInt minRem = lhsMin.srem(maxDivisor);
        APInt maxRem = lhsMax.srem(maxDivisor);
```
- **EN**: Implements logic around `inferRemS`, `smin`, `smax`, `getBitWidth`, and 7 more symbols.
- **CN**: 围绕 `inferRemS`、`smin`、`smax`、`getBitWidth` 等另外 7 个符号 实现具体逻辑。

### Lines 447-460
```cpp
        if (minRem.sle(maxRem)) {
          smin = minRem;
          smax = maxRem;
        }
      }
    }
  }
  return ConstantIntRanges::fromSigned(smin, smax);
}

//===----------------------------------------------------------------------===//
// Unsigned remainder (RemU)
//===----------------------------------------------------------------------===//

```
- **EN**: Implements logic around `sle`, `fromSigned`.
- **CN**: 围绕 `sle`、`fromSigned` 实现具体逻辑。

### Lines 461-487
```cpp
ConstantIntRanges
mlir::intrange::inferRemU(ArrayRef<ConstantIntRanges> argRanges) {
  const ConstantIntRanges &lhs = argRanges[0], &rhs = argRanges[1];
  const APInt &rhsMin = rhs.umin(), &rhsMax = rhs.umax();

  unsigned width = rhsMin.getBitWidth();
  APInt umin = APInt::getZero(width);
  // Remainder can't be larger than either of its arguments.
  APInt umax = llvm::APIntOps::umin((rhsMax - 1), lhs.umax());

  if (!rhsMin.isZero()) {
    // Special case: sweeping out a contiguous range in N/[modulus]
    if (rhsMin == rhsMax) {
      const APInt &lhsMin = lhs.umin(), &lhsMax = lhs.umax();
      if ((lhsMax - lhsMin).ult(rhsMax)) {
        APInt minRem = lhsMin.urem(rhsMax);
        APInt maxRem = lhsMax.urem(rhsMax);
        if (minRem.ule(maxRem)) {
          umin = minRem;
          umax = maxRem;
        }
      }
    }
  }
  return ConstantIntRanges::fromUnsigned(umin, umax);
}

```
- **EN**: Implements logic around `inferRemU`, `umin`, `getBitWidth`, `getZero`, and 5 more symbols.
- **CN**: 围绕 `inferRemU`、`umin`、`getBitWidth`、`getZero` 等另外 5 个符号 实现具体逻辑。

### Lines 488-504
```cpp
//===----------------------------------------------------------------------===//
// Max and min (MaxS, MaxU, MinS, MinU)
//===----------------------------------------------------------------------===//

ConstantIntRanges
mlir::intrange::inferMaxS(ArrayRef<ConstantIntRanges> argRanges) {
  const ConstantIntRanges &lhs = argRanges[0], &rhs = argRanges[1];

  const APInt &smin = lhs.smin().sgt(rhs.smin()) ? lhs.smin() : rhs.smin();
  const APInt &smax = lhs.smax().sgt(rhs.smax()) ? lhs.smax() : rhs.smax();
  return ConstantIntRanges::fromSigned(smin, smax);
}

ConstantIntRanges
mlir::intrange::inferMaxU(ArrayRef<ConstantIntRanges> argRanges) {
  const ConstantIntRanges &lhs = argRanges[0], &rhs = argRanges[1];

```
- **EN**: Implements logic around `inferMaxS`, `smin`, `smax`, `fromSigned`, and 1 more symbols.
- **CN**: 围绕 `inferMaxS`、`smin`、`smax`、`fromSigned` 等另外 1 个符号 实现具体逻辑。

### Lines 505-518
```cpp
  const APInt &umin = lhs.umin().ugt(rhs.umin()) ? lhs.umin() : rhs.umin();
  const APInt &umax = lhs.umax().ugt(rhs.umax()) ? lhs.umax() : rhs.umax();
  return ConstantIntRanges::fromUnsigned(umin, umax);
}

ConstantIntRanges
mlir::intrange::inferMinS(ArrayRef<ConstantIntRanges> argRanges) {
  const ConstantIntRanges &lhs = argRanges[0], &rhs = argRanges[1];

  const APInt &smin = lhs.smin().slt(rhs.smin()) ? lhs.smin() : rhs.smin();
  const APInt &smax = lhs.smax().slt(rhs.smax()) ? lhs.smax() : rhs.smax();
  return ConstantIntRanges::fromSigned(smin, smax);
}

```
- **EN**: Implements logic around `umin`, `umax`, `fromUnsigned`, `inferMinS`, and 3 more symbols.
- **CN**: 围绕 `umin`、`umax`、`fromUnsigned`、`inferMinS` 等另外 3 个符号 实现具体逻辑。

### Lines 519-532
```cpp
ConstantIntRanges
mlir::intrange::inferMinU(ArrayRef<ConstantIntRanges> argRanges) {
  const ConstantIntRanges &lhs = argRanges[0], &rhs = argRanges[1];

  const APInt &umin = lhs.umin().ult(rhs.umin()) ? lhs.umin() : rhs.umin();
  const APInt &umax = lhs.umax().ult(rhs.umax()) ? lhs.umax() : rhs.umax();
  return ConstantIntRanges::fromUnsigned(umin, umax);
}

//===----------------------------------------------------------------------===//
// Bitwise operators (And, Or, Xor)
//===----------------------------------------------------------------------===//

/// "Widen" bounds - if 0bvvvvv??? <= a <= 0bvvvvv???,
```
- **EN**: Implements logic around `inferMinU`, `umin`, `umax`, `fromUnsigned`.
- **CN**: 围绕 `inferMinU`、`umin`、`umax`、`fromUnsigned` 实现具体逻辑。

### Lines 533-556
```cpp
/// relax the bounds to 0bvvvvv000 <= a <= 0bvvvvv111, where vvvvv are the bits
/// that both bonuds have in common. This gives us a consertive approximation
/// for what values can be passed to bitwise operations.
static std::tuple<APInt, APInt>
widenBitwiseBounds(const ConstantIntRanges &bound) {
  APInt leftVal = bound.umin(), rightVal = bound.umax();
  unsigned bitwidth = leftVal.getBitWidth();
  unsigned differingBits = bitwidth - (leftVal ^ rightVal).countl_zero();
  leftVal.clearLowBits(differingBits);
  rightVal.setLowBits(differingBits);
  return std::make_tuple(std::move(leftVal), std::move(rightVal));
}

ConstantIntRanges
mlir::intrange::inferAnd(ArrayRef<ConstantIntRanges> argRanges) {
  auto [lhsZeros, lhsOnes] = widenBitwiseBounds(argRanges[0]);
  auto [rhsZeros, rhsOnes] = widenBitwiseBounds(argRanges[1]);
  auto andi = [](const APInt &a, const APInt &b) -> std::optional<APInt> {
    return a & b;
  };
  return minMaxBy(andi, {lhsZeros, lhsOnes}, {rhsZeros, rhsOnes},
                  /*isSigned=*/false);
}

```
- **EN**: Implements logic around `widenBitwiseBounds`, `umin`, `getBitWidth`, `countl_zero`, and 5 more symbols.
- **CN**: 围绕 `widenBitwiseBounds`、`umin`、`getBitWidth`、`countl_zero` 等另外 5 个符号 实现具体逻辑。

### Lines 557-576
```cpp
ConstantIntRanges
mlir::intrange::inferOr(ArrayRef<ConstantIntRanges> argRanges) {
  auto [lhsZeros, lhsOnes] = widenBitwiseBounds(argRanges[0]);
  auto [rhsZeros, rhsOnes] = widenBitwiseBounds(argRanges[1]);
  auto ori = [](const APInt &a, const APInt &b) -> std::optional<APInt> {
    return a | b;
  };
  return minMaxBy(ori, {lhsZeros, lhsOnes}, {rhsZeros, rhsOnes},
                  /*isSigned=*/false);
}

/// Get bitmask of all bits which can change while iterating in
/// [bound.umin(), bound.umax()].
static APInt getVaryingBitsMask(const ConstantIntRanges &bound) {
  APInt leftVal = bound.umin(), rightVal = bound.umax();
  unsigned bitwidth = leftVal.getBitWidth();
  unsigned differingBits = bitwidth - (leftVal ^ rightVal).countl_zero();
  return APInt::getLowBitsSet(bitwidth, differingBits);
}

```
- **EN**: Implements logic around `inferOr`, `widenBitwiseBounds`, `minMaxBy`, `getVaryingBitsMask`, and 4 more symbols.
- **CN**: 围绕 `inferOr`、`widenBitwiseBounds`、`minMaxBy`、`getVaryingBitsMask` 等另外 4 个符号 实现具体逻辑。

### Lines 577-591
```cpp
ConstantIntRanges
mlir::intrange::inferXor(ArrayRef<ConstantIntRanges> argRanges) {
  // Construct mask of varying bits for both ranges, xor values and then replace
  // masked bits with 0s and 1s to get min and max values respectively.
  ConstantIntRanges lhs = argRanges[0], rhs = argRanges[1];
  APInt mask = getVaryingBitsMask(lhs) | getVaryingBitsMask(rhs);
  APInt res = lhs.umin() ^ rhs.umin();
  APInt min = res & ~mask;
  APInt max = res | mask;
  return ConstantIntRanges::fromUnsigned(min, max);
}

//===----------------------------------------------------------------------===//
// Shifts (Shl, ShrS, ShrU)
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `inferXor`, `getVaryingBitsMask`, `umin`, `fromUnsigned`.
- **CN**: 围绕 `inferXor`、`getVaryingBitsMask`、`umin`、`fromUnsigned` 实现具体逻辑。

### Lines 592-617
```cpp

ConstantIntRanges
mlir::intrange::inferShl(ArrayRef<ConstantIntRanges> argRanges,
                         OverflowFlags ovfFlags) {
  const ConstantIntRanges &lhs = argRanges[0], &rhs = argRanges[1];
  const APInt &rhsUMin = rhs.umin(), &rhsUMax = rhs.umax();

  // The signed/unsigned overflow behavior of shl by `rhs` matches a mul with
  // 2^rhs.
  ConstArithStdFn ushl = [=](const APInt &l,
                             const APInt &r) -> std::optional<APInt> {
    bool overflowed = false;
    APInt result = any(ovfFlags & OverflowFlags::Nuw)
                       ? l.ushl_sat(r)
                       : l.ushl_ov(r, overflowed);
    return overflowed ? std::optional<APInt>() : result;
  };
  ConstArithStdFn sshl = [=](const APInt &l,
                             const APInt &r) -> std::optional<APInt> {
    bool overflowed = false;
    APInt result = any(ovfFlags & OverflowFlags::Nsw)
                       ? l.sshl_sat(r)
                       : l.sshl_ov(r, overflowed);
    return overflowed ? std::optional<APInt>() : result;
  };

```
- **EN**: Implements logic around `inferShl`, `umin`, `any`, `ushl_sat`, and 4 more symbols.
- **CN**: 围绕 `inferShl`、`umin`、`any`、`ushl_sat` 等另外 4 个符号 实现具体逻辑。

### Lines 618-634
```cpp
  ConstantIntRanges urange =
      minMaxBy(ushl, {lhs.umin(), lhs.umax()}, {rhsUMin, rhsUMax},
               /*isSigned=*/false);
  ConstantIntRanges srange =
      minMaxBy(sshl, {lhs.smin(), lhs.smax()}, {rhsUMin, rhsUMax},
               /*isSigned=*/true);
  return urange.intersection(srange);
}

ConstantIntRanges
mlir::intrange::inferShrS(ArrayRef<ConstantIntRanges> argRanges) {
  const ConstantIntRanges &lhs = argRanges[0], &rhs = argRanges[1];

  auto ashr = [](const APInt &l, const APInt &r) -> std::optional<APInt> {
    return r.uge(r.getBitWidth()) ? std::optional<APInt>() : l.ashr(r);
  };

```
- **EN**: Implements logic around `minMaxBy`, `intersection`, `inferShrS`, `uge`.
- **CN**: 围绕 `minMaxBy`、`intersection`、`inferShrS`、`uge` 实现具体逻辑。

### Lines 635-649
```cpp
  return minMaxBy(ashr, {lhs.smin(), lhs.smax()}, {rhs.umin(), rhs.umax()},
                  /*isSigned=*/true);
}

ConstantIntRanges
mlir::intrange::inferShrU(ArrayRef<ConstantIntRanges> argRanges) {
  const ConstantIntRanges &lhs = argRanges[0], &rhs = argRanges[1];

  auto lshr = [](const APInt &l, const APInt &r) -> std::optional<APInt> {
    return r.uge(r.getBitWidth()) ? std::optional<APInt>() : l.lshr(r);
  };
  return minMaxBy(lshr, {lhs.umin(), lhs.umax()}, {rhs.umin(), rhs.umax()},
                  /*isSigned=*/false);
}

```
- **EN**: Implements logic around `minMaxBy`, `inferShrU`, `uge`.
- **CN**: 围绕 `minMaxBy`、`inferShrU`、`uge` 实现具体逻辑。

### Lines 650-677
```cpp
//===----------------------------------------------------------------------===//
// Comparisons (Cmp)
//===----------------------------------------------------------------------===//

static intrange::CmpPredicate invertPredicate(intrange::CmpPredicate pred) {
  switch (pred) {
  case intrange::CmpPredicate::eq:
    return intrange::CmpPredicate::ne;
  case intrange::CmpPredicate::ne:
    return intrange::CmpPredicate::eq;
  case intrange::CmpPredicate::slt:
    return intrange::CmpPredicate::sge;
  case intrange::CmpPredicate::sle:
    return intrange::CmpPredicate::sgt;
  case intrange::CmpPredicate::sgt:
    return intrange::CmpPredicate::sle;
  case intrange::CmpPredicate::sge:
    return intrange::CmpPredicate::slt;
  case intrange::CmpPredicate::ult:
    return intrange::CmpPredicate::uge;
  case intrange::CmpPredicate::ule:
    return intrange::CmpPredicate::ugt;
  case intrange::CmpPredicate::ugt:
    return intrange::CmpPredicate::ule;
  case intrange::CmpPredicate::uge:
    return intrange::CmpPredicate::ult;
  }
  llvm_unreachable("unknown cmp predicate value");
```
- **EN**: Implements logic around `invertPredicate`, `llvm_unreachable`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `invertPredicate`、`llvm_unreachable` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 678-705
```cpp
}

static bool isStaticallyTrue(intrange::CmpPredicate pred,
                             const ConstantIntRanges &lhs,
                             const ConstantIntRanges &rhs) {
  switch (pred) {
  case intrange::CmpPredicate::sle:
    return lhs.smax().sle(rhs.smin());
  case intrange::CmpPredicate::slt:
    return lhs.smax().slt(rhs.smin());
  case intrange::CmpPredicate::ule:
    return lhs.umax().ule(rhs.umin());
  case intrange::CmpPredicate::ult:
    return lhs.umax().ult(rhs.umin());
  case intrange::CmpPredicate::sge:
    return lhs.smin().sge(rhs.smax());
  case intrange::CmpPredicate::sgt:
    return lhs.smin().sgt(rhs.smax());
  case intrange::CmpPredicate::uge:
    return lhs.umin().uge(rhs.umax());
  case intrange::CmpPredicate::ugt:
    return lhs.umin().ugt(rhs.umax());
  case intrange::CmpPredicate::eq: {
    std::optional<APInt> lhsConst = lhs.getConstantValue();
    std::optional<APInt> rhsConst = rhs.getConstantValue();
    return lhsConst && rhsConst && lhsConst == rhsConst;
  }
  case intrange::CmpPredicate::ne: {
```
- **EN**: Implements logic around `isStaticallyTrue`, `smax`, `umax`, `smin`, and 2 more symbols.
- **CN**: 围绕 `isStaticallyTrue`、`smax`、`umax`、`smin` 等另外 2 个符号 实现具体逻辑。

### Lines 706-719
```cpp
    // While equality requires that there is an interpration of the preceeding
    // computations that produces equal constants, whether that be signed or
    // unsigned, statically determining inequality requires that neither
    // interpretation produce potentially overlapping ranges.
    bool sne = isStaticallyTrue(intrange::CmpPredicate::slt, lhs, rhs) ||
               isStaticallyTrue(intrange::CmpPredicate::sgt, lhs, rhs);
    bool une = isStaticallyTrue(intrange::CmpPredicate::ult, lhs, rhs) ||
               isStaticallyTrue(intrange::CmpPredicate::ugt, lhs, rhs);
    return sne && une;
  }
  }
  return false;
}

```
- **EN**: Implements logic around `isStaticallyTrue`.
- **CN**: 围绕 `isStaticallyTrue` 实现具体逻辑。

### Lines 720-733
```cpp
std::optional<bool> mlir::intrange::evaluatePred(CmpPredicate pred,
                                                 const ConstantIntRanges &lhs,
                                                 const ConstantIntRanges &rhs) {
  if (isStaticallyTrue(pred, lhs, rhs))
    return true;
  if (isStaticallyTrue(invertPredicate(pred), lhs, rhs))
    return false;
  return std::nullopt;
}

//===----------------------------------------------------------------------===//
// Shaped type dimension accessors / ShapedDimOpInterface
//===----------------------------------------------------------------------===//

```
- **EN**: Implements logic around `evaluatePred`, `isStaticallyTrue`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `evaluatePred`、`isStaticallyTrue` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 734-754
```cpp
ConstantIntRanges
mlir::intrange::inferShapedDimOpInterface(ShapedDimOpInterface op,
                                          const IntegerValueRange &maybeDim) {
  unsigned width =
      ConstantIntRanges::getStorageBitwidth(op->getResult(0).getType());
  APInt zero = APInt::getZero(width);
  APInt typeMax = APInt::getSignedMaxValue(width);

  auto shapedTy = cast<ShapedType>(op.getShapedValue().getType());
  if (!shapedTy.hasRank())
    return ConstantIntRanges::fromSigned(zero, typeMax);

  int64_t rank = shapedTy.getRank();
  int64_t minDim = 0;
  int64_t maxDim = rank - 1;
  if (!maybeDim.isUninitialized()) {
    const ConstantIntRanges &dim = maybeDim.getValue();
    minDim = std::max(minDim, dim.smin().getSExtValue());
    maxDim = std::min(maxDim, dim.smax().getSExtValue());
  }

```
- **EN**: Implements logic around `inferShapedDimOpInterface`, `getStorageBitwidth`, `getZero`, `getSignedMaxValue`, and 8 more symbols.
- **CN**: 围绕 `inferShapedDimOpInterface`、`getStorageBitwidth`、`getZero`、`getSignedMaxValue` 等另外 8 个符号 实现具体逻辑。

### Lines 755-772
```cpp
  std::optional<ConstantIntRanges> result;
  auto joinResult = [&](const ConstantIntRanges &thisResult) {
    if (!result.has_value())
      result = thisResult;
    else
      result = result->rangeUnion(thisResult);
  };
  for (int64_t i = minDim; i <= maxDim; ++i) {
    int64_t length = shapedTy.getDimSize(i);

    if (ShapedType::isDynamic(length))
      joinResult(ConstantIntRanges::fromSigned(zero, typeMax));
    else
      joinResult(ConstantIntRanges::constant(APInt(width, length)));
  }
  return result.value_or(ConstantIntRanges::fromSigned(zero, typeMax));
}

```
- **EN**: Implements logic around `has_value`, `rangeUnion`, `getDimSize`, `isDynamic`, and 2 more symbols.
- **CN**: 围绕 `has_value`、`rangeUnion`、`getDimSize`、`isDynamic` 等另外 2 个符号 实现具体逻辑。

### Lines 773-800
```cpp
//===----------------------------------------------------------------------===//
// Affine expression inference
//===----------------------------------------------------------------------===//

static ConstantIntRanges clampToPositive(const ConstantIntRanges &val) {
  unsigned width = val.smin().getBitWidth();
  APInt one(width, 1);
  APInt clampedUMin = val.umin().ult(one) ? one : val.umin();
  APInt clampedSMin = val.smin().slt(one) ? one : val.smin();
  return ConstantIntRanges::fromUnsigned(clampedUMin, val.umax())
      .intersection(ConstantIntRanges::fromSigned(clampedSMin, val.smax()));
}

ConstantIntRanges
mlir::intrange::inferAffineExpr(AffineExpr expr,
                                ArrayRef<ConstantIntRanges> dimRanges,
                                ArrayRef<ConstantIntRanges> symbolRanges) {
  switch (expr.getKind()) {
  case AffineExprKind::Constant: {
    auto constExpr = cast<AffineConstantExpr>(expr);
    APInt value(indexMaxWidth, constExpr.getValue(), /*isSigned=*/true);
    return ConstantIntRanges::constant(value);
  }
  case AffineExprKind::DimId: {
    auto dimExpr = cast<AffineDimExpr>(expr);
    unsigned pos = dimExpr.getPosition();
    assert(pos < dimRanges.size() && "Dimension index out of bounds");
    return dimRanges[pos];
```
- **EN**: Implements logic around `clampToPositive`, `smin`, `one`, `umin`, and 9 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `clampToPositive`、`smin`、`one`、`umin` 等另外 9 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 801-828
```cpp
  }
  case AffineExprKind::SymbolId: {
    auto symbolExpr = cast<AffineSymbolExpr>(expr);
    unsigned pos = symbolExpr.getPosition();
    assert(pos < symbolRanges.size() && "Symbol index out of bounds");
    return symbolRanges[pos];
  }
  case AffineExprKind::Add: {
    auto binExpr = cast<AffineBinaryOpExpr>(expr);
    ConstantIntRanges lhs =
        inferAffineExpr(binExpr.getLHS(), dimRanges, symbolRanges);
    ConstantIntRanges rhs =
        inferAffineExpr(binExpr.getRHS(), dimRanges, symbolRanges);
    return inferAdd({lhs, rhs}, OverflowFlags::Nsw);
  }
  case AffineExprKind::Mul: {
    auto binExpr = cast<AffineBinaryOpExpr>(expr);
    ConstantIntRanges lhs =
        inferAffineExpr(binExpr.getLHS(), dimRanges, symbolRanges);
    ConstantIntRanges rhs =
        inferAffineExpr(binExpr.getRHS(), dimRanges, symbolRanges);
    return inferMul({lhs, rhs}, OverflowFlags::Nsw);
  }
  case AffineExprKind::Mod: {
    auto binExpr = cast<AffineBinaryOpExpr>(expr);
    ConstantIntRanges lhs =
        inferAffineExpr(binExpr.getLHS(), dimRanges, symbolRanges);
    ConstantIntRanges rhs =
```
- **EN**: Implements logic around `cast`, `getPosition`, `assert`, `inferAffineExpr`, and 2 more symbols.
- **CN**: 围绕 `cast`、`getPosition`、`assert`、`inferAffineExpr` 等另外 2 个符号 实现具体逻辑。

### Lines 829-843
```cpp
        inferAffineExpr(binExpr.getRHS(), dimRanges, symbolRanges);
    // Affine mod is Euclidean modulo: result is always in [0, rhs-1].
    // This assumes RHS is positive (enforced by affine expr semantics).
    const APInt &lhsMin = lhs.smin();
    const APInt &lhsMax = lhs.smax();
    const APInt &rhsMin = rhs.smin();
    const APInt &rhsMax = rhs.smax();
    unsigned width = rhsMin.getBitWidth();

    // Guard against division by zero.
    if (rhsMax.isZero())
      return ConstantIntRanges::maxRange(width);

    APInt zero = APInt::getZero(width);

```
- **EN**: Implements logic around `inferAffineExpr`, `smin`, `smax`, `getBitWidth`, and 3 more symbols.
- **CN**: 围绕 `inferAffineExpr`、`smin`、`smax`、`getBitWidth` 等另外 3 个符号 实现具体逻辑。

### Lines 844-871
```cpp
    // For Euclidean mod, result is in [0, max(rhs)-1].
    APInt umin = zero;
    APInt umax = rhsMax - 1;

    // Special case: if dividend is already in [0, min(rhs)), result equals
    // dividend. We use rhsMin to ensure this is safe for all possible divisor
    // values.
    if (rhsMin.isStrictlyPositive() && lhsMin.isNonNegative() &&
        lhsMax.ult(rhsMin)) {
      umin = lhsMin;
      umax = lhsMax;
    }
    // Special case: sweeping out a contiguous range with constant divisor.
    // Only applies when dividend is non-negative and the range does not
    // cross a modulus boundary (same quotient), ensuring contiguity.
    else if (rhsMin == rhsMax && lhsMin.isNonNegative() &&
             (lhsMax - lhsMin).ult(rhsMax) &&
             lhsMin.udiv(rhsMax) == lhsMax.udiv(rhsMax)) {
      // For non-negative dividends within the same modular period,
      // Euclidean mod is same as unsigned remainder and the result is
      // contiguous.
      umin = lhsMin.urem(rhsMax);
      umax = lhsMax.urem(rhsMax);
      // Result should be contiguous since we're not wrapping around.
      assert(umin.ule(umax) &&
             "Range should be contiguous for non-negative dividend");
    }

```
- **EN**: Implements logic around `isStrictlyPositive`, `ult`, `isNonNegative`, `udiv`, and 2 more symbols.
- **CN**: 围绕 `isStrictlyPositive`、`ult`、`isNonNegative`、`udiv` 等另外 2 个符号 实现具体逻辑。

### Lines 872-898
```cpp
    return ConstantIntRanges::fromUnsigned(umin, umax);
  }
  case AffineExprKind::FloorDiv: {
    auto binExpr = cast<AffineBinaryOpExpr>(expr);
    ConstantIntRanges lhs =
        inferAffineExpr(binExpr.getLHS(), dimRanges, symbolRanges);
    ConstantIntRanges rhs =
        inferAffineExpr(binExpr.getRHS(), dimRanges, symbolRanges);
    // Affine floordiv requires strictly positive divisor (> 0).
    // Clamp divisor lower bound to 1 for tighter range inference.
    ConstantIntRanges clampedRhs = clampToPositive(rhs);
    return inferFloorDivS({lhs, clampedRhs});
  }
  case AffineExprKind::CeilDiv: {
    auto binExpr = cast<AffineBinaryOpExpr>(expr);
    ConstantIntRanges lhs =
        inferAffineExpr(binExpr.getLHS(), dimRanges, symbolRanges);
    ConstantIntRanges rhs =
        inferAffineExpr(binExpr.getRHS(), dimRanges, symbolRanges);
    // Affine ceildiv requires strictly positive divisor (> 0).
    // Clamp divisor lower bound to 1 for tighter range inference.
    ConstantIntRanges clampedRhs = clampToPositive(rhs);
    return inferCeilDivS({lhs, clampedRhs});
  }
  }
  llvm_unreachable("unknown affine expression kind");
}
```
- **EN**: Implements logic around `fromUnsigned`, `cast`, `inferAffineExpr`, `clampToPositive`, and 3 more symbols.
- **CN**: 围绕 `fromUnsigned`、`cast`、`inferAffineExpr`、`clampToPositive` 等另外 3 个符号 实现具体逻辑。

## Key Concepts / 关键概念

- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Uses MLIR interfaces to describe cross-cutting behavior independently from concrete operation classes.
  - **CN**: 使用 MLIR 接口在不依赖具体操作类的前提下描述横切行为。
- **SSA and region-based IR / SSA 与基于 Region 的 IR**:
  - **EN**: The file works with MLIR operations, blocks, values, or regions that form MLIR's hierarchical SSA IR.
  - **CN**: 该文件处理 MLIR 操作、块、值或 Region，它们共同构成 MLIR 的层次化 SSA IR。
- **LLVM ADT containers / LLVM ADT 容器**:
  - **EN**: The implementation uses LLVM containers for performance-conscious storage and traversal.
  - **CN**: 该实现使用 LLVM 容器来进行兼顾性能的存储与遍历。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `mlir/Interfaces/Utils/InferIntRangeCommon.h`, `mlir/IR/AffineExpr.h`, `mlir/Interfaces/InferIntRangeInterface.h`, `mlir/Interfaces/ShapedOpInterfaces.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/STLExtras.h`, `llvm/Support/Debug.h`
- **Standard-library headers / 标准库头文件**: `<iterator>`, `<optional>`
- **Subsystem categories / 子系统类别**: MLIR interface declarations / MLIR 接口声明 (3), LLVM ADT containers and utilities / LLVM ADT 容器与工具 (2), core MLIR IR abstractions / 核心 MLIR IR 抽象 (1), LLVM support-library helpers / LLVM Support 库辅助工具 (1)
