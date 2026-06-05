# LoadStoreVectorizer.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Vectorize/LoadStoreVectorizer.cpp` | `llvm/lib/Transforms/Vectorize/LoadStoreVectorizer.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements gPU Load & Store Vectorizer within LLVM's vectorization and VPlan infrastructure layer. | 该源文件实现了 LLVM 向量化与 VPlan 基础设施层中的 LoadStoreVectorizer 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-145

```cpp
//===- LoadStoreVectorizer.cpp - GPU Load & Store Vectorizer --------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass merges loads/stores to/from sequential memory addresses into vector
// loads/stores.  Although there's nothing GPU-specific in here, this pass is
// motivated by the microarchitectural quirks of nVidia and AMD GPUs.
//
// (For simplicity below we talk about loads only, but everything also applies
// to stores.)
//
// This pass is intended to be run late in the pipeline, after other
// vectorization opportunities have been exploited.  So the assumption here is
// that immediately following our new vector load we'll need to extract out the
// individual elements of the load, so we can operate on them individually.
//
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
using EqClassKey =
    std::tuple<const Value * /* result of getUnderlyingObject() */,
               unsigned /* AddrSpace */,
               unsigned /* Load/Store element size bits */,
               char /* IsLoad; char b/c bool can't be a DenseMap key */
               >;
[[maybe_unused]] llvm::raw_ostream &operator<<(llvm::raw_ostream &OS,
                                               const EqClassKey &K) {
  const auto &[UnderlyingObject, AddrSpace, ElementSize, IsLoad] = K;
  return OS << (IsLoad ? "load" : "store") << " of " << *UnderlyingObject
            << " of element size " << ElementSize << " bits in addrspace "
            << AddrSpace;
}

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 146-274

```cpp
// A Chain is a set of instructions such that:
//  - All instructions have the same equivalence class, so in particular all are
//    loads, or all are stores.
//  - We know the address accessed by the i'th chain elem relative to the
//    chain's leader instruction, which is the first instr of the chain in BB
//    order.
//
// Chains have two canonical orderings:
//  - BB order, sorted by Instr->comesBefore.
//  - Offset order, sorted by OffsetFromLeader.
// This pass switches back and forth between these orders.
struct ChainElem {
  Instruction *Inst;
  APInt OffsetFromLeader;
  ChainElem(Instruction *Inst, APInt OffsetFromLeader)
      : Inst(std::move(Inst)), OffsetFromLeader(std::move(OffsetFromLeader)) {}
};
using Chain = SmallVector<ChainElem, 1>;

void sortChainInBBOrder(Chain &C) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
             DominatorTree &DT, ScalarEvolution &SE, TargetTransformInfo &TTI)
      : F(F), AA(AA), AC(AC), DT(DT), SE(SE), TTI(TTI),
        DL(F.getDataLayout()), Builder(SE.getContext()) {}

  bool run();

private:
  static const unsigned MaxDepth = 3;

  /// Runs the vectorizer on a "pseudo basic block", which is a range of
  /// instructions [Begin, End) within one BB all of which have
  /// isGuaranteedToTransferExecutionToSuccessor(I) == true.
  bool runOnPseudoBB(BasicBlock::iterator Begin, BasicBlock::iterator End);

```
- EN: Core entities appearing here include ChainElem, sortChainInBBOrder, sortChainInOffsetOrder, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AliasAnalysis, AssumptionCache, DataLayout, DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 ChainElem, sortChainInBBOrder, sortChainInOffsetOrder，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AliasAnalysis, AssumptionCache, DataLayout, DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 275-404

```cpp
  /// Runs the vectorizer on one equivalence class, i.e. one set of loads/stores
  /// in the same BB with the same value for getUnderlyingObject() etc.
  bool runOnEquivalenceClass(const EqClassKey &EqClassKey,
                             ArrayRef<Instruction *> EqClass);

  /// Runs the vectorizer on one chain, i.e. a subset of an equivalence class
  /// where all instructions access a known, constant offset from the first
  /// instruction.
  bool runOnChain(Chain &C);

  /// Splits the chain into subchains of instructions which read/write a
  /// contiguous block of memory. Discards any length-1 subchains (because
  /// there's nothing to vectorize in there). Also attempts to fill gaps with
  /// "extra" elements to artificially make chains contiguous in some cases.
  std::vector<Chain> splitChainByContiguity(Chain &C);

  /// Splits the chain into subchains where it's safe to hoist loads up to the
  /// beginning of the sub-chain and it's safe to sink loads up to the end of
  /// the sub-chain. Discards any length-1 subchains. Also attempts to extend
  /// non-power-of-two chains by adding "extra" elements in some cases.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addRequired<AAResultsWrapperPass>();
    AU.addRequired<AssumptionCacheTracker>();
    AU.addRequired<ScalarEvolutionWrapperPass>();
    AU.addRequired<DominatorTreeWrapperPass>();
    AU.addRequired<TargetTransformInfoWrapperPass>();
    AU.setPreservesCFG();
  }
};

} // end anonymous namespace

