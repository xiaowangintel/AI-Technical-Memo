# WebAssemblyISelLowering.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyISelLowering.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file defines the interfaces that WebAssembly uses to lower LLVM code into a selection DAG.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyISelLowering.h`，主要负责 WebAssembly 后端的SelectionDAG lowering 逻辑。 文件内容以接口、类型声明和协作关系说明为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//- WebAssemblyISelLowering.h - WebAssembly DAG Lowering Interface -*- C++ -*-//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-12

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// This file defines the interfaces that WebAssembly uses to lower LLVM
/// code into a selection DAG.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 13-19

```cpp
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_WEBASSEMBLY_WEBASSEMBLYISELLOWERING_H
#define LLVM_LIB_TARGET_WEBASSEMBLY_WEBASSEMBLYISELLOWERING_H

#include "llvm/CodeGen/TargetLowering.h"
```
- **EN**: Pulls in direct dependencies required by this SelectionDAG lowering logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该SelectionDAG lowering 逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 20-45

```cpp
namespace llvm {

class WebAssemblySubtarget;

class WebAssemblyTargetLowering final : public TargetLowering {
public:
  WebAssemblyTargetLowering(const TargetMachine &TM,
                            const WebAssemblySubtarget &STI);

  MVT getPointerTy(const DataLayout &DL, uint32_t AS = 0) const override;
  MVT getPointerMemTy(const DataLayout &DL, uint32_t AS = 0) const override;

private:
  /// Keep a pointer to the WebAssemblySubtarget around so that we can make the
  /// right decision when generating code for different targets.
  const WebAssemblySubtarget *Subtarget;

  AtomicExpansionKind
  shouldExpandAtomicRMWInIR(const AtomicRMWInst *) const override;
  bool shouldScalarizeBinop(SDValue VecOp) const override;
  FastISel *
  createFastISel(FunctionLoweringInfo &FuncInfo,
                 const TargetLibraryInfo *LibInfo,
                 const LibcallLoweringInfo *LibcallLowering) const override;
  MVT getScalarShiftAmountTy(const DataLayout &DL, EVT) const override;
  MachineBasicBlock *
```
- **EN**: Opens or re-enters namespace scope so backend symbols stay grouped under LLVM naming conventions. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里进入或重新进入命名空间作用域，使后端符号按照 LLVM 的命名约定组织在一起。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 46-71

```cpp
  EmitInstrWithCustomInserter(MachineInstr &MI,
                              MachineBasicBlock *MBB) const override;
  std::pair<unsigned, const TargetRegisterClass *>
  getRegForInlineAsmConstraint(const TargetRegisterInfo *TRI,
                               StringRef Constraint, MVT VT) const override;
  bool isCheapToSpeculateCttz(Type *Ty) const override;
  bool isCheapToSpeculateCtlz(Type *Ty) const override;
  bool isLegalAddressingMode(const DataLayout &DL, const AddrMode &AM, Type *Ty,
                             unsigned AS,
                             Instruction *I = nullptr) const override;
  bool allowsMisalignedMemoryAccesses(EVT, unsigned AddrSpace, Align Alignment,
                                      MachineMemOperand::Flags Flags,
                                      unsigned *Fast) const override;
  bool isIntDivCheap(EVT VT, AttributeList Attr) const override;
  bool isVectorLoadExtDesirable(SDValue ExtVal) const override;
  bool isOffsetFoldingLegal(const GlobalAddressSDNode *GA) const override;
  EVT getSetCCResultType(const DataLayout &DL, LLVMContext &Context,
                         EVT VT) const override;
  void getTgtMemIntrinsic(SmallVectorImpl<IntrinsicInfo> &Infos,
                          const CallBase &I, MachineFunction &MF,
                          unsigned Intrinsic) const override;

  void computeKnownBitsForTargetNode(const SDValue Op, KnownBits &Known,
                                     const APInt &DemandedElts,
                                     const SelectionDAG &DAG,
                                     unsigned Depth) const override;
```
- **EN**: Declares function entry points including `EmitInstrWithCustomInserter`, `getRegForInlineAsmConstraint`, `isCheapToSpeculateCttz` that other backend components call later. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `EmitInstrWithCustomInserter`, `getRegForInlineAsmConstraint`, `isCheapToSpeculateCttz`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 72-97

