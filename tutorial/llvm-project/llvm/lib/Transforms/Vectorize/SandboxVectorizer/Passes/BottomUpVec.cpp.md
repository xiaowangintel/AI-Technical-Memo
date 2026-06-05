# BottomUpVec.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Vectorize/SandboxVectorizer/Passes/BottomUpVec.cpp` | `llvm/lib/Transforms/Vectorize/SandboxVectorizer/Passes/BottomUpVec.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements a bottom-up vectorizer pass within LLVM's vectorization and VPlan infrastructure layer. | 该源文件实现了 LLVM 向量化与 VPlan 基础设施层中的 BottomUpVec 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-52

```cpp
//===- BottomUpVec.cpp - A bottom-up vectorizer pass ----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Vectorize/SandboxVectorizer/Passes/BottomUpVec.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/SandboxIR/Function.h"
#include "llvm/SandboxIR/Instruction.h"
#include "llvm/SandboxIR/Module.h"
#include "llvm/SandboxIR/Region.h"
#include "llvm/SandboxIR/Utils.h"
#include "llvm/Transforms/Vectorize/SandboxVectorizer/Debug.h"
#include "llvm/Transforms/Vectorize/SandboxVectorizer/VecUtils.h"

namespace llvm {

#ifndef NDEBUG
static cl::opt<bool>
    AlwaysVerify("sbvec-always-verify", cl::init(false), cl::Hidden,
                 cl::desc("Helps find bugs by verifying the IR whenever we "
                          "emit new instructions (*very* expensive)."));
#endif // NDEBUG

static constexpr unsigned long StopAtDisabled =
    std::numeric_limits<unsigned long>::max();
static cl::opt<unsigned long>
    StopAt("sbvec-stop-at", cl::init(StopAtDisabled), cl::Hidden,
           cl::desc("Vectorize if the invocation count is < than this. 0 "
                    "disables vectorization."));

static constexpr unsigned long StopBundleDisabled =
    std::numeric_limits<unsigned long>::max();
static cl::opt<unsigned long>
    StopBundle("sbvec-stop-bndl", cl::init(StopBundleDisabled), cl::Hidden,
               cl::desc("Vectorize up to this many bundles."));

namespace sandboxir {

static SmallVector<Value *, 4> getOperand(ArrayRef<Value *> Bndl,
                                          unsigned OpIdx) {
  SmallVector<Value *, 4> Operands;
  for (Value *BndlV : Bndl) {
    auto *BndlI = cast<Instruction>(BndlV);
    Operands.push_back(BndlI->getOperand(OpIdx));
  }
  return Operands;
}

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 53-99

```cpp
/// \Returns the BB iterator after the lowest instruction in \p Vals, or the top
/// of BB if no instruction found in \p Vals.
static BasicBlock::iterator getInsertPointAfterInstrs(ArrayRef<Value *> Vals,
                                                      BasicBlock *BB) {
  auto *BotI = VecUtils::getLastPHIOrSelf(VecUtils::getLowest(Vals, BB));
  if (BotI == nullptr)
    // We are using BB->begin() (or after PHIs) as the fallback insert point.
    return BB->empty()
               ? BB->begin()
               : std::next(
                     VecUtils::getLastPHIOrSelf(&*BB->begin())->getIterator());
  return std::next(BotI->getIterator());
}