char LoadStoreVectorizerLegacyPass::ID = 0;

```
- EN: Core entities appearing here include LoadStoreVectorizerLegacyPass, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, AssumptionCache, DominatorTree, ScalarEvolution. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 LoadStoreVectorizerLegacyPass，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AAResults, AssumptionCache, DominatorTree, ScalarEvolution 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 405-536

```cpp
INITIALIZE_PASS_BEGIN(LoadStoreVectorizerLegacyPass, DEBUG_TYPE,
                      "Vectorize load and Store instructions", false, false)
INITIALIZE_PASS_DEPENDENCY(SCEVAAWrapperPass)
INITIALIZE_PASS_DEPENDENCY(AssumptionCacheTracker);
INITIALIZE_PASS_DEPENDENCY(DominatorTreeWrapperPass)
INITIALIZE_PASS_DEPENDENCY(AAResultsWrapperPass)
INITIALIZE_PASS_DEPENDENCY(GlobalsAAWrapperPass)
INITIALIZE_PASS_DEPENDENCY(TargetTransformInfoWrapperPass)
INITIALIZE_PASS_END(LoadStoreVectorizerLegacyPass, DEBUG_TYPE,
                    "Vectorize load and store instructions", false, false)

Pass *llvm::createLoadStoreVectorizerPass() {
  return new LoadStoreVectorizerLegacyPass();
}

bool LoadStoreVectorizerLegacyPass::runOnFunction(Function &F) {
  // Don't vectorize when the attribute NoImplicitFloat is used.
  if (skipFunction(F) || F.hasFnAttribute(Attribute::NoImplicitFloat))
    return false;

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  return Changed;
}

bool Vectorizer::runOnEquivalenceClass(const EqClassKey &EqClassKey,
                                       ArrayRef<Instruction *> EqClass) {
  bool Changed = false;

  LLVM_DEBUG({
    dbgs() << "LSV: Running on equivalence class of size " << EqClass.size()
           << " keyed on " << EqClassKey << ":\n";
    for (Instruction *I : EqClass)
      dbgs() << "  " << *I << "\n";
  });

```
- EN: Core entities appearing here include runOnFunction, run, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, AAManager, AliasAnalysis, AssumptionCache. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 runOnFunction, run，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AAResults, AAManager, AliasAnalysis, AssumptionCache 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 537-661

```cpp
  std::vector<Chain> Chains = gatherChains(EqClass);
  LLVM_DEBUG(dbgs() << "LSV: Got " << Chains.size()
                    << " nontrivial chains.\n";);
  for (Chain &C : Chains)
    Changed |= runOnChain(C);
  return Changed;
}

bool Vectorizer::runOnChain(Chain &C) {
  LLVM_DEBUG({
    dbgs() << "LSV: Running on chain with " << C.size() << " instructions:\n";
    dumpChain(C);
  });

  // Split up the chain into increasingly smaller chains, until we can finally
  // vectorize the chains.
  //
  // (Don't be scared by the depth of the loop nest here.  These operations are
  // all at worst O(n lg n) in the number of instructions, and splitting chains
  // doesn't change the number of instrs.  So the whole loop nest is O(n lg n).)
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  return Impl(/*IsLoad=*/std::bool_constant<false>());
}

