# VPlan.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Vectorize/VPlan.cpp` | `llvm/lib/Transforms/Vectorize/VPlan.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements vectorizer Plan within LLVM's vectorization and VPlan infrastructure layer. | 该源文件实现了 LLVM 向量化与 VPlan 基础设施层中的 VPlan 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-129

```cpp
//===- VPlan.cpp - Vectorizer Plan ----------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///
/// \file
/// This is the LLVM vectorization plan. It represents a candidate for
/// vectorization, allowing to plan and optimize how to vectorize a given loop
/// before generating LLVM-IR.
/// The vectorizer uses vectorization plans to estimate the costs of potential
/// candidates and if profitable to execute the desired plan, generating vector
/// LLVM-IR code.
///
//===----------------------------------------------------------------------===//

#include "VPlan.h"
#include "LoopVectorizationPlanner.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

void VPRecipeBase::dump() const {
  VPSlotTracker SlotTracker(getParent() ? getParent()->getPlan() : nullptr);
  print(dbgs(), "", SlotTracker);
  dbgs() << "\n";
}
#endif

#if !defined(NDEBUG)
bool VPRecipeValue::isDefinedBy(const VPDef *D) const {
  return getDefiningRecipe() == D;
}
#endif

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 130-258

```cpp
VPRecipeBase *VPValue::getDefiningRecipe() {
  auto *RecipeValue = dyn_cast<VPRecipeValue>(this);
  if (!RecipeValue)
    return nullptr;
  if (auto *MultiDef = dyn_cast<VPMultiDefValue>(RecipeValue))
    return MultiDef->getDef();
  return static_cast<VPSingleDefRecipe *>(RecipeValue);
}

const VPRecipeBase *VPValue::getDefiningRecipe() const {
  return const_cast<VPValue *>(this)->getDefiningRecipe();
}

Value *VPValue::getLiveInIRValue() const {
  return cast<VPIRValue>(this)->getValue();
}

Type *VPIRValue::getType() const { return getUnderlyingValue()->getType(); }

VPRecipeValue::~VPRecipeValue() {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  iterator It = begin();
  while (It != end() && It->isPhi())
    It++;
  return It;
}

VPTransformState::VPTransformState(const TargetTransformInfo *TTI,
                                   ElementCount VF, LoopInfo *LI,
                                   DominatorTree *DT, AssumptionCache *AC,
                                   IRBuilderBase &Builder, VPlan *Plan,
                                   Loop *CurrentParentLoop, Type *CanonicalIVTy)
    : TTI(TTI), VF(VF), CFG(DT), LI(LI), AC(AC), Builder(Builder), Plan(Plan),
      CurrentParentLoop(CurrentParentLoop), TypeAnalysis(*Plan), VPDT(*Plan) {}

```
- EN: Core entities appearing here include ~VPRecipeValue, VPRecipeValue, y, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DominatorTree, LoopInfo, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 ~VPRecipeValue, VPRecipeValue, y，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DominatorTree, LoopInfo, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 259-392

```cpp
Value *VPTransformState::get(const VPValue *Def, const VPLane &Lane) {
  if (isa<VPIRValue, VPSymbolicValue>(Def))
    return Def->getUnderlyingValue();

  if (hasScalarValue(Def, Lane))
    return Data.VPV2Scalars[Def][Lane.mapToCacheIndex(VF)];

  if (!Lane.isFirstLane() && vputils::isSingleScalar(Def) &&
      hasScalarValue(Def, VPLane::getFirstLane())) {
    return Data.VPV2Scalars[Def][0];
  }

  // Look through BuildVector to avoid redundant extracts.
  // TODO: Remove once replicate regions are unrolled explicitly.
  if (Lane.getKind() == VPLane::Kind::First && match(Def, m_BuildVector())) {
    auto *BuildVector = cast<VPInstruction>(Def);
    return get(BuildVector->getOperand(Lane.getKnownLane()), true);
  }

  assert(hasVectorValue(Def));
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    // We must handle each element of a vectorized struct type.
    for (unsigned I = 0, E = StructTy->getNumElements(); I != E; I++) {
      Value *ScalarValue = Builder.CreateExtractValue(ScalarInst, I);
      Value *VectorValue = Builder.CreateExtractValue(WideValue, I);
      VectorValue =
          Builder.CreateInsertElement(VectorValue, ScalarValue, LaneExpr);
      WideValue = Builder.CreateInsertValue(WideValue, VectorValue, I);
    }
  } else {
    WideValue = Builder.CreateInsertElement(WideValue, ScalarInst, LaneExpr);
  }
  return WideValue;
}

