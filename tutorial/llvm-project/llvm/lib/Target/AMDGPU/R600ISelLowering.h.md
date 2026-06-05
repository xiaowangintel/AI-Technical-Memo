# R600ISelLowering.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/R600ISelLowering.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for R600ISelLowering in the LLVM AMDGPU backend. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM AMDGPU 后端中 R600ISelLowering 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: File banner, includes, and setup
```cpp
//===-- R600ISelLowering.h - R600 DAG Lowering Interface -*- C++ -*--------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// R600 DAG Lowering interface definition
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPU_R600ISELLOWERING_H
#define LLVM_LIB_TARGET_AMDGPU_R600ISELLOWERING_H

#include "AMDGPUISelLowering.h"
#include "llvm/CodeGen/MachineFunction.h"

namespace llvm {

class R600Subtarget;

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `R600Subtarget`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`R600Subtarget`。

### Lines 24-47: Declares class R600TargetLowering
```cpp
class R600TargetLowering final : public AMDGPUTargetLowering {

  const R600Subtarget *Subtarget;
public:
  R600TargetLowering(const TargetMachine &TM, const R600Subtarget &STI);

  const R600Subtarget *getSubtarget() const;

  MachineBasicBlock *
  EmitInstrWithCustomInserter(MachineInstr &MI,
                              MachineBasicBlock *BB) const override;
  SDValue LowerOperation(SDValue Op, SelectionDAG &DAG) const override;
  SDValue PerformDAGCombine(SDNode *N, DAGCombinerInfo &DCI) const override;
  void ReplaceNodeResults(SDNode * N,
                          SmallVectorImpl<SDValue> &Results,
                          SelectionDAG &DAG) const override;
  CCAssignFn *CCAssignFnForCall(CallingConv::ID CC, bool IsVarArg) const;
  SDValue LowerFormalArguments(SDValue Chain, CallingConv::ID CallConv,
                               bool isVarArg,
                               const SmallVectorImpl<ISD::InputArg> &Ins,
                               const SDLoc &DL, SelectionDAG &DAG,
                               SmallVectorImpl<SDValue> &InVals) const override;
  EVT getSetCCResultType(const DataLayout &DL, LLVMContext &,
                         EVT VT) const override;
```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `R600TargetLowering`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`R600TargetLowering`。

### Lines 48-66: Defines canMergeStoresTo
```cpp

  bool canMergeStoresTo(unsigned AS, EVT MemVT,
                        const MachineFunction &MF) const override;

  bool allowsMisalignedMemoryAccesses(
      EVT VT, unsigned AS, Align Alignment,
      MachineMemOperand::Flags Flags = MachineMemOperand::MONone,
      unsigned *IsFast = nullptr) const override;

  bool canCombineTruncStore(EVT ValVT, EVT MemVT, Align Alignment,
                            unsigned AddrSpace,
                            bool LegalOperations) const override {
    // R600 has "custom" lowering for truncating stores despite not supporting
    // those instructions. If we allow that custom lowering in the DAG combiner
    // then all truncates are merged into truncating stores, giving worse code
    // generation. This hook prevents the DAG combiner performing that combine.
    return isTruncStoreLegal(ValVT, MemVT, Alignment, AddrSpace);
  }

```
**EN:** This section contains concrete logic for canMergeStoresTo. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 canMergeStoresTo 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 67-88: Defines LowerImplicitParameter
```cpp
private:
  unsigned Gen;
  /// Each OpenCL kernel has nine implicit parameters that are stored in the
  /// first nine dwords of a Vertex Buffer.  These implicit parameters are
  /// lowered to load instructions which retrieve the values from the Vertex
  /// Buffer.
  SDValue LowerImplicitParameter(SelectionDAG &DAG, EVT VT, const SDLoc &DL,
                                 unsigned DwordOffset) const;

  void lowerImplicitParameter(MachineInstr *MI, MachineBasicBlock &BB,
      MachineRegisterInfo & MRI, unsigned dword_offset) const;
  SDValue OptimizeSwizzle(SDValue BuildVector, SDValue Swz[],
                          SelectionDAG &DAG, const SDLoc &DL) const;
  SDValue vectorToVerticalVector(SelectionDAG &DAG, SDValue Vector) const;

  SDValue lowerFrameIndex(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerEXTRACT_VECTOR_ELT(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerINSERT_VECTOR_ELT(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerGlobalAddress(AMDGPUMachineFunctionInfo *MFI, SDValue Op,
                             SelectionDAG &DAG) const override;
  SDValue LowerSELECT_CC(SDValue Op, SelectionDAG &DAG) const;

```
**EN:** This section contains concrete logic for LowerImplicitParameter. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 LowerImplicitParameter 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 89-110: Declares lowerPrivateTruncStore
```cpp
  SDValue lowerPrivateTruncStore(StoreSDNode *Store, SelectionDAG &DAG) const;
  SDValue LowerSTORE(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerFP_TO_UINT(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerFP_TO_SINT(SDValue Op, SelectionDAG &DAG) const;

  SDValue lowerPrivateExtLoad(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerLOAD(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerADDRSPACECAST(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerBRCOND(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerTrig(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerShiftParts(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerUADDSUBO(SDValue Op, SelectionDAG &DAG,
                        unsigned mainop, unsigned ovf) const;

  SDValue stackPtrToRegIndex(SDValue Ptr, unsigned StackWidth,
                                          SelectionDAG &DAG) const;
  void getStackAddress(unsigned StackWidth, unsigned ElemIdx,
                       unsigned &Channel, unsigned &PtrIncr) const;
  bool isZero(SDValue Op) const;
  bool isHWTrueValue(SDValue Op) const;
  bool isHWFalseValue(SDValue Op) const;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 111-125: Preprocessor guards and macros
```cpp
  bool FoldOperand(SDNode *ParentNode, unsigned SrcIdx, SDValue &Src,
                   SDValue &Neg, SDValue &Abs, SDValue &Sel, SDValue &Imm,
                   SelectionDAG &DAG) const;
  SDValue constBufferLoad(LoadSDNode *LoadNode, int Block,
                          SelectionDAG &DAG) const;

  SDNode *PostISelFolding(MachineSDNode *N, SelectionDAG &DAG) const override;

  TargetLowering::AtomicExpansionKind
  shouldExpandAtomicRMWInIR(const AtomicRMWInst *RMW) const override;
};

} // End namespace llvm;

#endif
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `R600Subtarget`, `R600TargetLowering`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; lowering / 降低
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPUISelLowering.h"`
- `"llvm/CodeGen/MachineFunction.h"`
