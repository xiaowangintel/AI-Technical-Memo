# RISCVTargetTransformInfo.h — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVTargetTransformInfo.h`
- **Repository**: llvm/llvm-project
- **Purpose**: Declares interfaces, data structures, and helper APIs for RISC-V cost modeling and transformation guidance for IR optimizations. / 声明面向 IR 优化的 RISC-V 代价模型与变换指导所需的接口、数据结构与辅助 API。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Commentary and design intent / 注释与设计意图
```cpp
//===- RISCVTargetTransformInfo.h - RISC-V specific TTI ---------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
/// \file
/// This file defines a TargetTransformInfoImplBase conforming object specific
/// to the RISC-V target machine. It uses the target's detailed information to
/// provide more precise answers to certain TTI queries, while letting the
/// target independent and default TTI implementations handle the rest.
///
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 15-31: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp

#ifndef LLVM_LIB_TARGET_RISCV_RISCVTARGETTRANSFORMINFO_H
#define LLVM_LIB_TARGET_RISCV_RISCVTARGETTRANSFORMINFO_H

#include "RISCVSubtarget.h"
#include "RISCVTargetMachine.h"
#include "llvm/Analysis/TargetTransformInfo.h"
#include "llvm/CodeGen/BasicTTIImpl.h"
#include "llvm/IR/Function.h"
#include <optional>

namespace llvm {

class RISCVTTIImpl final : public BasicTTIImplBase<RISCVTTIImpl> {
  using BaseT = BasicTTIImplBase<RISCVTTIImpl>;
  using TTI = TargetTransformInfo;
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 32-49: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  friend BaseT;

  const RISCVSubtarget *ST;
  const RISCVTargetLowering *TLI;

  const RISCVSubtarget *getST() const { return ST; }
  const RISCVTargetLowering *getTLI() const { return TLI; }

  /// This function returns an estimate for VL to be used in VL based terms
  /// of the cost model.  For fixed length vectors, this is simply the
  /// vector length.  For scalable vectors, we return results consistent
  /// with getVScaleForTuning under the assumption that clients are also
  /// using that when comparing costs between scalar and vector representation.
  /// This does unfortunately mean that we can both undershoot and overshot
  /// the true cost significantly if getVScaleForTuning is wildly off for the
  /// actual target hardware.
  unsigned getEstimatedVLFor(VectorType *Ty) const;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 50-64: Commentary and design intent / 注释与设计意图
```cpp
  /// This function calculates the costs for one or more RVV opcodes based
  /// on the vtype and the cost kind.
  /// \param Opcodes A list of opcodes of the RVV instruction to evaluate.
  /// \param VT The MVT of vtype associated with the RVV instructions.
  /// For widening/narrowing instructions where the result and source types
  /// differ, it is important to check the spec to determine whether the vtype
  /// refers to the result or source type.
  /// \param CostKind The type of cost to compute.
  InstructionCost getRISCVInstructionCost(ArrayRef<unsigned> OpCodes, MVT VT,
                                          TTI::TargetCostKind CostKind) const;

  // Return the cost of generating a PC relative address
  InstructionCost
  getStaticDataAddrGenerationCost(const TTI::TargetCostKind CostKind) const;
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 65-79: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  /// Return the cost of accessing a constant pool entry of the specified
  /// type.
  InstructionCost getConstantPoolLoadCost(Type *Ty,
                                          TTI::TargetCostKind CostKind) const;

  /// If this shuffle can be lowered as a masked slide pair (at worst),
  /// return a cost for it.
  InstructionCost getSlideCost(FixedVectorType *Tp, ArrayRef<int> Mask,
                               TTI::TargetCostKind CostKind) const;

public:
  explicit RISCVTTIImpl(const RISCVTargetMachine *TM, const Function &F)
      : BaseT(TM, F.getDataLayout()), ST(TM->getSubtargetImpl(F)),
        TLI(ST->getTargetLowering()) {}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 80-94: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  /// Return the cost of materializing an immediate for a value operand of
  /// a store instruction.
  InstructionCost getStoreImmCost(Type *VecTy, TTI::OperandValueInfo OpInfo,
                                  TTI::TargetCostKind CostKind) const;

  InstructionCost getIntImmCost(const APInt &Imm, Type *Ty,
                                TTI::TargetCostKind CostKind) const override;
  InstructionCost getIntImmCostInst(unsigned Opcode, unsigned Idx,
                                    const APInt &Imm, Type *Ty,
                                    TTI::TargetCostKind CostKind,
                                    Instruction *Inst = nullptr) const override;
  InstructionCost
  getIntImmCostIntrin(Intrinsic::ID IID, unsigned Idx, const APInt &Imm,
                      Type *Ty, TTI::TargetCostKind CostKind) const override;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 95-113: Commentary and design intent / 注释与设计意图
