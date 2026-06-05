# BranchProbability.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/BranchProbability.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements Branch Probability class.
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `BranchProbability` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===-------------- lib/Support/BranchProbability.cpp -----------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements Branch Probability class.
//
//===----------------------------------------------------------------------===//

#include "llvm/Support/BranchProbability.h"
#include "llvm/Config/llvm-config.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/Format.h"
#include "llvm/Support/raw_ostream.h"
#include <cassert>
#include <cmath>

using namespace llvm;

raw_ostream &BranchProbability::print(raw_ostream &OS) const {
  if (isUnknown())
    return OS << "?%";

  // Get a percentage rounded to two decimal digits. This avoids
  // implementation-defined rounding inside printf.
  double Percent = rint(((double)N / D) * 100.0 * 100.0) / 100.0;
  return OS << format("0x%08" PRIx32 " / 0x%08" PRIx32 " = %.2f%%", N, D,
                      Percent);
}

#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
LLVM_DUMP_METHOD void BranchProbability::dump() const { print(dbgs()) << '\n'; }
#endif

BranchProbability::BranchProbability(uint32_t Numerator, uint32_t Denominator) {
  assert(Denominator > 0 && "Denominator cannot be 0!");
  assert(Numerator <= Denominator && "Probability cannot be bigger than 1!");
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 7 direct dependencies, including `llvm/Support/BranchProbability.h`, `llvm/Config/llvm-config.h`, `llvm/Support/Debug.h`, `llvm/Support/Format.h`.
  CN: 引入了 7 个直接依赖，其中包括 `llvm/Support/BranchProbability.h`, `llvm/Config/llvm-config.h`, `llvm/Support/Debug.h`, `llvm/Support/Format.h`。
- EN: This section centers on `format`, `dump`, `BranchProbability` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `format`, `dump`, `BranchProbability` 等符号展开，负责为下游使用方写出、格式化或序列化结果。

### Lines 41-80

```cpp
  if (Denominator == D)
    N = Numerator;
  else {
    uint64_t Prob64 =
        (Numerator * static_cast<uint64_t>(D) + Denominator / 2) / Denominator;
    N = static_cast<uint32_t>(Prob64);
  }
}

BranchProbability
BranchProbability::getBranchProbability(uint64_t Numerator,
                                        uint64_t Denominator) {
  assert(Numerator <= Denominator && "Probability cannot be bigger than 1!");
  // Scale down Denominator to fit in a 32-bit integer.
  int Scale = 0;
  while (Denominator > UINT32_MAX) {
    Denominator >>= 1;
    Scale++;
  }
  return BranchProbability(Numerator >> Scale, Denominator);
}

BranchProbability BranchProbability::getBranchProbability(double Prob) {
  assert(0 <= Prob && Prob <= 1 && "Probability must be between 0 and 1!");
  return BranchProbability(std::round(Prob * D), D);
}

// If ConstD is not zero, then replace D by ConstD so that division and modulo
// operations by D can be optimized, in case this function is not inlined by the
// compiler.
template <uint32_t ConstD>
static uint64_t scale(uint64_t Num, uint32_t N, uint32_t D) {
  if (ConstD > 0)
    D = ConstD;

  assert(D && "divide by 0");

  // Fast path for multiplying by 1.0.
  if (!Num || D == N)
    return Num;
```
- EN: This section centers on `getBranchProbability`, `assert`, `BranchProbability` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `getBranchProbability`, `assert`, `BranchProbability` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 81-120

```cpp

  // Split Num into upper and lower parts to multiply, then recombine.
  uint64_t ProductHigh = (Num >> 32) * N;
  uint64_t ProductLow = (Num & UINT32_MAX) * N;

  // Split into 32-bit digits.
  uint32_t Upper32 = ProductHigh >> 32;
  uint32_t Lower32 = ProductLow & UINT32_MAX;
  uint32_t Mid32Partial = ProductHigh & UINT32_MAX;
  uint32_t Mid32 = Mid32Partial + (ProductLow >> 32);

  // Carry.
  Upper32 += Mid32 < Mid32Partial;

  uint64_t Rem = (uint64_t(Upper32) << 32) | Mid32;
  uint64_t UpperQ = Rem / D;

  // Check for overflow.
  if (UpperQ > UINT32_MAX)
    return UINT64_MAX;

  Rem = ((Rem % D) << 32) | Lower32;
  uint64_t LowerQ = Rem / D;
  uint64_t Q = (UpperQ << 32) + LowerQ;

  // Check for overflow.
  return Q < LowerQ ? UINT64_MAX : Q;
}

uint64_t BranchProbability::scale(uint64_t Num) const {
  return ::scale<D>(Num, N, D);
}

uint64_t BranchProbability::scaleByInverse(uint64_t Num) const {
  return ::scale<0>(Num, D, N);
}

BranchProbability BranchProbability::pow(unsigned N) const {
  BranchProbability Res = BranchProbability::getOne();
  for (unsigned I = 0; I < N; ++I)
```
- EN: This section centers on `scale`, `scaleByInverse`, `pow` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `scale`, `scaleByInverse`, `pow` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 121-123

```cpp
    Res *= *this;
  return Res;
}
```
- EN: This range returns the resulting value to its callers.
  CN: 这一段将结果返回给调用方。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `format`, `dump`, `BranchProbability`, `assert` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Formatting, emission, and outward serialization. / 重点关注格式化、输出与序列化。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/Support/BranchProbability.h`, `llvm/Config/llvm-config.h`, `llvm/Support/Debug.h`, `llvm/Support/Format.h`, `llvm/Support/raw_ostream.h`
- Standard library / 标准库: `cmath`
- Other/system headers / 其他或系统头文件: `cassert`
- Related symbols / 相关符号: `format`, `dump`, `BranchProbability`, `assert`, `getBranchProbability`
