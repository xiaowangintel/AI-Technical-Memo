# TailDuplication.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Passes/TailDuplication.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: bolt/Passes/TailDuplication.cpp This file implements the TailDuplication class.. It also sits in code that implements BOLT optimization and transformation passes. / 该文件实现 BOLT 优化与变换 Pass。 源码头部说明其职责是：bolt/Passes/TailDuplication.cpp This file implements the TailDuplication class.。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Passes/TailDuplication.cpp ------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the TailDuplication class.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-21

```cpp
#include "bolt/Passes/TailDuplication.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/MC/MCRegisterInfo.h"

#include <numeric>
#include <queue>

#define DEBUG_TYPE "taildup"
```

- EN: Pulls in 5 header(s) from local project, LLVM, system dependencies needed by this range. Establishes header-guard or prologue macros so the file can be compiled safely. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 5 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。这里的注释记录了设计假设、不变量或使用说明。

### Lines 22-39

```cpp
using namespace llvm;

namespace opts {

extern cl::OptionCategory BoltOptCategory;
extern cl::opt<bool> NoThreads;

static cl::opt<bolt::TailDuplication::DuplicationMode> TailDuplicationMode(
    "tail-duplication",
    cl::desc("duplicate unconditional branches that cross a cache line"),
    cl::init(bolt::TailDuplication::TD_NONE),
    cl::values(clEnumValN(bolt::TailDuplication::TD_NONE, "none",
                          "do not apply"),
               clEnumValN(bolt::TailDuplication::TD_AGGRESSIVE, "aggressive",
                          "aggressive strategy"),
               clEnumValN(bolt::TailDuplication::TD_MODERATE, "moderate",
                          "moderate strategy"),
               clEnumValN(bolt::TailDuplication::TD_CACHE, "cache",
```

- EN: Works inside namespace scope `llvm`, `opts` to organize symbols. Declares or implements routines including `desc`, `init`. Notable symbols here include `desc`, `init`, `llvm`, `opts`.
- CN: 这里位于命名空间 `llvm`, `opts` 中，用于组织符号作用域。这里声明或实现函数，例如 `desc`, `init`。这里较值得关注的符号包括 `desc`, `init`, `llvm`, `opts`。

### Lines 40-49

```cpp
                          "cache-aware duplication strategy")),
    cl::ZeroOrMore, cl::Hidden, cl::cat(BoltOptCategory));

static cl::opt<unsigned>
    TailDuplicationMinimumOffset("tail-duplication-minimum-offset",
                                 cl::desc("minimum offset needed between block "
                                          "and successor to allow duplication"),
                                 cl::ReallyHidden, cl::init(64),
                                 cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `cat`, `init`. Notable symbols here include `cat`, `init`.
- CN: 这里声明或实现函数，例如 `cat`, `init`。这里较值得关注的符号包括 `cat`, `init`。

### Lines 50-61

```cpp
static cl::opt<unsigned> TailDuplicationMaximumDuplication(
    "tail-duplication-maximum-duplication",
    cl::desc("tail blocks whose size (in bytes) exceeds the value are never "
             "duplicated"),
    cl::ZeroOrMore, cl::ReallyHidden, cl::init(24), cl::cat(BoltOptCategory));

