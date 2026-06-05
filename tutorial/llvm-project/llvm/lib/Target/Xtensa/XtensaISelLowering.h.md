# XtensaISelLowering.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Xtensa/XtensaISelLowering.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Customizes SelectionDAG lowering, legalization, calling convention handling, and target-specific DAG nodes.
  - **CN**: 定制 SelectionDAG 降低、合法化、调用约定处理以及目标专用 DAG 节点。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- XtensaISelLowering.h - Xtensa DAG Lowering Interface -----*- C++ -*-===//
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
// This file defines the interfaces that Xtensa uses to lower LLVM code into a
// selection DAG.
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_XTENSA_XTENSAISELLOWERING_H
#define LLVM_LIB_TARGET_XTENSA_XTENSAISELLOWERING_H

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 17-24
```cpp
#include "llvm/CodeGen/CallingConvLower.h"
#include "llvm/CodeGen/SelectionDAG.h"
#include "llvm/CodeGen/TargetLowering.h"

namespace llvm {

class XtensaSubtarget;

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/CodeGen/CallingConvLower.h`, `llvm/CodeGen/SelectionDAG.h`, `llvm/CodeGen/TargetLowering.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/CodeGen/CallingConvLower.h`, `llvm/CodeGen/SelectionDAG.h`, `llvm/CodeGen/TargetLowering.h`。

### Lines 25-33
```cpp
class XtensaTargetLowering : public TargetLowering {
public:
  explicit XtensaTargetLowering(const TargetMachine &TM,
                                const XtensaSubtarget &STI);

  MVT getScalarShiftAmountTy(const DataLayout &, EVT LHSTy) const override {
    return LHSTy.getSizeInBits() <= 32 ? MVT::i32 : MVT::i64;
  }

```
- **EN**: Introduces declarations for `XtensaTargetLowering`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `XtensaTargetLowering` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 34-43
```cpp
  MVT getRegisterTypeForCallingConv(LLVMContext &Context, CallingConv::ID CC,
                                    EVT VT) const override;

  EVT getSetCCResultType(const DataLayout &, LLVMContext &,
                         EVT VT) const override {
    if (!VT.isVector())
      return MVT::i32;
    return VT.changeVectorElementTypeToInteger();
  }

```
- **EN**: Implements logic around `getRegisterTypeForCallingConv`, `getSetCCResultType`, `changeVectorElementTypeToInteger`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getRegisterTypeForCallingConv`, `getSetCCResultType`, `changeVectorElementTypeToInteger` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 44-52
```cpp
  bool isOffsetFoldingLegal(const GlobalAddressSDNode *GA) const override;

  bool isFPImmLegal(const APFloat &Imm, EVT VT,
                    bool ForCodeSize) const override;

  std::pair<unsigned, const TargetRegisterClass *>
  getRegForInlineAsmConstraint(const TargetRegisterInfo *TRI,
                               StringRef Constraint, MVT VT) const override;

```
- **EN**: Implements logic around `isOffsetFoldingLegal`, `isFPImmLegal`, `getRegForInlineAsmConstraint`.
- **CN**: 围绕 `isOffsetFoldingLegal`, `isFPImmLegal`, `getRegForInlineAsmConstraint` 实现具体逻辑。

### Lines 53-59
```cpp
  TargetLowering::ConstraintType
  getConstraintType(StringRef Constraint) const override;

  TargetLowering::ConstraintWeight
  getSingleConstraintMatchWeight(AsmOperandInfo &Info,
                                 const char *Constraint) const override;

```
- **EN**: Implements logic around `getConstraintType`, `getSingleConstraintMatchWeight`.
- **CN**: 围绕 `getConstraintType`, `getSingleConstraintMatchWeight` 实现具体逻辑。

### Lines 60-71
```cpp
  void LowerAsmOperandForConstraint(SDValue Op, StringRef Constraint,
                                    std::vector<SDValue> &Ops,
                                    SelectionDAG &DAG) const override;

  SDValue LowerOperation(SDValue Op, SelectionDAG &DAG) const override;

  SDValue LowerFormalArguments(SDValue Chain, CallingConv::ID CallConv,
                               bool isVarArg,
                               const SmallVectorImpl<ISD::InputArg> &Ins,
                               const SDLoc &DL, SelectionDAG &DAG,
                               SmallVectorImpl<SDValue> &InVals) const override;

```
- **EN**: Implements logic around `LowerAsmOperandForConstraint`, `LowerOperation`, `LowerFormalArguments`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerAsmOperandForConstraint`, `LowerOperation`, `LowerFormalArguments` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 72-79
```cpp
  SDValue LowerCall(CallLoweringInfo &CLI,
                    SmallVectorImpl<SDValue> &InVals) const override;

  bool CanLowerReturn(CallingConv::ID CallConv, MachineFunction &MF,
                      bool isVarArg,
                      const SmallVectorImpl<ISD::OutputArg> &Outs,
                      LLVMContext &Context, const Type *RetTy) const override;

```
- **EN**: Implements logic around `LowerCall`, `CanLowerReturn`; this block handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `LowerCall`, `CanLowerReturn` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 80-88
```cpp
  SDValue LowerReturn(SDValue Chain, CallingConv::ID CallConv, bool IsVarArg,
                      const SmallVectorImpl<ISD::OutputArg> &Outs,
                      const SmallVectorImpl<SDValue> &OutVals, const SDLoc &DL,
                      SelectionDAG &DAG) const override;

  bool shouldInsertFencesForAtomic(const Instruction *I) const override {
    return true;
  }

```
- **EN**: Implements logic around `LowerReturn`, `shouldInsertFencesForAtomic`; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerReturn`, `shouldInsertFencesForAtomic` 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 89-96
```cpp
  AtomicExpansionKind
  shouldExpandAtomicRMWInIR(const AtomicRMWInst *) const override;

  bool decomposeMulByConstant(LLVMContext &Context, EVT VT,
                              SDValue C) const override;

  const XtensaSubtarget &getSubtarget() const { return Subtarget; }

