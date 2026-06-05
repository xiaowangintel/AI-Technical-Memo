# MergedLoadStoreMotion.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Scalar/MergedLoadStoreMotion.cpp` | `llvm/lib/Transforms/Scalar/MergedLoadStoreMotion.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements merge and hoist/sink load/stores within LLVM's scalar optimizations and loop/CFG transforms layer. | 该源文件实现了 LLVM 标量优化以及循环/CFG 变换层中的 MergedLoadStoreMotion 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-34

```cpp
//===- MergedLoadStoreMotion.cpp - merge and hoist/sink load/stores -------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
//! \file
//! This pass performs merges of loads and stores on both sides of a
//  diamond (hammock). It hoists the loads and sinks the stores.
//
// The algorithm iteratively hoists two loads to the same address out of a
// diamond (hammock) and merges them into a single load in the header. Similar
// it sinks and merges two stores to the tail block (footer). The algorithm
// iterates over the instructions of one side of the diamond and attempts to
// find a matching load/store on the other side. New tail/footer block may be
// insterted if the tail/footer block has more predecessors (not only the two
// predecessors that are forming the diamond). It hoists / sinks when it thinks
// it safe to do so.  This optimization helps with eg. hiding load latencies,
// triggering if-conversion, and reducing static code size.
//
// NOTE: This code no longer performs load hoisting, it is subsumed by GVNHoist.
//
//===----------------------------------------------------------------------===//
//
//
// Example:
// Diamond shaped code before merge:
//
//            header:
//                     br %cond, label %if.then, label %if.else
//                        +                    +
//                       +                      +
```
- EN: This commented region explains the pass goal, legality assumptions, or the high-level algorithm before readers dive into the executable code. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段注释先说明 pass 的目标、合法性假设或整体算法，再引导读者进入真正的实现代码。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 35-68

```cpp
//                      +                        +
//            if.then:                         if.else:
//               %lt = load %addr_l               %le = load %addr_l
//               <use %lt>                        <use %le>
//               <...>                            <...>
//               store %st, %addr_s               store %se, %addr_s
//               br label %if.end                 br label %if.end
//                     +                         +
//                      +                       +
//                       +                     +
//            if.end ("footer"):
//                     <...>
//
// Diamond shaped code after merge:
//
//            header:
//                     %l = load %addr_l
//                     br %cond, label %if.then, label %if.else
//                        +                    +
//                       +                      +
//                      +                        +
//            if.then:                         if.else:
//               <use %l>                         <use %l>
//               <...>                            <...>
//               br label %if.end                 br label %if.end
//                      +                        +
//                       +                      +
//                        +                    +
//            if.end ("footer"):
//                     %s.sink = phi [%st, if.then], [%se, if.else]
//                     <...>
//                     store %s.sink, %addr_s
//                     <...>
//
```
- EN: This commented region explains the pass goal, legality assumptions, or the high-level algorithm before readers dive into the executable code. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段注释先说明 pass 的目标、合法性假设或整体算法，再引导读者进入真正的实现代码。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 69-105

```cpp
//
//===----------------------- TODO -----------------------------------------===//
//
// 1) Generalize to regions other than diamonds
// 2) Be more aggressive merging memory operations
// Note that both changes require register pressure control
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Scalar/MergedLoadStoreMotion.h"
#include "llvm/Analysis/AliasAnalysis.h"
#include "llvm/Analysis/GlobalsModRef.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/Instructions.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/Transforms/Scalar.h"
#include "llvm/Transforms/Utils/BasicBlockUtils.h"
#include "llvm/Transforms/Utils/Local.h"

using namespace llvm;

#define DEBUG_TYPE "mldst-motion"

namespace {
//===----------------------------------------------------------------------===//
//                         MergedLoadStoreMotion Pass
//===----------------------------------------------------------------------===//
class MergedLoadStoreMotion {
  AliasAnalysis *AA = nullptr;

  // The mergeLoad/Store algorithms could have Size0 * Size1 complexity,
  // where Size0 and Size1 are the #instructions on the two sides of
  // the diamond. The constant chosen here is arbitrary. Compiler Time
  // Control is enforced by the check Size0 * Size1 < MagicCompileTimeControl.
  const unsigned MagicCompileTimeControl = 250;

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 106-143

```cpp
  const bool SplitFooterBB;
public:
  MergedLoadStoreMotion(bool SplitFooterBB) : SplitFooterBB(SplitFooterBB) {}
  bool run(Function &F, AliasAnalysis &AA);

