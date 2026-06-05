# ARMISelLowering.h — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMISelLowering.h`
- Repository: `llvm-project`
- Purpose (EN): This file defines the interfaces that ARM uses to lower LLVM code into a selection DAG.
- 用途 (CN): 声明 ARM 后端中的 `ARMISelLowering`，并提供与SelectionDAG 降级与目标降级规则相关的接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===- ARMISelLowering.h - ARM DAG Lowering Interface -----------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the interfaces that ARM uses to lower LLVM code into a
// selection DAG.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 14-15
```cpp
#ifndef LLVM_LIB_TARGET_ARM_ARMISELLOWERING_H
#define LLVM_LIB_TARGET_ARM_ARMISELLOWERING_H
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 17-31
```cpp
#include "MCTargetDesc/ARMBaseInfo.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/CodeGen/CallingConvLower.h"
#include "llvm/CodeGen/ISDOpcodes.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/SelectionDAGNodes.h"
#include "llvm/CodeGen/TargetLowering.h"
#include "llvm/CodeGen/ValueTypes.h"
#include "llvm/CodeGenTypes/MachineValueType.h"
#include "llvm/IR/Attributes.h"
#include "llvm/IR/CallingConv.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/InlineAsm.h"
#include "llvm/Support/CodeGen.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 32-33
```cpp
#include <optional>
#include <utility>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 35-35
```cpp
namespace llvm {
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 37-51
```cpp
class ARMBaseTargetMachine;
class ARMSubtarget;
class DataLayout;
class FastISel;
class FunctionLoweringInfo;
class GlobalValue;
class InstrItineraryData;
class Instruction;
class IRBuilderBase;
class MachineBasicBlock;
class MachineInstr;
class SelectionDAG;
class TargetLibraryInfo;
class TargetMachine;
class TargetRegisterInfo;
```
- EN: Declares `ARMBaseTargetMachine`, packaging target-specific state and APIs around `ARMISelLowering`.
- CN: 这里声明 `ARMBaseTargetMachine`，把与 `ARMISelLowering` 相关的目标特定状态和 API 组织在一起。

### Lines 52-52
```cpp
class VectorType;
```
- EN: Declares `VectorType`, packaging target-specific state and APIs around `ARMISelLowering`.
- CN: 这里声明 `VectorType`，把与 `ARMISelLowering` 相关的目标特定状态和 API 组织在一起。

### Lines 54-63
```cpp
  namespace ARM {
  /// Possible values of current rounding mode, which is specified in bits
  /// 23:22 of FPSCR.
  enum Rounding {
    RN = 0,    // Round to Nearest
    RP = 1,    // Round towards Plus infinity
    RM = 2,    // Round towards Minus infinity
    RZ = 3,    // Round towards Zero
    rmMask = 3 // Bit mask selecting rounding mode
  };
```
- EN: Defines enumeration `Rounding` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `Rounding`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 65-66
```cpp
  // Bit position of rounding mode bits in FPSCR.
  const unsigned RoundingBitsPos = 22;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 68-70
```cpp
  // Bits of floating-point status. These are NZCV flags, QC bit and cumulative
  // FP exception bits.
  const unsigned FPStatusBits = 0xf800009f;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 72-75
```cpp
  // Some bits in the FPSCR are not yet defined.  They must be preserved when
  // modifying the contents.
  const unsigned FPReservedBits = 0x00006060;
  } // namespace ARM
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 77-78
```cpp
  /// Define some predicates that are used for node matching.
  namespace ARM {
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 80-80
```cpp
    bool isBitFieldInvertedMask(unsigned v);
```
- EN: Declares `isBitFieldInvertedMask`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isBitFieldInvertedMask`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 82-82
```cpp
  } // end namespace ARM
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 84-85
```cpp
  //===--------------------------------------------------------------------===//
  //  ARMTargetLowering - ARM Implementation of the TargetLowering interface
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 87-99
```cpp
  class ARMTargetLowering : public TargetLowering {
    // Copying needed for an outgoing byval argument.
    enum ByValCopyKind {
      // Argument is already in the correct location, no copy needed.
      NoCopy,
      // Argument value is currently in the local stack frame, needs copying to
      // outgoing arguemnt area.
      CopyOnce,
      // Argument value is currently in the outgoing argument area, but not at
      // the correct offset, so needs copying via a temporary in local stack
      // space.
      CopyViaTemp,
    };
```
- EN: Declares `ARMTargetLowering`, packaging target-specific state and APIs around `ARMISelLowering`.
- CN: 这里声明 `ARMTargetLowering`，把与 `ARMISelLowering` 相关的目标特定状态和 API 组织在一起。

### Lines 101-103
```cpp
  public:
    explicit ARMTargetLowering(const TargetMachine &TM,
                               const ARMSubtarget &STI);
```
- EN: Declares `ARMTargetLowering`, a lowering routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ARMTargetLowering`，它是一个围绕目标机器策略展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 105-105
```cpp
    const ARMBaseTargetMachine &getTM() const;
```
- EN: Declares `getTM`, a query/helper routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTM`，它是一个围绕目标机器策略展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 107-108
```cpp
    unsigned getJumpTableEncoding() const override;
    bool useSoftFloat() const override;
```
- EN: Declares `getJumpTableEncoding`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getJumpTableEncoding`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 110-110
```cpp
    SDValue LowerOperation(SDValue Op, SelectionDAG &DAG) const override;
```
- EN: Declares `LowerOperation`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LowerOperation`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 112-115
```cpp
    /// ReplaceNodeResults - Replace the results of node with an illegal result
    /// type with new values built out of custom code.
    void ReplaceNodeResults(SDNode *N, SmallVectorImpl<SDValue>&Results,
                            SelectionDAG &DAG) const override;
```
- EN: Declares `ReplaceNodeResults`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ReplaceNodeResults`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 117-120
```cpp
    bool isSelectSupported(SelectSupportKind Kind) const override {
      // ARM does not support scalar condition selects on vectors.
      return (Kind != ScalarCondVectorVal);
    }
```
- EN: Implements `isSelectSupported`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isSelectSupported`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 122-122
```cpp
    bool isReadOnly(const GlobalValue *GV) const;
```
- EN: Declares `isReadOnly`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isReadOnly`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 124-126
```cpp
    /// getSetCCResultType - Return the value type to use for ISD::SETCC.
    EVT getSetCCResultType(const DataLayout &DL, LLVMContext &Context,
                           EVT VT) const override;
```
- EN: Declares `getSetCCResultType`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getSetCCResultType`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 128-130
```cpp
    MachineBasicBlock *
    EmitInstrWithCustomInserter(MachineInstr &MI,
                                MachineBasicBlock *MBB) const override;
```
- EN: Declares `EmitInstrWithCustomInserter`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `EmitInstrWithCustomInserter`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 132-133
```cpp
    void AdjustInstrPostInstrSelection(MachineInstr &MI,
                                       SDNode *Node) const override;
```
- EN: Declares `AdjustInstrPostInstrSelection`, a mutation/build routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `AdjustInstrPostInstrSelection`，它是一个围绕机器指令展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 135-135
```cpp
    bool supportKCFIBundles() const override;
```
- EN: Declares `supportKCFIBundles`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `supportKCFIBundles`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 137-139
```cpp
    MachineInstr *EmitKCFICheck(MachineBasicBlock &MBB,
                                MachineBasicBlock::instr_iterator &MBBI,
                                const TargetInstrInfo *TII) const override;
```
- EN: Declares `EmitKCFICheck`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `EmitKCFICheck`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 141-147
```cpp
    SDValue PerformCMOVCombine(SDNode *N, SelectionDAG &DAG) const;
    SDValue PerformBRCONDCombine(SDNode *N, SelectionDAG &DAG) const;
    SDValue PerformCMOVToBFICombine(SDNode *N, SelectionDAG &DAG) const;
    SDValue PerformIntrinsicCombine(SDNode *N, DAGCombinerInfo &DCI) const;
    SDValue PerformMVEExtCombine(SDNode *N, DAGCombinerInfo &DCI) const;
    SDValue PerformMVETruncCombine(SDNode *N, DAGCombinerInfo &DCI) const;
    SDValue PerformDAGCombine(SDNode *N, DAGCombinerInfo &DCI) const override;
```
- EN: Declares `PerformCMOVCombine`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `PerformCMOVCombine`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 149-154
```cpp
    bool SimplifyDemandedBitsForTargetNode(SDValue Op,
                                           const APInt &OriginalDemandedBits,
                                           const APInt &OriginalDemandedElts,
                                           KnownBits &Known,
                                           TargetLoweringOpt &TLO,
                                           unsigned Depth) const override;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 156-156
```cpp
    bool isDesirableToTransformToIntegerOp(unsigned Opc, EVT VT) const override;
```
- EN: Declares `isDesirableToTransformToIntegerOp`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isDesirableToTransformToIntegerOp`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 158-164
```cpp
    /// allowsMisalignedMemoryAccesses - Returns true if the target allows
    /// unaligned memory accesses of the specified type. Returns whether it
    /// is "fast" by reference in the second argument.
    bool allowsMisalignedMemoryAccesses(EVT VT, unsigned AddrSpace,
                                        Align Alignment,
                                        MachineMemOperand::Flags Flags,
                                        unsigned *Fast) const override;
```
- EN: Declares `allowsMisalignedMemoryAccesses`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `allowsMisalignedMemoryAccesses`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 166-167
```cpp
    EVT getOptimalMemOpType(LLVMContext &Context, const MemOp &Op,
                            const AttributeList &FuncAttributes) const override;
```
- EN: Declares `getOptimalMemOpType`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getOptimalMemOpType`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 169-172
```cpp
    bool isTruncateFree(Type *SrcTy, Type *DstTy) const override;
    bool isTruncateFree(EVT SrcVT, EVT DstVT) const override;
    bool isZExtFree(SDValue Val, EVT VT2) const override;
    Type* shouldConvertSplatType(ShuffleVectorInst* SVI) const override;
```
- EN: Declares `isTruncateFree`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isTruncateFree`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 174-174
```cpp
    bool isFNegFree(EVT VT) const override;
```
- EN: Declares `isFNegFree`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isFNegFree`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 176-176
```cpp
    bool isVectorLoadExtDesirable(SDValue ExtVal) const override;
```
- EN: Declares `isVectorLoadExtDesirable`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isVectorLoadExtDesirable`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 178-178
```cpp
    bool allowTruncateForTailCall(Type *Ty1, Type *Ty2) const override;
```
- EN: Declares `allowTruncateForTailCall`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `allowTruncateForTailCall`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 181-185
```cpp
    /// isLegalAddressingMode - Return true if the addressing mode represented
    /// by AM is legal for this target, for a load/store of the specified type.
    bool isLegalAddressingMode(const DataLayout &DL, const AddrMode &AM,
                               Type *Ty, unsigned AS,
                               Instruction *I = nullptr) const override;
```
- EN: Declares `isLegalAddressingMode`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isLegalAddressingMode`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 187-187
```cpp
    bool isLegalT2ScaledAddressingMode(const AddrMode &AM, EVT VT) const;
```
- EN: Declares `isLegalT2ScaledAddressingMode`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isLegalT2ScaledAddressingMode`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 189-191
```cpp
    /// Returns true if the addressing mode representing by AM is legal
    /// for the Thumb1 target, for a load/store of the specified type.
    bool isLegalT1ScaledAddressingMode(const AddrMode &AM, EVT VT) const;
```
- EN: Declares `isLegalT1ScaledAddressingMode`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isLegalT1ScaledAddressingMode`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 193-197
```cpp
    /// isLegalICmpImmediate - Return true if the specified immediate is legal
    /// icmp immediate, that is the target has icmp instructions which can
    /// compare a register against the immediate without having to materialize
    /// the immediate into a register.
    bool isLegalICmpImmediate(int64_t Imm) const override;
```
- EN: Declares `isLegalICmpImmediate`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isLegalICmpImmediate`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 199-203
```cpp
    /// isLegalAddImmediate - Return true if the specified immediate is legal
    /// add immediate, that is the target has add instructions which can
    /// add a register and the immediate without having to materialize
    /// the immediate into a register.
    bool isLegalAddImmediate(int64_t Imm) const override;
```
- EN: Declares `isLegalAddImmediate`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isLegalAddImmediate`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 205-210
```cpp
    /// getPreIndexedAddressParts - returns true by value, base pointer and
    /// offset pointer and addressing mode by reference if the node's address
    /// can be legally represented as pre-indexed load / store address.
    bool getPreIndexedAddressParts(SDNode *N, SDValue &Base, SDValue &Offset,
                                   ISD::MemIndexedMode &AM,
                                   SelectionDAG &DAG) const override;
```
- EN: Declares `getPreIndexedAddressParts`, a query/helper routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getPreIndexedAddressParts`，它是一个围绕SelectionDAG 降级展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 212-217
```cpp
    /// getPostIndexedAddressParts - returns true by value, base pointer and
    /// offset pointer and addressing mode by reference if this node can be
    /// combined with a load / store to form a post-indexed load / store.
    bool getPostIndexedAddressParts(SDNode *N, SDNode *Op, SDValue &Base,
                                    SDValue &Offset, ISD::MemIndexedMode &AM,
                                    SelectionDAG &DAG) const override;
```
- EN: Declares `getPostIndexedAddressParts`, a query/helper routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getPostIndexedAddressParts`，它是一个围绕SelectionDAG 降级展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 219-222
```cpp
    void computeKnownBitsForTargetNode(const SDValue Op, KnownBits &Known,
                                       const APInt &DemandedElts,
                                       const SelectionDAG &DAG,
                                       unsigned Depth) const override;
```
- EN: Declares `computeKnownBitsForTargetNode`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `computeKnownBitsForTargetNode`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 224-226
```cpp
    bool targetShrinkDemandedConstant(SDValue Op, const APInt &DemandedBits,
                                      const APInt &DemandedElts,
                                      TargetLoweringOpt &TLO) const override;
```
- EN: Declares `targetShrinkDemandedConstant`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `targetShrinkDemandedConstant`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 228-228
```cpp
    ConstraintType getConstraintType(StringRef Constraint) const override;
```
- EN: Declares `getConstraintType`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getConstraintType`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 230-233
```cpp
    /// Examine constraint string and operand type and determine a weight value.
    /// The operand object must already have been set up with the operand type.
    ConstraintWeight getSingleConstraintMatchWeight(
      AsmOperandInfo &info, const char *constraint) const override;
```
- EN: Declares `getSingleConstraintMatchWeight`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getSingleConstraintMatchWeight`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 235-237
```cpp
    std::pair<unsigned, const TargetRegisterClass *>
    getRegForInlineAsmConstraint(const TargetRegisterInfo *TRI,
                                 StringRef Constraint, MVT VT) const override;
```
- EN: Declares `getRegForInlineAsmConstraint`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getRegForInlineAsmConstraint`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 239-239
```cpp
    const char *LowerXConstraint(EVT ConstraintVT) const override;
```
- EN: Declares `LowerXConstraint`, a lowering routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LowerXConstraint`，它是一个围绕目标相关状态展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 241-247
```cpp
    /// LowerAsmOperandForConstraint - Lower the specified operand into the Ops
    /// vector.  If it is invalid, don't add anything to Ops. If hasMemory is
    /// true it means one of the asm constraint of the inline asm instruction
    /// being processed is 'm'.
    void LowerAsmOperandForConstraint(SDValue Op, StringRef Constraint,
                                      std::vector<SDValue> &Ops,
                                      SelectionDAG &DAG) const override;
```
- EN: Declares `LowerAsmOperandForConstraint`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LowerAsmOperandForConstraint`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 249-263
```cpp
    InlineAsm::ConstraintCode
    getInlineAsmMemConstraint(StringRef ConstraintCode) const override {
      if (ConstraintCode == "Q")
        return InlineAsm::ConstraintCode::Q;
      if (ConstraintCode.size() == 2) {
        if (ConstraintCode[0] == 'U') {
          switch(ConstraintCode[1]) {
          default:
            break;
          case 'm':
            return InlineAsm::ConstraintCode::Um;
          case 'n':
            return InlineAsm::ConstraintCode::Un;
          case 'q':
            return InlineAsm::ConstraintCode::Uq;
```
- EN: Implements `getInlineAsmMemConstraint`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getInlineAsmMemConstraint`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 264-276
```cpp
          case 's':
            return InlineAsm::ConstraintCode::Us;
          case 't':
            return InlineAsm::ConstraintCode::Ut;
          case 'v':
            return InlineAsm::ConstraintCode::Uv;
          case 'y':
            return InlineAsm::ConstraintCode::Uy;
          }
        }
      }
      return TargetLowering::getInlineAsmMemConstraint(ConstraintCode);
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 278-280
```cpp
    const ARMSubtarget* getSubtarget() const {
      return Subtarget;
    }
```
- EN: Implements `getSubtarget`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getSubtarget`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 282-285
```cpp
    /// getRegClassFor - Return the register class that should be used for the
    /// specified value type.
    const TargetRegisterClass *
    getRegClassFor(MVT VT, bool isDivergent = false) const override;
```
- EN: Declares `that`, packaging target-specific state and APIs around `ARMISelLowering`.
- CN: 这里声明 `that`，把与 `ARMISelLowering` 相关的目标特定状态和 API 组织在一起。

### Lines 287-288
```cpp
    bool shouldAlignPointerArgs(CallInst *CI, unsigned &MinSize,
                                Align &PrefAlign) const override;
```
- EN: Declares `shouldAlignPointerArgs`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `shouldAlignPointerArgs`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 290-295
```cpp
    /// createFastISel - This method returns a target specific FastISel object,
    /// or null if the target does not support "fast" ISel.
    FastISel *
    createFastISel(FunctionLoweringInfo &funcInfo,
                   const TargetLibraryInfo *libInfo,
                   const LibcallLoweringInfo *libcallLowering) const override;
```
- EN: Declares `createFastISel`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `createFastISel`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 297-297
```cpp
    Sched::Preference getSchedulingPreference(SDNode *N) const override;
```
- EN: Declares `getSchedulingPreference`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getSchedulingPreference`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 299-299
```cpp
    bool preferZeroCompareBranch() const override { return true; }
```
- EN: Implements `preferZeroCompareBranch`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `preferZeroCompareBranch`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 301-301
```cpp
    bool preferSelectsOverBooleanArithmetic(EVT VT) const override;
```
- EN: Declares `preferSelectsOverBooleanArithmetic`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `preferSelectsOverBooleanArithmetic`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 303-303
```cpp
    bool isMaskAndCmp0FoldingBeneficial(const Instruction &AndI) const override;
```
- EN: Declares `isMaskAndCmp0FoldingBeneficial`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isMaskAndCmp0FoldingBeneficial`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 305-308
```cpp
    bool hasAndNotCompare(SDValue V) const override {
      // We can use bics for any scalar.
      return V.getValueType().isScalarInteger();
    }
```
- EN: Implements `hasAndNotCompare`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasAndNotCompare`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 310-310
```cpp
    bool hasAndNot(SDValue Y) const override;
```
- EN: Declares `hasAndNot`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `hasAndNot`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 312-314
```cpp
    bool
    isShuffleMaskLegal(ArrayRef<int> M, EVT VT) const override;
    bool isOffsetFoldingLegal(const GlobalAddressSDNode *GA) const override;
```
- EN: Declares `isShuffleMaskLegal`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isShuffleMaskLegal`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 316-320
```cpp
    /// isFPImmLegal - Returns true if the target can instruction select the
    /// specified FP immediate natively. If false, the legalizer will
    /// materialize the FP immediate as a load from a constant pool.
    bool isFPImmLegal(const APFloat &Imm, EVT VT,
                      bool ForCodeSize = false) const override;
```
- EN: Declares `isFPImmLegal`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isFPImmLegal`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 322-324
```cpp
    void getTgtMemIntrinsic(SmallVectorImpl<IntrinsicInfo> &Infos,
                            const CallBase &I, MachineFunction &MF,
                            unsigned Intrinsic) const override;
```
- EN: Declares `getTgtMemIntrinsic`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTgtMemIntrinsic`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 326-329
```cpp
    /// Returns true if it is beneficial to convert a load of a constant
    /// to just the constant itself.
    bool shouldConvertConstantLoadToIntImm(const APInt &Imm,
                                           Type *Ty) const override;
```
- EN: Declares `shouldConvertConstantLoadToIntImm`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `shouldConvertConstantLoadToIntImm`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 331-334
```cpp
    /// Return true if EXTRACT_SUBVECTOR is cheap for this result type
    /// with this index.
    bool isExtractSubvectorCheap(EVT ResVT, EVT SrcVT,
                                 unsigned Index) const override;
```
- EN: Declares `isExtractSubvectorCheap`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isExtractSubvectorCheap`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 336-340
```cpp
    bool shouldFormOverflowOp(unsigned Opcode, EVT VT,
                              bool MathUsed) const override {
      // Using overflow ops for overflow checks only should beneficial on ARM.
      return TargetLowering::shouldFormOverflowOp(Opcode, VT, true);
    }
```
- EN: Implements `TargetLowering::shouldFormOverflowOp`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `TargetLowering::shouldFormOverflowOp`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 342-344
```cpp
    bool shouldReassociateReduction(unsigned Opc, EVT VT) const override {
      return Opc != ISD::VECREDUCE_ADD;
    }
```
- EN: Implements `shouldReassociateReduction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `shouldReassociateReduction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 346-350
```cpp
    /// Returns true if an argument of type Ty needs to be passed in a
    /// contiguous block of registers in calling convention CallConv.
    bool functionArgumentNeedsConsecutiveRegisters(
        Type *Ty, CallingConv::ID CallConv, bool isVarArg,
        const DataLayout &DL) const override;
```
- EN: Declares `functionArgumentNeedsConsecutiveRegisters`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `functionArgumentNeedsConsecutiveRegisters`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 352-355
```cpp
    /// If a physical register, this returns the register that receives the
    /// exception address on entry to an EH pad.
    Register
    getExceptionPointerRegister(const Constant *PersonalityFn) const override;
```
- EN: Declares `getExceptionPointerRegister`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getExceptionPointerRegister`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 357-360
```cpp
    /// If a physical register, this returns the register that receives the
    /// exception typeid on entry to a landing pad.
    Register
    getExceptionSelectorRegister(const Constant *PersonalityFn) const override;
```
- EN: Declares `getExceptionSelectorRegister`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getExceptionSelectorRegister`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 362-366
```cpp
    Instruction *makeDMB(IRBuilderBase &Builder, ARM_MB::MemBOpt Domain) const;
    Value *emitLoadLinked(IRBuilderBase &Builder, Type *ValueTy, Value *Addr,
                          AtomicOrdering Ord) const override;
    Value *emitStoreConditional(IRBuilderBase &Builder, Value *Val, Value *Addr,
                                AtomicOrdering Ord) const override;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 368-369
```cpp
    void
    emitAtomicCmpXchgNoStoreLLBalance(IRBuilderBase &Builder) const override;
```
- EN: Declares `emitAtomicCmpXchgNoStoreLLBalance`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitAtomicCmpXchgNoStoreLLBalance`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 371-374
```cpp
    Instruction *emitLeadingFence(IRBuilderBase &Builder, Instruction *Inst,
                                  AtomicOrdering Ord) const override;
    Instruction *emitTrailingFence(IRBuilderBase &Builder, Instruction *Inst,
                                   AtomicOrdering Ord) const override;
```
- EN: Declares `emitLeadingFence`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitLeadingFence`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 376-376
```cpp
    unsigned getMaxSupportedInterleaveFactor() const override;
```
- EN: Declares `getMaxSupportedInterleaveFactor`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getMaxSupportedInterleaveFactor`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 378-384
```cpp
    bool lowerInterleavedLoad(Instruction *Load, Value *Mask,
                              ArrayRef<ShuffleVectorInst *> Shuffles,
                              ArrayRef<unsigned> Indices, unsigned Factor,
                              const APInt &GapMask) const override;
    bool lowerInterleavedStore(Instruction *Store, Value *Mask,
                               ShuffleVectorInst *SVI, unsigned Factor,
                               const APInt &GapMask) const override;
```
- EN: Declares `lowerInterleavedLoad`, a lowering routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `lowerInterleavedLoad`，它是一个围绕目标相关状态展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 386-394
```cpp
    bool shouldInsertFencesForAtomic(const Instruction *I) const override;
    TargetLoweringBase::AtomicExpansionKind
    shouldExpandAtomicLoadInIR(LoadInst *LI) const override;
    TargetLoweringBase::AtomicExpansionKind
    shouldExpandAtomicStoreInIR(StoreInst *SI) const override;
    TargetLoweringBase::AtomicExpansionKind
    shouldExpandAtomicRMWInIR(const AtomicRMWInst *AI) const override;
    TargetLoweringBase::AtomicExpansionKind
    shouldExpandAtomicCmpXchgInIR(const AtomicCmpXchgInst *AI) const override;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 396-396
```cpp
    bool useLoadStackGuardNode(const Module &M) const override;
```
- EN: Declares `useLoadStackGuardNode`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `useLoadStackGuardNode`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 398-400
```cpp
    void
    insertSSPDeclarations(Module &M,
                          const LibcallLoweringInfo &Libcalls) const override;
```
- EN: Declares `insertSSPDeclarations`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `insertSSPDeclarations`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 402-403
```cpp
    bool canCombineStoreAndExtract(Type *VectorTy, Value *Idx,
                                   unsigned &Cost) const override;
```
- EN: Declares `canCombineStoreAndExtract`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `canCombineStoreAndExtract`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 405-410
```cpp
    bool canCreateUndefOrPoisonForTargetNode(SDValue Op,
                                             const APInt &DemandedElts,
                                             const SelectionDAG &DAG,
                                             UndefPoisonKind Kind,
                                             bool ConsiderFlags,
                                             unsigned Depth) const override;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 412-416
```cpp
    bool canMergeStoresTo(unsigned AddressSpace, EVT MemVT,
                          const MachineFunction &MF) const override {
      // Do not merge to larger than i32.
      return (MemVT.getSizeInBits() <= 32);
    }
```
- EN: Implements `canMergeStoresTo`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `canMergeStoresTo`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 418-419
```cpp
    bool isCheapToSpeculateCttz(Type *Ty) const override;
    bool isCheapToSpeculateCtlz(Type *Ty) const override;
```
- EN: Declares `isCheapToSpeculateCttz`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isCheapToSpeculateCttz`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 421-423
```cpp
    bool convertSetCCLogicToBitwiseLogic(EVT VT) const override {
      return VT.isScalarInteger();
    }
```
- EN: Implements `convertSetCCLogicToBitwiseLogic`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `convertSetCCLogicToBitwiseLogic`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 425-427
```cpp
    bool supportSwiftError() const override {
      return true;
    }
```
- EN: Implements `supportSwiftError`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `supportSwiftError`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 429-432
```cpp
    bool supportSplitCSR(MachineFunction *MF) const override {
      return MF->getFunction().getCallingConv() == CallingConv::CXX_FAST_TLS &&
             MF->getFunction().hasFnAttribute(Attribute::NoUnwind);
    }
```
- EN: Implements `supportSplitCSR`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `supportSplitCSR`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 434-436
```cpp
    bool hasStandaloneRem(EVT VT) const override {
      return HasStandaloneRem;
    }
```
- EN: Implements `hasStandaloneRem`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasStandaloneRem`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 438-440
```cpp
    ShiftLegalizationStrategy
    preferredShiftLegalizationStrategy(SelectionDAG &DAG, SDNode *N,
                                       unsigned ExpansionFactor) const override;
```
- EN: Declares `preferredShiftLegalizationStrategy`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `preferredShiftLegalizationStrategy`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 442-443
```cpp
    CCAssignFn *CCAssignFnForCall(CallingConv::ID CC, bool isVarArg) const;
    CCAssignFn *CCAssignFnForReturn(CallingConv::ID CC, bool isVarArg) const;
```
- EN: Declares `CCAssignFnForCall`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `CCAssignFnForCall`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 445-450
```cpp
    /// Returns true if \p VecTy is a legal interleaved access type. This
    /// function checks the vector element type and the overall width of the
    /// vector.
    bool isLegalInterleavedAccessType(unsigned Factor, FixedVectorType *VecTy,
                                      Align Alignment,
                                      const DataLayout &DL) const;
```
- EN: Declares `isLegalInterleavedAccessType`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isLegalInterleavedAccessType`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 452-453
```cpp
    bool isMulAddWithConstProfitable(SDValue AddNode,
                                     SDValue ConstNode) const override;
```
- EN: Declares `isMulAddWithConstProfitable`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isMulAddWithConstProfitable`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 455-455
```cpp
    bool alignLoopsWithOptSize() const override;
```
- EN: Declares `alignLoopsWithOptSize`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `alignLoopsWithOptSize`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 457-460
```cpp
    /// Returns the number of interleaved accesses that will be generated when
    /// lowering accesses of the given type.
    unsigned getNumInterleavedAccesses(VectorType *VecTy,
                                       const DataLayout &DL) const;
```
- EN: Declares `getNumInterleavedAccesses`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getNumInterleavedAccesses`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 462-462
```cpp
    void finalizeLowering(MachineFunction &MF) const override;
```
- EN: Declares `finalizeLowering`, a lowering routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `finalizeLowering`，它是一个围绕机器函数状态展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 464-466
```cpp
    /// Return the correct alignment for the current calling convention.
    Align getABIAlignmentForCallingConv(Type *ArgTy,
                                        const DataLayout &DL) const override;
```
- EN: Declares `getABIAlignmentForCallingConv`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getABIAlignmentForCallingConv`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 468-469
```cpp
    bool isDesirableToCommuteWithShift(const SDNode *N,
                                       CombineLevel Level) const override;
```
- EN: Declares `isDesirableToCommuteWithShift`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isDesirableToCommuteWithShift`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 471-471
```cpp
    bool isDesirableToCommuteXorWithShift(const SDNode *N) const override;
```
- EN: Declares `isDesirableToCommuteXorWithShift`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isDesirableToCommuteXorWithShift`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 473-473
```cpp
    bool shouldFoldConstantShiftPairToMask(const SDNode *N) const override;
```
- EN: Declares `shouldFoldConstantShiftPairToMask`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `shouldFoldConstantShiftPairToMask`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 475-477
```cpp
    /// Return true if it is profitable to fold a pair of shifts into a mask.
    bool shouldFoldMaskToVariableShiftPair(SDValue Y) const override {
      EVT VT = Y.getValueType();
```
- EN: Implements `shouldFoldMaskToVariableShiftPair`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `shouldFoldMaskToVariableShiftPair`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 479-480
```cpp
      if (VT.isVector())
        return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 482-483
```cpp
      return VT.getScalarSizeInBits() <= 32;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 485-487
```cpp
    bool shouldFoldSelectWithIdentityConstant(unsigned BinOpcode, EVT VT,
                                              unsigned SelectOpcode, SDValue X,
                                              SDValue Y) const override;
```
- EN: Declares `shouldFoldSelectWithIdentityConstant`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `shouldFoldSelectWithIdentityConstant`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 489-489
```cpp
    bool preferIncOfAddToSubOfNot(EVT VT) const override;
```
- EN: Declares `preferIncOfAddToSubOfNot`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `preferIncOfAddToSubOfNot`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 491-491
```cpp
    bool shouldConvertFpToSat(unsigned Op, EVT FPVT, EVT VT) const override;
```
- EN: Declares `shouldConvertFpToSat`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `shouldConvertFpToSat`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 493-495
```cpp
    bool isComplexDeinterleavingSupported() const override;
    bool isComplexDeinterleavingOperationSupported(
        ComplexDeinterleavingOperation Operation, Type *Ty) const override;
```
- EN: Declares `isComplexDeinterleavingSupported`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isComplexDeinterleavingSupported`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 497-500
```cpp
    Value *createComplexDeinterleavingIR(
        IRBuilderBase &B, ComplexDeinterleavingOperation OperationType,
        ComplexDeinterleavingRotation Rotation, Value *InputA, Value *InputB,
        Value *Accumulator = nullptr) const override;
```
- EN: Declares `createComplexDeinterleavingIR`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `createComplexDeinterleavingIR`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 502-502
```cpp
    bool useFPRegsForHalfType() const override { return true; }
```
- EN: Implements `useFPRegsForHalfType`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `useFPRegsForHalfType`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 504-507
```cpp
  protected:
    std::pair<const TargetRegisterClass *, uint8_t>
    findRepresentativeClass(const TargetRegisterInfo *TRI,
                            MVT VT) const override;
```
- EN: Declares `findRepresentativeClass`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `findRepresentativeClass`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 509-512
```cpp
  private:
    /// Subtarget - Keep a pointer to the ARMSubtarget around so that we can
    /// make the right decision when generating code for different targets.
    const ARMSubtarget *Subtarget;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 514-514
```cpp
    const TargetRegisterInfo *RegInfo;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 516-516
```cpp
    const InstrItineraryData *Itins;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 518-520
```cpp
    // TODO: remove this, and have shouldInsertFencesForAtomic do the proper
    // check.
    bool InsertFencesForAtomic;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 522-522
```cpp
    bool HasStandaloneRem = true;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 524-527
```cpp
    void addTypeForNEON(MVT VT, MVT PromotedLdStVT);
    void addDRTypeForNEON(MVT VT);
    void addQRTypeForNEON(MVT VT);
    std::pair<SDValue, SDValue> getARMXALUOOp(SDValue Op, SelectionDAG &DAG, SDValue &ARMcc) const;
```
- EN: Declares `addTypeForNEON`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `addTypeForNEON`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 529-529
```cpp
    using RegsToPassVector = SmallVector<std::pair<unsigned, SDValue>, 8>;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 531-540
```cpp
    void PassF64ArgInRegs(const SDLoc &dl, SelectionDAG &DAG, SDValue Chain,
                          SDValue &Arg, RegsToPassVector &RegsToPass,
                          CCValAssign &VA, CCValAssign &NextVA,
                          SDValue &StackPtr,
                          SmallVectorImpl<SDValue> &MemOpChains,
                          bool IsTailCall,
                          int SPDiff) const;
    SDValue GetF64FormalArgument(CCValAssign &VA, CCValAssign &NextVA,
                                 SDValue &Root, SelectionDAG &DAG,
                                 const SDLoc &dl) const;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 542-556
```cpp
    CallingConv::ID getEffectiveCallingConv(CallingConv::ID CC,
                                            bool isVarArg) const;
    CCAssignFn *CCAssignFnForNode(CallingConv::ID CC, bool Return,
                                  bool isVarArg) const;
    std::pair<SDValue, MachinePointerInfo>
    computeAddrForCallArg(const SDLoc &dl, SelectionDAG &DAG,
                          const CCValAssign &VA, SDValue StackPtr,
                          bool IsTailCall, int SPDiff) const;
    ByValCopyKind ByValNeedsCopyForTailCall(SelectionDAG &DAG, SDValue Src,
                                            SDValue Dst,
                                            ISD::ArgFlagsTy Flags) const;
    SDValue LowerEH_SJLJ_SETJMP(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerEH_SJLJ_LONGJMP(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerEH_SJLJ_SETUP_DISPATCH(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerINTRINSIC_VOID(SDValue Op, SelectionDAG &DAG,
```
- EN: Declares `getEffectiveCallingConv`, a query/helper routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getEffectiveCallingConv`，它是一个围绕SelectionDAG 降级展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 557-571
```cpp
                                    const ARMSubtarget *Subtarget) const;
    SDValue LowerINTRINSIC_WO_CHAIN(SDValue Op, SelectionDAG &DAG,
                                    const ARMSubtarget *Subtarget) const;
    SDValue LowerBlockAddress(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerConstantPool(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerGlobalAddress(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerGlobalAddressDarwin(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerGlobalAddressELF(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerGlobalAddressWindows(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerGlobalTLSAddress(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerToTLSGeneralDynamicModel(GlobalAddressSDNode *GA,
                                            SelectionDAG &DAG) const;
    SDValue LowerToTLSExecModels(GlobalAddressSDNode *GA,
                                 SelectionDAG &DAG,
                                 TLSModel::Model model) const;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 572-586
```cpp
    SDValue LowerGlobalTLSAddressDarwin(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerGlobalTLSAddressWindows(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerBR_JT(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerALUO(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerSELECT(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerSELECT_CC(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerBRCOND(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerBR_CC(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerFCOPYSIGN(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerRETURNADDR(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerFRAMEADDR(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerShiftRightParts(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerShiftLeftParts(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerGET_ROUNDING(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerSET_ROUNDING(SDValue Op, SelectionDAG &DAG) const;
```
- EN: Declares `LowerGlobalTLSAddressDarwin`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LowerGlobalTLSAddressDarwin`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 587-601
```cpp
    SDValue LowerSET_FPMODE(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerRESET_FPMODE(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerConstantFP(SDValue Op, SelectionDAG &DAG,
                            const ARMSubtarget *ST) const;
    SDValue LowerBUILD_VECTOR(SDValue Op, SelectionDAG &DAG,
                              const ARMSubtarget *ST) const;
    SDValue LowerINSERT_VECTOR_ELT(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerDivRem(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerDIV_Windows(SDValue Op, SelectionDAG &DAG, bool Signed) const;
    void ExpandDIV_Windows(SDValue Op, SelectionDAG &DAG, bool Signed,
                           SmallVectorImpl<SDValue> &Results) const;
    SDValue ExpandBITCAST(SDNode *N, SelectionDAG &DAG,
                          const ARMSubtarget *Subtarget) const;
    SDValue LowerWindowsDIVLibCall(SDValue Op, SelectionDAG &DAG, bool Signed,
                                   SDValue &Chain) const;
```
- EN: Declares `LowerSET_FPMODE`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LowerSET_FPMODE`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 602-616
```cpp
    SDValue LowerREM(SDNode *N, SelectionDAG &DAG) const;
    SDValue LowerDYNAMIC_STACKALLOC(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerFP_ROUND(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerFP_EXTEND(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerFP_TO_INT(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerINT_TO_FP(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerFSETCC(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerSPONENTRY(SDValue Op, SelectionDAG &DAG) const;
    void LowerLOAD(SDNode *N, SmallVectorImpl<SDValue> &Results,
                   SelectionDAG &DAG) const;
    SDValue LowerSTORE(SDValue Op, SelectionDAG &DAG,
                       const ARMSubtarget *Subtarget) const;
    std::pair<SDValue, SDValue>
    LowerAEABIUnalignedLoad(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerAEABIUnalignedStore(SDValue Op, SelectionDAG &DAG) const;
```
- EN: Declares `LowerREM`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LowerREM`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 617-621
```cpp
    SDValue LowerFP_TO_BF16(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerCMP(SDValue Op, SelectionDAG &DAG) const;
    SDValue LowerABS(SDValue Op, SelectionDAG &DAG) const;
    Register getRegisterByName(const char* RegName, LLT VT,
                               const MachineFunction &MF) const override;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 623-624
```cpp
    SDValue BuildSDIVPow2(SDNode *N, const APInt &Divisor, SelectionDAG &DAG,
                          SmallVectorImpl<SDNode *> &Created) const override;
```
- EN: Declares `BuildSDIVPow2`, a mutation/build routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildSDIVPow2`，它是一个围绕SelectionDAG 降级展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 626-627
```cpp
    bool isFMAFasterThanFMulAndFAdd(const MachineFunction &MF,
                                    EVT VT) const override;
```
- EN: Declares `isFMAFasterThanFMulAndFAdd`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isFMAFasterThanFMulAndFAdd`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 629-632
```cpp
    SDValue MoveToHPR(const SDLoc &dl, SelectionDAG &DAG, MVT LocVT, MVT ValVT,
                      SDValue Val) const;
    SDValue MoveFromHPR(const SDLoc &dl, SelectionDAG &DAG, MVT LocVT,
                        MVT ValVT, SDValue Val) const;
```
- EN: Declares `MoveToHPR`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `MoveToHPR`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 634-634
```cpp
    SDValue ReconstructShuffle(SDValue Op, SelectionDAG &DAG) const;
```
- EN: Declares `ReconstructShuffle`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ReconstructShuffle`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 636-641
```cpp
    SDValue LowerCallResult(SDValue Chain, SDValue InGlue,
                            CallingConv::ID CallConv, bool isVarArg,
                            const SmallVectorImpl<ISD::InputArg> &Ins,
                            const SDLoc &dl, SelectionDAG &DAG,
                            SmallVectorImpl<SDValue> &InVals, bool isThisReturn,
                            SDValue ThisVal, bool isCmseNSCall) const;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 643-646
```cpp
    void initializeSplitCSR(MachineBasicBlock *Entry) const override;
    void insertCopiesSplitCSR(
      MachineBasicBlock *Entry,
      const SmallVectorImpl<MachineBasicBlock *> &Exits) const override;
```
- EN: Declares `initializeSplitCSR`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `initializeSplitCSR`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 648-651
```cpp
    bool splitValueIntoRegisterParts(
        SelectionDAG & DAG, const SDLoc &DL, SDValue Val, SDValue *Parts,
        unsigned NumParts, MVT PartVT, std::optional<CallingConv::ID> CC)
        const override;
```
- EN: Declares `splitValueIntoRegisterParts`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `splitValueIntoRegisterParts`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 653-656
```cpp
    SDValue joinRegisterPartsIntoValue(
        SelectionDAG & DAG, const SDLoc &DL, const SDValue *Parts,
        unsigned NumParts, MVT PartVT, EVT ValueVT,
        std::optional<CallingConv::ID> CC) const override;
```
- EN: Declares `joinRegisterPartsIntoValue`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `joinRegisterPartsIntoValue`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 658-662
```cpp
    SDValue
    LowerFormalArguments(SDValue Chain, CallingConv::ID CallConv, bool isVarArg,
                         const SmallVectorImpl<ISD::InputArg> &Ins,
                         const SDLoc &dl, SelectionDAG &DAG,
                         SmallVectorImpl<SDValue> &InVals) const override;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 664-667
```cpp
    int StoreByValRegs(CCState &CCInfo, SelectionDAG &DAG, const SDLoc &dl,
                       SDValue &Chain, const Value *OrigArg,
                       unsigned InRegsParamRecordIdx, int ArgOffset,
                       unsigned ArgSize) const;
```
- EN: Declares `StoreByValRegs`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `StoreByValRegs`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 669-672
```cpp
    void VarArgStyleRegisters(CCState &CCInfo, SelectionDAG &DAG,
                              const SDLoc &dl, SDValue &Chain,
                              unsigned ArgOffset, unsigned TotalArgRegsSaveSize,
                              bool ForceMutable = false) const;
```
- EN: Declares `VarArgStyleRegisters`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `VarArgStyleRegisters`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 674-675
```cpp
    SDValue LowerCall(TargetLowering::CallLoweringInfo &CLI,
                      SmallVectorImpl<SDValue> &InVals) const override;
```
- EN: Declares `LowerCall`, a lowering routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LowerCall`，它是一个围绕目标相关状态展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 677-678
```cpp
    /// HandleByVal - Target-specific cleanup for ByVal support.
    void HandleByVal(CCState *, unsigned &, Align) const override;
```
- EN: Declares `HandleByVal`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `HandleByVal`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 680-685
```cpp
    /// IsEligibleForTailCallOptimization - Check whether the call is eligible
    /// for tail call optimization. Targets which want to do tail call
    /// optimization should implement this function.
    bool IsEligibleForTailCallOptimization(
        TargetLowering::CallLoweringInfo &CLI, CCState &CCInfo,
        SmallVectorImpl<CCValAssign> &ArgLocs, const bool isIndirect) const;
```
- EN: Declares `IsEligibleForTailCallOptimization`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `IsEligibleForTailCallOptimization`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 687-690
```cpp
    bool CanLowerReturn(CallingConv::ID CallConv,
                        MachineFunction &MF, bool isVarArg,
                        const SmallVectorImpl<ISD::OutputArg> &Outs,
                        LLVMContext &Context, const Type *RetTy) const override;
```
- EN: Declares `CanLowerReturn`, a lowering routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `CanLowerReturn`，它是一个围绕机器函数状态展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 692-695
```cpp
    SDValue LowerReturn(SDValue Chain, CallingConv::ID CallConv, bool isVarArg,
                        const SmallVectorImpl<ISD::OutputArg> &Outs,
                        const SmallVectorImpl<SDValue> &OutVals,
                        const SDLoc &dl, SelectionDAG &DAG) const override;
```
- EN: Declares `LowerReturn`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LowerReturn`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 697-697
```cpp
    bool isUsedByReturnOnly(SDNode *N, SDValue &Chain) const override;
```
- EN: Declares `isUsedByReturnOnly`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isUsedByReturnOnly`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 699-699
```cpp
    bool mayBeEmittedAsTailCall(const CallInst *CI) const override;
```
- EN: Declares `mayBeEmittedAsTailCall`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `mayBeEmittedAsTailCall`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 701-701
```cpp
    bool shouldConsiderGEPOffsetSplit() const override { return true; }
```
- EN: Implements `shouldConsiderGEPOffsetSplit`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `shouldConsiderGEPOffsetSplit`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 703-703
```cpp
    bool isUnsupportedFloatingType(EVT VT) const;
```
- EN: Declares `isUnsupportedFloatingType`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isUnsupportedFloatingType`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 705-705
```cpp
    ArrayRef<MCPhysReg> getRoundingControlRegisters() const override;
```
- EN: Declares `getRoundingControlRegisters`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getRoundingControlRegisters`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 707-712
```cpp
    SDValue getCMOV(const SDLoc &dl, EVT VT, SDValue FalseVal, SDValue TrueVal,
                    SDValue ARMcc, SDValue Flags, SelectionDAG &DAG) const;
    SDValue getARMCmp(SDValue LHS, SDValue RHS, ISD::CondCode CC,
                      SDValue &ARMcc, SelectionDAG &DAG, const SDLoc &dl) const;
    SDValue getVFPCmp(SDValue LHS, SDValue RHS, SelectionDAG &DAG,
                      const SDLoc &dl, bool Signaling = false) const;
```
- EN: Declares `getCMOV`, a query/helper routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getCMOV`，它是一个围绕SelectionDAG 降级展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 714-714
```cpp
    SDValue OptimizeVFPBrcond(SDValue Op, SelectionDAG &DAG) const;
```
- EN: Declares `OptimizeVFPBrcond`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `OptimizeVFPBrcond`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 716-717
```cpp
    void SetupEntryBlockForSjLj(MachineInstr &MI, MachineBasicBlock *MBB,
                                MachineBasicBlock *DispatchBB, int FI) const;
```
- EN: Declares `SetupEntryBlockForSjLj`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `SetupEntryBlockForSjLj`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 719-719
```cpp
    void EmitSjLjDispatchBlock(MachineInstr &MI, MachineBasicBlock *MBB) const;
```
- EN: Declares `EmitSjLjDispatchBlock`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `EmitSjLjDispatchBlock`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 721-722
```cpp
    MachineBasicBlock *EmitStructByval(MachineInstr &MI,
                                       MachineBasicBlock *MBB) const;
```
- EN: Declares `EmitStructByval`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `EmitStructByval`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 724-731
```cpp
    MachineBasicBlock *EmitLowered__chkstk(MachineInstr &MI,
                                           MachineBasicBlock *MBB) const;
    MachineBasicBlock *EmitLowered__dbzchk(MachineInstr &MI,
                                           MachineBasicBlock *MBB) const;
    void addMVEVectorTypes(bool HasMVEFP);
    void addAllExtLoads(const MVT From, const MVT To, LegalizeAction Action);
    void setAllExpand(MVT VT);
  };
```
- EN: Declares `EmitLowered__chkstk`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `EmitLowered__chkstk`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 733-738
```cpp
  enum VMOVModImmType {
    VMOVModImm,
    VMVNModImm,
    MVEVMVNModImm,
    OtherModImm
  };
```
- EN: Defines enumeration `VMOVModImmType` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `VMOVModImmType`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 740-740
```cpp
  namespace ARM {
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 742-744
```cpp
  FastISel *createFastISel(FunctionLoweringInfo &funcInfo,
                           const TargetLibraryInfo *libInfo,
                           const LibcallLoweringInfo *libcallLowering);
```
- EN: Declares `createFastISel`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `createFastISel`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 746-746
```cpp
  } // end namespace ARM
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 748-748
```cpp
} // end namespace llvm
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 750-750
```cpp
#endif // LLVM_LIB_TARGET_ARM_ARMISELLOWERING_H
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
- EN: Stack-frame and spill/reload concerns are reflected in the API or implementation.
  - CN: API 或实现中体现了栈帧以及溢出/重载相关问题。

## Dependencies / 依赖关系

- EN: Backend-local headers: `MCTargetDesc/ARMBaseInfo.h`.
  - CN: 后端本地头文件：`MCTargetDesc/ARMBaseInfo.h`。
- EN: LLVM infrastructure headers: `llvm/ADT/SmallVector.h`, `llvm/ADT/StringRef.h`, `llvm/CodeGen/CallingConvLower.h`, `llvm/CodeGen/ISDOpcodes.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/SelectionDAGNodes.h`, `llvm/CodeGen/TargetLowering.h`, `llvm/CodeGen/ValueTypes.h` ... (+6 more).
  - CN: LLVM 基础设施头文件：`llvm/ADT/SmallVector.h`, `llvm/ADT/StringRef.h`, `llvm/CodeGen/CallingConvLower.h`, `llvm/CodeGen/ISDOpcodes.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/SelectionDAGNodes.h`, `llvm/CodeGen/TargetLowering.h`, `llvm/CodeGen/ValueTypes.h` ... (+6 more)。
- EN: Standard/system headers: `optional`, `utility`.
  - CN: 标准库/系统头文件：`optional`, `utility`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
