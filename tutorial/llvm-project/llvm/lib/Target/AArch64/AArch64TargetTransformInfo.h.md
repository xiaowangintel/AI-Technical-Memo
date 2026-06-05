# AArch64TargetTransformInfo.h — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/AArch64TargetTransformInfo.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This file covers AArch64 specific TTI. / 该文件实现 AArch64 后端中的目标代价建模。
## Line-by-Line Analysis / 逐行分析
### Lines 1-31: Documented code section
```cpp
//===- AArch64TargetTransformInfo.h - AArch64 specific TTI ------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
/// \file
/// This file a TargetTransformInfoImplBase conforming object specific to the
/// AArch64 target machine. It uses the target's detailed information to
/// provide more precise answers to certain TTI queries, while letting the
/// target independent and default TTI implementations handle the rest.
///
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AARCH64_AARCH64TARGETTRANSFORMINFO_H
#define LLVM_LIB_TARGET_AARCH64_AARCH64TARGETTRANSFORMINFO_H

#include "AArch64.h"
#include "AArch64Subtarget.h"
#include "AArch64TargetMachine.h"
#include "llvm/Analysis/TargetTransformInfo.h"
#include "llvm/CodeGen/BasicTTIImpl.h"
#include "llvm/IR/FMF.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/Intrinsics.h"
#include "llvm/Support/InstructionCost.h"
#include <cstdint>
#include <optional>

namespace llvm {
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 32-61: Class APInt
```cpp

class APInt;
class Instruction;
class IntrinsicInst;
class Loop;
class SCEV;
class ScalarEvolution;
class Type;
class Value;
class VectorType;

class AArch64TTIImpl final : public BasicTTIImplBase<AArch64TTIImpl> {
  using BaseT = BasicTTIImplBase<AArch64TTIImpl>;
  using TTI = TargetTransformInfo;

  friend BaseT;

  const AArch64Subtarget *ST;
  const AArch64TargetLowering *TLI;

  static const FeatureBitset InlineInverseFeatures;

  const AArch64Subtarget *getST() const { return ST; }
  const AArch64TargetLowering *getTLI() const { return TLI; }

  enum MemIntrinsicType {
    VECTOR_LDST_TWO_ELEMENTS,
    VECTOR_LDST_THREE_ELEMENTS,
    VECTOR_LDST_FOUR_ELEMENTS
  };
```
**EN:** This block defines APInt, packaging state and behavior that the file reuses for target cost modeling.  
**CN:** 该代码块定义 APInt，把 目标代价建模 所需的状态与行为封装在一起供后续复用。
### Lines 62-94: Documented code section
```cpp

  /// Given a add/sub/mul operation, detect a widening addl/subl/mull pattern
  /// where both operands can be treated like extends. Returns the minimal type
  /// needed to compute the operation.
  Type *isBinExtWideningInstruction(unsigned Opcode, Type *DstTy,
                                    ArrayRef<const Value *> Args,
                                    Type *SrcOverrideTy = nullptr) const;
  /// Given a add/sub operation with a single extend operand, detect a
  /// widening addw/subw pattern.
  bool isSingleExtWideningInstruction(unsigned Opcode, Type *DstTy,
                                      ArrayRef<const Value *> Args,
                                      Type *SrcOverrideTy = nullptr) const;

  // A helper function called by 'getVectorInstrCost'.
  //
  // 'Val' and 'Index' are forwarded from 'getVectorInstrCost';
  // \param ScalarUserAndIdx encodes the information about extracts from a
  /// vector with 'Scalar' being the value being extracted,'User' being the user
  /// of the extract(nullptr if user is not known before vectorization) and
  /// 'Idx' being the extract lane.
  InstructionCost getVectorInstrCostHelper(
      unsigned Opcode, Type *Val, TTI::TargetCostKind CostKind, unsigned Index,
      const Instruction *I = nullptr, Value *Scalar = nullptr,
      ArrayRef<std::tuple<Value *, User *, int>> ScalarUserAndIdx = {},
      TTI::VectorInstrContext VIC = TTI::VectorInstrContext::None) const;

public:
  explicit AArch64TTIImpl(const AArch64TargetMachine *TM, const Function &F)
      : BaseT(TM, F.getDataLayout()), ST(TM->getSubtargetImpl(F)),
        TLI(ST->getTargetLowering()) {}

