# CacheMetrics.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Passes/CacheMetrics.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: Metrics for instruction cache. It also sits in code that implements BOLT optimization and transformation passes. / 该文件实现 BOLT 优化与变换 Pass。 源码头部说明其职责是：Metrics for instruction cache。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13

```cpp
//===- bolt/Passes/CacheMetrics.cpp - Metrics for instruction cache -------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the CacheMetrics class and functions for showing metrics
// of cache lines.
//
//===----------------------------------------------------------------------===//
```

- EN: Introduces type definitions such as `and`. Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `and`.
- CN: 这里引入类型定义，例如 `and`。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `and`。

### Lines 14-21

```cpp
#include "bolt/Passes/CacheMetrics.h"
#include "bolt/Core/BinaryBasicBlock.h"
#include "bolt/Core/BinaryFunction.h"
#include <unordered_map>

using namespace llvm;
using namespace bolt;
```

- EN: Pulls in 4 header(s) from local project, system dependencies needed by this range. Works inside namespace scope `llvm`, `bolt` to organize symbols. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `llvm`, `bolt`.
- CN: 这里引入 4 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `llvm`, `bolt`。

### Lines 22-30

```cpp
namespace {

/// The following constants are used to estimate the number of i-TLB cache
/// misses for a given code layout. Empirically the values result in high
/// correlations between the estimations and the perf measurements.
/// The constants do not affect the code layout algorithms.
constexpr unsigned ITLBPageSize = 4096;
constexpr unsigned ITLBEntries = 16;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 31-48

```cpp
/// Initialize and return a position map for binary basic blocks
void extractBasicBlockInfo(
    const BinaryFunctionListType &BinaryFunctions,
    std::unordered_map<BinaryBasicBlock *, uint64_t> &BBAddr,
    std::unordered_map<BinaryBasicBlock *, uint64_t> &BBSize) {

  for (BinaryFunction *BF : BinaryFunctions) {
    const BinaryContext &BC = BF->getBinaryContext();
    for (BinaryBasicBlock &BB : *BF) {
      if (BF->isSimple() || BC.HasRelocations) {
        // Use addresses/sizes as in the output binary
        BBAddr[&BB] = BB.getOutputAddressRange().first;
        BBSize[&BB] = BB.getOutputSize();
      } else {
        // Output ranges should match the input if the body hasn't changed
        BBAddr[&BB] = BB.getInputAddressRange().first + BF->getAddress();
        BBSize[&BB] = BB.getOriginalSize();
      }
```

- EN: Declares or implements routines including `getBinaryContext`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getBinaryContext`.
- CN: 这里声明或实现函数，例如 `getBinaryContext`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getBinaryContext`。

### Lines 49-66

```cpp
    }
  }
}

