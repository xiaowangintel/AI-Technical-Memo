# AMDGPUISelLowering.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUISelLowering.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for AMDGPUISelLowering in the LLVM AMDGPU backend. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM AMDGPU 后端中 AMDGPUISelLowering 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: File banner, includes, and setup
```cpp
//===-- AMDGPUISelLowering.h - AMDGPU Lowering Interface --------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// Interface definition of the TargetLowering class that is common
/// to all AMD GPUs.
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPU_AMDGPUISELLOWERING_H
#define LLVM_LIB_TARGET_AMDGPU_AMDGPUISELLOWERING_H

#include "llvm/CodeGen/CallingConvLower.h"
#include "llvm/CodeGen/TargetLowering.h"

namespace llvm {

class AMDGPUMachineFunctionInfo;
class AMDGPUSubtarget;
struct ArgDescriptor;

class AMDGPUTargetLowering : public TargetLowering {
private:
  const AMDGPUSubtarget *Subtarget;

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `AMDGPUMachineFunctionInfo`, `AMDGPUSubtarget`, `ArgDescriptor`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`AMDGPUMachineFunctionInfo`, `AMDGPUSubtarget`, `ArgDescriptor`。

### Lines 31-63: Declares getFFBX_U32
```cpp
  /// \returns AMDGPUISD::FFBH_U32 node if the incoming \p Op may have been
  /// legalized from a smaller type VT. Need to match pre-legalized type because
  /// the generic legalization inserts the add/sub between the select and
  /// compare.
  SDValue getFFBX_U32(SelectionDAG &DAG, SDValue Op, const SDLoc &DL, unsigned Opc) const;

public:
  /// \returns The minimum number of bits needed to store the value of \Op as an
  /// unsigned integer. Truncating to this size and then zero-extending to the
  /// original size will not change the value.
  static unsigned numBitsUnsigned(SDValue Op, SelectionDAG &DAG);

  /// \returns The minimum number of bits needed to store the value of \Op as a
  /// signed integer. Truncating to this size and then sign-extending to the
  /// original size will not change the value.
  static unsigned numBitsSigned(SDValue Op, SelectionDAG &DAG);

