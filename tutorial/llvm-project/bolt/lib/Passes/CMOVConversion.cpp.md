# CMOVConversion.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Passes/CMOVConversion.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: bolt/Passes/CMOVConversion.cpp This file implements the CMOV conversion pass.. It also sits in code that implements BOLT optimization and transformation passes. / 该文件实现 BOLT 优化与变换 Pass。 源码头部说明其职责是：bolt/Passes/CMOVConversion.cpp This file implements the CMOV conversion pass.。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Passes/CMOVConversion.cpp ------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the CMOV conversion pass.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-20

```cpp
#include "bolt/Passes/CMOVConversion.h"
#include "bolt/Core/BinaryBasicBlock.h"
#include "bolt/Core/BinaryContext.h"
#include "bolt/Utils/CommandLineOpts.h"
#include "llvm/ADT/PostOrderIterator.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/ErrorHandling.h"
```

- EN: Pulls in 7 header(s) from local project, LLVM dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 7 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 21-28

```cpp
#define DEBUG_TYPE "cmov"

using namespace llvm;

namespace opts {

extern cl::OptionCategory BoltOptCategory;
```

- EN: Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `llvm`, `opts` to organize symbols. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `llvm`, `opts` 中，用于组织符号作用域。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。这里的注释记录了设计假设、不变量或使用说明。

### Lines 29-40

```cpp
static cl::opt<int> BiasThreshold(
    "cmov-conversion-bias-threshold",
    cl::desc("minimum condition bias (pct) to perform a CMOV conversion, "
             "-1 to not account bias"),
    cl::ReallyHidden, cl::init(1), cl::cat(BoltOptCategory));

static cl::opt<int> MispredictionThreshold(
    "cmov-conversion-misprediction-threshold",
    cl::desc("minimum misprediction rate (pct) to perform a CMOV conversion, "
             "-1 to not account misprediction rate"),
    cl::ReallyHidden, cl::init(5), cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `desc`, `init`. Notable symbols here include `desc`, `init`.
- CN: 这里声明或实现函数，例如 `desc`, `init`。这里较值得关注的符号包括 `desc`, `init`。

### Lines 41-51

```cpp
static cl::opt<bool> ConvertStackMemOperand(
    "cmov-conversion-convert-stack-mem-operand",
    cl::desc("convert moves with stack memory operand (potentially unsafe)"),
    cl::ReallyHidden, cl::init(false), cl::cat(BoltOptCategory));

