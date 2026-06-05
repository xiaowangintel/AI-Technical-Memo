# LoopVersioningLICM.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Scalar/LoopVersioningLICM.cpp` | `llvm/lib/Transforms/Scalar/LoopVersioningLICM.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements lICM Loop Versioning within LLVM's scalar optimizations and loop/CFG transforms layer. | 该源文件实现了 LLVM 标量优化以及循环/CFG 变换层中的 LoopVersioningLICM 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-47

```cpp
//===- LoopVersioningLICM.cpp - LICM Loop Versioning ----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// When alias analysis is uncertain about the aliasing between any two accesses,
// it will return MayAlias. This uncertainty from alias analysis restricts LICM
// from proceeding further. In cases where alias analysis is uncertain we might
// use loop versioning as an alternative.
//
// Loop Versioning will create a version of the loop with aggressive aliasing
// assumptions in addition to the original with conservative (default) aliasing
// assumptions. The version of the loop making aggressive aliasing assumptions
// will have all the memory accesses marked as no-alias. These two versions of
// loop will be preceded by a memory runtime check. This runtime check consists
// of bound checks for all unique memory accessed in loop, and it ensures the
// lack of memory aliasing. The result of the runtime check determines which of
// the loop versions is executed: If the runtime check detects any memory
// aliasing, then the original loop is executed. Otherwise, the version with
// aggressive aliasing assumptions is used.
//
// Following are the top level steps:
//
// a) Perform LoopVersioningLICM's feasibility check.
// b) If loop is a candidate for versioning then create a memory bound check,
//    by considering all the memory accesses in loop body.
// c) Clone original loop and set all memory accesses as no-alias in new loop.
// d) Set original loop & versioned loop as a branch target of the runtime check
//    result.
//
// It transforms loop as shown below:
//
//                         +----------------+
//                         |Runtime Memcheck|
//                         +----------------+
//                                 |
//              +----------+----------------+----------+
//              |                                      |
//    +---------+----------+               +-----------+----------+
//    |Orig Loop Preheader |               |Cloned Loop Preheader |
//    +--------------------+               +----------------------+
//              |                                      |
//    +--------------------+               +----------------------+
//    |Orig Loop Body      |               |Cloned Loop Body      |
```
- EN: This commented region explains the pass goal, legality assumptions, or the high-level algorithm before readers dive into the executable code. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段注释先说明 pass 的目标、合法性假设或整体算法，再引导读者进入真正的实现代码。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 48-101

```cpp
//    +--------------------+               +----------------------+
//              |                                      |
//    +--------------------+               +----------------------+
//    |Orig Loop Exit Block|               |Cloned Loop Exit Block|
//    +--------------------+               +-----------+----------+
//              |                                      |
//              +----------+--------------+-----------+
//                                 |
//                           +-----+----+
//                           |Join Block|
//                           +----------+
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Scalar/LoopVersioningLICM.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/Analysis/AliasAnalysis.h"
#include "llvm/Analysis/AliasSetTracker.h"
#include "llvm/Analysis/GlobalsModRef.h"
#include "llvm/Analysis/LoopAccessAnalysis.h"
#include "llvm/Analysis/LoopInfo.h"
#include "llvm/Analysis/LoopPass.h"
#include "llvm/Analysis/OptimizationRemarkEmitter.h"
#include "llvm/Analysis/ScalarEvolution.h"
#include "llvm/IR/Dominators.h"
#include "llvm/IR/Instruction.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/LLVMContext.h"
#include "llvm/IR/MDBuilder.h"
#include "llvm/IR/Metadata.h"
#include "llvm/IR/Value.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/Transforms/Utils/LoopUtils.h"
#include "llvm/Transforms/Utils/LoopVersioning.h"
#include <cassert>

using namespace llvm;

#define DEBUG_TYPE "loop-versioning-licm"

static const char *LICMVersioningMetaData = "llvm.loop.licm_versioning.disable";

/// Threshold minimum allowed percentage for possible
/// invariant instructions in a loop.
static cl::opt<float>
    LVInvarThreshold("licm-versioning-invariant-threshold",
                     cl::desc("LoopVersioningLICM's minimum allowed percentage "
                              "of possible invariant instructions per loop"),
                     cl::init(25), cl::Hidden);

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 102-149

```cpp
/// Threshold for maximum allowed loop nest/depth
static cl::opt<unsigned> LVLoopDepthThreshold(
    "licm-versioning-max-depth-threshold",
    cl::desc(
        "LoopVersioningLICM's threshold for maximum allowed loop nest/depth"),
    cl::init(2), cl::Hidden);

