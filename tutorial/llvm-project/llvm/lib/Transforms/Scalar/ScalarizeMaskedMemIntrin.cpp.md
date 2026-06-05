# ScalarizeMaskedMemIntrin.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Scalar/ScalarizeMaskedMemIntrin.cpp` | `llvm/lib/Transforms/Scalar/ScalarizeMaskedMemIntrin.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements scalarize unsupported masked mem within LLVM's scalar optimizations and loop/CFG transforms layer. | 该源文件实现了 LLVM 标量优化以及循环/CFG 变换层中的 ScalarizeMaskedMemIntrin 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-105

```cpp
//===- ScalarizeMaskedMemIntrin.cpp - Scalarize unsupported masked mem ----===//
//                                    intrinsics
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass replaces masked memory intrinsics - when unsupported by the target
// - with a chain of basic blocks, that deal with the elements one-by-one if the
// appropriate mask bit is set.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Scalar/ScalarizeMaskedMemIntrin.h"
#include "llvm/ADT/Twine.h"
#include "llvm/Analysis/DomTreeUpdater.h"
#include "llvm/Analysis/TargetTransformInfo.h"
#include "llvm/Analysis/VectorUtils.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
}

static bool isConstantIntVector(Value *Mask) {
  Constant *C = dyn_cast<Constant>(Mask);
  if (!C)
    return false;

  unsigned NumElts = cast<FixedVectorType>(Mask->getType())->getNumElements();
  for (unsigned i = 0; i != NumElts; ++i) {
    Constant *CElt = C->getAggregateElement(i);
    if (!CElt || !isa<ConstantInt>(CElt))
      return false;
  }

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 106-212

```cpp
  return true;
}

static unsigned adjustForEndian(const DataLayout &DL, unsigned VectorWidth,
                                unsigned Idx) {
  return DL.isBigEndian() ? VectorWidth - 1 - Idx : Idx;
}

