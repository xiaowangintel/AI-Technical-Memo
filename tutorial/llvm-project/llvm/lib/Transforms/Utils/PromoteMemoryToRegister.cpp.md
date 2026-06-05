# PromoteMemoryToRegister.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/PromoteMemoryToRegister.cpp` | `llvm/lib/Transforms/Utils/PromoteMemoryToRegister.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements convert allocas to registers within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 PromoteMemoryToRegister 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-78

```cpp
//===- PromoteMemoryToRegister.cpp - Convert allocas to registers ---------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file promotes memory references to be register references.  It promotes
// alloca instructions which only have loads and stores as uses.  An alloca is
// transformed by using iterated dominator frontiers to place PHI nodes, then
// traversing the function in depth-first order to rewrite loads and stores as
// appropriate.
//
//===----------------------------------------------------------------------===//

#include "llvm/ADT/ArrayRef.h"
#include "llvm/ADT/BitVector.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/ADT/STLExtras.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

bool llvm::isAllocaPromotable(const AllocaInst *AI) {
  // Only allow direct and non-volatile loads and stores...
  // All loads and stores must use the same type (determined by the first one
  // seen). We don't require the type to match the alloca's declared type.
  Type *ExpectedType = nullptr;
  for (const User *U : AI->users()) {
    if (const LoadInst *LI = dyn_cast<LoadInst>(U)) {
      // Note that atomic loads can be transformed; atomic semantics do
      // not have any meaning for a local alloca.
      if (LI->isVolatile())
        return false;
      if (!ExpectedType)
        ExpectedType = LI->getType();
```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 79-155