namespace {

struct LoopVersioningLICM {
  // We don't explicitly pass in LoopAccessInfo to the constructor since the
  // loop versioning might return early due to instructions that are not safe
  // for versioning. By passing the proxy instead the construction of
  // LoopAccessInfo will take place only when it's necessary.
  LoopVersioningLICM(AliasAnalysis *AA, ScalarEvolution *SE,
                     OptimizationRemarkEmitter *ORE,
                     LoopAccessInfoManager &LAIs, LoopInfo &LI,
                     Loop *CurLoop)
      : AA(AA), SE(SE), LAIs(LAIs), LI(LI), CurLoop(CurLoop),
        LoopDepthThreshold(LVLoopDepthThreshold),
        InvariantThreshold(LVInvarThreshold), ORE(ORE) {}

  bool run(DominatorTree *DT);

private:
  // Current AliasAnalysis information
  AliasAnalysis *AA;

  // Current ScalarEvolution
  ScalarEvolution *SE;

  // Current Loop's LoopAccessInfo
  const LoopAccessInfo *LAI = nullptr;

  // Proxy for retrieving LoopAccessInfo.
  LoopAccessInfoManager &LAIs;

  LoopInfo &LI;

  // The current loop we are working on.
  Loop *CurLoop;

  // Maximum loop nest threshold
  unsigned LoopDepthThreshold;