static cl::opt<unsigned> TailDuplicationMinimumDuplication(
    "tail-duplication-minimum-duplication",
    cl::desc("tail blocks with size (in bytes) not exceeding the value are "
             "always duplicated"),
    cl::ReallyHidden, cl::init(2), cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `desc`, `init`. Notable symbols here include `desc`, `init`.
- CN: 这里声明或实现函数，例如 `desc`, `init`。这里较值得关注的符号包括 `desc`, `init`。

### Lines 62-71

```cpp
static cl::opt<bool> TailDuplicationConstCopyPropagation(
    "tail-duplication-const-copy-propagation",
    cl::desc("enable const and copy propagation after tail duplication"),
    cl::ReallyHidden, cl::init(false), cl::cat(BoltOptCategory));

static cl::opt<unsigned> TailDuplicationMaxCacheDistance(
    "tail-duplication-max-cache-distance",
    cl::desc("The weight of backward jumps for ExtTSP value"), cl::init(256),
    cl::ReallyHidden, cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `desc`, `init`, `cat`. Notable symbols here include `desc`, `init`, `cat`.
- CN: 这里声明或实现函数，例如 `desc`, `init`, `cat`。这里较值得关注的符号包括 `desc`, `init`, `cat`。

### Lines 72-79

```cpp
static cl::opt<double> TailDuplicationCacheBackwardWeight(
    "tail-duplication-cache-backward-weight",
    cl::desc(
        "The maximum distance (in bytes) of backward jumps for ExtTSP value"),
    cl::init(0.5), cl::ReallyHidden, cl::cat(BoltOptCategory));

} // namespace opts
```

- EN: Works inside namespace scope `opts` to organize symbols. Declares or implements routines including `distance`, `init`. Notable symbols here include `distance`, `init`, `opts`.
- CN: 这里位于命名空间 `opts` 中，用于组织符号作用域。这里声明或实现函数，例如 `distance`, `init`。这里较值得关注的符号包括 `distance`, `init`, `opts`。

### Lines 80-92

```cpp
namespace llvm {
namespace bolt {

void TailDuplication::getCallerSavedRegs(const MCInst &Inst, BitVector &Regs,
                                         BinaryContext &BC) const {
  if (!BC.MIB->isCall(Inst))
    return;
  BitVector CallRegs = BitVector(BC.MRI->getNumRegs(), false);
  BC.MIB->getCalleeSavedRegs(CallRegs);
  CallRegs.flip();
  Regs |= CallRegs;
}
```

- EN: Works inside namespace scope `llvm`, `bolt` to organize symbols. Declares or implements routines including `BitVector`, `getCalleeSavedRegs`. Notable symbols here include `BitVector`, `getCalleeSavedRegs`, `llvm`, `bolt`.
- CN: 这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里声明或实现函数，例如 `BitVector`, `getCalleeSavedRegs`。这里较值得关注的符号包括 `BitVector`, `getCalleeSavedRegs`, `llvm`, `bolt`。

### Lines 93-103

```cpp
bool TailDuplication::regIsPossiblyOverwritten(const MCInst &Inst, unsigned Reg,
                                               BinaryContext &BC) const {
  BitVector WrittenRegs = BitVector(BC.MRI->getNumRegs(), false);
  BC.MIB->getWrittenRegs(Inst, WrittenRegs);
  getCallerSavedRegs(Inst, WrittenRegs, BC);
  if (BC.MIB->isRep(Inst))
    BC.MIB->getRepRegs(WrittenRegs);
  const BitVector &AllAliases = BC.MIB->getAliases(Reg, false);
  return WrittenRegs.anyCommon(AllAliases);
}
```

- EN: Declares or implements routines including `BitVector`, `getWrittenRegs`, `getCallerSavedRegs`, `getRepRegs`, `getAliases`. Notable symbols here include `BitVector`, `getWrittenRegs`, `getCallerSavedRegs`, `getRepRegs`, `getAliases`.
- CN: 这里声明或实现函数，例如 `BitVector`, `getWrittenRegs`, `getCallerSavedRegs`, `getRepRegs`, `getAliases`。这里较值得关注的符号包括 `BitVector`, `getWrittenRegs`, `getCallerSavedRegs`, `getRepRegs`, `getAliases`。

### Lines 104-115

```cpp
bool TailDuplication::regIsDefinitelyOverwritten(const MCInst &Inst,
                                                 unsigned Reg,
                                                 BinaryContext &BC) const {
  BitVector WrittenRegs = BitVector(BC.MRI->getNumRegs(), false);
  BC.MIB->getWrittenRegs(Inst, WrittenRegs);
  getCallerSavedRegs(Inst, WrittenRegs, BC);
  if (BC.MIB->isRep(Inst))
    BC.MIB->getRepRegs(WrittenRegs);
  return (!regIsUsed(Inst, Reg, BC) && WrittenRegs.test(Reg) &&
          !BC.MIB->isConditionalMove(Inst));
}
```

- EN: Declares or implements routines including `BitVector`, `getWrittenRegs`, `getCallerSavedRegs`, `getRepRegs`, `isConditionalMove`. Notable symbols here include `BitVector`, `getWrittenRegs`, `getCallerSavedRegs`, `getRepRegs`, `isConditionalMove`.
- CN: 这里声明或实现函数，例如 `BitVector`, `getWrittenRegs`, `getCallerSavedRegs`, `getRepRegs`, `isConditionalMove`。这里较值得关注的符号包括 `BitVector`, `getWrittenRegs`, `getCallerSavedRegs`, `getRepRegs`, `isConditionalMove`。

### Lines 116-123

```cpp
bool TailDuplication::regIsUsed(const MCInst &Inst, unsigned Reg,
                                BinaryContext &BC) const {
  BitVector SrcRegs = BitVector(BC.MRI->getNumRegs(), false);
  BC.MIB->getSrcRegs(Inst, SrcRegs);
  const BitVector &SmallerAliases = BC.MIB->getAliases(Reg, true);
  return SrcRegs.anyCommon(SmallerAliases);
}
```

- EN: Declares or implements routines including `BitVector`, `getSrcRegs`, `getAliases`. Notable symbols here include `BitVector`, `getSrcRegs`, `getAliases`.
- CN: 这里声明或实现函数，例如 `BitVector`, `getSrcRegs`, `getAliases`。这里较值得关注的符号包括 `BitVector`, `getSrcRegs`, `getAliases`。

### Lines 124-141

```cpp
bool TailDuplication::isOverwrittenBeforeUsed(BinaryBasicBlock &StartBB,
                                              unsigned Reg) const {
  BinaryFunction *BF = StartBB.getFunction();
  BinaryContext &BC = BF->getBinaryContext();
  std::queue<BinaryBasicBlock *> Q;
  for (auto Itr = StartBB.succ_begin(); Itr != StartBB.succ_end(); ++Itr) {
    BinaryBasicBlock *NextBB = *Itr;
    Q.push(NextBB);
  }
  std::set<BinaryBasicBlock *> Visited;
  // Breadth first search through successive blocks and see if Reg is ever used
  // before its overwritten
  while (Q.size() > 0) {
    BinaryBasicBlock *CurrBB = Q.front();
    Q.pop();
    if (Visited.count(CurrBB))
      continue;
    Visited.insert(CurrBB);
```

- EN: Declares or implements routines including `getBinaryContext`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getBinaryContext`.
- CN: 这里声明或实现函数，例如 `getBinaryContext`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getBinaryContext`。

### Lines 142-159

```cpp
    bool Overwritten = false;
    for (auto Itr = CurrBB->begin(); Itr != CurrBB->end(); ++Itr) {
      MCInst &Inst = *Itr;
      if (regIsUsed(Inst, Reg, BC))
        return false;
      if (regIsDefinitelyOverwritten(Inst, Reg, BC)) {
        Overwritten = true;
        break;
      }
    }
    if (Overwritten)
      continue;
    for (auto Itr = CurrBB->succ_begin(); Itr != CurrBB->succ_end(); ++Itr) {
      BinaryBasicBlock *NextBB = *Itr;
      Q.push(NextBB);
    }
  }
  return true;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 160-167

```cpp
}

void TailDuplication::constantAndCopyPropagate(
    BinaryBasicBlock &OriginalBB,
    std::vector<BinaryBasicBlock *> &BlocksToPropagate) {
  BinaryFunction *BF = OriginalBB.getFunction();
  BinaryContext &BC = BF->getBinaryContext();
```

- EN: Declares or implements routines including `getBinaryContext`. Notable symbols here include `getBinaryContext`.
- CN: 这里声明或实现函数，例如 `getBinaryContext`。这里较值得关注的符号包括 `getBinaryContext`。

### Lines 168-177

```cpp
  BlocksToPropagate.insert(BlocksToPropagate.begin(), &OriginalBB);
  // Iterate through the original instructions to find one to propagate
  for (auto Itr = OriginalBB.begin(); Itr != OriginalBB.end();) {
    MCInst &OriginalInst = *Itr;
    // It must be a non conditional
    if (BC.MIB->isConditionalMove(OriginalInst)) {
      ++Itr;
      continue;
    }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 178-186

```cpp
    // Move immediate or move register
    if ((!BC.MII->get(OriginalInst.getOpcode()).isMoveImmediate() ||
         !OriginalInst.getOperand(1).isImm()) &&
        (!BC.MII->get(OriginalInst.getOpcode()).isMoveReg() ||
         !OriginalInst.getOperand(1).isReg())) {
      ++Itr;
      continue;
    }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 187-198

```cpp
    // True if this is constant propagation and not copy propagation
    bool ConstantProp = BC.MII->get(OriginalInst.getOpcode()).isMoveImmediate();
    // The Register to replaced
    unsigned Reg = OriginalInst.getOperand(0).getReg();
    // True if the register to replace was replaced everywhere it was used
    bool ReplacedEverywhere = true;
    // True if the register was definitely overwritten
    bool Overwritten = false;
    // True if the register to replace and the register to replace with (for
    // copy propagation) has not been overwritten and is still usable
    bool RegsActive = true;
```

- EN: Declares or implements routines including `get`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `get`.
- CN: 这里声明或实现函数，例如 `get`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `get`。

### Lines 199-216

```cpp
    // Iterate through successor blocks and through their instructions
    for (BinaryBasicBlock *NextBB : BlocksToPropagate) {
      for (auto PropagateItr =
               ((NextBB == &OriginalBB) ? Itr + 1 : NextBB->begin());
           PropagateItr < NextBB->end(); ++PropagateItr) {
        MCInst &PropagateInst = *PropagateItr;
        if (regIsUsed(PropagateInst, Reg, BC)) {
          bool Replaced = false;
          // If both registers are active for copy propagation or the register
          // to replace is active for constant propagation
          if (RegsActive) {
            // Set Replaced and so ReplacedEverwhere to false if it cannot be
            // replaced (no replacing that opcode, Register is src and dest)
            if (ConstantProp)
              Replaced = BC.MIB->replaceRegWithImm(
                  PropagateInst, Reg, OriginalInst.getOperand(1).getImm());
            else
              Replaced = BC.MIB->replaceRegWithReg(
```

- EN: Declares or implements routines including `end`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `end`.
- CN: 这里声明或实现函数，例如 `end`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `end`。

### Lines 217-227

```cpp
                  PropagateInst, Reg, OriginalInst.getOperand(1).getReg());
          }
          ReplacedEverywhere = ReplacedEverywhere && Replaced;
        }
        // For copy propagation, make sure no propagation happens after the
        // register to replace with is overwritten
        if (!ConstantProp &&
            regIsPossiblyOverwritten(PropagateInst,
                                     OriginalInst.getOperand(1).getReg(), BC))
          RegsActive = false;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 228-242

```cpp
        // Make sure no propagation happens after the register to replace is
        // overwritten
        if (regIsPossiblyOverwritten(PropagateInst, Reg, BC))
          RegsActive = false;

        // Record if the register to replace is overwritten
        if (regIsDefinitelyOverwritten(PropagateInst, Reg, BC)) {
          Overwritten = true;
          break;
        }
      }
      if (Overwritten)
        break;
    }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 243-260

```cpp
    // If the register was replaced everywhere and it was overwritten in either
    // one of the iterated through blocks or one of the successor blocks, delete
    // the original move instruction
    if (ReplacedEverywhere &&
        (Overwritten ||
         isOverwrittenBeforeUsed(
             *BlocksToPropagate[BlocksToPropagate.size() - 1], Reg))) {
      // If both registers are active for copy propagation or the register
      // to replace is active for constant propagation
      StaticInstructionDeletionCount++;
      DynamicInstructionDeletionCount += OriginalBB.getExecutionCount();
      Itr = OriginalBB.eraseInstruction(Itr);
    } else {
      ++Itr;
    }
  }
}
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 261-268

```cpp
bool TailDuplication::isInCacheLine(const BinaryBasicBlock &BB,
                                    const BinaryBasicBlock &Succ) const {
  if (&BB == &Succ)
    return true;

  uint64_t Distance = 0;
  int Direction = (Succ.getLayoutIndex() > BB.getLayoutIndex()) ? 1 : -1;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 269-277

```cpp
  for (unsigned I = BB.getLayoutIndex() + Direction; I != Succ.getLayoutIndex();
       I += Direction) {
    Distance += BB.getFunction()->getLayout().getBlock(I)->getOriginalSize();
    if (Distance > opts::TailDuplicationMinimumOffset)
      return false;
  }
  return true;
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 278-295

```cpp
std::vector<BinaryBasicBlock *>
TailDuplication::moderateDuplicate(BinaryBasicBlock &BB,
                                   BinaryBasicBlock &Tail) const {
  std::vector<BinaryBasicBlock *> BlocksToDuplicate;
  // The block must be hot
  if (BB.getKnownExecutionCount() == 0)
    return BlocksToDuplicate;
  // and its successor is not already in the same cache line
  if (isInCacheLine(BB, Tail))
    return BlocksToDuplicate;
  // and its size do not exceed the maximum allowed size
  if (Tail.getOriginalSize() > opts::TailDuplicationMaximumDuplication)
    return BlocksToDuplicate;
  // If duplicating would introduce a new branch, don't duplicate
  for (auto Itr = Tail.succ_begin(); Itr != Tail.succ_end(); ++Itr) {
    if ((*Itr)->getLayoutIndex() == Tail.getLayoutIndex() + 1)
      return BlocksToDuplicate;
  }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 296-311

```cpp

  BlocksToDuplicate.push_back(&Tail);
  return BlocksToDuplicate;
}

std::vector<BinaryBasicBlock *>
TailDuplication::aggressiveDuplicate(BinaryBasicBlock &BB,
                                     BinaryBasicBlock &Tail) const {
  std::vector<BinaryBasicBlock *> BlocksToDuplicate;
  // The block must be hot
  if (BB.getKnownExecutionCount() == 0)
    return BlocksToDuplicate;
  // and its successor is not already in the same cache line
  if (isInCacheLine(BB, Tail))
    return BlocksToDuplicate;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 312-325

```cpp
  BinaryBasicBlock *CurrBB = &Tail;
  while (CurrBB) {
    LLVM_DEBUG(dbgs() << "Aggressive tail duplication: adding "
                      << CurrBB->getName() << " to duplication list\n";);
    BlocksToDuplicate.push_back(CurrBB);

    if (CurrBB->hasJumpTable()) {
      LLVM_DEBUG(dbgs() << "Aggressive tail duplication: clearing duplication "
                           "list due to a JT in "
                        << CurrBB->getName() << '\n';);
      BlocksToDuplicate.clear();
      break;
    }
```

- EN: Declares or implements routines including `LLVM_DEBUG`, `getName`. Notable symbols here include `LLVM_DEBUG`, `getName`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`, `getName`。这里较值得关注的符号包括 `LLVM_DEBUG`, `getName`。

### Lines 326-343

```cpp
    // With no successors, we've reached the end and should duplicate all of
    // BlocksToDuplicate
    if (CurrBB->succ_size() == 0)
      break;

    // With two successors, if they're both a jump, we should duplicate all
    // blocks in BlocksToDuplicate. Otherwise, we cannot find a simple stream of
    // blocks to copy
    if (CurrBB->succ_size() >= 2) {
      if (CurrBB->getConditionalSuccessor(false)->getLayoutIndex() ==
              CurrBB->getLayoutIndex() + 1 ||
          CurrBB->getConditionalSuccessor(true)->getLayoutIndex() ==
              CurrBB->getLayoutIndex() + 1) {
        LLVM_DEBUG(dbgs() << "Aggressive tail duplication: clearing "
                             "duplication list, can't find a simple stream at "
                          << CurrBB->getName() << '\n';);
        BlocksToDuplicate.clear();
      }
```

- EN: Declares or implements routines including `getLayoutIndex`, `getConditionalSuccessor`, `LLVM_DEBUG`, `getName`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getLayoutIndex`, `getConditionalSuccessor`, `LLVM_DEBUG`, `getName`.
- CN: 这里声明或实现函数，例如 `getLayoutIndex`, `getConditionalSuccessor`, `LLVM_DEBUG`, `getName`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getLayoutIndex`, `getConditionalSuccessor`, `LLVM_DEBUG`, `getName`。

### Lines 344-361

```cpp
      break;
    }

    // With one successor, if its a jump, we should duplicate all blocks in
    // BlocksToDuplicate. Otherwise, we should keep going
    BinaryBasicBlock *SuccBB = CurrBB->getSuccessor();
    if (SuccBB->getLayoutIndex() != CurrBB->getLayoutIndex() + 1)
      break;
    CurrBB = SuccBB;
  }
  // Don't duplicate if its too much code
  unsigned DuplicationByteCount = std::accumulate(
      std::begin(BlocksToDuplicate), std::end(BlocksToDuplicate), 0,
      [](int value, BinaryBasicBlock *p) {
        return value + p->getOriginalSize();
      });
  if (DuplicationByteCount > opts::TailDuplicationMaximumDuplication) {
    LLVM_DEBUG(dbgs() << "Aggressive tail duplication: duplication byte count ("
```

- EN: Declares or implements routines including `getSuccessor`, `begin`, `LLVM_DEBUG`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getSuccessor`, `begin`, `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `getSuccessor`, `begin`, `LLVM_DEBUG`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getSuccessor`, `begin`, `LLVM_DEBUG`。

### Lines 362-370

```cpp
                      << DuplicationByteCount << ") exceeds maximum "
                      << opts::TailDuplicationMaximumDuplication << '\n';);
    BlocksToDuplicate.clear();
  }
  LLVM_DEBUG(dbgs() << "Aggressive tail duplication: found "
                    << BlocksToDuplicate.size() << " blocks to duplicate\n";);
  return BlocksToDuplicate;
}
```

- EN: Declares or implements routines including `LLVM_DEBUG`. Notable symbols here include `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`。这里较值得关注的符号包括 `LLVM_DEBUG`。

### Lines 371-386

```cpp
bool TailDuplication::shouldDuplicate(BinaryBasicBlock *Pred,
                                      BinaryBasicBlock *Tail) const {
  if (Pred == Tail)
    return false;
  // Cannot duplicate non-tail blocks
  if (Tail->succ_size() != 0)
    return false;
  // The blocks are already in the order
  if (Pred->getLayoutIndex() + 1 == Tail->getLayoutIndex())
    return false;
  // No tail duplication for blocks with jump tables
  if (Pred->hasJumpTable())
    return false;
  if (Tail->hasJumpTable())
    return false;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 387-394

```cpp
  return true;
}

double TailDuplication::cacheScore(uint64_t SrcAddr, uint64_t SrcSize,
                                   uint64_t DstAddr, uint64_t DstSize,
                                   uint64_t Count) const {
  assert(Count != BinaryBasicBlock::COUNT_NO_PROFILE);
```

- EN: Declares or implements routines including `assert`. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里较值得关注的符号包括 `assert`。

### Lines 395-404

```cpp
  bool IsForwardJump = SrcAddr <= DstAddr;
  uint64_t JumpDistance = 0;
  // Computing the length of the jump so that it takes the sizes of the two
  // blocks into consideration
  if (IsForwardJump) {
    JumpDistance = (DstAddr + DstSize) - (SrcAddr);
  } else {
    JumpDistance = (SrcAddr + SrcSize) - (DstAddr);
  }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 405-412

```cpp
  if (JumpDistance >= opts::TailDuplicationMaxCacheDistance)
    return 0;
  double Prob = 1.0 - static_cast<double>(JumpDistance) /
                          opts::TailDuplicationMaxCacheDistance;
  return (IsForwardJump ? 1.0 : opts::TailDuplicationCacheBackwardWeight) *
         Prob * Count;
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 413-422

```cpp
bool TailDuplication::cacheScoreImproved(const MCCodeEmitter *Emitter,
                                         BinaryFunction &BF,
                                         BinaryBasicBlock *Pred,
                                         BinaryBasicBlock *Tail) const {
  // Collect (estimated) basic block sizes
  DenseMap<const BinaryBasicBlock *, uint64_t> BBSize;
  for (const BinaryBasicBlock &BB : BF) {
    BBSize[&BB] = std::max<uint64_t>(BB.estimateSize(Emitter), 1);
  }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 423-430

```cpp
  // Build current addresses of basic blocks starting at the entry block
  DenseMap<BinaryBasicBlock *, uint64_t> CurAddr;
  uint64_t Addr = 0;
  for (BinaryBasicBlock *SrcBB : BF.getLayout().blocks()) {
    CurAddr[SrcBB] = Addr;
    Addr += BBSize[SrcBB];
  }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 431-440

```cpp
  // Build new addresses (after duplication) starting at the entry block
  DenseMap<BinaryBasicBlock *, uint64_t> NewAddr;
  Addr = 0;
  for (BinaryBasicBlock *SrcBB : BF.getLayout().blocks()) {
    NewAddr[SrcBB] = Addr;
    Addr += BBSize[SrcBB];
    if (SrcBB == Pred)
      Addr += BBSize[Tail];
  }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 441-453

```cpp
  // Compute the cache score for the existing layout of basic blocks
  double CurScore = 0;
  for (BinaryBasicBlock *SrcBB : BF.getLayout().blocks()) {
    auto BI = SrcBB->branch_info_begin();
    for (BinaryBasicBlock *DstBB : SrcBB->successors()) {
      if (SrcBB != DstBB) {
        CurScore += cacheScore(CurAddr[SrcBB], BBSize[SrcBB], CurAddr[DstBB],
                               BBSize[DstBB], BI->Count);
      }
      ++BI;
    }
  }
```

- EN: Declares or implements routines including `branch_info_begin`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `branch_info_begin`.
- CN: 这里声明或实现函数，例如 `branch_info_begin`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `branch_info_begin`。

### Lines 454-471

```cpp
  // Compute the cache score for the layout of blocks after tail duplication
  double NewScore = 0;
  for (BinaryBasicBlock *SrcBB : BF.getLayout().blocks()) {
    auto BI = SrcBB->branch_info_begin();
    for (BinaryBasicBlock *DstBB : SrcBB->successors()) {
      if (SrcBB != DstBB) {
        if (SrcBB == Pred && DstBB == Tail) {
          NewScore += cacheScore(NewAddr[SrcBB], BBSize[SrcBB],
                                 NewAddr[SrcBB] + BBSize[SrcBB], BBSize[DstBB],
                                 BI->Count);
        } else {
          NewScore += cacheScore(NewAddr[SrcBB], BBSize[SrcBB], NewAddr[DstBB],
                                 BBSize[DstBB], BI->Count);
        }
      }
      ++BI;
    }
  }
```

- EN: Declares or implements routines including `branch_info_begin`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `branch_info_begin`.
- CN: 这里声明或实现函数，例如 `branch_info_begin`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `branch_info_begin`。

### Lines 472-481

```cpp

  return NewScore > CurScore;
}

std::vector<BinaryBasicBlock *>
TailDuplication::cacheDuplicate(const MCCodeEmitter *Emitter,
                                BinaryFunction &BF, BinaryBasicBlock *Pred,
                                BinaryBasicBlock *Tail) const {
  std::vector<BinaryBasicBlock *> BlocksToDuplicate;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 482-499

```cpp
  // No need to duplicate cold basic blocks
  if (Pred->isCold() || Tail->isCold()) {
    return BlocksToDuplicate;
  }
  // Always duplicate "small" tail basic blocks, which might be beneficial for
  // code size, since a jump instruction is eliminated
  if (Tail->estimateSize(Emitter) <= opts::TailDuplicationMinimumDuplication) {
    BlocksToDuplicate.push_back(Tail);
    return BlocksToDuplicate;
  }
  // Never duplicate "large" tail basic blocks
  if (Tail->estimateSize(Emitter) > opts::TailDuplicationMaximumDuplication) {
    return BlocksToDuplicate;
  }
  // Do not append basic blocks after the last hot block in the current layout
  auto NextBlock = BF.getLayout().getBasicBlockAfter(Pred);
  if (NextBlock == nullptr || (!Pred->isCold() && NextBlock->isCold())) {
    return BlocksToDuplicate;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 500-509

```cpp
  }

