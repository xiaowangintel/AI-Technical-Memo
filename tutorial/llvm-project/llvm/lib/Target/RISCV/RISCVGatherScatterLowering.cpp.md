# RISCVGatherScatterLowering.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVGatherScatterLowering.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements lowering of gather/scatter vector memory operations for RISC-V. / 实现RISC-V gather/scatter 向量内存操作的下降。

## Line-by-Line Analysis / 逐行分析
### Lines 1-27: File prologue and imported dependencies / 文件前言与导入依赖
```cpp
//===- RISCVGatherScatterLowering.cpp - Gather/Scatter lowering -----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass custom lowers llvm.gather and llvm.scatter instructions to
// RISC-V intrinsics.
//
//===----------------------------------------------------------------------===//

#include "RISCV.h"
#include "RISCVTargetMachine.h"
#include "llvm/Analysis/InstSimplifyFolder.h"
#include "llvm/Analysis/LoopInfo.h"
#include "llvm/Analysis/ValueTracking.h"
#include "llvm/Analysis/VectorUtils.h"
#include "llvm/CodeGen/TargetPassConfig.h"
#include "llvm/IR/GetElementPtrTypeIterator.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/IntrinsicInst.h"
#include "llvm/IR/PatternMatch.h"
#include "llvm/Transforms/Utils/Local.h"
#include <optional>
```
**EN:** This opening block combines banner comments with the first wave of includes, giving readers both context and the main compile-time dependencies.

**CN:** 该开头区段同时包含说明性注释与首批 include，让读者先看到文件定位，再看到主要编译期依赖。

### Lines 28-42: Header guard and interface framing / 头文件保护与接口框架
```cpp
using namespace llvm;
using namespace PatternMatch;

#define DEBUG_TYPE "riscv-gather-scatter-lowering"

namespace {

class RISCVGatherScatterLowering : public FunctionPass {
  const RISCVSubtarget *ST = nullptr;
  const RISCVTargetLowering *TLI = nullptr;
  LoopInfo *LI = nullptr;
  const DataLayout *DL = nullptr;

  SmallVector<WeakTrackingVH> MaybeDeadPHIs;
```
**EN:** This block establishes include guards and the outer structure of the public interface.

**CN:** 该区段建立头文件保护并给出公共接口的外层结构。

### Lines 43-59: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // Cache of the BasePtr and Stride determined from this GEP. When a GEP is
  // used by multiple gathers/scatters, this allow us to reuse the scalar
  // instructions we created for the first gather/scatter for the others.
  DenseMap<GetElementPtrInst *, std::pair<Value *, Value *>> StridedAddrs;

