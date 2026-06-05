# AssumeBundleBuilder.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/AssumeBundleBuilder.cpp` | `llvm/lib/Transforms/Utils/AssumeBundleBuilder.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements tools to preserve informations within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 AssumeBundleBuilder 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-52

```cpp
//===- AssumeBundleBuilder.cpp - tools to preserve informations -*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/AssumeBundleBuilder.h"
#include "llvm/ADT/DepthFirstIterator.h"
#include "llvm/ADT/MapVector.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/Analysis/AssumeBundleQueries.h"
#include "llvm/Analysis/AssumptionCache.h"
#include "llvm/Analysis/ValueTracking.h"
#include "llvm/IR/Dominators.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/InstIterator.h"
#include "llvm/IR/IntrinsicInst.h"
#include "llvm/IR/Module.h"
#include "llvm/IR/Operator.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Compiler.h"
#include "llvm/Support/DebugCounter.h"
#include "llvm/Transforms/Utils/Local.h"

using namespace llvm;

namespace llvm {
LLVM_ABI cl::opt<bool> ShouldPreserveAllAttributes(
    "assume-preserve-all", cl::init(false), cl::Hidden,
    cl::desc("enable preservation of all attributes. even those that are "
             "unlikely to be useful"));

cl::opt<bool> EnableKnowledgeRetention(
    "enable-knowledge-retention", cl::init(false), cl::Hidden,
    cl::desc(
        "enable preservation of attributes throughout code transformation"));
} // namespace llvm

#define DEBUG_TYPE "assume-builder"

STATISTIC(NumAssumeBuilt, "Number of assume built by the assume builder");
STATISTIC(NumBundlesInAssumes, "Total number of Bundles in the assume built");
STATISTIC(NumAssumesMerged,
          "Number of assume merged by the assume simplify pass");
STATISTIC(NumAssumesRemoved,
          "Number of assume removed by the assume simplify pass");

DEBUG_COUNTER(BuildAssumeCounter, "assume-builder-counter",
              "Controls which assumes gets created");

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 53-106

```cpp
namespace {

bool isUsefullToPreserve(Attribute::AttrKind Kind) {
  switch (Kind) {
    case Attribute::NonNull:
    case Attribute::NoUndef:
    case Attribute::Alignment:
    case Attribute::Dereferenceable:
    case Attribute::DereferenceableOrNull:
    case Attribute::Cold:
      return true;
    default:
      return false;
  }
}

/// This function will try to transform the given knowledge into a more
/// canonical one. the canonical knowledge maybe the given one.
RetainedKnowledge canonicalizedKnowledge(RetainedKnowledge RK,
                                         const DataLayout &DL) {
  switch (RK.AttrKind) {
  default:
    return RK;
  case Attribute::NonNull:
    RK.WasOn = RK.WasOn->stripInBoundsOffsets();
    return RK;
  case Attribute::Alignment: {
    Value *V = RK.WasOn->stripInBoundsOffsets([&](const Value *Strip) {
      if (auto *GEP = dyn_cast<GEPOperator>(Strip))
        RK.ArgValue =
            MinAlign(RK.ArgValue, GEP->getMaxPreservedAlignment(DL).value());
    });
    RK.WasOn = V;
    return RK;
  }
  case Attribute::Dereferenceable:
  case Attribute::DereferenceableOrNull: {
    int64_t Offset = 0;
    Value *V = GetPointerBaseWithConstantOffset(RK.WasOn, Offset, DL,
                                                /*AllowNonInBounds*/ false);
    if (Offset < 0)
      return RK;
    RK.ArgValue = RK.ArgValue + Offset;
    RK.WasOn = V;
  }
  }
  return RK;
}

/// This class contain all knowledge that have been gather while building an
/// llvm.assume and the function to manipulate it.
struct AssumeBuilderState {
  Module *M;

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include isUsefullToPreserve, AssumeBuilderState, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 isUsefullToPreserve, AssumeBuilderState，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。

### Lines 107-155

```cpp
  using MapKey = std::pair<Value *, Attribute::AttrKind>;
  SmallMapVector<MapKey, uint64_t, 8> AssumedKnowledgeMap;
  Instruction *InstBeingModified = nullptr;
  AssumptionCache* AC = nullptr;
  DominatorTree* DT = nullptr;

  AssumeBuilderState(Module *M, Instruction *I = nullptr,
                     AssumptionCache *AC = nullptr, DominatorTree *DT = nullptr)
      : M(M), InstBeingModified(I), AC(AC), DT(DT) {}

  bool tryToPreserveWithoutAddingAssume(RetainedKnowledge RK) {
    if (!InstBeingModified || !RK.WasOn || !AC)
      return false;
    bool HasBeenPreserved = false;
    Use* ToUpdate = nullptr;
    getKnowledgeForValue(
        RK.WasOn, {RK.AttrKind}, *AC,
        [&](RetainedKnowledge RKOther, Instruction *Assume,
            const CallInst::BundleOpInfo *Bundle) {
          if (!isValidAssumeForContext(Assume, InstBeingModified, DT))
            return false;
          if (RKOther.ArgValue >= RK.ArgValue) {
            HasBeenPreserved = true;
            return true;
          } else if (isValidAssumeForContext(InstBeingModified, Assume, DT)) {
            HasBeenPreserved = true;
            IntrinsicInst *Intr = cast<IntrinsicInst>(Assume);
            ToUpdate = &Intr->op_begin()[Bundle->Begin + ABA_Argument];
            return true;
          }
          return false;
        });
    if (ToUpdate)
      ToUpdate->set(
          ConstantInt::get(Type::getInt64Ty(M->getContext()), RK.ArgValue));
    return HasBeenPreserved;
  }

  bool isKnowledgeWorthPreserving(RetainedKnowledge RK) {
    if (!RK)
      return false;
    if (!RK.WasOn)
      return true;
    if (RK.WasOn->getType()->isPointerTy()) {
      Value *UnderlyingPtr = getUnderlyingObject(RK.WasOn);
      if (isa<AllocaInst>(UnderlyingPtr) || isa<GlobalValue>(UnderlyingPtr))
        return false;
    }
    if (auto *Arg = dyn_cast<Argument>(RK.WasOn)) {
```
- EN: Core entities appearing here include tryToPreserveWithoutAddingAssume, isKnowledgeWorthPreserving, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 tryToPreserveWithoutAddingAssume, isKnowledgeWorthPreserving，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 156-204

```cpp
      if (Arg->hasAttribute(RK.AttrKind) &&
          (!Attribute::isIntAttrKind(RK.AttrKind) ||
           Arg->getAttribute(RK.AttrKind).getValueAsInt() >= RK.ArgValue))
        return false;
      return true;
    }
    if (auto *Inst = dyn_cast<Instruction>(RK.WasOn))
      if (wouldInstructionBeTriviallyDead(Inst)) {
        if (RK.WasOn->use_empty())
          return false;
        Use *SingleUse = RK.WasOn->getSingleUndroppableUse();
        if (SingleUse && SingleUse->getUser() == InstBeingModified)
          return false;
      }
    return true;
  }

  void addKnowledge(RetainedKnowledge RK) {
    RK = canonicalizedKnowledge(RK, M->getDataLayout());

    if (!isKnowledgeWorthPreserving(RK))
      return;

    if (tryToPreserveWithoutAddingAssume(RK))
      return;
    MapKey Key{RK.WasOn, RK.AttrKind};
    auto [Lookup, Inserted] = AssumedKnowledgeMap.try_emplace(Key, RK.ArgValue);
    if (Inserted)
      return;
    assert(((Lookup->second == 0 && RK.ArgValue == 0) ||
            (Lookup->second != 0 && RK.ArgValue != 0)) &&
           "inconsistent argument value");

    /// This is only desirable because for all attributes taking an argument
    /// higher is better.
    Lookup->second = std::max(Lookup->second, RK.ArgValue);
  }

  void addAttribute(Attribute Attr, Value *WasOn) {
    if (Attr.isTypeAttribute() || Attr.isStringAttribute() ||
        (!ShouldPreserveAllAttributes &&
         !isUsefullToPreserve(Attr.getKindAsEnum())))
      return;
    uint64_t AttrArg = 0;
    if (Attr.isIntAttribute())
      AttrArg = Attr.getValueAsInt();
    addKnowledge({Attr.getKindAsEnum(), AttrArg, WasOn});
  }

```
- EN: Core entities appearing here include addKnowledge, addAttribute, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout.
- CN: 此处出现的核心实体包括 addKnowledge, addAttribute，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。

### Lines 205-250

```cpp
  void addCall(const CallBase *Call) {
    auto addAttrList = [&](AttributeList AttrList, unsigned NumArgs) {
      for (unsigned Idx = 0; Idx < NumArgs; Idx++)
        for (Attribute Attr : AttrList.getParamAttrs(Idx)) {
          bool IsPoisonAttr = Attr.hasAttribute(Attribute::NonNull) ||
                              Attr.hasAttribute(Attribute::Alignment);
          if (!IsPoisonAttr || Call->isPassingUndefUB(Idx))
            addAttribute(Attr, Call->getArgOperand(Idx));
        }
      for (Attribute Attr : AttrList.getFnAttrs())
        addAttribute(Attr, nullptr);
    };
    addAttrList(Call->getAttributes(), Call->arg_size());
    if (Function *Fn = Call->getCalledFunction())
      addAttrList(Fn->getAttributes(), Fn->arg_size());
  }

  AssumeInst *build() {
    if (AssumedKnowledgeMap.empty())
      return nullptr;
    if (!DebugCounter::shouldExecute(BuildAssumeCounter))
      return nullptr;
    Function *FnAssume =
        Intrinsic::getOrInsertDeclaration(M, Intrinsic::assume);
    LLVMContext &C = M->getContext();
    SmallVector<OperandBundleDef, 8> OpBundle;
    for (auto &MapElem : AssumedKnowledgeMap) {
      SmallVector<Value *, 2> Args;
      if (MapElem.first.first)
        Args.push_back(MapElem.first.first);

      /// This is only valid because for all attribute that currently exist a
      /// value of 0 is useless. and should not be preserved.
      if (MapElem.second)
        Args.push_back(ConstantInt::get(Type::getInt64Ty(M->getContext()),
                                        MapElem.second));
      OpBundle.push_back(OperandBundleDefT<Value *>(
          std::string(Attribute::getNameFromAttrKind(MapElem.first.second)),
          Args));
      NumBundlesInAssumes++;
    }
    NumAssumeBuilt++;
    return cast<AssumeInst>(CallInst::Create(
        FnAssume, ArrayRef<Value *>({ConstantInt::getTrue(C)}), OpBundle));
  }

```
- EN: Core entities appearing here include addCall, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 addCall，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 251-297

```cpp
  void addAccessedPtr(Instruction *MemInst, Value *Pointer, Type *AccType,
                      MaybeAlign MA) {
    unsigned DerefSize = MemInst->getModule()
                             ->getDataLayout()
                             .getTypeStoreSize(AccType)
                             .getKnownMinValue();
    if (DerefSize != 0) {
      addKnowledge({Attribute::Dereferenceable, DerefSize, Pointer});
      if (!NullPointerIsDefined(MemInst->getFunction(),
                                Pointer->getType()->getPointerAddressSpace()))
        addKnowledge({Attribute::NonNull, 0u, Pointer});
    }
    if (MA.valueOrOne() > 1)
      addKnowledge({Attribute::Alignment, MA.valueOrOne().value(), Pointer});
  }

  void addInstruction(Instruction *I) {
    if (auto *Call = dyn_cast<CallBase>(I))
      return addCall(Call);
    if (auto *Load = dyn_cast<LoadInst>(I))
      return addAccessedPtr(I, Load->getPointerOperand(), Load->getType(),
                            Load->getAlign());
    if (auto *Store = dyn_cast<StoreInst>(I))
      return addAccessedPtr(I, Store->getPointerOperand(),
                            Store->getValueOperand()->getType(),
                            Store->getAlign());
    if (auto *RMW = dyn_cast<AtomicRMWInst>(I))
      return addAccessedPtr(I, RMW->getPointerOperand(),
                            RMW->getValOperand()->getType(), RMW->getAlign());
    if (auto *CmpXchg = dyn_cast<AtomicCmpXchgInst>(I))
      return addAccessedPtr(I, CmpXchg->getPointerOperand(),
                            CmpXchg->getCompareOperand()->getType(),
                            CmpXchg->getAlign());
    // TODO: Maybe we should look around and merge with other llvm.assume.
  }
};

} // namespace

AssumeInst *llvm::buildAssumeFromInst(Instruction *I) {
  if (!EnableKnowledgeRetention)
    return nullptr;
  AssumeBuilderState Builder(I->getModule());
  Builder.addInstruction(I);
  return Builder.build();
}

```
- EN: Core entities appearing here include addInstruction, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 addInstruction，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 298-345

```cpp
bool llvm::salvageKnowledge(Instruction *I, AssumptionCache *AC,
                            DominatorTree *DT) {
  if (!EnableKnowledgeRetention || I->isTerminator())
    return false;
  bool Changed = false;
  AssumeBuilderState Builder(I->getModule(), I, AC, DT);
  Builder.addInstruction(I);
  if (auto *Intr = Builder.build()) {
    Intr->insertBefore(I->getIterator());
    Changed = true;
    if (AC)
      AC->registerAssumption(Intr);
  }
  return Changed;
}

RetainedKnowledge llvm::simplifyRetainedKnowledge(AssumeInst *Assume,
                                                  RetainedKnowledge RK,
                                                  AssumptionCache *AC,
                                                  DominatorTree *DT) {
  AssumeBuilderState Builder(Assume->getModule(), Assume, AC, DT);
  RK = canonicalizedKnowledge(RK, Assume->getDataLayout());

  if (!Builder.isKnowledgeWorthPreserving(RK))
    return RetainedKnowledge::none();

  if (Builder.tryToPreserveWithoutAddingAssume(RK))
    return RetainedKnowledge::none();
  return RK;
}

namespace {

struct AssumeSimplify {
  Function &F;
  AssumptionCache &AC;
  DominatorTree *DT;
  LLVMContext &C;
  SmallDenseSet<IntrinsicInst *> CleanupToDo;
  StringMapEntry<uint32_t> *IgnoreTag;
  SmallDenseMap<BasicBlock *, SmallVector<IntrinsicInst *, 4>, 8> BBToAssume;
  bool MadeChange = false;

  AssumeSimplify(Function &F, AssumptionCache &AC, DominatorTree *DT,
                 LLVMContext &C)
      : F(F), AC(AC), DT(DT), C(C),
        IgnoreTag(C.getOrInsertBundleTag(IgnoreBundleTag)) {}

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include AssumeSimplify, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DataLayout, DominatorTree.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 AssumeSimplify，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DataLayout, DominatorTree 等分析结果。

### Lines 346-394

```cpp
  void buildMapping(bool FilterBooleanArgument) {
    BBToAssume.clear();
    for (Value *V : AC.assumptions()) {
      if (!V)
        continue;
      IntrinsicInst *Assume = cast<IntrinsicInst>(V);
      if (FilterBooleanArgument) {
        auto *Arg = dyn_cast<ConstantInt>(Assume->getOperand(0));
        if (!Arg || Arg->isZero())
          continue;
      }
      BBToAssume[Assume->getParent()].push_back(Assume);
    }

    for (auto &Elem : BBToAssume) {
      llvm::sort(Elem.second,
                 [](const IntrinsicInst *LHS, const IntrinsicInst *RHS) {
                   return LHS->comesBefore(RHS);
                 });
    }
  }

