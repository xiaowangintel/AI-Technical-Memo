# SIISelLowering.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/SIISelLowering.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for SIISelLowering in the LLVM AMDGPU backend. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM AMDGPU 后端中 SIISelLowering 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-31: File banner, includes, and setup
```cpp
//===-- SIISelLowering.h - SI DAG Lowering Interface ------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// SI DAG Lowering interface definition
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPU_SIISELLOWERING_H
#define LLVM_LIB_TARGET_AMDGPU_SIISELLOWERING_H

#include "AMDGPUArgumentUsageInfo.h"
#include "AMDGPUISelLowering.h"
#include "SIDefines.h"
#include "llvm/CodeGen/MachineFunction.h"

namespace llvm {

class GCNSubtarget;
class SIMachineFunctionInfo;
class SIRegisterInfo;

namespace AMDGPU {
struct ImageDimIntrinsicInfo;
}

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `GCNSubtarget`, `SIMachineFunctionInfo`, `SIRegisterInfo`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`GCNSubtarget`, `SIMachineFunctionInfo`, `SIRegisterInfo`。

### Lines 32-63: Declares class SITargetLowering
```cpp
class SITargetLowering final : public AMDGPUTargetLowering {
private:
  const GCNSubtarget *Subtarget;

public:
  MVT getRegisterTypeForCallingConv(LLVMContext &Context,
                                    CallingConv::ID CC,
                                    EVT VT) const override;
  unsigned getNumRegistersForCallingConv(LLVMContext &Context,
                                         CallingConv::ID CC,
                                         EVT VT) const override;

  unsigned getVectorTypeBreakdownForCallingConv(
    LLVMContext &Context, CallingConv::ID CC, EVT VT, EVT &IntermediateVT,
    unsigned &NumIntermediates, MVT &RegisterVT) const override;