// Translate a masked load intrinsic like
// <16 x i32 > @llvm.masked.load( <16 x i32>* %addr,
//                               <16 x i1> %mask, <16 x i32> %passthru)
// to a chain of basic blocks, with loading element one-by-one if
// the appropriate mask bit is set
//
//  %1 = bitcast i8* %addr to i32*
//  %2 = extractelement <16 x i1> %mask, i32 0
//  br i1 %2, label %cond.load, label %else
//
// cond.load:                                        ; preds = %0
//  %3 = getelementptr i32* %1, i32 0
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  if (isSplatValue(Mask, /*Index=*/0)) {
    Value *Predicate = Builder.CreateExtractElement(Mask, uint64_t(0ull),
                                                    Mask->getName() + ".first");
    Instruction *ThenTerm =
        SplitBlockAndInsertIfThen(Predicate, InsertPt, /*Unreachable=*/false,
                                  /*BranchWeights=*/nullptr, DTU);

    BasicBlock *CondBlock = ThenTerm->getParent();
    CondBlock->setName("cond.load");
    Builder.SetInsertPoint(CondBlock->getTerminator());
    LoadInst *Load = Builder.CreateAlignedLoad(VecType, Ptr, AlignVal,
                                               CI->getName() + ".cond.load");
    Load->copyMetadata(*CI);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DomTreeUpdater. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DomTreeUpdater 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 213-324

```cpp
    BasicBlock *PostLoad = ThenTerm->getSuccessor(0);
    Builder.SetInsertPoint(PostLoad, PostLoad->begin());
    PHINode *Phi = Builder.CreatePHI(VecType, /*NumReservedValues=*/2);
    Phi->addIncoming(Load, CondBlock);
    Phi->addIncoming(Src0, IfBlock);
    Phi->takeName(CI);

    CI->replaceAllUsesWith(Phi);
    CI->eraseFromParent();
    ModifiedDT = true;
    return;
  }
  // If the mask is not v1i1, use scalar bit test operations. This generates
  // better results on X86 at least. However, don't do this on GPUs and other
  // machines with divergence, as there each i1 needs a vector register.
  Value *SclrMask = nullptr;
  if (VectorWidth != 1 && !HasBranchDivergence) {
    Type *SclrMaskTy = Builder.getIntNTy(VectorWidth);
    SclrMask = Builder.CreateBitCast(Mask, SclrMaskTy, "scalar_mask");
  }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
//
// cond.store1:                                      ; preds = %else
//   %6 = extractelement <16 x i32> %val, i32 1
//   %7 = getelementptr i32* %1, i32 1
//   store i32 %6, i32* %7
//   br label %else2
//   . . .
static void scalarizeMaskedStore(const DataLayout &DL, bool HasBranchDivergence,
                                 CallInst *CI, DomTreeUpdater *DTU,
                                 bool &ModifiedDT) {
  Value *Src = CI->getArgOperand(0);
  Value *Ptr = CI->getArgOperand(1);
  Value *Mask = CI->getArgOperand(2);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DomTreeUpdater. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DomTreeUpdater 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 325-429

```cpp
  const Align AlignVal = CI->getParamAlign(1).valueOrOne();
  auto *VecType = cast<VectorType>(Src->getType());

  Type *EltTy = VecType->getElementType();

  IRBuilder<> Builder(CI->getContext());
  Instruction *InsertPt = CI;
  Builder.SetInsertPoint(InsertPt);
  Builder.SetCurrentDebugLocation(CI->getDebugLoc());

  // Short-cut if the mask is all-true.
  if (isa<Constant>(Mask) && cast<Constant>(Mask)->isAllOnesValue()) {
    StoreInst *Store = Builder.CreateAlignedStore(Src, Ptr, AlignVal);
    Store->takeName(CI);
    Store->copyMetadata(*CI);
    CI->eraseFromParent();
    return;
  }

  // Adjust alignment for the scalar instruction.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    //  %store i32 %OneElt, i32* %EltAddr
    //
    Instruction *ThenTerm =
        SplitBlockAndInsertIfThen(Predicate, InsertPt, /*Unreachable=*/false,
                                  /*BranchWeights=*/nullptr, DTU);

    BasicBlock *CondBlock = ThenTerm->getParent();
    CondBlock->setName("cond.store");

    Builder.SetInsertPoint(CondBlock->getTerminator());
    Value *OneElt = Builder.CreateExtractElement(Src, Idx);
    Value *Gep = Builder.CreateConstInBoundsGEP1_32(EltTy, Ptr, Idx);
    Builder.CreateAlignedStore(OneElt, Gep, AdjustedAlignVal);

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 430-537

```cpp
    // Create "else" block, fill it in the next iteration
    BasicBlock *NewIfBlock = ThenTerm->getSuccessor(0);
    NewIfBlock->setName("else");

    Builder.SetInsertPoint(NewIfBlock, NewIfBlock->begin());
  }
  CI->eraseFromParent();

  ModifiedDT = true;
}

// Translate a masked gather intrinsic like
// <16 x i32 > @llvm.masked.gather.v16i32( <16 x i32*> %Ptrs, i32 4,
//                               <16 x i1> %Mask, <16 x i32> %Src)
// to a chain of basic blocks, with loading element one-by-one if
// the appropriate mask bit is set
//
// %Ptrs = getelementptr i32, i32* %base, <16 x i64> %ind
// %Mask0 = extractelement <16 x i1> %Mask, i32 0
// br i1 %Mask0, label %cond.load, label %else
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    // On GPUs, use
    //  %cond = extrectelement %mask, Idx
    // instead

    Value *Predicate;
    if (SclrMask != nullptr) {
      Value *Mask = Builder.getInt(APInt::getOneBitSet(
          VectorWidth, adjustForEndian(DL, VectorWidth, Idx)));
      Predicate = Builder.CreateICmpNE(Builder.CreateAnd(SclrMask, Mask),
                                       Builder.getIntN(VectorWidth, 0));
    } else {
      Predicate = Builder.CreateExtractElement(Mask, Idx, "Mask" + Twine(Idx));
    }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DomTreeUpdater. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DomTreeUpdater 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 538-643

```cpp
    // Create "cond" block
    //
    //  %EltAddr = getelementptr i32* %1, i32 0
    //  %Elt = load i32* %EltAddr
    //  VResult = insertelement <16 x i32> VResult, i32 %Elt, i32 Idx
    //
    // We mark the branch weights as explicitly unknown given they would only
    // be derivable from the mask which we do not have VP information for.
    Instruction *ThenTerm =
        SplitBlockAndInsertIfThen(Predicate, InsertPt, /*Unreachable=*/false,
                                  getExplicitlyUnknownBranchWeightsIfProfiled(
                                      *CI->getFunction(), DEBUG_TYPE),
                                  DTU);

    BasicBlock *CondBlock = ThenTerm->getParent();
    CondBlock->setName("cond.load");

    Builder.SetInsertPoint(CondBlock->getTerminator());
    Value *Ptr = Builder.CreateExtractElement(Ptrs, Idx, "Ptr" + Twine(Idx));
    LoadInst *Load =
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // Shorten the way if the mask is a vector of constants.
  if (isConstantIntVector(Mask)) {
    for (unsigned Idx = 0; Idx < VectorWidth; ++Idx) {
      if (cast<Constant>(Mask)->getAggregateElement(Idx)->isNullValue())
        continue;
      Value *OneElt =
          Builder.CreateExtractElement(Src, Idx, "Elt" + Twine(Idx));
      Value *Ptr = Builder.CreateExtractElement(Ptrs, Idx, "Ptr" + Twine(Idx));
      Builder.CreateAlignedStore(OneElt, Ptr, AlignVal);
    }
    CI->eraseFromParent();
    return;
  }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DomTreeUpdater. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DomTreeUpdater 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 644-732

```cpp
  // If the mask is not v1i1, use scalar bit test operations. This generates
  // better results on X86 at least.
  Value *SclrMask = nullptr;
  if (VectorWidth != 1 && !HasBranchDivergence) {
    Type *SclrMaskTy = Builder.getIntNTy(VectorWidth);
    SclrMask = Builder.CreateBitCast(Mask, SclrMaskTy, "scalar_mask");
  }

  for (unsigned Idx = 0; Idx < VectorWidth; ++Idx) {
    // Fill the "else" block, created in the previous iteration
    //
    //  %Mask1 = and i16 %scalar_mask, i32 1 << Idx
    //  %cond = icmp ne i16 %mask_1, 0
    //  br i1 %Mask1, label %cond.store, label %else
    //
    // On GPUs, use
    //  %cond = extrectelement %mask, Idx
    // instead
    Value *Predicate;
    if (SclrMask != nullptr) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  BasicBlock *IfBlock = CI->getParent();

  Builder.SetInsertPoint(InsertPt);
  Builder.SetCurrentDebugLocation(CI->getDebugLoc());

  unsigned VectorWidth = VecType->getNumElements();

  // The result vector
  Value *VResult = PassThru;

  // Adjust alignment for the scalar instruction.
  const Align AdjustedAlignment =
      commonAlignment(Alignment, EltTy->getPrimitiveSizeInBits() / 8);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DomTreeUpdater. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DomTreeUpdater 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 733-835

```cpp
  // Shorten the way if the mask is a vector of constants.
  // Create a build_vector pattern, with loads/poisons as necessary and then
  // shuffle blend with the pass through value.
  if (isConstantIntVector(Mask)) {
    unsigned MemIndex = 0;
    VResult = PoisonValue::get(VecType);
    SmallVector<int, 16> ShuffleMask(VectorWidth, PoisonMaskElem);
    for (unsigned Idx = 0; Idx < VectorWidth; ++Idx) {
      Value *InsertElt;
      if (cast<Constant>(Mask)->getAggregateElement(Idx)->isNullValue()) {
        InsertElt = PoisonValue::get(EltTy);
        ShuffleMask[Idx] = Idx + VectorWidth;
      } else {
        Value *NewPtr =
            Builder.CreateConstInBoundsGEP1_32(EltTy, Ptr, MemIndex);
        InsertElt = Builder.CreateAlignedLoad(EltTy, NewPtr, AdjustedAlignment,
                                              "Load" + Twine(Idx));
        ShuffleMask[Idx] = Idx;
        ++MemIndex;
      }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    PHINode *ResultPhi = Builder.CreatePHI(VecType, 2, "res.phi.else");
    ResultPhi->addIncoming(NewVResult, CondBlock);
    ResultPhi->addIncoming(VResult, PrevIfBlock);
    VResult = ResultPhi;

    // Add a PHI for the pointer if this isn't the last iteration.
    if ((Idx + 1) != VectorWidth) {
      PHINode *PtrPhi = Builder.CreatePHI(Ptr->getType(), 2, "ptr.phi.else");
      PtrPhi->addIncoming(NewPtr, CondBlock);
      PtrPhi->addIncoming(Ptr, PrevIfBlock);
      Ptr = PtrPhi;
    }
  }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 836-939

```cpp
  CI->replaceAllUsesWith(VResult);
  CI->eraseFromParent();

  ModifiedDT = true;
}

static void scalarizeMaskedCompressStore(const DataLayout &DL,
                                         bool HasBranchDivergence, CallInst *CI,
                                         DomTreeUpdater *DTU,
                                         bool &ModifiedDT) {
  Value *Src = CI->getArgOperand(0);
  Value *Ptr = CI->getArgOperand(1);
  Value *Mask = CI->getArgOperand(2);
  Align Alignment = CI->getParamAlign(1).valueOrOne();

  auto *VecType = cast<FixedVectorType>(Src->getType());

  IRBuilder<> Builder(CI->getContext());
  Instruction *InsertPt = CI;
  BasicBlock *IfBlock = CI->getParent();
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    Value *OneElt = Builder.CreateExtractElement(Src, Idx);
    Builder.CreateAlignedStore(OneElt, Ptr, AdjustedAlignment);

    // Move the pointer if there are more blocks to come.
    Value *NewPtr;
    if ((Idx + 1) != VectorWidth)
      NewPtr = Builder.CreateConstInBoundsGEP1_32(EltTy, Ptr, 1);

    // Create "else" block, fill it in the next iteration
    BasicBlock *NewIfBlock = ThenTerm->getSuccessor(0);
    NewIfBlock->setName("else");
    BasicBlock *PrevIfBlock = IfBlock;
    IfBlock = NewIfBlock;

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DomTreeUpdater. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DomTreeUpdater 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 940-1042

```cpp
    Builder.SetInsertPoint(NewIfBlock, NewIfBlock->begin());

    // Add a PHI for the pointer if this isn't the last iteration.
    if ((Idx + 1) != VectorWidth) {
      PHINode *PtrPhi = Builder.CreatePHI(Ptr->getType(), 2, "ptr.phi.else");
      PtrPhi->addIncoming(NewPtr, CondBlock);
      PtrPhi->addIncoming(Ptr, PrevIfBlock);
      Ptr = PtrPhi;
    }
  }
  CI->eraseFromParent();

  ModifiedDT = true;
}