static cl::opt<bool> ConvertBasePtrStackMemOperand(
    "cmov-conversion-convert-rbp-stack-mem-operand",
    cl::desc("convert moves with rbp stack memory operand (unsafe, must be off "
             "for binaries compiled with -fomit-frame-pointer)"),
    cl::ReallyHidden, cl::init(false), cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `desc`, `init`. Notable symbols here include `desc`, `init`.
- CN: 这里声明或实现函数，例如 `desc`, `init`。这里较值得关注的符号包括 `desc`, `init`。

### Lines 52-68

```cpp
} // namespace opts

namespace llvm {
namespace bolt {

// Return true if the CFG conforms to the following subgraph:
// Predecessor
//   /     \
//  |     RHS
//   \     /
//     LHS
// Caller guarantees that LHS and RHS share the same predecessor.
bool isIfThenSubgraph(const BinaryBasicBlock &LHS,
                      const BinaryBasicBlock &RHS) {
  if (LHS.pred_size() != 2 || RHS.pred_size() != 1)
    return false;
```

- EN: Works inside namespace scope `opts`, `llvm`, `bolt` to organize symbols. Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `opts`, `llvm`, `bolt`.
- CN: 这里位于命名空间 `opts`, `llvm`, `bolt` 中，用于组织符号作用域。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `opts`, `llvm`, `bolt`。

### Lines 69-80

```cpp
  // Sanity check
  BinaryBasicBlock *Predecessor = *RHS.pred_begin();
  assert(Predecessor && LHS.isPredecessor(Predecessor) && "invalid subgraph");
  (void)Predecessor;

  if (!LHS.isPredecessor(&RHS))
    return false;
  if (RHS.succ_size() != 1)
    return false;
  return true;
}
```

- EN: Declares or implements routines including `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`。

### Lines 81-91

```cpp
bool matchCFGSubgraph(BinaryBasicBlock &BB, BinaryBasicBlock *&ConditionalSucc,
                      BinaryBasicBlock *&UnconditionalSucc,
                      bool &IsConditionalTaken) {
  BinaryBasicBlock *TakenSucc = BB.getConditionalSuccessor(true);
  BinaryBasicBlock *FallthroughSucc = BB.getConditionalSuccessor(false);
  bool IsIfThenTaken = isIfThenSubgraph(*FallthroughSucc, *TakenSucc);
  bool IsIfThenFallthrough = isIfThenSubgraph(*TakenSucc, *FallthroughSucc);
  if (!IsIfThenFallthrough && !IsIfThenTaken)
    return false;
  assert((!IsIfThenFallthrough || !IsIfThenTaken) && "Invalid subgraph");
```

- EN: Declares or implements routines including `isIfThenSubgraph`, `assert`. Notable symbols here include `isIfThenSubgraph`, `assert`.
- CN: 这里声明或实现函数，例如 `isIfThenSubgraph`, `assert`。这里较值得关注的符号包括 `isIfThenSubgraph`, `assert`。

### Lines 92-109

```cpp
  // Output parameters
  ConditionalSucc = IsIfThenTaken ? TakenSucc : FallthroughSucc;
  UnconditionalSucc = IsIfThenTaken ? FallthroughSucc : TakenSucc;
  IsConditionalTaken = IsIfThenTaken;
  return true;
}

// Return true if basic block instructions can be converted into cmov(s).
bool canConvertInstructions(const BinaryContext &BC, const BinaryBasicBlock &BB,
                            unsigned CC) {
  if (BB.empty())
    return false;
  const MCInst *LastInst = BB.getLastNonPseudoInstr();
  // Only pseudo instructions, can't be converted into CMOV
  if (LastInst == nullptr)
    return false;
  for (const MCInst &Inst : BB) {
    if (BC.MIB->isPseudo(Inst))
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 110-127

```cpp
      continue;
    // Unconditional branch as a last instruction is OK
    if (&Inst == LastInst && BC.MIB->isUnconditionalBranch(Inst))
      continue;
    MCInst Cmov(Inst);
    // GPR move is OK
    if (!BC.MIB->convertMoveToConditionalMove(
            Cmov, CC, opts::ConvertStackMemOperand,
            opts::ConvertBasePtrStackMemOperand)) {
      LLVM_DEBUG({
        dbgs() << BB.getName() << ": can't convert instruction ";
        BC.printInstruction(dbgs(), Cmov);
      });
      return false;
    }
  }
  return true;
}
```

- EN: Declares or implements routines including `Cmov`, `dbgs`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `Cmov`, `dbgs`.
- CN: 这里声明或实现函数，例如 `Cmov`, `dbgs`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `Cmov`, `dbgs`。

### Lines 128-145

```cpp

void convertMoves(const BinaryContext &BC, BinaryBasicBlock &BB, unsigned CC) {
  for (auto II = BB.begin(), IE = BB.end(); II != IE; ++II) {
    if (BC.MIB->isPseudo(*II))
      continue;
    if (BC.MIB->isUnconditionalBranch(*II)) {
      // XXX: this invalidates II but we return immediately
      BB.eraseInstruction(II);
      return;
    }
    bool Result = BC.MIB->convertMoveToConditionalMove(
        *II, CC, opts::ConvertStackMemOperand,
        opts::ConvertBasePtrStackMemOperand);
    assert(Result && "unexpected instruction");
    (void)Result;
  }
}
```

- EN: Declares or implements routines including `convertMoves`, `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `convertMoves`, `assert`.
- CN: 这里声明或实现函数，例如 `convertMoves`, `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `convertMoves`, `assert`。

### Lines 146-161

```cpp
// Returns misprediction rate if the profile data is available, -1 otherwise.
std::pair<int, uint64_t>
calculateMispredictionRate(const BinaryBasicBlock &BB) {
  uint64_t TotalExecCount = 0;
  uint64_t TotalMispredictionCount = 0;
  for (auto BI : BB.branch_info()) {
    TotalExecCount += BI.Count;
    if (BI.MispredictedCount != BinaryBasicBlock::COUNT_INFERRED)
      TotalMispredictionCount += BI.MispredictedCount;
  }
  if (!TotalExecCount)
    return {-1, TotalMispredictionCount};
  return {100.0f * TotalMispredictionCount / TotalExecCount,
          TotalMispredictionCount};
}
```

- EN: Declares or implements routines including `calculateMispredictionRate`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `calculateMispredictionRate`.
- CN: 这里声明或实现函数，例如 `calculateMispredictionRate`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `calculateMispredictionRate`。

### Lines 162-169

```cpp
// Returns conditional succ bias if the profile is available, -1 otherwise.
int calculateConditionBias(const BinaryBasicBlock &BB,
                           const BinaryBasicBlock &ConditionalSucc) {
  if (auto BranchStats = BB.getBranchStats(&ConditionalSucc))
    return BranchStats->first;
  return -1;
}
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 170-179

