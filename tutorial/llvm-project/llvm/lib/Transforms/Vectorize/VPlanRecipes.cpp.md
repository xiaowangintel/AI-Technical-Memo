# VPlanRecipes.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Vectorize/VPlanRecipes.cpp` | `llvm/lib/Transforms/Vectorize/VPlanRecipes.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements implementations for VPlan recipes within LLVM's vectorization and VPlan infrastructure layer. | 该源文件实现了 LLVM 向量化与 VPlan 基础设施层中的 VPlanRecipes 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-293

```cpp
//===- VPlanRecipes.cpp - Implementations for VPlan recipes ---------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///
/// \file
/// This file contains implementations for different VPlan recipes.
///
//===----------------------------------------------------------------------===//

#include "LoopVectorizationPlanner.h"
#include "VPlan.h"
#include "VPlanAnalysis.h"
#include "VPlanHelpers.h"
#include "VPlanPatternMatch.h"
#include "VPlanUtils.h"
#include "llvm/ADT/STLExtras.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  return getParent()->getRecipeList().erase(getIterator());
}

void VPRecipeBase::moveAfter(VPRecipeBase *InsertPos) {
  removeFromParent();
  insertAfter(InsertPos);
}

void VPRecipeBase::moveBefore(VPBasicBlock &BB,
                              iplist<VPRecipeBase>::iterator I) {
  removeFromParent();
  insertBefore(BB, I);
}

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 294-588

```cpp
InstructionCost VPRecipeBase::cost(ElementCount VF, VPCostContext &Ctx) {
  // Get the underlying instruction for the recipe, if there is one. It is used
  // to
  //   * decide if cost computation should be skipped for this recipe,
  //   * apply forced target instruction cost.
  Instruction *UI = nullptr;
  if (auto *S = dyn_cast<VPSingleDefRecipe>(this))
    UI = dyn_cast_or_null<Instruction>(S->getUnderlyingValue());
  else if (auto *IG = dyn_cast<VPInterleaveBase>(this))
    UI = IG->getInsertPos();
  else if (auto *WidenMem = dyn_cast<VPWidenMemoryRecipe>(this))
    UI = &WidenMem->getIngredient();

  InstructionCost RecipeCost;
  if (UI && Ctx.skipCostComputation(UI, VF.isVector())) {
    RecipeCost = 0;
  } else {
    RecipeCost = computeCost(VF, Ctx);
    if (ForceTargetInstructionCost.getNumOccurrences() > 0 &&
        RecipeCost.isValid()) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    return true;
  default:
    return false;
  }
}

static Instruction::BinaryOps getSubRecurOpcode(RecurKind Kind) {
  if (Kind == RecurKind::Sub)
    return Instruction::Add;
  if (Kind == RecurKind::FSub)
    return Instruction::FAdd;
  llvm_unreachable("RecurKind should be Sub/FSub.");
}

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include cost, isPhi, isScalarCast, which carry the local state, declarations, or transformation entry points for this region.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 cost, isPhi, isScalarCast，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 589-877

