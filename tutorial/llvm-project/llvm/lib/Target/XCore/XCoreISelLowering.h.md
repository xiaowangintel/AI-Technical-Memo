# XCoreISelLowering.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/XCore/XCoreISelLowering.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Customizes SelectionDAG lowering, legalization, calling convention handling, and target-specific DAG nodes.
  - **CN**: 定制 SelectionDAG 降低、合法化、调用约定处理以及目标专用 DAG 节点。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- XCoreISelLowering.h - XCore DAG Lowering Interface ------*- C++ -*-===//
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
// This file defines the interfaces that XCore uses to lower LLVM code into a
// selection DAG.
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_XCORE_XCOREISELLOWERING_H
#define LLVM_LIB_TARGET_XCORE_XCOREISELLOWERING_H

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 17-25
```cpp
#include "XCore.h"
#include "llvm/CodeGen/SelectionDAG.h"
#include "llvm/CodeGen/TargetLowering.h"

namespace llvm {

  // Forward delcarations
  class XCoreSubtarget;

```
- **EN**: Pulls in the headers needed for this implementation, including `XCore.h`, `llvm/CodeGen/SelectionDAG.h`, `llvm/CodeGen/TargetLowering.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `XCore.h`, `llvm/CodeGen/SelectionDAG.h`, `llvm/CodeGen/TargetLowering.h`。

### Lines 26-34
```cpp
  //===--------------------------------------------------------------------===//
  // TargetLowering Implementation
  //===--------------------------------------------------------------------===//
  class XCoreTargetLowering : public TargetLowering
  {
  public:
    explicit XCoreTargetLowering(const TargetMachine &TM,
                                 const XCoreSubtarget &Subtarget);

```
- **EN**: Introduces declarations for `XCoreTargetLowering`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `XCoreTargetLowering` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 35-43
```cpp
    using TargetLowering::isZExtFree;
    bool isZExtFree(SDValue Val, EVT VT2) const override;


    unsigned getJumpTableEncoding() const override;
    MVT getScalarShiftAmountTy(const DataLayout &DL, EVT) const override {
      return MVT::i32;
    }

```
- **EN**: Implements logic around `isZExtFree`, `getJumpTableEncoding`, `getScalarShiftAmountTy`; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `isZExtFree`, `getJumpTableEncoding`, `getScalarShiftAmountTy` 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 44-52
```cpp
    /// LowerOperation - Provide custom lowering hooks for some operations.
    SDValue LowerOperation(SDValue Op, SelectionDAG &DAG) const override;

    /// ReplaceNodeResults - Replace the results of node with an illegal result
    /// type with new values built out of custom code.
    ///
    void ReplaceNodeResults(SDNode *N, SmallVectorImpl<SDValue>&Results,
                            SelectionDAG &DAG) const override;

```
- **EN**: Implements logic around `LowerOperation`, `ReplaceNodeResults`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerOperation`, `ReplaceNodeResults` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 53-60
```cpp
    MachineBasicBlock *
    EmitInstrWithCustomInserter(MachineInstr &MI,
                                MachineBasicBlock *MBB) const override;

    bool isLegalAddressingMode(const DataLayout &DL, const AddrMode &AM,
                               Type *Ty, unsigned AS,
                               Instruction *I = nullptr) const override;

```
- **EN**: Implements logic around `EmitInstrWithCustomInserter`, `isLegalAddressingMode`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `EmitInstrWithCustomInserter`, `isLegalAddressingMode` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 61-67
```cpp
    /// If a physical register, this returns the register that receives the
    /// exception address on entry to an EH pad.
    Register
    getExceptionPointerRegister(const Constant *PersonalityFn) const override {
      return XCore::R0;
    }

```
- **EN**: Implements logic around `getExceptionPointerRegister`; this block returns target-specific results.
- **CN**: 围绕 `getExceptionPointerRegister` 实现具体逻辑；这一段返回目标相关结果。

### Lines 68-74
```cpp
    /// If a physical register, this returns the register that receives the
    /// exception typeid on entry to a landing pad.
    Register
    getExceptionSelectorRegister(const Constant *PersonalityFn) const override {
      return XCore::R1;
    }

```
- **EN**: Implements logic around `getExceptionSelectorRegister`; this block returns target-specific results.
- **CN**: 围绕 `getExceptionSelectorRegister` 实现具体逻辑；这一段返回目标相关结果。

### Lines 75-88
```cpp
  private:
    const TargetMachine &TM;
    const XCoreSubtarget &Subtarget;

    // Lower Operand helpers
    SDValue LowerCCCArguments(SDValue Chain, CallingConv::ID CallConv,
                              bool isVarArg,
                              const SmallVectorImpl<ISD::InputArg> &Ins,
                              const SDLoc &dl, SelectionDAG &DAG,
                              SmallVectorImpl<SDValue> &InVals) const;
    SDValue LowerCCCCallTo(SDValue Chain, SDValue Callee,
                           CallingConv::ID CallConv, bool isVarArg,
                           bool isTailCall,
                           const SmallVectorImpl<ISD::OutputArg> &Outs,
```
- **EN**: Implements logic around `LowerCCCArguments`, `LowerCCCCallTo`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerCCCArguments`, `LowerCCCCallTo` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 89-100
```cpp
                           const SmallVectorImpl<SDValue> &OutVals,
                           const SmallVectorImpl<ISD::InputArg> &Ins,
                           const SDLoc &dl, SelectionDAG &DAG,
                           SmallVectorImpl<SDValue> &InVals) const;
    SDValue getReturnAddressFrameIndex(SelectionDAG &DAG) const;
    SDValue getGlobalAddressWrapper(SDValue GA, const GlobalValue *GV,
                                    SelectionDAG &DAG) const;
    SDValue lowerLoadWordFromAlignedBasePlusOffset(const SDLoc &DL,
                                                   SDValue Chain, SDValue Base,
                                                   int64_t Offset,
                                                   SelectionDAG &DAG) const;

