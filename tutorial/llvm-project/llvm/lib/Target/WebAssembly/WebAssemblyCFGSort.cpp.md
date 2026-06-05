# WebAssemblyCFGSort.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyCFGSort.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file implements a CFG sorting pass.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyCFGSort.cpp`，主要负责 WebAssembly 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- WebAssemblyCFGSort.cpp - CFG Sorting ------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-18

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// This file implements a CFG sorting pass.
///
/// This pass reorders the blocks in a function to put them into topological
/// order, ignoring loop backedges, and without any loop or exception being
/// interrupted by a block not dominated by the its header, with special care
/// to keep the order as similar as possible to the original order.
///
////===----------------------------------------------------------------------===//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 19-25

```cpp
#include "WebAssembly.h"
#include "WebAssemblyExceptionInfo.h"
#include "WebAssemblySortRegion.h"
#include "WebAssemblyUtilities.h"
#include "llvm/ADT/PriorityQueue.h"
#include "llvm/CodeGen/MachineDominators.h"
#include "llvm/CodeGen/MachineFunction.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 26-34

```cpp
#include "llvm/CodeGen/MachineLoopInfo.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/Passes.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"
using namespace llvm;
using WebAssembly::SortRegion;
using WebAssembly::SortRegionInfo;
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Register assignment and register-class constraints matter here.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 35-44

```cpp
#define DEBUG_TYPE "wasm-cfg-sort"

// Option to disable EH pad first sorting. Only for testing unwind destination
// mismatches in CFGStackify.
static cl::opt<bool> WasmDisableEHPadSort(
    "wasm-disable-ehpad-sort", cl::ReallyHidden,
    cl::desc(
        "WebAssembly: Disable EH pad-first sort order. Testing purpose only."),
    cl::init(false));
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 该区间与栈帧布局或栈访问相关。

### Lines 45-77

```cpp
namespace {

class WebAssemblyCFGSort final : public MachineFunctionPass {
  StringRef getPassName() const override { return "WebAssembly CFG Sort"; }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    AU.addRequired<MachineDominatorTreeWrapperPass>();
    AU.addPreserved<MachineDominatorTreeWrapperPass>();
    AU.addRequired<MachineLoopInfoWrapperPass>();
    AU.addPreserved<MachineLoopInfoWrapperPass>();
    AU.addRequired<WebAssemblyExceptionInfo>();
    AU.addPreserved<WebAssemblyExceptionInfo>();
    MachineFunctionPass::getAnalysisUsage(AU);
  }

  bool runOnMachineFunction(MachineFunction &MF) override;

public:
  static char ID; // Pass identification, replacement for typeid
  WebAssemblyCFGSort() : MachineFunctionPass(ID) {}
};
} // end anonymous namespace

char WebAssemblyCFGSort::ID = 0;
INITIALIZE_PASS(WebAssemblyCFGSort, DEBUG_TYPE,
                "Reorders blocks in topological order", false, false)

FunctionPass *llvm::createWebAssemblyCFGSort() {
  return new WebAssemblyCFGSort();
}