  // Duplicate the tail only if it improves the cache score
  if (cacheScoreImproved(Emitter, BF, Pred, Tail)) {
    BlocksToDuplicate.push_back(Tail);
  }

  return BlocksToDuplicate;
}
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 510-524

```cpp
std::vector<BinaryBasicBlock *> TailDuplication::duplicateBlocks(
    BinaryBasicBlock &BB,
    const std::vector<BinaryBasicBlock *> &BlocksToDuplicate) const {
  BinaryFunction *BF = BB.getFunction();
  BinaryContext &BC = BF->getBinaryContext();

  // Ratio of this new branches execution count to the total size of the
  // successor's execution count.  Used to set this new branches execution count
  // and lower the old successor's execution count
  double ExecutionCountRatio =
      BB.getExecutionCount() >= BB.getSuccessor()->getExecutionCount()
          ? 1.0
          : (double)BB.getExecutionCount() /
                BB.getSuccessor()->getExecutionCount();
```

- EN: Declares or implements routines including `getBinaryContext`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getBinaryContext`.
- CN: 这里声明或实现函数，例如 `getBinaryContext`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getBinaryContext`。

### Lines 525-534

```cpp
  // Use the last branch info when adding a successor to LastBB
  BinaryBasicBlock::BinaryBranchInfo &LastBI =
      BB.getBranchInfo(*(BB.getSuccessor()));

  BinaryBasicBlock *LastOriginalBB = &BB;
  BinaryBasicBlock *LastDuplicatedBB = &BB;
  assert(LastDuplicatedBB->succ_size() == 1 &&
         "tail duplication cannot act on a block with more than 1 successor");
  LastDuplicatedBB->removeSuccessor(LastDuplicatedBB->getSuccessor());
```