  // Minimum invariant threshold
  float InvariantThreshold;

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics. Core entities appearing here include LoopVersioningLICM, which carry the local state, declarations, or transformation entry points for this region.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。 此处出现的核心实体包括 LoopVersioningLICM，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 150-196

```cpp
  // Counter to track num of load & store
  unsigned LoadAndStoreCounter = 0;

  // Counter to track num of invariant
  unsigned InvariantCounter = 0;

  // Read only loop marker.
  bool IsReadOnlyLoop = true;

  // OptimizationRemarkEmitter
  OptimizationRemarkEmitter *ORE;

  bool isLegalForVersioning();
  bool legalLoopStructure();
  bool legalLoopInstructions();
  bool legalLoopMemoryAccesses();
  bool isLoopAlreadyVisited();
  bool instructionSafeForVersioning(Instruction *I);
};

} // end anonymous namespace

/// Check loop structure and confirms it's good for LoopVersioningLICM.
bool LoopVersioningLICM::legalLoopStructure() {
  // Loop must be in loop simplify form.
  if (!CurLoop->isLoopSimplifyForm()) {
    LLVM_DEBUG(dbgs() << "    loop is not in loop-simplify form.\n");
    return false;
  }
  // Loop should be innermost loop, if not return false.
  if (!CurLoop->getSubLoops().empty()) {
    LLVM_DEBUG(dbgs() << "    loop is not innermost\n");
    return false;
  }
  // Loop should have a single backedge, if not return false.
  if (CurLoop->getNumBackEdges() != 1) {
    LLVM_DEBUG(dbgs() << "    loop has multiple backedges\n");
    return false;
  }
  // Loop must have a single exiting block, if not return false.
  if (!CurLoop->getExitingBlock()) {
    LLVM_DEBUG(dbgs() << "    loop has multiple exiting block\n");
    return false;
  }
  // We only handle bottom-tested loop, i.e. loop in which the condition is
  // checked at the end of each iteration. With that we can assume that all
  // instructions in the loop are executed the same number of times.
```
- EN: Core entities appearing here include legalLoopStructure, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as OptimizationRemarkEmitter. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 legalLoopStructure，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 OptimizationRemarkEmitter 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 197-243

```cpp
  if (CurLoop->getExitingBlock() != CurLoop->getLoopLatch()) {
    LLVM_DEBUG(dbgs() << "    loop is not bottom tested\n");
    return false;
  }
  // Parallel loops must not have aliasing loop-invariant memory accesses.
  // Hence we don't need to version anything in this case.
  if (CurLoop->isAnnotatedParallel()) {
    LLVM_DEBUG(dbgs() << "    Parallel loop is not worth versioning\n");
    return false;
  }
  // Loop depth more then LoopDepthThreshold are not allowed
  if (CurLoop->getLoopDepth() > LoopDepthThreshold) {
    LLVM_DEBUG(dbgs() << "    loop depth is more than threshold\n");
    return false;
  }
  // We need to be able to compute the loop trip count in order
  // to generate the bound checks.
  const SCEV *ExitCount = SE->getBackedgeTakenCount(CurLoop);
  if (isa<SCEVCouldNotCompute>(ExitCount)) {
    LLVM_DEBUG(dbgs() << "    loop does not have trip count\n");
    return false;
  }
  return true;
}

/// Check memory accesses in loop and confirms it's good for
/// LoopVersioningLICM.
bool LoopVersioningLICM::legalLoopMemoryAccesses() {
  // Loop over the body of this loop, construct AST.
  BatchAAResults BAA(*AA);
  AliasSetTracker AST(BAA);
  for (auto *Block : CurLoop->getBlocks()) {
    // Ignore blocks in subloops.
    if (LI.getLoopFor(Block) == CurLoop)
      AST.add(*Block);
  }

  // Memory check:
  // Transform phase will generate a versioned loop and also a runtime check to
  // ensure the pointers are independent and they don’t alias.
  // In version variant of loop, alias meta data asserts that all access are
  // mutually independent.
  //
  // Pointers aliasing in alias domain are avoided because with multiple
  // aliasing domains we may not be able to hoist potential loop invariant
  // access out of the loop.
  //
```
- EN: Core entities appearing here include legalLoopMemoryAccesses, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 legalLoopMemoryAccesses，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AAResults, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 244-294

```cpp
  // Iterate over alias tracker sets, and confirm AliasSets doesn't have any
  // must alias set.
  bool HasMayAlias = false;
  bool TypeSafety = false;
  bool HasMod = false;
  for (const auto &I : AST) {
    const AliasSet &AS = I;
    // Skip Forward Alias Sets, as this should be ignored as part of
    // the AliasSetTracker object.
    if (AS.isForwardingAliasSet())
      continue;
    // With MustAlias its not worth adding runtime bound check.
    if (AS.isMustAlias())
      return false;
    const Value *SomePtr = AS.begin()->Ptr;
    bool TypeCheck = true;
    // Check for Mod & MayAlias
    HasMayAlias |= AS.isMayAlias();
    HasMod |= AS.isMod();
    for (const auto &MemLoc : AS) {
      const Value *Ptr = MemLoc.Ptr;
      // Alias tracker should have pointers of same data type.
      //
      // FIXME: check no longer effective since opaque pointers?
      // If the intent is to check that the memory accesses use the
      // same data type (such that LICM can promote them), then we
      // can no longer see this from the pointer value types.
      TypeCheck = (TypeCheck && (SomePtr->getType() == Ptr->getType()));
    }
    // At least one alias tracker should have pointers of same data type.
    TypeSafety |= TypeCheck;
  }
  // Ensure types should be of same type.
  if (!TypeSafety) {
    LLVM_DEBUG(dbgs() << "    Alias tracker type safety failed!\n");
    return false;
  }
  // Ensure loop body shouldn't be read only.
  if (!HasMod) {
    LLVM_DEBUG(dbgs() << "    No memory modified in loop body\n");
    return false;
  }
  // Make sure alias set has may alias case.
  // If there no alias memory ambiguity, return false.
  if (!HasMayAlias) {
    LLVM_DEBUG(dbgs() << "    No ambiguity in memory access.\n");
    return false;
  }
  return true;
}

```
- EN: This region continues the LoopVersioningLICM implementation with local helper logic centered on Iterate, AliasSets, HasMayAlias, TypeSafety.
- CN: 这一段延续了 LoopVersioningLICM 的主体实现，围绕 Iterate, AliasSets, HasMayAlias, TypeSafety 等局部辅助逻辑展开。

### Lines 295-341

```cpp
/// Check loop instructions safe for Loop versioning.
/// It returns true if it's safe else returns false.
/// Consider following:
/// 1) Check all load store in loop body are non atomic & non volatile.
/// 2) Check function call safety, by ensuring its not accessing memory.
/// 3) Loop body shouldn't have any may throw instruction.
/// 4) Loop body shouldn't have any convergent or noduplicate instructions.
bool LoopVersioningLICM::instructionSafeForVersioning(Instruction *I) {
  assert(I != nullptr && "Null instruction found!");
  // Check function call safety
  if (auto *Call = dyn_cast<CallBase>(I)) {
    if (Call->isConvergent() || Call->cannotDuplicate()) {
      LLVM_DEBUG(dbgs() << "    Convergent call site found.\n");
      return false;
    }

    if (!AA->doesNotAccessMemory(Call)) {
      LLVM_DEBUG(dbgs() << "    Unsafe call site found.\n");
      return false;
    }
  }

  // Avoid loops with possiblity of throw
  if (I->mayThrow()) {
    LLVM_DEBUG(dbgs() << "    May throw instruction found in loop body\n");
    return false;
  }
  // If current instruction is load instructions
  // make sure it's a simple load (non atomic & non volatile)
  if (I->mayReadFromMemory()) {
    LoadInst *Ld = dyn_cast<LoadInst>(I);
    if (!Ld || !Ld->isSimple()) {
      LLVM_DEBUG(dbgs() << "    Found a non-simple load.\n");
      return false;
    }
    LoadAndStoreCounter++;
    Value *Ptr = Ld->getPointerOperand();
    // Check loop invariant.
    if (SE->isLoopInvariant(SE->getSCEV(Ptr), CurLoop))
      InvariantCounter++;
  }
  // If current instruction is store instruction
  // make sure it's a simple store (non atomic & non volatile)
  else if (I->mayWriteToMemory()) {
    StoreInst *St = dyn_cast<StoreInst>(I);
    if (!St || !St->isSimple()) {
      LLVM_DEBUG(dbgs() << "    Found a non-simple store.\n");
```
- EN: Core entities appearing here include instructionSafeForVersioning, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 instructionSafeForVersioning，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 342-388

```cpp
      return false;
    }
    LoadAndStoreCounter++;
    Value *Ptr = St->getPointerOperand();
    // Don't allow stores that we don't have runtime checks for, as we won't be
    // able to mark them noalias meaning they would prevent any code motion.
    auto &Pointers = LAI->getRuntimePointerChecking()->Pointers;
    if (!any_of(Pointers, [&](auto &P) { return P.PointerValue == Ptr; })) {
      LLVM_DEBUG(dbgs() << "    Found a store without a runtime check.\n");
      return false;
    }
    // Check loop invariant.
    if (SE->isLoopInvariant(SE->getSCEV(Ptr), CurLoop))
      InvariantCounter++;

    IsReadOnlyLoop = false;
  }
  return true;
}

/// Check loop instructions and confirms it's good for
/// LoopVersioningLICM.
bool LoopVersioningLICM::legalLoopInstructions() {
  // Resetting counters.
  LoadAndStoreCounter = 0;
  InvariantCounter = 0;
  IsReadOnlyLoop = true;
  using namespace ore;
  // Get LoopAccessInfo from current loop via the proxy.
  LAI = &LAIs.getInfo(*CurLoop, /*AllowPartial=*/true);
  // Check LoopAccessInfo for need of runtime check.
  if (LAI->getRuntimePointerChecking()->getChecks().empty()) {
    LLVM_DEBUG(dbgs() << "    LAA: Runtime check not found !!\n");
    return false;
  }
  // Iterate over loop blocks and instructions of each block and check
  // instruction safety.
  for (auto *Block : CurLoop->getBlocks())
    for (auto &Inst : *Block) {
      // If instruction is unsafe just return false.
      if (!instructionSafeForVersioning(&Inst)) {
        ORE->emit([&]() {
          return OptimizationRemarkMissed(DEBUG_TYPE, "IllegalLoopInst", &Inst)
                 << " Unsafe Loop Instruction";
        });
        return false;
      }
```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include legalLoopInstructions, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as LoopAccessInfo, SCEV.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 legalLoopInstructions，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 LoopAccessInfo, SCEV 等分析结果。

### Lines 389-441

```cpp
    }
  // Number of runtime-checks should be less then RuntimeMemoryCheckThreshold
  if (LAI->getNumRuntimePointerChecks() >
      VectorizerParams::RuntimeMemoryCheckThreshold) {
    LLVM_DEBUG(
        dbgs() << "    LAA: Runtime checks are more than threshold !!\n");
    ORE->emit([&]() {
      return OptimizationRemarkMissed(DEBUG_TYPE, "RuntimeCheck",
                                      CurLoop->getStartLoc(),
                                      CurLoop->getHeader())
             << "Number of runtime checks "
             << NV("RuntimeChecks", LAI->getNumRuntimePointerChecks())
             << " exceeds threshold "
             << NV("Threshold", VectorizerParams::RuntimeMemoryCheckThreshold);
    });
    return false;
  }
  // Loop should have at least one invariant load or store instruction.
  if (!InvariantCounter) {
    LLVM_DEBUG(dbgs() << "    Invariant not found !!\n");
    return false;
  }
  // Read only loop not allowed.
  if (IsReadOnlyLoop) {
    LLVM_DEBUG(dbgs() << "    Found a read-only loop!\n");
    return false;
  }
  // Profitability check:
  // Check invariant threshold, should be in limit.
  if (InvariantCounter * 100 < InvariantThreshold * LoadAndStoreCounter) {
    LLVM_DEBUG(
        dbgs()
        << "    Invariant load & store are less then defined threshold\n");
    LLVM_DEBUG(dbgs() << "    Invariant loads & stores: "
                      << ((InvariantCounter * 100) / LoadAndStoreCounter)
                      << "%\n");
    LLVM_DEBUG(dbgs() << "    Invariant loads & store threshold: "
                      << InvariantThreshold << "%\n");
    ORE->emit([&]() {
      return OptimizationRemarkMissed(DEBUG_TYPE, "InvariantThreshold",
                                      CurLoop->getStartLoc(),
                                      CurLoop->getHeader())
             << "Invariant load & store "
             << NV("LoadAndStoreCounter",
                   ((InvariantCounter * 100) / LoadAndStoreCounter))
             << " are less then defined threshold "
             << NV("Threshold", InvariantThreshold);
    });
    return false;
  }
  return true;
}

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 442-488

```cpp
/// It checks loop is already visited or not.
/// check loop meta data, if loop revisited return true
/// else false.
bool LoopVersioningLICM::isLoopAlreadyVisited() {
  // Check LoopVersioningLICM metadata into loop
  if (findStringMetadataForLoop(CurLoop, LICMVersioningMetaData)) {
    return true;
  }
  return false;
}

/// Checks legality for LoopVersioningLICM by considering following:
/// a) loop structure legality   b) loop instruction legality
/// c) loop memory access legality.
/// Return true if legal else returns false.
bool LoopVersioningLICM::isLegalForVersioning() {
  using namespace ore;
  LLVM_DEBUG(dbgs() << "Loop: " << *CurLoop);
  // Make sure not re-visiting same loop again.
  if (isLoopAlreadyVisited()) {
    LLVM_DEBUG(
        dbgs() << "    Revisiting loop in LoopVersioningLICM not allowed.\n\n");
    return false;
  }
  // Check loop structure leagality.
  if (!legalLoopStructure()) {
    LLVM_DEBUG(
        dbgs() << "    Loop structure not suitable for LoopVersioningLICM\n\n");
    ORE->emit([&]() {
      return OptimizationRemarkMissed(DEBUG_TYPE, "IllegalLoopStruct",
                                      CurLoop->getStartLoc(),
                                      CurLoop->getHeader())
             << " Unsafe Loop structure";
    });
    return false;
  }
  // Check loop instruction leagality.
  if (!legalLoopInstructions()) {
    LLVM_DEBUG(
        dbgs()
        << "    Loop instructions not suitable for LoopVersioningLICM\n\n");
    return false;
  }
  // Check loop memory access leagality.
  if (!legalLoopMemoryAccesses()) {
    LLVM_DEBUG(
        dbgs()
```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include isLoopAlreadyVisited, isLegalForVersioning, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 isLoopAlreadyVisited, isLegalForVersioning，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 489-542

```cpp
        << "    Loop memory access not suitable for LoopVersioningLICM\n\n");
    ORE->emit([&]() {
      return OptimizationRemarkMissed(DEBUG_TYPE, "IllegalLoopMemoryAccess",
                                      CurLoop->getStartLoc(),
                                      CurLoop->getHeader())
             << " Unsafe Loop memory access";
    });
    return false;
  }
  // Loop versioning is feasible, return true.
  LLVM_DEBUG(dbgs() << "    Loop Versioning found to be beneficial\n\n");
  ORE->emit([&]() {
    return OptimizationRemark(DEBUG_TYPE, "IsLegalForVersioning",
                              CurLoop->getStartLoc(), CurLoop->getHeader())
           << " Versioned loop for LICM."
           << " Number of runtime checks we had to insert "
           << NV("RuntimeChecks", LAI->getNumRuntimePointerChecks());
  });
  return true;
}

bool LoopVersioningLICM::run(DominatorTree *DT) {
  // Do not do the transformation if disabled by metadata.
  if (hasLICMVersioningTransformation(CurLoop) & TM_Disable)
    return false;

  bool Changed = false;

  // Check feasiblity of LoopVersioningLICM.
  // If versioning found to be feasible and beneficial then proceed
  // else simply return, by cleaning up memory.
  if (isLegalForVersioning()) {
    // Do loop versioning.
    // Create memcheck for memory accessed inside loop.
    // Clone original loop, and set blocks properly.
    LoopVersioning LVer(*LAI, LAI->getRuntimePointerChecking()->getChecks(),
                        CurLoop, &LI, DT, SE);
    LVer.versionLoop();
    // Set Loop Versioning metaData for original loop.
    addStringMetadataToLoop(LVer.getNonVersionedLoop(), LICMVersioningMetaData);
    // Set Loop Versioning metaData for version loop.
    addStringMetadataToLoop(LVer.getVersionedLoop(), LICMVersioningMetaData);
    // Set "llvm.mem.parallel_loop_access" metaData to versioned loop.
    // FIXME: "llvm.mem.parallel_loop_access" annotates memory access
    // instructions, not loops.
    addStringMetadataToLoop(LVer.getVersionedLoop(),
                            "llvm.mem.parallel_loop_access");
    // Update version loop with aggressive aliasing assumption.
    LVer.annotateLoopWithNoAlias();
    Changed = true;
  }
  return Changed;
}

```
- EN: Core entities appearing here include run, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 run，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 543-556

```cpp
PreservedAnalyses LoopVersioningLICMPass::run(Loop &L, LoopAnalysisManager &AM,
                                              LoopStandardAnalysisResults &LAR,
                                              LPMUpdater &U) {
  AliasAnalysis *AA = &LAR.AA;
  ScalarEvolution *SE = &LAR.SE;
  DominatorTree *DT = &LAR.DT;
  const Function *F = L.getHeader()->getParent();
  OptimizationRemarkEmitter ORE(F);

  LoopAccessInfoManager LAIs(*SE, *AA, *DT, LAR.LI, nullptr, nullptr, &LAR.AC);
  if (!LoopVersioningLICM(AA, SE, &ORE, LAIs, LAR.LI, &L).run(DT))
    return PreservedAnalyses::all();
  return getLoopPassPreservedAnalyses();
}
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AliasAnalysis, DominatorTree, LoopAccessInfo, LoopAccessInfoManager.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AliasAnalysis, DominatorTree, LoopAccessInfo, LoopAccessInfoManager 等分析结果。

## Key Concepts / 关键概念

- EN: Domain: `Scalar` focuses on scalar optimizations and loop/CFG transforms.
  - CN: 领域：`Scalar` 主要处理 标量优化以及循环/CFG 变换。
- EN: Primary entities: `LoopVersioningLICM, legalLoopStructure, legalLoopMemoryAccesses, instructionSafeForVersioning, legalLoopInstructions, isLoopAlreadyVisited, isLegalForVersioning, run` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`LoopVersioningLICM, legalLoopStructure, legalLoopMemoryAccesses, instructionSafeForVersioning, legalLoopInstructions, isLoopAlreadyVisited, isLegalForVersioning, run` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AAResults, AliasAnalysis, DominatorTree, LoopAccessInfo, LoopAccessInfoManager, LoopInfo, OptimizationRemarkEmitter, ScalarEvolution` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AAResults, AliasAnalysis, DominatorTree, LoopAccessInfo, LoopAccessInfoManager, LoopInfo, OptimizationRemarkEmitter, ScalarEvolution` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `LVInvarThreshold, LVLoopDepthThreshold` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `LVInvarThreshold, LVLoopDepthThreshold` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/AliasAnalysis.h`, `llvm/Analysis/AliasSetTracker.h`, `llvm/Analysis/GlobalsModRef.h`, `llvm/Analysis/LoopAccessAnalysis.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/LoopPass.h`, `llvm/Analysis/OptimizationRemarkEmitter.h`, `llvm/Analysis/ScalarEvolution.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/AliasAnalysis.h`, `llvm/Analysis/AliasSetTracker.h`, `llvm/Analysis/GlobalsModRef.h`, `llvm/Analysis/LoopAccessAnalysis.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/LoopPass.h`, `llvm/Analysis/OptimizationRemarkEmitter.h`, `llvm/Analysis/ScalarEvolution.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Dominators.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/LLVMContext.h`, `llvm/IR/MDBuilder.h`, `llvm/IR/Metadata.h`, `llvm/IR/Value.h`, `llvm/Transforms/Scalar/LoopVersioningLICM.h`, `llvm/Transforms/Utils/LoopUtils.h`, `llvm/Transforms/Utils/LoopVersioning.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Dominators.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/LLVMContext.h`, `llvm/IR/MDBuilder.h`, `llvm/IR/Metadata.h`, `llvm/IR/Value.h`, `llvm/Transforms/Scalar/LoopVersioningLICM.h`, `llvm/Transforms/Utils/LoopUtils.h`, `llvm/Transforms/Utils/LoopVersioning.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringRef.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringRef.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `cassert` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`cassert` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `AAResults`, `AliasAnalysis`, `DominatorTree`, `LoopAccessInfo`, `LoopAccessInfoManager`, `LoopInfo`, `OptimizationRemarkEmitter`, `ScalarEvolution` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AAResults`, `AliasAnalysis`, `DominatorTree`, `LoopAccessInfo`, `LoopAccessInfoManager`, `LoopInfo`, `OptimizationRemarkEmitter`, `ScalarEvolution` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