```cpp
      else if (LI->getType() != ExpectedType)
        return false;
    } else if (const StoreInst *SI = dyn_cast<StoreInst>(U)) {
      if (SI->getValueOperand() == AI)
        return false; // Don't allow a store OF the AI, only INTO the AI.
      // Note that atomic stores can be transformed; atomic semantics do
      // not have any meaning for a local alloca.
      if (SI->isVolatile())
        return false;
      Type *StoreType = SI->getValueOperand()->getType();
      if (!ExpectedType)
        ExpectedType = StoreType;
      else if (StoreType != ExpectedType)
        return false;
    } else if (const IntrinsicInst *II = dyn_cast<IntrinsicInst>(U)) {
      if (!II->isLifetimeStartOrEnd() && !II->isDroppable() &&
          II->getIntrinsicID() != Intrinsic::fake_use)
        return false;
    } else if (const BitCastInst *BCI = dyn_cast<BitCastInst>(U)) {
      if (!onlyUsedByLifetimeMarkersOrDroppableInsts(BCI))
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        DVRAssigns.push_back(DVR);
    }
  }

  /// Update assignment tracking debug info given for the to-be-deleted store
  /// \p ToDelete that stores to this alloca.
  void updateForDeletedStore(
      StoreInst *ToDelete, DIBuilder &DIB,
      SmallPtrSet<DbgVariableRecord *, 8> *DVRAssignsToDelete) const {
    // There's nothing to do if the alloca doesn't have any variables using
    // assignment tracking.
    if (DVRAssigns.empty())
      return;

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include AssignmentTrackingInfo, init, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 AssignmentTrackingInfo, init，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 156-237

```cpp
    // Insert a dbg.value where the linked dbg.assign is and remember to delete
    // the dbg.assign later. Demoting to dbg.value isn't necessary for
    // correctness but does reduce compile time and memory usage by reducing
    // unnecessary function-local metadata. Remember that we've seen a
    // dbg.assign for each variable fragment for the untracked store handling
    // (after this loop).
    SmallSet<DebugVariableAggregate, 2> VarHasDbgAssignForStore;
    auto InsertValueForAssign = [&](auto *DbgAssign, auto *&AssignList) {
      VarHasDbgAssignForStore.insert(DebugVariableAggregate(DbgAssign));
      AssignList->insert(DbgAssign);
      createDebugValue(DIB, DbgAssign->getValue(), DbgAssign->getVariable(),
                       DbgAssign->getExpression(), DbgAssign->getDebugLoc(),
                       DbgAssign);
    };
    for (auto *Assign : at::getDVRAssignmentMarkers(ToDelete))
      InsertValueForAssign(Assign, DVRAssignsToDelete);

    // It's possible for variables using assignment tracking to have no
    // dbg.assign linked to this store. These are variables in DVRAssigns that
    // are missing from VarHasDbgAssignForStore. Since there isn't a dbg.assign
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    UsingBlocks.clear();
    OnlyStore = nullptr;
    OnlyBlock = nullptr;
    OnlyUsedInOneBlock = true;
    ValueType = nullptr;
    DPUsers.clear();
    AssignmentTracking.clear();
  }

  /// Scan the uses of the specified alloca, filling in the AllocaInfo used
  /// by the rest of the pass to reason about the uses of this alloca.
  void AnalyzeAlloca(AllocaInst *AI) {
    clear();

```
- EN: Core entities appearing here include updateForNewPhi, AllocaInfo, clear, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 updateForNewPhi, AllocaInfo, clear，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 238-317

```cpp
    // As we scan the uses of the alloca instruction, keep track of stores,
    // and decide whether all of the loads and stores to the alloca are within
    // the same basic block.
    for (User *U : AI->users()) {
      Instruction *User = cast<Instruction>(U);

      if (StoreInst *SI = dyn_cast<StoreInst>(User)) {
        // Remember the basic blocks which define new values for the alloca
        DefiningBlocks.push_back(SI->getParent());
        OnlyStore = SI;
        if (!ValueType)
          ValueType = SI->getValueOperand()->getType();
        else
          assert(ValueType == SI->getValueOperand()->getType() &&
                 "All stores were checked to have used the same type");
      } else {
        LoadInst *LI = cast<LoadInst>(User);
        // Otherwise it must be a load instruction, keep track of variable
        // reads.
        UsingBlocks.push_back(LI->getParent());
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    Vals[Idx] = Val;
  }

  void init(size_t Idx, const T &Val) {
    assert(Undo.empty());
    Vals[Idx] = Val;
  }
};

/// Data package used by RenamePass().
struct RenamePassData {
  RenamePassData(BasicBlock *B, BasicBlock *P, size_t V, size_t L)
      : BB(B), Pred(P), UndoVals(V), UndoLocs(L) {}

```
- EN: Core entities appearing here include undo, set, init, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 undo, set, init，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 318-396

```cpp
  BasicBlock *BB;
  BasicBlock *Pred;

  size_t UndoVals;
  size_t UndoLocs;
};

/// This assigns and keeps a per-bb relative ordering of load/store
/// instructions in the block that directly load or store an alloca.
///
/// This functionality is important because it avoids scanning large basic
/// blocks multiple times when promoting many allocas in the same block.
class LargeBlockInfo {
  /// For each instruction that we track, keep the index of the
  /// instruction.
  ///
  /// The index starts out as the number of the instruction from the start of
  /// the block.
  DenseMap<const Instruction *, unsigned> InstNumbers;

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  AssumptionCache *AC;

  const SimplifyQuery SQ;

  /// Reverse mapping of Allocas.
  DenseMap<AllocaInst *, unsigned> AllocaLookup;

  /// The PhiNodes we're adding.
  ///
  /// That map is used to simplify some Phi nodes as we iterate over it, so
  /// it should have deterministic iterators.  We could use a MapVector, but
  /// since basic blocks have numbers, using these are more efficient.
  DenseMap<std::pair<unsigned, unsigned>, PHINode *> NewPhiNodes;

```
- EN: Core entities appearing here include LargeBlockInfo, isInterestingInstruction, getInstructionIndex, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 LargeBlockInfo, isInterestingInstruction, getInstructionIndex，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 397-476

```cpp
  /// For each PHI node, keep track of which entry in Allocas it corresponds
  /// to.
  DenseMap<PHINode *, unsigned> PhiToAllocaMap;

  /// For each alloca, we keep track of the dbg.declare record that
  /// describes it, if any, so that we can convert it to a dbg.value
  /// record if the alloca gets promoted.
  SmallVector<AllocaInfo::DPUserVec, 8> AllocaDPUsers;

  /// For each alloca, keep an instance of a helper class that gives us an easy
  /// way to update assignment tracking debug info if the alloca is promoted.
  SmallVector<AssignmentTrackingInfo, 8> AllocaATInfo;
  /// For each alloca, the type used by all loads/stores of this alloca.
  SmallVector<Type *, 8> AllocaValueTypes;
  /// A set of dbg.assigns to delete because they've been demoted to
  /// dbg.values. Call cleanUpDbgAssigns to delete them.
  SmallPtrSet<DbgVariableRecord *, 8> DVRAssignsToDelete;

  /// The set of basic blocks the renamer has already visited.
  BitVector Visited;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  bool QueuePhiNode(BasicBlock *BB, unsigned AllocaIdx, unsigned &Version);

  /// Delete dbg.assigns that have been demoted to dbg.values.
  void cleanUpDbgAssigns() {
    for (auto *DVR : DVRAssignsToDelete)
      DVR->eraseFromParent();
    DVRAssignsToDelete.clear();
  }

  void pushToWorklist(BasicBlock *BB, BasicBlock *Pred) {
    Worklist.emplace_back(BB, Pred, IncomingVals.undoSize(),
                          IncomingLocs.undoSize());
  }

```
- EN: Core entities appearing here include RemoveFromAllocasList, getNumPreds, cleanUpDbgAssigns, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DataLayout, DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 RemoveFromAllocasList, getNumPreds, cleanUpDbgAssigns，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DataLayout, DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 477-555

```cpp
  RenamePassData popFromWorklist() {
    RenamePassData R = Worklist.back();
    Worklist.pop_back();
    IncomingVals.undo(R.UndoVals);
    IncomingLocs.undo(R.UndoLocs);
    return R;
  }
};

} // end anonymous namespace