  bool areInlineCompatible(const Function *Caller,
                           const Function *Callee) const override;
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 95-128: Core AArch64 backend logic
```cpp

  bool areTypesABICompatible(const Function *Caller, const Function *Callee,
                             ArrayRef<Type *> Types) const override;

  unsigned getInlineCallPenalty(const Function *F, const CallBase &Call,
                                unsigned DefaultCallPenalty) const override;

  APInt getFeatureMask(const Function &F) const override;
  APInt getPriorityMask(const Function &F) const override;

  bool isMultiversionedFunction(const Function &F) const override;

  /// \name Scalar TTI Implementations
  /// @{

  using BaseT::getIntImmCost;
  InstructionCost getIntImmCost(int64_t Val) const;
  InstructionCost getIntImmCost(const APInt &Imm, Type *Ty,
                                TTI::TargetCostKind CostKind) const override;
  InstructionCost getIntImmCostInst(unsigned Opcode, unsigned Idx,
                                    const APInt &Imm, Type *Ty,
                                    TTI::TargetCostKind CostKind,
                                    Instruction *Inst = nullptr) const override;
  InstructionCost
  getIntImmCostIntrin(Intrinsic::ID IID, unsigned Idx, const APInt &Imm,
                      Type *Ty, TTI::TargetCostKind CostKind) const override;
  TTI::PopcntSupportKind getPopcntSupport(unsigned TyWidth) const override;

  /// @}

  /// \name Vector TTI Implementations
  /// @{

  bool enableInterleavedAccessVectorization() const override { return true; }
```
**EN:** This block continues the file's main target cost modeling logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的目标代价建模主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 129-162: Function enableMaskedInterleavedAccessVectorization
```cpp

  bool enableMaskedInterleavedAccessVectorization() const override {
    return ST->hasSVE();
  }

  unsigned getNumberOfRegisters(unsigned ClassID) const override {
    bool Vector = (ClassID == 1);
    if (Vector) {
      if (ST->hasNEON())
        return 32;
      return 0;
    }
    return 31;
  }

  InstructionCost
  getIntrinsicInstrCost(const IntrinsicCostAttributes &ICA,
                        TTI::TargetCostKind CostKind) const override;

  std::optional<Instruction *>
  instCombineIntrinsic(InstCombiner &IC, IntrinsicInst &II) const override;

  std::optional<Value *> simplifyDemandedVectorEltsIntrinsic(
      InstCombiner &IC, IntrinsicInst &II, APInt DemandedElts, APInt &UndefElts,
      APInt &UndefElts2, APInt &UndefElts3,
      std::function<void(Instruction *, unsigned, APInt, APInt &)>
          SimplifyAndSetOp) const override;

  TypeSize
  getRegisterBitWidth(TargetTransformInfo::RegisterKind K) const override;

  unsigned getMinVectorRegisterBitWidth() const override {
    return ST->getMinVectorRegisterBitWidth();
  }
```
**EN:** This block implements enableMaskedInterleavedAccessVectorization, advancing the file's target cost modeling flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 enableMaskedInterleavedAccessVectorization，通过 AArch64 专用的决策与数据处理推进本文件的目标代价建模流程。
### Lines 163-196: Function getVScaleForTuning
```cpp

  std::optional<unsigned> getVScaleForTuning() const override {
    return ST->getVScaleForTuning();
  }

  bool shouldMaximizeVectorBandwidth(
      TargetTransformInfo::RegisterKind K) const override;

  /// Try to return an estimate cost factor that can be used as a multiplier
  /// when scalarizing an operation for a vector with ElementCount \p VF.
  /// For scalable vectors this currently takes the most pessimistic view based
  /// upon the maximum possible value for vscale.
  unsigned getMaxNumElements(ElementCount VF) const {
    if (!VF.isScalable())
      return VF.getFixedValue();

    return VF.getKnownMinValue() * ST->getVScaleForTuning();
  }

  unsigned getMaxInterleaveFactor(ElementCount VF) const override;

  bool prefersVectorizedAddressing() const override;

  /// Check whether Opcode1 has less throughput according to the scheduling
  /// model than Opcode2.
  bool hasKnownLowerThroughputFromSchedulingModel(unsigned Opcode1,
                                                  unsigned Opcode2) const;

  InstructionCost
  getMemIntrinsicInstrCost(const MemIntrinsicCostAttributes &MICA,
                           TTI::TargetCostKind CostKind) const override;

  InstructionCost getMaskedMemoryOpCost(const MemIntrinsicCostAttributes &MICA,
                                        TTI::TargetCostKind CostKind) const;
```
**EN:** This block implements getVScaleForTuning, advancing the file's target cost modeling flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 getVScaleForTuning，通过 AArch64 专用的决策与数据处理推进本文件的目标代价建模流程。
### Lines 197-221: Core AArch64 backend logic
```cpp

  InstructionCost getGatherScatterOpCost(const MemIntrinsicCostAttributes &MICA,
                                         TTI::TargetCostKind CostKind) const;

  bool isExtPartOfAvgExpr(const Instruction *ExtUser, Type *Dst,
                          Type *Src) const;

  InstructionCost
  getCastInstrCost(unsigned Opcode, Type *Dst, Type *Src,
                   TTI::CastContextHint CCH, TTI::TargetCostKind CostKind,
                   const Instruction *I = nullptr) const override;

  InstructionCost
  getExtractWithExtendCost(unsigned Opcode, Type *Dst, VectorType *VecTy,
                           unsigned Index,
                           TTI::TargetCostKind CostKind) const override;

  InstructionCost getCFInstrCost(unsigned Opcode, TTI::TargetCostKind CostKind,
                                 const Instruction *I = nullptr) const override;

  InstructionCost
  getVectorInstrCost(unsigned Opcode, Type *Val, TTI::TargetCostKind CostKind,
                     unsigned Index, const Value *Op0, const Value *Op1,
                     TTI::VectorInstrContext VIC =
                         TTI::VectorInstrContext::None) const override;
```
**EN:** This block continues the file's main target cost modeling logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的目标代价建模主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 222-254: Documented code section
```cpp

  /// \param ScalarUserAndIdx encodes the information about extracts from a
  /// vector with 'Scalar' being the value being extracted,'User' being the user
  /// of the extract(nullptr if user is not known before vectorization) and
  /// 'Idx' being the extract lane.
  InstructionCost getVectorInstrCost(
      unsigned Opcode, Type *Val, TTI::TargetCostKind CostKind, unsigned Index,
      Value *Scalar,
      ArrayRef<std::tuple<Value *, User *, int>> ScalarUserAndIdx,
      TTI::VectorInstrContext VIC =
          TTI::VectorInstrContext::None) const override;

  InstructionCost
  getVectorInstrCost(const Instruction &I, Type *Val,
                     TTI::TargetCostKind CostKind, unsigned Index,
                     TTI::VectorInstrContext VIC =
                         TTI::VectorInstrContext::None) const override;

  InstructionCost
  getIndexedVectorInstrCostFromEnd(unsigned Opcode, Type *Val,
                                   TTI::TargetCostKind CostKind,
                                   unsigned Index) const override;

  InstructionCost
  getMinMaxReductionCost(Intrinsic::ID IID, VectorType *Ty, FastMathFlags FMF,
                         TTI::TargetCostKind CostKind) const override;

  InstructionCost
  getArithmeticReductionCostSVE(unsigned Opcode, VectorType *ValTy,
                                TTI::TargetCostKind CostKind) const;

  InstructionCost getSpliceCost(VectorType *Tp, int Index,
                                TTI::TargetCostKind CostKind) const;
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 255-287: Core AArch64 backend logic
```cpp