protected:
  SDValue LowerEXTRACT_SUBVECTOR(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerCONCAT_VECTORS(SDValue Op, SelectionDAG &DAG) const;
  /// Split a vector store into multiple scalar stores.
  /// \returns The resulting chain.

  SDValue LowerCTLS(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerFCEIL(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerFTRUNC(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerFRINT(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerFNEARBYINT(SDValue Op, SelectionDAG &DAG) const;

  SDValue LowerFROUNDEVEN(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerFROUND(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerFFLOOR(SDValue Op, SelectionDAG &DAG) const;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 64-94: Declares allowApproxFunc
```cpp
  static bool allowApproxFunc(const SelectionDAG &DAG, SDNodeFlags Flags);
  static bool needsDenormHandlingF32(const SelectionDAG &DAG, SDValue Src,
                                     SDNodeFlags Flags);
  SDValue getIsLtSmallestNormal(SelectionDAG &DAG, SDValue Op,
                                SDNodeFlags Flags) const;
  SDValue getIsFinite(SelectionDAG &DAG, SDValue Op, SDNodeFlags Flags) const;
  std::pair<SDValue, SDValue> getScaledLogInput(SelectionDAG &DAG,
                                                const SDLoc SL, SDValue Op,
                                                SDNodeFlags Flags) const;

  SDValue LowerFLOG2(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerFLOGCommon(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerFLOG10(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerFLOGUnsafe(SDValue Op, const SDLoc &SL, SelectionDAG &DAG,
                          bool IsLog10, SDNodeFlags Flags) const;
  SDValue lowerFEXP2(SDValue Op, SelectionDAG &DAG) const;

  SDValue lowerFEXPUnsafeImpl(SDValue Op, const SDLoc &SL, SelectionDAG &DAG,
                              SDNodeFlags Flags, bool IsExp10) const;

  SDValue lowerFEXPUnsafe(SDValue Op, const SDLoc &SL, SelectionDAG &DAG,
                          SDNodeFlags Flags) const;
  SDValue lowerFEXP10Unsafe(SDValue Op, const SDLoc &SL, SelectionDAG &DAG,
                            SDNodeFlags Flags) const;
  SDValue lowerFEXP(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerFEXPF64(SDValue Op, SelectionDAG &DAG) const;

  SDValue lowerCTLZResults(SDValue Op, SelectionDAG &DAG) const;

  SDValue LowerCTLZ_CTTZ(SDValue Op, SelectionDAG &DAG) const;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 95-119: Declares LowerINT_TO_FP32
```cpp
  SDValue LowerINT_TO_FP32(SDValue Op, SelectionDAG &DAG, bool Signed) const;
  SDValue LowerINT_TO_FP64(SDValue Op, SelectionDAG &DAG, bool Signed) const;
  SDValue LowerUINT_TO_FP(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerSINT_TO_FP(SDValue Op, SelectionDAG &DAG) const;

  SDValue LowerFP_TO_INT64(SDValue Op, SelectionDAG &DAG, bool Signed) const;
  SDValue LowerFP_TO_FP16(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerFP_TO_INT(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerFP_TO_INT_SAT(SDValue Op, SelectionDAG &DAG) const;

  SDValue LowerF64ToF16Safe(SDValue Src, const SDLoc &DL,
                            SelectionDAG &DAG) const;

  SDValue LowerSIGN_EXTEND_INREG(SDValue Op, SelectionDAG &DAG) const;

protected:
  /// Check whether value Val can be supported by v_mov_b64, for the current
  /// target.
  bool isInt64ImmLegal(SDNode *Val, SelectionDAG &DAG) const;
  bool shouldCombineMemoryType(EVT VT) const;
  SDValue performLoadCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performStoreCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performAssertSZExtCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performIntrinsicWOChainCombine(SDNode *N, DAGCombinerInfo &DCI) const;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 120-151: Defines splitBinaryBitConstantOpImpl
```cpp
  SDValue splitBinaryBitConstantOpImpl(DAGCombinerInfo &DCI, const SDLoc &SL,
                                       unsigned Opc, SDValue LHS,
                                       uint32_t ValLo, uint32_t ValHi) const;
  SDValue performShlCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performSraCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performSrlCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performTruncateCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performMulCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performMulLoHiCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performMulhsCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performMulhuCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performCtlz_CttzCombine(const SDLoc &SL, SDValue Cond, SDValue LHS,
                             SDValue RHS, DAGCombinerInfo &DCI) const;

  SDValue foldFreeOpFromSelect(TargetLowering::DAGCombinerInfo &DCI,
                               SDValue N) const;
  SDValue performSelectCombine(SDNode *N, DAGCombinerInfo &DCI) const;

  TargetLowering::NegatibleCost
  getConstantNegateCost(const ConstantFPSDNode *C) const;

  bool isConstantCostlierToNegate(SDValue N) const;
  bool isConstantCheaperToNegate(SDValue N) const;
  SDValue performFNegCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performFAbsCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performRcpCombine(SDNode *N, DAGCombinerInfo &DCI) const;

  static EVT getEquivalentMemType(LLVMContext &Context, EVT VT);

  virtual SDValue LowerGlobalAddress(AMDGPUMachineFunctionInfo *MFI, SDValue Op,
                                     SelectionDAG &DAG) const;

```
**EN:** This section contains concrete logic for splitBinaryBitConstantOpImpl. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 splitBinaryBitConstantOpImpl 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 152-185: Defines split64BitValue
```cpp
  /// Return 64-bit value Op as two 32-bit integers.
  std::pair<SDValue, SDValue> split64BitValue(SDValue Op,
                                              SelectionDAG &DAG) const;
  SDValue getLoHalf64(SDValue Op, SelectionDAG &DAG) const;
  SDValue getHiHalf64(SDValue Op, SelectionDAG &DAG) const;

  /// Split a vector type into two parts. The first part is a power of two
  /// vector. The second part is whatever is left over, and is a scalar if it
  /// would otherwise be a 1-vector.
  std::pair<EVT, EVT> getSplitDestVTs(const EVT &VT, SelectionDAG &DAG) const;

  /// Split a vector value into two parts of types LoVT and HiVT. HiVT could be
  /// scalar.
  std::pair<SDValue, SDValue> splitVector(const SDValue &N, const SDLoc &DL,
                                          const EVT &LoVT, const EVT &HighVT,
                                          SelectionDAG &DAG) const;

  /// Split a vector load into 2 loads of half the vector.
  SDValue SplitVectorLoad(SDValue Op, SelectionDAG &DAG) const;

  /// Widen a suitably aligned v3 load. For all other cases, split the input
  /// vector load.
  SDValue WidenOrSplitVectorLoad(SDValue Op, SelectionDAG &DAG) const;

  /// Split a vector store into 2 stores of half the vector.
  SDValue SplitVectorStore(SDValue Op, SelectionDAG &DAG) const;

  SDValue LowerSTORE(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerSDIVREM(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerUDIVREM(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerDIVREM24(SDValue Op, SelectionDAG &DAG, bool sign) const;
  void LowerUDIVREM64(SDValue Op, SelectionDAG &DAG,
                                    SmallVectorImpl<SDValue> &Results) const;

```
**EN:** This section contains concrete logic for split64BitValue. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 split64BitValue 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 186-217: Defines analyzeFormalArgumentsCompute
```cpp
  void analyzeFormalArgumentsCompute(
    CCState &State,
    const SmallVectorImpl<ISD::InputArg> &Ins) const;

public:
  AMDGPUTargetLowering(const TargetMachine &TM, const TargetSubtargetInfo &STI,
                       const AMDGPUSubtarget &AMDGPUSTI);

  bool mayIgnoreSignedZero(SDValue Op) const;

  static inline SDValue stripBitcast(SDValue Val) {
    return Val.getOpcode() == ISD::BITCAST ? Val.getOperand(0) : Val;
  }

  static bool shouldFoldFNegIntoSrc(SDNode *FNeg, SDValue FNegSrc);
  static bool allUsesHaveSourceMods(const SDNode *N,
                                    unsigned CostThreshold = 4);
  bool isFAbsFree(EVT VT) const override;
  bool isFNegFree(EVT VT) const override;
  bool isTruncateFree(EVT Src, EVT Dest) const override;
  bool isTruncateFree(Type *Src, Type *Dest) const override;

  bool isZExtFree(Type *Src, Type *Dest) const override;
  bool isZExtFree(EVT Src, EVT Dest) const override;

  SDValue getNegatedExpression(SDValue Op, SelectionDAG &DAG,
                               bool LegalOperations, bool ForCodeSize,
                               NegatibleCost &Cost,
                               unsigned Depth) const override;

  bool isNarrowingProfitable(SDNode *N, EVT SrcVT, EVT DestVT) const override;

```
**EN:** This section contains concrete logic for analyzeFormalArgumentsCompute. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 analyzeFormalArgumentsCompute 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 218-249: Defines isDesirableToCommuteWithShift
```cpp
  bool isDesirableToCommuteWithShift(const SDNode *N,
                                     CombineLevel Level) const override;

  EVT getTypeForExtReturn(LLVMContext &Context, EVT VT,
                          ISD::NodeType ExtendKind) const override;

  unsigned getVectorIdxWidth(const DataLayout &) const override;
  bool isSelectSupported(SelectSupportKind) const override;

  bool isFPImmLegal(const APFloat &Imm, EVT VT,
                    bool ForCodeSize) const override;
  bool ShouldShrinkFPConstant(EVT VT) const override;
  bool shouldReduceLoadWidth(SDNode *Load, ISD::LoadExtType ExtType, EVT ExtVT,
                             std::optional<unsigned> ByteOffset) const override;

  bool isLoadBitCastBeneficial(EVT, EVT, const SelectionDAG &DAG,
                               const MachineMemOperand &MMO) const final;

  bool storeOfVectorConstantIsCheap(bool IsZero, EVT MemVT,
                                    unsigned NumElem,
                                    unsigned AS) const override;
  bool aggressivelyPreferBuildVectorSources(EVT VecVT) const override;
  bool isCheapToSpeculateCttz(Type *Ty) const override;
  bool isCheapToSpeculateCtlz(Type *Ty) const override;

  bool isSDNodeAlwaysUniform(const SDNode *N) const override;

  // FIXME: This hook should not exist
  AtomicExpansionKind shouldCastAtomicLoadInIR(LoadInst *LI) const override {
    return AtomicExpansionKind::None;
  }

```
**EN:** This section contains concrete logic for isDesirableToCommuteWithShift. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 isDesirableToCommuteWithShift 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 250-280: Defines shouldCastAtomicStoreInIR
```cpp
  AtomicExpansionKind shouldCastAtomicStoreInIR(StoreInst *SI) const override {
    return AtomicExpansionKind::None;
  }

  AtomicExpansionKind shouldCastAtomicRMWIInIR(AtomicRMWInst *) const override {
    return AtomicExpansionKind::None;
  }

  bool shouldIssueAtomicLoadForAtomicEmulationLoop() const override {
    return false;
  }

  static CCAssignFn *CCAssignFnForCall(CallingConv::ID CC, bool IsVarArg);
  static CCAssignFn *CCAssignFnForReturn(CallingConv::ID CC, bool IsVarArg);

  SDValue LowerReturn(SDValue Chain, CallingConv::ID CallConv, bool isVarArg,
                      const SmallVectorImpl<ISD::OutputArg> &Outs,
                      const SmallVectorImpl<SDValue> &OutVals, const SDLoc &DL,
                      SelectionDAG &DAG) const override;

  SDValue addTokenForArgument(SDValue Chain,
                              SelectionDAG &DAG,
                              MachineFrameInfo &MFI,
                              int ClobberedFI) const;

  SDValue lowerUnhandledCall(CallLoweringInfo &CLI,
                             SmallVectorImpl<SDValue> &InVals,
                             StringRef Reason) const;
  SDValue LowerCall(CallLoweringInfo &CLI,
                    SmallVectorImpl<SDValue> &InVals) const override;

```
**EN:** This section contains concrete logic for shouldCastAtomicStoreInIR. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 shouldCastAtomicStoreInIR 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 281-313: Declares LowerDYNAMIC_STACKALLOC
```cpp
  SDValue LowerDYNAMIC_STACKALLOC(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerOperation(SDValue Op, SelectionDAG &DAG) const override;
  SDValue PerformDAGCombine(SDNode *N, DAGCombinerInfo &DCI) const override;

  bool SimplifyDemandedBitsForTargetNode(SDValue Op,
                                         const APInt &OriginalDemandedBits,
                                         const APInt &OriginalDemandedElts,
                                         KnownBits &Known,
                                         TargetLoweringOpt &TLO,
                                         unsigned Depth) const override;

  void ReplaceNodeResults(SDNode * N,
                          SmallVectorImpl<SDValue> &Results,
                          SelectionDAG &DAG) const override;

  SDValue combineFMinMaxLegacyImpl(const SDLoc &DL, EVT VT, SDValue LHS,
                                   SDValue RHS, SDValue True, SDValue False,
                                   SDValue CC, DAGCombinerInfo &DCI) const;

  SDValue combineFMinMaxLegacy(const SDLoc &DL, EVT VT, SDValue LHS,
                               SDValue RHS, SDValue True, SDValue False,
                               SDValue CC, DAGCombinerInfo &DCI) const;

  // FIXME: Turn off MergeConsecutiveStores() before Instruction Selection for
  // AMDGPU.  Commit r319036,
  // (https://github.com/llvm/llvm-project/commit/db77e57ea86d941a4262ef60261692f4cb6893e6)
  // turned on MergeConsecutiveStores() before Instruction Selection for all
  // targets.  Enough AMDGPU compiles go into an infinite loop (
  // MergeConsecutiveStores() merges two stores; LegalizeStoreOps() un-merges;
  // MergeConsecutiveStores() re-merges, etc. ) to warrant turning it off for
  // now.
  bool mergeStoresAfterLegalization(EVT) const override { return false; }

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 314-344: Defines isFsqrtCheap
```cpp
  bool isFsqrtCheap(SDValue Operand, SelectionDAG &DAG) const override {
    return true;
  }
  SDValue getSqrtEstimate(SDValue Operand, SelectionDAG &DAG, int Enabled,
                           int &RefinementSteps, bool &UseOneConstNR,
                           bool Reciprocal) const override;
  SDValue getRecipEstimate(SDValue Operand, SelectionDAG &DAG, int Enabled,
                           int &RefinementSteps) const override;

  virtual SDNode *PostISelFolding(MachineSDNode *N,
                                  SelectionDAG &DAG) const = 0;

  /// Determine which of the bits specified in \p Mask are known to be
  /// either zero or one and return them in the \p KnownZero and \p KnownOne
  /// bitsets.
  void computeKnownBitsForTargetNode(const SDValue Op,
                                     KnownBits &Known,
                                     const APInt &DemandedElts,
                                     const SelectionDAG &DAG,
                                     unsigned Depth = 0) const override;

  unsigned ComputeNumSignBitsForTargetNode(SDValue Op, const APInt &DemandedElts,
                                           const SelectionDAG &DAG,
                                           unsigned Depth = 0) const override;

  unsigned computeNumSignBitsForTargetInstr(GISelValueTracking &Analysis,
                                            Register R,
                                            const APInt &DemandedElts,
                                            const MachineRegisterInfo &MRI,
                                            unsigned Depth = 0) const override;

```
**EN:** This section contains concrete logic for isFsqrtCheap. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 isFsqrtCheap 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 345-378: Defines canCreateUndefOrPoisonForTargetNode
```cpp
  bool canCreateUndefOrPoisonForTargetNode(
      SDValue Op, const APInt &DemandedElts, const SelectionDAG &DAG,
      UndefPoisonKind Kind, bool ConsiderFlags, unsigned Depth) const override;

  bool isKnownNeverNaNForTargetNode(SDValue Op, const APInt &DemandedElts,
                                    const SelectionDAG &DAG, bool SNaN = false,
                                    unsigned Depth = 0) const override;

  bool isReassocProfitable(MachineRegisterInfo &MRI, Register N0,
                           Register N1) const override;

  /// Helper function that adds Reg to the LiveIn list of the DAG's
  /// MachineFunction.
  ///
  /// \returns a RegisterSDNode representing Reg if \p RawReg is true, otherwise
  /// a copy from the register.
  SDValue CreateLiveInRegister(SelectionDAG &DAG,
                               const TargetRegisterClass *RC,
                               Register Reg, EVT VT,
                               const SDLoc &SL,
                               bool RawReg = false) const;
  SDValue CreateLiveInRegister(SelectionDAG &DAG,
                               const TargetRegisterClass *RC,
                               Register Reg, EVT VT) const {
    return CreateLiveInRegister(DAG, RC, Reg, VT, SDLoc(DAG.getEntryNode()));
  }

  // Returns the raw live in register rather than a copy from it.
  SDValue CreateLiveInRegisterRaw(SelectionDAG &DAG,
                                  const TargetRegisterClass *RC,
                                  Register Reg, EVT VT) const {
    return CreateLiveInRegister(DAG, RC, Reg, VT, SDLoc(DAG.getEntryNode()), true);
  }

```
**EN:** This section contains concrete logic for canCreateUndefOrPoisonForTargetNode. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 canCreateUndefOrPoisonForTargetNode 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 379-410: Defines loadStackInputValue
```cpp
  /// Similar to CreateLiveInRegister, except value maybe loaded from a stack
  /// slot rather than passed in a register.
  SDValue loadStackInputValue(SelectionDAG &DAG,
                              EVT VT,
                              const SDLoc &SL,
                              int64_t Offset) const;

  SDValue storeStackInputValue(SelectionDAG &DAG,
                               const SDLoc &SL,
                               SDValue Chain,
                               SDValue ArgVal,
                               int64_t Offset) const;

  SDValue loadInputValue(SelectionDAG &DAG,
                         const TargetRegisterClass *RC,
                         EVT VT, const SDLoc &SL,
                         const ArgDescriptor &Arg) const;

  enum ImplicitParameter {
    FIRST_IMPLICIT,
    PRIVATE_BASE,
    SHARED_BASE,
    QUEUE_PTR,
  };

  /// Helper function that returns the byte offset of the given
  /// type of implicit parameter.
  uint32_t getImplicitParameterOffset(const MachineFunction &MF,
                                      const ImplicitParameter Param) const;
  uint32_t getImplicitParameterOffset(const uint64_t ExplicitKernArgSize,
                                      const ImplicitParameter Param) const;

```
**EN:** This section contains concrete logic for loadStackInputValue. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `ImplicitParameter`.
**CN:** 本节包含与 loadStackInputValue 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`ImplicitParameter`。

### Lines 411-428: Preprocessor guards and macros
```cpp
  MVT getFenceOperandTy(const DataLayout &DL) const override {
    return MVT::i32;
  }

  bool hasMultipleConditionRegisters(EVT VT) const override {
    // FIXME: This is only partially true. If we have to do vector compares, any
    // SGPR pair can be a condition register. If we have a uniform condition, we
    // are better off doing SALU operations, where there is only one SCC. For
    // now, we don't have a way of knowing during instruction selection if a
    // condition will be uniform and we always use vector compares. Assume we
    // are using vector compares until that is fixed.
    return true;
  }
};

} // End namespace llvm

#endif
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `AMDGPUMachineFunctionInfo`, `AMDGPUSubtarget`, `ArgDescriptor`, `AMDGPUTargetLowering`, `ImplicitParameter`
- **Main themes / 核心主题**: subtarget modeling / 子目标建模; lowering / 降低; legalization / 合法化; machine-function state / MachineFunction 状态
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"llvm/CodeGen/CallingConvLower.h"`
- `"llvm/CodeGen/TargetLowering.h"`
