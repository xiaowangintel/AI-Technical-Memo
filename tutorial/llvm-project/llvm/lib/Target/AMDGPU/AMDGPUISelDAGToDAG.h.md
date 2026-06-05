# AMDGPUISelDAGToDAG.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUISelDAGToDAG.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for AMDGPUISelDAGToDAG in the LLVM AMDGPU backend. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM AMDGPU 后端中 AMDGPUISelDAGToDAG 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-27: File banner, includes, and setup
```cpp
//===-- AMDGPUISelDAGToDAG.h - A dag to dag inst selector for AMDGPU ----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//==-----------------------------------------------------------------------===//
//
/// \file
/// Defines an instruction selector for the AMDGPU target.
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPU_AMDGPUISELDAGTODAG_H
#define LLVM_LIB_TARGET_AMDGPU_AMDGPUISELDAGTODAG_H

#include "AMDGPUSelectionDAGInfo.h"
#include "GCNSubtarget.h"
#include "SIMachineFunctionInfo.h"
#include "SIModeRegisterDefaults.h"
#include "llvm/Analysis/ValueTracking.h"
#include "llvm/CodeGen/SelectionDAGISel.h"
#include "llvm/Support/AMDGPUAddrSpace.h"
#include "llvm/Target/TargetMachine.h"

namespace llvm {

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 28-58: Declares class AMDGPUDAGToDAGISel
```cpp
static inline bool getConstantValue(SDValue N, uint32_t &Out) {
  // This is only used for packed vectors, where using 0 for undef should
  // always be good.
  if (N.isUndef()) {
    Out = 0;
    return true;
  }

  if (const ConstantSDNode *C = dyn_cast<ConstantSDNode>(N)) {
    Out = C->getAPIntValue().getSExtValue();
    return true;
  }

  if (const ConstantFPSDNode *C = dyn_cast<ConstantFPSDNode>(N)) {
    Out = C->getValueAPF().bitcastToAPInt().getSExtValue();
    return true;
  }

  return false;
}

/// AMDGPU specific code to select AMDGPU machine instructions for
/// SelectionDAG operations.
class AMDGPUDAGToDAGISel : public SelectionDAGISel {
  // Subtarget - Keep a pointer to the AMDGPU Subtarget around so that we can
  // make the right decision when generating code for different targets.
  const GCNSubtarget *Subtarget;

  // Default FP mode for the current function.
  SIModeRegisterDefaults Mode;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUDAGToDAGISel`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUDAGToDAGISel`。

### Lines 59-91: Declares fp16SrcZerosHighBits
```cpp
  // Instructions that will be lowered with a final instruction that zeros the
  // high result bits.
  bool fp16SrcZerosHighBits(unsigned Opc) const;

public:
  AMDGPUDAGToDAGISel() = delete;

  explicit AMDGPUDAGToDAGISel(TargetMachine &TM, CodeGenOptLevel OptLevel);

  bool runOnMachineFunction(MachineFunction &MF) override;
  bool matchLoadD16FromBuildVector(SDNode *N) const;
  void PreprocessISelDAG() override;
  void Select(SDNode *N) override;
  void PostprocessISelDAG() override;

protected:
  void SelectBuildVector(SDNode *N, unsigned RegClassID);
  void SelectVectorShuffle(SDNode *N);
  bool isSDWAOperand(const SDNode *N) const;

private:
  std::pair<SDValue, SDValue> foldFrameIndex(SDValue N) const;

  bool isInlineImmediate(const SDNode *N) const;

  bool isInlineImmediate(const APInt &Imm) const {
    return Subtarget->getInstrInfo()->isInlineConstant(Imm);
  }

  bool isInlineImmediate(const APFloat &Imm) const {
    return Subtarget->getInstrInfo()->isInlineConstant(Imm);
  }

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 92-121: Declares isVGPRImm
```cpp
  bool isVGPRImm(const SDNode *N) const;
  bool isUniformLoad(const SDNode *N) const;
  bool isUniformBr(const SDNode *N) const;

  MachineSDNode *buildRegSequence16(SmallVectorImpl<SDValue> &Elts,
                                    const SDLoc &DL) const;
  MachineSDNode *buildRegSequence32(SmallVectorImpl<SDValue> &Elts,
                                    const SDLoc &DL) const;
  MachineSDNode *buildRegSequence(SmallVectorImpl<SDValue> &Elts,
                                  const SDLoc &DL, unsigned ElementSize) const;

