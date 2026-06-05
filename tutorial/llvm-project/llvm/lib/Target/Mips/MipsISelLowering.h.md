# MipsISelLowering.h — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsISelLowering.h`
- Repository: `llvm-project`
- Purpose (EN): This file defines the interfaces that Mips uses to lower LLVM code into a selection DAG.
- 用途 (CN): 声明 Mips 后端中的 `MipsISelLowering`，并提供与SelectionDAG 降级与目标降级规则相关的接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===- MipsISelLowering.h - Mips DAG Lowering Interface ---------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the interfaces that Mips uses to lower LLVM code into a
// selection DAG.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 14-15
```cpp
#ifndef LLVM_LIB_TARGET_MIPS_MIPSISELLOWERING_H
#define LLVM_LIB_TARGET_MIPS_MIPSISELLOWERING_H
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 17-31
```cpp
#include "MCTargetDesc/MipsABIInfo.h"
#include "MCTargetDesc/MipsBaseInfo.h"
#include "MCTargetDesc/MipsMCTargetDesc.h"
#include "Mips.h"
#include "MipsSelectionDAGInfo.h"
#include "llvm/CodeGen/CallingConvLower.h"
#include "llvm/CodeGen/ISDOpcodes.h"
#include "llvm/CodeGen/MachineMemOperand.h"
#include "llvm/CodeGen/SelectionDAG.h"
#include "llvm/CodeGen/SelectionDAGNodes.h"
#include "llvm/CodeGen/TargetLowering.h"
#include "llvm/CodeGen/ValueTypes.h"
#include "llvm/CodeGenTypes/MachineValueType.h"
#include "llvm/IR/CallingConv.h"
#include "llvm/IR/InlineAsm.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 32-37
```cpp
#include "llvm/IR/Type.h"
#include "llvm/Target/TargetMachine.h"
#include <algorithm>
#include <deque>
#include <utility>
#include <vector>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 39-39
```cpp
namespace llvm {
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 41-52
```cpp
class Argument;
class FastISel;
class FunctionLoweringInfo;
class MachineBasicBlock;
class MachineFrameInfo;
class MachineInstr;
class MipsCCState;
class MipsFunctionInfo;
class MipsSubtarget;
class MipsTargetMachine;
class TargetLibraryInfo;
class TargetRegisterClass;
```
- EN: Declares `Argument`, packaging target-specific state and APIs around `MipsISelLowering`.
- CN: 这里声明 `Argument`，把与 `MipsISelLowering` 相关的目标特定状态和 API 组织在一起。

### Lines 54-56
```cpp
  //===--------------------------------------------------------------------===//
  // TargetLowering Implementation
  //===--------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 58-59
```cpp
  class MipsTargetLowering : public TargetLowering  {
    bool isMicroMips;
```
- EN: Declares `MipsTargetLowering`, packaging target-specific state and APIs around `MipsISelLowering`.
- CN: 这里声明 `MipsTargetLowering`，把与 `MipsISelLowering` 相关的目标特定状态和 API 组织在一起。

### Lines 61-63
```cpp
  public:
    explicit MipsTargetLowering(const MipsTargetMachine &TM,
                                const MipsSubtarget &STI);
```
- EN: Declares `MipsTargetLowering`, a lowering routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `MipsTargetLowering`，它是一个围绕目标机器策略展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 65-66
```cpp
    static const MipsTargetLowering *create(const MipsTargetMachine &TM,
                                            const MipsSubtarget &STI);
```
- EN: Declares `create`, a query/helper routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `create`，它是一个围绕目标机器策略展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 68-73
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

### Lines 75-77
```cpp
    MVT getScalarShiftAmountTy(const DataLayout &, EVT) const override {
      return MVT::i32;
    }
```
- EN: Implements `getScalarShiftAmountTy`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getScalarShiftAmountTy`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 79-80
```cpp
    EVT getTypeForExtReturn(LLVMContext &Context, EVT VT,
                            ISD::NodeType) const override;
```
- EN: Declares `getTypeForExtReturn`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTypeForExtReturn`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 82-85
```cpp
    bool isCheapToSpeculateCttz(Type *Ty) const override;
    bool isCheapToSpeculateCtlz(Type *Ty) const override;
    bool hasBitTest(SDValue X, SDValue Y) const override;
    bool shouldFoldConstantShiftPairToMask(const SDNode *N) const override;
```
- EN: Declares `isCheapToSpeculateCttz`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isCheapToSpeculateCttz`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 87-90
```cpp
    /// Return the register type for a given MVT, ensuring vectors are treated
    /// as a series of gpr sized integers.
    MVT getRegisterTypeForCallingConv(LLVMContext &Context, CallingConv::ID CC,
                                      EVT VT) const override;
```
- EN: Declares `getRegisterTypeForCallingConv`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getRegisterTypeForCallingConv`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 92-96
```cpp
    /// Return the number of registers for a given MVT, ensuring vectors are
    /// treated as a series of gpr sized integers.
    unsigned getNumRegistersForCallingConv(LLVMContext &Context,
                                           CallingConv::ID CC,
                                           EVT VT) const override;
```
- EN: Declares `getNumRegistersForCallingConv`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getNumRegistersForCallingConv`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 98-101
```cpp
    /// Break down vectors to the correct number of gpr sized integers.
    unsigned getVectorTypeBreakdownForCallingConv(
        LLVMContext &Context, CallingConv::ID CC, EVT VT, EVT &IntermediateVT,
        unsigned &NumIntermediates, MVT &RegisterVT) const override;
```
- EN: Declares `getVectorTypeBreakdownForCallingConv`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getVectorTypeBreakdownForCallingConv`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 103-110
```cpp
    /// Return the correct alignment for the current calling convention.
    Align getABIAlignmentForCallingConv(Type *ArgTy,
                                        const DataLayout &DL) const override {
      const Align ABIAlign = DL.getABITypeAlign(ArgTy);
      if (ArgTy->isVectorTy())
        return std::min(ABIAlign, Align(8));
      return ABIAlign;
    }
```
- EN: Implements `getABIAlignmentForCallingConv`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getABIAlignmentForCallingConv`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 112-114
```cpp
    ISD::NodeType getExtendForAtomicOps() const override {
      return ISD::SIGN_EXTEND;
    }
```
- EN: Implements `getExtendForAtomicOps`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getExtendForAtomicOps`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 116-117
```cpp
    /// LowerOperation - Provide custom lowering hooks for some operations.
    SDValue LowerOperation(SDValue Op, SelectionDAG &DAG) const override;
```
- EN: Declares `LowerOperation`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LowerOperation`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 119-123
```cpp
    /// ReplaceNodeResults - Replace the results of node with an illegal result
    /// type with new values built out of custom code.
    ///
    void ReplaceNodeResults(SDNode *N, SmallVectorImpl<SDValue>&Results,
                            SelectionDAG &DAG) const override;
```
- EN: Declares `ReplaceNodeResults`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ReplaceNodeResults`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 125-127
```cpp
    /// getSetCCResultType - get the ISD::SETCC result ValueType
    EVT getSetCCResultType(const DataLayout &DL, LLVMContext &Context,
                           EVT VT) const override;
```
- EN: Declares `getSetCCResultType`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getSetCCResultType`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 129-129
```cpp
    SDValue PerformDAGCombine(SDNode *N, DAGCombinerInfo &DCI) const override;
```
- EN: Declares `PerformDAGCombine`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `PerformDAGCombine`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 131-133
```cpp
    MachineBasicBlock *
    EmitInstrWithCustomInserter(MachineInstr &MI,
                                MachineBasicBlock *MBB) const override;
```
- EN: Declares `EmitInstrWithCustomInserter`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `EmitInstrWithCustomInserter`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 135-136
```cpp
    void AdjustInstrPostInstrSelection(MachineInstr &MI,
                                       SDNode *Node) const override;
```
- EN: Declares `AdjustInstrPostInstrSelection`, a mutation/build routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `AdjustInstrPostInstrSelection`，它是一个围绕机器指令展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 138-138
```cpp
    void HandleByVal(CCState *, unsigned &, Align) const override;
```
- EN: Declares `HandleByVal`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `HandleByVal`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 140-141
```cpp
    Register getRegisterByName(const char* RegName, LLT VT,
                               const MachineFunction &MF) const override;
```
- EN: Declares `getRegisterByName`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getRegisterByName`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 143-148
```cpp
    /// If a physical register, this returns the register that receives the
    /// exception address on entry to an EH pad.
    Register
    getExceptionPointerRegister(const Constant *PersonalityFn) const override {
      return ABI.IsN64() ? Mips::A0_64 : Mips::A0;
    }
```
- EN: Implements `getExceptionPointerRegister`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getExceptionPointerRegister`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 150-155
```cpp
    /// If a physical register, this returns the register that receives the
    /// exception typeid on entry to a landing pad.
    Register
    getExceptionSelectorRegister(const Constant *PersonalityFn) const override {
      return ABI.IsN64() ? Mips::A1_64 : Mips::A1;
    }
```
- EN: Implements `getExceptionSelectorRegister`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getExceptionSelectorRegister`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 157-159
```cpp
    bool isJumpTableRelative() const override {
      return getTargetMachine().isPositionIndependent();
    }
```
- EN: Implements `isJumpTableRelative`, a query/helper routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isJumpTableRelative`，它是一个围绕目标机器策略展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 161-161
```cpp
   CCAssignFn *CCAssignFnForCall() const;
```
- EN: Declares `CCAssignFnForCall`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `CCAssignFnForCall`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 163-163
```cpp
   CCAssignFn *CCAssignFnForReturn() const;
```
- EN: Declares `CCAssignFnForReturn`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `CCAssignFnForReturn`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 165-166
```cpp
  protected:
    SDValue getGlobalReg(SelectionDAG &DAG, EVT Ty) const;
```
- EN: Declares `getGlobalReg`, a query/helper routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getGlobalReg`，它是一个围绕SelectionDAG 降级展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 168-182
```cpp
    // This method creates the following nodes, which are necessary for
    // computing a local symbol's address:
    //
    // (add (load (wrapper $gp, %got(sym)), %lo(sym))
    template <class NodeTy>
    SDValue getAddrLocal(NodeTy *N, const SDLoc &DL, EVT Ty, SelectionDAG &DAG,
                         bool IsN32OrN64) const {
      unsigned GOTFlag = IsN32OrN64 ? MipsII::MO_GOT_PAGE : MipsII::MO_GOT;
      SDValue GOT = DAG.getNode(MipsISD::Wrapper, DL, Ty, getGlobalReg(DAG, Ty),
                                getTargetNode(N, Ty, DAG, GOTFlag));
      SDValue Load =
          DAG.getLoad(Ty, DL, DAG.getEntryNode(), GOT,
                      MachinePointerInfo::getGOT(DAG.getMachineFunction()));
      unsigned LoFlag = IsN32OrN64 ? MipsII::MO_GOT_OFST : MipsII::MO_ABS_LO;
      SDValue Lo = DAG.getNode(MipsISD::Lo, DL, Ty,
```
- EN: Declares `NodeTy`, packaging target-specific state and APIs around `MipsISelLowering`.
- CN: 这里声明 `NodeTy`，把与 `MipsISelLowering` 相关的目标特定状态和 API 组织在一起。

### Lines 183-185
```cpp
                               getTargetNode(N, Ty, DAG, LoFlag));
      return DAG.getNode(ISD::ADD, DL, Ty, Load, Lo);
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 187-198
```cpp
    // This method creates the following nodes, which are necessary for
    // computing a global symbol's address:
    //
    // (load (wrapper $gp, %got(sym)))
    template <class NodeTy>
    SDValue getAddrGlobal(NodeTy *N, const SDLoc &DL, EVT Ty, SelectionDAG &DAG,
                          unsigned Flag, SDValue Chain,
                          const MachinePointerInfo &PtrInfo) const {
      SDValue Tgt = DAG.getNode(MipsISD::Wrapper, DL, Ty, getGlobalReg(DAG, Ty),
                                getTargetNode(N, Ty, DAG, Flag));
      return DAG.getLoad(Ty, DL, Chain, Tgt, PtrInfo);
    }
```
- EN: Declares `NodeTy`, packaging target-specific state and APIs around `MipsISelLowering`.
- CN: 这里声明 `NodeTy`，把与 `MipsISelLowering` 相关的目标特定状态和 API 组织在一起。

### Lines 200-214
```cpp
    // This method creates the following nodes, which are necessary for
    // computing a global symbol's address in large-GOT mode:
    //
    // (load (wrapper (add %hi(sym), $gp), %lo(sym)))
    template <class NodeTy>
    SDValue getAddrGlobalLargeGOT(NodeTy *N, const SDLoc &DL, EVT Ty,
                                  SelectionDAG &DAG, unsigned HiFlag,
                                  unsigned LoFlag, SDValue Chain,
                                  const MachinePointerInfo &PtrInfo) const {
      SDValue Hi = DAG.getNode(MipsISD::GotHi, DL, Ty,
                               getTargetNode(N, Ty, DAG, HiFlag));
      Hi = DAG.getNode(ISD::ADD, DL, Ty, Hi, getGlobalReg(DAG, Ty));
      SDValue Wrapper = DAG.getNode(MipsISD::Wrapper, DL, Ty, Hi,
                                    getTargetNode(N, Ty, DAG, LoFlag));
      return DAG.getLoad(Ty, DL, Chain, Wrapper, PtrInfo);
```
- EN: Declares `NodeTy`, packaging target-specific state and APIs around `MipsISelLowering`.
- CN: 这里声明 `NodeTy`，把与 `MipsISelLowering` 相关的目标特定状态和 API 组织在一起。

### Lines 215-215
```cpp
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 217-231
```cpp
    // This method creates the following nodes, which are necessary for
    // computing a symbol's address in non-PIC mode:
    //
    // (add %hi(sym), %lo(sym))
    //
    // This method covers O32, N32 and N64 in sym32 mode.
    template <class NodeTy>
    SDValue getAddrNonPIC(NodeTy *N, const SDLoc &DL, EVT Ty,
                          SelectionDAG &DAG) const {
      SDValue Hi = getTargetNode(N, Ty, DAG, MipsII::MO_ABS_HI);
      SDValue Lo = getTargetNode(N, Ty, DAG, MipsII::MO_ABS_LO);
      return DAG.getNode(ISD::ADD, DL, Ty,
                         DAG.getNode(MipsISD::Hi, DL, Ty, Hi),
                         DAG.getNode(MipsISD::Lo, DL, Ty, Lo));
   }
```
- EN: Declares `NodeTy`, packaging target-specific state and APIs around `MipsISelLowering`.
- CN: 这里声明 `NodeTy`，把与 `MipsISelLowering` 相关的目标特定状态和 API 组织在一起。

### Lines 233-244
```cpp
   // This method creates the following nodes, which are necessary for
   // computing a symbol's address in non-PIC mode for N64.
   //
   // (add (shl (add (shl (add %highest(sym), %higher(sim)), 16), %high(sym)),
   //            16), %lo(%sym))
   //
   // FIXME: This method is not efficent for (micro)MIPS64R6.
   template <class NodeTy>
   SDValue getAddrNonPICSym64(NodeTy *N, const SDLoc &DL, EVT Ty,
                          SelectionDAG &DAG) const {
      SDValue Hi = getTargetNode(N, Ty, DAG, MipsII::MO_ABS_HI);
      SDValue Lo = getTargetNode(N, Ty, DAG, MipsII::MO_ABS_LO);
```
- EN: Declares `NodeTy`, packaging target-specific state and APIs around `MipsISelLowering`.
- CN: 这里声明 `NodeTy`，把与 `MipsISelLowering` 相关的目标特定状态和 API 组织在一起。

### Lines 246-257
```cpp
      SDValue Highest =
          DAG.getNode(MipsISD::Highest, DL, Ty,
                      getTargetNode(N, Ty, DAG, MipsII::MO_HIGHEST));
      SDValue Higher = getTargetNode(N, Ty, DAG, MipsII::MO_HIGHER);
      SDValue HigherPart =
          DAG.getNode(ISD::ADD, DL, Ty, Highest,
                      DAG.getNode(MipsISD::Higher, DL, Ty, Higher));
      SDValue Cst = DAG.getConstant(16, DL, MVT::i32);
      SDValue Shift = DAG.getNode(ISD::SHL, DL, Ty, HigherPart, Cst);
      SDValue Add = DAG.getNode(ISD::ADD, DL, Ty, Shift,
                                DAG.getNode(MipsISD::Hi, DL, Ty, Hi));
      SDValue Shift2 = DAG.getNode(ISD::SHL, DL, Ty, Add, Cst);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 259-261
```cpp
      return DAG.getNode(ISD::ADD, DL, Ty, Shift2,
                         DAG.getNode(MipsISD::Lo, DL, Ty, Lo));
   }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 263-275
```cpp
    // This method creates the following nodes, which are necessary for
    // computing a symbol's address using gp-relative addressing:
    //
    // (add $gp, %gp_rel(sym))
    template <class NodeTy>
    SDValue getAddrGPRel(NodeTy *N, const SDLoc &DL, EVT Ty,
                         SelectionDAG &DAG, bool IsN64) const {
      SDValue GPRel = getTargetNode(N, Ty, DAG, MipsII::MO_GPREL);
      return DAG.getNode(
          ISD::ADD, DL, Ty,
          DAG.getRegister(IsN64 ? Mips::GP_64 : Mips::GP, Ty),
          DAG.getNode(MipsISD::GPRel, DL, DAG.getVTList(Ty), GPRel));
    }
```
- EN: Declares `NodeTy`, packaging target-specific state and APIs around `MipsISelLowering`.
- CN: 这里声明 `NodeTy`，把与 `MipsISelLowering` 相关的目标特定状态和 API 组织在一起。

### Lines 277-290
```cpp
    // This method creates the following nodes, which are necessary for
    // loading a dllimported symbol:
    //
    // (lw (add (shl(%high(sym), 16), %low(sym)))
    template <class NodeTy>
    SDValue getDllimportSymbol(NodeTy *N, const SDLoc &DL, EVT Ty,
                               SelectionDAG &DAG) const {
      SDValue Hi =
          getTargetNode(N, Ty, DAG, MipsII::MO_ABS_HI | MipsII::MO_DLLIMPORT);
      SDValue Lo =
          getTargetNode(N, Ty, DAG, MipsII::MO_ABS_LO | MipsII::MO_DLLIMPORT);
      return DAG.getNode(ISD::ADD, DL, Ty, DAG.getNode(MipsISD::Lo, DL, Ty, Lo),
                         DAG.getNode(MipsISD::Hi, DL, Ty, Hi));
    }
```
- EN: Declares `NodeTy`, packaging target-specific state and APIs around `MipsISelLowering`.
- CN: 这里声明 `NodeTy`，把与 `MipsISelLowering` 相关的目标特定状态和 API 组织在一起。

### Lines 292-302
```cpp
    // This method creates the following nodes, which are necessary for
    // loading a dllimported global variable:
    //
    // (lw (lw (add (shl(%high(sym), 16), %low(sym))))
    template <class NodeTy>
    SDValue getDllimportVariable(NodeTy *N, const SDLoc &DL, EVT Ty,
                                 SelectionDAG &DAG, SDValue Chain,
                                 const MachinePointerInfo &PtrInfo) const {
      return DAG.getLoad(Ty, DL, Chain, getDllimportSymbol(N, DL, Ty, DAG),
                         PtrInfo);
    }
```
- EN: Declares `NodeTy`, packaging target-specific state and APIs around `MipsISelLowering`.
- CN: 这里声明 `NodeTy`，把与 `MipsISelLowering` 相关的目标特定状态和 API 组织在一起。

### Lines 304-312
```cpp
    /// This function fills Ops, which is the list of operands that will later
    /// be used when a function call node is created. It also generates
    /// copyToReg nodes to set up argument registers.
    virtual void
    getOpndList(SmallVectorImpl<SDValue> &Ops,
                std::deque<std::pair<unsigned, SDValue>> &RegsToPass,
                bool IsPICCall, bool GlobalOrExternal, bool InternalLinkage,
                bool IsCallReloc, CallLoweringInfo &CLI, SDValue Callee,
                SDValue Chain) const;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 314-316
```cpp
  protected:
    SDValue lowerLOAD(SDValue Op, SelectionDAG &DAG) const;
    SDValue lowerSTORE(SDValue Op, SelectionDAG &DAG) const;
```
- EN: Declares `lowerLOAD`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `lowerLOAD`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 318-321
```cpp
    // Subtarget Info
    const MipsSubtarget &Subtarget;
    // Cache the ABI from the TargetMachine, we use it everywhere.
    const MipsABIInfo &ABI;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 323-326
```cpp
  private:
    // Create a TargetGlobalAddress node.
    SDValue getTargetNode(GlobalAddressSDNode *N, EVT Ty, SelectionDAG &DAG,
                          unsigned Flag) const;
```
- EN: Declares `getTargetNode`, a query/helper routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetNode`，它是一个围绕SelectionDAG 降级展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 328-330
```cpp
    // Create a TargetExternalSymbol node.
    SDValue getTargetNode(ExternalSymbolSDNode *N, EVT Ty, SelectionDAG &DAG,
                          unsigned Flag) const;
```
- EN: Declares `getTargetNode`, a query/helper routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetNode`，它是一个围绕SelectionDAG 降级展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 332-334
```cpp
    // Create a TargetBlockAddress node.
    SDValue getTargetNode(BlockAddressSDNode *N, EVT Ty, SelectionDAG &DAG,
                          unsigned Flag) const;
```
- EN: Declares `getTargetNode`, a query/helper routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetNode`，它是一个围绕SelectionDAG 降级展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 336-338
```cpp
    // Create a TargetJumpTable node.
    SDValue getTargetNode(JumpTableSDNode *N, EVT Ty, SelectionDAG &DAG,
                          unsigned Flag) const;
```
- EN: Declares `getTargetNode`, a query/helper routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetNode`，它是一个围绕SelectionDAG 降级展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 340-342
```cpp
    // Create a TargetConstantPool node.
    SDValue getTargetNode(ConstantPoolSDNode *N, EVT Ty, SelectionDAG &DAG,
                          unsigned Flag) const;
```
- EN: Declares `getTargetNode`, a query/helper routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetNode`，它是一个围绕SelectionDAG 降级展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 344-350
```cpp
    // Lower Operand helpers
    SDValue LowerCallResult(SDValue Chain, SDValue InGlue,
                            CallingConv::ID CallConv, bool isVarArg,
                            const SmallVectorImpl<ISD::InputArg> &Ins,
                            const SDLoc &dl, SelectionDAG &DAG,
                            SmallVectorImpl<SDValue> &InVals,
                            TargetLowering::CallLoweringInfo &CLI) const;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 352-366
```cpp
    // Lower Operand specifics
    SDValue lowerBRCOND(SDValue Op, SelectionDAG &DAG) const;
    SDValue lowerConstantPool(SDValue Op, SelectionDAG &DAG) const;
    SDValue lowerGlobalAddress(SDValue Op, SelectionDAG &DAG) const;
    SDValue lowerBlockAddress(SDValue Op, SelectionDAG &DAG) const;
    SDValue lowerGlobalTLSAddress(SDValue Op, SelectionDAG &DAG) const;
    SDValue lowerJumpTable(SDValue Op, SelectionDAG &DAG) const;
    SDValue lowerSELECT(SDValue Op, SelectionDAG &DAG) const;
    SDValue lowerSETCC(SDValue Op, SelectionDAG &DAG) const;
    SDValue lowerFSETCC(SDValue Op, SelectionDAG &DAG) const;
    SDValue lowerVASTART(SDValue Op, SelectionDAG &DAG) const;
    SDValue lowerVAARG(SDValue Op, SelectionDAG &DAG) const;
    SDValue lowerFCOPYSIGN(SDValue Op, SelectionDAG &DAG) const;
    SDValue lowerFABS(SDValue Op, SelectionDAG &DAG) const;
    SDValue lowerFABS32(SDValue Op, SelectionDAG &DAG,
```
- EN: Declares `lowerBRCOND`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `lowerBRCOND`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 367-381
```cpp
                        bool HasExtractInsert) const;
    SDValue lowerFABS64(SDValue Op, SelectionDAG &DAG,
                        bool HasExtractInsert) const;
    SDValue lowerFCANONICALIZE(SDValue Op, SelectionDAG &DAG) const;
    SDValue lowerFRAMEADDR(SDValue Op, SelectionDAG &DAG) const;
    SDValue lowerRETURNADDR(SDValue Op, SelectionDAG &DAG) const;
    SDValue lowerEH_RETURN(SDValue Op, SelectionDAG &DAG) const;
    SDValue lowerATOMIC_FENCE(SDValue Op, SelectionDAG& DAG) const;
    SDValue lowerShiftLeftParts(SDValue Op, SelectionDAG& DAG) const;
    SDValue lowerShiftRightParts(SDValue Op, SelectionDAG& DAG,
                                 bool IsSRA) const;
    SDValue lowerEH_DWARF_CFA(SDValue Op, SelectionDAG &DAG) const;
    SDValue lowerFP_TO_SINT(SDValue Op, SelectionDAG &DAG) const;
    SDValue lowerSTRICT_FP_TO_INT(SDValue Op, SelectionDAG &DAG) const;
    SDValue lowerREADCYCLECOUNTER(SDValue Op, SelectionDAG &DAG) const;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 383-388
```cpp
    /// isEligibleForTailCallOptimization - Check whether the call is eligible
    /// for tail call optimization.
    virtual bool
    isEligibleForTailCallOptimization(const CCState &CCInfo,
                                      unsigned NextStackOffset,
                                      const MipsFunctionInfo &FI) const = 0;
```
- EN: Declares `isEligibleForTailCallOptimization`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isEligibleForTailCallOptimization`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 390-399
```cpp
    /// copyByValArg - Copy argument registers which were used to pass a byval
    /// argument to the stack. Create a stack frame object for the byval
    /// argument.
    void copyByValRegs(SDValue Chain, const SDLoc &DL,
                       std::vector<SDValue> &OutChains, SelectionDAG &DAG,
                       const ISD::ArgFlagsTy &Flags,
                       SmallVectorImpl<SDValue> &InVals,
                       const Argument *FuncArg, unsigned FirstReg,
                       unsigned LastReg, const CCValAssign &VA,
                       MipsCCState &State) const;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 401-408
```cpp
    /// passByValArg - Pass a byval argument in registers or on stack.
    void passByValArg(SDValue Chain, const SDLoc &DL,
                      std::deque<std::pair<unsigned, SDValue>> &RegsToPass,
                      SmallVectorImpl<SDValue> &MemOpChains, SDValue StackPtr,
                      MachineFrameInfo &MFI, SelectionDAG &DAG, SDValue Arg,
                      unsigned FirstReg, unsigned LastReg,
                      const ISD::ArgFlagsTy &Flags, bool isLittle,
                      const CCValAssign &VA) const;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 410-415
```cpp
    /// writeVarArgRegs - Write variable function arguments passed in registers
    /// to the stack. Also create a stack frame object for the first variable
    /// argument.
    void writeVarArgRegs(std::vector<SDValue> &OutChains, SDValue Chain,
                         const SDLoc &DL, SelectionDAG &DAG,
                         CCState &State) const;
```
- EN: Declares `writeVarArgRegs`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `writeVarArgRegs`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 417-421
```cpp
    SDValue
    LowerFormalArguments(SDValue Chain, CallingConv::ID CallConv, bool isVarArg,
                         const SmallVectorImpl<ISD::InputArg> &Ins,
                         const SDLoc &dl, SelectionDAG &DAG,
                         SmallVectorImpl<SDValue> &InVals) const override;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 423-425
```cpp
    SDValue passArgOnStack(SDValue StackPtr, unsigned Offset, SDValue Chain,
                           SDValue Arg, const SDLoc &DL, bool IsTailCall,
                           SelectionDAG &DAG) const;
```
- EN: Declares `passArgOnStack`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `passArgOnStack`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 427-428
```cpp
    SDValue LowerCall(TargetLowering::CallLoweringInfo &CLI,
                      SmallVectorImpl<SDValue> &InVals) const override;
```
- EN: Declares `LowerCall`, a lowering routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LowerCall`，它是一个围绕目标相关状态展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 430-433
```cpp
    bool CanLowerReturn(CallingConv::ID CallConv, MachineFunction &MF,
                        bool isVarArg,
                        const SmallVectorImpl<ISD::OutputArg> &Outs,
                        LLVMContext &Context, const Type *RetTy) const override;
```
- EN: Declares `CanLowerReturn`, a lowering routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `CanLowerReturn`，它是一个围绕机器函数状态展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 435-438
```cpp
    SDValue LowerReturn(SDValue Chain, CallingConv::ID CallConv, bool isVarArg,
                        const SmallVectorImpl<ISD::OutputArg> &Outs,
                        const SmallVectorImpl<SDValue> &OutVals,
                        const SDLoc &dl, SelectionDAG &DAG) const override;
```
- EN: Declares `LowerReturn`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LowerReturn`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 440-441
```cpp
    SDValue LowerInterruptReturn(SmallVectorImpl<SDValue> &RetOps,
                                 const SDLoc &DL, SelectionDAG &DAG) const;
```
- EN: Declares `LowerInterruptReturn`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LowerInterruptReturn`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 443-443
```cpp
    bool shouldSignExtendTypeInLibCall(Type *Ty, bool IsSigned) const override;
```
- EN: Declares `shouldSignExtendTypeInLibCall`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `shouldSignExtendTypeInLibCall`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 445-446
```cpp
    // Inline asm support
    ConstraintType getConstraintType(StringRef Constraint) const override;
```
- EN: Declares `getConstraintType`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getConstraintType`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 448-451
```cpp
    /// Examine constraint string and operand type and determine a weight value.
    /// The operand object must already have been set up with the operand type.
    ConstraintWeight getSingleConstraintMatchWeight(
      AsmOperandInfo &info, const char *constraint) const override;
```
- EN: Declares `getSingleConstraintMatchWeight`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getSingleConstraintMatchWeight`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 453-456
```cpp
    /// This function parses registers that appear in inline-asm constraints.
    /// It returns pair (0, 0) on failure.
    std::pair<unsigned, const TargetRegisterClass *>
    parseRegForInlineAsmConstraint(StringRef C, MVT VT) const;
```
- EN: Declares `parseRegForInlineAsmConstraint`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `parseRegForInlineAsmConstraint`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 458-460
```cpp
    std::pair<unsigned, const TargetRegisterClass *>
    getRegForInlineAsmConstraint(const TargetRegisterInfo *TRI,
                                 StringRef Constraint, MVT VT) const override;
```
- EN: Declares `getRegForInlineAsmConstraint`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getRegForInlineAsmConstraint`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 462-468
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

### Lines 470-479
```cpp
    InlineAsm::ConstraintCode
    getInlineAsmMemConstraint(StringRef ConstraintCode) const override {
      if (ConstraintCode == "o")
        return InlineAsm::ConstraintCode::o;
      if (ConstraintCode == "R")
        return InlineAsm::ConstraintCode::R;
      if (ConstraintCode == "ZC")
        return InlineAsm::ConstraintCode::ZC;
      return TargetLowering::getInlineAsmMemConstraint(ConstraintCode);
    }
```
- EN: Implements `getInlineAsmMemConstraint`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getInlineAsmMemConstraint`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 481-483
```cpp
    bool isLegalAddressingMode(const DataLayout &DL, const AddrMode &AM,
                               Type *Ty, unsigned AS,
                               Instruction *I = nullptr) const override;
```
- EN: Declares `isLegalAddressingMode`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isLegalAddressingMode`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 485-485
```cpp
    bool isOffsetFoldingLegal(const GlobalAddressSDNode *GA) const override;
```
- EN: Declares `isOffsetFoldingLegal`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isOffsetFoldingLegal`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 487-488
```cpp
    EVT getOptimalMemOpType(LLVMContext &Context, const MemOp &Op,
                            const AttributeList &FuncAttributes) const override;
```
- EN: Declares `getOptimalMemOpType`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getOptimalMemOpType`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 490-494
```cpp
    /// isFPImmLegal - Returns true if the target can instruction select the
    /// specified FP immediate natively. If false, the legalizer will
    /// materialize the FP immediate as a load from a constant pool.
    bool isFPImmLegal(const APFloat &Imm, EVT VT,
                      bool ForCodeSize) const override;
```
- EN: Declares `isFPImmLegal`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isFPImmLegal`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 496-497
```cpp
    bool isLegalICmpImmediate(int64_t Imm) const override;
    bool isLegalAddImmediate(int64_t Imm) const override;
```
- EN: Declares `isLegalICmpImmediate`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isLegalICmpImmediate`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 499-502
```cpp
    unsigned getJumpTableEncoding() const override;
    SDValue getPICJumpTableRelocBase(SDValue Table,
                                     SelectionDAG &DAG) const override;
    bool useSoftFloat() const override;
```
- EN: Declares `getJumpTableEncoding`, a query/helper routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getJumpTableEncoding`，它是一个围绕SelectionDAG 降级展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 504-506
```cpp
    bool shouldInsertFencesForAtomic(const Instruction *I) const override {
      return true;
    }
```
- EN: Implements `shouldInsertFencesForAtomic`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `shouldInsertFencesForAtomic`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 508-508
```cpp
    int getCPURegisterIndex(StringRef Name) const;
```
- EN: Declares `getCPURegisterIndex`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getCPURegisterIndex`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 510-510
```cpp
    ArrayRef<MCPhysReg> getRoundingControlRegisters() const override;
```
- EN: Declares `getRoundingControlRegisters`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getRoundingControlRegisters`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 512-516
```cpp
    /// Emit a sign-extension using sll/sra, seb, or seh appropriately.
    MachineBasicBlock *emitSignExtendToI32InReg(MachineInstr &MI,
                                                MachineBasicBlock *BB,
                                                unsigned Size, unsigned DstReg,
                                                unsigned SrcRec) const;
```
- EN: Declares `emitSignExtendToI32InReg`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitSignExtendToI32InReg`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 518-532
```cpp
    MachineBasicBlock *emitAtomicBinary(MachineInstr &MI,
                                        MachineBasicBlock *BB) const;
    MachineBasicBlock *emitAtomicBinaryPartword(MachineInstr &MI,
                                                MachineBasicBlock *BB,
                                                unsigned Size) const;
    MachineBasicBlock *emitAtomicCmpSwap(MachineInstr &MI,
                                         MachineBasicBlock *BB) const;
    MachineBasicBlock *emitAtomicCmpSwapPartword(MachineInstr &MI,
                                                 MachineBasicBlock *BB,
                                                 unsigned Size) const;
    MachineBasicBlock *emitSEL_D(MachineInstr &MI, MachineBasicBlock *BB) const;
    MachineBasicBlock *emitPseudoSELECT(MachineInstr &MI, MachineBasicBlock *BB,
                                        bool isFPCmp, unsigned Opc) const;
    MachineBasicBlock *emitPseudoD_SELECT(MachineInstr &MI,
                                          MachineBasicBlock *BB) const;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 533-537
```cpp
    MachineBasicBlock *emitLDR_W(MachineInstr &MI, MachineBasicBlock *BB) const;
    MachineBasicBlock *emitLDR_D(MachineInstr &MI, MachineBasicBlock *BB) const;
    MachineBasicBlock *emitSTR_W(MachineInstr &MI, MachineBasicBlock *BB) const;
    MachineBasicBlock *emitSTR_D(MachineInstr &MI, MachineBasicBlock *BB) const;
  };
```
- EN: Declares `emitLDR_W`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitLDR_W`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 539-545
```cpp
  /// Create MipsTargetLowering objects.
  const MipsTargetLowering *
  createMips16TargetLowering(const MipsTargetMachine &TM,
                             const MipsSubtarget &STI);
  const MipsTargetLowering *
  createMipsSETargetLowering(const MipsTargetMachine &TM,
                             const MipsSubtarget &STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 547-547
```cpp
namespace Mips {
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 549-551
```cpp
FastISel *createFastISel(FunctionLoweringInfo &funcInfo,
                         const TargetLibraryInfo *libInfo,
                         const LibcallLoweringInfo *libcallLowering);
```
- EN: Declares `createFastISel`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `createFastISel`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 553-553
```cpp
} // end namespace Mips
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 555-555
```cpp
} // end namespace llvm
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 557-557
```cpp
#endif // LLVM_LIB_TARGET_MIPS_MIPSISELLOWERING_H
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

- EN: Backend-local headers: `MCTargetDesc/MipsABIInfo.h`, `MCTargetDesc/MipsBaseInfo.h`, `MCTargetDesc/MipsMCTargetDesc.h`, `Mips.h`, `MipsSelectionDAGInfo.h`.
  - CN: 后端本地头文件：`MCTargetDesc/MipsABIInfo.h`, `MCTargetDesc/MipsBaseInfo.h`, `MCTargetDesc/MipsMCTargetDesc.h`, `Mips.h`, `MipsSelectionDAGInfo.h`。
- EN: LLVM infrastructure headers: `llvm/CodeGen/CallingConvLower.h`, `llvm/CodeGen/ISDOpcodes.h`, `llvm/CodeGen/MachineMemOperand.h`, `llvm/CodeGen/SelectionDAG.h`, `llvm/CodeGen/SelectionDAGNodes.h`, `llvm/CodeGen/TargetLowering.h`, `llvm/CodeGen/ValueTypes.h`, `llvm/CodeGenTypes/MachineValueType.h` ... (+4 more).
  - CN: LLVM 基础设施头文件：`llvm/CodeGen/CallingConvLower.h`, `llvm/CodeGen/ISDOpcodes.h`, `llvm/CodeGen/MachineMemOperand.h`, `llvm/CodeGen/SelectionDAG.h`, `llvm/CodeGen/SelectionDAGNodes.h`, `llvm/CodeGen/TargetLowering.h`, `llvm/CodeGen/ValueTypes.h`, `llvm/CodeGenTypes/MachineValueType.h` ... (+4 more)。
- EN: Standard/system headers: `algorithm`, `deque`, `utility`, `vector`.
  - CN: 标准库/系统头文件：`algorithm`, `deque`, `utility`, `vector`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
