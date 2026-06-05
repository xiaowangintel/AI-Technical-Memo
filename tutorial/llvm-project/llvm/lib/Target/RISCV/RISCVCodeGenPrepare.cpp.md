# RISCVCodeGenPrepare.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVCodeGenPrepare.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements RISC-V-specific code-generation preparation transforms. / 实现RISC-V 专用代码生成预处理变换。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Commentary and design intent / 注释与设计意图
```cpp
//===----- RISCVCodeGenPrepare.cpp ----------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This is a RISC-V specific version of CodeGenPrepare.
// It munges the code in the input function to better prepare it for
// SelectionDAG-based code generation. This works around limitations in it's
// basic-block-at-a-time approach.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 15-30: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp

#include "RISCV.h"
#include "RISCVTargetMachine.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/Analysis/ValueTracking.h"
#include "llvm/CodeGen/TargetPassConfig.h"
#include "llvm/IR/Dominators.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/InstVisitor.h"
#include "llvm/IR/IntrinsicInst.h"
#include "llvm/IR/Intrinsics.h"
#include "llvm/IR/PatternMatch.h"
#include "llvm/InitializePasses.h"
#include "llvm/Pass.h"
#include "llvm/Transforms/Utils/Local.h"
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 31-55: Header guard and interface framing / 头文件保护与接口框架
```cpp
using namespace llvm;

#define DEBUG_TYPE "riscv-codegenprepare"
#define PASS_NAME "RISC-V CodeGenPrepare"