public:
  static char ID; // Pass identification, replacement for typeid

  RISCVGatherScatterLowering() : FunctionPass(ID) {}

  bool runOnFunction(Function &F) override;

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    AU.addRequired<TargetPassConfig>();
    AU.addRequired<LoopInfoWrapperPass>();
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 60-74: Function implementation: getPassName / 函数实现：getPassName
```cpp

  StringRef getPassName() const override {
    return "RISC-V gather/scatter lowering";
  }

private:
  bool tryCreateStridedLoadStore(IntrinsicInst *II);

  std::pair<Value *, Value *> determineBaseAndStride(Instruction *Ptr,
                                                     IRBuilderBase &Builder);

  bool matchStridedRecurrence(Value *Index, Loop *L, Value *&Stride,
                              PHINode *&BasePtr, BinaryOperator *&Inc,
                              IRBuilderBase &Builder);
};
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 75-91: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp

} // end anonymous namespace

char RISCVGatherScatterLowering::ID = 0;

INITIALIZE_PASS(RISCVGatherScatterLowering, DEBUG_TYPE,
                "RISC-V gather/scatter lowering pass", false, false)

FunctionPass *llvm::createRISCVGatherScatterLoweringPass() {
  return new RISCVGatherScatterLowering();
}

// TODO: Should we consider the mask when looking for a stride?
static std::pair<Value *, Value *> matchStridedConstant(Constant *StartC) {
  if (!isa<FixedVectorType>(StartC->getType()))
    return std::make_pair(nullptr, nullptr);
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 92-105: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  unsigned NumElts = cast<FixedVectorType>(StartC->getType())->getNumElements();

  // Check that the start value is a strided constant.
  auto *StartVal =
      dyn_cast_or_null<ConstantInt>(StartC->getAggregateElement((unsigned)0));
  if (!StartVal)
    return std::make_pair(nullptr, nullptr);
  APInt StrideVal(StartVal->getValue().getBitWidth(), 0);
  ConstantInt *Prev = StartVal;
  for (unsigned i = 1; i != NumElts; ++i) {
    auto *C = dyn_cast_or_null<ConstantInt>(StartC->getAggregateElement(i));
    if (!C)
      return std::make_pair(nullptr, nullptr);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 106-119: Definitions and supporting logic / 定义与支撑逻辑
```cpp
    APInt LocalStride = C->getValue() - Prev->getValue();
    if (i == 1)
      StrideVal = LocalStride;
    else if (StrideVal != LocalStride)
      return std::make_pair(nullptr, nullptr);

    Prev = C;
  }

  Value *Stride = ConstantInt::get(StartVal->getType(), StrideVal);

  return std::make_pair(StartVal, Stride);
}
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 120-141: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
static std::pair<Value *, Value *> matchStridedStart(Value *Start,
                                                     IRBuilderBase &Builder) {
  // Base case, start is a strided constant.
  auto *StartC = dyn_cast<Constant>(Start);
  if (StartC)
    return matchStridedConstant(StartC);

  // Base case, start is a stepvector
  if (match(Start, m_Intrinsic<Intrinsic::stepvector>())) {
    auto *Ty = Start->getType()->getScalarType();
    return std::make_pair(ConstantInt::get(Ty, 0), ConstantInt::get(Ty, 1));
  }

  // Not a constant, maybe it's a strided constant with a splat added or
  // multiplied.
  auto *BO = dyn_cast<BinaryOperator>(Start);
  if (!BO || (BO->getOpcode() != Instruction::Add &&
              BO->getOpcode() != Instruction::Or &&
              BO->getOpcode() != Instruction::Shl &&
              BO->getOpcode() != Instruction::Mul))
    return std::make_pair(nullptr, nullptr);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 142-155: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  if (BO->getOpcode() == Instruction::Or &&
      !cast<PossiblyDisjointInst>(BO)->isDisjoint())
    return std::make_pair(nullptr, nullptr);

  // Look for an operand that is splatted.
  unsigned OtherIndex = 0;
  Value *Splat = getSplatValue(BO->getOperand(1));
  if (!Splat && Instruction::isCommutative(BO->getOpcode())) {
    Splat = getSplatValue(BO->getOperand(0));
    OtherIndex = 1;
  }
  if (!Splat)
    return std::make_pair(nullptr, nullptr);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 156-183: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  Value *Stride;
  std::tie(Start, Stride) = matchStridedStart(BO->getOperand(OtherIndex),
                                              Builder);
  if (!Start)
    return std::make_pair(nullptr, nullptr);

  Builder.SetInsertPoint(BO);
  Builder.SetCurrentDebugLocation(DebugLoc());
  // Add the splat value to the start or multiply the start and stride by the
  // splat.
  switch (BO->getOpcode()) {
  default:
    llvm_unreachable("Unexpected opcode");
  case Instruction::Or:
    Start = Builder.CreateDisjointOr(Start, Splat);
    break;
  case Instruction::Add:
    Start = Builder.CreateAdd(Start, Splat);
    break;
  case Instruction::Mul:
    Start = Builder.CreateMul(Start, Splat);
    Stride = Builder.CreateMul(Stride, Splat);
    break;
  case Instruction::Shl:
    Start = Builder.CreateShl(Start, Splat);
    Stride = Builder.CreateShl(Stride, Splat);
    break;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 184-203: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  return std::make_pair(Start, Stride);
}

// Recursively, walk about the use-def chain until we find a Phi with a strided
// start value. Build and update a scalar recurrence as we unwind the recursion.
// We also update the Stride as we unwind. Our goal is to move all of the
// arithmetic out of the loop.
bool RISCVGatherScatterLowering::matchStridedRecurrence(Value *Index, Loop *L,
                                                        Value *&Stride,
                                                        PHINode *&BasePtr,
                                                        BinaryOperator *&Inc,
                                                        IRBuilderBase &Builder) {
  // Our base case is a Phi.
  if (auto *Phi = dyn_cast<PHINode>(Index)) {
    // A phi node we want to perform this function on should be from the
    // loop header.
    if (Phi->getParent() != L->getHeader())
      return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 204-217: Definitions and supporting logic / 定义与支撑逻辑
```cpp
    Value *Step, *Start;
    if (!matchSimpleRecurrence(Phi, Inc, Start, Step) ||
        Inc->getOpcode() != Instruction::Add)
      return false;
    assert(Phi->getNumIncomingValues() == 2 && "Expected 2 operand phi.");
    unsigned IncrementingBlock = Phi->getIncomingValue(0) == Inc ? 0 : 1;
    assert(Phi->getIncomingValue(IncrementingBlock) == Inc &&
           "Expected one operand of phi to be Inc");

    // Step should be a splat.
    Step = getSplatValue(Step);
    if (!Step)
      return false;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 218-234: Definitions and supporting logic / 定义与支撑逻辑
```cpp
    std::tie(Start, Stride) = matchStridedStart(Start, Builder);
    if (!Start)
      return false;
    assert(Stride != nullptr);

    // Build scalar phi and increment.
    BasePtr =
        PHINode::Create(Start->getType(), 2, Phi->getName() + ".scalar", Phi->getIterator());
    Inc = BinaryOperator::CreateAdd(BasePtr, Step, Inc->getName() + ".scalar",
                                    Inc->getIterator());
    BasePtr->addIncoming(Start, Phi->getIncomingBlock(1 - IncrementingBlock));
    BasePtr->addIncoming(Inc, Phi->getIncomingBlock(IncrementingBlock));

    // Note that this Phi might be eligible for removal.
    MaybeDeadPHIs.push_back(Phi);
    return true;
  }
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 235-255: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  // Otherwise look for binary operator.
  auto *BO = dyn_cast<BinaryOperator>(Index);
  if (!BO)
    return false;

  switch (BO->getOpcode()) {
  default:
    return false;
  case Instruction::Or:
    // We need to be able to treat Or as Add.
    if (!cast<PossiblyDisjointInst>(BO)->isDisjoint())
      return false;
    break;
  case Instruction::Add:
    break;
  case Instruction::Shl:
    break;
  case Instruction::Mul:
    break;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 256-270: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  // We should have one operand in the loop and one splat.
  Value *OtherOp;
  if (isa<Instruction>(BO->getOperand(0)) &&
      L->contains(cast<Instruction>(BO->getOperand(0)))) {
    Index = cast<Instruction>(BO->getOperand(0));
    OtherOp = BO->getOperand(1);
  } else if (isa<Instruction>(BO->getOperand(1)) &&
             L->contains(cast<Instruction>(BO->getOperand(1))) &&
             Instruction::isCommutative(BO->getOpcode())) {
    Index = cast<Instruction>(BO->getOperand(1));
    OtherOp = BO->getOperand(0);
  } else {
    return false;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 271-284: Definitions and supporting logic / 定义与支撑逻辑
```cpp

  // Make sure other op is loop invariant.
  if (!L->isLoopInvariant(OtherOp))
    return false;

  // Make sure we have a splat.
  Value *SplatOp = getSplatValue(OtherOp);
  if (!SplatOp)
    return false;

  // Recurse up the use-def chain.
  if (!matchStridedRecurrence(Index, L, Stride, BasePtr, Inc, Builder))
    return false;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 285-306: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // Locate the Step and Start values from the recurrence.
  unsigned StepIndex = Inc->getOperand(0) == BasePtr ? 1 : 0;
  unsigned StartBlock = BasePtr->getOperand(0) == Inc ? 1 : 0;
  Value *Step = Inc->getOperand(StepIndex);
  Value *Start = BasePtr->getOperand(StartBlock);

  // We need to adjust the start value in the preheader.
  Builder.SetInsertPoint(
      BasePtr->getIncomingBlock(StartBlock)->getTerminator());
  Builder.SetCurrentDebugLocation(DebugLoc());

  // TODO: Share this switch with matchStridedStart?
  switch (BO->getOpcode()) {
  default:
    llvm_unreachable("Unexpected opcode!");
  case Instruction::Add:
  case Instruction::Or: {
    // An add only affects the start value. It's ok to do this for Or because
    // we already checked that there are no common set bits.
    Start = Builder.CreateAdd(Start, SplatOp, "start");
    break;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 307-323: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  case Instruction::Mul: {
    Start = Builder.CreateMul(Start, SplatOp, "start");
    Stride = Builder.CreateMul(Stride, SplatOp, "stride");
    break;
  }
  case Instruction::Shl: {
    Start = Builder.CreateShl(Start, SplatOp, "start");
    Stride = Builder.CreateShl(Stride, SplatOp, "stride");
    break;
  }
  }

  // If the Step was defined inside the loop, adjust it before its definition
  // instead of in the preheader.
  if (auto *StepI = dyn_cast<Instruction>(Step); StepI && L->contains(StepI))
    Builder.SetInsertPoint(*StepI->getInsertionPointAfterDef());
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 324-338: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  switch (BO->getOpcode()) {
  default:
    break;
  case Instruction::Mul:
    Step = Builder.CreateMul(Step, SplatOp, "step");
    break;
  case Instruction::Shl:
    Step = Builder.CreateShl(Step, SplatOp, "step");
    break;
  }

  Inc->setOperand(StepIndex, Step);
  BasePtr->setIncomingValue(StartBlock, Start);
  return true;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 339-353: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

std::pair<Value *, Value *>
RISCVGatherScatterLowering::determineBaseAndStride(Instruction *Ptr,
                                                   IRBuilderBase &Builder) {

  // A gather/scatter of a splat is a zero strided load/store.
  if (auto *BasePtr = getSplatValue(Ptr)) {
    Type *IntPtrTy = DL->getIntPtrType(BasePtr->getType());
    return std::make_pair(BasePtr, ConstantInt::get(IntPtrTy, 0));
  }

  auto *GEP = dyn_cast<GetElementPtrInst>(Ptr);
  if (!GEP)
    return std::make_pair(nullptr, nullptr);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 354-375: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  auto I = StridedAddrs.find(GEP);
  if (I != StridedAddrs.end())
    return I->second;

  SmallVector<Value *, 2> Ops(GEP->operands());

  // If the base pointer is a vector, check if it's strided.
  Value *Base = GEP->getPointerOperand();
  if (auto *BaseInst = dyn_cast<Instruction>(Base);
      BaseInst && BaseInst->getType()->isVectorTy()) {
    // If GEP's offset is scalar then we can add it to the base pointer's base.
    auto IsScalar = [](Value *Idx) { return !Idx->getType()->isVectorTy(); };
    if (all_of(GEP->indices(), IsScalar)) {
      auto [BaseBase, Stride] = determineBaseAndStride(BaseInst, Builder);
      if (BaseBase) {
        Builder.SetInsertPoint(GEP);
        SmallVector<Value *> Indices(GEP->indices());
        Value *OffsetBase =
            Builder.CreateGEP(GEP->getSourceElementType(), BaseBase, Indices,
                              GEP->getName() + "offset", GEP->isInBounds());
        return {OffsetBase, Stride};
      }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 376-389: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    }
  }

  // Base pointer needs to be a scalar.
  Value *ScalarBase = Base;
  if (ScalarBase->getType()->isVectorTy()) {
    ScalarBase = getSplatValue(ScalarBase);
    if (!ScalarBase)
      return std::make_pair(nullptr, nullptr);
  }

  std::optional<unsigned> VecOperand;
  unsigned TypeScale = 0;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 390-404: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // Look for a vector operand and scale.
  gep_type_iterator GTI = gep_type_begin(GEP);
  for (unsigned i = 1, e = GEP->getNumOperands(); i != e; ++i, ++GTI) {
    if (!Ops[i]->getType()->isVectorTy())
      continue;

    if (VecOperand)
      return std::make_pair(nullptr, nullptr);

    VecOperand = i;

    TypeSize TS = GTI.getSequentialElementStride(*DL);
    if (TS.isScalable())
      return std::make_pair(nullptr, nullptr);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 405-427: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    TypeScale = TS.getFixedValue();
  }

  // We need to find a vector index to simplify.
  if (!VecOperand)
    return std::make_pair(nullptr, nullptr);

  // We can't extract the stride if the arithmetic is done at a different size
  // than the pointer type. Adding the stride later may not wrap correctly.
  // Technically we could handle wider indices, but I don't expect that in
  // practice.  Handle one special case here - constants.  This simplifies
  // writing test cases.
  Value *VecIndex = Ops[*VecOperand];
  Type *VecIntPtrTy = DL->getIntPtrType(GEP->getType());
  if (VecIndex->getType() != VecIntPtrTy) {
    auto *VecIndexC = dyn_cast<Constant>(VecIndex);
    if (!VecIndexC)
      return std::make_pair(nullptr, nullptr);
    if (VecIndex->getType()->getScalarSizeInBits() > VecIntPtrTy->getScalarSizeInBits())
      VecIndex = ConstantFoldCastInstruction(Instruction::Trunc, VecIndexC, VecIntPtrTy);
    else
      VecIndex = ConstantFoldCastInstruction(Instruction::SExt, VecIndexC, VecIntPtrTy);
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 428-441: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  // Handle the non-recursive case.  This is what we see if the vectorizer
  // decides to use a scalar IV + vid on demand instead of a vector IV.
  auto [Start, Stride] = matchStridedStart(VecIndex, Builder);
  if (Start) {
    assert(Stride);
    Builder.SetInsertPoint(GEP);

    // Replace the vector index with the scalar start and build a scalar GEP.
    Ops[*VecOperand] = Start;
    Type *SourceTy = GEP->getSourceElementType();
    Value *BasePtr =
        Builder.CreateGEP(SourceTy, ScalarBase, ArrayRef(Ops).drop_front());
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 442-459: Definitions and supporting logic / 定义与支撑逻辑
```cpp
    // Convert stride to pointer size if needed.
    Type *IntPtrTy = DL->getIntPtrType(BasePtr->getType());
    assert(Stride->getType() == IntPtrTy && "Unexpected type");

    // Scale the stride by the size of the indexed type.
    if (TypeScale != 1)
      Stride = Builder.CreateMul(Stride, ConstantInt::get(IntPtrTy, TypeScale));

    auto P = std::make_pair(BasePtr, Stride);
    StridedAddrs[GEP] = P;
    return P;
  }

  // Make sure we're in a loop and that has a pre-header and a single latch.
  Loop *L = LI->getLoopFor(GEP->getParent());
  if (!L || !L->getLoopPreheader() || !L->getLoopLatch())
    return std::make_pair(nullptr, nullptr);
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 460-477: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  BinaryOperator *Inc;
  PHINode *BasePhi;
  if (!matchStridedRecurrence(VecIndex, L, Stride, BasePhi, Inc, Builder))
    return std::make_pair(nullptr, nullptr);

  assert(BasePhi->getNumIncomingValues() == 2 && "Expected 2 operand phi.");
  unsigned IncrementingBlock = BasePhi->getOperand(0) == Inc ? 0 : 1;
  assert(BasePhi->getIncomingValue(IncrementingBlock) == Inc &&
         "Expected one operand of phi to be Inc");

  Builder.SetInsertPoint(GEP);

  // Replace the vector index with the scalar phi and build a scalar GEP.
  Ops[*VecOperand] = BasePhi;
  Type *SourceTy = GEP->getSourceElementType();
  Value *BasePtr =
      Builder.CreateGEP(SourceTy, ScalarBase, ArrayRef(Ops).drop_front());
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 478-493: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  // Final adjustments to stride should go in the start block.
  Builder.SetInsertPoint(
      BasePhi->getIncomingBlock(1 - IncrementingBlock)->getTerminator());

  // Convert stride to pointer size if needed.
  Type *IntPtrTy = DL->getIntPtrType(BasePtr->getType());
  assert(Stride->getType() == IntPtrTy && "Unexpected type");

  // Scale the stride by the size of the indexed type.
  if (TypeScale != 1)
    Stride = Builder.CreateMul(Stride, ConstantInt::get(IntPtrTy, TypeScale));

  auto P = std::make_pair(BasePtr, Stride);
  StridedAddrs[GEP] = P;
  return P;
}
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 494-521: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

bool RISCVGatherScatterLowering::tryCreateStridedLoadStore(IntrinsicInst *II) {
  VectorType *DataType;
  Value *StoreVal = nullptr, *Ptr, *Mask, *EVL = nullptr;
  Align Alignment;
  switch (II->getIntrinsicID()) {
  case Intrinsic::masked_gather:
    DataType = cast<VectorType>(II->getType());
    Ptr = II->getArgOperand(0);
    Alignment = II->getParamAlign(0).valueOrOne();
    Mask = II->getArgOperand(1);
    break;
  case Intrinsic::vp_gather:
    DataType = cast<VectorType>(II->getType());
    Ptr = II->getArgOperand(0);
    // FIXME: Falling back to ABI alignment is incorrect.
    Alignment = II->getParamAlign(0).value_or(
        DL->getABITypeAlign(DataType->getElementType()));
    Mask = II->getArgOperand(1);
    EVL = II->getArgOperand(2);
    break;
  case Intrinsic::masked_scatter:
    DataType = cast<VectorType>(II->getArgOperand(0)->getType());
    StoreVal = II->getArgOperand(0);
    Ptr = II->getArgOperand(1);
    Alignment = II->getParamAlign(1).valueOrOne();
    Mask = II->getArgOperand(2);
    break;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 522-535: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  case Intrinsic::vp_scatter:
    DataType = cast<VectorType>(II->getArgOperand(0)->getType());
    StoreVal = II->getArgOperand(0);
    Ptr = II->getArgOperand(1);
    // FIXME: Falling back to ABI alignment is incorrect.
    Alignment = II->getParamAlign(1).value_or(
        DL->getABITypeAlign(DataType->getElementType()));
    Mask = II->getArgOperand(2);
    EVL = II->getArgOperand(3);
    break;
  default:
    llvm_unreachable("Unexpected intrinsic");
  }
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 536-549: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  // Make sure the operation will be supported by the backend.
  EVT DataTypeVT = TLI->getValueType(*DL, DataType);
  if (!TLI->isLegalStridedLoadStore(DataTypeVT, Alignment))
    return false;

  // FIXME: Let the backend type legalize by splitting/widening?
  if (!TLI->isTypeLegal(DataTypeVT))
    return false;

  // Pointer should be an instruction.
  auto *PtrI = dyn_cast<Instruction>(Ptr);
  if (!PtrI)
    return false;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 550-565: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  LLVMContext &Ctx = PtrI->getContext();
  IRBuilder Builder(Ctx, InstSimplifyFolder(*DL));
  Builder.SetInsertPoint(PtrI);

  Value *BasePtr, *Stride;
  std::tie(BasePtr, Stride) = determineBaseAndStride(PtrI, Builder);
  if (!BasePtr)
    return false;
  assert(Stride != nullptr);

  Builder.SetInsertPoint(II);

  if (!EVL)
    EVL = Builder.CreateElementCount(
        Builder.getInt32Ty(), cast<VectorType>(DataType)->getElementCount());
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 566-582: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  Value *Call;

  if (!StoreVal) {
    Call = Builder.CreateIntrinsic(
        Intrinsic::experimental_vp_strided_load,
        {DataType, BasePtr->getType(), Stride->getType()},
        {BasePtr, Stride, Mask, EVL});

    // Merge llvm.masked.gather's passthru
    if (II->getIntrinsicID() == Intrinsic::masked_gather)
      Call = Builder.CreateSelect(Mask, Call, II->getArgOperand(2));
  } else
    Call = Builder.CreateIntrinsic(
        Intrinsic::experimental_vp_strided_store,
        {DataType, BasePtr->getType(), Stride->getType()},
        {StoreVal, BasePtr, Stride, Mask, EVL});
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 583-596: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  Call->takeName(II);
  II->replaceAllUsesWith(Call);
  II->eraseFromParent();

  if (PtrI->use_empty())
    RecursivelyDeleteTriviallyDeadInstructions(PtrI);

  return true;
}

bool RISCVGatherScatterLowering::runOnFunction(Function &F) {
  if (skipFunction(F))
    return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 597-610: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  auto &TPC = getAnalysis<TargetPassConfig>();
  auto &TM = TPC.getTM<RISCVTargetMachine>();
  ST = &TM.getSubtarget<RISCVSubtarget>(F);
  if (!ST->hasVInstructions() || !ST->useRVVForFixedLengthVectors())
    return false;

  TLI = ST->getTargetLowering();
  DL = &F.getDataLayout();
  LI = &getAnalysis<LoopInfoWrapperPass>().getLoopInfo();

  StridedAddrs.clear();

  SmallVector<IntrinsicInst *, 4> Worklist;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 611-627: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  bool Changed = false;

  for (BasicBlock &BB : F) {
    for (Instruction &I : BB) {
      IntrinsicInst *II = dyn_cast<IntrinsicInst>(&I);
      if (!II)
        continue;
      switch (II->getIntrinsicID()) {
      case Intrinsic::masked_gather:
      case Intrinsic::masked_scatter:
      case Intrinsic::vp_gather:
      case Intrinsic::vp_scatter:
        Worklist.push_back(II);
        break;
      default:
        break;
      }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 628-642: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    }
  }

  // Rewrite gather/scatter to form strided load/store if possible.
  for (auto *II : Worklist)
    Changed |= tryCreateStridedLoadStore(II);

  // Remove any dead phis.
  while (!MaybeDeadPHIs.empty()) {
    if (auto *Phi = dyn_cast_or_null<PHINode>(MaybeDeadPHIs.pop_back_val()))
      RecursivelyDeleteDeadPHINode(Phi);
  }

  return Changed;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

## Key Concepts / 关键概念
- **RISC-V backend structure** / **RISC-V 后端结构**

## Dependencies / 依赖关系
- `RISCV.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVTargetMachine.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/Analysis/InstSimplifyFolder.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/Analysis/LoopInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/Analysis/ValueTracking.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/Analysis/VectorUtils.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/TargetPassConfig.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/IR/GetElementPtrTypeIterator.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/IR/IRBuilder.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/IR/IntrinsicInst.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/IR/PatternMatch.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/Transforms/Utils/Local.h` — Directly referenced by this file. / 该文件直接引用的依赖。
