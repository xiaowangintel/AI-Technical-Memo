# LongJmp.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Passes/LongJmp.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: bolt/Passes/LongJmp.cpp This file implements the LongJmpPass class.. It also sits in code that implements BOLT optimization and transformation passes. / 该文件实现 BOLT 优化与变换 Pass。 源码头部说明其职责是：bolt/Passes/LongJmp.cpp This file implements the LongJmpPass class.。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Passes/LongJmp.cpp --------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the LongJmpPass class.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-21

```cpp
#include "bolt/Passes/LongJmp.h"
#include "bolt/Core/ParallelUtilities.h"
#include "bolt/Utils/CommandLineOpts.h"
#include "llvm/Support/MathExtras.h"

#define DEBUG_TYPE "longjmp"

using namespace llvm;
```

- EN: Pulls in 4 header(s) from local project, LLVM dependencies needed by this range. Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `llvm` to organize symbols. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches.
- CN: 这里引入 4 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `llvm` 中，用于组织符号作用域。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。

### Lines 22-29

```cpp
namespace opts {
extern cl::OptionCategory BoltCategory;
extern cl::OptionCategory BoltOptCategory;
extern llvm::cl::opt<unsigned> AlignText;
extern cl::opt<unsigned> AlignFunctions;
extern cl::opt<bool> UseOldText;
extern cl::opt<bool> HotFunctionsAtEnd;
```

- EN: Works inside namespace scope `opts` to organize symbols. Notable symbols here include `opts`.
- CN: 这里位于命名空间 `opts` 中，用于组织符号作用域。这里较值得关注的符号包括 `opts`。

### Lines 30-37

```cpp
static cl::opt<bool> GroupStubs("group-stubs",
                                cl::desc("share stubs across functions"),
                                cl::init(true), cl::cat(BoltOptCategory));
}

namespace llvm {
namespace bolt {
```

- EN: Works inside namespace scope `llvm`, `bolt` to organize symbols. Declares or implements routines including `desc`, `init`. Notable symbols here include `desc`, `init`, `llvm`, `bolt`.
- CN: 这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里声明或实现函数，例如 `desc`, `init`。这里较值得关注的符号包括 `desc`, `init`, `llvm`, `bolt`。

### Lines 38-49

```cpp
constexpr unsigned ColdFragAlign = 16;

static void relaxStubToShortJmp(BinaryBasicBlock &StubBB, const MCSymbol *Tgt) {
  const BinaryContext &BC = StubBB.getFunction()->getBinaryContext();
  InstructionListType Seq;
  BC.MIB->createShortJmp(Seq, Tgt, BC.Ctx.get());
  StubBB.clear();
  StubBB.addInstructions(Seq.begin(), Seq.end());
  if (BC.usesBTI())
    BC.MIB->applyBTIFixupToTarget(StubBB);
}
```

- EN: Declares or implements routines including `relaxStubToShortJmp`, `createShortJmp`, `applyBTIFixupToTarget`. Notable symbols here include `relaxStubToShortJmp`, `createShortJmp`, `applyBTIFixupToTarget`.
- CN: 这里声明或实现函数，例如 `relaxStubToShortJmp`, `createShortJmp`, `applyBTIFixupToTarget`。这里较值得关注的符号包括 `relaxStubToShortJmp`, `createShortJmp`, `applyBTIFixupToTarget`。

### Lines 50-59

```cpp
static void relaxStubToLongJmp(BinaryBasicBlock &StubBB, const MCSymbol *Tgt) {
  const BinaryContext &BC = StubBB.getFunction()->getBinaryContext();
  InstructionListType Seq;
  BC.MIB->createLongJmp(Seq, Tgt, BC.Ctx.get());
  StubBB.clear();
  StubBB.addInstructions(Seq.begin(), Seq.end());
  if (BC.usesBTI())
    BC.MIB->applyBTIFixupToTarget(StubBB);
}
```

- EN: Declares or implements routines including `relaxStubToLongJmp`, `createLongJmp`, `applyBTIFixupToTarget`. Notable symbols here include `relaxStubToLongJmp`, `createLongJmp`, `applyBTIFixupToTarget`.
- CN: 这里声明或实现函数，例如 `relaxStubToLongJmp`, `createLongJmp`, `applyBTIFixupToTarget`。这里较值得关注的符号包括 `relaxStubToLongJmp`, `createLongJmp`, `applyBTIFixupToTarget`。

### Lines 60-74

```cpp
static BinaryBasicBlock *getBBAtHotColdSplitPoint(BinaryFunction &Func) {
  if (!Func.isSplit() || Func.empty())
    return nullptr;

  assert(!(*Func.begin()).isCold() && "Entry cannot be cold");
  for (auto I = Func.getLayout().block_begin(),
            E = Func.getLayout().block_end();
       I != E; ++I) {
    auto Next = std::next(I);
    if (Next != E && (*Next)->isCold())
      return *I;
  }
  llvm_unreachable("No hot-cold split point found");
}
```

- EN: Declares or implements routines including `getBBAtHotColdSplitPoint`, `assert`, `next`, `llvm_unreachable`. Notable symbols here include `getBBAtHotColdSplitPoint`, `assert`, `next`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `getBBAtHotColdSplitPoint`, `assert`, `next`, `llvm_unreachable`。这里较值得关注的符号包括 `getBBAtHotColdSplitPoint`, `assert`, `next`, `llvm_unreachable`。

### Lines 75-86

```cpp
static bool mayNeedStub(const BinaryContext &BC, const MCInst &Inst) {
  if (BC.isAArch64() && BC.MIB->isShortRangeBranch(Inst) &&
      !opts::CompactCodeModel) {
    BC.errs() << "BOLT-ERROR: short range branch not supported"
              << " outside compact code model\n";
    BC.printInstruction(BC.errs(), Inst);
    exit(1);
  }
  return (BC.MIB->isBranch(Inst) || BC.MIB->isCall(Inst)) &&
         !BC.MIB->isIndirectBranch(Inst) && !BC.MIB->isIndirectCall(Inst);
}
```

- EN: Declares or implements routines including `mayNeedStub`, `exit`, `isIndirectBranch`. Notable symbols here include `mayNeedStub`, `exit`, `isIndirectBranch`.
- CN: 这里声明或实现函数，例如 `mayNeedStub`, `exit`, `isIndirectBranch`。这里较值得关注的符号包括 `mayNeedStub`, `exit`, `isIndirectBranch`。

### Lines 87-101

```cpp
std::pair<std::unique_ptr<BinaryBasicBlock>, MCSymbol *>
LongJmpPass::createNewStub(BinaryBasicBlock &SourceBB, const MCSymbol *TgtSym,
                           bool TgtIsFunc, uint64_t AtAddress) {
  BinaryFunction &Func = *SourceBB.getFunction();
  const BinaryContext &BC = Func.getBinaryContext();
  const bool IsCold = SourceBB.isCold();
  MCSymbol *StubSym = BC.Ctx->createNamedTempSymbol("Stub");
  std::unique_ptr<BinaryBasicBlock> StubBB = Func.createBasicBlock(StubSym);
  MCInst Inst;
  BC.MIB->createUncondBranch(Inst, TgtSym, BC.Ctx.get());
  if (TgtIsFunc)
    BC.MIB->convertJmpToTailCall(Inst);
  StubBB->addInstruction(Inst);
  StubBB->setExecutionCount(0);
```

- EN: Declares or implements routines including `createNamedTempSymbol`, `createUncondBranch`, `convertJmpToTailCall`, `addInstruction`, `setExecutionCount`. Notable symbols here include `createNamedTempSymbol`, `createUncondBranch`, `convertJmpToTailCall`, `addInstruction`, `setExecutionCount`.
- CN: 这里声明或实现函数，例如 `createNamedTempSymbol`, `createUncondBranch`, `convertJmpToTailCall`, `addInstruction`, `setExecutionCount`。这里较值得关注的符号包括 `createNamedTempSymbol`, `createUncondBranch`, `convertJmpToTailCall`, `addInstruction`, `setExecutionCount`。

### Lines 102-114

```cpp
  // Register this in stubs maps
  auto registerInMap = [&](StubGroupsTy &Map) {
    StubGroupTy &StubGroup = Map[TgtSym];
    StubGroup.insert(
        llvm::lower_bound(
            StubGroup, std::make_pair(AtAddress, nullptr),
            [&](const std::pair<uint64_t, BinaryBasicBlock *> &LHS,
                const std::pair<uint64_t, BinaryBasicBlock *> &RHS) {
              return LHS.first < RHS.first;
            }),
        std::make_pair(AtAddress, StubBB.get()));
  };
```