namespace {
class RISCVCodeGenPrepare : public InstVisitor<RISCVCodeGenPrepare, bool> {
  Function &F;
  const DataLayout *DL;
  const DominatorTree *DT;
  const RISCVSubtarget *ST;

public:
  RISCVCodeGenPrepare(Function &F, const DominatorTree *DT,
                      const RISCVSubtarget *ST)
      : F(F), DL(&F.getDataLayout()), DT(DT), ST(ST) {}
  bool run();
  bool visitInstruction(Instruction &I) { return false; }
  bool visitAnd(BinaryOperator &BO);
  bool visitIntrinsicInst(IntrinsicInst &I);
  bool expandVPStrideLoad(IntrinsicInst &I);
  bool expandMulReduction(IntrinsicInst &I);
  bool widenVPMerge(Instruction *I);
  bool visitFreezeInst(FreezeInst &BO);
};
```
**EN:** This block establishes include guards and the outer structure of the public interface.

**CN:** 该区段建立头文件保护并给出公共接口的外层结构。

### Lines 56-72: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
} // namespace

namespace {
class RISCVCodeGenPrepareLegacyPass : public FunctionPass {
public:
  static char ID;

  RISCVCodeGenPrepareLegacyPass() : FunctionPass(ID) {}

  bool runOnFunction(Function &F) override;
  StringRef getPassName() const override { return PASS_NAME; }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    AU.addRequired<DominatorTreeWrapperPass>();
    AU.addRequired<TargetPassConfig>();
  }
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 73-87: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
};
} // namespace

// Try to optimize (i64 (and (zext/sext (i32 X), C1))) if C1 has bit 31 set,
// but bits 63:32 are zero. If we know that bit 31 of X is 0, we can fill
// the upper 32 bits with ones.
bool RISCVCodeGenPrepare::visitAnd(BinaryOperator &BO) {
  if (!ST->is64Bit())
    return false;

  if (!BO.getType()->isIntegerTy(64))
    return false;

  using namespace PatternMatch;
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 88-103: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  // Left hand side should be a zext nneg.
  Value *LHSSrc;
  if (!match(BO.getOperand(0), m_NNegZExt(m_Value(LHSSrc))))
    return false;

  if (!LHSSrc->getType()->isIntegerTy(32))
    return false;

  // Right hand side should be a constant.
  Value *RHS = BO.getOperand(1);

  auto *CI = dyn_cast<ConstantInt>(RHS);
  if (!CI)
    return false;
  uint64_t C = CI->getZExtValue();
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 104-131: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  // Look for constants that fit in 32 bits but not simm12, and can be made
  // into simm12 by sign extending bit 31. This will allow use of ANDI.
  // TODO: Is worth making simm32?
  if (!isUInt<32>(C) || isInt<12>(C) || !isInt<12>(SignExtend64<32>(C)))
    return false;

  // Sign extend the constant and replace the And operand.
  C = SignExtend64<32>(C);
  BO.setOperand(1, ConstantInt::get(RHS->getType(), C));

  return true;
}

// With EVL tail folding, an AnyOf reduction will generate an i1 vp.merge like
// follows:
//
// loop:
//   %phi = phi <vscale x 4 x i1> [zeroinitializer, %entry], [%freeze, %loop]
//   %cmp = icmp ...
//   %rec = call <vscale x 4 x i1> @llvm.vp.merge(%cmp, i1 true, %phi, %evl)
//   %freeze = freeze <vscale x 4 x i1> %rec [optional]
//   ...
// middle:
//   %res = call i1 @llvm.vector.reduce.or(<vscale x 4 x i1> %freeze)
//
// However RVV doesn't have any tail undisturbed mask instructions and so we
// need a convoluted sequence of mask instructions to lower the i1 vp.merge: see
// llvm/test/CodeGen/RISCV/rvv/vpmerge-sdnode.ll.
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 132-151: Commentary and design intent / 注释与设计意图
```cpp
//
// To avoid that this widens the i1 vp.merge to an i8 vp.merge, which will
// generate a single vmerge.vim:
//
// loop:
//   %phi = phi <vscale x 4 x i8> [zeroinitializer, %entry], [%freeze, %loop]
//   %cmp = icmp ...
//   %rec = call <vscale x 4 x i8> @llvm.vp.merge(%cmp, i8 true, %phi, %evl)
//   %freeze = freeze <vscale x 4 x i8> %rec
//   %trunc = trunc <vscale x 4 x i8> %freeze to <vscale x 4 x i1>
//   ...
// middle:
//   %res = call i1 @llvm.vector.reduce.or(<vscale x 4 x i1> %trunc)
//
// The trunc will normally be sunk outside of the loop, but even if there are
// users inside the loop it is still profitable.
bool RISCVCodeGenPrepare::widenVPMerge(Instruction *Root) {
  if (!Root->getType()->getScalarType()->isIntegerTy(1))
    return false;
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 152-168: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
  Value *Mask, *True, *PhiV, *EVL;
  using namespace PatternMatch;
  auto m_VPMerge = m_Intrinsic<Intrinsic::vp_merge>(
      m_Value(Mask), m_Value(True), m_Value(PhiV), m_Value(EVL));
  if (!match(Root, m_CombineOr(m_VPMerge, m_Freeze(m_VPMerge))))
    return false;

  auto *Phi = dyn_cast<PHINode>(PhiV);
  if (!Phi || !Phi->hasOneUse() || Phi->getNumIncomingValues() != 2 ||
      !match(Phi->getIncomingValue(0), m_Zero()) ||
      Phi->getIncomingValue(1) != Root)
    return false;

  Type *WideTy =
      VectorType::get(IntegerType::getInt8Ty(Root->getContext()),
                      cast<VectorType>(Root->getType())->getElementCount());
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 169-183: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  IRBuilder<> Builder(Phi);
  PHINode *WidePhi = Builder.CreatePHI(WideTy, 2);
  WidePhi->addIncoming(ConstantAggregateZero::get(WideTy),
                       Phi->getIncomingBlock(0));
  Builder.SetInsertPoint(Root);
  Value *WideTrue = Builder.CreateZExt(True, WideTy);
  Value *WideMerge = Builder.CreateIntrinsic(Intrinsic::vp_merge, {WideTy},
                                             {Mask, WideTrue, WidePhi, EVL});
  if (isa<FreezeInst>(Root))
    WideMerge = Builder.CreateFreeze(WideMerge);
  WidePhi->addIncoming(WideMerge, Phi->getIncomingBlock(1));
  Value *Trunc = Builder.CreateTrunc(WideMerge, Root->getType());

  Root->replaceAllUsesWith(Trunc);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 184-197: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // Break the cycle and delete the old chain.
  Phi->setIncomingValue(1, Phi->getIncomingValue(0));
  llvm::RecursivelyDeleteTriviallyDeadInstructions(Root);

  return true;
}

bool RISCVCodeGenPrepare::visitFreezeInst(FreezeInst &I) {
  if (auto *II = dyn_cast<IntrinsicInst>(I.getOperand(0)))
    if (II->getIntrinsicID() == Intrinsic::vp_merge)
      return widenVPMerge(&I);
  return false;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 198-225: Commentary and design intent / 注释与设计意图
```cpp
// LLVM vector reduction intrinsics return a scalar result, but on RISC-V vector
// reduction instructions write the result in the first element of a vector
// register. So when a reduction in a loop uses a scalar phi, we end up with
// unnecessary scalar moves:
//
// loop:
// vfmv.s.f v10, fa0
// vfredosum.vs v8, v8, v10
// vfmv.f.s fa0, v8
//
// This mainly affects ordered fadd reductions and VP reductions that have a
// scalar start value, since other types of reduction typically use element-wise
// vectorisation in the loop body. This tries to vectorize any scalar phis that
// feed into these reductions:
//
// loop:
// %phi = phi <float> [ ..., %entry ], [ %acc, %loop ]
// %acc = call float @llvm.vector.reduce.fadd.nxv2f32(float %phi,
//                                                    <vscale x 2 x float> %vec)
//
// ->
//
// loop:
// %phi = phi <vscale x 2 x float> [ ..., %entry ], [ %acc.vec, %loop ]
// %phi.scalar = extractelement <vscale x 2 x float> %phi, i64 0
// %acc = call float @llvm.vector.reduce.fadd.nxv2f32(float %x,
//                                                    <vscale x 2 x float> %vec)
// %acc.vec = insertelement <vscale x 2 x float> poison, float %acc.next, i64 0
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 226-242: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
//
// Which eliminates the scalar -> vector -> scalar crossing during instruction
// selection.
bool RISCVCodeGenPrepare::visitIntrinsicInst(IntrinsicInst &I) {
  if (expandVPStrideLoad(I))
    return true;

  if (expandMulReduction(I))
    return true;

  if (widenVPMerge(&I))
    return true;

  if (I.getIntrinsicID() != Intrinsic::vector_reduce_fadd &&
      !isa<VPReductionIntrinsic>(&I))
    return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 243-257: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  auto *PHI = dyn_cast<PHINode>(I.getOperand(0));
  if (!PHI || !PHI->hasOneUse() ||
      !llvm::is_contained(PHI->incoming_values(), &I))
    return false;

  Type *VecTy = I.getOperand(1)->getType();
  IRBuilder<> Builder(PHI);
  auto *VecPHI = Builder.CreatePHI(VecTy, PHI->getNumIncomingValues());

  for (auto *BB : PHI->blocks()) {
    Builder.SetInsertPoint(BB->getTerminator());
    Value *InsertElt = Builder.CreateInsertElement(
        VecTy, PHI->getIncomingValueForBlock(BB), (uint64_t)0);
    VecPHI->addIncoming(InsertElt, BB);
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 258-276: Function implementation: VectorType::get / 函数实现：VectorType::get
```cpp

  Builder.SetInsertPoint(&I);
  I.setOperand(0, Builder.CreateExtractElement(VecPHI, (uint64_t)0));

  PHI->eraseFromParent();

  return true;
}

// Extract pieces of size PieceEC from Vec, then build a binary tree of
// element-wise multiplies reducing to a single piece.
static Value *buildMulTree(IRBuilder<> &Builder, ElementCount PieceEC,
                           Value *Vec) {
  auto *VecTy = cast<VectorType>(Vec->getType());
  auto *PieceTy = VectorType::get(VecTy->getElementType(), PieceEC);
  unsigned PieceElts = PieceEC.getKnownMinValue();
  unsigned NumPieces = VecTy->getElementCount().getKnownMinValue() / PieceElts;
  assert(isPowerOf2_32(NumPieces));
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 277-295: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  SmallVector<Value *, 8> Pieces(NumPieces);
  for (unsigned i = 0; i < NumPieces; i++)
    Pieces[i] = Builder.CreateExtractVector(PieceTy, Vec, i * PieceElts);

  while (Pieces.size() > 1) {
    for (unsigned i = 0; i < Pieces.size() / 2; i++)
      Pieces[i] =
          Builder.CreateMul(Pieces[i * 2], Pieces[i * 2 + 1], "bin.rdx");
    Pieces.truncate(Pieces.size() / 2);
  }
  return Pieces[0];
}

// Partially expand a vector_reduce_mul wider than M1 to reduce
// register pressure and the number of vsetvlis required.
bool RISCVCodeGenPrepare::expandMulReduction(IntrinsicInst &II) {
  if (II.getIntrinsicID() != Intrinsic::vector_reduce_mul)
    return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 296-321: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  if (!ST->hasVInstructions())
    return false;

  Value *TmpVec = II.getArgOperand(0);
  auto *VecTy = cast<VectorType>(TmpVec->getType());
  unsigned EltSize = VecTy->getScalarSizeInBits();

  if (auto *ScalTy = dyn_cast<ScalableVectorType>(VecTy)) {
    unsigned MinElts = ScalTy->getMinNumElements();

    if (auto VLen = ST->getRealVLen()) {
      // If VLEN is exactly known, convert to a fixed vector reduction and
      // recurse to let the fixed path handle it (shuffle reduction instead
      // of a scalar loop).
      unsigned VScale = *VLen / RISCV::RVVBitsPerBlock;
      auto *FixedTy =
          FixedVectorType::get(VecTy->getElementType(), MinElts * VScale);
      IRBuilder<> Builder(&II);
      Value *Fixed = Builder.CreateExtractVector(FixedTy, TmpVec, (uint64_t)0);
      auto *FixedRdx = cast<IntrinsicInst>(Builder.CreateIntrinsic(
          Intrinsic::vector_reduce_mul, {FixedTy}, {Fixed}));
      II.replaceAllUsesWith(FixedRdx);
      II.eraseFromParent();
      expandMulReduction(*FixedRdx);
      return true;
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 322-335: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

    unsigned M1MinElts = RISCV::RVVBitsPerBlock / EltSize;
    if (MinElts <= M1MinElts || !isPowerOf2_32(MinElts / M1MinElts))
      return false;

    IRBuilder<> Builder(&II);
    auto M1EC = ElementCount::getScalable(M1MinElts);
    Value *Reduced = buildMulTree(Builder, M1EC, TmpVec);
    Value *Rdx = Builder.CreateIntrinsic(Intrinsic::vector_reduce_mul,
                                         {Reduced->getType()}, {Reduced});
    II.replaceAllUsesWith(Rdx);
    II.eraseFromParent();
    return true;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 336-363: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  unsigned VF = cast<FixedVectorType>(VecTy)->getNumElements();
  unsigned MinVLen = ST->getRealMinVLen();
  unsigned M1VF = MinVLen / EltSize;

  if (!isPowerOf2_32(VF) || VF <= M1VF)
    return false;

  IRBuilder<> Builder(&II);
  auto M1EC = ElementCount::getFixed(M1VF);
  auto *M1Ty = VectorType::get(VecTy->getElementType(), M1EC);

  // When VLEN is exactly known, extract m1 pieces and build a mul tree.
  // This greatly reduces register pressure during the reduction, and
  // avoids all but one vsetvli (the one from original LMUL to m1).
  // TODO: Generalize to handle the splitting case.
  if (MinVLen == ST->getRealMaxVLen() && VF <= 8 * M1VF) {
    TmpVec = buildMulTree(Builder, M1EC, TmpVec);
  } else {
    // For non-exact VLEN, shuffle-reduce at the original vector width down to
    // m1, then extract.  This prioritizes reducing the number of vsetvli
    // over maximal reduction of LMUL for the intermediate states.
    SmallVector<int, 32> ShuffleMask(VF);
    for (unsigned LiveElts = VF; LiveElts > M1VF; LiveElts /= 2) {
      unsigned Half = LiveElts / 2;
      std::iota(ShuffleMask.begin(), ShuffleMask.begin() + Half, Half);
      std::fill(ShuffleMask.begin() + Half, ShuffleMask.end(), -1);
      Value *Shuf =
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 364-377: Function implementation / 函数实现
```cpp
          Builder.CreateShuffleVector(TmpVec, ShuffleMask, "rdx.shuf");
      TmpVec = Builder.CreateMul(TmpVec, Shuf, "bin.rdx");
    }
    // Extract the M1-sized subvector and emit the final reduction intrinsic.
    // This is the reason we're here - to force a vsetvli toggle once at m1.
    TmpVec = Builder.CreateExtractVector(M1Ty, TmpVec, (uint64_t)0, "rdx.sub");
  }

  Value *Rdx =
      Builder.CreateIntrinsic(Intrinsic::vector_reduce_mul, {M1Ty}, {TmpVec});
  II.replaceAllUsesWith(Rdx);
  II.eraseFromParent();
  return true;
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 378-394: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp

// Always expand zero strided loads so we match more .vx splat patterns, even if
// we have +optimized-zero-stride-loads. RISCVDAGToDAGISel::Select will convert
// it back to a strided load if it's optimized.
bool RISCVCodeGenPrepare::expandVPStrideLoad(IntrinsicInst &II) {
  Value *BasePtr, *VL;

  using namespace PatternMatch;
  if (!match(&II, m_Intrinsic<Intrinsic::experimental_vp_strided_load>(
                      m_Value(BasePtr), m_Zero(), m_AllOnes(), m_Value(VL))))
    return false;

  // If SEW>XLEN then a splat will get lowered as a zero strided load anyway, so
  // avoid expanding here.
  if (II.getType()->getScalarSizeInBits() > ST->getXLen())
    return false;
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 395-411: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  if (!isKnownNonZero(VL, {*DL, DT, nullptr, &II}))
    return false;

  auto *VTy = cast<VectorType>(II.getType());

  IRBuilder<> Builder(&II);
  Type *STy = VTy->getElementType();
  Value *Val = Builder.CreateLoad(STy, BasePtr);
  Value *Res = Builder.CreateIntrinsic(
      Intrinsic::vp_merge, VTy,
      {II.getOperand(2), Builder.CreateVectorSplat(VTy->getElementCount(), Val),
       PoisonValue::get(VTy), VL});

  II.replaceAllUsesWith(Res);
  II.eraseFromParent();
  return true;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 412-425: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

bool RISCVCodeGenPrepare::run() {
  bool MadeChange = false;
  for (auto &BB : F)
    for (Instruction &I : llvm::make_early_inc_range(BB))
      MadeChange |= visit(I);

  return MadeChange;
}

bool RISCVCodeGenPrepareLegacyPass::runOnFunction(Function &F) {
  if (skipFunction(F))
    return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 426-440: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  auto &TPC = getAnalysis<TargetPassConfig>();
  auto &TM = TPC.getTM<RISCVTargetMachine>();
  auto ST = &TM.getSubtarget<RISCVSubtarget>(F);
  auto DT = &getAnalysis<DominatorTreeWrapperPass>().getDomTree();

  RISCVCodeGenPrepare RVCGP(F, DT, ST);
  return RVCGP.run();
}

INITIALIZE_PASS_BEGIN(RISCVCodeGenPrepareLegacyPass, DEBUG_TYPE, PASS_NAME,
                      false, false)
INITIALIZE_PASS_DEPENDENCY(TargetPassConfig)
INITIALIZE_PASS_END(RISCVCodeGenPrepareLegacyPass, DEBUG_TYPE, PASS_NAME, false,
                    false)
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 441-454: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
char RISCVCodeGenPrepareLegacyPass::ID = 0;

FunctionPass *llvm::createRISCVCodeGenPrepareLegacyPass() {
  return new RISCVCodeGenPrepareLegacyPass();
}

PreservedAnalyses RISCVCodeGenPreparePass::run(Function &F,
                                               FunctionAnalysisManager &FAM) {
  DominatorTree *DT = &FAM.getResult<DominatorTreeAnalysis>(F);
  auto ST = &TM->getSubtarget<RISCVSubtarget>(F);
  bool Changed = RISCVCodeGenPrepare(F, DT, ST).run();
  if (!Changed)
    return PreservedAnalyses::all();
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 455-458: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  PreservedAnalyses PA = PreservedAnalyses::none();
  PA.preserveSet<CFGAnalyses>();
  return PA;
}
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

## Key Concepts / 关键概念
- **RISC-V backend structure** / **RISC-V 后端结构**

## Dependencies / 依赖关系
- `RISCV.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVTargetMachine.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/ADT/Statistic.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/Analysis/ValueTracking.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/TargetPassConfig.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/IR/Dominators.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/IR/IRBuilder.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/IR/InstVisitor.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/IR/IntrinsicInst.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/IR/Intrinsics.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/IR/PatternMatch.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/InitializePasses.h` — Directly referenced by this file. / 该文件直接引用的依赖。