```cpp
Value *VPInstruction::generate(VPTransformState &State) {
  IRBuilderBase &Builder = State.Builder;

  if (Instruction::isBinaryOp(getOpcode())) {
    bool OnlyFirstLaneUsed = vputils::onlyFirstLaneUsed(this);
    Value *A = State.get(getOperand(0), OnlyFirstLaneUsed);
    Value *B = State.get(getOperand(1), OnlyFirstLaneUsed);
    auto *Res =
        Builder.CreateBinOp((Instruction::BinaryOps)getOpcode(), A, B, Name);
    if (auto *I = dyn_cast<Instruction>(Res))
      applyFlags(*I);
    return Res;
  }

  switch (getOpcode()) {
  case VPInstruction::Not: {
    bool OnlyFirstLaneUsed = vputils::onlyFirstLaneUsed(this);
    Value *A = State.get(getOperand(0), OnlyFirstLaneUsed);
    return Builder.CreateNot(A, Name);
  }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  case VPInstruction::AnyOf: {
    Value *Res = Builder.CreateFreeze(State.get(getOperand(0)));
    for (VPValue *Op : drop_begin(operands()))
      Res = Builder.CreateOr(Res, Builder.CreateFreeze(State.get(Op)));
    return State.VF.isScalar() ? Res : Builder.CreateOrReduce(Res);
  }
  case VPInstruction::ExtractLane: {
    assert(getNumOperands() != 2 && "ExtractLane from single source should be "
                                    "simplified to ExtractElement.");
    Value *LaneToExtract = State.get(getOperand(0), true);
    Type *IdxTy = State.TypeAnalysis.inferScalarType(getOperand(0));
    Value *Res = nullptr;
    Value *RuntimeVF = getRuntimeVF(Builder, IdxTy, State.VF);

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 878-1174

```cpp
    for (unsigned Idx = 1; Idx != getNumOperands(); ++Idx) {
      Value *VectorStart =
          Builder.CreateMul(RuntimeVF, ConstantInt::get(IdxTy, Idx - 1));
      Value *VectorIdx = Idx == 1
                             ? LaneToExtract
                             : Builder.CreateSub(LaneToExtract, VectorStart);
      Value *Ext = State.VF.isScalar()
                       ? State.get(getOperand(Idx))
                       : Builder.CreateExtractElement(
                             State.get(getOperand(Idx)), VectorIdx);
      if (Res) {
        Value *Cmp = Builder.CreateICmpUGE(LaneToExtract, VectorStart);
        Res = Builder.CreateSelect(Cmp, Ext, Res);
      } else {
        Res = Ext;
      }
    }
    return Res;
  }
  case VPInstruction::FirstActiveLane: {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  }
  }
  llvm_unreachable("called for unsupported opcode");
}

InstructionCost VPInstruction::computeCost(ElementCount VF,
                                           VPCostContext &Ctx) const {
  if (Instruction::isBinaryOp(getOpcode())) {
    if (!getUnderlyingValue() && getOpcode() != Instruction::FMul) {
      // TODO: Compute cost for VPInstructions without underlying values once
      // the legacy cost model has been retired.
      return 0;
    }

```
- EN: Core entities appearing here include m_VPValue, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 m_VPValue，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1175-1488

```cpp
    assert(!doesGeneratePerAllLanes() &&
           "Should only generate a vector value or single scalar, not scalars "
           "for all lanes.");
    return getCostForRecipeWithOpcode(
        getOpcode(),
        vputils::onlyFirstLaneUsed(this) ? ElementCount::getFixed(1) : VF, Ctx);
  }

  switch (getOpcode()) {
  case Instruction::Select: {
    llvm::CmpPredicate Pred = CmpInst::BAD_ICMP_PREDICATE;
    match(getOperand(0), m_Cmp(Pred, m_VPValue(), m_VPValue()));
    auto *CondTy = Ctx.Types.inferScalarType(getOperand(0));
    auto *VecTy = Ctx.Types.inferScalarType(getOperand(1));
    if (!vputils::onlyFirstLaneUsed(this)) {
      CondTy = toVectorTy(CondTy, VF);
      VecTy = toVectorTy(VecTy, VF);
    }
    return Ctx.TTI.getCmpSelInstrCost(Instruction::Select, VecTy, CondTy, Pred,
                                      Ctx.CostKind);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    // Before replicating, it will have only a single operand.
    return getNumOperands() > 1;
  case VPInstruction::PtrAdd:
    return Op == getOperand(0) || vputils::onlyFirstLaneUsed(this);
  case VPInstruction::WidePtrAdd:
    // WidePtrAdd supports scalar and vector base addresses.
    return false;
  case VPInstruction::ExitingIVValue:
  case VPInstruction::ExtractLane:
    return Op == getOperand(0);
  };
  llvm_unreachable("switch should return");
}

```
- EN: Core entities appearing here include isVectorToScalar, isSingleScalar, execute, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 isVectorToScalar, isSingleScalar, execute，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1489-1781

```cpp
bool VPInstruction::usesFirstPartOnly(const VPValue *Op) const {
  assert(is_contained(operands(), Op) && "Op must be an operand of the recipe");
  if (Instruction::isBinaryOp(getOpcode()))
    return vputils::onlyFirstPartUsed(this);

  switch (getOpcode()) {
  default:
    return false;
  case Instruction::FCmp:
  case Instruction::ICmp:
  case Instruction::Select:
    return vputils::onlyFirstPartUsed(this);
  case VPInstruction::BranchOnCount:
  case VPInstruction::BranchOnCond:
  case VPInstruction::BranchOnTwoConds:
  case VPInstruction::CanonicalIVIncrementForPart:
    return true;
  };
  llvm_unreachable("switch should return");
}
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  // Advance the insert point after the wrapped IR instruction. This allows
  // interleaving VPIRInstructions and other recipes.
  State.Builder.SetInsertPoint(Phi->getParent(), std::next(Phi->getIterator()));
}

