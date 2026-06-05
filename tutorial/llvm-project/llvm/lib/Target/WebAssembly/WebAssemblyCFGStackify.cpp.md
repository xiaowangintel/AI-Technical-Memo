# WebAssemblyCFGStackify.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyCFGStackify.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file implements a CFG stacking pass.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyCFGStackify.cpp`，主要负责 WebAssembly 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- WebAssemblyCFGStackify.cpp - CFG Stackification -------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。 该区间与栈帧布局或栈访问相关。

### Lines 7-21

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// This file implements a CFG stacking pass.
///
/// This pass inserts BLOCK, LOOP, TRY, and TRY_TABLE markers to mark the start
/// of scopes, since scope boundaries serve as the labels for WebAssembly's
/// control transfers.
///
/// This is sufficient to convert arbitrary CFGs into a form that works on
/// WebAssembly, provided that all loops are single-entry.
///
/// In case we use exceptions, this pass also fixes mismatches in unwind
/// destinations created during transforming CFG into wasm structured format.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 22-28

```cpp
//===----------------------------------------------------------------------===//

#include "Utils/WebAssemblyTypeUtilities.h"
#include "WebAssembly.h"
#include "WebAssemblyExceptionInfo.h"
#include "WebAssemblyMachineFunctionInfo.h"
#include "WebAssemblySortRegion.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 29-35

```cpp
#include "WebAssemblySubtarget.h"
#include "WebAssemblyTargetMachine.h"
#include "WebAssemblyUtilities.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/BinaryFormat/Wasm.h"
#include "llvm/CodeGen/MachineDominators.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 36-46

```cpp
#include "llvm/CodeGen/MachineLoopInfo.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/Target/TargetMachine.h"
using namespace llvm;
using WebAssembly::SortRegionInfo;

#define DEBUG_TYPE "wasm-cfg-stackify"

STATISTIC(NumCallUnwindMismatches, "Number of call unwind mismatches found");
STATISTIC(NumCatchUnwindMismatches, "Number of catch unwind mismatches found");
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. The logic interacts with LLVM's MC layer.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 47-84

```cpp
namespace {
class WebAssemblyCFGStackify final : public MachineFunctionPass {
  MachineDominatorTree *MDT;

  StringRef getPassName() const override { return "WebAssembly CFG Stackify"; }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addRequired<MachineDominatorTreeWrapperPass>();
    AU.addRequired<MachineLoopInfoWrapperPass>();
    AU.addRequired<WebAssemblyExceptionInfo>();
    MachineFunctionPass::getAnalysisUsage(AU);
  }

  bool runOnMachineFunction(MachineFunction &MF) override;

  // For each block whose label represents the end of a scope, record the block
  // which holds the beginning of the scope. This will allow us to quickly skip
  // over scoped regions when walking blocks.
  SmallVector<MachineBasicBlock *, 8> ScopeTops;
  void updateScopeTops(MachineBasicBlock *Begin, MachineBasicBlock *End) {
    int BeginNo = Begin->getNumber();
    int EndNo = End->getNumber();
    if (!ScopeTops[EndNo] || ScopeTops[EndNo]->getNumber() > BeginNo)
      ScopeTops[EndNo] = Begin;
  }

  // Placing markers.
  void placeMarkers(MachineFunction &MF);
  void placeBlockMarker(MachineBasicBlock &MBB);
  void placeLoopMarker(MachineBasicBlock &MBB);
  void placeTryMarker(MachineBasicBlock &MBB);
  void placeTryTableMarker(MachineBasicBlock &MBB);

  // Unwind mismatch fixing for exception handling
  // - Common functions
  bool fixCallUnwindMismatches(MachineFunction &MF);
  bool fixCatchUnwindMismatches(MachineFunction &MF);
  void recalculateScopeTops(MachineFunction &MF);
```
- **EN**: Declares a backend-facing type `WebAssemblyCFGStackify`, `getPassName`, `getAnalysisUsage` and outlines the API or state that nearby code will rely on. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里声明面向后端的类型 `WebAssemblyCFGStackify`, `getPassName`, `getAnalysisUsage`，并勾勒出周边代码会依赖的接口或状态。 该区间与栈帧布局或栈访问相关。

### Lines 85-122

```cpp
  // - Legacy EH
  void addNestedTryDelegate(MachineInstr *RangeBegin, MachineInstr *RangeEnd,
                            MachineBasicBlock *UnwindDest);
  void removeUnnecessaryInstrs(MachineFunction &MF);
  // - Standard EH (exnref)
  void addNestedTryTable(MachineInstr *RangeBegin, MachineInstr *RangeEnd,
                         MachineBasicBlock *UnwindDest);
  MachineBasicBlock *getTrampolineBlock(MachineBasicBlock *UnwindDest);

  // Wrap-up
  using EndMarkerInfo =
      std::pair<const MachineBasicBlock *, const MachineInstr *>;
  unsigned getBranchDepth(const SmallVectorImpl<EndMarkerInfo> &Stack,
                          const MachineBasicBlock *MBB);
  unsigned getDelegateDepth(const SmallVectorImpl<EndMarkerInfo> &Stack,
                            const MachineBasicBlock *MBB);
  unsigned getRethrowDepth(const SmallVectorImpl<EndMarkerInfo> &Stack,
                           const MachineBasicBlock *EHPadToRethrow);
  void rewriteDepthImmediates(MachineFunction &MF);
  void fixEndsAtEndOfFunction(MachineFunction &MF);
  void cleanupFunctionData(MachineFunction &MF);

  // For each BLOCK|LOOP|TRY|TRY_TABLE, the corresponding
  // END_(BLOCK|LOOP|TRY|TRY_TABLE) or DELEGATE (in case of TRY).
  DenseMap<const MachineInstr *, MachineInstr *> BeginToEnd;
  // For each END_(BLOCK|LOOP|TRY|TRY_TABLE) or DELEGATE, the corresponding
  // BLOCK|LOOP|TRY|TRY_TABLE.
  DenseMap<const MachineInstr *, MachineInstr *> EndToBegin;
  // <TRY marker, EH pad> map
  DenseMap<const MachineInstr *, MachineBasicBlock *> TryToEHPad;
  // <EH pad, TRY marker> map
  DenseMap<const MachineBasicBlock *, MachineInstr *> EHPadToTry;

  DenseMap<const MachineBasicBlock *, MachineBasicBlock *>
      UnwindDestToTrampoline;

  // We need an appendix block to place 'end_loop' or 'end_try' marker when the
  // loop / exception bottom block is the last block in a function
```
- **EN**: Declares function entry points including `addNestedTryDelegate`, `removeUnnecessaryInstrs`, `EH` that other backend components call later. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `addNestedTryDelegate`, `removeUnnecessaryInstrs`, `EH`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 123-160

```cpp
  MachineBasicBlock *AppendixBB = nullptr;
  MachineBasicBlock *getAppendixBlock(MachineFunction &MF) {
    if (!AppendixBB) {
      AppendixBB = MF.CreateMachineBasicBlock();
      // Give it a fake predecessor so that AsmPrinter prints its label.
      AppendixBB->addSuccessor(AppendixBB);
      // If the caller trampoline BB exists, insert the appendix BB before it.
      // Otherwise insert it at the end of the function.
      if (CallerTrampolineBB)
        MF.insert(CallerTrampolineBB->getIterator(), AppendixBB);
      else
        MF.push_back(AppendixBB);
    }
    return AppendixBB;
  }

  // Create a caller-dedicated trampoline BB to be used for fixing unwind
  // mismatches where the unwind destination is the caller.
  MachineBasicBlock *CallerTrampolineBB = nullptr;
  MachineBasicBlock *getCallerTrampolineBlock(MachineFunction &MF) {
    if (!CallerTrampolineBB) {
      CallerTrampolineBB = MF.CreateMachineBasicBlock();
      MF.push_back(CallerTrampolineBB);
    }
    return CallerTrampolineBB;
  }

  // Before running rewriteDepthImmediates function, 'delegate' has a BB as its
  // destination operand. getFakeCallerBlock() returns a fake BB that will be
  // used for the operand when 'delegate' needs to rethrow to the caller. This
  // will be rewritten as an immediate value that is the number of block depths
  // + 1 in rewriteDepthImmediates, and this fake BB will be removed at the end
  // of the pass.
  MachineBasicBlock *FakeCallerBB = nullptr;
  MachineBasicBlock *getFakeCallerBlock(MachineFunction &MF) {
    if (!FakeCallerBB)
      FakeCallerBB = MF.CreateMachineBasicBlock();
    return FakeCallerBB;
```
- **EN**: Implements helper routine(s) `getAppendixBlock`, `CreateMachineBasicBlock`, `addSuccessor` for this portion of the WebAssembly backend backend implementation logic.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `getAppendixBlock`, `CreateMachineBasicBlock`, `addSuccessor`。

### Lines 161-198

```cpp
  }

  // Helper functions to register / unregister scope information created by
  // marker instructions.
  void registerScope(MachineInstr *Begin, MachineInstr *End);
  void registerTryScope(MachineInstr *Begin, MachineInstr *End,
                        MachineBasicBlock *EHPad);
  void unregisterScope(MachineInstr *Begin);

public:
  static char ID; // Pass identification, replacement for typeid
  WebAssemblyCFGStackify() : MachineFunctionPass(ID) {}
  ~WebAssemblyCFGStackify() override { releaseMemory(); }
  void releaseMemory() override;
};
} // end anonymous namespace

char WebAssemblyCFGStackify::ID = 0;
INITIALIZE_PASS(
    WebAssemblyCFGStackify, DEBUG_TYPE,
    "Insert BLOCK/LOOP/TRY/TRY_TABLE markers for WebAssembly scopes", false,
    false)

FunctionPass *llvm::createWebAssemblyCFGStackify() {
  return new WebAssemblyCFGStackify();
}

/// Test whether Pred has any terminators explicitly branching to MBB, as
/// opposed to falling through. Note that it's possible (eg. in unoptimized
/// code) for a branch instruction to both branch to a block and fallthrough
/// to it, so we check the actual branch operands to see if there are any
/// explicit mentions.
static bool explicitlyBranchesTo(MachineBasicBlock *Pred,
                                 MachineBasicBlock *MBB) {
  for (MachineInstr &MI : Pred->terminators())
    for (MachineOperand &MO : MI.explicit_operands())
      if (MO.isMBB() && MO.getMBB() == MBB)
        return true;
```
- **EN**: Implements helper routine(s) `registerScope`, `registerTryScope`, `unregisterScope` for this portion of the WebAssembly backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `registerScope`, `registerTryScope`, `unregisterScope`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 199-213

```cpp
  return false;
}

// Returns an iterator to the earliest position possible within the MBB,
// satisfying the restrictions given by BeforeSet and AfterSet. BeforeSet
// contains instructions that should go before the marker, and AfterSet contains
// ones that should go after the marker. In this function, AfterSet is only
// used for validation checking.
template <typename Container>
static MachineBasicBlock::iterator
getEarliestInsertPos(MachineBasicBlock *MBB, const Container &BeforeSet,
                     const Container &AfterSet) {
  auto InsertPos = MBB->end();
  while (InsertPos != MBB->begin()) {
    if (BeforeSet.count(&*std::prev(InsertPos))) {
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Returns an iterator to the earliest position possible within the MBB,". Notable symbols in this range include `getEarliestInsertPos`, `end`, `begin`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Returns an iterator to the earliest position possible within the MBB,”。 该区间中较显眼的符号包括 `getEarliestInsertPos`, `end`, `begin`。

### Lines 214-237

```cpp
#ifndef NDEBUG
      // Validation check
      for (auto Pos = InsertPos, E = MBB->begin(); Pos != E; --Pos)
        assert(!AfterSet.count(&*std::prev(Pos)));
#endif
      break;
    }
    --InsertPos;
  }
  return InsertPos;
}

// Returns an iterator to the latest position possible within the MBB,
// satisfying the restrictions given by BeforeSet and AfterSet. BeforeSet
// contains instructions that should go before the marker, and AfterSet contains
// ones that should go after the marker. In this function, BeforeSet is only
// used for validation checking.
template <typename Container>
static MachineBasicBlock::iterator
getLatestInsertPos(MachineBasicBlock *MBB, const Container &BeforeSet,
                   const Container &AfterSet) {
  auto InsertPos = MBB->begin();
  while (InsertPos != MBB->end()) {
    if (AfterSet.count(&*InsertPos)) {
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Notable symbols in this range include `begin`, `count`, `prev`.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 该区间中较显眼的符号包括 `begin`, `count`, `prev`。

### Lines 238-275

```cpp
#ifndef NDEBUG
      // Validation check
      for (auto Pos = InsertPos, E = MBB->end(); Pos != E; ++Pos)
        assert(!BeforeSet.count(&*Pos));
#endif
      break;
    }
    ++InsertPos;
  }
  return InsertPos;
}

void WebAssemblyCFGStackify::registerScope(MachineInstr *Begin,
                                           MachineInstr *End) {
  BeginToEnd[Begin] = End;
  EndToBegin[End] = Begin;
}

// When 'End' is not an 'end_try' but a 'delegate', EHPad is nullptr.
void WebAssemblyCFGStackify::registerTryScope(MachineInstr *Begin,
                                              MachineInstr *End,
                                              MachineBasicBlock *EHPad) {
  registerScope(Begin, End);
  TryToEHPad[Begin] = EHPad;
  EHPadToTry[EHPad] = Begin;
}

