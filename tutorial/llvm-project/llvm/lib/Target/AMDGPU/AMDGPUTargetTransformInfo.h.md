# AMDGPUTargetTransformInfo.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUTargetTransformInfo.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for AMDGPUTargetTransformInfo in the LLVM AMDGPU backend. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM AMDGPU 后端中 AMDGPUTargetTransformInfo 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-26: File banner, includes, and setup
```cpp
//===- AMDGPUTargetTransformInfo.h - AMDGPU specific TTI --------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// This file a TargetTransformInfoImplBase conforming object specific to the
/// AMDGPU target machine. It uses the target's detailed information to
/// provide more precise answers to certain TTI queries, while letting the
/// target independent and default TTI implementations handle the rest.
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPU_AMDGPUTARGETTRANSFORMINFO_H
#define LLVM_LIB_TARGET_AMDGPU_AMDGPUTARGETTRANSFORMINFO_H

#include "AMDGPU.h"
#include "llvm/CodeGen/BasicTTIImpl.h"
#include "llvm/Support/AMDGPUAddrSpace.h"
#include <optional>

namespace llvm {

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 27-59: Declares class AMDGPUTargetMachine
```cpp
class AMDGPUTargetMachine;
class GCNSubtarget;
class InstCombiner;
class Loop;
class ScalarEvolution;
class SITargetLowering;
class Type;
class Value;

class AMDGPUTTIImpl final : public BasicTTIImplBase<AMDGPUTTIImpl> {
  using BaseT = BasicTTIImplBase<AMDGPUTTIImpl>;
  using TTI = TargetTransformInfo;

  friend BaseT;

  Triple TargetTriple;

  const TargetSubtargetInfo *ST;
  const TargetLoweringBase *TLI;

  const TargetSubtargetInfo *getST() const { return ST; }
  const TargetLoweringBase *getTLI() const { return TLI; }

public:
  explicit AMDGPUTTIImpl(const AMDGPUTargetMachine *TM, const Function &F);

  void getUnrollingPreferences(Loop *L, ScalarEvolution &SE,
                               TTI::UnrollingPreferences &UP,
                               OptimizationRemarkEmitter *ORE) const override;

  void getPeelingPreferences(Loop *L, ScalarEvolution &SE,
                             TTI::PeelingPreferences &PP) const override;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUTargetMachine`, `GCNSubtarget`, `InstCombiner`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUTargetMachine`, `GCNSubtarget`, `InstCombiner`。

### Lines 60-90: Declares class GCNTTIImpl
```cpp
  uint64_t getMaxMemIntrinsicInlineSizeThreshold() const override;
};

class GCNTTIImpl final : public BasicTTIImplBase<GCNTTIImpl> {
  using BaseT = BasicTTIImplBase<GCNTTIImpl>;
  using TTI = TargetTransformInfo;

  friend BaseT;

  const GCNSubtarget *ST;
  const SITargetLowering *TLI;
  AMDGPUTTIImpl CommonTTI;
  bool IsGraphics;
  bool HasFP32Denormals;
  bool HasFP64FP16Denormals;
  static constexpr bool InlinerVectorBonusPercent = 0;

  static const FeatureBitset InlineFeatureIgnoreList;

  const GCNSubtarget *getST() const { return ST; }
  const SITargetLowering *getTLI() const { return TLI; }

  static inline int getFullRateInstrCost() {
    return TargetTransformInfo::TCC_Basic;
  }

  static inline int getHalfRateInstrCost(TTI::TargetCostKind CostKind) {
    return CostKind == TTI::TCK_CodeSize ? 2
                                         : 2 * TargetTransformInfo::TCC_Basic;
  }

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `GCNTTIImpl`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`GCNTTIImpl`。

### Lines 91-122: Defines getQuarterRateInstrCost
```cpp
  // TODO: The size is usually 8 bytes, but takes 4x as many cycles. Maybe
  // should be 2 or 4.
  static inline int getQuarterRateInstrCost(TTI::TargetCostKind CostKind) {
    return CostKind == TTI::TCK_CodeSize ? 2
                                         : 4 * TargetTransformInfo::TCC_Basic;
  }

  int getTransInstrCost(TTI::TargetCostKind CostKind) const;

  // On some parts, normal fp64 operations are half rate, and others
  // quarter. This also applies to some integer operations.
  int get64BitInstrCost(TTI::TargetCostKind CostKind) const;