- EN: Declares or implements routines including `make_pair`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `make_pair`.
- CN: 这里声明或实现函数，例如 `make_pair`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `make_pair`。

### Lines 115-128

```cpp
  Stubs[&Func].insert(StubBB.get());
  StubBits[StubBB.get()] = BC.MIB->getUncondBranchEncodingSize();
  if (IsCold) {
    registerInMap(ColdLocalStubs[&Func]);
    if (opts::GroupStubs && TgtIsFunc)
      registerInMap(ColdStubGroups);
    ++NumColdStubs;
  } else {
    registerInMap(HotLocalStubs[&Func]);
    if (opts::GroupStubs && TgtIsFunc)
      registerInMap(HotStubGroups);
    ++NumHotStubs;
  }
```

- EN: Declares or implements routines including `registerInMap`. Notable symbols here include `registerInMap`.
- CN: 这里声明或实现函数，例如 `registerInMap`。这里较值得关注的符号包括 `registerInMap`。

### Lines 129-146

```cpp
  return std::make_pair(std::move(StubBB), StubSym);
}

BinaryBasicBlock *LongJmpPass::lookupStubFromGroup(
    const StubGroupsTy &StubGroups, const BinaryFunction &Func,
    const MCInst &Inst, const MCSymbol *TgtSym, uint64_t DotAddress) const {
  const BinaryContext &BC = Func.getBinaryContext();
  auto CandidatesIter = StubGroups.find(TgtSym);
  if (CandidatesIter == StubGroups.end())
    return nullptr;
  const StubGroupTy &Candidates = CandidatesIter->second;
  if (Candidates.empty())
    return nullptr;
  auto Cand = llvm::lower_bound(
      Candidates, std::make_pair(DotAddress, nullptr),
      [&](const std::pair<uint64_t, BinaryBasicBlock *> &LHS,
          const std::pair<uint64_t, BinaryBasicBlock *> &RHS) {
        return LHS.first < RHS.first;
```

- EN: Declares or implements routines including `make_pair`. Notable symbols here include `make_pair`.
- CN: 这里声明或实现函数，例如 `make_pair`。这里较值得关注的符号包括 `make_pair`。

### Lines 147-162

```cpp
      });
  if (Cand == Candidates.end()) {
    Cand = std::prev(Cand);
  } else if (Cand != Candidates.begin()) {
    const StubTy *LeftCand = std::prev(Cand);
    if (Cand->first - DotAddress > DotAddress - LeftCand->first)
      Cand = LeftCand;
  }
  int BitsAvail = BC.MIB->getPCRelEncodingSize(Inst) - 1;
  assert(BitsAvail < 63 && "PCRelEncodingSize is too large to use int64_t to"
                           "check for out-of-bounds.");
  int64_t MaxVal = (1ULL << BitsAvail) - 1;
  int64_t MinVal = -(1ULL << BitsAvail);
  uint64_t PCRelTgtAddress = Cand->first;
  int64_t PCOffset = (int64_t)(PCRelTgtAddress - DotAddress);
```

- EN: Declares or implements routines including `prev`, `if`, `getPCRelEncodingSize`. Notable symbols here include `prev`, `if`, `getPCRelEncodingSize`.
- CN: 这里声明或实现函数，例如 `prev`, `if`, `getPCRelEncodingSize`。这里较值得关注的符号包括 `prev`, `if`, `getPCRelEncodingSize`。

### Lines 163-172

```cpp
  LLVM_DEBUG({
    if (Candidates.size() > 1)
      dbgs() << "Considering stub group with " << Candidates.size()
             << " candidates. DotAddress is " << Twine::utohexstr(DotAddress)
             << ", chosen candidate address is "
             << Twine::utohexstr(Cand->first) << "\n";
  });
  return (PCOffset < MinVal || PCOffset > MaxVal) ? nullptr : Cand->second;
}
```

- EN: Declares or implements routines including `dbgs`, `utohexstr`. Notable symbols here include `dbgs`, `utohexstr`.
- CN: 这里声明或实现函数，例如 `dbgs`, `utohexstr`。这里较值得关注的符号包括 `dbgs`, `utohexstr`。

### Lines 173-182