```cpp
void CMOVConversion::Stats::dumpTo(raw_ostream &OS) {
  OS << "converted static " << StaticPerformed << "/" << StaticPossible
     << formatv(" ({0:P}) ", getStaticRatio())
     << "hammock(s) into CMOV sequences, with dynamic execution count "
     << DynamicPerformed << "/" << DynamicPossible
     << formatv(" ({0:P}), ", getDynamicRatio()) << "saving " << RemovedMP
     << "/" << PossibleMP << formatv(" ({0:P}) ", getMPRatio())
     << "mispredictions\n";
}
```

- EN: Declares or implements routines including `dumpTo`, `formatv`. Notable symbols here include `dumpTo`, `formatv`.
- CN: 这里声明或实现函数，例如 `dumpTo`, `formatv`。这里较值得关注的符号包括 `dumpTo`, `formatv`。

### Lines 180-194

```cpp
void CMOVConversion::runOnFunction(BinaryFunction &Function) {
  BinaryContext &BC = Function.getBinaryContext();
  bool Modified = false;
  // Function-local stats
  Stats Local;
  // Traverse blocks in RPO, merging block with a converted cmov with its
  // successor.
  for (BinaryBasicBlock *BB : post_order(&Function)) {
    uint64_t BBExecCount = BB->getKnownExecutionCount();
    if (BB->empty() ||          // The block must have instructions
        BBExecCount == 0 ||     // must be hot
        BB->succ_size() != 2 || // with two successors
        BB->hasJumpTable())     // no jump table
      continue;
```

- EN: Declares or implements routines including `runOnFunction`, `getKnownExecutionCount`, `succ_size`, `hasJumpTable`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `runOnFunction`, `getKnownExecutionCount`, `succ_size`, `hasJumpTable`.
- CN: 这里声明或实现函数，例如 `runOnFunction`, `getKnownExecutionCount`, `succ_size`, `hasJumpTable`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `runOnFunction`, `getKnownExecutionCount`, `succ_size`, `hasJumpTable`。

### Lines 195-202

```cpp
    assert(BB->isValid() && "traversal internal error");

    // Check branch instruction
    auto BranchInstrIter = BB->getLastNonPseudo();
    if (BranchInstrIter == BB->rend() ||
        !BC.MIB->isConditionalBranch(*BranchInstrIter))
      continue;
```

- EN: Declares or implements routines including `assert`, `getLastNonPseudo`, `isConditionalBranch`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`, `getLastNonPseudo`, `isConditionalBranch`.
- CN: 这里声明或实现函数，例如 `assert`, `getLastNonPseudo`, `isConditionalBranch`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`, `getLastNonPseudo`, `isConditionalBranch`。

### Lines 203-211

```cpp
    // Check successors
    BinaryBasicBlock *ConditionalSucc, *UnconditionalSucc;
    bool IsConditionalTaken;
    if (!matchCFGSubgraph(*BB, ConditionalSucc, UnconditionalSucc,
                          IsConditionalTaken)) {
      LLVM_DEBUG(dbgs() << BB->getName() << ": couldn't match hammock\n");
      continue;
    }
```

- EN: Declares or implements routines including `LLVM_DEBUG`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `LLVM_DEBUG`。

### Lines 212-224

```cpp
    unsigned CC = BC.MIB->getCondCode(*BranchInstrIter);
    if (!IsConditionalTaken)
      CC = BC.MIB->getInvertedCondCode(CC);
    // Check contents of the conditional block
    if (!canConvertInstructions(BC, *ConditionalSucc, CC))
      continue;

    int ConditionBias = calculateConditionBias(*BB, *ConditionalSucc);
    int MispredictionRate = 0;
    uint64_t MispredictionCount = 0;
    std::tie(MispredictionRate, MispredictionCount) =
        calculateMispredictionRate(*BB);
```

- EN: Declares or implements routines including `getCondCode`, `getInvertedCondCode`, `calculateConditionBias`, `tie`, `calculateMispredictionRate`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getCondCode`, `getInvertedCondCode`, `calculateConditionBias`, `tie`, `calculateMispredictionRate`.
- CN: 这里声明或实现函数，例如 `getCondCode`, `getInvertedCondCode`, `calculateConditionBias`, `tie`, `calculateMispredictionRate`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getCondCode`, `getInvertedCondCode`, `calculateConditionBias`, `tie`, `calculateMispredictionRate`。

### Lines 225-237

```cpp
    Local.StaticPossible++;
    Local.DynamicPossible += BBExecCount;
    Local.PossibleMP += MispredictionCount;

    // If the conditional successor is never executed, don't convert it
    if (ConditionBias < opts::BiasThreshold) {
      LLVM_DEBUG(dbgs() << BB->getName() << "->" << ConditionalSucc->getName()
                        << " bias = " << ConditionBias
                        << ", less than threshold " << opts::BiasThreshold
                        << '\n');
      continue;
    }
```