void VPPhiAccessors::removeIncomingValueFor(VPBlockBase *IncomingBlock) const {
  VPRecipeBase *R = const_cast<VPRecipeBase *>(getAsRecipe());
  assert(R->getNumOperands() == R->getParent()->getNumPredecessors() &&
         "Number of phi operands must match number of predecessors");
  unsigned Position = R->getParent()->getIndexForPredecessor(IncomingBlock);
  R->removeOperand(Position);
}

```
- EN: Core entities appearing here include usesFirstPartOnly, dump, execute, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 usesFirstPartOnly, dump, execute，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1782-2074

```cpp
VPValue *
VPPhiAccessors::getIncomingValueForBlock(const VPBasicBlock *VPBB) const {
  VPRecipeBase *R = const_cast<VPRecipeBase *>(getAsRecipe());
  return getIncomingValue(R->getParent()->getIndexForPredecessor(VPBB));
}

void VPPhiAccessors::setIncomingValueForBlock(const VPBasicBlock *VPBB,
                                              VPValue *V) const {
  VPRecipeBase *R = const_cast<VPRecipeBase *>(getAsRecipe());
  R->setOperand(R->getParent()->getIndexForPredecessor(VPBB), V);
}

#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
void VPPhiAccessors::printPhiOperands(raw_ostream &O,
                                      VPSlotTracker &SlotTracker) const {
  interleaveComma(enumerate(getAsRecipe()->operands()), O,
                  [this, &O, &SlotTracker](auto Op) {
                    O << "[ ";
                    Op.value()->printAsOperand(O, SlotTracker);
                    O << ", ";
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  });
}

#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
void VPWidenIntrinsicRecipe::printRecipe(raw_ostream &O, const Twine &Indent,
                                         VPSlotTracker &SlotTracker) const {
  O << Indent << "WIDEN-INTRINSIC ";
  if (ResultTy->isVoidTy()) {
    O << "void ";
  } else {
    printAsOperand(O, SlotTracker);
    O << " = ";
  }

```
- EN: Core entities appearing here include getIncomingValueForBlock, applyMetadata, intersect, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 getIncomingValueForBlock, applyMetadata, intersect，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2075-2364

```cpp
  O << "call";
  printFlags(O);
  O << getIntrinsicName() << "(";

  interleaveComma(operands(), O, [&O, &SlotTracker](VPValue *Op) {
    Op->printAsOperand(O, SlotTracker);
  });
  O << ")";
}
#endif

