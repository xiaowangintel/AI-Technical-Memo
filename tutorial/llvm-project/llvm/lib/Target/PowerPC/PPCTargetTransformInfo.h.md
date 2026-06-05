# PPCTargetTransformInfo.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCTargetTransformInfo.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file a TargetTransformInfoImplBase conforming object specific to the PPC target machine. It uses the target's detailed information to provide more precise answers to certain TTI queries, while letting the target independent and default TTI implementations handle the rest.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCTargetTransformInfo.h`，主要负责 PowerPC 后端的目标变换代价建模。 文件内容以接口、类型声明和协作关系说明为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- PPCTargetTransformInfo.h - PPC specific TTI -------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-13

```cpp
//===----------------------------------------------------------------------===//
/// \file
/// This file a TargetTransformInfoImplBase conforming object specific to the
/// PPC target machine. It uses the target's detailed information to
/// provide more precise answers to certain TTI queries, while letting the
/// target independent and default TTI implementations handle the rest.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 14-20

```cpp
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_POWERPC_PPCTARGETTRANSFORMINFO_H
#define LLVM_LIB_TARGET_POWERPC_PPCTARGETTRANSFORMINFO_H

#include "PPCTargetMachine.h"
#include "llvm/Analysis/TargetTransformInfo.h"
```
- **EN**: Pulls in direct dependencies required by this target transform cost modeling, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该目标变换代价建模所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 21-46

```cpp
#include "llvm/CodeGen/BasicTTIImpl.h"
#include "llvm/CodeGen/TargetLowering.h"
#include <optional>

namespace llvm {

class PPCTTIImpl final : public BasicTTIImplBase<PPCTTIImpl> {
  typedef BasicTTIImplBase<PPCTTIImpl> BaseT;
  typedef TargetTransformInfo TTI;
  friend BaseT;

  const PPCSubtarget *ST;
  const PPCTargetLowering *TLI;

  const PPCSubtarget *getST() const { return ST; }
  const PPCTargetLowering *getTLI() const { return TLI; }

public:
  explicit PPCTTIImpl(const PPCTargetMachine *TM, const Function &F)
      : BaseT(TM, F.getDataLayout()), ST(TM->getSubtargetImpl(F)),
        TLI(ST->getTargetLowering()) {}

  std::optional<Instruction *>
  instCombineIntrinsic(InstCombiner &IC, IntrinsicInst &II) const override;

  /// \name Scalar TTI Implementations
```
- **EN**: Pulls in direct dependencies required by this target transform cost modeling, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该目标变换代价建模所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 47-72

```cpp
  /// @{

  using BaseT::getIntImmCost;
  InstructionCost getIntImmCost(const APInt &Imm, Type *Ty,
                                TTI::TargetCostKind CostKind) const override;

  InstructionCost getIntImmCostInst(unsigned Opcode, unsigned Idx,
                                    const APInt &Imm, Type *Ty,
                                    TTI::TargetCostKind CostKind,
                                    Instruction *Inst = nullptr) const override;
  InstructionCost
  getIntImmCostIntrin(Intrinsic::ID IID, unsigned Idx, const APInt &Imm,
                      Type *Ty, TTI::TargetCostKind CostKind) const override;

  InstructionCost
  getInstructionCost(const User *U, ArrayRef<const Value *> Operands,
                     TTI::TargetCostKind CostKind) const override;

  TTI::PopcntSupportKind getPopcntSupport(unsigned TyWidth) const override;
  bool isHardwareLoopProfitable(Loop *L, ScalarEvolution &SE,
                                AssumptionCache &AC, TargetLibraryInfo *LibInfo,
                                HardwareLoopInfo &HWLoopInfo) const override;
  bool canSaveCmp(Loop *L, CondBrInst **BI, ScalarEvolution *SE, LoopInfo *LI,
                  DominatorTree *DT, AssumptionCache *AC,
                  TargetLibraryInfo *LibInfo) const override;
  bool getTgtMemIntrinsic(IntrinsicInst *Inst,
```
- **EN**: Implements helper routine(s) `getIntImmCost`, `getIntImmCostInst`, `getIntImmCostIntrin` for this portion of the PowerPC backend target transform cost modeling.
- **CN**: 这里实现了 PowerPC 后端该部分目标变换代价建模所需的辅助例程 `getIntImmCost`, `getIntImmCostInst`, `getIntImmCostIntrin`。

### Lines 73-92

```cpp
                          MemIntrinsicInfo &Info) const override;
  void getUnrollingPreferences(Loop *L, ScalarEvolution &SE,
                               TTI::UnrollingPreferences &UP,
                               OptimizationRemarkEmitter *ORE) const override;
  void getPeelingPreferences(Loop *L, ScalarEvolution &SE,
                             TTI::PeelingPreferences &PP) const override;
  bool isLSRCostLess(const TargetTransformInfo::LSRCost &C1,
                     const TargetTransformInfo::LSRCost &C2) const override;
  bool isNumRegsMajorCostOfLSR() const override;
  bool shouldBuildRelLookupTables() const override;
  /// @}