void WebAssemblyCFGStackify::unregisterScope(MachineInstr *Begin) {
  assert(BeginToEnd.count(Begin));
  MachineInstr *End = BeginToEnd[Begin];
  assert(EndToBegin.count(End));
  BeginToEnd.erase(Begin);
  EndToBegin.erase(End);
  MachineBasicBlock *EHPad = TryToEHPad.lookup(Begin);
  if (EHPad) {
    assert(EHPadToTry.count(EHPad));
    TryToEHPad.erase(Begin);
    EHPadToTry.erase(EHPad);
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 276-313

```cpp
  }
}

/// Insert a BLOCK marker for branches to MBB (if needed).
// TODO Consider a more generalized way of handling block (and also loop and
// try) signatures when we implement the multi-value proposal later.
void WebAssemblyCFGStackify::placeBlockMarker(MachineBasicBlock &MBB) {
  assert(!MBB.isEHPad());
  MachineFunction &MF = *MBB.getParent();
  const auto &TII = *MF.getSubtarget<WebAssemblySubtarget>().getInstrInfo();
  const auto &MFI = *MF.getInfo<WebAssemblyFunctionInfo>();

  // First compute the nearest common dominator of all forward non-fallthrough
  // predecessors so that we minimize the time that the BLOCK is on the stack,
  // which reduces overall stack height.
  MachineBasicBlock *Header = nullptr;
  bool IsBranchedTo = false;
  int MBBNumber = MBB.getNumber();
  for (MachineBasicBlock *Pred : MBB.predecessors()) {
    if (Pred->getNumber() < MBBNumber) {
      Header = Header ? MDT->findNearestCommonDominator(Header, Pred) : Pred;
      if (explicitlyBranchesTo(Pred, &MBB))
        IsBranchedTo = true;
    }
  }
  if (!Header)
    return;
  if (!IsBranchedTo)
    return;

  assert(&MBB != &MF.front() && "Header blocks shouldn't have predecessors");
  MachineBasicBlock *LayoutPred = MBB.getPrevNode();

  // If the nearest common dominator is inside a more deeply nested context,
  // walk out to the nearest scope which isn't more deeply nested.
  for (MachineFunction::iterator I(LayoutPred), E(Header); I != E; --I) {
    if (MachineBasicBlock *ScopeTop = ScopeTops[I->getNumber()]) {
      if (ScopeTop->getNumber() > Header->getNumber()) {
```
- **EN**: Implements helper routine(s) `MBB`, `block`, `placeBlockMarker` for this portion of the WebAssembly backend backend implementation logic. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `MBB`, `block`, `placeBlockMarker`。 该区间与栈帧布局或栈访问相关。

### Lines 314-337

```cpp
        // Skip over an intervening scope.
        I = std::next(ScopeTop->getIterator());
      } else {
        // We found a scope level at an appropriate depth.
        Header = ScopeTop;
        break;
      }
    }
  }

  // Decide where in MBB to put the BLOCK.

  // Instructions that should go before the BLOCK.
  SmallPtrSet<const MachineInstr *, 4> BeforeSet;
  // Instructions that should go after the BLOCK.
  SmallPtrSet<const MachineInstr *, 4> AfterSet;
  for (const auto &MI : *Header) {
    // If there is a previously placed LOOP marker and the bottom block of the
    // loop is above MBB, it should be after the BLOCK, because the loop is
    // nested in this BLOCK. Otherwise it should be before the BLOCK.
    if (MI.getOpcode() == WebAssembly::LOOP) {
      auto *LoopBottom = BeginToEnd[&MI]->getParent()->getPrevNode();
      if (MBB.getNumber() > LoopBottom->getNumber())
        AfterSet.insert(&MI);
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Skip over an intervening scope.". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Skip over an intervening scope.”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 338-352

```cpp
#ifndef NDEBUG
      else
        BeforeSet.insert(&MI);
#endif
    }

    // If there is a previously placed BLOCK/TRY/TRY_TABLE marker and its
    // corresponding END marker is before the current BLOCK's END marker, that
    // should be placed after this BLOCK. Otherwise it should be placed before
    // this BLOCK marker.
    if (MI.getOpcode() == WebAssembly::BLOCK ||
        MI.getOpcode() == WebAssembly::TRY ||
        MI.getOpcode() == WebAssembly::TRY_TABLE) {
      if (BeginToEnd[&MI]->getParent()->getNumber() <= MBB.getNumber())
        AfterSet.insert(&MI);
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Notable symbols in this range include `insert`, `getOpcode`, `getParent`.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 该区间中较显眼的符号包括 `insert`, `getOpcode`, `getParent`。

### Lines 353-365

```cpp
#ifndef NDEBUG
      else
        BeforeSet.insert(&MI);
#endif
    }

#ifndef NDEBUG
    // All END_(BLOCK|LOOP|TRY|TRY_TABLE) markers should be before the BLOCK.
    if (MI.getOpcode() == WebAssembly::END_BLOCK ||
        MI.getOpcode() == WebAssembly::END_LOOP ||
        MI.getOpcode() == WebAssembly::END_TRY ||
        MI.getOpcode() == WebAssembly::END_TRY_TABLE)
      BeforeSet.insert(&MI);
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Notable symbols in this range include `insert`, `END_`, `getOpcode`.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 该区间中较显眼的符号包括 `insert`, `END_`, `getOpcode`。

### Lines 366-395

```cpp
#endif

    // Terminators should go after the BLOCK.
    if (MI.isTerminator())
      AfterSet.insert(&MI);
  }

  // Local expression tree should go after the BLOCK.
  for (auto I = Header->getFirstTerminator(), E = Header->begin(); I != E;
       --I) {
    if (std::prev(I)->isDebugInstr() || std::prev(I)->isPosition())
      continue;
    if (WebAssembly::isChild(*std::prev(I), MFI))
      AfterSet.insert(&*std::prev(I));
    else
      break;
  }

  // Add the BLOCK.
  WebAssembly::BlockType ReturnType = WebAssembly::BlockType::Void;
  auto InsertPos = getLatestInsertPos(Header, BeforeSet, AfterSet);
  MachineInstr *Begin =
      BuildMI(*Header, InsertPos, Header->findDebugLoc(InsertPos),
              TII.get(WebAssembly::BLOCK))
          .addImm(int64_t(ReturnType));

  // Decide where in MBB to put the END_BLOCK.
  BeforeSet.clear();
  AfterSet.clear();
  for (auto &MI : MBB) {
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 396-414

```cpp
#ifndef NDEBUG
    // END_BLOCK should precede existing LOOP markers.
    if (MI.getOpcode() == WebAssembly::LOOP)
      AfterSet.insert(&MI);
#endif

    // If there is a previously placed END_LOOP marker and the header of the
    // loop is above this block's header, the END_LOOP should be placed after
    // the END_BLOCK, because the loop contains this block. Otherwise the
    // END_LOOP should be placed before the END_BLOCK. The same for END_TRY.
    //
    // Note that while there can be existing END_TRYs, there can't be
    // END_TRY_TABLEs; END_TRYs are placed when its corresponding EH pad is
    // processed, so they are placed below MBB (EH pad) in placeTryMarker. But
    // END_TRY_TABLE is placed like a END_BLOCK, so they can't be here already.
    if (MI.getOpcode() == WebAssembly::END_LOOP ||
        MI.getOpcode() == WebAssembly::END_TRY) {
      if (EndToBegin[&MI]->getParent()->getNumber() >= Header->getNumber())
        BeforeSet.insert(&MI);
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Notable symbols in this range include `getOpcode`, `insert`, `MBB`.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 该区间中较显眼的符号包括 `getOpcode`, `insert`, `MBB`。

### Lines 415-452

```cpp
#ifndef NDEBUG
      else
        AfterSet.insert(&MI);
#endif
    }
  }

  // Mark the end of the block.
  InsertPos = getEarliestInsertPos(&MBB, BeforeSet, AfterSet);
  MachineInstr *End = BuildMI(MBB, InsertPos, MBB.findPrevDebugLoc(InsertPos),
                              TII.get(WebAssembly::END_BLOCK));
  registerScope(Begin, End);

  // Track the farthest-spanning scope that ends at this point.
  updateScopeTops(Header, &MBB);
}

/// Insert a LOOP marker for a loop starting at MBB (if it's a loop header).
void WebAssemblyCFGStackify::placeLoopMarker(MachineBasicBlock &MBB) {
  MachineFunction &MF = *MBB.getParent();
  const auto &MLI = getAnalysis<MachineLoopInfoWrapperPass>().getLI();
  const auto &WEI = getAnalysis<WebAssemblyExceptionInfo>();
  SortRegionInfo SRI(MLI, WEI);
  const auto &TII = *MF.getSubtarget<WebAssemblySubtarget>().getInstrInfo();

  MachineLoop *Loop = MLI.getLoopFor(&MBB);
  if (!Loop || Loop->getHeader() != &MBB)
    return;

  // The operand of a LOOP is the first block after the loop. If the loop is the
  // bottom of the function, insert a dummy block at the end.
  MachineBasicBlock *Bottom = SRI.getBottom(Loop);
  auto Iter = std::next(Bottom->getIterator());
  if (Iter == MF.end()) {
    getAppendixBlock(MF);
    Iter = std::next(Bottom->getIterator());
  }
  MachineBasicBlock *AfterLoop = &*Iter;
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 453-461

```cpp

  // Decide where in Header to put the LOOP.
  SmallPtrSet<const MachineInstr *, 4> BeforeSet;
  SmallPtrSet<const MachineInstr *, 4> AfterSet;
  for (const auto &MI : MBB) {
    // LOOP marker should be after any existing loop that ends here. Otherwise
    // we assume the instruction belongs to the loop.
    if (MI.getOpcode() == WebAssembly::END_LOOP)
      BeforeSet.insert(&MI);
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Decide where in Header to put the LOOP.". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Decide where in Header to put the LOOP.”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 462-476

```cpp
#ifndef NDEBUG
    else
      AfterSet.insert(&MI);
#endif
  }

  // Mark the beginning of the loop.
  auto InsertPos = getEarliestInsertPos(&MBB, BeforeSet, AfterSet);
  MachineInstr *Begin = BuildMI(MBB, InsertPos, MBB.findDebugLoc(InsertPos),
                                TII.get(WebAssembly::LOOP))
                            .addImm(int64_t(WebAssembly::BlockType::Void));

  // Decide where in MBB to put the END_LOOP.
  BeforeSet.clear();
  AfterSet.clear();
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 477-514

```cpp
#ifndef NDEBUG
  for (const auto &MI : MBB)
    // Existing END_LOOP markers belong to parent loops of this loop
    if (MI.getOpcode() == WebAssembly::END_LOOP)
      AfterSet.insert(&MI);
#endif

  // Mark the end of the loop (using arbitrary debug location that branched to
  // the loop end as its location).
  InsertPos = getEarliestInsertPos(AfterLoop, BeforeSet, AfterSet);
  DebugLoc EndDL = AfterLoop->pred_empty()
                       ? DebugLoc()
                       : (*AfterLoop->pred_rbegin())->findBranchDebugLoc();
  MachineInstr *End =
      BuildMI(*AfterLoop, InsertPos, EndDL, TII.get(WebAssembly::END_LOOP));
  registerScope(Begin, End);

  assert((!ScopeTops[AfterLoop->getNumber()] ||
          ScopeTops[AfterLoop->getNumber()]->getNumber() < MBB.getNumber()) &&
         "With block sorting the outermost loop for a block should be first.");
  updateScopeTops(&MBB, AfterLoop);
}

void WebAssemblyCFGStackify::placeTryMarker(MachineBasicBlock &MBB) {
  assert(MBB.isEHPad());
  MachineFunction &MF = *MBB.getParent();
  auto &MDT = getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
  const auto &TII = *MF.getSubtarget<WebAssemblySubtarget>().getInstrInfo();
  const auto &MLI = getAnalysis<MachineLoopInfoWrapperPass>().getLI();
  const auto &WEI = getAnalysis<WebAssemblyExceptionInfo>();
  SortRegionInfo SRI(MLI, WEI);
  const auto &MFI = *MF.getInfo<WebAssemblyFunctionInfo>();

  // Compute the nearest common dominator of all unwind predecessors
  MachineBasicBlock *Header = nullptr;
  int MBBNumber = MBB.getNumber();
  for (auto *Pred : MBB.predecessors()) {
    if (Pred->getNumber() < MBBNumber) {
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 515-552

```cpp
      Header = Header ? MDT.findNearestCommonDominator(Header, Pred) : Pred;
      assert(!explicitlyBranchesTo(Pred, &MBB) &&
             "Explicit branch to an EH pad!");
    }
  }
  if (!Header)
    return;

  // If this try is at the bottom of the function, insert a dummy block at the
  // end.
  WebAssemblyException *WE = WEI.getExceptionFor(&MBB);
  assert(WE);
  MachineBasicBlock *Bottom = SRI.getBottom(WE);
  auto Iter = std::next(Bottom->getIterator());
  if (Iter == MF.end()) {
    getAppendixBlock(MF);
    Iter = std::next(Bottom->getIterator());
  }
  MachineBasicBlock *Cont = &*Iter;

  // If the nearest common dominator is inside a more deeply nested context,
  // walk out to the nearest scope which isn't more deeply nested.
  for (MachineFunction::iterator I(Bottom), E(Header); I != E; --I) {
    if (MachineBasicBlock *ScopeTop = ScopeTops[I->getNumber()]) {
      if (ScopeTop->getNumber() > Header->getNumber()) {
        // Skip over an intervening scope.
        I = std::next(ScopeTop->getIterator());
      } else {
        // We found a scope level at an appropriate depth.
        Header = ScopeTop;
        break;
      }
    }
  }

  // Decide where in Header to put the TRY.

  // Instructions that should go before the TRY.