Value *BottomUpVec::createVectorInstr(ArrayRef<Value *> Bndl,
                                      ArrayRef<Value *> Operands) {
  auto CreateVectorInstr = [](ArrayRef<Value *> Bndl,
                              ArrayRef<Value *> Operands) -> Value * {
    assert(all_of(Bndl, [](auto *V) { return isa<Instruction>(V); }) &&
           "Expect Instructions!");
    auto &Ctx = Bndl[0]->getContext();

    Type *ScalarTy = VecUtils::getElementType(Utils::getExpectedType(Bndl[0]));
    auto *VecTy = VecUtils::getWideType(ScalarTy, VecUtils::getNumLanes(Bndl));

    BasicBlock::iterator WhereIt = getInsertPointAfterInstrs(
        Bndl, cast<Instruction>(Bndl[0])->getParent());

    auto Opcode = cast<Instruction>(Bndl[0])->getOpcode();
    switch (Opcode) {
    case Instruction::Opcode::ZExt:
    case Instruction::Opcode::SExt:
    case Instruction::Opcode::FPToUI:
    case Instruction::Opcode::FPToSI:
    case Instruction::Opcode::FPExt:
    case Instruction::Opcode::PtrToInt:
    case Instruction::Opcode::IntToPtr:
    case Instruction::Opcode::SIToFP:
    case Instruction::Opcode::UIToFP:
    case Instruction::Opcode::Trunc:
    case Instruction::Opcode::FPTrunc:
    case Instruction::Opcode::BitCast: {
      assert(Operands.size() == 1u && "Casts are unary!");
      return CastInst::create(VecTy, Opcode, Operands[0], WhereIt, Ctx,
                              "VCast");
    }
    case Instruction::Opcode::FCmp:
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 100-146

```cpp
    case Instruction::Opcode::ICmp: {
      auto Pred = cast<CmpInst>(Bndl[0])->getPredicate();
      assert(all_of(drop_begin(Bndl),
                    [Pred](auto *SBV) {
                      return cast<CmpInst>(SBV)->getPredicate() == Pred;
                    }) &&
             "Expected same predicate across bundle.");
      return CmpInst::create(Pred, Operands[0], Operands[1], WhereIt, Ctx,
                             "VCmp");
    }
    case Instruction::Opcode::Select: {
      return SelectInst::create(Operands[0], Operands[1], Operands[2], WhereIt,
                                Ctx, "Vec");
    }
    case Instruction::Opcode::FNeg: {
      auto *UOp0 = cast<UnaryOperator>(Bndl[0]);
      auto OpC = UOp0->getOpcode();
      return UnaryOperator::createWithCopiedFlags(OpC, Operands[0], UOp0,
                                                  WhereIt, Ctx, "Vec");
    }
    case Instruction::Opcode::Add:
    case Instruction::Opcode::FAdd:
    case Instruction::Opcode::Sub:
    case Instruction::Opcode::FSub:
    case Instruction::Opcode::Mul:
    case Instruction::Opcode::FMul:
    case Instruction::Opcode::UDiv:
    case Instruction::Opcode::SDiv:
    case Instruction::Opcode::FDiv:
    case Instruction::Opcode::URem:
    case Instruction::Opcode::SRem:
    case Instruction::Opcode::FRem:
    case Instruction::Opcode::Shl:
    case Instruction::Opcode::LShr:
    case Instruction::Opcode::AShr:
    case Instruction::Opcode::And:
    case Instruction::Opcode::Or:
    case Instruction::Opcode::Xor: {
      auto *BinOp0 = cast<BinaryOperator>(Bndl[0]);
      auto *LHS = Operands[0];
      auto *RHS = Operands[1];
      return BinaryOperator::createWithCopiedFlags(
          BinOp0->getOpcode(), LHS, RHS, BinOp0, WhereIt, Ctx, "Vec");
    }
    case Instruction::Opcode::Load: {
      auto *Ld0 = cast<LoadInst>(Bndl[0]);
      Value *Ptr = Ld0->getPointerOperand();
```
- EN: This region continues the BottomUpVec implementation with local helper logic centered on Instruction, Opcode, ICmp, Pred.
- CN: 这一段延续了 BottomUpVec 的主体实现，围绕 Instruction, Opcode, ICmp, Pred 等局部辅助逻辑展开。

### Lines 147-197

```cpp
      return LoadInst::create(VecTy, Ptr, Ld0->getAlign(), WhereIt, Ctx,
                              "VecL");
    }
    case Instruction::Opcode::Store: {
      auto Align = cast<StoreInst>(Bndl[0])->getAlign();
      Value *Val = Operands[0];
      Value *Ptr = Operands[1];
      return StoreInst::create(Val, Ptr, Align, WhereIt, Ctx);
    }
    case Instruction::Opcode::UncondBr:
    case Instruction::Opcode::CondBr:
    case Instruction::Opcode::Ret:
    case Instruction::Opcode::PHI:
    case Instruction::Opcode::AddrSpaceCast:
    case Instruction::Opcode::Call:
    case Instruction::Opcode::GetElementPtr:
      llvm_unreachable("Unimplemented");
      break;
    default:
      llvm_unreachable("Unimplemented");
      break;
    }
    llvm_unreachable("Missing switch case!");
    // TODO: Propagate debug info.
  };

  auto *NewI = CreateVectorInstr(Bndl, Operands);
  LLVM_DEBUG(dbgs() << DEBUG_PREFIX << "New instr: " << *NewI << "\n");
  return NewI;
}

void BottomUpVec::tryEraseDeadInstrs() {
  DenseMap<BasicBlock *, SmallVector<Instruction *>> SortedDeadInstrCandidates;
  // The dead instrs could span BBs, so we need to collect and sort them per BB.
  for (auto *DeadI : DeadInstrCandidates)
    SortedDeadInstrCandidates[DeadI->getParent()].push_back(DeadI);
  for (auto &Pair : SortedDeadInstrCandidates)
    sort(Pair.second,
         [](Instruction *I1, Instruction *I2) { return I1->comesBefore(I2); });
  for (const auto &Pair : SortedDeadInstrCandidates) {
    for (Instruction *I : reverse(Pair.second)) {
      if (I->hasNUses(0)) {
        // Erase the dead instructions bottom-to-top.
        LLVM_DEBUG(dbgs() << DEBUG_PREFIX << "Erase dead: " << *I << "\n");
        I->eraseFromParent();
      }
    }
  }
  DeadInstrCandidates.clear();
}

```
- EN: Core entities appearing here include tryEraseDeadInstrs, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 tryEraseDeadInstrs，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 198-244

```cpp
Value *BottomUpVec::createShuffle(Value *VecOp, const ShuffleMask &Mask,
                                  BasicBlock *UserBB) {
  BasicBlock::iterator WhereIt = getInsertPointAfterInstrs({VecOp}, UserBB);
  return ShuffleVectorInst::create(VecOp, VecOp, Mask, WhereIt,
                                   VecOp->getContext(), "VShuf");
}

Value *BottomUpVec::createPack(ArrayRef<Value *> ToPack, BasicBlock *UserBB) {
  BasicBlock::iterator WhereIt = getInsertPointAfterInstrs(ToPack, UserBB);

  Type *ScalarTy = VecUtils::getCommonScalarType(ToPack);
  unsigned Lanes = VecUtils::getNumLanes(ToPack);
  Type *VecTy = VecUtils::getWideType(ScalarTy, Lanes);

  // Create a series of pack instructions.
  Value *LastInsert = PoisonValue::get(VecTy);

  Context &Ctx = ToPack[0]->getContext();

  unsigned InsertIdx = 0;
  for (Value *Elm : ToPack) {
    // An element can be either scalar or vector. We need to generate different
    // IR for each case.
    if (Elm->getType()->isVectorTy()) {
      unsigned NumElms =
          cast<FixedVectorType>(Elm->getType())->getNumElements();
      for (auto ExtrLane : seq<int>(0, NumElms)) {
        // We generate extract-insert pairs, for each lane in `Elm`.
        Constant *ExtrLaneC =
            ConstantInt::getSigned(Type::getInt32Ty(Ctx), ExtrLane);
        // This may return a Constant if Elm is a Constant.
        auto *ExtrI =
            ExtractElementInst::create(Elm, ExtrLaneC, WhereIt, Ctx, "VPack");
        if (!isa<Constant>(ExtrI))
          WhereIt = std::next(cast<Instruction>(ExtrI)->getIterator());
        Constant *InsertLaneC =
            ConstantInt::getSigned(Type::getInt32Ty(Ctx), InsertIdx++);
        // This may also return a Constant if ExtrI is a Constant.
        auto *InsertI = InsertElementInst::create(
            LastInsert, ExtrI, InsertLaneC, WhereIt, Ctx, "VPack");
        LastInsert = InsertI;
        if (!isa<Constant>(InsertI))
          WhereIt = std::next(cast<Instruction>(LastInsert)->getIterator());
      }
    } else {
      Constant *InsertLaneC =
          ConstantInt::getSigned(Type::getInt32Ty(Ctx), InsertIdx++);
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 245-291

```cpp
      // This may be folded into a Constant if LastInsert is a Constant. In
      // that case we only collect the last constant.
      LastInsert = InsertElementInst::create(LastInsert, Elm, InsertLaneC,
                                             WhereIt, Ctx, "Pack");
      if (auto *NewI = dyn_cast<Instruction>(LastInsert))
        WhereIt = std::next(NewI->getIterator());
    }
  }
  return LastInsert;
}