  /// \name Vector TTI Implementations
  /// @{
  bool useColdCCForColdCall(Function &F) const override;
  bool enableAggressiveInterleaving(bool LoopHasReductions) const override;
  TTI::MemCmpExpansionOptions
  enableMemCmpExpansion(bool OptSize, bool IsZeroCmp) const override;
  bool enableInterleavedAccessVectorization() const override;
```
- **EN**: Implements helper routine(s) `getUnrollingPreferences`, `getPeelingPreferences`, `isLSRCostLess` for this portion of the PowerPC backend target transform cost modeling.
- **CN**: 这里实现了 PowerPC 后端该部分目标变换代价建模所需的辅助例程 `getUnrollingPreferences`, `getPeelingPreferences`, `isLSRCostLess`。

### Lines 93-118

```cpp
  enum PPCRegisterClass {
    GPRRC, FPRRC, VRRC, VSXRC
  };
  unsigned getNumberOfRegisters(unsigned ClassID) const override;
  unsigned getRegisterClassForType(bool Vector,
                                   Type *Ty = nullptr) const override;
  const char *getRegisterClassName(unsigned ClassID) const override;
  TypeSize
  getRegisterBitWidth(TargetTransformInfo::RegisterKind K) const override;
  unsigned getCacheLineSize() const override;
  unsigned getPrefetchDistance() const override;
  unsigned getMaxInterleaveFactor(ElementCount VF) const override;
  InstructionCost vectorCostAdjustmentFactor(unsigned Opcode, Type *Ty1,
                                             Type *Ty2) const;
  InstructionCost getArithmeticInstrCost(
      unsigned Opcode, Type *Ty, TTI::TargetCostKind CostKind,
      TTI::OperandValueInfo Op1Info = {TTI::OK_AnyValue, TTI::OP_None},
      TTI::OperandValueInfo Op2Info = {TTI::OK_AnyValue, TTI::OP_None},
      ArrayRef<const Value *> Args = {},
      const Instruction *CxtI = nullptr) const override;
  InstructionCost
  getShuffleCost(TTI::ShuffleKind Kind, VectorType *DstTy, VectorType *SrcTy,
                 ArrayRef<int> Mask, TTI::TargetCostKind CostKind, int Index,
                 VectorType *SubTp, ArrayRef<const Value *> Args = {},
                 const Instruction *CxtI = nullptr) const override;
  InstructionCost
```
- **EN**: Implements helper routine(s) `getNumberOfRegisters`, `getRegisterClassForType`, `getRegisterClassName` for this portion of the PowerPC backend target transform cost modeling. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 PowerPC 后端该部分目标变换代价建模所需的辅助例程 `getNumberOfRegisters`, `getRegisterClassForType`, `getRegisterClassName`。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 119-144

```cpp
  getCastInstrCost(unsigned Opcode, Type *Dst, Type *Src,
                   TTI::CastContextHint CCH, TTI::TargetCostKind CostKind,
                   const Instruction *I = nullptr) const override;
  InstructionCost getCFInstrCost(unsigned Opcode, TTI::TargetCostKind CostKind,
                                 const Instruction *I = nullptr) const override;
  InstructionCost getCmpSelInstrCost(
      unsigned Opcode, Type *ValTy, Type *CondTy, CmpInst::Predicate VecPred,
      TTI::TargetCostKind CostKind,
      TTI::OperandValueInfo Op1Info = {TTI::OK_AnyValue, TTI::OP_None},
      TTI::OperandValueInfo Op2Info = {TTI::OK_AnyValue, TTI::OP_None},
      const Instruction *I = nullptr) const override;
  using BaseT::getVectorInstrCost;
  InstructionCost
  getVectorInstrCost(unsigned Opcode, Type *Val, TTI::TargetCostKind CostKind,
                     unsigned Index, const Value *Op0, const Value *Op1,
                     TTI::VectorInstrContext VIC =
                         TTI::VectorInstrContext::None) const override;
  InstructionCost getMemoryOpCost(
      unsigned Opcode, Type *Src, Align Alignment, unsigned AddressSpace,
      TTI::TargetCostKind CostKind,
      TTI::OperandValueInfo OpInfo = {TTI::OK_AnyValue, TTI::OP_None},
      const Instruction *I = nullptr) const override;
  InstructionCost getInterleavedMemoryOpCost(
      unsigned Opcode, Type *VecTy, unsigned Factor, ArrayRef<unsigned> Indices,
      Align Alignment, unsigned AddressSpace, TTI::TargetCostKind CostKind,
      bool UseMaskForCond = false, bool UseMaskForGaps = false) const override;
```
- **EN**: Implements helper routine(s) `getCastInstrCost`, `getCFInstrCost`, `getCmpSelInstrCost` for this portion of the PowerPC backend target transform cost modeling. These declarations feed generated pattern-matching logic.
- **CN**: 这里实现了 PowerPC 后端该部分目标变换代价建模所需的辅助例程 `getCastInstrCost`, `getCFInstrCost`, `getCmpSelInstrCost`。 这些声明会进入生成式模式匹配逻辑。

### Lines 145-170

```cpp
  InstructionCost
  getIntrinsicInstrCost(const IntrinsicCostAttributes &ICA,
                        TTI::TargetCostKind CostKind) const override;
  bool areInlineCompatible(const Function *Caller,
                           const Function *Callee) const override;
  bool areTypesABICompatible(const Function *Caller, const Function *Callee,
                             ArrayRef<Type *> Types) const override;
  bool supportsTailCallFor(const CallBase *CB) const override;