  std::pair<InstructionCost, MVT> getTypeLegalizationCost(Type *Ty) const;

  /// \returns true if V might be divergent even when all of its operands
  /// are uniform.
  bool isSourceOfDivergence(const Value *V) const;

  /// Returns true for the target specific set of operations which produce
  /// uniform result even taking non-uniform arguments.
  bool isAlwaysUniform(const Value *V) const;

public:
  explicit GCNTTIImpl(const AMDGPUTargetMachine *TM, const Function &F);

  bool hasBranchDivergence(const Function *F = nullptr) const override;

  void getUnrollingPreferences(Loop *L, ScalarEvolution &SE,
                               TTI::UnrollingPreferences &UP,
                               OptimizationRemarkEmitter *ORE) const override;

```
**EN:** This section contains concrete logic for getQuarterRateInstrCost. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 getQuarterRateInstrCost 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 123-156: Defines getPeelingPreferences
```cpp
  void getPeelingPreferences(Loop *L, ScalarEvolution &SE,
                             TTI::PeelingPreferences &PP) const override;

  TTI::PopcntSupportKind getPopcntSupport(unsigned TyWidth) const override {
    assert(isPowerOf2_32(TyWidth) && "Ty width must be power of 2");
    return TTI::PSK_FastHardware;
  }

  unsigned getNumberOfRegisters(unsigned RCID) const override;
  TypeSize
  getRegisterBitWidth(TargetTransformInfo::RegisterKind Vector) const override;
  unsigned getMinVectorRegisterBitWidth() const override;
  unsigned getMaximumVF(unsigned ElemWidth, unsigned Opcode) const override;
  unsigned getLoadVectorFactor(unsigned VF, unsigned LoadSize,
                               unsigned ChainSizeInBytes,
                               VectorType *VecTy) const override;
  unsigned getStoreVectorFactor(unsigned VF, unsigned StoreSize,
                                unsigned ChainSizeInBytes,
                                VectorType *VecTy) const override;
  unsigned getLoadStoreVecRegBitWidth(unsigned AddrSpace) const override;

  bool isLegalToVectorizeMemChain(unsigned ChainSizeInBytes, Align Alignment,
                                  unsigned AddrSpace) const;
  bool isLegalToVectorizeLoadChain(unsigned ChainSizeInBytes, Align Alignment,
                                   unsigned AddrSpace) const override;
  bool isLegalToVectorizeStoreChain(unsigned ChainSizeInBytes, Align Alignment,
                                    unsigned AddrSpace) const override;

  uint64_t getMaxMemIntrinsicInlineSizeThreshold() const override;
  Type *getMemcpyLoopLoweringType(
      LLVMContext &Context, Value *Length, unsigned SrcAddrSpace,
      unsigned DestAddrSpace, Align SrcAlign, Align DestAlign,
      std::optional<uint32_t> AtomicElementSize) const override;

```
**EN:** This section contains concrete logic for getPeelingPreferences. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 getPeelingPreferences 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 157-188: Defines getMemcpyLoopResidualLoweringType
```cpp
  void getMemcpyLoopResidualLoweringType(
      SmallVectorImpl<Type *> &OpsOut, LLVMContext &Context,
      unsigned RemainingBytes, unsigned SrcAddrSpace, unsigned DestAddrSpace,
      Align SrcAlign, Align DestAlign,
      std::optional<uint32_t> AtomicCpySize) const override;
  unsigned getMaxInterleaveFactor(ElementCount VF) const override;

  bool getTgtMemIntrinsic(IntrinsicInst *Inst,
                          MemIntrinsicInfo &Info) const override;

  InstructionCost getArithmeticInstrCost(
      unsigned Opcode, Type *Ty, TTI::TargetCostKind CostKind,
      TTI::OperandValueInfo Op1Info = {TTI::OK_AnyValue, TTI::OP_None},
      TTI::OperandValueInfo Op2Info = {TTI::OK_AnyValue, TTI::OP_None},
      ArrayRef<const Value *> Args = {},
      const Instruction *CxtI = nullptr) const override;

  InstructionCost getCFInstrCost(unsigned Opcode, TTI::TargetCostKind CostKind,
                                 const Instruction *I = nullptr) const override;

  bool isInlineAsmSourceOfDivergence(const CallInst *CI,
                                     ArrayRef<unsigned> Indices = {}) const;