```
- EN: Core entities appearing here include hasScalarValue, setDebugLocFrom, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 hasScalarValue, setDebugLocFrom，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 393-518

```cpp
BasicBlock *VPBasicBlock::createEmptyBasicBlock(VPTransformState &State) {
  auto &CFG = State.CFG;
  // BB stands for IR BasicBlocks. VPBB stands for VPlan VPBasicBlocks.
  // Pred stands for Predessor. Prev stands for Previous - last visited/created.
  BasicBlock *PrevBB = CFG.PrevBB;
  BasicBlock *NewBB = BasicBlock::Create(PrevBB->getContext(), getName(),
                                         PrevBB->getParent(), CFG.ExitBB);
  LLVM_DEBUG(dbgs() << "LV: created " << NewBB->getName() << '\n');

  return NewBB;
}

void VPBasicBlock::connectToPredecessors(VPTransformState &State) {
  auto &CFG = State.CFG;
  BasicBlock *NewBB = CFG.VPBB2IRBB[this];

  // Register NewBB in its loop. In innermost loops its the same for all
  // BB's.
  Loop *ParentLoop = State.CurrentParentLoop;
  // If this block has a sole successor that is an exit block or is an exit
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
void VPBasicBlock::execute(VPTransformState *State) {
  if (VPBlockUtils::isHeader(this, State->VPDT)) {
    // Create and register the new vector loop.
    Loop *PrevParentLoop = State->CurrentParentLoop;
    State->CurrentParentLoop = State->LI->AllocateLoop();

    // Insert the new loop into the loop nest and register the new basic blocks
    // before calling any utilities such as SCEV that require valid LoopInfo.
    if (PrevParentLoop)
      PrevParentLoop->addChildLoop(State->CurrentParentLoop);
    else
      State->LI->addTopLevelLoop(State->CurrentParentLoop);
  }

```
- EN: Core entities appearing here include connectToPredecessors, execute, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, LoopInfo, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 connectToPredecessors, execute，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, LoopInfo, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 519-644

```cpp
  // 1. Create an IR basic block.
  BasicBlock *NewBB = createEmptyBasicBlock(*State);

  State->Builder.SetInsertPoint(NewBB);
  // Temporarily terminate with unreachable until CFG is rewired.
  UnreachableInst *Terminator = State->Builder.CreateUnreachable();
  State->Builder.SetInsertPoint(Terminator);

  State->CFG.PrevBB = NewBB;
  State->CFG.VPBB2IRBB[this] = NewBB;
  connectToPredecessors(*State);

  // 2. Fill the IR basic block with IR instructions.
  executeRecipes(State, NewBB);

  // If this block is a latch, update CurrentParentLoop.
  if (VPBlockUtils::isLatch(this, State->VPDT))
    State->CurrentParentLoop = State->CurrentParentLoop->getParentLoop();
}

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  }

  assert(
      !IsCondBranch && !IsBranchOnTwoConds &&
      "block with 0 or 1 successors terminated by conditional branch recipe");
  return false;
}

VPRecipeBase *VPBasicBlock::getTerminator() {
  if (hasConditionalTerminator(this))
    return &back();
  return nullptr;
}

```
- EN: Core entities appearing here include executeRecipes, n, hasConditionalTerminator, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 executeRecipes, n, hasConditionalTerminator，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 645-774

```cpp
const VPRecipeBase *VPBasicBlock::getTerminator() const {
  if (hasConditionalTerminator(this))
    return &back();
  return nullptr;
}

bool VPBasicBlock::isExiting() const {
  return getParent() && getParent()->getExitingBasicBlock() == this;
}