  InstructionCost getArithmeticInstrCost(
      unsigned Opcode, Type *Ty, TTI::TargetCostKind CostKind,
      TTI::OperandValueInfo Op1Info = {TTI::OK_AnyValue, TTI::OP_None},
      TTI::OperandValueInfo Op2Info = {TTI::OK_AnyValue, TTI::OP_None},
      ArrayRef<const Value *> Args = {},
      const Instruction *CxtI = nullptr) const override;

  InstructionCost
  getAddressComputationCost(Type *PtrTy, ScalarEvolution *SE, const SCEV *Ptr,
                            TTI::TargetCostKind CostKind) const override;

  InstructionCost getCmpSelInstrCost(
      unsigned Opcode, Type *ValTy, Type *CondTy, CmpInst::Predicate VecPred,
      TTI::TargetCostKind CostKind,
      TTI::OperandValueInfo Op1Info = {TTI::OK_AnyValue, TTI::OP_None},
      TTI::OperandValueInfo Op2Info = {TTI::OK_AnyValue, TTI::OP_None},
      const Instruction *I = nullptr) const override;

  TTI::MemCmpExpansionOptions
  enableMemCmpExpansion(bool OptSize, bool IsZeroCmp) const override;
  bool useNeonVector(const Type *Ty) const;