std::vector<Chain> Vectorizer::splitChainByContiguity(Chain &C) {
  if (C.empty())
    return {};

  sortChainInOffsetOrder(C);

  LLVM_DEBUG({
    dbgs() << "LSV: splitChainByContiguity considering chain:\n";
    dumpChain(C);
  });

```
- EN: Core entities appearing here include runOnChain, splitChainByMayAliasInstrs, splitChainByContiguity, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 runOnChain, splitChainByMayAliasInstrs, splitChainByContiguity，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AAResults 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 662-793

```cpp
  // If the chain is not contiguous, we try to fill the gap with "extra"
  // elements to artificially make it contiguous, to try to enable
  // vectorization. We only fill gaps if there is potential to end up with a
  // legal masked load/store given the target, address space, and element type.
  // At this point, when querying the TTI, optimistically assume max alignment
  // and max vector size, as splitChainByAlignment will ensure the final vector
  // shape passes the legalization check.
  unsigned AS = getLoadStoreAddressSpace(C[0].Inst);
  Type *ElementType = getLoadStoreType(C[0].Inst)->getScalarType();
  unsigned MaxVecRegBits = TTI.getLoadStoreVecRegBitWidth(AS);
  Align OptimisticAlign = Align(MaxVecRegBits / 8);
  unsigned int MaxVectorNumElems =
      MaxVecRegBits / DL.getTypeSizeInBits(ElementType);
  // Note: This check decides whether to try to fill gaps based on the masked
  // legality of the target's maximum vector size (getLoadStoreVecRegBitWidth).
  // If a target *does not* support a masked load/store with this max vector
  // size, but *does* support a masked load/store with a *smaller* vector size,
  // that optimization will be missed. This does not occur in any of the targets
  // that currently support this API.
  FixedVectorType *OptimisticVectorType =
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    else
      Ret.push_back({*It});
    // In certain cases when handling redundant elements with partial overlaps,
    // the previous element may still extend beyond the current element. Only
    // update Prev if the current element is the new end of the chain.
    if (ReadEnd.sge(PrevReadEnd))
      Prev = *It;
  }

  // Filter out length-1 chains, these are uninteresting.
  llvm::erase_if(Ret, [](const auto &Chain) { return Chain.size() <= 1; });
  return Ret;
}

```
- EN: This region continues the LoadStoreVectorizer implementation with local helper logic centered on TTI, Inst, Type, ElementType.
- CN: 这一段延续了 LoadStoreVectorizer 的主体实现，围绕 TTI, Inst, Type, ElementType 等局部辅助逻辑展开。

### Lines 794-921

```cpp
Type *Vectorizer::getChainElemTy(const Chain &C) {
  assert(!C.empty());
  // The rules are:
  //  - If there are any pointer types in the chain, use an integer type.
  //  - Prefer an integer type if it appears in the chain.
  //  - Otherwise, use the first type in the chain.
  //
  // The rule about pointer types is a simplification when we merge e.g.  a load
  // of a ptr and a double.  There's no direct conversion from a ptr to a
  // double; it requires a ptrtoint followed by a bitcast.
  //
  // It's unclear to me if the other rules have any practical effect, but we do
  // it to match this pass's previous behavior.
  if (any_of(C, [](const ChainElem &E) {
        return getLoadStoreType(E.Inst)->getScalarType()->isPointerTy();
      })) {
    return Type::getIntNTy(
        F.getContext(),
        DL.getTypeSizeInBits(getLoadStoreType(C[0].Inst)->getScalarType()));
  }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      unsigned VF = 8 * VecRegBytes / VecElemBits;

      // Check that TTI is happy with this vectorization factor.
      unsigned TargetVF = GetVectorFactor(VF, VecElemBits,
                                          VecElemBits * NumVecElems / 8, VecTy);
      if (TargetVF != VF && TargetVF < NumVecElems) {
        LLVM_DEBUG(
            dbgs() << "LSV: splitChainByAlignment discarding candidate chain "
                      "because TargetVF="
                   << TargetVF << " != VF=" << VF
                   << " and TargetVF < NumVecElems=" << NumVecElems << "\n");
        continue;
      }

```
- EN: Core entities appearing here include splitChainByAlignment, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 splitChainByAlignment，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 922-1064

```cpp
      // If we're loading/storing from an alloca, align it if possible.
      //
      // FIXME: We eagerly upgrade the alignment, regardless of whether TTI
      // tells us this is beneficial.  This feels a bit odd, but it matches
      // existing tests.  This isn't *so* bad, because at most we align to 4
      // bytes (current value of StackAdjustedAlignment).
      //
      // FIXME: We will upgrade the alignment of the alloca even if it turns out
      // we can't vectorize for some other reason.
      Value *PtrOperand = getLoadStorePointerOperand(C[CBegin].Inst);
      bool IsAllocaAccess = AS == DL.getAllocaAddrSpace() &&
                            isa<AllocaInst>(PtrOperand->stripPointerCasts());
      Align Alignment = getLoadStoreAlignment(C[CBegin].Inst);
      Align PrefAlign = Align(StackAdjustedAlignment);
      if (IsAllocaAccess && Alignment.value() % SizeBytes != 0 &&
          accessIsAllowedAndFast(SizeBytes, AS, PrefAlign, VecElemBits)) {
        Align NewAlign = getOrEnforceKnownAlignment(
            PtrOperand, PrefAlign, DL, C[CBegin].Inst, nullptr, &DT);
        if (NewAlign >= Alignment) {
          LLVM_DEBUG(dbgs()
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
            (IsLoadChain ? !TTI.isLegalMaskedLoad(
                               FixedVectorType::get(VecElemTy, NumVecElems),
                               Alignment, AS, TTI::MaskKind::ConstantMask)
                         : !TTI.isLegalMaskedStore(
                               FixedVectorType::get(VecElemTy, NumVecElems),
                               Alignment, AS, TTI::MaskKind::ConstantMask))) {
          LLVM_DEBUG(dbgs()
                     << "LSV: splitChainByAlignment discarding candidate chain "
                        "because it contains extra loads/stores that we cannot "
                        "legally vectorize into a masked load/store \n");
          continue;
        }
      }

```
- EN: Core entities appearing here include accessIsAllowedAndFast, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 accessIsAllowedAndFast，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1065-1211

```cpp
      // Hooray, we can vectorize this chain!
      Chain &NewChain = Ret.emplace_back();
      for (unsigned I = CBegin; I <= CEnd; ++I)
        NewChain.emplace_back(C[I]);
      for (ChainElem E : ExtendingLoadsStores)
        NewChain.emplace_back(E);
      CBegin = CEnd; // Skip over the instructions we've added to the chain.
      break;
    }
  }
  return Ret;
}

