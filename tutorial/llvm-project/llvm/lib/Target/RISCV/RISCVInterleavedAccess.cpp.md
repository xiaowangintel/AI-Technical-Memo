# RISCVInterleavedAccess.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInterleavedAccess.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements lowering and optimization of interleaved memory accesses for RISC-V. / 实现RISC-V 交错内存访问的下降与优化。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: File prologue and imported dependencies / 文件前言与导入依赖
```cpp
//===-- RISCVInterleavedAccess.cpp - RISC-V Interleaved Access Transform --===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Functions and callbacks related to the InterleavedAccessPass.
//
//===----------------------------------------------------------------------===//

#include "RISCV.h"
#include "RISCVISelLowering.h"
#include "RISCVSubtarget.h"
#include "llvm/Analysis/ValueTracking.h"
#include "llvm/Analysis/VectorUtils.h"
#include "llvm/CodeGen/ValueTypes.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/IntrinsicsRISCV.h"
#include "llvm/IR/Module.h"
#include "llvm/IR/PatternMatch.h"
```
**EN:** This opening block combines banner comments with the first wave of includes, giving readers both context and the main compile-time dependencies.

**CN:** 该开头区段同时包含说明性注释与首批 include，让读者先看到文件定位，再看到主要编译期依赖。

### Lines 25-39: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
using namespace llvm;