```
- **EN**: Implements helper routine(s) `findNearestCommonDominator`, `explicitlyBranchesTo`, `getExceptionFor` for this portion of the WebAssembly backend backend implementation logic.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `findNearestCommonDominator`, `explicitlyBranchesTo`, `getExceptionFor`。

### Lines 553-563

```cpp
  SmallPtrSet<const MachineInstr *, 4> BeforeSet;
  // Instructions that should go after the TRY.
  SmallPtrSet<const MachineInstr *, 4> AfterSet;
  for (const auto &MI : *Header) {
    // If there is a previously placed LOOP marker and the bottom block of the
    // loop is above MBB, it should be after the TRY, because the loop is nested
    // in this TRY. Otherwise it should be before the TRY.
    if (MI.getOpcode() == WebAssembly::LOOP) {
      auto *LoopBottom = BeginToEnd[&MI]->getParent()->getPrevNode();
      if (MBB.getNumber() > LoopBottom->getNumber())
        AfterSet.insert(&MI);
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Instructions that should go after the TRY.". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Instructions that should go after the TRY.”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 564-575

```cpp
#ifndef NDEBUG
      else
        BeforeSet.insert(&MI);
#endif
    }

    // All previously inserted BLOCK/TRY markers should be after the TRY because
    // they are all nested blocks/trys.
    if (MI.getOpcode() == WebAssembly::BLOCK ||
        MI.getOpcode() == WebAssembly::TRY)
      AfterSet.insert(&MI);
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Notable symbols in this range include `insert`, `getOpcode`.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 该区间中较显眼的符号包括 `insert`, `getOpcode`。

### Lines 576-613

```cpp
#ifndef NDEBUG
    // All END_(BLOCK/LOOP/TRY) markers should be before the TRY.
    if (MI.getOpcode() == WebAssembly::END_BLOCK ||
        MI.getOpcode() == WebAssembly::END_LOOP ||
        MI.getOpcode() == WebAssembly::END_TRY)
      BeforeSet.insert(&MI);
#endif

    // Terminators should go after the TRY.
    if (MI.isTerminator())
      AfterSet.insert(&MI);
  }

  // If Header unwinds to MBB (= Header contains 'invoke'), the try block should
  // contain the call within it. So the call should go after the TRY. The
  // exception is when the header's terminator is a rethrow instruction, in
  // which case that instruction, not a call instruction before it, is gonna
  // throw.
  MachineInstr *ThrowingCall = nullptr;
  if (MBB.isPredecessor(Header)) {
    auto TermPos = Header->getFirstTerminator();
    if (TermPos == Header->end() ||
        TermPos->getOpcode() != WebAssembly::RETHROW) {
      for (auto &MI : reverse(*Header)) {
        if (MI.isCall()) {
          AfterSet.insert(&MI);
          ThrowingCall = &MI;
          // Possibly throwing calls are usually wrapped by EH_LABEL
          // instructions. We don't want to split them and the call.
          if (MI.getIterator() != Header->begin() &&
              std::prev(MI.getIterator())->isEHLabel()) {
            AfterSet.insert(&*std::prev(MI.getIterator()));
            ThrowingCall = &*std::prev(MI.getIterator());
          }
          break;
        }
      }
    }
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 614-643

```cpp
  }

  // Local expression tree should go after the TRY.
  // For BLOCK placement, we start the search from the previous instruction of a
  // BB's terminator, but in TRY's case, we should start from the previous
  // instruction of a call that can throw, or a EH_LABEL that precedes the call,
  // because the return values of the call's previous instructions can be
  // stackified and consumed by the throwing call.
  auto SearchStartPt = ThrowingCall ? MachineBasicBlock::iterator(ThrowingCall)
                                    : Header->getFirstTerminator();
  for (auto I = SearchStartPt, E = Header->begin(); I != E; --I) {
    if (std::prev(I)->isDebugInstr() || std::prev(I)->isPosition())
      continue;
    if (WebAssembly::isChild(*std::prev(I), MFI))
      AfterSet.insert(&*std::prev(I));
    else
      break;
  }

  // Add the TRY.
  auto InsertPos = getLatestInsertPos(Header, BeforeSet, AfterSet);
  MachineInstr *Begin =
      BuildMI(*Header, InsertPos, Header->findDebugLoc(InsertPos),
              TII.get(WebAssembly::TRY))
          .addImm(int64_t(WebAssembly::BlockType::Void));

  // Decide where in Cont to put the END_TRY.
  BeforeSet.clear();
  AfterSet.clear();
  for (const auto &MI : *Cont) {
```
- **EN**: Implements helper routine(s) `iterator`, `getFirstTerminator`, `begin` for this portion of the WebAssembly backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `iterator`, `getFirstTerminator`, `begin`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 644-652

```cpp
#ifndef NDEBUG
    // END_TRY should precede existing LOOP markers.
    if (MI.getOpcode() == WebAssembly::LOOP)
      AfterSet.insert(&MI);

    // All END_TRY markers placed earlier belong to exceptions that contains
    // this one.
    if (MI.getOpcode() == WebAssembly::END_TRY)
      AfterSet.insert(&MI);
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Notable symbols in this range include `getOpcode`, `insert`.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 该区间中较显眼的符号包括 `getOpcode`, `insert`。

### Lines 653-663

```cpp
#endif

    // If there is a previously placed END_LOOP marker and its header is after
    // where TRY marker is, this loop is contained within the 'catch' part, so
    // the END_TRY marker should go after that. Otherwise, the whole try-catch
    // is contained within this loop, so the END_TRY should go before that.
    if (MI.getOpcode() == WebAssembly::END_LOOP) {
      // For a LOOP to be after TRY, LOOP's BB should be after TRY's BB; if they
      // are in the same BB, LOOP is always before TRY.
      if (EndToBegin[&MI]->getParent()->getNumber() > Header->getNumber())
        BeforeSet.insert(&MI);
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Notable symbols in this range include `getOpcode`, `getParent`, `getNumber`.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 该区间中较显眼的符号包括 `getOpcode`, `getParent`, `getNumber`。

### Lines 664-701

```cpp
#ifndef NDEBUG
      else
        AfterSet.insert(&MI);
#endif
    }

    // It is not possible for an END_BLOCK to be already in this block.
  }

  // Mark the end of the TRY.
  InsertPos = getEarliestInsertPos(Cont, BeforeSet, AfterSet);
  MachineInstr *End = BuildMI(*Cont, InsertPos, Bottom->findBranchDebugLoc(),
                              TII.get(WebAssembly::END_TRY));
  registerTryScope(Begin, End, &MBB);

  // Track the farthest-spanning scope that ends at this point. We create two
  // mappings: (BB with 'end_try' -> BB with 'try') and (BB with 'catch' -> BB
  // with 'try'). We need to create 'catch' -> 'try' mapping here too because
  // markers should not span across 'catch'. For example, this should not
  // happen:
  //
  // try
  //   block     --|  (X)
  // catch         |
  //   end_block --|
  // end_try
  for (auto *End : {&MBB, Cont})
    updateScopeTops(Header, End);
}

void WebAssemblyCFGStackify::placeTryTableMarker(MachineBasicBlock &MBB) {
  assert(MBB.isEHPad());
  MachineFunction &MF = *MBB.getParent();
  auto &MDT = getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
  const auto &TII = *MF.getSubtarget<WebAssemblySubtarget>().getInstrInfo();
  const auto &MLI = getAnalysis<MachineLoopInfoWrapperPass>().getLI();
  const auto &WEI = getAnalysis<WebAssemblyExceptionInfo>();
  SortRegionInfo SRI(MLI, WEI);
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 702-739

```cpp
  const auto &MFI = *MF.getInfo<WebAssemblyFunctionInfo>();

  // Compute the nearest common dominator of all unwind predecessors
  MachineBasicBlock *Header = nullptr;
  int MBBNumber = MBB.getNumber();
  for (auto *Pred : MBB.predecessors()) {
    if (Pred->getNumber() < MBBNumber) {
      Header = Header ? MDT.findNearestCommonDominator(Header, Pred) : Pred;
      assert(!explicitlyBranchesTo(Pred, &MBB) &&
             "Explicit branch to an EH pad!");
    }
  }
  if (!Header)
    return;

  // Unlike the end_try marker, we don't place an end marker at the end of
  // exception bottom, i.e., at the end of the old 'catch' block. But we still
  // consider the try-catch part as a scope when computing ScopeTops.
  WebAssemblyException *WE = WEI.getExceptionFor(&MBB);
  assert(WE);
  MachineBasicBlock *Bottom = SRI.getBottom(WE);
  auto Iter = std::next(Bottom->getIterator());
  if (Iter == MF.end())
    Iter--;
  MachineBasicBlock *Cont = &*Iter;

  // If the nearest common dominator is inside a more deeply nested context,
  // walk out to the nearest scope which isn't more deeply nested.
  for (MachineFunction::iterator I(Bottom), E(Header); I != E; --I) {
    if (MachineBasicBlock *ScopeTop = ScopeTops[I->getNumber()]) {
      if (ScopeTop->getNumber() > Header->getNumber()) {
        // Skip over an intervening scope.
        I = std::next(ScopeTop->getIterator());
      } else {
        // We found a scope level at an appropriate depth.
        Header = ScopeTop;
        break;
      }
```
- **EN**: Implements helper routine(s) `getNumber`, `predecessors`, `findNearestCommonDominator` for this portion of the WebAssembly backend backend implementation logic.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `getNumber`, `predecessors`, `findNearestCommonDominator`。

### Lines 740-756

```cpp
    }
  }

  // Decide where in Header to put the TRY_TABLE.

  // Instructions that should go before the TRY_TABLE.
  SmallPtrSet<const MachineInstr *, 4> BeforeSet;
  // Instructions that should go after the TRY_TABLE.
  SmallPtrSet<const MachineInstr *, 4> AfterSet;
  for (const auto &MI : *Header) {
    // If there is a previously placed LOOP marker and the bottom block of the
    // loop is above MBB, it should be after the TRY_TABLE, because the loop is
    // nested in this TRY_TABLE. Otherwise it should be before the TRY_TABLE.
    if (MI.getOpcode() == WebAssembly::LOOP) {
      auto *LoopBottom = BeginToEnd[&MI]->getParent()->getPrevNode();
      if (MBB.getNumber() > LoopBottom->getNumber())
        AfterSet.insert(&MI);
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Decide where in Header to put the TRY_TABLE.". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Decide where in Header to put the TRY_TABLE.”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 757-768

```cpp
#ifndef NDEBUG
      else
        BeforeSet.insert(&MI);
#endif
    }

    // All previously inserted BLOCK/TRY_TABLE markers should be after the
    // TRY_TABLE because they are all nested blocks/try_tables.
    if (MI.getOpcode() == WebAssembly::BLOCK ||
        MI.getOpcode() == WebAssembly::TRY_TABLE)
      AfterSet.insert(&MI);
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Notable symbols in this range include `insert`, `getOpcode`.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 该区间中较显眼的符号包括 `insert`, `getOpcode`。

### Lines 769-806

```cpp
#ifndef NDEBUG
    // All END_(BLOCK/LOOP/TRY_TABLE) markers should be before the TRY_TABLE.
    if (MI.getOpcode() == WebAssembly::END_BLOCK ||
        MI.getOpcode() == WebAssembly::END_LOOP ||
        MI.getOpcode() == WebAssembly::END_TRY_TABLE)
      BeforeSet.insert(&MI);
#endif

    // Terminators should go after the TRY_TABLE.
    if (MI.isTerminator())
      AfterSet.insert(&MI);
  }

  // If Header unwinds to MBB (= Header contains 'invoke'), the try_table block
  // should contain the call within it. So the call should go after the
  // TRY_TABLE. The exception is when the header's terminator is a rethrow
  // instruction, in which case that instruction, not a call instruction before
  // it, is gonna throw.
  MachineInstr *ThrowingCall = nullptr;
  if (MBB.isPredecessor(Header)) {
    auto TermPos = Header->getFirstTerminator();
    if (TermPos == Header->end() ||
        TermPos->getOpcode() != WebAssembly::RETHROW) {
      for (auto &MI : reverse(*Header)) {
        if (MI.isCall()) {
          AfterSet.insert(&MI);
          ThrowingCall = &MI;
          // Possibly throwing calls are usually wrapped by EH_LABEL
          // instructions. We don't want to split them and the call.
          if (MI.getIterator() != Header->begin() &&
              std::prev(MI.getIterator())->isEHLabel()) {
            AfterSet.insert(&*std::prev(MI.getIterator()));
            ThrowingCall = &*std::prev(MI.getIterator());
          }
          break;
        }
      }
    }
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 807-844

```cpp
  }

  // Local expression tree should go after the TRY_TABLE.
  // For BLOCK placement, we start the search from the previous instruction of a
  // BB's terminator, but in TRY_TABLE's case, we should start from the previous
  // instruction of a call that can throw, or a EH_LABEL that precedes the call,
  // because the return values of the call's previous instructions can be
  // stackified and consumed by the throwing call.
  auto SearchStartPt = ThrowingCall ? MachineBasicBlock::iterator(ThrowingCall)
                                    : Header->getFirstTerminator();
  for (auto I = SearchStartPt, E = Header->begin(); I != E; --I) {
    if (std::prev(I)->isDebugInstr() || std::prev(I)->isPosition())
      continue;
    if (WebAssembly::isChild(*std::prev(I), MFI))
      AfterSet.insert(&*std::prev(I));
    else
      break;
  }

  // Add the TRY_TABLE and a BLOCK for the catch destination. We currently
  // generate only one CATCH clause for a TRY_TABLE, so we need one BLOCK for
  // its destination.
  //
  // Header:
  //   block
  //     try_table (catch ... $MBB)
  //       ...
  //
  // MBB:
  //     end_try_table
  //   end_block                 ;; destination of (catch ...)
  //   ... catch handler body ...
  auto InsertPos = getLatestInsertPos(Header, BeforeSet, AfterSet);
  MachineInstrBuilder BlockMIB =
      BuildMI(*Header, InsertPos, Header->findDebugLoc(InsertPos),
              TII.get(WebAssembly::BLOCK));
  auto *Block = BlockMIB.getInstr();
  MachineInstrBuilder TryTableMIB =
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Local expression tree should go after the TRY_TABLE.". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Local expression tree should go after the TRY_TABLE.”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 845-882

```cpp
      BuildMI(*Header, InsertPos, Header->findDebugLoc(InsertPos),
              TII.get(WebAssembly::TRY_TABLE))
          .addImm(int64_t(WebAssembly::BlockType::Void))
          .addImm(1); // # of catch clauses
  auto *TryTable = TryTableMIB.getInstr();

  // Add a CATCH_*** clause to the TRY_TABLE. These are pseudo instructions
  // following the destination END_BLOCK to simulate block return values,
  // because we currently don't support them.
  const auto &TLI =
      *MF.getSubtarget<WebAssemblySubtarget>().getTargetLowering();
  WebAssembly::BlockType PtrTy =
      TLI.getPointerTy(MF.getDataLayout()) == MVT::i32
          ? WebAssembly::BlockType::I32
          : WebAssembly::BlockType::I64;
  auto *Catch = WebAssembly::findCatch(&MBB);
  switch (Catch->getOpcode()) {
  case WebAssembly::CATCH:
    // CATCH's destination block's return type is the extracted value type,
    // which is currently the thrown value's pointer type for all supported
    // tags.
    BlockMIB.addImm(int64_t(PtrTy));
    TryTableMIB.addImm(wasm::WASM_OPCODE_CATCH);
    for (const auto &Use : Catch->uses()) {
      // The only use operand a CATCH can have is the tag symbol.
      TryTableMIB.addExternalSymbol(Use.getSymbolName());
      break;
    }
    TryTableMIB.addMBB(&MBB);
    break;
  case WebAssembly::CATCH_REF:
    // CATCH_REF's destination block's return type is the extracted value type
    // followed by an exnref, which is (i32, exnref) in our case. We assign the
    // actual multiavlue signature in MCInstLower. MO_CATCH_BLOCK_SIG signals
    // that this operand is used for catch_ref's multivalue destination.
    BlockMIB.addImm(int64_t(WebAssembly::BlockType::Multivalue));
    Block->getOperand(0).setTargetFlags(WebAssemblyII::MO_CATCH_BLOCK_SIG);
    TryTableMIB.addImm(wasm::WASM_OPCODE_CATCH_REF);
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Add a CATCH_*** clause to the TRY_TABLE. These are pseudo instructions". Subtarget feature gating influences the behavior here.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Add a CATCH_*** clause to the TRY_TABLE. These are pseudo instructions”。 子目标特性裁剪会影响这里的行为。

### Lines 883-907

```cpp
    for (const auto &Use : Catch->uses()) {
      TryTableMIB.addExternalSymbol(Use.getSymbolName());
      break;
    }
    TryTableMIB.addMBB(&MBB);
    break;
  case WebAssembly::CATCH_ALL:
    // CATCH_ALL's destination block's return type is void.
    BlockMIB.addImm(int64_t(WebAssembly::BlockType::Void));
    TryTableMIB.addImm(wasm::WASM_OPCODE_CATCH_ALL);
    TryTableMIB.addMBB(&MBB);
    break;
  case WebAssembly::CATCH_ALL_REF:
    // CATCH_ALL_REF's destination block's return type is exnref.
    BlockMIB.addImm(int64_t(WebAssembly::BlockType::Exnref));
    TryTableMIB.addImm(wasm::WASM_OPCODE_CATCH_ALL_REF);
    TryTableMIB.addMBB(&MBB);
    break;
  }

  // Decide where in MBB to put the END_TRY_TABLE, and the END_BLOCK for the
  // CATCH destination.
  BeforeSet.clear();
  AfterSet.clear();
  for (const auto &MI : MBB) {
```
- **EN**: Implements helper routine(s) `uses`, `addExternalSymbol`, `getSymbolName` for this portion of the WebAssembly backend backend implementation logic.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `uses`, `addExternalSymbol`, `getSymbolName`。

### Lines 908-920

```cpp
#ifndef NDEBUG
    // END_TRY_TABLE should precede existing LOOP markers.
    if (MI.getOpcode() == WebAssembly::LOOP)
      AfterSet.insert(&MI);
#endif

    // If there is a previously placed END_LOOP marker and the header of the
    // loop is above this try_table's header, the END_LOOP should be placed
    // after the END_TRY_TABLE, because the loop contains this block. Otherwise
    // the END_LOOP should be placed before the END_TRY_TABLE.
    if (MI.getOpcode() == WebAssembly::END_LOOP) {
      if (EndToBegin[&MI]->getParent()->getNumber() >= Header->getNumber())
        BeforeSet.insert(&MI);
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Notable symbols in this range include `getOpcode`, `insert`, `getParent`.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 该区间中较显眼的符号包括 `getOpcode`, `insert`, `getParent`。

### Lines 921-932

```cpp
#ifndef NDEBUG
      else
        AfterSet.insert(&MI);
#endif
    }

#ifndef NDEBUG
    // CATCH, CATCH_REF, CATCH_ALL, and CATCH_ALL_REF are pseudo-instructions
    // that simulate the block return value, so they should be placed after the
    // END_TRY_TABLE.
    if (WebAssembly::isCatch(MI.getOpcode()))
      AfterSet.insert(&MI);
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Notable symbols in this range include `insert`, `isCatch`, `getOpcode`.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 该区间中较显眼的符号包括 `insert`, `isCatch`, `getOpcode`。

### Lines 933-970

```cpp
#endif
  }

  // Mark the end of the TRY_TABLE and the BLOCK.
  InsertPos = getEarliestInsertPos(&MBB, BeforeSet, AfterSet);
  MachineInstr *EndTryTable =
      BuildMI(MBB, InsertPos, MBB.findPrevDebugLoc(InsertPos),
              TII.get(WebAssembly::END_TRY_TABLE));
  registerTryScope(TryTable, EndTryTable, &MBB);
  MachineInstr *EndBlock =
      BuildMI(MBB, InsertPos, MBB.findPrevDebugLoc(InsertPos),
              TII.get(WebAssembly::END_BLOCK));
  registerScope(Block, EndBlock);

  // Track the farthest-spanning scope that ends at this point.
  // Unlike the end_try, even if we don't put a end marker at the end of catch
  // block, we still have to create two mappings: (BB with 'end_try_table' -> BB
  // with 'try_table') and (BB after the (conceptual) catch block -> BB with
  // 'try_table').
  //
  // This is what can happen if we don't create the latter mapping:
  //
  // Suppoe in the legacy EH we have this code:
  // try
  //   try
  //     code1
  //   catch (a)
  //   end_try
  //   code2
  // catch (b)
  // end_try
  //
  // If we don't create the latter mapping, try_table markers would be placed
  // like this:
  // try_table
  //   code1
  // end_try_table (a)
  // try_table
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 971-1008

```cpp
  //   code2
  // end_try_table (b)
  //
  // This does not reflect the original structure, and more important problem
  // is, in case 'code1' has an unwind mismatch and should unwind to
  // 'end_try_table (b)' rather than 'end_try_table (a)', we don't have a way to
  // make it jump after 'end_try_table (b)' without creating another block. So
  // even if we don't place 'end_try' marker at the end of 'catch' block
  // anymore, we create ScopeTops mapping the same way as the legacy exception,
  // so the resulting code will look like:
  // try_table
  //   try_table
  //     code1
  //   end_try_table (a)
  //   code2
  // end_try_table (b)
  for (auto *End : {&MBB, Cont})
    updateScopeTops(Header, End);
}

void WebAssemblyCFGStackify::removeUnnecessaryInstrs(MachineFunction &MF) {
  const auto &TII = *MF.getSubtarget<WebAssemblySubtarget>().getInstrInfo();

  // When there is an unconditional branch right before a catch instruction and
  // it branches to the end of end_try marker, we don't need the branch, because
  // if there is no exception, the control flow transfers to that point anyway.
  // bb0:
  //   try
  //     ...
  //     br bb2      <- Not necessary
  // bb1 (ehpad):
  //   catch
  //     ...
  // bb2:            <- Continuation BB
  //   end
  //
  // A more involved case: When the BB where 'end' is located is an another EH
  // pad, the Cont (= continuation) BB is that EH pad's 'end' BB. For example,
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "code2". Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“code2”。 该区间与栈帧布局或栈访问相关。

### Lines 1009-1046

```cpp
  // bb0:
  //   try
  //     try
  //       ...
  //       br bb3      <- Not necessary
  // bb1 (ehpad):
  //     catch
  // bb2 (ehpad):
  //     end
  //   catch
  //     ...
  // bb3:            <- Continuation BB
  //   end
  //
  // When the EH pad at hand is bb1, its matching end_try is in bb2. But it is
  // another EH pad, so bb0's continuation BB becomes bb3. So 'br bb3' in the
  // code can be deleted. This is why we run 'while' until 'Cont' is not an EH
  // pad.
  for (auto &MBB : MF) {
    if (!MBB.isEHPad())
      continue;

    MachineBasicBlock *TBB = nullptr, *FBB = nullptr;
    SmallVector<MachineOperand, 4> Cond;
    MachineBasicBlock *EHPadLayoutPred = MBB.getPrevNode();

    MachineBasicBlock *Cont = &MBB;
    while (Cont->isEHPad()) {
      MachineInstr *Try = EHPadToTry[Cont];
      MachineInstr *EndTry = BeginToEnd[Try];
      // We started from an EH pad, so the end marker cannot be a delegate
      assert(EndTry->getOpcode() != WebAssembly::DELEGATE);
      Cont = EndTry->getParent();
    }

    bool Analyzable = !TII.analyzeBranch(*EHPadLayoutPred, TBB, FBB, Cond);
    // This condition means either
    // 1. This BB ends with a single unconditional branch whose destinaion is
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "bb0:". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“bb0:”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 1047-1084

```cpp
    //    Cont.
    // 2. This BB ends with a conditional branch followed by an unconditional
    //    branch, and the unconditional branch's destination is Cont.
    // In both cases, we want to remove the last (= unconditional) branch.
    if (Analyzable && ((Cond.empty() && TBB && TBB == Cont) ||
                       (!Cond.empty() && FBB && FBB == Cont))) {
      bool ErasedUncondBr = false;
      (void)ErasedUncondBr;
      for (auto I = EHPadLayoutPred->end(), E = EHPadLayoutPred->begin();
           I != E; --I) {
        auto PrevI = std::prev(I);
        if (PrevI->isTerminator()) {
          assert(PrevI->getOpcode() == WebAssembly::BR);
          PrevI->eraseFromParent();
          ErasedUncondBr = true;
          break;
        }
      }
      assert(ErasedUncondBr && "Unconditional branch not erased!");
    }
  }

  // When there are block / end_block markers that overlap with try / end_try
  // markers, and the block and try markers' return types are the same, the
  // block /end_block markers are not necessary, because try / end_try markers
  // also can serve as boundaries for branches.
  // block         <- Not necessary
  //   try
  //     ...
  //   catch
  //     ...
  //   end
  // end           <- Not necessary
  SmallVector<MachineInstr *, 32> ToDelete;
  for (auto &MBB : MF) {
    for (auto &MI : MBB) {
      if (MI.getOpcode() != WebAssembly::TRY)
        continue;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Cont.". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Cont.”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 1085-1122

```cpp
      MachineInstr *Try = &MI, *EndTry = BeginToEnd[Try];
      if (EndTry->getOpcode() == WebAssembly::DELEGATE)
        continue;

      MachineBasicBlock *TryBB = Try->getParent();
      MachineBasicBlock *Cont = EndTry->getParent();
      int64_t RetType = Try->getOperand(0).getImm();
      for (auto B = Try->getIterator(), E = std::next(EndTry->getIterator());
           B != TryBB->begin() && E != Cont->end() &&
           std::prev(B)->getOpcode() == WebAssembly::BLOCK &&
           E->getOpcode() == WebAssembly::END_BLOCK &&
           std::prev(B)->getOperand(0).getImm() == RetType;
           --B, ++E) {
        ToDelete.push_back(&*std::prev(B));
        ToDelete.push_back(&*E);
      }
    }
  }
  for (auto *MI : ToDelete) {
    if (MI->getOpcode() == WebAssembly::BLOCK)
      unregisterScope(MI);
    MI->eraseFromParent();
  }
}

// When MBB is split into MBB and Split, we should unstackify defs in MBB that
// have their uses in Split.
static void unstackifyVRegsUsedInSplitBB(MachineBasicBlock &MBB,
                                         MachineBasicBlock &Split) {
  MachineFunction &MF = *MBB.getParent();
  const auto &TII = *MF.getSubtarget<WebAssemblySubtarget>().getInstrInfo();
  auto &MFI = *MF.getInfo<WebAssemblyFunctionInfo>();
  auto &MRI = MF.getRegInfo();

  for (auto &MI : Split) {
    for (auto &MO : MI.explicit_uses()) {
      if (!MO.isReg() || MO.getReg().isPhysical())
        continue;
```
- **EN**: Implements helper routine(s) `getOpcode`, `getParent`, `getOperand` for this portion of the WebAssembly backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `getOpcode`, `getParent`, `getOperand`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 1123-1160

```cpp
      if (MachineInstr *Def = MRI.getUniqueVRegDef(MO.getReg()))
        if (Def->getParent() == &MBB)
          MFI.unstackifyVReg(MO.getReg());
    }
  }

  // In RegStackify, when a register definition is used multiple times,
  //    Reg = INST ...
  //    INST ..., Reg, ...
  //    INST ..., Reg, ...
  //    INST ..., Reg, ...
  //
  // we introduce a TEE, which has the following form:
  //    DefReg = INST ...
  //    TeeReg, Reg = TEE_... DefReg
  //    INST ..., TeeReg, ...
  //    INST ..., Reg, ...
  //    INST ..., Reg, ...
  // with DefReg and TeeReg stackified but Reg not stackified.
  //
  // But the invariant that TeeReg should be stackified can be violated while we
  // unstackify registers in the split BB above. In this case, we convert TEEs
  // into two COPYs. This COPY will be eventually eliminated in ExplicitLocals.
  //    DefReg = INST ...
  //    TeeReg = COPY DefReg
  //    Reg = COPY DefReg
  //    INST ..., TeeReg, ...
  //    INST ..., Reg, ...
  //    INST ..., Reg, ...
  for (MachineInstr &MI : llvm::make_early_inc_range(MBB)) {
    if (!WebAssembly::isTee(MI.getOpcode()))
      continue;
    Register TeeReg = MI.getOperand(0).getReg();
    Register Reg = MI.getOperand(1).getReg();
    Register DefReg = MI.getOperand(2).getReg();
    if (!MFI.isVRegStackified(TeeReg)) {
      // Now we are not using TEE anymore, so unstackify DefReg too
      MFI.unstackifyVReg(DefReg);
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "In RegStackify, when a register definition is used multiple times,". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“In RegStackify, when a register definition is used multiple times,”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 1161-1198

```cpp
      unsigned CopyOpc =
          WebAssembly::getCopyOpcodeForRegClass(MRI.getRegClass(DefReg));
      BuildMI(MBB, &MI, MI.getDebugLoc(), TII.get(CopyOpc), TeeReg)
          .addReg(DefReg);
      BuildMI(MBB, &MI, MI.getDebugLoc(), TII.get(CopyOpc), Reg).addReg(DefReg);
      MI.eraseFromParent();
    }
  }
}

// Wrap the given range of instructions with a try-delegate that targets
// 'UnwindDest'. RangeBegin and RangeEnd are inclusive.
void WebAssemblyCFGStackify::addNestedTryDelegate(
    MachineInstr *RangeBegin, MachineInstr *RangeEnd,
    MachineBasicBlock *UnwindDest) {
  auto *BeginBB = RangeBegin->getParent();
  auto *EndBB = RangeEnd->getParent();
  MachineFunction &MF = *BeginBB->getParent();
  const auto &MFI = *MF.getInfo<WebAssemblyFunctionInfo>();
  const auto &TII = *MF.getSubtarget<WebAssemblySubtarget>().getInstrInfo();

  // Local expression tree before the first call of this range should go
  // after the nested TRY.
  SmallPtrSet<const MachineInstr *, 4> AfterSet;
  AfterSet.insert(RangeBegin);
  for (auto I = MachineBasicBlock::iterator(RangeBegin), E = BeginBB->begin();
       I != E; --I) {
    if (std::prev(I)->isDebugInstr() || std::prev(I)->isPosition())
      continue;
    if (WebAssembly::isChild(*std::prev(I), MFI))
      AfterSet.insert(&*std::prev(I));
    else
      break;
  }

  // Create the nested try instruction.
  auto TryPos = getLatestInsertPos(
      BeginBB, SmallPtrSet<const MachineInstr *, 4>(), AfterSet);
```
- **EN**: Implements helper routine(s) `getCopyOpcodeForRegClass`, `getRegClass`, `BuildMI` for this portion of the WebAssembly backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `getCopyOpcodeForRegClass`, `getRegClass`, `BuildMI`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 1199-1236

```cpp
  MachineInstr *Try = BuildMI(*BeginBB, TryPos, RangeBegin->getDebugLoc(),
                              TII.get(WebAssembly::TRY))
                          .addImm(int64_t(WebAssembly::BlockType::Void));

  // Create a BB to insert the 'delegate' instruction.
  MachineBasicBlock *DelegateBB = MF.CreateMachineBasicBlock();
  // If the destination of 'delegate' is not the caller, adds the destination to
  // the BB's successors.
  if (UnwindDest != FakeCallerBB)
    DelegateBB->addSuccessor(UnwindDest);

  auto SplitPos = std::next(RangeEnd->getIterator());
  if (SplitPos == EndBB->end()) {
    // If the range's end instruction is at the end of the BB, insert the new
    // delegate BB after the current BB.
    MF.insert(std::next(EndBB->getIterator()), DelegateBB);
    EndBB->addSuccessor(DelegateBB);

  } else {
    // When the split pos is in the middle of a BB, we split the BB into two and
    // put the 'delegate' BB in between. We normally create a split BB and make
    // it a successor of the original BB (CatchAfterSplit == false), but in case
    // the BB is an EH pad and there is a 'catch' after the split pos
    // (CatchAfterSplit == true), we should preserve the BB's property,
    // including that it is an EH pad, in the later part of the BB, where the
    // 'catch' is.
    bool CatchAfterSplit = false;
    if (EndBB->isEHPad()) {
      for (auto I = MachineBasicBlock::iterator(SplitPos), E = EndBB->end();
           I != E; ++I) {
        if (WebAssembly::isCatch(I->getOpcode())) {
          CatchAfterSplit = true;
          break;
        }
      }
    }

    MachineBasicBlock *PreBB = nullptr, *PostBB = nullptr;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Create a BB to insert the 'delegate' instruction.". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Create a BB to insert the 'delegate' instruction.”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 1237-1274

```cpp
    if (!CatchAfterSplit) {
      // If the range's end instruction is in the middle of the BB, we split the
      // BB into two and insert the delegate BB in between.
      // - Before:
      // bb:
      //   range_end
      //   other_insts
      //
      // - After:
      // pre_bb: (previous 'bb')
      //   range_end
      // delegate_bb: (new)
      //   delegate
      // post_bb: (new)
      //   other_insts
      PreBB = EndBB;
      PostBB = MF.CreateMachineBasicBlock();
      MF.insert(std::next(PreBB->getIterator()), PostBB);
      MF.insert(std::next(PreBB->getIterator()), DelegateBB);
      PostBB->splice(PostBB->end(), PreBB, SplitPos, PreBB->end());
      PostBB->transferSuccessors(PreBB);
    } else {
      // - Before:
      // ehpad:
      //   range_end
      //   catch
      //   ...
      //
      // - After:
      // pre_bb: (new)
      //   range_end
      // delegate_bb: (new)
      //   delegate
      // post_bb: (previous 'ehpad')
      //   catch
      //   ...
      assert(EndBB->isEHPad());
      PreBB = MF.CreateMachineBasicBlock();
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "If the range's end instruction is in the middle of the BB, we split the". Notable symbols in this range include `pre_bb:`, `delegate_bb:`, `post_bb:`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“If the range's end instruction is in the middle of the BB, we split the”。 该区间中较显眼的符号包括 `pre_bb:`, `delegate_bb:`, `post_bb:`。

### Lines 1275-1312

```cpp
      PostBB = EndBB;
      MF.insert(PostBB->getIterator(), PreBB);
      MF.insert(PostBB->getIterator(), DelegateBB);
      PreBB->splice(PreBB->end(), PostBB, PostBB->begin(), SplitPos);
      // We don't need to transfer predecessors of the EH pad to 'PreBB',
      // because an EH pad's predecessors are all through unwind edges and they
      // should still unwind to the EH pad, not PreBB.
    }
    unstackifyVRegsUsedInSplitBB(*PreBB, *PostBB);
    PreBB->addSuccessor(DelegateBB);
    PreBB->addSuccessor(PostBB);
  }

  // Add a 'delegate' instruction in the delegate BB created above.
  MachineInstr *Delegate = BuildMI(DelegateBB, RangeEnd->getDebugLoc(),
                                   TII.get(WebAssembly::DELEGATE))
                               .addMBB(UnwindDest);
  registerTryScope(Try, Delegate, nullptr);
}

// Given an unwind destination, return a trampoline BB. A trampoline BB is a
// destination of a nested try_table inserted to fix an unwind mismatch. It
// contains an end_block, which is the target of the try_table, and a throw_ref,
// to rethrow the exception to the right try_table.
// try_table (catch ... )
//   block exnref
//     ...
//     try_table (catch_all_ref N)
//       some code
//     end_try_table
//     ...
//     unreachable
//   end_block                      ;; Trampoline BB
//   throw_ref
// end_try_table
MachineBasicBlock *
WebAssemblyCFGStackify::getTrampolineBlock(MachineBasicBlock *UnwindDest) {
  // We need one trampoline BB per unwind destination, even though there are
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "We don't need to transfer predecessors of the EH pad to 'PreBB',". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“We don't need to transfer predecessors of the EH pad to 'PreBB',”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 1313-1350

```cpp
  // multiple try_tables target the same unwind destination. If we have already
  // created one for the given UnwindDest, return it.
  auto It = UnwindDestToTrampoline.find(UnwindDest);
  if (It != UnwindDestToTrampoline.end())
    return It->second;

  auto &MF = *UnwindDest->getParent();
  auto &MRI = MF.getRegInfo();
  const auto &TII = *MF.getSubtarget<WebAssemblySubtarget>().getInstrInfo();

  MachineInstr *Block = nullptr;
  MachineBasicBlock *TrampolineBB = nullptr;
  DebugLoc EndDebugLoc;

  if (UnwindDest == getFakeCallerBlock(MF)) {
    // If the unwind destination is the caller, create a caller-dedicated
    // trampoline BB at the end of the function and wrap the whole function with
    // a block.
    auto BeginPos = MF.begin()->begin();
    while (WebAssembly::isArgument(BeginPos->getOpcode()))
      BeginPos++;
    Block = BuildMI(*MF.begin(), BeginPos, MF.begin()->begin()->getDebugLoc(),
                    TII.get(WebAssembly::BLOCK))
                .addImm(int64_t(WebAssembly::BlockType::Exnref));
    TrampolineBB = getCallerTrampolineBlock(MF);
    MachineBasicBlock *PrevBB = &*std::prev(CallerTrampolineBB->getIterator());
    EndDebugLoc = PrevBB->findPrevDebugLoc(PrevBB->end());
  } else {
    // If the unwind destination is another EH pad, create a trampoline BB for
    // the unwind dest and insert a block instruction right after the target
    // try_table.
    auto *TargetBeginTry = EHPadToTry[UnwindDest];
    auto *TargetEndTry = BeginToEnd[TargetBeginTry];
    auto *TargetBeginBB = TargetBeginTry->getParent();
    auto *TargetEndBB = TargetEndTry->getParent();

    Block = BuildMI(*TargetBeginBB, std::next(TargetBeginTry->getIterator()),
                    TargetBeginTry->getDebugLoc(), TII.get(WebAssembly::BLOCK))
```
- **EN**: Implements helper routine(s) `find`, `end`, `getParent` for this portion of the WebAssembly backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `find`, `end`, `getParent`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 1351-1388

```cpp
                .addImm(int64_t(WebAssembly::BlockType::Exnref));
    TrampolineBB = MF.CreateMachineBasicBlock();
    EndDebugLoc = TargetEndTry->getDebugLoc();
    MF.insert(TargetEndBB->getIterator(), TrampolineBB);
    TrampolineBB->addSuccessor(UnwindDest);
  }

  // Insert an end_block, catch_all_ref (pseudo instruction), and throw_ref
  // instructions in the trampoline BB.
  MachineInstr *EndBlock =
      BuildMI(TrampolineBB, EndDebugLoc, TII.get(WebAssembly::END_BLOCK));
  auto ExnReg = MRI.createVirtualRegister(&WebAssembly::EXNREFRegClass);
  BuildMI(TrampolineBB, EndDebugLoc, TII.get(WebAssembly::CATCH_ALL_REF))
      .addDef(ExnReg);
  BuildMI(TrampolineBB, EndDebugLoc, TII.get(WebAssembly::THROW_REF))
      .addReg(ExnReg);

  // The trampoline BB's return type is exnref because it is a target of
  // catch_all_ref. But the body type of the block we just created is not. We
  // add an 'unreachable' right before the 'end_block' to make the code valid.
  MachineBasicBlock *TrampolineLayoutPred = TrampolineBB->getPrevNode();
  BuildMI(TrampolineLayoutPred, TrampolineLayoutPred->findBranchDebugLoc(),
          TII.get(WebAssembly::UNREACHABLE));

  registerScope(Block, EndBlock);
  UnwindDestToTrampoline[UnwindDest] = TrampolineBB;
  return TrampolineBB;
}

// Wrap the given range of instructions with a try_table-end_try_table that
// targets 'UnwindDest'. RangeBegin and RangeEnd are inclusive.
void WebAssemblyCFGStackify::addNestedTryTable(MachineInstr *RangeBegin,
                                               MachineInstr *RangeEnd,
                                               MachineBasicBlock *UnwindDest) {
  auto *BeginBB = RangeBegin->getParent();
  auto *EndBB = RangeEnd->getParent();

  MachineFunction &MF = *BeginBB->getParent();
```
- **EN**: Implements helper routine(s) `addImm`, `int64_t`, `CreateMachineBasicBlock` for this portion of the WebAssembly backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `addImm`, `int64_t`, `CreateMachineBasicBlock`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 1389-1426

```cpp
  const auto &MFI = *MF.getInfo<WebAssemblyFunctionInfo>();
  const auto &TII = *MF.getSubtarget<WebAssemblySubtarget>().getInstrInfo();

  // Get the trampoline BB that the new try_table will unwind to.
  auto *TrampolineBB = getTrampolineBlock(UnwindDest);

  // Local expression tree before the first call of this range should go
  // after the nested TRY_TABLE.
  SmallPtrSet<const MachineInstr *, 4> AfterSet;
  AfterSet.insert(RangeBegin);
  for (auto I = MachineBasicBlock::iterator(RangeBegin), E = BeginBB->begin();
       I != E; --I) {
    if (std::prev(I)->isDebugInstr() || std::prev(I)->isPosition())
      continue;
    if (WebAssembly::isChild(*std::prev(I), MFI))
      AfterSet.insert(&*std::prev(I));
    else
      break;
  }

  // Create the nested try_table instruction.
  auto TryTablePos = getLatestInsertPos(
      BeginBB, SmallPtrSet<const MachineInstr *, 4>(), AfterSet);
  MachineInstr *TryTable =
      BuildMI(*BeginBB, TryTablePos, RangeBegin->getDebugLoc(),
              TII.get(WebAssembly::TRY_TABLE))
          .addImm(int64_t(WebAssembly::BlockType::Void))
          .addImm(1) // # of catch clauses
          .addImm(wasm::WASM_OPCODE_CATCH_ALL_REF)
          .addMBB(TrampolineBB);

  // Create a BB to insert the 'end_try_table' instruction.
  MachineBasicBlock *EndTryTableBB = MF.CreateMachineBasicBlock();
  EndTryTableBB->addSuccessor(TrampolineBB);

  auto SplitPos = std::next(RangeEnd->getIterator());
  if (SplitPos == EndBB->end()) {
    // If the range's end instruction is at the end of the BB, insert the new
```
- **EN**: Implements helper routine(s) `getInstrInfo`, `getTrampolineBlock`, `insert` for this portion of the WebAssembly backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `getInstrInfo`, `getTrampolineBlock`, `insert`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 1427-1464

```cpp
    // end_try_table BB after the current BB.
    MF.insert(std::next(EndBB->getIterator()), EndTryTableBB);
    EndBB->addSuccessor(EndTryTableBB);

  } else {
    // When the split pos is in the middle of a BB, we split the BB into two and
    // put the 'end_try_table' BB in between. We normally create a split BB and
    // make it a successor of the original BB (CatchAfterSplit == false), but in
    // case the BB is an EH pad and there is a 'catch' after split pos
    // (CatchAfterSplit == true), we should preserve the BB's property,
    // including that it is an EH pad, in the later part of the BB, where the
    // 'catch' is.
    bool CatchAfterSplit = false;
    if (EndBB->isEHPad()) {
      for (auto I = MachineBasicBlock::iterator(SplitPos), E = EndBB->end();
           I != E; ++I) {
        if (WebAssembly::isCatch(I->getOpcode())) {
          CatchAfterSplit = true;
          break;
        }
      }
    }

    MachineBasicBlock *PreBB = nullptr, *PostBB = nullptr;
    if (!CatchAfterSplit) {
      // If the range's end instruction is in the middle of the BB, we split the
      // BB into two and insert the end_try_table BB in between.
      // - Before:
      // bb:
      //   range_end
      //   other_insts
      //
      // - After:
      // pre_bb: (previous 'bb')
      //   range_end
      // end_try_table_bb: (new)
      //   end_try_table
      // post_bb: (new)
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "end_try_table BB after the current BB.". Notable symbols in this range include `insert`, `next`, `getIterator`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“end_try_table BB after the current BB.”。 该区间中较显眼的符号包括 `insert`, `next`, `getIterator`。

### Lines 1465-1502

```cpp
      //   other_insts
      PreBB = EndBB;
      PostBB = MF.CreateMachineBasicBlock();
      MF.insert(std::next(PreBB->getIterator()), PostBB);
      MF.insert(std::next(PreBB->getIterator()), EndTryTableBB);
      PostBB->splice(PostBB->end(), PreBB, SplitPos, PreBB->end());
      PostBB->transferSuccessors(PreBB);
    } else {
      // - Before:
      // ehpad:
      //   range_end
      //   catch
      //   ...
      //
      // - After:
      // pre_bb: (new)
      //   range_end
      // end_try_table_bb: (new)
      //   end_try_table
      // post_bb: (previous 'ehpad')
      //   catch
      //   ...
      assert(EndBB->isEHPad());
      PreBB = MF.CreateMachineBasicBlock();
      PostBB = EndBB;
      MF.insert(PostBB->getIterator(), PreBB);
      MF.insert(PostBB->getIterator(), EndTryTableBB);
      PreBB->splice(PreBB->end(), PostBB, PostBB->begin(), SplitPos);
      // We don't need to transfer predecessors of the EH pad to 'PreBB',
      // because an EH pad's predecessors are all through unwind edges and they
      // should still unwind to the EH pad, not PreBB.
    }
    unstackifyVRegsUsedInSplitBB(*PreBB, *PostBB);
    PreBB->addSuccessor(EndTryTableBB);
    PreBB->addSuccessor(PostBB);
  }

  // Add a 'end_try_table' instruction in the EndTryTable BB created above.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "other_insts". Notable symbols in this range include `CreateMachineBasicBlock`, `insert`, `next`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“other_insts”。 该区间中较显眼的符号包括 `CreateMachineBasicBlock`, `insert`, `next`。

### Lines 1503-1540

```cpp
  MachineInstr *EndTryTable = BuildMI(EndTryTableBB, RangeEnd->getDebugLoc(),
                                      TII.get(WebAssembly::END_TRY_TABLE));
  registerTryScope(TryTable, EndTryTable, TrampolineBB);
}

// In the standard (exnref) EH, we fix unwind mismatches by adding a new
// block~end_block inside of the unwind destination try_table~end_try_table:
// try_table ...
//   block exnref                   ;; (new)
//     ...
//     try_table (catch_all_ref N)  ;; (new) to trampoline BB
//       code
//     end_try_table                ;; (new)
//     ...
//   end_block                      ;; (new) trampoline BB
//   throw_ref                      ;; (new)
// end_try_table
//
// To do this, we will create a new BB that will contain the new 'end_block' and
// 'throw_ref' and insert it before the 'end_try_table' BB.
//
// But there are cases when there are 'end_loop'(s) before the 'end_try_table'
// in the same BB. (There can't be 'end_block' before 'end_try_table' in the
// same BB because EH pads can't be directly branched to.) Then after fixing
// unwind mismatches this will create the mismatching markers like below:
// bb0:
//   try_table
//   block exnref
//   ...
//   loop
//   ...
// new_bb:
//   end_block
// end_try_table_bb:
//   end_loop
//   end_try_table
//
// So if an end_try_table BB has an end_loop before the end_try_table, we split
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "In the standard (exnref) EH, we fix unwind mismatches by adding a new". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“In the standard (exnref) EH, we fix unwind mismatches by adding a new”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 1541-1578

```cpp
// the BB with the end_loop as a separate BB before the end_try_table BB, so
// that after we fix the unwind mismatch, the code will be like:
// bb0:
//   try_table
//   block exnref
//   ...
//   loop
//   ...
// end_loop_bb:
//   end_loop
// new_bb:
//   end_block
// end_try_table_bb:
//   end_try_table
static void splitEndLoopBB(MachineBasicBlock *EndTryTableBB) {
  auto &MF = *EndTryTableBB->getParent();
  MachineInstr *EndTryTable = nullptr, *EndLoop = nullptr;
  for (auto &MI : reverse(*EndTryTableBB)) {
    if (MI.getOpcode() == WebAssembly::END_TRY_TABLE) {
      EndTryTable = &MI;
      continue;
    }
    if (EndTryTable && MI.getOpcode() == WebAssembly::END_LOOP) {
      EndLoop = &MI;
      break;
    }
  }
  if (!EndLoop)
    return;

  auto *EndLoopBB = MF.CreateMachineBasicBlock();
  MF.insert(EndTryTableBB->getIterator(), EndLoopBB);
  auto SplitPos = std::next(EndLoop->getIterator());
  EndLoopBB->splice(EndLoopBB->end(), EndTryTableBB, EndTryTableBB->begin(),
                    SplitPos);
  EndLoopBB->addSuccessor(EndTryTableBB);
}
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "the BB with the end_loop as a separate BB before the end_try_table BB, so". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“the BB with the end_loop as a separate BB before the end_try_table BB, so”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 1579-1616

```cpp
// Print the BB name in the form of bb.NUMBER.ORIGINAL_NAME.
// e.g., bb.3.catch.start
[[maybe_unused]] static std::string getBBName(const MachineBasicBlock *MBB) {
  std::string Name = "bb.";
  Name += Twine(MBB->getNumber()).str();
  if (MBB->getBasicBlock()) {
    Name += ".";
    Name += MBB->getBasicBlock()->getName();
  }
  return Name;
}

bool WebAssemblyCFGStackify::fixCallUnwindMismatches(MachineFunction &MF) {
  // This function is used for both the legacy EH and the standard (exnref) EH,
  // and the reason we have unwind mismatches is the same for the both of them,
  // but the code examples in the comments are going to be different. To make
  // the description less confusing, we write the basically same comments twice,
  // once for the legacy EH and the standard EH.
  //
  // -- Legacy EH --------------------------------------------------------------
  //
  // Linearizing the control flow by placing TRY / END_TRY markers can create
  // mismatches in unwind destinations for throwing instructions, such as calls.
  //
  // We use the 'delegate' instruction to fix the unwind mismatches. 'delegate'
  // instruction delegates an exception to an outer 'catch'. It can target not
  // only 'catch' but all block-like structures including another 'delegate',
  // but with slightly different semantics than branches. When it targets a
  // 'catch', it will delegate the exception to that catch. It is being
  // discussed how to define the semantics when 'delegate''s target is a non-try
  // block: it will either be a validation failure or it will target the next
  // outer try-catch. But anyway our LLVM backend currently does not generate
  // such code. The example below illustrates where the 'delegate' instruction
  // in the middle will delegate the exception to, depending on the value of N.
  // try
  //   try
  //     block
  //       try
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Print the BB name in the form of bb.NUMBER.ORIGINAL_NAME.". Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Print the BB name in the form of bb.NUMBER.ORIGINAL_NAME.”。 该区间与栈帧布局或栈访问相关。

### Lines 1617-1654

```cpp
  //         try
  //           call @foo
  //         delegate N    ;; Where will this delegate to?
  //       catch           ;; N == 0
  //       end
  //     end               ;; N == 1 (invalid; will not be generated)
  //   delegate            ;; N == 2
  // catch                 ;; N == 3
  // end
  //                       ;; N == 4 (to caller)
  //
  // 1. When an instruction may throw, but the EH pad it will unwind to can be
  //    different from the original CFG.
  //
  // Example: we have the following CFG:
  // bb0:
  //   call @foo    ; if it throws, unwind to bb2
  // bb1:
  //   call @bar    ; if it throws, unwind to bb3
  // bb2 (ehpad):
  //   catch
  //   ...
  // bb3 (ehpad)
  //   catch
  //   ...
  //
  // And the CFG is sorted in this order. Then after placing TRY markers, it
  // will look like: (BB markers are omitted)
  // try
  //   try
  //     call @foo
  //     call @bar   ;; if it throws, unwind to bb3
  //   catch         ;; ehpad (bb2)
  //     ...
  //   end_try
  // catch           ;; ehpad (bb3)
  //   ...
  // end_try
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "try". Notable symbols in this range include `bb2`, `bb3`, `like:`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“try”。 该区间中较显眼的符号包括 `bb2`, `bb3`, `like:`。

### Lines 1655-1692

```cpp
  //
  // Now if bar() throws, it is going to end up in bb2, not bb3, where it is
  // supposed to end up. We solve this problem by wrapping the mismatching call
  // with an inner try-delegate that rethrows the exception to the right
  // 'catch'.
  //
  // try
  //   try
  //     call @foo
  //     try               ;; (new)
  //       call @bar
  //     delegate 1 (bb3)  ;; (new)
  //   catch               ;; ehpad (bb2)
  //     ...
  //   end_try
  // catch                 ;; ehpad (bb3)
  //   ...
  // end_try
  //
  // ---
  // 2. The same as 1, but in this case an instruction unwinds to a caller
  //    function and not another EH pad.
  //
  // Example: we have the following CFG:
  // bb0:
  //   call @foo       ; if it throws, unwind to bb2
  // bb1:
  //   call @bar       ; if it throws, unwind to caller
  // bb2 (ehpad):
  //   catch
  //   ...
  //
  // And the CFG is sorted in this order. Then after placing TRY markers, it
  // will look like:
  // try
  //   call @foo
  //   call @bar     ;; if it throws, unwind to caller
  // catch           ;; ehpad (bb2)
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Now if bar() throws, it is going to end up in bb2, not bb3, where it is". Notable symbols in this range include `bar`, `ehpad`, `bb2`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Now if bar() throws, it is going to end up in bb2, not bb3, where it is”。 该区间中较显眼的符号包括 `bar`, `ehpad`, `bb2`。

### Lines 1693-1730

```cpp
  //   ...
  // end_try
  //
  // Now if bar() throws, it is going to end up in bb2, when it is supposed
  // throw up to the caller. We solve this problem in the same way, but in this
  // case 'delegate's immediate argument is the number of block depths + 1,
  // which means it rethrows to the caller.
  // try
  //   call @foo
  //   try                  ;; (new)
  //     call @bar
  //   delegate 1 (caller)  ;; (new)
  // catch                  ;; ehpad (bb2)
  //   ...
  // end_try
  //
  // Before rewriteDepthImmediates, delegate's argument is a BB. In case of the
  // caller, it will take a fake BB generated by getFakeCallerBlock(), which
  // will be converted to a correct immediate argument later.
  //
  // In case there are multiple calls in a BB that may throw to the caller, they
  // can be wrapped together in one nested try-delegate scope. (In 1, this
  // couldn't happen, because may-throwing instruction there had an unwind
  // destination, i.e., it was an invoke before, and there could be only one
  // invoke within a BB.)
  //
  // -- Standard EH ------------------------------------------------------------
  //
  // Linearizing the control flow by placing TRY / END_TRY_TABLE markers can
  // create mismatches in unwind destinations for throwing instructions, such as
  // calls.
  //
  // We use the a nested 'try_table'~'end_try_table' instruction to fix the
  // unwind mismatches. try_table's catch clauses take an immediate argument
  // that specifics which block we should branch to.
  //
  // 1. When an instruction may throw, but the EH pad it will unwind to can be
  //    different from the original CFG.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "...". Notable symbols in this range include `bar`, `ehpad`, `getFakeCallerBlock`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“...”。 该区间中较显眼的符号包括 `bar`, `ehpad`, `getFakeCallerBlock`。

### Lines 1731-1768

```cpp
  //
  // Example: we have the following CFG:
  // bb0:
  //   call @foo    ; if it throws, unwind to bb2
  // bb1:
  //   call @bar    ; if it throws, unwind to bb3
  // bb2 (ehpad):
  //   catch
  //   ...
  // bb3 (ehpad)
  //   catch
  //   ...
  //
  // And the CFG is sorted in this order. Then after placing TRY_TABLE markers
  // (and BLOCK markers for the TRY_TABLE's destinations), it will look like:
  // (BB markers are omitted)
  // block
  //   try_table (catch ... 0)
  //     block
  //       try_table (catch ... 0)
  //         call @foo
  //         call @bar              ;; if it throws, unwind to bb3
  //       end_try_table
  //     end_block                  ;; ehpad (bb2)
  //     ...
  //   end_try_table
  // end_block                      ;; ehpad (bb3)
  // ...
  //
  // Now if bar() throws, it is going to end up in bb2, not bb3, where it is
  // supposed to end up. We solve this problem by wrapping the mismatching call
  // with an inner try_table~end_try_table that sends the exception to the the
  // 'trampoline' block, which rethrows, or 'bounces' it to the right
  // end_try_table:
  // block
  //   try_table (catch ... 0)
  //     block exnref                       ;; (new)
  //       block
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Example: we have the following CFG:". Notable symbols in this range include `bb2`, `bb3`, `try_table`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Example: we have the following CFG:”。 该区间中较显眼的符号包括 `bb2`, `bb3`, `try_table`。

### Lines 1769-1806

```cpp
  //         try_table (catch ... 0)
  //           call @foo
  //           try_table (catch_all_ref 2)  ;; (new) to trampoline BB
  //             call @bar
  //           end_try_table                ;; (new)
  //         end_try_table
  //       end_block                        ;; ehpad (bb2)
  //       ...
  //     end_block                          ;; (new) trampoline BB
  //     throw_ref                          ;; (new)
  //   end_try_table
  // end_block                              ;; ehpad (bb3)
  //
  // ---
  // 2. The same as 1, but in this case an instruction unwinds to a caller
  //    function and not another EH pad.
  //
  // Example: we have the following CFG:
  // bb0:
  //   call @foo       ; if it throws, unwind to bb2
  // bb1:
  //   call @bar       ; if it throws, unwind to caller
  // bb2 (ehpad):
  //   catch
  //   ...
  //
  // And the CFG is sorted in this order. Then after placing TRY_TABLE markers
  // (and BLOCK markers for the TRY_TABLE's destinations), it will look like:
  // block
  //   try_table (catch ... 0)
  //     call @foo
  //     call @bar              ;; if it throws, unwind to caller
  //   end_try_table
  // end_block                  ;; ehpad (bb2)
  // ...
  //
  // Now if bar() throws, it is going to end up in bb2, when it is supposed
  // throw up to the caller. We solve this problem in the same way, but in this
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "try_table (catch ... 0)". Notable symbols in this range include `try_table`, `ehpad`, `bb2`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“try_table (catch ... 0)”。 该区间中较显眼的符号包括 `try_table`, `ehpad`, `bb2`。

### Lines 1807-1844

```cpp
  // case 'catch_all_ref's immediate argument is the number of block depths + 1,
  // which means it rethrows to the caller.
  // block exnref                       ;; (new)
  //   block
  //     try_table (catch ... 0)
  //       call @foo
  //       try_table (catch_all_ref 2)  ;; (new) to trampoline BB
  //         call @bar
  //       end_try_table                ;; (new)
  //     end_try_table
  //   end_block                        ;; ehpad (bb2)
  //   ...
  // end_block                          ;; (new) caller trampoline BB
  // throw_ref                          ;; (new) throw to the caller
  //
  // Before rewriteDepthImmediates, try_table's catch clauses' argument is a
  // trampoline BB from which we throw_ref the exception to the right
  // end_try_table. In case of the caller, it will take a new caller-dedicated
  // trampoline BB generated by getCallerTrampolineBlock(), which throws the
  // exception to the caller.
  //
  // In case there are multiple calls in a BB that may throw to the caller, they
  // can be wrapped together in one nested try_table-end_try_table scope. (In 1,
  // this couldn't happen, because may-throwing instruction there had an unwind
  // destination, i.e., it was an invoke before, and there could be only one
  // invoke within a BB.)

  SmallVector<const MachineBasicBlock *, 8> EHPadStack;
  // Range of intructions to be wrapped in a new nested try~delegate or
  // try_table~end_try_table. A range exists in a single BB and does not span
  // multiple BBs.
  using TryRange = std::pair<MachineInstr *, MachineInstr *>;
  // In original CFG, <unwind destination BB, a vector of try/try_table ranges>
  DenseMap<MachineBasicBlock *, SmallVector<TryRange, 4>> UnwindDestToTryRanges;

  // Gather possibly throwing calls (i.e., previously invokes) whose current
  // unwind destination is not the same as the original CFG. (Case 1)
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "case 'catch_all_ref's immediate argument is the number of block depths + 1,". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“case 'catch_all_ref's immediate argument is the number of block depths + 1,”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 1845-1882

```cpp
  for (auto &MBB : reverse(MF)) {
    bool SeenThrowableInstInBB = false;
    for (auto &MI : reverse(MBB)) {
      if (WebAssembly::isTry(MI.getOpcode()))
        EHPadStack.pop_back();
      else if (MI.getOpcode() == WebAssembly::DELEGATE)
        EHPadStack.push_back(MI.getOperand(0).getMBB());
      else if (WebAssembly::WasmUseLegacyEH &&
               WebAssembly::isCatch(MI.getOpcode()))
        EHPadStack.push_back(MI.getParent());
      else if (MI.getOpcode() == WebAssembly::END_TRY_TABLE)
        // In case of the legacy EH, 'catch' instruction is always an EH pad for
        // the 'try' body that precedes it. But in the standard EH, because
        // fixCatchUnwindMismatches runs before this, a new try_table's
        // trampoline BB will be separated from try_table ~ end_try_table body:
        //
        // bb0:
        //   try_table (catch_all_ref %far_away_trampoline)
        //     ...
        //   end_try_table
        // ...
        // far_away_trampoline:
        //   catch_all_ref
        //   throw_ref
        //
        // And there can be multiple try_tables that target a single trampoline:
        //
        // bb0:
        //   try_table (catch_all_ref %far_away_trampolinle_bb)
        //     ...
        //   end_try_table
        // ...
        // bb1:
        //   try_table (catch_all_ref %far_away_trampolinle_bb)
        //     ...
        //   end_try_table
        // ...
        // far_away_trampoline:
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "In case of the legacy EH, 'catch' instruction is always an EH pad for". Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“In case of the legacy EH, 'catch' instruction is always an EH pad for”。 该区间与栈帧布局或栈访问相关。

### Lines 1883-1920

```cpp
        //   catch_all_ref
        //   throw_ref
        //
        // So we can't call WebAssembly::isCatch to add its parent EH pad to
        // EHPadStack. Now we add to EHPadStack at end_try_table marker, by
        // getting its matching try_table's destination. This works when the
        // destination EH pad is either a normal EH pad or a trampoline created
        // in fixCatchUnwindMismatches.
        //
        // Note that we don't need to distinguish this case in
        // fixCatchUnwindMismatches because it runs before
        // fixCallUnwindMismatches and there is no new try_tables and
        // trampolines when it runs.
        EHPadStack.push_back(TryToEHPad[EndToBegin[&MI]]);

      // In this loop we only gather calls that have an EH pad to unwind. So
      // there will be at most 1 such call (= invoke) in a BB, so after we've
      // seen one, we can skip the rest of BB. Also if MBB has no EH pad
      // successor or MI does not throw, this is not an invoke.
      if (SeenThrowableInstInBB || !MBB.hasEHPadSuccessor() ||
          !WebAssembly::mayThrow(MI))
        continue;
      SeenThrowableInstInBB = true;

      // If the EH pad on the stack top is where this instruction should unwind
      // next, we're good.
      MachineBasicBlock *UnwindDest = nullptr;
      for (auto *Succ : MBB.successors()) {
        // Even though semantically a BB can have multiple successors in case an
        // exception is not caught by a catchpad, the first unwind destination
        // should appear first in the successor list, based on the calculation
        // in findUnwindDestinations() in SelectionDAGBuilder.cpp.
        if (Succ->isEHPad()) {
          UnwindDest = Succ;
          break;
        }
      }
      if (EHPadStack.back() == UnwindDest)
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "catch_all_ref". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“catch_all_ref”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 1921-1958

```cpp
        continue;

      // Include EH_LABELs in the range before and after the invoke
      MachineInstr *RangeBegin = &MI, *RangeEnd = &MI;
      if (RangeBegin->getIterator() != MBB.begin() &&
          std::prev(RangeBegin->getIterator())->isEHLabel())
        RangeBegin = &*std::prev(RangeBegin->getIterator());
      if (std::next(RangeEnd->getIterator()) != MBB.end() &&
          std::next(RangeEnd->getIterator())->isEHLabel())
        RangeEnd = &*std::next(RangeEnd->getIterator());

      // If not, record the range.
      UnwindDestToTryRanges[UnwindDest].push_back(
          TryRange(RangeBegin, RangeEnd));
      LLVM_DEBUG(dbgs() << "- Call unwind mismatch: MBB = " << getBBName(&MBB)
                        << "\nCall = " << MI
                        << "\nOriginal dest = " << getBBName(UnwindDest)
                        << "  Current dest = " << getBBName(EHPadStack.back())
                        << "\n\n");
    }
  }

  assert(EHPadStack.empty());

  // Gather possibly throwing calls that are supposed to unwind up to the caller
  // if they throw, but currently unwind to an incorrect destination. Unlike the
  // loop above, there can be multiple calls within a BB that unwind to the
  // caller, which we should group together in a range. (Case 2)

  MachineInstr *RangeBegin = nullptr, *RangeEnd = nullptr; // inclusive

  // Record the range.
  auto RecordCallerMismatchRange = [&](const MachineBasicBlock *CurrentDest) {
    UnwindDestToTryRanges[getFakeCallerBlock(MF)].push_back(
        TryRange(RangeBegin, RangeEnd));
    LLVM_DEBUG(dbgs() << "- Call unwind mismatch: MBB = "
                      << getBBName(RangeBegin->getParent())
                      << "\nRange begin = " << *RangeBegin
```
- **EN**: Implements helper routine(s) `getIterator`, `begin`, `prev` for this portion of the WebAssembly backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `getIterator`, `begin`, `prev`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 1959-1996

```cpp
                      << "Range end = " << *RangeEnd
                      << "\nOriginal dest = caller  Current dest = "
                      << getBBName(CurrentDest) << "\n\n");
    RangeBegin = RangeEnd = nullptr; // Reset range pointers
  };

  for (auto &MBB : reverse(MF)) {
    bool SeenThrowableInstInBB = false;
    for (auto &MI : reverse(MBB)) {
      bool MayThrow = WebAssembly::mayThrow(MI);

      // If MBB has an EH pad successor and this is the last instruction that
      // may throw, this instruction unwinds to the EH pad and not to the
      // caller.
      if (MBB.hasEHPadSuccessor() && MayThrow && !SeenThrowableInstInBB)
        SeenThrowableInstInBB = true;

      // We wrap up the current range when we see a marker even if we haven't
      // finished a BB.
      else if (RangeEnd && WebAssembly::isMarker(MI.getOpcode()))
        RecordCallerMismatchRange(EHPadStack.back());

      // If EHPadStack is empty, that means it correctly unwinds to the caller
      // if it throws, so we're good. A delegate targeting FakeCallerBB also
      // correctly unwinds to the caller. If MI does not throw, we're good too.
      else if (EHPadStack.empty() || EHPadStack.back() == FakeCallerBB ||
               !MayThrow) {
      }

      // We found an instruction that unwinds to the caller but currently has an
      // incorrect unwind destination. Create a new range or increment the
      // currently existing range.
      else {
        if (!RangeEnd)
          RangeBegin = RangeEnd = &MI;
        else
          RangeBegin = &MI;
      }
```
- **EN**: Implements helper routine(s) `getBBName`, `reverse`, `mayThrow` for this portion of the WebAssembly backend backend implementation logic. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `getBBName`, `reverse`, `mayThrow`。 该区间与栈帧布局或栈访问相关。

### Lines 1997-2034

```cpp

      // Update EHPadStack.
      if (WebAssembly::isTry(MI.getOpcode()))
        EHPadStack.pop_back();
      else if (MI.getOpcode() == WebAssembly::DELEGATE)
        EHPadStack.push_back(MI.getOperand(0).getMBB());
      else if (WebAssembly::WasmUseLegacyEH &&
               WebAssembly::isCatch(MI.getOpcode()))
        EHPadStack.push_back(MI.getParent());
      else if (!WebAssembly::WasmUseLegacyEH &&
               MI.getOpcode() == WebAssembly::END_TRY_TABLE)
        EHPadStack.push_back(TryToEHPad[EndToBegin[&MI]]);
    }

    if (RangeEnd)
      RecordCallerMismatchRange(EHPadStack.back());
  }

  assert(EHPadStack.empty());

  // We don't have any unwind destination mismatches to resolve.
  if (UnwindDestToTryRanges.empty())
    return false;

  // When end_loop is before end_try_table within the same BB in unwind
  // destinations, we should split the end_loop into another BB.
  if (!WebAssembly::WasmUseLegacyEH)
    for (auto &[UnwindDest, _] : UnwindDestToTryRanges) {
      auto It = EHPadToTry.find(UnwindDest);
      // If UnwindDest is the fake caller block, it will not be in EHPadToTry
      // map
      if (It != EHPadToTry.end()) {
        auto *TryTable = It->second;
        auto *EndTryTable = BeginToEnd[TryTable];
        splitEndLoopBB(EndTryTable->getParent());
      }
    }
```
- **EN**: Implements helper routine(s) `isTry`, `getOpcode`, `pop_back` for this portion of the WebAssembly backend backend implementation logic. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `isTry`, `getOpcode`, `pop_back`。 该区间与栈帧布局或栈访问相关。

### Lines 2035-2072

```cpp
  // Now we fix the mismatches by wrapping calls with inner try-delegates.
  for (auto &P : UnwindDestToTryRanges) {
    NumCallUnwindMismatches += P.second.size();
    MachineBasicBlock *UnwindDest = P.first;
    auto &TryRanges = P.second;

    for (auto Range : TryRanges) {
      MachineInstr *RangeBegin = nullptr, *RangeEnd = nullptr;
      std::tie(RangeBegin, RangeEnd) = Range;
      auto *MBB = RangeBegin->getParent();

      // If this BB has an EH pad successor, i.e., ends with an 'invoke', and if
      // the current range contains the invoke, now we are going to wrap the
      // invoke with try-delegate or try_table-end_try_table, making the
      // 'delegate' or 'end_try_table' BB the new successor instead, so remove
      // the EH pad succesor here. The BB may not have an EH pad successor if
      // calls in this BB throw to the caller.
      if (UnwindDest != getFakeCallerBlock(MF)) {
        MachineBasicBlock *EHPad = nullptr;
        for (auto *Succ : MBB->successors()) {
          if (Succ->isEHPad()) {
            EHPad = Succ;
            break;
          }
        }
        if (EHPad)
          MBB->removeSuccessor(EHPad);
      }

      if (WebAssembly::WasmUseLegacyEH)
        addNestedTryDelegate(RangeBegin, RangeEnd, UnwindDest);
      else
        addNestedTryTable(RangeBegin, RangeEnd, UnwindDest);
    }
  }

  return true;
}
```
- **EN**: Implements helper routine(s) `size`, `tie`, `getParent` for this portion of the WebAssembly backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `size`, `tie`, `getParent`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 2073-2110

```cpp

