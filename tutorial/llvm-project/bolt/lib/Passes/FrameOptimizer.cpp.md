# FrameOptimizer.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Passes/FrameOptimizer.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: bolt/Passes/FrameOptimizer.cpp This file implements the FrameOptimizerPass class.. It also sits in code that implements BOLT optimization and transformation passes. / 该文件实现 BOLT 优化与变换 Pass。 源码头部说明其职责是：bolt/Passes/FrameOptimizer.cpp This file implements the FrameOptimizerPass class.。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Passes/FrameOptimizer.cpp -------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the FrameOptimizerPass class.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-23

```cpp
#include "bolt/Passes/FrameOptimizer.h"
#include "bolt/Core/BinaryFunctionCallGraph.h"
#include "bolt/Core/ParallelUtilities.h"
#include "bolt/Passes/DataflowInfoManager.h"
#include "bolt/Passes/ShrinkWrapping.h"
#include "bolt/Passes/StackAvailableExpressions.h"
#include "bolt/Passes/StackReachingUses.h"
#include "bolt/Utils/CommandLineOpts.h"
#include "llvm/Support/Timer.h"
#include <deque>
```

- EN: Pulls in 10 header(s) from local project, LLVM, system dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 10 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 24-32

```cpp
#define DEBUG_TYPE "fop"

using namespace llvm;

namespace opts {
extern cl::opt<unsigned> Verbosity;
extern cl::opt<bool> TimeOpts;
extern cl::OptionCategory BoltOptCategory;
```

- EN: Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `llvm`, `opts` to organize symbols. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `llvm`, `opts` 中，用于组织符号作用域。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。这里的注释记录了设计假设、不变量或使用说明。

### Lines 33-45

```cpp
using namespace bolt;

cl::opt<FrameOptimizationType>
FrameOptimization("frame-opt",
  cl::init(FOP_NONE),
  cl::desc("optimize stack frame accesses"),
  cl::values(
    clEnumValN(FOP_NONE, "none", "do not perform frame optimization"),
    clEnumValN(FOP_HOT, "hot", "perform FOP on hot functions"),
    clEnumValN(FOP_ALL, "all", "perform FOP on all functions")),
  cl::ZeroOrMore,
  cl::cat(BoltOptCategory));
```

- EN: Works inside namespace scope `bolt` to organize symbols. Declares or implements routines including `init`, `desc`, `clEnumValN`, `cat`. Notable symbols here include `init`, `desc`, `clEnumValN`, `cat`, `bolt`.
- CN: 这里位于命名空间 `bolt` 中，用于组织符号作用域。这里声明或实现函数，例如 `init`, `desc`, `clEnumValN`, `cat`。这里较值得关注的符号包括 `init`, `desc`, `clEnumValN`, `cat`, `bolt`。

### Lines 46-55

```cpp
static cl::opt<bool> RemoveStores(
    "frame-opt-rm-stores", cl::init(FOP_NONE),
    cl::desc("apply additional analysis to remove stores (experimental)"),
    cl::cat(BoltOptCategory));

} // namespace opts

namespace llvm {
namespace bolt {
```

- EN: Works inside namespace scope `opts`, `llvm`, `bolt` to organize symbols. Declares or implements routines including `init`, `desc`, `cat`. Notable symbols here include `init`, `desc`, `cat`, `opts`, `llvm`, `bolt`.
- CN: 这里位于命名空间 `opts`, `llvm`, `bolt` 中，用于组织符号作用域。这里声明或实现函数，例如 `init`, `desc`, `cat`。这里较值得关注的符号包括 `init`, `desc`, `cat`, `opts`, `llvm`, `bolt`。

### Lines 56-73

```cpp
void FrameOptimizerPass::removeUnnecessaryLoads(const RegAnalysis &RA,
                                                const FrameAnalysis &FA,
                                                BinaryFunction &BF) {
  StackAvailableExpressions SAE(RA, FA, BF);
  SAE.run();

  LLVM_DEBUG(dbgs() << "Performing unnecessary loads removal\n");
  std::deque<std::pair<BinaryBasicBlock *, MCInst *>> ToErase;
  bool Changed = false;
  const auto ExprEnd = SAE.expr_end();
  MCPlusBuilder *MIB = BF.getBinaryContext().MIB.get();
  for (BinaryBasicBlock &BB : BF) {
    LLVM_DEBUG(dbgs() << "\tNow at BB " << BB.getName() << "\n");
    const MCInst *Prev = nullptr;
    for (MCInst &Inst : BB) {
      LLVM_DEBUG({
        dbgs() << "\t\tNow at ";
        Inst.dump();
```