- EN: Declares or implements routines including `assert`, `removeSuccessor`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`, `removeSuccessor`.
- CN: 这里声明或实现函数，例如 `assert`, `removeSuccessor`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`, `removeSuccessor`。

### Lines 535-542

```cpp
  std::vector<std::unique_ptr<BinaryBasicBlock>> DuplicatedBlocks;
  std::vector<BinaryBasicBlock *> DuplicatedBlocksToReturn;

  for (BinaryBasicBlock *CurBB : BlocksToDuplicate) {
    DuplicatedBlocks.emplace_back(
        BF->createBasicBlock((BC.Ctx)->createNamedTempSymbol("tail-dup")));
    BinaryBasicBlock *NewBB = DuplicatedBlocks.back().get();
```

- EN: Declares or implements routines including `createBasicBlock`. Notable symbols here include `createBasicBlock`.
- CN: 这里声明或实现函数，例如 `createBasicBlock`。这里较值得关注的符号包括 `createBasicBlock`。

### Lines 543-550

```cpp
    NewBB->addInstructions(CurBB->begin(), CurBB->end());
    // Set execution count as if it was just a copy of the original
    NewBB->setExecutionCount(CurBB->getExecutionCount());
    NewBB->setIsCold(CurBB->isCold());
    LastDuplicatedBB->addSuccessor(NewBB, LastBI);

    DuplicatedBlocksToReturn.push_back(NewBB);
```