bool WebAssemblyCFGStackify::fixCatchUnwindMismatches(MachineFunction &MF) {
  // This function is used for both the legacy EH and the standard (exnref) EH,
  // and the reason we have unwind mismatches is the same for the both of them,
  // but the code examples in the comments are going to be different. To make
  // the description less confusing, we write the basically same comments twice,
  // once for the legacy EH and the standard EH.
  //
  // -- Legacy EH --------------------------------------------------------------
  //
  // There is another kind of unwind destination mismatches besides call unwind
  // mismatches, which we will call "catch unwind mismatches". See this example
  // after the marker placement:
  // try
  //   try
  //     call @foo
  //   catch __cpp_exception  ;; ehpad A (next unwind dest: caller)
  //     ...
  //   end_try
  // catch_all                ;; ehpad B
  //   ...
  // end_try
  //
  // 'call @foo's unwind destination is the ehpad A. But suppose 'call @foo'
  // throws a foreign exception that is not caught by ehpad A, and its next
  // destination should be the caller. But after control flow linearization,
  // another EH pad can be placed in between (e.g. ehpad B here), making the
  // next unwind destination incorrect. In this case, the foreign exception will
  // instead go to ehpad B and will be caught there instead. In this example the
  // correct next unwind destination is the caller, but it can be another outer
  // catch in other cases.
  //
  // There is no specific 'call' or 'throw' instruction to wrap with a
  // try-delegate, so we wrap the whole try-catch-end with a try-delegate and
  // make it rethrow to the right destination, which is the caller in the
  // example below:
  // try
  //   try                     ;; (new)
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This function is used for both the legacy EH and the standard (exnref) EH,". Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This function is used for both the legacy EH and the standard (exnref) EH,”。 该区间与栈帧布局或栈访问相关。

### Lines 2111-2148

```cpp
  //     try
  //       call @foo
  //     catch __cpp_exception ;; ehpad A (next unwind dest: caller)
  //       ...
  //     end_try
  //   delegate 1 (caller)     ;; (new)
  // catch_all                 ;; ehpad B
  //   ...
  // end_try
  //
  // The right destination may be another EH pad or the caller. (The example
  // here shows the case it is the caller.)
  //
  // -- Standard EH ------------------------------------------------------------
  //
  // There is another kind of unwind destination mismatches besides call unwind
  // mismatches, which we will call "catch unwind mismatches". See this example
  // after the marker placement:
  // block
  //   try_table (catch_all_ref 0)
  //     block
  //       try_table (catch ... 0)
  //         call @foo
  //       end_try_table
  //     end_block                  ;; ehpad A (next unwind dest: caller)
  //     ...
  //   end_try_table
  // end_block                      ;; ehpad B
  // ...
  //
  // 'call @foo's unwind destination is the ehpad A. But suppose 'call @foo'
  // throws a foreign exception that is not caught by ehpad A, and its next
  // destination should be the caller. But after control flow linearization,
  // another EH pad can be placed in between (e.g. ehpad B here), making the
  // next unwind destination incorrect. In this case, the foreign exception will
  // instead go to ehpad B and will be caught there instead. In this example the
  // correct next unwind destination is the caller, but it can be another outer
  // catch in other cases.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "try". Notable symbols in this range include `A`, `try_table`, `between`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“try”。 该区间中较显眼的符号包括 `A`, `try_table`, `between`。

### Lines 2149-2186

```cpp
  //
  // There is no specific 'call' or 'throw' instruction to wrap with an inner
  // try_table-end_try_table, so we wrap the whole try_table-end_try_table with
  // an inner try_table-end_try_table that sends the exception to a trampoline
  // BB. We rethrow the sent exception using a throw_ref to the right
  // destination, which is the caller in the example below:
  // block exnref
  //   block
  //     try_table (catch_all_ref 0)
  //       try_table (catch_all_ref 2)  ;; (new) to trampoline
  //         block
  //           try_table (catch ... 0)
  //             call @foo
  //           end_try_table
  //         end_block                  ;; ehpad A (next unwind dest: caller)
  //       end_try_table                ;; (new)
  //       ...
  //     end_try_table
  //   end_block                        ;; ehpad B
  //   ...
  // end_block                          ;; (new) caller trampoline BB
  // throw_ref                          ;; (new) throw to the caller
  //
  // The right destination may be another EH pad or the caller. (The example
  // here shows the case it is the caller.)

  // Returns whether the next unwind destination exists when an exception is not
  // caught by the given EHPad. It is guaranteed that the next successor of the
  // given EHPad's predecessor is the next unwind destination, due to the order
  // we add successors in findUnwindDestinations in SelectionDAGBuilder.
  auto HasUnwindDest = [&](const MachineBasicBlock *EHPad) {
    assert(!EHPad->pred_empty() && "EHPad has no predecessors");
    auto *InvokeBB = *EHPad->pred_begin();
    for (auto I = InvokeBB->succ_begin(), E = InvokeBB->succ_end(); I != E; ++I)
      if (*I == EHPad)
        return std::next(I) != E;
    llvm_unreachable("EHPad not found in its predecessor's successors");
  };
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "There is no specific 'call' or 'throw' instruction to wrap with an inner". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“There is no specific 'call' or 'throw' instruction to wrap with an inner”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 2187-2224

```cpp

  // Returns the next unwind destination when an exception is not caught by the
  // given EHPad. Returns nullptr when it doesn't exist.
  auto GetUnwindDest = [&](const MachineBasicBlock *EHPad) {
    assert(!EHPad->pred_empty() && "EHPad has no predecessors");
    auto *InvokeBB = *EHPad->pred_begin();
    for (auto I = InvokeBB->succ_begin(), E = InvokeBB->succ_end(); I != E;
         ++I) {
      if (*I == EHPad) {
        auto *Next = std::next(I);
        return Next == E ? nullptr : *Next;
      }
    }
    llvm_unreachable("EHPad not found in its predecessor's successors");
  };

  SmallVector<const MachineBasicBlock *, 8> EHPadStack;
  // For EH pads that have catch unwind mismatches, a map of <EH pad, its
  // correct unwind destination>.
  DenseMap<MachineBasicBlock *, MachineBasicBlock *> EHPadToUnwindDest;

  for (auto &MBB : reverse(MF)) {
    for (auto &MI : reverse(MBB)) {
      if (WebAssembly::isTry(MI.getOpcode())) {
        EHPadStack.pop_back();
      } else if (MI.getOpcode() == WebAssembly::DELEGATE) {
        EHPadStack.push_back(&MBB);
      } else if (WebAssembly::isCatch(MI.getOpcode())) {
        auto *EHPad = &MBB;

        // catch_all always catches an exception, so we don't need to do
        // anything
        if (WebAssembly::isCatchAll(MI.getOpcode())) {
        }

        // This can happen when the unwind dest was removed during the
        // optimization, e.g. because it was unreachable.
        else if (EHPadStack.empty() && HasUnwindDest(EHPad)) {
```
- **EN**: Implements helper routine(s) `pred_empty`, `pred_begin`, `succ_begin` for this portion of the WebAssembly backend backend implementation logic. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `pred_empty`, `pred_begin`, `succ_begin`。 该区间与栈帧布局或栈访问相关。

### Lines 2225-2262

```cpp
          LLVM_DEBUG(dbgs() << "EHPad (" << getBBName(EHPad)
                            << "'s unwind destination does not exist anymore"
                            << "\n\n");
        }

        // The EHPad's next unwind destination is the caller, but we incorrectly
        // unwind to another EH pad.
        else if (!EHPadStack.empty() && EHPadStack.back() != FakeCallerBB &&
                 !HasUnwindDest(EHPad)) {
          EHPadToUnwindDest[EHPad] = getFakeCallerBlock(MF);
          LLVM_DEBUG(dbgs()
                     << "- Catch unwind mismatch:\nEHPad = " << getBBName(EHPad)
                     << "  Original dest = caller  Current dest = "
                     << getBBName(EHPadStack.back()) << "\n\n");
        }

        // The EHPad's next unwind destination is an EH pad, whereas we
        // incorrectly unwind to another EH pad.
        else if (!EHPadStack.empty() && HasUnwindDest(EHPad)) {
          auto *UnwindDest = GetUnwindDest(EHPad);
          if (EHPadStack.back() != UnwindDest) {
            EHPadToUnwindDest[EHPad] = UnwindDest;
            LLVM_DEBUG(dbgs() << "- Catch unwind mismatch:\nEHPad = "
                              << getBBName(EHPad) << "  Original dest = "
                              << getBBName(UnwindDest) << "  Current dest = "
                              << getBBName(EHPadStack.back()) << "\n\n");
          }
        }

        EHPadStack.push_back(EHPad);
      }
    }
  }

  assert(EHPadStack.empty());
  if (EHPadToUnwindDest.empty())
    return false;
```
- **EN**: Implements helper routine(s) `dbgs`, `EHPad`, `getBBName` for this portion of the WebAssembly backend backend implementation logic. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `dbgs`, `EHPad`, `getBBName`。 该区间与栈帧布局或栈访问相关。

### Lines 2263-2300

```cpp
  // When end_loop is before end_try_table within the same BB in unwind
  // destinations, we should split the end_loop into another BB.
  for (auto &[_, UnwindDest] : EHPadToUnwindDest) {
    auto It = EHPadToTry.find(UnwindDest);
    // If UnwindDest is the fake caller block, it will not be in EHPadToTry map
    if (It != EHPadToTry.end()) {
      auto *TryTable = It->second;
      auto *EndTryTable = BeginToEnd[TryTable];
      splitEndLoopBB(EndTryTable->getParent());
    }
  }

  NumCatchUnwindMismatches += EHPadToUnwindDest.size();
  SmallPtrSet<MachineBasicBlock *, 4> NewEndTryBBs;

  for (auto &[EHPad, UnwindDest] : EHPadToUnwindDest) {
    MachineInstr *Try = EHPadToTry[EHPad];
    MachineInstr *EndTry = BeginToEnd[Try];
    if (WebAssembly::WasmUseLegacyEH) {
      addNestedTryDelegate(Try, EndTry, UnwindDest);
      NewEndTryBBs.insert(EndTry->getParent());
    } else {
      addNestedTryTable(Try, EndTry, UnwindDest);
    }
  }

  if (!WebAssembly::WasmUseLegacyEH)
    return true;

  // Adding a try-delegate wrapping an existing try-catch-end can make existing
  // branch destination BBs invalid. For example,
  //
  // - Before:
  // bb0:
  //   block
  //     br bb3
  // bb1:
  //     try
```
- **EN**: Implements helper routine(s) `find`, `end`, `splitEndLoopBB` for this portion of the WebAssembly backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `find`, `end`, `splitEndLoopBB`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 2301-2338

```cpp
  //       ...
  // bb2: (ehpad)
  //     catch
  // bb3:
  //     end_try
  //   end_block   ;; 'br bb3' targets here
  //
  // Suppose this try-catch-end has a catch unwind mismatch, so we need to wrap
  // this with a try-delegate. Then this becomes:
  //
  // - After:
  // bb0:
  //   block
  //     br bb3    ;; invalid destination!
  // bb1:
  //     try       ;; (new instruction)
  //       try
  //         ...
  // bb2: (ehpad)
  //       catch
  // bb3:
  //       end_try ;; 'br bb3' still incorrectly targets here!
  // delegate_bb:  ;; (new BB)
  //     delegate  ;; (new instruction)
  // split_bb:     ;; (new BB)
  //   end_block
  //
  // Now 'br bb3' incorrectly branches to an inner scope.
  //
  // As we can see in this case, when branches target a BB that has both
  // 'end_try' and 'end_block' and the BB is split to insert a 'delegate', we
  // have to remap existing branch destinations so that they target not the
  // 'end_try' BB but the new 'end_block' BB. There can be multiple 'delegate's
  // in between, so we try to find the next BB with 'end_block' instruction. In
  // this example, the 'br bb3' instruction should be remapped to 'br split_bb'.
  for (auto &MBB : MF) {
    for (auto &MI : MBB) {
      if (MI.isTerminator()) {
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "...". Notable symbols in this range include `bb2:`, `isTerminator`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“...”。 该区间中较显眼的符号包括 `bb2:`, `isTerminator`。

### Lines 2339-2376

```cpp
        for (auto &MO : MI.operands()) {
          if (MO.isMBB() && NewEndTryBBs.count(MO.getMBB())) {
            auto *BrDest = MO.getMBB();
            bool FoundEndBlock = false;
            for (; std::next(BrDest->getIterator()) != MF.end();
                 BrDest = BrDest->getNextNode()) {
              for (const auto &MI : *BrDest) {
                if (MI.getOpcode() == WebAssembly::END_BLOCK) {
                  FoundEndBlock = true;
                  break;
                }
              }
              if (FoundEndBlock)
                break;
            }
            assert(FoundEndBlock);
            MO.setMBB(BrDest);
          }
        }
      }
    }
  }

  return true;
}

