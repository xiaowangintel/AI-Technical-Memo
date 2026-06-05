# BalancedPartitioning.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/BalancedPartitioning.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements BalancedPartitioning, a recursive balanced graph partitioning algorithm.
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `BalancedPartitioning` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===- BalancedPartitioning.cpp -------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements BalancedPartitioning, a recursive balanced graph
// partitioning algorithm.
//
//===----------------------------------------------------------------------===//

#include "llvm/Support/BalancedPartitioning.h"
#include "llvm/Config/llvm-config.h" // for LLVM_ENABLE_THREADS
#include "llvm/Support/Debug.h"
#include "llvm/Support/Format.h"
#include "llvm/Support/FormatVariadic.h"
#include "llvm/Support/ThreadPool.h"

using namespace llvm;
#define DEBUG_TYPE "balanced-partitioning"

void BPFunctionNode::dump(raw_ostream &OS) const {
  OS << formatv("{{ID={0} Utilities={{{1:$[,]}} Bucket={2}}", Id,
                make_range(UtilityNodes.begin(), UtilityNodes.end()), Bucket);
}

template <typename Func>
void BalancedPartitioning::BPThreadPool::async(Func &&F) {
#if LLVM_ENABLE_THREADS
  // This new thread could spawn more threads, so mark it as active
  ++NumActiveThreads;
  TheThreadPool.async([this, F]() {
    // Run the task
    F();

    // This thread will no longer spawn new threads, so mark it as inactive
    if (--NumActiveThreads == 0) {
      // There are no more active threads, so mark as finished and notify
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 6 direct dependencies, including `llvm/Support/BalancedPartitioning.h`, `llvm/Config/llvm-config.h`, `llvm/Support/Debug.h`, `llvm/Support/Format.h`.
  CN: 引入了 6 个直接依赖，其中包括 `llvm/Support/BalancedPartitioning.h`, `llvm/Config/llvm-config.h`, `llvm/Support/Debug.h`, `llvm/Support/Format.h`。
- EN: This section centers on `dump`, `make_range`, `async` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `dump`, `make_range`, `async` 等符号展开，负责创建并初始化辅助对象或状态。

### Lines 41-80

```cpp
      {
        std::unique_lock<std::mutex> lock(mtx);
        assert(!IsFinishedSpawning);
        IsFinishedSpawning = true;
      }
      cv.notify_one();
    }
  });
#else
  llvm_unreachable("threads are disabled");
#endif
}

void BalancedPartitioning::BPThreadPool::wait() {
#if LLVM_ENABLE_THREADS
  // TODO: We could remove the mutex and condition variable and use
  // std::atomic::wait() instead, but that isn't available until C++20
  {
    std::unique_lock<std::mutex> lock(mtx);
    cv.wait(lock, [&]() { return IsFinishedSpawning; });
    assert(IsFinishedSpawning && NumActiveThreads == 0);
  }
  // Now we can call ThreadPool::wait() since all tasks have been submitted
  TheThreadPool.wait();
#else
  llvm_unreachable("threads are disabled");
#endif
}

BalancedPartitioning::BalancedPartitioning(
    const BalancedPartitioningConfig &Config)
    : Config(Config) {
  // Pre-computing log2 values
  Log2Cache[0] = 0.0;
  for (unsigned I = 1; I < LOG_CACHE_SIZE; I++)
    Log2Cache[I] = std::log2(I);
}

void BalancedPartitioning::run(std::vector<BPFunctionNode> &Nodes) const {
  LLVM_DEBUG(
```
- EN: This section centers on `lock`, `assert`, `llvm_unreachable` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `lock`, `assert`, `llvm_unreachable` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码遍历集合、区间或记录，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 81-120

```cpp
      dbgs() << format(
          "Partitioning %d nodes using depth %d and %d iterations per split\n",
          Nodes.size(), Config.SplitDepth, Config.IterationsPerSplit));
  std::optional<BPThreadPool> TP;
#if LLVM_ENABLE_THREADS
  DefaultThreadPool TheThreadPool;
  if (Config.TaskSplitDepth > 1)
    TP.emplace(TheThreadPool);
#endif

  // Record the input order
  for (unsigned I = 0; I < Nodes.size(); I++)
    Nodes[I].InputOrderIndex = I;

  auto NodesRange = llvm::make_range(Nodes.begin(), Nodes.end());
  auto BisectTask = [this, NodesRange, &TP]() {
    bisect(NodesRange, /*RecDepth=*/0, /*RootBucket=*/1, /*Offset=*/0, TP);
  };
  if (TP) {
    TP->async(std::move(BisectTask));
    TP->wait();
  } else {
    BisectTask();
  }

  llvm::stable_sort(NodesRange, [](const auto &L, const auto &R) {
    return L.Bucket < R.Bucket;
  });

  LLVM_DEBUG(dbgs() << "Balanced partitioning completed\n");
}

void BalancedPartitioning::bisect(const FunctionNodeRange Nodes,
                                  unsigned RecDepth, unsigned RootBucket,
                                  unsigned Offset,
                                  std::optional<BPThreadPool> &TP) const {
  unsigned NumNodes = llvm::size(Nodes);
  if (NumNodes <= 1 || RecDepth >= Config.SplitDepth) {
    // We've reach the lowest level of the recursion tree. Fall back to the
    // original order and assign to buckets.
```
- EN: This section centers on `dbgs`, `bisect`, `BisectTask` and performs utility computation and state updates.
  CN: 这一段主要围绕 `dbgs`, `bisect`, `BisectTask` 等符号展开，负责执行工具性计算并更新状态。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 121-160

```cpp
    llvm::sort(Nodes, [](const auto &L, const auto &R) {
      return L.InputOrderIndex < R.InputOrderIndex;
    });
    for (auto &N : Nodes)
      N.Bucket = Offset++;
    return;
  }

  LLVM_DEBUG(dbgs() << format("Bisect with %d nodes and root bucket %d\n",
                              NumNodes, RootBucket));

  std::mt19937 RNG(RootBucket);

  unsigned LeftBucket = 2 * RootBucket;
  unsigned RightBucket = 2 * RootBucket + 1;

  // Split into two and assign to the left and right buckets
  split(Nodes, LeftBucket);

  runIterations(Nodes, LeftBucket, RightBucket, RNG);

  // Split nodes wrt the resulting buckets
  auto NodesMid =
      llvm::partition(Nodes, [&](auto &N) { return N.Bucket == LeftBucket; });
  unsigned MidOffset = Offset + std::distance(Nodes.begin(), NodesMid);

  auto LeftNodes = llvm::make_range(Nodes.begin(), NodesMid);
  auto RightNodes = llvm::make_range(NodesMid, Nodes.end());

  auto LeftRecTask = [this, LeftNodes, RecDepth, LeftBucket, Offset, &TP]() {
    bisect(LeftNodes, RecDepth + 1, LeftBucket, Offset, TP);
  };
  auto RightRecTask = [this, RightNodes, RecDepth, RightBucket, MidOffset,
                       &TP]() {
    bisect(RightNodes, RecDepth + 1, RightBucket, MidOffset, TP);
  };

  if (TP && RecDepth < Config.TaskSplitDepth && NumNodes >= 4) {
    TP->async(std::move(LeftRecTask));
    TP->async(std::move(RightRecTask));
```
- EN: This section centers on `sort`, `RNG`, `split` and performs utility computation and state updates.
  CN: 这一段主要围绕 `sort`, `RNG`, `split` 等符号展开，负责执行工具性计算并更新状态。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 161-200

```cpp
  } else {
    LeftRecTask();
    RightRecTask();
  }
}

void BalancedPartitioning::runIterations(const FunctionNodeRange Nodes,
                                         unsigned LeftBucket,
                                         unsigned RightBucket,
                                         std::mt19937 &RNG) const {
  unsigned NumNodes = llvm::size(Nodes);
  DenseMap<BPFunctionNode::UtilityNodeT, unsigned> UtilityNodeIndex;
  for (auto &N : Nodes)
    for (auto &UN : N.UtilityNodes)
      ++UtilityNodeIndex[UN];
  // Remove utility nodes if they have just one edge or are connected to all
  // functions
  for (auto &N : Nodes)
    llvm::erase_if(N.UtilityNodes, [&](auto &UN) {
      unsigned UNI = UtilityNodeIndex[UN];
      return UNI == 1 || UNI == NumNodes;
    });

  // Renumber utility nodes so they can be used to index into Signatures
  UtilityNodeIndex.clear();
  for (auto &N : Nodes)
    for (auto &UN : N.UtilityNodes)
      UN = UtilityNodeIndex.insert({UN, UtilityNodeIndex.size()}).first->second;

  // Initialize signatures
  SignaturesT Signatures(/*Size=*/UtilityNodeIndex.size());
  for (auto &N : Nodes) {
    for (auto &UN : N.UtilityNodes) {
      assert(UN < Signatures.size());
      if (N.Bucket == LeftBucket) {
        Signatures[UN].LeftCount++;
      } else {
        Signatures[UN].RightCount++;
      }
    }
```
- EN: This section centers on `LeftRecTask`, `RightRecTask`, `runIterations` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `LeftRecTask`, `RightRecTask`, `runIterations` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 201-240

```cpp
  }

  for (unsigned I = 0; I < Config.IterationsPerSplit; I++) {
    unsigned NumMovedNodes =
        runIteration(Nodes, LeftBucket, RightBucket, Signatures, RNG);
    if (NumMovedNodes == 0)
      break;
  }
}

unsigned BalancedPartitioning::runIteration(const FunctionNodeRange Nodes,
                                            unsigned LeftBucket,
                                            unsigned RightBucket,
                                            SignaturesT &Signatures,
                                            std::mt19937 &RNG) const {
  // Init signature cost caches
  for (auto &Signature : Signatures) {
    if (Signature.CachedGainIsValid)
      continue;
    unsigned L = Signature.LeftCount;
    unsigned R = Signature.RightCount;
    assert((L > 0 || R > 0) && "incorrect signature");
    float Cost = logCost(L, R);
    Signature.CachedGainLR = 0.f;
    Signature.CachedGainRL = 0.f;
    if (L > 0)
      Signature.CachedGainLR = Cost - logCost(L - 1, R + 1);
    if (R > 0)
      Signature.CachedGainRL = Cost - logCost(L + 1, R - 1);
    Signature.CachedGainIsValid = true;
  }

  // Compute move gains
  using GainPair = std::pair<float, BPFunctionNode *>;
  std::vector<GainPair> Gains;
  for (auto &N : Nodes) {
    bool FromLeftToRight = (N.Bucket == LeftBucket);
    float Gain = moveGain(N, FromLeftToRight, Signatures);
    Gains.push_back(std::make_pair(Gain, &N));
  }
```
- EN: This section centers on `runIteration`, `assert` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `runIteration`, `assert` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 241-280

```cpp

  // Collect left and right gains
  auto LeftEnd = llvm::partition(
      Gains, [&](const auto &GP) { return GP.second->Bucket == LeftBucket; });
  auto LeftRange = llvm::make_range(Gains.begin(), LeftEnd);
  auto RightRange = llvm::make_range(LeftEnd, Gains.end());

  // Sort gains in descending order
  auto LargerGain = [](const auto &L, const auto &R) {
    return L.first > R.first;
  };
  llvm::stable_sort(LeftRange, LargerGain);
  llvm::stable_sort(RightRange, LargerGain);

  unsigned NumMovedDataVertices = 0;
  for (auto [LeftPair, RightPair] : llvm::zip(LeftRange, RightRange)) {
    auto &[LeftGain, LeftNode] = LeftPair;
    auto &[RightGain, RightNode] = RightPair;
    // Stop when the gain is no longer beneficial
    if (LeftGain + RightGain <= 0.f)
      break;
    // Try to exchange the nodes between buckets
    if (moveFunctionNode(*LeftNode, LeftBucket, RightBucket, Signatures, RNG))
      ++NumMovedDataVertices;
    if (moveFunctionNode(*RightNode, LeftBucket, RightBucket, Signatures, RNG))
      ++NumMovedDataVertices;
  }
  return NumMovedDataVertices;
}

bool BalancedPartitioning::moveFunctionNode(BPFunctionNode &N,
                                            unsigned LeftBucket,
                                            unsigned RightBucket,
                                            SignaturesT &Signatures,
                                            std::mt19937 &RNG) const {
  // Sometimes we skip the move. This helps to escape local optima
  if (std::uniform_real_distribution<float>(0.f, 1.f)(RNG) <=
      Config.SkipProbability)
    return false;

```
- EN: This section centers on `stable_sort`, `moveFunctionNode` and performs utility computation and state updates.
  CN: 这一段主要围绕 `stable_sort`, `moveFunctionNode` 等符号展开，负责执行工具性计算并更新状态。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 281-320

```cpp
  bool FromLeftToRight = (N.Bucket == LeftBucket);
  // Update the current bucket
  N.Bucket = (FromLeftToRight ? RightBucket : LeftBucket);

  // Update signatures and invalidate gain cache
  if (FromLeftToRight) {
    for (auto &UN : N.UtilityNodes) {
      auto &Signature = Signatures[UN];
      Signature.LeftCount--;
      Signature.RightCount++;
      Signature.CachedGainIsValid = false;
    }
  } else {
    for (auto &UN : N.UtilityNodes) {
      auto &Signature = Signatures[UN];
      Signature.LeftCount++;
      Signature.RightCount--;
      Signature.CachedGainIsValid = false;
    }
  }
  return true;
}

void BalancedPartitioning::split(const FunctionNodeRange Nodes,
                                 unsigned StartBucket) const {
  unsigned NumNodes = llvm::size(Nodes);
  auto NodesMid = Nodes.begin() + (NumNodes + 1) / 2;

  llvm::sort(Nodes, [](auto &L, auto &R) {
    return L.InputOrderIndex < R.InputOrderIndex;
  });

  for (auto &N : llvm::make_range(Nodes.begin(), NodesMid))
    N.Bucket = StartBucket;
  for (auto &N : llvm::make_range(NodesMid, Nodes.end()))
    N.Bucket = StartBucket + 1;
}

float BalancedPartitioning::moveGain(const BPFunctionNode &N,
                                     bool FromLeftToRight,
```
- EN: This section centers on `split`, `sort` and performs utility computation and state updates.
  CN: 这一段主要围绕 `split`, `sort` 等符号展开，负责执行工具性计算并更新状态。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 321-335

```cpp
                                     const SignaturesT &Signatures) {
  float Gain = 0.f;
  for (auto &UN : N.UtilityNodes)
    Gain += (FromLeftToRight ? Signatures[UN].CachedGainLR
                             : Signatures[UN].CachedGainRL);
  return Gain;
}

float BalancedPartitioning::logCost(unsigned X, unsigned Y) const {
  return -(X * log2Cached(X + 1) + Y * log2Cached(Y + 1));
}

float BalancedPartitioning::log2Cached(unsigned i) const {
  return (i < LOG_CACHE_SIZE) ? Log2Cache[i] : std::log2(i);
}
```
- EN: This section centers on `logCost`, `log2Cached` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `logCost`, `log2Cached` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and returns the resulting value to its callers.
  CN: 在这一段中，代码遍历集合、区间或记录，并将结果返回给调用方。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `dump`, `make_range`, `async`, `F` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Formatting, emission, and outward serialization. / 重点关注格式化、输出与序列化。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/Support/BalancedPartitioning.h`, `llvm/Config/llvm-config.h`, `llvm/Support/Debug.h`, `llvm/Support/Format.h`, `llvm/Support/FormatVariadic.h`, `llvm/Support/ThreadPool.h`
- Standard library / 标准库: None / 无
- Other/system headers / 其他或系统头文件: None / 无
- Related symbols / 相关符号: `dump`, `make_range`, `async`, `F`, `lock`