  InstructionCost getMemoryOpCost(
      unsigned Opcode, Type *Src, Align Alignment, unsigned AddressSpace,
      TTI::TargetCostKind CostKind,
      TTI::OperandValueInfo OpInfo = {TTI::OK_AnyValue, TTI::OP_None},
      const Instruction *I = nullptr) const override;

  InstructionCost
  getCostOfKeepingLiveOverCall(ArrayRef<Type *> Tys) const override;

  bool isLegalMaskedExpandLoad(Type *DataTy, Align Alignment) const override;
```
**EN:** This block continues the file's main target cost modeling logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的目标代价建模主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 288-322: Core AArch64 backend logic
```cpp

  void getUnrollingPreferences(Loop *L, ScalarEvolution &SE,
                               TTI::UnrollingPreferences &UP,
                               OptimizationRemarkEmitter *ORE) const override;

  void getPeelingPreferences(Loop *L, ScalarEvolution &SE,
                             TTI::PeelingPreferences &PP) const override;

  Value *
  getOrCreateResultFromMemIntrinsic(IntrinsicInst *Inst, Type *ExpectedType,
                                    bool CanCreate = true) const override;

  bool getTgtMemIntrinsic(IntrinsicInst *Inst,
                          MemIntrinsicInfo &Info) const override;

  bool isElementTypeLegalForScalableVector(Type *Ty) const override {
    if (Ty->isPointerTy())
      return true;

    if (Ty->isBFloatTy() && ST->hasBF16())
      return true;

    if (Ty->isHalfTy() || Ty->isFloatTy() || Ty->isDoubleTy())
      return true;

    if (Ty->isIntegerTy(1) || Ty->isIntegerTy(8) || Ty->isIntegerTy(16) ||
        Ty->isIntegerTy(32) || Ty->isIntegerTy(64))
      return true;

    return false;
  }

  bool isLegalMaskedLoadStore(Type *DataType, Align Alignment) const {
    if (!ST->isSVEorStreamingSVEAvailable())
      return false;
```
**EN:** This block continues the file's main target cost modeling logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的目标代价建模主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 323-356: Function isLegalMaskedLoad
```cpp

    // For fixed vectors, avoid scalarization if using SVE for them.
    if (isa<FixedVectorType>(DataType) && !ST->useSVEForFixedLengthVectors() &&
        DataType->getPrimitiveSizeInBits() != 128)
      return false; // Fall back to scalarization of masked operations.

    return isElementTypeLegalForScalableVector(DataType->getScalarType());
  }

  bool isLegalMaskedLoad(Type *DataType, Align Alignment,
                         unsigned /*AddressSpace*/,
                         TTI::MaskKind /*MaskKind*/) const override {
    return isLegalMaskedLoadStore(DataType, Alignment);
  }

  bool isLegalMaskedStore(Type *DataType, Align Alignment,
                          unsigned /*AddressSpace*/,
                          TTI::MaskKind /*MaskKind*/) const override {
    return isLegalMaskedLoadStore(DataType, Alignment);
  }

  bool isElementTypeLegalForCompressStore(Type *Ty) const {
    return Ty->isFloatTy() || Ty->isDoubleTy() || Ty->isIntegerTy(32) ||
           Ty->isIntegerTy(64);
  }