  /// Remove all asumes in CleanupToDo if there boolean argument is true and
  /// ForceCleanup is set or the assume doesn't hold valuable knowledge.
  void RunCleanup(bool ForceCleanup) {
    for (IntrinsicInst *Assume : CleanupToDo) {
      auto *Arg = dyn_cast<ConstantInt>(Assume->getOperand(0));
      if (!Arg || Arg->isZero() ||
          (!ForceCleanup &&
           !isAssumeWithEmptyBundle(cast<AssumeInst>(*Assume))))
        continue;
      MadeChange = true;
      if (ForceCleanup)
        NumAssumesMerged++;
      else
        NumAssumesRemoved++;
      Assume->eraseFromParent();
    }
    CleanupToDo.clear();
  }

  /// Remove knowledge stored in assume when it is already know by an attribute
  /// or an other assume. This can when valid update an existing knowledge in an
  /// attribute or an other assume.
  void dropRedundantKnowledge() {
    struct MapValue {
      IntrinsicInst *Assume;
      uint64_t ArgValue;
      CallInst::BundleOpInfo *BOI;
```
- EN: Core entities appearing here include buildMapping, RunCleanup, dropRedundantKnowledge, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 buildMapping, RunCleanup, dropRedundantKnowledge，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 395-443

```cpp
    };
    buildMapping(false);
    SmallDenseMap<std::pair<Value *, Attribute::AttrKind>,
                  SmallVector<MapValue, 2>, 16>
        Knowledge;
    for (BasicBlock *BB : depth_first(&F))
      for (Value *V : BBToAssume[BB]) {
        if (!V)
          continue;
        IntrinsicInst *Assume = cast<IntrinsicInst>(V);
        for (CallInst::BundleOpInfo &BOI : Assume->bundle_op_infos()) {
          auto RemoveFromAssume = [&]() {
            CleanupToDo.insert(Assume);
            if (BOI.Begin != BOI.End) {
              Use *U = &Assume->op_begin()[BOI.Begin + ABA_WasOn];
              U->set(PoisonValue::get(U->get()->getType()));
            }
            BOI.Tag = IgnoreTag;
          };
          if (BOI.Tag == IgnoreTag) {
            CleanupToDo.insert(Assume);
            continue;
          }
          RetainedKnowledge RK =
            getKnowledgeFromBundle(cast<AssumeInst>(*Assume), BOI);
          if (auto *Arg = dyn_cast_or_null<Argument>(RK.WasOn)) {
            bool HasSameKindAttr = Arg->hasAttribute(RK.AttrKind);
            if (HasSameKindAttr)
              if (!Attribute::isIntAttrKind(RK.AttrKind) ||
                  Arg->getAttribute(RK.AttrKind).getValueAsInt() >=
                      RK.ArgValue) {
                RemoveFromAssume();
                continue;
              }
            if (isValidAssumeForContext(
                    Assume, &*F.getEntryBlock().getFirstInsertionPt()) ||
                Assume == &*F.getEntryBlock().getFirstInsertionPt()) {
              if (HasSameKindAttr)
                Arg->removeAttr(RK.AttrKind);
              Arg->addAttr(Attribute::get(C, RK.AttrKind, RK.ArgValue));
              MadeChange = true;
              RemoveFromAssume();
              continue;
            }
          }
          auto &Lookup = Knowledge[{RK.WasOn, RK.AttrKind}];
          for (MapValue &Elem : Lookup) {
            if (!isValidAssumeForContext(Elem.Assume, Assume, DT))
              continue;
```
- EN: This region continues the AssumeBundleBuilder implementation with local helper logic centered on SmallDenseMap, Value, Attribute, AttrKind.
- CN: 这一段延续了 AssumeBundleBuilder 的主体实现，围绕 SmallDenseMap, Value, Attribute, AttrKind 等局部辅助逻辑展开。

### Lines 444-489

```cpp
            if (Elem.ArgValue >= RK.ArgValue) {
              RemoveFromAssume();
              continue;
            } else if (isValidAssumeForContext(Assume, Elem.Assume, DT)) {
              Elem.Assume->op_begin()[Elem.BOI->Begin + ABA_Argument].set(
                  ConstantInt::get(Type::getInt64Ty(C), RK.ArgValue));
              MadeChange = true;
              RemoveFromAssume();
              continue;
            }
          }
          Lookup.push_back({Assume, RK.ArgValue, &BOI});
        }
      }
  }