  void selectWMMAModsNegAbs(unsigned ModOpcode, unsigned &Mods,
                            SmallVectorImpl<SDValue> &Elts, SDValue &Src,
                            const SDLoc &DL, unsigned ElementSize) const;

  // Returns true if ISD::AND SDNode `N`'s masking of the shift amount operand's
  // `ShAmtBits` bits is unneeded.
  bool isUnneededShiftMask(const SDNode *N, unsigned ShAmtBits) const;

  bool isBaseWithConstantOffset64(SDValue Addr, SDValue &LHS,
                                  SDValue &RHS) const;

  MachineSDNode *buildSMovImm64(SDLoc &DL, uint64_t Val, EVT VT) const;

  SDNode *packConstantV2I16(const SDNode *N, SelectionDAG &DAG) const;

  SDNode *glueCopyToOp(SDNode *N, SDValue NewChain, SDValue Glue) const;
  SDNode *glueCopyToM0(SDNode *N, SDValue Val) const;
  SDNode *glueCopyToM0LDSInit(SDNode *N) const;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 122-152: Declares getOperandRegClass
```cpp
  const TargetRegisterClass *getOperandRegClass(SDNode *N, unsigned OpNo) const;
  virtual bool SelectADDRVTX_READ(SDValue Addr, SDValue &Base, SDValue &Offset);
  virtual bool SelectADDRIndirect(SDValue Addr, SDValue &Base, SDValue &Offset);
  bool isDSOffsetLegal(SDValue Base, unsigned Offset) const;
  bool isDSOffset2Legal(SDValue Base, unsigned Offset0, unsigned Offset1,
                        unsigned Size) const;

  bool isFlatScratchBaseLegal(SDValue Addr) const;
  bool isFlatScratchBaseLegalSV(SDValue Addr) const;
  bool isFlatScratchBaseLegalSVImm(SDValue Addr) const;
  bool isSOffsetLegalWithImmOffset(SDValue *SOffset, bool Imm32Only,
                                   bool IsBuffer, int64_t ImmOffset = 0) const;

  bool SelectDS1Addr1Offset(SDValue Ptr, SDValue &Base, SDValue &Offset) const;
  bool SelectDS64Bit4ByteAligned(SDValue Ptr, SDValue &Base, SDValue &Offset0,
                                 SDValue &Offset1) const;
  bool SelectDS128Bit8ByteAligned(SDValue Ptr, SDValue &Base, SDValue &Offset0,
                                  SDValue &Offset1) const;
  bool SelectDSReadWrite2(SDValue Ptr, SDValue &Base, SDValue &Offset0,
                          SDValue &Offset1, unsigned Size) const;
  bool SelectMUBUF(SDValue Addr, SDValue &SRsrc, SDValue &VAddr,
                   SDValue &SOffset, SDValue &Offset, SDValue &Offen,
                   SDValue &Idxen, SDValue &Addr64) const;
  bool SelectMUBUFAddr64(SDValue Addr, SDValue &SRsrc, SDValue &VAddr,
                         SDValue &SOffset, SDValue &Offset) const;
  bool SelectMUBUFScratchOffen(SDNode *Parent, SDValue Addr, SDValue &RSrc,
                               SDValue &VAddr, SDValue &SOffset,
                               SDValue &ImmOffset) const;
  bool SelectMUBUFScratchOffset(SDNode *Parent, SDValue Addr, SDValue &SRsrc,
                                SDValue &Soffset, SDValue &Offset) const;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 153-186: Defines SelectMUBUFOffset
```cpp
  bool SelectMUBUFOffset(SDValue Addr, SDValue &SRsrc, SDValue &Soffset,
                         SDValue &Offset) const;
  bool SelectBUFSOffset(SDValue Addr, SDValue &SOffset) const;