bool Vectorizer::vectorizeChain(Chain &C) {
  if (C.size() < 2)
    return false;

  bool ChainContainsExtraLoadsStores = llvm::any_of(
      C, [this](const ChainElem &E) { return ExtraElements.contains(E.Inst); });

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    //   ptr1  = gep a, 1
    //   loadv = load <2 x i32> ptr0
    //   load0 = extractelement loadv, 0
    //   load1 = extractelement loadv, 1
    //   ptr0 = gep a, 0
    //
    // Notice that loadv uses ptr0, which is defined *after* it!
    reorder(VecInst);
  } else {
    // Stores get sunk to the location of the last store in the chain.
    Builder.SetInsertPoint(llvm::max_element(C, [](auto &A, auto &B) {
                             return A.Inst->comesBefore(B.Inst);
                           })->Inst);

```
- EN: Core entities appearing here include vectorizeChain, min_element, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 vectorizeChain, min_element，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1212-1341

```cpp
    // Build the vector to store.
    Value *Vec = PoisonValue::get(VecTy);
    auto InsertElem = [&](Value *V, unsigned VecIdx) {
      if (V->getType() != VecElemTy)
        V = Builder.CreateBitOrPointerCast(V, VecElemTy);
      Vec = Builder.CreateInsertElement(Vec, V, Builder.getInt32(VecIdx));
    };
    for (const ChainElem &E : C) {
      auto *I = cast<StoreInst>(E.Inst);
      unsigned EOffset =
          (E.OffsetFromLeader - C[0].OffsetFromLeader).getZExtValue();
      unsigned VecIdx = 8 * EOffset / DL.getTypeSizeInBits(VecElemTy);
      if (FixedVectorType *VT =
              dyn_cast<FixedVectorType>(getLoadStoreType(I))) {
        for (int J = 0, JE = VT->getNumElements(); J < JE; ++J) {
          InsertElem(Builder.CreateExtractElement(I->getValueOperand(),
                                                  Builder.getInt32(J)),
                     VecIdx++);
        }
      } else {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
           (IOffset + IElemSize).sgt(ChainElemOffset)) ||
          (ChainElemOffset.sle(IOffset) &&
           (ChainElemOffset + ChainElemSize).sgt(OffsetIt->second))) {
        LLVM_DEBUG({
          // Double check that AA also sees this alias.  If not, we probably
          // have a bug.
          ModRefInfo MR =
              BatchAA.getModRefInfo(I, MemoryLocation::get(ChainElem));
          assert(IsLoadChain ? isModSet(MR) : isModOrRefSet(MR));
          dbgs() << "LSV: Found alias in chain: " << *I << "\n";
        });
        return false; // We found an aliasing instruction; bail.
      }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AAResults 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1342-1481

```cpp
      continue; // We're confident there's no alias.
    }

    LLVM_DEBUG(dbgs() << "LSV: Querying AA for " << *I << "\n");
    ModRefInfo MR = BatchAA.getModRefInfo(I, MemoryLocation::get(ChainElem));
    if (IsLoadChain ? isModSet(MR) : isModOrRefSet(MR)) {
      LLVM_DEBUG(dbgs() << "LSV: Found alias in chain:\n"
                        << "  Aliasing instruction:\n"
                        << "    " << *I << '\n'
                        << "  Aliased instruction and pointer:\n"
                        << "    " << *ChainElem << '\n'
                        << "    " << *getLoadStorePointerOperand(ChainElem)
                        << '\n');

      return false;
    }
  }
  return true;
}

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // index.
  if (GEPA->getNumOperands() != GEPB->getNumOperands() ||
      GEPA->getPointerOperand() != GEPB->getPointerOperand() ||
      GEPA->getSourceElementType() != GEPB->getSourceElementType())
    return std::nullopt;
  gep_type_iterator GTIA = gep_type_begin(GEPA);
  gep_type_iterator GTIB = gep_type_begin(GEPB);
  for (unsigned I = 0, E = GEPA->getNumIndices() - 1; I < E; ++I) {
    if (GTIA.getOperand() != GTIB.getOperand())
      return std::nullopt;
    ++GTIA;
    ++GTIB;
  }

```
- EN: Core entities appearing here include checkNoWrapFlags, isAddLike, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 checkNoWrapFlags, isAddLike，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 1482-1620

```cpp
  Instruction *OpA = dyn_cast<Instruction>(GTIA.getOperand());
  Instruction *OpB = dyn_cast<Instruction>(GTIB.getOperand());
  if (!OpA || !OpB || OpA->getOpcode() != OpB->getOpcode() ||
      OpA->getType() != OpB->getType())
    return std::nullopt;

  uint64_t Stride = GTIA.getSequentialElementStride(DL);

  // Only look through a ZExt/SExt.
  if (!isa<SExtInst>(OpA) && !isa<ZExtInst>(OpA))
    return std::nullopt;

  bool Signed = isa<SExtInst>(OpA);

  // At this point A could be a function parameter, i.e. not an instruction
  Value *ValA = OpA->getOperand(0);
  OpB = dyn_cast<Instruction>(OpB->getOperand(0));
  if (!OpB || ValA->getType() != OpB->getType())
    return std::nullopt;

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    return;

  // The reduced key has all elements of the ECClassKey except the underlying
  // object. Check that EqClassKey has 4 elements and define the reduced key.
  static_assert(std::tuple_size_v<EqClassKey> == 4,
                "EqClassKey has changed - EqClassReducedKey needs changes too");
  using EqClassReducedKey =
      std::tuple<std::tuple_element_t<1, EqClassKey> /* AddrSpace */,
                 std::tuple_element_t<2, EqClassKey> /* Element size */,
                 std::tuple_element_t<3, EqClassKey> /* IsLoad; */>;
  using ECReducedKeyToUnderlyingObjectMap =
      MapVector<EqClassReducedKey,
                SmallPtrSet<std::tuple_element_t<0, EqClassKey>, 4>>;

```
- EN: Core entities appearing here include checkNoWrapFlags, mergeEquivalenceClasses, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV.
- CN: 此处出现的核心实体包括 checkNoWrapFlags, mergeEquivalenceClasses，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。

### Lines 1621-1750

```cpp
  // Form a map from the reduced key (without the underlying object) to the
  // underlying objects: 1 reduced key to many underlying objects, to form
  // groups of potentially merge-able equivalence classes.
  ECReducedKeyToUnderlyingObjectMap RedKeyToUOMap;
  bool FoundPotentiallyOptimizableEC = false;
  for (const auto &EC : EQClasses) {
    const auto &Key = EC.first;
    EqClassReducedKey RedKey{std::get<1>(Key), std::get<2>(Key),
                             std::get<3>(Key)};
    auto &UOMap = RedKeyToUOMap[RedKey];
    UOMap.insert(std::get<0>(Key));
    if (UOMap.size() > 1)
      FoundPotentiallyOptimizableEC = true;
  }
  if (!FoundPotentiallyOptimizableEC)
    return;

  LLVM_DEBUG({
    dbgs() << "LSV: mergeEquivalenceClasses: before merging:\n";
    for (const auto &EC : EQClasses) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  for (Instruction &I : make_range(Begin, End)) {
    auto *LI = dyn_cast<LoadInst>(&I);
    auto *SI = dyn_cast<StoreInst>(&I);
    if (!LI && !SI)
      continue;

    if ((LI && !LI->isSimple()) || (SI && !SI->isSimple()))
      continue;

    if ((LI && !TTI.isLegalToVectorizeLoad(LI)) ||
        (SI && !TTI.isLegalToVectorizeStore(SI)))
      continue;

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1751-1884

```cpp
    Type *Ty = getLoadStoreType(&I);
    if (!VectorType::isValidElementType(Ty->getScalarType()))
      continue;

    // Skip weird non-byte sizes. They probably aren't worth the effort of
    // handling correctly.
    unsigned TySize = DL.getTypeSizeInBits(Ty);
    if ((TySize % 8) != 0)
      continue;

    // Skip vectors of pointers. The vectorizeLoadChain/vectorizeStoreChain
    // functions are currently using an integer type for the vectorized
    // load/store, and does not support casting between the integer type and a
    // vector of pointers (e.g. i64 to <2 x i16*>)
    if (Ty->isVectorTy() && Ty->isPtrOrPtrVectorTy())
      continue;

    Value *Ptr = getLoadStorePointerOperand(&I);
    unsigned AS = Ptr->getType()->getPointerAddressSpace();
    unsigned VecRegSize = TTI.getLoadStoreVecRegBitWidth(AS);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      MRU.push_front(*E);
      Chains.insert(E);
    }
  }

  std::vector<Chain> Ret;
  Ret.reserve(Chains.size());
  // Iterate over MRU rather than Chains so the order is deterministic.
  for (auto &E : MRU)
    if (E.second.size() > 1)
      Ret.emplace_back(std::move(E.second));
  return Ret;
}