```cpp

  TargetLoweringBase::LegalizeTypeAction
  getPreferredVectorAction(MVT VT) const override;
  bool isFMAFasterThanFMulAndFAdd(const MachineFunction &MF,
                                  EVT VT) const override;

  bool isProfitableToCombineMinNumMaxNum(EVT VT) const override {
    // Prefer leaving cmp + select alone to form pmin/pmax,
    // or relaxed_fmin/relaxed_fmax with appropriate FMF.
    return false;
  }

  SDValue LowerCall(CallLoweringInfo &CLI,
                    SmallVectorImpl<SDValue> &InVals) const override;
  bool CanLowerReturn(CallingConv::ID CallConv, MachineFunction &MF,
                      bool isVarArg,
                      const SmallVectorImpl<ISD::OutputArg> &Outs,
                      LLVMContext &Context, const Type *RetTy) const override;
  SDValue LowerReturn(SDValue Chain, CallingConv::ID CallConv, bool isVarArg,
                      const SmallVectorImpl<ISD::OutputArg> &Outs,
                      const SmallVectorImpl<SDValue> &OutVals, const SDLoc &dl,
                      SelectionDAG &DAG) const override;
  SDValue LowerFormalArguments(SDValue Chain, CallingConv::ID CallConv,
                               bool IsVarArg,
                               const SmallVectorImpl<ISD::InputArg> &Ins,
                               const SDLoc &DL, SelectionDAG &DAG,
```
- **EN**: Implements helper routine(s) `getPreferredVectorAction`, `isFMAFasterThanFMulAndFAdd`, `isProfitableToCombineMinNumMaxNum` for this portion of the WebAssembly backend SelectionDAG lowering logic. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里实现了 WebAssembly 后端该部分SelectionDAG lowering 逻辑所需的辅助例程 `getPreferredVectorAction`, `isFMAFasterThanFMulAndFAdd`, `isProfitableToCombineMinNumMaxNum`。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 98-123

```cpp
                               SmallVectorImpl<SDValue> &InVals) const override;

  void ReplaceNodeResults(SDNode *N, SmallVectorImpl<SDValue> &Results,
                          SelectionDAG &DAG) const override;

  bool
  shouldSimplifyDemandedVectorElts(SDValue Op,
                                   const TargetLoweringOpt &TLO) const override;

  // Custom lowering hooks.
  SDValue LowerOperation(SDValue Op, SelectionDAG &DAG) const override;
  SDValue LowerFrameIndex(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerRETURNADDR(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerFRAMEADDR(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerGlobalAddress(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerGlobalTLSAddress(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerExternalSymbol(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerBR_JT(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerJumpTable(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerVASTART(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerCopyToReg(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerIntrinsic(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerSIGN_EXTEND_INREG(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerEXTEND_VECTOR_INREG(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerBUILD_VECTOR(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerVECTOR_SHUFFLE(SDValue Op, SelectionDAG &DAG) const;
```
- **EN**: Declares function entry points including `ReplaceNodeResults`, `shouldSimplifyDemandedVectorElts`, `LowerOperation` that other backend components call later. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `ReplaceNodeResults`, `shouldSimplifyDemandedVectorElts`, `LowerOperation`。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 124-141

```cpp
  SDValue LowerSETCC(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerAccessVectorElement(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerShift(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerFP_TO_INT_SAT(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerFMIN(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerFMAX(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerLoad(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerStore(SDValue Op, SelectionDAG &DAG) const;
  SDValue LowerMUL_LOHI(SDValue Op, SelectionDAG &DAG) const;
  SDValue Replace128Op(SDNode *N, SelectionDAG &DAG) const;
  SDValue LowerUADDO(SDValue Op, SelectionDAG &DAG) const;

  // Custom DAG combine hooks
  SDValue
  PerformDAGCombine(SDNode *N,
                    TargetLowering::DAGCombinerInfo &DCI) const override;
};
```
- **EN**: Declares function entry points including `LowerSETCC`, `LowerAccessVectorElement`, `LowerShift` that other backend components call later. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `LowerSETCC`, `LowerAccessVectorElement`, `LowerShift`。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 142-149

```cpp
namespace WebAssembly {
FastISel *createFastISel(FunctionLoweringInfo &funcInfo,
                         const TargetLibraryInfo *libInfo,
                         const LibcallLoweringInfo *libcallLowering);
} // end namespace WebAssembly

} // end namespace llvm
```
- **EN**: Opens or re-enters namespace scope so backend symbols stay grouped under LLVM naming conventions. Notable symbols in this range include `createFastISel`.
- **CN**: 这里进入或重新进入命名空间作用域，使后端符号按照 LLVM 的命名约定组织在一起。 该区间中较显眼的符号包括 `createFastISel`。

### Lines 150-150

```cpp
#endif
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- SelectionDAG lowering logic / SelectionDAG lowering 逻辑
- SelectionDAG lowering / SelectionDAG lowering
- Calling convention handling / 调用约定处理
- Register modeling / 寄存器建模
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Target machine configuration / 目标机器配置

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `llvm/CodeGen/TargetLowering.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