/// Given a LoadInst LI this adds assume(LI != null) after it.
static void addAssumeNonNull(AssumptionCache *AC, LoadInst *LI) {
  Function *AssumeIntrinsic =
      Intrinsic::getOrInsertDeclaration(LI->getModule(), Intrinsic::assume);
  ICmpInst *LoadNotNull = new ICmpInst(ICmpInst::ICMP_NE, LI,
                                       Constant::getNullValue(LI->getType()));
  LoadNotNull->insertAfter(LI->getIterator());
  CallInst *CI = CallInst::Create(AssumeIntrinsic, {LoadNotNull});
  CI->insertAfter(LoadNotNull->getIterator());
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        Instruction *Inst = cast<Instruction>(UU.getUser());

        // Drop the use of I in droppable instructions.
        if (Inst->isDroppable()) {
          Inst->dropDroppableUse(UU);
          continue;
        }
        Inst->eraseFromParent();
      }
    }
    I->eraseFromParent();
  }
}

```
- EN: Core entities appearing here include popFromWorklist, addAssumeNonNull, removeIntrinsicUsers, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DataLayout, DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 popFromWorklist, addAssumeNonNull, removeIntrinsicUsers，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DataLayout, DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 556-634

```cpp
/// Rewrite as many loads as possible given a single store.
///
/// When there is only a single store, we can use the domtree to trivially
/// replace all of the dominated loads with the stored value. Do so, and return
/// true if this has successfully promoted the alloca entirely. If this returns
/// false there were some loads which were not dominated by the single store
/// and thus must be phi-ed with undef. We fall back to the standard alloca
/// promotion algorithm in that case.
static bool rewriteSingleStoreAlloca(
    AllocaInst *AI, AllocaInfo &Info, LargeBlockInfo &LBI, const DataLayout &DL,
    DominatorTree &DT, AssumptionCache *AC,
    SmallPtrSet<DbgVariableRecord *, 8> *DVRAssignsToDelete) {
  StoreInst *OnlyStore = Info.OnlyStore;
  Value *ReplVal = OnlyStore->getOperand(0);
  // Loads may either load the stored value or uninitialized memory (undef).
  // If the stored value may be poison, then replacing an uninitialized memory
  // load with it would be incorrect. If the store dominates the load, we know
  // it is always initialized.
  bool RequireDominatingStore =
      isa<Instruction>(ReplVal) || !isGuaranteedNotToBePoison(ReplVal);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    LI->replaceAllUsesWith(ReplVal);
    LI->eraseFromParent();
    LBI.deleteValue(LI);
  }

  // Finally, after the scan, check to see if the store is all that is left.
  if (!Info.UsingBlocks.empty())
    return false; // If not, we'll have to fall back for the remainder.

  DIBuilder DIB(*AI->getModule(), /*AllowUnresolved*/ false);
  // Update assignment tracking info for the store we're going to delete.
  Info.AssignmentTracking.updateForDeletedStore(Info.OnlyStore, DIB,
                                                DVRAssignsToDelete);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DataLayout, DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DataLayout, DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 635-706

