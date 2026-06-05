# DivisionByConstantInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/DivisionByConstantInfo.cpp`
- Repository: `llvm-project`
- Purpose (EN): / This file implements support for optimizing divisions by a constant
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `DivisionByConstantInfo` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===----- DivisionByConstantInfo.cpp - division by constant -*- C++ -*----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///
/// This file implements support for optimizing divisions by a constant
///
//===----------------------------------------------------------------------===//

#include "llvm/Support/DivisionByConstantInfo.h"

using namespace llvm;

/// Calculate the magic numbers required to implement a signed integer division
/// by a constant as a sequence of multiplies, adds and shifts.  Requires that
/// the divisor not be 0, 1, or -1.  Taken from "Hacker's Delight", Henry S.
/// Warren, Jr., Chapter 10.
SignedDivisionByConstantInfo SignedDivisionByConstantInfo::get(const APInt &D) {
  assert(!D.isZero() && "Precondition violation.");

  // We'd be endlessly stuck in the loop.
  assert(D.getBitWidth() >= 3 && "Does not work at smaller bitwidths.");

  APInt Delta;
  APInt SignedMin = APInt::getSignedMinValue(D.getBitWidth());
  struct SignedDivisionByConstantInfo Retval;

  APInt AD = D.abs();
  APInt T = SignedMin + (D.lshr(D.getBitWidth() - 1));
  APInt ANC = T - 1 - T.urem(AD);   // absolute value of NC
  unsigned P = D.getBitWidth() - 1; // initialize P
  APInt Q1, R1, Q2, R2;
  // initialize Q1 = 2P/abs(NC); R1 = rem(2P,abs(NC))
  APInt::udivrem(SignedMin, ANC, Q1, R1);
  // initialize Q2 = 2P/abs(D); R2 = rem(2P,abs(D))
  APInt::udivrem(SignedMin, AD, Q2, R2);
  do {
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 1 direct dependencies, including `llvm/Support/DivisionByConstantInfo.h`.
  CN: 引入了 1 个直接依赖，其中包括 `llvm/Support/DivisionByConstantInfo.h`。
- EN: This section centers on `get`, `assert`, `udivrem` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `get`, `assert`, `udivrem` 等符号展开，负责查询现有状态并返回计算出的元数据。

### Lines 41-80

```cpp
    P = P + 1;
    Q1 <<= 1;      // update Q1 = 2P/abs(NC)
    R1 <<= 1;      // update R1 = rem(2P/abs(NC))
    if (R1.uge(ANC)) { // must be unsigned comparison
      ++Q1;
      R1 -= ANC;
    }
    Q2 <<= 1;     // update Q2 = 2P/abs(D)
    R2 <<= 1;     // update R2 = rem(2P/abs(D))
    if (R2.uge(AD)) { // must be unsigned comparison
      ++Q2;
      R2 -= AD;
    }
    // Delta = AD - R2
    Delta = AD;
    Delta -= R2;
  } while (Q1.ult(Delta) || (Q1 == Delta && R1.isZero()));

  Retval.Magic = std::move(Q2);
  ++Retval.Magic;
  if (D.isNegative())
    Retval.Magic.negate();                  // resulting magic number
  Retval.ShiftAmount = P - D.getBitWidth(); // resulting shift
  return Retval;
}

/// Calculate the magic numbers required to implement an unsigned integer
/// division by a constant as a sequence of multiplies, adds and shifts.
/// Requires that the divisor not be 0.  Taken from "Hacker's Delight", Henry
/// S. Warren, Jr., chapter 10.
/// LeadingZeros can be used to simplify the calculation if the upper bits
/// of the divided value are known zero.
UnsignedDivisionByConstantInfo
UnsignedDivisionByConstantInfo::get(const APInt &D, unsigned LeadingZeros,
                                    bool AllowEvenDivisorOptimization,
                                    bool AllowWidenOptimization) {
  assert(!D.isZero() && !D.isOne() && "Precondition violation.");
  assert(D.getBitWidth() > 1 && "Does not work at smaller bitwidths.");

  APInt Delta;
```
- EN: This section centers on `get`, `assert` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `get`, `assert` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 81-120

```cpp
  struct UnsignedDivisionByConstantInfo Retval;
  Retval.IsAdd = false; // initialize "add" indicator
  Retval.Widen = false; // initialize widen indicator
  APInt AllOnes =
      APInt::getLowBitsSet(D.getBitWidth(), D.getBitWidth() - LeadingZeros);
  APInt SignedMin = APInt::getSignedMinValue(D.getBitWidth());
  APInt SignedMax = APInt::getSignedMaxValue(D.getBitWidth());

  // Calculate NC, the largest dividend such that NC.urem(D) == D-1.
  APInt NC = AllOnes - (AllOnes + 1 - D).urem(D);
  assert(NC.urem(D) == D - 1 && "Unexpected NC value");
  unsigned P = D.getBitWidth() - 1; // initialize P
  APInt Q1, R1, Q2, R2;
  // initialize Q1 = 2P/NC; R1 = rem(2P,NC)
  APInt::udivrem(SignedMin, NC, Q1, R1);
  // initialize Q2 = (2P-1)/D; R2 = rem((2P-1),D)
  APInt::udivrem(SignedMax, D, Q2, R2);
  do {
    P = P + 1;
    if (R1.uge(NC - R1)) {
      // update Q1
      Q1 <<= 1;
      ++Q1;
      // update R1
      R1 <<= 1;
      R1 -= NC;
    } else {
      Q1 <<= 1; // update Q1
      R1 <<= 1; // update R1
    }
    if ((R2 + 1).uge(D - R2)) {
      if (Q2.uge(SignedMax))
        Retval.IsAdd = true;
      // update Q2
      Q2 <<= 1;
      ++Q2;
      // update R2
      R2 <<= 1;
      ++R2;
      R2 -= D;
```
- EN: This section centers on `getLowBitsSet`, `assert`, `udivrem` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `getLowBitsSet`, `assert`, `udivrem` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 121-160

```cpp
    } else {
      if (Q2.uge(SignedMin))
        Retval.IsAdd = true;
      // update Q2
      Q2 <<= 1;
      // update R2
      R2 <<= 1;
      ++R2;
    }
    // Delta = D - 1 - R2
    Delta = D;
    --Delta;
    Delta -= R2;
  } while (P < D.getBitWidth() * 2 &&
           (Q1.ult(Delta) || (Q1 == Delta && R1.isZero())));

  if (Retval.IsAdd && !D[0] && AllowEvenDivisorOptimization) {
    unsigned PreShift = D.countr_zero();
    APInt ShiftedD = D.lshr(PreShift);
    Retval =
        UnsignedDivisionByConstantInfo::get(ShiftedD, LeadingZeros + PreShift);
    assert(!Retval.IsAdd && Retval.PreShift == 0);
    Retval.PreShift = PreShift;
    return Retval;
  }

  Retval.Magic = std::move(Q2);             // resulting magic number
  ++Retval.Magic;
  Retval.PostShift = P - D.getBitWidth(); // resulting shift
  // Reduce shift amount for IsAdd.
  if (Retval.IsAdd) {
    assert(Retval.PostShift > 0 && "Unexpected shift");
    Retval.PostShift -= 1;
  }
  Retval.PreShift = 0;

  // For IsAdd case with AllowWidenOptimization, compute widened magic.
  // This is for optimizing 32-bit division using 64-bit multiplication.
  // The actual magic constant is 2^W + Magic ((W+1)-bit).
  // We pre-shift it left by (W*2 - OriginalShift) to avoid runtime shift.
```
- EN: This section centers on `get`, `assert` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `get`, `assert` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 161-173

```cpp
  if (Retval.IsAdd && AllowWidenOptimization) {
    unsigned W = D.getBitWidth();
    unsigned OriginalShift = Retval.PostShift + W + 1;
    // Since PostShift >= 1, shift amount is at most W-2, so W*2 bits suffice.
    Retval.Magic = (APInt::getOneBitSet(W * 2, W) + Retval.Magic.zext(W * 2))
                       .shl(W * 2 - OriginalShift);
    Retval.IsAdd = false;
    Retval.PostShift = 0;
    Retval.Widen = true;
  }

  return Retval;
}
```
- EN: In this range, the code checks conditions and handles edge cases and returns the resulting value to its callers.
  CN: 在这一段中，代码检查条件并处理边界情况，并将结果返回给调用方。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `SignedDivisionByConstantInfo`, `UnsignedDivisionByConstantInfo`, `get`, `assert`, `udivrem`, `getLowBitsSet` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Helper routines coordinate local state, control flow, and result construction. / 通过辅助例程协调局部状态、控制流与结果构造。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/Support/DivisionByConstantInfo.h`
- Standard library / 标准库: None / 无
- Other/system headers / 其他或系统头文件: None / 无
- Related symbols / 相关符号: `SignedDivisionByConstantInfo`, `UnsignedDivisionByConstantInfo`, `get`, `assert`, `udivrem`, `getLowBitsSet`