void VPHistogramRecipe::execute(VPTransformState &State) {
  IRBuilderBase &Builder = State.Builder;

  Value *Address = State.get(getOperand(0));
  Value *IncAmt = State.get(getOperand(1), /*IsScalar=*/true);
  VectorType *VTy = cast<VectorType>(Address->getType());

  // The histogram intrinsic requires a mask even if the recipe doesn't;
  // if the mask operand was omitted then all lanes should be executed and
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    OS << "fmuladd";
    break;
  case RecurKind::AnyOf:
    OS << "any-of";
    break;
  case RecurKind::FindIV:
    OS << "find-iv";
    break;
  case RecurKind::FindLast:
    OS << "find-last";
    break;
  }
}

```
- EN: Core entities appearing here include interleaveComma, execute, FastMathFlagsTy, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 interleaveComma, execute, FastMathFlagsTy，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2365-2657

```cpp
void VPIRFlags::printFlags(raw_ostream &O) const {
  switch (OpType) {
  case OperationType::Cmp:
    O << " " << CmpInst::getPredicateName(getPredicate());
    break;
  case OperationType::FCmp:
    O << " " << CmpInst::getPredicateName(getPredicate());
    getFastMathFlags().print(O);
    break;
  case OperationType::DisjointOp:
    if (DisjointFlags.IsDisjoint)
      O << " disjoint";
    break;
  case OperationType::PossiblyExactOp:
    if (ExactFlags.IsExact)
      O << " exact";
    break;
  case OperationType::OverflowingBinOp:
    if (WrapFlags.HasNUW)
      O << " nuw";
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
void VPDerivedIVRecipe::printRecipe(raw_ostream &O, const Twine &Indent,
                                    VPSlotTracker &SlotTracker) const {
  O << Indent;
  printAsOperand(O, SlotTracker);
  O << " = DERIVED-IV ";
  getStartValue()->printAsOperand(O, SlotTracker);
  O << " + ";
  getOperand(1)->printAsOperand(O, SlotTracker);
  O << " * ";
  getStepValue()->printAsOperand(O, SlotTracker);
}
#endif

```
- EN: Core entities appearing here include printFlags, execute, isCanonical, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 printFlags, execute, isCanonical，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2658-2968

```cpp
void VPScalarIVStepsRecipe::execute(VPTransformState &State) {
  // Fast-math-flags propagate from the original induction instruction.
  IRBuilder<>::FastMathFlagGuard FMFG(State.Builder);
  State.Builder.setFastMathFlags(getFastMathFlags());

  /// Compute scalar induction steps. \p ScalarIV is the scalar induction
  /// variable on which to base the steps, \p Step is the size of the step.

  Value *BaseIV = State.get(getOperand(0), VPLane(0));
  Value *Step = State.get(getStepValue(), VPLane(0));
  IRBuilderBase &Builder = State.Builder;

  // Ensure step has the same type as that of scalar IV.
  Type *BaseIVTy = BaseIV->getType()->getScalarType();
  assert(BaseIVTy == Step->getType() && "Types of BaseIV and Step must match!");

  // We build scalar steps for both integer and floating-point induction
  // variables. Here, we determine the kind of arithmetic we will perform.
  Instruction::BinaryOps AddOp;
  Instruction::BinaryOps MulOp;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    assert(isInLoop() &&
           "The reduction must either be ordered, partial or in-loop");
    Value *PrevInChain = State.get(getChainOp(), /*IsScalar*/ true);
    NewRed = createSimpleReduction(State.Builder, NewVecOp, Kind);
    if (RecurrenceDescriptor::isMinMaxRecurrenceKind(Kind))
      NextInChain = createMinMaxOp(State.Builder, Kind, NewRed, PrevInChain);
    else
      NextInChain = State.Builder.CreateBinOp(
          (Instruction::BinaryOps)RecurrenceDescriptor::getOpcode(Kind),
          PrevInChain, NewRed);
  }
  State.set(this, NextInChain, /*IsScalar*/ !isPartialReduction());
}

```
- EN: Core entities appearing here include execute, usesFirstLaneOnly, materializeOffset, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 execute, usesFirstLaneOnly, materializeOffset，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2969-3291

```cpp
void VPReductionEVLRecipe::execute(VPTransformState &State) {

  auto &Builder = State.Builder;
  // Propagate the fast-math flags carried by the underlying instruction.
  IRBuilderBase::FastMathFlagGuard FMFGuard(Builder);
  Builder.setFastMathFlags(getFastMathFlags());

  RecurKind Kind = getRecurrenceKind();
  Value *Prev = State.get(getChainOp(), /*IsScalar*/ true);
  Value *VecOp = State.get(getVecOp());
  Value *EVL = State.get(getEVL(), VPLane(0));

  Value *Mask;
  if (VPValue *CondOp = getCondOp())
    Mask = State.get(CondOp);
  else
    Mask = Builder.CreateVectorSplat(State.VF, Builder.getTrue());

  Value *NewRed;
  if (isOrdered()) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      auto *Ext1 = cast<VPWidenCastRecipe>(ExpressionRecipes[1]);
      O << " " << Instruction::getOpcodeName(Ext1->getOpcode()) << " to "
        << *Ext1->getResultType() << ")";
    }
    if (Red->isConditional()) {
      O << ", ";
      Red->getCondOp()->printAsOperand(O, SlotTracker);
    }
    O << ")";
    break;
  }
  }
}