#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
void VPBlockBase::print(raw_ostream &O) const {
  VPSlotTracker SlotTracker(getPlan());
  print(O, "", SlotTracker);
}

void VPBlockBase::printSuccessors(raw_ostream &O, const Twine &Indent) const {
  if (!hasSuccessors()) {
    O << Indent << "No successors\n";
  } else {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
}

const VPBasicBlock *VPBasicBlock::getCFGPredecessor(unsigned Idx) const {
  const VPBlockBase *Pred = nullptr;
  if (hasPredecessors()) {
    Pred = getPredecessors()[Idx];
  } else {
    auto *Region = getParent();
    assert(Region && !Region->isReplicator() && Region->getEntry() == this &&
           "must be in the entry block of a non-replicate region");
    assert(Idx < 2 && Region->getNumPredecessors() == 1 &&
           "loop region has a single predecessor (preheader), its entry block "
           "has 2 incoming blocks");

```
- EN: Core entities appearing here include isExiting, print, printSuccessors, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 isExiting, print, printSuccessors，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 775-902

```cpp
    // Idx ==  0 selects the predecessor of the region, Idx == 1 selects the
    // region itself whose exiting block feeds the phi across the backedge.
    Pred = Idx == 0 ? Region->getSinglePredecessor() : Region;
  }
  return Pred->getExitingBasicBlock();
}

InstructionCost VPRegionBlock::cost(ElementCount VF, VPCostContext &Ctx) {
  if (!isReplicator()) {
    // Neglect the cost of canonical IV, matching the legacy cost model.
    InstructionCost Cost = 0;
    for (VPBlockBase *Block : vp_depth_first_shallow(getEntry()))
      Cost += Block->cost(VF, Ctx);
    InstructionCost BackedgeCost =
        ForceTargetInstructionCost.getNumOccurrences()
            ? InstructionCost(ForceTargetInstructionCost)
            : Ctx.TTI.getCFInstrCost(Instruction::UncondBr, Ctx.CostKind);
    LLVM_DEBUG(dbgs() << "Cost of " << BackedgeCost << " for VF " << VF
                      << ": vector loop backedge\n");
    Cost += BackedgeCost;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    if (auto *VPBB = dyn_cast<VPBasicBlock>(VPB)) {
      // Replace all operands of recipes and all VPValues defined in VPBB with
      // DummyValue so the block can be deleted.
      for (VPRecipeBase &R : *VPBB) {
        for (auto *Def : R.definedValues())
          Def->replaceAllUsesWith(&DummyValue);

        for (unsigned I = 0, E = R.getNumOperands(); I != E; I++)
          R.setOperand(I, &DummyValue);
      }
    } else if (auto *CanIV = cast<VPRegionBlock>(VPB)->getCanonicalIV()) {
      CanIV->replaceAllUsesWith(&DummyValue);
    }

```
- EN: Core entities appearing here include cost, dissolveToCFGLoop, VectorTripCount, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 cost, dissolveToCFGLoop, VectorTripCount，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 903-1027

```cpp
    delete VPB;
  }
  for (VPValue *VPV : getLiveIns())
    delete VPV;
  delete BackedgeTakenCount;
}

VPIRBasicBlock *VPlan::getExitBlock(BasicBlock *IRBB) const {
  auto Iter = find_if(getExitBlocks(), [IRBB](const VPIRBasicBlock *VPIRBB) {
    return VPIRBB->getIRBasicBlock() == IRBB;
  });
  assert(Iter != getExitBlocks().end() && "no exit block found");
  return *Iter;
}

bool VPlan::isExitBlock(VPBlockBase *VPBB) {
  return is_contained(ExitBlocks, VPBB);
}

/// To make RUN_VPLAN_PASS print final VPlan.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    }

    Loop *OrigLoop =
        State->LI->getLoopFor(getScalarHeader()->getIRBasicBlock());
    auto Blocks = OrigLoop->getBlocksVector();
    Blocks.push_back(ScalarPh);
    while (!OrigLoop->isInnermost())
      State->LI->erase(*OrigLoop->begin());
    State->LI->erase(OrigLoop);
    for (auto *BB : Blocks)
      State->LI->removeBlock(BB);
    DeleteDeadBlocks(Blocks, &State->CFG.DTU);
  }