static void maybeUpdateTerminator(MachineBasicBlock *MBB) {
```
- **EN**: Declares a backend-facing type `WebAssemblyCFGSort`, `getPassName`, `getAnalysisUsage` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `WebAssemblyCFGSort`, `getPassName`, `getAnalysisUsage`，并勾勒出周边代码会依赖的接口或状态。

### Lines 78-84

```cpp
#ifndef NDEBUG
  bool AnyBarrier = false;
#endif
  bool AllAnalyzable = true;
  for (const MachineInstr &Term : MBB->terminators()) {
#ifndef NDEBUG
    AnyBarrier |= Term.isBarrier();
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 85-101

```cpp
#endif
    AllAnalyzable &= Term.isBranch() && !Term.isIndirectBranch();
  }
  assert((AnyBarrier || AllAnalyzable) &&
         "analyzeBranch needs to analyze any block with a fallthrough");

  // Find the layout successor from the original block order.
  MachineFunction *MF = MBB->getParent();
  MachineBasicBlock *OriginalSuccessor =
      unsigned(MBB->getNumber() + 1) < MF->getNumBlockIDs()
          ? MF->getBlockNumbered(MBB->getNumber() + 1)
          : nullptr;

  if (AllAnalyzable)
    MBB->updateTerminator(OriginalSuccessor);
}
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Notable symbols in this range include `isBranch`, `isIndirectBranch`, `getParent`.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 该区间中较显眼的符号包括 `isBranch`, `isIndirectBranch`, `getParent`。

### Lines 102-136

```cpp
namespace {
// EH pads are selected first regardless of the block comparison order.
// When only one of the BBs is an EH pad, we give a higher priority to it, to
// prevent common mismatches between possibly throwing calls and ehpads they
// unwind to, as in the example below:
//
// bb0:
//   call @foo      // If this throws, unwind to bb2
// bb1:
//   call @bar      // If this throws, unwind to bb3
// bb2 (ehpad):
//   handler_bb2
// bb3 (ehpad):
//   handler_bb3
// continuing code
//
// Because this pass tries to preserve the original BB order, this order will
// not change. But this will result in this try-catch structure in CFGStackify,
// resulting in a mismatch:
// try
//   try
//     call @foo
//     call @bar    // This should unwind to bb3, not bb2!
//   catch
//     handler_bb2
//   end
// catch
//   handler_bb3
// end
// continuing code
//
// If we give a higher priority to an EH pad whenever it is ready in this
// example, when both bb1 and bb2 are ready, we would pick up bb2 first.

/// Sort blocks by their number.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "EH pads are selected first regardless of the block comparison order.". Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“EH pads are selected first regardless of the block comparison order.”。 该区间与栈帧布局或栈访问相关。

### Lines 137-150

```cpp
struct CompareBlockNumbers {
  bool operator()(const MachineBasicBlock *A,
                  const MachineBasicBlock *B) const {
    if (!WasmDisableEHPadSort) {
      if (A->isEHPad() && !B->isEHPad())
        return false;
      if (!A->isEHPad() && B->isEHPad())
        return true;
    }

    return A->getNumber() > B->getNumber();
  }
};
/// Sort blocks by their number in the opposite order..
```
- **EN**: Declares a backend-facing type `CompareBlockNumbers`, `operator`, `isEHPad` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `CompareBlockNumbers`, `operator`, `isEHPad`，并勾勒出周边代码会依赖的接口或状态。

### Lines 151-165

```cpp
struct CompareBlockNumbersBackwards {
  bool operator()(const MachineBasicBlock *A,
                  const MachineBasicBlock *B) const {
    if (!WasmDisableEHPadSort) {
      if (A->isEHPad() && !B->isEHPad())
        return false;
      if (!A->isEHPad() && B->isEHPad())
        return true;
    }

    return A->getNumber() < B->getNumber();
  }
};
/// Bookkeeping for a region to help ensure that we don't mix blocks not
/// dominated by the its header among its blocks.
```
- **EN**: Declares a backend-facing type `CompareBlockNumbersBackwards`, `operator`, `isEHPad` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `CompareBlockNumbersBackwards`, `operator`, `isEHPad`，并勾勒出周边代码会依赖的接口或状态。

### Lines 166-203

```cpp
struct Entry {
  const SortRegion *TheRegion;
  unsigned NumBlocksLeft;

  /// List of blocks not dominated by Loop's header that are deferred until
  /// after all of Loop's blocks have been seen.
  std::vector<MachineBasicBlock *> Deferred;

  explicit Entry(const SortRegion *R)
      : TheRegion(R), NumBlocksLeft(R->getNumBlocks()) {}
};
} // end anonymous namespace

/// Sort the blocks, taking special care to make sure that regions are not
/// interrupted by blocks not dominated by their header.
/// TODO: There are many opportunities for improving the heuristics here.
/// Explore them.
static void sortBlocks(MachineFunction &MF, const MachineLoopInfo &MLI,
                       const WebAssemblyExceptionInfo &WEI,
                       MachineDominatorTree &MDT) {
  // Remember original layout ordering, so we can update terminators after
  // reordering to point to the original layout successor.
  MF.RenumberBlocks();

  // Prepare for a topological sort: Record the number of predecessors each
  // block has, ignoring loop backedges.
  SmallVector<unsigned, 16> NumPredsLeft(MF.getNumBlockIDs(), 0);
  for (MachineBasicBlock &MBB : MF) {
    unsigned N = MBB.pred_size();
    if (MachineLoop *L = MLI.getLoopFor(&MBB))
      if (L->getHeader() == &MBB)
        for (const MachineBasicBlock *Pred : MBB.predecessors())
          if (L->contains(Pred))
            --N;
    NumPredsLeft[MBB.getNumber()] = N;
  }

  // Topological sort the CFG, with additional constraints:
```
- **EN**: Declares a backend-facing type `Entry`, `TheRegion`, `NumBlocksLeft` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `Entry`, `TheRegion`, `NumBlocksLeft`，并勾勒出周边代码会依赖的接口或状态。

### Lines 204-241

```cpp
  //  - Between a region header and the last block in the region, there can be
  //    no blocks not dominated by its header.
  //  - It's desirable to preserve the original block order when possible.
  // We use two ready lists; Preferred and Ready. Preferred has recently
  // processed successors, to help preserve block sequences from the original
  // order. Ready has the remaining ready blocks. EH blocks are picked first
  // from both queues.
  PriorityQueue<MachineBasicBlock *, std::vector<MachineBasicBlock *>,
                CompareBlockNumbers>
      Preferred;
  PriorityQueue<MachineBasicBlock *, std::vector<MachineBasicBlock *>,
                CompareBlockNumbersBackwards>
      Ready;

  SortRegionInfo SRI(MLI, WEI);
  SmallVector<Entry, 4> Entries;
  for (MachineBasicBlock *MBB = &MF.front();;) {
    const SortRegion *R = SRI.getRegionFor(MBB);
    if (R) {
      // If MBB is a region header, add it to the active region list. We can't
      // put any blocks that it doesn't dominate until we see the end of the
      // region.
      if (R->getHeader() == MBB)
        Entries.push_back(Entry(R));
      // For each active region the block is in, decrement the count. If MBB is
      // the last block in an active region, take it off the list and pick up
      // any blocks deferred because the header didn't dominate them.
      for (Entry &E : Entries)
        if (E.TheRegion->contains(MBB) && --E.NumBlocksLeft == 0)
          for (auto *DeferredBlock : E.Deferred)
            Ready.push(DeferredBlock);
      while (!Entries.empty() && Entries.back().NumBlocksLeft == 0)
        Entries.pop_back();
    }
    // The main topological sort logic.
    for (MachineBasicBlock *Succ : MBB->successors()) {
      // Ignore backedges.
      if (MachineLoop *SuccL = MLI.getLoopFor(Succ))
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "- Between a region header and the last block in the region, there can be". Notable symbols in this range include `SRI`, `front`, `getRegionFor`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“- Between a region header and the last block in the region, there can be”。 该区间中较显眼的符号包括 `SRI`, `front`, `getRegionFor`。

### Lines 242-279

```cpp
        if (SuccL->getHeader() == Succ && SuccL->contains(MBB))
          continue;
      // Decrement the predecessor count. If it's now zero, it's ready.
      if (--NumPredsLeft[Succ->getNumber()] == 0)
        Preferred.push(Succ);
    }
    // Determine the block to follow MBB. First try to find a preferred block,
    // to preserve the original block order when possible.
    MachineBasicBlock *Next = nullptr;
    while (!Preferred.empty()) {
      Next = Preferred.top();
      Preferred.pop();
      // If X isn't dominated by the top active region header, defer it until
      // that region is done.
      if (!Entries.empty() &&
          !MDT.dominates(Entries.back().TheRegion->getHeader(), Next)) {
        Entries.back().Deferred.push_back(Next);
        Next = nullptr;
        continue;
      }
      // If Next was originally ordered before MBB, and it isn't because it was
      // loop-rotated above the header, it's not preferred.
      if (Next->getNumber() < MBB->getNumber() &&
          (WasmDisableEHPadSort || !Next->isEHPad()) &&
          (!R || !R->contains(Next) ||
           R->getHeader()->getNumber() < Next->getNumber())) {
        Ready.push(Next);
        Next = nullptr;
        continue;
      }
      break;
    }
    // If we didn't find a suitable block in the Preferred list, check the
    // general Ready list.
    if (!Next) {
      // If there are no more blocks to process, we're done.
      if (Ready.empty()) {
        maybeUpdateTerminator(MBB);
```
- **EN**: Implements helper routine(s) `getHeader`, `contains`, `getNumber` for this portion of the WebAssembly backend backend implementation logic.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `getHeader`, `contains`, `getNumber`。

### Lines 280-302

```cpp
        break;
      }
      for (;;) {
        Next = Ready.top();
        Ready.pop();
        // If Next isn't dominated by the top active region header, defer it
        // until that region is done.
        if (!Entries.empty() &&
            !MDT.dominates(Entries.back().TheRegion->getHeader(), Next)) {
          Entries.back().Deferred.push_back(Next);
          continue;
        }
        break;
      }
    }
    // Move the next block into place and iterate.
    Next->moveAfter(MBB);
    maybeUpdateTerminator(MBB);
    MBB = Next;
  }
  assert(Entries.empty() && "Active sort region list not finished");
  MF.RenumberBlocks();
```
- **EN**: Implements helper routine(s) `top`, `pop`, `empty` for this portion of the WebAssembly backend backend implementation logic.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `top`, `pop`, `empty`。

### Lines 303-340

```cpp
#ifndef NDEBUG
  for (auto &MBB : MF) {
    assert(MBB.getNumber() >= 0 && "Renumbered blocks should be non-negative.");
    const SortRegion *Region = SRI.getRegionFor(&MBB);

    if (Region && &MBB == Region->getHeader()) {
      // Region header.
      if (Region->isLoop()) {
        // Loop header. The loop predecessor should be sorted above, and the
        // other predecessors should be backedges below.
        for (auto *Pred : MBB.predecessors())
          assert(
              (Pred->getNumber() < MBB.getNumber() || Region->contains(Pred)) &&
              "Loop header predecessors must be loop predecessors or "
              "backedges");
      } else {
        // Exception header. All predecessors should be sorted above.
        for (auto *Pred : MBB.predecessors())
          assert(Pred->getNumber() < MBB.getNumber() &&
                 "Non-loop-header predecessors should be topologically sorted");
      }
    } else {
      // Not a region header. All predecessors should be sorted above.
      for (auto *Pred : MBB.predecessors())
        assert(Pred->getNumber() < MBB.getNumber() &&
               "Non-loop-header predecessors should be topologically sorted");
    }
  }

  SmallSet<const SortRegion *, 8> Regions;
  for (auto &MBB : MF) {
    const SortRegion *Region = SRI.getRegionFor(&MBB);
    if (Region)
      Regions.insert(Region);
  }

  SmallVector<std::pair<int, int>, 8> RegionIntervals(Regions.size(), {-1, -1});
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Notable symbols in this range include `getNumber`, `getRegionFor`, `getHeader`.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 该区间中较显眼的符号包括 `getNumber`, `getRegionFor`, `getHeader`。

### Lines 341-378

```cpp
  unsigned RegionIdx = 0;
  for (auto *Region : Regions) {
    assert(Region->getHeader() != &MF.front() &&
           "The function entry block shouldn't actually be a region header");

    auto *Header = Region->getHeader();
    auto *Bottom = SRI.getBottom(Region);

    assert(Header && "Regions must have a header");
    assert(Bottom && "Regions must have a bottom");

    std::pair<int, int> Interval = {Header->getNumber(), Bottom->getNumber()};
    assert(Interval.first <= Interval.second &&
           "Region bottoms must be sorted after region headers");

    RegionIntervals[RegionIdx++] = Interval;

    for (auto *MBB : Region->blocks()) {
      assert(MBB->getNumber() >= Interval.first &&
             MBB->getNumber() <= Interval.second &&
             "All blocks within a region must have numbers within the region's "
             "interval");
    }
  }

  for (const auto &IntervalA : RegionIntervals) {
    for (const auto &IntervalB : RegionIntervals) {
      auto AContainsB = IntervalA.first <= IntervalB.first &&
                        IntervalA.second >= IntervalB.second;
      auto BContainsA = IntervalB.first <= IntervalA.first &&
                        IntervalB.second >= IntervalA.second;
      auto Disjoint = IntervalA.second < IntervalB.first ||
                      IntervalA.first > IntervalB.second;
      assert((AContainsB || BContainsA || Disjoint) &&
             "Regions must be fully contained within their parents and not "
             "overlap their siblings");
    }
  }
```
- **EN**: Implements helper routine(s) `getHeader`, `front`, `getBottom` for this portion of the WebAssembly backend backend implementation logic.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `getHeader`, `front`, `getBottom`。

### Lines 379-397

```cpp
#endif
}

bool WebAssemblyCFGSort::runOnMachineFunction(MachineFunction &MF) {
  LLVM_DEBUG(dbgs() << "********** CFG Sorting **********\n"
                       "********** Function: "
                    << MF.getName() << '\n');

  const auto &MLI = getAnalysis<MachineLoopInfoWrapperPass>().getLI();
  const auto &WEI = getAnalysis<WebAssemblyExceptionInfo>();
  auto &MDT = getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
  // Liveness is not tracked for VALUE_STACK physreg.
  MF.getRegInfo().invalidateLiveness();

  // Sort the blocks, with contiguous sort regions.
  sortBlocks(MF, MLI, WEI, MDT);

  return true;
}
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Notable symbols in this range include `runOnMachineFunction`, `dbgs`, `getName`.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 该区间中较显眼的符号包括 `runOnMachineFunction`, `dbgs`, `getName`。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Backend implementation logic / 后端实现逻辑
- Register modeling / 寄存器建模
- Stack frame management / 栈帧管理
- Instruction selection or opcode handling / 指令选择或操作码处理
- Object format integration / 目标文件格式集成
- Declarative TableGen records / 声明式 TableGen 记录
- Vector or SIMD behavior / 向量或 SIMD 行为

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `WebAssembly.h`
- `WebAssemblyExceptionInfo.h`
- `WebAssemblySortRegion.h`
- `WebAssemblyUtilities.h`
- `llvm/ADT/PriorityQueue.h`
- `llvm/CodeGen/MachineDominators.h`
- `llvm/CodeGen/MachineFunction.h`
- `llvm/CodeGen/MachineLoopInfo.h`
- `llvm/CodeGen/MachineRegisterInfo.h`
- `llvm/CodeGen/Passes.h`
- `llvm/Support/Debug.h`
- `llvm/Support/raw_ostream.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM Support utilities / LLVM Support 工具层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