void WebAssemblyCFGStackify::recalculateScopeTops(MachineFunction &MF) {
  // Renumber BBs and recalculate ScopeTop info because new BBs might have been
  // created and inserted during fixing unwind mismatches.
  MF.RenumberBlocks();
  ScopeTops.clear();
  ScopeTops.resize(MF.getNumBlockIDs());
  for (auto &MBB : reverse(MF)) {
    for (auto &MI : reverse(MBB)) {
      if (ScopeTops[MBB.getNumber()])
        break;
      switch (MI.getOpcode()) {
      case WebAssembly::END_BLOCK:
```
- **EN**: Implements helper routine(s) `operands`, `isMBB`, `count` for this portion of the WebAssembly backend backend implementation logic. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `operands`, `isMBB`, `count`。 该区间与栈帧布局或栈访问相关。

### Lines 2377-2414

```cpp
      case WebAssembly::END_LOOP:
      case WebAssembly::END_TRY:
      case WebAssembly::END_TRY_TABLE:
      case WebAssembly::DELEGATE:
        updateScopeTops(EndToBegin[&MI]->getParent(), &MBB);
        break;
      case WebAssembly::CATCH_LEGACY:
      case WebAssembly::CATCH_ALL_LEGACY:
        updateScopeTops(EHPadToTry[&MBB]->getParent(), &MBB);
        break;
      }
    }
  }
}

