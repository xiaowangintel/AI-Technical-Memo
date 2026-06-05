# LowerMemIntrinsics.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/LowerMemIntrinsics.cpp` | `llvm/lib/Transforms/Utils/LowerMemIntrinsics.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements Lower Mem Intrinsics within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 LowerMemIntrinsics 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-94

```cpp
//===- LowerMemIntrinsics.cpp ----------------------------------*- C++ -*--===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/LowerMemIntrinsics.h"
#include "llvm/Analysis/ScalarEvolution.h"
#include "llvm/Analysis/TargetTransformInfo.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/IntrinsicInst.h"
#include "llvm/IR/MDBuilder.h"
#include "llvm/IR/ProfDataUtils.h"
#include "llvm/ProfileData/InstrProf.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/MathExtras.h"
#include "llvm/Transforms/Utils/BasicBlockUtils.h"
#include "llvm/Transforms/Utils/LoopUtils.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  uint64_t Total = 0;
  SmallVector<InstrProfValueData> ProfData =
      getValueProfDataFromInst(I, InstrProfValueKind::IPVK_MemOPSize,
                               std::numeric_limits<uint32_t>::max(), Total);
  if (!Total)
    return std::nullopt;
  uint64_t TripCount = 0;
  for (const auto &P : ProfData)
    TripCount += P.Count * P.Value;
  return std::round(1.0 * TripCount / Total);
}

} // namespace

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 95-200

```cpp
/// Insert the control flow and loop counters for a memcpy/memset loop
/// expansion.
///
/// This function inserts IR corresponding to the following C code before
/// \p InsertBefore:
/// \code
/// LoopUnits = (Len / MainLoopStep) * MainLoopStep;
/// ResidualUnits = Len - LoopUnits;
/// MainLoopIndex = 0;
/// if (LoopUnits > 0) {
///   do {
///     // MainLoopIP
///     MainLoopIndex += MainLoopStep;
///   } while (MainLoopIndex < LoopUnits);
/// }
/// for (size_t i = 0; i < ResidualUnits; i += ResidualLoopStep) {
///   ResidualLoopIndex = LoopUnits + i;
///   // ResidualLoopIP
/// }
/// \endcode
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      // TODO: This could also use known bits to check if a non-constant loop
      // count is guaranteed to be a multiple of MainLoopStep, in which case we
      // could omit the residual loop. It's unclear if that is worthwhile.
    } else {
      ResidualUnits = getRuntimeLoopRemainder(PreLoopBuilder, Len,
                                              CIMainLoopStep, MainLoopStep);
      LoopUnits = getRuntimeLoopUnits(PreLoopBuilder, Len, CIMainLoopStep,
                                      MainLoopStep, ResidualUnits);
    }
  } else if (auto *CLen = dyn_cast<ConstantInt>(Len)) {
    MustTakeMainLoop = CLen->getZExtValue() > 0;
    MayTakeMainLoop = MustTakeMainLoop;
  }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 201-306