```
- EN: Core entities appearing here include isExitBlock, execute, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, LoopInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 isExitBlock, execute，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, LoopInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1028-1157

```cpp
  State->CFG.DTU.flush();

  VPBasicBlock *Header = vputils::getFirstLoopHeader(*this, State->VPDT);
  if (!Header)
    return;

  auto *LatchVPBB = cast<VPBasicBlock>(Header->getPredecessors()[1]);
  BasicBlock *VectorLatchBB = State->CFG.VPBB2IRBB[LatchVPBB];

  // Fix the latch value of canonical, reduction and first-order recurrences
  // phis in the vector loop.
  for (VPRecipeBase &R : Header->phis()) {
    // Skip phi-like recipes that generate their backedege values themselves.
    if (isa<VPWidenPHIRecipe>(&R))
      continue;

    auto *PhiR = cast<VPSingleDefRecipe>(&R);
    // VPInstructions currently model scalar Phis only.
    bool NeedsScalar = isa<VPInstruction>(PhiR) ||
                       (isa<VPReductionPHIRecipe>(PhiR) &&
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
void VPlan::print(raw_ostream &O) const {
  VPSlotTracker SlotTracker(this);

  O << "VPlan '" << getName() << "' {";

  printLiveIns(O);

  ReversePostOrderTraversal<VPBlockShallowTraversalWrapper<const VPBlockBase *>>
      RPOT(getEntry());
  for (const VPBlockBase *Block : RPOT) {
    O << '\n';
    Block->print(O, "", SlotTracker);
  }

```
- EN: Core entities appearing here include cost, isOuterLoop, printLiveIns, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 cost, isOuterLoop, printLiveIns，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1158-1288

```cpp
  O << "}\n";
}

std::string VPlan::getName() const {
  std::string Out;
  raw_string_ostream RSO(Out);
  RSO << Name << " for ";
  if (!VFs.empty()) {
    RSO << "VF={" << VFs[0];
    for (ElementCount VF : drop_begin(VFs))
      RSO << "," << VF;
    RSO << "},";
  }

  if (UFs.empty()) {
    RSO << "UF>=1";
  } else {
    RSO << "UF={" << UFs[0];
    for (unsigned UF : drop_begin(UFs))
      RSO << "," << UF;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // Map and propagate materialized state for symbolic values.
  for (auto [OldSV, NewSV] :
       {std::pair{&VectorTripCount, &NewPlan->VectorTripCount},
        {&VF, &NewPlan->VF},
        {&UF, &NewPlan->UF},
        {&VFxUF, &NewPlan->VFxUF},
        {BackedgeTakenCount, NewPlan->BackedgeTakenCount}}) {
    if (!OldSV)
      continue;
    Old2NewVPValues[OldSV] = NewSV;
    if (OldSV->isMaterialized())
      NewSV->markMaterialized();
  }

```
- EN: Core entities appearing here include getName, printDOT, vp_depth_first_shallow, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 getName, printDOT, vp_depth_first_shallow，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1289-1417

```cpp
  remapOperands(Entry, NewEntry, Old2NewVPValues);

  // Initialize remaining fields of cloned VPlan.
  NewPlan->VFs = VFs;
  NewPlan->UFs = UFs;
  // TODO: Adjust names.
  NewPlan->Name = Name;
  if (TripCount) {
    assert(Old2NewVPValues.contains(TripCount) &&
           "TripCount must have been added to Old2NewVPValues");
    NewPlan->TripCount = Old2NewVPValues[TripCount];
  }

  // Transfer all cloned blocks (the second half of all current blocks) from
  // current to new VPlan.
  unsigned NumBlocksAfterCloning = CreatedBlocks.size();
  for (unsigned I :
       seq<unsigned>(NumBlocksBeforeCloning, NumBlocksAfterCloning))
    NewPlan->CreatedBlocks.push_back(this->CreatedBlocks[I]);
  CreatedBlocks.truncate(NumBlocksBeforeCloning);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
void VPlanPrinter::dumpEdges(const VPBlockBase *Block) {
  auto &Successors = Block->getSuccessors();
  if (Successors.size() == 1)
    drawEdge(Block, Successors.front(), false, "");
  else if (Successors.size() == 2) {
    drawEdge(Block, Successors.front(), false, "T");
    drawEdge(Block, Successors.back(), false, "F");
  } else {
    unsigned SuccessorNumber = 0;
    for (auto *Successor : Successors)
      drawEdge(Block, Successor, false, Twine(SuccessorNumber++));
  }
}

```
- EN: Core entities appearing here include getUID, getOrCreateName, dump, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 getUID, getOrCreateName, dump，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1418-1541

```cpp
void VPlanPrinter::dumpBasicBlock(const VPBasicBlock *BasicBlock) {
  // Implement dot-formatted dump by performing plain-text dump into the
  // temporary storage followed by some post-processing.
  OS << Indent << getUID(BasicBlock) << " [label =\n";
  bumpIndent(1);
  std::string Str;
  raw_string_ostream SS(Str);
  // Use no indentation as we need to wrap the lines into quotes ourselves.
  BasicBlock->print(SS, "", SlotTracker);

  // We need to process each line of the output separately, so split
  // single-string plain-text dump.
  SmallVector<StringRef, 0> Lines;
  StringRef(Str).rtrim('\n').split(Lines, "\n");

  auto EmitLine = [&](StringRef Line, StringRef Suffix) {
    OS << Indent << '"' << DOT::EscapeString(Line.str()) << "\\l\"" << Suffix;
  };

  // Don't need the "+" after the last line.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
}

#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
void VPValue::printAsOperand(raw_ostream &OS, VPSlotTracker &Tracker) const {
  OS << Tracker.getOrCreateName(this);
}

void VPUser::printOperands(raw_ostream &O, VPSlotTracker &SlotTracker) const {
  interleaveComma(operands(), O, [&O, &SlotTracker](VPValue *Op) {
    Op->printAsOperand(O, SlotTracker);
  });
}
#endif

```
- EN: Core entities appearing here include dumpBasicBlock, dumpRegion, isDefinedInsideLoopRegions, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 dumpBasicBlock, dumpRegion, isDefinedInsideLoopRegions，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1542-1675

```cpp
void VPSlotTracker::assignName(const VPValue *V) {
  assert(!VPValue2Name.contains(V) && "VPValue already has a name!");
  auto *UV = V->getUnderlyingValue();
  auto *VPI = dyn_cast_or_null<VPInstruction>(V);
  if (!UV && !(VPI && !VPI->getName().empty())) {
    VPValue2Name[V] = (Twine("vp<%") + Twine(NextSlot) + ">").str();
    NextSlot++;
    return;
  }

  // Use the name of the underlying Value, wrapped in "ir<>", and versioned by
  // appending ".Number" to the name if there are multiple uses.
  std::string Name;
  if (UV)
    Name = getName(UV);
  else
    Name = VPI->getName();

  assert(!Name.empty() && "Name cannot be empty.");
  StringRef Prefix = UV ? "ir<" : "vp<%";
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                                               VPValue *TrueVal,
                                               VPValue *FalseVal, DebugLoc DL) {
  assert(VPTypeAnalysis(*getInsertBlock()->getPlan())
             .inferScalarType(ChainOp)
             ->isIntegerTy(1) &&
         "ChainOp must be i1 for AnyOf reduction");
  VPIRFlags Flags(RecurKind::Or, /*IsOrdered=*/false, /*IsInLoop=*/false,
                  FastMathFlags());
  auto *OrReduce =
      createNaryOp(VPInstruction::ComputeReductionResult, {ChainOp}, Flags, DL);
  auto *Freeze = createNaryOp(Instruction::Freeze, {OrReduce}, DL);
  return createSelect(Freeze, TrueVal, FalseVal, DL, "rdx.select");
}

```
- EN: Core entities appearing here include assignName, assignNames, getName, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 assignName, assignNames, getName，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1676-1799

```cpp
bool LoopVectorizationPlanner::getDecisionAndClampRange(
    const std::function<bool(ElementCount)> &Predicate, VFRange &Range) {
  assert(!Range.isEmpty() && "Trying to test an empty VF range.");
  bool PredicateAtRangeStart = Predicate(Range.Start);

  for (ElementCount TmpVF : VFRange(Range.Start * 2, Range.End))
    if (Predicate(TmpVF) != PredicateAtRangeStart) {
      Range.End = TmpVF;
      break;
    }

  return PredicateAtRangeStart;
}

VPlan &LoopVectorizationPlanner::getPlanFor(ElementCount VF) const {
  assert(count_if(VPlans,
                  [VF](const VPlanPtr &Plan) { return Plan->hasVF(VF); }) ==
             1 &&
         "Multiple VPlans for VF.");

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      LoopVectorizeHints Hints(VectorLoop, /*InterleaveOnlyWhenForced*/ false,
                               *ORE);
      Hints.setAlreadyVectorized();
    }
  }
  // Tag the vector loop body so downstream passes can identify it.  Only
  // emit when remarks are enabled.
  if (ORE->enabled())
    VectorLoop->addIntLoopAttribute("llvm.loop.vectorize.body", 1);
  TargetTransformInfo::UnrollingPreferences UP;
  TTI.getUnrollingPreferences(VectorLoop, *PSE.getSE(), UP, ORE);
  if (!UP.UnrollVectorizedLoop || VectorizingEpilogue)
    addRuntimeUnrollDisableMetaData(VectorLoop);

```
- EN: Core entities appearing here include addRuntimeUnrollDisableMetaData, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 addRuntimeUnrollDisableMetaData，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1800-1922

```cpp
  // Set/update profile weights for the vector and remainder loops as original
  // loop iterations are now distributed among them. Note that original loop
  // becomes the scalar remainder loop after vectorization.
  //
  // For cases like foldTailByMasking() and requiresScalarEpiloque() we may
  // end up getting slightly roughened result but that should be OK since
  // profile is not inherently precise anyway. Note also possible bypass of
  // vector code caused by legality checks is ignored, assigning all the weight
  // to the vector loop, optimistically.
  //
  // For scalable vectorization we can't know at compile time how many
  // iterations of the loop are handled in one vector iteration, so instead
  // use the value of vscale used for tuning.
  unsigned AverageVectorTripCount = 0;
  unsigned RemainderAverageTripCount = 0;
  auto EC = VectorLoop->getLoopPreheader()->getParent()->getEntryCount();
  auto IsProfiled = EC && EC->getCount();
  if (!OrigAverageTripCount) {
    if (!IsProfiled)
      return;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      continue;
    Tys.push_back(toVectorizedTy(Types.inferScalarType(Op), VF));
  }
  return ScalarizationCost +
         TTI.getOperandsScalarizationOverhead(Tys, CostKind, VIC);
}

bool VPCostContext::useEmulatedMaskMemRefHack(const VPReplicateRecipe *R,
                                              ElementCount VF) {
  const Instruction *UI = R->getUnderlyingInstr();
  if (isa<LoadInst>(UI))
    return true;
  assert(isa<StoreInst>(UI) && "R must either be a load or store");

```
- EN: Core entities appearing here include printPlans, getOperandInfo, to_vector, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 printPlans, getOperandInfo, to_vector，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1923-1962

```cpp
  if (!NumPredStores) {
    // Count the number of predicated stores in the VPlan, caching the result.
    // Only stores where scatter is not legal are counted, matching the legacy
    // cost model behavior.
    const VPlan &Plan = *R->getParent()->getPlan();
    NumPredStores = 0;
    for (const VPRegionBlock *VPRB :
         VPBlockUtils::blocksOnly<const VPRegionBlock>(
             vp_depth_first_shallow(Plan.getVectorLoopRegion()->getEntry()))) {
      assert(VPRB->isReplicator() && "must only contain replicate regions");
      for (const VPBasicBlock *VPBB :
           VPBlockUtils::blocksOnly<const VPBasicBlock>(
               vp_depth_first_shallow(VPRB->getEntry()))) {
        for (const VPRecipeBase &Recipe : *VPBB) {
          auto *RepR = dyn_cast<VPReplicateRecipe>(&Recipe);
          if (!RepR)
            continue;
          if (!isa<StoreInst>(RepR->getUnderlyingInstr()))
            continue;
          // Check if scatter is legal for this store. If so, don't count it.
          Type *Ty = Types.inferScalarType(RepR->getOperand(0));
          auto *VTy = VectorType::get(Ty, VF);
          const Align Alignment =
              getLoadStoreAlignment(RepR->getUnderlyingInstr());
          if (!TTI.isLegalMaskedScatter(VTy, Alignment))
            ++(*NumPredStores);
        }
      }
    }
  }
  return *NumPredStores > NumberOfStoresToPredicate;
}

bool VPCostContext::isFreeScalarIntrinsic(Intrinsic::ID ID) {
  return is_contained({Intrinsic::assume, Intrinsic::lifetime_end,
                       Intrinsic::lifetime_start, Intrinsic::sideeffect,
                       Intrinsic::pseudoprobe,
                       Intrinsic::experimental_noalias_scope_decl},
                      ID);
}
```
- EN: Core entities appearing here include vp_depth_first_shallow, isFreeScalarIntrinsic, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 vp_depth_first_shallow, isFreeScalarIntrinsic，它们承载了该区间涉及的局部状态、声明或变换入口。

## Key Concepts / 关键概念

- EN: Domain: `Vectorize` focuses on vectorization and VPlan infrastructure.
  - CN: 领域：`Vectorize` 主要处理 向量化与 VPlan 基础设施。
- EN: Primary entities: `print, dump, isDefinedBy, ~VPRecipeValue, VPRecipeValue, y, setPlan, getFirstNonPhi` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`print, dump, isDefinedBy, ~VPRecipeValue, VPRecipeValue, y, setPlan, getFirstNonPhi` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AssumptionCache, DominatorTree, DomTreeUpdater, LoopInfo, OptimizationRemarkEmitter, SCEV, TargetTransformInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AssumptionCache, DominatorTree, DomTreeUpdater, LoopInfo, OptimizationRemarkEmitter, SCEV, TargetTransformInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `PrintVPlansInDotFormat` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `PrintVPlansInDotFormat` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/OptimizationRemarkEmitter.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/OptimizationRemarkEmitter.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/BasicBlock.h`, `llvm/IR/CFG.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/Type.h`, `llvm/IR/Value.h`, `llvm/Transforms/Utils/BasicBlockUtils.h`, `llvm/Transforms/Utils/LoopVersioning.h`, `llvm/Transforms/Vectorize/LoopVectorizationLegality.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/BasicBlock.h`, `llvm/IR/CFG.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/Type.h`, `llvm/IR/Value.h`, `llvm/Transforms/Utils/BasicBlockUtils.h`, `llvm/Transforms/Utils/LoopVersioning.h`, `llvm/Transforms/Vectorize/LoopVectorizationLegality.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/GraphWriter.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/PostOrderIterator.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringExtras.h`, `llvm/ADT/Twine.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/GraphWriter.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/PostOrderIterator.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringExtras.h`, `llvm/ADT/Twine.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `VPlan.h`, `LoopVectorizationPlanner.h`, `VPlanCFG.h`, `VPlanDominatorTree.h`, `VPlanHelpers.h`, `VPlanPatternMatch.h`, `VPlanTransforms.h`, `VPlanUtils.h` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`VPlan.h`, `LoopVectorizationPlanner.h`, `VPlanCFG.h`, `VPlanDominatorTree.h`, `VPlanHelpers.h`, `VPlanPatternMatch.h`, `VPlanTransforms.h`, `VPlanUtils.h` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `AssumptionCache`, `DominatorTree`, `DomTreeUpdater`, `LoopInfo`, `OptimizationRemarkEmitter`, `SCEV`, `TargetTransformInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AssumptionCache`, `DominatorTree`, `DomTreeUpdater`, `LoopInfo`, `OptimizationRemarkEmitter`, `SCEV`, `TargetTransformInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