```cpp
BinaryBasicBlock *
LongJmpPass::lookupGlobalStub(const BinaryBasicBlock &SourceBB,
                              const MCInst &Inst, const MCSymbol *TgtSym,
                              uint64_t DotAddress) const {
  const BinaryFunction &Func = *SourceBB.getFunction();
  const StubGroupsTy &StubGroups =
      SourceBB.isCold() ? ColdStubGroups : HotStubGroups;
  return lookupStubFromGroup(StubGroups, Func, Inst, TgtSym, DotAddress);
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 183-195

```cpp
BinaryBasicBlock *LongJmpPass::lookupLocalStub(const BinaryBasicBlock &SourceBB,
                                               const MCInst &Inst,
                                               const MCSymbol *TgtSym,
                                               uint64_t DotAddress) const {
  const BinaryFunction &Func = *SourceBB.getFunction();
  const DenseMap<const BinaryFunction *, StubGroupsTy> &StubGroups =
      SourceBB.isCold() ? ColdLocalStubs : HotLocalStubs;
  const auto Iter = StubGroups.find(&Func);
  if (Iter == StubGroups.end())
    return nullptr;
  return lookupStubFromGroup(Iter->second, Func, Inst, TgtSym, DotAddress);
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 196-205

```cpp
std::unique_ptr<BinaryBasicBlock>
LongJmpPass::replaceTargetWithStub(BinaryBasicBlock &BB, MCInst &Inst,
                                   uint64_t DotAddress,
                                   uint64_t StubCreationAddress) {
  const BinaryFunction &Func = *BB.getFunction();
  const BinaryContext &BC = Func.getBinaryContext();
  std::unique_ptr<BinaryBasicBlock> NewBB;
  const MCSymbol *TgtSym = BC.MIB->getTargetSymbol(Inst);
  assert(TgtSym && "getTargetSymbol failed");
```

- EN: Declares or implements routines including `getTargetSymbol`, `assert`. Notable symbols here include `getTargetSymbol`, `assert`.
- CN: 这里声明或实现函数，例如 `getTargetSymbol`, `assert`。这里较值得关注的符号包括 `getTargetSymbol`, `assert`。

### Lines 206-223

```cpp
  BinaryBasicBlock::BinaryBranchInfo BI{0, 0};
  BinaryBasicBlock *TgtBB = BB.getSuccessor(TgtSym, BI);
  auto LocalStubsIter = Stubs.find(&Func);

  // If already using stub and the stub is from another function, create a local
  // stub, since the foreign stub is now out of range
  if (!TgtBB) {
    auto SSIter = SharedStubs.find(TgtSym);
    if (SSIter != SharedStubs.end()) {
      TgtSym = BC.MIB->getTargetSymbol(*SSIter->second->begin());
      --NumSharedStubs;
    }
  } else if (LocalStubsIter != Stubs.end() &&
             LocalStubsIter->second.count(TgtBB)) {
    // The TgtBB and TgtSym now are the local out-of-range stub and its label.
    // So, we are attempting to restore BB to its previous state without using
    // this stub.
    TgtSym = BC.MIB->getTargetSymbol(*TgtBB->begin());
```

- EN: Declares or implements routines including `getTargetSymbol`, `if`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getTargetSymbol`, `if`.
- CN: 这里声明或实现函数，例如 `getTargetSymbol`, `if`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getTargetSymbol`, `if`。

### Lines 224-235

```cpp
    assert(TgtSym &&
           "First instruction is expected to contain a target symbol.");
    BinaryBasicBlock *TgtBBSucc = TgtBB->getSuccessor(TgtSym, BI);

    // TgtBB might have no successor. e.g. a stub for a function call.
    if (TgtBBSucc) {
      BB.replaceSuccessor(TgtBB, TgtBBSucc, BI.Count, BI.MispredictedCount);
      assert(TgtBB->getExecutionCount() >= BI.Count &&
             "At least equal or greater than the branch count.");
      TgtBB->setExecutionCount(TgtBB->getExecutionCount() - BI.Count);
    }
```

- EN: Declares or implements routines including `getSuccessor`, `assert`, `setExecutionCount`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getSuccessor`, `assert`, `setExecutionCount`.
- CN: 这里声明或实现函数，例如 `getSuccessor`, `assert`, `setExecutionCount`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getSuccessor`, `assert`, `setExecutionCount`。

### Lines 236-250

```cpp
    TgtBB = TgtBBSucc;
  }

  BinaryBasicBlock *StubBB = lookupLocalStub(BB, Inst, TgtSym, DotAddress);
  // If not found, look it up in globally shared stub maps if it is a function
  // call (TgtBB is not set)
  if (!StubBB && !TgtBB) {
    StubBB = lookupGlobalStub(BB, Inst, TgtSym, DotAddress);
    if (StubBB) {
      SharedStubs[StubBB->getLabel()] = StubBB;
      ++NumSharedStubs;
    }
  }
  MCSymbol *StubSymbol = StubBB ? StubBB->getLabel() : nullptr;
```

- EN: Declares or implements routines including `lookupLocalStub`, `lookupGlobalStub`, `getLabel`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `lookupLocalStub`, `lookupGlobalStub`, `getLabel`.
- CN: 这里声明或实现函数，例如 `lookupLocalStub`, `lookupGlobalStub`, `getLabel`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `lookupLocalStub`, `lookupGlobalStub`, `getLabel`。

### Lines 251-268

```cpp
  if (!StubBB) {
    std::tie(NewBB, StubSymbol) =
        createNewStub(BB, TgtSym, /*is func?*/ !TgtBB, StubCreationAddress);
    StubBB = NewBB.get();
  }

  // Local branch
  if (TgtBB) {
    uint64_t OrigCount = BI.Count;
    uint64_t OrigMispreds = BI.MispredictedCount;
    BB.replaceSuccessor(TgtBB, StubBB, OrigCount, OrigMispreds);
    StubBB->setExecutionCount(StubBB->getExecutionCount() + OrigCount);
    if (NewBB) {
      StubBB->addSuccessor(TgtBB, OrigCount, OrigMispreds);
      StubBB->setIsCold(BB.isCold());
    }
    // Call / tail call
  } else {
```

- EN: Declares or implements routines including `tie`, `createNewStub`, `setExecutionCount`, `addSuccessor`, `setIsCold`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `tie`, `createNewStub`, `setExecutionCount`, `addSuccessor`, `setIsCold`.
- CN: 这里声明或实现函数，例如 `tie`, `createNewStub`, `setExecutionCount`, `addSuccessor`, `setIsCold`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `tie`, `createNewStub`, `setExecutionCount`, `addSuccessor`, `setIsCold`。

### Lines 269-279

```cpp
    StubBB->setExecutionCount(StubBB->getExecutionCount() +
                              BB.getExecutionCount());
    if (NewBB) {
      assert(TgtBB == nullptr);
      StubBB->setIsCold(BB.isCold());
      // Set as entry point because this block is valid but we have no preds
      StubBB->getFunction()->addEntryPoint(*StubBB);
    }
  }
  BC.MIB->replaceBranchTarget(Inst, StubSymbol, BC.Ctx.get());
```

- EN: Declares or implements routines including `setExecutionCount`, `assert`, `setIsCold`, `getFunction`, `replaceBranchTarget`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `setExecutionCount`, `assert`, `setIsCold`, `getFunction`, `replaceBranchTarget`.
- CN: 这里声明或实现函数，例如 `setExecutionCount`, `assert`, `setIsCold`, `getFunction`, `replaceBranchTarget`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `setExecutionCount`, `assert`, `setIsCold`, `getFunction`, `replaceBranchTarget`。

### Lines 280-291

```cpp
  return NewBB;
}

void LongJmpPass::updateStubGroups() {
  auto update = [&](StubGroupsTy &StubGroups) {
    for (auto &KeyVal : StubGroups) {
      for (StubTy &Elem : KeyVal.second)
        Elem.first = BBAddresses[Elem.second];
      llvm::sort(KeyVal.second, llvm::less_first());
    }
  };
```

- EN: Declares or implements routines including `updateStubGroups`, `sort`. Notable symbols here include `updateStubGroups`, `sort`.
- CN: 这里声明或实现函数，例如 `updateStubGroups`, `sort`。这里较值得关注的符号包括 `updateStubGroups`, `sort`。

### Lines 292-299

```cpp
  for (auto &KeyVal : HotLocalStubs)
    update(KeyVal.second);
  for (auto &KeyVal : ColdLocalStubs)
    update(KeyVal.second);
  update(HotStubGroups);
  update(ColdStubGroups);
}
```

- EN: Declares or implements routines including `update`. Notable symbols here include `update`.
- CN: 这里声明或实现函数，例如 `update`。这里较值得关注的符号包括 `update`。

### Lines 300-316

```cpp
void LongJmpPass::tentativeBBLayout(const BinaryFunction &Func) {
  const BinaryContext &BC = Func.getBinaryContext();
  uint64_t HotDot = HotAddresses[&Func];
  uint64_t ColdDot = ColdAddresses[&Func];
  bool Cold = false;
  for (const BinaryBasicBlock *BB : Func.getLayout().blocks()) {
    if (Cold || BB->isCold()) {
      Cold = true;
      BBAddresses[BB] = ColdDot;
      ColdDot += BC.computeCodeSize(BB->begin(), BB->end());
    } else {
      BBAddresses[BB] = HotDot;
      HotDot += BC.computeCodeSize(BB->begin(), BB->end());
    }
  }
}
```

- EN: Declares or implements routines including `tentativeBBLayout`. Notable symbols here include `tentativeBBLayout`.
- CN: 这里声明或实现函数，例如 `tentativeBBLayout`。这里较值得关注的符号包括 `tentativeBBLayout`。

### Lines 317-334

```cpp
uint64_t LongJmpPass::tentativeLayoutRelocColdPart(
    const BinaryContext &BC, BinaryFunctionListType &SortedFunctions,
    uint64_t DotAddress) {
  DotAddress = alignTo(DotAddress, llvm::Align(opts::AlignFunctions));
  for (BinaryFunction *Func : SortedFunctions) {
    if (!Func->isSplit())
      continue;
    DotAddress = alignTo(DotAddress, Func->getMinAlignment());
    uint64_t Pad =
        offsetToAlignment(DotAddress, llvm::Align(Func->getAlignment()));
    if (Pad <= Func->getMaxColdAlignmentBytes())
      DotAddress += Pad;
    ColdAddresses[Func] = DotAddress;
    LLVM_DEBUG(dbgs() << Func->getPrintName() << " cold tentative: "
                      << Twine::utohexstr(DotAddress) << "\n");
    DotAddress += Func->estimateColdSize();
    DotAddress = alignTo(DotAddress, Func->getConstantIslandAlignment());
    DotAddress += Func->estimateConstantIslandSize();
```

- EN: Declares or implements routines including `alignTo`, `offsetToAlignment`, `LLVM_DEBUG`, `utohexstr`, `estimateColdSize`, and 1 more. Notable symbols here include `alignTo`, `offsetToAlignment`, `LLVM_DEBUG`, `utohexstr`, `estimateColdSize`, `estimateConstantIslandSize`.
- CN: 这里声明或实现函数，例如 `alignTo`, `offsetToAlignment`, `LLVM_DEBUG`, `utohexstr`, `estimateColdSize`, and 1 more。这里较值得关注的符号包括 `alignTo`, `offsetToAlignment`, `LLVM_DEBUG`, `utohexstr`, `estimateColdSize`, `estimateConstantIslandSize`。

### Lines 335-352

```cpp
  }
  return DotAddress;
}

uint64_t
LongJmpPass::tentativeLayoutRelocMode(const BinaryContext &BC,
                                      BinaryFunctionListType &SortedFunctions,
                                      uint64_t DotAddress) {
  // Compute hot cold frontier
  int64_t LastHotIndex = -1u;
  uint32_t CurrentIndex = 0;
  if (opts::HotFunctionsAtEnd) {
    for (BinaryFunction *BF : SortedFunctions) {
      if (BF->hasValidIndex()) {
        LastHotIndex = CurrentIndex;
        break;
      }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 353-361

```cpp
      ++CurrentIndex;
    }
  } else {
    for (BinaryFunction *BF : SortedFunctions) {
      if (!BF->hasValidIndex()) {
        LastHotIndex = CurrentIndex;
        break;
      }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 362-379

```cpp
      ++CurrentIndex;
    }
  }

  // Hot
  CurrentIndex = 0;
  bool ColdLayoutDone = false;
  auto runColdLayout = [&]() {
    // Mirror the extra hugify alignment inserted by final section allocation
    // after the last non-cold section. Account for it before assigning cold
    // fragment addresses so range checks see the hot-to-cold gap.
    if (opts::Hugify && !BC.HasFixedLoadAddress && !opts::HotFunctionsAtEnd)
      DotAddress = alignTo(DotAddress, opts::AlignText);
    DotAddress = tentativeLayoutRelocColdPart(BC, SortedFunctions, DotAddress);
    ColdLayoutDone = true;
    if (opts::HotFunctionsAtEnd)
      DotAddress = alignTo(DotAddress, opts::AlignText);
  };
```

- EN: Declares or implements routines including `alignTo`, `tentativeLayoutRelocColdPart`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `alignTo`, `tentativeLayoutRelocColdPart`.
- CN: 这里声明或实现函数，例如 `alignTo`, `tentativeLayoutRelocColdPart`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `alignTo`, `tentativeLayoutRelocColdPart`。

### Lines 380-388

```cpp
  for (BinaryFunction *Func : SortedFunctions) {
    if (!BC.shouldEmit(*Func)) {
      HotAddresses[Func] = Func->getAddress();
      continue;
    }

    if (!ColdLayoutDone && CurrentIndex >= LastHotIndex)
      runColdLayout();
```

- EN: Declares or implements routines including `getAddress`, `runColdLayout`. Notable symbols here include `getAddress`, `runColdLayout`.
- CN: 这里声明或实现函数，例如 `getAddress`, `runColdLayout`。这里较值得关注的符号包括 `getAddress`, `runColdLayout`。

### Lines 389-401

```cpp
    DotAddress = alignTo(DotAddress, Func->getMinAlignment());
    uint64_t Pad =
        offsetToAlignment(DotAddress, llvm::Align(Func->getAlignment()));
    if (Pad <= Func->getMaxAlignmentBytes())
      DotAddress += Pad;
    HotAddresses[Func] = DotAddress;
    LLVM_DEBUG(dbgs() << Func->getPrintName() << " tentative: "
                      << Twine::utohexstr(DotAddress) << "\n");
    if (!Func->isSplit())
      DotAddress += Func->estimateSize();
    else
      DotAddress += Func->estimateHotSize();
```

- EN: Declares or implements routines including `alignTo`, `offsetToAlignment`, `LLVM_DEBUG`, `utohexstr`, `estimateSize`, and 1 more. Notable symbols here include `alignTo`, `offsetToAlignment`, `LLVM_DEBUG`, `utohexstr`, `estimateSize`, `estimateHotSize`.
- CN: 这里声明或实现函数，例如 `alignTo`, `offsetToAlignment`, `LLVM_DEBUG`, `utohexstr`, `estimateSize`, and 1 more。这里较值得关注的符号包括 `alignTo`, `offsetToAlignment`, `LLVM_DEBUG`, `utohexstr`, `estimateSize`, `estimateHotSize`。

### Lines 402-410

```cpp
    DotAddress = alignTo(DotAddress, Func->getConstantIslandAlignment());
    DotAddress += Func->estimateConstantIslandSize();
    ++CurrentIndex;
  }

  // Ensure that tentative code layout always runs for cold blocks.
  if (!ColdLayoutDone)
    runColdLayout();
```

- EN: Declares or implements routines including `alignTo`, `estimateConstantIslandSize`, `runColdLayout`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `alignTo`, `estimateConstantIslandSize`, `runColdLayout`.
- CN: 这里声明或实现函数，例如 `alignTo`, `estimateConstantIslandSize`, `runColdLayout`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `alignTo`, `estimateConstantIslandSize`, `runColdLayout`。

### Lines 411-421

```cpp
  // BBs
  for (BinaryFunction *Func : SortedFunctions)
    tentativeBBLayout(*Func);

  return DotAddress;
}

void LongJmpPass::tentativeLayout(const BinaryContext &BC,
                                  BinaryFunctionListType &SortedFunctions) {
  uint64_t DotAddress = BC.LayoutStartAddress;
```

- EN: Declares or implements routines including `tentativeBBLayout`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `tentativeBBLayout`.
- CN: 这里声明或实现函数，例如 `tentativeBBLayout`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `tentativeBBLayout`。

### Lines 422-431

```cpp
  if (!BC.HasRelocations) {
    for (BinaryFunction *Func : SortedFunctions) {
      HotAddresses[Func] = Func->getAddress();
      DotAddress = alignTo(DotAddress, ColdFragAlign);
      ColdAddresses[Func] = DotAddress;
      if (Func->isSplit())
        DotAddress += Func->estimateColdSize();
      tentativeBBLayout(*Func);
    }
```

- EN: Declares or implements routines including `getAddress`, `alignTo`, `estimateColdSize`, `tentativeBBLayout`. Notable symbols here include `getAddress`, `alignTo`, `estimateColdSize`, `tentativeBBLayout`.
- CN: 这里声明或实现函数，例如 `getAddress`, `alignTo`, `estimateColdSize`, `tentativeBBLayout`。这里较值得关注的符号包括 `getAddress`, `alignTo`, `estimateColdSize`, `tentativeBBLayout`。

### Lines 432-439

```cpp
    return;
  }

  // Relocation mode
  uint64_t EstimatedTextSize = 0;
  if (opts::UseOldText) {
    EstimatedTextSize = tentativeLayoutRelocMode(BC, SortedFunctions, 0);
```

- EN: Declares or implements routines including `tentativeLayoutRelocMode`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `tentativeLayoutRelocMode`.
- CN: 这里声明或实现函数，例如 `tentativeLayoutRelocMode`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `tentativeLayoutRelocMode`。

### Lines 440-450

```cpp
    // Initial padding
    if (EstimatedTextSize <= BC.OldTextSectionSize) {
      DotAddress = BC.OldTextSectionAddress;
      uint64_t Pad =
          offsetToAlignment(DotAddress, llvm::Align(opts::AlignText));
      if (Pad + EstimatedTextSize <= BC.OldTextSectionSize) {
        DotAddress += Pad;
      }
    }
  }
```

- EN: Declares or implements routines including `offsetToAlignment`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `offsetToAlignment`.
- CN: 这里声明或实现函数，例如 `offsetToAlignment`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `offsetToAlignment`。

### Lines 451-466

```cpp
  if (!EstimatedTextSize || EstimatedTextSize > BC.OldTextSectionSize)
    DotAddress = alignTo(BC.LayoutStartAddress, opts::AlignText);

  tentativeLayoutRelocMode(BC, SortedFunctions, DotAddress);
}

bool LongJmpPass::usesStub(const BinaryFunction &Func,
                           const MCInst &Inst) const {
  const MCSymbol *TgtSym = Func.getBinaryContext().MIB->getTargetSymbol(Inst);
  const BinaryBasicBlock *TgtBB = Func.getBasicBlockForLabel(TgtSym);
  auto Iter = Stubs.find(&Func);
  if (Iter != Stubs.end())
    return Iter->second.count(TgtBB);
  return false;
}
```

- EN: Declares or implements routines including `alignTo`, `tentativeLayoutRelocMode`. Notable symbols here include `alignTo`, `tentativeLayoutRelocMode`.
- CN: 这里声明或实现函数，例如 `alignTo`, `tentativeLayoutRelocMode`。这里较值得关注的符号包括 `alignTo`, `tentativeLayoutRelocMode`。

### Lines 467-484

```cpp
uint64_t LongJmpPass::getSymbolAddress(const BinaryContext &BC,
                                       const MCSymbol *Target,
                                       const BinaryBasicBlock *TgtBB) const {
  if (TgtBB) {
    auto Iter = BBAddresses.find(TgtBB);
    assert(Iter != BBAddresses.end() && "Unrecognized BB");
    return Iter->second;
  }
  uint64_t EntryID = 0;
  const BinaryFunction *TargetFunc = BC.getFunctionForSymbol(Target, &EntryID);
  auto Iter = HotAddresses.find(TargetFunc);
  if (Iter == HotAddresses.end() || (TargetFunc && EntryID)) {
    // Look at BinaryContext's resolution for this symbol - this is a symbol not
    // mapped to a BinaryFunction
    ErrorOr<uint64_t> ValueOrError = BC.getSymbolValue(*Target);
    assert(ValueOrError && "Unrecognized symbol");
    return *ValueOrError;
  }
```

- EN: Declares or implements routines including `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`。

### Lines 485-495

```cpp
  return Iter->second;
}

Error LongJmpPass::relaxStub(BinaryBasicBlock &StubBB, bool &Modified) {
  BinaryFunction &Func = *StubBB.getFunction();
  BinaryContext &BC = Func.getBinaryContext();
  const int Bits = StubBits[&StubBB];
  // Already working with the largest range?
  if (Bits == static_cast<int>(BC.AsmInfo->getCodePointerSize() * 8))
    return Error::success();
```

- EN: Declares or implements routines including `relaxStub`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `relaxStub`.
- CN: 这里声明或实现函数，例如 `relaxStub`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `relaxStub`。

### Lines 496-508

```cpp
  const static int RangeShortJmp = BC.MIB->getShortJmpEncodingSize();
  const static int RangeSingleInstr = BC.MIB->getUncondBranchEncodingSize();
  const static uint64_t ShortJmpMask = ~((1ULL << RangeShortJmp) - 1);
  const static uint64_t SingleInstrMask =
      ~((1ULL << (RangeSingleInstr - 1)) - 1);

  const MCSymbol *RealTargetSym = BC.MIB->getTargetSymbol(*StubBB.begin());
  const BinaryBasicBlock *TgtBB = Func.getBasicBlockForLabel(RealTargetSym);
  uint64_t TgtAddress = getSymbolAddress(BC, RealTargetSym, TgtBB);
  uint64_t DotAddress = BBAddresses[&StubBB];
  uint64_t PCRelTgtAddress = DotAddress > TgtAddress ? DotAddress - TgtAddress
                                                     : TgtAddress - DotAddress;
```

- EN: Declares or implements routines including `getShortJmpEncodingSize`, `getUncondBranchEncodingSize`, `getTargetSymbol`, `getSymbolAddress`. Notable symbols here include `getShortJmpEncodingSize`, `getUncondBranchEncodingSize`, `getTargetSymbol`, `getSymbolAddress`.
- CN: 这里声明或实现函数，例如 `getShortJmpEncodingSize`, `getUncondBranchEncodingSize`, `getTargetSymbol`, `getSymbolAddress`。这里较值得关注的符号包括 `getShortJmpEncodingSize`, `getUncondBranchEncodingSize`, `getTargetSymbol`, `getSymbolAddress`。

### Lines 509-517

```cpp
  // If it fits in one instruction, do not relax
  if (!(PCRelTgtAddress & SingleInstrMask))
    return Error::success();

  // Fits short jmp
  if (!(PCRelTgtAddress & ShortJmpMask)) {
    if (Bits >= RangeShortJmp)
      return Error::success();
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 518-527

```cpp
    LLVM_DEBUG(dbgs() << "Relaxing stub to short jump. PCRelTgtAddress = "
                      << Twine::utohexstr(PCRelTgtAddress)
                      << " RealTargetSym = " << RealTargetSym->getName()
                      << "\n");
    relaxStubToShortJmp(StubBB, RealTargetSym);
    StubBits[&StubBB] = RangeShortJmp;
    Modified = true;
    return Error::success();
  }
```

- EN: Declares or implements routines including `LLVM_DEBUG`, `utohexstr`, `getName`, `relaxStubToShortJmp`. Notable symbols here include `LLVM_DEBUG`, `utohexstr`, `getName`, `relaxStubToShortJmp`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`, `utohexstr`, `getName`, `relaxStubToShortJmp`。这里较值得关注的符号包括 `LLVM_DEBUG`, `utohexstr`, `getName`, `relaxStubToShortJmp`。

### Lines 528-542

```cpp
  // The long jmp uses absolute address on AArch64
  // So we could not use it for PIC binaries
  if (BC.isAArch64() && !BC.HasFixedLoadAddress)
    return createFatalBOLTError(
        "BOLT-ERROR: Unable to relax stub for PIC binary\n");

  LLVM_DEBUG(dbgs() << "Relaxing stub to long jump. PCRelTgtAddress = "
                    << Twine::utohexstr(PCRelTgtAddress)
                    << " RealTargetSym = " << RealTargetSym->getName() << "\n");
  relaxStubToLongJmp(StubBB, RealTargetSym);
  StubBits[&StubBB] = static_cast<int>(BC.AsmInfo->getCodePointerSize() * 8);
  Modified = true;
  return Error::success();
}
```

- EN: Declares or implements routines including `LLVM_DEBUG`, `utohexstr`, `getName`, `relaxStubToLongJmp`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `LLVM_DEBUG`, `utohexstr`, `getName`, `relaxStubToLongJmp`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`, `utohexstr`, `getName`, `relaxStubToLongJmp`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `LLVM_DEBUG`, `utohexstr`, `getName`, `relaxStubToLongJmp`。

### Lines 543-557

```cpp
bool LongJmpPass::needsStub(const BinaryBasicBlock &BB, const MCInst &Inst,
                            uint64_t DotAddress) const {
  const BinaryFunction &Func = *BB.getFunction();
  const BinaryContext &BC = Func.getBinaryContext();
  const MCSymbol *TgtSym = BC.MIB->getTargetSymbol(Inst);
  assert(TgtSym && "getTargetSymbol failed");

  const BinaryBasicBlock *TgtBB = Func.getBasicBlockForLabel(TgtSym);
  // Check for shared stubs from foreign functions
  if (!TgtBB) {
    auto SSIter = SharedStubs.find(TgtSym);
    if (SSIter != SharedStubs.end())
      TgtBB = SSIter->second;
  }
```

- EN: Declares or implements routines including `getTargetSymbol`, `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getTargetSymbol`, `assert`.
- CN: 这里声明或实现函数，例如 `getTargetSymbol`, `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getTargetSymbol`, `assert`。

### Lines 558-566

```cpp
  int BitsAvail = BC.MIB->getPCRelEncodingSize(Inst) - 1;
  assert(BitsAvail < 63 && "PCRelEncodingSize is too large to use int64_t to"
                           "check for out-of-bounds.");
  int64_t MaxVal = (1ULL << BitsAvail) - 1;
  int64_t MinVal = -(1ULL << BitsAvail);

  uint64_t PCRelTgtAddress = getSymbolAddress(BC, TgtSym, TgtBB);
  int64_t PCOffset = (int64_t)(PCRelTgtAddress - DotAddress);
```

- EN: Declares or implements routines including `getPCRelEncodingSize`, `getSymbolAddress`. Notable symbols here include `getPCRelEncodingSize`, `getSymbolAddress`.
- CN: 这里声明或实现函数，例如 `getPCRelEncodingSize`, `getSymbolAddress`。这里较值得关注的符号包括 `getPCRelEncodingSize`, `getSymbolAddress`。

### Lines 567-577

```cpp
  return PCOffset < MinVal || PCOffset > MaxVal;
}

Error LongJmpPass::relax(BinaryFunction &Func, bool &Modified) {
  const BinaryContext &BC = Func.getBinaryContext();

  assert(BC.isAArch64() && "Unsupported arch");
  constexpr int InsnSize = 4; // AArch64
  std::vector<std::pair<BinaryBasicBlock *, std::unique_ptr<BinaryBasicBlock>>>
      Insertions;
```

- EN: Declares or implements routines including `relax`, `assert`. Notable symbols here include `relax`, `assert`.
- CN: 这里声明或实现函数，例如 `relax`, `assert`。这里较值得关注的符号包括 `relax`, `assert`。

### Lines 578-590

```cpp
  BinaryBasicBlock *Frontier = getBBAtHotColdSplitPoint(Func);
  uint64_t FrontierAddress = Frontier ? BBAddresses[Frontier] : 0;
  if (FrontierAddress)
    FrontierAddress += Frontier->getNumNonPseudos() * InsnSize;

  // Add necessary stubs for branch targets we know we can't fit in the
  // instruction
  for (BinaryBasicBlock &BB : Func) {
    uint64_t DotAddress = BBAddresses[&BB];
    // Stubs themselves are relaxed on the next loop
    if (Stubs[&Func].count(&BB))
      continue;
```

- EN: Declares or implements routines including `getBBAtHotColdSplitPoint`, `getNumNonPseudos`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getBBAtHotColdSplitPoint`, `getNumNonPseudos`.
- CN: 这里声明或实现函数，例如 `getBBAtHotColdSplitPoint`, `getNumNonPseudos`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getBBAtHotColdSplitPoint`, `getNumNonPseudos`。

### Lines 591-599

```cpp
    for (MCInst &Inst : BB) {
      if (BC.MIB->isPseudo(Inst))
        continue;

      if (!mayNeedStub(BC, Inst)) {
        DotAddress += InsnSize;
        continue;
      }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 600-617

```cpp
      // Check and relax direct branch or call
      if (!needsStub(BB, Inst, DotAddress)) {
        DotAddress += InsnSize;
        continue;
      }
      Modified = true;

      // Insert stubs close to the patched BB if call, but far away from the
      // hot path if a branch, since this branch target is the cold region
      // (but first check that the far away stub will be in range).
      BinaryBasicBlock *InsertionPoint = &BB;
      if (Func.isSimple() && !BC.MIB->isCall(Inst) && FrontierAddress &&
          !BB.isCold()) {
        int BitsAvail = BC.MIB->getPCRelEncodingSize(Inst) - 1;
        uint64_t Mask = ~((1ULL << BitsAvail) - 1);
        assert(FrontierAddress > DotAddress &&
               "Hot code should be before the frontier");
        uint64_t PCRelTgt = FrontierAddress - DotAddress;
```

- EN: Declares or implements routines including `getPCRelEncodingSize`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getPCRelEncodingSize`.
- CN: 这里声明或实现函数，例如 `getPCRelEncodingSize`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getPCRelEncodingSize`。

### Lines 618-626

```cpp
        if (!(PCRelTgt & Mask))
          InsertionPoint = Frontier;
      }
      // Always put stubs at the end of the function if non-simple. We can't
      // change the layout of non-simple functions because it has jump tables
      // that we do not control.
      if (!Func.isSimple())
        InsertionPoint = &*std::prev(Func.end());
```

- EN: Declares or implements routines including `prev`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `prev`.
- CN: 这里声明或实现函数，例如 `prev`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `prev`。

### Lines 627-637

```cpp
      // Create a stub to handle a far-away target
      Insertions.emplace_back(InsertionPoint,
                              replaceTargetWithStub(BB, Inst, DotAddress,
                                                    InsertionPoint == Frontier
                                                        ? FrontierAddress
                                                        : DotAddress));

      DotAddress += InsnSize;
    }
  }
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 638-646

```cpp
  // Relax stubs if necessary
  for (BinaryBasicBlock &BB : Func) {
    if (!Stubs[&Func].count(&BB) || !BB.isValid())
      continue;

    if (auto E = relaxStub(BB, Modified))
      return Error(std::move(E));
  }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 647-655

```cpp
  for (std::pair<BinaryBasicBlock *, std::unique_ptr<BinaryBasicBlock>> &Elmt :
       Insertions) {
    if (!Elmt.second)
      continue;
    std::vector<std::unique_ptr<BinaryBasicBlock>> NewBBs;
    NewBBs.emplace_back(std::move(Elmt.second));
    Func.insertBasicBlocks(Elmt.first, std::move(NewBBs), true);
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 656-666

```cpp
  return Error::success();
}

void LongJmpPass::relaxLocalBranches(BinaryFunction &BF) {
  BinaryContext &BC = BF.getBinaryContext();
  auto &MIB = BC.MIB;

  // Quick path.
  if (!BF.isSplit() && BF.estimateSize() < ShortestJumpSpan)
    return;
```

- EN: Declares or implements routines including `relaxLocalBranches`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `relaxLocalBranches`.
- CN: 这里声明或实现函数，例如 `relaxLocalBranches`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `relaxLocalBranches`。

### Lines 667-677

```cpp
  auto isBranchOffsetInRange = [&](const MCInst &Inst, int64_t Offset) {
    const unsigned Bits = MIB->getPCRelEncodingSize(Inst);
    return isIntN(Bits, Offset);
  };

  auto isBlockInRange = [&](const MCInst &Inst, uint64_t InstAddress,
                            const BinaryBasicBlock &BB) {
    const int64_t Offset = BB.getOutputStartAddress() - InstAddress;
    return isBranchOffsetInRange(Inst, Offset);
  };
```

- EN: Declares or implements routines including `getPCRelEncodingSize`. Notable symbols here include `getPCRelEncodingSize`.
- CN: 这里声明或实现函数，例如 `getPCRelEncodingSize`。这里较值得关注的符号包括 `getPCRelEncodingSize`。

### Lines 678-693

```cpp
  // Keep track of *all* function trampolines that are going to be added to the
  // function layout at the end of relaxation.
  std::vector<std::pair<BinaryBasicBlock *, std::unique_ptr<BinaryBasicBlock>>>
      FunctionTrampolines;

  // Function fragments are relaxed independently.
  for (FunctionFragment &FF : BF.getLayout().fragments()) {
    // Fill out code size estimation for the fragment. Use output BB address
    // ranges to store offsets from the start of the function fragment.
    uint64_t CodeSize = 0;
    for (BinaryBasicBlock *BB : FF) {
      BB->setOutputStartAddress(CodeSize);
      CodeSize += BB->estimateSize();
      BB->setOutputEndAddress(CodeSize);
    }
```

- EN: Declares or implements routines including `setOutputStartAddress`, `estimateSize`, `setOutputEndAddress`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `setOutputStartAddress`, `estimateSize`, `setOutputEndAddress`.
- CN: 这里声明或实现函数，例如 `setOutputStartAddress`, `estimateSize`, `setOutputEndAddress`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `setOutputStartAddress`, `estimateSize`, `setOutputEndAddress`。

### Lines 694-703

```cpp
    // Dynamically-updated size of the fragment.
    uint64_t FragmentSize = CodeSize;

    // Size of the trampoline in bytes.
    constexpr uint64_t TrampolineSize = 4;

    // Trampolines created for the fragment. DestinationBB -> TrampolineBB.
    // NB: here we store only the first trampoline created for DestinationBB.
    DenseMap<const BinaryBasicBlock *, BinaryBasicBlock *> FragmentTrampolines;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 704-713

```cpp
    // Create a trampoline code after \p BB or at the end of the fragment if BB
    // is nullptr. If \p UpdateOffsets is true, update FragmentSize and offsets
    // for basic blocks affected by the insertion of the trampoline.
    auto addTrampolineAfter = [&](BinaryBasicBlock *BB,
                                  BinaryBasicBlock *TargetBB, uint64_t Count,
                                  bool UpdateOffsets = true) {
      FunctionTrampolines.emplace_back(BB ? BB : FF.back(),
                                       BF.createBasicBlock());
      BinaryBasicBlock *TrampolineBB = FunctionTrampolines.back().second.get();
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 714-727

```cpp
      MCInst Inst;
      {
        auto L = BC.scopeLock();
        MIB->createUncondBranch(Inst, TargetBB->getLabel(), BC.Ctx.get());
      }
      TrampolineBB->addInstruction(Inst);
      TrampolineBB->addSuccessor(TargetBB, Count);
      TrampolineBB->setExecutionCount(Count);
      const uint64_t TrampolineAddress =
          BB ? BB->getOutputEndAddress() : FragmentSize;
      TrampolineBB->setOutputStartAddress(TrampolineAddress);
      TrampolineBB->setOutputEndAddress(TrampolineAddress + TrampolineSize);
      TrampolineBB->setFragmentNum(FF.getFragmentNum());
```

- EN: Declares or implements routines including `createUncondBranch`, `addInstruction`, `addSuccessor`, `setExecutionCount`, `getOutputEndAddress`, and 3 more. Notable symbols here include `createUncondBranch`, `addInstruction`, `addSuccessor`, `setExecutionCount`, `getOutputEndAddress`, `setOutputStartAddress`.
- CN: 这里声明或实现函数，例如 `createUncondBranch`, `addInstruction`, `addSuccessor`, `setExecutionCount`, `getOutputEndAddress`, and 3 more。这里较值得关注的符号包括 `createUncondBranch`, `addInstruction`, `addSuccessor`, `setExecutionCount`, `getOutputEndAddress`, `setOutputStartAddress`。

### Lines 728-735

```cpp
      if (!FragmentTrampolines.lookup(TargetBB))
        FragmentTrampolines[TargetBB] = TrampolineBB;

      if (!UpdateOffsets)
        return TrampolineBB;

      FragmentSize += TrampolineSize;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 736-749

```cpp
      // If the trampoline was added at the end of the fragment, offsets of
      // other fragments should stay intact.
      if (!BB)
        return TrampolineBB;

      // Update offsets for blocks after BB.
      for (BinaryBasicBlock *IBB : FF) {
        if (IBB->getOutputStartAddress() >= TrampolineAddress) {
          IBB->setOutputStartAddress(IBB->getOutputStartAddress() +
                                     TrampolineSize);
          IBB->setOutputEndAddress(IBB->getOutputEndAddress() + TrampolineSize);
        }
      }
```

- EN: Declares or implements routines including `setOutputStartAddress`, `setOutputEndAddress`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `setOutputStartAddress`, `setOutputEndAddress`.
- CN: 这里声明或实现函数，例如 `setOutputStartAddress`, `setOutputEndAddress`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `setOutputStartAddress`, `setOutputEndAddress`。

### Lines 750-766

```cpp
      // Update offsets for trampolines in this fragment that are placed after
      // the new trampoline. Note that trampoline blocks are not part of the
      // function/fragment layout until we add them right before the return
      // from relaxLocalBranches().
      for (auto &Pair : FunctionTrampolines) {
        BinaryBasicBlock *IBB = Pair.second.get();
        if (IBB->getFragmentNum() != TrampolineBB->getFragmentNum())
          continue;
        if (IBB == TrampolineBB)
          continue;
        if (IBB->getOutputStartAddress() >= TrampolineAddress) {
          IBB->setOutputStartAddress(IBB->getOutputStartAddress() +
                                     TrampolineSize);
          IBB->setOutputEndAddress(IBB->getOutputEndAddress() + TrampolineSize);
        }
      }
```

- EN: Declares or implements routines including `setOutputStartAddress`, `setOutputEndAddress`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `setOutputStartAddress`, `setOutputEndAddress`.
- CN: 这里声明或实现函数，例如 `setOutputStartAddress`, `setOutputEndAddress`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `setOutputStartAddress`, `setOutputEndAddress`。

### Lines 767-776

```cpp
      return TrampolineBB;
    };

    // Pre-populate trampolines by splitting unconditional branches from the
    // containing basic block.
    for (BinaryBasicBlock *BB : FF) {
      MCInst *Inst = BB->getLastNonPseudoInstr();
      if (!Inst || !MIB->isUnconditionalBranch(*Inst))
        continue;
```

- EN: Declares or implements routines including `getLastNonPseudoInstr`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getLastNonPseudoInstr`.
- CN: 这里声明或实现函数，例如 `getLastNonPseudoInstr`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getLastNonPseudoInstr`。

### Lines 777-788

```cpp
      const MCSymbol *TargetSymbol = MIB->getTargetSymbol(*Inst);
      BB->eraseInstruction(BB->findInstruction(Inst));
      BB->setOutputEndAddress(BB->getOutputEndAddress() - TrampolineSize);

      BinaryBasicBlock::BinaryBranchInfo BI;
      BinaryBasicBlock *TargetBB = BB->getSuccessor(TargetSymbol, BI);

      BinaryBasicBlock *TrampolineBB =
          addTrampolineAfter(BB, TargetBB, BI.Count, /*UpdateOffsets*/ false);
      BB->replaceSuccessor(TargetBB, TrampolineBB, BI.Count);
    }
```

- EN: Declares or implements routines including `getTargetSymbol`, `eraseInstruction`, `setOutputEndAddress`, `getSuccessor`, `addTrampolineAfter`, and 1 more. Notable symbols here include `getTargetSymbol`, `eraseInstruction`, `setOutputEndAddress`, `getSuccessor`, `addTrampolineAfter`, `replaceSuccessor`.
- CN: 这里声明或实现函数，例如 `getTargetSymbol`, `eraseInstruction`, `setOutputEndAddress`, `getSuccessor`, `addTrampolineAfter`, and 1 more。这里较值得关注的符号包括 `getTargetSymbol`, `eraseInstruction`, `setOutputEndAddress`, `getSuccessor`, `addTrampolineAfter`, `replaceSuccessor`。

### Lines 789-800

```cpp
    /// Relax the branch \p Inst in basic block \p BB that targets \p TargetBB.
    /// \p InstAddress contains offset of the branch from the start of the
    /// containing function fragment.
    auto relaxBranch = [&](BinaryBasicBlock *BB, MCInst &Inst,
                           uint64_t InstAddress, BinaryBasicBlock *TargetBB) {
      BinaryFunction *BF = BB->getParent();

      // Use branch taken count for optimal relaxation.
      const uint64_t Count = BB->getBranchInfo(*TargetBB).Count;
      assert(Count != BinaryBasicBlock::COUNT_NO_PROFILE &&
             "Expected valid branch execution count");
```

- EN: Declares or implements routines including `getParent`, `getBranchInfo`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getParent`, `getBranchInfo`.
- CN: 这里声明或实现函数，例如 `getParent`, `getBranchInfo`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getParent`, `getBranchInfo`。

### Lines 801-811

```cpp
      // Try to reuse an existing trampoline without introducing any new code.
      BinaryBasicBlock *TrampolineBB = FragmentTrampolines.lookup(TargetBB);
      if (TrampolineBB && isBlockInRange(Inst, InstAddress, *TrampolineBB)) {
        BB->replaceSuccessor(TargetBB, TrampolineBB, Count);
        TrampolineBB->setExecutionCount(TrampolineBB->getExecutionCount() +
                                        Count);
        auto L = BC.scopeLock();
        MIB->replaceBranchTarget(Inst, TrampolineBB->getLabel(), BC.Ctx.get());
        return;
      }
```

- EN: Declares or implements routines including `replaceSuccessor`, `setExecutionCount`, `replaceBranchTarget`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `replaceSuccessor`, `setExecutionCount`, `replaceBranchTarget`.
- CN: 这里声明或实现函数，例如 `replaceSuccessor`, `setExecutionCount`, `replaceBranchTarget`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `replaceSuccessor`, `setExecutionCount`, `replaceBranchTarget`。

### Lines 812-822

```cpp
      // For cold branches, check if we can introduce a trampoline at the end
      // of the fragment that is within the branch reach. Note that such
      // trampoline may change address later and become unreachable in which
      // case we will need further relaxation.
      const int64_t OffsetToEnd = FragmentSize - InstAddress;
      if (Count == 0 && isBranchOffsetInRange(Inst, OffsetToEnd)) {
        TrampolineBB = addTrampolineAfter(nullptr, TargetBB, Count);
        BB->replaceSuccessor(TargetBB, TrampolineBB, Count);
        auto L = BC.scopeLock();
        MIB->replaceBranchTarget(Inst, TrampolineBB->getLabel(), BC.Ctx.get());
```

- EN: Declares or implements routines including `addTrampolineAfter`, `replaceSuccessor`, `replaceBranchTarget`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `addTrampolineAfter`, `replaceSuccessor`, `replaceBranchTarget`.
- CN: 这里声明或实现函数，例如 `addTrampolineAfter`, `replaceSuccessor`, `replaceBranchTarget`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `addTrampolineAfter`, `replaceSuccessor`, `replaceBranchTarget`。

### Lines 823-831

```cpp
        return;
      }

      // If the other successor is a fall-through, invert the condition code.
      BinaryBasicBlock *NextBB =
          BF->getLayout().getBasicBlockAfter(BB, /*IgnoreSplits*/ false);
      bool IsReversibleBranch = MIB->isReversibleBranch(Inst);
      bool ShouldReverseBranch = BB->getConditionalSuccessor(false) == NextBB;
```

- EN: Declares or implements routines including `getLayout`, `isReversibleBranch`, `getConditionalSuccessor`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getLayout`, `isReversibleBranch`, `getConditionalSuccessor`.
- CN: 这里声明或实现函数，例如 `getLayout`, `isReversibleBranch`, `getConditionalSuccessor`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getLayout`, `isReversibleBranch`, `getConditionalSuccessor`。

### Lines 832-840

```cpp
      // Create a trampoline basic block for the fall-through target of the
      // branch if its condition cannot be inverted.
      if (ShouldReverseBranch && !IsReversibleBranch) {
        const uint64_t NextCount = BB->getBranchInfo(*NextBB).Count;
        BinaryBasicBlock *FallThrough =
            addTrampolineAfter(BB, NextBB, NextCount);
        BB->replaceSuccessor(NextBB, FallThrough, NextCount);
      }
```

- EN: Declares or implements routines including `getBranchInfo`, `addTrampolineAfter`, `replaceSuccessor`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getBranchInfo`, `addTrampolineAfter`, `replaceSuccessor`.
- CN: 这里声明或实现函数，例如 `getBranchInfo`, `addTrampolineAfter`, `replaceSuccessor`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getBranchInfo`, `addTrampolineAfter`, `replaceSuccessor`。

### Lines 841-854

```cpp
      // Create a trampoline basic block for the taken target of the branch.
      TrampolineBB = addTrampolineAfter(BB, TargetBB, Count);

      if (ShouldReverseBranch && IsReversibleBranch) {
        BB->swapConditionalSuccessors();
        auto L = BC.scopeLock();
        MIB->reverseBranchCondition(Inst, NextBB->getLabel(), BC.Ctx.get());
      } else {
        auto L = BC.scopeLock();
        MIB->replaceBranchTarget(Inst, TrampolineBB->getLabel(), BC.Ctx.get());
      }
      BB->replaceSuccessor(TargetBB, TrampolineBB, Count);
    };
```

- EN: Declares or implements routines including `addTrampolineAfter`, `swapConditionalSuccessors`, `reverseBranchCondition`, `replaceBranchTarget`, `replaceSuccessor`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `addTrampolineAfter`, `swapConditionalSuccessors`, `reverseBranchCondition`, `replaceBranchTarget`, `replaceSuccessor`.
- CN: 这里声明或实现函数，例如 `addTrampolineAfter`, `swapConditionalSuccessors`, `reverseBranchCondition`, `replaceBranchTarget`, `replaceSuccessor`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `addTrampolineAfter`, `swapConditionalSuccessors`, `reverseBranchCondition`, `replaceBranchTarget`, `replaceSuccessor`。

### Lines 855-867

```cpp
    bool MayNeedRelaxation;
    uint64_t NumIterations = 0;
    do {
      MayNeedRelaxation = false;
      ++NumIterations;
      for (auto BBI = FF.begin(); BBI != FF.end(); ++BBI) {
        BinaryBasicBlock *BB = *BBI;
        uint64_t NextInstOffset = BB->getOutputStartAddress();
        for (MCInst &Inst : *BB) {
          const size_t InstAddress = NextInstOffset;
          if (!MIB->isPseudo(Inst))
            NextInstOffset += 4;
```

- EN: Declares or implements routines including `getOutputStartAddress`. Notable symbols here include `getOutputStartAddress`.
- CN: 这里声明或实现函数，例如 `getOutputStartAddress`。这里较值得关注的符号包括 `getOutputStartAddress`。

### Lines 868-876

```cpp
          if (!mayNeedStub(BF.getBinaryContext(), Inst))
            continue;

          const size_t BitsAvailable = MIB->getPCRelEncodingSize(Inst);

          // Span of +/-128MB.
          if (BitsAvailable == LongestJumpBits)
            continue;
```

- EN: Declares or implements routines including `getPCRelEncodingSize`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getPCRelEncodingSize`.
- CN: 这里声明或实现函数，例如 `getPCRelEncodingSize`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getPCRelEncodingSize`。

### Lines 877-886

```cpp
          const MCSymbol *TargetSymbol = MIB->getTargetSymbol(Inst);
          BinaryBasicBlock *TargetBB = BB->getSuccessor(TargetSymbol);
          assert(TargetBB &&
                 "Basic block target expected for conditional branch.");

          // Check if the relaxation is needed.
          if (TargetBB->getFragmentNum() == FF.getFragmentNum() &&
              isBlockInRange(Inst, InstAddress, *TargetBB))
            continue;
```

- EN: Declares or implements routines including `getTargetSymbol`, `getSuccessor`, `isBlockInRange`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getTargetSymbol`, `getSuccessor`, `isBlockInRange`.
- CN: 这里声明或实现函数，例如 `getTargetSymbol`, `getSuccessor`, `isBlockInRange`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getTargetSymbol`, `getSuccessor`, `isBlockInRange`。

### Lines 887-897

```cpp
          relaxBranch(BB, Inst, InstAddress, TargetBB);

          MayNeedRelaxation = true;
        }
      }

      // We may have added new instructions, but the whole fragment is less than
      // the minimum branch span.
      if (FragmentSize < ShortestJumpSpan)
        MayNeedRelaxation = false;
```

- EN: Declares or implements routines including `relaxBranch`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `relaxBranch`.
- CN: 这里声明或实现函数，例如 `relaxBranch`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `relaxBranch`。

### Lines 898-908

```cpp
    } while (MayNeedRelaxation);

    LLVM_DEBUG({
      if (NumIterations > 2) {
        dbgs() << "BOLT-DEBUG: relaxed fragment " << FF.getFragmentNum().get()
               << " of " << BF << " in " << NumIterations << " iterations\n";
      }
    });
    (void)NumIterations;
  }
```

- EN: Declares or implements routines including `while`, `dbgs`. Notable symbols here include `while`, `dbgs`.
- CN: 这里声明或实现函数，例如 `while`, `dbgs`。这里较值得关注的符号包括 `while`, `dbgs`。

### Lines 909-918

```cpp
  // Add trampoline blocks from all fragments to the layout.
  DenseMap<BinaryBasicBlock *, std::vector<std::unique_ptr<BinaryBasicBlock>>>
      Insertions;
  for (std::pair<BinaryBasicBlock *, std::unique_ptr<BinaryBasicBlock>> &Pair :
       FunctionTrampolines) {
    if (!Pair.second)
      continue;
    Insertions[Pair.first].emplace_back(std::move(Pair.second));
  }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 919-927

```cpp
  for (auto &Pair : Insertions) {
    BF.insertBasicBlocks(Pair.first, std::move(Pair.second),
                         /*UpdateLayout*/ true, /*UpdateCFI*/ true,
                         /*RecomputeLPs*/ false);
  }
}

Error LongJmpPass::runOnFunctions(BinaryContext &BC) {
```

- EN: Declares or implements routines including `runOnFunctions`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `runOnFunctions`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `runOnFunctions`。

### Lines 928-935

```cpp
  assert((opts::CompactCodeModel ||
          opts::SplitStrategy != opts::SplitFunctionsStrategy::CDSplit) &&
         "LongJmp cannot work with functions split in more than two fragments");

  if (opts::CompactCodeModel) {
    BC.outs()
        << "BOLT-INFO: relaxing branches for compact code model (<128MB)\n";
```

- EN: Declares or implements routines including `model`. Notable symbols here include `model`.
- CN: 这里声明或实现函数，例如 `model`。这里较值得关注的符号包括 `model`。

### Lines 936-944

```cpp
    ParallelUtilities::WorkFuncTy WorkFun = [&](BinaryFunction &BF) {
      relaxLocalBranches(BF);
    };

    ParallelUtilities::PredicateTy SkipPredicate =
        [&](const BinaryFunction &BF) {
          return !BC.shouldEmit(BF) || !BF.isSimple();
        };
```

- EN: Declares or implements routines including `relaxLocalBranches`. Notable symbols here include `relaxLocalBranches`.
- CN: 这里声明或实现函数，例如 `relaxLocalBranches`。这里较值得关注的符号包括 `relaxLocalBranches`。

### Lines 945-962

```cpp
    ParallelUtilities::runOnEachFunction(
        BC, ParallelUtilities::SchedulingPolicy::SP_INST_LINEAR, WorkFun,
        SkipPredicate, "RelaxLocalBranches");

    return Error::success();
  }

  BC.outs() << "BOLT-INFO: Starting stub-insertion pass\n";
  BinaryFunctionListType Sorted = BC.getOutputBinaryFunctions();
  bool Modified;
  uint32_t Iterations = 0;
  do {
    ++Iterations;
    Modified = false;
    tentativeLayout(BC, Sorted);
    updateStubGroups();
    for (BinaryFunction *Func : Sorted) {
      if (auto E = relax(*Func, Modified))
```

- EN: Declares or implements routines including `tentativeLayout`, `updateStubGroups`. Notable symbols here include `tentativeLayout`, `updateStubGroups`.
- CN: 这里声明或实现函数，例如 `tentativeLayout`, `updateStubGroups`。这里较值得关注的符号包括 `tentativeLayout`, `updateStubGroups`。

### Lines 963-977

```cpp
        return Error(std::move(E));
      // Don't ruin non-simple functions, they can't afford to have the layout
      // changed.
      if (Modified && Func->isSimple())
        Func->fixBranches();
    }
  } while (Modified);
  BC.outs() << "BOLT-INFO: Inserted " << NumHotStubs
            << " stubs in the hot area and " << NumColdStubs
            << " stubs in the cold area. Shared " << NumSharedStubs
            << " times, iterated " << Iterations << " times.\n";
  return Error::success();
}
} // namespace bolt
} // namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Declares or implements routines including `fixBranches`, `while`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `fixBranches`, `while`, `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里声明或实现函数，例如 `fixBranches`, `while`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `fixBranches`, `while`, `bolt`, `llvm`。

## Key Concepts / 关键概念

- `desc`: function or method entry point / 函数或方法入口
- `init`: function or method entry point / 函数或方法入口
- `relaxStubToShortJmp`: function or method entry point / 函数或方法入口
- `createShortJmp`: function or method entry point / 函数或方法入口
- `applyBTIFixupToTarget`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关
- `llvm`: namespace scope / 命名空间作用域
- `opts`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/LongJmp.h`, `bolt/Core/ParallelUtilities.h`, `bolt/Utils/CommandLineOpts.h`
- LLVM headers / LLVM 头文件: `llvm/Support/MathExtras.h`
- Directory context / 目录上下文: `bolt/lib/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Passes` 下的相邻文件通常与本文件协作组成对应子系统