- EN: Declares or implements routines including `addInstructions`, `setExecutionCount`, `setIsCold`, `addSuccessor`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `addInstructions`, `setExecutionCount`, `setIsCold`, `addSuccessor`.
- CN: 这里声明或实现函数，例如 `addInstructions`, `setExecutionCount`, `setIsCold`, `addSuccessor`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `addInstructions`, `setExecutionCount`, `setIsCold`, `addSuccessor`。

### Lines 551-560

```cpp
    // As long as its not the first block, adjust both original and duplicated
    // to what they should be
    if (LastDuplicatedBB != &BB) {
      LastOriginalBB->adjustExecutionCount(1.0 - ExecutionCountRatio);
      LastDuplicatedBB->adjustExecutionCount(ExecutionCountRatio);
    }

    if (CurBB->succ_size() == 1)
      LastBI = CurBB->getBranchInfo(*(CurBB->getSuccessor()));
```

- EN: Declares or implements routines including `adjustExecutionCount`, `getBranchInfo`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `adjustExecutionCount`, `getBranchInfo`.
- CN: 这里声明或实现函数，例如 `adjustExecutionCount`, `getBranchInfo`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `adjustExecutionCount`, `getBranchInfo`。

### Lines 561-568

```cpp
    LastOriginalBB = CurBB;
    LastDuplicatedBB = NewBB;
  }

  LastDuplicatedBB->addSuccessors(
      LastOriginalBB->succ_begin(), LastOriginalBB->succ_end(),
      LastOriginalBB->branch_info_begin(), LastOriginalBB->branch_info_end());
```