  bool SelectFlatOffsetImpl(SDNode *N, SDValue Addr, SDValue &VAddr,
                            SDValue &Offset, uint64_t FlatVariant) const;
  bool SelectFlatOffset(SDNode *N, SDValue Addr, SDValue &VAddr,
                        SDValue &Offset) const;
  bool SelectGlobalOffset(SDNode *N, SDValue Addr, SDValue &VAddr,
                          SDValue &Offset) const;
  bool SelectScratchOffset(SDNode *N, SDValue Addr, SDValue &VAddr,
                           SDValue &Offset) const;
  bool SelectGlobalSAddr(SDNode *N, SDValue Addr, SDValue &SAddr,
                         SDValue &VOffset, SDValue &Offset, bool &ScaleOffset,
                         bool NeedIOffset = true) const;
  bool SelectGlobalSAddr(SDNode *N, SDValue Addr, SDValue &SAddr,
                         SDValue &VOffset, SDValue &Offset,
                         SDValue &CPol) const;
  bool SelectGlobalSAddrCPol(SDNode *N, SDValue Addr, SDValue &SAddr,
                             SDValue &VOffset, SDValue &Offset,
                             SDValue &CPol) const;
  bool SelectGlobalSAddrCPolM0(SDNode *N, SDValue Addr, SDValue &SAddr,
                               SDValue &VOffset, SDValue &Offset,
                               SDValue &CPol) const;
  bool SelectGlobalSAddrGLC(SDNode *N, SDValue Addr, SDValue &SAddr,
                            SDValue &VOffset, SDValue &Offset,
                            SDValue &CPol) const;
  bool SelectGlobalSAddrNoIOffset(SDNode *N, SDValue Addr, SDValue &SAddr,
                                  SDValue &VOffset, SDValue &CPol) const;
  bool SelectGlobalSAddrNoIOffsetM0(SDNode *N, SDValue Addr, SDValue &SAddr,
                                    SDValue &VOffset, SDValue &CPol) const;
  bool SelectScratchSAddr(SDNode *N, SDValue Addr, SDValue &SAddr,
                          SDValue &Offset) const;
  bool checkFlatScratchSVSSwizzleBug(SDValue VAddr, SDValue SAddr,
```
**EN:** This section contains concrete logic for SelectMUBUFOffset. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 SelectMUBUFOffset 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 187-220: Defines SelectScratchSVAddr
```cpp
                                     uint64_t ImmOffset) const;
  bool SelectScratchSVAddr(SDNode *N, SDValue Addr, SDValue &VAddr,
                           SDValue &SAddr, SDValue &Offset,
                           SDValue &CPol) const;

  bool SelectSMRDOffset(SDNode *N, SDValue ByteOffsetNode, SDValue *SOffset,
                        SDValue *Offset, bool Imm32Only = false,
                        bool IsBuffer = false, bool HasSOffset = false,
                        int64_t ImmOffset = 0,
                        bool *ScaleOffset = nullptr) const;
  SDValue Expand32BitAddress(SDValue Addr) const;
  bool SelectSMRDBaseOffset(SDNode *N, SDValue Addr, SDValue &SBase,
                            SDValue *SOffset, SDValue *Offset,
                            bool Imm32Only = false, bool IsBuffer = false,
                            bool HasSOffset = false, int64_t ImmOffset = 0,
                            bool *ScaleOffset = nullptr) const;
  bool SelectSMRD(SDNode *N, SDValue Addr, SDValue &SBase, SDValue *SOffset,
                  SDValue *Offset, bool Imm32Only = false,
                  bool *ScaleOffset = nullptr) const;
  bool SelectSMRDImm(SDValue Addr, SDValue &SBase, SDValue &Offset) const;
  bool SelectSMRDImm32(SDValue Addr, SDValue &SBase, SDValue &Offset) const;
  bool SelectScaleOffset(SDNode *N, SDValue &Offset, bool IsSigned) const;
  bool SelectSMRDSgpr(SDNode *N, SDValue Addr, SDValue &SBase, SDValue &SOffset,
                      SDValue &CPol) const;
  bool SelectSMRDSgprImm(SDNode *N, SDValue Addr, SDValue &SBase,
                         SDValue &SOffset, SDValue &Offset,
                         SDValue &CPol) const;
  bool SelectSMRDBufferImm(SDValue N, SDValue &Offset) const;
  bool SelectSMRDBufferImm32(SDValue N, SDValue &Offset) const;
  bool SelectSMRDBufferSgprImm(SDValue N, SDValue &SOffset,
                               SDValue &Offset) const;
  bool SelectSMRDPrefetchImm(SDValue Addr, SDValue &SBase,
                             SDValue &Offset) const;
  bool SelectMOVRELOffset(SDValue Index, SDValue &Base, SDValue &Offset) const;
```
**EN:** This section contains concrete logic for SelectScratchSVAddr. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 SelectScratchSVAddr 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 221-253: Defines SelectVOP3ModsImpl
```cpp

