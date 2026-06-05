# MergeICmps.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Scalar/MergeICmps.cpp` | `llvm/lib/Transforms/Scalar/MergeICmps.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements optimize chains of integer comparisons within LLVM's scalar optimizations and loop/CFG transforms layer. | 该源文件实现了 LLVM 标量优化以及循环/CFG 变换层中的 MergeICmps 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-79

```cpp
//===- MergeICmps.cpp - Optimize chains of integer comparisons ------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass turns chains of integer comparisons into memcmp (the memcmp is
// later typically inlined as a chain of efficient hardware comparisons). This
// typically benefits c++ member or nonmember operator==().
//
// The basic idea is to replace a longer chain of integer comparisons loaded
// from contiguous memory locations into a shorter chain of larger integer
// comparisons. Benefits are double:
//  - There are less jumps, and therefore less opportunities for mispredictions
//    and I-cache misses.
//  - Code size is smaller, both because jumps are removed and because the
//    encoding of a 2*n byte compare is smaller than that of two n-byte
//    compares.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

namespace llvm {
extern cl::opt<bool> ProfcheckDisableMetadataFixes;
} // namespace llvm
namespace {

// A BCE atom "Binary Compare Expression Atom" represents an integer load
// that is a constant offset from a base value, e.g. `a` or `o.c` in the example
// at the top.
struct BCEAtom {
  BCEAtom() = default;
  BCEAtom(GetElementPtrInst *GEP, LoadInst *LoadI, int BaseId, APInt Offset)
      : GEP(GEP), LoadI(LoadI), BaseId(BaseId), Offset(std::move(Offset)) {}

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 80-163

```cpp
  BCEAtom(const BCEAtom &) = delete;
  BCEAtom &operator=(const BCEAtom &) = delete;

  BCEAtom(BCEAtom &&that) = default;
  BCEAtom &operator=(BCEAtom &&that) {
    if (this == &that)
      return *this;
    GEP = that.GEP;
    LoadI = that.LoadI;
    BaseId = that.BaseId;
    Offset = std::move(that.Offset);
    return *this;
  }

  // We want to order BCEAtoms by (Base, Offset). However we cannot use
  // the pointer values for Base because these are non-deterministic.
  // To make sure that the sort order is stable, we first assign to each atom
  // base value an index based on its order of appearance in the chain of
  // comparisons. We call this index `BaseOrdering`. For example, for:
  //    b[3] == c[2] && a[1] == d[1] && b[4] == c[3]
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  }
  Value *Addr = LoadI->getOperand(0);
  if (Addr->getType()->getPointerAddressSpace() != 0) {
    LLVM_DEBUG(dbgs() << "from non-zero AddressSpace\n");
    return {};
  }
  const auto &DL = LoadI->getDataLayout();
  if (!isDereferenceablePointer(Addr, LoadI->getType(), DL)) {
    LLVM_DEBUG(dbgs() << "not dereferenceable\n");
    // We need to make sure that we can do comparison in any order, so we
    // require memory to be unconditionally dereferenceable.
    return {};
  }

```
- EN: Core entities appearing here include BaseIdentifier, getBaseId, visitICmpLoadOperand, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout.
- CN: 此处出现的核心实体包括 BaseIdentifier, getBaseId, visitICmpLoadOperand，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。

### Lines 164-246

```cpp
  APInt Offset = APInt(DL.getIndexTypeSizeInBits(Addr->getType()), 0);
  Value *Base = Addr;
  auto *GEP = dyn_cast<GetElementPtrInst>(Addr);
  if (GEP) {
    LLVM_DEBUG(dbgs() << "GEP\n");
    if (GEP->isUsedOutsideOfBlock(LoadI->getParent())) {
      LLVM_DEBUG(dbgs() << "used outside of block\n");
      return {};
    }
    if (!GEP->accumulateConstantOffset(DL, Offset))
      return {};
    Base = GEP->getPointerOperand();
  }
  return BCEAtom(GEP, LoadI, BaseId.getBaseId(Base), Offset);
}

namespace {
// A comparison between two BCE atoms, e.g. `a == o.a` in the example at the
// top.
// Note: the terminology is misleading: the comparison is symmetric, so there
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // The basic block where this comparison happens.
  BasicBlock *BB;
  // Instructions relating to the BCECmp and branch.
  InstructionSet BlockInsts;
  // The block requires splitting.
  bool RequireSplit = false;
  // Original order of this block in the chain.
  unsigned OrigOrder = 0;

private:
  BCECmp Cmp;
};
} // namespace

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include BCECmp, Lhs, BCECmpBlock, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AliasAnalysis.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 BCECmp, Lhs, BCECmpBlock，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AliasAnalysis 等分析结果。

