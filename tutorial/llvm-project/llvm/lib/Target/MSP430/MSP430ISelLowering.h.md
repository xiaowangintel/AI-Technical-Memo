# MSP430ISelLowering.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/MSP430/MSP430ISelLowering.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Customizes SelectionDAG lowering, legalization, calling convention handling, and target-specific DAG nodes.
  - **CN**: 定制 SelectionDAG 降低、合法化、调用约定处理以及目标专用 DAG 节点。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- MSP430ISelLowering.h - MSP430 DAG Lowering Interface ----*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-12
```cpp
//
// This file defines the interfaces that MSP430 uses to lower LLVM code into a
// selection DAG.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 13-20
```cpp

#ifndef LLVM_LIB_TARGET_MSP430_MSP430ISELLOWERING_H
#define LLVM_LIB_TARGET_MSP430_MSP430ISELLOWERING_H

#include "MSP430.h"
#include "llvm/CodeGen/SelectionDAG.h"
#include "llvm/CodeGen/TargetLowering.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `MSP430.h`, `llvm/CodeGen/SelectionDAG.h`, `llvm/CodeGen/TargetLowering.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `MSP430.h`, `llvm/CodeGen/SelectionDAG.h`, `llvm/CodeGen/TargetLowering.h`。

### Lines 21-27
```cpp
namespace llvm {
  class MSP430Subtarget;
  class MSP430TargetLowering : public TargetLowering {
  public:
    explicit MSP430TargetLowering(const TargetMachine &TM,
                                  const MSP430Subtarget &STI);

```
- **EN**: Introduces declarations for `llvm`, `MSP430Subtarget`, `MSP430TargetLowering`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `MSP430Subtarget`, `MSP430TargetLowering` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 28-35
```cpp
    MVT getScalarShiftAmountTy(const DataLayout &, EVT) const override {
      return MVT::i8;
    }

    MVT::SimpleValueType getCmpLibcallReturnType() const override {
      return MVT::i16;
    }

```
- **EN**: Implements logic around `getScalarShiftAmountTy`, `getCmpLibcallReturnType`; this block returns target-specific results.
- **CN**: 围绕 `getScalarShiftAmountTy`, `getCmpLibcallReturnType` 实现具体逻辑；这一段返回目标相关结果。