  using MergeIterator = SmallVectorImpl<IntrinsicInst *>::iterator;

  /// Merge all Assumes from Begin to End in and insert the resulting assume as
  /// high as possible in the basicblock.
  void mergeRange(BasicBlock *BB, MergeIterator Begin, MergeIterator End) {
    if (Begin == End || std::next(Begin) == End)
      return;
    /// Provide no additional information so that AssumeBuilderState doesn't
    /// try to do any punning since it already has been done better.
    AssumeBuilderState Builder(F.getParent());

    /// For now it is initialized to the best value it could have
    BasicBlock::iterator InsertPt = BB->getFirstNonPHIIt();
    if (isa<LandingPadInst>(InsertPt))
      InsertPt = std::next(InsertPt);
    for (IntrinsicInst *I : make_range(Begin, End)) {
      CleanupToDo.insert(I);
      for (CallInst::BundleOpInfo &BOI : I->bundle_op_infos()) {
        RetainedKnowledge RK =
          getKnowledgeFromBundle(cast<AssumeInst>(*I), BOI);
        if (!RK)
          continue;
        Builder.addKnowledge(RK);
        if (auto *I = dyn_cast_or_null<Instruction>(RK.WasOn))
          if (I->getParent() == InsertPt->getParent() &&
              (InsertPt->comesBefore(I) || &*InsertPt == I))
            InsertPt = I->getNextNode()->getIterator();
      }
    }

```
- EN: Core entities appearing here include mergeRange, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 mergeRange，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 490-539

```cpp
    /// Adjust InsertPt if it is before Begin, since mergeAssumes only
    /// guarantees we can place the resulting assume between Begin and End.
    if (InsertPt->comesBefore(*Begin))
      for (auto It = (*Begin)->getIterator(), E = InsertPt->getIterator();
           It != E; --It)
        if (!isGuaranteedToTransferExecutionToSuccessor(&*It)) {
          InsertPt = std::next(It);
          break;
        }
    auto *MergedAssume = Builder.build();
    if (!MergedAssume)
      return;
    MadeChange = true;
    MergedAssume->insertBefore(InsertPt);
    AC.registerAssumption(MergedAssume);
  }

