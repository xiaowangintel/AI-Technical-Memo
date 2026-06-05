# SpeculateAnalyses.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/Orc/SpeculateAnalyses.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements ORC JIT infrastructure such as execution sessions, JITDylibs, materialization, symbol lookup, and execution helpers.
  - **CN**: 实现 ORC JIT 基础设施，例如 ExecutionSession、JITDylib、物化流程、符号查找以及执行辅助组件。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```cpp
//===-- SpeculateAnalyses.cpp  --*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/ExecutionEngine/Orc/SpeculateAnalyses.h"
#include "llvm/ADT/ArrayRef.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/Analysis/BlockFrequencyInfo.h"
#include "llvm/Analysis/BranchProbabilityInfo.h"
#include "llvm/Analysis/CFG.h"
#include "llvm/IR/PassManager.h"
#include "llvm/Passes/PassBuilder.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ExecutionEngine/Orc/SpeculateAnalyses.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/STLExtras.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ExecutionEngine/Orc/SpeculateAnalyses.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/STLExtras.h`。

### Lines 19-36
```cpp
#include "llvm/Support/ErrorHandling.h"

namespace {
using namespace llvm;
SmallVector<const BasicBlock *, 8> findBBwithCalls(const Function &F,
                                                   bool IndirectCall = false) {
  SmallVector<const BasicBlock *, 8> BBs;

  auto findCallInst = [&IndirectCall](const Instruction &I) {
    if (auto Call = dyn_cast<CallBase>(&I))
      if (!isa<PseudoProbeInst>(Call))
        return Call->isIndirectCall() ? IndirectCall : true;
    return false;
  };
  for (auto &BB : F)
    if (findCallInst(*BB.getTerminator()) || llvm::any_of(BB, findCallInst))
      BBs.emplace_back(&BB);

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/Support/ErrorHandling.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/Support/ErrorHandling.h`。