- EN: Declares or implements routines including `LLVM_DEBUG`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `LLVM_DEBUG`。

### Lines 238-245

```cpp
    // Check the misprediction rate of a branch
    if (MispredictionRate < opts::MispredictionThreshold) {
      LLVM_DEBUG(dbgs() << BB->getName() << " misprediction rate = "
                        << MispredictionRate << ", less than threshold "
                        << opts::MispredictionThreshold << '\n');
      continue;
    }
```

- EN: Declares or implements routines including `LLVM_DEBUG`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `LLVM_DEBUG`。

### Lines 246-253

```cpp
    // remove conditional branch
    BB->eraseInstruction(std::prev(BranchInstrIter.base()));
    BB->removeAllSuccessors();
    // Convert instructions from the conditional successor into cmov's in BB.
    convertMoves(BC, *ConditionalSucc, CC);
    BB->addInstructions(ConditionalSucc->begin(), ConditionalSucc->end());
    ConditionalSucc->markValid(false);
```

- EN: Declares or implements routines including `eraseInstruction`, `removeAllSuccessors`, `convertMoves`, `addInstructions`, `markValid`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `eraseInstruction`, `removeAllSuccessors`, `convertMoves`, `addInstructions`, `markValid`.
- CN: 这里声明或实现函数，例如 `eraseInstruction`, `removeAllSuccessors`, `convertMoves`, `addInstructions`, `markValid`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `eraseInstruction`, `removeAllSuccessors`, `convertMoves`, `addInstructions`, `markValid`。

### Lines 254-271

```cpp
    // RPO traversal guarantees that the successor is visited and merged if
    // necessary. Merge the unconditional successor into the current block.
    BB->addInstructions(UnconditionalSucc->begin(), UnconditionalSucc->end());
    UnconditionalSucc->moveAllSuccessorsTo(BB);
    UnconditionalSucc->markValid(false);
    Local.StaticPerformed++;
    Local.DynamicPerformed += BBExecCount;
    Local.RemovedMP += MispredictionCount;
    Modified = true;
  }
  if (Modified)
    Function.eraseInvalidBBs();
  if (opts::Verbosity > 1) {
    BC.outs() << "BOLT-INFO: CMOVConversion: " << Function << ", ";
    Local.dumpTo(BC.outs());
  }
  Global = Global + Local;
}
```

- EN: Declares or implements routines including `addInstructions`, `moveAllSuccessorsTo`, `markValid`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `addInstructions`, `moveAllSuccessorsTo`, `markValid`.
- CN: 这里声明或实现函数，例如 `addInstructions`, `moveAllSuccessorsTo`, `markValid`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `addInstructions`, `moveAllSuccessorsTo`, `markValid`。

### Lines 272-285

```cpp

Error CMOVConversion::runOnFunctions(BinaryContext &BC) {
  if (!BC.isX86()) {
    BC.errs() << "BOLT-ERROR: " << getName() << " is specific to X86\n";
    exit(1);
  }

  for (auto &It : BC.getBinaryFunctions()) {
    BinaryFunction &Function = It.second;
    if (!shouldOptimize(Function))
      continue;
    runOnFunction(Function);
  }
```

- EN: Declares or implements routines including `runOnFunctions`, `exit`, `runOnFunction`. Notable symbols here include `runOnFunctions`, `exit`, `runOnFunction`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`, `exit`, `runOnFunction`。这里较值得关注的符号包括 `runOnFunctions`, `exit`, `runOnFunction`。

### Lines 286-292

```cpp
  BC.outs() << "BOLT-INFO: CMOVConversion total: ";
  Global.dumpTo(BC.outs());
  return Error::success();
}

} // end namespace bolt
} // end namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Contains local control flow that updates state or selects among execution branches. Notable symbols here include `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里较值得关注的符号包括 `bolt`, `llvm`。

## Key Concepts / 关键概念

- `desc`: function or method entry point / 函数或方法入口
- `init`: function or method entry point / 函数或方法入口
- `assert`: function or method entry point / 函数或方法入口
- `isIfThenSubgraph`: function or method entry point / 函数或方法入口
- `Cmov`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关
- `llvm`: namespace scope / 命名空间作用域
- `opts`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/CMOVConversion.h`, `bolt/Core/BinaryBasicBlock.h`, `bolt/Core/BinaryContext.h`, `bolt/Utils/CommandLineOpts.h`
- LLVM headers / LLVM 头文件: `llvm/ADT/PostOrderIterator.h`, `llvm/Support/CommandLine.h`, `llvm/Support/ErrorHandling.h`
- Directory context / 目录上下文: `bolt/lib/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Passes` 下的相邻文件通常与本文件协作组成对应子系统
