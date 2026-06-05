# RewriteStatepointsForGC.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Scalar/RewriteStatepointsForGC.cpp` | `llvm/lib/Transforms/Scalar/RewriteStatepointsForGC.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements make GC relocations explicit within LLVM's scalar optimizations and loop/CFG transforms layer. | 该源文件实现了 LLVM 标量优化以及循环/CFG 变换层中的 RewriteStatepointsForGC 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-217

```cpp
//===- RewriteStatepointsForGC.cpp - Make GC relocations explicit ---------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Rewrite call/invoke instructions so as to make potential relocations
// performed by the garbage collector explicit in the IR.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Scalar/RewriteStatepointsForGC.h"

#include "llvm/ADT/ArrayRef.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/ADT/DenseSet.h"
#include "llvm/ADT/MapVector.h"
#include "llvm/ADT/STLExtras.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  /// The *new* gc.statepoint instruction itself.  This produces the token
  /// that normal path gc.relocates and the gc.result are tied to.
  GCStatepointInst *StatepointToken;

  /// Instruction to which exceptional gc relocates are attached
  /// Makes it easier to iterate through them during relocationViaAlloca.
  Instruction *UnwindToken;

  /// Record live values we are rematerialized instead of relocating.
  /// They are not included into 'LiveSet' field.
  /// Maps rematerialized copy to it's original value.
  RematerializedValueMapTy RematerializedValues;
};

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 218-435

```cpp
struct RematerizlizationCandidateRecord {
  // Chain from derived pointer to base.
  SmallVector<Instruction *, 3> ChainToBase;
  // Original base.
  Value *RootOfChain;
  // Cost of chain.
  InstructionCost Cost;
};
using RematCandTy = MapVector<Value *, RematerizlizationCandidateRecord>;

} // end anonymous namespace

static ArrayRef<Use> GetDeoptBundleOperands(const CallBase *Call) {
  std::optional<OperandBundleUse> DeoptBundle =
      Call->getOperandBundle(LLVMContext::OB_deopt);

  if (!DeoptBundle) {
    assert(AllowStatepointWithNoDeoptInfo &&
           "Found non-leaf call without deopt info!");
    return {};
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    auto *BDV = findBaseDefiningValue(BC->getOperand(0), Cache, KnownBases);
    Cache[BC] = BDV;
    return BDV;
  }

  // We assume that functions in the source language only return base
  // pointers.  This should probably be generalized via attributes to support
  // both source language and internal functions.
  if (isa<CallInst>(I) || isa<InvokeInst>(I)) {
    Cache[I] = I;
    setKnownBase(I, /* IsKnownBase */true, KnownBases);
    return I;
  }

```
- EN: Core entities appearing here include RematerizlizationCandidateRecord, GetDeoptBundleOperands, isGCPointerType, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree.
- CN: 此处出现的核心实体包括 RematerizlizationCandidateRecord, GetDeoptBundleOperands, isGCPointerType，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。

### Lines 436-658

```cpp
  // A PHI or Select is a base defining value.  The outer findBasePointer
  // algorithm is responsible for constructing a base value for this BDV.
  assert((isa<SelectInst>(I) || isa<PHINode>(I)) &&
         "unknown vector instruction - no base found for vector element");
  Cache[I] = I;
  setKnownBase(I, /* IsKnownBase */false, KnownBases);
  return I;
}

