# MipsSEISelLowering.h — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsSEISelLowering.h`
- Repository: `llvm-project`
- Purpose (EN): Declares `MipsSEISelLowering` for the Mips backend and exposes interfaces for SelectionDAG lowering and target-lowering rules.
- 用途 (CN): 声明 Mips 后端中的 `MipsSEISelLowering`，并提供与SelectionDAG 降级与目标降级规则相关的接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===- MipsSEISelLowering.h - MipsSE DAG Lowering Interface -----*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Subclass of MipsTargetLowering specialized for mips32/64.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 13-14
```cpp
#ifndef LLVM_LIB_TARGET_MIPS_MIPSSEISELLOWERING_H
#define LLVM_LIB_TARGET_MIPS_MIPSSEISELLOWERING_H
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 16-18
```cpp
#include "MipsISelLowering.h"
#include "llvm/CodeGen/SelectionDAGNodes.h"
#include "llvm/CodeGenTypes/MachineValueType.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 20-20
```cpp
namespace llvm {
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 22-27
```cpp
class MachineBasicBlock;
class MachineInstr;
class MipsSubtarget;
class MipsTargetMachine;
class SelectionDAG;
class TargetRegisterClass;
```
- EN: Declares `MachineBasicBlock`, packaging target-specific state and APIs around `MipsSEISelLowering`.
- CN: 这里声明 `MachineBasicBlock`，把与 `MipsSEISelLowering` 相关的目标特定状态和 API 组织在一起。

### Lines 29-32
```cpp
  class MipsSETargetLowering : public MipsTargetLowering  {
  public:
    explicit MipsSETargetLowering(const MipsTargetMachine &TM,
                                  const MipsSubtarget &STI);
```
- EN: Declares `MipsSETargetLowering`, packaging target-specific state and APIs around `MipsSEISelLowering`.
- CN: 这里声明 `MipsSETargetLowering`，把与 `MipsSEISelLowering` 相关的目标特定状态和 API 组织在一起。

### Lines 34-36
```cpp
    /// Enable MSA support for the given integer type and Register
    /// class.
    void addMSAIntType(MVT::SimpleValueType Ty, const TargetRegisterClass *RC);
```
- EN: Declares `addMSAIntType`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `addMSAIntType`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 38-41
```cpp
    /// Enable MSA support for the given floating-point type and
    /// Register class.
    void addMSAFloatType(MVT::SimpleValueType Ty,
                         const TargetRegisterClass *RC);
```
- EN: Declares `addMSAFloatType`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `addMSAFloatType`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 43-46
```cpp
    bool allowsMisalignedMemoryAccesses(
        EVT VT, unsigned AS = 0, Align Alignment = Align(1),
        MachineMemOperand::Flags Flags = MachineMemOperand::MONone,
        unsigned *Fast = nullptr) const override;
```
- EN: Declares `allowsMisalignedMemoryAccesses`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `allowsMisalignedMemoryAccesses`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 48-49
```cpp
    TargetLoweringBase::LegalizeTypeAction
    getPreferredVectorAction(MVT VT) const override;
```
- EN: Declares `getPreferredVectorAction`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getPreferredVectorAction`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 51-51
```cpp
    SDValue LowerOperation(SDValue Op, SelectionDAG &DAG) const override;
```
- EN: Declares `LowerOperation`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LowerOperation`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 53-53
```cpp
    SDValue PerformDAGCombine(SDNode *N, DAGCombinerInfo &DCI) const override;
```
- EN: Declares `PerformDAGCombine`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `PerformDAGCombine`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 55-57
```cpp
    MachineBasicBlock *
    EmitInstrWithCustomInserter(MachineInstr &MI,
                                MachineBasicBlock *MBB) const override;
```
- EN: Declares `EmitInstrWithCustomInserter`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `EmitInstrWithCustomInserter`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 59-61
```cpp
    bool isShuffleMaskLegal(ArrayRef<int> Mask, EVT VT) const override {
      return false;
    }
```
- EN: Implements `isShuffleMaskLegal`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isShuffleMaskLegal`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 63-63
```cpp
    const TargetRegisterClass *getRepRegClassFor(MVT VT) const override;
```
- EN: Declares `getRepRegClassFor`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getRepRegClassFor`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 65-68
```cpp
  private:
    bool isEligibleForTailCallOptimization(
        const CCState &CCInfo, unsigned NextStackOffset,
        const MipsFunctionInfo &FI) const override;
```
- EN: Declares `isEligibleForTailCallOptimization`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isEligibleForTailCallOptimization`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 70-75
```cpp
    void
    getOpndList(SmallVectorImpl<SDValue> &Ops,
                std::deque<std::pair<unsigned, SDValue>> &RegsToPass,
                bool IsPICCall, bool GlobalOrExternal, bool InternalLinkage,
                bool IsCallReloc, CallLoweringInfo &CLI, SDValue Callee,
                SDValue Chain) const override;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 77-81
```cpp
    SDValue lowerR5900FPOp(SDValue Op, SelectionDAG &DAG,
                           RTLIB::Libcall LC) const;
    SDValue lowerLOAD(SDValue Op, SelectionDAG &DAG) const;
    SDValue lowerSTORE(SDValue Op, SelectionDAG &DAG) const;
    SDValue lowerBITCAST(SDValue Op, SelectionDAG &DAG) const;
```
- EN: Declares `lowerR5900FPOp`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `lowerR5900FPOp`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 83-84
```cpp
    SDValue lowerMulDiv(SDValue Op, unsigned NewOpc, bool HasLo, bool HasHi,
                        SelectionDAG &DAG) const;
```
- EN: Declares `lowerMulDiv`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `lowerMulDiv`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 86-94
```cpp
    SDValue lowerINTRINSIC_WO_CHAIN(SDValue Op, SelectionDAG &DAG) const;
    SDValue lowerINTRINSIC_W_CHAIN(SDValue Op, SelectionDAG &DAG) const;
    SDValue lowerINTRINSIC_VOID(SDValue Op, SelectionDAG &DAG) const;
    SDValue lowerEXTRACT_VECTOR_ELT(SDValue Op, SelectionDAG &DAG) const;
    SDValue lowerBUILD_VECTOR(SDValue Op, SelectionDAG &DAG) const;
    /// Lower VECTOR_SHUFFLE into one of a number of instructions
    /// depending on the indices in the shuffle.
    SDValue lowerVECTOR_SHUFFLE(SDValue Op, SelectionDAG &DAG) const;
    SDValue lowerSELECT(SDValue Op, SelectionDAG &DAG) const;
```
- EN: Declares `lowerINTRINSIC_WO_CHAIN`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `lowerINTRINSIC_WO_CHAIN`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 96-110
```cpp
    MachineBasicBlock *emitBPOSGE32(MachineInstr &MI,
                                    MachineBasicBlock *BB) const;
    MachineBasicBlock *emitMSACBranchPseudo(MachineInstr &MI,
                                            MachineBasicBlock *BB,
                                            unsigned BranchOp) const;
    /// Emit the COPY_FW pseudo instruction
    MachineBasicBlock *emitCOPY_FW(MachineInstr &MI,
                                   MachineBasicBlock *BB) const;
    /// Emit the COPY_FD pseudo instruction
    MachineBasicBlock *emitCOPY_FD(MachineInstr &MI,
                                   MachineBasicBlock *BB) const;
    /// Emit the INSERT_FW pseudo instruction
    MachineBasicBlock *emitINSERT_FW(MachineInstr &MI,
                                     MachineBasicBlock *BB) const;
    /// Emit the INSERT_FD pseudo instruction
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 111-125
```cpp
    MachineBasicBlock *emitINSERT_FD(MachineInstr &MI,
                                     MachineBasicBlock *BB) const;
    /// Emit the INSERT_([BHWD]|F[WD])_VIDX pseudo instruction
    MachineBasicBlock *emitINSERT_DF_VIDX(MachineInstr &MI,
                                          MachineBasicBlock *BB,
                                          unsigned EltSizeInBytes,
                                          bool IsFP) const;
    /// Emit the FILL_FW pseudo instruction
    MachineBasicBlock *emitFILL_FW(MachineInstr &MI,
                                   MachineBasicBlock *BB) const;
    /// Emit the FILL_FD pseudo instruction
    MachineBasicBlock *emitFILL_FD(MachineInstr &MI,
                                   MachineBasicBlock *BB) const;
    /// Emit the FEXP2_W_1 pseudo instructions.
    MachineBasicBlock *emitFEXP2_W_1(MachineInstr &MI,
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 126-140
```cpp
                                     MachineBasicBlock *BB) const;
    /// Emit the FEXP2_D_1 pseudo instructions.
    MachineBasicBlock *emitFEXP2_D_1(MachineInstr &MI,
                                     MachineBasicBlock *BB) const;
    /// Emit the FILL_FW pseudo instruction
    MachineBasicBlock *emitLD_F16_PSEUDO(MachineInstr &MI,
                                   MachineBasicBlock *BB) const;
    /// Emit the FILL_FD pseudo instruction
    MachineBasicBlock *emitST_F16_PSEUDO(MachineInstr &MI,
                                   MachineBasicBlock *BB) const;
    /// Emit the FEXP2_W_1 pseudo instructions.
    MachineBasicBlock *emitFPEXTEND_PSEUDO(MachineInstr &MI,
                                           MachineBasicBlock *BB,
                                           bool IsFGR64) const;
    /// Emit the FEXP2_D_1 pseudo instructions.
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 141-144
```cpp
    MachineBasicBlock *emitFPROUND_PSEUDO(MachineInstr &MI,
                                          MachineBasicBlock *BBi,
                                          bool IsFGR64) const;
  };
```
- EN: Declares `emitFPROUND_PSEUDO`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitFPROUND_PSEUDO`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 146-146
```cpp
} // end namespace llvm
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 148-148
```cpp
#endif // LLVM_LIB_TARGET_MIPS_MIPSSEISELLOWERING_H
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

## Key Concepts / 关键概念

- EN: Primary role: SelectionDAG lowering and target-lowering rules.
  - CN: 核心职责：SelectionDAG 降级与目标降级规则。
- EN: MachineInstr-level manipulation is central to this file.
  - CN: 该文件以 MachineInstr 层面的操作为核心。
- EN: MachineBasicBlock flow and branch structure matter to the implementation.
  - CN: 实现高度依赖 MachineBasicBlock 流程与分支结构。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。

## Dependencies / 依赖关系

- EN: Backend-local headers: `MipsISelLowering.h`.
  - CN: 后端本地头文件：`MipsISelLowering.h`。
- EN: LLVM infrastructure headers: `llvm/CodeGen/SelectionDAGNodes.h`, `llvm/CodeGenTypes/MachineValueType.h`.
  - CN: LLVM 基础设施头文件：`llvm/CodeGen/SelectionDAGNodes.h`, `llvm/CodeGenTypes/MachineValueType.h`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