```cpp
  // Record debuginfo for the store and remove the declaration's
  // debuginfo.
  for (DbgVariableRecord *DbgItem : Info.DPUsers) {
    if (DbgItem->isAddressOfVariable()) {
      ConvertDebugDeclareToDebugValue(DbgItem, Info.OnlyStore, DIB);
      DbgItem->eraseFromParent();
    } else if (DbgItem->isValueOfVariable() &&
               DbgItem->getExpression()->startsWithDeref()) {
      InsertDebugValueAtStoreLoc(DbgItem, Info.OnlyStore, DIB);
      DbgItem->eraseFromParent();
    } else if (DbgItem->getExpression()->startsWithDeref()) {
      DbgItem->eraseFromParent();
    }
  }

  // Remove dbg.assigns linked to the alloca as these are now redundant.
  at::deleteAssignmentMarkers(AI);

  // Remove the (now dead) store and alloca.
  Info.OnlyStore->eraseFromParent();
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  // Sort the stores by their index, making it efficient to do a lookup with a
  // binary search.
  llvm::sort(StoresByIndex, less_first());

  // Walk all of the loads from this alloca, replacing them with the nearest
  // store above them, if any.
  for (User *U : make_early_inc_range(AI->users())) {
    LoadInst *LI = dyn_cast<LoadInst>(U);
    if (!LI)
      continue;

    unsigned LoadIdx = LBI.getInstructionIndex(LI);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DataLayout, DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DataLayout, DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 707-786

```cpp
    // Find the nearest store that has a lower index than this load.
    StoresByIndexTy::iterator I = llvm::lower_bound(
        StoresByIndex,
        std::make_pair(LoadIdx, static_cast<StoreInst *>(nullptr)),
        less_first());
    Value *ReplVal;
    if (I == StoresByIndex.begin()) {
      if (StoresByIndex.empty())
        // If there are no stores, the load takes the undef value.
        ReplVal = UndefValue::get(LI->getType());
      else
        // There is no store before this load, bail out (load may be affected
        // by the following stores - see main comment).
        return false;
    } else {
      // Otherwise, there was a store before this load, the load takes its
      // value.
      ReplVal = std::prev(I)->second->getOperand(0);
    }

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  AllocaATInfo.resize(Allocas.size());
  AllocaDPUsers.resize(Allocas.size());
  AllocaValueTypes.resize(Allocas.size());

  AllocaInfo Info;
  LargeBlockInfo LBI;
  ForwardIDFCalculator IDF(DT);

  NoSignedZeros = F.getFnAttribute("no-signed-zeros-fp-math").getValueAsBool();

  for (unsigned AllocaNum = 0; AllocaNum != Allocas.size(); ++AllocaNum) {
    AllocaInst *AI = Allocas[AllocaNum];

```
- EN: Core entities appearing here include run, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 run，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 787-868

```cpp
    assert(isAllocaPromotable(AI) && "Cannot promote non-promotable alloca!");
    assert(AI->getParent()->getParent() == &F &&
           "All allocas should be in the same function, which is same as DF!");

    removeIntrinsicUsers(AI);

    if (AI->use_empty()) {
      // If there are no uses of the alloca, just delete it now.
      AI->eraseFromParent();

      // Remove the alloca from the Allocas list, since it has been processed
      RemoveFromAllocasList(AllocaNum);
      ++NumDeadAlloca;
      continue;
    }

    // Calculate the set of read and write-locations for each alloca.  This is
    // analogous to finding the 'uses' and 'definitions' of each variable.
    Info.AnalyzeAlloca(AI);

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    // dead phi nodes.
    IDF.setLiveInBlocks(LiveInBlocks);
    IDF.setDefiningBlocks(DefBlocks);
    SmallVector<BasicBlock *, 32> PHIBlocks;
    IDF.calculate(PHIBlocks);
    llvm::sort(PHIBlocks, [](BasicBlock *A, BasicBlock *B) {
      return A->getNumber() < B->getNumber();
    });

    unsigned CurrentVersion = 0;
    for (BasicBlock *BB : PHIBlocks)
      QueuePhiNode(BB, AllocaNum, CurrentVersion);
  }