void BottomUpVec::collectPotentiallyDeadInstrs(ArrayRef<Value *> Bndl) {
  for (Value *V : Bndl)
    DeadInstrCandidates.insert(cast<Instruction>(V));
  // Also collect the GEPs of vectorized loads and stores.
  auto Opcode = cast<Instruction>(Bndl[0])->getOpcode();
  switch (Opcode) {
  case Instruction::Opcode::Load: {
    for (Value *V : drop_begin(Bndl))
      if (auto *Ptr =
              dyn_cast<Instruction>(cast<LoadInst>(V)->getPointerOperand()))
        DeadInstrCandidates.insert(Ptr);
    break;
  }
  case Instruction::Opcode::Store: {
    for (Value *V : drop_begin(Bndl))
      if (auto *Ptr =
              dyn_cast<Instruction>(cast<StoreInst>(V)->getPointerOperand()))
        DeadInstrCandidates.insert(Ptr);
    break;
  }
  default:
    break;
  }
}

Action *BottomUpVec::vectorizeRec(ArrayRef<Value *> Bndl,
                                  ArrayRef<Value *> UserBndl, unsigned Depth,
                                  LegalityAnalysis &Legality) {
  bool StopForDebug =
      DebugBndlCnt++ >= StopBundle && StopBundle != StopBundleDisabled;
  LLVM_DEBUG(dbgs() << DEBUG_PREFIX << "canVectorize() Bundle:\n";
             VecUtils::dump(Bndl));
  const auto &LegalityRes = StopForDebug ? Legality.getForcedPackForDebugging()
                                         : Legality.canVectorize(Bndl);
  LLVM_DEBUG(dbgs() << DEBUG_PREFIX << "Legality: " << LegalityRes << "\n");
  auto ActionPtr =
```
- EN: Core entities appearing here include collectPotentiallyDeadInstrs, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 collectPotentiallyDeadInstrs，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 292-342

```cpp
      std::make_unique<Action>(&LegalityRes, Bndl, UserBndl, Depth);
  SmallVector<Action *> Operands;
  switch (LegalityRes.getSubclassID()) {
  case LegalityResultID::Widen: {
    auto *I = cast<Instruction>(Bndl[0]);
    switch (I->getOpcode()) {
    case Instruction::Opcode::Load:
      break;
    case Instruction::Opcode::Store: {
      // Don't recurse towards the pointer operand.
      Action *OpA =
          vectorizeRec(getOperand(Bndl, 0), Bndl, Depth + 1, Legality);
      Operands.push_back(OpA);
      break;
    }
    default:
      // Visit all operands.
      for (auto OpIdx : seq<unsigned>(I->getNumOperands())) {
        Action *OpA =
            vectorizeRec(getOperand(Bndl, OpIdx), Bndl, Depth + 1, Legality);
        Operands.push_back(OpA);
      }
      break;
    }
    // Update the maps to mark Bndl as "vectorized".
    IMaps->registerVector(Bndl, ActionPtr.get());
    break;
  }
  case LegalityResultID::DiamondReuse:
  case LegalityResultID::DiamondReuseWithShuffle:
  case LegalityResultID::DiamondReuseMultiInput:
  case LegalityResultID::Pack:
    break;
  }
  // Create actions in post-order.
  ActionPtr->Operands = std::move(Operands);
  auto *Action = ActionPtr.get();
  Actions.push_back(std::move(ActionPtr));
  return Action;
}