  bool isLegalMaskedCompressStore(Type *DataType,
                                  Align Alignment) const override {
    if (!ST->isSVEAvailable())
      return false;

    if (isa<FixedVectorType>(DataType) &&
        DataType->getPrimitiveSizeInBits() < 128)
      return false;
```
**EN:** This block implements isLegalMaskedLoad, advancing the file's target cost modeling flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 isLegalMaskedLoad，通过 AArch64 专用的决策与数据处理推进本文件的目标代价建模流程。
### Lines 357-380: Function isLegalMaskedGatherScatter
```cpp

    return isElementTypeLegalForCompressStore(DataType->getScalarType());
  }

  bool isLegalMaskedGatherScatter(Type *DataType) const {
    if (!ST->isSVEAvailable())
      return false;

    // For fixed vectors, scalarize if not using SVE for them.
    auto *DataTypeFVTy = dyn_cast<FixedVectorType>(DataType);
    if (DataTypeFVTy && (!ST->useSVEForFixedLengthVectors() ||
                         DataTypeFVTy->getNumElements() < 2))
      return false;

    return isElementTypeLegalForScalableVector(DataType->getScalarType());
  }

  bool isLegalMaskedGather(Type *DataType, Align Alignment) const override {
    return isLegalMaskedGatherScatter(DataType);
  }

  bool isLegalMaskedScatter(Type *DataType, Align Alignment) const override {
    return isLegalMaskedGatherScatter(DataType);
  }
```
**EN:** This block implements isLegalMaskedGatherScatter, advancing the file's target cost modeling flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 isLegalMaskedGatherScatter，通过 AArch64 专用的决策与数据处理推进本文件的目标代价建模流程。
### Lines 381-409: Function isLegalBroadcastLoad
```cpp

  bool isLegalBroadcastLoad(Type *ElementTy,
                            ElementCount NumElements) const override {
    // Return true if we can generate a `ld1r` splat load instruction.
    if (!ST->hasNEON() || NumElements.isScalable())
      return false;
    switch (unsigned ElementBits = ElementTy->getScalarSizeInBits()) {
    case 8:
    case 16:
    case 32:
    case 64: {
      // We accept bit-widths >= 64bits and elements {8,16,32,64} bits.
      unsigned VectorBits = NumElements.getFixedValue() * ElementBits;
      return VectorBits >= 64;
    }
    }
    return false;
  }

  std::optional<bool> isLegalNTStoreLoad(Type *DataType,
                                         Align Alignment) const {
    // Currently we only support NT load and store lowering for little-endian
    // targets.
    //
    // Coordinated with LDNP and STNP constraints in
    // `llvm/lib/Target/AArch64/AArch64InstrInfo.td` and
    // `AArch64ISelLowering.cpp`
    if (!ST->isLittleEndian())
      return false;
```
**EN:** This block implements isLegalBroadcastLoad, advancing the file's target cost modeling flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 isLegalBroadcastLoad，通过 AArch64 专用的决策与数据处理推进本文件的目标代价建模流程。
### Lines 410-439: Documented code section
```cpp

    // NOTE: The logic below is mostly geared towards LV, which calls it with
    //       vectors with 2 elements. We might want to improve that, if other
    //       users show up.
    // Nontemporal vector loads/stores can be directly lowered to LDNP/STNP, if
    // the vector can be halved so that each half fits into a register. That's
    // the case if the element type fits into a register and the number of
    // elements is a power of 2 > 1.
    if (auto *DataTypeTy = dyn_cast<FixedVectorType>(DataType)) {
      unsigned NumElements = DataTypeTy->getNumElements();
      unsigned EltSize = DataTypeTy->getElementType()->getScalarSizeInBits();
      return NumElements > 1 && isPowerOf2_64(NumElements) && EltSize >= 8 &&
             EltSize <= 128 && isPowerOf2_64(EltSize);
    }
    return std::nullopt;
  }

  bool isLegalNTStore(Type *DataType, Align Alignment) const override {
    if (auto Result = isLegalNTStoreLoad(DataType, Alignment))
      return *Result;
    // Fallback to target independent logic
    return BaseT::isLegalNTStore(DataType, Alignment);
  }