static void scalarizeMaskedVectorHistogram(const DataLayout &DL, CallInst *CI,
                                           DomTreeUpdater *DTU,
                                           bool &ModifiedDT) {
  // If we extend histogram to return a result someday (like the updated vector)
  // then we'll need to support it here.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    Value *UpdateOp =
        CreateHistogramUpdateValue(cast<IntrinsicInst>(CI), Load, Inc);
    Builder.CreateStore(UpdateOp, Ptr);

    // Create "else" block, fill it in the next iteration
    BasicBlock *NewIfBlock = ThenTerm->getSuccessor(0);
    NewIfBlock->setName("else");
    Builder.SetInsertPoint(NewIfBlock, NewIfBlock->begin());
  }

  CI->eraseFromParent();
  ModifiedDT = true;
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DomTreeUpdater. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DomTreeUpdater 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1043-1142

```cpp
static bool runImpl(Function &F, const TargetTransformInfo &TTI,
                    DominatorTree *DT) {
  std::optional<DomTreeUpdater> DTU;
  if (DT)
    DTU.emplace(DT, DomTreeUpdater::UpdateStrategy::Lazy);

  bool EverMadeChange = false;
  bool MadeChange = true;
  auto &DL = F.getDataLayout();
  bool HasBranchDivergence = TTI.hasBranchDivergence(&F);
  while (MadeChange) {
    MadeChange = false;
    for (BasicBlock &BB : llvm::make_early_inc_range(F)) {
      bool ModifiedDTOnIteration = false;
      MadeChange |= optimizeBlock(BB, ModifiedDTOnIteration, TTI, DL,
                                  HasBranchDivergence, DTU ? &*DTU : nullptr);

      // Restart BB iteration if the dominator tree of the Function was changed
      if (ModifiedDTOnIteration)
        break;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      return true;
    case Intrinsic::masked_load:
      // Scalarize unsupported vector masked load
      if (TTI.isLegalMaskedLoad(
              CI->getType(), CI->getParamAlign(0).valueOrOne(),
              cast<PointerType>(CI->getArgOperand(0)->getType())
                  ->getAddressSpace(),
              isConstantIntVector(CI->getArgOperand(1))
                  ? TTI::MaskKind::ConstantMask
                  : TTI::MaskKind::VariableOrConstantMask))
        return false;
      scalarizeMaskedLoad(DL, HasBranchDivergence, CI, DTU, ModifiedDT);
      return true;
    case Intrinsic::masked_store:
```
- EN: Core entities appearing here include runOnFunction, run, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DominatorTree, DomTreeUpdater, TargetTransformInfo.
- CN: 此处出现的核心实体包括 runOnFunction, run，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DominatorTree, DomTreeUpdater, TargetTransformInfo 等分析结果。