  /// Merge assume when they are in the same BasicBlock and for all instruction
  /// between them isGuaranteedToTransferExecutionToSuccessor returns true.
  void mergeAssumes() {
    buildMapping(true);

    SmallVector<MergeIterator, 4> SplitPoints;
    for (auto &Elem : BBToAssume) {
      SmallVectorImpl<IntrinsicInst *> &AssumesInBB = Elem.second;
      if (AssumesInBB.size() < 2)
        continue;
      /// AssumesInBB is already sorted by order in the block.

      BasicBlock::iterator It = AssumesInBB.front()->getIterator();
      BasicBlock::iterator E = AssumesInBB.back()->getIterator();
      SplitPoints.push_back(AssumesInBB.begin());
      MergeIterator LastSplit = AssumesInBB.begin();
      for (; It != E; ++It)
        if (!isGuaranteedToTransferExecutionToSuccessor(&*It)) {
          for (; (*LastSplit)->comesBefore(&*It); ++LastSplit)
            ;
          if (SplitPoints.back() != LastSplit)
            SplitPoints.push_back(LastSplit);
        }
      SplitPoints.push_back(AssumesInBB.end());
      for (auto SplitIt = SplitPoints.begin();
           SplitIt != std::prev(SplitPoints.end()); SplitIt++) {
        mergeRange(Elem.first, *SplitIt, *(SplitIt + 1));
      }
      SplitPoints.clear();
    }
  }
};

```
- EN: Core entities appearing here include mergeAssumes, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 mergeAssumes，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 540-584

```cpp
bool simplifyAssumes(Function &F, AssumptionCache *AC, DominatorTree *DT) {
  AssumeSimplify AS(F, *AC, DT, F.getContext());

  /// Remove knowledge that is already known by a dominating other assume or an
  /// attribute.
  AS.dropRedundantKnowledge();

  /// Remove assume that are empty.
  AS.RunCleanup(false);

  /// Merge assume in the same basicblock when possible.
  AS.mergeAssumes();

  /// Remove assume that were merged.
  AS.RunCleanup(true);
  return AS.MadeChange;
}

} // namespace