- EN: Declares or implements routines including `SAE`, `LLVM_DEBUG`, `dbgs`. Notable symbols here include `SAE`, `LLVM_DEBUG`, `dbgs`.
- CN: 这里声明或实现函数，例如 `SAE`, `LLVM_DEBUG`, `dbgs`。这里较值得关注的符号包括 `SAE`, `LLVM_DEBUG`, `dbgs`。

### Lines 74-91

```cpp
        for (auto I = Prev ? SAE.expr_begin(*Prev) : SAE.expr_begin(BB);
             I != ExprEnd; ++I) {
          dbgs() << "\t\t\tReached by: ";
          (*I)->dump();
        }
      });
      // if Inst is a load from stack and the current available expressions show
      // this value is available in a register or immediate, replace this load
      // with move from register or from immediate.
      ErrorOr<const FrameIndexEntry &> FIEX = FA.getFIEFor(Inst);
      if (!FIEX) {
        Prev = &Inst;
        continue;
      }
      // FIXME: Change to remove IsSimple == 0. We're being conservative here,
      // but once replaceMemOperandWithReg is ready, we should feed it with all
      // sorts of complex instructions.
      if (FIEX->IsLoad == false || FIEX->IsSimple == false ||
```

- EN: Declares or implements routines including `dbgs`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `dbgs`.
- CN: 这里声明或实现函数，例如 `dbgs`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `dbgs`。

### Lines 92-109

```cpp
          FIEX->StackOffset >= 0) {
        Prev = &Inst;
        continue;
      }

      for (auto I = Prev ? SAE.expr_begin(*Prev) : SAE.expr_begin(BB);
           I != ExprEnd; ++I) {
        const MCInst *AvailableInst = *I;
        ErrorOr<const FrameIndexEntry &> FIEY = FA.getFIEFor(*AvailableInst);
        if (!FIEY)
          continue;
        assert(FIEY->IsStore && FIEY->IsSimple);
        if (FIEX->StackOffset != FIEY->StackOffset || FIEX->Size != FIEY->Size)
          continue;
        // TODO: Change push/pops to stack adjustment instruction
        if (MIB->isPop(Inst))
          continue;
```

- EN: Declares or implements routines including `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`。

### Lines 110-127

```cpp
        ++NumRedundantLoads;
        FreqRedundantLoads += BB.getKnownExecutionCount();
        Changed = true;
        LLVM_DEBUG(dbgs() << "Redundant load instruction: ");
        LLVM_DEBUG(Inst.dump());
        LLVM_DEBUG(dbgs() << "Related store instruction: ");
        LLVM_DEBUG(AvailableInst->dump());
        LLVM_DEBUG(dbgs() << "@BB: " << BB.getName() << "\n");
        // Replace load
        if (FIEY->IsStoreFromReg) {
          if (!MIB->replaceMemOperandWithReg(Inst, FIEY->RegOrImm)) {
            LLVM_DEBUG(dbgs() << "FAILED to change operand to a reg\n");
            break;
          }
          FreqLoadsChangedToReg += BB.getKnownExecutionCount();
          MIB->removeAnnotation(Inst, "FrameAccessEntry");
          LLVM_DEBUG(dbgs() << "Changed operand to a reg\n");
          if (MIB->isRedundantMove(Inst)) {
```

- EN: Declares or implements routines including `LLVM_DEBUG`, `removeAnnotation`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `LLVM_DEBUG`, `removeAnnotation`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`, `removeAnnotation`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `LLVM_DEBUG`, `removeAnnotation`。

### Lines 128-145

```cpp
            ++NumLoadsDeleted;
            FreqLoadsDeleted += BB.getKnownExecutionCount();
            LLVM_DEBUG(dbgs() << "Created a redundant move\n");
            // Delete it!
            ToErase.push_front(std::make_pair(&BB, &Inst));
          }
        } else {
          char Buf[8] = {0, 0, 0, 0, 0, 0, 0, 0};
          support::ulittle64_t::ref(Buf + 0) = FIEY->RegOrImm;
          LLVM_DEBUG(dbgs() << "Changing operand to an imm... ");
          if (!MIB->replaceMemOperandWithImm(Inst, StringRef(Buf, 8), 0)) {
            LLVM_DEBUG(dbgs() << "FAILED\n");
          } else {
            FreqLoadsChangedToImm += BB.getKnownExecutionCount();
            MIB->removeAnnotation(Inst, "FrameAccessEntry");
            LLVM_DEBUG(dbgs() << "Ok\n");
          }
        }