```cpp
  /// \name EVL Support for predicated vectorization.
  /// Whether the target supports the %evl parameter of VP intrinsic efficiently
  /// in hardware. (see LLVM Language Reference - "Vector Predication
  /// Intrinsics",
  /// https://llvm.org/docs/LangRef.html#vector-predication-intrinsics and
  /// "IR-level VP intrinsics",
  /// https://llvm.org/docs/Proposals/VectorPredication.html#ir-level-vp-intrinsics).
  bool hasActiveVectorLength() const override;

  TargetTransformInfo::PopcntSupportKind
  getPopcntSupport(unsigned TyWidth) const override;

  InstructionCost getPartialReductionCost(
      unsigned Opcode, Type *InputTypeA, Type *InputTypeB, Type *AccumType,
      ElementCount VF, TTI::PartialReductionExtendKind OpAExtend,
      TTI::PartialReductionExtendKind OpBExtend, std::optional<unsigned> BinOp,
      TTI::TargetCostKind CostKind,
      std::optional<FastMathFlags> FMF) const override;
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 114-130: Function implementation: supportsScalableVectors / 函数实现：supportsScalableVectors
```cpp
  bool shouldExpandReduction(const IntrinsicInst *II) const override;
  bool supportsScalableVectors() const override {
    // VLEN=32 support is incomplete.
    return ST->hasVInstructions() &&
           (ST->getRealMinVLen() >= RISCV::RVVBitsPerBlock);
  }
  bool enableOrderedReductions() const override { return true; }
  bool enableScalableVectorization() const override {
    return ST->hasVInstructions();
  }
  bool preferTailFoldingOverEpilogue(TailFoldingInfo *TFI) const override {
    return ST->hasVInstructions();
  }
  TailFoldingStyle getPreferredTailFoldingStyle() const override {
    return ST->hasVInstructions() ? TailFoldingStyle::DataWithEVL
                                  : TailFoldingStyle::None;
  }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 131-148: Function implementation: preferEpilogueVectorization / 函数实现：preferEpilogueVectorization
