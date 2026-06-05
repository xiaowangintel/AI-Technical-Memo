# SparcISelLowering.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Sparc/SparcISelLowering.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Customizes SelectionDAG lowering, legalization, calling convention handling, and target-specific DAG nodes.
  - **CN**: 定制 SelectionDAG 降低、合法化、调用约定处理以及目标专用 DAG 节点。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- SparcISelLowering.h - Sparc DAG Lowering Interface ------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-16
```cpp
//
// This file defines the interfaces that Sparc uses to lower LLVM code into a
// selection DAG.
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_SPARC_SPARCISELLOWERING_H
#define LLVM_LIB_TARGET_SPARC_SPARCISELLOWERING_H

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 17-28
```cpp
#include "Sparc.h"
#include "llvm/CodeGen/TargetLowering.h"

namespace llvm {
  class SparcSubtarget;

  class SparcTargetLowering : public TargetLowering {
    const SparcSubtarget *Subtarget;
  public:
    SparcTargetLowering(const TargetMachine &TM, const SparcSubtarget &STI);
    SDValue LowerOperation(SDValue Op, SelectionDAG &DAG) const override;

```
- **EN**: Pulls in the headers needed for this implementation, including `Sparc.h`, `llvm/CodeGen/TargetLowering.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `Sparc.h`, `llvm/CodeGen/TargetLowering.h`。

### Lines 29-39
```cpp
    bool useSoftFloat() const override;

    /// computeKnownBitsForTargetNode - Determine which of the bits specified
    /// in Mask are known to be either zero or one and return them in the
    /// KnownZero/KnownOne bitsets.
    void computeKnownBitsForTargetNode(const SDValue Op,
                                       KnownBits &Known,
                                       const APInt &DemandedElts,
                                       const SelectionDAG &DAG,
                                       unsigned Depth = 0) const override;

```
- **EN**: Implements logic around `useSoftFloat`, `computeKnownBitsForTargetNode`; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `useSoftFloat`, `computeKnownBitsForTargetNode` 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 40-51
```cpp
    MachineBasicBlock *
    EmitInstrWithCustomInserter(MachineInstr &MI,
                                MachineBasicBlock *MBB) const override;

    ConstraintType getConstraintType(StringRef Constraint) const override;
    ConstraintWeight
    getSingleConstraintMatchWeight(AsmOperandInfo &info,
                                   const char *constraint) const override;
    void LowerAsmOperandForConstraint(SDValue Op, StringRef Constraint,
                                      std::vector<SDValue> &Ops,
                                      SelectionDAG &DAG) const override;

```
- **EN**: Implements logic around `EmitInstrWithCustomInserter`, `getConstraintType`, `getSingleConstraintMatchWeight`, `LowerAsmOperandForConstraint`; this block handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `EmitInstrWithCustomInserter`, `getConstraintType`, `getSingleConstraintMatchWeight`, `LowerAsmOperandForConstraint` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 52-60
```cpp
    std::pair<unsigned, const TargetRegisterClass *>
    getRegForInlineAsmConstraint(const TargetRegisterInfo *TRI,
                                 StringRef Constraint, MVT VT) const override;

    bool isOffsetFoldingLegal(const GlobalAddressSDNode *GA) const override;
    MVT getScalarShiftAmountTy(const DataLayout &, EVT) const override {
      return MVT::i32;
    }

```
- **EN**: Implements logic around `getRegForInlineAsmConstraint`, `isOffsetFoldingLegal`, `getScalarShiftAmountTy`; this block returns target-specific results.
- **CN**: 围绕 `getRegForInlineAsmConstraint`, `isOffsetFoldingLegal`, `getScalarShiftAmountTy` 实现具体逻辑；这一段返回目标相关结果。

### Lines 61-70
```cpp
    Register getRegisterByName(const char* RegName, LLT VT,
                               const MachineFunction &MF) const override;

    /// If a physical register, this returns the register that receives the
    /// exception address on entry to an EH pad.
    Register
    getExceptionPointerRegister(const Constant *PersonalityFn) const override {
      return SP::I0;
    }

```
- **EN**: Implements logic around `getRegisterByName`, `getExceptionPointerRegister`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getRegisterByName`, `getExceptionPointerRegister` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 71-77
```cpp
    /// If a physical register, this returns the register that receives the
    /// exception typeid on entry to a landing pad.
    Register
    getExceptionSelectorRegister(const Constant *PersonalityFn) const override {
      return SP::I1;
    }

```
- **EN**: Implements logic around `getExceptionSelectorRegister`; this block returns target-specific results.
- **CN**: 围绕 `getExceptionSelectorRegister` 实现具体逻辑；这一段返回目标相关结果。

### Lines 78-84
```cpp
    /// Override to support customized stack guard loading.
    bool useLoadStackGuardNode(const Module &M) const override;