```
- EN: Core entities appearing here include execute, ExpressionRecipes, any_of, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 execute, ExpressionRecipes, any_of，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 3292-3608

```cpp
void VPReductionRecipe::printRecipe(raw_ostream &O, const Twine &Indent,
                                    VPSlotTracker &SlotTracker) const {
  if (isPartialReduction())
    O << Indent << "PARTIAL-REDUCE ";
  else
    O << Indent << "REDUCE ";
  printAsOperand(O, SlotTracker);
  O << " = ";
  getChainOp()->printAsOperand(O, SlotTracker);
  O << " +";
  printFlags(O);
  O << " reduce.";
  printRecurrenceKind(O, getRecurrenceKind());
  O << " (";
  getVecOp()->printAsOperand(O, SlotTracker);
  if (isConditional()) {
    O << ", ";
    getCondOp()->printAsOperand(O, SlotTracker);
  }
  O << ")";
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  case Instruction::UIToFP:
  case Instruction::Trunc:
  case Instruction::FPTrunc:
  case Instruction::Select:
  case Instruction::AddrSpaceCast: {
    return getCostForRecipeWithOpcode(getOpcode(), ElementCount::getFixed(1),
                                      Ctx) *
           (isSingleScalar() ? 1 : VF.getFixedValue());
  }
  case Instruction::ExtractValue:
  case Instruction::InsertValue:
    return Ctx.TTI.getInsertExtractValueCost(getOpcode(), Ctx.CostKind);
  }

```
- EN: Core entities appearing here include execute, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as ScalarEvolution, SCEV, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 execute，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 ScalarEvolution, SCEV, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 3609-3907

```cpp
  return Ctx.getLegacyCost(UI, VF);
}