```

- EN: Declares or implements routines including `LLVM_DEBUG`, `ref`, `removeAnnotation`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `LLVM_DEBUG`, `ref`, `removeAnnotation`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`, `ref`, `removeAnnotation`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `LLVM_DEBUG`, `ref`, `removeAnnotation`。

### Lines 146-155

```cpp
        LLVM_DEBUG(dbgs() << "Changed to: ");
        LLVM_DEBUG(Inst.dump());
        break;
      }
      Prev = &Inst;
    }
  }
  if (Changed)
    LLVM_DEBUG(dbgs() << "FOP modified \"" << BF.getPrintName() << "\"\n");
```

- EN: Declares or implements routines including `LLVM_DEBUG`. Notable symbols here include `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`。这里较值得关注的符号包括 `LLVM_DEBUG`。

### Lines 156-166

```cpp
  // TODO: Implement an interface of eraseInstruction that works out the
  // complete list of elements to remove.
  for (std::pair<BinaryBasicBlock *, MCInst *> I : ToErase)
    I.first->eraseInstruction(I.first->findInstruction(I.second));
}

void FrameOptimizerPass::removeUnusedStores(const FrameAnalysis &FA,
                                            BinaryFunction &BF) {
  StackReachingUses SRU(FA, BF);
  SRU.run();
```

- EN: Declares or implements routines including `eraseInstruction`, `SRU`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `eraseInstruction`, `SRU`.
- CN: 这里声明或实现函数，例如 `eraseInstruction`, `SRU`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `eraseInstruction`, `SRU`。

### Lines 167-184

```cpp
  LLVM_DEBUG(dbgs() << "Performing unused stores removal\n");
  std::vector<std::pair<BinaryBasicBlock *, MCInst *>> ToErase;
  bool Changed = false;
  for (BinaryBasicBlock &BB : BF) {
    LLVM_DEBUG(dbgs() << "\tNow at BB " << BB.getName() << "\n");
    const MCInst *Prev = nullptr;
    for (MCInst &Inst : llvm::reverse(BB)) {
      LLVM_DEBUG({
        dbgs() << "\t\tNow at ";
        Inst.dump();
        for (auto I = Prev ? SRU.expr_begin(*Prev) : SRU.expr_begin(BB);
             I != SRU.expr_end(); ++I) {
          dbgs() << "\t\t\tReached by: ";
          (*I)->dump();
        }
      });
      ErrorOr<const FrameIndexEntry &> FIEX = FA.getFIEFor(Inst);
      if (!FIEX) {
```

- EN: Declares or implements routines including `LLVM_DEBUG`, `dbgs`. Notable symbols here include `LLVM_DEBUG`, `dbgs`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`, `dbgs`。这里较值得关注的符号包括 `LLVM_DEBUG`, `dbgs`。

### Lines 185-192

```cpp
        Prev = &Inst;
        continue;
      }
      if (FIEX->IsLoad || !FIEX->IsSimple || FIEX->StackOffset >= 0) {
        Prev = &Inst;
        continue;
      }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 193-201

```cpp
      if (SRU.isStoreUsed(*FIEX,
                          Prev ? SRU.expr_begin(*Prev) : SRU.expr_begin(BB))) {
        Prev = &Inst;
        continue;
      }
      // TODO: Change push/pops to stack adjustment instruction
      if (BF.getBinaryContext().MIB->isPush(Inst))
        continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 202-215

```cpp
      ++NumRedundantStores;
      FreqRedundantStores += BB.getKnownExecutionCount();
      Changed = true;
      LLVM_DEBUG(dbgs() << "Unused store instruction: ");
      LLVM_DEBUG(Inst.dump());
      LLVM_DEBUG(dbgs() << "@BB: " << BB.getName() << "\n");
      LLVM_DEBUG(dbgs() << "FIE offset = " << FIEX->StackOffset
                        << " size = " << (int)FIEX->Size << "\n");
      // Delete it!
      ToErase.emplace_back(&BB, &Inst);
      Prev = &Inst;
    }
  }