```
- EN: Core entities appearing here include gatherChains, InstrListElem, InstrListElemDenseMapInfo, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 gatherChains, InstrListElem, InstrListElemDenseMapInfo，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1885-2023

```cpp
std::optional<APInt> Vectorizer::getConstantOffset(Value *PtrA, Value *PtrB,
                                                   Instruction *ContextInst,
                                                   unsigned Depth) {
  LLVM_DEBUG(dbgs() << "LSV: getConstantOffset, PtrA=" << *PtrA
                    << ", PtrB=" << *PtrB << ", ContextInst= " << *ContextInst
                    << ", Depth=" << Depth << "\n");
  // We'll ultimately return a value of this bit width, even if computations
  // happen in a different width.
  unsigned OrigBitWidth = DL.getIndexTypeSizeInBits(PtrA->getType());
  APInt OffsetA(OrigBitWidth, 0);
  APInt OffsetB(OrigBitWidth, 0);
  PtrA = PtrA->stripAndAccumulateInBoundsConstantOffsets(DL, OffsetA);
  PtrB = PtrB->stripAndAccumulateInBoundsConstantOffsets(DL, OffsetB);
  unsigned NewPtrBitWidth = DL.getTypeStoreSizeInBits(PtrA->getType());
  if (NewPtrBitWidth != DL.getTypeStoreSizeInBits(PtrB->getType()))
    return std::nullopt;

  // If we have to shrink the pointer, stripAndAccumulateInBoundsConstantOffsets
  // should properly handle a possible overflow and the value should fit into
  // the smallest data type used in the cast/gep chain.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    unsigned EOffset =
        (E.OffsetFromLeader - C[0].OffsetFromLeader).getZExtValue();
    unsigned VecIdx =
        8 * EOffset / DL.getTypeSizeInBits(VecTy->getScalarType());
    if (FixedVectorType *VT =
            dyn_cast<FixedVectorType>(getLoadStoreType(E.Inst)))
      for (unsigned J = 0; J < VT->getNumElements(); ++J)
        MaskElts[VecIdx + J] = Builder.getInt1(true);
    else
      MaskElts[VecIdx] = Builder.getInt1(true);
  }
  return ConstantVector::get(MaskElts);
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2024-2041