/// In normal assembly languages, when the end of a function is unreachable,
/// because the function ends in an infinite loop or a noreturn call or similar,
/// it isn't necessary to worry about the function return type at the end of
/// the function, because it's never reached. However, in WebAssembly, blocks
/// that end at the function end need to have a return type signature that
/// matches the function signature, even though it's unreachable. This function
/// checks for such cases and fixes up the signatures.
void WebAssemblyCFGStackify::fixEndsAtEndOfFunction(MachineFunction &MF) {
  const auto &MFI = *MF.getInfo<WebAssemblyFunctionInfo>();

  if (MFI.getResults().empty())
    return;

  // MCInstLower will add the proper types to multivalue signatures based on the
  // function return type
  WebAssembly::BlockType RetType =
      MFI.getResults().size() > 1
          ? WebAssembly::BlockType::Multivalue
          : WebAssembly::BlockType(
                WebAssembly::toValType(MFI.getResults().front()));

  SmallVector<MachineBasicBlock::reverse_iterator, 4> Worklist;
  Worklist.push_back(MF.rbegin()->rbegin());
```
- **EN**: Implements helper routine(s) `updateScopeTops`, `getParent`, `fixEndsAtEndOfFunction` for this portion of the WebAssembly backend backend implementation logic. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `updateScopeTops`, `getParent`, `fixEndsAtEndOfFunction`。 该区间与栈帧布局或栈访问相关。

### Lines 2415-2452

```cpp

  auto Process = [&](MachineBasicBlock::reverse_iterator It) {
    auto *MBB = It->getParent();
    while (It != MBB->rend()) {
      MachineInstr &MI = *It++;
      if (MI.isPosition() || MI.isDebugInstr())
        continue;
      switch (MI.getOpcode()) {
      case WebAssembly::END_TRY: {
        // If a 'try''s return type is fixed, both its try body and catch body
        // should satisfy the return type, so we need to search 'end'
        // instructions before its corresponding 'catch' too.
        auto *EHPad = TryToEHPad.lookup(EndToBegin[&MI]);
        assert(EHPad);
        auto NextIt =
            std::next(WebAssembly::findCatch(EHPad)->getReverseIterator());
        if (NextIt != EHPad->rend())
          Worklist.push_back(NextIt);
        [[fallthrough]];
      }
      case WebAssembly::END_BLOCK:
      case WebAssembly::END_LOOP:
      case WebAssembly::END_TRY_TABLE:
      case WebAssembly::DELEGATE:
        EndToBegin[&MI]->getOperand(0).setImm(int32_t(RetType));
        continue;
      default:
        // Something other than an `end`. We're done for this BB.
        return;
      }
    }
    // We've reached the beginning of a BB. Continue the search in the previous
    // BB.
    Worklist.push_back(MBB->getPrevNode()->rbegin());
  };

  while (!Worklist.empty())
    Process(Worklist.pop_back_val());
```
- **EN**: Implements helper routine(s) `getParent`, `rend`, `isPosition` for this portion of the WebAssembly backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `getParent`, `rend`, `isPosition`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 2453-2490

```cpp
}