  using BaseT::getVectorInstrCost;
  InstructionCost
  getVectorInstrCost(unsigned Opcode, Type *ValTy, TTI::TargetCostKind CostKind,
                     unsigned Index, const Value *Op0, const Value *Op1,
                     TTI::VectorInstrContext VIC =
                         TTI::VectorInstrContext::None) const override;

  bool isReadRegisterSourceOfDivergence(const IntrinsicInst *ReadReg) const;

```
**EN:** This section contains concrete logic for getMemcpyLoopResidualLoweringType. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 getMemcpyLoopResidualLoweringType 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 189-221: Defines isValidAddrSpaceCast
```cpp
  bool isValidAddrSpaceCast(unsigned FromAS, unsigned ToAS) const override {
    // Address space casts must cast between different address spaces.
    if (FromAS == ToAS)
      return false;

    // Casts between any aliasing address spaces are valid.
    return AMDGPU::addrspacesMayAlias(FromAS, ToAS);
  }

  bool addrspacesMayAlias(unsigned AS0, unsigned AS1) const override {
    return AMDGPU::addrspacesMayAlias(AS0, AS1);
  }

  unsigned getFlatAddressSpace() const override {
    // Don't bother running InferAddressSpaces pass on graphics shaders which
    // don't use flat addressing.
    if (IsGraphics)
      return -1;
    return AMDGPUAS::FLAT_ADDRESS;
  }

  bool collectFlatAddressOperands(SmallVectorImpl<int> &OpIndexes,
                                  Intrinsic::ID IID) const override;

  bool
  canHaveNonUndefGlobalInitializerInAddressSpace(unsigned AS) const override {
    return AS != AMDGPUAS::LOCAL_ADDRESS && AS != AMDGPUAS::REGION_ADDRESS &&
           AS != AMDGPUAS::PRIVATE_ADDRESS;
  }

  Value *rewriteIntrinsicWithAddressSpace(IntrinsicInst *II, Value *OldV,
                                          Value *NewV) const override;

```
**EN:** This section contains concrete logic for isValidAddrSpaceCast. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPU::addrspacesMayAlias`.
**CN:** 本节包含与 isValidAddrSpaceCast 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPU::addrspacesMayAlias`。

### Lines 222-255: Defines canSimplifyLegacyMulToMul
```cpp
  bool canSimplifyLegacyMulToMul(const Instruction &I, const Value *Op0,
                                 const Value *Op1, InstCombiner &IC) const;

  bool simplifyDemandedLaneMaskArg(InstCombiner &IC, IntrinsicInst &II,
                                   unsigned LaneAgIdx) const;

  std::optional<Instruction *>
  instCombineIntrinsic(InstCombiner &IC, IntrinsicInst &II) const override;

  Value *simplifyAMDGCNLaneIntrinsicDemanded(InstCombiner &IC,
                                             IntrinsicInst &II,
                                             const APInt &DemandedElts,
                                             APInt &UndefElts) const;

  Instruction *hoistLaneIntrinsicThroughOperand(InstCombiner &IC,
                                                IntrinsicInst &II) const;

  std::optional<Value *> simplifyDemandedVectorEltsIntrinsic(
      InstCombiner &IC, IntrinsicInst &II, APInt DemandedElts, APInt &UndefElts,
      APInt &UndefElts2, APInt &UndefElts3,
      std::function<void(Instruction *, unsigned, APInt, APInt &)>
          SimplifyAndSetOp) const override;

  InstructionCost getVectorSplitCost() const { return 0; }

  InstructionCost
  getShuffleCost(TTI::ShuffleKind Kind, VectorType *DstTy, VectorType *SrcTy,
                 ArrayRef<int> Mask, TTI::TargetCostKind CostKind, int Index,
                 VectorType *SubTp, ArrayRef<const Value *> Args = {},
                 const Instruction *CxtI = nullptr) const override;

  bool isProfitableToSinkOperands(Instruction *I,
                                  SmallVectorImpl<Use *> &Ops) const override;

```
**EN:** This section contains concrete logic for canSimplifyLegacyMulToMul. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 canSimplifyLegacyMulToMul 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 256-289: Defines areInlineCompatible
```cpp
  bool areInlineCompatible(const Function *Caller,
                           const Function *Callee) const override;

  int getInliningLastCallToStaticBonus() const override;
  unsigned getInliningThresholdMultiplier() const override { return 11; }
  unsigned adjustInliningThreshold(const CallBase *CB) const override;
  unsigned getCallerAllocaCost(const CallBase *CB,
                               const AllocaInst *AI) const override;