private:
  BasicBlock *getDiamondTail(BasicBlock *BB);
  bool isDiamondHead(BasicBlock *BB);
  // Routines for sinking stores
  StoreInst *canSinkFromBlock(BasicBlock *BB, StoreInst *SI);
  PHINode *getPHIOperand(BasicBlock *BB, StoreInst *S0, StoreInst *S1);
  bool isStoreSinkBarrierInRange(const Instruction &Start,
                                 const Instruction &End, MemoryLocation Loc);
  bool canSinkStoresAndGEPs(StoreInst *S0, StoreInst *S1) const;
  void sinkStoresAndGEPs(BasicBlock *BB, StoreInst *SinkCand,
                         StoreInst *ElseInst);
  bool mergeStores(BasicBlock *BB);
};
} // end anonymous namespace

///
/// Return tail block of a diamond.
///
BasicBlock *MergedLoadStoreMotion::getDiamondTail(BasicBlock *BB) {
  assert(isDiamondHead(BB) && "Basic block is not head of a diamond");
  return BB->getTerminator()->getSuccessor(0)->getSingleSuccessor();
}

///
/// True when BB is the head of a diamond (hammock)
///
bool MergedLoadStoreMotion::isDiamondHead(BasicBlock *BB) {
  if (!BB)
    return false;
  auto *BI = dyn_cast<CondBrInst>(BB->getTerminator());
  if (!BI)
    return false;

```
- EN: Core entities appearing here include isDiamondHead, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AliasAnalysis. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 isDiamondHead，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AliasAnalysis 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 144-178

```cpp
  BasicBlock *Succ0 = BI->getSuccessor(0);
  BasicBlock *Succ1 = BI->getSuccessor(1);

  if (!Succ0->getSinglePredecessor())
    return false;
  if (!Succ1->getSinglePredecessor())
    return false;

  BasicBlock *Succ0Succ = Succ0->getSingleSuccessor();
  BasicBlock *Succ1Succ = Succ1->getSingleSuccessor();
  // Ignore triangles.
  if (!Succ0Succ || !Succ1Succ || Succ0Succ != Succ1Succ)
    return false;
  return true;
}


///
/// True when instruction is a sink barrier for a store
/// located in Loc
///
/// Whenever an instruction could possibly read or modify the
/// value being stored or protect against the store from
/// happening it is considered a sink barrier.
///
bool MergedLoadStoreMotion::isStoreSinkBarrierInRange(const Instruction &Start,
                                                      const Instruction &End,
                                                      MemoryLocation Loc) {
  for (const Instruction &Inst :
       make_range(Start.getIterator(), End.getIterator()))
    if (Inst.mayThrow())
      return true;
  return AA->canInstructionRangeModRef(Start, End, Loc, ModRefInfo::ModRef);
}

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 179-212

```cpp
///
/// Check if \p BB contains a store to the same address as \p SI
///
/// \return The store in \p  when it is safe to sink. Otherwise return Null.
///
StoreInst *MergedLoadStoreMotion::canSinkFromBlock(BasicBlock *BB1,
                                                   StoreInst *Store0) {
  LLVM_DEBUG(dbgs() << "can Sink? : "; Store0->dump(); dbgs() << "\n");
  BasicBlock *BB0 = Store0->getParent();
  for (Instruction &Inst : reverse(*BB1)) {
    auto *Store1 = dyn_cast<StoreInst>(&Inst);
    if (!Store1)
      continue;

    MemoryLocation Loc0 = MemoryLocation::get(Store0);
    MemoryLocation Loc1 = MemoryLocation::get(Store1);

    if (AA->isMustAlias(Loc0, Loc1) &&
        !isStoreSinkBarrierInRange(*Store1->getNextNode(), BB1->back(), Loc1) &&
        !isStoreSinkBarrierInRange(*Store0->getNextNode(), BB0->back(), Loc0) &&
        Store0->hasSameSpecialState(Store1) &&
        CastInst::isBitOrNoopPointerCastable(
            Store0->getValueOperand()->getType(),
            Store1->getValueOperand()->getType(),
            Store0->getDataLayout()))
      return Store1;
  }
  return nullptr;
}

///
/// Create a PHI node in BB for the operands of S0 and S1
///
PHINode *MergedLoadStoreMotion::getPHIOperand(BasicBlock *BB, StoreInst *S0,
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 213-246

```cpp
                                              StoreInst *S1) {
  // Create a phi if the values mismatch.
  Value *Opd1 = S0->getValueOperand();
  Value *Opd2 = S1->getValueOperand();
  if (Opd1 == Opd2)
    return nullptr;

  auto *NewPN = PHINode::Create(Opd1->getType(), 2, Opd2->getName() + ".sink");
  NewPN->insertBefore(BB->begin());
  NewPN->applyMergedLocation(S0->getDebugLoc(), S1->getDebugLoc());
  NewPN->addIncoming(Opd1, S0->getParent());
  NewPN->addIncoming(Opd2, S1->getParent());
  return NewPN;
}