#ifndef NDEBUG
void BottomUpVec::ActionsVector::print(raw_ostream &OS) const {
  for (auto [Idx, Action] : enumerate(Actions)) {
    Action->print(OS);
    OS << "\n";
  }
}
void BottomUpVec::ActionsVector::dump() const { print(dbgs()); }
#endif // NDEBUG

```
- EN: Core entities appearing here include print, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 print，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 343-385

```cpp
void BottomUpVec::emitUnpacksForExternalUses(const ArrayRef<Value *> Bndl,
                                             Value *Vec) {
  // Find where we should emit the unpacks.
  BasicBlock::iterator WhereIt;
  if (auto *VecI = dyn_cast<Instruction>(Vec)) {
    WhereIt = std::next(VecI->getIterator());
  } else {
    // If Vec is a constant then it should be safe to emit the unpacks at the
    // top of the block.
    // Note: Extracts from constants are usually folded to constants.
    assert(isa<Constant>(Vec) && "Expected constant!");
    assert(isa<Instruction>(Bndl[0]) &&
           "A widened Bndl should contain instrs!");
    BasicBlock *BB = cast<Instruction>(Bndl[0])->getParent();
    WhereIt =
        BB->empty()
            ? BB->begin()
            : std::next(
                  VecUtils::getLastPHIOrSelf(&*BB->begin())->getIterator());
  }

  for (auto [Lane, Elm] : VecUtils::enumerateLanes(Bndl)) {
    for (User *U : Elm->users()) {
      // Skip users that we just vectorized.
      if (IMaps->isVectorized(U))
        continue;
      auto *LastUnpackV = VecUtils::unpack(Vec, Elm->getType(), Lane, WhereIt);
      Elm->replaceAllUsesWith(LastUnpackV);
    }
  }
}