```

- EN: Declares or implements routines including `LLVM_DEBUG`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `LLVM_DEBUG`。

### Lines 216-226

```cpp
  for (std::pair<BinaryBasicBlock *, MCInst *> I : ToErase)
    I.first->eraseInstruction(I.first->findInstruction(I.second));

  if (Changed)
    LLVM_DEBUG(dbgs() << "FOP modified \"" << BF.getPrintName() << "\"\n");
}

Error FrameOptimizerPass::runOnFunctions(BinaryContext &BC) {
  if (opts::FrameOptimization == FOP_NONE)
    return Error::success();
```

- EN: Declares or implements routines including `eraseInstruction`, `LLVM_DEBUG`, `runOnFunctions`. Notable symbols here include `eraseInstruction`, `LLVM_DEBUG`, `runOnFunctions`.
- CN: 这里声明或实现函数，例如 `eraseInstruction`, `LLVM_DEBUG`, `runOnFunctions`。这里较值得关注的符号包括 `eraseInstruction`, `LLVM_DEBUG`, `runOnFunctions`。

### Lines 227-235

```cpp
  if (!BC.isX86()) {
    BC.errs() << "BOLT-ERROR: " << getName() << " is supported only on X86\n";
    exit(1);
  }

  std::unique_ptr<BinaryFunctionCallGraph> CG;
  std::unique_ptr<FrameAnalysis> FA;
  std::unique_ptr<RegAnalysis> RA;
```

- EN: Declares or implements routines including `exit`. Notable symbols here include `exit`.
- CN: 这里声明或实现函数，例如 `exit`。这里较值得关注的符号包括 `exit`。

### Lines 236-247

```cpp
  {
    NamedRegionTimer T1("callgraph", "create call graph", "FOP",
                        "FOP breakdown", opts::TimeOpts);
    CG = std::make_unique<BinaryFunctionCallGraph>(buildCallGraph(BC));
  }

  {
    NamedRegionTimer T1("frameanalysis", "frame analysis", "FOP",
                        "FOP breakdown", opts::TimeOpts);
    FA = std::make_unique<FrameAnalysis>(BC, *CG);
  }
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 248-265

```cpp
  {
    NamedRegionTimer T1("reganalysis", "reg analysis", "FOP", "FOP breakdown",
                        opts::TimeOpts);
    RA = std::make_unique<RegAnalysis>(BC, &BC.getBinaryFunctions(), CG.get());
  }

  // Perform caller-saved register optimizations, then callee-saved register
  // optimizations (shrink wrapping)
  for (auto &I : BC.getBinaryFunctions()) {
    if (!FA->hasFrameInfo(I.second))
      continue;
    // Restrict pass execution if user asked to only run on hot functions
    if (opts::FrameOptimization == FOP_HOT) {
      if (I.second.getKnownExecutionCount() < BC.getHotThreshold())
        continue;
      LLVM_DEBUG(
          dbgs() << "Considering " << I.second.getPrintName()
                 << " for frame optimizations because its execution count ( "
```

- EN: Declares or implements routines including `dbgs`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `dbgs`.
- CN: 这里声明或实现函数，例如 `dbgs`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `dbgs`。

### Lines 266-277

```cpp
                 << I.second.getKnownExecutionCount()
                 << " ) exceeds our hotness threshold ( "
                 << BC.getHotThreshold() << " )\n");
    }

    {
      NamedRegionTimer T1("removeloads", "remove loads", "FOP", "FOP breakdown",
                          opts::TimeOpts);
      if (!FA->hasStackArithmetic(I.second))
        removeUnnecessaryLoads(*RA, *FA, I.second);
    }
```

- EN: Declares or implements routines including `threshold`, `removeUnnecessaryLoads`. Notable symbols here include `threshold`, `removeUnnecessaryLoads`.
- CN: 这里声明或实现函数，例如 `threshold`, `removeUnnecessaryLoads`。这里较值得关注的符号包括 `threshold`, `removeUnnecessaryLoads`。

### Lines 278-288

```cpp
    if (opts::RemoveStores) {
      NamedRegionTimer T1("removestores", "remove stores", "FOP",
                          "FOP breakdown", opts::TimeOpts);
      if (!FA->hasStackArithmetic(I.second))
        removeUnusedStores(*FA, I.second);
    }
    // Don't even start shrink wrapping if no profiling info is available
    if (I.second.getKnownExecutionCount() == 0)
      continue;
  }
```

- EN: Declares or implements routines including `removeUnusedStores`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `removeUnusedStores`.
- CN: 这里声明或实现函数，例如 `removeUnusedStores`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `removeUnusedStores`。

### Lines 289-306

```cpp
  {
    NamedRegionTimer T1("shrinkwrapping", "shrink wrapping", "FOP",
                        "FOP breakdown", opts::TimeOpts);
    if (Error E = performShrinkWrapping(*RA, *FA, BC))
      return Error(std::move(E));
  }

  BC.outs() << "BOLT-INFO: FOP optimized " << NumRedundantLoads
            << " redundant load(s) and " << NumRedundantStores
            << " unused store(s)\n";
  BC.outs() << "BOLT-INFO: Frequency of redundant loads is "
            << FreqRedundantLoads << " and frequency of unused stores is "
            << FreqRedundantStores << "\n";
  BC.outs() << "BOLT-INFO: Frequency of loads changed to use a register is "
            << FreqLoadsChangedToReg
            << " and frequency of loads changed to use an immediate is "
            << FreqLoadsChangedToImm << "\n";
  BC.outs() << "BOLT-INFO: FOP deleted " << NumLoadsDeleted
```

- EN: Declares or implements routines including `load`, `store`. Notable symbols here include `load`, `store`.
- CN: 这里声明或实现函数，例如 `load`, `store`。这里较值得关注的符号包括 `load`, `store`。

### Lines 307-324

```cpp
            << " load(s) (dyn count: " << FreqLoadsDeleted << ") and "
            << NumRedundantStores << " store(s)\n";
  FA->printStats();
  ShrinkWrapping::printStats(BC);
  return Error::success();
}

Error FrameOptimizerPass::performShrinkWrapping(const RegAnalysis &RA,
                                                const FrameAnalysis &FA,
                                                BinaryContext &BC) {
  // Initialize necessary annotations to allow safe parallel accesses to
  // annotation index in MIB
  BC.MIB->getOrCreateAnnotationIndex(CalleeSavedAnalysis::getSaveTagName());
  BC.MIB->getOrCreateAnnotationIndex(CalleeSavedAnalysis::getRestoreTagName());
  BC.MIB->getOrCreateAnnotationIndex(StackLayoutModifier::getTodoTagName());
  BC.MIB->getOrCreateAnnotationIndex(StackLayoutModifier::getSlotTagName());
  BC.MIB->getOrCreateAnnotationIndex(
      StackLayoutModifier::getOffsetCFIRegTagName());
```

- EN: Declares or implements routines including `load`, `store`, `printStats`, `getOrCreateAnnotationIndex`, `getOffsetCFIRegTagName`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `load`, `store`, `printStats`, `getOrCreateAnnotationIndex`, `getOffsetCFIRegTagName`.
- CN: 这里声明或实现函数，例如 `load`, `store`, `printStats`, `getOrCreateAnnotationIndex`, `getOffsetCFIRegTagName`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `load`, `store`, `printStats`, `getOrCreateAnnotationIndex`, `getOffsetCFIRegTagName`。

### Lines 325-341

```cpp
  BC.MIB->getOrCreateAnnotationIndex("ReachingDefs");
  BC.MIB->getOrCreateAnnotationIndex("ReachingUses");
  BC.MIB->getOrCreateAnnotationIndex("LivenessAnalysis");
  BC.MIB->getOrCreateAnnotationIndex("StackReachingUses");
  BC.MIB->getOrCreateAnnotationIndex("PostDominatorAnalysis");
  BC.MIB->getOrCreateAnnotationIndex("DominatorAnalysis");
  BC.MIB->getOrCreateAnnotationIndex("StackPointerTracking");
  BC.MIB->getOrCreateAnnotationIndex("StackPointerTrackingForInternalCalls");
  BC.MIB->getOrCreateAnnotationIndex("StackAvailableExpressions");
  BC.MIB->getOrCreateAnnotationIndex("StackAllocationAnalysis");
  BC.MIB->getOrCreateAnnotationIndex("ShrinkWrap-Todo");
  BC.MIB->getOrCreateAnnotationIndex("PredictiveStackPointerTracking");
  BC.MIB->getOrCreateAnnotationIndex("ReachingInsnsBackward");
  BC.MIB->getOrCreateAnnotationIndex("ReachingInsns");
  BC.MIB->getOrCreateAnnotationIndex("AccessesDeletedPos");
  BC.MIB->getOrCreateAnnotationIndex("DeleteMe");
```

- EN: Declares or implements routines including `getOrCreateAnnotationIndex`. Notable symbols here include `getOrCreateAnnotationIndex`.
- CN: 这里声明或实现函数，例如 `getOrCreateAnnotationIndex`。这里较值得关注的符号包括 `getOrCreateAnnotationIndex`。

### Lines 342-356

```cpp
  std::vector<std::pair<uint64_t, const BinaryFunction *>> Top10Funcs;
  auto LogFunc = [&](BinaryFunction &BF) {
    auto Lower = llvm::lower_bound(
        Top10Funcs, BF.getKnownExecutionCount(),
        [](const std::pair<uint64_t, const BinaryFunction *> &Elmt,
           uint64_t Value) { return Elmt.first > Value; });
    if (Lower == Top10Funcs.end() && Top10Funcs.size() >= 10)
      return;
    Top10Funcs.insert(Lower,
                      std::make_pair<>(BF.getKnownExecutionCount(), &BF));
    if (Top10Funcs.size() > 10)
      Top10Funcs.resize(10);
  };
  (void)LogFunc;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 357-365

```cpp
  ParallelUtilities::PredicateTy SkipPredicate = [&](const BinaryFunction &BF) {
    if (BF.getFunctionScore() == 0)
      return true;

    return false;
  };

  const bool HotOnly = opts::FrameOptimization == FOP_HOT;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 366-383

```cpp
  Error SWError = Error::success();

  ParallelUtilities::WorkFuncWithAllocTy WorkFunction =
      [&](BinaryFunction &BF, MCPlusBuilder::AllocatorIdTy AllocatorId) {
        DataflowInfoManager Info(BF, &RA, &FA, AllocatorId);
        ShrinkWrapping SW(FA, BF, Info, AllocatorId);

        auto ChangedOrErr = SW.perform(HotOnly);
        if (auto E = ChangedOrErr.takeError()) {
          std::lock_guard<std::mutex> Lock(FuncsChangedMutex);
          SWError = joinErrors(std::move(SWError), Error(std::move(E)));
          return;
        }
        const bool Changed = *ChangedOrErr;
        if (Changed) {
          std::lock_guard<std::mutex> Lock(FuncsChangedMutex);
          FuncsChanged.insert(&BF);
          LLVM_DEBUG(LogFunc(BF));
```

- EN: Declares or implements routines including `success`, `Info`, `SW`, `Lock`, `joinErrors`, and 1 more. Notable symbols here include `success`, `Info`, `SW`, `Lock`, `joinErrors`, `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `success`, `Info`, `SW`, `Lock`, `joinErrors`, and 1 more。这里较值得关注的符号包括 `success`, `Info`, `SW`, `Lock`, `joinErrors`, `LLVM_DEBUG`。

### Lines 384-398

```cpp
        }
      };

  ParallelUtilities::runOnEachFunctionWithUniqueAllocId(
      BC, ParallelUtilities::SchedulingPolicy::SP_INST_QUADRATIC, WorkFunction,
      SkipPredicate, "shrink-wrapping");

  if (!Top10Funcs.empty()) {
    BC.outs() << "BOLT-INFO: top 10 functions changed by shrink wrapping:\n";
    for (const auto &Elmt : Top10Funcs)
      BC.outs() << Elmt.first << " : " << Elmt.second->getPrintName() << "\n";
  }
  return SWError;
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 399-400

```cpp
} // namespace bolt
} // namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Notable symbols here include `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里较值得关注的符号包括 `bolt`, `llvm`。

## Key Concepts / 关键概念

- `init`: function or method entry point / 函数或方法入口
- `desc`: function or method entry point / 函数或方法入口
- `clEnumValN`: function or method entry point / 函数或方法入口
- `cat`: function or method entry point / 函数或方法入口
- `SAE`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关
- `llvm`: namespace scope / 命名空间作用域
- `opts`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/FrameOptimizer.h`, `bolt/Core/BinaryFunctionCallGraph.h`, `bolt/Core/ParallelUtilities.h`, `bolt/Passes/DataflowInfoManager.h`, `bolt/Passes/ShrinkWrapping.h`, `bolt/Passes/StackAvailableExpressions.h`, `bolt/Passes/StackReachingUses.h`, `bolt/Utils/CommandLineOpts.h`
- LLVM headers / LLVM 头文件: `llvm/Support/Timer.h`
- System headers / 系统头文件: `deque`
- Directory context / 目录上下文: `bolt/lib/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Passes` 下的相邻文件通常与本文件协作组成对应子系统
