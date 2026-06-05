# Utils.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `polly/lib/CodeGen/Utils.cpp` | `polly/lib/CodeGen/Utils.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Implements Polly code generation, AST lowering, and runtime emission logic. The leading comment describes it as: Utility functions for the code generation. | 实现 Polly 的代码生成、AST 降级与运行时生成逻辑。 文件开头注释将其概括为：Utility functions for the code generation。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

````cpp
//===--- Utils.cpp - Utility functions for the code generation --*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains utility functions for the code generation.
//
//===----------------------------------------------------------------------===//

````
- **EN**: This block records the standard LLVM file banner and license metadata; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 给出 LLVM 标准文件头与许可证元数据; 保留用于解释意图、用法或算法背景的注释.

### Lines 13-28

````cpp
#include "polly/CodeGen/Utils.h"
#include "polly/CodeGen/IRBuilder.h"
#include "polly/ScopInfo.h"
#include "llvm/Analysis/LoopInfo.h"
#include "llvm/Analysis/RegionInfo.h"
#include "llvm/Transforms/Utils/BasicBlockUtils.h"

using namespace llvm;

// Alternative to llvm::SplitCriticalEdge.
//
// Creates a new block which branches to Succ. The edge to split is redirected
// to the new block.
//
// The issue with llvm::SplitCriticalEdge is that it does nothing if the edge is
// not critical.
````
- **EN**: This block imports Polly, LLVM-family headers needed by the surrounding code; touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 引入周边逻辑所需的 Polly、LLVM-family 头文件; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 29-44

````cpp
// The issue with llvm::SplitEdge is that it does not always create the middle
// block, but reuses Prev/Succ if it can. We always want a new middle block.
static BasicBlock *splitEdge(BasicBlock *Prev, BasicBlock *Succ,
                             const char *Suffix, DominatorTree *DT,
                             LoopInfo *LI, RegionInfo *RI) {
  assert(Prev && Succ);

  // Before:
  //   \    /     /   //
  //    Prev     /    //
  //     |  \___/     //
  //     |   ___      //
  //     |  /   \     //
  //    Succ     \    //
  //   /    \     \   //

````
- **EN**: This block declares or defines routines around `splitEdge`; adds defensive checks for invariants or impossible states; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `splitEdge` 相关的例程; 加入用于保护不变量或异常状态的断言/检查; 保留用于解释意图、用法或算法背景的注释.

### Lines 45-62

````cpp
  // The algorithm to update DominatorTree and LoopInfo of
  // llvm::SplitCriticalEdge is more efficient than
  // llvm::SplitBlockPredecessors, which is more general. In the future we might
  // either modify llvm::SplitCriticalEdge to allow skipping the critical edge
  // check; or Copy&Paste it here.
  BasicBlock *MiddleBlock = SplitBlockPredecessors(
      Succ, ArrayRef<BasicBlock *>(Prev), Suffix, DT, LI);

  if (RI) {
    Region *PrevRegion = RI->getRegionFor(Prev);
    Region *SuccRegion = RI->getRegionFor(Succ);
    if (PrevRegion->contains(MiddleBlock)) {
      RI->setRegionFor(MiddleBlock, PrevRegion);
    } else {
      RI->setRegionFor(MiddleBlock, SuccRegion);
    }
  }

````
- **EN**: This block declares or defines routines around `SplitBlockPredecessors`, `getRegionFor`, `setRegionFor`; contains control flow with 2 conditional check(s); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `SplitBlockPredecessors`, `getRegionFor`, `setRegionFor` 相关的例程; 包含控制流结构：2 处条件判断; 保留用于解释意图、用法或算法背景的注释.

### Lines 63-76

````cpp
  // After:
  //   \    /     /   //
  //    Prev     /    //
  //     |  \___/     //
  //     |            //
  // MiddleBlock      //
  //     |   ___      //
  //     |  /   \     //
  //    Succ     \    //
  //   /    \     \   //

  return MiddleBlock;
}

````
- **EN**: This block emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 77-94