  TargetTransformInfo::VPLegalization
  getVPLegalizationStrategy(const VPIntrinsic &PI) const override;

  bool hasActiveVectorLength() const override;

  bool
  isLegalMaskedStore(Type *DataType, Align Alignment, unsigned AddressSpace,
                     TTI::MaskKind MaskKind =
                         TTI::MaskKind::VariableOrConstantMask) const override;
  bool
  isLegalMaskedLoad(Type *DataType, Align Alignment, unsigned AddressSpace,
                    TTI::MaskKind MaskKind =
                        TTI::MaskKind::VariableOrConstantMask) const override;

  InstructionCost
  getMemIntrinsicInstrCost(const MemIntrinsicCostAttributes &MICA,
                           TTI::TargetCostKind CostKind) const override;
```
- **EN**: Declares function entry points including `getIntrinsicInstrCost`, `areInlineCompatible`, `areTypesABICompatible` that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `getIntrinsicInstrCost`, `areInlineCompatible`, `areTypesABICompatible`。

### Lines 171-189

```cpp

  InstructionCost getPartialReductionCost(
      unsigned Opcode, Type *InputTypeA, Type *InputTypeB, Type *AccumType,
      ElementCount VF, TTI::PartialReductionExtendKind OpAExtend,
      TTI::PartialReductionExtendKind OpBExtend, std::optional<unsigned> BinOp,
      TTI::TargetCostKind CostKind,
      std::optional<FastMathFlags> FMF) const override {
    return InstructionCost::getInvalid();
  }

private:
  // The following constant is used for estimating costs on power9.
  static const InstructionCost::CostType P9PipelineFlushEstimate = 80;

  /// @}
};

} // end namespace llvm
```
- **EN**: Implements helper routine(s) `getPartialReductionCost`, `getInvalid` for this portion of the PowerPC backend target transform cost modeling.
- **CN**: 这里实现了 PowerPC 后端该部分目标变换代价建模所需的辅助例程 `getPartialReductionCost`, `getInvalid`。

### Lines 190-190

```cpp
#endif
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- Target transform cost modeling / 目标变换代价建模
- Register modeling / 寄存器建模
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Target machine configuration / 目标机器配置
- Declarative TableGen records / 声明式 TableGen 记录
- Vector or SIMD behavior / 向量或 SIMD 行为

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `PPCTargetMachine.h`
- `llvm/Analysis/TargetTransformInfo.h`
- `llvm/CodeGen/BasicTTIImpl.h`
- `llvm/CodeGen/TargetLowering.h`
- `optional`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