```cpp
  // The case where both loops are omitted (i.e., the length is known zero) is
  // already handled at the beginning of this function.
  assert((MayTakeMainLoop || MayTakeResidualLoop) &&
         "At least one of the loops must be generated");

  BasicBlock *MainLoopBB = nullptr;
  CondBrInst *MainLoopBr = nullptr;

  // Construct the main loop unless we statically known that it is not taken.
  if (MayTakeMainLoop) {
    MainLoopBB = BasicBlock::Create(Ctx, BBNamePrefix + "-expansion-main-body",
                                    ParentFunc, PostLoopBB);
    IRBuilder<> LoopBuilder(MainLoopBB);
    LoopBuilder.SetCurrentDebugLocation(DbgLoc);

    PHINode *LoopIndex = LoopBuilder.CreatePHI(LenType, 2, "loop-index");
    LEI.MainLoopIndex = LoopIndex;
    LoopIndex->addIncoming(ConstantInt::get(LenType, 0U), PreLoopBB);

    Value *NewIndex = LoopBuilder.CreateAdd(
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    IRBuilder<> ResBuilder(ResidualLoopBB);
    ResBuilder.SetCurrentDebugLocation(DbgLoc);
    PHINode *ResidualIndex =
        ResBuilder.CreatePHI(LenType, 2, "residual-loop-index");
    ResidualIndex->addIncoming(Zero, PredOfResLoopBody);

    // Add the offset at the end of the main loop to the loop counter of the
    // residual loop to get the proper index. If the main loop was omitted, we
    // can also omit the addition.
    if (MainLoopBB)
      LEI.ResidualLoopIndex = ResBuilder.CreateAdd(LoopUnits, ResidualIndex);
    else
      LEI.ResidualLoopIndex = ResidualIndex;

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 307-410

```cpp
    Value *ResNewIndex = ResBuilder.CreateAdd(
        ResidualIndex, ConstantInt::get(LenType, ResidualLoopStep));
    ResidualIndex->addIncoming(ResNewIndex, ResidualLoopBB);

    // One argument of the addition is a loop-variant PHI, so it must be an
    // Instruction (i.e., it cannot be a Constant).
    LEI.ResidualLoopIP = cast<Instruction>(ResNewIndex);

    // Stay in the residual loop until all ResidualUnits are handled.
    CondBrInst *BR = ResBuilder.CreateCondBr(
        ResBuilder.CreateICmpULT(ResNewIndex, ResidualUnits), ResidualLoopBB,
        PostLoopBB);

    if (ExpectedUnits.has_value()) {
      uint64_t BackedgeTakenCount =
          (ExpectedUnits.value() % MainLoopStep) / ResidualLoopStep;
      if (BackedgeTakenCount > 0)
        BackedgeTakenCount -= 1; // The last iteration goes to the False target.
      MDBuilder MDB(ParentFunc->getContext());
      setFittedBranchWeights(*BR, {BackedgeTakenCount, 1},
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                                     std::optional<uint64_t> AverageTripCount) {
  // No need to expand zero length copies.
  if (CopyLen->isZero())
    return;

  BasicBlock *PreLoopBB = InsertBefore->getParent();
  Function *ParentFunc = PreLoopBB->getParent();
  LLVMContext &Ctx = PreLoopBB->getContext();
  const DataLayout &DL = ParentFunc->getDataLayout();
  MDBuilder MDB(Ctx);
  MDNode *NewDomain = MDB.createAnonymousAliasScopeDomain("MemCopyDomain");
  StringRef Name = "MemCopyAliasScope";
  MDNode *NewScope = MDB.createAnonymousAliasScope(NewDomain, Name);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 411-517

```cpp
  unsigned SrcAS = cast<PointerType>(SrcAddr->getType())->getAddressSpace();
  unsigned DstAS = cast<PointerType>(DstAddr->getType())->getAddressSpace();

  Type *TypeOfCopyLen = CopyLen->getType();
  Type *LoopOpType = TTI.getMemcpyLoopLoweringType(
      Ctx, CopyLen, SrcAS, DstAS, SrcAlign, DstAlign, AtomicElementSize);
  assert((!AtomicElementSize || !LoopOpType->isVectorTy()) &&
         "Atomic memcpy lowering is not supported for vector operand type");

  Type *Int8Type = Type::getInt8Ty(Ctx);
  TypeSize LoopOpSize = DL.getTypeStoreSize(LoopOpType);
  assert(LoopOpSize.isFixed() && "LoopOpType cannot be a scalable vector type");
  assert((!AtomicElementSize || LoopOpSize % *AtomicElementSize == 0) &&
         "Atomic memcpy lowering is not supported for selected operand size");

  uint64_t LoopEndCount =
      alignDown(CopyLen->getZExtValue(), LoopOpSize.getFixedValue());

  // Skip the loop expansion entirely if the loop would never be taken.
  if (LoopEndCount != 0) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    if (!CanOverlap) {
      // Indicate that stores don't overlap loads.
      Store->setMetadata(LLVMContext::MD_noalias, MDNode::get(Ctx, NewScope));
    }
    if (AtomicElementSize) {
      Load->setAtomic(AtomicOrdering::Unordered);
      Store->setAtomic(AtomicOrdering::Unordered);
    }
    BytesCopied += OperandSize;
  }
  assert(BytesCopied == CopyLen->getZExtValue() &&
         "Bytes copied should match size in the call!");
}

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 518-619

```cpp
void llvm::createMemCpyLoopUnknownSize(
    Instruction *InsertBefore, Value *SrcAddr, Value *DstAddr, Value *CopyLen,
    Align SrcAlign, Align DstAlign, bool SrcIsVolatile, bool DstIsVolatile,
    bool CanOverlap, const TargetTransformInfo &TTI,
    std::optional<uint32_t> AtomicElementSize,
    std::optional<uint64_t> AverageTripCount) {
  BasicBlock *PreLoopBB = InsertBefore->getParent();
  Function *ParentFunc = PreLoopBB->getParent();
  const DataLayout &DL = ParentFunc->getDataLayout();
  LLVMContext &Ctx = PreLoopBB->getContext();
  MDBuilder MDB(Ctx);
  MDNode *NewDomain = MDB.createAnonymousAliasScopeDomain("MemCopyDomain");
  StringRef Name = "MemCopyAliasScope";
  MDNode *NewScope = MDB.createAnonymousAliasScope(NewDomain, Name);

  unsigned SrcAS = cast<PointerType>(SrcAddr->getType())->getAddressSpace();
  unsigned DstAS = cast<PointerType>(DstAddr->getType())->getAddressSpace();

  Type *LoopOpType = TTI.getMemcpyLoopLoweringType(
      Ctx, CopyLen, SrcAS, DstAS, SrcAlign, DstAlign, AtomicElementSize);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  Value *ResDstGEP = ResLoopBuilder.CreateInBoundsGEP(Int8Type, DstAddr,
                                                      LEI.ResidualLoopIndex);
  StoreInst *ResStore = ResLoopBuilder.CreateAlignedStore(
      ResLoad, ResDstGEP, ResDstAlign, DstIsVolatile);
  if (!CanOverlap) {
    // Indicate that stores don't overlap loads.
    ResStore->setMetadata(LLVMContext::MD_noalias, MDNode::get(Ctx, NewScope));
  }
  if (AtomicElementSize) {
    ResLoad->setAtomic(AtomicOrdering::Unordered);
    ResStore->setAtomic(AtomicOrdering::Unordered);
  }
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 620-721

```cpp
// If \p Addr1 and \p Addr2 are pointers to different address spaces, create an
// addresspacecast to obtain a pair of pointers in the same addressspace. The
// caller needs to ensure that addrspacecasting is possible.
// No-op if the pointers are in the same address space.
static std::pair<Value *, Value *>
tryInsertCastToCommonAddrSpace(IRBuilderBase &B, Value *Addr1, Value *Addr2,
                               const TargetTransformInfo &TTI) {
  Value *ResAddr1 = Addr1;
  Value *ResAddr2 = Addr2;

  unsigned AS1 = cast<PointerType>(Addr1->getType())->getAddressSpace();
  unsigned AS2 = cast<PointerType>(Addr2->getType())->getAddressSpace();
  if (AS1 != AS2) {
    if (TTI.isValidAddrSpaceCast(AS2, AS1))
      ResAddr2 = B.CreateAddrSpaceCast(Addr2, Addr1->getType());
    else if (TTI.isValidAddrSpaceCast(AS1, AS2))
      ResAddr1 = B.CreateAddrSpaceCast(Addr1, Addr2->getType());
    else
      llvm_unreachable("Can only lower memmove between address spaces if they "
                       "support addrspacecast");
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  Value *RuntimeLoopBytes = CopyLen;
  Value *RuntimeLoopRemainder = nullptr;
  Value *SkipResidualCondition = nullptr;
  if (RequiresResidual) {
    RuntimeLoopRemainder =
        getRuntimeLoopRemainder(PLBuilder, CopyLen, CILoopOpSize, LoopOpSize);
    RuntimeLoopBytes = getRuntimeLoopUnits(PLBuilder, CopyLen, CILoopOpSize,
                                           LoopOpSize, RuntimeLoopRemainder);
    SkipResidualCondition =
        PLBuilder.CreateICmpEQ(RuntimeLoopRemainder, Zero, "skip_residual");
  }
  Value *SkipMainCondition =
      PLBuilder.CreateICmpEQ(RuntimeLoopBytes, Zero, "skip_main");

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 722-823

```cpp
  // Create the a comparison of src and dst, based on which we jump to either
  // the forward-copy part of the function (if src >= dst) or the backwards-copy
  // part (if src < dst).
  // SplitBlockAndInsertIfThenElse conveniently creates the basic if-then-else
  // structure. Its block terminators (unconditional branches) are replaced by
  // the appropriate conditional branches when the loop is built.
  // If the pointers are in different address spaces, they need to be converted
  // to a compatible one. Cases where memory ranges in the different address
  // spaces cannot overlap are lowered as memcpy and not handled here.
  auto [CmpSrcAddr, CmpDstAddr] =
      tryInsertCastToCommonAddrSpace(PLBuilder, SrcAddr, DstAddr, TTI);
  Value *PtrCompare =
      PLBuilder.CreateICmpULT(CmpSrcAddr, CmpDstAddr, "compare_src_dst");
  Instruction *ThenTerm, *ElseTerm;
  SplitBlockAndInsertIfThenElse(PtrCompare, InsertBefore->getIterator(),
                                &ThenTerm, &ElseTerm);

  // If the LoopOpSize is greater than 1, each part of the function consists of
  // four blocks:
  //   memmove_copy_backwards:
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      // After the residual loop, go to an intermediate block.
      BasicBlock *IntermediateBB = BasicBlock::Create(
          F->getContext(), "memmove_bwd_middle", F, MainLoopBB);
      // Later code expects a terminator in the PredBB.
      IRBuilder<> IntermediateBuilder(IntermediateBB);
      IntermediateBuilder.SetCurrentDebugLocation(DbgLoc);
      IntermediateBuilder.CreateUnreachable();
      ResidualLoopBuilder.CreateCondBr(
          ResidualLoopBuilder.CreateICmpEQ(ResidualIndex, RuntimeLoopBytes),
          IntermediateBB, ResidualLoopBB);

      ResidualLoopPhi->addIncoming(ResidualIndex, ResidualLoopBB);
      ResidualLoopPhi->addIncoming(CopyLen, CopyBackwardsBB);

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 824-933

```cpp
      // How to get to the residual:
      CondBrInst *BrInst =
          CondBrInst::Create(SkipResidualCondition, IntermediateBB,
                             ResidualLoopBB, ThenTerm->getIterator());
      BrInst->setDebugLoc(DbgLoc);
      ThenTerm->eraseFromParent();

      PredBB = IntermediateBB;
    }

    // main loop
    IRBuilder<> MainLoopBuilder(MainLoopBB);
    MainLoopBuilder.SetCurrentDebugLocation(DbgLoc);
    PHINode *MainLoopPhi = MainLoopBuilder.CreatePHI(ILengthType, 0);
    Value *MainIndex =
        MainLoopBuilder.CreateSub(MainLoopPhi, CILoopOpSize, "bwd_main_index");
    Value *LoadGEP =
        MainLoopBuilder.CreateInBoundsGEP(Int8Type, SrcAddr, MainIndex);
    Value *Element = MainLoopBuilder.CreateAlignedLoad(
        LoopOpType, LoadGEP, PartSrcAlign, SrcIsVolatile, "element");
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                                                              ResidualLoopPhi);
      ResidualLoopBuilder.CreateAlignedStore(Element, StoreGEP,
                                             ResidualDstAlign, DstIsVolatile);
      Value *ResidualIndex =
          ResidualLoopBuilder.CreateAdd(ResidualLoopPhi, CIResidualLoopOpSize);
      ResidualLoopBuilder.CreateCondBr(
          ResidualLoopBuilder.CreateICmpEQ(ResidualIndex, CopyLen), ExitBB,
          ResidualLoopBB);
      ResidualLoopPhi->addIncoming(ResidualIndex, ResidualLoopBB);
      ResidualLoopPhi->addIncoming(RuntimeLoopBytes, IntermediateBB);
    }
  }
}

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 934-1033

```cpp
// Similar to createMemMoveLoopUnknownSize, only the trip counts are computed at
// compile time, obsolete loops and branches are omitted, and the residual code
// is straight-line code instead of a loop.
static void createMemMoveLoopKnownSize(Instruction *InsertBefore,
                                       Value *SrcAddr, Value *DstAddr,
                                       ConstantInt *CopyLen, Align SrcAlign,
                                       Align DstAlign, bool SrcIsVolatile,
                                       bool DstIsVolatile,
                                       const TargetTransformInfo &TTI) {
  // No need to expand zero length moves.
  if (CopyLen->isZero())
    return;

  Type *TypeOfCopyLen = CopyLen->getType();
  BasicBlock *OrigBB = InsertBefore->getParent();
  Function *F = OrigBB->getParent();
  const DataLayout &DL = F->getDataLayout();
  LLVMContext &Ctx = OrigBB->getContext();
  unsigned SrcAS = cast<PointerType>(SrcAddr->getType())->getAddressSpace();
  unsigned DstAS = cast<PointerType>(DstAddr->getType())->getAddressSpace();
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    // instructions as in the forward case, only in reverse. So we generate code
    // the same way, except that we change the IRBuilder insert point for each
    // load/store pair so that each one is inserted before the previous one
    // instead of after it.
    IRBuilder<> BwdResBuilder(CopyBackwardsBB,
                              CopyBackwardsBB->getFirstNonPHIIt());
    BwdResBuilder.SetCurrentDebugLocation(DbgLoc);
    SmallVector<Type *, 5> RemainingOps;
    TTI.getMemcpyLoopResidualLoweringType(RemainingOps, Ctx, RemainingBytes,
                                          SrcAS, DstAS, PartSrcAlign,
                                          PartDstAlign);
    for (auto *OpTy : RemainingOps) {
      // reverse the order of the emitted operations
      BwdResBuilder.SetInsertPoint(CopyBackwardsBB,
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1034-1143

```cpp
                                   CopyBackwardsBB->getFirstNonPHIIt());
      GenerateResidualLdStPair(OpTy, BwdResBuilder, BytesCopied);
    }
  }
  if (BytesCopiedInLoop != 0) {
    BasicBlock *LoopBB = CopyBackwardsBB;
    BasicBlock *PredBB = OrigBB;
    if (RemainingBytes != 0) {
      // if we introduce residual code, it needs its separate BB
      LoopBB = CopyBackwardsBB->splitBasicBlock(
          CopyBackwardsBB->getTerminator(), "memmove_bwd_loop");
      PredBB = CopyBackwardsBB;
    } else {
      CopyBackwardsBB->setName("memmove_bwd_loop");
    }
    IRBuilder<> LoopBuilder(LoopBB->getTerminator());
    LoopBuilder.SetCurrentDebugLocation(DbgLoc);
    PHINode *LoopPhi = LoopBuilder.CreatePHI(ILengthType, 0);
    Value *Index = LoopBuilder.CreateSub(LoopPhi, CILoopOpSize, "bwd_index");
    Value *LoadGEP = LoopBuilder.CreateInBoundsGEP(Int8Type, SrcAddr, Index);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  Value *Result = SetValue;
  if (DstSize != SetValueSize) {
    if (!SetValueType->isIntegerTy() && !SetValueType->isFloatingPointTy()) {
      // If the type cannot be put into a vector, bitcast to iN first.
      LLVMContext &Ctx = SetValue->getContext();
      Result = B.CreateBitCast(Result, Type::getIntNTy(Ctx, SetValueSize * 8),
                               "setvalue.toint");
    }
    // Form a sufficiently large vector consisting of SetValue, repeated.
    Result =
        B.CreateVectorSplat(DstSize / SetValueSize, Result, "setvalue.splat");
  }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1144-1248

```cpp
  // The value has the right size, but we might have to bitcast it to the right
  // type.
  Result = B.CreateBitCast(Result, DstType, "setvalue.splat.cast");
  return Result;
}

static void
createMemSetLoopKnownSize(Instruction *InsertBefore, Value *DstAddr,
                          ConstantInt *Len, Value *SetValue, Align DstAlign,
                          bool IsVolatile, const TargetTransformInfo *TTI,
                          std::optional<uint64_t> AverageTripCount) {
  // No need to expand zero length memsets.
  if (Len->isZero())
    return;

  BasicBlock *PreLoopBB = InsertBefore->getParent();
  Function *ParentFunc = PreLoopBB->getParent();
  const DataLayout &DL = ParentFunc->getDataLayout();
  LLVMContext &Ctx = PreLoopBB->getContext();

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    if (OpTy != PreviousOpTy)
      SplatSetValue = createMemSetSplat(DL, RBuilder, SetValue, OpTy);

    Value *DstGEP = RBuilder.CreateInBoundsGEP(
        Int8Type, DstAddr, ConstantInt::get(TypeOfLen, BytesSet));
    RBuilder.CreateAlignedStore(SplatSetValue, DstGEP, PartDstAlign,
                                IsVolatile);
    BytesSet += OperandSize;
    PreviousOpTy = OpTy;
  }
  assert(BytesSet == Len->getZExtValue() &&
         "Bytes set should match size in the call!");
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1249-1350

```cpp
static void
createMemSetLoopUnknownSize(Instruction *InsertBefore, Value *DstAddr,
                            Value *Len, Value *SetValue, Align DstAlign,
                            bool IsVolatile, const TargetTransformInfo *TTI,
                            std::optional<uint64_t> AverageTripCount) {
  BasicBlock *PreLoopBB = InsertBefore->getParent();
  Function *ParentFunc = PreLoopBB->getParent();
  const DataLayout &DL = ParentFunc->getDataLayout();
  LLVMContext &Ctx = PreLoopBB->getContext();

  unsigned DstAS = cast<PointerType>(DstAddr->getType())->getAddressSpace();

  Type *Int8Type = Type::getInt8Ty(Ctx);
  assert(SetValue->getType() == Int8Type && "Can only set bytes");

  Type *LoopOpType = Int8Type;
  if (TTI) {
    LoopOpType = TTI->getMemcpyLoopLoweringType(
        Ctx, Len, DstAS, DstAS, DstAlign, DstAlign, std::nullopt);
  }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  Type *OriginalType = SetValue->getType();
  TypeSize OriginalTypeStoreSize = DL.getTypeStoreSize(OriginalType);
  TypeSize OriginalTypeAllocSize = DL.getTypeAllocSize(OriginalType);

  // The semantics of memset.pattern restrict what vectorization we can do: It
  // has to behave like a series of stores of the SetValue type at offsets that
  // are spaced by the alloc size of the SetValue type. If store and alloc size
  // of the SetValue type don't match, the bytes that aren't covered by these
  // stores must not be overwritten. We therefore only vectorize memset.pattern
  // if the store and alloc sizes of the SetValue are equal and properly divide
  // the size of the preferred lowering type (and only if store and alloc size
  // for the preferred lowering type are also equal).

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1351-1455

```cpp
  unsigned MainLoopStep = 1;
  Type *MainLoopType = OriginalType;
  TypeSize MainLoopAllocSize = OriginalTypeAllocSize;
  unsigned ResidualLoopStep = 0;
  Type *ResidualLoopType = nullptr;

  if (PreferredLoopOpStoreSize == PreferredLoopOpAllocSize &&
      OriginalTypeStoreSize == OriginalTypeAllocSize &&
      OriginalTypeStoreSize < PreferredLoopOpStoreSize &&
      PreferredLoopOpStoreSize % OriginalTypeStoreSize == 0) {
    // Multiple instances of SetValue can be combined to reach the preferred
    // loop op size.
    MainLoopStep = PreferredLoopOpStoreSize / OriginalTypeStoreSize;
    MainLoopType = PreferredLoopOpType;
    MainLoopAllocSize = PreferredLoopOpStoreSize;

    ResidualLoopStep = 1;
    ResidualLoopType = OriginalType;
  }

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        /*SrcAddr=*/Memcpy->getRawSource(),
        /*DstAddr=*/Memcpy->getRawDest(),
        /*CopyLen=*/Memcpy->getLength(),
        /*SrcAlign=*/Memcpy->getSourceAlign().valueOrOne(),
        /*DstAlign=*/Memcpy->getDestAlign().valueOrOne(),
        /*SrcIsVolatile=*/Memcpy->isVolatile(),
        /*DstIsVolatile=*/Memcpy->isVolatile(),
        /*CanOverlap=*/CanOverlap,
        /*TTI=*/TTI,
        /*AtomicElementSize=*/std::nullopt,
        /*AverageTripCount=*/TripCount);
  }
}

```
- EN: Core entities appearing here include canOverlap, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as ScalarEvolution, SCEV, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 canOverlap，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 ScalarEvolution, SCEV, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1456-1557

```cpp
bool llvm::expandMemMoveAsLoop(MemMoveInst *Memmove,
                               const TargetTransformInfo &TTI) {
  Value *CopyLen = Memmove->getLength();
  Value *SrcAddr = Memmove->getRawSource();
  Value *DstAddr = Memmove->getRawDest();
  Align SrcAlign = Memmove->getSourceAlign().valueOrOne();
  Align DstAlign = Memmove->getDestAlign().valueOrOne();
  bool SrcIsVolatile = Memmove->isVolatile();
  bool DstIsVolatile = SrcIsVolatile;
  IRBuilder<> CastBuilder(Memmove);
  CastBuilder.SetCurrentDebugLocation(Memmove->getStableDebugLoc());

  unsigned SrcAS = SrcAddr->getType()->getPointerAddressSpace();
  unsigned DstAS = DstAddr->getType()->getPointerAddressSpace();
  if (SrcAS != DstAS) {
    if (!TTI.addrspacesMayAlias(SrcAS, DstAS)) {
      // We may not be able to emit a pointer comparison, but we don't have
      // to. Expand as memcpy.
      auto AverageTripCount = getAverageMemOpLoopTripCount(*Memmove);
      if (ConstantInt *CI = dyn_cast<ConstantInt>(CopyLen)) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

void llvm::expandMemSetPatternAsLoop(MemSetPatternInst *Memset,
                                     const TargetTransformInfo *TTI) {
  createMemSetPatternLoop(
      /*InsertBefore=*/Memset,
      /*DstAddr=*/Memset->getRawDest(),
      /*Len=*/Memset->getLength(),
      /*SetValue=*/Memset->getValue(),
      /*DstAlign=*/Memset->getDestAlign().valueOrOne(),
      /*IsVolatile=*/Memset->isVolatile(),
      /*TTI=*/TTI,
      /*AverageTripCount=*/getAverageMemOpLoopTripCount(*Memset));
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1558-1594

```cpp
void llvm::expandMemSetPatternAsLoop(MemSetPatternInst *MemSet,
                                     const TargetTransformInfo &TTI) {
  expandMemSetPatternAsLoop(MemSet, &TTI);
}

void llvm::expandAtomicMemCpyAsLoop(AnyMemCpyInst *AtomicMemcpy,
                                    const TargetTransformInfo &TTI,
                                    ScalarEvolution *SE) {
  assert(AtomicMemcpy->isAtomic());
  if (ConstantInt *CI = dyn_cast<ConstantInt>(AtomicMemcpy->getLength())) {
    createMemCpyLoopKnownSize(
        /*InsertBefore=*/AtomicMemcpy,
        /*SrcAddr=*/AtomicMemcpy->getRawSource(),
        /*DstAddr=*/AtomicMemcpy->getRawDest(),
        /*CopyLen=*/CI,
        /*SrcAlign=*/AtomicMemcpy->getSourceAlign().valueOrOne(),
        /*DstAlign=*/AtomicMemcpy->getDestAlign().valueOrOne(),
        /*SrcIsVolatile=*/AtomicMemcpy->isVolatile(),
        /*DstIsVolatile=*/AtomicMemcpy->isVolatile(),
        /*CanOverlap=*/false, // SrcAddr & DstAddr may not overlap by spec.
        /*TTI=*/TTI,
        /*AtomicElementSize=*/AtomicMemcpy->getElementSizeInBytes());
  } else {
    createMemCpyLoopUnknownSize(
        /*InsertBefore=*/AtomicMemcpy,
        /*SrcAddr=*/AtomicMemcpy->getRawSource(),
        /*DstAddr=*/AtomicMemcpy->getRawDest(),
        /*CopyLen=*/AtomicMemcpy->getLength(),
        /*SrcAlign=*/AtomicMemcpy->getSourceAlign().valueOrOne(),
        /*DstAlign=*/AtomicMemcpy->getDestAlign().valueOrOne(),
        /*SrcIsVolatile=*/AtomicMemcpy->isVolatile(),
        /*DstIsVolatile=*/AtomicMemcpy->isVolatile(),
        /*CanOverlap=*/false, // SrcAddr & DstAddr may not overlap by spec.
        /*TargetTransformInfo=*/TTI,
        /*AtomicElementSize=*/AtomicMemcpy->getElementSizeInBytes());
  }
}
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as ScalarEvolution, TargetTransformInfo.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 ScalarEvolution, TargetTransformInfo 等分析结果。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `LoopExpansionInfo, getAverageMemOpLoopTripCount, canOverlap` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`LoopExpansionInfo, getAverageMemOpLoopTripCount, canOverlap` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DataLayout, ScalarEvolution, SCEV, TargetTransformInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DataLayout, ScalarEvolution, SCEV, TargetTransformInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/ScalarEvolution.h`, `llvm/Analysis/TargetTransformInfo.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/ScalarEvolution.h`, `llvm/Analysis/TargetTransformInfo.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/IRBuilder.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/MDBuilder.h`, `llvm/IR/ProfDataUtils.h`, `llvm/Transforms/Utils/LowerMemIntrinsics.h`, `llvm/Transforms/Utils/BasicBlockUtils.h`, `llvm/Transforms/Utils/LoopUtils.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/IRBuilder.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/MDBuilder.h`, `llvm/IR/ProfDataUtils.h`, `llvm/Transforms/Utils/LowerMemIntrinsics.h`, `llvm/Transforms/Utils/BasicBlockUtils.h`, `llvm/Transforms/Utils/LoopUtils.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Debug.h`, `llvm/Support/MathExtras.h`, `llvm/ProfileData/InstrProf.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Debug.h`, `llvm/Support/MathExtras.h`, `llvm/ProfileData/InstrProf.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `limits`, `optional` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`limits`, `optional` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `DataLayout`, `ScalarEvolution`, `SCEV`, `TargetTransformInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DataLayout`, `ScalarEvolution`, `SCEV`, `TargetTransformInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