///
/// Check if 2 stores can be sunk, optionally together with corresponding GEPs.
///
bool MergedLoadStoreMotion::canSinkStoresAndGEPs(StoreInst *S0,
                                                 StoreInst *S1) const {
  if (S0->getPointerOperand() == S1->getPointerOperand())
    return true;
  auto *GEP0 = dyn_cast<GetElementPtrInst>(S0->getPointerOperand());
  auto *GEP1 = dyn_cast<GetElementPtrInst>(S1->getPointerOperand());
  return GEP0 && GEP1 && GEP0->isIdenticalTo(GEP1) && GEP0->hasOneUse() &&
         (GEP0->getParent() == S0->getParent()) && GEP1->hasOneUse() &&
         (GEP1->getParent() == S1->getParent());
}

///
/// Merge two stores to same address and sink into \p BB
///
/// Optionally also sinks GEP instruction computing the store address
///
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 247-279

```cpp
void MergedLoadStoreMotion::sinkStoresAndGEPs(BasicBlock *BB, StoreInst *S0,
                                              StoreInst *S1) {
  Value *Ptr0 = S0->getPointerOperand();
  Value *Ptr1 = S1->getPointerOperand();
  // Only one definition?
  LLVM_DEBUG(dbgs() << "Sink Instruction into BB \n"; BB->dump();
             dbgs() << "Instruction Left\n"; S0->dump(); dbgs() << "\n";
             dbgs() << "Instruction Right\n"; S1->dump(); dbgs() << "\n");
  // Hoist the instruction.
  BasicBlock::iterator InsertPt = BB->getFirstInsertionPt();
  // Intersect optional metadata.
  S0->andIRFlags(S1);

  combineMetadataForCSE(S0, S1, true);
  S0->applyMergedLocation(S0->getDebugLoc(), S1->getDebugLoc());
  S0->mergeDIAssignID(S1);

  // Insert bitcast for conflicting typed stores (or just use original value if
  // same type).
  IRBuilder<> Builder(S0);
  auto Cast = Builder.CreateBitOrPointerCast(S0->getValueOperand(),
                                             S1->getValueOperand()->getType());
  S0->setOperand(0, Cast);

  // Create the new store to be inserted at the join point.
  StoreInst *SNew = cast<StoreInst>(S0->clone());
  SNew->insertBefore(InsertPt);
  // New PHI operand? Use it.
  if (PHINode *NewPN = getPHIOperand(BB, S0, S1))
    SNew->setOperand(0, NewPN);
  S0->eraseFromParent();
  S1->eraseFromParent();

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 280-313

```cpp
  if (Ptr0 != Ptr1) {
    auto *GEP0 = cast<GetElementPtrInst>(Ptr0);
    auto *GEP1 = cast<GetElementPtrInst>(Ptr1);
    Instruction *GEPNew = GEP0->clone();
    GEPNew->insertBefore(SNew->getIterator());
    GEPNew->applyMergedLocation(GEP0->getDebugLoc(), GEP1->getDebugLoc());
    SNew->setOperand(1, GEPNew);
    GEP0->replaceAllUsesWith(GEPNew);
    GEP0->eraseFromParent();
    GEP1->replaceAllUsesWith(GEPNew);
    GEP1->eraseFromParent();
  }
}