  MachinePointerInfo getKernargSegmentPtrInfo(MachineFunction &MF) const;

private:
  SDValue lowerKernArgParameterPtr(SelectionDAG &DAG, const SDLoc &SL,
                                   SDValue Chain, uint64_t Offset) const;
  SDValue getImplicitArgPtr(SelectionDAG &DAG, const SDLoc &SL) const;
  SDValue getLDSKernelId(SelectionDAG &DAG, const SDLoc &SL) const;
  SDValue lowerKernargMemParameter(SelectionDAG &DAG, EVT VT, EVT MemVT,
                                   const SDLoc &SL, SDValue Chain,
                                   uint64_t Offset, Align Alignment,
                                   bool Signed,
                                   const ISD::InputArg *Arg = nullptr) const;
  SDValue loadImplicitKernelArgument(SelectionDAG &DAG, MVT VT, const SDLoc &DL,
                                     Align Alignment,
                                     ImplicitParameter Param) const;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `SITargetLowering`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`SITargetLowering`。

### Lines 64-95: Defines convertABITypeToValueType
```cpp
  SDValue convertABITypeToValueType(SelectionDAG &DAG, SDValue Val,
                                    CCValAssign &VA, const SDLoc &SL) const;

  SDValue lowerStackParameter(SelectionDAG &DAG, CCValAssign &VA,
                              const SDLoc &SL, SDValue Chain,
                              const ISD::InputArg &Arg) const;
  SDValue lowerWorkGroupId(
      SelectionDAG &DAG, const SIMachineFunctionInfo &MFI, EVT VT,
      AMDGPUFunctionArgInfo::PreloadedValue ClusterIdPV,
      AMDGPUFunctionArgInfo::PreloadedValue ClusterMaxIdPV,
      AMDGPUFunctionArgInfo::PreloadedValue ClusterWorkGroupIdPV) const;
  SDValue getPreloadedValue(SelectionDAG &DAG,
                            const SIMachineFunctionInfo &MFI,
                            EVT VT,
                            AMDGPUFunctionArgInfo::PreloadedValue) const;

  SDValue LowerGlobalAddress(AMDGPUMachineFunctionInfo *MFI, SDValue Op,
                             SelectionDAG &DAG) const override;
  SDValue LowerExternalSymbol(SDValue Op, SelectionDAG &DAG) const;

  SDValue lowerImplicitZextParam(SelectionDAG &DAG, SDValue Op,
                                 MVT VT, unsigned Offset) const;
  SDValue lowerImage(SDValue Op, const AMDGPU::ImageDimIntrinsicInfo *Intr,
                     SelectionDAG &DAG, bool WithChain) const;
  SDValue lowerSBuffer(EVT VT, SDLoc DL, SDValue Rsrc, SDValue Offset,
                       SDValue CachePolicy, SelectionDAG &DAG) const;

  SDValue lowerRawBufferAtomicIntrin(SDValue Op, SelectionDAG &DAG,
                                     unsigned NewOpcode) const;
  SDValue lowerStructBufferAtomicIntrin(SDValue Op, SelectionDAG &DAG,
                                        unsigned NewOpcode) const;

```
**EN:** This section contains concrete logic for convertABITypeToValueType. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 convertABITypeToValueType 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 96-115: Declares lowerWaveID
```cpp
  SDValue lowerWaveID(SelectionDAG &DAG, SDValue Op) const;
  SDValue lowerConstHwRegRead(SelectionDAG &DAG, SDValue Op,
                              AMDGPU::Hwreg::Id HwReg, unsigned LowBit,
                              unsigned Width) const;
  SDValue lowerWorkitemID(SelectionDAG &DAG, SDValue Op, unsigned Dim,
                          const ArgDescriptor &ArgDesc) const;

  SDValue LowerINTRINSIC_WO_CHAIN(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerINTRINSIC_W_CHAIN(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerINTRINSIC_VOID(SDValue Op, SelectionDAG &DAG) const;

  // The raw.tbuffer and struct.tbuffer intrinsics have two offset args: offset
  // (the offset that is included in bounds checking and swizzling, to be split
  // between the instruction's voffset and immoffset fields) and soffset (the
  // offset that is excluded from bounds checking and swizzling, to go in the
  // instruction's soffset field).  This function takes the first kind of
  // offset and figures out how to split it between voffset and immoffset.
  std::pair<SDValue, SDValue> splitBufferOffsets(SDValue Offset,
                                                 SelectionDAG &DAG) const;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 116-148: Declares widenLoad
```cpp
  SDValue widenLoad(LoadSDNode *Ld, DAGCombinerInfo &DCI) const;
  SDValue LowerLOAD(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerSELECT(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerFastUnsafeFDIV(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerFastUnsafeFDIV64(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerFDIV_FAST(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerFDIV16(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerFDIV32(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerFDIV64(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerFDIV(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerFFREXP(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerSTORE(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerTrig(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerFSQRTF16(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerFSQRTF32(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerFSQRTF64(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerATOMIC_CMP_SWAP(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerBRCOND(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerRETURNADDR(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerSPONENTRY(SDValue Op, SelectionDAG &DAG) const;
  SDValue adjustLoadValueType(unsigned Opcode, MemSDNode *M,
                              SelectionDAG &DAG, ArrayRef<SDValue> Ops,
                              bool IsIntrinsic = false) const;

  SDValue lowerIntrinsicLoad(MemSDNode *M, bool IsFormat, SelectionDAG &DAG,
                             ArrayRef<SDValue> Ops) const;

  // Call DAG.getMemIntrinsicNode for a load, but first widen a dwordx3 type to
  // dwordx4 if on SI.
  SDValue getMemIntrinsicNode(unsigned Opcode, const SDLoc &DL, SDVTList VTList,
                              ArrayRef<SDValue> Ops, EVT MemVT,
                              MachineMemOperand *MMO, SelectionDAG &DAG) const;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 149-178: Defines handleD16VData
```cpp
  SDValue handleD16VData(SDValue VData, SelectionDAG &DAG,
                         bool ImageStore = false) const;

  /// Converts \p Op, which must be of floating point type, to the
  /// floating point type \p VT, by either extending or truncating it.
  SDValue getFPExtOrFPRound(SelectionDAG &DAG,
                            SDValue Op,
                            const SDLoc &DL,
                            EVT VT) const;

  SDValue convertArgType(
    SelectionDAG &DAG, EVT VT, EVT MemVT, const SDLoc &SL, SDValue Val,
    bool Signed, const ISD::InputArg *Arg = nullptr) const;

  /// Custom lowering for ISD::FP_ROUND for MVT::f16.
  SDValue lowerFP_ROUND(SDValue Op, SelectionDAG &DAG) const;
  SDValue splitFP_ROUNDVectorOp(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerFMINNUM_FMAXNUM(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerFMINIMUMNUM_FMAXIMUMNUM(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerFMINIMUM_FMAXIMUM(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerFLDEXP(SDValue Op, SelectionDAG &DAG) const;
  SDValue promoteUniformOpToI32(SDValue Op, DAGCombinerInfo &DCI) const;
  SDValue lowerFCOPYSIGN(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerMUL(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerXMULO(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerXMUL_LOHI(SDValue Op, SelectionDAG &DAG) const;

  SDValue getSegmentAperture(unsigned AS, const SDLoc &DL,
                             SelectionDAG &DAG) const;

```
**EN:** This section contains concrete logic for handleD16VData. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 handleD16VData 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 179-210: Declares lowerADDRSPACECAST
```cpp
  SDValue lowerADDRSPACECAST(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerINSERT_SUBVECTOR(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerINSERT_VECTOR_ELT(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerEXTRACT_VECTOR_ELT(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerVECTOR_SHUFFLE(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerSCALAR_TO_VECTOR(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerBUILD_VECTOR(SDValue Op, SelectionDAG &DAG) const;

  SDValue lowerTRAP(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerTrapEndpgm(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerTrapHsaQueuePtr(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerTrapHsa(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerDEBUGTRAP(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerINLINEASM(SDValue Op, SelectionDAG &DAG) const;

  SDNode *adjustWritemask(MachineSDNode *&N, SelectionDAG &DAG) const;

  SDValue performUCharToFloatCombine(SDNode *N,
                                     DAGCombinerInfo &DCI) const;
  SDValue performFCopySignCombine(SDNode *N, DAGCombinerInfo &DCI) const;

  SDValue performSHLPtrCombine(SDNode *N,
                               unsigned AS,
                               EVT MemVT,
                               DAGCombinerInfo &DCI) const;

  SDValue performMemSDNodeCombine(MemSDNode *N, DAGCombinerInfo &DCI) const;

  SDValue splitBinaryBitConstantOp(DAGCombinerInfo &DCI, const SDLoc &SL,
                                   unsigned Opc, SDValue LHS,
                                   const ConstantSDNode *CRHS) const;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 211-241: Declares performAndCombine
```cpp
  SDValue performAndCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performOrCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performXorCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performZeroOrAnyExtendCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performSignExtendInRegCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performClassCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue getCanonicalConstantFP(SelectionDAG &DAG, const SDLoc &SL, EVT VT,
                                 const APFloat &C) const;
  SDValue performFCanonicalizeCombine(SDNode *N, DAGCombinerInfo &DCI) const;

  SDValue performFPMed3ImmCombine(SelectionDAG &DAG, const SDLoc &SL,
                                  SDValue Op0, SDValue Op1,
                                  bool IsKnownNoNaNs) const;
  SDValue performIntMed3ImmCombine(SelectionDAG &DAG, const SDLoc &SL,
                                   SDValue Src, SDValue MinVal, SDValue MaxVal,
                                   bool Signed) const;
  SDValue performMinMaxCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performFMed3Combine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performCvtPkRTZCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performExtractVectorEltCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performInsertVectorEltCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performFPRoundCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performSelectCombine(SDNode *N, DAGCombinerInfo &DCI) const;

  SDValue reassociateScalarOps(SDNode *N, SelectionDAG &DAG) const;
  unsigned getFusedOpcode(const SelectionDAG &DAG,
                          const SDNode *N0, const SDNode *N1) const;
  SDValue tryFoldToMad64_32(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue foldAddSub64WithZeroLowBitsTo32(SDNode *N,
                                          DAGCombinerInfo &DCI) const;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 242-272: Declares performAddCombine
```cpp
  SDValue performAddCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performPtrAddCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performAddCarrySubCarryCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performSubCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performFAddCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performFSubCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performFDivCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performFMulCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performFMACombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performSetCCCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performCvtF32UByteNCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performClampCombine(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue performRcpCombine(SDNode *N, DAGCombinerInfo &DCI) const;

  bool isLegalMUBUFAddressingMode(const AddrMode &AM) const;

  unsigned isCFIntrinsic(const SDNode *Intr) const;

public:
  /// \returns True if fixup needs to be emitted for given global value \p GV,
  /// false otherwise.
  bool shouldEmitFixup(const GlobalValue *GV) const;

  /// \returns True if GOT relocation needs to be emitted for given global value
  /// \p GV, false otherwise.
  bool shouldEmitGOTReloc(const GlobalValue *GV) const;

  /// \returns True if PC-relative relocation needs to be emitted for given
  /// global value \p GV, false otherwise.
  bool shouldEmitPCReloc(const GlobalValue *GV) const;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 273-301: Declares shouldUseLDSConstAddress
```cpp
  /// \returns true if this should use a literal constant for an LDS address,
  /// and not emit a relocation for an LDS global.
  bool shouldUseLDSConstAddress(const GlobalValue *GV) const;

  /// Check if EXTRACT_VECTOR_ELT/INSERT_VECTOR_ELT (<n x e>, var-idx) should be
  /// expanded into a set of cmp/select instructions.
  static bool shouldExpandVectorDynExt(unsigned EltSize, unsigned NumElem,
                                       bool IsDivergentIdx,
                                       const GCNSubtarget *Subtarget);

  bool shouldExpandVectorDynExt(SDNode *N) const;

  bool shouldPreservePtrArith(const Function &F, EVT PtrVT) const override;

  bool canTransformPtrArithOutOfBounds(const Function &F,
                                       EVT PtrVT) const override;

private:
  /// Returns true if the first real instruction in MBB is 8 bytes and could
  /// be split by a 32-byte fetch window boundary. Used on GFX950 to avoid
  /// instruction fetch delays.
  bool needsFetchWindowAlignment(const MachineBasicBlock &MBB) const;

  // Analyze a combined offset from an amdgcn_s_buffer_load intrinsic and store
  // the three offsets (voffset, soffset and instoffset) into the SDValue[3]
  // array pointed to by Offsets.
  void setBufferOffsets(SDValue CombinedOffset, SelectionDAG &DAG,
                        SDValue *Offsets, Align Alignment = Align(4)) const;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 302-333: Declares bufferRsrcPtrToVector
```cpp
  // Convert the i128 that an addrspace(8) pointer is natively represented as
  // into the v4i32 that all the buffer intrinsics expect to receive. We can't
  // add register classes for i128 on pain of the promotion logic going haywire,
  // so this slightly ugly hack is what we've got. If passed a non-pointer
  // argument (as would be seen in older buffer intrinsics), does nothing.
  SDValue bufferRsrcPtrToVector(SDValue MaybePointer, SelectionDAG &DAG) const;

  // Wrap a 64-bit pointer into a v4i32 (which is how all SelectionDAG code
  // represents ptr addrspace(8)) using the flags specified in the intrinsic.
  SDValue lowerPointerAsRsrcIntrin(SDNode *Op, SelectionDAG &DAG) const;

  // Handle 8 bit and 16 bit buffer loads
  SDValue handleByteShortBufferLoads(SelectionDAG &DAG, EVT LoadVT, SDLoc DL,
                                     ArrayRef<SDValue> Ops,
                                     MachineMemOperand *MMO,
                                     bool IsTFE = false) const;

  // Handle 8 bit and 16 bit buffer stores
  SDValue handleByteShortBufferStores(SelectionDAG &DAG, EVT VDataType,
                                      SDLoc DL, SDValue Ops[],
                                      MemSDNode *M) const;

public:
  SITargetLowering(const TargetMachine &tm, const GCNSubtarget &STI);

  const GCNSubtarget *getSubtarget() const;

  ArrayRef<MCPhysReg> getRoundingControlRegisters() const override;

  bool isFPExtFoldable(const SelectionDAG &DAG, unsigned Opcode, EVT DestVT,
                       EVT SrcVT) const override;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 334-367: Defines isFPExtFoldable
```cpp
  bool isFPExtFoldable(const MachineInstr &MI, unsigned Opcode, LLT DestTy,
                       LLT SrcTy) const override;

  bool isShuffleMaskLegal(ArrayRef<int> /*Mask*/, EVT /*VT*/) const override;

  // While address space 7 should never make it to codegen, it still needs to
  // have a MVT to prevent some analyses that query this function from breaking.
  // We use the custum MVT::amdgpuBufferFatPointer and
  // amdgpu::amdgpuBufferStridedPointer for this, though we use v8i32 for the
  // memory type (which is probably unused).
  MVT getPointerTy(const DataLayout &DL, unsigned AS) const override;
  MVT getPointerMemTy(const DataLayout &DL, unsigned AS) const override;

  void getTgtMemIntrinsic(SmallVectorImpl<IntrinsicInfo> &, const CallBase &,
                          MachineFunction &MF,
                          unsigned IntrinsicID) const override;

  void CollectTargetIntrinsicOperands(const CallInst &I,
                                      SmallVectorImpl<SDValue> &Ops,
                                      SelectionDAG &DAG) const override;

  bool getAddrModeArguments(const IntrinsicInst *I,
                            SmallVectorImpl<Value *> &Ops,
                            Type *&AccessTy) const override;

  bool isLegalFlatAddressingMode(const AddrMode &AM, unsigned AddrSpace) const;
  bool isLegalGlobalAddressingMode(const AddrMode &AM) const;
  bool isLegalAddressingMode(const DataLayout &DL, const AddrMode &AM, Type *Ty,
                             unsigned AS,
                             Instruction *I = nullptr) const override;

  bool canMergeStoresTo(unsigned AS, EVT MemVT,
                        const MachineFunction &MF) const override;

```
**EN:** This section contains concrete logic for isFPExtFoldable. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 isFPExtFoldable 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 368-399: Defines allowsMisalignedMemoryAccessesImpl
```cpp
  bool allowsMisalignedMemoryAccessesImpl(
      unsigned Size, unsigned AddrSpace, Align Alignment,
      MachineMemOperand::Flags Flags = MachineMemOperand::MONone,
      unsigned *IsFast = nullptr) const;

  bool allowsMisalignedMemoryAccesses(
      LLT Ty, unsigned AddrSpace, Align Alignment,
      MachineMemOperand::Flags Flags = MachineMemOperand::MONone,
      unsigned *IsFast = nullptr) const override {
    if (IsFast)
      *IsFast = 0;
    return allowsMisalignedMemoryAccessesImpl(Ty.getSizeInBits(), AddrSpace,
                                              Alignment, Flags, IsFast);
  }

  bool allowsMisalignedMemoryAccesses(
      EVT VT, unsigned AS, Align Alignment,
      MachineMemOperand::Flags Flags = MachineMemOperand::MONone,
      unsigned *IsFast = nullptr) const override;

  EVT getOptimalMemOpType(LLVMContext &Context, const MemOp &Op,
                          const AttributeList &FuncAttributes) const override;

  bool isMemOpHasNoClobberedMemOperand(const SDNode *N) const;

  static bool isNonGlobalAddrSpace(unsigned AS);

  bool isFreeAddrSpaceCast(unsigned SrcAS, unsigned DestAS) const override;

  TargetLoweringBase::LegalizeTypeAction
  getPreferredVectorAction(MVT VT) const override;

```
**EN:** This section contains concrete logic for allowsMisalignedMemoryAccessesImpl. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 allowsMisalignedMemoryAccessesImpl 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 400-433: Defines shouldConvertConstantLoadToIntImm
```cpp
  bool shouldConvertConstantLoadToIntImm(const APInt &Imm,
                                        Type *Ty) const override;

  bool isExtractSubvectorCheap(EVT ResVT, EVT SrcVT,
                               unsigned Index) const override;
  bool isExtractVecEltCheap(EVT VT, unsigned Index) const override;

  bool isTypeDesirableForOp(unsigned Op, EVT VT) const override;

  bool isOffsetFoldingLegal(const GlobalAddressSDNode *GA) const override;

  unsigned combineRepeatedFPDivisors() const override {
    // Combine multiple FDIVs with the same divisor into multiple FMULs by the
    // reciprocal.
    return 2;
  }

  bool supportSplitCSR(MachineFunction *MF) const override;
  void initializeSplitCSR(MachineBasicBlock *Entry) const override;
  void insertCopiesSplitCSR(
    MachineBasicBlock *Entry,
    const SmallVectorImpl<MachineBasicBlock *> &Exits) const override;

  SDValue LowerFormalArguments(SDValue Chain, CallingConv::ID CallConv,
                               bool isVarArg,
                               const SmallVectorImpl<ISD::InputArg> &Ins,
                               const SDLoc &DL, SelectionDAG &DAG,
                               SmallVectorImpl<SDValue> &InVals) const override;

  bool CanLowerReturn(CallingConv::ID CallConv,
                      MachineFunction &MF, bool isVarArg,
                      const SmallVectorImpl<ISD::OutputArg> &Outs,
                      LLVMContext &Context, const Type *RetTy) const override;

```
**EN:** This section contains concrete logic for shouldConvertConstantLoadToIntImm. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 shouldConvertConstantLoadToIntImm 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 434-464: Defines LowerReturn
```cpp
  SDValue LowerReturn(SDValue Chain, CallingConv::ID CallConv, bool IsVarArg,
                      const SmallVectorImpl<ISD::OutputArg> &Outs,
                      const SmallVectorImpl<SDValue> &OutVals, const SDLoc &DL,
                      SelectionDAG &DAG) const override;

  void passSpecialInputs(
    CallLoweringInfo &CLI,
    CCState &CCInfo,
    const SIMachineFunctionInfo &Info,
    SmallVectorImpl<std::pair<unsigned, SDValue>> &RegsToPass,
    SmallVectorImpl<SDValue> &MemOpChains,
    SDValue Chain) const;

  SDValue LowerCallResult(SDValue Chain, SDValue InGlue,
                          CallingConv::ID CallConv, bool isVarArg,
                          const SmallVectorImpl<ISD::InputArg> &Ins,
                          const SDLoc &DL, SelectionDAG &DAG,
                          SmallVectorImpl<SDValue> &InVals, bool isThisReturn,
                          SDValue ThisVal) const;

  bool mayBeEmittedAsTailCall(const CallInst *) const override;

  bool isEligibleForTailCallOptimization(
    SDValue Callee, CallingConv::ID CalleeCC, bool isVarArg,
    const SmallVectorImpl<ISD::OutputArg> &Outs,
    const SmallVectorImpl<SDValue> &OutVals,
    const SmallVectorImpl<ISD::InputArg> &Ins, SelectionDAG &DAG) const;

  SDValue LowerCall(CallLoweringInfo &CLI,
                    SmallVectorImpl<SDValue> &InVals) const override;

```
**EN:** This section contains concrete logic for LowerReturn. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 LowerReturn 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 465-496: Declares LowerDYNAMIC_STACKALLOC
```cpp
  SDValue LowerDYNAMIC_STACKALLOC(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerSTACKSAVE(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerGET_ROUNDING(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerSET_ROUNDING(SDValue Op, SelectionDAG &DAG) const;

  SDValue lowerPREFETCH(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerFP_EXTEND(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerGET_FPENV(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerSET_FPENV(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerROTR(SDValue Op, SelectionDAG &DAG) const;

  Register getRegisterByName(const char* RegName, LLT VT,
                             const MachineFunction &MF) const override;

  MachineBasicBlock *splitKillBlock(MachineInstr &MI,
                                    MachineBasicBlock *BB) const;

  void bundleInstWithWaitcnt(MachineInstr &MI) const;
  MachineBasicBlock *emitGWSMemViolTestLoop(MachineInstr &MI,
                                            MachineBasicBlock *BB) const;

  MachineBasicBlock *
  EmitInstrWithCustomInserter(MachineInstr &MI,
                              MachineBasicBlock *BB) const override;

  bool enableAggressiveFMAFusion(EVT VT) const override;
  bool enableAggressiveFMAFusion(LLT Ty) const override;
  EVT getSetCCResultType(const DataLayout &DL, LLVMContext &Context,
                         EVT VT) const override;
  MVT getScalarShiftAmountTy(const DataLayout &, EVT) const override;
  LLT getPreferredShiftAmountTy(LLT Ty) const override;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 497-518: Defines isFMAFasterThanFMulAndFAdd
```cpp
  bool isFMAFasterThanFMulAndFAdd(const MachineFunction &MF,
                                  EVT VT) const override;
  bool isFMAFasterThanFMulAndFAdd(const MachineFunction &MF,
                                  const LLT Ty) const override;
  bool isFMADLegal(const SelectionDAG &DAG, const SDNode *N) const override;
  bool isFMADLegal(const MachineInstr &MI, const LLT Ty) const override;

  SDValue splitUnaryVectorOp(SDValue Op, SelectionDAG &DAG) const;
  SDValue splitBinaryVectorOp(SDValue Op, SelectionDAG &DAG) const;
  SDValue splitTernaryVectorOp(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerOperation(SDValue Op, SelectionDAG &DAG) const override;
  void ReplaceNodeResults(SDNode *N, SmallVectorImpl<SDValue> &Results,
                          SelectionDAG &DAG) const override;

  SDValue PerformDAGCombine(SDNode *N, DAGCombinerInfo &DCI) const override;
  SDNode *PostISelFolding(MachineSDNode *N, SelectionDAG &DAG) const override;
  void AddMemOpInit(MachineInstr &MI) const;
  void AdjustInstrPostInstrSelection(MachineInstr &MI,
                                     SDNode *Node) const override;

  SDNode *legalizeTargetIndependentNode(SDNode *Node, SelectionDAG &DAG) const;

```
**EN:** This section contains concrete logic for isFMAFasterThanFMulAndFAdd. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 isFMAFasterThanFMulAndFAdd 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 519-540: Defines wrapAddr64Rsrc
```cpp
  MachineSDNode *wrapAddr64Rsrc(SelectionDAG &DAG, const SDLoc &DL,
                                SDValue Ptr) const;
  MachineSDNode *buildRSRC(SelectionDAG &DAG, const SDLoc &DL, SDValue Ptr,
                           uint32_t RsrcDword1, uint64_t RsrcDword2And3) const;
  std::pair<unsigned, const TargetRegisterClass *>
  getRegForInlineAsmConstraint(const TargetRegisterInfo *TRI,
                               StringRef Constraint, MVT VT) const override;
  ConstraintType getConstraintType(StringRef Constraint) const override;
  void LowerAsmOperandForConstraint(SDValue Op, StringRef Constraint,
                                    std::vector<SDValue> &Ops,
                                    SelectionDAG &DAG) const override;
  bool getAsmOperandConstVal(SDValue Op, uint64_t &Val) const;
  bool checkAsmConstraintVal(SDValue Op, StringRef Constraint,
                             uint64_t Val) const;
  bool checkAsmConstraintValA(SDValue Op,
                              uint64_t Val,
                              unsigned MaxSize = 64) const;
  SDValue copyToM0(SelectionDAG &DAG, SDValue Chain, const SDLoc &DL,
                   SDValue V) const;

  void finalizeLowering(MachineFunction &MF) const override;

```
**EN:** This section contains concrete logic for wrapAddr64Rsrc. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 wrapAddr64Rsrc 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 541-568: Defines computeKnownBitsForTargetNode
```cpp
  void computeKnownBitsForTargetNode(const SDValue Op, KnownBits &Known,
                                     const APInt &DemandedElts,
                                     const SelectionDAG &DAG,
                                     unsigned Depth = 0) const override;
  void computeKnownBitsForFrameIndex(int FrameIdx,
                                     KnownBits &Known,
                                     const MachineFunction &MF) const override;
  void computeKnownBitsForTargetInstr(GISelValueTracking &Analysis, Register R,
                                      KnownBits &Known,
                                      const APInt &DemandedElts,
                                      const MachineRegisterInfo &MRI,
                                      unsigned Depth = 0) const override;

  Align computeKnownAlignForTargetInstr(GISelValueTracking &Analysis,
                                        Register R,
                                        const MachineRegisterInfo &MRI,
                                        unsigned Depth = 0) const override;
  bool isSDNodeSourceOfDivergence(const SDNode *N, FunctionLoweringInfo *FLI,
                                  UniformityInfo *UA) const override;

  bool hasMemSDNodeUser(SDNode *N) const;

  bool isReassocProfitable(SelectionDAG &DAG, SDValue N0,
                           SDValue N1) const override;

  bool isReassocProfitable(MachineRegisterInfo &MRI, Register N0,
                           Register N1) const override;

```
**EN:** This section contains concrete logic for computeKnownBitsForTargetNode. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 computeKnownBitsForTargetNode 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 569-602: Defines isCanonicalized
```cpp
  bool isCanonicalized(SelectionDAG &DAG, SDValue Op,
                       SDNodeFlags UserFlags = {}, unsigned MaxDepth = 5) const;
  bool isCanonicalized(Register Reg, const MachineFunction &MF,
                       unsigned MaxDepth = 5) const;
  bool denormalsEnabledForType(const SelectionDAG &DAG, EVT VT) const;
  bool denormalsEnabledForType(LLT Ty, const MachineFunction &MF) const;

  bool isKnownNeverNaNForTargetNode(SDValue Op, const APInt &DemandedElts,
                                    const SelectionDAG &DAG, bool SNaN = false,
                                    unsigned Depth = 0) const override;
  AtomicExpansionKind
  shouldExpandAtomicRMWInIR(const AtomicRMWInst *) const override;
  AtomicExpansionKind shouldExpandAtomicLoadInIR(LoadInst *LI) const override;
  AtomicExpansionKind shouldExpandAtomicStoreInIR(StoreInst *SI) const override;
  AtomicExpansionKind
  shouldExpandAtomicCmpXchgInIR(const AtomicCmpXchgInst *AI) const override;

  void emitExpandAtomicAddrSpacePredicate(Instruction *AI) const;
  void emitExpandAtomicRMW(AtomicRMWInst *AI) const override;
  void emitExpandAtomicCmpXchg(AtomicCmpXchgInst *CI) const override;
  void emitExpandAtomicLoad(LoadInst *LI) const override;
  void emitExpandAtomicStore(StoreInst *SI) const override;

  LoadInst *
  lowerIdempotentRMWIntoFencedLoad(AtomicRMWInst *AI) const override;

  const TargetRegisterClass *getRegClassFor(MVT VT,
                                            bool isDivergent) const override;
  bool requiresUniformRegister(MachineFunction &MF,
                               const Value *V) const override;
  Align getPrefLoopAlignment(MachineLoop *ML) const override;
  unsigned
  getMaxPermittedBytesForAlignment(MachineBasicBlock *MBB) const override;

```
**EN:** This section contains concrete logic for isCanonicalized. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 isCanonicalized 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 603-634: Defines allocateHSAUserSGPRs
```cpp
  void allocateHSAUserSGPRs(CCState &CCInfo,
                            MachineFunction &MF,
                            const SIRegisterInfo &TRI,
                            SIMachineFunctionInfo &Info) const;

  void allocatePreloadKernArgSGPRs(CCState &CCInfo,
                                   SmallVectorImpl<CCValAssign> &ArgLocs,
                                   const SmallVectorImpl<ISD::InputArg> &Ins,
                                   MachineFunction &MF,
                                   const SIRegisterInfo &TRI,
                                   SIMachineFunctionInfo &Info) const;

  void allocateLDSKernelId(CCState &CCInfo, MachineFunction &MF,
                           const SIRegisterInfo &TRI,
                           SIMachineFunctionInfo &Info) const;

  void allocateSystemSGPRs(CCState &CCInfo,
                           MachineFunction &MF,
                           SIMachineFunctionInfo &Info,
                           CallingConv::ID CallConv,
                           bool IsShader) const;

  void allocateSpecialEntryInputVGPRs(CCState &CCInfo,
                                      MachineFunction &MF,
                                      const SIRegisterInfo &TRI,
                                      SIMachineFunctionInfo &Info) const;
  void allocateSpecialInputSGPRs(
    CCState &CCInfo,
    MachineFunction &MF,
    const SIRegisterInfo &TRI,
    SIMachineFunctionInfo &Info) const;

```
**EN:** This section contains concrete logic for allocateHSAUserSGPRs. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 allocateHSAUserSGPRs 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 635-654: Preprocessor guards and macros
```cpp
  void allocateSpecialInputVGPRs(CCState &CCInfo,
                                 MachineFunction &MF,
                                 const SIRegisterInfo &TRI,
                                 SIMachineFunctionInfo &Info) const;
  void allocateSpecialInputVGPRsFixed(CCState &CCInfo,
                                      MachineFunction &MF,
                                      const SIRegisterInfo &TRI,
                                      SIMachineFunctionInfo &Info) const;

  MachineMemOperand::Flags
  getTargetMMOFlags(const Instruction &I) const override;
};

// Returns true if argument is a boolean value which is not serialized into
// memory or argument and does not require v_cndmask_b32 to be deserialized.
bool isBoolSGPR(SDValue V);

} // End namespace llvm

#endif
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `GCNSubtarget`, `SIMachineFunctionInfo`, `SIRegisterInfo`, `ImageDimIntrinsicInfo`, `SITargetLowering`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; lowering / 降低
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPUArgumentUsageInfo.h"`
- `"AMDGPUISelLowering.h"`
- `"SIDefines.h"`
- `"llvm/CodeGen/MachineFunction.h"`