- EN: Declares or implements routines including `succ_begin`, `branch_info_begin`. Notable symbols here include `succ_begin`, `branch_info_begin`.
- CN: 这里声明或实现函数，例如 `succ_begin`, `branch_info_begin`。这里较值得关注的符号包括 `succ_begin`, `branch_info_begin`。

### Lines 569-576

```cpp
  LastOriginalBB->adjustExecutionCount(1.0 - ExecutionCountRatio);
  LastDuplicatedBB->adjustExecutionCount(ExecutionCountRatio);

  BF->insertBasicBlocks(&BB, std::move(DuplicatedBlocks));

  return DuplicatedBlocksToReturn;
}
```

- EN: Declares or implements routines including `adjustExecutionCount`, `insertBasicBlocks`. Notable symbols here include `adjustExecutionCount`, `insertBasicBlocks`.
- CN: 这里声明或实现函数，例如 `adjustExecutionCount`, `insertBasicBlocks`。这里较值得关注的符号包括 `adjustExecutionCount`, `insertBasicBlocks`。

### Lines 577-585

```cpp
void TailDuplication::runOnFunction(BinaryFunction &Function) {
  // Create a separate MCCodeEmitter to allow lock-free execution
  BinaryContext::IndependentCodeEmitter Emitter;
  if (!opts::NoThreads) {
    Emitter = Function.getBinaryContext().createIndependentMCCodeEmitter();
  }

  Function.getLayout().updateLayoutIndices();
```