````cpp
std::pair<polly::BBPair, CondBrInst *>
polly::executeScopConditionally(Scop &S, Value *RTC, DominatorTree &DT,
                                RegionInfo &RI, LoopInfo &LI) {
  Region &R = S.getRegion();
  PollyIRBuilder Builder(S.getEntry());

  // Before:
  //
  //      \   /      //
  //    EnteringBB   //
  //   _____|_____   //
  //  /  EntryBB  \  //
  //  |  (region) |  //
  //  \_ExitingBB_/  //
  //        |        //
  //      ExitBB     //
  //      /    \     //

````
- **EN**: This block declares or defines routines around `executeScopConditionally`, `getRegion`, `Builder`; touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `executeScopConditionally`, `getRegion`, `Builder` 相关的例程; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 95-114

````cpp
  // Create a fork block.
  BasicBlock *EnteringBB = S.getEnteringBlock();
  BasicBlock *EntryBB = S.getEntry();
  assert(EnteringBB && "Must be a simple region");
  BasicBlock *SplitBlock =
      splitEdge(EnteringBB, EntryBB, ".split_new_and_old", &DT, &LI, &RI);
  SplitBlock->setName("polly.split_new_and_old");

  // If EntryBB is the exit block of the region that includes Prev, exclude
  // SplitBlock from that region by making it itself the exit block. This is
  // trivially possible because there is just one edge to EnteringBB.
  // This is necessary because we will add an outgoing edge from SplitBlock,
  // which would violate the single exit block requirement of PrevRegion.
  Region *PrevRegion = RI.getRegionFor(EnteringBB);
  while (PrevRegion->getExit() == EntryBB) {
    PrevRegion->replaceExit(SplitBlock);
    PrevRegion = PrevRegion->getParent();
  }
  RI.setRegionFor(SplitBlock, PrevRegion);

````
- **EN**: This block declares or defines routines around `getEnteringBlock`, `getEntry`, `splitEdge`, `setName` (+4 more); contains control flow with 1 loop construct(s); adds defensive checks for invariants or impossible states; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `getEnteringBlock`, `getEntry`, `splitEdge`, `setName` (+4 more) 相关的例程; 包含控制流结构：1 处循环; 加入用于保护不变量或异常状态的断言/检查; 保留用于解释意图、用法或算法背景的注释.

### Lines 115-126

````cpp
  // Create a join block
  BasicBlock *ExitingBB = S.getExitingBlock();
  BasicBlock *ExitBB = S.getExit();
  assert(ExitingBB && "Must be a simple region");
  BasicBlock *MergeBlock =
      splitEdge(ExitingBB, ExitBB, ".merge_new_and_old", &DT, &LI, &RI);
  MergeBlock->setName("polly.merge_new_and_old");

  // Exclude the join block from the region.
  R.replaceExitRecursive(MergeBlock);
  RI.setRegionFor(MergeBlock, R.getParent());

````
- **EN**: This block declares or defines routines around `getExitingBlock`, `getExit`, `splitEdge`, `setName` (+2 more); adds defensive checks for invariants or impossible states; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `getExitingBlock`, `getExit`, `splitEdge`, `setName` (+2 more) 相关的例程; 加入用于保护不变量或异常状态的断言/检查; 保留用于解释意图、用法或算法背景的注释.

### Lines 127-140

````cpp
  //      \   /      //
  //    EnteringBB   //
  //        |        //
  //    SplitBlock   //
  //   _____|_____   //
  //  /  EntryBB  \  //
  //  |  (region) |  //
  //  \_ExitingBB_/  //
  //        |        //
  //    MergeBlock   //
  //        |        //
  //      ExitBB     //
  //      /    \     //

````
- **EN**: This block preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 保留用于解释意图、用法或算法背景的注释.

### Lines 141-159

````cpp
  // Create the start and exiting block.
  Function *F = SplitBlock->getParent();
  BasicBlock *StartBlock =
      BasicBlock::Create(F->getContext(), "polly.start", F);
  BasicBlock *ExitingBlock =
      BasicBlock::Create(F->getContext(), "polly.exiting", F);
  SplitBlock->getTerminator()->eraseFromParent();
  Builder.SetInsertPoint(SplitBlock);
  CondBrInst *CondBr = Builder.CreateCondBr(RTC, StartBlock, S.getEntry());

  if (Loop *L = LI.getLoopFor(SplitBlock)) {
    L->addBasicBlockToLoop(StartBlock, LI);
    L->addBasicBlockToLoop(ExitingBlock, LI);
  }
  DT.addNewBlock(StartBlock, SplitBlock);
  DT.addNewBlock(ExitingBlock, StartBlock);
  RI.setRegionFor(StartBlock, RI.getRegionFor(SplitBlock));
  RI.setRegionFor(ExitingBlock, RI.getRegionFor(SplitBlock));

````
- **EN**: This block declares or defines routines around `getParent`, `Create`, `getTerminator`, `SetInsertPoint` (+4 more); contains control flow with 1 conditional check(s); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `getParent`, `Create`, `getTerminator`, `SetInsertPoint` (+4 more) 相关的例程; 包含控制流结构：1 处条件判断; 保留用于解释意图、用法或算法背景的注释.

### Lines 160-173

````cpp
  //      \   /                    //
  //    EnteringBB                 //
  //        |                      //
  //    SplitBlock---------\       //
  //   _____|_____         |       //
  //  /  EntryBB  \    StartBlock  //
  //  |  (region) |        |       //
  //  \_ExitingBB_/   ExitingBlock //
  //        |                      //
  //    MergeBlock                 //
  //        |                      //
  //      ExitBB                   //
  //      /    \                   //

````
- **EN**: This block preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 保留用于解释意图、用法或算法背景的注释.

### Lines 174-189

````cpp
  // Connect start block to exiting block.
  Builder.SetInsertPoint(StartBlock);
  Builder.CreateBr(ExitingBlock);
  DT.changeImmediateDominator(ExitingBlock, StartBlock);

  // Connect exiting block to join block.
  Builder.SetInsertPoint(ExitingBlock);
  Builder.CreateBr(MergeBlock);
  DT.changeImmediateDominator(MergeBlock, SplitBlock);

  //      \   /                    //
  //    EnteringBB                 //
  //        |                      //
  //    SplitBlock---------\       //
  //   _____|_____         |       //
  //  /  EntryBB  \    StartBlock  //
````
- **EN**: This block declares or defines routines around `SetInsertPoint`, `CreateBr`, `changeImmediateDominator`; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `SetInsertPoint`, `CreateBr`, `changeImmediateDominator` 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 190-201

````cpp
  //  |  (region) |        |       //
  //  \_ExitingBB_/   ExitingBlock //
  //        |              |       //
  //    MergeBlock---------/       //
  //        |                      //
  //      ExitBB                   //
  //      /    \                   //
  //

  // Split the edge between SplitBlock and EntryBB, to avoid a critical edge.
  splitEdge(SplitBlock, EntryBB, ".pre_entry_bb", &DT, &LI, &RI);

````
- **EN**: This block declares or defines routines around `splitEdge`; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `splitEdge` 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 202-217

````cpp
  //      \   /                    //
  //    EnteringBB                 //
  //        |                      //
  //    SplitBlock---------\       //
  //        |              |       //
  //    PreEntryBB         |       //
  //   _____|_____         |       //
  //  /  EntryBB  \    StartBlock  //
  //  |  (region) |        |       //
  //  \_ExitingBB_/   ExitingBlock //
  //        |              |       //
  //    MergeBlock---------/       //
  //        |                      //
  //      ExitBB                   //
  //      /    \                   //

````
- **EN**: This block preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 保留用于解释意图、用法或算法背景的注释.

### Lines 218-219

````cpp
  return std::make_pair(std::make_pair(StartBlock, ExitingBlock), CondBr);
}
````
- **EN**: This block emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 包含返回路径，用于向调用者交回结果或状态码.