```cpp
  std::optional<unsigned> getMaxVScale() const override;
  std::optional<unsigned> getVScaleForTuning() const override;

  TypeSize
  getRegisterBitWidth(TargetTransformInfo::RegisterKind K) const override;

  unsigned getRegUsageForType(Type *Ty) const override;

  unsigned getMaximumVF(unsigned ElemWidth, unsigned Opcode) const override;

  bool preferAlternateOpcodeVectorization() const override;

  bool preferEpilogueVectorization(ElementCount Iters) const override {
    // Epilogue vectorization is usually unprofitable - tail folding or
    // a smaller VF would have been better.  This a blunt hammer - we
    // should re-examine this once vectorization is better tuned.
    return false;
  }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 149-163: Function implementation: shouldConsiderVectorizationRegPressure / 函数实现：shouldConsiderVectorizationRegPressure
```cpp

  bool shouldConsiderVectorizationRegPressure() const override { return true; }

  InstructionCost
  getMemIntrinsicInstrCost(const MemIntrinsicCostAttributes &MICA,
                           TTI::TargetCostKind CostKind) const override;

  InstructionCost getMaskedMemoryOpCost(const MemIntrinsicCostAttributes &MICA,
                                        TTI::TargetCostKind CostKind) const;

  InstructionCost
  getPointersChainCost(ArrayRef<const Value *> Ptrs, const Value *Base,
                       const TTI::PointersChainInfo &Info, Type *AccessTy,
                       TTI::TargetCostKind CostKind) const override;
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 164-177: Function implementation: getMinVectorRegisterBitWidth / 函数实现：getMinVectorRegisterBitWidth
```cpp
  void getUnrollingPreferences(Loop *L, ScalarEvolution &SE,
                               TTI::UnrollingPreferences &UP,
                               OptimizationRemarkEmitter *ORE) const override;

  void getPeelingPreferences(Loop *L, ScalarEvolution &SE,
                             TTI::PeelingPreferences &PP) const override;

  bool getTgtMemIntrinsic(IntrinsicInst *Inst,
                          MemIntrinsicInfo &Info) const override;

  unsigned getMinVectorRegisterBitWidth() const override {
    return ST->useRVVForFixedLengthVectors() ? 16 : 0;
  }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 178-191: Function implementation / 函数实现
```cpp
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
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 192-207: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  InstructionCost
  getIntrinsicInstrCost(const IntrinsicCostAttributes &ICA,
                        TTI::TargetCostKind CostKind) const override;

  InstructionCost
  getAddressComputationCost(Type *PTy, ScalarEvolution *SE, const SCEV *Ptr,
                            TTI::TargetCostKind CostKind) const override;

  InstructionCost getInterleavedMemoryOpCost(
      unsigned Opcode, Type *VecTy, unsigned Factor, ArrayRef<unsigned> Indices,
      Align Alignment, unsigned AddressSpace, TTI::TargetCostKind CostKind,
      bool UseMaskForCond = false, bool UseMaskForGaps = false) const override;

  InstructionCost getGatherScatterOpCost(const MemIntrinsicCostAttributes &MICA,
                                         TTI::TargetCostKind CostKind) const;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 208-222: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  InstructionCost
  getExpandCompressMemoryOpCost(const MemIntrinsicCostAttributes &MICA,
                                TTI::TargetCostKind CostKind) const;

  InstructionCost getStridedMemoryOpCost(const MemIntrinsicCostAttributes &MICA,
                                         TTI::TargetCostKind CostKind) const;

  InstructionCost
  getCostOfKeepingLiveOverCall(ArrayRef<Type *> Tys) const override;

  InstructionCost
  getCastInstrCost(unsigned Opcode, Type *Dst, Type *Src,
                   TTI::CastContextHint CCH, TTI::TargetCostKind CostKind,
                   const Instruction *I = nullptr) const override;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 223-236: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  InstructionCost
  getMinMaxReductionCost(Intrinsic::ID IID, VectorType *Ty, FastMathFlags FMF,
                         TTI::TargetCostKind CostKind) const override;

  std::optional<InstructionCost> getCombinedArithmeticInstructionCost(
      unsigned ISDOpcode, Type *Ty, TTI::TargetCostKind CostKind,
      TTI::OperandValueInfo Opd1Info, TTI::OperandValueInfo Opd2Info,
      ArrayRef<const Value *> Args, const Instruction *CxtI) const;

  InstructionCost
  getArithmeticReductionCost(unsigned Opcode, VectorType *Ty,
                             std::optional<FastMathFlags> FMF,
                             TTI::TargetCostKind CostKind) const override;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 237-254: Function implementation / 函数实现