PreservedAnalyses AssumeSimplifyPass::run(Function &F,
                                          FunctionAnalysisManager &AM) {
  if (!EnableKnowledgeRetention)
    return PreservedAnalyses::all();
  if (!simplifyAssumes(F, &AM.getResult<AssumptionAnalysis>(F),
                       AM.getCachedResult<DominatorTreeAnalysis>(F)))
    return PreservedAnalyses::all();
  PreservedAnalyses PA;
  PA.preserveSet<CFGAnalyses>();
  return PA;
}

PreservedAnalyses AssumeBuilderPass::run(Function &F,
                                         FunctionAnalysisManager &AM) {
  AssumptionCache *AC = &AM.getResult<AssumptionAnalysis>(F);
  DominatorTree* DT = AM.getCachedResult<DominatorTreeAnalysis>(F);
  bool Changed = false;
  for (Instruction &I : instructions(F))
    Changed |= salvageKnowledge(&I, AC, DT);
  if (!Changed)
    PreservedAnalyses::all();
  PreservedAnalyses PA;
  PA.preserveSet<CFGAnalyses>();
  return PA;
}
```
- EN: Core entities appearing here include simplifyAssumes, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 simplifyAssumes，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `AssumeBuilderState, AssumeSimplify, MapValue, isUsefullToPreserve, tryToPreserveWithoutAddingAssume, isKnowledgeWorthPreserving, addKnowledge, addAttribute` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`AssumeBuilderState, AssumeSimplify, MapValue, isUsefullToPreserve, tryToPreserveWithoutAddingAssume, isKnowledgeWorthPreserving, addKnowledge, addAttribute` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AssumptionCache, DataLayout, DominatorTree, ValueTracking` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AssumptionCache, DataLayout, DominatorTree, ValueTracking` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/AssumeBundleQueries.h`, `llvm/Analysis/AssumptionCache.h`, `llvm/Analysis/ValueTracking.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/AssumeBundleQueries.h`, `llvm/Analysis/AssumptionCache.h`, `llvm/Analysis/ValueTracking.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/InstIterator.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/Module.h`, `llvm/IR/Operator.h`, `llvm/Transforms/Utils/AssumeBundleBuilder.h`, `llvm/Transforms/Utils/Local.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/InstIterator.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/Module.h`, `llvm/IR/Operator.h`, `llvm/Transforms/Utils/AssumeBundleBuilder.h`, `llvm/Transforms/Utils/Local.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/CommandLine.h`, `llvm/Support/Compiler.h`, `llvm/Support/DebugCounter.h`, `llvm/ADT/DepthFirstIterator.h`, `llvm/ADT/MapVector.h`, `llvm/ADT/Statistic.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/CommandLine.h`, `llvm/Support/Compiler.h`, `llvm/Support/DebugCounter.h`, `llvm/ADT/DepthFirstIterator.h`, `llvm/ADT/MapVector.h`, `llvm/ADT/Statistic.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Runtime analysis dependencies: `AssumptionCache`, `DataLayout`, `DominatorTree`, `ValueTracking` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AssumptionCache`, `DataLayout`, `DominatorTree`, `ValueTracking` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