- EN: Declares or implements routines including `runOnFunction`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `runOnFunction`.
- CN: 这里声明或实现函数，例如 `runOnFunction`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `runOnFunction`。

### Lines 586-593

```cpp
  // New blocks will be added and layout will change,
  // so make a copy here to iterate over the original layout
  BinaryFunction::BasicBlockOrderType BlockLayout(
      Function.getLayout().block_begin(), Function.getLayout().block_end());
  bool ModifiedFunction = false;
  for (BinaryBasicBlock *BB : BlockLayout) {
    AllDynamicCount += BB->getKnownExecutionCount();
```

- EN: Declares or implements routines including `getKnownExecutionCount`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getKnownExecutionCount`.
- CN: 这里声明或实现函数，例如 `getKnownExecutionCount`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getKnownExecutionCount`。

### Lines 594-601

```cpp
    // The block must be with one successor
    if (BB->succ_size() != 1)
      continue;
    BinaryBasicBlock *Tail = BB->getSuccessor();
    // Verify that the tail should be duplicated
    if (!shouldDuplicate(BB, Tail))
      continue;
```

- EN: Declares or implements routines including `getSuccessor`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getSuccessor`.
- CN: 这里声明或实现函数，例如 `getSuccessor`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getSuccessor`。

### Lines 602-612

```cpp
    std::vector<BinaryBasicBlock *> BlocksToDuplicate;
    if (opts::TailDuplicationMode == TailDuplication::TD_AGGRESSIVE) {
      BlocksToDuplicate = aggressiveDuplicate(*BB, *Tail);
    } else if (opts::TailDuplicationMode == TailDuplication::TD_MODERATE) {
      BlocksToDuplicate = moderateDuplicate(*BB, *Tail);
    } else if (opts::TailDuplicationMode == TailDuplication::TD_CACHE) {
      BlocksToDuplicate = cacheDuplicate(Emitter.MCE.get(), Function, BB, Tail);
    } else {
      llvm_unreachable("unknown tail duplication mode");
    }
```