/// Calculate TSP metric, which quantifies the number of fallthrough jumps in
/// the ordering of basic blocks. The method returns a pair
/// (the number of fallthrough branches, the total number of branches)
std::pair<uint64_t, uint64_t>
calcTSPScore(const BinaryFunctionListType &BinaryFunctions,
             const std::unordered_map<BinaryBasicBlock *, uint64_t> &BBAddr,
             const std::unordered_map<BinaryBasicBlock *, uint64_t> &BBSize) {
  uint64_t Score = 0;
  uint64_t JumpCount = 0;
  for (BinaryFunction *BF : BinaryFunctions) {
    if (!BF->hasProfile())
      continue;
    for (BinaryBasicBlock *SrcBB : BF->getLayout().blocks()) {
      auto BI = SrcBB->branch_info_begin();
```

- EN: Declares or implements routines including `branch_info_begin`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `branch_info_begin`.
- CN: 这里声明或实现函数，例如 `branch_info_begin`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `branch_info_begin`。

### Lines 67-74

```cpp
      for (BinaryBasicBlock *DstBB : SrcBB->successors()) {
        if (SrcBB != DstBB && BI->Count != BinaryBasicBlock::COUNT_NO_PROFILE) {
          JumpCount += BI->Count;

          auto BBAddrIt = BBAddr.find(SrcBB);
          assert(BBAddrIt != BBAddr.end());
          uint64_t SrcBBAddr = BBAddrIt->second;
```

- EN: Declares or implements routines including `assert`. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里较值得关注的符号包括 `assert`。

### Lines 75-82

```cpp
          auto BBSizeIt = BBSize.find(SrcBB);
          assert(BBSizeIt != BBSize.end());
          uint64_t SrcBBSize = BBSizeIt->second;

          BBAddrIt = BBAddr.find(DstBB);
          assert(BBAddrIt != BBAddr.end());
          uint64_t DstBBAddr = BBAddrIt->second;
```

- EN: Declares or implements routines including `assert`. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里较值得关注的符号包括 `assert`。

### Lines 83-92

```cpp
          if (SrcBBAddr + SrcBBSize == DstBBAddr)
            Score += BI->Count;
        }
        ++BI;
      }
    }
  }
  return std::make_pair(Score, JumpCount);
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 93-100

```cpp
using Predecessors = std::vector<std::pair<BinaryFunction *, uint64_t>>;

/// Build a simplified version of the call graph: For every function, keep
/// its callers and the frequencies of the calls
std::unordered_map<const BinaryFunction *, Predecessors>
extractFunctionCalls(const BinaryFunctionListType &BinaryFunctions) {
  std::unordered_map<const BinaryFunction *, Predecessors> Calls;
```

- EN: Declares or implements routines including `extractFunctionCalls`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `extractFunctionCalls`.
- CN: 这里声明或实现函数，例如 `extractFunctionCalls`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `extractFunctionCalls`。

### Lines 101-108

```cpp
  for (BinaryFunction *SrcFunction : BinaryFunctions) {
    const BinaryContext &BC = SrcFunction->getBinaryContext();
    for (const BinaryBasicBlock *BB : SrcFunction->getLayout().blocks()) {
      // Find call instructions and extract target symbols from each one
      for (const MCInst &Inst : *BB) {
        if (!BC.MIB->isCall(Inst))
          continue;
```

- EN: Declares or implements routines including `getBinaryContext`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getBinaryContext`.
- CN: 这里声明或实现函数，例如 `getBinaryContext`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getBinaryContext`。

### Lines 109-121

```cpp
        // Call info
        const MCSymbol *DstSym = BC.MIB->getTargetSymbol(Inst);
        uint64_t Count = BB->getKnownExecutionCount();
        // Ignore calls w/o information
        if (DstSym == nullptr || Count == 0)
          continue;

        const BinaryFunction *DstFunction = BC.getFunctionForSymbol(DstSym);
        // Ignore recursive calls
        if (DstFunction == nullptr || DstFunction->getLayout().block_empty() ||
            DstFunction == SrcFunction)
          continue;
```

- EN: Declares or implements routines including `getTargetSymbol`, `getKnownExecutionCount`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getTargetSymbol`, `getKnownExecutionCount`.
- CN: 这里声明或实现函数，例如 `getTargetSymbol`, `getKnownExecutionCount`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getTargetSymbol`, `getKnownExecutionCount`。

### Lines 122-129

```cpp
        // Record the call
        Calls[DstFunction].emplace_back(SrcFunction, Count);
      }
    }
  }
  return Calls;
}
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 130-147

```cpp
/// Compute expected hit ratio of the i-TLB cache (optimized by HFSortPlus alg).
/// Given an assignment of functions to the i-TLB pages), we divide all
/// functions calls into two categories:
/// - 'short' ones that have a caller-callee distance less than a page;
/// - 'long' ones where the distance exceeds a page.
/// The short calls are likely to result in a i-TLB cache hit. For the long
/// ones, the hit/miss result depends on the 'hotness' of the page (i.e., how
/// often the page is accessed). Assuming that functions are sent to the i-TLB
/// cache in a random order, the probability that a page is present in the cache
/// is proportional to the number of samples corresponding to the functions on
/// the page. The following procedure detects short and long calls, and
/// estimates the expected number of cache misses for the long ones.
double expectedCacheHitRatio(
    const BinaryFunctionListType &BinaryFunctions,
    const std::unordered_map<BinaryBasicBlock *, uint64_t> &BBAddr,
    const std::unordered_map<BinaryBasicBlock *, uint64_t> &BBSize) {
  std::unordered_map<const BinaryFunction *, Predecessors> Calls =
      extractFunctionCalls(BinaryFunctions);
```

- EN: Declares or implements routines including `extractFunctionCalls`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `extractFunctionCalls`.
- CN: 这里声明或实现函数，例如 `extractFunctionCalls`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `extractFunctionCalls`。

### Lines 148-159

```cpp
  // Compute 'hotness' of the functions
  double TotalSamples = 0;
  std::unordered_map<BinaryFunction *, double> FunctionSamples;
  for (BinaryFunction *BF : BinaryFunctions) {
    double Samples = 0;
    for (std::pair<BinaryFunction *, uint64_t> Pair : Calls[BF])
      Samples += Pair.second;
    Samples = std::max(Samples, (double)BF->getKnownExecutionCount());
    FunctionSamples[BF] = Samples;
    TotalSamples += Samples;
  }
```

- EN: Declares or implements routines including `max`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `max`.
- CN: 这里声明或实现函数，例如 `max`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `max`。

### Lines 160-168

```cpp
  // Compute 'hotness' of the pages
  std::unordered_map<uint64_t, double> PageSamples;
  for (BinaryFunction *BF : BinaryFunctions) {
    if (BF->getLayout().block_empty())
      continue;
    auto BBAddrIt = BBAddr.find(BF->getLayout().block_front());
    assert(BBAddrIt != BBAddr.end());
    const uint64_t Page = BBAddrIt->second / ITLBPageSize;
```

- EN: Declares or implements routines including `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`。

### Lines 169-183

```cpp
    auto FunctionSamplesIt = FunctionSamples.find(BF);
    assert(FunctionSamplesIt != FunctionSamples.end());
    PageSamples[Page] += FunctionSamplesIt->second;
  }

  // Computing the expected number of misses for every function
  double Misses = 0;
  for (BinaryFunction *BF : BinaryFunctions) {
    // Skip the function if it has no samples
    auto FunctionSamplesIt = FunctionSamples.find(BF);
    assert(FunctionSamplesIt != FunctionSamples.end());
    double Samples = FunctionSamplesIt->second;
    if (BF->getLayout().block_empty() || Samples == 0.0)
      continue;
```

- EN: Declares or implements routines including `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`。

### Lines 184-194

```cpp
    auto BBAddrIt = BBAddr.find(BF->getLayout().block_front());
    assert(BBAddrIt != BBAddr.end());
    const uint64_t Page = BBAddrIt->second / ITLBPageSize;
    // The probability that the page is not present in the cache
    const double MissProb =
        pow(1.0 - PageSamples[Page] / TotalSamples, ITLBEntries);

    // Processing all callers of the function
    for (std::pair<BinaryFunction *, uint64_t> Pair : Calls[BF]) {
      BinaryFunction *SrcFunction = Pair.first;
```

- EN: Declares or implements routines including `assert`, `pow`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`, `pow`.
- CN: 这里声明或实现函数，例如 `assert`, `pow`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`, `pow`。

### Lines 195-209

```cpp
      BBAddrIt = BBAddr.find(SrcFunction->getLayout().block_front());
      assert(BBAddrIt != BBAddr.end());
      const uint64_t SrcPage = BBAddrIt->second / ITLBPageSize;
      // Is this a 'long' or a 'short' call?
      if (Page != SrcPage) {
        // This is a miss
        Misses += MissProb * Pair.second;
      }
      Samples -= Pair.second;
    }
    assert(Samples >= 0.0 && "Function samples computed incorrectly");
    // The remaining samples likely come from the jitted code
    Misses += Samples * MissProb;
  }
```

- EN: Declares or implements routines including `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`。

### Lines 210-223

```cpp
  return 100.0 * (1.0 - Misses / TotalSamples);
}

} // namespace