// WebAssembly functions end with an end instruction, as if the function body
// were a block.
static void appendEndToFunction(MachineFunction &MF,
                                const WebAssemblyInstrInfo &TII) {
  BuildMI(MF.back(), MF.back().end(),
          MF.back().findPrevDebugLoc(MF.back().end()),
          TII.get(WebAssembly::END_FUNCTION));
}

// We added block~end_block and try_table~end_try_table markers in
// placeTryTableMarker. But When catch clause's destination has a return type,
// as in the case of catch with a concrete tag, catch_ref, and catch_all_ref.
// For example:
// block exnref
//   try_table (catch_all_ref 0)
//     ...
//   end_try_table
// end_block
// ... use exnref ...
//
// This code is not valid because the block's body type is not exnref. So we add
// an unreachable after the 'end_try_table' to make the code valid here:
// block exnref
//   try_table (catch_all_ref 0)
//     ...
//   end_try_table
//   unreachable      (new)
// end_block
//
// Because 'unreachable' is a terminator we also need to split the BB.
static void addUnreachableAfterTryTables(MachineFunction &MF,
                                         const WebAssemblyInstrInfo &TII) {
  std::vector<MachineInstr *> EndTryTables;
  for (auto &MBB : MF)
    for (auto &MI : MBB)
      if (MI.getOpcode() == WebAssembly::END_TRY_TABLE)
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "WebAssembly functions end with an end instruction, as if the function body". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“WebAssembly functions end with an end instruction, as if the function body”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 2491-2528

```cpp
        EndTryTables.push_back(&MI);

  for (auto *EndTryTable : EndTryTables) {
    auto *MBB = EndTryTable->getParent();
    auto *NewEndTryTableBB = MF.CreateMachineBasicBlock();
    MF.insert(MBB->getIterator(), NewEndTryTableBB);
    auto SplitPos = std::next(EndTryTable->getIterator());
    NewEndTryTableBB->splice(NewEndTryTableBB->end(), MBB, MBB->begin(),
                             SplitPos);
    NewEndTryTableBB->addSuccessor(MBB);
    BuildMI(NewEndTryTableBB, EndTryTable->getDebugLoc(),
            TII.get(WebAssembly::UNREACHABLE));
  }
}

/// Insert BLOCK/LOOP/TRY/TRY_TABLE markers at appropriate places.
void WebAssemblyCFGStackify::placeMarkers(MachineFunction &MF) {
  // We allocate one more than the number of blocks in the function to
  // accommodate for the possible fake block we may insert at the end.
  ScopeTops.resize(MF.getNumBlockIDs() + 1);
  // Place the LOOP for MBB if MBB is the header of a loop.
  for (auto &MBB : MF)
    placeLoopMarker(MBB);

  const MCAsmInfo &MCAI = MF.getTarget().getMCAsmInfo();
  for (auto &MBB : MF) {
    if (MBB.isEHPad()) {
      // Place the TRY/TRY_TABLE for MBB if MBB is the EH pad of an exception.
      if (MCAI.getExceptionHandlingType() == ExceptionHandling::Wasm &&
          MF.getFunction().hasPersonalityFn()) {
        if (WebAssembly::WasmUseLegacyEH)
          placeTryMarker(MBB);
        else
          placeTryTableMarker(MBB);
      }
    } else {
      // Place the BLOCK for MBB if MBB is branched to from above.
      placeBlockMarker(MBB);
```
- **EN**: Implements helper routine(s) `push_back`, `getParent`, `CreateMachineBasicBlock` for this portion of the WebAssembly backend backend implementation logic. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `push_back`, `getParent`, `CreateMachineBasicBlock`。 该区间与栈帧布局或栈访问相关。

### Lines 2529-2566

```cpp
    }
  }

  if (MCAI.getExceptionHandlingType() == ExceptionHandling::Wasm &&
      MF.getFunction().hasPersonalityFn()) {
    const auto &TII = *MF.getSubtarget<WebAssemblySubtarget>().getInstrInfo();
    // Add an 'unreachable' after 'end_try_table's.
    addUnreachableAfterTryTables(MF, TII);
    // Fix mismatches in unwind destinations induced by linearizing the code.
    // Run fixCatchUnwindMismatches() first so that fixCallUnwindMismatches()
    // will see and correct any new call/rethrow unwind mismatches introduced by
    // fixCatchUnwindMismatches().
    fixCatchUnwindMismatches(MF);
    fixCallUnwindMismatches(MF);
    // addUnreachableAfterTryTables and fixUnwindMismatches create new BBs, so
    // we need to recalculate ScopeTops.
    recalculateScopeTops(MF);
  }
}

unsigned WebAssemblyCFGStackify::getBranchDepth(
    const SmallVectorImpl<EndMarkerInfo> &Stack, const MachineBasicBlock *MBB) {
  unsigned Depth = 0;
  for (auto X : reverse(Stack)) {
    if (X.first == MBB)
      break;
    ++Depth;
  }
  assert(Depth < Stack.size() && "Branch destination should be in scope");
  return Depth;
}

unsigned WebAssemblyCFGStackify::getDelegateDepth(
    const SmallVectorImpl<EndMarkerInfo> &Stack, const MachineBasicBlock *MBB) {
  if (MBB == FakeCallerBB)
    return Stack.size();
  // Delegate's destination is either a catch or a another delegate BB. When the
  // destination is another delegate, we can compute the argument in the same
```
- **EN**: Implements helper routine(s) `getExceptionHandlingType`, `getFunction`, `hasPersonalityFn` for this portion of the WebAssembly backend backend implementation logic. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `getExceptionHandlingType`, `getFunction`, `hasPersonalityFn`。 该区间与栈帧布局或栈访问相关。

### Lines 2567-2604

```cpp
  // way as branches, because the target delegate BB only contains the single
  // delegate instruction.
  if (!MBB->isEHPad()) // Target is a delegate BB
    return getBranchDepth(Stack, MBB);

  // When the delegate's destination is a catch BB, we need to use its
  // corresponding try's end_try BB because Stack contains each marker's end BB.
  // Also we need to check if the end marker instruction matches, because a
  // single BB can contain multiple end markers, like this:
  // bb:
  //   END_BLOCK
  //   END_TRY
  //   END_BLOCK
  //   END_TRY
  //   ...
  //
  // In case of branches getting the immediate that targets any of these is
  // fine, but delegate has to exactly target the correct try.
  unsigned Depth = 0;
  const MachineInstr *EndTry = BeginToEnd[EHPadToTry[MBB]];
  for (auto X : reverse(Stack)) {
    if (X.first == EndTry->getParent() && X.second == EndTry)
      break;
    ++Depth;
  }
  assert(Depth < Stack.size() && "Delegate destination should be in scope");
  return Depth;
}

unsigned WebAssemblyCFGStackify::getRethrowDepth(
    const SmallVectorImpl<EndMarkerInfo> &Stack,
    const MachineBasicBlock *EHPadToRethrow) {
  unsigned Depth = 0;
  for (auto X : reverse(Stack)) {
    const MachineInstr *End = X.second;
    if (End->getOpcode() == WebAssembly::END_TRY) {
      auto *EHPad = TryToEHPad[EndToBegin[End]];
      if (EHPadToRethrow == EHPad)
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "way as branches, because the target delegate BB only contains the single". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“way as branches, because the target delegate BB only contains the single”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 2605-2642

```cpp
        break;
    }
    ++Depth;
  }
  assert(Depth < Stack.size() && "Rethrow destination should be in scope");
  return Depth;
}