  bool SelectVOP3ModsImpl(SDValue In, SDValue &Src, unsigned &SrcMods,
                          bool IsCanonicalizing = true,
                          bool AllowAbs = true) const;
  bool SelectVOP3Mods(SDValue In, SDValue &Src, SDValue &SrcMods) const;
  bool SelectVOP3ModsNonCanonicalizing(SDValue In, SDValue &Src,
                                       SDValue &SrcMods) const;
  bool SelectVOP3BMods(SDValue In, SDValue &Src, SDValue &SrcMods) const;
  bool SelectVOP3NoMods(SDValue In, SDValue &Src) const;
  bool SelectVOP3Mods0(SDValue In, SDValue &Src, SDValue &SrcMods,
                       SDValue &Clamp, SDValue &Omod) const;
  bool SelectVOP3BMods0(SDValue In, SDValue &Src, SDValue &SrcMods,
                        SDValue &Clamp, SDValue &Omod) const;
  bool SelectVOP3NoMods0(SDValue In, SDValue &Src, SDValue &SrcMods,
                         SDValue &Clamp, SDValue &Omod) const;

  bool SelectVINTERPModsImpl(SDValue In, SDValue &Src, SDValue &SrcMods,
                             bool OpSel) const;
  bool SelectVINTERPMods(SDValue In, SDValue &Src, SDValue &SrcMods) const;
  bool SelectVINTERPModsHi(SDValue In, SDValue &Src, SDValue &SrcMods) const;

  bool SelectVOP3OMods(SDValue In, SDValue &Src, SDValue &Clamp,
                       SDValue &Omod) const;

  bool SelectVOP3PMods(SDValue In, SDValue &Src, SDValue &SrcMods,
                       bool IsDOT = false) const;
  bool SelectVOP3PModsDOT(SDValue In, SDValue &Src, SDValue &SrcMods) const;
  bool SelectVOP3PNoModsDOT(SDValue In, SDValue &Src) const;
  bool SelectVOP3PModsF32(SDValue In, SDValue &Src, SDValue &SrcMods) const;
  bool SelectVOP3PNoModsF32(SDValue In, SDValue &Src) const;

  bool SelectWMMAOpSelVOP3PMods(SDValue In, SDValue &Src) const;

```
**EN:** This section contains concrete logic for SelectVOP3ModsImpl. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 SelectVOP3ModsImpl 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 254-284: Defines SelectWMMAModsF32NegAbs
```cpp
  bool SelectWMMAModsF32NegAbs(SDValue In, SDValue &Src,
                               SDValue &SrcMods) const;
  bool SelectWMMAModsF16Neg(SDValue In, SDValue &Src, SDValue &SrcMods) const;
  bool SelectWMMAModsF16NegAbs(SDValue In, SDValue &Src,
                               SDValue &SrcMods) const;
  bool SelectWMMAVISrc(SDValue In, SDValue &Src) const;

  bool SelectSWMMACIndex8(SDValue In, SDValue &Src, SDValue &IndexKey) const;
  bool SelectSWMMACIndex16(SDValue In, SDValue &Src, SDValue &IndexKey) const;
  bool SelectSWMMACIndex32(SDValue In, SDValue &Src, SDValue &IndexKey) const;

  bool SelectVOP3OpSel(SDValue In, SDValue &Src, SDValue &SrcMods) const;

  bool SelectVOP3OpSelMods(SDValue In, SDValue &Src, SDValue &SrcMods) const;
  bool SelectVOP3PMadMixModsImpl(SDValue In, SDValue &Src, unsigned &Mods,
                                 MVT VT) const;
  bool SelectVOP3PMadMixModsExt(SDValue In, SDValue &Src,
                                SDValue &SrcMods) const;
  bool SelectVOP3PMadMixMods(SDValue In, SDValue &Src, SDValue &SrcMods) const;
  bool SelectVOP3PMadMixBF16ModsExt(SDValue In, SDValue &Src,
                                    SDValue &SrcMods) const;
  bool SelectVOP3PMadMixBF16Mods(SDValue In, SDValue &Src,
                                 SDValue &SrcMods) const;