```cpp
void Vectorizer::deleteExtraElements() {
  for (auto *ExtraElement : ExtraElements) {
    if (isa<LoadInst>(ExtraElement)) {
      [[maybe_unused]] bool Deleted =
          RecursivelyDeleteTriviallyDeadInstructions(ExtraElement);
      assert(Deleted && "Extra Load should always be trivially dead");
    } else {
      // Unlike Extra Loads, Extra Stores won't be "dead", but should all be
      // deleted regardless. They will have either been combined into a masked
      // store, or will be left behind and need to be cleaned up.
      auto *PtrOperand = getLoadStorePointerOperand(ExtraElement);
      ExtraElement->eraseFromParent();
      RecursivelyDeleteTriviallyDeadInstructions(PtrOperand);
    }
  }

  ExtraElements.clear();
}
```
- EN: Core entities appearing here include deleteExtraElements, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 deleteExtraElements，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Vectorize` focuses on vectorization and VPlan infrastructure.
  - CN: 领域：`Vectorize` 主要处理 向量化与 VPlan 基础设施。
- EN: Primary entities: `ChainElem, Vectorizer, LoadStoreVectorizerLegacyPass, InstrListElem, InstrListElemDenseMapInfo, sortChainInBBOrder, sortChainInOffsetOrder, sort` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`ChainElem, Vectorizer, LoadStoreVectorizerLegacyPass, InstrListElem, InstrListElemDenseMapInfo, sortChainInBBOrder, sortChainInOffsetOrder, sort` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AAResults, AAManager, AliasAnalysis, AssumptionCache, DataLayout, DominatorTree, ScalarEvolution, SCEV` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AAResults, AAManager, AliasAnalysis, AssumptionCache, DataLayout, DominatorTree, ScalarEvolution, SCEV` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/AliasAnalysis.h`, `llvm/Analysis/AssumptionCache.h`, `llvm/Analysis/MemoryLocation.h`, `llvm/Analysis/ScalarEvolution.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/Analysis/ValueTracking.h`, `llvm/Analysis/VectorUtils.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/AliasAnalysis.h`, `llvm/Analysis/AssumptionCache.h`, `llvm/Analysis/MemoryLocation.h`, `llvm/Analysis/ScalarEvolution.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/Analysis/ValueTracking.h`, `llvm/Analysis/VectorUtils.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Attributes.h`, `llvm/IR/BasicBlock.h`, `llvm/IR/ConstantRange.h`, `llvm/IR/Constants.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/GetElementPtrTypeIterator.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Attributes.h`, `llvm/IR/BasicBlock.h`, `llvm/IR/ConstantRange.h`, `llvm/IR/Constants.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/GetElementPtrTypeIterator.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Alignment.h`, `llvm/Support/Casting.h`, `llvm/Support/Debug.h`, `llvm/Support/KnownBits.h`, `llvm/Support/MathExtras.h`, `llvm/Support/ModRef.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/APInt.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/DenseMap.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Alignment.h`, `llvm/Support/Casting.h`, `llvm/Support/Debug.h`, `llvm/Support/KnownBits.h`, `llvm/Support/MathExtras.h`, `llvm/Support/ModRef.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/APInt.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/DenseMap.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `algorithm`, `cassert`, `cstdint`, `cstdlib`, `iterator`, `numeric`, `optional`, `tuple` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`algorithm`, `cassert`, `cstdint`, `cstdlib`, `iterator`, `numeric`, `optional`, `tuple` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `AAResults`, `AAManager`, `AliasAnalysis`, `AssumptionCache`, `DataLayout`, `DominatorTree`, `ScalarEvolution`, `SCEV` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AAResults`, `AAManager`, `AliasAnalysis`, `AssumptionCache`, `DataLayout`, `DominatorTree`, `ScalarEvolution`, `SCEV` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