InstructionCost VPReplicateRecipe::computeCallCost(
    Function *CalledFn, Type *ResultTy, ArrayRef<const VPValue *> ArgOps,
    bool IsSingleScalar, ElementCount VF, VPCostContext &Ctx) {
  SmallVector<Type *, 4> Tys = map_to_vector<4>(
      ArgOps, [&](const VPValue *Op) { return Ctx.Types.inferScalarType(Op); });

  Intrinsic::ID IntrinID = CalledFn->getIntrinsicID();
  auto GetIntrinsicCost = [&] {
    if (!IntrinID)
      return InstructionCost::getInvalid();
    return Ctx.TTI.getIntrinsicInstrCost(
        IntrinsicCostAttributes(IntrinID, ResultTy, Tys), Ctx.CostKind);
  };

  if (IntrinID && VPCostContext::isFreeScalarIntrinsic(IntrinID)) {
    assert(GetIntrinsicCost() == 0 && "scalarizing intrinsic should be free");
    return 0;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  VPValue *StoredValue = getStoredValue();
  bool CreateScatter = !isConsecutive();

  auto &Builder = State.Builder;

  CallInst *NewSI = nullptr;
  Value *StoredVal = State.get(StoredValue);
  Value *EVL = State.get(getEVL(), VPLane(0));
  Value *Mask = nullptr;
  if (VPValue *VPMask = getMask())
    Mask = State.get(VPMask);
  else
    Mask = Builder.CreateVectorSplat(State.VF, Builder.getTrue());

```
- EN: Core entities appearing here include execute, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 execute，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 3908-4198

```cpp
  Value *Addr = State.get(getAddr(), !CreateScatter);
  if (CreateScatter) {
    NewSI = Builder.CreateIntrinsic(Type::getVoidTy(EVL->getContext()),
                                    Intrinsic::vp_scatter,
                                    {StoredVal, Addr, Mask, EVL});
  } else {
    NewSI = Builder.CreateIntrinsic(Type::getVoidTy(EVL->getContext()),
                                    Intrinsic::vp_store,
                                    {StoredVal, Addr, Mask, EVL});
  }
  NewSI->addParamAttr(
      1, Attribute::getWithAlignment(NewSI->getContext(), Alignment));
  applyMetadata(*NewSI);
}

InstructionCost VPWidenStoreEVLRecipe::computeCost(ElementCount VF,
                                                   VPCostContext &Ctx) const {
  if (!Consecutive || IsMasked)
    return VPWidenMemoryRecipe::computeCost(VF, Ctx);

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    StoredVecs.push_back(StoredVec);
  }

  // Interleave all the smaller vectors into one wider vector.
  Value *IVec = interleaveVectors(State.Builder, StoredVecs, "interleaved.vec");
  Instruction *NewStoreInstr;
  if (BlockInMask || MaskForGaps) {
    Value *GroupMask = CreateGroupMask(MaskForGaps);
    NewStoreInstr = State.Builder.CreateMaskedStore(
        IVec, ResAddr, Group->getAlign(), GroupMask);
  } else
    NewStoreInstr =
        State.Builder.CreateAlignedStore(IVec, ResAddr, Group->getAlign());

```
- EN: Core entities appearing here include execute, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 execute，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 4199-4491

```cpp
  applyMetadata(*NewStoreInstr);
  // TODO: Also manage existing metadata using VPIRMetadata.
  Group->addMetadata(NewStoreInstr);
}

#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
void VPInterleaveRecipe::printRecipe(raw_ostream &O, const Twine &Indent,
                                     VPSlotTracker &SlotTracker) const {
  const InterleaveGroup<Instruction> *IG = getInterleaveGroup();
  O << Indent << "INTERLEAVE-GROUP with factor " << IG->getFactor() << " at ";
  IG->getInsertPos()->printAsOperand(O, false);
  O << ", ";
  getAddr()->printAsOperand(O, SlotTracker);
  VPValue *Mask = getMask();
  if (Mask) {
    O << ", ";
    Mask->printAsOperand(O, SlotTracker);
  }

  unsigned OpIdx = 0;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  O << " = WIDEN-CANONICAL-INDUCTION ";
  printOperands(O, SlotTracker);
}
#endif