Value *BottomUpVec::emitVectors() {
  Value *NewVec = nullptr;
  for (const auto &ActionPtr : Actions) {
    ArrayRef<Value *> Bndl = ActionPtr->Bndl;
    ArrayRef<Value *> UserBndl = ActionPtr->UserBndl;
    const LegalityResult &LegalityRes = *ActionPtr->LegalityRes;
    unsigned Depth = ActionPtr->Depth;
    auto *UserBB = !UserBndl.empty()
                       ? cast<Instruction>(UserBndl.front())->getParent()
                       : cast<Instruction>(Bndl[0])->getParent();

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 386-434

```cpp
    switch (LegalityRes.getSubclassID()) {
    case LegalityResultID::Widen: {
      auto *I = cast<Instruction>(Bndl[0]);
      SmallVector<Value *, 2> VecOperands;
      switch (I->getOpcode()) {
      case Instruction::Opcode::Load:
        VecOperands.push_back(cast<LoadInst>(I)->getPointerOperand());
        break;
      case Instruction::Opcode::Store: {
        VecOperands.push_back(ActionPtr->Operands[0]->Vec);
        VecOperands.push_back(cast<StoreInst>(I)->getPointerOperand());
        break;
      }
      default:
        // Visit all operands.
        for (Action *OpA : ActionPtr->Operands) {
          auto *VecOp = OpA->Vec;
          VecOperands.push_back(VecOp);
        }
        break;
      }
      NewVec = createVectorInstr(ActionPtr->Bndl, VecOperands);
      // Collect any potentially dead scalar instructions, including the
      // original scalars and pointer operands of loads/stores.
      if (NewVec != nullptr)
        collectPotentiallyDeadInstrs(Bndl);

      // Emit unpacks for all external uses, if any.
      emitUnpacksForExternalUses(ActionPtr->Bndl, NewVec);
      break;
    }
    case LegalityResultID::DiamondReuse: {
      NewVec = cast<DiamondReuse>(LegalityRes).getVector()->Vec;
      break;
    }
    case LegalityResultID::DiamondReuseWithShuffle: {
      auto *VecOp = cast<DiamondReuseWithShuffle>(LegalityRes).getVector()->Vec;
      const ShuffleMask &Mask =
          cast<DiamondReuseWithShuffle>(LegalityRes).getMask();
      NewVec = createShuffle(VecOp, Mask, UserBB);
      assert(NewVec->getType() == VecOp->getType() &&
             "Expected same type! Bad mask ?");
      break;
    }
    case LegalityResultID::DiamondReuseMultiInput: {
      const auto &Descr =
          cast<DiamondReuseMultiInput>(LegalityRes).getCollectDescr();
      Type *ResTy = VecUtils::getWideType(Bndl[0]->getType(), Bndl.size());

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 435-481

```cpp
      // TODO: Try to get WhereIt without creating a vector.
      SmallVector<Value *, 4> DescrInstrs;
      for (const auto &ElmDescr : Descr.getDescrs()) {
        auto *V = ElmDescr.needsExtract() ? ElmDescr.getValue()->Vec
                                          : ElmDescr.getScalar();
        if (auto *I = dyn_cast<Instruction>(V))
          DescrInstrs.push_back(I);
      }
      BasicBlock::iterator WhereIt =
          getInsertPointAfterInstrs(DescrInstrs, UserBB);

      Value *LastV = PoisonValue::get(ResTy);
      Context &Ctx = LastV->getContext();
      unsigned Lane = 0;
      for (const auto &ElmDescr : Descr.getDescrs()) {
        Value *VecOp = nullptr;
        Value *ValueToInsert;
        if (ElmDescr.needsExtract()) {
          VecOp = ElmDescr.getValue()->Vec;
          ConstantInt *IdxC =
              ConstantInt::get(Type::getInt32Ty(Ctx), ElmDescr.getExtractIdx());
          ValueToInsert = ExtractElementInst::create(
              VecOp, IdxC, WhereIt, VecOp->getContext(), "VExt");
        } else {
          ValueToInsert = ElmDescr.getScalar();
        }
        auto NumLanesToInsert = VecUtils::getNumLanes(ValueToInsert);
        if (NumLanesToInsert == 1) {
          // If we are inserting a scalar element then we need a single insert.
          //   %VIns = insert %DstVec,  %SrcScalar, Lane
          ConstantInt *LaneC = ConstantInt::get(Type::getInt32Ty(Ctx), Lane);
          LastV = InsertElementInst::create(LastV, ValueToInsert, LaneC,
                                            WhereIt, Ctx, "VIns");
        } else {
          // If we are inserting a vector element then we need to extract and
          // insert each vector element one by one with a chain of extracts and
          // inserts, for example:
          //   %VExt0 = extract %SrcVec, 0
          //   %VIns0 = insert  %DstVec, %Vect0, Lane + 0
          //   %VExt1 = extract %SrcVec, 1
          //   %VIns1 = insert  %VIns0,  %Vect0, Lane + 1
          for (unsigned LnCnt = 0; LnCnt != NumLanesToInsert; ++LnCnt) {
            auto *ExtrIdxC = ConstantInt::get(Type::getInt32Ty(Ctx), LnCnt);
            auto *ExtrI = ExtractElementInst::create(ValueToInsert, ExtrIdxC,
                                                     WhereIt, Ctx, "VExt");
            unsigned InsLane = Lane + LnCnt;
            auto *InsLaneC = ConstantInt::get(Type::getInt32Ty(Ctx), InsLane);
```
- EN: This region continues the BottomUpVec implementation with local helper logic centered on TODO, Try, WhereIt, SmallVector.
- CN: 这一段延续了 BottomUpVec 的主体实现，围绕 TODO, Try, WhereIt, SmallVector 等局部辅助逻辑展开。

### Lines 482-535

```cpp
            LastV = InsertElementInst::create(LastV, ExtrI, InsLaneC, WhereIt,
                                              Ctx, "VIns");
          }
        }
        Lane += NumLanesToInsert;
      }
      NewVec = LastV;
      break;
    }
    case LegalityResultID::Pack: {
      // If we can't vectorize the seeds then just return.
      if (Depth == 0)
        return nullptr;
      NewVec = createPack(Bndl, UserBB);
      break;
    }
    }
    if (NewVec != nullptr) {
      Change = true;
      ActionPtr->Vec = NewVec;
    }
