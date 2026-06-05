# MipsISelLowering.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsISelLowering.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file defines the interfaces that Mips uses to lower LLVM code into a selection DAG.
- 用途 (CN): 实现 Mips 后端中的 `MipsISelLowering`，重点处理SelectionDAG 降级与目标降级规则。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===- MipsISelLowering.cpp - Mips DAG Lowering Implementation ------------===//
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

### Lines 14-28
```cpp
#include "MipsISelLowering.h"
#include "MCTargetDesc/MipsBaseInfo.h"
#include "MCTargetDesc/MipsInstPrinter.h"
#include "MCTargetDesc/MipsMCTargetDesc.h"
#include "MipsCCState.h"
#include "MipsInstrInfo.h"
#include "MipsMachineFunction.h"
#include "MipsRegisterInfo.h"
#include "MipsSubtarget.h"
#include "MipsTargetMachine.h"
#include "MipsTargetObjectFile.h"
#include "llvm/ADT/APFloat.h"
#include "llvm/ADT/ArrayRef.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/Statistic.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 29-43
```cpp
#include "llvm/ADT/StringRef.h"
#include "llvm/ADT/StringSwitch.h"
#include "llvm/CodeGen/CallingConvLower.h"
#include "llvm/CodeGen/FunctionLoweringInfo.h"
#include "llvm/CodeGen/ISDOpcodes.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineJumpTableInfo.h"
#include "llvm/CodeGen/MachineMemOperand.h"
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/SelectionDAG.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 44-58
```cpp
#include "llvm/CodeGen/SelectionDAGNodes.h"
#include "llvm/CodeGen/TargetFrameLowering.h"
#include "llvm/CodeGen/TargetInstrInfo.h"
#include "llvm/CodeGen/TargetRegisterInfo.h"
#include "llvm/CodeGen/ValueTypes.h"
#include "llvm/CodeGenTypes/MachineValueType.h"
#include "llvm/IR/CallingConv.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/IR/DebugLoc.h"
#include "llvm/IR/DerivedTypes.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/GlobalValue.h"
#include "llvm/IR/Module.h"
#include "llvm/IR/Type.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 59-73
```cpp
#include "llvm/IR/Value.h"
#include "llvm/MC/MCContext.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/CodeGen.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Compiler.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/MathExtras.h"
#include "llvm/Target/TargetMachine.h"
#include "llvm/Target/TargetOptions.h"
#include <algorithm>
#include <cassert>
#include <cctype>
#include <cstdint>
#include <deque>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 74-78
```cpp
#include <iterator>
#include <regex>
#include <string>
#include <utility>
#include <vector>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 80-80
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 82-82
```cpp
#define DEBUG_TYPE "mips-lower"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 84-84
```cpp
STATISTIC(NumTailCalls, "Number of tail calls");
```
- EN: Declares `STATISTIC`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `STATISTIC`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 86-87
```cpp
extern cl::opt<bool> EmitJalrReloc;
extern cl::opt<bool> NoZeroDivCheck;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 89-91
```cpp
static cl::opt<bool> UseMipsTailCalls("mips-tail-calls", cl::Hidden,
                                      cl::desc("MIPS: permit tail calls."),
                                      cl::init(false));