### Lines 37-46
```cpp
  return BBs;
}
} // namespace

// Implementations of Queries shouldn't need to lock the resources
// such as LLVMContext, each argument (function) has a non-shared LLVMContext
// Plus, if Queries contain states necessary locking scheme should be provided.
namespace llvm {
namespace orc {

```
- **EN**: Introduces declarations for `llvm`, `orc`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `orc` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 47-60
```cpp
// Collect direct calls only
void SpeculateQuery::findCalles(const BasicBlock *BB,
                                DenseSet<StringRef> &CallesNames) {
  assert(BB != nullptr && "Traversing Null BB to find calls?");

  auto getCalledFunction = [&CallesNames](const CallBase *Call) {
    auto CalledValue = Call->getCalledOperand()->stripPointerCasts();
    if (auto DirectCall = dyn_cast<Function>(CalledValue))
      CallesNames.insert(DirectCall->getName());
  };
  for (auto &I : *BB)
    if (auto CI = dyn_cast<CallInst>(&I))
      getCalledFunction(CI);

```
- **EN**: Implements logic around `findCalles`, `assert`, `getCalledOperand`, `insert`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `findCalles`, `assert`, `getCalledOperand`, `insert`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 61-70
```cpp
  if (auto II = dyn_cast<InvokeInst>(BB->getTerminator()))
    getCalledFunction(II);
}

bool SpeculateQuery::isStraightLine(const Function &F) {
  return llvm::all_of(F, [](const BasicBlock &BB) {
    return BB.getSingleSuccessor() != nullptr;
  });
}

```
- **EN**: Implements logic around `getCalledFunction`, `isStraightLine`, `all_of`, `getSingleSuccessor`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `getCalledFunction`, `isStraightLine`, `all_of`, `getSingleSuccessor` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 71-83
```cpp
// BlockFreqQuery Implementations

size_t BlockFreqQuery::numBBToGet(size_t numBB) {
  // small CFG
  if (numBB < 4)
    return numBB;
  // mid-size CFG
  else if (numBB < 20)
    return (numBB / 2);
  else
    return (numBB / 2) + (numBB / 4);
}

```
- **EN**: Implements logic around `numBBToGet`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `numBBToGet` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 84-92
```cpp
BlockFreqQuery::ResultTy BlockFreqQuery::operator()(Function &F) {
  DenseMap<StringRef, DenseSet<StringRef>> CallerAndCalles;
  DenseSet<StringRef> Calles;
  SmallVector<std::pair<const BasicBlock *, uint64_t>, 8> BBFreqs;

  PassBuilder PB;
  FunctionAnalysisManager FAM;
  PB.registerFunctionAnalyses(FAM);

```
- **EN**: Implements logic around `operator`, `registerFunctionAnalyses`; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `operator`, `registerFunctionAnalyses` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 93-102
```cpp
  auto IBBs = findBBwithCalls(F);

  if (IBBs.empty())
    return std::nullopt;

  auto &BFI = FAM.getResult<BlockFrequencyAnalysis>(F);

  for (const auto I : IBBs)
    BBFreqs.push_back({I, BFI.getBlockFreq(I).getFrequency()});

```
- **EN**: Implements logic around `findBBwithCalls`, `getResult<BlockFrequencyAnalysis>`, `push_back`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `findBBwithCalls`, `getResult<BlockFrequencyAnalysis>`, `push_back` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 103-112
```cpp
  assert(IBBs.size() == BBFreqs.size() && "BB Count Mismatch");

  llvm::sort(BBFreqs, [](decltype(BBFreqs)::const_reference BBF,
                         decltype(BBFreqs)::const_reference BBS) {
    return BBF.second > BBS.second ? true : false;
  });

  // ignoring number of direct calls in a BB
  auto Topk = numBBToGet(BBFreqs.size());

```
- **EN**: Implements logic around `assert`, `sort`, `decltype`, `numBBToGet`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `assert`, `sort`, `decltype`, `numBBToGet` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 113-122
```cpp
  for (size_t i = 0; i < Topk; i++)
    findCalles(BBFreqs[i].first, Calles);

  assert(!Calles.empty() && "Running Analysis on Function with no calls?");

  CallerAndCalles.insert({F.getName(), std::move(Calles)});

  return CallerAndCalles;
}

```
- **EN**: Implements logic around `findCalles`, `assert`, `insert`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `findCalles`, `assert`, `insert` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 123-134
```cpp
// SequenceBBQuery Implementation
std::size_t SequenceBBQuery::getHottestBlocks(std::size_t TotalBlocks) {
  if (TotalBlocks == 1)
    return TotalBlocks;
  return TotalBlocks / 2;
}

// FIXME : find good implementation.
SequenceBBQuery::BlockListTy
SequenceBBQuery::rearrangeBB(const Function &F, const BlockListTy &BBList) {
  BlockListTy RearrangedBBSet;

```
- **EN**: Implements logic around `getHottestBlocks`, `rearrangeBB`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `getHottestBlocks`, `rearrangeBB` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 135-143
```cpp
  for (auto &Block : F)
    if (llvm::is_contained(BBList, &Block))
      RearrangedBBSet.push_back(&Block);

  assert(RearrangedBBSet.size() == BBList.size() &&
         "BasicBlock missing while rearranging?");
  return RearrangedBBSet;
}

```
- **EN**: Implements logic around `push_back`, `assert`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `push_back`, `assert` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 144-161
```cpp
void SequenceBBQuery::traverseToEntryBlock(const BasicBlock *AtBB,
                                           const BlockListTy &CallerBlocks,
                                           const BackEdgesInfoTy &BackEdgesInfo,
                                           const BranchProbabilityInfo *BPI,
                                           VisitedBlocksInfoTy &VisitedBlocks) {
  auto Itr = VisitedBlocks.find(AtBB);
  if (Itr != VisitedBlocks.end()) { // already visited.
    if (!Itr->second.Upward)
      return;
    Itr->second.Upward = false;
  } else {
    // Create hint for newly discoverd blocks.
    WalkDirection BlockHint;
    BlockHint.Upward = false;
    // FIXME: Expensive Check
    if (llvm::is_contained(CallerBlocks, AtBB))
      BlockHint.CallerBlock = true;
    VisitedBlocks.insert(std::make_pair(AtBB, BlockHint));
```
- **EN**: Implements logic around `traverseToEntryBlock`, `find`, `insert`; this block manipulates JITLink graph structures or link-time passes; executes or prepares runtime behavior for LLVM IR/JIT code.
- **CN**: 围绕 `traverseToEntryBlock`, `find`, `insert` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，执行或准备 LLVM IR/JIT 代码的运行时行为。

### Lines 162-170
```cpp
  }

  const_pred_iterator PIt = pred_begin(AtBB), EIt = pred_end(AtBB);
  // Move this check to top, when we have code setup to launch speculative
  // compiles for function in entry BB, this triggers the speculative compiles
  // before running the program.
  if (PIt == EIt) // No Preds.
    return;

```
- **EN**: Implements logic around `pred_begin`.
- **CN**: 围绕 `pred_begin` 实现具体逻辑。