#ifndef NDEBUG
    if (AlwaysVerify) {
      // This helps find broken IR by constantly verifying the function. Note
      // that this is very expensive and should only be used for debugging.
      Instruction *I0 = isa<Instruction>(Bndl[0])
                            ? cast<Instruction>(Bndl[0])
                            : cast<Instruction>(UserBndl[0]);
      assert(!Utils::verifyFunction(I0->getParent()->getParent(), dbgs()) &&
             "Broken function!");
    }
#endif // NDEBUG
  }
  return NewVec;
}

bool BottomUpVec::tryVectorize(ArrayRef<Value *> Bndl,
                               LegalityAnalysis &Legality) {
  Change = false;
  if (LLVM_UNLIKELY(BottomUpInvocationCnt++ >= StopAt &&
                    StopAt != StopAtDisabled))
    return false;
  DeadInstrCandidates.clear();
  Legality.clear();
  Actions.clear();
  DebugBndlCnt = 0;
  vectorizeRec(Bndl, {}, /*Depth=*/0, Legality);
  LLVM_DEBUG(dbgs() << DEBUG_PREFIX << "BottomUpVec: Vectorization Actions:\n";
             Actions.dump());
  emitVectors();
  tryEraseDeadInstrs();
  return Change;
}

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 536-554