### Lines 247-336

```cpp
bool BCECmpBlock::canSinkBCECmpInst(const Instruction *Inst,
                                    AliasAnalysis &AA) const {
  // If this instruction may clobber the loads and is in middle of the BCE cmp
  // block instructions, then bail for now.
  if (Inst->mayWriteToMemory()) {
    auto MayClobber = [&](LoadInst *LI) {
      // If a potentially clobbering instruction comes before the load,
      // we can still safely sink the load.
      return (Inst->getParent() != LI->getParent() || !Inst->comesBefore(LI)) &&
             isModSet(AA.getModRefInfo(Inst, MemoryLocation::get(LI)));
    };
    if (MayClobber(Cmp.Lhs.LoadI) || MayClobber(Cmp.Rhs.LoadI))
      return false;
  }
  // Make sure this instruction does not use any of the BCE cmp block
  // instructions as operand.
  return llvm::none_of(Inst->operands(), [&](const Value *Op) {
    const Instruction *OpI = dyn_cast<Instruction>(Op);
    return OpI && BlockInsts.contains(OpI);
  });
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  LLVM_DEBUG(dbgs() << "cmp "
                    << (ExpectedPredicate == ICmpInst::ICMP_EQ ? "eq" : "ne")
                    << "\n");
  auto Lhs = visitICmpLoadOperand(CmpI->getOperand(0), BaseId);
  if (!Lhs.BaseId)
    return std::nullopt;
  auto Rhs = visitICmpLoadOperand(CmpI->getOperand(1), BaseId);
  if (!Rhs.BaseId)
    return std::nullopt;
  const auto &DL = CmpI->getDataLayout();
  return BCECmp(std::move(Lhs), std::move(Rhs),
                DL.getTypeSizeInBits(CmpI->getOperand(0)->getType()), CmpI);
}

```
- EN: Core entities appearing here include split, canSplit, doesOtherWork, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AliasAnalysis, DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 split, canSplit, doesOtherWork，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AliasAnalysis, DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 337-417

```cpp
// Visit the given comparison block. If this is a comparison between two valid
// BCE atoms, returns the comparison.
static std::optional<BCECmpBlock>
visitCmpBlock(Value *const Val, BasicBlock *const Block,
              const BasicBlock *const PhiBlock, BaseIdentifier &BaseId) {
  if (Block->empty())
    return std::nullopt;
  auto *Term = Block->getTerminator();
  Value *Cond;
  ICmpInst::Predicate ExpectedPredicate;
  if (isa<UncondBrInst>(Term)) {
    // In this case, we expect an incoming value which is the result of the
    // comparison. This is the last link in the chain of comparisons (note
    // that this does not mean that this is the last incoming value, blocks
    // can be reordered).
    Cond = Val;
    ExpectedPredicate = ICmpInst::ICMP_EQ;
  } else if (auto *BranchI = dyn_cast<CondBrInst>(Term)) {
    // In this case, we expect a constant incoming value (the comparison is
    // chained).
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
public:
  using ContiguousBlocks = std::vector<BCECmpBlock>;

  BCECmpChain(const std::vector<BasicBlock *> &Blocks, PHINode &Phi,
              AliasAnalysis &AA);

  bool simplify(const TargetLibraryInfo &TLI, AliasAnalysis &AA,
                DomTreeUpdater &DTU);

  bool atLeastOneMerged() const {
    return any_of(MergedBlocks_,
                  [](const auto &Blocks) { return Blocks.size() > 1; });
  }

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include BCECmpChain, atLeastOneMerged, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AliasAnalysis, DomTreeUpdater, TargetLibraryInfo.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 BCECmpChain, atLeastOneMerged，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AliasAnalysis, DomTreeUpdater, TargetLibraryInfo 等分析结果。

### Lines 418-495

```cpp
private:
  PHINode &Phi_;
  // The list of all blocks in the chain, grouped by contiguity.
  std::vector<ContiguousBlocks> MergedBlocks_;
  // The original entry block (before sorting);
  BasicBlock *EntryBlock_;
};
} // namespace

static bool areContiguous(const BCECmpBlock &First, const BCECmpBlock &Second) {
  return First.Lhs().BaseId == Second.Lhs().BaseId &&
         First.Rhs().BaseId == Second.Rhs().BaseId &&
         First.Lhs().Offset + First.SizeBits() / 8 == Second.Lhs().Offset &&
         First.Rhs().Offset + First.SizeBits() / 8 == Second.Rhs().Offset;
}