### Lines 171-186
```cpp
  DenseSet<const BasicBlock *> PredSkipNodes;

  // Since we are checking for predecessor's backedges, this Block
  // occurs in second position.
  for (auto &I : BackEdgesInfo)
    if (I.second == AtBB)
      PredSkipNodes.insert(I.first);

  // Skip predecessors which source of back-edges.
  for (; PIt != EIt; ++PIt)
    // checking EdgeHotness is cheaper
    if (BPI->isEdgeHot(*PIt, AtBB) && !PredSkipNodes.count(*PIt))
      traverseToEntryBlock(*PIt, CallerBlocks, BackEdgesInfo, BPI,
                           VisitedBlocks);
}

```
- **EN**: Implements logic around `insert`, `traverseToEntryBlock`; this block manipulates JITLink graph structures or link-time passes; executes or prepares runtime behavior for LLVM IR/JIT code.
- **CN**: 围绕 `insert`, `traverseToEntryBlock` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，执行或准备 LLVM IR/JIT 代码的运行时行为。

### Lines 187-204
```cpp
void SequenceBBQuery::traverseToExitBlock(const BasicBlock *AtBB,
                                          const BlockListTy &CallerBlocks,
                                          const BackEdgesInfoTy &BackEdgesInfo,
                                          const BranchProbabilityInfo *BPI,
                                          VisitedBlocksInfoTy &VisitedBlocks) {
  auto Itr = VisitedBlocks.find(AtBB);
  if (Itr != VisitedBlocks.end()) { // already visited.
    if (!Itr->second.Downward)
      return;
    Itr->second.Downward = false;
  } else {
    // Create hint for newly discoverd blocks.
    WalkDirection BlockHint;
    BlockHint.Downward = false;
    // FIXME: Expensive Check
    if (llvm::is_contained(CallerBlocks, AtBB))
      BlockHint.CallerBlock = true;
    VisitedBlocks.insert(std::make_pair(AtBB, BlockHint));
```
- **EN**: Implements logic around `traverseToExitBlock`, `find`, `insert`; this block manipulates JITLink graph structures or link-time passes; executes or prepares runtime behavior for LLVM IR/JIT code.
- **CN**: 围绕 `traverseToExitBlock`, `find`, `insert` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，执行或准备 LLVM IR/JIT 代码的运行时行为。

### Lines 205-213
```cpp
  }

  const_succ_iterator PIt = succ_begin(AtBB), EIt = succ_end(AtBB);
  if (PIt == EIt) // No succs.
    return;

  // If there are hot edges, then compute SuccSkipNodes.
  DenseSet<const BasicBlock *> SuccSkipNodes;

```
- **EN**: Implements logic around `succ_begin`; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `succ_begin` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 214-225
```cpp
  // Since we are checking for successor's backedges, this Block
  // occurs in first position.
  for (auto &I : BackEdgesInfo)
    if (I.first == AtBB)
      SuccSkipNodes.insert(I.second);

  for (; PIt != EIt; ++PIt)
    if (BPI->isEdgeHot(AtBB, *PIt) && !SuccSkipNodes.count(*PIt))
      traverseToExitBlock(*PIt, CallerBlocks, BackEdgesInfo, BPI,
                          VisitedBlocks);
}

```
- **EN**: Implements logic around `insert`, `traverseToExitBlock`; this block manipulates JITLink graph structures or link-time passes; executes or prepares runtime behavior for LLVM IR/JIT code.
- **CN**: 围绕 `insert`, `traverseToExitBlock` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，执行或准备 LLVM IR/JIT 代码的运行时行为。

### Lines 226-235
```cpp
// Get Block frequencies for blocks and take most frequently executed block,
// walk towards the entry block from those blocks and discover the basic blocks
// with call.
SequenceBBQuery::BlockListTy
SequenceBBQuery::queryCFG(Function &F, const BlockListTy &CallerBlocks) {

  BlockFreqInfoTy BBFreqs;
  VisitedBlocksInfoTy VisitedBlocks;
  BackEdgesInfoTy BackEdgesInfo;

```
- **EN**: Implements logic around `queryCFG`; this block manipulates JITLink graph structures or link-time passes; executes or prepares runtime behavior for LLVM IR/JIT code.
- **CN**: 围绕 `queryCFG` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，执行或准备 LLVM IR/JIT 代码的运行时行为。