```cpp
  InstructionCost
  getExtendedReductionCost(unsigned Opcode, bool IsUnsigned, Type *ResTy,
                           VectorType *ValTy, std::optional<FastMathFlags> FMF,
                           TTI::TargetCostKind CostKind) const override;

  InstructionCost getMemoryOpCost(
      unsigned Opcode, Type *Src, Align Alignment, unsigned AddressSpace,
      TTI::TargetCostKind CostKind,
      TTI::OperandValueInfo OpdInfo = {TTI::OK_AnyValue, TTI::OP_None},
      const Instruction *I = nullptr) const override;

  InstructionCost getCmpSelInstrCost(
      unsigned Opcode, Type *ValTy, Type *CondTy, CmpInst::Predicate VecPred,
      TTI::TargetCostKind CostKind,
      TTI::OperandValueInfo Op1Info = {TTI::OK_AnyValue, TTI::OP_None},
      TTI::OperandValueInfo Op2Info = {TTI::OK_AnyValue, TTI::OP_None},
      const Instruction *I = nullptr) const override;
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 255-269: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  InstructionCost getCFInstrCost(unsigned Opcode, TTI::TargetCostKind CostKind,
                                 const Instruction *I = nullptr) const override;

  using BaseT::getVectorInstrCost;
  InstructionCost
  getVectorInstrCost(unsigned Opcode, Type *Val, TTI::TargetCostKind CostKind,
                     unsigned Index, const Value *Op0, const Value *Op1,
                     TTI::VectorInstrContext VIC =
                         TTI::VectorInstrContext::None) const override;

  InstructionCost
  getIndexedVectorInstrCostFromEnd(unsigned Opcode, Type *Val,
                                   TTI::TargetCostKind CostKind,
                                   unsigned Index) const override;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 270-284: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  InstructionCost getArithmeticInstrCost(
      unsigned Opcode, Type *Ty, TTI::TargetCostKind CostKind,
      TTI::OperandValueInfo Op1Info = {TTI::OK_AnyValue, TTI::OP_None},
      TTI::OperandValueInfo Op2Info = {TTI::OK_AnyValue, TTI::OP_None},
      ArrayRef<const Value *> Args = {},
      const Instruction *CxtI = nullptr) const override;

  bool isElementTypeLegalForScalableVector(Type *Ty) const override {
    return TLI->isLegalElementTypeForRVV(TLI->getValueType(DL, Ty));
  }

  bool isLegalMaskedLoadStore(Type *DataType, Align Alignment) const {
    if (!ST->hasVInstructions())
      return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 285-302: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    EVT DataTypeVT = TLI->getValueType(DL, DataType);

    // Only support fixed vectors if we know the minimum vector size.
    if (DataTypeVT.isFixedLengthVector() && !ST->useRVVForFixedLengthVectors())
      return false;

    EVT ElemType = DataTypeVT.getScalarType();
    if (!ST->enableUnalignedVectorMem() && Alignment < ElemType.getStoreSize())
      return false;

    return TLI->isLegalElementTypeForRVV(ElemType);
  }

  bool isLegalMaskedLoad(Type *DataType, Align Alignment,
                         unsigned /*AddressSpace*/,
                         TTI::MaskKind /*MaskKind*/) const override {
    return isLegalMaskedLoadStore(DataType, Alignment);
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 303-318: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  bool isLegalMaskedStore(Type *DataType, Align Alignment,
                          unsigned /*AddressSpace*/,
                          TTI::MaskKind /*MaskKind*/) const override {
    return isLegalMaskedLoadStore(DataType, Alignment);
  }

  bool isLegalMaskedGatherScatter(Type *DataType, Align Alignment) const {
    if (!ST->hasVInstructions())
      return false;

    EVT DataTypeVT = TLI->getValueType(DL, DataType);

    // Only support fixed vectors if we know the minimum vector size.
    if (DataTypeVT.isFixedLengthVector() && !ST->useRVVForFixedLengthVectors())
      return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 319-334: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    // We also need to check if the vector of address is valid.
    EVT PointerTypeVT = EVT(TLI->getPointerTy(DL));
    if (DataTypeVT.isScalableVector() &&
        !TLI->isLegalElementTypeForRVV(PointerTypeVT))
      return false;

    EVT ElemType = DataTypeVT.getScalarType();
    if (!ST->enableUnalignedVectorMem() && Alignment < ElemType.getStoreSize())
      return false;

    return TLI->isLegalElementTypeForRVV(ElemType);
  }

  bool isLegalMaskedGather(Type *DataType, Align Alignment) const override {
    return isLegalMaskedGatherScatter(DataType, Alignment);
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 335-349: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  bool isLegalMaskedScatter(Type *DataType, Align Alignment) const override {
    return isLegalMaskedGatherScatter(DataType, Alignment);
  }

  bool forceScalarizeMaskedGather(VectorType *VTy,
                                  Align Alignment) const override {
    // Scalarize masked gather for RV64 if EEW=64 indices aren't supported.
    return ST->is64Bit() && !ST->hasVInstructionsI64();
  }

  bool forceScalarizeMaskedScatter(VectorType *VTy,
                                   Align Alignment) const override {
    // Scalarize masked scatter for RV64 if EEW=64 indices aren't supported.
    return ST->is64Bit() && !ST->hasVInstructionsI64();
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 350-364: Function implementation: isLegalStridedLoadStore / 函数实现：isLegalStridedLoadStore
```cpp

  bool isLegalStridedLoadStore(Type *DataType, Align Alignment) const override {
    EVT DataTypeVT = TLI->getValueType(DL, DataType);
    return TLI->isLegalStridedLoadStore(DataTypeVT, Alignment);
  }

  bool isLegalInterleavedAccessType(VectorType *VTy, unsigned Factor,
                                    Align Alignment,
                                    unsigned AddrSpace) const override {
    return TLI->isLegalInterleavedAccessType(VTy, Factor, Alignment, AddrSpace,
                                             DL);
  }

  bool isLegalMaskedExpandLoad(Type *DataType, Align Alignment) const override;
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 365-392: Function implementation: getVPLegalizationStrategy / 函数实现：getVPLegalizationStrategy
```cpp
  bool isLegalMaskedCompressStore(Type *DataTy, Align Alignment) const override;

  /// \returns How the target needs this vector-predicated operation to be
  /// transformed.
  TargetTransformInfo::VPLegalization
  getVPLegalizationStrategy(const VPIntrinsic &PI) const override {
    using VPLegalization = TargetTransformInfo::VPLegalization;
    static const Intrinsic::ID Supported[] = {
        Intrinsic::experimental_vp_strided_load,
        Intrinsic::experimental_vp_strided_store,
        Intrinsic::experimental_vp_reverse,
        Intrinsic::experimental_vp_splice,
        Intrinsic::vp_cttz_elts,
        Intrinsic::vp_gather,
        Intrinsic::vp_load,
        Intrinsic::vp_load_ff,
        Intrinsic::vp_merge,
        Intrinsic::vp_reduce_add,
        Intrinsic::vp_reduce_and,
        Intrinsic::vp_reduce_fadd,
        Intrinsic::vp_reduce_fmax,
        Intrinsic::vp_reduce_fmaximum,
        Intrinsic::vp_reduce_fmin,
        Intrinsic::vp_reduce_fminimum,
        Intrinsic::vp_reduce_fmul,
        Intrinsic::vp_reduce_mul,
        Intrinsic::vp_reduce_or,
        Intrinsic::vp_reduce_smax,
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 393-411: Definitions and supporting logic / 定义与支撑逻辑
```cpp
        Intrinsic::vp_reduce_smin,
        Intrinsic::vp_reduce_umax,
        Intrinsic::vp_reduce_umin,
        Intrinsic::vp_reduce_xor,
        Intrinsic::vp_scatter,
        Intrinsic::vp_sdiv,
        Intrinsic::vp_srem,
        Intrinsic::vp_store,
        Intrinsic::vp_udiv,
        Intrinsic::vp_urem};
    if (!ST->hasVInstructions() ||
        (PI.getIntrinsicID() == Intrinsic::vp_reduce_mul &&
         cast<VectorType>(PI.getArgOperand(1)->getType())
                 ->getElementType()
                 ->getIntegerBitWidth() != 1) ||
        !is_contained(Supported, PI.getIntrinsicID()))
      return VPLegalization(VPLegalization::Discard, VPLegalization::Convert);
    return VPLegalization(VPLegalization::Legal, VPLegalization::Legal);
  }
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 412-439: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  bool isLegalToVectorizeReduction(const RecurrenceDescriptor &RdxDesc,
                                   ElementCount VF) const override {
    if (!VF.isScalable())
      return true;

    Type *Ty = RdxDesc.getRecurrenceType();
    if (!TLI->isLegalElementTypeForRVV(TLI->getValueType(DL, Ty)))
      return false;

    switch (RdxDesc.getRecurrenceKind()) {
    case RecurKind::Add:
    case RecurKind::Sub:
    case RecurKind::AddChainWithSubs:
    case RecurKind::And:
    case RecurKind::Or:
    case RecurKind::Xor:
    case RecurKind::SMin:
    case RecurKind::SMax:
    case RecurKind::UMin:
    case RecurKind::UMax:
    case RecurKind::FMin:
    case RecurKind::FMax:
    case RecurKind::FindLast:
      return true;
    case RecurKind::AnyOf:
    case RecurKind::FAdd:
    case RecurKind::FMulAdd:
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 440-457: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
      // We can't promote f16/bf16 fadd reductions and scalable vectors can't be
      // expanded.
      if (Ty->isBFloatTy() || (Ty->isHalfTy() && !ST->hasVInstructionsF16()))
        return false;
      return true;
    default:
      return false;
    }
  }

  unsigned getMaxInterleaveFactor(ElementCount VF) const override {
    // Don't interleave if the loop has been vectorized with scalable vectors.
    if (VF.isScalable())
      return 1;
    // If the loop will not be vectorized, don't interleave the loop.
    // Let regular unroll to unroll the loop.
    return VF.isScalar() ? 1 : ST->getMaxInterleaveFactor();
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 458-484: Enumeration and symbolic state / 枚举与符号状态
```cpp

  bool enableInterleavedAccessVectorization() const override { return true; }

  bool enableMaskedInterleavedAccessVectorization() const override {
    return ST->hasVInstructions();
  }

  unsigned getMinTripCountTailFoldingThreshold() const override;

  enum RISCVRegisterClass { GPRRC, FPRRC, VRRC };
  unsigned getNumberOfRegisters(unsigned ClassID) const override {
    switch (ClassID) {
    case RISCVRegisterClass::GPRRC:
      // 31 = 32 GPR - x0 (zero register)
      // FIXME: Should we exclude fixed registers like SP, TP or GP?
      return 31;
    case RISCVRegisterClass::FPRRC:
      if (ST->hasStdExtF())
        return 32;
      return 0;
    case RISCVRegisterClass::VRRC:
      // Although there are 32 vector registers, v0 is special in that it is the
      // only register that can be used to hold a mask.
      // FIXME: Should we conservatively return 31 as the number of usable
      // vector registers?
      return ST->hasVInstructions() ? 32 : 0;
    }
```
**EN:** This block defines named constants or state encodings that make later target logic more explicit and less error-prone.

**CN:** 该区段定义具名常量或状态编码，使后续目标相关逻辑更清晰且更不易出错。

### Lines 485-503: Type declaration for getRegisterClassForType / getRegisterClassForType 的类型声明
```cpp
    llvm_unreachable("unknown register class");
  }

  TTI::AddressingModeKind
  getPreferredAddressingMode(const Loop *L, ScalarEvolution *SE) const override;

  unsigned getRegisterClassForType(bool Vector,
                                   Type *Ty = nullptr) const override {
    if (Vector)
      return RISCVRegisterClass::VRRC;
    if (!Ty)
      return RISCVRegisterClass::GPRRC;

    Type *ScalarTy = Ty->getScalarType();
    if ((ScalarTy->isHalfTy() && ST->hasStdExtZfhmin()) ||
        (ScalarTy->isFloatTy() && ST->hasStdExtF()) ||
        (ScalarTy->isDoubleTy() && ST->hasStdExtD())) {
      return RISCVRegisterClass::FPRRC;
    }
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 504-518: Type declaration for getRegisterClassName / getRegisterClassName 的类型声明
```cpp

    return RISCVRegisterClass::GPRRC;
  }

  const char *getRegisterClassName(unsigned ClassID) const override {
    switch (ClassID) {
    case RISCVRegisterClass::GPRRC:
      return "RISCV::GPRRC";
    case RISCVRegisterClass::FPRRC:
      return "RISCV::FPRRC";
    case RISCVRegisterClass::VRRC:
      return "RISCV::VRRC";
    }
    llvm_unreachable("unknown register class");
  }
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 519-533: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  bool isLSRCostLess(const TargetTransformInfo::LSRCost &C1,
                     const TargetTransformInfo::LSRCost &C2) const override;

  bool shouldConsiderAddressTypePromotion(
      const Instruction &I,
      bool &AllowPromotionWithoutCommonHeader) const override;
  std::optional<unsigned> getMinPageSize() const override { return 4096; }
  /// Return true if the (vector) instruction I will be lowered to an
  /// instruction with a scalar splat operand for the given Operand number.
  bool canSplatOperand(Instruction *I, int Operand) const;
  /// Return true if a vector instruction will lower to a target instruction
  /// able to splat the given operand.
  bool canSplatOperand(unsigned Opcode, int Operand) const;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 534-549: Function implementation: enableSelectOptimize / 函数实现：enableSelectOptimize