/// Helper function for findBasePointer - Will return a value which either a)
/// defines the base pointer for the input, b) blocks the simple search
/// (i.e. a PHI or Select of two derived pointers), or c) involves a change
/// from pointer to vector type or back.
static Value *findBaseDefiningValue(Value *I, DefiningValueMapTy &Cache,
                                    IsKnownBaseMapTy &KnownBases) {
  assert(I->getType()->isPtrOrPtrVectorTy() &&
         "Illegal to ask for the base pointer of a non-pointer type");
  auto Cached = Cache.find(I);
  if (Cached != Cache.end())
    return Cached->second;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
/// Return a base pointer for this value if known.  Otherwise, return it's
/// base defining value.
static Value *findBaseOrBDV(Value *I, DefiningValueMapTy &Cache,
                            IsKnownBaseMapTy &KnownBases) {
  Value *Def = findBaseDefiningValueCached(I, Cache, KnownBases);
  auto Found = Cache.find(Def);
  if (Found != Cache.end()) {
    // Either a base-of relation, or a self reference.  Caller must check.
    return Found->second;
  }
  // Only a BDV available
  return Def;
}

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 659-885

```cpp
#ifndef NDEBUG
/// This value is a base pointer that is not generated by RS4GC, i.e. it already
/// exists in the code.
static bool isOriginalBaseResult(Value *V) {
  // no recursion possible
  return !isa<PHINode>(V) && !isa<SelectInst>(V) &&
         !isa<ExtractElementInst>(V) && !isa<InsertElementInst>(V) &&
         !isa<ShuffleVectorInst>(V);
}
#endif

static bool isKnownBase(Value *V, const IsKnownBaseMapTy &KnownBases) {
  auto It = KnownBases.find(V);
  assert(It != KnownBases.end() && "Value not present in the map");
  return It->second;
}

static void setKnownBase(Value *V, bool IsKnownBase,
                         IsKnownBaseMapTy &KnownBases) {
#ifndef NDEBUG
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    } else if (auto *IE = dyn_cast<InsertElementInst>(BDV)) {
      F(IE->getOperand(0));
      F(IE->getOperand(1));
    } else if (auto *SV = dyn_cast<ShuffleVectorInst>(BDV)) {
      // For a canonical broadcast, ignore the undef argument
      // (without this, we insert a parallel base shuffle for every broadcast)
      F(SV->getOperand(0));
      if (!SV->isZeroEltSplat())
        F(SV->getOperand(1));
    } else {
      llvm_unreachable("unexpected BDV type");
    }
  };

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include isOriginalBaseResult, isKnownBase, areBothVectorOrScalar, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 isOriginalBaseResult, isKnownBase, areBothVectorOrScalar，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 886-1098

```cpp

  // Recursively fill in all base defining values reachable from the initial
  // one for which we don't already know a definite base value for
  /* scope */ {
    SmallVector<Value*, 16> Worklist;
    Worklist.push_back(Def);
    States.insert({Def, BDVState(Def)});
    while (!Worklist.empty()) {
      Value *Current = Worklist.pop_back_val();
      assert(!isOriginalBaseResult(Current) && "why did it get added?");

      auto visitIncomingValue = [&](Value *InVal) {
        Value *Base = findBaseOrBDV(InVal, Cache, KnownBases);
        if (isKnownBase(Base, KnownBases) && areBothVectorOrScalar(Base, InVal))
          // Known bases won't need new instructions introduced and can be
          // ignored safely. However, this can only be done when InVal and Base
          // are both scalar or both vector. Otherwise, we need to find a
          // correct BDV for InVal, by creating an entry in the lattice
          // (States).
          return;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    BDVState State = Pair.second;
    // Only values that do not have known bases or those that have differing
    // type (scalar versus vector) from a possible known base should be in the
    // lattice.
    assert((!isKnownBase(I, KnownBases) ||
            !areBothVectorOrScalar(I, State.getBaseValue())) &&
           "why did it get added?");
    assert(!State.isUnknown() && "Optimistic algorithm didn't complete!");

    // Since we're joining a vector and scalar base, they can never be the
    // same.  As a result, we should always see insert element having reached
    // the conflict state.
    assert(!isa<InsertElementInst>(I) || State.isConflict());

```
- EN: Core entities appearing here include visitBDVOperands, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 visitBDVOperands，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1099-1321

```cpp
    if (!State.isConflict())
      continue;

    auto getMangledName = [](Instruction *I) -> std::string {
      if (isa<PHINode>(I)) {
        return suffixed_name_or(I, ".base", "base_phi");
      } else if (isa<SelectInst>(I)) {
        return suffixed_name_or(I, ".base", "base_select");
      } else if (isa<ExtractElementInst>(I)) {
        return suffixed_name_or(I, ".base", "base_ee");
      } else if (isa<InsertElementInst>(I)) {
        return suffixed_name_or(I, ".base", "base_ie");
      } else {
        return suffixed_name_or(I, ".base", "base_sv");
      }
    };

    Instruction *BaseInst = I->clone();
    BaseInst->insertBefore(I->getIterator());
    BaseInst->setName(getMangledName(I));
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                             PointerToBaseTy &PointerToBase, DominatorTree *DT,
                             DefiningValueMapTy &DVCache,
                             IsKnownBaseMapTy &KnownBases) {
  for (Value *ptr : live) {
    Value *base = findBasePointer(ptr, DVCache, KnownBases);
    assert(base && "failed to find base pointer");
    PointerToBase[ptr] = base;
    assert((!isa<Instruction>(base) || !isa<Instruction>(ptr) ||
            DT->dominates(cast<Instruction>(base)->getParent(),
                          cast<Instruction>(ptr)->getParent())) &&
           "The base we found better dominate the derived pointer");
  }
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1322-1535

```cpp
/// Find the required based pointers (and adjust the live set) for the given
/// parse point.
static void findBasePointers(DominatorTree &DT, DefiningValueMapTy &DVCache,
                             CallBase *Call,
                             PartiallyConstructedSafepointRecord &result,
                             PointerToBaseTy &PointerToBase,
                             IsKnownBaseMapTy &KnownBases) {
  StatepointLiveSetTy PotentiallyDerivedPointers = result.LiveSet;
  // We assume that all pointers passed to deopt are base pointers; as an
  // optimization, we can use this to avoid separately materializing the base
  // pointer graph.  This is only relevant since we're very conservative about
  // generating new conflict nodes during base pointer insertion.  If we were
  // smarter there, this would be irrelevant.
  if (auto Opt = Call->getOperandBundle(LLVMContext::OB_deopt))
    for (Value *V : Opt->Inputs) {
      if (!PotentiallyDerivedPointers.count(V))
        continue;
      PotentiallyDerivedPointers.remove(V);
      PointerToBase[V] = V;
    }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // towards a single unified pointer type anyways, we can just cast everything
  // to an i8* of the right address space.  A bitcast is added later to convert
  // gc_relocate to the actual value's type.
  auto getGCRelocateDecl = [&](Type *Ty) {
    assert(isHandledGCPointerType(Ty, GC));
    auto AS = Ty->getScalarType()->getPointerAddressSpace();
    Type *NewTy = PointerType::get(M->getContext(), AS);
    if (auto *VT = dyn_cast<VectorType>(Ty))
      NewTy = FixedVectorType::get(NewTy,
                                   cast<FixedVectorType>(VT)->getNumElements());
    return Intrinsic::getOrInsertDeclaration(
        M, Intrinsic::experimental_gc_relocate, {NewTy});
  };

```
- EN: Core entities appearing here include PartiallyConstructedSafepointRecord, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 PartiallyConstructedSafepointRecord，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1536-1764

```cpp
  // Lazily populated map from input types to the canonicalized form mentioned
  // in the comment above.  This should probably be cached somewhere more
  // broadly.
  DenseMap<Type *, Function *> TypeToDeclMap;

  for (unsigned i = 0; i < LiveVariables.size(); i++) {
    // Generate the gc.relocate call and save the result
    Value *BaseIdx = Builder.getInt32(FindIndex(LiveVariables, BasePtrs[i]));
    Value *LiveIdx = Builder.getInt32(i);

    Type *Ty = LiveVariables[i]->getType();
    auto [It, Inserted] = TypeToDeclMap.try_emplace(Ty);
    if (Inserted)
      It->second = getGCRelocateDecl(Ty);
    Function *GCRelocateDecl = It->second;

    // only specify a debug name if we can give a useful one
    CallInst *Reloc = Builder.CreateCall(
        GCRelocateDecl, {StatepointToken, BaseIdx, LiveIdx},
        suffixed_name_or(LiveVariables[i], ".relocated", ""));
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
              ConstantPointerNull::get(cast<PointerType>(Derived->getType()));
        else {
          assert(PointerToBase.count(Derived));
          Base = PointerToBase.find(Derived)->second;
        }
        unsigned AddressSpace = Derived->getType()->getPointerAddressSpace();
        unsigned IntPtrSize = DL.getPointerSizeInBits(AddressSpace);
        Value *Base_int = Builder.CreatePtrToInt(
            Base, Type::getIntNTy(Context, IntPtrSize));
        Value *Derived_int = Builder.CreatePtrToInt(
            Derived, Type::getIntNTy(Context, IntPtrSize));
        return std::make_pair(Base, Builder.CreateSub(Derived_int, Base_int));
      };

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include DeferredReplacement, createRAUW, createDelete, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 DeferredReplacement, createRAUW, createDelete，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。

### Lines 1765-1977

```cpp
      auto *Dest = CallArgs[0];
      Value *DestBase, *DestOffset;
      std::tie(DestBase, DestOffset) = GetBaseAndOffset(Dest);

      auto *Source = CallArgs[1];
      Value *SourceBase, *SourceOffset;
      std::tie(SourceBase, SourceOffset) = GetBaseAndOffset(Source);

      auto *LengthInBytes = CallArgs[2];
      auto *ElementSizeCI = cast<ConstantInt>(CallArgs[3]);

      CallArgs.clear();
      CallArgs.push_back(DestBase);
      CallArgs.push_back(DestOffset);
      CallArgs.push_back(SourceBase);
      CallArgs.push_back(SourceOffset);
      CallArgs.push_back(LengthInBytes);

      SmallVector<Type *, 8> DomainTy;
      for (Value *Arg : CallArgs)
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
//
// It receives iterator to the statepoint gc relocates and emits a store to the
// assigned location (via allocaMap) for the each one of them.  It adds the
// visited values into the visitedLiveValues set, which we will later use them
// for validation checking.
static void
insertRelocationStores(iterator_range<Value::user_iterator> GCRelocs,
                       DenseMap<Value *, AllocaInst *> &AllocaMap,
                       DenseSet<Value *> &VisitedLiveValues) {
  for (User *U : GCRelocs) {
    GCRelocateInst *Relocate = dyn_cast<GCRelocateInst>(U);
    if (!Relocate)
      continue;

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1978-2201

```cpp
    Value *OriginalValue = Relocate->getDerivedPtr();
    assert(AllocaMap.count(OriginalValue));
    Value *Alloca = AllocaMap[OriginalValue];

    // Emit store into the related alloca.
    assert(Relocate->getNextNode() &&
           "Should always have one since it's not a terminator");
    new StoreInst(Relocate, Alloca, std::next(Relocate->getIterator()));

#ifndef NDEBUG
    VisitedLiveValues.insert(OriginalValue);
#endif
  }
}

// Helper function for the "relocationViaAlloca". Similar to the
// "insertRelocationStores" but works for rematerialized values.
static void insertRematerializationStores(
    const RematerializedValueMapTy &RematerializedValues,
    DenseMap<Value *, AllocaInst *> &AllocaMap,
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        BasicBlock *NormalDest = Invoke->getNormalDest();
        Store->insertBefore(NormalDest->getFirstNonPHIIt());
      } else {
        assert(!Inst->isTerminator() &&
               "The only terminator that can produce a value is "
               "InvokeInst which is handled above.");
        Store->insertAfter(Inst->getIterator());
      }
    } else {
      assert(isa<Argument>(Def));
      Store->insertAfter(cast<Instruction>(Alloca)->getIterator());
    }
  }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2202-2419

```cpp
  assert(PromotableAllocas.size() == Live.size() + NumRematerializedValues &&
         "we must have the same allocas with lives");
  (void) NumRematerializedValues;
  if (!PromotableAllocas.empty()) {
    // Apply mem2reg to promote alloca to SSA
    PromoteMemToReg(PromotableAllocas, DT);
  }

#ifndef NDEBUG
  for (auto &I : F.getEntryBlock())
    if (isa<AllocaInst>(I))
      InitialAllocaNum--;
  assert(InitialAllocaNum == 0 && "We must not introduce any extra allocas");
#endif
}

/// Insert holders so that each Value is obviously live through the entire
/// lifetime of the call.
static void insertUseHolderAfter(CallBase *Call, const ArrayRef<Value *> Values,
                                 SmallVectorImpl<CallInst *> &Holders) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  }
}

// Try to rematerialize derived pointers immediately before their uses
// (instead of rematerializing after every statepoint it is live through).
// This can be beneficial when derived pointer is live across many
// statepoints, but uses are rare.
static void rematerializeLiveValuesAtUses(
    RematCandTy &RematerizationCandidates,
    MutableArrayRef<PartiallyConstructedSafepointRecord> Records,
    PointerToBaseTy &PointerToBase) {
  if (!RematDerivedAtUses)
    return;

```
- EN: Core entities appearing here include PartiallyConstructedSafepointRecord, AreEquivalentPhiNodes, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DominatorTree, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 PartiallyConstructedSafepointRecord, AreEquivalentPhiNodes，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DominatorTree, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2420-2644

```cpp
  SmallVector<Instruction *, 32> LiveValuesToBeDeleted;

  LLVM_DEBUG(dbgs() << "Rematerialize derived pointers at uses, "
                    << "Num statepoints: " << Records.size() << '\n');

  for (auto &It : RematerizationCandidates) {
    Instruction *Cand = cast<Instruction>(It.first);
    auto &Record = It.second;

    if (Record.Cost >= RematerializationThreshold)
      continue;

    if (Cand->user_empty())
      continue;

    if (Cand->hasOneUse())
      if (auto *U = dyn_cast<Instruction>(Cand->getUniqueUndroppableUser()))
        if (U->getParent() == Cand->getParent())
          continue;

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                                 suffixed_name_or(Base, ".int", ""));
      Value *DerivedInt =
          Builder.CreatePtrToInt(Derived, Type::getIntNTy(Context, IntPtrSize),
                                 suffixed_name_or(Derived, ".int", ""));
      Value *Offset = Builder.CreateSub(DerivedInt, BaseInt);
      Callsite->replaceAllUsesWith(Offset);
      Offset->takeName(Callsite);
      Callsite->eraseFromParent();
      break;
    }
    default:
      llvm_unreachable("Unknown intrinsic");
    }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2645-2865

```cpp
  return Changed;
}

static bool insertParsePoints(Function &F, DominatorTree &DT,
                              TargetTransformInfo &TTI,
                              SmallVectorImpl<CallBase *> &ToUpdate,
                              DefiningValueMapTy &DVCache,
                              IsKnownBaseMapTy &KnownBases) {
  std::unique_ptr<GCStrategy> GC = findGCStrategy(F);

#ifndef NDEBUG
  // Validate the input
  std::set<CallBase *> Uniqued;
  Uniqued.insert(ToUpdate.begin(), ToUpdate.end());
  assert(Uniqued.size() == ToUpdate.size() && "no duplicates please!");

  for (CallBase *Call : ToUpdate)
    assert(Call->getFunction() == &F);
#endif

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
           "statepoint must be reachable or liveness is meaningless");
    for (Value *V : Info.StatepointToken->gc_live()) {
      if (!isa<Instruction>(V))
        // Non-instruction values trivial dominate all possible uses
        continue;
      auto *LiveInst = cast<Instruction>(V);
      assert(DT.isReachableFromEntry(LiveInst->getParent()) &&
             "unreachable values should never be live");
      assert(DT.dominates(LiveInst, Info.StatepointToken) &&
             "basic SSA liveness expectation violated by liveness analysis");
    }
#endif
  }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2866-3091

```cpp
#ifndef NDEBUG
  // Validation check
  for (auto *Ptr : Live)
    assert(isHandledGCPointerType(Ptr->getType(), GC.get()) &&
           "must be a gc pointer type");
#endif

  relocationViaAlloca(F, DT, Live.getArrayRef(), Records);
  return !Records.empty();
}

// List of all parameter and return attributes which must be stripped when
// lowering from the abstract machine model.  Note that we list attributes
// here which aren't valid as return attributes, that is okay.
static AttributeMask getParamAndReturnAttributesToRemove() {
  AttributeMask R;
  R.addAttribute(Attribute::Dereferenceable);
  R.addAttribute(Attribute::DereferenceableOrNull);
  R.addAttribute(Attribute::ReadNone);
  R.addAttribute(Attribute::ReadOnly);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      // NOTE removeUnreachableBlocks() is stronger than
      // DominatorTree::isReachableFromEntry(). In other words
      // removeUnreachableBlocks can remove some blocks for which
      // isReachableFromEntry() returns true.
      assert(DT.isReachableFromEntry(I.getParent()) &&
            "no unreachable blocks expected");
      ParsePointNeeded.push_back(cast<CallBase>(&I));
    }
    if (auto *CI = dyn_cast<CallInst>(&I))
      if (CI->getIntrinsicID() == Intrinsic::experimental_gc_get_pointer_base ||
          CI->getIntrinsicID() == Intrinsic::experimental_gc_get_pointer_offset)
        Intrinsics.emplace_back(CI);
  }