### Lines 36-45
```cpp
    /// LowerOperation - Provide custom lowering hooks for some operations.
    SDValue LowerOperation(SDValue Op, SelectionDAG &DAG) const override;

    SDValue LowerShifts(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerGlobalAddress(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerBlockAddress(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerExternalSymbol(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerBR_CC(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerSETCC(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerSELECT_CC(SDValue Op, SelectionDAG &DAG) const;
```
- **EN**: Implements logic around `LowerOperation`, `LowerShifts`, `LowerGlobalAddress`, `LowerBlockAddress`, ...; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerOperation`, `LowerShifts`, `LowerGlobalAddress`, `LowerBlockAddress`, ... 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 46-52
```cpp
    SDValue LowerSIGN_EXTEND(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerRETURNADDR(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerFRAMEADDR(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerVASTART(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerJumpTable(SDValue Op, SelectionDAG &DAG) const;
    SDValue getReturnAddressFrameIndex(SelectionDAG &DAG) const;

```
- **EN**: Implements logic around `LowerSIGN_EXTEND`, `LowerRETURNADDR`, `LowerFRAMEADDR`, `LowerVASTART`, ...; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerSIGN_EXTEND`, `LowerRETURNADDR`, `LowerFRAMEADDR`, `LowerVASTART`, ... 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 53-58
```cpp
    TargetLowering::ConstraintType
    getConstraintType(StringRef Constraint) const override;
    std::pair<unsigned, const TargetRegisterClass *>
    getRegForInlineAsmConstraint(const TargetRegisterInfo *TRI,
                                 StringRef Constraint, MVT VT) const override;

```
- **EN**: Implements logic around `getConstraintType`, `getRegForInlineAsmConstraint`.
- **CN**: 围绕 `getConstraintType`, `getRegForInlineAsmConstraint` 实现具体逻辑。

### Lines 59-64
```cpp
    /// isTruncateFree - Return true if it's free to truncate a value of type
    /// Ty1 to type Ty2. e.g. On msp430 it's free to truncate a i16 value in
    /// register R15W to i8 by referencing its sub-register R15B.
    bool isTruncateFree(Type *Ty1, Type *Ty2) const override;
    bool isTruncateFree(EVT VT1, EVT VT2) const override;

```
- **EN**: Implements logic around `isTruncateFree`; this block returns target-specific results.
- **CN**: 围绕 `isTruncateFree` 实现具体逻辑；这一段返回目标相关结果。

### Lines 65-69
```cpp
    /// isZExtFree - Return true if any actual instruction that defines a value
    /// of type Ty1 implicit zero-extends the value to Ty2 in the result
    /// register. This does not necessarily include registers defined in unknown
    /// ways, such as incoming arguments, or copies from unknown virtual
    /// registers. Also, if isTruncateFree(Ty2, Ty1) is true, this does not
```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 70-75
```cpp
    /// necessarily apply to truncate instructions. e.g. on msp430, all
    /// instructions that define 8-bit values implicit zero-extend the result
    /// out to 16 bits.
    bool isZExtFree(Type *Ty1, Type *Ty2) const override;
    bool isZExtFree(EVT VT1, EVT VT2) const override;

```
- **EN**: Implements logic around `isZExtFree`.
- **CN**: 围绕 `isZExtFree` 实现具体逻辑。

### Lines 76-84
```cpp
    bool isLegalICmpImmediate(int64_t) const override;
    bool shouldAvoidTransformToShift(EVT VT, unsigned Amount) const override;

    MachineBasicBlock *
    EmitInstrWithCustomInserter(MachineInstr &MI,
                                MachineBasicBlock *BB) const override;
    MachineBasicBlock *EmitShiftInstr(MachineInstr &MI,
                                      MachineBasicBlock *BB) const;

```
- **EN**: Implements logic around `isLegalICmpImmediate`, `shouldAvoidTransformToShift`, `EmitInstrWithCustomInserter`, `EmitShiftInstr`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `isLegalICmpImmediate`, `shouldAvoidTransformToShift`, `EmitInstrWithCustomInserter`, `EmitShiftInstr` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 85-94
```cpp
  private:
    SDValue LowerCCCCallTo(SDValue Chain, SDValue Callee,
                           CallingConv::ID CallConv, bool isVarArg,
                           bool isTailCall,
                           const SmallVectorImpl<ISD::OutputArg> &Outs,
                           const SmallVectorImpl<SDValue> &OutVals,
                           const SmallVectorImpl<ISD::InputArg> &Ins,
                           const SDLoc &dl, SelectionDAG &DAG,
                           SmallVectorImpl<SDValue> &InVals) const;

```
- **EN**: Implements logic around `LowerCCCCallTo`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerCCCCallTo` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 95-100
```cpp
    SDValue LowerCCCArguments(SDValue Chain, CallingConv::ID CallConv,
                              bool isVarArg,
                              const SmallVectorImpl<ISD::InputArg> &Ins,
                              const SDLoc &dl, SelectionDAG &DAG,
                              SmallVectorImpl<SDValue> &InVals) const;

```
- **EN**: Implements logic around `LowerCCCArguments`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerCCCArguments` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 101-106
```cpp
    SDValue LowerCallResult(SDValue Chain, SDValue InGlue,
                            CallingConv::ID CallConv, bool isVarArg,
                            const SmallVectorImpl<ISD::InputArg> &Ins,
                            const SDLoc &dl, SelectionDAG &DAG,
                            SmallVectorImpl<SDValue> &InVals) const;

```
- **EN**: Implements logic around `LowerCallResult`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerCallResult` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 107-115
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

### Lines 116-121
```cpp
    bool CanLowerReturn(CallingConv::ID CallConv,
                        MachineFunction &MF,
                        bool IsVarArg,
                        const SmallVectorImpl<ISD::OutputArg> &Outs,
                        LLVMContext &Context, const Type *RetTy) const override;

```
- **EN**: Implements logic around `CanLowerReturn`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `CanLowerReturn` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 122-126
```cpp
    SDValue LowerReturn(SDValue Chain, CallingConv::ID CallConv, bool isVarArg,
                        const SmallVectorImpl<ISD::OutputArg> &Outs,
                        const SmallVectorImpl<SDValue> &OutVals,
                        const SDLoc &dl, SelectionDAG &DAG) const override;

```
- **EN**: Implements logic around `LowerReturn`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerReturn` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 127-132
```cpp
    bool getPostIndexedAddressParts(SDNode *N, SDNode *Op, SDValue &Base,
                                    SDValue &Offset, ISD::MemIndexedMode &AM,
                                    SelectionDAG &DAG) const override;
  };
} // namespace llvm

```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 133-133
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

- **Direct includes / 直接包含**: `MSP430.h`, `llvm/CodeGen/SelectionDAG.h`, `llvm/CodeGen/TargetLowering.h`
- **LLVM subsystems / LLVM 子系统**: CodeGen
- **Generated macros / 生成宏**: `GET_MSP430_MSP430ISELLOWERING_H`