```
- **EN**: Implements logic around `getReturnAddressFrameIndex`, `getGlobalAddressWrapper`, `lowerLoadWordFromAlignedBasePlusOffset`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `getReturnAddressFrameIndex`, `getGlobalAddressWrapper`, `lowerLoadWordFromAlignedBasePlusOffset` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 101-114
```cpp
    // Lower Operand specifics
    SDValue LowerLOAD(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerSTORE(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerEH_RETURN(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerGlobalAddress(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerGlobalTLSAddress(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerBlockAddress(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerConstantPool(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerBR_JT(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerVAARG(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerVASTART(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerUMUL_LOHI(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerSMUL_LOHI(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerFRAMEADDR(SDValue Op, SelectionDAG &DAG) const;
```
- **EN**: Implements logic around `LowerLOAD`, `LowerSTORE`, `LowerEH_RETURN`, `LowerGlobalAddress`, ...; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerLOAD`, `LowerSTORE`, `LowerEH_RETURN`, `LowerGlobalAddress`, ... 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 115-121
```cpp
    SDValue LowerFRAME_TO_ARGS_OFFSET(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerRETURNADDR(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerINIT_TRAMPOLINE(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerADJUST_TRAMPOLINE(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerINTRINSIC_WO_CHAIN(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerATOMIC_FENCE(SDValue Op, SelectionDAG &DAG) const;

```
- **EN**: Implements logic around `LowerFRAME_TO_ARGS_OFFSET`, `LowerRETURNADDR`, `LowerINIT_TRAMPOLINE`, `LowerADJUST_TRAMPOLINE`, ...; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerFRAME_TO_ARGS_OFFSET`, `LowerRETURNADDR`, `LowerINIT_TRAMPOLINE`, `LowerADJUST_TRAMPOLINE`, ... 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 122-130
```cpp
    // Inline asm support
    std::pair<unsigned, const TargetRegisterClass *>
    getRegForInlineAsmConstraint(const TargetRegisterInfo *TRI,
                                 StringRef Constraint, MVT VT) const override;

    // Expand specifics
    SDValue TryExpandADDWithMul(SDNode *Op, SelectionDAG &DAG) const;
    SDValue ExpandADDSUB(SDNode *Op, SelectionDAG &DAG) const;

```
- **EN**: Implements logic around `getRegForInlineAsmConstraint`, `TryExpandADDWithMul`, `ExpandADDSUB`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `getRegForInlineAsmConstraint`, `TryExpandADDWithMul`, `ExpandADDSUB` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 131-138
```cpp
    SDValue PerformDAGCombine(SDNode *N, DAGCombinerInfo &DCI) const override;

    void computeKnownBitsForTargetNode(const SDValue Op,
                                       KnownBits &Known,
                                       const APInt &DemandedElts,
                                       const SelectionDAG &DAG,
                                       unsigned Depth = 0) const override;

```
- **EN**: Implements logic around `PerformDAGCombine`, `computeKnownBitsForTargetNode`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `PerformDAGCombine`, `computeKnownBitsForTargetNode` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 139-148
```cpp
    SDValue
    LowerFormalArguments(SDValue Chain, CallingConv::ID CallConv, bool isVarArg,
                         const SmallVectorImpl<ISD::InputArg> &Ins,
                         const SDLoc &dl, SelectionDAG &DAG,
                         SmallVectorImpl<SDValue> &InVals) const override;

    SDValue
      LowerCall(TargetLowering::CallLoweringInfo &CLI,
                SmallVectorImpl<SDValue> &InVals) const override;

```
- **EN**: Implements logic around `LowerFormalArguments`, `LowerCall`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerFormalArguments`, `LowerCall` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 149-160
```cpp
    SDValue LowerReturn(SDValue Chain, CallingConv::ID CallConv, bool isVarArg,
                        const SmallVectorImpl<ISD::OutputArg> &Outs,
                        const SmallVectorImpl<SDValue> &OutVals,
                        const SDLoc &dl, SelectionDAG &DAG) const override;

    bool CanLowerReturn(CallingConv::ID CallConv, MachineFunction &MF,
                        bool isVarArg,
                        const SmallVectorImpl<ISD::OutputArg> &ArgsFlags,
                        LLVMContext &Context, const Type *RetTy) const override;
  };
}

```
- **EN**: Implements logic around `LowerReturn`, `CanLowerReturn`; this block handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `LowerReturn`, `CanLowerReturn` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 161-161
```cpp
#endif
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

- **Direct includes / 直接包含**: `XCore.h`, `llvm/CodeGen/SelectionDAG.h`, `llvm/CodeGen/TargetLowering.h`
- **LLVM subsystems / LLVM 子系统**: CodeGen
- **Generated macros / 生成宏**: `GET_XCORE_XCOREISELLOWERING_H`