```
- EN: Declares `cl::desc`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::desc`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 93-96
```cpp
static const MCPhysReg Mips64DPRegs[8] = {
  Mips::D12_64, Mips::D13_64, Mips::D14_64, Mips::D15_64,
  Mips::D16_64, Mips::D17_64, Mips::D18_64, Mips::D19_64
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 98-104
```cpp
// The MIPS MSA ABI passes vector arguments in the integer register set.
// The number of integer registers used is dependant on the ABI used.
MVT MipsTargetLowering::getRegisterTypeForCallingConv(LLVMContext &Context,
                                                      CallingConv::ID CC,
                                                      EVT VT) const {
  if (!VT.isVector())
    return getRegisterType(Context, VT);
```
- EN: Implements `MipsTargetLowering::getRegisterTypeForCallingConv`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::getRegisterTypeForCallingConv`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 106-110
```cpp
  if (VT.isPow2VectorType() && VT.getVectorElementType().isRound())
    return Subtarget.isABI_O32() || VT.getSizeInBits() == 32 ? MVT::i32
                                                             : MVT::i64;
  return getRegisterType(Context, VT.getVectorElementType());
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 112-122
```cpp
unsigned MipsTargetLowering::getNumRegistersForCallingConv(LLVMContext &Context,
                                                           CallingConv::ID CC,
                                                           EVT VT) const {
  if (VT.isVector()) {
    if (VT.isPow2VectorType() && VT.getVectorElementType().isRound())
      return divideCeil(VT.getSizeInBits(), Subtarget.isABI_O32() ? 32 : 64);
    return VT.getVectorNumElements() *
           getNumRegisters(Context, VT.getVectorElementType());
  }
  return MipsTargetLowering::getNumRegisters(Context, VT);
}
```
- EN: Implements `MipsTargetLowering::getNumRegistersForCallingConv`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::getNumRegistersForCallingConv`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 124-137
```cpp
unsigned MipsTargetLowering::getVectorTypeBreakdownForCallingConv(
    LLVMContext &Context, CallingConv::ID CC, EVT VT, EVT &IntermediateVT,
    unsigned &NumIntermediates, MVT &RegisterVT) const {
  if (VT.isPow2VectorType() && VT.getVectorElementType().isRound()) {
    IntermediateVT = getRegisterTypeForCallingConv(Context, CC, VT);
    RegisterVT = IntermediateVT.getSimpleVT();
    NumIntermediates = getNumRegistersForCallingConv(Context, CC, VT);
    return NumIntermediates;
  }
  IntermediateVT = VT.getVectorElementType();
  NumIntermediates = VT.getVectorNumElements();
  RegisterVT = getRegisterType(Context, IntermediateVT);
  return NumIntermediates * getNumRegisters(Context, IntermediateVT);
}
```
- EN: Implements `MipsTargetLowering::getVectorTypeBreakdownForCallingConv`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::getVectorTypeBreakdownForCallingConv`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 139-143
```cpp
SDValue MipsTargetLowering::getGlobalReg(SelectionDAG &DAG, EVT Ty) const {
  MachineFunction &MF = DAG.getMachineFunction();
  MipsFunctionInfo *FI = MF.getInfo<MipsFunctionInfo>();
  return DAG.getRegister(FI->getGlobalBaseReg(MF), Ty);
}
```
- EN: Implements `MipsTargetLowering::getGlobalReg`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::getGlobalReg`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 145-149
```cpp
SDValue MipsTargetLowering::getTargetNode(GlobalAddressSDNode *N, EVT Ty,
                                          SelectionDAG &DAG,
                                          unsigned Flag) const {
  return DAG.getTargetGlobalAddress(N->getGlobal(), SDLoc(N), Ty, 0, Flag);
}
```
- EN: Implements `MipsTargetLowering::getTargetNode`, a query/helper routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::getTargetNode`，它是一个围绕SelectionDAG 降级展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 151-155
```cpp
SDValue MipsTargetLowering::getTargetNode(ExternalSymbolSDNode *N, EVT Ty,
                                          SelectionDAG &DAG,
                                          unsigned Flag) const {
  return DAG.getTargetExternalSymbol(N->getSymbol(), Ty, Flag);
}
```
- EN: Implements `MipsTargetLowering::getTargetNode`, a query/helper routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::getTargetNode`，它是一个围绕SelectionDAG 降级展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 157-161
```cpp
SDValue MipsTargetLowering::getTargetNode(BlockAddressSDNode *N, EVT Ty,
                                          SelectionDAG &DAG,
                                          unsigned Flag) const {
  return DAG.getTargetBlockAddress(N->getBlockAddress(), Ty, 0, Flag);
}
```
- EN: Implements `MipsTargetLowering::getTargetNode`, a query/helper routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::getTargetNode`，它是一个围绕SelectionDAG 降级展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 163-167
```cpp
SDValue MipsTargetLowering::getTargetNode(JumpTableSDNode *N, EVT Ty,
                                          SelectionDAG &DAG,
                                          unsigned Flag) const {
  return DAG.getTargetJumpTable(N->getIndex(), Ty, Flag);
}
```
- EN: Implements `MipsTargetLowering::getTargetNode`, a query/helper routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::getTargetNode`，它是一个围绕SelectionDAG 降级展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 169-174
```cpp
SDValue MipsTargetLowering::getTargetNode(ConstantPoolSDNode *N, EVT Ty,
                                          SelectionDAG &DAG,
                                          unsigned Flag) const {
  return DAG.getTargetConstantPool(N->getConstVal(), Ty, N->getAlign(),
                                   N->getOffset(), Flag);
}
```
- EN: Implements `MipsTargetLowering::getTargetNode`, a query/helper routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::getTargetNode`，它是一个围绕SelectionDAG 降级展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 176-187
```cpp
MipsTargetLowering::MipsTargetLowering(const MipsTargetMachine &TM,
                                       const MipsSubtarget &STI)
    : TargetLowering(TM, STI), Subtarget(STI), ABI(TM.getABI()) {
  // Mips does not have i1 type, so use i32 for
  // setcc operations results (slt, sgt, ...).
  setBooleanContents(ZeroOrOneBooleanContent);
  setBooleanVectorContents(ZeroOrNegativeOneBooleanContent);
  // The cmp.cond.fmt instruction in MIPS32r6/MIPS64r6 uses 0 and -1 like MSA
  // does. Integer booleans still use 0 and 1.
  if (Subtarget.hasMips32r6())
    setBooleanContents(ZeroOrOneBooleanContent,
                       ZeroOrNegativeOneBooleanContent);
```
- EN: Implements `MipsTargetLowering::MipsTargetLowering`, a lowering routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::MipsTargetLowering`，它是一个围绕目标机器策略展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 189-194
```cpp
  // Load extented operations for i1 types must be promoted
  for (MVT VT : MVT::integer_valuetypes()) {
    setLoadExtAction(ISD::EXTLOAD,  VT, MVT::i1,  Promote);
    setLoadExtAction(ISD::ZEXTLOAD, VT, MVT::i1,  Promote);
    setLoadExtAction(ISD::SEXTLOAD, VT, MVT::i1,  Promote);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 196-201
```cpp
  // MIPS doesn't have extending float->double load/store.  Set LoadExtAction
  // for f32, f16
  for (MVT VT : MVT::fp_valuetypes()) {
    setLoadExtAction(ISD::EXTLOAD, VT, MVT::f32, Expand);
    setLoadExtAction(ISD::EXTLOAD, VT, MVT::f16, Expand);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 203-208
```cpp
  // Set LoadExtAction for f16 vectors to Expand
  for (MVT VT : MVT::fp_fixedlen_vector_valuetypes()) {
    MVT F16VT = MVT::getVectorVT(MVT::f16, VT.getVectorNumElements());
    if (F16VT.isValid())
      setLoadExtAction(ISD::EXTLOAD, VT, F16VT, Expand);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 210-211
```cpp
  setTruncStoreAction(MVT::f32, MVT::f16, Expand);
  setTruncStoreAction(MVT::f64, MVT::f16, Expand);
```
- EN: Declares `setTruncStoreAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setTruncStoreAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 213-213
```cpp
  setTruncStoreAction(MVT::f64, MVT::f32, Expand);
```
- EN: Declares `setTruncStoreAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setTruncStoreAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 215-219
```cpp
  // Used by legalize types to correctly generate the setcc result.
  // Without this, every float setcc comes with a AND/OR with the result,
  // we don't want this, since the fpcmp result goes to a flag register,
  // which is used implicitly by brcond and select operations.
  AddPromotedToType(ISD::SETCC, MVT::i1, MVT::i32);
```
- EN: Declares `AddPromotedToType`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `AddPromotedToType`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 221-235
```cpp
  // Mips Custom Operations
  setOperationAction(ISD::BR_JT,              MVT::Other, Expand);
  setOperationAction(ISD::GlobalAddress,      MVT::i32,   Custom);
  setOperationAction(ISD::BlockAddress,       MVT::i32,   Custom);
  setOperationAction(ISD::GlobalTLSAddress,   MVT::i32,   Custom);
  setOperationAction(ISD::JumpTable,          MVT::i32,   Custom);
  if (!Subtarget.inMips16Mode())
    setOperationAction(ISD::ConstantPool, MVT::i32, Custom);
  setOperationAction(ISD::SELECT,             MVT::f32,   Custom);
  setOperationAction(ISD::SELECT,             MVT::f64,   Custom);
  setOperationAction(ISD::SELECT,             MVT::i32,   Custom);
  setOperationAction(ISD::SETCC,              MVT::f32,   Custom);
  setOperationAction(ISD::SETCC,              MVT::f64,   Custom);
  setOperationAction(ISD::BRCOND,             MVT::Other, Custom);
  setOperationAction(ISD::FABS,               MVT::f32,   Custom);
```
- EN: Declares `setOperationAction`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 236-241
```cpp
  setOperationAction(ISD::FABS,               MVT::f64,   Custom);
  setOperationAction(ISD::FCOPYSIGN,          MVT::f32,   Custom);
  setOperationAction(ISD::FCOPYSIGN,          MVT::f64,   Custom);
  setOperationAction(ISD::FP_TO_SINT,         MVT::i32,   Custom);
  setOperationAction(ISD::STRICT_FP_TO_SINT,  MVT::i32,   Custom);
  setOperationAction(ISD::STRICT_FP_TO_UINT,  MVT::i32,   Custom);
```
- EN: Declares `setOperationAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 243-246
```cpp
  setOperationAction(ISD::STRICT_FSETCC, MVT::f32, Custom);
  setOperationAction(ISD::STRICT_FSETCCS, MVT::f32, Custom);
  setOperationAction(ISD::STRICT_FSETCC, MVT::f64, Custom);
  setOperationAction(ISD::STRICT_FSETCCS, MVT::f64, Custom);
```
- EN: Declares `setOperationAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 248-250
```cpp
  if (Subtarget.hasMips32r2() ||
      getTargetMachine().getTargetTriple().isOSLinux())
    setOperationAction(ISD::READCYCLECOUNTER, MVT::i64, Custom);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 252-266
```cpp
  // Lower fmin/fmax/fclass operations for MIPS R6.
  if (Subtarget.hasMips32r6()) {
    setOperationAction(ISD::FMINNUM_IEEE, MVT::f32, Legal);
    setOperationAction(ISD::FMAXNUM_IEEE, MVT::f32, Legal);
    setOperationAction(ISD::FMINNUM, MVT::f32, Legal);
    setOperationAction(ISD::FMAXNUM, MVT::f32, Legal);
    setOperationAction(ISD::FMINNUM_IEEE, MVT::f64, Legal);
    setOperationAction(ISD::FMAXNUM_IEEE, MVT::f64, Legal);
    setOperationAction(ISD::FMINNUM, MVT::f64, Legal);
    setOperationAction(ISD::FMAXNUM, MVT::f64, Legal);
    setOperationAction(ISD::IS_FPCLASS, MVT::f32, Legal);
    setOperationAction(ISD::IS_FPCLASS, MVT::f64, Legal);
    setOperationAction(ISD::FCANONICALIZE, MVT::f32, Legal);
    setOperationAction(ISD::FCANONICALIZE, MVT::f64, Legal);
  } else {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 267-269
```cpp
    setOperationAction(ISD::FCANONICALIZE, MVT::f32, Custom);
    setOperationAction(ISD::FCANONICALIZE, MVT::f64, Custom);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 271-285
```cpp
  if (Subtarget.isGP64bit()) {
    setOperationAction(ISD::GlobalAddress,      MVT::i64,   Custom);
    setOperationAction(ISD::BlockAddress,       MVT::i64,   Custom);
    setOperationAction(ISD::GlobalTLSAddress,   MVT::i64,   Custom);
    setOperationAction(ISD::JumpTable,          MVT::i64,   Custom);
    if (!Subtarget.inMips16Mode())
      setOperationAction(ISD::ConstantPool, MVT::i64, Custom);
    setOperationAction(ISD::SELECT,             MVT::i64,   Custom);
    if (Subtarget.hasMips64r6()) {
      setOperationAction(ISD::LOAD,               MVT::i64,   Legal);
      setOperationAction(ISD::STORE,              MVT::i64,   Legal);
    } else {
      setOperationAction(ISD::LOAD,               MVT::i64,   Custom);
      setOperationAction(ISD::STORE,              MVT::i64,   Custom);
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 286-292
```cpp
    setOperationAction(ISD::FP_TO_SINT,         MVT::i64,   Custom);
    setOperationAction(ISD::STRICT_FP_TO_UINT,  MVT::i64,   Custom);
    setOperationAction(ISD::STRICT_FP_TO_SINT,  MVT::i64,   Custom);
    setOperationAction(ISD::SHL_PARTS,          MVT::i64,   Custom);
    setOperationAction(ISD::SRA_PARTS,          MVT::i64,   Custom);
    setOperationAction(ISD::SRL_PARTS,          MVT::i64,   Custom);
  }
```
- EN: Declares `setOperationAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 294-298
```cpp
  if (!Subtarget.isGP64bit()) {
    setOperationAction(ISD::SHL_PARTS,          MVT::i32,   Custom);
    setOperationAction(ISD::SRA_PARTS,          MVT::i32,   Custom);
    setOperationAction(ISD::SRL_PARTS,          MVT::i32,   Custom);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 300-302
```cpp
  setOperationAction(ISD::EH_DWARF_CFA,         MVT::i32,   Custom);
  if (Subtarget.isGP64bit())
    setOperationAction(ISD::EH_DWARF_CFA,       MVT::i64,   Custom);
```
- EN: Declares `setOperationAction`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 304-311
```cpp
  setOperationAction(ISD::SDIV, MVT::i32, Expand);
  setOperationAction(ISD::SREM, MVT::i32, Expand);
  setOperationAction(ISD::UDIV, MVT::i32, Expand);
  setOperationAction(ISD::UREM, MVT::i32, Expand);
  setOperationAction(ISD::SDIV, MVT::i64, Expand);
  setOperationAction(ISD::SREM, MVT::i64, Expand);
  setOperationAction(ISD::UDIV, MVT::i64, Expand);
  setOperationAction(ISD::UREM, MVT::i64, Expand);
```
- EN: Declares `setOperationAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 313-326
```cpp
  // Operations not directly supported by Mips.
  setOperationAction(ISD::BR_CC,             MVT::f32,   Expand);
  setOperationAction(ISD::BR_CC,             MVT::f64,   Expand);
  setOperationAction(ISD::BR_CC,             MVT::i32,   Expand);
  setOperationAction(ISD::BR_CC,             MVT::i64,   Expand);
  setOperationAction(ISD::SELECT_CC,         MVT::i32,   Expand);
  setOperationAction(ISD::SELECT_CC,         MVT::i64,   Expand);
  setOperationAction(ISD::SELECT_CC,         MVT::f32,   Expand);
  setOperationAction(ISD::SELECT_CC,         MVT::f64,   Expand);
  setOperationAction(ISD::UINT_TO_FP,        MVT::i32,   Expand);
  setOperationAction(ISD::UINT_TO_FP,        MVT::i64,   Expand);
  setOperationAction(ISD::FP_TO_UINT,        MVT::i32,   Expand);
  setOperationAction(ISD::FP_TO_UINT,        MVT::i64,   Expand);
  setOperationAction(ISD::SIGN_EXTEND_INREG, MVT::i1,    Expand);
```
- EN: Declares `setOperationAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 328-340
```cpp
  if (Subtarget.hasCnMips()) {
    setOperationAction(ISD::CTPOP,           MVT::i32,   Legal);
    setOperationAction(ISD::CTPOP,           MVT::i64,   Legal);
  } else {
    setOperationAction(ISD::CTPOP,           MVT::i32,   Expand);
    setOperationAction(ISD::CTPOP,           MVT::i64,   Expand);
  }
  setOperationAction(ISD::CTTZ,              MVT::i32,   Expand);
  setOperationAction(ISD::CTTZ,              MVT::i64,   Expand);
  setOperationAction(ISD::ROTL,              MVT::i32,   Expand);
  setOperationAction(ISD::ROTL,              MVT::i64,   Expand);
  setOperationAction(ISD::DYNAMIC_STACKALLOC, MVT::i32,  Expand);
  setOperationAction(ISD::DYNAMIC_STACKALLOC, MVT::i64,  Expand);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 342-343
```cpp
  if (!Subtarget.hasMips32r2())
    setOperationAction(ISD::ROTR, MVT::i32,   Expand);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 345-346
```cpp
  if (!Subtarget.hasMips64r2())
    setOperationAction(ISD::ROTR, MVT::i64,   Expand);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 348-362
```cpp
  setOperationAction(ISD::FSIN,              MVT::f32,   Expand);
  setOperationAction(ISD::FSIN,              MVT::f64,   Expand);
  setOperationAction(ISD::FCOS,              MVT::f32,   Expand);
  setOperationAction(ISD::FCOS,              MVT::f64,   Expand);
  setOperationAction(ISD::FSINCOS,           MVT::f32,   Expand);
  setOperationAction(ISD::FSINCOS,           MVT::f64,   Expand);
  setOperationAction(ISD::FPOW,              MVT::f32,   Expand);
  setOperationAction(ISD::FPOW,              MVT::f64,   Expand);
  setOperationAction(ISD::FLOG,              MVT::f32,   Expand);
  setOperationAction(ISD::FLOG2,             MVT::f32,   Expand);
  setOperationAction(ISD::FLOG10,            MVT::f32,   Expand);
  setOperationAction(ISD::FEXP,              MVT::f32,   Expand);
  setOperationAction(ISD::FMA,               MVT::f32,   Expand);
  setOperationAction(ISD::FMA,               MVT::f64,   Expand);
  setOperationAction(ISD::FREM, MVT::f32, LibCall);
```
- EN: Declares `setOperationAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 363-363
```cpp
  setOperationAction(ISD::FREM, MVT::f64, LibCall);
```
- EN: Declares `setOperationAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 365-369
```cpp
  // Lower f16 conversion operations into library calls
  setOperationAction(ISD::FP16_TO_FP,        MVT::f32,   Expand);
  setOperationAction(ISD::FP_TO_FP16,        MVT::f32,   Expand);
  setOperationAction(ISD::FP16_TO_FP,        MVT::f64,   Expand);
  setOperationAction(ISD::FP_TO_FP16,        MVT::f64,   Expand);
```
- EN: Declares `setOperationAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 371-371
```cpp
  setOperationAction(ISD::EH_RETURN, MVT::Other, Custom);
```
- EN: Declares `setOperationAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 373-376
```cpp
  setOperationAction(ISD::VASTART,           MVT::Other, Custom);
  setOperationAction(ISD::VAARG,             MVT::Other, Custom);
  setOperationAction(ISD::VACOPY,            MVT::Other, Expand);
  setOperationAction(ISD::VAEND,             MVT::Other, Expand);
```
- EN: Declares `setOperationAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 378-380
```cpp
  // Use the default for now
  setOperationAction(ISD::STACKSAVE,         MVT::Other, Expand);
  setOperationAction(ISD::STACKRESTORE,      MVT::Other, Expand);
```
- EN: Declares `setOperationAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 382-385
```cpp
  if (!Subtarget.isGP64bit()) {
    setOperationAction(ISD::ATOMIC_LOAD,     MVT::i64,   Expand);
    setOperationAction(ISD::ATOMIC_STORE,    MVT::i64,   Expand);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 387-390
```cpp
  if (!Subtarget.hasMips32r2()) {
    setOperationAction(ISD::SIGN_EXTEND_INREG, MVT::i8,  Expand);
    setOperationAction(ISD::SIGN_EXTEND_INREG, MVT::i16, Expand);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 392-396
```cpp
  // MIPS16 lacks MIPS32's clz and clo instructions.
  if (!Subtarget.hasMips32() || Subtarget.inMips16Mode())
    setOperationAction(ISD::CTLZ, MVT::i32, Expand);
  if (!Subtarget.hasMips64())
    setOperationAction(ISD::CTLZ, MVT::i64, Expand);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 398-401
```cpp
  if (!Subtarget.hasMips32r2())
    setOperationAction(ISD::BSWAP, MVT::i32, Expand);
  if (!Subtarget.hasMips64r2())
    setOperationAction(ISD::BSWAP, MVT::i64, Expand);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 403-413
```cpp
  if (Subtarget.isGP64bit() && Subtarget.hasMips64r6()) {
    setLoadExtAction(ISD::SEXTLOAD, MVT::i64, MVT::i32, Legal);
    setLoadExtAction(ISD::ZEXTLOAD, MVT::i64, MVT::i32, Legal);
    setLoadExtAction(ISD::EXTLOAD, MVT::i64, MVT::i32, Legal);
    setTruncStoreAction(MVT::i64, MVT::i32, Legal);
  } else if (Subtarget.isGP64bit()) {
    setLoadExtAction(ISD::SEXTLOAD, MVT::i64, MVT::i32, Custom);
    setLoadExtAction(ISD::ZEXTLOAD, MVT::i64, MVT::i32, Custom);
    setLoadExtAction(ISD::EXTLOAD, MVT::i64, MVT::i32, Custom);
    setTruncStoreAction(MVT::i64, MVT::i32, Custom);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 415-415
```cpp
  setOperationAction(ISD::TRAP, MVT::Other, Legal);
```
- EN: Declares `setOperationAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 417-419
```cpp
  setTargetDAGCombine({ISD::SDIVREM, ISD::UDIVREM, ISD::SELECT, ISD::AND,
                       ISD::OR, ISD::ADD, ISD::SUB, ISD::AssertZext, ISD::SHL,
                       ISD::SIGN_EXTEND});
```
- EN: Implements `setTargetDAGCombine`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `setTargetDAGCombine`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 421-427
```cpp
  // R5900 has no LL/SC instructions for atomic operations
  if (Subtarget.isR5900())
    setMaxAtomicSizeInBitsSupported(0);
  else if (Subtarget.isGP64bit())
    setMaxAtomicSizeInBitsSupported(64);
  else
    setMaxAtomicSizeInBitsSupported(32);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 429-429
```cpp
  setMinFunctionAlignment(Subtarget.isGP64bit() ? Align(8) : Align(4));
```
- EN: Declares `setMinFunctionAlignment`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setMinFunctionAlignment`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 431-434
```cpp
  // The arguments on the stack are defined in terms of 4-byte slots on O32
  // and 8-byte slots on N32/N64.
  setMinStackArgumentAlignment((ABI.IsN32() || ABI.IsN64()) ? Align(8)
                                                            : Align(4));
```
- EN: Declares `setMinStackArgumentAlignment`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setMinStackArgumentAlignment`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 436-436
```cpp
  setStackPointerRegisterToSaveRestore(ABI.IsN64() ? Mips::SP_64 : Mips::SP);
```
- EN: Declares `setStackPointerRegisterToSaveRestore`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setStackPointerRegisterToSaveRestore`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 438-438
```cpp
  MaxStoresPerMemcpy = 16;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 440-441
```cpp
  isMicroMips = Subtarget.inMicroMipsMode();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 443-447
```cpp
const MipsTargetLowering *
MipsTargetLowering::create(const MipsTargetMachine &TM,
                           const MipsSubtarget &STI) {
  if (STI.inMips16Mode())
    return createMips16TargetLowering(TM, STI);
```
- EN: Implements `MipsTargetLowering::create`, a query/helper routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::create`，它是一个围绕目标机器策略展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 449-450
```cpp
  return createMipsSETargetLowering(TM, STI);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 452-457
```cpp
// Create a fast isel object.
FastISel *MipsTargetLowering::createFastISel(
    FunctionLoweringInfo &funcInfo, const TargetLibraryInfo *libInfo,
    const LibcallLoweringInfo *libcallLowering) const {
  const MipsTargetMachine &TM =
      static_cast<const MipsTargetMachine &>(funcInfo.MF->getTarget());
```
- EN: Implements `MipsTargetLowering::createFastISel`, a query/helper routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::createFastISel`，它是一个围绕目标机器策略展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 459-462
```cpp
  // We support only the standard encoding [MIPS32,MIPS32R5] ISAs.
  bool UseFastISel = TM.Options.EnableFastISel && Subtarget.hasMips32() &&
                     !Subtarget.hasMips32r6() && !Subtarget.inMips16Mode() &&
                     !Subtarget.inMicroMipsMode();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 464-468
```cpp
  // Disable if either of the following is true:
  // We do not generate PIC, the ABI is not O32, XGOT is being used.
  if (!TM.isPositionIndependent() || !TM.getABI().IsO32() ||
      Subtarget.useXGOT())
    UseFastISel = false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 470-472
```cpp
  return UseFastISel ? Mips::createFastISel(funcInfo, libInfo, libcallLowering)
                     : nullptr;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 474-479
```cpp
EVT MipsTargetLowering::getSetCCResultType(const DataLayout &, LLVMContext &,
                                           EVT VT) const {
  if (!VT.isVector())
    return MVT::i32;
  return VT.changeVectorElementTypeToInteger();
}
```
- EN: Implements `MipsTargetLowering::getSetCCResultType`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::getSetCCResultType`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 481-485
```cpp
static SDValue performDivRemCombine(SDNode *N, SelectionDAG &DAG,
                                    TargetLowering::DAGCombinerInfo &DCI,
                                    const MipsSubtarget &Subtarget) {
  if (DCI.isBeforeLegalizeOps())
    return SDValue();
```
- EN: Implements `performDivRemCombine`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `performDivRemCombine`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 487-492
```cpp
  EVT Ty = N->getValueType(0);
  unsigned LO = (Ty == MVT::i32) ? Mips::LO0 : Mips::LO0_64;
  unsigned HI = (Ty == MVT::i32) ? Mips::HI0 : Mips::HI0_64;
  unsigned Opc = N->getOpcode() == ISD::SDIVREM ? MipsISD::DivRem16 :
                                                  MipsISD::DivRemU16;
  SDLoc DL(N);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 494-497
```cpp
  SDValue DivRem = DAG.getNode(Opc, DL, MVT::Glue,
                               N->getOperand(0), N->getOperand(1));
  SDValue InChain = DAG.getEntryNode();
  SDValue InGlue = DivRem;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 499-506
```cpp
  // insert MFLO
  if (N->hasAnyUseOfValue(0)) {
    SDValue CopyFromLo = DAG.getCopyFromReg(InChain, DL, LO, Ty,
                                            InGlue);
    DAG.ReplaceAllUsesOfValueWith(SDValue(N, 0), CopyFromLo);
    InChain = CopyFromLo.getValue(1);
    InGlue = CopyFromLo.getValue(2);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 508-513
```cpp
  // insert MFHI
  if (N->hasAnyUseOfValue(1)) {
    SDValue CopyFromHi = DAG.getCopyFromReg(InChain, DL,
                                            HI, Ty, InGlue);
    DAG.ReplaceAllUsesOfValueWith(SDValue(N, 1), CopyFromHi);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 515-516
```cpp
  return SDValue();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 518-532
```cpp
static Mips::CondCode condCodeToFCC(ISD::CondCode CC) {
  switch (CC) {
  default: llvm_unreachable("Unknown fp condition code!");
  case ISD::SETEQ:
  case ISD::SETOEQ: return Mips::FCOND_OEQ;
  case ISD::SETUNE: return Mips::FCOND_UNE;
  case ISD::SETLT:
  case ISD::SETOLT: return Mips::FCOND_OLT;
  case ISD::SETGT:
  case ISD::SETOGT: return Mips::FCOND_OGT;
  case ISD::SETLE:
  case ISD::SETOLE: return Mips::FCOND_OLE;
  case ISD::SETGE:
  case ISD::SETOGE: return Mips::FCOND_OGE;
  case ISD::SETULT: return Mips::FCOND_ULT;
```
- EN: Implements `condCodeToFCC`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `condCodeToFCC`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 533-542
```cpp
  case ISD::SETULE: return Mips::FCOND_ULE;
  case ISD::SETUGT: return Mips::FCOND_UGT;
  case ISD::SETUGE: return Mips::FCOND_UGE;
  case ISD::SETUO:  return Mips::FCOND_UN;
  case ISD::SETO:   return Mips::FCOND_OR;
  case ISD::SETNE:
  case ISD::SETONE: return Mips::FCOND_ONE;
  case ISD::SETUEQ: return Mips::FCOND_UEQ;
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 544-548
```cpp
/// This function returns true if the floating point conditional branches and
/// conditional moves which use condition code CC should be inverted.
static bool invertFPCondCodeUser(Mips::CondCode CC) {
  if (CC >= Mips::FCOND_F && CC <= Mips::FCOND_NGT)
    return false;
```
- EN: Implements `invertFPCondCodeUser`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `invertFPCondCodeUser`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 550-551
```cpp
  assert((CC >= Mips::FCOND_T && CC <= Mips::FCOND_GT) &&
         "Illegal Condition Code");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 553-554
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 556-562
```cpp
// Creates and returns an FPCmp node from a setcc node.
// Returns Op if setcc is not a floating point comparison.
static SDValue createFPCmp(SelectionDAG &DAG, const SDValue &Op) {
  // must be a SETCC node
  if (Op.getOpcode() != ISD::SETCC && Op.getOpcode() != ISD::STRICT_FSETCC &&
      Op.getOpcode() != ISD::STRICT_FSETCCS)
    return Op;
```
- EN: Implements `createFPCmp`, a query/helper routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `createFPCmp`，它是一个围绕SelectionDAG 降级展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 564-564
```cpp
  SDValue LHS = Op.getOperand(0);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 566-567
```cpp
  if (!LHS.getValueType().isFloatingPoint())
    return Op;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 569-570
```cpp
  SDValue RHS = Op.getOperand(1);
  SDLoc DL(Op);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 572-574
```cpp
  // Assume the 3rd operand is a CondCodeSDNode. Add code to check the type of
  // node if necessary.
  ISD::CondCode CC = cast<CondCodeSDNode>(Op.getOperand(2))->get();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 576-578
```cpp
  return DAG.getNode(MipsISD::FPCmp, DL, MVT::Glue, LHS, RHS,
                     DAG.getConstant(condCodeToFCC(CC), DL, MVT::i32));
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 580-585
```cpp
// Creates and returns a CMovFPT/F node.
static SDValue createCMovFP(SelectionDAG &DAG, SDValue Cond, SDValue True,
                            SDValue False, const SDLoc &DL) {
  ConstantSDNode *CC = cast<ConstantSDNode>(Cond.getOperand(2));
  bool invert = invertFPCondCodeUser((Mips::CondCode)CC->getSExtValue());
  SDValue FCC0 = DAG.getRegister(Mips::FCC0, MVT::i32);
```
- EN: Implements `createCMovFP`, a query/helper routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `createCMovFP`，它是一个围绕SelectionDAG 降级展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 587-589
```cpp
  return DAG.getNode((invert ? MipsISD::CMovFP_F : MipsISD::CMovFP_T), DL,
                     True.getValueType(), True, FCC0, False, Cond);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 591-595
```cpp
static SDValue performSELECTCombine(SDNode *N, SelectionDAG &DAG,
                                    TargetLowering::DAGCombinerInfo &DCI,
                                    const MipsSubtarget &Subtarget) {
  if (DCI.isBeforeLegalizeOps())
    return SDValue();
```
- EN: Implements `performSELECTCombine`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `performSELECTCombine`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 597-597
```cpp
  SDValue SetCC = N->getOperand(0);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 599-601
```cpp
  if ((SetCC.getOpcode() != ISD::SETCC) ||
      !SetCC.getOperand(0).getValueType().isInteger())
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 603-604
```cpp
  SDValue False = N->getOperand(2);
  EVT FalseTy = False.getValueType();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 606-607
```cpp
  if (!FalseTy.isInteger())
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 609-609
```cpp
  ConstantSDNode *FalseC = dyn_cast<ConstantSDNode>(False);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 611-619
```cpp
  // If the RHS (False) is 0, we swap the order of the operands
  // of ISD::SELECT (obviously also inverting the condition) so that we can
  // take advantage of conditional moves using the $0 register.
  // Example:
  //   return (a != 0) ? x : 0;
  //     load $reg, x
  //     movz $reg, $0, a
  if (!FalseC)
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 621-621
```cpp
  const SDLoc DL(N);
```
- EN: Declares `DL`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `DL`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 623-625
```cpp
  if (!FalseC->getZExtValue()) {
    ISD::CondCode CC = cast<CondCodeSDNode>(SetCC.getOperand(2))->get();
    SDValue True = N->getOperand(1);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 627-629
```cpp
    SetCC = DAG.getSetCC(DL, SetCC.getValueType(), SetCC.getOperand(0),
                         SetCC.getOperand(1),
                         ISD::getSetCCInverse(CC, SetCC.getValueType()));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 631-632
```cpp
    return DAG.getNode(ISD::SELECT, DL, FalseTy, SetCC, False, True);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 634-637
```cpp
  // If both operands are integer constants there's a possibility that we
  // can do some interesting optimizations.
  SDValue True = N->getOperand(1);
  ConstantSDNode *TrueC = dyn_cast<ConstantSDNode>(True);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 639-640
```cpp
  if (!TrueC || !True.getValueType().isInteger())
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 642-646
```cpp
  // We'll also ignore MVT::i64 operands as this optimizations proves
  // to be ineffective because of the required sign extensions as the result
  // of a SETCC operator is always MVT::i32 for non-vector types.
  if (True.getValueType() == MVT::i64)
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 648-648
```cpp
  int64_t Diff = TrueC->getSExtValue() - FalseC->getSExtValue();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 650-654
```cpp
  // 1)  (a < x) ? y : y-1
  //  slti $reg1, a, x
  //  addiu $reg2, $reg1, y-1
  if (Diff == 1)
    return DAG.getNode(ISD::ADD, DL, SetCC.getValueType(), SetCC, False);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 656-666
```cpp
  // 2)  (a < x) ? y-1 : y
  //  slti $reg1, a, x
  //  xor $reg1, $reg1, 1
  //  addiu $reg2, $reg1, y-1
  if (Diff == -1) {
    ISD::CondCode CC = cast<CondCodeSDNode>(SetCC.getOperand(2))->get();
    SetCC = DAG.getSetCC(DL, SetCC.getValueType(), SetCC.getOperand(0),
                         SetCC.getOperand(1),
                         ISD::getSetCCInverse(CC, SetCC.getValueType()));
    return DAG.getNode(ISD::ADD, DL, SetCC.getValueType(), SetCC, True);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 668-670
```cpp
  // Could not optimize.
  return SDValue();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 672-676
```cpp
static SDValue performCMovFPCombine(SDNode *N, SelectionDAG &DAG,
                                    TargetLowering::DAGCombinerInfo &DCI,
                                    const MipsSubtarget &Subtarget) {
  if (DCI.isBeforeLegalizeOps())
    return SDValue();
```
- EN: Implements `performCMovFPCombine`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `performCMovFPCombine`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 678-678
```cpp
  SDValue ValueIfTrue = N->getOperand(0), ValueIfFalse = N->getOperand(2);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 680-682
```cpp
  ConstantSDNode *FalseC = dyn_cast<ConstantSDNode>(ValueIfFalse);
  if (!FalseC || FalseC->getZExtValue())
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 684-692
```cpp
  // Since RHS (False) is 0, we swap the order of the True/False operands
  // (obviously also inverting the condition) so that we can
  // take advantage of conditional moves using the $0 register.
  // Example:
  //   return (a != 0) ? x : 0;
  //     load $reg, x
  //     movz $reg, $0, a
  unsigned Opc = (N->getOpcode() == MipsISD::CMovFP_T) ? MipsISD::CMovFP_F :
                                                         MipsISD::CMovFP_T;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 694-697
```cpp
  SDValue FCC = N->getOperand(1), Glue = N->getOperand(3);
  return DAG.getNode(Opc, SDLoc(N), ValueIfFalse.getValueType(),
                     ValueIfFalse, FCC, ValueIfTrue, Glue);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 699-703
```cpp
static SDValue performANDCombine(SDNode *N, SelectionDAG &DAG,
                                 TargetLowering::DAGCombinerInfo &DCI,
                                 const MipsSubtarget &Subtarget) {
  if (DCI.isBeforeLegalizeOps() || !Subtarget.hasExtractInsert())
    return SDValue();
```
- EN: Implements `performANDCombine`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `performANDCombine`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 705-709
```cpp
  SDValue FirstOperand = N->getOperand(0);
  unsigned FirstOperandOpc = FirstOperand.getOpcode();
  SDValue Mask = N->getOperand(1);
  EVT ValTy = N->getValueType(0);
  SDLoc DL(N);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 711-715
```cpp
  uint64_t Pos = 0;
  unsigned SMPos, SMSize;
  ConstantSDNode *CN;
  SDValue NewOperand;
  unsigned Opc;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 717-720
```cpp
  // Op's second operand must be a shifted mask.
  if (!(CN = dyn_cast<ConstantSDNode>(Mask)) ||
      !isShiftedMask_64(CN->getZExtValue(), SMPos, SMSize))
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 722-725
```cpp
  if (FirstOperandOpc == ISD::SRA || FirstOperandOpc == ISD::SRL) {
    // Pattern match EXT.
    //  $dst = and ((sra or srl) $src , pos), (2**size - 1)
    //  => ext $dst, $src, pos, size
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 727-729
```cpp
    // The second operand of the shift must be an immediate.
    if (!(CN = dyn_cast<ConstantSDNode>(FirstOperand.getOperand(1))))
      return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 731-731
```cpp
    Pos = CN->getZExtValue();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 733-736
```cpp
    // Return if the shifted mask does not start at bit 0 or the sum of its size
    // and Pos exceeds the word's size.
    if (SMPos != 0 || Pos + SMSize > ValTy.getSizeInBits())
      return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 738-745
```cpp
    Opc = MipsISD::Ext;
    NewOperand = FirstOperand.getOperand(0);
  } else if (FirstOperandOpc == ISD::SHL && Subtarget.hasCnMips()) {
    // Pattern match CINS.
    //  $dst = and (shl $src , pos), mask
    //  => cins $dst, $src, pos, size
    // mask is a shifted mask with consecutive 1's, pos = shift amount,
    // size = population count.
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 747-749
```cpp
    // The second operand of the shift must be an immediate.
    if (!(CN = dyn_cast<ConstantSDNode>(FirstOperand.getOperand(1))))
      return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 751-751
```cpp
    Pos = CN->getZExtValue();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 753-755
```cpp
    if (SMPos != Pos || Pos >= ValTy.getSizeInBits() || SMSize >= 32 ||
        Pos + SMSize > ValTy.getSizeInBits())
      return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 757-764
```cpp
    NewOperand = FirstOperand.getOperand(0);
    // SMSize is 'location' (position) in this case, not size.
    SMSize--;
    Opc = MipsISD::CIns;
  } else {
    // Pattern match EXT.
    //  $dst = and $src, (2**size - 1) , if size > 16
    //  => ext $dst, $src, pos, size , pos = 0
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 766-768
```cpp
    // If the mask is <= 0xffff, andi can be used instead.
    if (CN->getZExtValue() <= 0xffff)
      return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 770-772
```cpp
    // Return if the mask doesn't start at position 0.
    if (SMPos)
      return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 774-780
```cpp
    Opc = MipsISD::Ext;
    NewOperand = FirstOperand;
  }
  return DAG.getNode(Opc, DL, ValTy, NewOperand,
                     DAG.getConstant(Pos, DL, MVT::i32),
                     DAG.getConstant(SMSize, DL, MVT::i32));
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 782-786
```cpp
static SDValue performORCombine(SDNode *N, SelectionDAG &DAG,
                                TargetLowering::DAGCombinerInfo &DCI,
                                const MipsSubtarget &Subtarget) {
  if (DCI.isBeforeLegalizeOps() || !Subtarget.hasExtractInsert())
    return SDValue();
```
- EN: Implements `performORCombine`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `performORCombine`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 788-790
```cpp
  SDValue FirstOperand = N->getOperand(0), SecondOperand = N->getOperand(1);
  unsigned SMPos0, SMSize0, SMPos1, SMSize1;
  ConstantSDNode *CN, *CN1;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 792-806
```cpp
  if ((FirstOperand.getOpcode() == ISD::AND &&
       SecondOperand.getOpcode() == ISD::SHL) ||
      (FirstOperand.getOpcode() == ISD::SHL &&
       SecondOperand.getOpcode() == ISD::AND)) {
    // Pattern match INS.
    //   $dst = or (and $src1, (2**size0 - 1)), (shl $src2, size0)
    //   ==> ins $src1, $src2, pos, size, pos = size0, size = 32 - pos;
    //   Or:
    //   $dst = or (shl $src2, size0), (and $src1, (2**size0 - 1))
    //   ==> ins $src1, $src2, pos, size, pos = size0, size = 32 - pos;
    SDValue AndOperand0 = FirstOperand.getOpcode() == ISD::AND
                              ? FirstOperand.getOperand(0)
                              : SecondOperand.getOperand(0);
    SDValue ShlOperand0 = FirstOperand.getOpcode() == ISD::AND
                              ? SecondOperand.getOperand(0)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 807-813
```cpp
                              : FirstOperand.getOperand(0);
    SDValue AndMask = FirstOperand.getOpcode() == ISD::AND
                          ? FirstOperand.getOperand(1)
                          : SecondOperand.getOperand(1);
    if (!(CN = dyn_cast<ConstantSDNode>(AndMask)) ||
        !isShiftedMask_64(CN->getZExtValue(), SMPos0, SMSize0))
      return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 815-820
```cpp
    SDValue ShlShift = FirstOperand.getOpcode() == ISD::AND
                           ? SecondOperand.getOperand(1)
                           : FirstOperand.getOperand(1);
    if (!(CN = dyn_cast<ConstantSDNode>(ShlShift)))
      return SDValue();
    uint64_t ShlShiftValue = CN->getZExtValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 822-823
```cpp
    if (SMPos0 != 0 || SMSize0 != ShlShiftValue)
      return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 825-833
```cpp
    SDLoc DL(N);
    EVT ValTy = N->getValueType(0);
    SMPos1 = ShlShiftValue;
    assert(SMPos1 < ValTy.getSizeInBits());
    SMSize1 = (ValTy == MVT::i64 ? 64 : 32) - SMPos1;
    return DAG.getNode(MipsISD::Ins, DL, ValTy, ShlOperand0,
                       DAG.getConstant(SMPos1, DL, MVT::i32),
                       DAG.getConstant(SMSize1, DL, MVT::i32), AndOperand0);
  }
```
- EN: Declares `DL`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `DL`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 835-837
```cpp
  // See if Op's first operand matches (and $src1 , mask0).
  if (FirstOperand.getOpcode() != ISD::AND)
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 839-845
```cpp
  // Pattern match INS.
  //  $dst = or (and $src1 , mask0), (and (shl $src, pos), mask1),
  //  where mask1 = (2**size - 1) << pos, mask0 = ~mask1
  //  => ins $dst, $src, size, pos, $src1
  if (!(CN = dyn_cast<ConstantSDNode>(FirstOperand.getOperand(1))) ||
      !isShiftedMask_64(~CN->getSExtValue(), SMPos0, SMSize0))
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 847-849
```cpp
  // See if Op's second operand matches (and (shl $src, pos), mask1).
  if (SecondOperand.getOpcode() == ISD::AND &&
      SecondOperand.getOperand(0).getOpcode() == ISD::SHL) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 851-853
```cpp
    if (!(CN = dyn_cast<ConstantSDNode>(SecondOperand.getOperand(1))) ||
        !isShiftedMask_64(CN->getZExtValue(), SMPos1, SMSize1))
      return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 855-857
```cpp
    // The shift masks must have the same position and size.
    if (SMPos0 != SMPos1 || SMSize0 != SMSize1)
      return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 859-859
```cpp
    SDValue Shl = SecondOperand.getOperand(0);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 861-862
```cpp
    if (!(CN = dyn_cast<ConstantSDNode>(Shl.getOperand(1))))
      return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 864-864
```cpp
    unsigned Shamt = CN->getZExtValue();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 866-870
```cpp
    // Return if the shift amount and the first bit position of mask are not the
    // same.
    EVT ValTy = N->getValueType(0);
    if ((Shamt != SMPos0) || (SMPos0 + SMSize0 > ValTy.getSizeInBits()))
      return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 872-886
```cpp
    SDLoc DL(N);
    return DAG.getNode(MipsISD::Ins, DL, ValTy, Shl.getOperand(0),
                       DAG.getConstant(SMPos0, DL, MVT::i32),
                       DAG.getConstant(SMSize0, DL, MVT::i32),
                       FirstOperand.getOperand(0));
  } else {
    // Pattern match DINS.
    //  $dst = or (and $src, mask0), mask1
    //  where mask0 = ((1 << SMSize0) -1) << SMPos0
    //  => dins $dst, $src, pos, size
    if (~CN->getSExtValue() == ((((int64_t)1 << SMSize0) - 1) << SMPos0) &&
        ((SMSize0 + SMPos0 <= 64 && Subtarget.hasMips64r2()) ||
         (SMSize0 + SMPos0 <= 32))) {
      // Check if AND instruction has constant as argument
      bool isConstCase = SecondOperand.getOpcode() != ISD::AND;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 887-897
```cpp
      if (SecondOperand.getOpcode() == ISD::AND) {
        if (!(CN1 = dyn_cast<ConstantSDNode>(SecondOperand->getOperand(1))))
          return SDValue();
      } else {
        if (!(CN1 = dyn_cast<ConstantSDNode>(N->getOperand(1))))
          return SDValue();
      }
      // Don't generate INS if constant OR operand doesn't fit into bits
      // cleared by constant AND operand.
      if (CN->getSExtValue() & CN1->getSExtValue())
        return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 899-913
```cpp
      SDLoc DL(N);
      EVT ValTy = N->getOperand(0)->getValueType(0);
      SDValue Const1;
      SDValue SrlX;
      if (!isConstCase) {
        Const1 = DAG.getConstant(SMPos0, DL, MVT::i32);
        SrlX = DAG.getNode(ISD::SRL, DL, SecondOperand->getValueType(0),
                           SecondOperand, Const1);
      }
      return DAG.getNode(
          MipsISD::Ins, DL, N->getValueType(0),
          isConstCase
              ? DAG.getSignedConstant(CN1->getSExtValue() >> SMPos0, DL, ValTy)
              : SrlX,
          DAG.getConstant(SMPos0, DL, MVT::i32),
```
- EN: Declares `DL`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `DL`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 914-921
```cpp
          DAG.getConstant(ValTy.getSizeInBits() / 8 < 8 ? SMSize0 & 31
                                                        : SMSize0,
                          DL, MVT::i32),
          FirstOperand->getOperand(0));
    }
    return SDValue();
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 923-929
```cpp
static SDValue performMADD_MSUBCombine(SDNode *ROOTNode, SelectionDAG &CurDAG,
                                       const MipsSubtarget &Subtarget) {
  // ROOTNode must have a multiplication as an operand for the match to be
  // successful.
  if (ROOTNode->getOperand(0).getOpcode() != ISD::MUL &&
      ROOTNode->getOperand(1).getOpcode() != ISD::MUL)
    return SDValue();
```
- EN: Implements `performMADD_MSUBCombine`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `performMADD_MSUBCombine`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 931-937
```cpp
  // In the case where we have a multiplication as the left operand of
  // of a subtraction, we can't combine into a MipsISD::MSub node as the
  // the instruction definition of msub(u) places the multiplication on
  // on the right.
  if (ROOTNode->getOpcode() == ISD::SUB &&
      ROOTNode->getOperand(0).getOpcode() == ISD::MUL)
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 939-941
```cpp
  // We don't handle vector types here.
  if (ROOTNode->getValueType(0).isVector())
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 943-957
```cpp
  // For MIPS64, madd / msub instructions are inefficent to use with 64 bit
  // arithmetic. E.g.
  // (add (mul a b) c) =>
  //   let res = (madd (mthi (drotr c 32))x(mtlo c) a b) in
  //   MIPS64:   (or (dsll (mfhi res) 32) (dsrl (dsll (mflo res) 32) 32)
  //   or
  //   MIPS64R2: (dins (mflo res) (mfhi res) 32 32)
  //
  // The overhead of setting up the Hi/Lo registers and reassembling the
  // result makes this a dubious optimzation for MIPS64. The core of the
  // problem is that Hi/Lo contain the upper and lower 32 bits of the
  // operand and result.
  //
  // It requires a chain of 4 add/mul for MIPS64R2 to get better code
  // density than doing it naively, 5 for MIPS64. Additionally, using
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 958-963
```cpp
  // madd/msub on MIPS64 requires the operands actually be 32 bit sign
  // extended operands, not true 64 bit values.
  //
  // FIXME: For the moment, disable this completely for MIPS64.
  if (Subtarget.hasMips64())
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 965-967
```cpp
  SDValue Mult = ROOTNode->getOperand(0).getOpcode() == ISD::MUL
                     ? ROOTNode->getOperand(0)
                     : ROOTNode->getOperand(1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 969-971
```cpp
  SDValue AddOperand = ROOTNode->getOperand(0).getOpcode() == ISD::MUL
                     ? ROOTNode->getOperand(1)
                     : ROOTNode->getOperand(0);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 973-976
```cpp
  // Transform this to a MADD only if the user of this node is the add.
  // If there are other users of the mul, this function returns here.
  if (!Mult.hasOneUse())
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 978-984
```cpp
  // maddu and madd are unusual instructions in that on MIPS64 bits 63..31
  // must be in canonical form, i.e. sign extended. For MIPS32, the operands
  // of the multiply must have 32 or more sign bits, otherwise we cannot
  // perform this optimization. We have to check this here as we're performing
  // this optimization pre-legalization.
  SDValue MultLHS = Mult->getOperand(0);
  SDValue MultRHS = Mult->getOperand(1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 986-989
```cpp
  bool IsSigned = MultLHS->getOpcode() == ISD::SIGN_EXTEND &&
                  MultRHS->getOpcode() == ISD::SIGN_EXTEND;
  bool IsUnsigned = MultLHS->getOpcode() == ISD::ZERO_EXTEND &&
                    MultRHS->getOpcode() == ISD::ZERO_EXTEND;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 991-992
```cpp
  if (!IsSigned && !IsUnsigned)
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 994-1000
```cpp
  // Initialize accumulator.
  SDLoc DL(ROOTNode);
  SDValue BottomHalf, TopHalf;
  std::tie(BottomHalf, TopHalf) =
      CurDAG.SplitScalar(AddOperand, DL, MVT::i32, MVT::i32);
  SDValue ACCIn =
      CurDAG.getNode(MipsISD::MTLOHI, DL, MVT::Untyped, BottomHalf, TopHalf);
```
- EN: Declares `DL`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `DL`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1002-1009
```cpp
  // Create MipsMAdd(u) / MipsMSub(u) node.
  bool IsAdd = ROOTNode->getOpcode() == ISD::ADD;
  unsigned Opcode = IsAdd ? (IsUnsigned ? MipsISD::MAddu : MipsISD::MAdd)
                          : (IsUnsigned ? MipsISD::MSubu : MipsISD::MSub);
  SDValue MAddOps[3] = {
      CurDAG.getNode(ISD::TRUNCATE, DL, MVT::i32, Mult->getOperand(0)),
      CurDAG.getNode(ISD::TRUNCATE, DL, MVT::i32, Mult->getOperand(1)), ACCIn};
  SDValue MAdd = CurDAG.getNode(Opcode, DL, MVT::Untyped, MAddOps);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1011-1016
```cpp
  SDValue ResLo = CurDAG.getNode(MipsISD::MFLO, DL, MVT::i32, MAdd);
  SDValue ResHi = CurDAG.getNode(MipsISD::MFHI, DL, MVT::i32, MAdd);
  SDValue Combined =
      CurDAG.getNode(ISD::BUILD_PAIR, DL, MVT::i64, ResLo, ResHi);
  return Combined;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1018-1025
```cpp
static SDValue performSUBCombine(SDNode *N, SelectionDAG &DAG,
                                 TargetLowering::DAGCombinerInfo &DCI,
                                 const MipsSubtarget &Subtarget) {
  // (sub v0 (mul v1, v2)) => (msub v1, v2, v0)
  if (DCI.isBeforeLegalizeOps()) {
    if (Subtarget.hasMips32() && !Subtarget.hasMips32r6() &&
        !Subtarget.inMips16Mode() && N->getValueType(0) == MVT::i64)
      return performMADD_MSUBCombine(N, DAG, Subtarget);
```
- EN: Implements `performSUBCombine`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `performSUBCombine`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1027-1028
```cpp
    return SDValue();
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1030-1031
```cpp
  return SDValue();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1033-1040
```cpp
static SDValue performADDCombine(SDNode *N, SelectionDAG &DAG,
                                 TargetLowering::DAGCombinerInfo &DCI,
                                 const MipsSubtarget &Subtarget) {
  // (add v0 (mul v1, v2)) => (madd v1, v2, v0)
  if (DCI.isBeforeLegalizeOps()) {
    if (Subtarget.hasMips32() && !Subtarget.hasMips32r6() &&
        !Subtarget.inMips16Mode() && N->getValueType(0) == MVT::i64)
      return performMADD_MSUBCombine(N, DAG, Subtarget);
```
- EN: Implements `performADDCombine`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `performADDCombine`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1042-1043
```cpp
    return SDValue();
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1045-1054
```cpp
  // When loading from a jump table, push the Lo node to the position that
  // allows folding it into a load immediate.
  // (add v0, (add v1, abs_lo(tjt))) => (add (add v0, v1), abs_lo(tjt))
  // (add (add abs_lo(tjt), v1), v0) => (add (add v0, v1), abs_lo(tjt))
  SDValue InnerAdd = N->getOperand(1);
  SDValue Index = N->getOperand(0);
  if (InnerAdd.getOpcode() != ISD::ADD)
    std::swap(InnerAdd, Index);
  if (InnerAdd.getOpcode() != ISD::ADD)
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1056-1059
```cpp
  SDValue Lo = InnerAdd.getOperand(0);
  SDValue Other = InnerAdd.getOperand(1);
  if (Lo.getOpcode() != MipsISD::Lo)
    std::swap(Lo, Other);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1061-1063
```cpp
  if ((Lo.getOpcode() != MipsISD::Lo) ||
      (Lo.getOperand(0).getOpcode() != ISD::TargetJumpTable))
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1065-1066
```cpp
  EVT ValTy = N->getValueType(0);
  SDLoc DL(N);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1068-1070
```cpp
  SDValue Add1 = DAG.getNode(ISD::ADD, DL, ValTy, Index, Other);
  return DAG.getNode(ISD::ADD, DL, ValTy, Add1, Lo);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1072-1077
```cpp
static SDValue performSHLCombine(SDNode *N, SelectionDAG &DAG,
                                 TargetLowering::DAGCombinerInfo &DCI,
                                 const MipsSubtarget &Subtarget) {
  // Pattern match CINS.
  //  $dst = shl (and $src , imm), pos
  //  => cins $dst, $src, pos, size
```
- EN: Implements `performSHLCombine`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `performSHLCombine`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1079-1080
```cpp
  if (DCI.isBeforeLegalizeOps() || !Subtarget.hasCnMips())
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1082-1086
```cpp
  SDValue FirstOperand = N->getOperand(0);
  unsigned FirstOperandOpc = FirstOperand.getOpcode();
  SDValue SecondOperand = N->getOperand(1);
  EVT ValTy = N->getValueType(0);
  SDLoc DL(N);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1088-1091
```cpp
  uint64_t Pos = 0;
  unsigned SMPos, SMSize;
  ConstantSDNode *CN;
  SDValue NewOperand;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1093-1095
```cpp
  // The second operand of the shift must be an immediate.
  if (!(CN = dyn_cast<ConstantSDNode>(SecondOperand)))
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1097-1097
```cpp
  Pos = CN->getZExtValue();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1099-1100
```cpp
  if (Pos >= ValTy.getSizeInBits())
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1102-1103
```cpp
  if (FirstOperandOpc != ISD::AND)
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1105-1108
```cpp
  // AND's second operand must be a shifted mask.
  if (!(CN = dyn_cast<ConstantSDNode>(FirstOperand.getOperand(1))) ||
      !isShiftedMask_64(CN->getZExtValue(), SMPos, SMSize))
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1110-1113
```cpp
  // Return if the shifted mask does not start at bit 0 or the sum of its size
  // and Pos exceeds the word's size.
  if (SMPos != 0 || SMSize > 32 || Pos + SMSize > ValTy.getSizeInBits())
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1115-1117
```cpp
  NewOperand = FirstOperand.getOperand(0);
  // SMSize is 'location' (position) in this case, not size.
  SMSize--;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1119-1122
```cpp
  return DAG.getNode(MipsISD::CIns, DL, ValTy, NewOperand,
                     DAG.getConstant(Pos, DL, MVT::i32),
                     DAG.getConstant(SMSize, DL, MVT::i32));
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1124-1129
```cpp
static SDValue performSignExtendCombine(SDNode *N, SelectionDAG &DAG,
                                        TargetLowering::DAGCombinerInfo &DCI,
                                        const MipsSubtarget &Subtarget) {
  if (DCI.Level != AfterLegalizeDAG || !Subtarget.isGP64bit()) {
    return SDValue();
  }
```
- EN: Implements `performSignExtendCombine`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `performSignExtendCombine`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1131-1132
```cpp
  SDValue N0 = N->getOperand(0);
  EVT VT = N->getValueType(0);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1134-1141
```cpp
  // Pattern match XOR.
  // $dst = sign_extend (xor (trunc $src, i32), imm)
  // => $dst = xor (signext_inreg $src, i32), imm
  if (N0.getOpcode() == ISD::XOR &&
      N0.getOperand(0).getOpcode() == ISD::TRUNCATE &&
      N0.getOperand(1).getOpcode() == ISD::Constant) {
    SDValue TruncateSource = N0.getOperand(0).getOperand(0);
    auto *ConstantOperand = dyn_cast<ConstantSDNode>(N0->getOperand(1));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1143-1145
```cpp
    SDValue FirstOperand =
        DAG.getNode(ISD::SIGN_EXTEND_INREG, SDLoc(N0), VT, TruncateSource,
                    DAG.getValueType(N0.getOperand(0).getValueType()));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1147-1150
```cpp
    int64_t ConstImm = ConstantOperand->getSExtValue();
    return DAG.getNode(ISD::XOR, SDLoc(N0), VT, FirstOperand,
                       DAG.getConstant(ConstImm, SDLoc(N0), VT));
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1152-1153
```cpp
  return SDValue();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1155-1158
```cpp
SDValue  MipsTargetLowering::PerformDAGCombine(SDNode *N, DAGCombinerInfo &DCI)
  const {
  SelectionDAG &DAG = DCI.DAG;
  unsigned Opc = N->getOpcode();
```
- EN: Implements `MipsTargetLowering::PerformDAGCombine`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::PerformDAGCombine`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1160-1174
```cpp
  switch (Opc) {
  default: break;
  case ISD::SDIVREM:
  case ISD::UDIVREM:
    return performDivRemCombine(N, DAG, DCI, Subtarget);
  case ISD::SELECT:
    return performSELECTCombine(N, DAG, DCI, Subtarget);
  case MipsISD::CMovFP_F:
  case MipsISD::CMovFP_T:
    return performCMovFPCombine(N, DAG, DCI, Subtarget);
  case ISD::AND:
    return performANDCombine(N, DAG, DCI, Subtarget);
  case ISD::OR:
    return performORCombine(N, DAG, DCI, Subtarget);
  case ISD::ADD:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1175-1182
```cpp
    return performADDCombine(N, DAG, DCI, Subtarget);
  case ISD::SHL:
    return performSHLCombine(N, DAG, DCI, Subtarget);
  case ISD::SUB:
    return performSUBCombine(N, DAG, DCI, Subtarget);
  case ISD::SIGN_EXTEND:
    return performSignExtendCombine(N, DAG, DCI, Subtarget);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1184-1185
```cpp
  return SDValue();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1187-1189
```cpp
bool MipsTargetLowering::isCheapToSpeculateCttz(Type *Ty) const {
  return Subtarget.hasMips32();
}
```
- EN: Implements `MipsTargetLowering::isCheapToSpeculateCttz`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::isCheapToSpeculateCttz`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1191-1193
```cpp
bool MipsTargetLowering::isCheapToSpeculateCtlz(Type *Ty) const {
  return Subtarget.hasMips32();
}
```
- EN: Implements `MipsTargetLowering::isCheapToSpeculateCtlz`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::isCheapToSpeculateCtlz`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1195-1200
```cpp
bool MipsTargetLowering::hasBitTest(SDValue X, SDValue Y) const {
  // We can use ANDI+SLTIU as a bit test. Y contains the bit position.
  // For MIPSR2 or later, we may be able to use the `ext` instruction or its
  // double-word variants.
  if (auto *C = dyn_cast<ConstantSDNode>(Y))
    return C->getAPIntValue().ule(15);
```
- EN: Implements `MipsTargetLowering::hasBitTest`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::hasBitTest`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1202-1203
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1205-1211
```cpp
bool MipsTargetLowering::shouldFoldConstantShiftPairToMask(
    const SDNode *N) const {
  assert(((N->getOpcode() == ISD::SHL &&
           N->getOperand(0).getOpcode() == ISD::SRL) ||
          (N->getOpcode() == ISD::SRL &&
           N->getOperand(0).getOpcode() == ISD::SHL)) &&
         "Expected shift-shift mask");
```
- EN: Implements `MipsTargetLowering::shouldFoldConstantShiftPairToMask`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::shouldFoldConstantShiftPairToMask`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1213-1216
```cpp
  if (N->getOperand(0).getValueType().isVector())
    return false;
  return true;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1218-1223
```cpp
void
MipsTargetLowering::ReplaceNodeResults(SDNode *N,
                                       SmallVectorImpl<SDValue> &Results,
                                       SelectionDAG &DAG) const {
  return LowerOperationWrapper(N, Results, DAG);
}
```
- EN: Implements `MipsTargetLowering::ReplaceNodeResults`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::ReplaceNodeResults`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1225-1239
```cpp
SDValue MipsTargetLowering::
LowerOperation(SDValue Op, SelectionDAG &DAG) const
{
  switch (Op.getOpcode())
  {
  case ISD::BRCOND:             return lowerBRCOND(Op, DAG);
  case ISD::ConstantPool:       return lowerConstantPool(Op, DAG);
  case ISD::GlobalAddress:      return lowerGlobalAddress(Op, DAG);
  case ISD::BlockAddress:       return lowerBlockAddress(Op, DAG);
  case ISD::GlobalTLSAddress:   return lowerGlobalTLSAddress(Op, DAG);
  case ISD::JumpTable:          return lowerJumpTable(Op, DAG);
  case ISD::SELECT:             return lowerSELECT(Op, DAG);
  case ISD::SETCC:              return lowerSETCC(Op, DAG);
  case ISD::STRICT_FSETCC:
  case ISD::STRICT_FSETCCS:
```
- EN: Implements `LowerOperation`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `LowerOperation`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1240-1254
```cpp
    return lowerFSETCC(Op, DAG);
  case ISD::VASTART:            return lowerVASTART(Op, DAG);
  case ISD::VAARG:              return lowerVAARG(Op, DAG);
  case ISD::FCOPYSIGN:          return lowerFCOPYSIGN(Op, DAG);
  case ISD::FABS:               return lowerFABS(Op, DAG);
  case ISD::FCANONICALIZE:
    return lowerFCANONICALIZE(Op, DAG);
  case ISD::FRAMEADDR:          return lowerFRAMEADDR(Op, DAG);
  case ISD::RETURNADDR:         return lowerRETURNADDR(Op, DAG);
  case ISD::EH_RETURN:          return lowerEH_RETURN(Op, DAG);
  case ISD::ATOMIC_FENCE:       return lowerATOMIC_FENCE(Op, DAG);
  case ISD::SHL_PARTS:          return lowerShiftLeftParts(Op, DAG);
  case ISD::SRA_PARTS:          return lowerShiftRightParts(Op, DAG, true);
  case ISD::SRL_PARTS:          return lowerShiftRightParts(Op, DAG, false);
  case ISD::LOAD:               return lowerLOAD(Op, DAG);
```
- EN: Declares `lowerFSETCC`, a lowering routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `lowerFSETCC`，它是一个围绕目标相关状态展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1255-1265
```cpp
  case ISD::STORE:              return lowerSTORE(Op, DAG);
  case ISD::EH_DWARF_CFA:       return lowerEH_DWARF_CFA(Op, DAG);
  case ISD::STRICT_FP_TO_SINT:
  case ISD::STRICT_FP_TO_UINT:
    return lowerSTRICT_FP_TO_INT(Op, DAG);
  case ISD::FP_TO_SINT:         return lowerFP_TO_SINT(Op, DAG);
  case ISD::READCYCLECOUNTER:
    return lowerREADCYCLECOUNTER(Op, DAG);
  }
  return SDValue();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1267-1269
```cpp
//===----------------------------------------------------------------------===//
//  Lower helper functions
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 1271-1280
```cpp
// addLiveIn - This helper function adds the specified physical register to the
// MachineFunction as a live in value.  It also creates a corresponding
// virtual register for it.
static unsigned
addLiveIn(MachineFunction &MF, unsigned PReg, const TargetRegisterClass *RC)
{
  Register VReg = MF.getRegInfo().createVirtualRegister(RC);
  MF.getRegInfo().addLiveIn(PReg, VReg);
  return VReg;
}
```
- EN: Implements `addLiveIn`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addLiveIn`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1282-1287
```cpp
static MachineBasicBlock *insertDivByZeroTrap(MachineInstr &MI,
                                              MachineBasicBlock &MBB,
                                              const TargetInstrInfo &TII,
                                              bool Is64Bit, bool IsMicroMips) {
  if (NoZeroDivCheck)
    return &MBB;
```
- EN: Implements `insertDivByZeroTrap`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `insertDivByZeroTrap`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1289-1297
```cpp
  // Insert instruction "teq $divisor_reg, $zero, 7".
  MachineBasicBlock::iterator I(MI);
  MachineInstrBuilder MIB;
  MachineOperand &Divisor = MI.getOperand(2);
  MIB = BuildMI(MBB, std::next(I), MI.getDebugLoc(),
                TII.get(IsMicroMips ? Mips::TEQ_MM : Mips::TEQ))
            .addReg(Divisor.getReg(), getKillRegState(Divisor.isKill()))
            .addReg(Mips::ZERO)
            .addImm(7);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1299-1301
```cpp
  // Use the 32-bit sub-register if this is a 64-bit division.
  if (Is64Bit)
    MIB->getOperand(0).setSubReg(Mips::sub_32);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1303-1304
```cpp
  // Clear Divisor's kill flag.
  Divisor.setIsKill(false);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1306-1307
```cpp
  // We would normally delete the original instruction here but in this case
  // we only needed to inject an additional instruction rather than replace it.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 1309-1310
```cpp
  return &MBB;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1312-1325
```cpp
MachineBasicBlock *
MipsTargetLowering::EmitInstrWithCustomInserter(MachineInstr &MI,
                                                MachineBasicBlock *BB) const {
  switch (MI.getOpcode()) {
  default:
    llvm_unreachable("Unexpected instr type to insert");
  case Mips::ATOMIC_LOAD_ADD_I8:
    return emitAtomicBinaryPartword(MI, BB, 1);
  case Mips::ATOMIC_LOAD_ADD_I16:
    return emitAtomicBinaryPartword(MI, BB, 2);
  case Mips::ATOMIC_LOAD_ADD_I32:
    return emitAtomicBinary(MI, BB);
  case Mips::ATOMIC_LOAD_ADD_I64:
    return emitAtomicBinary(MI, BB);
```
- EN: Implements `MipsTargetLowering::EmitInstrWithCustomInserter`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::EmitInstrWithCustomInserter`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1327-1334
```cpp
  case Mips::ATOMIC_LOAD_AND_I8:
    return emitAtomicBinaryPartword(MI, BB, 1);
  case Mips::ATOMIC_LOAD_AND_I16:
    return emitAtomicBinaryPartword(MI, BB, 2);
  case Mips::ATOMIC_LOAD_AND_I32:
    return emitAtomicBinary(MI, BB);
  case Mips::ATOMIC_LOAD_AND_I64:
    return emitAtomicBinary(MI, BB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1336-1343
```cpp
  case Mips::ATOMIC_LOAD_OR_I8:
    return emitAtomicBinaryPartword(MI, BB, 1);
  case Mips::ATOMIC_LOAD_OR_I16:
    return emitAtomicBinaryPartword(MI, BB, 2);
  case Mips::ATOMIC_LOAD_OR_I32:
    return emitAtomicBinary(MI, BB);
  case Mips::ATOMIC_LOAD_OR_I64:
    return emitAtomicBinary(MI, BB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1345-1352
```cpp
  case Mips::ATOMIC_LOAD_XOR_I8:
    return emitAtomicBinaryPartword(MI, BB, 1);
  case Mips::ATOMIC_LOAD_XOR_I16:
    return emitAtomicBinaryPartword(MI, BB, 2);
  case Mips::ATOMIC_LOAD_XOR_I32:
    return emitAtomicBinary(MI, BB);
  case Mips::ATOMIC_LOAD_XOR_I64:
    return emitAtomicBinary(MI, BB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1354-1361
```cpp
  case Mips::ATOMIC_LOAD_NAND_I8:
    return emitAtomicBinaryPartword(MI, BB, 1);
  case Mips::ATOMIC_LOAD_NAND_I16:
    return emitAtomicBinaryPartword(MI, BB, 2);
  case Mips::ATOMIC_LOAD_NAND_I32:
    return emitAtomicBinary(MI, BB);
  case Mips::ATOMIC_LOAD_NAND_I64:
    return emitAtomicBinary(MI, BB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1363-1370
```cpp
  case Mips::ATOMIC_LOAD_SUB_I8:
    return emitAtomicBinaryPartword(MI, BB, 1);
  case Mips::ATOMIC_LOAD_SUB_I16:
    return emitAtomicBinaryPartword(MI, BB, 2);
  case Mips::ATOMIC_LOAD_SUB_I32:
    return emitAtomicBinary(MI, BB);
  case Mips::ATOMIC_LOAD_SUB_I64:
    return emitAtomicBinary(MI, BB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1372-1379
```cpp
  case Mips::ATOMIC_SWAP_I8:
    return emitAtomicBinaryPartword(MI, BB, 1);
  case Mips::ATOMIC_SWAP_I16:
    return emitAtomicBinaryPartword(MI, BB, 2);
  case Mips::ATOMIC_SWAP_I32:
    return emitAtomicBinary(MI, BB);
  case Mips::ATOMIC_SWAP_I64:
    return emitAtomicBinary(MI, BB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1381-1388
```cpp
  case Mips::ATOMIC_CMP_SWAP_I8:
    return emitAtomicCmpSwapPartword(MI, BB, 1);
  case Mips::ATOMIC_CMP_SWAP_I16:
    return emitAtomicCmpSwapPartword(MI, BB, 2);
  case Mips::ATOMIC_CMP_SWAP_I32:
    return emitAtomicCmpSwap(MI, BB);
  case Mips::ATOMIC_CMP_SWAP_I64:
    return emitAtomicCmpSwap(MI, BB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1390-1397
```cpp
  case Mips::ATOMIC_LOAD_MIN_I8:
    return emitAtomicBinaryPartword(MI, BB, 1);
  case Mips::ATOMIC_LOAD_MIN_I16:
    return emitAtomicBinaryPartword(MI, BB, 2);
  case Mips::ATOMIC_LOAD_MIN_I32:
    return emitAtomicBinary(MI, BB);
  case Mips::ATOMIC_LOAD_MIN_I64:
    return emitAtomicBinary(MI, BB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1399-1406
```cpp
  case Mips::ATOMIC_LOAD_MAX_I8:
    return emitAtomicBinaryPartword(MI, BB, 1);
  case Mips::ATOMIC_LOAD_MAX_I16:
    return emitAtomicBinaryPartword(MI, BB, 2);
  case Mips::ATOMIC_LOAD_MAX_I32:
    return emitAtomicBinary(MI, BB);
  case Mips::ATOMIC_LOAD_MAX_I64:
    return emitAtomicBinary(MI, BB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1408-1415
```cpp
  case Mips::ATOMIC_LOAD_UMIN_I8:
    return emitAtomicBinaryPartword(MI, BB, 1);
  case Mips::ATOMIC_LOAD_UMIN_I16:
    return emitAtomicBinaryPartword(MI, BB, 2);
  case Mips::ATOMIC_LOAD_UMIN_I32:
    return emitAtomicBinary(MI, BB);
  case Mips::ATOMIC_LOAD_UMIN_I64:
    return emitAtomicBinary(MI, BB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1417-1424
```cpp
  case Mips::ATOMIC_LOAD_UMAX_I8:
    return emitAtomicBinaryPartword(MI, BB, 1);
  case Mips::ATOMIC_LOAD_UMAX_I16:
    return emitAtomicBinaryPartword(MI, BB, 2);
  case Mips::ATOMIC_LOAD_UMAX_I32:
    return emitAtomicBinary(MI, BB);
  case Mips::ATOMIC_LOAD_UMAX_I64:
    return emitAtomicBinary(MI, BB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1426-1440
```cpp
  case Mips::PseudoSDIV:
  case Mips::PseudoUDIV:
  case Mips::DIV:
  case Mips::DIVU:
  case Mips::MOD:
  case Mips::MODU:
    return insertDivByZeroTrap(MI, *BB, *Subtarget.getInstrInfo(), false,
                               false);
  case Mips::SDIV_MM_Pseudo:
  case Mips::UDIV_MM_Pseudo:
  case Mips::SDIV_MM:
  case Mips::UDIV_MM:
  case Mips::DIV_MMR6:
  case Mips::DIVU_MMR6:
  case Mips::MOD_MMR6:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1441-1449
```cpp
  case Mips::MODU_MMR6:
    return insertDivByZeroTrap(MI, *BB, *Subtarget.getInstrInfo(), false, true);
  case Mips::PseudoDSDIV:
  case Mips::PseudoDUDIV:
  case Mips::DDIV:
  case Mips::DDIVU:
  case Mips::DMOD:
  case Mips::DMODU:
    return insertDivByZeroTrap(MI, *BB, *Subtarget.getInstrInfo(), true, false);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1451-1465
```cpp
  case Mips::PseudoSELECT_I:
  case Mips::PseudoSELECT_I64:
  case Mips::PseudoSELECT_S:
  case Mips::PseudoSELECT_D32:
  case Mips::PseudoSELECT_D64:
    return emitPseudoSELECT(MI, BB, false, Mips::BNE);
  case Mips::PseudoSELECTFP_F_I:
  case Mips::PseudoSELECTFP_F_I64:
  case Mips::PseudoSELECTFP_F_S:
  case Mips::PseudoSELECTFP_F_D32:
  case Mips::PseudoSELECTFP_F_D64:
    return emitPseudoSELECT(MI, BB, true, Mips::BC1F);
  case Mips::PseudoSELECTFP_T_I:
  case Mips::PseudoSELECTFP_T_I64:
  case Mips::PseudoSELECTFP_T_S:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1466-1480
```cpp
  case Mips::PseudoSELECTFP_T_D32:
  case Mips::PseudoSELECTFP_T_D64:
    return emitPseudoSELECT(MI, BB, true, Mips::BC1T);
  case Mips::PseudoD_SELECT_I:
  case Mips::PseudoD_SELECT_I64:
    return emitPseudoD_SELECT(MI, BB);
  case Mips::LDR_W:
    return emitLDR_W(MI, BB);
  case Mips::LDR_D:
    return emitLDR_D(MI, BB);
  case Mips::STR_W:
    return emitSTR_W(MI, BB);
  case Mips::STR_D:
    return emitSTR_D(MI, BB);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1481-1481
```cpp
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1483-1487
```cpp
// This function also handles Mips::ATOMIC_SWAP_I32 (when BinOpcode == 0), and
// Mips::ATOMIC_LOAD_NAND_I32 (when Nand == true)
MachineBasicBlock *
MipsTargetLowering::emitAtomicBinary(MachineInstr &MI,
                                     MachineBasicBlock *BB) const {
```
- EN: Implements `MipsTargetLowering::emitAtomicBinary`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::emitAtomicBinary`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1489-1492
```cpp
  MachineFunction *MF = BB->getParent();
  MachineRegisterInfo &RegInfo = MF->getRegInfo();
  const TargetInstrInfo *TII = Subtarget.getInstrInfo();
  DebugLoc DL = MI.getDebugLoc();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1494-1508
```cpp
  unsigned AtomicOp;
  bool NeedsAdditionalReg = false;
  switch (MI.getOpcode()) {
  case Mips::ATOMIC_LOAD_ADD_I32:
    AtomicOp = Mips::ATOMIC_LOAD_ADD_I32_POSTRA;
    break;
  case Mips::ATOMIC_LOAD_SUB_I32:
    AtomicOp = Mips::ATOMIC_LOAD_SUB_I32_POSTRA;
    break;
  case Mips::ATOMIC_LOAD_AND_I32:
    AtomicOp = Mips::ATOMIC_LOAD_AND_I32_POSTRA;
    break;
  case Mips::ATOMIC_LOAD_OR_I32:
    AtomicOp = Mips::ATOMIC_LOAD_OR_I32_POSTRA;
    break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1509-1523
```cpp
  case Mips::ATOMIC_LOAD_XOR_I32:
    AtomicOp = Mips::ATOMIC_LOAD_XOR_I32_POSTRA;
    break;
  case Mips::ATOMIC_LOAD_NAND_I32:
    AtomicOp = Mips::ATOMIC_LOAD_NAND_I32_POSTRA;
    break;
  case Mips::ATOMIC_SWAP_I32:
    AtomicOp = Mips::ATOMIC_SWAP_I32_POSTRA;
    break;
  case Mips::ATOMIC_LOAD_ADD_I64:
    AtomicOp = Mips::ATOMIC_LOAD_ADD_I64_POSTRA;
    break;
  case Mips::ATOMIC_LOAD_SUB_I64:
    AtomicOp = Mips::ATOMIC_LOAD_SUB_I64_POSTRA;
    break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1524-1538
```cpp
  case Mips::ATOMIC_LOAD_AND_I64:
    AtomicOp = Mips::ATOMIC_LOAD_AND_I64_POSTRA;
    break;
  case Mips::ATOMIC_LOAD_OR_I64:
    AtomicOp = Mips::ATOMIC_LOAD_OR_I64_POSTRA;
    break;
  case Mips::ATOMIC_LOAD_XOR_I64:
    AtomicOp = Mips::ATOMIC_LOAD_XOR_I64_POSTRA;
    break;
  case Mips::ATOMIC_LOAD_NAND_I64:
    AtomicOp = Mips::ATOMIC_LOAD_NAND_I64_POSTRA;
    break;
  case Mips::ATOMIC_SWAP_I64:
    AtomicOp = Mips::ATOMIC_SWAP_I64_POSTRA;
    break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1539-1553
```cpp
  case Mips::ATOMIC_LOAD_MIN_I32:
    AtomicOp = Mips::ATOMIC_LOAD_MIN_I32_POSTRA;
    NeedsAdditionalReg = true;
    break;
  case Mips::ATOMIC_LOAD_MAX_I32:
    AtomicOp = Mips::ATOMIC_LOAD_MAX_I32_POSTRA;
    NeedsAdditionalReg = true;
    break;
  case Mips::ATOMIC_LOAD_UMIN_I32:
    AtomicOp = Mips::ATOMIC_LOAD_UMIN_I32_POSTRA;
    NeedsAdditionalReg = true;
    break;
  case Mips::ATOMIC_LOAD_UMAX_I32:
    AtomicOp = Mips::ATOMIC_LOAD_UMAX_I32_POSTRA;
    NeedsAdditionalReg = true;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1554-1568
```cpp
    break;
  case Mips::ATOMIC_LOAD_MIN_I64:
    AtomicOp = Mips::ATOMIC_LOAD_MIN_I64_POSTRA;
    NeedsAdditionalReg = true;
    break;
  case Mips::ATOMIC_LOAD_MAX_I64:
    AtomicOp = Mips::ATOMIC_LOAD_MAX_I64_POSTRA;
    NeedsAdditionalReg = true;
    break;
  case Mips::ATOMIC_LOAD_UMIN_I64:
    AtomicOp = Mips::ATOMIC_LOAD_UMIN_I64_POSTRA;
    NeedsAdditionalReg = true;
    break;
  case Mips::ATOMIC_LOAD_UMAX_I64:
    AtomicOp = Mips::ATOMIC_LOAD_UMAX_I64_POSTRA;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1569-1573
```cpp
    NeedsAdditionalReg = true;
    break;
  default:
    llvm_unreachable("Unknown pseudo atomic for replacement!");
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1575-1578
```cpp
  Register OldVal = MI.getOperand(0).getReg();
  Register Ptr = MI.getOperand(1).getReg();
  Register Incr = MI.getOperand(2).getReg();
  Register Scratch = RegInfo.createVirtualRegister(RegInfo.getRegClass(OldVal));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1580-1580
```cpp
  MachineBasicBlock::iterator II(MI);
```
- EN: Declares `II`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `II`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1582-1586
```cpp
  // The scratch registers here with the EarlyClobber | Define | Implicit
  // flags is used to persuade the register allocator and the machine
  // verifier to accept the usage of this register. This has to be a real
  // register which has an UNDEF value but is dead after the instruction which
  // is unique among the registers chosen for the instruction.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 1588-1596
```cpp
  // The EarlyClobber flag has the semantic properties that the operand it is
  // attached to is clobbered before the rest of the inputs are read. Hence it
  // must be unique among the operands to the instruction.
  // The Define flag is needed to coerce the machine verifier that an Undef
  // value isn't a problem.
  // The Dead flag is needed as the value in scratch isn't used by any other
  // instruction. Kill isn't used as Dead is more precise.
  // The implicit flag is here due to the interaction between the other flags
  // and the machine verifier.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 1598-1612
```cpp
  // For correctness purpose, a new pseudo is introduced here. We need this
  // new pseudo, so that FastRegisterAllocator does not see an ll/sc sequence
  // that is spread over >1 basic blocks. A register allocator which
  // introduces (or any codegen infact) a store, can violate the expectations
  // of the hardware.
  //
  // An atomic read-modify-write sequence starts with a linked load
  // instruction and ends with a store conditional instruction. The atomic
  // read-modify-write sequence fails if any of the following conditions
  // occur between the execution of ll and sc:
  //   * A coherent store is completed by another process or coherent I/O
  //     module into the block of synchronizable physical memory containing
  //     the word. The size and alignment of the block is
  //     implementation-dependent.
  //   * A coherent store is executed between an LL and SC sequence on the
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 1613-1615
```cpp
  //     same processor to the block of synchornizable physical memory
  //     containing the word.
  //
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 1617-1618
```cpp
  Register PtrCopy = RegInfo.createVirtualRegister(RegInfo.getRegClass(Ptr));
  Register IncrCopy = RegInfo.createVirtualRegister(RegInfo.getRegClass(Incr));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1620-1621
```cpp
  BuildMI(*BB, II, DL, TII->get(Mips::COPY), IncrCopy).addReg(Incr);
  BuildMI(*BB, II, DL, TII->get(Mips::COPY), PtrCopy).addReg(Ptr);
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1623-1635
```cpp
  MachineInstrBuilder MIB =
      BuildMI(*BB, II, DL, TII->get(AtomicOp))
          .addReg(OldVal, RegState::Define | RegState::EarlyClobber)
          .addReg(PtrCopy)
          .addReg(IncrCopy)
          .addReg(Scratch, RegState::Define | RegState::EarlyClobber |
                               RegState::Implicit | RegState::Dead);
  if (NeedsAdditionalReg) {
    Register Scratch2 =
        RegInfo.createVirtualRegister(RegInfo.getRegClass(OldVal));
    MIB.addReg(Scratch2, RegState::Define | RegState::EarlyClobber |
                             RegState::Implicit | RegState::Dead);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1637-1637
```cpp
  MI.eraseFromParent();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1639-1640
```cpp
  return BB;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1642-1646
```cpp
MachineBasicBlock *MipsTargetLowering::emitSignExtendToI32InReg(
    MachineInstr &MI, MachineBasicBlock *BB, unsigned Size, unsigned DstReg,
    unsigned SrcReg) const {
  const TargetInstrInfo *TII = Subtarget.getInstrInfo();
  const DebugLoc &DL = MI.getDebugLoc();
```
- EN: Implements `MipsTargetLowering::emitSignExtendToI32InReg`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::emitSignExtendToI32InReg`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1648-1651
```cpp
  if (Subtarget.hasMips32r2() && Size == 1) {
    BuildMI(BB, DL, TII->get(Mips::SEB), DstReg).addReg(SrcReg);
    return BB;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1653-1656
```cpp
  if (Subtarget.hasMips32r2() && Size == 2) {
    BuildMI(BB, DL, TII->get(Mips::SEH), DstReg).addReg(SrcReg);
    return BB;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1658-1661
```cpp
  MachineFunction *MF = BB->getParent();
  MachineRegisterInfo &RegInfo = MF->getRegInfo();
  const TargetRegisterClass *RC = getRegClassFor(MVT::i32);
  Register ScrReg = RegInfo.createVirtualRegister(RC);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1663-1664
```cpp
  assert(Size < 32);
  int64_t ShiftImm = 32 - (Size * 8);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1666-1667
```cpp
  BuildMI(BB, DL, TII->get(Mips::SLL), ScrReg).addReg(SrcReg).addImm(ShiftImm);
  BuildMI(BB, DL, TII->get(Mips::SRA), DstReg).addReg(ScrReg).addImm(ShiftImm);
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1669-1670
```cpp
  return BB;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1672-1675
```cpp
MachineBasicBlock *MipsTargetLowering::emitAtomicBinaryPartword(
    MachineInstr &MI, MachineBasicBlock *BB, unsigned Size) const {
  assert((Size == 1 || Size == 2) &&
         "Unsupported size for EmitAtomicBinaryPartial.");
```
- EN: Implements `MipsTargetLowering::emitAtomicBinaryPartword`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::emitAtomicBinaryPartword`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1677-1684
```cpp
  MachineFunction *MF = BB->getParent();
  MachineRegisterInfo &RegInfo = MF->getRegInfo();
  const TargetRegisterClass *RC = getRegClassFor(MVT::i32);
  const bool ArePtrs64bit = ABI.ArePtrs64bit();
  const TargetRegisterClass *RCp =
    getRegClassFor(ArePtrs64bit ? MVT::i64 : MVT::i32);
  const TargetInstrInfo *TII = Subtarget.getInstrInfo();
  DebugLoc DL = MI.getDebugLoc();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1686-1688
```cpp
  Register Dest = MI.getOperand(0).getReg();
  Register Ptr = MI.getOperand(1).getReg();
  Register Incr = MI.getOperand(2).getReg();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1690-1700
```cpp
  Register AlignedAddr = RegInfo.createVirtualRegister(RCp);
  Register ShiftAmt = RegInfo.createVirtualRegister(RC);
  Register Mask = RegInfo.createVirtualRegister(RC);
  Register Mask2 = RegInfo.createVirtualRegister(RC);
  Register Incr2 = RegInfo.createVirtualRegister(RC);
  Register MaskLSB2 = RegInfo.createVirtualRegister(RCp);
  Register PtrLSB2 = RegInfo.createVirtualRegister(RC);
  Register MaskUpper = RegInfo.createVirtualRegister(RC);
  Register Scratch = RegInfo.createVirtualRegister(RC);
  Register Scratch2 = RegInfo.createVirtualRegister(RC);
  Register Scratch3 = RegInfo.createVirtualRegister(RC);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1702-1716
```cpp
  unsigned AtomicOp = 0;
  bool NeedsAdditionalReg = false;
  switch (MI.getOpcode()) {
  case Mips::ATOMIC_LOAD_NAND_I8:
    AtomicOp = Mips::ATOMIC_LOAD_NAND_I8_POSTRA;
    break;
  case Mips::ATOMIC_LOAD_NAND_I16:
    AtomicOp = Mips::ATOMIC_LOAD_NAND_I16_POSTRA;
    break;
  case Mips::ATOMIC_SWAP_I8:
    AtomicOp = Mips::ATOMIC_SWAP_I8_POSTRA;
    break;
  case Mips::ATOMIC_SWAP_I16:
    AtomicOp = Mips::ATOMIC_SWAP_I16_POSTRA;
    break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1717-1731
```cpp
  case Mips::ATOMIC_LOAD_ADD_I8:
    AtomicOp = Mips::ATOMIC_LOAD_ADD_I8_POSTRA;
    break;
  case Mips::ATOMIC_LOAD_ADD_I16:
    AtomicOp = Mips::ATOMIC_LOAD_ADD_I16_POSTRA;
    break;
  case Mips::ATOMIC_LOAD_SUB_I8:
    AtomicOp = Mips::ATOMIC_LOAD_SUB_I8_POSTRA;
    break;
  case Mips::ATOMIC_LOAD_SUB_I16:
    AtomicOp = Mips::ATOMIC_LOAD_SUB_I16_POSTRA;
    break;
  case Mips::ATOMIC_LOAD_AND_I8:
    AtomicOp = Mips::ATOMIC_LOAD_AND_I8_POSTRA;
    break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1732-1746
```cpp
  case Mips::ATOMIC_LOAD_AND_I16:
    AtomicOp = Mips::ATOMIC_LOAD_AND_I16_POSTRA;
    break;
  case Mips::ATOMIC_LOAD_OR_I8:
    AtomicOp = Mips::ATOMIC_LOAD_OR_I8_POSTRA;
    break;
  case Mips::ATOMIC_LOAD_OR_I16:
    AtomicOp = Mips::ATOMIC_LOAD_OR_I16_POSTRA;
    break;
  case Mips::ATOMIC_LOAD_XOR_I8:
    AtomicOp = Mips::ATOMIC_LOAD_XOR_I8_POSTRA;
    break;
  case Mips::ATOMIC_LOAD_XOR_I16:
    AtomicOp = Mips::ATOMIC_LOAD_XOR_I16_POSTRA;
    break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1747-1761
```cpp
  case Mips::ATOMIC_LOAD_MIN_I8:
    AtomicOp = Mips::ATOMIC_LOAD_MIN_I8_POSTRA;
    NeedsAdditionalReg = true;
    break;
  case Mips::ATOMIC_LOAD_MIN_I16:
    AtomicOp = Mips::ATOMIC_LOAD_MIN_I16_POSTRA;
    NeedsAdditionalReg = true;
    break;
  case Mips::ATOMIC_LOAD_MAX_I8:
    AtomicOp = Mips::ATOMIC_LOAD_MAX_I8_POSTRA;
    NeedsAdditionalReg = true;
    break;
  case Mips::ATOMIC_LOAD_MAX_I16:
    AtomicOp = Mips::ATOMIC_LOAD_MAX_I16_POSTRA;
    NeedsAdditionalReg = true;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1762-1776
```cpp
    break;
  case Mips::ATOMIC_LOAD_UMIN_I8:
    AtomicOp = Mips::ATOMIC_LOAD_UMIN_I8_POSTRA;
    NeedsAdditionalReg = true;
    break;
  case Mips::ATOMIC_LOAD_UMIN_I16:
    AtomicOp = Mips::ATOMIC_LOAD_UMIN_I16_POSTRA;
    NeedsAdditionalReg = true;
    break;
  case Mips::ATOMIC_LOAD_UMAX_I8:
    AtomicOp = Mips::ATOMIC_LOAD_UMAX_I8_POSTRA;
    NeedsAdditionalReg = true;
    break;
  case Mips::ATOMIC_LOAD_UMAX_I16:
    AtomicOp = Mips::ATOMIC_LOAD_UMAX_I16_POSTRA;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1777-1781
```cpp
    NeedsAdditionalReg = true;
    break;
  default:
    llvm_unreachable("Unknown subword atomic pseudo for expansion!");
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1783-1787
```cpp
  // insert new blocks after the current block
  const BasicBlock *LLVM_BB = BB->getBasicBlock();
  MachineBasicBlock *exitMBB = MF->CreateMachineBasicBlock(LLVM_BB);
  MachineFunction::iterator It = ++BB->getIterator();
  MF->insert(It, exitMBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1789-1792
```cpp
  // Transfer the remainder of BB and its successor edges to exitMBB.
  exitMBB->splice(exitMBB->begin(), BB,
                  std::next(MachineBasicBlock::iterator(MI)), BB->end());
  exitMBB->transferSuccessorsAndUpdatePHIs(BB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1794-1794
```cpp
  BB->addSuccessor(exitMBB, BranchProbability::getOne());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1796-1804
```cpp
  //  thisMBB:
  //    addiu   masklsb2,$0,-4                # 0xfffffffc
  //    and     alignedaddr,ptr,masklsb2
  //    andi    ptrlsb2,ptr,3
  //    sll     shiftamt,ptrlsb2,3
  //    ori     maskupper,$0,255               # 0xff
  //    sll     mask,maskupper,shiftamt
  //    nor     mask2,$0,mask
  //    sll     incr2,incr,shiftamt
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 1806-1820
```cpp
  int64_t MaskImm = (Size == 1) ? 255 : 65535;
  BuildMI(BB, DL, TII->get(ABI.GetPtrAddiuOp()), MaskLSB2)
    .addReg(ABI.GetNullPtr()).addImm(-4);
  BuildMI(BB, DL, TII->get(ABI.GetPtrAndOp()), AlignedAddr)
    .addReg(Ptr).addReg(MaskLSB2);
  BuildMI(BB, DL, TII->get(Mips::ANDi), PtrLSB2)
      .addReg(Ptr, {}, ArePtrs64bit ? Mips::sub_32 : 0)
      .addImm(3);
  if (Subtarget.isLittle()) {
    BuildMI(BB, DL, TII->get(Mips::SLL), ShiftAmt).addReg(PtrLSB2).addImm(3);
  } else {
    Register Off = RegInfo.createVirtualRegister(RC);
    BuildMI(BB, DL, TII->get(Mips::XORi), Off)
      .addReg(PtrLSB2).addImm((Size == 1) ? 3 : 2);
    BuildMI(BB, DL, TII->get(Mips::SLL), ShiftAmt).addReg(Off).addImm(3);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1821-1827
```cpp
  }
  BuildMI(BB, DL, TII->get(Mips::ORi), MaskUpper)
    .addReg(Mips::ZERO).addImm(MaskImm);
  BuildMI(BB, DL, TII->get(Mips::SLLV), Mask)
    .addReg(MaskUpper).addReg(ShiftAmt);
  BuildMI(BB, DL, TII->get(Mips::NOR), Mask2).addReg(Mips::ZERO).addReg(Mask);
  BuildMI(BB, DL, TII->get(Mips::SLLV), Incr2).addReg(Incr).addReg(ShiftAmt);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1830-1832
```cpp
  // The purposes of the flags on the scratch registers is explained in
  // emitAtomicBinary. In summary, we need a scratch register which is going to
  // be undef, that is unique among registers chosen for the instruction.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 1834-1848
```cpp
  MachineInstrBuilder MIB =
      BuildMI(BB, DL, TII->get(AtomicOp))
          .addReg(Dest, RegState::Define | RegState::EarlyClobber)
          .addReg(AlignedAddr)
          .addReg(Incr2)
          .addReg(Mask)
          .addReg(Mask2)
          .addReg(ShiftAmt)
          .addReg(Scratch, RegState::EarlyClobber | RegState::Define |
                               RegState::Dead | RegState::Implicit)
          .addReg(Scratch2, RegState::EarlyClobber | RegState::Define |
                                RegState::Dead | RegState::Implicit)
          .addReg(Scratch3, RegState::EarlyClobber | RegState::Define |
                                RegState::Dead | RegState::Implicit);
  if (NeedsAdditionalReg) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1849-1852
```cpp
    Register Scratch4 = RegInfo.createVirtualRegister(RC);
    MIB.addReg(Scratch4, RegState::EarlyClobber | RegState::Define |
                             RegState::Dead | RegState::Implicit);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1854-1854
```cpp
  MI.eraseFromParent(); // The instruction is gone now.
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1856-1857
```cpp
  return exitMBB;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1859-1862
```cpp
// Lower atomic compare and swap to a pseudo instruction, taking care to
// define a scratch register for the pseudo instruction's expansion. The
// instruction is expanded after the register allocator as to prevent
// the insertion of stores between the linked load and the store conditional.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 1864-1866
```cpp
MachineBasicBlock *
MipsTargetLowering::emitAtomicCmpSwap(MachineInstr &MI,
                                      MachineBasicBlock *BB) const {
```
- EN: Implements `MipsTargetLowering::emitAtomicCmpSwap`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::emitAtomicCmpSwap`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1868-1870
```cpp
  assert((MI.getOpcode() == Mips::ATOMIC_CMP_SWAP_I32 ||
          MI.getOpcode() == Mips::ATOMIC_CMP_SWAP_I64) &&
         "Unsupported atomic pseudo for EmitAtomicCmpSwap.");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1872-1872
```cpp
  const unsigned Size = MI.getOpcode() == Mips::ATOMIC_CMP_SWAP_I32 ? 4 : 8;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1874-1878
```cpp
  MachineFunction *MF = BB->getParent();
  MachineRegisterInfo &MRI = MF->getRegInfo();
  const TargetRegisterClass *RC = getRegClassFor(MVT::getIntegerVT(Size * 8));
  const TargetInstrInfo *TII = Subtarget.getInstrInfo();
  DebugLoc DL = MI.getDebugLoc();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1880-1886
```cpp
  unsigned AtomicOp = MI.getOpcode() == Mips::ATOMIC_CMP_SWAP_I32
                          ? Mips::ATOMIC_CMP_SWAP_I32_POSTRA
                          : Mips::ATOMIC_CMP_SWAP_I64_POSTRA;
  Register Dest = MI.getOperand(0).getReg();
  Register Ptr = MI.getOperand(1).getReg();
  Register OldVal = MI.getOperand(2).getReg();
  Register NewVal = MI.getOperand(3).getReg();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1888-1889
```cpp
  Register Scratch = MRI.createVirtualRegister(RC);
  MachineBasicBlock::iterator II(MI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1891-1894
```cpp
  // We need to create copies of the various registers and kill them at the
  // atomic pseudo. If the copies are not made, when the atomic is expanded
  // after fast register allocation, the spills will end up outside of the
  // blocks that their values are defined in, causing livein errors.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 1896-1898
```cpp
  Register PtrCopy = MRI.createVirtualRegister(MRI.getRegClass(Ptr));
  Register OldValCopy = MRI.createVirtualRegister(MRI.getRegClass(OldVal));
  Register NewValCopy = MRI.createVirtualRegister(MRI.getRegClass(NewVal));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1900-1902
```cpp
  BuildMI(*BB, II, DL, TII->get(Mips::COPY), PtrCopy).addReg(Ptr);
  BuildMI(*BB, II, DL, TII->get(Mips::COPY), OldValCopy).addReg(OldVal);
  BuildMI(*BB, II, DL, TII->get(Mips::COPY), NewValCopy).addReg(NewVal);
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1904-1906
```cpp
  // The purposes of the flags on the scratch registers is explained in
  // emitAtomicBinary. In summary, we need a scratch register which is going to
  // be undef, that is unique among registers chosen for the instruction.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 1908-1914
```cpp
  BuildMI(*BB, II, DL, TII->get(AtomicOp))
      .addReg(Dest, RegState::Define | RegState::EarlyClobber)
      .addReg(PtrCopy, RegState::Kill)
      .addReg(OldValCopy, RegState::Kill)
      .addReg(NewValCopy, RegState::Kill)
      .addReg(Scratch, RegState::EarlyClobber | RegState::Define |
                           RegState::Dead | RegState::Implicit);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1916-1916
```cpp
  MI.eraseFromParent(); // The instruction is gone now.
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1918-1919
```cpp
  return BB;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1921-1924
```cpp
MachineBasicBlock *MipsTargetLowering::emitAtomicCmpSwapPartword(
    MachineInstr &MI, MachineBasicBlock *BB, unsigned Size) const {
  assert((Size == 1 || Size == 2) &&
      "Unsupported size for EmitAtomicCmpSwapPartial.");
```
- EN: Implements `MipsTargetLowering::emitAtomicCmpSwapPartword`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::emitAtomicCmpSwapPartword`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1926-1933
```cpp
  MachineFunction *MF = BB->getParent();
  MachineRegisterInfo &RegInfo = MF->getRegInfo();
  const TargetRegisterClass *RC = getRegClassFor(MVT::i32);
  const bool ArePtrs64bit = ABI.ArePtrs64bit();
  const TargetRegisterClass *RCp =
    getRegClassFor(ArePtrs64bit ? MVT::i64 : MVT::i32);
  const TargetInstrInfo *TII = Subtarget.getInstrInfo();
  DebugLoc DL = MI.getDebugLoc();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1935-1938
```cpp
  Register Dest = MI.getOperand(0).getReg();
  Register Ptr = MI.getOperand(1).getReg();
  Register CmpVal = MI.getOperand(2).getReg();
  Register NewVal = MI.getOperand(3).getReg();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1940-1953
```cpp
  Register AlignedAddr = RegInfo.createVirtualRegister(RCp);
  Register ShiftAmt = RegInfo.createVirtualRegister(RC);
  Register Mask = RegInfo.createVirtualRegister(RC);
  Register Mask2 = RegInfo.createVirtualRegister(RC);
  Register ShiftedCmpVal = RegInfo.createVirtualRegister(RC);
  Register ShiftedNewVal = RegInfo.createVirtualRegister(RC);
  Register MaskLSB2 = RegInfo.createVirtualRegister(RCp);
  Register PtrLSB2 = RegInfo.createVirtualRegister(RC);
  Register MaskUpper = RegInfo.createVirtualRegister(RC);
  Register MaskedCmpVal = RegInfo.createVirtualRegister(RC);
  Register MaskedNewVal = RegInfo.createVirtualRegister(RC);
  unsigned AtomicOp = MI.getOpcode() == Mips::ATOMIC_CMP_SWAP_I8
                          ? Mips::ATOMIC_CMP_SWAP_I8_POSTRA
                          : Mips::ATOMIC_CMP_SWAP_I16_POSTRA;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1955-1966
```cpp
  // The scratch registers here with the EarlyClobber | Define | Dead | Implicit
  // flags are used to coerce the register allocator and the machine verifier to
  // accept the usage of these registers.
  // The EarlyClobber flag has the semantic properties that the operand it is
  // attached to is clobbered before the rest of the inputs are read. Hence it
  // must be unique among the operands to the instruction.
  // The Define flag is needed to coerce the machine verifier that an Undef
  // value isn't a problem.
  // The Dead flag is needed as the value in scratch isn't used by any other
  // instruction. Kill isn't used as Dead is more precise.
  Register Scratch = RegInfo.createVirtualRegister(RC);
  Register Scratch2 = RegInfo.createVirtualRegister(RC);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1968-1972
```cpp
  // insert new blocks after the current block
  const BasicBlock *LLVM_BB = BB->getBasicBlock();
  MachineBasicBlock *exitMBB = MF->CreateMachineBasicBlock(LLVM_BB);
  MachineFunction::iterator It = ++BB->getIterator();
  MF->insert(It, exitMBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1974-1977
```cpp
  // Transfer the remainder of BB and its successor edges to exitMBB.
  exitMBB->splice(exitMBB->begin(), BB,
                  std::next(MachineBasicBlock::iterator(MI)), BB->end());
  exitMBB->transferSuccessorsAndUpdatePHIs(BB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1979-1979
```cpp
  BB->addSuccessor(exitMBB, BranchProbability::getOne());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1981-1995
```cpp
  //  thisMBB:
  //    addiu   masklsb2,$0,-4                # 0xfffffffc
  //    and     alignedaddr,ptr,masklsb2
  //    andi    ptrlsb2,ptr,3
  //    xori    ptrlsb2,ptrlsb2,3              # Only for BE
  //    sll     shiftamt,ptrlsb2,3
  //    ori     maskupper,$0,255               # 0xff
  //    sll     mask,maskupper,shiftamt
  //    nor     mask2,$0,mask
  //    andi    maskedcmpval,cmpval,255
  //    sll     shiftedcmpval,maskedcmpval,shiftamt
  //    andi    maskednewval,newval,255
  //    sll     shiftednewval,maskednewval,shiftamt
  int64_t MaskImm = (Size == 1) ? 255 : 65535;
  BuildMI(BB, DL, TII->get(ArePtrs64bit ? Mips::DADDiu : Mips::ADDiu), MaskLSB2)
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1996-2010
```cpp
    .addReg(ABI.GetNullPtr()).addImm(-4);
  BuildMI(BB, DL, TII->get(ArePtrs64bit ? Mips::AND64 : Mips::AND), AlignedAddr)
    .addReg(Ptr).addReg(MaskLSB2);
  BuildMI(BB, DL, TII->get(Mips::ANDi), PtrLSB2)
      .addReg(Ptr, {}, ArePtrs64bit ? Mips::sub_32 : 0)
      .addImm(3);
  if (Subtarget.isLittle()) {
    BuildMI(BB, DL, TII->get(Mips::SLL), ShiftAmt).addReg(PtrLSB2).addImm(3);
  } else {
    Register Off = RegInfo.createVirtualRegister(RC);
    BuildMI(BB, DL, TII->get(Mips::XORi), Off)
      .addReg(PtrLSB2).addImm((Size == 1) ? 3 : 2);
    BuildMI(BB, DL, TII->get(Mips::SLL), ShiftAmt).addReg(Off).addImm(3);
  }
  BuildMI(BB, DL, TII->get(Mips::ORi), MaskUpper)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2011-2022
```cpp
    .addReg(Mips::ZERO).addImm(MaskImm);
  BuildMI(BB, DL, TII->get(Mips::SLLV), Mask)
    .addReg(MaskUpper).addReg(ShiftAmt);
  BuildMI(BB, DL, TII->get(Mips::NOR), Mask2).addReg(Mips::ZERO).addReg(Mask);
  BuildMI(BB, DL, TII->get(Mips::ANDi), MaskedCmpVal)
    .addReg(CmpVal).addImm(MaskImm);
  BuildMI(BB, DL, TII->get(Mips::SLLV), ShiftedCmpVal)
    .addReg(MaskedCmpVal).addReg(ShiftAmt);
  BuildMI(BB, DL, TII->get(Mips::ANDi), MaskedNewVal)
    .addReg(NewVal).addImm(MaskImm);
  BuildMI(BB, DL, TII->get(Mips::SLLV), ShiftedNewVal)
    .addReg(MaskedNewVal).addReg(ShiftAmt);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2024-2026
```cpp
  // The purposes of the flags on the scratch registers are explained in
  // emitAtomicBinary. In summary, we need a scratch register which is going to
  // be undef, that is unique among the register chosen for the instruction.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 2028-2039
```cpp
  BuildMI(BB, DL, TII->get(AtomicOp))
      .addReg(Dest, RegState::Define | RegState::EarlyClobber)
      .addReg(AlignedAddr)
      .addReg(Mask)
      .addReg(ShiftedCmpVal)
      .addReg(Mask2)
      .addReg(ShiftedNewVal)
      .addReg(ShiftAmt)
      .addReg(Scratch, RegState::EarlyClobber | RegState::Define |
                           RegState::Dead | RegState::Implicit)
      .addReg(Scratch2, RegState::EarlyClobber | RegState::Define |
                            RegState::Dead | RegState::Implicit);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2041-2041
```cpp
  MI.eraseFromParent(); // The instruction is gone now.
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2043-2044
```cpp
  return exitMBB;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2046-2052
```cpp
SDValue MipsTargetLowering::lowerREADCYCLECOUNTER(SDValue Op,
                                                  SelectionDAG &DAG) const {
  SmallVector<SDValue, 3> Results;
  SDLoc DL(Op);
  MachineFunction &MF = DAG.getMachineFunction();
  unsigned RdhwrOpc, DestReg;
  EVT PtrVT = getPointerTy(DAG.getDataLayout());
```
- EN: Implements `MipsTargetLowering::lowerREADCYCLECOUNTER`, a lowering routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::lowerREADCYCLECOUNTER`，它是一个围绕机器函数状态展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2054-2068
```cpp
  if (PtrVT == MVT::i64) {
    RdhwrOpc = Mips::RDHWR64;
    DestReg = MF.getRegInfo().createVirtualRegister(getRegClassFor(MVT::i64));
    SDNode *Rdhwr = DAG.getMachineNode(RdhwrOpc, DL, MVT::i64, MVT::Glue,
                                       DAG.getRegister(Mips::HWR2, MVT::i32),
                                       DAG.getTargetConstant(0, DL, MVT::i32));
    SDValue Chain = DAG.getCopyToReg(DAG.getEntryNode(), DL, DestReg,
                                     SDValue(Rdhwr, 0), SDValue(Rdhwr, 1));
    SDValue ResNode =
        DAG.getCopyFromReg(Chain, DL, DestReg, MVT::i64, Chain.getValue(1));
    Results.push_back(ResNode);
    Results.push_back(ResNode.getValue(1));
  } else {
    RdhwrOpc = Mips::RDHWR;
    DestReg = MF.getRegInfo().createVirtualRegister(getRegClassFor(MVT::i32));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2069-2079
```cpp
    SDNode *Rdhwr = DAG.getMachineNode(RdhwrOpc, DL, MVT::i32, MVT::Glue,
                                       DAG.getRegister(Mips::HWR2, MVT::i32),
                                       DAG.getTargetConstant(0, DL, MVT::i32));
    SDValue Chain = DAG.getCopyToReg(DAG.getEntryNode(), DL, DestReg,
                                     SDValue(Rdhwr, 0), SDValue(Rdhwr, 1));
    SDValue ResNode =
        DAG.getCopyFromReg(Chain, DL, DestReg, MVT::i32, Chain.getValue(1));
    Results.push_back(DAG.getNode(ISD::BUILD_PAIR, DL, MVT::i64, ResNode,
                                  DAG.getConstant(0, DL, MVT::i32)));
    Results.push_back(ResNode.getValue(1));
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2081-2082
```cpp
  return DAG.getMergeValues(Results, DL);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2084-2089
```cpp
SDValue MipsTargetLowering::lowerBRCOND(SDValue Op, SelectionDAG &DAG) const {
  // The first operand is the chain, the second is the condition, the third is
  // the block to branch to if the condition is true.
  SDValue Chain = Op.getOperand(0);
  SDValue Dest = Op.getOperand(2);
  SDLoc DL(Op);
```
- EN: Implements `MipsTargetLowering::lowerBRCOND`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::lowerBRCOND`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2091-2092
```cpp
  assert(!Subtarget.hasMips32r6() && !Subtarget.hasMips64r6());
  SDValue CondRes = createFPCmp(DAG, Op.getOperand(1));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2094-2096
```cpp
  // Return if flag is not set by a floating point comparison.
  if (CondRes.getOpcode() != MipsISD::FPCmp)
    return Op;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2098-2105
```cpp
  SDValue CCNode  = CondRes.getOperand(2);
  Mips::CondCode CC = (Mips::CondCode)CCNode->getAsZExtVal();
  unsigned Opc = invertFPCondCodeUser(CC) ? Mips::BRANCH_F : Mips::BRANCH_T;
  SDValue BrCode = DAG.getConstant(Opc, DL, MVT::i32);
  SDValue FCC0 = DAG.getRegister(Mips::FCC0, MVT::i32);
  return DAG.getNode(MipsISD::FPBrcond, DL, Op.getValueType(), Chain, BrCode,
                     FCC0, Dest, CondRes);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2107-2111
```cpp
SDValue MipsTargetLowering::
lowerSELECT(SDValue Op, SelectionDAG &DAG) const
{
  assert(!Subtarget.hasMips32r6() && !Subtarget.hasMips64r6());
  SDValue Cond = createFPCmp(DAG, Op.getOperand(0));
```
- EN: Implements `lowerSELECT`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `lowerSELECT`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2113-2115
```cpp
  // Return if flag is not set by a floating point comparison.
  if (Cond.getOpcode() != MipsISD::FPCmp)
    return Op;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2117-2119
```cpp
  return createCMovFP(DAG, Cond, Op.getOperand(1), Op.getOperand(2),
                      SDLoc(Op));
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2121-2123
```cpp
SDValue MipsTargetLowering::lowerSETCC(SDValue Op, SelectionDAG &DAG) const {
  assert(!Subtarget.hasMips32r6() && !Subtarget.hasMips64r6());
  SDValue Cond = createFPCmp(DAG, Op);
```
- EN: Implements `MipsTargetLowering::lowerSETCC`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::lowerSETCC`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2125-2126
```cpp
  assert(Cond.getOpcode() == MipsISD::FPCmp &&
         "Floating point operand expected.");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2128-2130
```cpp
  SDLoc DL(Op);
  SDValue True  = DAG.getConstant(1, DL, MVT::i32);
  SDValue False = DAG.getConstant(0, DL, MVT::i32);
```
- EN: Declares `DL`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `DL`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2132-2133
```cpp
  return createCMovFP(DAG, Cond, True, False, DL);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2135-2136
```cpp
SDValue MipsTargetLowering::lowerFSETCC(SDValue Op, SelectionDAG &DAG) const {
  assert(!Subtarget.hasMips32r6() && !Subtarget.hasMips64r6());
```
- EN: Implements `MipsTargetLowering::lowerFSETCC`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::lowerFSETCC`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2138-2142
```cpp
  SDLoc DL(Op);
  SDValue Chain = Op.getOperand(0);
  SDValue LHS = Op.getOperand(1);
  SDValue RHS = Op.getOperand(2);
  ISD::CondCode CC = cast<CondCodeSDNode>(Op.getOperand(3))->get();
```
- EN: Declares `DL`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `DL`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2144-2148
```cpp
  SDValue Cond = DAG.getNode(MipsISD::FPCmp, DL, MVT::Glue, LHS, RHS,
                             DAG.getConstant(condCodeToFCC(CC), DL, MVT::i32));
  SDValue True = DAG.getConstant(1, DL, MVT::i32);
  SDValue False = DAG.getConstant(0, DL, MVT::i32);
  SDValue CMovFP = createCMovFP(DAG, Cond, True, False, DL);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2150-2151
```cpp
  return DAG.getMergeValues({CMovFP, Chain}, DL);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2153-2157
```cpp
SDValue MipsTargetLowering::lowerGlobalAddress(SDValue Op,
                                               SelectionDAG &DAG) const {
  EVT Ty = Op.getValueType();
  GlobalAddressSDNode *N = cast<GlobalAddressSDNode>(Op);
  const GlobalValue *GV = N->getGlobal();
```
- EN: Implements `MipsTargetLowering::lowerGlobalAddress`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::lowerGlobalAddress`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2159-2165
```cpp
  if (GV->hasDLLImportStorageClass()) {
    assert(Subtarget.isTargetWindows() &&
           "Windows is the only supported COFF target");
    return getDllimportVariable(
        N, SDLoc(N), Ty, DAG, DAG.getEntryNode(),
        MachinePointerInfo::getGOT(DAG.getMachineFunction()));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2167-2174
```cpp
  if (!isPositionIndependent()) {
    const MipsTargetObjectFile *TLOF =
        static_cast<const MipsTargetObjectFile *>(
            getTargetMachine().getObjFileLowering());
    const GlobalObject *GO = GV->getAliaseeObject();
    if (GO && TLOF->IsGlobalInSmallSection(GO, getTargetMachine()))
      // %gp_rel relocation
      return getAddrGPRel(N, SDLoc(N), Ty, DAG, ABI.IsN64());
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2176-2180
```cpp
                                // %hi/%lo relocation
    return Subtarget.hasSym32() ? getAddrNonPIC(N, SDLoc(N), Ty, DAG)
                                // %highest/%higher/%hi/%lo relocation
                                : getAddrNonPICSym64(N, SDLoc(N), Ty, DAG);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2182-2194
```cpp
  // Every other architecture would use shouldAssumeDSOLocal in here, but
  // mips is special.
  // * In PIC code mips requires got loads even for local statics!
  // * To save on got entries, for local statics the got entry contains the
  //   page and an additional add instruction takes care of the low bits.
  // * It is legal to access a hidden symbol with a non hidden undefined,
  //   so one cannot guarantee that all access to a hidden symbol will know
  //   it is hidden.
  // * Mips linkers don't support creating a page and a full got entry for
  //   the same symbol.
  // * Given all that, we have to use a full got entry for hidden symbols :-(
  if (GV->hasLocalLinkage())
    return getAddrLocal(N, SDLoc(N), Ty, DAG, ABI.IsN32() || ABI.IsN64());
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2196-2200
```cpp
  if (Subtarget.useXGOT())
    return getAddrGlobalLargeGOT(
        N, SDLoc(N), Ty, DAG, MipsII::MO_GOT_HI16, MipsII::MO_GOT_LO16,
        DAG.getEntryNode(),
        MachinePointerInfo::getGOT(DAG.getMachineFunction()));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2202-2206
```cpp
  return getAddrGlobal(
      N, SDLoc(N), Ty, DAG,
      (ABI.IsN32() || ABI.IsN64()) ? MipsII::MO_GOT_DISP : MipsII::MO_GOT,
      DAG.getEntryNode(), MachinePointerInfo::getGOT(DAG.getMachineFunction()));
}
```
- EN: Declares `getAddrGlobal`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getAddrGlobal`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2208-2211
```cpp
SDValue MipsTargetLowering::lowerBlockAddress(SDValue Op,
                                              SelectionDAG &DAG) const {
  BlockAddressSDNode *N = cast<BlockAddressSDNode>(Op);
  EVT Ty = Op.getValueType();
```
- EN: Implements `MipsTargetLowering::lowerBlockAddress`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::lowerBlockAddress`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2213-2215
```cpp
  if (!isPositionIndependent())
    return Subtarget.hasSym32() ? getAddrNonPIC(N, SDLoc(N), Ty, DAG)
                                : getAddrNonPICSym64(N, SDLoc(N), Ty, DAG);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2217-2218
```cpp
  return getAddrLocal(N, SDLoc(N), Ty, DAG, ABI.IsN32() || ABI.IsN64());
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2220-2225
```cpp
SDValue MipsTargetLowering::
lowerGlobalTLSAddress(SDValue Op, SelectionDAG &DAG) const
{
  // If the relocation model is PIC, use the General Dynamic TLS Model or
  // Local Dynamic TLS model, otherwise use the Initial Exec or
  // Local Exec TLS Model.
```
- EN: Implements `lowerGlobalTLSAddress`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `lowerGlobalTLSAddress`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2227-2229
```cpp
  GlobalAddressSDNode *GA = cast<GlobalAddressSDNode>(Op);
  if (DAG.getTarget().useEmulatedTLS())
    return LowerToTLSEmulatedModel(GA, DAG);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2231-2233
```cpp
  SDLoc DL(GA);
  const GlobalValue *GV = GA->getGlobal();
  EVT PtrVT = getPointerTy(DAG.getDataLayout());
```
- EN: Declares `DL`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `DL`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2235-2235
```cpp
  TLSModel::Model model = getTargetMachine().getTLSModel(GV);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2237-2240
```cpp
  if (model == TLSModel::GeneralDynamic || model == TLSModel::LocalDynamic) {
    // General Dynamic and Local Dynamic TLS Model.
    unsigned Flag = (model == TLSModel::LocalDynamic) ? MipsII::MO_TLSLDM
                                                      : MipsII::MO_TLSGD;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2242-2246
```cpp
    SDValue TGA = DAG.getTargetGlobalAddress(GV, DL, PtrVT, 0, Flag);
    SDValue Argument = DAG.getNode(MipsISD::Wrapper, DL, PtrVT,
                                   getGlobalReg(DAG, PtrVT), TGA);
    unsigned PtrSize = PtrVT.getSizeInBits();
    IntegerType *PtrTy = Type::getIntNTy(*DAG.getContext(), PtrSize);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2248-2248
```cpp
    SDValue TlsGetAddr = DAG.getExternalSymbol("__tls_get_addr", PtrVT);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2250-2251
```cpp
    ArgListTy Args;
    Args.emplace_back(Argument, PtrTy);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2253-2257
```cpp
    TargetLowering::CallLoweringInfo CLI(DAG);
    CLI.setDebugLoc(DL)
        .setChain(DAG.getEntryNode())
        .setLibCallee(CallingConv::C, PtrTy, TlsGetAddr, std::move(Args));
    std::pair<SDValue, SDValue> CallResult = LowerCallTo(CLI);
```
- EN: Declares `CLI`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `CLI`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2259-2259
```cpp
    SDValue Ret = CallResult.first;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2261-2262
```cpp
    if (model != TLSModel::LocalDynamic)
      return Ret;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2264-2272
```cpp
    SDValue TGAHi = DAG.getTargetGlobalAddress(GV, DL, PtrVT, 0,
                                               MipsII::MO_DTPREL_HI);
    SDValue Hi = DAG.getNode(MipsISD::TlsHi, DL, PtrVT, TGAHi);
    SDValue TGALo = DAG.getTargetGlobalAddress(GV, DL, PtrVT, 0,
                                               MipsII::MO_DTPREL_LO);
    SDValue Lo = DAG.getNode(MipsISD::Lo, DL, PtrVT, TGALo);
    SDValue Add = DAG.getNode(ISD::ADD, DL, PtrVT, Hi, Ret);
    return DAG.getNode(ISD::ADD, DL, PtrVT, Add, Lo);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2274-2288
```cpp
  SDValue Offset;
  if (model == TLSModel::InitialExec) {
    // Initial Exec TLS Model
    SDValue TGA = DAG.getTargetGlobalAddress(GV, DL, PtrVT, 0,
                                             MipsII::MO_GOTTPREL);
    TGA = DAG.getNode(MipsISD::Wrapper, DL, PtrVT, getGlobalReg(DAG, PtrVT),
                      TGA);
    Offset =
        DAG.getLoad(PtrVT, DL, DAG.getEntryNode(), TGA, MachinePointerInfo());
  } else {
    // Local Exec TLS Model
    assert(model == TLSModel::LocalExec);
    SDValue TGAHi = DAG.getTargetGlobalAddress(GV, DL, PtrVT, 0,
                                               MipsII::MO_TPREL_HI);
    SDValue TGALo = DAG.getTargetGlobalAddress(GV, DL, PtrVT, 0,
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2289-2293
```cpp
                                               MipsII::MO_TPREL_LO);
    SDValue Hi = DAG.getNode(MipsISD::TlsHi, DL, PtrVT, TGAHi);
    SDValue Lo = DAG.getNode(MipsISD::Lo, DL, PtrVT, TGALo);
    Offset = DAG.getNode(ISD::ADD, DL, PtrVT, Hi, Lo);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2295-2297
```cpp
  SDValue ThreadPointer = DAG.getNode(MipsISD::ThreadPointer, DL, PtrVT);
  return DAG.getNode(ISD::ADD, DL, PtrVT, ThreadPointer, Offset);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2299-2303
```cpp
SDValue MipsTargetLowering::
lowerJumpTable(SDValue Op, SelectionDAG &DAG) const
{
  JumpTableSDNode *N = cast<JumpTableSDNode>(Op);
  EVT Ty = Op.getValueType();
```
- EN: Implements `lowerJumpTable`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `lowerJumpTable`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2305-2307
```cpp
  if (!isPositionIndependent())
    return Subtarget.hasSym32() ? getAddrNonPIC(N, SDLoc(N), Ty, DAG)
                                : getAddrNonPICSym64(N, SDLoc(N), Ty, DAG);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2309-2310
```cpp
  return getAddrLocal(N, SDLoc(N), Ty, DAG, ABI.IsN32() || ABI.IsN64());
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2312-2316
```cpp
SDValue MipsTargetLowering::
lowerConstantPool(SDValue Op, SelectionDAG &DAG) const
{
  ConstantPoolSDNode *N = cast<ConstantPoolSDNode>(Op);
  EVT Ty = Op.getValueType();
```
- EN: Implements `lowerConstantPool`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `lowerConstantPool`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2318-2321
```cpp
  if (!isPositionIndependent()) {
    const MipsTargetObjectFile *TLOF =
        static_cast<const MipsTargetObjectFile *>(
            getTargetMachine().getObjFileLowering());
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2323-2326
```cpp
    if (TLOF->IsConstantInSmallSection(DAG.getDataLayout(), N->getConstVal(),
                                       getTargetMachine()))
      // %gp_rel relocation
      return getAddrGPRel(N, SDLoc(N), Ty, DAG, ABI.IsN64());
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2328-2330
```cpp
    return Subtarget.hasSym32() ? getAddrNonPIC(N, SDLoc(N), Ty, DAG)
                                : getAddrNonPICSym64(N, SDLoc(N), Ty, DAG);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2332-2333
```cpp
 return getAddrLocal(N, SDLoc(N), Ty, DAG, ABI.IsN32() || ABI.IsN64());
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2335-2337
```cpp
SDValue MipsTargetLowering::lowerVASTART(SDValue Op, SelectionDAG &DAG) const {
  MachineFunction &MF = DAG.getMachineFunction();
  MipsFunctionInfo *FuncInfo = MF.getInfo<MipsFunctionInfo>();
```
- EN: Implements `MipsTargetLowering::lowerVASTART`, a lowering routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::lowerVASTART`，它是一个围绕机器函数状态展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2339-2341
```cpp
  SDLoc DL(Op);
  SDValue FI = DAG.getFrameIndex(FuncInfo->getVarArgsFrameIndex(),
                                 getPointerTy(MF.getDataLayout()));
```
- EN: Declares `DL`, a target-specific routine centered on stack-frame access. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `DL`，它是一个围绕栈帧访问展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2343-2348
```cpp
  // vastart just stores the address of the VarArgsFrameIndex slot into the
  // memory location argument.
  const Value *SV = cast<SrcValueSDNode>(Op.getOperand(2))->getValue();
  return DAG.getStore(Op.getOperand(0), DL, FI, Op.getOperand(1),
                      MachinePointerInfo(SV));
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2350-2359
```cpp
SDValue MipsTargetLowering::lowerVAARG(SDValue Op, SelectionDAG &DAG) const {
  SDNode *Node = Op.getNode();
  EVT VT = Node->getValueType(0);
  SDValue Chain = Node->getOperand(0);
  SDValue VAListPtr = Node->getOperand(1);
  const Align Align =
      llvm::MaybeAlign(Node->getConstantOperandVal(3)).valueOrOne();
  const Value *SV = cast<SrcValueSDNode>(Node->getOperand(2))->getValue();
  SDLoc DL(Node);
  unsigned ArgSlotSizeInBytes = (ABI.IsN32() || ABI.IsN64()) ? 8 : 4;
```
- EN: Implements `MipsTargetLowering::lowerVAARG`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::lowerVAARG`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2361-2363
```cpp
  SDValue VAListLoad = DAG.getLoad(getPointerTy(DAG.getDataLayout()), DL, Chain,
                                   VAListPtr, MachinePointerInfo(SV));
  SDValue VAList = VAListLoad;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2365-2375
```cpp
  // Re-align the pointer if necessary.
  // It should only ever be necessary for 64-bit types on O32 since the minimum
  // argument alignment is the same as the maximum type alignment for N32/N64.
  //
  // FIXME: We currently align too often. The code generator doesn't notice
  //        when the pointer is still aligned from the last va_arg (or pair of
  //        va_args for the i64 on O32 case).
  if (Align > getMinStackArgumentAlignment()) {
    VAList = DAG.getNode(
        ISD::ADD, DL, VAList.getValueType(), VAList,
        DAG.getConstant(Align.value() - 1, DL, VAList.getValueType()));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2377-2380
```cpp
    VAList = DAG.getNode(ISD::AND, DL, VAList.getValueType(), VAList,
                         DAG.getSignedConstant(-(int64_t)Align.value(), DL,
                                               VAList.getValueType()));
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2382-2392
```cpp
  // Increment the pointer, VAList, to the next vaarg.
  auto &TD = DAG.getDataLayout();
  unsigned ArgSizeInBytes =
      TD.getTypeAllocSize(VT.getTypeForEVT(*DAG.getContext()));
  SDValue Tmp3 =
      DAG.getNode(ISD::ADD, DL, VAList.getValueType(), VAList,
                  DAG.getConstant(alignTo(ArgSizeInBytes, ArgSlotSizeInBytes),
                                  DL, VAList.getValueType()));
  // Store the incremented VAList to the legalized pointer
  Chain = DAG.getStore(VAListLoad.getValue(1), DL, Tmp3, VAListPtr,
                       MachinePointerInfo(SV));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2394-2406
```cpp
  // In big-endian mode we must adjust the pointer when the load size is smaller
  // than the argument slot size. We must also reduce the known alignment to
  // match. For example in the N64 ABI, we must add 4 bytes to the offset to get
  // the correct half of the slot, and reduce the alignment from 8 (slot
  // alignment) down to 4 (type alignment).
  if (!Subtarget.isLittle() && ArgSizeInBytes < ArgSlotSizeInBytes) {
    unsigned Adjustment = ArgSlotSizeInBytes - ArgSizeInBytes;
    VAList = DAG.getNode(ISD::ADD, DL, VAListPtr.getValueType(), VAList,
                         DAG.getIntPtrConstant(Adjustment, DL));
  }
  // Load the actual argument out of the pointer VAList
  return DAG.getLoad(VT, DL, Chain, VAList, MachinePointerInfo());
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2408-2415
```cpp
static SDValue lowerFCOPYSIGN32(SDValue Op, SelectionDAG &DAG,
                                bool HasExtractInsert) {
  EVT TyX = Op.getOperand(0).getValueType();
  EVT TyY = Op.getOperand(1).getValueType();
  SDLoc DL(Op);
  SDValue Const1 = DAG.getConstant(1, DL, MVT::i32);
  SDValue Const31 = DAG.getConstant(31, DL, MVT::i32);
  SDValue Res;
```
- EN: Implements `lowerFCOPYSIGN32`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `lowerFCOPYSIGN32`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2417-2426
```cpp
  // If operand is of type f64, extract the upper 32-bit. Otherwise, bitcast it
  // to i32.
  SDValue X = (TyX == MVT::f32) ?
    DAG.getNode(ISD::BITCAST, DL, MVT::i32, Op.getOperand(0)) :
    DAG.getNode(MipsISD::ExtractElementF64, DL, MVT::i32, Op.getOperand(0),
                Const1);
  SDValue Y = (TyY == MVT::f32) ?
    DAG.getNode(ISD::BITCAST, DL, MVT::i32, Op.getOperand(1)) :
    DAG.getNode(MipsISD::ExtractElementF64, DL, MVT::i32, Op.getOperand(1),
                Const1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2428-2442
```cpp
  if (HasExtractInsert) {
    // ext  E, Y, 31, 1  ; extract bit31 of Y
    // ins  X, E, 31, 1  ; insert extracted bit at bit31 of X
    SDValue E = DAG.getNode(MipsISD::Ext, DL, MVT::i32, Y, Const31, Const1);
    Res = DAG.getNode(MipsISD::Ins, DL, MVT::i32, E, Const31, Const1, X);
  } else {
    // sll SllX, X, 1
    // srl SrlX, SllX, 1
    // srl SrlY, Y, 31
    // sll SllY, SrlX, 31
    // or  Or, SrlX, SllY
    SDValue SllX = DAG.getNode(ISD::SHL, DL, MVT::i32, X, Const1);
    SDValue SrlX = DAG.getNode(ISD::SRL, DL, MVT::i32, SllX, Const1);
    SDValue SrlY = DAG.getNode(ISD::SRL, DL, MVT::i32, Y, Const31);
    SDValue SllY = DAG.getNode(ISD::SHL, DL, MVT::i32, SrlY, Const31);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2443-2444
```cpp
    Res = DAG.getNode(ISD::OR, DL, MVT::i32, SrlX, SllY);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2446-2447
```cpp
  if (TyX == MVT::f32)
    return DAG.getNode(ISD::BITCAST, DL, Op.getOperand(0).getValueType(), Res);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2449-2453
```cpp
  SDValue LowX = DAG.getNode(MipsISD::ExtractElementF64, DL, MVT::i32,
                             Op.getOperand(0),
                             DAG.getConstant(0, DL, MVT::i32));
  return DAG.getNode(MipsISD::BuildPairF64, DL, MVT::f64, LowX, Res);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2455-2461
```cpp
static SDValue lowerFCOPYSIGN64(SDValue Op, SelectionDAG &DAG,
                                bool HasExtractInsert) {
  unsigned WidthX = Op.getOperand(0).getValueSizeInBits();
  unsigned WidthY = Op.getOperand(1).getValueSizeInBits();
  EVT TyX = MVT::getIntegerVT(WidthX), TyY = MVT::getIntegerVT(WidthY);
  SDLoc DL(Op);
  SDValue Const1 = DAG.getConstant(1, DL, MVT::i32);
```
- EN: Implements `lowerFCOPYSIGN64`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `lowerFCOPYSIGN64`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2463-2465
```cpp
  // Bitcast to integer nodes.
  SDValue X = DAG.getNode(ISD::BITCAST, DL, TyX, Op.getOperand(0));
  SDValue Y = DAG.getNode(ISD::BITCAST, DL, TyY, Op.getOperand(1));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2467-2471
```cpp
  if (HasExtractInsert) {
    // ext  E, Y, width(Y) - 1, 1  ; extract bit width(Y)-1 of Y
    // ins  X, E, width(X) - 1, 1  ; insert extracted bit at bit width(X)-1 of X
    SDValue E = DAG.getNode(MipsISD::Ext, DL, TyY, Y,
                            DAG.getConstant(WidthY - 1, DL, MVT::i32), Const1);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2473-2476
```cpp
    if (WidthX > WidthY)
      E = DAG.getNode(ISD::ZERO_EXTEND, DL, TyX, E);
    else if (WidthY > WidthX)
      E = DAG.getNode(ISD::TRUNCATE, DL, TyX, E);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2478-2482
```cpp
    SDValue I = DAG.getNode(MipsISD::Ins, DL, TyX, E,
                            DAG.getConstant(WidthX - 1, DL, MVT::i32), Const1,
                            X);
    return DAG.getNode(ISD::BITCAST, DL, Op.getOperand(0).getValueType(), I);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2484-2492
```cpp
  // (d)sll SllX, X, 1
  // (d)srl SrlX, SllX, 1
  // (d)srl SrlY, Y, width(Y)-1
  // (d)sll SllY, SrlX, width(Y)-1
  // or     Or, SrlX, SllY
  SDValue SllX = DAG.getNode(ISD::SHL, DL, TyX, X, Const1);
  SDValue SrlX = DAG.getNode(ISD::SRL, DL, TyX, SllX, Const1);
  SDValue SrlY = DAG.getNode(ISD::SRL, DL, TyY, Y,
                             DAG.getConstant(WidthY - 1, DL, MVT::i32));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2494-2497
```cpp
  if (WidthX > WidthY)
    SrlY = DAG.getNode(ISD::ZERO_EXTEND, DL, TyX, SrlY);
  else if (WidthY > WidthX)
    SrlY = DAG.getNode(ISD::TRUNCATE, DL, TyX, SrlY);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2499-2503
```cpp
  SDValue SllY = DAG.getNode(ISD::SHL, DL, TyX, SrlY,
                             DAG.getConstant(WidthX - 1, DL, MVT::i32));
  SDValue Or = DAG.getNode(ISD::OR, DL, TyX, SrlX, SllY);
  return DAG.getNode(ISD::BITCAST, DL, Op.getOperand(0).getValueType(), Or);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2505-2508
```cpp
SDValue
MipsTargetLowering::lowerFCOPYSIGN(SDValue Op, SelectionDAG &DAG) const {
  if (Subtarget.isGP64bit())
    return lowerFCOPYSIGN64(Op, DAG, Subtarget.hasExtractInsert());
```
- EN: Implements `MipsTargetLowering::lowerFCOPYSIGN`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::lowerFCOPYSIGN`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2510-2511
```cpp
  return lowerFCOPYSIGN32(Op, DAG, Subtarget.hasExtractInsert());
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2513-2516
```cpp
SDValue MipsTargetLowering::lowerFABS32(SDValue Op, SelectionDAG &DAG,
                                        bool HasExtractInsert) const {
  SDLoc DL(Op);
  SDValue Res, Const1 = DAG.getConstant(1, DL, MVT::i32);
```
- EN: Implements `MipsTargetLowering::lowerFABS32`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::lowerFABS32`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2518-2519
```cpp
  if (Op->getFlags().hasNoNaNs() || Subtarget.inAbs2008Mode())
    return DAG.getNode(MipsISD::FAbs, DL, Op.getValueType(), Op.getOperand(0));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2521-2526
```cpp
  // If operand is of type f64, extract the upper 32-bit. Otherwise, bitcast it
  // to i32.
  SDValue X = (Op.getValueType() == MVT::f32)
                  ? DAG.getNode(ISD::BITCAST, DL, MVT::i32, Op.getOperand(0))
                  : DAG.getNode(MipsISD::ExtractElementF64, DL, MVT::i32,
                                Op.getOperand(0), Const1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2528-2538
```cpp
  // Clear MSB.
  if (HasExtractInsert)
    Res = DAG.getNode(MipsISD::Ins, DL, MVT::i32,
                      DAG.getRegister(Mips::ZERO, MVT::i32),
                      DAG.getConstant(31, DL, MVT::i32), Const1, X);
  else {
    // TODO: Provide DAG patterns which transform (and x, cst)
    // back to a (shl (srl x (clz cst)) (clz cst)) sequence.
    SDValue SllX = DAG.getNode(ISD::SHL, DL, MVT::i32, X, Const1);
    Res = DAG.getNode(ISD::SRL, DL, MVT::i32, SllX, Const1);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2540-2541
```cpp
  if (Op.getValueType() == MVT::f32)
    return DAG.getNode(ISD::BITCAST, DL, MVT::f32, Res);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2543-2551
```cpp
  // FIXME: For mips32r2, the sequence of (BuildPairF64 (ins (ExtractElementF64
  // Op 1), $zero, 31 1) (ExtractElementF64 Op 0)) and the Op has one use, we
  // should be able to drop the usage of mfc1/mtc1 and rewrite the register in
  // place.
  SDValue LowX =
      DAG.getNode(MipsISD::ExtractElementF64, DL, MVT::i32, Op.getOperand(0),
                  DAG.getConstant(0, DL, MVT::i32));
  return DAG.getNode(MipsISD::BuildPairF64, DL, MVT::f64, LowX, Res);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2553-2556
```cpp
SDValue MipsTargetLowering::lowerFABS64(SDValue Op, SelectionDAG &DAG,
                                        bool HasExtractInsert) const {
  SDLoc DL(Op);
  SDValue Res, Const1 = DAG.getConstant(1, DL, MVT::i32);
```
- EN: Implements `MipsTargetLowering::lowerFABS64`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::lowerFABS64`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2558-2559
```cpp
  if (Op->getFlags().hasNoNaNs() || Subtarget.inAbs2008Mode())
    return DAG.getNode(MipsISD::FAbs, DL, Op.getValueType(), Op.getOperand(0));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2561-2562
```cpp
  // Bitcast to integer node.
  SDValue X = DAG.getNode(ISD::BITCAST, DL, MVT::i64, Op.getOperand(0));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2564-2572
```cpp
  // Clear MSB.
  if (HasExtractInsert)
    Res = DAG.getNode(MipsISD::Ins, DL, MVT::i64,
                      DAG.getRegister(Mips::ZERO_64, MVT::i64),
                      DAG.getConstant(63, DL, MVT::i32), Const1, X);
  else {
    SDValue SllX = DAG.getNode(ISD::SHL, DL, MVT::i64, X, Const1);
    Res = DAG.getNode(ISD::SRL, DL, MVT::i64, SllX, Const1);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2574-2575
```cpp
  return DAG.getNode(ISD::BITCAST, DL, MVT::f64, Res);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2577-2579
```cpp
SDValue MipsTargetLowering::lowerFABS(SDValue Op, SelectionDAG &DAG) const {
  if ((ABI.IsN32() || ABI.IsN64()) && (Op.getValueType() == MVT::f64))
    return lowerFABS64(Op, DAG, Subtarget.hasExtractInsert());
```
- EN: Implements `MipsTargetLowering::lowerFABS`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::lowerFABS`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2581-2582
```cpp
  return lowerFABS32(Op, DAG, Subtarget.hasExtractInsert());
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2584-2589
```cpp
SDValue MipsTargetLowering::lowerFCANONICALIZE(SDValue Op,
                                               SelectionDAG &DAG) const {
  SDLoc DL(Op);
  EVT VT = Op.getValueType();
  SDValue Operand = Op.getOperand(0);
  SDNodeFlags Flags = Op->getFlags();
```
- EN: Implements `MipsTargetLowering::lowerFCANONICALIZE`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::lowerFCANONICALIZE`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2591-2592
```cpp
  if (Flags.hasNoNaNs() || DAG.isKnownNeverNaN(Operand))
    return Operand;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2594-2596
```cpp
  SDValue Quiet = DAG.getNode(ISD::FADD, DL, VT, Operand, Operand);
  return DAG.getSelectCC(DL, Operand, Operand, Quiet, Operand, ISD::SETUO);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2598-2605
```cpp
SDValue MipsTargetLowering::
lowerFRAMEADDR(SDValue Op, SelectionDAG &DAG) const {
  // check the depth
  if (Op.getConstantOperandVal(0) != 0) {
    DAG.getContext()->emitError(
        "return address can be determined only for current frame");
    return SDValue();
  }
```
- EN: Implements `lowerFRAMEADDR`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `lowerFRAMEADDR`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2607-2614
```cpp
  MachineFrameInfo &MFI = DAG.getMachineFunction().getFrameInfo();
  MFI.setFrameAddressIsTaken(true);
  EVT VT = Op.getValueType();
  SDLoc DL(Op);
  SDValue FrameAddr = DAG.getCopyFromReg(
      DAG.getEntryNode(), DL, ABI.IsN64() ? Mips::FP_64 : Mips::FP, VT);
  return FrameAddr;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2616-2623
```cpp
SDValue MipsTargetLowering::lowerRETURNADDR(SDValue Op,
                                            SelectionDAG &DAG) const {
  // check the depth
  if (Op.getConstantOperandVal(0) != 0) {
    DAG.getContext()->emitError(
        "return address can be determined only for current frame");
    return SDValue();
  }
```
- EN: Implements `MipsTargetLowering::lowerRETURNADDR`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::lowerRETURNADDR`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2625-2629
```cpp
  MachineFunction &MF = DAG.getMachineFunction();
  MachineFrameInfo &MFI = MF.getFrameInfo();
  MVT VT = Op.getSimpleValueType();
  unsigned RA = ABI.IsN64() ? Mips::RA_64 : Mips::RA;
  MFI.setReturnAddressIsTaken(true);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2631-2634
```cpp
  // Return RA, which contains the return address. Mark it an implicit live-in.
  Register Reg = MF.addLiveIn(RA, getRegClassFor(VT));
  return DAG.getCopyFromReg(DAG.getEntryNode(), SDLoc(Op), Reg, VT);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2636-2643
```cpp
// An EH_RETURN is the result of lowering llvm.eh.return which in turn is
// generated from __builtin_eh_return (offset, handler)
// The effect of this is to adjust the stack pointer by "offset"
// and then branch to "handler".
SDValue MipsTargetLowering::lowerEH_RETURN(SDValue Op, SelectionDAG &DAG)
                                                                     const {
  MachineFunction &MF = DAG.getMachineFunction();
  MipsFunctionInfo *MipsFI = MF.getInfo<MipsFunctionInfo>();
```
- EN: Implements `MipsTargetLowering::lowerEH_RETURN`, a lowering routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::lowerEH_RETURN`，它是一个围绕机器函数状态展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2645-2650
```cpp
  MipsFI->setCallsEhReturn();
  SDValue Chain     = Op.getOperand(0);
  SDValue Offset    = Op.getOperand(1);
  SDValue Handler   = Op.getOperand(2);
  SDLoc DL(Op);
  EVT Ty = ABI.IsN64() ? MVT::i64 : MVT::i32;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2652-2662
```cpp
  // Store stack offset in V1, store jump target in V0. Glue CopyToReg and
  // EH_RETURN nodes, so that instructions are emitted back-to-back.
  unsigned OffsetReg = ABI.IsN64() ? Mips::V1_64 : Mips::V1;
  unsigned AddrReg = ABI.IsN64() ? Mips::V0_64 : Mips::V0;
  Chain = DAG.getCopyToReg(Chain, DL, OffsetReg, Offset, SDValue());
  Chain = DAG.getCopyToReg(Chain, DL, AddrReg, Handler, Chain.getValue(1));
  return DAG.getNode(MipsISD::EH_RETURN, DL, MVT::Other, Chain,
                     DAG.getRegister(OffsetReg, Ty),
                     DAG.getRegister(AddrReg, getPointerTy(MF.getDataLayout())),
                     Chain.getValue(1));
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2664-2671
```cpp
SDValue MipsTargetLowering::lowerATOMIC_FENCE(SDValue Op,
                                              SelectionDAG &DAG) const {
  // FIXME: Need pseudo-fence for 'singlethread' fences
  // FIXME: Set SType for weaker fences where supported/appropriate.
  unsigned SType = 0;
  SDLoc DL(Op);
  SyncScope::ID FenceSSID =
      static_cast<SyncScope::ID>(Op.getConstantOperandVal(2));
```
- EN: Implements `MipsTargetLowering::lowerATOMIC_FENCE`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::lowerATOMIC_FENCE`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2673-2675
```cpp
  if (Subtarget.hasMips2() && FenceSSID == SyncScope::System)
    return DAG.getNode(MipsISD::Sync, DL, MVT::Other, Op.getOperand(0),
                       DAG.getTargetConstant(SType, DL, MVT::i32));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2677-2684
```cpp
  // singlethread fences only synchronize with signal handlers on the same
  // thread and thus only need to preserve instruction order, not actually
  // enforce memory ordering.
  if ((Subtarget.hasMips1() && !Subtarget.hasMips2()) ||
      FenceSSID == SyncScope::SingleThread) {
    // MEMBARRIER is a compiler barrier; it codegens to a no-op.
    return DAG.getNode(ISD::MEMBARRIER, DL, MVT::Other, Op.getOperand(0));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2686-2687
```cpp
  return Op;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2689-2692
```cpp
SDValue MipsTargetLowering::lowerShiftLeftParts(SDValue Op,
                                                SelectionDAG &DAG) const {
  SDLoc DL(Op);
  MVT VT = Subtarget.isGP64bit() ? MVT::i64 : MVT::i32;
```
- EN: Implements `MipsTargetLowering::lowerShiftLeftParts`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::lowerShiftLeftParts`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2694-2708
```cpp
  SDValue Lo = Op.getOperand(0), Hi = Op.getOperand(1);
  SDValue Shamt = Op.getOperand(2);
  // if shamt < (VT.bits):
  //  lo = (shl lo, shamt)
  //  hi = (or (shl hi, shamt) (srl (srl lo, 1), (xor shamt, (VT.bits-1))))
  // else:
  //  lo = 0
  //  hi = (shl lo, shamt[4:0])
  SDValue Not =
      DAG.getNode(ISD::XOR, DL, MVT::i32, Shamt,
                  DAG.getConstant(VT.getSizeInBits() - 1, DL, MVT::i32));
  SDValue ShiftRight1Lo = DAG.getNode(ISD::SRL, DL, VT, Lo,
                                      DAG.getConstant(1, DL, VT));
  SDValue ShiftRightLo = DAG.getNode(ISD::SRL, DL, VT, ShiftRight1Lo, Not);
  SDValue ShiftLeftHi = DAG.getNode(ISD::SHL, DL, VT, Hi, Shamt);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2709-2715
```cpp
  SDValue Or = DAG.getNode(ISD::OR, DL, VT, ShiftLeftHi, ShiftRightLo);
  SDValue ShiftLeftLo = DAG.getNode(ISD::SHL, DL, VT, Lo, Shamt);
  SDValue Cond = DAG.getNode(ISD::AND, DL, MVT::i32, Shamt,
                             DAG.getConstant(VT.getSizeInBits(), DL, MVT::i32));
  Lo = DAG.getNode(ISD::SELECT, DL, VT, Cond,
                   DAG.getConstant(0, DL, VT), ShiftLeftLo);
  Hi = DAG.getNode(ISD::SELECT, DL, VT, Cond, ShiftLeftLo, Or);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2717-2719
```cpp
  SDValue Ops[2] = {Lo, Hi};
  return DAG.getMergeValues(Ops, DL);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2721-2726
```cpp
SDValue MipsTargetLowering::lowerShiftRightParts(SDValue Op, SelectionDAG &DAG,
                                                 bool IsSRA) const {
  SDLoc DL(Op);
  SDValue Lo = Op.getOperand(0), Hi = Op.getOperand(1);
  SDValue Shamt = Op.getOperand(2);
  MVT VT = Subtarget.isGP64bit() ? MVT::i64 : MVT::i32;
```
- EN: Implements `MipsTargetLowering::lowerShiftRightParts`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::lowerShiftRightParts`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2728-2742
```cpp
  // if shamt < (VT.bits):
  //  lo = (or (shl (shl hi, 1), (xor shamt, (VT.bits-1))) (srl lo, shamt))
  //  if isSRA:
  //    hi = (sra hi, shamt)
  //  else:
  //    hi = (srl hi, shamt)
  // else:
  //  if isSRA:
  //   lo = (sra hi, shamt[4:0])
  //   hi = (sra hi, 31)
  //  else:
  //   lo = (srl hi, shamt[4:0])
  //   hi = 0
  SDValue Not =
      DAG.getNode(ISD::XOR, DL, MVT::i32, Shamt,
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2743-2754
```cpp
                  DAG.getConstant(VT.getSizeInBits() - 1, DL, MVT::i32));
  SDValue ShiftLeft1Hi = DAG.getNode(ISD::SHL, DL, VT, Hi,
                                     DAG.getConstant(1, DL, VT));
  SDValue ShiftLeftHi = DAG.getNode(ISD::SHL, DL, VT, ShiftLeft1Hi, Not);
  SDValue ShiftRightLo = DAG.getNode(ISD::SRL, DL, VT, Lo, Shamt);
  SDValue Or = DAG.getNode(ISD::OR, DL, VT, ShiftLeftHi, ShiftRightLo);
  SDValue ShiftRightHi = DAG.getNode(IsSRA ? ISD::SRA : ISD::SRL,
                                     DL, VT, Hi, Shamt);
  SDValue Cond = DAG.getNode(ISD::AND, DL, MVT::i32, Shamt,
                             DAG.getConstant(VT.getSizeInBits(), DL, MVT::i32));
  SDValue Ext = DAG.getNode(ISD::SRA, DL, VT, Hi,
                            DAG.getConstant(VT.getSizeInBits() - 1, DL, VT));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2756-2763
```cpp
  if (!(Subtarget.hasMips4() || Subtarget.hasMips32())) {
    SDVTList VTList = DAG.getVTList(VT, VT);
    return DAG.getNode(Subtarget.isGP64bit() ? MipsISD::DOUBLE_SELECT_I64
                                             : MipsISD::DOUBLE_SELECT_I,
                       DL, VTList, Cond, ShiftRightHi,
                       IsSRA ? Ext : DAG.getConstant(0, DL, VT), Or,
                       ShiftRightHi);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2765-2767
```cpp
  Lo = DAG.getNode(ISD::SELECT, DL, VT, Cond, ShiftRightHi, Or);
  Hi = DAG.getNode(ISD::SELECT, DL, VT, Cond,
                   IsSRA ? Ext : DAG.getConstant(0, DL, VT), ShiftRightHi);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2769-2771
```cpp
  SDValue Ops[2] = {Lo, Hi};
  return DAG.getMergeValues(Ops, DL);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2773-2779
```cpp
static SDValue createLoadLR(unsigned Opc, SelectionDAG &DAG, LoadSDNode *LD,
                            SDValue Chain, SDValue Src, unsigned Offset) {
  SDValue Ptr = LD->getBasePtr();
  EVT VT = LD->getValueType(0), MemVT = LD->getMemoryVT();
  EVT BasePtrVT = Ptr.getValueType();
  SDLoc DL(LD);
  SDVTList VTList = DAG.getVTList(VT, MVT::Other);
```
- EN: Implements `createLoadLR`, a query/helper routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `createLoadLR`，它是一个围绕SelectionDAG 降级展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2781-2783
```cpp
  if (Offset)
    Ptr = DAG.getNode(ISD::ADD, DL, BasePtrVT, Ptr,
                      DAG.getConstant(Offset, DL, BasePtrVT));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2785-2788
```cpp
  SDValue Ops[] = { Chain, Ptr, Src };
  return DAG.getMemIntrinsicNode(Opc, DL, VTList, Ops, MemVT,
                                 LD->getMemOperand());
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2790-2793
```cpp
// Expand an unaligned 32 or 64-bit integer load node.
SDValue MipsTargetLowering::lowerLOAD(SDValue Op, SelectionDAG &DAG) const {
  LoadSDNode *LD = cast<LoadSDNode>(Op);
  EVT MemVT = LD->getMemoryVT();
```
- EN: Implements `MipsTargetLowering::lowerLOAD`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::lowerLOAD`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2795-2796
```cpp
  if (Subtarget.systemSupportsUnalignedAccess())
    return Op;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2798-2801
```cpp
  // Return if load is aligned or if MemVT is neither i32 nor i64.
  if ((LD->getAlign().value() >= (MemVT.getSizeInBits() / 8)) ||
      ((MemVT != MVT::i32) && (MemVT != MVT::i64)))
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2803-2806
```cpp
  bool IsLittle = Subtarget.isLittle();
  EVT VT = Op.getValueType();
  ISD::LoadExtType ExtType = LD->getExtensionType();
  SDValue Chain = LD->getChain(), Undef = DAG.getUNDEF(VT);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2808-2808
```cpp
  assert((VT == MVT::i32) || (VT == MVT::i64));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2810-2820
```cpp
  // Expand
  //  (set dst, (i64 (load baseptr)))
  // to
  //  (set tmp, (ldl (add baseptr, 7), undef))
  //  (set dst, (ldr baseptr, tmp))
  if ((VT == MVT::i64) && (ExtType == ISD::NON_EXTLOAD)) {
    SDValue LDL = createLoadLR(MipsISD::LDL, DAG, LD, Chain, Undef,
                               IsLittle ? 7 : 0);
    return createLoadLR(MipsISD::LDR, DAG, LD, LDL.getValue(1), LDL,
                        IsLittle ? 0 : 7);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2822-2825
```cpp
  SDValue LWL = createLoadLR(MipsISD::LWL, DAG, LD, Chain, Undef,
                             IsLittle ? 3 : 0);
  SDValue LWR = createLoadLR(MipsISD::LWR, DAG, LD, LWL.getValue(1), LWL,
                             IsLittle ? 0 : 3);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2827-2836
```cpp
  // Expand
  //  (set dst, (i32 (load baseptr))) or
  //  (set dst, (i64 (sextload baseptr))) or
  //  (set dst, (i64 (extload baseptr)))
  // to
  //  (set tmp, (lwl (add baseptr, 3), undef))
  //  (set dst, (lwr baseptr, tmp))
  if ((VT == MVT::i32) || (ExtType == ISD::SEXTLOAD) ||
      (ExtType == ISD::EXTLOAD))
    return LWR;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2838-2838
```cpp
  assert((VT == MVT::i64) && (ExtType == ISD::ZEXTLOAD));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2840-2853
```cpp
  // Expand
  //  (set dst, (i64 (zextload baseptr)))
  // to
  //  (set tmp0, (lwl (add baseptr, 3), undef))
  //  (set tmp1, (lwr baseptr, tmp0))
  //  (set tmp2, (shl tmp1, 32))
  //  (set dst, (srl tmp2, 32))
  SDLoc DL(LD);
  SDValue Const32 = DAG.getConstant(32, DL, MVT::i32);
  SDValue SLL = DAG.getNode(ISD::SHL, DL, MVT::i64, LWR, Const32);
  SDValue SRL = DAG.getNode(ISD::SRL, DL, MVT::i64, SLL, Const32);
  SDValue Ops[] = { SRL, LWR.getValue(1) };
  return DAG.getMergeValues(Ops, DL);
}
```
- EN: Declares `DL`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `DL`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2855-2860
```cpp
static SDValue createStoreLR(unsigned Opc, SelectionDAG &DAG, StoreSDNode *SD,
                             SDValue Chain, unsigned Offset) {
  SDValue Ptr = SD->getBasePtr(), Value = SD->getValue();
  EVT MemVT = SD->getMemoryVT(), BasePtrVT = Ptr.getValueType();
  SDLoc DL(SD);
  SDVTList VTList = DAG.getVTList(MVT::Other);
```
- EN: Implements `createStoreLR`, a query/helper routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `createStoreLR`，它是一个围绕SelectionDAG 降级展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2862-2864
```cpp
  if (Offset)
    Ptr = DAG.getNode(ISD::ADD, DL, BasePtrVT, Ptr,
                      DAG.getConstant(Offset, DL, BasePtrVT));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2866-2869
```cpp
  SDValue Ops[] = { Chain, Value, Ptr };
  return DAG.getMemIntrinsicNode(Opc, DL, VTList, Ops, MemVT,
                                 SD->getMemOperand());
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2871-2875
```cpp
// Expand an unaligned 32 or 64-bit integer store node.
static SDValue lowerUnalignedIntStore(StoreSDNode *SD, SelectionDAG &DAG,
                                      bool IsLittle) {
  SDValue Value = SD->getValue(), Chain = SD->getChain();
  EVT VT = Value.getValueType();
```
- EN: Implements `lowerUnalignedIntStore`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `lowerUnalignedIntStore`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2877-2887
```cpp
  // Expand
  //  (store val, baseptr) or
  //  (truncstore val, baseptr)
  // to
  //  (swl val, (add baseptr, 3))
  //  (swr val, baseptr)
  if ((VT == MVT::i32) || SD->isTruncatingStore()) {
    SDValue SWL = createStoreLR(MipsISD::SWL, DAG, SD, Chain,
                                IsLittle ? 3 : 0);
    return createStoreLR(MipsISD::SWR, DAG, SD, SWL, IsLittle ? 0 : 3);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2889-2889
```cpp
  assert(VT == MVT::i64);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2891-2898
```cpp
  // Expand
  //  (store val, baseptr)
  // to
  //  (sdl val, (add baseptr, 7))
  //  (sdr val, baseptr)
  SDValue SDL = createStoreLR(MipsISD::SDL, DAG, SD, Chain, IsLittle ? 7 : 0);
  return createStoreLR(MipsISD::SDR, DAG, SD, SDL, IsLittle ? 0 : 7);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2900-2903
```cpp
// Lower (store (fp_to_sint $fp) $ptr) to (store (TruncIntFP $fp), $ptr).
static SDValue lowerFP_TO_SINT_STORE(StoreSDNode *SD, SelectionDAG &DAG,
                                     bool SingleFloat) {
  SDValue Val = SD->getValue();
```
- EN: Implements `lowerFP_TO_SINT_STORE`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `lowerFP_TO_SINT_STORE`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2905-2907
```cpp
  if (Val.getOpcode() != ISD::FP_TO_SINT ||
      (Val.getValueSizeInBits() > 32 && SingleFloat))
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2909-2915
```cpp
  EVT FPTy = EVT::getFloatingPointVT(Val.getValueSizeInBits());
  SDValue Tr = DAG.getNode(MipsISD::TruncIntFP, SDLoc(Val), FPTy,
                           Val.getOperand(0));
  return DAG.getStore(SD->getChain(), SDLoc(SD), Tr, SD->getBasePtr(),
                      SD->getPointerInfo(), SD->getAlign(),
                      SD->getMemOperand()->getFlags());
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2917-2919
```cpp
SDValue MipsTargetLowering::lowerSTORE(SDValue Op, SelectionDAG &DAG) const {
  StoreSDNode *SD = cast<StoreSDNode>(Op);
  EVT MemVT = SD->getMemoryVT();
```
- EN: Implements `MipsTargetLowering::lowerSTORE`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::lowerSTORE`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2921-2925
```cpp
  // Lower unaligned integer stores.
  if (!Subtarget.systemSupportsUnalignedAccess() &&
      (SD->getAlign().value() < (MemVT.getSizeInBits() / 8)) &&
      ((MemVT == MVT::i32) || (MemVT == MVT::i64)))
    return lowerUnalignedIntStore(SD, DAG, Subtarget.isLittle());
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2927-2928
```cpp
  return lowerFP_TO_SINT_STORE(SD, DAG, Subtarget.isSingleFloat());
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2930-2931
```cpp
SDValue MipsTargetLowering::lowerEH_DWARF_CFA(SDValue Op,
                                              SelectionDAG &DAG) const {
```
- EN: Implements `MipsTargetLowering::lowerEH_DWARF_CFA`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::lowerEH_DWARF_CFA`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2933-2939
```cpp
  // Return a fixed StackObject with offset 0 which points to the old stack
  // pointer.
  MachineFrameInfo &MFI = DAG.getMachineFunction().getFrameInfo();
  EVT ValTy = Op->getValueType(0);
  int FI = MFI.CreateFixedObject(Op.getValueSizeInBits() / 8, 0, false);
  return DAG.getFrameIndex(FI, ValTy);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2941-2944
```cpp
SDValue MipsTargetLowering::lowerFP_TO_SINT(SDValue Op,
                                            SelectionDAG &DAG) const {
  if (Op.getValueSizeInBits() > 32 && Subtarget.isSingleFloat())
    return SDValue();
```
- EN: Implements `MipsTargetLowering::lowerFP_TO_SINT`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::lowerFP_TO_SINT`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2946-2950
```cpp
  EVT FPTy = EVT::getFloatingPointVT(Op.getValueSizeInBits());
  SDValue Trunc = DAG.getNode(MipsISD::TruncIntFP, SDLoc(Op), FPTy,
                              Op.getOperand(0));
  return DAG.getNode(ISD::BITCAST, SDLoc(Op), Op.getValueType(), Trunc);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2952-2956
```cpp
SDValue MipsTargetLowering::lowerSTRICT_FP_TO_INT(SDValue Op,
                                                  SelectionDAG &DAG) const {
  assert(Op->isStrictFPOpcode());
  SDValue SrcVal = Op.getOperand(1);
  SDLoc Loc(Op);
```
- EN: Implements `MipsTargetLowering::lowerSTRICT_FP_TO_INT`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::lowerSTRICT_FP_TO_INT`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2958-2961
```cpp
  SDValue Result =
      DAG.getNode(Op.getOpcode() == ISD::STRICT_FP_TO_SINT ? ISD::FP_TO_SINT
                                                           : ISD::FP_TO_UINT,
                  Loc, Op.getValueType(), SrcVal);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2963-2964
```cpp
  return DAG.getMergeValues({Result, Op.getOperand(0)}, Loc);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2966-2969
```cpp
ArrayRef<MCPhysReg> MipsTargetLowering::getRoundingControlRegisters() const {
  static const MCPhysReg RCRegs[] = {Mips::FCR31};
  return RCRegs;
}
```
- EN: Implements `MipsTargetLowering::getRoundingControlRegisters`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::getRoundingControlRegisters`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2971-2973
```cpp
//===----------------------------------------------------------------------===//
//                      Calling Convention Implementation
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 2975-2989
```cpp
//===----------------------------------------------------------------------===//
// TODO: Implement a generic logic using tblgen that can support this.
// Mips O32 ABI rules:
// ---
// i32 - Passed in A0, A1, A2, A3 and stack
// f32 - Only passed in f32 registers if no int reg has been used yet to hold
//       an argument. Otherwise, passed in A1, A2, A3 and stack.
// f64 - Only passed in two aliased f32 registers if no int reg has been used
//       yet to hold an argument. Otherwise, use A2, A3 and stack. If A1 is
//       not used, it must be shadowed. If only A3 is available, shadow it and
//       go to stack.
// vXiX - Received as scalarized i32s, passed in A0 - A3 and the stack.
// vXf32 - Passed in either a pair of registers {A0, A1}, {A2, A3} or {A0 - A3}
//         with the remainder spilled to the stack.
// vXf64 - Passed in either {A0, A1, A2, A3} or {A2, A3} and in both cases
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 2990-2993
```cpp
//         spilling the remainder to the stack.
//
//  For vararg functions, all arguments are passed in A0, A1, A2, A3 and stack.
//===----------------------------------------------------------------------===//
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 2995-3000
```cpp
static bool CC_MipsO32(unsigned ValNo, MVT ValVT, MVT LocVT,
                       CCValAssign::LocInfo LocInfo, ISD::ArgFlagsTy ArgFlags,
                       Type *OrigTy, CCState &State,
                       ArrayRef<MCPhysReg> F64Regs) {
  const MipsSubtarget &Subtarget = static_cast<const MipsSubtarget &>(
      State.getMachineFunction().getSubtarget());
```
- EN: Implements `CC_MipsO32`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `CC_MipsO32`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3002-3002
```cpp
  static const MCPhysReg IntRegs[] = { Mips::A0, Mips::A1, Mips::A2, Mips::A3 };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3004-3004
```cpp
  static const MCPhysReg F32Regs[] = { Mips::F12, Mips::F14 };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3006-3006
```cpp
  static const MCPhysReg FloatVectorIntRegs[] = { Mips::A0, Mips::A2 };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3008-3010
```cpp
  // Do not process byval args here.
  if (ArgFlags.isByVal())
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3012-3023
```cpp
  // Promote i8 and i16
  if (ArgFlags.isInReg() && !Subtarget.isLittle()) {
    if (LocVT == MVT::i8 || LocVT == MVT::i16 || LocVT == MVT::i32) {
      LocVT = MVT::i32;
      if (ArgFlags.isSExt())
        LocInfo = CCValAssign::SExtUpper;
      else if (ArgFlags.isZExt())
        LocInfo = CCValAssign::ZExtUpper;
      else
        LocInfo = CCValAssign::AExtUpper;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3025-3034
```cpp
  // Promote i8 and i16
  if (LocVT == MVT::i8 || LocVT == MVT::i16) {
    LocVT = MVT::i32;
    if (ArgFlags.isSExt())
      LocInfo = CCValAssign::SExt;
    else if (ArgFlags.isZExt())
      LocInfo = CCValAssign::ZExt;
    else
      LocInfo = CCValAssign::AExt;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3036-3036
```cpp
  unsigned Reg;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3038-3045
```cpp
  // f32 and f64 are allocated in A0, A1, A2, A3 when either of the following
  // is true: function is vararg, argument is 3rd or higher, there is previous
  // argument which is not f32 or f64.
  bool AllocateFloatsInIntReg = State.isVarArg() || ValNo > 1 ||
                                State.getFirstUnallocated(F32Regs) != ValNo;
  Align OrigAlign = ArgFlags.getNonZeroOrigAlign();
  bool isI64 = (ValVT == MVT::i32 && OrigAlign == Align(8));
  bool isVectorFloat = OrigTy->isVectorTy() && OrigTy->isFPOrFPVectorTy();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3047-3061
```cpp
  // The MIPS vector ABI for floats passes them in a pair of registers
  if (ValVT == MVT::i32 && isVectorFloat) {
    // This is the start of an vector that was scalarized into an unknown number
    // of components. It doesn't matter how many there are. Allocate one of the
    // notional 8 byte aligned registers which map onto the argument stack, and
    // shadow the register lost to alignment requirements.
    if (ArgFlags.isSplit()) {
      Reg = State.AllocateReg(FloatVectorIntRegs);
      if (Reg == Mips::A2)
        State.AllocateReg(Mips::A1);
      else if (Reg == 0)
        State.AllocateReg(Mips::A3);
    } else {
      // If we're an intermediate component of the split, we can just attempt to
      // allocate a register directly.
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3062-3076
```cpp
      Reg = State.AllocateReg(IntRegs);
    }
  } else if (ValVT == MVT::i32 ||
             (ValVT == MVT::f32 && AllocateFloatsInIntReg)) {
    Reg = State.AllocateReg(IntRegs);
    // If this is the first part of an i64 arg,
    // the allocated register must be either A0 or A2.
    if (isI64 && (Reg == Mips::A1 || Reg == Mips::A3))
      Reg = State.AllocateReg(IntRegs);
    LocVT = MVT::i32;
  } else if (ValVT == MVT::f64 && AllocateFloatsInIntReg) {
    // Allocate int register and shadow next int register. If first
    // available register is Mips::A1 or Mips::A3, shadow it too.
    Reg = State.AllocateReg(IntRegs);
    if (Reg == Mips::A1 || Reg == Mips::A3)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3077-3077
```cpp
      Reg = State.AllocateReg(IntRegs);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3079-3080
```cpp
    if (Reg) {
      LocVT = MVT::i32;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3082-3096
```cpp
      State.addLoc(
          CCValAssign::getCustomReg(ValNo, ValVT, Reg, LocVT, LocInfo));
      MCRegister HiReg = State.AllocateReg(IntRegs);
      assert(HiReg);
      State.addLoc(
          CCValAssign::getCustomReg(ValNo, ValVT, HiReg, LocVT, LocInfo));
      return false;
    }
  } else if (ValVT.isFloatingPoint() && !AllocateFloatsInIntReg) {
    // we are guaranteed to find an available float register
    if (ValVT == MVT::f32) {
      Reg = State.AllocateReg(F32Regs);
      // Shadow int register
      State.AllocateReg(IntRegs);
    } else {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3097-3105
```cpp
      Reg = State.AllocateReg(F64Regs);
      // Shadow int registers
      MCRegister Reg2 = State.AllocateReg(IntRegs);
      if (Reg2 == Mips::A1 || Reg2 == Mips::A3)
        State.AllocateReg(IntRegs);
      State.AllocateReg(IntRegs);
    }
  } else
    llvm_unreachable("Cannot handle this ValVT.");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3107-3111
```cpp
  if (!Reg) {
    unsigned Offset = State.AllocateStack(ValVT.getStoreSize(), OrigAlign);
    State.addLoc(CCValAssign::getMem(ValNo, ValVT, Offset, LocVT, LocInfo));
  } else
    State.addLoc(CCValAssign::getReg(ValNo, ValVT, Reg, LocVT, LocInfo));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3113-3114
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3116-3120
```cpp
static bool CC_MipsO32_FP32(unsigned ValNo, MVT ValVT, MVT LocVT,
                            CCValAssign::LocInfo LocInfo,
                            ISD::ArgFlagsTy ArgFlags, Type *OrigTy,
                            CCState &State) {
  static const MCPhysReg F64Regs[] = { Mips::D6, Mips::D7 };
```
- EN: Implements `CC_MipsO32_FP32`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `CC_MipsO32_FP32`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3122-3124
```cpp
  return CC_MipsO32(ValNo, ValVT, LocVT, LocInfo, ArgFlags, OrigTy, State,
                    F64Regs);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3126-3130
```cpp
static bool CC_MipsO32_FP64(unsigned ValNo, MVT ValVT, MVT LocVT,
                            CCValAssign::LocInfo LocInfo,
                            ISD::ArgFlagsTy ArgFlags, Type *OrigTy,
                            CCState &State) {
  static const MCPhysReg F64Regs[] = { Mips::D12_64, Mips::D14_64 };
```
- EN: Implements `CC_MipsO32_FP64`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `CC_MipsO32_FP64`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3132-3134
```cpp
  return CC_MipsO32(ValNo, ValVT, LocVT, LocInfo, ArgFlags, OrigTy, State,
                    F64Regs);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3136-3139
```cpp
[[maybe_unused]] static bool CC_MipsO32(unsigned ValNo, MVT ValVT, MVT LocVT,
                                        CCValAssign::LocInfo LocInfo,
                                        ISD::ArgFlagsTy ArgFlags, Type *OrigTy,
                                        CCState &State);
```
- EN: Declares `CC_MipsO32`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `CC_MipsO32`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3141-3141
```cpp
#include "MipsGenCallingConv.inc"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 3143-3145
```cpp
 CCAssignFn *MipsTargetLowering::CCAssignFnForCall() const{
   return CC_Mips_FixedArg;
 }
```
- EN: Implements `MipsTargetLowering::CCAssignFnForCall`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::CCAssignFnForCall`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3147-3152
```cpp
 CCAssignFn *MipsTargetLowering::CCAssignFnForReturn() const{
   return RetCC_Mips;
 }
//===----------------------------------------------------------------------===//
//                  Call Calling Convention Implementation
//===----------------------------------------------------------------------===//
```
- EN: Implements `MipsTargetLowering::CCAssignFnForReturn`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::CCAssignFnForReturn`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3154-3163
```cpp
SDValue MipsTargetLowering::passArgOnStack(SDValue StackPtr, unsigned Offset,
                                           SDValue Chain, SDValue Arg,
                                           const SDLoc &DL, bool IsTailCall,
                                           SelectionDAG &DAG) const {
  if (!IsTailCall) {
    SDValue PtrOff =
        DAG.getNode(ISD::ADD, DL, getPointerTy(DAG.getDataLayout()), StackPtr,
                    DAG.getIntPtrConstant(Offset, DL));
    return DAG.getStore(Chain, DL, Arg, PtrOff, MachinePointerInfo());
  }
```
- EN: Implements `MipsTargetLowering::passArgOnStack`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::passArgOnStack`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3165-3170
```cpp
  MachineFrameInfo &MFI = DAG.getMachineFunction().getFrameInfo();
  int FI = MFI.CreateFixedObject(Arg.getValueSizeInBits() / 8, Offset, false);
  SDValue FIN = DAG.getFrameIndex(FI, getPointerTy(DAG.getDataLayout()));
  return DAG.getStore(Chain, DL, Arg, FIN, MachinePointerInfo(), MaybeAlign(),
                      MachineMemOperand::MOVolatile);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3172-3186
```cpp
void MipsTargetLowering::
getOpndList(SmallVectorImpl<SDValue> &Ops,
            std::deque<std::pair<unsigned, SDValue>> &RegsToPass,
            bool IsPICCall, bool GlobalOrExternal, bool InternalLinkage,
            bool IsCallReloc, CallLoweringInfo &CLI, SDValue Callee,
            SDValue Chain) const {
  // Insert node "GP copy globalreg" before call to function.
  //
  // R_MIPS_CALL* operators (emitted when non-internal functions are called
  // in PIC mode) allow symbols to be resolved via lazy binding.
  // The lazy binding stub requires GP to point to the GOT.
  // Note that we don't need GP to point to the GOT for indirect calls
  // (when R_MIPS_CALL* is not used for the call) because Mips linker generates
  // lazy binding stub for a function only when R_MIPS_CALL* are the only relocs
  // used for the function (that is, Mips linker doesn't generate lazy binding
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3187-3192
```cpp
  // stub for a function whose address is taken in the program).
  if (IsPICCall && !InternalLinkage && IsCallReloc) {
    unsigned GPReg = ABI.IsN64() ? Mips::GP_64 : Mips::GP;
    EVT Ty = ABI.IsN64() ? MVT::i64 : MVT::i32;
    RegsToPass.push_back(std::make_pair(GPReg, getGlobalReg(CLI.DAG, Ty)));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3194-3198
```cpp
  // Build a sequence of copy-to-reg nodes chained together with token
  // chain and flag operands which copy the outgoing args into registers.
  // The InGlue in necessary since all emitted instructions must be
  // stuck together.
  SDValue InGlue;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3200-3203
```cpp
  for (auto &R : RegsToPass) {
    Chain = CLI.DAG.getCopyToReg(Chain, CLI.DL, R.first, R.second, InGlue);
    InGlue = Chain.getValue(1);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3205-3208
```cpp
  // Add argument registers to the end of the list so that they are
  // known live into the call.
  for (auto &R : RegsToPass)
    Ops.push_back(CLI.DAG.getRegister(R.first, R.second.getValueType()));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3210-3224
```cpp
  // Add a register mask operand representing the call-preserved registers.
  const TargetRegisterInfo *TRI = Subtarget.getRegisterInfo();
  const uint32_t *Mask =
      TRI->getCallPreservedMask(CLI.DAG.getMachineFunction(), CLI.CallConv);
  assert(Mask && "Missing call preserved mask for calling convention");
  if (Subtarget.inMips16HardFloat()) {
    if (GlobalAddressSDNode *G = dyn_cast<GlobalAddressSDNode>(CLI.Callee)) {
      StringRef Sym = G->getGlobal()->getName();
      Function *F = G->getGlobal()->getParent()->getFunction(Sym);
      if (F && F->hasFnAttribute("__Mips16RetHelper")) {
        Mask = MipsRegisterInfo::getMips16RetHelperMask();
      }
    }
  }
  Ops.push_back(CLI.DAG.getRegisterMask(Mask));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3226-3228
```cpp
  if (InGlue.getNode())
    Ops.push_back(InGlue);
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3230-3244
```cpp
void MipsTargetLowering::AdjustInstrPostInstrSelection(MachineInstr &MI,
                                                       SDNode *Node) const {
  switch (MI.getOpcode()) {
    default:
      return;
    case Mips::JALR:
    case Mips::JALRPseudo:
    case Mips::JALR64:
    case Mips::JALR64Pseudo:
    case Mips::JALR16_MM:
    case Mips::JALRC16_MMR6:
    case Mips::TAILCALLREG:
    case Mips::TAILCALLREG64:
    case Mips::TAILCALLR6REG:
    case Mips::TAILCALL64R6REG:
```
- EN: Implements `MipsTargetLowering::AdjustInstrPostInstrSelection`, a mutation/build routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::AdjustInstrPostInstrSelection`，它是一个围绕机器指令展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3245-3259
```cpp
    case Mips::TAILCALLREG_MM:
    case Mips::TAILCALLREG_MMR6: {
      if (!EmitJalrReloc ||
          Subtarget.inMips16Mode() ||
          !isPositionIndependent() ||
          Node->getNumOperands() < 1 ||
          Node->getOperand(0).getNumOperands() < 2) {
        return;
      }
      // We are after the callee address, set by LowerCall().
      // If added to MI, asm printer will emit .reloc R_MIPS_JALR for the
      // symbol.
      const SDValue TargetAddr = Node->getOperand(0).getOperand(1);
      StringRef Sym;
      if (const GlobalAddressSDNode *G =
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3260-3274
```cpp
              dyn_cast_or_null<const GlobalAddressSDNode>(TargetAddr)) {
        // We must not emit the R_MIPS_JALR relocation against data symbols
        // since this will cause run-time crashes if the linker replaces the
        // call instruction with a relative branch to the data symbol.
        if (!isa<Function>(G->getGlobal())) {
          LLVM_DEBUG(dbgs() << "Not adding R_MIPS_JALR against data symbol "
                            << G->getGlobal()->getName() << "\n");
          return;
        }
        Sym = G->getGlobal()->getName();
      }
      else if (const ExternalSymbolSDNode *ES =
                   dyn_cast_or_null<const ExternalSymbolSDNode>(TargetAddr)) {
        Sym = ES->getSymbol();
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3276-3277
```cpp
      if (Sym.empty())
        return;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3279-3285
```cpp
      MachineFunction *MF = MI.getParent()->getParent();
      MCSymbol *S = MF->getContext().getOrCreateSymbol(Sym);
      LLVM_DEBUG(dbgs() << "Adding R_MIPS_JALR against " << Sym << "\n");
      MI.addOperand(MachineOperand::CreateMCSymbol(S, MipsII::MO_JALR));
    }
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3287-3301
```cpp
/// LowerCall - functions arguments are copied from virtual regs to
/// (physical regs)/(stack frame), CALLSEQ_START and CALLSEQ_END are emitted.
SDValue
MipsTargetLowering::LowerCall(TargetLowering::CallLoweringInfo &CLI,
                              SmallVectorImpl<SDValue> &InVals) const {
  SelectionDAG &DAG                     = CLI.DAG;
  SDLoc DL                              = CLI.DL;
  SmallVectorImpl<ISD::OutputArg> &Outs = CLI.Outs;
  SmallVectorImpl<SDValue> &OutVals     = CLI.OutVals;
  SmallVectorImpl<ISD::InputArg> &Ins   = CLI.Ins;
  SDValue Chain                         = CLI.Chain;
  SDValue Callee                        = CLI.Callee;
  bool &IsTailCall                      = CLI.IsTailCall;
  CallingConv::ID CallConv              = CLI.CallConv;
  bool IsVarArg                         = CLI.IsVarArg;
```
- EN: Implements `MipsTargetLowering::LowerCall`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::LowerCall`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3302-3302
```cpp
  const CallBase *CB = CLI.CB;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3304-3308
```cpp
  MachineFunction &MF = DAG.getMachineFunction();
  MachineFrameInfo &MFI = MF.getFrameInfo();
  const TargetFrameLowering *TFL = Subtarget.getFrameLowering();
  MipsFunctionInfo *FuncInfo = MF.getInfo<MipsFunctionInfo>();
  bool IsPIC = isPositionIndependent();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3310-3314
```cpp
  // Analyze operands of the call, assigning locations to each operand.
  SmallVector<CCValAssign, 16> ArgLocs;
  MipsCCState CCInfo(
      CallConv, IsVarArg, DAG.getMachineFunction(), ArgLocs, *DAG.getContext(),
      MipsCCState::getSpecialCallingConvForCallee(Callee.getNode(), Subtarget));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3316-3317
```cpp
  const ExternalSymbolSDNode *ES =
      dyn_cast_or_null<const ExternalSymbolSDNode>(Callee.getNode());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3319-3333
```cpp
  // There is one case where CALLSEQ_START..CALLSEQ_END can be nested, which
  // is during the lowering of a call with a byval argument which produces
  // a call to memcpy. For the O32 case, this causes the caller to allocate
  // stack space for the reserved argument area for the callee, then recursively
  // again for the memcpy call. In the NEWABI case, this doesn't occur as those
  // ABIs mandate that the callee allocates the reserved argument area. We do
  // still produce nested CALLSEQ_START..CALLSEQ_END with zero space though.
  //
  // If the callee has a byval argument and memcpy is used, we are mandated
  // to already have produced a reserved argument area for the callee for O32.
  // Therefore, the reserved argument area can be reused for both calls.
  //
  // Other cases of calling memcpy cannot have a chain with a CALLSEQ_START
  // present, as we have yet to hook that node onto the chain.
  //
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 3334-3339
```cpp
  // Hence, the CALLSEQ_START and CALLSEQ_END nodes can be eliminated in this
  // case. GCC does a similar trick, in that wherever possible, it calculates
  // the maximum out going argument area (including the reserved area), and
  // preallocates the stack space on entrance to the caller.
  //
  // FIXME: We should do the same for efficiency and space.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 3341-3345
```cpp
  // Note: The check on the calling convention below must match
  //       MipsABIInfo::GetCalleeAllocdArgSizeInBytes().
  bool MemcpyInByVal = ES && StringRef(ES->getSymbol()) == "memcpy" &&
                       CallConv != CallingConv::Fast &&
                       Chain.getOpcode() == ISD::CALLSEQ_START;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3347-3351
```cpp
  // Allocate the reserved argument area. It seems strange to do this from the
  // caller side but removing it breaks the frame size calculation.
  unsigned ReservedArgArea =
      MemcpyInByVal ? 0 : ABI.GetCalleeAllocdArgSizeInBytes(CallConv);
  CCInfo.AllocateStack(ReservedArgArea, Align(1));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3353-3353
```cpp
  CCInfo.AnalyzeCallOperands(Outs, CC_Mips);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3355-3356
```cpp
  // Get a count of how many bytes are to be pushed on the stack.
  unsigned StackSize = CCInfo.getStackSize();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3358-3361
```cpp
  // Call site info for function parameters tracking and call base type info.
  MachineFunction::CallSiteInfo CSInfo;
  // Set type id for call site info.
  setTypeIdForCallsiteInfo(CB, MF, CSInfo);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3363-3377
```cpp
  // Check if it's really possible to do a tail call.
  // For non-musttail calls, restrict to functions that won't require $gp
  // restoration. In PIC mode, calling external functions via tail call can
  // cause issues with $gp register handling (see D24763).
  bool IsMustTail = CLI.CB && CLI.CB->isMustTailCall();
  bool CalleeIsLocal = true;
  if (GlobalAddressSDNode *G = dyn_cast<GlobalAddressSDNode>(Callee)) {
    const GlobalValue *GV = G->getGlobal();
    bool HasLocalLinkage = GV->hasLocalLinkage() || GV->hasPrivateLinkage();
    bool HasHiddenVisibility =
        GV->hasHiddenVisibility() || GV->hasProtectedVisibility();
    if (GV->isDeclarationForLinker())
      CalleeIsLocal = HasLocalLinkage || HasHiddenVisibility;
    else
      CalleeIsLocal = GV->isDSOLocal();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3378-3378
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3380-3394
```cpp
  if (IsTailCall) {
    if (!UseMipsTailCalls) {
      IsTailCall = false;
      if (IsMustTail)
        report_fatal_error("failed to perform tail call elimination on a call "
                           "site marked musttail");
    } else {
      bool Eligible = isEligibleForTailCallOptimization(
          CCInfo, StackSize, *MF.getInfo<MipsFunctionInfo>());
      if (!Eligible || !CalleeIsLocal) {
        IsTailCall = false;
        if (IsMustTail)
          report_fatal_error(
              "failed to perform tail call elimination on a call "
              "site marked musttail");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3395-3397
```cpp
      }
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3399-3400
```cpp
  if (IsTailCall)
    ++NumTailCalls;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3402-3406
```cpp
  // Chain is the output chain of the last Load/Store or CopyToReg node.
  // ByValChain is the output chain of the last Memcpy node created for copying
  // byval arguments to the stack.
  unsigned StackAlignment = TFL->getStackAlignment();
  StackSize = alignTo(StackSize, StackAlignment);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3408-3409
```cpp
  if (!(IsTailCall || MemcpyInByVal))
    Chain = DAG.getCALLSEQ_START(Chain, StackSize, 0, DL);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3411-3415
```cpp
  SDValue StackPtr =
      DAG.getCopyFromReg(Chain, DL, ABI.IsN64() ? Mips::SP_64 : Mips::SP,
                         getPointerTy(DAG.getDataLayout()));
  std::deque<std::pair<unsigned, SDValue>> RegsToPass;
  SmallVector<SDValue, 8> MemOpChains;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3417-3417
```cpp
  CCInfo.rewindByValRegsInfo();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3419-3425
```cpp
  // Walk the register/memloc assignments, inserting copies/loads.
  for (unsigned i = 0, e = ArgLocs.size(), OutIdx = 0; i != e; ++i, ++OutIdx) {
    SDValue Arg = OutVals[OutIdx];
    CCValAssign &VA = ArgLocs[i];
    MVT ValVT = VA.getValVT(), LocVT = VA.getLocVT();
    ISD::ArgFlagsTy Flags = Outs[OutIdx].Flags;
    bool UseUpperBits = false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3427-3431
```cpp
    // ByVal Arg.
    if (Flags.isByVal()) {
      unsigned FirstByValReg, LastByValReg;
      unsigned ByValIdx = CCInfo.getInRegsParamsProcessed();
      CCInfo.getInRegsParamInfo(ByValIdx, FirstByValReg, LastByValReg);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3433-3443
```cpp
      assert(Flags.getByValSize() &&
             "ByVal args of size 0 should have been ignored by front-end.");
      assert(ByValIdx < CCInfo.getInRegsParamsCount());
      assert(!IsTailCall &&
             "Do not tail-call optimize if there is a byval argument.");
      passByValArg(Chain, DL, RegsToPass, MemOpChains, StackPtr, MFI, DAG, Arg,
                   FirstByValReg, LastByValReg, Flags, Subtarget.isLittle(),
                   VA);
      CCInfo.nextInRegsParam();
      continue;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3445-3459
```cpp
    // Promote the value if needed.
    switch (VA.getLocInfo()) {
    default:
      llvm_unreachable("Unknown loc info!");
    case CCValAssign::Full:
      if (VA.isRegLoc()) {
        if ((ValVT == MVT::f32 && LocVT == MVT::i32) ||
            (ValVT == MVT::f64 && LocVT == MVT::i64) ||
            (ValVT == MVT::i64 && LocVT == MVT::f64))
          Arg = DAG.getNode(ISD::BITCAST, DL, LocVT, Arg);
        else if (ValVT == MVT::f64 && LocVT == MVT::i32) {
          SDValue Lo = DAG.getNode(MipsISD::ExtractElementF64, DL, MVT::i32,
                                   Arg, DAG.getConstant(0, DL, MVT::i32));
          SDValue Hi = DAG.getNode(MipsISD::ExtractElementF64, DL, MVT::i32,
                                   Arg, DAG.getConstant(1, DL, MVT::i32));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3460-3461
```cpp
          if (!Subtarget.isLittle())
            std::swap(Lo, Hi);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3463-3463
```cpp
          assert(VA.needsCustom());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3465-3479
```cpp
          Register LocRegLo = VA.getLocReg();
          Register LocRegHigh = ArgLocs[++i].getLocReg();
          RegsToPass.push_back(std::make_pair(LocRegLo, Lo));
          RegsToPass.push_back(std::make_pair(LocRegHigh, Hi));
          continue;
        }
      }
      break;
    case CCValAssign::BCvt:
      Arg = DAG.getNode(ISD::BITCAST, DL, LocVT, Arg);
      break;
    case CCValAssign::SExtUpper:
      UseUpperBits = true;
      [[fallthrough]];
    case CCValAssign::SExt:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3480-3494
```cpp
      Arg = DAG.getNode(ISD::SIGN_EXTEND, DL, LocVT, Arg);
      break;
    case CCValAssign::ZExtUpper:
      UseUpperBits = true;
      [[fallthrough]];
    case CCValAssign::ZExt:
      Arg = DAG.getNode(ISD::ZERO_EXTEND, DL, LocVT, Arg);
      break;
    case CCValAssign::AExtUpper:
      UseUpperBits = true;
      [[fallthrough]];
    case CCValAssign::AExt:
      Arg = DAG.getNode(ISD::ANY_EXTEND, DL, LocVT, Arg);
      break;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3496-3502
```cpp
    if (UseUpperBits) {
      unsigned ValSizeInBits = Outs[OutIdx].ArgVT.getSizeInBits();
      unsigned LocSizeInBits = VA.getLocVT().getSizeInBits();
      Arg = DAG.getNode(
          ISD::SHL, DL, VA.getLocVT(), Arg,
          DAG.getConstant(LocSizeInBits - ValSizeInBits, DL, VA.getLocVT()));
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3504-3507
```cpp
    // Arguments that can be passed on register must be kept at
    // RegsToPass vector
    if (VA.isRegLoc()) {
      RegsToPass.push_back(std::make_pair(VA.getLocReg(), Arg));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3509-3512
```cpp
      // If the parameter is passed through reg $D, which splits into
      // two physical registers, avoid creating call site info.
      if (Mips::AFGR64RegClass.contains(VA.getLocReg()))
        continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3514-3517
```cpp
      // Collect CSInfo about which register passes which parameter.
      const TargetOptions &Options = DAG.getTarget().Options;
      if (Options.EmitCallSiteInfo)
        CSInfo.ArgRegPairs.emplace_back(VA.getLocReg(), i);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3519-3520
```cpp
      continue;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3522-3523
```cpp
    // Register can't get to this point...
    assert(VA.isMemLoc());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3525-3529
```cpp
    // emit ISD::STORE whichs stores the
    // parameter value to a stack Location
    MemOpChains.push_back(passArgOnStack(StackPtr, VA.getLocMemOffset(),
                                         Chain, Arg, DL, IsTailCall, DAG));
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3531-3534
```cpp
  // Transform all store nodes into one single node because all store
  // nodes are independent of each other.
  if (!MemOpChains.empty())
    Chain = DAG.getNode(ISD::TokenFactor, DL, MVT::Other, MemOpChains);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3536-3538
```cpp
  // If the callee is a GlobalAddress/ExternalSymbol node (quite common, every
  // direct call is) turn it into a TargetGlobalAddress/TargetExternalSymbol
  // node so that legalize doesn't hack it.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 3540-3541
```cpp
  EVT Ty = Callee.getValueType();
  bool GlobalOrExternal = false, IsCallReloc = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3543-3557
```cpp
  // The long-calls feature is ignored in case of PIC.
  // While we do not support -mshared / -mno-shared properly,
  // ignore long-calls in case of -mabicalls too.
  if (!Subtarget.isABICalls() && !IsPIC) {
    // If the function should be called using "long call",
    // get its address into a register to prevent using
    // of the `jal` instruction for the direct call.
    if (auto *N = dyn_cast<ExternalSymbolSDNode>(Callee)) {
      if (Subtarget.useLongCalls())
        Callee = Subtarget.hasSym32()
                     ? getAddrNonPIC(N, SDLoc(N), Ty, DAG)
                     : getAddrNonPICSym64(N, SDLoc(N), Ty, DAG);
    } else if (auto *N = dyn_cast<GlobalAddressSDNode>(Callee)) {
      bool UseLongCalls = Subtarget.useLongCalls();
      // If the function has long-call/far/near attribute
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3558-3570
```cpp
      // it overrides command line switch pased to the backend.
      if (auto *F = dyn_cast<Function>(N->getGlobal())) {
        if (F->hasFnAttribute("long-call"))
          UseLongCalls = true;
        else if (F->hasFnAttribute("short-call"))
          UseLongCalls = false;
      }
      if (UseLongCalls)
        Callee = Subtarget.hasSym32()
                     ? getAddrNonPIC(N, SDLoc(N), Ty, DAG)
                     : getAddrNonPICSym64(N, SDLoc(N), Ty, DAG);
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3572-3583
```cpp
  bool InternalLinkage = false;
  if (GlobalAddressSDNode *G = dyn_cast<GlobalAddressSDNode>(Callee)) {
    if (Subtarget.isTargetCOFF() &&
        G->getGlobal()->hasDLLImportStorageClass()) {
      assert(Subtarget.isTargetWindows() &&
             "Windows is the only supported COFF target");
      auto PtrInfo = MachinePointerInfo();
      Callee = DAG.getLoad(Ty, DL, Chain,
                           getDllimportSymbol(G, SDLoc(G), Ty, DAG), PtrInfo);
    } else if (IsPIC) {
      const GlobalValue *Val = G->getGlobal();
      InternalLinkage = Val->hasInternalLinkage();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3585-3599
```cpp
      if (InternalLinkage)
        Callee = getAddrLocal(G, DL, Ty, DAG, ABI.IsN32() || ABI.IsN64());
      else if (Subtarget.useXGOT()) {
        Callee = getAddrGlobalLargeGOT(G, DL, Ty, DAG, MipsII::MO_CALL_HI16,
                                       MipsII::MO_CALL_LO16, Chain,
                                       FuncInfo->callPtrInfo(MF, Val));
        IsCallReloc = true;
      } else {
        Callee = getAddrGlobal(G, DL, Ty, DAG, MipsII::MO_GOT_CALL, Chain,
                               FuncInfo->callPtrInfo(MF, Val));
        IsCallReloc = true;
      }
    } else
      Callee = DAG.getTargetGlobalAddress(G->getGlobal(), DL,
                                          getPointerTy(DAG.getDataLayout()), 0,
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3600-3604
```cpp
                                          MipsII::MO_NO_FLAG);
    GlobalOrExternal = true;
  }
  else if (ExternalSymbolSDNode *S = dyn_cast<ExternalSymbolSDNode>(Callee)) {
    const char *Sym = S->getSymbol();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3606-3618
```cpp
    if (!IsPIC) // static
      Callee = DAG.getTargetExternalSymbol(
          Sym, getPointerTy(DAG.getDataLayout()), MipsII::MO_NO_FLAG);
    else if (Subtarget.useXGOT()) {
      Callee = getAddrGlobalLargeGOT(S, DL, Ty, DAG, MipsII::MO_CALL_HI16,
                                     MipsII::MO_CALL_LO16, Chain,
                                     FuncInfo->callPtrInfo(MF, Sym));
      IsCallReloc = true;
    } else { // PIC
      Callee = getAddrGlobal(S, DL, Ty, DAG, MipsII::MO_GOT_CALL, Chain,
                             FuncInfo->callPtrInfo(MF, Sym));
      IsCallReloc = true;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3620-3621
```cpp
    GlobalOrExternal = true;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3623-3624
```cpp
  SmallVector<SDValue, 8> Ops(1, Chain);
  SDVTList NodeTys = DAG.getVTList(MVT::Other, MVT::Glue);
```
- EN: Declares `Ops`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `Ops`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3626-3627
```cpp
  getOpndList(Ops, RegsToPass, IsPIC, GlobalOrExternal, InternalLinkage,
              IsCallReloc, CLI, Callee, Chain);
```
- EN: Declares `getOpndList`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getOpndList`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3629-3634
```cpp
  if (IsTailCall) {
    MF.getFrameInfo().setHasTailCall();
    SDValue Ret = DAG.getNode(MipsISD::TailCall, DL, MVT::Other, Ops);
    DAG.addCallSiteInfo(Ret.getNode(), std::move(CSInfo));
    return Ret;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3636-3637
```cpp
  Chain = DAG.getNode(MipsISD::JmpLink, DL, NodeTys, Ops);
  SDValue InGlue = Chain.getValue(1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3639-3639
```cpp
  DAG.addCallSiteInfo(Chain.getNode(), std::move(CSInfo));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3641-3646
```cpp
  // Create the CALLSEQ_END node in the case of where it is not a call to
  // memcpy.
  if (!(MemcpyInByVal)) {
    Chain = DAG.getCALLSEQ_END(Chain, StackSize, 0, InGlue, DL);
    InGlue = Chain.getValue(1);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3648-3652
```cpp
  // Handle result values, copying them out of physregs into vregs that we
  // return.
  return LowerCallResult(Chain, InGlue, CallConv, IsVarArg, Ins, DL, DAG,
                         InVals, CLI);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3654-3664
```cpp
/// LowerCallResult - Lower the result values of a call into the
/// appropriate copies out of appropriate physical registers.
SDValue MipsTargetLowering::LowerCallResult(
    SDValue Chain, SDValue InGlue, CallingConv::ID CallConv, bool IsVarArg,
    const SmallVectorImpl<ISD::InputArg> &Ins, const SDLoc &DL,
    SelectionDAG &DAG, SmallVectorImpl<SDValue> &InVals,
    TargetLowering::CallLoweringInfo &CLI) const {
  // Assign locations to each value returned by this call.
  SmallVector<CCValAssign, 16> RVLocs;
  MipsCCState CCInfo(CallConv, IsVarArg, DAG.getMachineFunction(), RVLocs,
                     *DAG.getContext());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3666-3666
```cpp
  CCInfo.AnalyzeCallResult(Ins, RetCC_Mips);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3668-3671
```cpp
  // Copy all of the result registers out of their specified physreg.
  for (unsigned i = 0; i != RVLocs.size(); ++i) {
    CCValAssign &VA = RVLocs[i];
    assert(VA.isRegLoc() && "Can only return in registers!");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3673-3676
```cpp
    SDValue Val = DAG.getCopyFromReg(Chain, DL, RVLocs[i].getLocReg(),
                                     RVLocs[i].getLocVT(), InGlue);
    Chain = Val.getValue(1);
    InGlue = Val.getValue(2);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3678-3686
```cpp
    if (VA.isUpperBitsInLoc()) {
      unsigned ValSizeInBits = Ins[i].ArgVT.getSizeInBits();
      unsigned LocSizeInBits = VA.getLocVT().getSizeInBits();
      unsigned Shift =
          VA.getLocInfo() == CCValAssign::ZExtUpper ? ISD::SRL : ISD::SRA;
      Val = DAG.getNode(
          Shift, DL, VA.getLocVT(), Val,
          DAG.getConstant(LocSizeInBits - ValSizeInBits, DL, VA.getLocVT()));
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3688-3702
```cpp
    switch (VA.getLocInfo()) {
    default:
      llvm_unreachable("Unknown loc info!");
    case CCValAssign::Full:
      break;
    case CCValAssign::BCvt:
      Val = DAG.getNode(ISD::BITCAST, DL, VA.getValVT(), Val);
      break;
    case CCValAssign::AExt:
    case CCValAssign::AExtUpper:
      Val = DAG.getNode(ISD::TRUNCATE, DL, VA.getValVT(), Val);
      break;
    case CCValAssign::ZExt:
    case CCValAssign::ZExtUpper:
      Val = DAG.getNode(ISD::AssertZext, DL, VA.getLocVT(), Val,
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3703-3712
```cpp
                        DAG.getValueType(VA.getValVT()));
      Val = DAG.getNode(ISD::TRUNCATE, DL, VA.getValVT(), Val);
      break;
    case CCValAssign::SExt:
    case CCValAssign::SExtUpper:
      Val = DAG.getNode(ISD::AssertSext, DL, VA.getLocVT(), Val,
                        DAG.getValueType(VA.getValVT()));
      Val = DAG.getNode(ISD::TRUNCATE, DL, VA.getValVT(), Val);
      break;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3714-3715
```cpp
    InVals.push_back(Val);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3717-3718
```cpp
  return Chain;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3720-3724
```cpp
static SDValue UnpackFromArgumentSlot(SDValue Val, const CCValAssign &VA,
                                      EVT ArgVT, const SDLoc &DL,
                                      SelectionDAG &DAG) {
  MVT LocVT = VA.getLocVT();
  EVT ValVT = VA.getValVT();
```
- EN: Implements `UnpackFromArgumentSlot`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `UnpackFromArgumentSlot`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3726-3740
```cpp
  // Shift into the upper bits if necessary.
  switch (VA.getLocInfo()) {
  default:
    break;
  case CCValAssign::AExtUpper:
  case CCValAssign::SExtUpper:
  case CCValAssign::ZExtUpper: {
    unsigned ValSizeInBits = ArgVT.getSizeInBits();
    unsigned LocSizeInBits = VA.getLocVT().getSizeInBits();
    unsigned Opcode =
        VA.getLocInfo() == CCValAssign::ZExtUpper ? ISD::SRL : ISD::SRA;
    Val = DAG.getNode(
        Opcode, DL, VA.getLocVT(), Val,
        DAG.getConstant(LocSizeInBits - ValSizeInBits, DL, VA.getLocVT()));
    break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3741-3742
```cpp
  }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3744-3758
```cpp
  // If this is an value smaller than the argument slot size (32-bit for O32,
  // 64-bit for N32/N64), it has been promoted in some way to the argument slot
  // size. Extract the value and insert any appropriate assertions regarding
  // sign/zero extension.
  switch (VA.getLocInfo()) {
  default:
    llvm_unreachable("Unknown loc info!");
  case CCValAssign::Full:
    break;
  case CCValAssign::AExtUpper:
  case CCValAssign::AExt:
    Val = DAG.getNode(ISD::TRUNCATE, DL, ValVT, Val);
    break;
  case CCValAssign::SExtUpper:
  case CCValAssign::SExt:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3759-3770
```cpp
    Val = DAG.getNode(ISD::AssertSext, DL, LocVT, Val, DAG.getValueType(ValVT));
    Val = DAG.getNode(ISD::TRUNCATE, DL, ValVT, Val);
    break;
  case CCValAssign::ZExtUpper:
  case CCValAssign::ZExt:
    Val = DAG.getNode(ISD::AssertZext, DL, LocVT, Val, DAG.getValueType(ValVT));
    Val = DAG.getNode(ISD::TRUNCATE, DL, ValVT, Val);
    break;
  case CCValAssign::BCvt:
    Val = DAG.getNode(ISD::BITCAST, DL, ValVT, Val);
    break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3772-3773
```cpp
  return Val;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3775-3786
```cpp
//===----------------------------------------------------------------------===//
//             Formal Arguments Calling Convention Implementation
//===----------------------------------------------------------------------===//
/// LowerFormalArguments - transform physical registers into virtual registers
/// and generate load operations for arguments places on the stack.
SDValue MipsTargetLowering::LowerFormalArguments(
    SDValue Chain, CallingConv::ID CallConv, bool IsVarArg,
    const SmallVectorImpl<ISD::InputArg> &Ins, const SDLoc &DL,
    SelectionDAG &DAG, SmallVectorImpl<SDValue> &InVals) const {
  MachineFunction &MF = DAG.getMachineFunction();
  MachineFrameInfo &MFI = MF.getFrameInfo();
  MipsFunctionInfo *MipsFI = MF.getInfo<MipsFunctionInfo>();
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 3788-3788
```cpp
  MipsFI->setVarArgsFrameIndex(0);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3790-3791
```cpp
  // Used with vargs to acumulate store chains.
  std::vector<SDValue> OutChains;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3793-3799
```cpp
  // Assign locations to all of the incoming arguments.
  SmallVector<CCValAssign, 16> ArgLocs;
  MipsCCState CCInfo(CallConv, IsVarArg, DAG.getMachineFunction(), ArgLocs,
                     *DAG.getContext());
  CCInfo.AllocateStack(ABI.GetCalleeAllocdArgSizeInBytes(CallConv), Align(1));
  const Function &Func = DAG.getMachineFunction().getFunction();
  Function::const_arg_iterator FuncArg = Func.arg_begin();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3801-3803
```cpp
  if (Func.hasFnAttribute("interrupt") && !Func.arg_empty())
    report_fatal_error(
        "Functions with the interrupt attribute cannot have arguments!");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3805-3807
```cpp
  CCInfo.AnalyzeFormalArguments(Ins, CC_Mips_FixedArg);
  MipsFI->setFormalArgInfo(CCInfo.getStackSize(),
                           CCInfo.getInRegsParamsCount() > 0);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3809-3810
```cpp
  unsigned CurArgIdx = 0;
  CCInfo.rewindByValRegsInfo();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3812-3820
```cpp
  for (unsigned i = 0, e = ArgLocs.size(), InsIdx = 0; i != e; ++i, ++InsIdx) {
    CCValAssign &VA = ArgLocs[i];
    if (Ins[InsIdx].isOrigArg()) {
      std::advance(FuncArg, Ins[InsIdx].getOrigArgIndex() - CurArgIdx);
      CurArgIdx = Ins[InsIdx].getOrigArgIndex();
    }
    EVT ValVT = VA.getValVT();
    ISD::ArgFlagsTy Flags = Ins[InsIdx].Flags;
    bool IsRegLoc = VA.isRegLoc();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3822-3826
```cpp
    if (Flags.isByVal()) {
      assert(Ins[InsIdx].isOrigArg() && "Byval arguments cannot be implicit");
      unsigned FirstByValReg, LastByValReg;
      unsigned ByValIdx = CCInfo.getInRegsParamsProcessed();
      CCInfo.getInRegsParamInfo(ByValIdx, FirstByValReg, LastByValReg);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3828-3835
```cpp
      assert(Flags.getByValSize() &&
             "ByVal args of size 0 should have been ignored by front-end.");
      assert(ByValIdx < CCInfo.getInRegsParamsCount());
      copyByValRegs(Chain, DL, OutChains, DAG, Flags, InVals, &*FuncArg,
                    FirstByValReg, LastByValReg, VA, CCInfo);
      CCInfo.nextInRegsParam();
      continue;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3837-3841
```cpp
    // Arguments stored on registers
    if (IsRegLoc) {
      MVT RegVT = VA.getLocVT();
      Register ArgReg = VA.getLocReg();
      const TargetRegisterClass *RC = getRegClassFor(RegVT);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3843-3846
```cpp
      // Transform the arguments stored on
      // physical registers into virtual ones
      unsigned Reg = addLiveIn(DAG.getMachineFunction(), ArgReg, RC);
      SDValue ArgValue = DAG.getCopyFromReg(Chain, DL, Reg, RegVT);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3848-3849
```cpp
      ArgValue =
          UnpackFromArgumentSlot(ArgValue, VA, Ins[InsIdx].ArgVT, DL, DAG);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3851-3865
```cpp
      // Handle floating point arguments passed in integer registers and
      // long double arguments passed in floating point registers.
      if ((RegVT == MVT::i32 && ValVT == MVT::f32) ||
          (RegVT == MVT::i64 && ValVT == MVT::f64) ||
          (RegVT == MVT::f64 && ValVT == MVT::i64))
        ArgValue = DAG.getNode(ISD::BITCAST, DL, ValVT, ArgValue);
      else if (ABI.IsO32() && RegVT == MVT::i32 &&
               ValVT == MVT::f64) {
        assert(VA.needsCustom() && "Expected custom argument for f64 split");
        CCValAssign &NextVA = ArgLocs[++i];
        unsigned Reg2 =
            addLiveIn(DAG.getMachineFunction(), NextVA.getLocReg(), RC);
        SDValue ArgValue2 = DAG.getCopyFromReg(Chain, DL, Reg2, RegVT);
        if (!Subtarget.isLittle())
          std::swap(ArgValue, ArgValue2);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3866-3868
```cpp
        ArgValue = DAG.getNode(MipsISD::BuildPairF64, DL, MVT::f64,
                               ArgValue, ArgValue2);
      }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3870-3872
```cpp
      InVals.push_back(ArgValue);
    } else { // VA.isRegLoc()
      MVT LocVT = VA.getLocVT();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3874-3874
```cpp
      assert(!VA.needsCustom() && "unexpected custom memory argument");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3876-3877
```cpp
      // Only arguments pased on the stack should make it here. 
      assert(VA.isMemLoc());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3879-3881
```cpp
      // The stack pointer offset is relative to the caller stack frame.
      int FI = MFI.CreateFixedObject(LocVT.getSizeInBits() / 8,
                                     VA.getLocMemOffset(), true);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3883-3888
```cpp
      // Create load nodes to retrieve arguments from the stack
      SDValue FIN = DAG.getFrameIndex(FI, getPointerTy(DAG.getDataLayout()));
      SDValue ArgValue = DAG.getLoad(
          LocVT, DL, Chain, FIN,
          MachinePointerInfo::getFixedStack(DAG.getMachineFunction(), FI));
      OutChains.push_back(ArgValue.getValue(1));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3890-3891
```cpp
      ArgValue =
          UnpackFromArgumentSlot(ArgValue, VA, Ins[InsIdx].ArgVT, DL, DAG);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3893-3895
```cpp
      InVals.push_back(ArgValue);
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3897-3897
```cpp
  for (unsigned i = 0, e = ArgLocs.size(), InsIdx = 0; i != e; ++i, ++InsIdx) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3899-3902
```cpp
    if (ArgLocs[i].needsCustom()) {
      ++i;
      continue;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3904-3918
```cpp
    // The mips ABIs for returning structs by value requires that we copy
    // the sret argument into $v0 for the return. Save the argument into
    // a virtual register so that we can access it from the return points.
    if (Ins[InsIdx].Flags.isSRet()) {
      unsigned Reg = MipsFI->getSRetReturnReg();
      if (!Reg) {
        Reg = MF.getRegInfo().createVirtualRegister(
            getRegClassFor(ABI.IsN64() ? MVT::i64 : MVT::i32));
        MipsFI->setSRetReturnReg(Reg);
      }
      SDValue Copy = DAG.getCopyToReg(DAG.getEntryNode(), DL, Reg, InVals[i]);
      Chain = DAG.getNode(ISD::TokenFactor, DL, MVT::Other, Copy, Chain);
      break;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3920-3921
```cpp
  if (IsVarArg)
    writeVarArgRegs(OutChains, Chain, DL, DAG, CCInfo);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3923-3928
```cpp
  // All stores are grouped in one node to allow the matching between
  // the size of Ins and InVals. This only happens when on varg functions
  if (!OutChains.empty()) {
    OutChains.push_back(Chain);
    Chain = DAG.getNode(ISD::TokenFactor, DL, MVT::Other, OutChains);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3930-3931
```cpp
  return Chain;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3933-3935
```cpp
//===----------------------------------------------------------------------===//
//               Return Value Calling Convention Implementation
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 3937-3945
```cpp
bool
MipsTargetLowering::CanLowerReturn(CallingConv::ID CallConv,
                                   MachineFunction &MF, bool IsVarArg,
                                   const SmallVectorImpl<ISD::OutputArg> &Outs,
                                   LLVMContext &Context, const Type *RetTy) const {
  SmallVector<CCValAssign, 16> RVLocs;
  MipsCCState CCInfo(CallConv, IsVarArg, MF, RVLocs, Context);
  return CCInfo.CheckReturn(Outs, RetCC_Mips);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3947-3950
```cpp
bool MipsTargetLowering::shouldSignExtendTypeInLibCall(Type *Ty,
                                                       bool IsSigned) const {
  if ((ABI.IsN32() || ABI.IsN64()) && Ty->isIntegerTy(32))
    return true;
```
- EN: Implements `MipsTargetLowering::shouldSignExtendTypeInLibCall`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::shouldSignExtendTypeInLibCall`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3952-3953
```cpp
  return IsSigned;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3955-3960
```cpp
SDValue
MipsTargetLowering::LowerInterruptReturn(SmallVectorImpl<SDValue> &RetOps,
                                         const SDLoc &DL,
                                         SelectionDAG &DAG) const {
  MachineFunction &MF = DAG.getMachineFunction();
  MipsFunctionInfo *MipsFI = MF.getInfo<MipsFunctionInfo>();
```
- EN: Implements `MipsTargetLowering::LowerInterruptReturn`, a lowering routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::LowerInterruptReturn`，它是一个围绕机器函数状态展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3962-3962
```cpp
  MipsFI->setISR();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3964-3965
```cpp
  return DAG.getNode(MipsISD::ERet, DL, MVT::Other, RetOps);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3967-3976
```cpp
SDValue
MipsTargetLowering::LowerReturn(SDValue Chain, CallingConv::ID CallConv,
                                bool IsVarArg,
                                const SmallVectorImpl<ISD::OutputArg> &Outs,
                                const SmallVectorImpl<SDValue> &OutVals,
                                const SDLoc &DL, SelectionDAG &DAG) const {
  // CCValAssign - represent the assignment of
  // the return value to a location
  SmallVector<CCValAssign, 16> RVLocs;
  MachineFunction &MF = DAG.getMachineFunction();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3978-3979
```cpp
  // CCState - Info about the registers and stack slot.
  MipsCCState CCInfo(CallConv, IsVarArg, MF, RVLocs, *DAG.getContext());
```
- EN: Declares `CCInfo`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `CCInfo`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3981-3982
```cpp
  // Analyze return values.
  CCInfo.AnalyzeReturn(Outs, RetCC_Mips);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3984-3985
```cpp
  SDValue Glue;
  SmallVector<SDValue, 4> RetOps(1, Chain);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3987-3992
```cpp
  // Copy the result values into the output registers.
  for (unsigned i = 0; i != RVLocs.size(); ++i) {
    SDValue Val = OutVals[i];
    CCValAssign &VA = RVLocs[i];
    assert(VA.isRegLoc() && "Can only return in registers!");
    bool UseUpperBits = false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3994-4008
```cpp
    switch (VA.getLocInfo()) {
    default:
      llvm_unreachable("Unknown loc info!");
    case CCValAssign::Full:
      break;
    case CCValAssign::BCvt:
      Val = DAG.getNode(ISD::BITCAST, DL, VA.getLocVT(), Val);
      break;
    case CCValAssign::AExtUpper:
      UseUpperBits = true;
      [[fallthrough]];
    case CCValAssign::AExt:
      Val = DAG.getNode(ISD::ANY_EXTEND, DL, VA.getLocVT(), Val);
      break;
    case CCValAssign::ZExtUpper:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4009-4020
```cpp
      UseUpperBits = true;
      [[fallthrough]];
    case CCValAssign::ZExt:
      Val = DAG.getNode(ISD::ZERO_EXTEND, DL, VA.getLocVT(), Val);
      break;
    case CCValAssign::SExtUpper:
      UseUpperBits = true;
      [[fallthrough]];
    case CCValAssign::SExt:
      Val = DAG.getNode(ISD::SIGN_EXTEND, DL, VA.getLocVT(), Val);
      break;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4022-4028
```cpp
    if (UseUpperBits) {
      unsigned ValSizeInBits = Outs[i].ArgVT.getSizeInBits();
      unsigned LocSizeInBits = VA.getLocVT().getSizeInBits();
      Val = DAG.getNode(
          ISD::SHL, DL, VA.getLocVT(), Val,
          DAG.getConstant(LocSizeInBits - ValSizeInBits, DL, VA.getLocVT()));
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4030-4030
```cpp
    Chain = DAG.getCopyToReg(Chain, DL, VA.getLocReg(), Val, Glue);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4032-4035
```cpp
    // Guarantee that all emitted copies are stuck together with flags.
    Glue = Chain.getValue(1);
    RetOps.push_back(DAG.getRegister(VA.getLocReg(), VA.getLocVT()));
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4037-4043
```cpp
  // The mips ABIs for returning structs by value requires that we copy
  // the sret argument into $v0 for the return. We saved the argument into
  // a virtual register in the entry block, so now we copy the value out
  // and into $v0.
  if (MF.getFunction().hasStructRetAttr()) {
    MipsFunctionInfo *MipsFI = MF.getInfo<MipsFunctionInfo>();
    unsigned Reg = MipsFI->getSRetReturnReg();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4045-4049
```cpp
    if (!Reg)
      llvm_unreachable("sret virtual register not created in the entry block");
    SDValue Val =
        DAG.getCopyFromReg(Chain, DL, Reg, getPointerTy(DAG.getDataLayout()));
    unsigned V0 = ABI.IsN64() ? Mips::V0_64 : Mips::V0;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4051-4054
```cpp
    Chain = DAG.getCopyToReg(Chain, DL, V0, Val, Glue);
    Glue = Chain.getValue(1);
    RetOps.push_back(DAG.getRegister(V0, getPointerTy(DAG.getDataLayout())));
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4056-4056
```cpp
  RetOps[0] = Chain;  // Update chain.
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4058-4060
```cpp
  // Add the glue if we have it.
  if (Glue.getNode())
    RetOps.push_back(Glue);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4062-4064
```cpp
  // ISRs must use "eret".
  if (DAG.getMachineFunction().getFunction().hasFnAttribute("interrupt"))
    return LowerInterruptReturn(RetOps, DL, DAG);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4066-4068
```cpp
  // Standard return on Mips is a "jr $ra"
  return DAG.getNode(MipsISD::Ret, DL, MVT::Other, RetOps);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4070-4072
```cpp
//===----------------------------------------------------------------------===//
//                           Mips Inline Assembly Support
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 4074-4088
```cpp
/// getConstraintType - Given a constraint letter, return the type of
/// constraint it is for this target.
MipsTargetLowering::ConstraintType
MipsTargetLowering::getConstraintType(StringRef Constraint) const {
  // Mips specific constraints
  // GCC config/mips/constraints.md
  //
  // 'd' : An address register. Equivalent to r
  //       unless generating MIPS16 code.
  // 'y' : Equivalent to r; retained for
  //       backwards compatibility.
  // 'c' : A register suitable for use in an indirect
  //       jump. This will always be $25 for -mabicalls.
  // 'l' : The lo register. 1 word storage.
  // 'x' : The hilo register pair. Double word storage.
```
- EN: Implements `MipsTargetLowering::getConstraintType`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::getConstraintType`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4089-4102
```cpp
  if (Constraint.size() == 1) {
    switch (Constraint[0]) {
      default : break;
      case 'd':
      case 'y':
      case 'f':
      case 'c':
      case 'l':
      case 'x':
        return C_RegisterClass;
      case 'R':
        return C_Memory;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4104-4105
```cpp
  if (Constraint == "ZC")
    return C_Memory;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4107-4108
```cpp
  return TargetLowering::getConstraintType(Constraint);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4110-4124
```cpp
/// Examine constraint type and operand type and determine a weight value.
/// This object must already have been set up with the operand type
/// and the current alternative constraint selected.
TargetLowering::ConstraintWeight
MipsTargetLowering::getSingleConstraintMatchWeight(
    AsmOperandInfo &info, const char *constraint) const {
  ConstraintWeight weight = CW_Invalid;
  Value *CallOperandVal = info.CallOperandVal;
    // If we don't have a value, we can't do a match,
    // but allow it at the lowest weight.
  if (!CallOperandVal)
    return CW_Default;
  Type *type = CallOperandVal->getType();
  // Look at the constraint type.
  switch (*constraint) {
```
- EN: Implements `MipsTargetLowering::getSingleConstraintMatchWeight`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::getSingleConstraintMatchWeight`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4125-4139
```cpp
  default:
    weight = TargetLowering::getSingleConstraintMatchWeight(info, constraint);
    break;
  case 'd':
  case 'y':
    if (type->isIntegerTy())
      weight = CW_Register;
    break;
  case 'f': // FPU or MSA register
    if (Subtarget.hasMSA() && type->isVectorTy() &&
        type->getPrimitiveSizeInBits().getFixedValue() == 128)
      weight = CW_Register;
    else if (type->isFloatTy())
      weight = CW_Register;
    break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4140-4154
```cpp
  case 'c': // $25 for indirect jumps
  case 'l': // lo register
  case 'x': // hilo register pair
    if (type->isIntegerTy())
      weight = CW_SpecificReg;
    break;
  case 'I': // signed 16 bit immediate
  case 'J': // integer zero
  case 'K': // unsigned 16 bit immediate
  case 'L': // signed 32 bit immediate where lower 16 bits are 0
  case 'N': // immediate in the range of -65535 to -1 (inclusive)
  case 'O': // signed 15 bit immediate (+- 16383)
  case 'P': // immediate in the range of 65535 to 1 (inclusive)
    if (isa<ConstantInt>(CallOperandVal))
      weight = CW_Constant;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4155-4161
```cpp
    break;
  case 'R':
    weight = CW_Memory;
    break;
  }
  return weight;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4163-4170
```cpp
/// This is a helper function to parse a physical register string and split it
/// into non-numeric and numeric parts (Prefix and Reg). The first boolean flag
/// that is returned indicates whether parsing was successful. The second flag
/// is true if the numeric part exists.
static std::pair<bool, bool> parsePhysicalReg(StringRef C, StringRef &Prefix,
                                              unsigned long long &Reg) {
  if (C.front() != '{' || C.back() != '}')
    return std::make_pair(false, false);
```
- EN: Implements `parsePhysicalReg`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `parsePhysicalReg`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4172-4174
```cpp
  // Search for the first numeric character.
  StringRef::const_iterator I, B = C.begin() + 1, E = C.end() - 1;
  I = std::find_if(B, E, isdigit);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4176-4176
```cpp
  Prefix = StringRef(B, I - B);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4178-4180
```cpp
  // The second flag is set to false if no numeric characters were found.
  if (I == E)
    return std::make_pair(true, false);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4182-4185
```cpp
  // Parse the numeric characters.
  return std::make_pair(!getAsUnsignedInteger(StringRef(I, E - I), 10, Reg),
                        true);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4187-4192
```cpp
EVT MipsTargetLowering::getTypeForExtReturn(LLVMContext &Context, EVT VT,
                                            ISD::NodeType) const {
  bool Cond = !Subtarget.isABI_O32() && VT.getSizeInBits() == 32;
  EVT MinVT = getRegisterType(Cond ? MVT::i64 : MVT::i32);
  return VT.bitsLT(MinVT) ? MinVT : VT;
}
```
- EN: Implements `MipsTargetLowering::getTypeForExtReturn`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::getTypeForExtReturn`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4194-4200
```cpp
std::pair<unsigned, const TargetRegisterClass *> MipsTargetLowering::
parseRegForInlineAsmConstraint(StringRef C, MVT VT) const {
  const TargetRegisterInfo *TRI =
      Subtarget.getRegisterInfo();
  const TargetRegisterClass *RC;
  StringRef Prefix;
  unsigned long long Reg;
```
- EN: Implements `parseRegForInlineAsmConstraint`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `parseRegForInlineAsmConstraint`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4202-4202
```cpp
  std::pair<bool, bool> R = parsePhysicalReg(C, Prefix, Reg);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4204-4205
```cpp
  if (!R.first)
    return std::make_pair(0U, nullptr);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4207-4210
```cpp
  if ((Prefix == "hi" || Prefix == "lo")) { // Parse hi/lo.
    // No numeric characters follow "hi" or "lo".
    if (R.second)
      return std::make_pair(0U, nullptr);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4212-4216
```cpp
    RC = TRI->getRegClass(Prefix == "hi" ?
                          Mips::HI32RegClassID : Mips::LO32RegClassID);
    return std::make_pair(*(RC->begin()), RC);
  } else if (Prefix.starts_with("$msa")) {
    // Parse $msa(ir|csr|access|save|modify|request|map|unmap)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4218-4220
```cpp
    // No numeric characters follow the name.
    if (R.second)
      return std::make_pair(0U, nullptr);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4222-4231
```cpp
    Reg = StringSwitch<unsigned long long>(Prefix)
              .Case("$msair", Mips::MSAIR)
              .Case("$msacsr", Mips::MSACSR)
              .Case("$msaaccess", Mips::MSAAccess)
              .Case("$msasave", Mips::MSASave)
              .Case("$msamodify", Mips::MSAModify)
              .Case("$msarequest", Mips::MSARequest)
              .Case("$msamap", Mips::MSAMap)
              .Case("$msaunmap", Mips::MSAUnmap)
              .Default(0);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4233-4234
```cpp
    if (!Reg)
      return std::make_pair(0U, nullptr);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4236-4238
```cpp
    RC = TRI->getRegClass(Mips::MSACtrlRegClassID);
    return std::make_pair(Reg, RC);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4240-4241
```cpp
  if (!R.second)
    return std::make_pair(0U, nullptr);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4243-4253
```cpp
  if (Prefix == "$f") { // Parse $f0-$f31.
    // If the targets is single float only, always select 32-bit registers,
    // otherwise if the size of FP registers is 64-bit or Reg is an even number,
    // select the 64-bit register class. Otherwise, select the 32-bit register
    // class.
    if (VT == MVT::Other) {
      if (Subtarget.isSingleFloat())
        VT = MVT::f32;
      else
        VT = (Subtarget.isFP64bit() || !(Reg % 2)) ? MVT::f64 : MVT::f32;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4255-4255
```cpp
    RC = getRegClassFor(VT);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4257-4268
```cpp
    if (RC == &Mips::AFGR64RegClass) {
      assert(Reg % 2 == 0);
      Reg >>= 1;
    }
  } else if (Prefix == "$fcc") // Parse $fcc0-$fcc7.
    RC = TRI->getRegClass(Mips::FCCRegClassID);
  else if (Prefix == "$w") { // Parse $w0-$w31.
    RC = getRegClassFor((VT == MVT::Other) ? MVT::v16i8 : VT);
  } else { // Parse $0-$31.
    assert(Prefix == "$");
    RC = getRegClassFor((VT == MVT::Other) ? MVT::i32 : VT);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4270-4272
```cpp
  assert(Reg < RC->getNumRegs());
  return std::make_pair(*(RC->begin() + Reg), RC);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4274-4288
```cpp
/// Given a register class constraint, like 'r', if this corresponds directly
/// to an LLVM register class, return a register of 0 and the register class
/// pointer.
std::pair<unsigned, const TargetRegisterClass *>
MipsTargetLowering::getRegForInlineAsmConstraint(const TargetRegisterInfo *TRI,
                                                 StringRef Constraint,
                                                 MVT VT) const {
  if (Constraint.size() == 1) {
    switch (Constraint[0]) {
    case 'd': // Address register. Same as 'r' unless generating MIPS16 code.
    case 'y': // Same as 'r'. Exists for compatibility.
    case 'r':
      if ((VT == MVT::i32 || VT == MVT::i16 || VT == MVT::i8 ||
           VT == MVT::i1) ||
          (VT == MVT::f32 && Subtarget.useSoftFloat())) {
```
- EN: Declares `constraint`, packaging target-specific state and APIs around `MipsISelLowering`.
- CN: 这里声明 `constraint`，把与 `MipsISelLowering` 相关的目标特定状态和 API 组织在一起。

### Lines 4289-4303
```cpp
        if (Subtarget.inMips16Mode())
          return std::make_pair(0U, &Mips::CPU16RegsRegClass);
        return std::make_pair(0U, &Mips::GPR32RegClass);
      }
      if ((VT == MVT::i64 || (VT == MVT::f64 && Subtarget.useSoftFloat()) ||
           (VT == MVT::f64 && Subtarget.isSingleFloat())) &&
          !Subtarget.isGP64bit())
        return std::make_pair(0U, &Mips::GPR32RegClass);
      if ((VT == MVT::i64 || (VT == MVT::f64 && Subtarget.useSoftFloat()) ||
           (VT == MVT::f64 && Subtarget.isSingleFloat())) &&
          Subtarget.isGP64bit())
        return std::make_pair(0U, &Mips::GPR64RegClass);
      // This will generate an error message
      return std::make_pair(0U, nullptr);
    case 'f': // FPU or MSA register
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4304-4318
```cpp
      if (VT == MVT::v16i8)
        return std::make_pair(0U, &Mips::MSA128BRegClass);
      else if (VT == MVT::v8i16 || VT == MVT::v8f16)
        return std::make_pair(0U, &Mips::MSA128HRegClass);
      else if (VT == MVT::v4i32 || VT == MVT::v4f32)
        return std::make_pair(0U, &Mips::MSA128WRegClass);
      else if (VT == MVT::v2i64 || VT == MVT::v2f64)
        return std::make_pair(0U, &Mips::MSA128DRegClass);
      else if (VT == MVT::f32)
        return std::make_pair(0U, &Mips::FGR32RegClass);
      else if ((VT == MVT::f64) && (!Subtarget.isSingleFloat())) {
        if (Subtarget.isFP64bit())
          return std::make_pair(0U, &Mips::FGR64RegClass);
        return std::make_pair(0U, &Mips::AFGR64RegClass);
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4319-4333
```cpp
      break;
    case 'c': // register suitable for indirect jump
      if (VT == MVT::i32)
        return std::make_pair((unsigned)Mips::T9, &Mips::GPR32RegClass);
      if (VT == MVT::i64)
        return std::make_pair((unsigned)Mips::T9_64, &Mips::GPR64RegClass);
      // This will generate an error message
      return std::make_pair(0U, nullptr);
    case 'l': // use the `lo` register to store values
              // that are no bigger than a word
      if (VT == MVT::i32 || VT == MVT::i16 || VT == MVT::i8)
        return std::make_pair((unsigned)Mips::LO0, &Mips::LO32RegClass);
      return std::make_pair((unsigned)Mips::LO0_64, &Mips::LO64RegClass);
    case 'x': // use the concatenated `hi` and `lo` registers
              // to store doubleword values
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4334-4338
```cpp
      // Fixme: Not triggering the use of both hi and low
      // This will generate an error message
      return std::make_pair(0U, nullptr);
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4340-4342
```cpp
  if (!Constraint.empty()) {
    std::pair<unsigned, const TargetRegisterClass *> R;
    R = parseRegForInlineAsmConstraint(Constraint, VT);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4344-4346
```cpp
    if (R.second)
      return R;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4348-4349
```cpp
  return TargetLowering::getRegForInlineAsmConstraint(TRI, Constraint, VT);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4351-4358
```cpp
/// LowerAsmOperandForConstraint - Lower the specified operand into the Ops
/// vector.  If it is invalid, don't add anything to Ops.
void MipsTargetLowering::LowerAsmOperandForConstraint(SDValue Op,
                                                      StringRef Constraint,
                                                      std::vector<SDValue> &Ops,
                                                      SelectionDAG &DAG) const {
  SDLoc DL(Op);
  SDValue Result;
```
- EN: Implements `MipsTargetLowering::LowerAsmOperandForConstraint`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::LowerAsmOperandForConstraint`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4360-4362
```cpp
  // Only support length 1 constraints for now.
  if (Constraint.size() > 1)
    return;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4364-4378
```cpp
  char ConstraintLetter = Constraint[0];
  switch (ConstraintLetter) {
  default: break; // This will fall through to the generic implementation
  case 'I': // Signed 16 bit constant
    // If this fails, the parent routine will give an error
    if (ConstantSDNode *C = dyn_cast<ConstantSDNode>(Op)) {
      EVT Type = Op.getValueType();
      int64_t Val = C->getSExtValue();
      if (isInt<16>(Val)) {
        Result = DAG.getSignedTargetConstant(Val, DL, Type);
        break;
      }
    }
    return;
  case 'J': // integer zero
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4379-4393
```cpp
    if (ConstantSDNode *C = dyn_cast<ConstantSDNode>(Op)) {
      EVT Type = Op.getValueType();
      int64_t Val = C->getZExtValue();
      if (Val == 0) {
        Result = DAG.getTargetConstant(0, DL, Type);
        break;
      }
    }
    return;
  case 'K': // unsigned 16 bit immediate
    if (ConstantSDNode *C = dyn_cast<ConstantSDNode>(Op)) {
      EVT Type = Op.getValueType();
      uint64_t Val = C->getZExtValue();
      if (isUInt<16>(Val)) {
        Result = DAG.getTargetConstant(Val, DL, Type);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4394-4408
```cpp
        break;
      }
    }
    return;
  case 'L': // signed 32 bit immediate where lower 16 bits are 0
    if (ConstantSDNode *C = dyn_cast<ConstantSDNode>(Op)) {
      EVT Type = Op.getValueType();
      int64_t Val = C->getSExtValue();
      if ((isInt<32>(Val)) && ((Val & 0xffff) == 0)){
        Result = DAG.getSignedTargetConstant(Val, DL, Type);
        break;
      }
    }
    return;
  case 'N': // immediate in the range of -65535 to -1 (inclusive)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4409-4423
```cpp
    if (ConstantSDNode *C = dyn_cast<ConstantSDNode>(Op)) {
      EVT Type = Op.getValueType();
      int64_t Val = C->getSExtValue();
      if ((Val >= -65535) && (Val <= -1)) {
        Result = DAG.getSignedTargetConstant(Val, DL, Type);
        break;
      }
    }
    return;
  case 'O': // signed 15 bit immediate
    if (ConstantSDNode *C = dyn_cast<ConstantSDNode>(Op)) {
      EVT Type = Op.getValueType();
      int64_t Val = C->getSExtValue();
      if ((isInt<15>(Val))) {
        Result = DAG.getSignedTargetConstant(Val, DL, Type);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4424-4438
```cpp
        break;
      }
    }
    return;
  case 'P': // immediate in the range of 1 to 65535 (inclusive)
    if (ConstantSDNode *C = dyn_cast<ConstantSDNode>(Op)) {
      EVT Type = Op.getValueType();
      int64_t Val = C->getSExtValue();
      if ((Val <= 65535) && (Val >= 1)) {
        Result = DAG.getTargetConstant(Val, DL, Type);
        break;
      }
    }
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4440-4443
```cpp
  if (Result.getNode()) {
    Ops.push_back(Result);
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4445-4446
```cpp
  TargetLowering::LowerAsmOperandForConstraint(Op, Constraint, Ops, DAG);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4448-4454
```cpp
bool MipsTargetLowering::isLegalAddressingMode(const DataLayout &DL,
                                               const AddrMode &AM, Type *Ty,
                                               unsigned AS,
                                               Instruction *I) const {
  // No global is ever allowed as a base.
  if (AM.BaseGV)
    return false;
```
- EN: Implements `MipsTargetLowering::isLegalAddressingMode`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::isLegalAddressingMode`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4456-4465
```cpp
  switch (AM.Scale) {
  case 0: // "r+i" or just "i", depending on HasBaseReg.
    break;
  case 1:
    if (!AM.HasBaseReg) // allow "r+i".
      break;
    return false; // disallow "r+r" or "r+r+i".
  default:
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4467-4468
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4470-4474
```cpp
bool
MipsTargetLowering::isOffsetFoldingLegal(const GlobalAddressSDNode *GA) const {
  // The Mips target isn't yet aware of offsets.
  return false;
}
```
- EN: Implements `MipsTargetLowering::isOffsetFoldingLegal`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::isOffsetFoldingLegal`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4476-4480
```cpp
EVT MipsTargetLowering::getOptimalMemOpType(
    LLVMContext &Context, const MemOp &Op,
    const AttributeList &FuncAttributes) const {
  if (Subtarget.hasMips64())
    return MVT::i64;
```
- EN: Implements `MipsTargetLowering::getOptimalMemOpType`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::getOptimalMemOpType`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4482-4483
```cpp
  return MVT::i32;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4485-4492
```cpp
bool MipsTargetLowering::isFPImmLegal(const APFloat &Imm, EVT VT,
                                      bool ForCodeSize) const {
  if (VT != MVT::f32 && VT != MVT::f64)
    return false;
  if (Imm.isNegZero())
    return false;
  return Imm.isZero();
}
```
- EN: Implements `MipsTargetLowering::isFPImmLegal`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::isFPImmLegal`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4494-4496
```cpp
bool MipsTargetLowering::isLegalICmpImmediate(int64_t Imm) const {
  return isInt<16>(Imm);
}
```
- EN: Implements `MipsTargetLowering::isLegalICmpImmediate`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::isLegalICmpImmediate`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4498-4500
```cpp
bool MipsTargetLowering::isLegalAddImmediate(int64_t Imm) const {
  return isInt<16>(Imm);
}
```
- EN: Implements `MipsTargetLowering::isLegalAddImmediate`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::isLegalAddImmediate`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4502-4508
```cpp
unsigned MipsTargetLowering::getJumpTableEncoding() const {
  if (!isPositionIndependent())
    return MachineJumpTableInfo::EK_BlockAddress;
  if (ABI.IsN64())
    return MachineJumpTableInfo::EK_GPRel64BlockAddress;
  return MachineJumpTableInfo::EK_GPRel32BlockAddress;
}
```
- EN: Implements `MipsTargetLowering::getJumpTableEncoding`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::getJumpTableEncoding`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4510-4515
```cpp
SDValue MipsTargetLowering::getPICJumpTableRelocBase(SDValue Table,
                                                     SelectionDAG &DAG) const {
  if (!isPositionIndependent())
    return Table;
  return DAG.getGLOBAL_OFFSET_TABLE(getPointerTy(DAG.getDataLayout()));
}
```
- EN: Implements `MipsTargetLowering::getPICJumpTableRelocBase`, a query/helper routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::getPICJumpTableRelocBase`，它是一个围绕SelectionDAG 降级展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4517-4519
```cpp
bool MipsTargetLowering::useSoftFloat() const {
  return Subtarget.useSoftFloat();
}
```
- EN: Implements `MipsTargetLowering::useSoftFloat`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::useSoftFloat`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4521-4534
```cpp
void MipsTargetLowering::copyByValRegs(
    SDValue Chain, const SDLoc &DL, std::vector<SDValue> &OutChains,
    SelectionDAG &DAG, const ISD::ArgFlagsTy &Flags,
    SmallVectorImpl<SDValue> &InVals, const Argument *FuncArg,
    unsigned FirstReg, unsigned LastReg, const CCValAssign &VA,
    MipsCCState &State) const {
  MachineFunction &MF = DAG.getMachineFunction();
  MachineFrameInfo &MFI = MF.getFrameInfo();
  unsigned GPRSizeInBytes = Subtarget.getGPRSizeInBytes();
  unsigned NumRegs = LastReg - FirstReg;
  unsigned RegAreaSize = NumRegs * GPRSizeInBytes;
  unsigned FrameObjSize = std::max(Flags.getByValSize(), RegAreaSize);
  int FrameObjOffset;
  ArrayRef<MCPhysReg> ByValArgRegs = ABI.GetByValArgRegs();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4536-4541
```cpp
  if (RegAreaSize)
    FrameObjOffset =
        (int)ABI.GetCalleeAllocdArgSizeInBytes(State.getCallingConv()) -
        (int)((ByValArgRegs.size() - FirstReg) * GPRSizeInBytes);
  else
    FrameObjOffset = VA.getLocMemOffset();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4543-4552
```cpp
  // Create frame object.
  EVT PtrTy = getPointerTy(DAG.getDataLayout());
  // Make the fixed object stored to mutable so that the load instructions
  // referencing it have their memory dependencies added.
  // Set the frame object as isAliased which clears the underlying objects
  // vector in ScheduleDAGInstrs::buildSchedGraph() resulting in addition of all
  // stores as dependencies for loads referencing this fixed object.
  int FI = MFI.CreateFixedObject(FrameObjSize, FrameObjOffset, false, true);
  SDValue FIN = DAG.getFrameIndex(FI, PtrTy);
  InVals.push_back(FIN);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4554-4555
```cpp
  if (!NumRegs)
    return;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4557-4559
```cpp
  // Copy arg registers.
  MVT RegTy = MVT::getIntegerVT(GPRSizeInBytes * 8);
  const TargetRegisterClass *RC = getRegClassFor(RegTy);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4561-4571
```cpp
  for (unsigned I = 0; I < NumRegs; ++I) {
    unsigned ArgReg = ByValArgRegs[FirstReg + I];
    unsigned VReg = addLiveIn(MF, ArgReg, RC);
    unsigned Offset = I * GPRSizeInBytes;
    SDValue StorePtr = DAG.getNode(ISD::ADD, DL, PtrTy, FIN,
                                   DAG.getConstant(Offset, DL, PtrTy));
    SDValue Store = DAG.getStore(Chain, DL, DAG.getRegister(VReg, RegTy),
                                 StorePtr, MachinePointerInfo(FuncArg, Offset));
    OutChains.push_back(Store);
  }
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4573-4587
```cpp
// Copy byVal arg to registers and stack.
void MipsTargetLowering::passByValArg(
    SDValue Chain, const SDLoc &DL,
    std::deque<std::pair<unsigned, SDValue>> &RegsToPass,
    SmallVectorImpl<SDValue> &MemOpChains, SDValue StackPtr,
    MachineFrameInfo &MFI, SelectionDAG &DAG, SDValue Arg, unsigned FirstReg,
    unsigned LastReg, const ISD::ArgFlagsTy &Flags, bool isLittle,
    const CCValAssign &VA) const {
  unsigned ByValSizeInBytes = Flags.getByValSize();
  unsigned OffsetInBytes = 0; // From beginning of struct
  unsigned RegSizeInBytes = Subtarget.getGPRSizeInBytes();
  Align Alignment =
      std::min(Flags.getNonZeroByValAlign(), Align(RegSizeInBytes));
  EVT PtrTy = getPointerTy(DAG.getDataLayout()),
      RegTy = MVT::getIntegerVT(RegSizeInBytes * 8);
```
- EN: Declares `unsigned`, packaging target-specific state and APIs around `MipsISelLowering`.
- CN: 这里声明 `unsigned`，把与 `MipsISelLowering` 相关的目标特定状态和 API 组织在一起。

### Lines 4588-4588
```cpp
  unsigned NumRegs = LastReg - FirstReg;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4590-4593
```cpp
  if (NumRegs) {
    ArrayRef<MCPhysReg> ArgRegs = ABI.GetByValArgRegs();
    bool LeftoverBytes = (NumRegs * RegSizeInBytes > ByValSizeInBytes);
    unsigned I = 0;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4595-4604
```cpp
    // Copy words to registers.
    for (; I < NumRegs - LeftoverBytes; ++I, OffsetInBytes += RegSizeInBytes) {
      SDValue LoadPtr = DAG.getNode(ISD::ADD, DL, PtrTy, Arg,
                                    DAG.getConstant(OffsetInBytes, DL, PtrTy));
      SDValue LoadVal = DAG.getLoad(RegTy, DL, Chain, LoadPtr,
                                    MachinePointerInfo(), Alignment);
      MemOpChains.push_back(LoadVal.getValue(1));
      unsigned ArgReg = ArgRegs[FirstReg + I];
      RegsToPass.push_back(std::make_pair(ArgReg, LoadVal));
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4606-4608
```cpp
    // Return if the struct has been fully copied.
    if (ByValSizeInBytes == OffsetInBytes)
      return;
```
- EN: Declares `has`, packaging target-specific state and APIs around `MipsISelLowering`.
- CN: 这里声明 `has`，把与 `MipsISelLowering` 相关的目标特定状态和 API 组织在一起。

### Lines 4610-4612
```cpp
    // Copy the remainder of the byval argument with sub-word loads and shifts.
    if (LeftoverBytes) {
      SDValue Val;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4614-4616
```cpp
      for (unsigned LoadSizeInBytes = RegSizeInBytes / 2, TotalBytesLoaded = 0;
           OffsetInBytes < ByValSizeInBytes; LoadSizeInBytes /= 2) {
        unsigned RemainingSizeInBytes = ByValSizeInBytes - OffsetInBytes;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4618-4619
```cpp
        if (RemainingSizeInBytes < LoadSizeInBytes)
          continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4621-4628
```cpp
        // Load subword.
        SDValue LoadPtr = DAG.getNode(ISD::ADD, DL, PtrTy, Arg,
                                      DAG.getConstant(OffsetInBytes, DL,
                                                      PtrTy));
        SDValue LoadVal = DAG.getExtLoad(
            ISD::ZEXTLOAD, DL, RegTy, Chain, LoadPtr, MachinePointerInfo(),
            MVT::getIntegerVT(LoadSizeInBytes * 8), Alignment);
        MemOpChains.push_back(LoadVal.getValue(1));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4630-4631
```cpp
        // Shift the loaded value.
        unsigned Shamt;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4633-4636
```cpp
        if (isLittle)
          Shamt = TotalBytesLoaded * 8;
        else
          Shamt = (RegSizeInBytes - (TotalBytesLoaded + LoadSizeInBytes)) * 8;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4638-4639
```cpp
        SDValue Shift = DAG.getNode(ISD::SHL, DL, RegTy, LoadVal,
                                    DAG.getConstant(Shamt, DL, MVT::i32));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4641-4644
```cpp
        if (Val.getNode())
          Val = DAG.getNode(ISD::OR, DL, RegTy, Val, Shift);
        else
          Val = Shift;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4646-4649
```cpp
        OffsetInBytes += LoadSizeInBytes;
        TotalBytesLoaded += LoadSizeInBytes;
        Alignment = std::min(Alignment, Align(LoadSizeInBytes));
      }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4651-4655
```cpp
      unsigned ArgReg = ArgRegs[FirstReg + I];
      RegsToPass.push_back(std::make_pair(ArgReg, Val));
      return;
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4657-4668
```cpp
  // Copy remainder of byval arg to it with memcpy.
  unsigned MemCpySize = ByValSizeInBytes - OffsetInBytes;
  SDValue Src = DAG.getNode(ISD::ADD, DL, PtrTy, Arg,
                            DAG.getConstant(OffsetInBytes, DL, PtrTy));
  SDValue Dst = DAG.getNode(ISD::ADD, DL, PtrTy, StackPtr,
                            DAG.getIntPtrConstant(VA.getLocMemOffset(), DL));
  Chain = DAG.getMemcpy(
      Chain, DL, Dst, Src, DAG.getConstant(MemCpySize, DL, PtrTy),
      Align(Alignment), /*isVolatile=*/false, /*AlwaysInline=*/false,
      /*CI=*/nullptr, std::nullopt, MachinePointerInfo(), MachinePointerInfo());
  MemOpChains.push_back(Chain);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4670-4681
```cpp
void MipsTargetLowering::writeVarArgRegs(std::vector<SDValue> &OutChains,
                                         SDValue Chain, const SDLoc &DL,
                                         SelectionDAG &DAG,
                                         CCState &State) const {
  ArrayRef<MCPhysReg> ArgRegs = ABI.getVarArgRegs(Subtarget.isGP64bit());
  unsigned Idx = State.getFirstUnallocated(ArgRegs);
  unsigned RegSizeInBytes = Subtarget.getGPRSizeInBytes();
  MVT RegTy = MVT::getIntegerVT(RegSizeInBytes * 8);
  const TargetRegisterClass *RC = getRegClassFor(RegTy);
  MachineFunction &MF = DAG.getMachineFunction();
  MachineFrameInfo &MFI = MF.getFrameInfo();
  MipsFunctionInfo *MipsFI = MF.getInfo<MipsFunctionInfo>();
```
- EN: Implements `MipsTargetLowering::writeVarArgRegs`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::writeVarArgRegs`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4683-4684
```cpp
  // Offset of the first variable argument from stack pointer.
  int VaArgOffset;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4686-4692
```cpp
  if (ArgRegs.size() == Idx)
    VaArgOffset = alignTo(State.getStackSize(), RegSizeInBytes);
  else {
    VaArgOffset =
        (int)ABI.GetCalleeAllocdArgSizeInBytes(State.getCallingConv()) -
        (int)(RegSizeInBytes * (ArgRegs.size() - Idx));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4694-4697
```cpp
  // Record the frame index of the first variable argument
  // which is a value necessary to VASTART.
  int FI = MFI.CreateFixedObject(RegSizeInBytes, VaArgOffset, true);
  MipsFI->setVarArgsFrameIndex(FI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4699-4713
```cpp
  // Copy the integer registers that have not been used for argument passing
  // to the argument register save area. For O32, the save area is allocated
  // in the caller's stack frame, while for N32/64, it is allocated in the
  // callee's stack frame.
  for (unsigned I = Idx; I < ArgRegs.size();
       ++I, VaArgOffset += RegSizeInBytes) {
    unsigned Reg = addLiveIn(MF, ArgRegs[I], RC);
    SDValue ArgValue = DAG.getCopyFromReg(Chain, DL, Reg, RegTy);
    FI = MFI.CreateFixedObject(RegSizeInBytes, VaArgOffset, true);
    SDValue PtrOff = DAG.getFrameIndex(FI, getPointerTy(DAG.getDataLayout()));
    SDValue Store =
        DAG.getStore(Chain, DL, ArgValue, PtrOff, MachinePointerInfo());
    cast<StoreSDNode>(Store.getNode())->getMemOperand()->setValue(
        (Value *)nullptr);
    OutChains.push_back(Store);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4714-4715
```cpp
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4717-4719
```cpp
void MipsTargetLowering::HandleByVal(CCState *State, unsigned &Size,
                                     Align Alignment) const {
  const TargetFrameLowering *TFL = Subtarget.getFrameLowering();
```
- EN: Implements `MipsTargetLowering::HandleByVal`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::HandleByVal`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4721-4721
```cpp
  assert(Size && "Byval argument's size shouldn't be 0.");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4723-4723
```cpp
  Alignment = std::min(Alignment, TFL->getStackAlign());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4725-4726
```cpp
  unsigned FirstReg = 0;
  unsigned NumRegs = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4728-4733
```cpp
  if (State->getCallingConv() != CallingConv::Fast) {
    unsigned RegSizeInBytes = Subtarget.getGPRSizeInBytes();
    ArrayRef<MCPhysReg> IntArgRegs = ABI.GetByValArgRegs();
    // FIXME: The O32 case actually describes no shadow registers.
    const MCPhysReg *ShadowRegs =
        ABI.IsO32() ? IntArgRegs.data() : Mips64DPRegs;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4735-4739
```cpp
    // We used to check the size as well but we can't do that anymore since
    // CCState::HandleByVal() rounds up the size after calling this function.
    assert(
        Alignment >= Align(RegSizeInBytes) &&
        "Byval argument's alignment should be a multiple of RegSizeInBytes.");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4741-4741
```cpp
    FirstReg = State->getFirstUnallocated(IntArgRegs);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4743-4750
```cpp
    // If Alignment > RegSizeInBytes, the first arg register must be even.
    // FIXME: This condition happens to do the right thing but it's not the
    //        right way to test it. We want to check that the stack frame offset
    //        of the register is aligned.
    if ((Alignment > RegSizeInBytes) && (FirstReg % 2)) {
      State->AllocateReg(IntArgRegs[FirstReg], ShadowRegs[FirstReg]);
      ++FirstReg;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4752-4757
```cpp
    // Mark the registers allocated.
    Size = alignTo(Size, RegSizeInBytes);
    for (unsigned I = FirstReg; Size > 0 && (I < IntArgRegs.size());
         Size -= RegSizeInBytes, ++I, ++NumRegs)
      State->AllocateReg(IntArgRegs[I], ShadowRegs[I]);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4759-4760
```cpp
  State->addInRegsParamInfo(FirstReg, FirstReg + NumRegs);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4762-4768
```cpp
MachineBasicBlock *MipsTargetLowering::emitPseudoSELECT(MachineInstr &MI,
                                                        MachineBasicBlock *BB,
                                                        bool isFPCmp,
                                                        unsigned Opc) const {
  assert(!(Subtarget.hasMips4() || Subtarget.hasMips32()) &&
         "Subtarget already supports SELECT nodes with the use of"
         "conditional-move instructions.");
```
- EN: Implements `MipsTargetLowering::emitPseudoSELECT`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::emitPseudoSELECT`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4770-4772
```cpp
  const TargetInstrInfo *TII =
      Subtarget.getInstrInfo();
  DebugLoc DL = MI.getDebugLoc();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4774-4779
```cpp
  // To "insert" a SELECT instruction, we actually have to insert the
  // diamond control-flow pattern.  The incoming instruction knows the
  // destination vreg to set, the condition code register to branch on, the
  // true/false values to select between, and a branch opcode to use.
  const BasicBlock *LLVM_BB = BB->getBasicBlock();
  MachineFunction::iterator It = ++BB->getIterator();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4781-4792
```cpp
  //  thisMBB:
  //  ...
  //   TrueVal = ...
  //   setcc r1, r2, r3
  //   bNE   r1, r0, copy1MBB
  //   fallthrough --> copy0MBB
  MachineBasicBlock *thisMBB  = BB;
  MachineFunction *F = BB->getParent();
  MachineBasicBlock *copy0MBB = F->CreateMachineBasicBlock(LLVM_BB);
  MachineBasicBlock *sinkMBB  = F->CreateMachineBasicBlock(LLVM_BB);
  F->insert(It, copy0MBB);
  F->insert(It, sinkMBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4794-4797
```cpp
  // Transfer the remainder of BB and its successor edges to sinkMBB.
  sinkMBB->splice(sinkMBB->begin(), BB,
                  std::next(MachineBasicBlock::iterator(MI)), BB->end());
  sinkMBB->transferSuccessorsAndUpdatePHIs(BB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4799-4801
```cpp
  // Next, add the true and fallthrough blocks as its successors.
  BB->addSuccessor(copy0MBB);
  BB->addSuccessor(sinkMBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4803-4814
```cpp
  if (isFPCmp) {
    // bc1[tf] cc, sinkMBB
    BuildMI(BB, DL, TII->get(Opc))
        .addReg(MI.getOperand(1).getReg())
        .addMBB(sinkMBB);
  } else {
    // bne rs, $0, sinkMBB
    BuildMI(BB, DL, TII->get(Opc))
        .addReg(MI.getOperand(1).getReg())
        .addReg(Mips::ZERO)
        .addMBB(sinkMBB);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4816-4819
```cpp
  //  copy0MBB:
  //   %FalseValue = ...
  //   # fallthrough to sinkMBB
  BB = copy0MBB;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4821-4822
```cpp
  // Update machine-CFG edges
  BB->addSuccessor(sinkMBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4824-4827
```cpp
  //  sinkMBB:
  //   %Result = phi [ %TrueValue, thisMBB ], [ %FalseValue, copy0MBB ]
  //  ...
  BB = sinkMBB;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4829-4833
```cpp
  BuildMI(*BB, BB->begin(), DL, TII->get(Mips::PHI), MI.getOperand(0).getReg())
      .addReg(MI.getOperand(2).getReg())
      .addMBB(thisMBB)
      .addReg(MI.getOperand(3).getReg())
      .addMBB(copy0MBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4835-4835
```cpp
  MI.eraseFromParent(); // The pseudo instruction is gone now.
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4837-4838
```cpp
  return BB;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4840-4845
```cpp
MachineBasicBlock *
MipsTargetLowering::emitPseudoD_SELECT(MachineInstr &MI,
                                       MachineBasicBlock *BB) const {
  assert(!(Subtarget.hasMips4() || Subtarget.hasMips32()) &&
         "Subtarget already supports SELECT nodes with the use of"
         "conditional-move instructions.");
```
- EN: Implements `MipsTargetLowering::emitPseudoD_SELECT`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::emitPseudoD_SELECT`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4847-4848
```cpp
  const TargetInstrInfo *TII = Subtarget.getInstrInfo();
  DebugLoc DL = MI.getDebugLoc();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4850-4856
```cpp
  // D_SELECT substitutes two SELECT nodes that goes one after another and
  // have the same condition operand. On machines which don't have
  // conditional-move instruction, it reduces unnecessary branch instructions
  // which are result of using two diamond patterns that are result of two
  // SELECT pseudo instructions.
  const BasicBlock *LLVM_BB = BB->getBasicBlock();
  MachineFunction::iterator It = ++BB->getIterator();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4858-4869
```cpp
  //  thisMBB:
  //  ...
  //   TrueVal = ...
  //   setcc r1, r2, r3
  //   bNE   r1, r0, copy1MBB
  //   fallthrough --> copy0MBB
  MachineBasicBlock *thisMBB = BB;
  MachineFunction *F = BB->getParent();
  MachineBasicBlock *copy0MBB = F->CreateMachineBasicBlock(LLVM_BB);
  MachineBasicBlock *sinkMBB = F->CreateMachineBasicBlock(LLVM_BB);
  F->insert(It, copy0MBB);
  F->insert(It, sinkMBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4871-4874
```cpp
  // Transfer the remainder of BB and its successor edges to sinkMBB.
  sinkMBB->splice(sinkMBB->begin(), BB,
                  std::next(MachineBasicBlock::iterator(MI)), BB->end());
  sinkMBB->transferSuccessorsAndUpdatePHIs(BB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4876-4878
```cpp
  // Next, add the true and fallthrough blocks as its successors.
  BB->addSuccessor(copy0MBB);
  BB->addSuccessor(sinkMBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4880-4884
```cpp
  // bne rs, $0, sinkMBB
  BuildMI(BB, DL, TII->get(Mips::BNE))
      .addReg(MI.getOperand(2).getReg())
      .addReg(Mips::ZERO)
      .addMBB(sinkMBB);
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4886-4889
```cpp
  //  copy0MBB:
  //   %FalseValue = ...
  //   # fallthrough to sinkMBB
  BB = copy0MBB;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4891-4892
```cpp
  // Update machine-CFG edges
  BB->addSuccessor(sinkMBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4894-4897
```cpp
  //  sinkMBB:
  //   %Result = phi [ %TrueValue, thisMBB ], [ %FalseValue, copy0MBB ]
  //  ...
  BB = sinkMBB;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4899-4909
```cpp
  // Use two PHI nodes to select two reults
  BuildMI(*BB, BB->begin(), DL, TII->get(Mips::PHI), MI.getOperand(0).getReg())
      .addReg(MI.getOperand(3).getReg())
      .addMBB(thisMBB)
      .addReg(MI.getOperand(5).getReg())
      .addMBB(copy0MBB);
  BuildMI(*BB, BB->begin(), DL, TII->get(Mips::PHI), MI.getOperand(1).getReg())
      .addReg(MI.getOperand(4).getReg())
      .addMBB(thisMBB)
      .addReg(MI.getOperand(6).getReg())
      .addMBB(copy0MBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4911-4911
```cpp
  MI.eraseFromParent(); // The pseudo instruction is gone now.
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4913-4914
```cpp
  return BB;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4916-4918
```cpp
// Copies the function MipsAsmParser::matchCPURegisterName.
int MipsTargetLowering::getCPURegisterIndex(StringRef Name) const {
  int CC;
```
- EN: Implements `MipsTargetLowering::getCPURegisterIndex`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::getCPURegisterIndex`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4920-4934
```cpp
  CC = StringSwitch<unsigned>(Name)
           .Case("zero", 0)
           .Case("at", 1)
           .Case("AT", 1)
           .Case("a0", 4)
           .Case("a1", 5)
           .Case("a2", 6)
           .Case("a3", 7)
           .Case("v0", 2)
           .Case("v1", 3)
           .Case("s0", 16)
           .Case("s1", 17)
           .Case("s2", 18)
           .Case("s3", 19)
           .Case("s4", 20)
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4935-4949
```cpp
           .Case("s5", 21)
           .Case("s6", 22)
           .Case("s7", 23)
           .Case("k0", 26)
           .Case("k1", 27)
           .Case("gp", 28)
           .Case("sp", 29)
           .Case("fp", 30)
           .Case("s8", 30)
           .Case("ra", 31)
           .Case("t0", 8)
           .Case("t1", 9)
           .Case("t2", 10)
           .Case("t3", 11)
           .Case("t4", 12)
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4950-4955
```cpp
           .Case("t5", 13)
           .Case("t6", 14)
           .Case("t7", 15)
           .Case("t8", 24)
           .Case("t9", 25)
           .Default(-1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4957-4958
```cpp
  if (!(ABI.IsN32() || ABI.IsN64()))
    return CC;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4960-4964
```cpp
  // Although SGI documentation just cuts out t0-t3 for n32/n64,
  // GNU pushes the values of t0-t3 to override the o32/o64 values for t4-t7
  // We are supporting both cases, so for t0-t3 we'll just push them to t4-t7.
  if (8 <= CC && CC <= 11)
    CC += 4;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4966-4974
```cpp
  if (CC == -1)
    CC = StringSwitch<unsigned>(Name)
             .Case("a4", 8)
             .Case("a5", 9)
             .Case("a6", 10)
             .Case("a7", 11)
             .Case("kt0", 26)
             .Case("kt1", 27)
             .Default(-1);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4976-4977
```cpp
  return CC;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4979-4987
```cpp
// FIXME? Maybe this could be a TableGen attribute on some registers and
// this table could be generated automatically from RegInfo.
Register
MipsTargetLowering::getRegisterByName(const char *RegName, LLT VT,
                                      const MachineFunction &MF) const {
  // 1. Delete symbol '$'.
  std::string newRegName = RegName;
  if (StringRef(RegName).starts_with("$"))
    newRegName = StringRef(RegName).substr(1);
```
- EN: Implements `MipsTargetLowering::getRegisterByName`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::getRegisterByName`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4989-4998
```cpp
  // 2. Get register index value.
  std::smatch matchResult;
  int regIdx;
  static const std::regex matchStr("^[0-9]*$");
  if (std::regex_match(newRegName, matchResult, matchStr))
    regIdx = std::stoi(newRegName);
  else {
    newRegName = StringRef(newRegName).lower();
    regIdx = getCPURegisterIndex(StringRef(newRegName));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5000-5007
```cpp
  // 3. Get register.
  if (regIdx >= 0 && regIdx < 32) {
    const MCRegisterInfo *MRI = MF.getContext().getRegisterInfo();
    const MCRegisterClass &RC = Subtarget.isGP64bit()
                                    ? MRI->getRegClass(Mips::GPR64RegClassID)
                                    : MRI->getRegClass(Mips::GPR32RegClassID);
    return RC.getRegister(regIdx);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5009-5011
```cpp
  report_fatal_error(
      Twine("Invalid register name \"" + StringRef(RegName) + "\"."));
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5013-5019
```cpp
MachineBasicBlock *MipsTargetLowering::emitLDR_W(MachineInstr &MI,
                                                 MachineBasicBlock *BB) const {
  MachineFunction *MF = BB->getParent();
  MachineRegisterInfo &MRI = MF->getRegInfo();
  const TargetInstrInfo *TII = Subtarget.getInstrInfo();
  const bool IsLittle = Subtarget.isLittle();
  DebugLoc DL = MI.getDebugLoc();
```
- EN: Implements `MipsTargetLowering::emitLDR_W`, a emission/printing routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::emitLDR_W`，它是一个围绕机器函数状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5021-5023
```cpp
  Register Dest = MI.getOperand(0).getReg();
  Register Address = MI.getOperand(1).getReg();
  unsigned Imm = MI.getOperand(2).getImm();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5025-5025
```cpp
  MachineBasicBlock::iterator I(MI);
```
- EN: Declares `I`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `I`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5027-5041
```cpp
  if (Subtarget.hasMips32r6() || Subtarget.hasMips64r6()) {
    // Mips release 6 can load from adress that is not naturally-aligned.
    Register Temp = MRI.createVirtualRegister(&Mips::GPR32RegClass);
    BuildMI(*BB, I, DL, TII->get(Mips::LW))
        .addDef(Temp)
        .addUse(Address)
        .addImm(Imm);
    BuildMI(*BB, I, DL, TII->get(Mips::FILL_W)).addDef(Dest).addUse(Temp);
  } else {
    // Mips release 5 needs to use instructions that can load from an unaligned
    // memory address.
    Register LoadHalf = MRI.createVirtualRegister(&Mips::GPR32RegClass);
    Register LoadFull = MRI.createVirtualRegister(&Mips::GPR32RegClass);
    Register Undef = MRI.createVirtualRegister(&Mips::GPR32RegClass);
    BuildMI(*BB, I, DL, TII->get(Mips::IMPLICIT_DEF)).addDef(Undef);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5042-5053
```cpp
    BuildMI(*BB, I, DL, TII->get(Mips::LWR))
        .addDef(LoadHalf)
        .addUse(Address)
        .addImm(Imm + (IsLittle ? 0 : 3))
        .addUse(Undef);
    BuildMI(*BB, I, DL, TII->get(Mips::LWL))
        .addDef(LoadFull)
        .addUse(Address)
        .addImm(Imm + (IsLittle ? 3 : 0))
        .addUse(LoadHalf);
    BuildMI(*BB, I, DL, TII->get(Mips::FILL_W)).addDef(Dest).addUse(LoadFull);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5055-5057
```cpp
  MI.eraseFromParent();
  return BB;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5059-5065
```cpp
MachineBasicBlock *MipsTargetLowering::emitLDR_D(MachineInstr &MI,
                                                 MachineBasicBlock *BB) const {
  MachineFunction *MF = BB->getParent();
  MachineRegisterInfo &MRI = MF->getRegInfo();
  const TargetInstrInfo *TII = Subtarget.getInstrInfo();
  const bool IsLittle = Subtarget.isLittle();
  DebugLoc DL = MI.getDebugLoc();
```
- EN: Implements `MipsTargetLowering::emitLDR_D`, a emission/printing routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::emitLDR_D`，它是一个围绕机器函数状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5067-5069
```cpp
  Register Dest = MI.getOperand(0).getReg();
  Register Address = MI.getOperand(1).getReg();
  unsigned Imm = MI.getOperand(2).getImm();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5071-5071
```cpp
  MachineBasicBlock::iterator I(MI);
```
- EN: Declares `I`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `I`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5073-5087
```cpp
  if (Subtarget.hasMips32r6() || Subtarget.hasMips64r6()) {
    // Mips release 6 can load from adress that is not naturally-aligned.
    if (Subtarget.isGP64bit()) {
      Register Temp = MRI.createVirtualRegister(&Mips::GPR64RegClass);
      BuildMI(*BB, I, DL, TII->get(Mips::LD))
          .addDef(Temp)
          .addUse(Address)
          .addImm(Imm);
      BuildMI(*BB, I, DL, TII->get(Mips::FILL_D)).addDef(Dest).addUse(Temp);
    } else {
      Register Wtemp = MRI.createVirtualRegister(&Mips::MSA128WRegClass);
      Register Lo = MRI.createVirtualRegister(&Mips::GPR32RegClass);
      Register Hi = MRI.createVirtualRegister(&Mips::GPR32RegClass);
      BuildMI(*BB, I, DL, TII->get(Mips::LW))
          .addDef(Lo)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5088-5102
```cpp
          .addUse(Address)
          .addImm(Imm + (IsLittle ? 0 : 4));
      BuildMI(*BB, I, DL, TII->get(Mips::LW))
          .addDef(Hi)
          .addUse(Address)
          .addImm(Imm + (IsLittle ? 4 : 0));
      BuildMI(*BB, I, DL, TII->get(Mips::FILL_W)).addDef(Wtemp).addUse(Lo);
      BuildMI(*BB, I, DL, TII->get(Mips::INSERT_W), Dest)
          .addUse(Wtemp)
          .addUse(Hi)
          .addImm(1);
    }
  } else {
    // Mips release 5 needs to use instructions that can load from an unaligned
    // memory address.
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5103-5117
```cpp
    Register LoHalf = MRI.createVirtualRegister(&Mips::GPR32RegClass);
    Register LoFull = MRI.createVirtualRegister(&Mips::GPR32RegClass);
    Register LoUndef = MRI.createVirtualRegister(&Mips::GPR32RegClass);
    Register HiHalf = MRI.createVirtualRegister(&Mips::GPR32RegClass);
    Register HiFull = MRI.createVirtualRegister(&Mips::GPR32RegClass);
    Register HiUndef = MRI.createVirtualRegister(&Mips::GPR32RegClass);
    Register Wtemp = MRI.createVirtualRegister(&Mips::MSA128WRegClass);
    BuildMI(*BB, I, DL, TII->get(Mips::IMPLICIT_DEF)).addDef(LoUndef);
    BuildMI(*BB, I, DL, TII->get(Mips::LWR))
        .addDef(LoHalf)
        .addUse(Address)
        .addImm(Imm + (IsLittle ? 0 : 7))
        .addUse(LoUndef);
    BuildMI(*BB, I, DL, TII->get(Mips::LWL))
        .addDef(LoFull)
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5118-5132
```cpp
        .addUse(Address)
        .addImm(Imm + (IsLittle ? 3 : 4))
        .addUse(LoHalf);
    BuildMI(*BB, I, DL, TII->get(Mips::IMPLICIT_DEF)).addDef(HiUndef);
    BuildMI(*BB, I, DL, TII->get(Mips::LWR))
        .addDef(HiHalf)
        .addUse(Address)
        .addImm(Imm + (IsLittle ? 4 : 3))
        .addUse(HiUndef);
    BuildMI(*BB, I, DL, TII->get(Mips::LWL))
        .addDef(HiFull)
        .addUse(Address)
        .addImm(Imm + (IsLittle ? 7 : 0))
        .addUse(HiHalf);
    BuildMI(*BB, I, DL, TII->get(Mips::FILL_W)).addDef(Wtemp).addUse(LoFull);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5133-5137
```cpp
    BuildMI(*BB, I, DL, TII->get(Mips::INSERT_W), Dest)
        .addUse(Wtemp)
        .addUse(HiFull)
        .addImm(1);
  }
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5139-5141
```cpp
  MI.eraseFromParent();
  return BB;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5143-5149
```cpp
MachineBasicBlock *MipsTargetLowering::emitSTR_W(MachineInstr &MI,
                                                 MachineBasicBlock *BB) const {
  MachineFunction *MF = BB->getParent();
  MachineRegisterInfo &MRI = MF->getRegInfo();
  const TargetInstrInfo *TII = Subtarget.getInstrInfo();
  const bool IsLittle = Subtarget.isLittle();
  DebugLoc DL = MI.getDebugLoc();
```
- EN: Implements `MipsTargetLowering::emitSTR_W`, a emission/printing routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::emitSTR_W`，它是一个围绕机器函数状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5151-5153
```cpp
  Register StoreVal = MI.getOperand(0).getReg();
  Register Address = MI.getOperand(1).getReg();
  unsigned Imm = MI.getOperand(2).getImm();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5155-5155
```cpp
  MachineBasicBlock::iterator I(MI);
```
- EN: Declares `I`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `I`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5157-5171
```cpp
  if (Subtarget.hasMips32r6() || Subtarget.hasMips64r6()) {
    // Mips release 6 can store to adress that is not naturally-aligned.
    Register BitcastW = MRI.createVirtualRegister(&Mips::MSA128WRegClass);
    Register Tmp = MRI.createVirtualRegister(&Mips::GPR32RegClass);
    BuildMI(*BB, I, DL, TII->get(Mips::COPY)).addDef(BitcastW).addUse(StoreVal);
    BuildMI(*BB, I, DL, TII->get(Mips::COPY_S_W))
        .addDef(Tmp)
        .addUse(BitcastW)
        .addImm(0);
    BuildMI(*BB, I, DL, TII->get(Mips::SW))
        .addUse(Tmp)
        .addUse(Address)
        .addImm(Imm);
  } else {
    // Mips release 5 needs to use instructions that can store to an unaligned
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5172-5186
```cpp
    // memory address.
    Register Tmp = MRI.createVirtualRegister(&Mips::GPR32RegClass);
    BuildMI(*BB, I, DL, TII->get(Mips::COPY_S_W))
        .addDef(Tmp)
        .addUse(StoreVal)
        .addImm(0);
    BuildMI(*BB, I, DL, TII->get(Mips::SWR))
        .addUse(Tmp)
        .addUse(Address)
        .addImm(Imm + (IsLittle ? 0 : 3));
    BuildMI(*BB, I, DL, TII->get(Mips::SWL))
        .addUse(Tmp)
        .addUse(Address)
        .addImm(Imm + (IsLittle ? 3 : 0));
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5188-5188
```cpp
  MI.eraseFromParent();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5190-5191
```cpp
  return BB;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5193-5199
```cpp
MachineBasicBlock *MipsTargetLowering::emitSTR_D(MachineInstr &MI,
                                                 MachineBasicBlock *BB) const {
  MachineFunction *MF = BB->getParent();
  MachineRegisterInfo &MRI = MF->getRegInfo();
  const TargetInstrInfo *TII = Subtarget.getInstrInfo();
  const bool IsLittle = Subtarget.isLittle();
  DebugLoc DL = MI.getDebugLoc();
```
- EN: Implements `MipsTargetLowering::emitSTR_D`, a emission/printing routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetLowering::emitSTR_D`，它是一个围绕机器函数状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5201-5203
```cpp
  Register StoreVal = MI.getOperand(0).getReg();
  Register Address = MI.getOperand(1).getReg();
  unsigned Imm = MI.getOperand(2).getImm();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5205-5205
```cpp
  MachineBasicBlock::iterator I(MI);
```
- EN: Declares `I`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `I`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5207-5221
```cpp
  if (Subtarget.hasMips32r6() || Subtarget.hasMips64r6()) {
    // Mips release 6 can store to adress that is not naturally-aligned.
    if (Subtarget.isGP64bit()) {
      Register BitcastD = MRI.createVirtualRegister(&Mips::MSA128DRegClass);
      Register Lo = MRI.createVirtualRegister(&Mips::GPR64RegClass);
      BuildMI(*BB, I, DL, TII->get(Mips::COPY))
          .addDef(BitcastD)
          .addUse(StoreVal);
      BuildMI(*BB, I, DL, TII->get(Mips::COPY_S_D))
          .addDef(Lo)
          .addUse(BitcastD)
          .addImm(0);
      BuildMI(*BB, I, DL, TII->get(Mips::SD))
          .addUse(Lo)
          .addUse(Address)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5222-5236
```cpp
          .addImm(Imm);
    } else {
      Register BitcastW = MRI.createVirtualRegister(&Mips::MSA128WRegClass);
      Register Lo = MRI.createVirtualRegister(&Mips::GPR32RegClass);
      Register Hi = MRI.createVirtualRegister(&Mips::GPR32RegClass);
      BuildMI(*BB, I, DL, TII->get(Mips::COPY))
          .addDef(BitcastW)
          .addUse(StoreVal);
      BuildMI(*BB, I, DL, TII->get(Mips::COPY_S_W))
          .addDef(Lo)
          .addUse(BitcastW)
          .addImm(0);
      BuildMI(*BB, I, DL, TII->get(Mips::COPY_S_W))
          .addDef(Hi)
          .addUse(BitcastW)
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5237-5251
```cpp
          .addImm(1);
      BuildMI(*BB, I, DL, TII->get(Mips::SW))
          .addUse(Lo)
          .addUse(Address)
          .addImm(Imm + (IsLittle ? 0 : 4));
      BuildMI(*BB, I, DL, TII->get(Mips::SW))
          .addUse(Hi)
          .addUse(Address)
          .addImm(Imm + (IsLittle ? 4 : 0));
    }
  } else {
    // Mips release 5 needs to use instructions that can store to an unaligned
    // memory address.
    Register Bitcast = MRI.createVirtualRegister(&Mips::MSA128WRegClass);
    Register Lo = MRI.createVirtualRegister(&Mips::GPR32RegClass);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5252-5266
```cpp
    Register Hi = MRI.createVirtualRegister(&Mips::GPR32RegClass);
    BuildMI(*BB, I, DL, TII->get(Mips::COPY)).addDef(Bitcast).addUse(StoreVal);
    BuildMI(*BB, I, DL, TII->get(Mips::COPY_S_W))
        .addDef(Lo)
        .addUse(Bitcast)
        .addImm(0);
    BuildMI(*BB, I, DL, TII->get(Mips::COPY_S_W))
        .addDef(Hi)
        .addUse(Bitcast)
        .addImm(1);
    BuildMI(*BB, I, DL, TII->get(Mips::SWR))
        .addUse(Lo)
        .addUse(Address)
        .addImm(Imm + (IsLittle ? 0 : 3));
    BuildMI(*BB, I, DL, TII->get(Mips::SWL))
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5267-5278
```cpp
        .addUse(Lo)
        .addUse(Address)
        .addImm(Imm + (IsLittle ? 3 : 0));
    BuildMI(*BB, I, DL, TII->get(Mips::SWR))
        .addUse(Hi)
        .addUse(Address)
        .addImm(Imm + (IsLittle ? 4 : 7));
    BuildMI(*BB, I, DL, TII->get(Mips::SWL))
        .addUse(Hi)
        .addUse(Address)
        .addImm(Imm + (IsLittle ? 7 : 4));
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5280-5282
```cpp
  MI.eraseFromParent();
  return BB;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

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

- EN: Backend-local headers: `MipsISelLowering.h`, `MCTargetDesc/MipsBaseInfo.h`, `MCTargetDesc/MipsInstPrinter.h`, `MCTargetDesc/MipsMCTargetDesc.h`, `MipsCCState.h`, `MipsInstrInfo.h`, `MipsMachineFunction.h`, `MipsRegisterInfo.h` ... (+4 more).
  - CN: 后端本地头文件：`MipsISelLowering.h`, `MCTargetDesc/MipsBaseInfo.h`, `MCTargetDesc/MipsInstPrinter.h`, `MCTargetDesc/MipsMCTargetDesc.h`, `MipsCCState.h`, `MipsInstrInfo.h`, `MipsMachineFunction.h`, `MipsRegisterInfo.h` ... (+4 more)。
- EN: LLVM infrastructure headers: `llvm/ADT/APFloat.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/StringSwitch.h`, `llvm/CodeGen/CallingConvLower.h`, `llvm/CodeGen/FunctionLoweringInfo.h` ... (+36 more).
  - CN: LLVM 基础设施头文件：`llvm/ADT/APFloat.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/StringSwitch.h`, `llvm/CodeGen/CallingConvLower.h`, `llvm/CodeGen/FunctionLoweringInfo.h` ... (+36 more)。
- EN: Standard/system headers: `algorithm`, `cassert`, `cctype`, `cstdint`, `deque`, `iterator`, `regex`, `string` ... (+2 more).
  - CN: 标准库/系统头文件：`algorithm`, `cassert`, `cctype`, `cstdint`, `deque`, `iterator`, `regex`, `string` ... (+2 more)。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