static unsigned getMinOrigOrder(const BCECmpChain::ContiguousBlocks &Blocks) {
  unsigned MinOrigOrder = std::numeric_limits<unsigned>::max();
  for (const BCECmpBlock &Block : Blocks)
    MinOrigOrder = std::min(MinOrigOrder, Block.OrigOrder);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  BaseIdentifier BaseId;
  for (BasicBlock *const Block : Blocks) {
    assert(Block && "invalid block");
    if (Block->hasAddressTaken()) {
      LLVM_DEBUG(dbgs() << "cannot merge blocks with blockaddress\n");
      return;
    }
    std::optional<BCECmpBlock> Comparison = visitCmpBlock(
        Phi.getIncomingValueForBlock(Block), Block, Phi.getParent(), BaseId);
    if (!Comparison) {
      LLVM_DEBUG(dbgs() << "chain with invalid BCECmpBlock, no merge.\n");
      return;
    }
    if (Comparison->doesOtherWork()) {
```
- EN: Core entities appearing here include areContiguous, getMinOrigOrder, mergeBlocks, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AliasAnalysis.
- CN: 此处出现的核心实体包括 areContiguous, getMinOrigOrder, mergeBlocks，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AliasAnalysis 等分析结果。

### Lines 496-574

```cpp
      LLVM_DEBUG(dbgs() << "block '" << Comparison->BB->getName()
                        << "' does extra work besides compare\n");
      if (Comparisons.empty()) {
        // This is the initial block in the chain, in case this block does other
        // work, we can try to split the block and move the irrelevant
        // instructions to the predecessor.
        //
        // If this is not the initial block in the chain, splitting it wont
        // work.
        //
        // As once split, there will still be instructions before the BCE cmp
        // instructions that do other work in program order, i.e. within the
        // chain before sorting. Unless we can abort the chain at this point
        // and start anew.
        //
        // NOTE: we only handle blocks a with single predecessor for now.
        if (Comparison->canSplit(AA)) {
          LLVM_DEBUG(dbgs()
                     << "Split initial block '" << Comparison->BB->getName()
                     << "' that does extra work besides compare\n");
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

namespace {

// A class to compute the name of a set of merged basic blocks.
// This is optimized for the common case of no block names.
class MergedBlockName {
  // Storage for the uncommon case of several named blocks.
  SmallString<16> Scratch;

public:
  explicit MergedBlockName(ArrayRef<BCECmpBlock> Comparisons)
      : Name(makeName(Comparisons)) {}
  const StringRef Name;

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include MergedBlockName, which carry the local state, declarations, or transformation entry points for this region.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 MergedBlockName，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 575-665

```cpp
private:
  StringRef makeName(ArrayRef<BCECmpBlock> Comparisons) {
    assert(!Comparisons.empty() && "no basic block");
    // Fast path: only one block, or no names at all.
    if (Comparisons.size() == 1)
      return Comparisons[0].BB->getName();
    const int size = std::accumulate(Comparisons.begin(), Comparisons.end(), 0,
                                     [](int i, const BCECmpBlock &Cmp) {
                                       return i + Cmp.BB->getName().size();
                                     });
    if (size == 0)
      return StringRef("", 0);

    // Slow path: at least two blocks, at least one block with a name.
    Scratch.clear();
    // We'll have `size` bytes for name and `Comparisons.size() - 1` bytes for
    // separators.
    Scratch.reserve(size + Comparisons.size() - 1);
    const auto append = [this](StringRef str) {
      Scratch.append(str.begin(), str.end());
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      BasicBlock::Create(Context, MergedBlockName(Comparisons).Name,
                         NextCmpBlock->getParent(), InsertBefore);
  IRBuilder<> Builder(BB);
  // Add the GEPs from the first BCECmpBlock.
  Value *Lhs, *Rhs;
  if (FirstCmp.Lhs().GEP)
    Lhs = Builder.Insert(FirstCmp.Lhs().GEP->clone());
  else
    Lhs = FirstCmp.Lhs().LoadI->getPointerOperand();
  if (FirstCmp.Rhs().GEP)
    Rhs = Builder.Insert(FirstCmp.Rhs().GEP->clone());
  else
    Rhs = FirstCmp.Rhs().LoadI->getPointerOperand();

```
- EN: Core entities appearing here include makeName, computeMergedBranchWeights, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AliasAnalysis, DomTreeUpdater, TargetLibraryInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 makeName, computeMergedBranchWeights，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AliasAnalysis, DomTreeUpdater, TargetLibraryInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 666-754

```cpp
  Value *IsEqual = nullptr;
  LLVM_DEBUG(dbgs() << "Merging " << Comparisons.size() << " comparisons -> "
                    << BB->getName() << "\n");

  // If there is one block that requires splitting, we do it now, i.e.
  // just before we know we will collapse the chain. The instructions
  // can be executed before any of the instructions in the chain.
  const auto *ToSplit = llvm::find_if(
      Comparisons, [](const BCECmpBlock &B) { return B.RequireSplit; });
  if (ToSplit != Comparisons.end()) {
    LLVM_DEBUG(dbgs() << "Splitting non_BCE work to header\n");
    ToSplit->split(BB, AA);
  }

  if (Comparisons.size() == 1) {
    LLVM_DEBUG(dbgs() << "Only one comparison, updating branches\n");
    // Use clone to keep the metadata
    Instruction *const LhsLoad = Builder.Insert(FirstCmp.Lhs().LoadI->clone());
    Instruction *const RhsLoad = Builder.Insert(FirstCmp.Rhs().LoadI->clone());
    LhsLoad->replaceUsesOfWith(LhsLoad->getOperand(0), Lhs);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  }

  // Replace the original cmp chain with the new cmp chain by pointing all
  // predecessors of EntryBlock_ to NextCmpBlock instead. This makes all cmp
  // blocks in the old chain unreachable.
  while (!pred_empty(EntryBlock_)) {
    BasicBlock* const Pred = *pred_begin(EntryBlock_);
    LLVM_DEBUG(dbgs() << "Updating jump into old chain from " << Pred->getName()
                      << "\n");
    Pred->getTerminator()->replaceUsesOfWith(EntryBlock_, NextCmpBlock);
    DTU.applyUpdates({{DominatorTree::Delete, Pred, EntryBlock_},
                      {DominatorTree::Insert, Pred, NextCmpBlock}});
  }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AliasAnalysis, DataLayout, DominatorTree, DomTreeUpdater. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AliasAnalysis, DataLayout, DominatorTree, DomTreeUpdater 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 755-836

```cpp
  // If the old cmp chain was the function entry, we need to update the function
  // entry.
  const bool ChainEntryIsFnEntry = EntryBlock_->isEntryBlock();
  if (ChainEntryIsFnEntry && DTU.hasDomTree()) {
    LLVM_DEBUG(dbgs() << "Changing function entry from "
                      << EntryBlock_->getName() << " to "
                      << NextCmpBlock->getName() << "\n");
    DTU.getDomTree().setNewRoot(NextCmpBlock);
    DTU.applyUpdates({{DominatorTree::Delete, NextCmpBlock, EntryBlock_}});
  }
  EntryBlock_ = nullptr;

  // Delete merged blocks. This also removes incoming values in phi.
  SmallVector<BasicBlock *, 16> DeadBlocks;
  for (const auto &Blocks : MergedBlocks_) {
    for (const BCECmpBlock &Block : Blocks) {
      LLVM_DEBUG(dbgs() << "Deleting merged block " << Block.BB->getName()
                        << "\n");
      DeadBlocks.push_back(Block.BB);
    }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // We are looking for something that has the following structure:
  //   bb1 --eq--> bb2 --eq--> bb3 --eq--> bb4 --+
  //     \            \           \               \
  //      ne           ne          ne              \
  //       \            \           \               v
  //        +------------+-----------+----------> bb_phi
  //
  //  - The last basic block (bb4 here) must branch unconditionally to bb_phi.
  //    It's the only block that contributes a non-constant value to the Phi.
  //  - All other blocks (b1, b2, b3) must have exactly two successors, one of
  //    them being the phi block.
  //  - All intermediate blocks (bb2, bb3) must have only one predecessor.
  //  - Blocks cannot do other work besides the comparison, see doesOtherWork()

```
- EN: Core entities appearing here include getOrderedBlocks, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AliasAnalysis, DominatorTree, DomTreeUpdater, TargetLibraryInfo.
- CN: 此处出现的核心实体包括 getOrderedBlocks，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AliasAnalysis, DominatorTree, DomTreeUpdater, TargetLibraryInfo 等分析结果。

### Lines 837-912

```cpp
  // The blocks are not necessarily ordered in the phi, so we start from the
  // last block and reconstruct the order.
  BasicBlock *LastBlock = nullptr;
  for (unsigned I = 0; I < Phi.getNumIncomingValues(); ++I) {
    if (isa<ConstantInt>(Phi.getIncomingValue(I))) continue;
    if (LastBlock) {
      // There are several non-constant values.
      LLVM_DEBUG(dbgs() << "skip: several non-constant values\n");
      return false;
    }
    if (!isa<ICmpInst>(Phi.getIncomingValue(I)) ||
        cast<ICmpInst>(Phi.getIncomingValue(I))->getParent() !=
            Phi.getIncomingBlock(I)) {
      // Non-constant incoming value is not from a cmp instruction or not
      // produced by the last block. We could end up processing the value
      // producing block more than once.
      //
      // This is an uncommon case, so we bail.
      LLVM_DEBUG(
          dbgs()
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  DomTreeUpdater DTU(DT, /*PostDominatorTree*/ nullptr,
                     DomTreeUpdater::UpdateStrategy::Eager);

  bool MadeChange = false;

  for (BasicBlock &BB : llvm::drop_begin(F)) {
    // A Phi operation is always first in a basic block.
    if (auto *const Phi = dyn_cast<PHINode>(&*BB.begin()))
      MadeChange |= processPhi(*Phi, TLI, AA, DTU);
  }

  return MadeChange;
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AliasAnalysis, DominatorTree, DomTreeUpdater, PostDominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AliasAnalysis, DominatorTree, DomTreeUpdater, PostDominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 913-925

```cpp
PreservedAnalyses MergeICmpsPass::run(Function &F,
                                      FunctionAnalysisManager &AM) {
  auto &TLI = AM.getResult<TargetLibraryAnalysis>(F);
  auto &TTI = AM.getResult<TargetIRAnalysis>(F);
  auto &AA = AM.getResult<AAManager>(F);
  auto *DT = AM.getCachedResult<DominatorTreeAnalysis>(F);
  const bool MadeChanges = runImpl(F, TLI, TTI, AA, DT);
  if (!MadeChanges)
    return PreservedAnalyses::all();
  PreservedAnalyses PA;
  PA.preserve<DominatorTreeAnalysis>();
  return PA;
}
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AAManager, DominatorTree.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AAManager, DominatorTree 等分析结果。

## Key Concepts / 关键概念

- EN: Domain: `Scalar` focuses on scalar optimizations and loop/CFG transforms.
  - CN: 领域：`Scalar` 主要处理 标量优化以及循环/CFG 变换。
- EN: Primary entities: `BCEAtom, BaseIdentifier, BCECmp, BCECmpBlock, BCECmpChain, MergedBlockName, getBaseId, visitICmpLoadOperand` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`BCEAtom, BaseIdentifier, BCECmp, BCECmpBlock, BCECmpChain, MergedBlockName, getBaseId, visitICmpLoadOperand` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AAManager, AliasAnalysis, DataLayout, DominatorTree, DomTreeUpdater, PostDominatorTree, TargetLibraryInfo, TargetTransformInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AAManager, AliasAnalysis, DataLayout, DominatorTree, DomTreeUpdater, PostDominatorTree, TargetLibraryInfo, TargetTransformInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/AliasAnalysis.h`, `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/Loads.h`, `llvm/Analysis/TargetLibraryInfo.h`, `llvm/Analysis/TargetTransformInfo.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/AliasAnalysis.h`, `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/Loads.h`, `llvm/Analysis/TargetLibraryInfo.h`, `llvm/Analysis/TargetTransformInfo.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Instruction.h`, `llvm/IR/ProfDataUtils.h`, `llvm/Transforms/Scalar/MergeICmps.h`, `llvm/Transforms/Utils/BasicBlockUtils.h`, `llvm/Transforms/Utils/BuildLibCalls.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Instruction.h`, `llvm/IR/ProfDataUtils.h`, `llvm/Transforms/Scalar/MergeICmps.h`, `llvm/Transforms/Utils/BasicBlockUtils.h`, `llvm/Transforms/Utils/BuildLibCalls.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/ADT/SmallString.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/ADT/SmallString.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `algorithm`, `numeric`, `utility`, `vector` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`algorithm`, `numeric`, `utility`, `vector` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `AAManager`, `AliasAnalysis`, `DataLayout`, `DominatorTree`, `DomTreeUpdater`, `PostDominatorTree`, `TargetLibraryInfo`, `TargetTransformInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AAManager`, `AliasAnalysis`, `DataLayout`, `DominatorTree`, `DomTreeUpdater`, `PostDominatorTree`, `TargetLibraryInfo`, `TargetTransformInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