### Lines 1143-1192

```cpp
      if (TTI.isLegalMaskedStore(
              CI->getArgOperand(0)->getType(),
              CI->getParamAlign(1).valueOrOne(),
              cast<PointerType>(CI->getArgOperand(1)->getType())
                  ->getAddressSpace(),
              isConstantIntVector(CI->getArgOperand(2))
                  ? TTI::MaskKind::ConstantMask
                  : TTI::MaskKind::VariableOrConstantMask))
        return false;
      scalarizeMaskedStore(DL, HasBranchDivergence, CI, DTU, ModifiedDT);
      return true;
    case Intrinsic::masked_gather: {
      Align Alignment = CI->getParamAlign(0).valueOrOne();
      Type *LoadTy = CI->getType();
      if (TTI.isLegalMaskedGather(LoadTy, Alignment) &&
          !TTI.forceScalarizeMaskedGather(cast<VectorType>(LoadTy), Alignment))
        return false;
      scalarizeMaskedGather(DL, HasBranchDivergence, CI, DTU, ModifiedDT);
      return true;
    }
    case Intrinsic::masked_scatter: {
      Align Alignment = CI->getParamAlign(1).valueOrOne();
      Type *StoreTy = CI->getArgOperand(0)->getType();
      if (TTI.isLegalMaskedScatter(StoreTy, Alignment) &&
          !TTI.forceScalarizeMaskedScatter(cast<VectorType>(StoreTy),
                                           Alignment))
        return false;
      scalarizeMaskedScatter(DL, HasBranchDivergence, CI, DTU, ModifiedDT);
      return true;
    }
    case Intrinsic::masked_expandload:
      if (TTI.isLegalMaskedExpandLoad(
              CI->getType(),
              CI->getAttributes().getParamAttrs(0).getAlignment().valueOrOne()))
        return false;
      scalarizeMaskedExpandLoad(DL, HasBranchDivergence, CI, DTU, ModifiedDT);
      return true;
    case Intrinsic::masked_compressstore:
      if (TTI.isLegalMaskedCompressStore(
              CI->getArgOperand(0)->getType(),
              CI->getAttributes().getParamAttrs(1).getAlignment().valueOrOne()))
        return false;
      scalarizeMaskedCompressStore(DL, HasBranchDivergence, CI, DTU,
                                   ModifiedDT);
      return true;
    }
  }

  return false;
}
```
- EN: This region continues the ScalarizeMaskedMemIntrin implementation with local helper logic centered on TTI, PointerType, MaskKind, ConstantMask.
- CN: 这一段延续了 ScalarizeMaskedMemIntrin 的主体实现，围绕 TTI, PointerType, MaskKind, ConstantMask 等局部辅助逻辑展开。