void WebAssemblyCFGStackify::rewriteDepthImmediates(MachineFunction &MF) {
  // Now rewrite references to basic blocks to be depth immediates.
  SmallVector<EndMarkerInfo, 8> Stack;

  auto RewriteOperands = [&](MachineInstr &MI) {
    // Rewrite MBB operands to be depth immediates.
    SmallVector<MachineOperand, 4> Ops(MI.operands());
    while (MI.getNumOperands() > 0)
      MI.removeOperand(MI.getNumOperands() - 1);
    for (auto MO : Ops) {
      if (MO.isMBB()) {
        if (MI.getOpcode() == WebAssembly::DELEGATE)
          MO = MachineOperand::CreateImm(getDelegateDepth(Stack, MO.getMBB()));
        else if (MI.getOpcode() == WebAssembly::RETHROW)
          MO = MachineOperand::CreateImm(getRethrowDepth(Stack, MO.getMBB()));
        else
          MO = MachineOperand::CreateImm(getBranchDepth(Stack, MO.getMBB()));
      }
      MI.addOperand(MF, MO);
    }
  };

  for (auto &MBB : reverse(MF)) {
    for (MachineInstr &MI : llvm::reverse(MBB)) {
      switch (MI.getOpcode()) {
      case WebAssembly::BLOCK:
      case WebAssembly::TRY:
        assert(ScopeTops[Stack.back().first->getNumber()]->getNumber() <=
                   MBB.getNumber() &&
               "Block/try/try_table marker should be balanced");
```
- **EN**: Implements helper routine(s) `size`, `rewriteDepthImmediates`, `Ops` for this portion of the WebAssembly backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `size`, `rewriteDepthImmediates`, `Ops`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 2643-2680

```cpp
        Stack.pop_back();
        break;

      case WebAssembly::TRY_TABLE:
        assert(ScopeTops[Stack.back().first->getNumber()]->getNumber() <=
                   MBB.getNumber() &&
               "Block/try/try_table marker should be balanced");
        Stack.pop_back();
        RewriteOperands(MI);
        break;

      case WebAssembly::LOOP:
        assert(Stack.back().first == &MBB && "Loop top should be balanced");
        Stack.pop_back();
        break;

      case WebAssembly::END_BLOCK:
      case WebAssembly::END_TRY:
      case WebAssembly::END_TRY_TABLE:
        Stack.push_back(std::make_pair(&MBB, &MI));
        break;

      case WebAssembly::END_LOOP:
        Stack.push_back(std::make_pair(EndToBegin[&MI]->getParent(), &MI));
        break;

      case WebAssembly::DELEGATE:
        RewriteOperands(MI);
        Stack.push_back(std::make_pair(&MBB, &MI));
        break;

      default:
        if (MI.isTerminator())
          RewriteOperands(MI);
        break;
      }
    }
  }
```
- **EN**: Declares function entry points including `pop_back`, `back`, `getNumber` that other backend components call later. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `pop_back`, `back`, `getNumber`。 该区间与栈帧布局或栈访问相关。

### Lines 2681-2718

```cpp
  assert(Stack.empty() && "Control flow should be balanced");
}

void WebAssemblyCFGStackify::cleanupFunctionData(MachineFunction &MF) {
  if (FakeCallerBB)
    MF.deleteMachineBasicBlock(FakeCallerBB);
  AppendixBB = FakeCallerBB = CallerTrampolineBB = nullptr;
}

void WebAssemblyCFGStackify::releaseMemory() {
  ScopeTops.clear();
  BeginToEnd.clear();
  EndToBegin.clear();
  TryToEHPad.clear();
  EHPadToTry.clear();
  UnwindDestToTrampoline.clear();
}

bool WebAssemblyCFGStackify::runOnMachineFunction(MachineFunction &MF) {
  LLVM_DEBUG(dbgs() << "********** CFG Stackifying **********\n"
                       "********** Function: "
                    << MF.getName() << '\n');
  const MCAsmInfo &MCAI = MF.getTarget().getMCAsmInfo();
  MDT = &getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();

  releaseMemory();

  // Liveness is not tracked for VALUE_STACK physreg.
  MF.getRegInfo().invalidateLiveness();

  // Place the BLOCK/LOOP/TRY/TRY_TABLE markers to indicate the beginnings of
  // scopes.
  placeMarkers(MF);

  // Remove unnecessary instructions possibly introduced by try/end_trys.
  if (MCAI.getExceptionHandlingType() == ExceptionHandling::Wasm &&
      MF.getFunction().hasPersonalityFn() && WebAssembly::WasmUseLegacyEH)
    removeUnnecessaryInstrs(MF);
```
- **EN**: Implements helper routine(s) `empty`, `cleanupFunctionData`, `deleteMachineBasicBlock` for this portion of the WebAssembly backend backend implementation logic. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `empty`, `cleanupFunctionData`, `deleteMachineBasicBlock`。 该区间与栈帧布局或栈访问相关。

### Lines 2719-2735

```cpp

  // Convert MBB operands in terminators to relative depth immediates.
  rewriteDepthImmediates(MF);

  // Fix up block/loop/try/try_table signatures at the end of the function to
  // conform to WebAssembly's rules.
  fixEndsAtEndOfFunction(MF);

  // Add an end instruction at the end of the function body.
  const auto &TII = *MF.getSubtarget<WebAssemblySubtarget>().getInstrInfo();
  appendEndToFunction(MF, TII);

  cleanupFunctionData(MF);

  MF.getInfo<WebAssemblyFunctionInfo>()->setCFGStackified();
  return true;
}
```
- **EN**: Declares function entry points including `rewriteDepthImmediates`, `fixEndsAtEndOfFunction`, `getInstrInfo` that other backend components call later. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `rewriteDepthImmediates`, `fixEndsAtEndOfFunction`, `getInstrInfo`。 该区间与栈帧布局或栈访问相关。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Backend implementation logic / 后端实现逻辑
- SelectionDAG lowering / SelectionDAG lowering
- Stack frame management / 栈帧管理
- Subtarget features / 子目标特性
- Assembly or MC emission / 汇编或 MC 发射
- Instruction selection or opcode handling / 指令选择或操作码处理
- Target machine configuration / 目标机器配置

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `Utils/WebAssemblyTypeUtilities.h`
- `WebAssembly.h`
- `WebAssemblyExceptionInfo.h`
- `WebAssemblyMachineFunctionInfo.h`
- `WebAssemblySortRegion.h`
- `WebAssemblySubtarget.h`
- `WebAssemblyTargetMachine.h`
- `WebAssemblyUtilities.h`
- `llvm/ADT/Statistic.h`
- `llvm/BinaryFormat/Wasm.h`
- `llvm/CodeGen/MachineDominators.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
- `llvm/CodeGen/MachineLoopInfo.h`
- `llvm/MC/MCAsmInfo.h`
- `llvm/Target/TargetMachine.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM MC layer / LLVM MC 层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