  int getInlinerVectorBonusPercent() const override {
    return InlinerVectorBonusPercent;
  }

  InstructionCost
  getArithmeticReductionCost(unsigned Opcode, VectorType *Ty,
                             std::optional<FastMathFlags> FMF,
                             TTI::TargetCostKind CostKind) const override;

  InstructionCost getPartialReductionCost(
      unsigned Opcode, Type *InputTypeA, Type *InputTypeB, Type *AccumType,
      ElementCount VF, TTI::PartialReductionExtendKind OpAExtend,
      TTI::PartialReductionExtendKind OpBExtend, std::optional<unsigned> BinOp,
      TTI::TargetCostKind CostKind,
      std::optional<FastMathFlags> FMF) const override {
    return InstructionCost::getInvalid();
  }

  InstructionCost
  getIntrinsicInstrCost(const IntrinsicCostAttributes &ICA,
                        TTI::TargetCostKind CostKind) const override;
  InstructionCost
  getMinMaxReductionCost(Intrinsic::ID IID, VectorType *Ty, FastMathFlags FMF,
                         TTI::TargetCostKind CostKind) const override;

```
**EN:** This section contains concrete logic for areInlineCompatible. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `InstructionCost::getInvalid`.
**CN:** 本节包含与 areInlineCompatible 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`InstructionCost::getInvalid`。

### Lines 290-323: Defines getCacheLineSize
```cpp
  /// Data cache line size for LoopDataPrefetch pass. Has no use before GFX12.
  unsigned getCacheLineSize() const override { return 128; }

  /// How much before a load we should place the prefetch instruction.
  /// This is currently measured in number of IR instructions.
  unsigned getPrefetchDistance() const override;

  /// \return if target want to issue a prefetch in address space \p AS.
  bool shouldPrefetchAddressSpace(unsigned AS) const override;
  void collectKernelLaunchBounds(
      const Function &F,
      SmallVectorImpl<std::pair<StringRef, int64_t>> &LB) const override;

  enum class KnownIEEEMode { Unknown, On, Off };

  /// Return KnownIEEEMode::On if we know if the use context can assume
  /// "amdgpu-ieee"="true" and KnownIEEEMode::Off if we can assume
  /// "amdgpu-ieee"="false".
  KnownIEEEMode fpenvIEEEMode(const Instruction &I) const;

  /// Account for loads of i8 vector types to have reduced cost. For
  /// example the cost of load 4 i8s values is one is the cost of loading
  /// a single i32 value.
  InstructionCost getMemoryOpCost(
      unsigned Opcode, Type *Src, Align Alignment, unsigned AddressSpace,
      TTI::TargetCostKind CostKind,
      TTI::OperandValueInfo OpInfo = {TTI::OK_AnyValue, TTI::OP_None},
      const Instruction *I = nullptr) const override;

  /// When counting parts on AMD GPUs, account for i8s being grouped
  /// together under a single i32 value. Otherwise fall back to base
  /// implementation.
  unsigned getNumberOfParts(Type *Tp) const override;

```
**EN:** This section contains concrete logic for getCacheLineSize. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `KnownIEEEMode`.
**CN:** 本节包含与 getCacheLineSize 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`KnownIEEEMode`。

### Lines 324-342: Preprocessor guards and macros
```cpp
  ValueUniformity getValueUniformity(const Value *V) const override;

  InstructionCost getScalingFactorCost(Type *Ty, GlobalValue *BaseGV,
                                       StackOffset BaseOffset, bool HasBaseReg,
                                       int64_t Scale,
                                       unsigned AddrSpace) const override;

  bool isLSRCostLess(const TTI::LSRCost &A,
                     const TTI::LSRCost &B) const override;
  bool isNumRegsMajorCostOfLSR() const override;
  bool shouldDropLSRSolutionIfLessProfitable() const override;

  bool isUniform(const Instruction *I,
                 const SmallBitVector &UniformArgs) const override;
};

} // end namespace llvm

#endif // LLVM_LIB_TARGET_AMDGPU_AMDGPUTARGETTRANSFORMINFO_H
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `AMDGPUTargetMachine`, `GCNSubtarget`, `InstCombiner`, `Loop`, `ScalarEvolution`, `SITargetLowering`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; lowering / 降低
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPU.h"`
- `"llvm/CodeGen/BasicTTIImpl.h"`
- `"llvm/Support/AMDGPUAddrSpace.h"`
- `<optional>`