///
/// True when two stores are equivalent and can sink into the footer
///
/// Starting from a diamond head block, iterate over the instructions in one
/// successor block and try to match a store in the second successor.
///
bool MergedLoadStoreMotion::mergeStores(BasicBlock *HeadBB) {

  bool MergedStores = false;
  BasicBlock *TailBB = getDiamondTail(HeadBB);
  BasicBlock *SinkBB = TailBB;
  assert(SinkBB && "Footer of a diamond cannot be empty");

  succ_iterator SI = succ_begin(HeadBB);
  assert(SI != succ_end(HeadBB) && "Diamond head cannot have zero successors");
  BasicBlock *Pred0 = *SI;
  ++SI;
  assert(SI != succ_end(HeadBB) && "Diamond head cannot have single successor");
  BasicBlock *Pred1 = *SI;
  // tail block  of a diamond/hammock?
```
- EN: Core entities appearing here include mergeStores, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 mergeStores，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 314-352

```cpp
  if (Pred0 == Pred1)
    return false; // No.
  // bail out early if we can not merge into the footer BB
  if (!SplitFooterBB && TailBB->hasNPredecessorsOrMore(3))
    return false;
  // #Instructions in Pred1 for Compile Time Control
  unsigned Size1 = Pred1->size();
  unsigned NStores = 0;

  for (BasicBlock::reverse_iterator RBI = Pred0->rbegin(), RBE = Pred0->rend();
       RBI != RBE;) {

    Instruction *I = &*RBI;
    ++RBI;

    // Don't sink non-simple (atomic, volatile) stores.
    auto *S0 = dyn_cast<StoreInst>(I);
    if (!S0 || !S0->isSimple())
      continue;

    ++NStores;
    if (NStores * Size1 >= MagicCompileTimeControl)
      break;
    if (StoreInst *S1 = canSinkFromBlock(Pred1, S0)) {
      if (!canSinkStoresAndGEPs(S0, S1))
        // Don't attempt to sink below stores that had to stick around
        // But after removal of a store and some of its feeding
        // instruction search again from the beginning since the iterator
        // is likely stale at this point.
        break;

      if (SinkBB == TailBB && TailBB->hasNPredecessorsOrMore(3)) {
        // We have more than 2 predecessors. Insert a new block
        // postdominating 2 predecessors we're going to sink from.
        SinkBB = SplitBlockPredecessors(TailBB, {Pred0, Pred1}, ".sink.split");
        if (!SinkBB)
          break;
      }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 353-387

```cpp
      MergedStores = true;
      sinkStoresAndGEPs(SinkBB, S0, S1);
      RBI = Pred0->rbegin();
      RBE = Pred0->rend();
      LLVM_DEBUG(dbgs() << "Search again\n"; Instruction *I = &*RBI; I->dump());
    }
  }
  return MergedStores;
}

bool MergedLoadStoreMotion::run(Function &F, AliasAnalysis &AA) {
  this->AA = &AA;

  bool Changed = false;
  LLVM_DEBUG(dbgs() << "Instruction Merger\n");

  // Merge unconditional branches, allowing PRE to catch more
  // optimization opportunities.
  // This loop doesn't care about newly inserted/split blocks 
  // since they never will be diamond heads.
  for (BasicBlock &BB : make_early_inc_range(F))
    // Hoist equivalent loads and sink stores
    // outside diamonds when possible
    if (isDiamondHead(&BB))
      Changed |= mergeStores(&BB);
  return Changed;
}

PreservedAnalyses
MergedLoadStoreMotionPass::run(Function &F, FunctionAnalysisManager &AM) {
  MergedLoadStoreMotion Impl(Options.SplitFooterBB);
  auto &AA = AM.getResult<AAManager>(F);
  if (!Impl.run(F, AA))
    return PreservedAnalyses::all();

```
- EN: Core entities appearing here include run, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AAManager, AliasAnalysis. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 run，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AAManager, AliasAnalysis 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 388-401

```cpp
  PreservedAnalyses PA;
  if (!Options.SplitFooterBB)
    PA.preserveSet<CFGAnalyses>();
  return PA;
}

void MergedLoadStoreMotionPass::printPipeline(
    raw_ostream &OS, function_ref<StringRef(StringRef)> MapClassName2PassName) {
  static_cast<PassInfoMixin<MergedLoadStoreMotionPass> *>(this)->printPipeline(
      OS, MapClassName2PassName);
  OS << '<';
  OS << (Options.SplitFooterBB ? "" : "no-") << "split-footer-bb";
  OS << '>';
}
```
- EN: This region continues the MergedLoadStoreMotion implementation with local helper logic centered on PreservedAnalyses, Options, SplitFooterBB, CFGAnalyses.
- CN: 这一段延续了 MergedLoadStoreMotion 的主体实现，围绕 PreservedAnalyses, Options, SplitFooterBB, CFGAnalyses 等局部辅助逻辑展开。

## Key Concepts / 关键概念

- EN: Domain: `Scalar` focuses on scalar optimizations and loop/CFG transforms.
  - CN: 领域：`Scalar` 主要处理 标量优化以及循环/CFG 变换。
- EN: Primary entities: `MergedLoadStoreMotion, isDiamondHead, mergeStores, run` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`MergedLoadStoreMotion, isDiamondHead, mergeStores, run` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AAManager, AliasAnalysis, DataLayout` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AAManager, AliasAnalysis, DataLayout` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/AliasAnalysis.h`, `llvm/Analysis/GlobalsModRef.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/AliasAnalysis.h`, `llvm/Analysis/GlobalsModRef.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/IRBuilder.h`, `llvm/IR/Instructions.h`, `llvm/Transforms/Scalar/MergedLoadStoreMotion.h`, `llvm/Transforms/Scalar.h`, `llvm/Transforms/Utils/BasicBlockUtils.h`, `llvm/Transforms/Utils/Local.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/IRBuilder.h`, `llvm/IR/Instructions.h`, `llvm/Transforms/Scalar/MergedLoadStoreMotion.h`, `llvm/Transforms/Scalar.h`, `llvm/Transforms/Utils/BasicBlockUtils.h`, `llvm/Transforms/Utils/Local.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Runtime analysis dependencies: `AAManager`, `AliasAnalysis`, `DataLayout` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AAManager`, `AliasAnalysis`, `DataLayout` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