## Key Concepts / 关键概念

- **Code generation**
  - **CN**: 代码生成
- **Loop transformation**
  - **CN**: 循环变换
- **SCoP modeling**
  - **CN**: SCoP 建模
- **Debug and diagnostics**
  - **CN**: 调试与诊断

## Dependencies / 依赖关系

- **Polly headers**: `polly/CodeGen/Utils.h`, `polly/CodeGen/IRBuilder.h`, `polly/ScopInfo.h` — Direct Polly APIs, data structures, or pass interfaces used here.
  **Polly headers（CN）**：`polly/CodeGen/Utils.h`, `polly/CodeGen/IRBuilder.h`, `polly/ScopInfo.h` —— 此处直接使用的 Polly API、数据结构或 Pass 接口。
- **LLVM-family headers**: `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/RegionInfo.h`, `llvm/Transforms/Utils/BasicBlockUtils.h` — LLVM/Clang/MLIR infrastructure for IR, passes, utilities, or diagnostics.
  **LLVM-family headers（CN）**：`llvm/Analysis/LoopInfo.h`, `llvm/Analysis/RegionInfo.h`, `llvm/Transforms/Utils/BasicBlockUtils.h` —— 用于 IR、Pass、工具或诊断的 LLVM/Clang/MLIR 基础设施。