void CacheMetrics::printAll(raw_ostream &OS,
                            const BinaryFunctionListType &BFs) {
  // Stats related to hot-cold code splitting
  size_t NumFunctions = 0;
  size_t NumProfiledFunctions = 0;
  size_t NumHotFunctions = 0;
  size_t NumBlocks = 0;
  size_t NumHotBlocks = 0;
```

- EN: Works inside namespace scope `void` to organize symbols. Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `void`.
- CN: 这里位于命名空间 `void` 中，用于组织符号作用域。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `void`。

### Lines 224-241

```cpp
  size_t TotalCodeMinAddr = std::numeric_limits<size_t>::max();
  size_t TotalCodeMaxAddr = 0;
  size_t HotCodeMinAddr = std::numeric_limits<size_t>::max();
  size_t HotCodeMaxAddr = 0;

  for (BinaryFunction *BF : BFs) {
    NumFunctions++;
    if (BF->hasProfile())
      NumProfiledFunctions++;
    if (BF->hasValidIndex())
      NumHotFunctions++;
    for (const BinaryBasicBlock &BB : *BF) {
      NumBlocks++;
      size_t BBAddrMin = BB.getOutputAddressRange().first;
      size_t BBAddrMax = BB.getOutputAddressRange().second;
      TotalCodeMinAddr = std::min(TotalCodeMinAddr, BBAddrMin);
      TotalCodeMaxAddr = std::max(TotalCodeMaxAddr, BBAddrMax);
      if (BF->hasValidIndex() && !BB.isCold()) {
```

- EN: Declares or implements routines including `max`, `min`. Notable symbols here include `max`, `min`.
- CN: 这里声明或实现函数，例如 `max`, `min`。这里较值得关注的符号包括 `max`, `min`。

### Lines 242-257

```cpp
        NumHotBlocks++;
        HotCodeMinAddr = std::min(HotCodeMinAddr, BBAddrMin);
        HotCodeMaxAddr = std::max(HotCodeMaxAddr, BBAddrMax);
      }
    }
  }

  OS << format("  There are %zu functions;", NumFunctions)
     << format(" %zu (%.2lf%%) are in the hot section,", NumHotFunctions,
               100.0 * NumHotFunctions / NumFunctions)
     << format(" %zu (%.2lf%%) have profile\n", NumProfiledFunctions,
               100.0 * NumProfiledFunctions / NumFunctions);
  OS << format("  There are %zu basic blocks;", NumBlocks)
     << format(" %zu (%.2lf%%) are in the hot section\n", NumHotBlocks,
               100.0 * NumHotBlocks / NumBlocks);
```

- EN: Declares or implements routines including `min`, `max`, `format`. Notable symbols here include `min`, `max`, `format`.
- CN: 这里声明或实现函数，例如 `min`, `max`, `format`。这里较值得关注的符号包括 `min`, `max`, `format`。

### Lines 258-267

```cpp
  assert(TotalCodeMinAddr <= TotalCodeMaxAddr && "incorrect output addresses");
  size_t HotCodeSize = HotCodeMaxAddr - HotCodeMinAddr;
  size_t TotalCodeSize = TotalCodeMaxAddr - TotalCodeMinAddr;

  size_t HugePage2MB = 2 << 20;
  OS << format("  Hot code takes %.2lf%% of binary (%zu bytes out of %zu, "
               "%.2lf huge pages)\n",
               100.0 * HotCodeSize / TotalCodeSize, HotCodeSize, TotalCodeSize,
               double(HotCodeSize) / HugePage2MB);
```

- EN: Declares or implements routines including `assert`, `double`. Notable symbols here include `assert`, `double`.
- CN: 这里声明或实现函数，例如 `assert`, `double`。这里较值得关注的符号包括 `assert`, `double`。

### Lines 268-275

```cpp
  // Stats related to expected cache performance
  std::unordered_map<BinaryBasicBlock *, uint64_t> BBAddr;
  std::unordered_map<BinaryBasicBlock *, uint64_t> BBSize;
  extractBasicBlockInfo(BFs, BBAddr, BBSize);

  OS << "  Expected i-TLB cache hit ratio: "
     << format("%.2lf%%\n", expectedCacheHitRatio(BFs, BBAddr, BBSize));
```

- EN: Declares or implements routines including `extractBasicBlockInfo`, `format`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `extractBasicBlockInfo`, `format`.
- CN: 这里声明或实现函数，例如 `extractBasicBlockInfo`, `format`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `extractBasicBlockInfo`, `format`。

### Lines 276-281

```cpp
  auto Stats = calcTSPScore(BFs, BBAddr, BBSize);
  OS << "  TSP score: "
     << format("%.2lf%% (%zu out of %zu)\n",
               100.0 * Stats.first / std::max<uint64_t>(Stats.second, 1),
               Stats.first, Stats.second);
}
```

- EN: Declares or implements routines including `calcTSPScore`, `format`. Notable symbols here include `calcTSPScore`, `format`.
- CN: 这里声明或实现函数，例如 `calcTSPScore`, `format`。这里较值得关注的符号包括 `calcTSPScore`, `format`。

## Key Concepts / 关键概念

- `and`: class or struct interface / 类或结构体接口
- `getBinaryContext`: function or method entry point / 函数或方法入口
- `branch_info_begin`: function or method entry point / 函数或方法入口
- `assert`: function or method entry point / 函数或方法入口
- `extractFunctionCalls`: function or method entry point / 函数或方法入口
- `getTargetSymbol`: function or method entry point / 函数或方法入口
- `llvm`: namespace scope / 命名空间作用域
- `bolt`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/CacheMetrics.h`, `bolt/Core/BinaryBasicBlock.h`, `bolt/Core/BinaryFunction.h`
- System headers / 系统头文件: `unordered_map`
- Directory context / 目录上下文: `bolt/lib/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Passes` 下的相邻文件通常与本文件协作组成对应子系统