  bool isLegalNTLoad(Type *DataType, Align Alignment) const override {
    if (auto Result = isLegalNTStoreLoad(DataType, Alignment))
      return *Result;
    // Fallback to target independent logic
    return BaseT::isLegalNTLoad(DataType, Alignment);
  }
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 440-474: Function getPartialReductionCost
```cpp

  InstructionCost getPartialReductionCost(
      unsigned Opcode, Type *InputTypeA, Type *InputTypeB, Type *AccumType,
      ElementCount VF, TTI::PartialReductionExtendKind OpAExtend,
      TTI::PartialReductionExtendKind OpBExtend, std::optional<unsigned> BinOp,
      TTI::TargetCostKind CostKind,
      std::optional<FastMathFlags> FMF) const override;

  bool enableOrderedReductions() const override { return true; }

  InstructionCost getInterleavedMemoryOpCost(
      unsigned Opcode, Type *VecTy, unsigned Factor, ArrayRef<unsigned> Indices,
      Align Alignment, unsigned AddressSpace, TTI::TargetCostKind CostKind,
      bool UseMaskForCond = false, bool UseMaskForGaps = false) const override;

  bool shouldConsiderAddressTypePromotion(
      const Instruction &I,
      bool &AllowPromotionWithoutCommonHeader) const override;

  bool shouldExpandReduction(const IntrinsicInst *II) const override {
    return false;
  }

  unsigned getGISelRematGlobalCost() const override { return 2; }

  unsigned getMinTripCountTailFoldingThreshold() const override {
    return ST->hasSVE() ? 5 : 0;
  }

  TailFoldingStyle getPreferredTailFoldingStyle() const override {
    return ST->hasSVE() ? TailFoldingStyle::DataAndControlFlow
                        : TailFoldingStyle::DataWithoutLaneMask;
  }

  bool preferFixedOverScalableIfEqualCost(bool IsEpilogue) const override;
```
**EN:** This block implements getPartialReductionCost, advancing the file's target cost modeling flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 getPartialReductionCost，通过 AArch64 专用的决策与数据处理推进本文件的目标代价建模流程。
### Lines 475-506: Function getEpilogueVectorizationMinVF
```cpp

  unsigned getEpilogueVectorizationMinVF() const override;

  bool preferTailFoldingOverEpilogue(TailFoldingInfo *TFI) const override;

  bool supportsScalableVectors() const override {
    return ST->isSVEorStreamingSVEAvailable();
  }

  bool enableScalableVectorization() const override;

  bool isLegalToVectorizeReduction(const RecurrenceDescriptor &RdxDesc,
                                   ElementCount VF) const override;

  bool preferPredicatedReductionSelect() const override { return ST->hasSVE(); }

  /// FP16 and BF16 operations are lowered to fptrunc(op(fpext, fpext) if the
  /// architecture features are not present.
  std::optional<InstructionCost> getFP16BF16PromoteCost(
      Type *Ty, TTI::TargetCostKind CostKind, TTI::OperandValueInfo Op1Info,
      TTI::OperandValueInfo Op2Info, bool IncludeTrunc, bool CanUseSVE,
      std::function<InstructionCost(Type *)> InstCost) const;

  InstructionCost
  getArithmeticReductionCost(unsigned Opcode, VectorType *Ty,
                             std::optional<FastMathFlags> FMF,
                             TTI::TargetCostKind CostKind) const override;

  InstructionCost
  getExtendedReductionCost(unsigned Opcode, bool IsUnsigned, Type *ResTy,
                           VectorType *ValTy, std::optional<FastMathFlags> FMF,
                           TTI::TargetCostKind CostKind) const override;
```
**EN:** This block implements getEpilogueVectorizationMinVF, advancing the file's target cost modeling flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 getEpilogueVectorizationMinVF，通过 AArch64 专用的决策与数据处理推进本文件的目标代价建模流程。
### Lines 507-540: Function getMulAccReductionCost
```cpp

  InstructionCost getMulAccReductionCost(
      bool IsUnsigned, unsigned RedOpcode, Type *ResTy, VectorType *Ty,
      TTI::TargetCostKind CostKind = TTI::TCK_RecipThroughput) const override;

  InstructionCost
  getShuffleCost(TTI::ShuffleKind Kind, VectorType *DstTy, VectorType *SrcTy,
                 ArrayRef<int> Mask, TTI::TargetCostKind CostKind, int Index,
                 VectorType *SubTp, ArrayRef<const Value *> Args = {},
                 const Instruction *CxtI = nullptr) const override;

  InstructionCost
  getScalarizationOverhead(VectorType *Ty, const APInt &DemandedElts,
                           bool Insert, bool Extract,
                           TTI::TargetCostKind CostKind,
                           bool ForPoisonSrc = true, ArrayRef<Value *> VL = {},
                           TTI::VectorInstrContext VIC =
                               TTI::VectorInstrContext::None) const override;

  /// Return the cost of the scaling factor used in the addressing
  /// mode represented by AM for this target, for a load/store
  /// of the specified type.
  /// If the AM is supported, the return value must be >= 0.
  /// If the AM is not supported, it returns an invalid cost.
  InstructionCost getScalingFactorCost(Type *Ty, GlobalValue *BaseGV,
                                       StackOffset BaseOffset, bool HasBaseReg,
                                       int64_t Scale,
                                       unsigned AddrSpace) const override;

  bool enableSelectOptimize() const override {
    return ST->enableSelectOptimize();
  }

  bool shouldTreatInstructionLikeSelect(const Instruction *I) const override;
```
**EN:** This block implements getMulAccReductionCost, advancing the file's target cost modeling flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 getMulAccReductionCost，通过 AArch64 专用的决策与数据处理推进本文件的目标代价建模流程。
### Lines 541-569: Function getStoreMinimumVF
```cpp

  unsigned getStoreMinimumVF(unsigned VF, Type *ScalarMemTy, Type *ScalarValTy,
                             Align Alignment,
                             unsigned AddrSpace) const override {
    // We can vectorize store v4i8.
    if (ScalarMemTy->isIntegerTy(8) && isPowerOf2_32(VF) && VF >= 4)
      return 4;

    return BaseT::getStoreMinimumVF(VF, ScalarMemTy, ScalarValTy, Alignment,
                                    AddrSpace);
  }

  std::optional<unsigned> getMinPageSize() const override { return 4096; }

  bool isLSRCostLess(const TargetTransformInfo::LSRCost &C1,
                     const TargetTransformInfo::LSRCost &C2) const override;

  bool isProfitableToSinkOperands(Instruction *I,
                                  SmallVectorImpl<Use *> &Ops) const override;

  bool enableAggressiveInterleaving(bool) const override {
    return ST->enableAggressiveInterleaving();
  }
  /// @}
};

} // end namespace llvm

#endif // LLVM_LIB_TARGET_AARCH64_AARCH64TARGETTRANSFORMINFO_H
```
**EN:** This block implements getStoreMinimumVF, advancing the file's target cost modeling flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 getStoreMinimumVF，通过 AArch64 专用的决策与数据处理推进本文件的目标代价建模流程。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
- **EN:** Cost modeling used by mid-level optimizers **CN:** 供中层优化器使用的代价建模
## Dependencies / 依赖关系
- **EN:** Target-local includes: AArch64.h, AArch64Subtarget.h, AArch64TargetMachine.h **CN:** 目标本地依赖：AArch64.h, AArch64Subtarget.h, AArch64TargetMachine.h
- **EN:** Core LLVM interfaces: llvm/Analysis/TargetTransformInfo.h, llvm/CodeGen/BasicTTIImpl.h, llvm/IR/FMF.h, llvm/IR/Function.h, llvm/IR/Intrinsics.h, llvm/Support/InstructionCost.h **CN:** 核心 LLVM 接口：llvm/Analysis/TargetTransformInfo.h, llvm/CodeGen/BasicTTIImpl.h, llvm/IR/FMF.h, llvm/IR/Function.h, llvm/IR/Intrinsics.h, llvm/Support/InstructionCost.h
- **EN:** Standard-library support: cstdint, optional **CN:** 标准库支持：cstdint, optional
- **EN:** Closely connected with neighboring AArch64 backend components responsible for target cost modeling. **CN:** 与周边负责目标代价建模的 AArch64 后端组件紧密协作。