```
- EN: Core entities appearing here include getParamAndReturnAttributesToRemove, stripNonValidAttributesFromPrototype, stripInvalidMetadataFromInstruction, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, DomTreeUpdater, TargetLibraryInfo, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 getParamAndReturnAttributesToRemove, stripNonValidAttributesFromPrototype, stripInvalidMetadataFromInstruction，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, DomTreeUpdater, TargetLibraryInfo, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 3092-3306

```cpp
  // Return early if no work to do.
  if (ParsePointNeeded.empty() && Intrinsics.empty())
    return MadeChange;

  // As a prepass, go ahead and aggressively destroy single entry phi nodes.
  // These are created by LCSSA.  They have the effect of increasing the size
  // of liveness sets for no good reason.  It may be harder to do this post
  // insertion since relocations and base phis can confuse things.
  for (BasicBlock &BB : F)
    if (BB.getUniquePredecessor())
      MadeChange |= FoldSingleEntryPHINodes(&BB);

  // Before we start introducing relocations, we want to tweak the IR a bit to
  // avoid unfortunate code generation effects.  The main example is that we
  // want to try to make sure the comparison feeding a branch is after any
  // safepoints.  Otherwise, we end up with a comparison of pre-relocation
  // values feeding a branch after relocation.  This is semantically correct,
  // but results in extra register pressure since both the pre-relocation and
  // post-relocation copies must be available in registers.  For code without
  // relocations this is handled elsewhere, but teaching the scheduler to
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

    auto &Out = Data.LiveOut[&BB] = SetVector<Value *>();
    computeLiveOutSeed(&BB, Out, GC);
    auto &In = Data.LiveIn[&BB] = Data.LiveSet[&BB];
    In.set_union(Out);
    In.set_subtract(Data.KillSet[&BB]);
    if (!In.empty())
      Worklist.insert_range(predecessors(&BB));
  }

  // Propagate that liveness until stable
  while (!Worklist.empty()) {
    BasicBlock *BB = Worklist.pop_back_val();

```
- EN: Core entities appearing here include computeKillSet, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 computeKillSet，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 3307-3377

```cpp
    // Compute our new liveout set, then exit early if it hasn't changed despite
    // the contribution of our successor.
    SetVector<Value *> &LiveOut = Data.LiveOut[BB];
    const auto OldLiveOutSize = LiveOut.size();
    for (BasicBlock *Succ : successors(BB)) {
      assert(Data.LiveIn.count(Succ));
      LiveOut.set_union(Data.LiveIn[Succ]);
    }
    // assert OutLiveOut is a subset of LiveOut
    if (OldLiveOutSize == LiveOut.size()) {
      // If the sets are the same size, then we didn't actually add anything
      // when unioning our successors LiveIn.  Thus, the LiveIn of this block
      // hasn't changed.
      continue;
    }

    // Apply the effects of this basic block
    SetVector<Value *> LiveTmp = LiveOut;
    LiveTmp.set_union(Data.LiveSet[BB]);
    LiveTmp.set_subtract(Data.KillSet[BB]);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                                  CallBase *Call,
                                  PartiallyConstructedSafepointRecord &Info,
                                  PointerToBaseTy &PointerToBase,
                                  GCStrategy *GC) {
  StatepointLiveSetTy Updated;
  findLiveSetAtInst(Call, RevisedLivenessData, Updated, GC);

  // We may have base pointers which are now live that weren't before.  We need
  // to update the PointerToBase structure to reflect this.
  for (auto *V : Updated)
    PointerToBase.insert({ V, V });

  Info.LiveSet = Updated;
}
```
- EN: This region continues the RewriteStatepointsForGC implementation with local helper logic centered on Compute, SetVector, Value, LiveOut.
- CN: 这一段延续了 RewriteStatepointsForGC 的主体实现，围绕 Compute, SetVector, Value, LiveOut 等局部辅助逻辑展开。

## Key Concepts / 关键概念

- EN: Domain: `Scalar` focuses on scalar optimizations and loop/CFG transforms.
  - CN: 领域：`Scalar` 主要处理 标量优化以及循环/CFG 变换。
- EN: Primary entities: `GCPtrLivenessData, PartiallyConstructedSafepointRecord, RematerizlizationCandidateRecord, BDVState, StatusTy, DeferredReplacement, GetDeoptBundleOperands, isGCPointerType` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`GCPtrLivenessData, PartiallyConstructedSafepointRecord, RematerizlizationCandidateRecord, BDVState, StatusTy, DeferredReplacement, GetDeoptBundleOperands, isGCPointerType` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DataLayout, DominatorTree, DomTreeUpdater, TargetLibraryInfo, TargetTransformInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DataLayout, DominatorTree, DomTreeUpdater, TargetLibraryInfo, TargetTransformInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `PrintLiveSet, PrintLiveSetSize, PrintBasePointers, RematerializationThreshold, ClobberNonLiveOverride` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `PrintLiveSet, PrintLiveSetSize, PrintBasePointers, RematerializationThreshold, ClobberNonLiveOverride` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/TargetLibraryInfo.h`, `llvm/Analysis/TargetTransformInfo.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/TargetLibraryInfo.h`, `llvm/Analysis/TargetTransformInfo.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Argument.h`, `llvm/IR/AttributeMask.h`, `llvm/IR/Attributes.h`, `llvm/IR/BasicBlock.h`, `llvm/IR/CallingConv.h`, `llvm/IR/Constant.h`, `llvm/IR/Constants.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/GCStrategy.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Argument.h`, `llvm/IR/AttributeMask.h`, `llvm/IR/Attributes.h`, `llvm/IR/BasicBlock.h`, `llvm/IR/CallingConv.h`, `llvm/IR/Constant.h`, `llvm/IR/Constants.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/GCStrategy.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Compiler.h`, `llvm/Support/Debug.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/DenseSet.h`, `llvm/ADT/MapVector.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Compiler.h`, `llvm/Support/Debug.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/DenseSet.h`, `llvm/ADT/MapVector.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `cassert`, `cstddef`, `cstdint`, `iterator`, `optional`, `set`, `string`, `utility` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`cassert`, `cstddef`, `cstdint`, `iterator`, `optional`, `set`, `string`, `utility` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `DataLayout`, `DominatorTree`, `DomTreeUpdater`, `TargetLibraryInfo`, `TargetTransformInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DataLayout`, `DominatorTree`, `DomTreeUpdater`, `TargetLibraryInfo`, `TargetTransformInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