```
- EN: Core entities appearing here include sort, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 sort，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 869-949

```cpp
  if (Allocas.empty()) {
    cleanUpDbgAssigns();
    return; // All of the allocas must have been trivial!
  }
  LBI.clear();

  // Set the incoming values for the basic block to be null values for all of
  // the alloca's.  We do this in case there is a load of a value that has not
  // been stored yet.  In this case, it will get this null value.
  IncomingVals.resize(Allocas.size());
  for (unsigned i = 0, e = Allocas.size(); i != e; ++i)
    IncomingVals.init(i, UndefValue::get(AllocaValueTypes[i]));

  // When handling debug info, treat all incoming values as if they have
  // compiler-generated (empty) locations, representing the uninitialized
  // alloca, until proven otherwise.
  IncomingLocs.resize(Allocas.size());
  for (unsigned i = 0, e = Allocas.size(); i != e; ++i)
    IncomingLocs.init(i, DebugLoc::getCompilerGenerated());

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      PHINode *PN = I->second;

      // If this PHI node merges one value and/or undefs, get the value.
      if (Value *V = simplifyInstruction(PN, SQ)) {
        PN->replaceAllUsesWith(V);
        PN->eraseFromParent();
        NewPhiNodes.erase(I++);
        EliminatedAPHI = true;
        continue;
      }
      ++I;
    }
  }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 950-1033