### Lines 236-246
```cpp
  PassBuilder PB;
  FunctionAnalysisManager FAM;
  PB.registerFunctionAnalyses(FAM);

  auto &BFI = FAM.getResult<BlockFrequencyAnalysis>(F);

  llvm::FindFunctionBackedges(F, BackEdgesInfo);

  for (const auto I : CallerBlocks)
    BBFreqs.push_back({I, BFI.getBlockFreq(I).getFrequency()});

```
- **EN**: Implements logic around `registerFunctionAnalyses`, `getResult<BlockFrequencyAnalysis>`, `FindFunctionBackedges`, `push_back`; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `registerFunctionAnalyses`, `getResult<BlockFrequencyAnalysis>`, `FindFunctionBackedges`, `push_back` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 247-255
```cpp
  llvm::sort(BBFreqs, [](decltype(BBFreqs)::const_reference Bbf,
                         decltype(BBFreqs)::const_reference Bbs) {
    return Bbf.second > Bbs.second;
  });

  ArrayRef<std::pair<const BasicBlock *, uint64_t>> HotBlocksRef(BBFreqs);
  HotBlocksRef =
      HotBlocksRef.drop_back(BBFreqs.size() - getHottestBlocks(BBFreqs.size()));

```
- **EN**: Implements logic around `sort`, `decltype`, `HotBlocksRef`, `drop_back`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `sort`, `decltype`, `HotBlocksRef`, `drop_back` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 256-269
```cpp
  BranchProbabilityInfo *BPI =
      FAM.getCachedResult<BranchProbabilityAnalysis>(F);

  // visit NHotBlocks,
  // traverse upwards to entry
  // traverse downwards to end.

  for (auto I : HotBlocksRef) {
    traverseToEntryBlock(I.first, CallerBlocks, BackEdgesInfo, BPI,
                         VisitedBlocks);
    traverseToExitBlock(I.first, CallerBlocks, BackEdgesInfo, BPI,
                        VisitedBlocks);
  }

```
- **EN**: Implements logic around `getCachedResult<BranchProbabilityAnalysis>`, `traverseToEntryBlock`, `traverseToExitBlock`; this block manipulates JITLink graph structures or link-time passes; executes or prepares runtime behavior for LLVM IR/JIT code.
- **CN**: 围绕 `getCachedResult<BranchProbabilityAnalysis>`, `traverseToEntryBlock`, `traverseToExitBlock` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，执行或准备 LLVM IR/JIT 代码的运行时行为。

### Lines 270-284
```cpp
  BlockListTy MinCallerBlocks;
  for (auto &I : VisitedBlocks)
    if (I.second.CallerBlock)
      MinCallerBlocks.push_back(std::move(I.first));

  return rearrangeBB(F, MinCallerBlocks);
}

SpeculateQuery::ResultTy SequenceBBQuery::operator()(Function &F) {
  // reduce the number of lists!
  DenseMap<StringRef, DenseSet<StringRef>> CallerAndCalles;
  DenseSet<StringRef> Calles;
  BlockListTy SequencedBlocks;
  BlockListTy CallerBlocks;

```
- **EN**: Implements logic around `push_back`, `rearrangeBB`, `operator`; this block manipulates JITLink graph structures or link-time passes; executes or prepares runtime behavior for LLVM IR/JIT code; returns subsystem-specific computed results.
- **CN**: 围绕 `push_back`, `rearrangeBB`, `operator` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，执行或准备 LLVM IR/JIT 代码的运行时行为，返回子系统相关的计算结果。

### Lines 285-293
```cpp
  CallerBlocks = findBBwithCalls(F);
  if (CallerBlocks.empty())
    return std::nullopt;

  if (isStraightLine(F))
    SequencedBlocks = rearrangeBB(F, CallerBlocks);
  else
    SequencedBlocks = queryCFG(F, CallerBlocks);

```
- **EN**: Implements logic around `findBBwithCalls`, `rearrangeBB`, `queryCFG`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `findBBwithCalls`, `rearrangeBB`, `queryCFG` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 294-302
```cpp
  for (const auto *BB : SequencedBlocks)
    findCalles(BB, Calles);

  CallerAndCalles.insert({F.getName(), std::move(Calles)});
  return CallerAndCalles;
}

} // namespace orc
} // namespace llvm
```
- **EN**: Introduces declarations for `orc`, `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `orc`, `llvm` 等声明，定义本文件后续使用的数据结构或接口。

## Key Concepts / 关键概念

- **ORC JIT / ORC JIT**:
  - **EN**: Coordinates JITDylibs, symbol materialization, execution sessions, and asynchronous compilation flows
  - **CN**: 协调 JITDylib、符号物化、ExecutionSession 与异步编译流程

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/ExecutionEngine/Orc/SpeculateAnalyses.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVector.h`, `llvm/Analysis/BlockFrequencyInfo.h`, `llvm/Analysis/BranchProbabilityInfo.h`, `llvm/Analysis/CFG.h`, `llvm/IR/PassManager.h`, `llvm/Passes/PassBuilder.h`, `llvm/Support/ErrorHandling.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine, IR, Support