```
- **EN**: Implements logic around `shouldExpandAtomicRMWInIR`, `decomposeMulByConstant`, `getSubtarget`; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `shouldExpandAtomicRMWInIR`, `decomposeMulByConstant`, `getSubtarget` 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 97-103
```cpp
  MachineBasicBlock *
  EmitInstrWithCustomInserter(MachineInstr &MI,
                              MachineBasicBlock *BB) const override;

private:
  const XtensaSubtarget &Subtarget;

```
- **EN**: Implements logic around `EmitInstrWithCustomInserter`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `EmitInstrWithCustomInserter` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 104-111
```cpp
  SDValue LowerBR_JT(SDValue Op, SelectionDAG &DAG) const;

  SDValue LowerImmediate(SDValue Op, SelectionDAG &DAG) const;

  SDValue LowerGlobalAddress(SDValue Op, SelectionDAG &DAG) const;

  SDValue LowerGlobalTLSAddress(SDValue Op, SelectionDAG &DAG) const;

```
- **EN**: Implements logic around `LowerBR_JT`, `LowerImmediate`, `LowerGlobalAddress`, `LowerGlobalTLSAddress`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerBR_JT`, `LowerImmediate`, `LowerGlobalAddress`, `LowerGlobalTLSAddress` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 112-119
```cpp
  SDValue LowerBlockAddress(SDValue Op, SelectionDAG &DAG) const;

  SDValue LowerJumpTable(SDValue Op, SelectionDAG &DAG) const;

  SDValue LowerConstantPool(SDValue Op, SelectionDAG &DAG) const;

  SDValue LowerCTPOP(SDValue Op, SelectionDAG &DAG) const;

```
- **EN**: Implements logic around `LowerBlockAddress`, `LowerJumpTable`, `LowerConstantPool`, `LowerCTPOP`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerBlockAddress`, `LowerJumpTable`, `LowerConstantPool`, `LowerCTPOP` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 120-127
```cpp
  SDValue LowerSELECT_CC(SDValue Op, SelectionDAG &DAG) const;

  SDValue LowerRETURNADDR(SDValue Op, SelectionDAG &DAG) const;

  SDValue LowerDYNAMIC_STACKALLOC(SDValue Op, SelectionDAG &DAG) const;

  SDValue LowerSTACKSAVE(SDValue Op, SelectionDAG &DAG) const;

```
- **EN**: Implements logic around `LowerSELECT_CC`, `LowerRETURNADDR`, `LowerDYNAMIC_STACKALLOC`, `LowerSTACKSAVE`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerSELECT_CC`, `LowerRETURNADDR`, `LowerDYNAMIC_STACKALLOC`, `LowerSTACKSAVE` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 128-135
```cpp
  SDValue LowerSTACKRESTORE(SDValue Op, SelectionDAG &DAG) const;

  SDValue LowerVASTART(SDValue Op, SelectionDAG &DAG) const;

  SDValue LowerVAARG(SDValue Op, SelectionDAG &DAG) const;

  SDValue LowerVACOPY(SDValue Op, SelectionDAG &DAG) const;

```
- **EN**: Implements logic around `LowerSTACKRESTORE`, `LowerVASTART`, `LowerVAARG`, `LowerVACOPY`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerSTACKRESTORE`, `LowerVASTART`, `LowerVAARG`, `LowerVACOPY` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 136-143
```cpp
  SDValue LowerFRAMEADDR(SDValue Op, SelectionDAG &DAG) const;

  SDValue LowerShiftLeftParts(SDValue Op, SelectionDAG &DAG) const;

  SDValue LowerShiftRightParts(SDValue Op, SelectionDAG &DAG, bool IsSRA) const;

  SDValue getAddrPCRel(SDValue Op, SelectionDAG &DAG) const;

```
- **EN**: Implements logic around `LowerFRAMEADDR`, `LowerShiftLeftParts`, `LowerShiftRightParts`, `getAddrPCRel`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerFRAMEADDR`, `LowerShiftLeftParts`, `LowerShiftRightParts`, `getAddrPCRel` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 144-151
```cpp
  CCAssignFn *CCAssignFnForCall(CallingConv::ID CC, bool IsVarArg) const;

  MachineBasicBlock *emitSelectCC(MachineInstr &MI,
                                  MachineBasicBlock *BB) const;
};

} // end namespace llvm

```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 152-152
```cpp
#endif /* LLVM_LIB_TARGET_XTENSA_XTENSAISELLOWERING_H */
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

- **Direct includes / 直接包含**: `llvm/CodeGen/CallingConvLower.h`, `llvm/CodeGen/SelectionDAG.h`, `llvm/CodeGen/TargetLowering.h`
- **LLVM subsystems / LLVM 子系统**: CodeGen
- **Generated macros / 生成宏**: `GET_XTENSA_XTENSAISELLOWERING_H`