```cpp
bool BottomUpVec::runOnRegion(Region &Rgn, const Analyses &A) {
  const auto &SeedSlice = Rgn.getAux();
  assert(SeedSlice.size() >= 2 && "Bad slice!");
  Function &F = *SeedSlice[0]->getParent()->getParent();
  IMaps = std::make_unique<InstrMaps>();
  LegalityAnalysis Legality(A.getAA(), A.getScalarEvolution(),
                            F.getParent()->getDataLayout(), F.getContext(),
                            *IMaps);

  // TODO: Refactor to remove the unnecessary copy to SeedSliceVals.
  SmallVector<Value *> SeedSliceVals(SeedSlice.begin(), SeedSlice.end());
  // Try to vectorize starting from the seed slice. The returned value
  // is true if we found vectorizable code and generated some vector
  // code for it. It does not mean that the code is profitable.
  return tryVectorize(SeedSliceVals, Legality);
}

} // namespace sandboxir
} // namespace llvm
```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include runOnRegion, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, ScalarEvolution.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 runOnRegion，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, ScalarEvolution 等分析结果。

## Key Concepts / 关键概念

- EN: Domain: `Vectorize` focuses on vectorization and VPlan infrastructure.
  - CN: 领域：`Vectorize` 主要处理 向量化与 VPlan 基础设施。
- EN: Primary entities: `tryEraseDeadInstrs, collectPotentiallyDeadInstrs, print, runOnRegion` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`tryEraseDeadInstrs, collectPotentiallyDeadInstrs, print, runOnRegion` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DataLayout, ScalarEvolution` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DataLayout, ScalarEvolution` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `AlwaysVerify, StopAt, StopBundle` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `AlwaysVerify, StopAt, StopBundle` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: IR/transform headers: `llvm/Transforms/Vectorize/SandboxVectorizer/Passes/BottomUpVec.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/Debug.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/VecUtils.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/Transforms/Vectorize/SandboxVectorizer/Passes/BottomUpVec.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/Debug.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/VecUtils.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/ADT/SmallVector.h`, `llvm/SandboxIR/Function.h`, `llvm/SandboxIR/Instruction.h`, `llvm/SandboxIR/Module.h`, `llvm/SandboxIR/Region.h`, `llvm/SandboxIR/Utils.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/ADT/SmallVector.h`, `llvm/SandboxIR/Function.h`, `llvm/SandboxIR/Instruction.h`, `llvm/SandboxIR/Module.h`, `llvm/SandboxIR/Region.h`, `llvm/SandboxIR/Utils.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Runtime analysis dependencies: `DataLayout`, `ScalarEvolution` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DataLayout`, `ScalarEvolution` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