  bool SelectBITOP3(SDValue In, SDValue &Src0, SDValue &Src1, SDValue &Src2,
                   SDValue &Tbl) const;

  SDValue getHi16Elt(SDValue In) const;

  SDValue getMaterializedScalarImm32(int64_t Val, const SDLoc &DL) const;

```
**EN:** This section contains concrete logic for SelectWMMAModsF32NegAbs. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 SelectWMMAModsF32NegAbs 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 285-316: Header dependencies and setup
```cpp
  void SelectADD_SUB_I64(SDNode *N);
  void SelectAddcSubb(SDNode *N);
  void SelectUADDO_USUBO(SDNode *N);
  void SelectDIV_SCALE(SDNode *N);
  void SelectMAD_64_32(SDNode *N);
  void SelectMUL_LOHI(SDNode *N);
  void SelectFMA_W_CHAIN(SDNode *N);
  void SelectFMUL_W_CHAIN(SDNode *N);
  SDNode *getBFE32(bool IsSigned, const SDLoc &DL, SDValue Val, uint32_t Offset,
                   uint32_t Width);
  void SelectS_BFEFromShifts(SDNode *N);
  void SelectS_BFE(SDNode *N);
  bool isCBranchSCC(const SDNode *N) const;
  void SelectBRCOND(SDNode *N);
  void SelectFMAD_FMA(SDNode *N);
  void SelectFP_EXTEND(SDNode *N);
  void SelectDSAppendConsume(SDNode *N, unsigned IntrID);
  void SelectDSBvhStackIntrinsic(SDNode *N, unsigned IntrID);
  void SelectTensorLoadStore(SDNode *N, unsigned IntrID);
  void SelectDS_GWS(SDNode *N, unsigned IntrID);
  void SelectInterpP1F16(SDNode *N);
  void SelectINTRINSIC_W_CHAIN(SDNode *N);
  void SelectINTRINSIC_WO_CHAIN(SDNode *N);
  void SelectINTRINSIC_VOID(SDNode *N);
  void SelectWAVE_ADDRESS(SDNode *N);
  void SelectSTACKRESTORE(SDNode *N);

protected:
  // Include the pieces autogenerated from the target description.
#include "AMDGPUGenDAGISel.inc"
};

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。

### Lines 317-338: Preprocessor guards and macros
```cpp
class AMDGPUISelDAGToDAGPass : public SelectionDAGISelPass {
public:
  AMDGPUISelDAGToDAGPass(TargetMachine &TM);

  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);
};

class AMDGPUDAGToDAGISelLegacy : public SelectionDAGISelLegacy {
public:
  static char ID;

  AMDGPUDAGToDAGISelLegacy(TargetMachine &TM, CodeGenOptLevel OptLevel);

  bool runOnMachineFunction(MachineFunction &MF) override;
  void getAnalysisUsage(AnalysisUsage &AU) const override;
  StringRef getPassName() const override;
};

} // namespace llvm

#endif // LLVM_LIB_TARGET_AMDGPU_AMDGPUISELDAGTODAG_H
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `AMDGPUISelDAGToDAGPass`, `AMDGPUDAGToDAGISelLegacy`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`AMDGPUISelDAGToDAGPass`, `AMDGPUDAGToDAGISelLegacy`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `AMDGPUDAGToDAGISel`, `AMDGPUISelDAGToDAGPass`, `AMDGPUDAGToDAGISelLegacy`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; lowering / 降低
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPUSelectionDAGInfo.h"`
- `"GCNSubtarget.h"`
- `"SIMachineFunctionInfo.h"`
- `"SIModeRegisterDefaults.h"`
- `"llvm/Analysis/ValueTracking.h"`
- `"llvm/CodeGen/SelectionDAGISel.h"`
- `"llvm/Support/AMDGPUAddrSpace.h"`
- `"llvm/Target/TargetMachine.h"`
- `"AMDGPUGenDAGISel.inc"`