bool RISCVTargetLowering::isLegalInterleavedAccessType(
    VectorType *VTy, unsigned Factor, Align Alignment, unsigned AddrSpace,
    const DataLayout &DL) const {
  EVT VT = getValueType(DL, VTy);
  // Don't lower vlseg/vsseg for vector types that can't be split.
  if (!isTypeLegal(VT))
    return false;

  if (!isLegalElementTypeForRVV(VT.getScalarType()) ||
      !allowsMemoryAccessForAlignment(VTy->getContext(), DL, VT, AddrSpace,
                                      Alignment))
    return false;
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 40-58: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  MVT ContainerVT = VT.getSimpleVT();

  if (auto *FVTy = dyn_cast<FixedVectorType>(VTy)) {
    if (!Subtarget.useRVVForFixedLengthVectors())
      return false;
    // Sometimes the interleaved access pass picks up splats as interleaves of
    // one element. Don't lower these.
    if (FVTy->getNumElements() < 2)
      return false;

    ContainerVT = getContainerForFixedLengthVector(VT.getSimpleVT());
  }

  // Need to make sure that EMUL * NFIELDS ≤ 8
  auto [LMUL, Fractional] = RISCVVType::decodeVLMUL(getLMUL(ContainerVT));
  if (Fractional)
    return true;
  return Factor * LMUL <= 8;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 59-77: Definitions and supporting logic / 定义与支撑逻辑
```cpp

static const Intrinsic::ID FixedVlsegIntrIds[] = {
    Intrinsic::riscv_seg2_load_mask, Intrinsic::riscv_seg3_load_mask,
    Intrinsic::riscv_seg4_load_mask, Intrinsic::riscv_seg5_load_mask,
    Intrinsic::riscv_seg6_load_mask, Intrinsic::riscv_seg7_load_mask,
    Intrinsic::riscv_seg8_load_mask};

static const Intrinsic::ID FixedVlssegIntrIds[] = {
    Intrinsic::riscv_sseg2_load_mask, Intrinsic::riscv_sseg3_load_mask,
    Intrinsic::riscv_sseg4_load_mask, Intrinsic::riscv_sseg5_load_mask,
    Intrinsic::riscv_sseg6_load_mask, Intrinsic::riscv_sseg7_load_mask,
    Intrinsic::riscv_sseg8_load_mask};

static const Intrinsic::ID ScalableVlssegIntrIds[] = {
    Intrinsic::riscv_vlsseg2_mask, Intrinsic::riscv_vlsseg3_mask,
    Intrinsic::riscv_vlsseg4_mask, Intrinsic::riscv_vlsseg5_mask,
    Intrinsic::riscv_vlsseg6_mask, Intrinsic::riscv_vlsseg7_mask,
    Intrinsic::riscv_vlsseg8_mask};
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 78-95: Definitions and supporting logic / 定义与支撑逻辑
```cpp
static const Intrinsic::ID ScalableVlsegIntrIds[] = {
    Intrinsic::riscv_vlseg2_mask, Intrinsic::riscv_vlseg3_mask,
    Intrinsic::riscv_vlseg4_mask, Intrinsic::riscv_vlseg5_mask,
    Intrinsic::riscv_vlseg6_mask, Intrinsic::riscv_vlseg7_mask,
    Intrinsic::riscv_vlseg8_mask};

static const Intrinsic::ID FixedVssegIntrIds[] = {
    Intrinsic::riscv_seg2_store_mask, Intrinsic::riscv_seg3_store_mask,
    Intrinsic::riscv_seg4_store_mask, Intrinsic::riscv_seg5_store_mask,
    Intrinsic::riscv_seg6_store_mask, Intrinsic::riscv_seg7_store_mask,
    Intrinsic::riscv_seg8_store_mask};

static const Intrinsic::ID FixedVsssegIntrIds[] = {
    Intrinsic::riscv_sseg2_store_mask, Intrinsic::riscv_sseg3_store_mask,
    Intrinsic::riscv_sseg4_store_mask, Intrinsic::riscv_sseg5_store_mask,
    Intrinsic::riscv_sseg6_store_mask, Intrinsic::riscv_sseg7_store_mask,
    Intrinsic::riscv_sseg8_store_mask};
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 96-114: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
static const Intrinsic::ID ScalableVssegIntrIds[] = {
    Intrinsic::riscv_vsseg2_mask, Intrinsic::riscv_vsseg3_mask,
    Intrinsic::riscv_vsseg4_mask, Intrinsic::riscv_vsseg5_mask,
    Intrinsic::riscv_vsseg6_mask, Intrinsic::riscv_vsseg7_mask,
    Intrinsic::riscv_vsseg8_mask};

static bool isMultipleOfN(const Value *V, const DataLayout &DL, unsigned N) {
  assert(N);
  if (N == 1)
    return true;

  using namespace PatternMatch;
  // Right now we're only recognizing the simplest pattern.
  uint64_t C;
  if (match(V, m_CombineOr(m_ConstantInt(C),
                           m_NUWMul(m_Value(), m_ConstantInt(C)))) &&
      C && C % N == 0)
    return true;
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 115-128: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  if (isPowerOf2_32(N)) {
    KnownBits KB = llvm::computeKnownBits(V, DL);
    return KB.countMinTrailingZeros() >= Log2_32(N);
  }

  return false;
}

/// Do the common operand retrieval and validition required by the
/// routines below.
static bool getMemOperands(unsigned Factor, VectorType *VTy, Type *XLenTy,
                           Instruction *I, Value *&Ptr, Value *&Mask,
                           Value *&VL, Align &Alignment) {
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 129-151: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  IRBuilder<> Builder(I);
  const DataLayout &DL = I->getDataLayout();
  ElementCount EC = VTy->getElementCount();
  if (auto *LI = dyn_cast<LoadInst>(I)) {
    assert(LI->isSimple());
    Ptr = LI->getPointerOperand();
    Alignment = LI->getAlign();
    assert(!Mask && "Unexpected mask on a load");
    Mask = Builder.getAllOnesMask(EC);
    VL = isa<FixedVectorType>(VTy) ? Builder.CreateElementCount(XLenTy, EC)
                                   : Constant::getAllOnesValue(XLenTy);
    return true;
  }
  if (auto *SI = dyn_cast<StoreInst>(I)) {
    assert(SI->isSimple());
    Ptr = SI->getPointerOperand();
    Alignment = SI->getAlign();
    assert(!Mask && "Unexpected mask on a store");
    Mask = Builder.getAllOnesMask(EC);
    VL = isa<FixedVectorType>(VTy) ? Builder.CreateElementCount(XLenTy, EC)
                                   : Constant::getAllOnesValue(XLenTy);
    return true;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 152-165: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  auto *II = cast<IntrinsicInst>(I);
  switch (II->getIntrinsicID()) {
  default:
    llvm_unreachable("Unsupported intrinsic type");
  case Intrinsic::vp_load:
  case Intrinsic::vp_store: {
    auto *VPLdSt = cast<VPIntrinsic>(I);
    Ptr = VPLdSt->getMemoryPointerParam();
    Alignment = VPLdSt->getPointerAlignment().value_or(
        DL.getABITypeAlign(VTy->getElementType()));

    assert(Mask && "vp.load and vp.store needs a mask!");
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 166-179: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    Value *WideEVL = VPLdSt->getVectorLengthParam();
    // Conservatively check if EVL is a multiple of factor, otherwise some
    // (trailing) elements might be lost after the transformation.
    if (!isMultipleOfN(WideEVL, I->getDataLayout(), Factor))
      return false;

    auto *FactorC = ConstantInt::get(WideEVL->getType(), Factor);
    VL = Builder.CreateZExt(Builder.CreateExactUDiv(WideEVL, FactorC), XLenTy);
    return true;
  }
  case Intrinsic::masked_load: {
    Ptr = II->getOperand(0);
    Alignment = II->getParamAlign(0).valueOrOne();
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 180-193: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    if (!isa<UndefValue>(II->getOperand(2)))
      return false;

    assert(Mask && "masked.load needs a mask!");

    VL = isa<FixedVectorType>(VTy)
             ? Builder.CreateElementCount(XLenTy, VTy->getElementCount())
             : Constant::getAllOnesValue(XLenTy);
    return true;
  }
  case Intrinsic::masked_store: {
    Ptr = II->getOperand(1);
    Alignment = II->getParamAlign(1).valueOrOne();
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 194-221: Function implementation: Constant::getAllOnesValue / 函数实现：Constant::getAllOnesValue
```cpp
    assert(Mask && "masked.store needs a mask!");

    VL = isa<FixedVectorType>(VTy)
             ? Builder.CreateElementCount(XLenTy, VTy->getElementCount())
             : Constant::getAllOnesValue(XLenTy);
    return true;
  }
  }
}

/// Lower an interleaved load into a vlsegN intrinsic.
///
/// E.g. Lower an interleaved load (Factor = 2):
/// %wide.vec = load <8 x i32>, <8 x i32>* %ptr
/// %v0 = shuffle %wide.vec, undef, <0, 2, 4, 6>  ; Extract even elements
/// %v1 = shuffle %wide.vec, undef, <1, 3, 5, 7>  ; Extract odd elements
///
/// Into:
/// %ld2 = { <4 x i32>, <4 x i32> } call llvm.riscv.seg2.load.v4i32.p0.i64(
///                                        %ptr, i64 4)
/// %vec0 = extractelement { <4 x i32>, <4 x i32> } %ld2, i32 0
/// %vec1 = extractelement { <4 x i32>, <4 x i32> } %ld2, i32 1
bool RISCVTargetLowering::lowerInterleavedLoad(
    Instruction *Load, Value *Mask, ArrayRef<ShuffleVectorInst *> Shuffles,
    ArrayRef<unsigned> Indices, unsigned Factor, const APInt &GapMask) const {
  assert(Indices.size() == Shuffles.size());
  assert(GapMask.getBitWidth() == Factor);
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 222-236: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  // We only support cases where the skipped fields are the trailing ones.
  if (!GapMask.isMask())
    return false;
  IRBuilder<> Builder(Load);

  unsigned MaskFactor = GapMask.popcount();
  const DataLayout &DL = Load->getDataLayout();
  auto *VTy = cast<FixedVectorType>(Shuffles[0]->getType());
  auto *XLenTy = Builder.getIntNTy(Subtarget.getXLen());

  Value *Ptr, *VL;
  Align Alignment;
  if (!getMemOperands(MaskFactor, VTy, XLenTy, Load, Ptr, Mask, VL, Alignment))
    return false;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 237-254: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  Type *PtrTy = Ptr->getType();
  unsigned AS = PtrTy->getPointerAddressSpace();
  if (!isLegalInterleavedAccessType(VTy, MaskFactor, Alignment, AS, DL))
    return false;

  CallInst *SegLoad = nullptr;
  if (MaskFactor < Factor && MaskFactor != 1) {
    // Lower to strided segmented load.
    unsigned ScalarSizeInBytes = DL.getTypeStoreSize(VTy->getElementType());
    Value *Stride = ConstantInt::get(XLenTy, Factor * ScalarSizeInBytes);
    SegLoad = Builder.CreateIntrinsic(FixedVlssegIntrIds[MaskFactor - 2],
                                      {VTy, PtrTy, XLenTy, XLenTy},
                                      {Ptr, Stride, Mask, VL});
  } else {
    // Lower to normal segmented load.
    SegLoad = Builder.CreateIntrinsic(FixedVlsegIntrIds[Factor - 2],
                                      {VTy, PtrTy, XLenTy}, {Ptr, Mask, VL});
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 255-268: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  for (unsigned i = 0; i < Shuffles.size(); i++) {
    unsigned FactorIdx = Indices[i];
    if (FactorIdx >= MaskFactor) {
      // Replace masked-off factors (that are still extracted) with poison.
      Shuffles[i]->replaceAllUsesWith(PoisonValue::get(VTy));
    } else {
      Value *SubVec = Builder.CreateExtractValue(SegLoad, FactorIdx);
      Shuffles[i]->replaceAllUsesWith(SubVec);
    }
  }

  return true;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 269-292: Commentary and design intent / 注释与设计意图
```cpp

/// Lower an interleaved store into a vssegN intrinsic.
///
/// E.g. Lower an interleaved store (Factor = 3):
/// %i.vec = shuffle <8 x i32> %v0, <8 x i32> %v1,
///                  <0, 4, 8, 1, 5, 9, 2, 6, 10, 3, 7, 11>
/// store <12 x i32> %i.vec, <12 x i32>* %ptr
///
/// Into:
/// %sub.v0 = shuffle <8 x i32> %v0, <8 x i32> v1, <0, 1, 2, 3>
/// %sub.v1 = shuffle <8 x i32> %v0, <8 x i32> v1, <4, 5, 6, 7>
/// %sub.v2 = shuffle <8 x i32> %v0, <8 x i32> v1, <8, 9, 10, 11>
/// call void llvm.riscv.seg3.store.v4i32.p0.i64(%sub.v0, %sub.v1, %sub.v2,
///                                              %ptr, i32 4)
///
/// Note that the new shufflevectors will be removed and we'll only generate one
/// vsseg3 instruction in CodeGen.
bool RISCVTargetLowering::lowerInterleavedStore(Instruction *Store,
                                                Value *LaneMask,
                                                ShuffleVectorInst *SVI,
                                                unsigned Factor,
                                                const APInt &GapMask) const {
  assert(GapMask.getBitWidth() == Factor);
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 293-307: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  // We only support cases where the skipped fields are the trailing ones.
  // TODO: Lower to strided store if there is only a single active field.
  unsigned MaskFactor = GapMask.popcount();
  if (MaskFactor < 2 || !GapMask.isMask())
    return false;

  IRBuilder<> Builder(Store);
  const DataLayout &DL = Store->getDataLayout();
  auto Mask = SVI->getShuffleMask();
  auto *ShuffleVTy = cast<FixedVectorType>(SVI->getType());
  // Given SVI : <n*factor x ty>, then VTy : <n x ty>
  auto *VTy = FixedVectorType::get(ShuffleVTy->getElementType(),
                                   ShuffleVTy->getNumElements() / Factor);
  auto *XLenTy = Builder.getIntNTy(Subtarget.getXLen());
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 308-330: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  Value *Ptr, *VL;
  Align Alignment;
  if (!getMemOperands(MaskFactor, VTy, XLenTy, Store, Ptr, LaneMask, VL,
                      Alignment))
    return false;

  Type *PtrTy = Ptr->getType();
  unsigned AS = PtrTy->getPointerAddressSpace();
  if (!isLegalInterleavedAccessType(VTy, MaskFactor, Alignment, AS, DL))
    return false;

  Function *SegStoreFunc;
  if (MaskFactor < Factor)
    // Strided segmented store.
    SegStoreFunc = Intrinsic::getOrInsertDeclaration(
        Store->getModule(), FixedVsssegIntrIds[MaskFactor - 2],
        {VTy, PtrTy, XLenTy, XLenTy});
  else
    // Normal segmented store.
    SegStoreFunc = Intrinsic::getOrInsertDeclaration(
        Store->getModule(), FixedVssegIntrIds[Factor - 2],
        {VTy, PtrTy, XLenTy});
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 331-344: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  SmallVector<Value *, 10> Ops;
  SmallVector<int, 16> NewShuffleMask;

  for (unsigned i = 0; i < MaskFactor; i++) {
    // Collect shuffle mask for this lane.
    for (unsigned j = 0; j < VTy->getNumElements(); j++)
      NewShuffleMask.push_back(Mask[i + Factor * j]);

    Value *Shuffle = Builder.CreateShuffleVector(
        SVI->getOperand(0), SVI->getOperand(1), NewShuffleMask);
    Ops.push_back(Shuffle);

    NewShuffleMask.clear();
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 345-364: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  Ops.push_back(Ptr);
  if (MaskFactor < Factor) {
    // Insert the stride argument.
    unsigned ScalarSizeInBytes = DL.getTypeStoreSize(VTy->getElementType());
    Ops.push_back(ConstantInt::get(XLenTy, Factor * ScalarSizeInBytes));
  }
  Ops.append({LaneMask, VL});
  Builder.CreateCall(SegStoreFunc, Ops);

  return true;
}

bool RISCVTargetLowering::lowerDeinterleaveIntrinsicToLoad(
    Instruction *Load, Value *Mask, IntrinsicInst *DI,
    const APInt &GapMask) const {
  const unsigned Factor = getDeinterleaveIntrinsicFactor(DI->getIntrinsicID());
  assert(GapMask.getBitWidth() == Factor);
  if (Factor > 8)
    return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 365-379: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  // We only support cases where the skipped fields are the trailing ones.
  if (!GapMask.isMask())
    return false;
  IRBuilder<> Builder(Load);

  VectorType *ResVTy = getDeinterleavedVectorType(DI);

  unsigned MaskFactor = GapMask.getActiveBits();
  // For MaskFactor of 1, we still want to lower it with segmented load
  // (of the original Factor), because the sole field extraction will eventually
  // turn it into a strided load.
  bool UseStridedSeg = MaskFactor < Factor && MaskFactor > 1;
  const DataLayout &DL = Load->getDataLayout();
  auto *XLenTy = Builder.getIntNTy(Subtarget.getXLen());
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 380-404: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  Value *Ptr, *VL;
  Align Alignment;
  if (!getMemOperands(Factor, ResVTy, XLenTy, Load, Ptr, Mask, VL, Alignment))
    return false;

  Type *PtrTy = Ptr->getType();
  unsigned AS = PtrTy->getPointerAddressSpace();
  if (!isLegalInterleavedAccessType(ResVTy, Factor, Alignment, AS, DL))
    return false;

  unsigned ElementSizeInBytes = DL.getTypeStoreSize(ResVTy->getElementType());
  Value *Return;
  if (isa<FixedVectorType>(ResVTy)) {
    Value *SegLoad;
    if (UseStridedSeg) {
      // Lower to strided segmented load.
      Value *Stride = ConstantInt::get(XLenTy, Factor * ElementSizeInBytes);
      SegLoad = Builder.CreateIntrinsic(FixedVlssegIntrIds[MaskFactor - 2],
                                        {ResVTy, PtrTy, XLenTy, XLenTy},
                                        {Ptr, Stride, Mask, VL});
    } else {
      SegLoad =
          Builder.CreateIntrinsic(FixedVlsegIntrIds[Factor - 2],
                                  {ResVTy, PtrTy, XLenTy}, {Ptr, Mask, VL});
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 405-432: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

    if (MaskFactor != Factor) {
      // Replace masked-off factors with poisons.
      SmallVector<Type *, 8> AggrTypes{Factor, ResVTy};
      Return = PoisonValue::get(StructType::get(Load->getContext(), AggrTypes));
      for (unsigned I = 0; I < MaskFactor; ++I) {
        Value *SubVec = Builder.CreateExtractValue(SegLoad, I);
        Return = Builder.CreateInsertValue(Return, SubVec, I);
      }
    } else {
      Return = SegLoad;
    }
  } else {
    unsigned SEW = DL.getTypeSizeInBits(ResVTy->getElementType());
    unsigned NumElts = ResVTy->getElementCount().getKnownMinValue();
    Type *VecTupTy = TargetExtType::get(
        Load->getContext(), "riscv.vector.tuple",
        ScalableVectorType::get(Builder.getInt8Ty(), NumElts * SEW / 8),
        UseStridedSeg ? MaskFactor : Factor);
    Function *SegLoadFunc;
    if (UseStridedSeg) {
      // Lower to strided segmented load.
      SegLoadFunc = Intrinsic::getOrInsertDeclaration(
          Load->getModule(), ScalableVlssegIntrIds[MaskFactor - 2],
          {VecTupTy, PtrTy, XLenTy, Mask->getType()});
    } else {
      SegLoadFunc = Intrinsic::getOrInsertDeclaration(
          Load->getModule(), ScalableVlsegIntrIds[Factor - 2],
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 433-447: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
          {VecTupTy, PtrTy, Mask->getType(), VL->getType()});
    }

    SmallVector<Value *, 8> Operands = {
        PoisonValue::get(VecTupTy),
        Ptr,
        Mask,
        VL,
        ConstantInt::get(XLenTy,
                         RISCVVType::TAIL_AGNOSTIC | RISCVVType::MASK_AGNOSTIC),
        ConstantInt::get(XLenTy, Log2_64(SEW))};
    if (UseStridedSeg) {
      Value *Stride = ConstantInt::get(XLenTy, Factor * ElementSizeInBytes);
      Operands.insert(std::next(Operands.begin(), 2), Stride);
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 448-463: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

    CallInst *Vlseg = Builder.CreateCall(SegLoadFunc, Operands);

    SmallVector<Type *, 8> AggrTypes{Factor, ResVTy};
    Return = PoisonValue::get(StructType::get(Load->getContext(), AggrTypes));
    for (unsigned i = 0; i < MaskFactor; ++i) {
      Value *VecExtract = Builder.CreateIntrinsic(
          Intrinsic::riscv_tuple_extract, {ResVTy, VecTupTy},
          {Vlseg, Builder.getInt32(i)});
      Return = Builder.CreateInsertValue(Return, VecExtract, i);
    }
  }

  DI->replaceAllUsesWith(Return);
  return true;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 464-485: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

bool RISCVTargetLowering::lowerInterleaveIntrinsicToStore(
    Instruction *Store, Value *Mask, ArrayRef<Value *> InterleaveValues) const {
  unsigned Factor = InterleaveValues.size();
  if (Factor > 8)
    return false;

  IRBuilder<> Builder(Store);

  auto *InVTy = cast<VectorType>(InterleaveValues[0]->getType());
  const DataLayout &DL = Store->getDataLayout();
  Type *XLenTy = Builder.getIntNTy(Subtarget.getXLen());

  Value *Ptr, *VL;
  Align Alignment;
  if (!getMemOperands(Factor, InVTy, XLenTy, Store, Ptr, Mask, VL, Alignment))
    return false;
  Type *PtrTy = Ptr->getType();
  unsigned AS = Ptr->getType()->getPointerAddressSpace();
  if (!isLegalInterleavedAccessType(InVTy, Factor, Alignment, AS, DL))
    return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 486-500: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  if (isa<FixedVectorType>(InVTy)) {
    Function *VssegNFunc = Intrinsic::getOrInsertDeclaration(
        Store->getModule(), FixedVssegIntrIds[Factor - 2],
        {InVTy, PtrTy, XLenTy});
    SmallVector<Value *, 10> Ops(InterleaveValues);
    Ops.append({Ptr, Mask, VL});
    Builder.CreateCall(VssegNFunc, Ops);
    return true;
  }
  unsigned SEW = DL.getTypeSizeInBits(InVTy->getElementType());
  unsigned NumElts = InVTy->getElementCount().getKnownMinValue();
  Type *VecTupTy = TargetExtType::get(
      Store->getContext(), "riscv.vector.tuple",
      ScalableVectorType::get(Builder.getInt8Ty(), NumElts * SEW / 8), Factor);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 501-515: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  Value *StoredVal = PoisonValue::get(VecTupTy);
  for (unsigned i = 0; i < Factor; ++i)
    StoredVal = Builder.CreateIntrinsic(
        Intrinsic::riscv_tuple_insert, {VecTupTy, InVTy},
        {StoredVal, InterleaveValues[i], Builder.getInt32(i)});

  Function *VssegNFunc = Intrinsic::getOrInsertDeclaration(
      Store->getModule(), ScalableVssegIntrIds[Factor - 2],
      {VecTupTy, PtrTy, Mask->getType(), VL->getType()});

  Value *Operands[] = {StoredVal, Ptr, Mask, VL,
                       ConstantInt::get(XLenTy, Log2_64(SEW))};
  Builder.CreateCall(VssegNFunc, Operands);
  return true;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

## Key Concepts / 关键概念
- **RISC-V backend structure** / **RISC-V 后端结构**

## Dependencies / 依赖关系
- `RISCV.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVISelLowering.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVSubtarget.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/Analysis/ValueTracking.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/Analysis/VectorUtils.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/ValueTypes.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/IR/IRBuilder.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/IR/Instructions.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/IR/IntrinsicsRISCV.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/IR/Module.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/IR/PatternMatch.h` — Directly referenced by this file. / 该文件直接引用的依赖。