## Key Concepts / 关键概念

- EN: Domain: `Scalar` focuses on scalar optimizations and loop/CFG transforms.
  - CN: 领域：`Scalar` 主要处理 标量优化以及循环/CFG 变换。
- EN: Primary entities: `ScalarizeMaskedMemIntrinLegacyPass, isConstantIntVector, runOnFunction, run` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`ScalarizeMaskedMemIntrinLegacyPass, isConstantIntVector, runOnFunction, run` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DataLayout, DominatorTree, DomTreeUpdater, TargetTransformInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DataLayout, DominatorTree, DomTreeUpdater, TargetTransformInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/Analysis/VectorUtils.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/Analysis/VectorUtils.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/BasicBlock.h`, `llvm/IR/Constant.h`, `llvm/IR/Constants.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/ProfDataUtils.h`, `llvm/IR/Type.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/BasicBlock.h`, `llvm/IR/Constant.h`, `llvm/IR/Constants.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/ProfDataUtils.h`, `llvm/IR/Type.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Casting.h`, `llvm/ADT/Twine.h`, `llvm/InitializePasses.h`, `llvm/Pass.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Casting.h`, `llvm/ADT/Twine.h`, `llvm/InitializePasses.h`, `llvm/Pass.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `cassert`, `optional` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`cassert`, `optional` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `DataLayout`, `DominatorTree`, `DomTreeUpdater`, `TargetTransformInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DataLayout`, `DominatorTree`, `DomTreeUpdater`, `TargetTransformInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