```cpp
  // At this point, the renamer has added entries to PHI nodes for all reachable
  // code.  Unfortunately, there may be unreachable blocks which the renamer
  // hasn't traversed.  If this is the case, the PHI nodes may not
  // have incoming values for all predecessors.  Loop over all PHI nodes we have
  // created, inserting poison values if they are missing any incoming values.
  for (const auto &PhiNode : NewPhiNodes) {
    // We want to do this once per basic block.  As such, only process a block
    // when we find the PHI that is the first entry in the block.
    PHINode *SomePHI = PhiNode.second;
    BasicBlock *BB = SomePHI->getParent();
    if (&BB->front() != SomePHI)
      continue;

    // Only do work here if there the PHI nodes are missing incoming values.  We
    // know that all PHI nodes that were inserted in a block will have the same
    // number of incoming values, so we can just check any of them.
    if (SomePHI->getNumIncomingValues() == getNumPreds(BB))
      continue;

    // Get the preds for BB.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // To determine liveness, we must iterate through the predecessors of blocks
  // where the def is live.  Blocks are added to the worklist if we need to
  // check their predecessors.  Start with all the using blocks.
  SmallVector<BasicBlock *, 64> LiveInBlockWorklist(Info.UsingBlocks.begin(),
                                                    Info.UsingBlocks.end());

  // If any of the using blocks is also a definition block, check to see if the
  // definition occurs before or after the use.  If it happens before the use,
  // the value isn't really live-in.
  for (unsigned i = 0, e = LiveInBlockWorklist.size(); i != e; ++i) {
    BasicBlock *BB = LiveInBlockWorklist[i];
    if (!DefBlocks.count(BB))
      continue;

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1034-1113

```cpp
    // Okay, this is a block that both uses and defines the value.  If the first
    // reference to the alloca is a def (store), then we know it isn't live-in.
    for (BasicBlock::iterator I = BB->begin();; ++I) {
      if (StoreInst *SI = dyn_cast<StoreInst>(I)) {
        if (SI->getOperand(1) != AI)
          continue;

        // We found a store to the alloca before a load.  The alloca is not
        // actually live-in here.
        LiveInBlockWorklist[i] = LiveInBlockWorklist.back();
        LiveInBlockWorklist.pop_back();
        --i;
        --e;
        break;
      }

      if (LoadInst *LI = dyn_cast<LoadInst>(I))
        // Okay, we found a load before a store to the alloca.  It is actually
        // live into this block.
        if (LI->getOperand(0) == AI)
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  PhiToAllocaMap[PN] = AllocaNo;
  return true;
}

/// Update the debug location of a phi. \p ApplyMergedLoc indicates whether to
/// create a merged location incorporating \p DL, or to set \p DL directly.
static void updateForIncomingValueLocation(PHINode *PN, DebugLoc DL,
                                           bool ApplyMergedLoc) {
  if (ApplyMergedLoc)
    PN->applyMergedLocation(PN->getDebugLoc(), DL);
  else
    PN->setDebugLoc(DL);
}

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1114-1193

```cpp
/// Recursively traverse the CFG of the function, renaming loads and
/// stores to the allocas which we are promoting.
///
/// IncomingVals indicates what value each Alloca contains on exit from the
/// predecessor block Pred.
void PromoteMem2Reg::RenamePass(BasicBlock *BB, BasicBlock *Pred) {
  // If we are inserting any phi nodes into this BB, they will already be in the
  // block.
  if (PHINode *APN = dyn_cast<PHINode>(BB->begin())) {
    // If we have PHI nodes to update, compute the number of edges from Pred to
    // BB.
    if (PhiToAllocaMap.count(APN)) {
      // We want to be able to distinguish between PHI nodes being inserted by
      // this invocation of mem2reg from those phi nodes that already existed in
      // the IR before mem2reg was run.  We determine that APN is being inserted
      // because it is missing incoming edges.  All other PHI nodes being
      // inserted by this pass of mem2reg will have the same number of incoming
      // operands so far.  Remember this count.
      unsigned NewPHINumOperands = APN->getNumOperands();

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  Visited.set(BB->getNumber());

  for (BasicBlock::iterator II = BB->begin(); !II->isTerminator();) {
    Instruction *I = &*II++; // get the instruction, increment iterator

    if (LoadInst *LI = dyn_cast<LoadInst>(I)) {
      AllocaInst *Src = dyn_cast<AllocaInst>(LI->getPointerOperand());
      if (!Src)
        continue;

      auto AI = AllocaLookup.find(Src);
      if (AI == AllocaLookup.end())
        continue;

```
- EN: Core entities appearing here include RenamePass, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 RenamePass，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1194-1243