```cpp
  bool isProfitableToSinkOperands(Instruction *I,
                                  SmallVectorImpl<Use *> &Ops) const override;

  TTI::MemCmpExpansionOptions
  enableMemCmpExpansion(bool OptSize, bool IsZeroCmp) const override;

  bool enableSelectOptimize() const override {
    return ST->enableSelectOptimize();
  }

  bool shouldTreatInstructionLikeSelect(const Instruction *I) const override;

  bool
  shouldCopyAttributeWhenOutliningFrom(const Function *Caller,
                                       const Attribute &Attr) const override;
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 550-556: Header guard and interface framing / 头文件保护与接口框架
```cpp
  std::optional<Instruction *>
  instCombineIntrinsic(InstCombiner &IC, IntrinsicInst &II) const override;
};

} // end namespace llvm

#endif // LLVM_LIB_TARGET_RISCV_RISCVTARGETTRANSFORMINFO_H
```
**EN:** This block establishes include guards and the outer structure of the public interface.

**CN:** 该区段建立头文件保护并给出公共接口的外层结构。

## Key Concepts / 关键概念
- **Cost modeling** / **代价建模**

## Dependencies / 依赖关系
- `RISCVSubtarget.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVTargetMachine.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/Analysis/TargetTransformInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/BasicTTIImpl.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/IR/Function.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `optional` — Directly referenced by this file. / 该文件直接引用的依赖。