- EN: Declares or implements routines including `aggressiveDuplicate`, `if`, `moderateDuplicate`, `cacheDuplicate`, `llvm_unreachable`. Notable symbols here include `aggressiveDuplicate`, `if`, `moderateDuplicate`, `cacheDuplicate`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `aggressiveDuplicate`, `if`, `moderateDuplicate`, `cacheDuplicate`, `llvm_unreachable`。这里较值得关注的符号包括 `aggressiveDuplicate`, `if`, `moderateDuplicate`, `cacheDuplicate`, `llvm_unreachable`。

### Lines 613-624

```cpp
    if (BlocksToDuplicate.empty())
      continue;

    // Apply the duplication
    ModifiedFunction = true;
    DuplicationsDynamicCount += BB->getExecutionCount();
    auto DuplicatedBlocks = duplicateBlocks(*BB, BlocksToDuplicate);
    for (BinaryBasicBlock *BB : DuplicatedBlocks) {
      DuplicatedBlockCount++;
      DuplicatedByteCount += BB->estimateSize(Emitter.MCE.get());
    }
```

- EN: Declares or implements routines including `getExecutionCount`, `duplicateBlocks`, `estimateSize`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getExecutionCount`, `duplicateBlocks`, `estimateSize`.
- CN: 这里声明或实现函数，例如 `getExecutionCount`, `duplicateBlocks`, `estimateSize`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getExecutionCount`, `duplicateBlocks`, `estimateSize`。

### Lines 625-634

```cpp
    if (opts::TailDuplicationConstCopyPropagation) {
      constantAndCopyPropagate(*BB, DuplicatedBlocks);
      BinaryBasicBlock *FirstBB = BlocksToDuplicate[0];
      if (FirstBB->pred_size() == 1) {
        BinaryBasicBlock *PredBB = *FirstBB->pred_begin();
        if (PredBB->succ_size() == 1)
          constantAndCopyPropagate(*PredBB, BlocksToDuplicate);
      }
    }
```

- EN: Declares or implements routines including `constantAndCopyPropagate`, `pred_begin`. Notable symbols here include `constantAndCopyPropagate`, `pred_begin`.
- CN: 这里声明或实现函数，例如 `constantAndCopyPropagate`, `pred_begin`。这里较值得关注的符号包括 `constantAndCopyPropagate`, `pred_begin`。

### Lines 635-645

```cpp
    // Layout indices might be stale after duplication
    Function.getLayout().updateLayoutIndices();
  }
  if (ModifiedFunction)
    ModifiedFunctions++;
}

Error TailDuplication::runOnFunctions(BinaryContext &BC) {
  if (opts::TailDuplicationMode == TailDuplication::TD_NONE)
    return Error::success();
```

- EN: Declares or implements routines including `runOnFunctions`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `runOnFunctions`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `runOnFunctions`。

### Lines 646-663

```cpp
  for (auto &It : BC.getBinaryFunctions()) {
    BinaryFunction &Function = It.second;
    if (!shouldOptimize(Function))
      continue;
    runOnFunction(Function);
  }

  BC.outs()
      << "BOLT-INFO: tail duplication"
      << format(" modified %zu (%.2f%%) functions;", ModifiedFunctions,
                100.0 * ModifiedFunctions / BC.getBinaryFunctions().size())
      << format(" duplicated %zu blocks (%zu bytes) responsible for",
                DuplicatedBlockCount, DuplicatedByteCount)
      << format(" %zu dynamic executions (%.2f%% of all block executions)",
                DuplicationsDynamicCount,
                100.0 * DuplicationsDynamicCount / AllDynamicCount)
      << "\n";
```

- EN: Declares or implements routines including `runOnFunction`, `format`. Notable symbols here include `runOnFunction`, `format`.
- CN: 这里声明或实现函数，例如 `runOnFunction`, `format`。这里较值得关注的符号包括 `runOnFunction`, `format`。

### Lines 664-674

```cpp
  if (opts::TailDuplicationConstCopyPropagation) {
    BC.outs() << "BOLT-INFO: tail duplication "
              << format(
                     "applied %zu static and %zu dynamic propagation deletions",
                     StaticInstructionDeletionCount,
                     DynamicInstructionDeletionCount)
              << "\n";
  }
  return Error::success();
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 675-676

```cpp
} // end namespace bolt
} // end namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Notable symbols here include `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里较值得关注的符号包括 `bolt`, `llvm`。

## Key Concepts / 关键概念

- `desc`: function or method entry point / 函数或方法入口
- `init`: function or method entry point / 函数或方法入口
- `cat`: function or method entry point / 函数或方法入口
- `distance`: function or method entry point / 函数或方法入口
- `BitVector`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关
- `llvm`: namespace scope / 命名空间作用域
- `opts`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/TailDuplication.h`
- LLVM headers / LLVM 头文件: `llvm/ADT/DenseMap.h`, `llvm/MC/MCRegisterInfo.h`
- System headers / 系统头文件: `numeric`, `queue`
- Directory context / 目录上下文: `bolt/lib/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Passes` 下的相邻文件通常与本文件协作组成对应子系统