    /// getSetCCResultType - Return the ISD::SETCC ValueType
    EVT getSetCCResultType(const DataLayout &DL, LLVMContext &Context,
                           EVT VT) const override;

```
- **EN**: Implements logic around `useLoadStackGuardNode`, `getSetCCResultType`; this block returns target-specific results.
- **CN**: 围绕 `useLoadStackGuardNode`, `getSetCCResultType` 实现具体逻辑；这一段返回目标相关结果。

### Lines 85-98
```cpp
    SDValue
    LowerFormalArguments(SDValue Chain, CallingConv::ID CallConv, bool isVarArg,
                         const SmallVectorImpl<ISD::InputArg> &Ins,
                         const SDLoc &dl, SelectionDAG &DAG,
                         SmallVectorImpl<SDValue> &InVals) const override;
    SDValue LowerFormalArguments_32(SDValue Chain, CallingConv::ID CallConv,
                                    bool isVarArg,
                                    const SmallVectorImpl<ISD::InputArg> &Ins,
                                    const SDLoc &dl, SelectionDAG &DAG,
                                    SmallVectorImpl<SDValue> &InVals) const;
    SDValue LowerFormalArguments_64(SDValue Chain, CallingConv::ID CallConv,
                                    bool isVarArg,
                                    const SmallVectorImpl<ISD::InputArg> &Ins,
                                    const SDLoc &dl, SelectionDAG &DAG,
```
- **EN**: Implements logic around `LowerFormalArguments`, `LowerFormalArguments_32`, `LowerFormalArguments_64`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerFormalArguments`, `LowerFormalArguments_32`, `LowerFormalArguments_64` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 99-108
```cpp
                                    SmallVectorImpl<SDValue> &InVals) const;

    SDValue
      LowerCall(TargetLowering::CallLoweringInfo &CLI,
                SmallVectorImpl<SDValue> &InVals) const override;
    SDValue LowerCall_32(TargetLowering::CallLoweringInfo &CLI,
                         SmallVectorImpl<SDValue> &InVals) const;
    SDValue LowerCall_64(TargetLowering::CallLoweringInfo &CLI,
                         SmallVectorImpl<SDValue> &InVals) const;

```
- **EN**: Implements logic around `LowerCall`, `LowerCall_32`, `LowerCall_64`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerCall`, `LowerCall_32`, `LowerCall_64` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 109-122
```cpp
    bool CanLowerReturn(CallingConv::ID CallConv, MachineFunction &MF,
                        bool isVarArg,
                        const SmallVectorImpl<ISD::OutputArg> &Outs,
                        LLVMContext &Context, const Type *RetTy) const override;

    SDValue LowerReturn(SDValue Chain, CallingConv::ID CallConv, bool isVarArg,
                        const SmallVectorImpl<ISD::OutputArg> &Outs,
                        const SmallVectorImpl<SDValue> &OutVals,
                        const SDLoc &dl, SelectionDAG &DAG) const override;
    SDValue LowerReturn_32(SDValue Chain, CallingConv::ID CallConv,
                           bool IsVarArg,
                           const SmallVectorImpl<ISD::OutputArg> &Outs,
                           const SmallVectorImpl<SDValue> &OutVals,
                           const SDLoc &DL, SelectionDAG &DAG) const;
```
- **EN**: Implements logic around `CanLowerReturn`, `LowerReturn`, `LowerReturn_32`; this block handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `CanLowerReturn`, `LowerReturn`, `LowerReturn_32` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 123-133
```cpp
    SDValue LowerReturn_64(SDValue Chain, CallingConv::ID CallConv,
                           bool IsVarArg,
                           const SmallVectorImpl<ISD::OutputArg> &Outs,
                           const SmallVectorImpl<SDValue> &OutVals,
                           const SDLoc &DL, SelectionDAG &DAG) const;

    SDValue LowerGlobalAddress(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerGlobalTLSAddress(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerConstantPool(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerBlockAddress(SDValue Op, SelectionDAG &DAG) const;

```
- **EN**: Implements logic around `LowerReturn_64`, `LowerGlobalAddress`, `LowerGlobalTLSAddress`, `LowerConstantPool`, ...; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerReturn_64`, `LowerGlobalAddress`, `LowerGlobalTLSAddress`, `LowerConstantPool`, ... 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 134-145
```cpp
    SDValue withTargetFlags(SDValue Op, unsigned TF, SelectionDAG &DAG) const;
    SDValue makeHiLoPair(SDValue Op, unsigned HiTF, unsigned LoTF,
                         SelectionDAG &DAG) const;
    SDValue makeAddress(SDValue Op, SelectionDAG &DAG) const;

    SDValue LowerF128_LibCallArg(SDValue Chain, ArgListTy &Args, SDValue Arg,
                                 const SDLoc &DL, SelectionDAG &DAG) const;
    SDValue LowerF128Op(SDValue Op, SelectionDAG &DAG, RTLIB::Libcall LibFunc,
                        unsigned numArgs) const;
    SDValue LowerF128Compare(SDValue LHS, SDValue RHS, unsigned &SPCC,
                             const SDLoc &DL, SelectionDAG &DAG) const;

```
- **EN**: Implements logic around `withTargetFlags`, `makeHiLoPair`, `makeAddress`, `LowerF128_LibCallArg`, ...; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `withTargetFlags`, `makeHiLoPair`, `makeAddress`, `LowerF128_LibCallArg`, ... 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 146-152
```cpp
    SDValue LowerINTRINSIC_WO_CHAIN(SDValue Op, SelectionDAG &DAG) const;

    SDValue PerformBITCASTCombine(SDNode *N, DAGCombinerInfo &DCI) const;

    SDValue bitcastConstantFPToInt(ConstantFPSDNode *C, const SDLoc &DL,
                                   SelectionDAG &DAG) const;

```
- **EN**: Implements logic around `LowerINTRINSIC_WO_CHAIN`, `PerformBITCASTCombine`, `bitcastConstantFPToInt`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerINTRINSIC_WO_CHAIN`, `PerformBITCASTCombine`, `bitcastConstantFPToInt` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 153-164
```cpp
    SDValue PerformDAGCombine(SDNode *N, DAGCombinerInfo &DCI) const override;

    bool IsEligibleForTailCallOptimization(CCState &CCInfo,
                                           CallLoweringInfo &CLI,
                                           MachineFunction &MF) const;

    bool ShouldShrinkFPConstant(EVT VT) const override {
      // Do not shrink FP constpool if VT == MVT::f128.
      // (ldd, call _Q_fdtoq) is more expensive than two ldds.
      return VT != MVT::f128;
    }

```
- **EN**: Implements logic around `PerformDAGCombine`, `IsEligibleForTailCallOptimization`, `ShouldShrinkFPConstant`; this block returns target-specific results; handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `PerformDAGCombine`, `IsEligibleForTailCallOptimization`, `ShouldShrinkFPConstant` 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 165-171
```cpp
    bool isFNegFree(EVT VT) const override;

    bool isFPImmLegal(const APFloat &Imm, EVT VT,
                      bool ForCodeSize) const override;

    bool isCtlzFast() const override;

```
- **EN**: Implements logic around `isFNegFree`, `isFPImmLegal`, `isCtlzFast`.
- **CN**: 围绕 `isFNegFree`, `isFPImmLegal`, `isCtlzFast` 实现具体逻辑。

### Lines 172-179
```cpp
    bool isCheapToSpeculateCtlz(Type *Ty) const override {
      return isCtlzFast();
    }

    bool isCheapToSpeculateCttz(Type *Ty) const override;

    bool enableAggressiveFMAFusion(EVT VT) const override { return true; };

```
- **EN**: Implements logic around `isCheapToSpeculateCtlz`, `isCtlzFast`, `isCheapToSpeculateCttz`, `enableAggressiveFMAFusion`; this block returns target-specific results.
- **CN**: 围绕 `isCheapToSpeculateCtlz`, `isCtlzFast`, `isCheapToSpeculateCttz`, `enableAggressiveFMAFusion` 实现具体逻辑；这一段返回目标相关结果。

### Lines 180-187
```cpp
    bool isFMAFasterThanFMulAndFAdd(const MachineFunction &MF,
                                    EVT VT) const override;

    Instruction *emitLeadingFence(IRBuilderBase &Builder, Instruction *Inst,
                                  AtomicOrdering Ord) const override;
    Instruction *emitTrailingFence(IRBuilderBase &Builder, Instruction *Inst,
                                   AtomicOrdering Ord) const override;

```
- **EN**: Implements logic around `isFMAFasterThanFMulAndFAdd`, `emitLeadingFence`, `emitTrailingFence`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `isFMAFasterThanFMulAndFAdd`, `emitLeadingFence`, `emitTrailingFence` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 188-194
```cpp
    bool shouldInsertFencesForAtomic(const Instruction *I) const override {
      // FIXME: We insert fences for each atomics and generate
      // sub-optimal code for PSO/TSO. (Approximately nobody uses any
      // mode but TSO, which makes this even more silly)
      return true;
    }

```
- **EN**: Implements logic around `shouldInsertFencesForAtomic`; this block returns target-specific results.
- **CN**: 围绕 `shouldInsertFencesForAtomic` 实现具体逻辑；这一段返回目标相关结果。

### Lines 195-201
```cpp
    AtomicExpansionKind
    shouldExpandAtomicRMWInIR(const AtomicRMWInst *AI) const override;

    void ReplaceNodeResults(SDNode *N,
                            SmallVectorImpl<SDValue>& Results,
                            SelectionDAG &DAG) const override;

```
- **EN**: Implements logic around `shouldExpandAtomicRMWInIR`, `ReplaceNodeResults`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `shouldExpandAtomicRMWInIR`, `ReplaceNodeResults` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 202-209
```cpp
    MachineBasicBlock *expandSelectCC(MachineInstr &MI, MachineBasicBlock *BB,
                                      unsigned BROpcode) const;

    void AdjustInstrPostInstrSelection(MachineInstr &MI,
                                       SDNode *Node) const override;
  };
} // end namespace llvm

```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 210-210
```cpp
#endif // LLVM_LIB_TARGET_SPARC_SPARCISELLOWERING_H
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

## Key Concepts / 关键概念

- **SelectionDAG lowering / SelectionDAG 降低**:
  - **EN**: Custom lowering/legalization rules for target operations
  - **CN**: 为目标操作定制 lowering 与合法化规则
- **DAG legalization / DAG 合法化**:
  - **EN**: Explains how generic IR-style operations are rewritten for this target
  - **CN**: 说明如何把通用操作改写为该目标可接受的形式
- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `Sparc.h`, `llvm/CodeGen/TargetLowering.h`
- **LLVM subsystems / LLVM 子系统**: CodeGen
- **Generated macros / 生成宏**: `GET_SPARC_SPARCISELLOWERING_H`