void VPFirstOrderRecurrencePHIRecipe::execute(VPTransformState &State) {
  auto &Builder = State.Builder;
  // Create a vector from the initial value.
  auto *VectorInit = getStartValue()->getLiveInIRValue();

  Type *VecTy = State.VF.isScalar()
                    ? VectorInit->getType()
                    : VectorType::get(VectorInit->getType(), State.VF);

```
- EN: Core entities appearing here include execute, onlyScalarsGenerated, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, SCEV, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 execute, onlyScalarsGenerated，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, SCEV, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 4492-4630

```cpp
  BasicBlock *VectorPH =
      State.CFG.VPBB2IRBB.at(getParent()->getCFGPredecessor(0));
  if (State.VF.isVector()) {
    auto *IdxTy = Builder.getInt32Ty();
    auto *One = ConstantInt::get(IdxTy, 1);
    IRBuilder<>::InsertPointGuard Guard(Builder);
    Builder.SetInsertPoint(VectorPH->getTerminator());
    auto *RuntimeVF = getRuntimeVF(Builder, IdxTy, State.VF);
    auto *LastIdx = Builder.CreateSub(RuntimeVF, One);
    VectorInit = Builder.CreateInsertElement(
        PoisonValue::get(VecTy), VectorInit, LastIdx, "vector.recur.init");
  }

  // Create a phi node for the new recurrence.
  PHINode *Phi = PHINode::Create(VecTy, 2, "vector.recur");
  Phi->insertBefore(State.CFG.PrevBB->getFirstInsertionPt());
  Phi->addIncoming(VectorInit, VectorPH);
  State.set(this, Phi);
}

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  printOperands(O, SlotTracker);
}
#endif

#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
void VPCurrentIterationPHIRecipe::printRecipe(
    raw_ostream &O, const Twine &Indent, VPSlotTracker &SlotTracker) const {
  O << Indent << "CURRENT-ITERATION-PHI ";

  printAsOperand(O, SlotTracker);
  O << " = phi ";
  printOperands(O, SlotTracker);
}
#endif
```
- EN: Core entities appearing here include execute, usesFirstLaneOnly, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 execute, usesFirstLaneOnly，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Vectorize` focuses on vectorization and VPlan infrastructure.
  - CN: 领域：`Vectorize` 主要处理 向量化与 VPlan 基础设施。
- EN: Primary entities: `mayWriteToMemory, mayReadFromMemory, mayHaveSideEffects, isSafeToSpeculativelyExecute, insertBefore, insertAfter, removeFromParent, eraseFromParent` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`mayWriteToMemory, mayReadFromMemory, mayHaveSideEffects, isSafeToSpeculativelyExecute, insertBefore, insertAfter, removeFromParent, eraseFromParent` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AssumptionCache, DataLayout, LoopInfo, ScalarEvolution, SCEV, TargetTransformInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AssumptionCache, DataLayout, LoopInfo, ScalarEvolution, SCEV, TargetTransformInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/AssumptionCache.h`, `llvm/Analysis/IVDescriptors.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/ScalarEvolutionExpressions.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/AssumptionCache.h`, `llvm/Analysis/IVDescriptors.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/ScalarEvolutionExpressions.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/BasicBlock.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/Intrinsics.h`, `llvm/IR/Type.h`, `llvm/IR/Value.h`, `llvm/Transforms/Utils/BasicBlockUtils.h`, `llvm/Transforms/Utils/LoopUtils.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/BasicBlock.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/Intrinsics.h`, `llvm/IR/Type.h`, `llvm/IR/Value.h`, `llvm/Transforms/Utils/BasicBlockUtils.h`, `llvm/Transforms/Utils/LoopUtils.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/SmallVectorExtras.h`, `llvm/ADT/Twine.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/SmallVectorExtras.h`, `llvm/ADT/Twine.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `LoopVectorizationPlanner.h`, `VPlan.h`, `VPlanAnalysis.h`, `VPlanHelpers.h`, `VPlanPatternMatch.h`, `VPlanUtils.h`, `cassert` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`LoopVectorizationPlanner.h`, `VPlan.h`, `VPlanAnalysis.h`, `VPlanHelpers.h`, `VPlanPatternMatch.h`, `VPlanUtils.h`, `cassert` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `AssumptionCache`, `DataLayout`, `LoopInfo`, `ScalarEvolution`, `SCEV`, `TargetTransformInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AssumptionCache`, `DataLayout`, `LoopInfo`, `ScalarEvolution`, `SCEV`, `TargetTransformInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