```cpp
      Value *V = IncomingVals[AI->second];
      convertMetadataToAssumes(LI, V, SQ.DL, AC, &DT);

      // Anything using the load now uses the current value.
      LI->replaceAllUsesWith(V);
      LI->eraseFromParent();
    } else if (StoreInst *SI = dyn_cast<StoreInst>(I)) {
      // Delete this instruction and mark the name as the current holder of the
      // value
      AllocaInst *Dest = dyn_cast<AllocaInst>(SI->getPointerOperand());
      if (!Dest)
        continue;

      auto ai = AllocaLookup.find(Dest);
      if (ai == AllocaLookup.end())
        continue;

      // what value were we writing?
      unsigned AllocaNo = ai->second;
      IncomingVals.set(AllocaNo, SI->getOperand(0));

      // Record debuginfo for the store before removing it.
      IncomingLocs.set(AllocaNo, SI->getDebugLoc());
      AllocaATInfo[AllocaNo].updateForDeletedStore(SI, DIB,
                                                   &DVRAssignsToDelete);
      for (DbgVariableRecord *DbgItem : AllocaDPUsers[ai->second])
        if (DbgItem->isAddressOfVariable())
          ConvertDebugDeclareToDebugValue(DbgItem, SI, DIB);
      SI->eraseFromParent();
    }
  }

  // 'Recurse' to our successors.

  // Keep track of the successors so we don't visit the same successor twice
  SmallPtrSet<BasicBlock *, 8> VisitedSuccs;

  for (BasicBlock *S : reverse(successors(BB)))
    if (VisitedSuccs.insert(S).second)
      pushToWorklist(S, BB);
}

void llvm::PromoteMemToReg(ArrayRef<AllocaInst *> Allocas, DominatorTree &DT,
                           AssumptionCache *AC) {
  // If there is nothing to do, bail out...
  if (Allocas.empty())
    return;

  PromoteMem2Reg(Allocas, DT, AC).run();
}
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `AssignmentTrackingInfo, AllocaInfo, RenamePassData, LargeBlockInfo, PromoteMem2Reg, isAllocaPromotable, init, updateForNewPhi` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`AssignmentTrackingInfo, AllocaInfo, RenamePassData, LargeBlockInfo, PromoteMem2Reg, isAllocaPromotable, init, updateForNewPhi` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AssumptionCache, DataLayout, DominatorTree, ValueTracking` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AssumptionCache, DataLayout, DominatorTree, ValueTracking` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/AssumptionCache.h`, `llvm/Analysis/InstructionSimplify.h`, `llvm/Analysis/IteratedDominanceFrontier.h`, `llvm/Analysis/ValueTracking.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/AssumptionCache.h`, `llvm/Analysis/InstructionSimplify.h`, `llvm/Analysis/IteratedDominanceFrontier.h`, `llvm/Analysis/ValueTracking.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/BasicBlock.h`, `llvm/IR/CFG.h`, `llvm/IR/Constant.h`, `llvm/IR/Constants.h`, `llvm/IR/DIBuilder.h`, `llvm/IR/DebugInfo.h`, `llvm/IR/DebugProgramInstruction.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/BasicBlock.h`, `llvm/IR/CFG.h`, `llvm/IR/Constant.h`, `llvm/IR/Constants.h`, `llvm/IR/DIBuilder.h`, `llvm/IR/DebugInfo.h`, `llvm/IR/DebugProgramInstruction.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Casting.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/BitVector.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h`, `llvm/ADT/Twine.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Casting.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/BitVector.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h`, `llvm/ADT/Twine.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `algorithm`, `cassert`, `iterator`, `utility`, `vector` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`algorithm`, `cassert`, `iterator`, `utility`, `vector` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `AssumptionCache`, `DataLayout`, `DominatorTree`, `ValueTracking` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AssumptionCache`, `DataLayout`, `DominatorTree`, `ValueTracking` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
