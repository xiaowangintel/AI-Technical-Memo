# MipsSEISelLowering.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsSEISelLowering.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `MipsSEISelLowering` for the Mips backend, focusing on SelectionDAG lowering and target-lowering rules.
- 用途 (CN): 实现 Mips 后端中的 `MipsSEISelLowering`，重点处理SelectionDAG 降级与目标降级规则。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===- MipsSEISelLowering.cpp - MipsSE DAG Lowering Interface -------------===//
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

### Lines 13-27
```cpp
#include "MipsSEISelLowering.h"
#include "MipsMachineFunction.h"
#include "MipsRegisterInfo.h"
#include "MipsSubtarget.h"
#include "llvm/ADT/APInt.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/CodeGen/CallingConvLower.h"
#include "llvm/CodeGen/ISDOpcodes.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineMemOperand.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 28-42
```cpp
#include "llvm/CodeGen/SelectionDAG.h"
#include "llvm/CodeGen/SelectionDAGNodes.h"
#include "llvm/CodeGen/TargetInstrInfo.h"
#include "llvm/CodeGen/TargetLowering.h"
#include "llvm/CodeGen/TargetSubtargetInfo.h"
#include "llvm/CodeGen/ValueTypes.h"
#include "llvm/CodeGenTypes/MachineValueType.h"
#include "llvm/IR/DebugLoc.h"
#include "llvm/IR/Intrinsics.h"
#include "llvm/IR/IntrinsicsMips.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/raw_ostream.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 43-49
```cpp
#include "llvm/TargetParser/Triple.h"
#include <algorithm>
#include <cassert>
#include <cstddef>
#include <cstdint>
#include <iterator>
#include <utility>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 51-51
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 53-53
```cpp
#define DEBUG_TYPE "mips-isel"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 55-58
```cpp
static cl::opt<bool> NoDPLoadStore("mno-ldc1-sdc1", cl::init(false),
                                   cl::desc("Expand double precision loads and "
                                            "stores to their single precision "
                                            "counterparts"));
```
- EN: Declares `cl::init`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::init`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 60-74
```cpp
// Widen the v2 vectors to the register width, i.e. v2i16 -> v8i16,
// v2i32 -> v4i32, etc, to ensure the correct rail size is used, i.e.
// INST.h for v16, INST.w for v32, INST.d for v64.
TargetLoweringBase::LegalizeTypeAction
MipsSETargetLowering::getPreferredVectorAction(MVT VT) const {
  if (this->Subtarget.hasMSA()) {
    switch (VT.SimpleTy) {
    // Leave v2i1 vectors to be promoted to larger ones.
    // Other i1 types will be promoted by default.
    case MVT::v2i1:
      return TypePromoteInteger;
      break;
    // 16-bit vector types (v2 and longer)
    case MVT::v2i8:
    // 32-bit vector types (v2 and longer)
```
- EN: Implements `MipsSETargetLowering::getPreferredVectorAction`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSETargetLowering::getPreferredVectorAction`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 75-89
```cpp
    case MVT::v2i16:
    case MVT::v4i8:
    // 64-bit vector types (v2 and longer)
    case MVT::v2i32:
    case MVT::v4i16:
    case MVT::v8i8:
      return TypeWidenVector;
      break;
    // Only word (.w) and doubleword (.d) are available for floating point
    // vectors. That means floating point vectors should be either v2f64
    // or v4f32.
    // Here we only explicitly widen the f32 types - f16 will be promoted
    // by default.
    case MVT::v2f32:
    case MVT::v3f32:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 90-97
```cpp
      return TypeWidenVector;
    // v2i64 is already 128-bit wide.
    default:
      break;
    }
  }
  return TargetLoweringBase::getPreferredVectorAction(VT);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 99-103
```cpp
MipsSETargetLowering::MipsSETargetLowering(const MipsTargetMachine &TM,
                                           const MipsSubtarget &STI)
    : MipsTargetLowering(TM, STI) {
  // Set up the register classes
  addRegisterClass(MVT::i32, &Mips::GPR32RegClass);
```
- EN: Implements `MipsSETargetLowering::MipsSETargetLowering`, a lowering routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSETargetLowering::MipsSETargetLowering`，它是一个围绕寄存器管理展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 105-106
```cpp
  if (Subtarget.isGP64bit())
    addRegisterClass(MVT::i64, &Mips::GPR64RegClass);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 108-118
```cpp
  if (Subtarget.hasDSP() || Subtarget.hasMSA()) {
    // Expand all truncating stores and extending loads.
    for (MVT VT0 : MVT::fixedlen_vector_valuetypes()) {
      for (MVT VT1 : MVT::fixedlen_vector_valuetypes()) {
        setTruncStoreAction(VT0, VT1, Expand);
        setLoadExtAction(ISD::SEXTLOAD, VT0, VT1, Expand);
        setLoadExtAction(ISD::ZEXTLOAD, VT0, VT1, Expand);
        setLoadExtAction(ISD::EXTLOAD, VT0, VT1, Expand);
      }
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 120-121
```cpp
  if (Subtarget.hasDSP()) {
    MVT::SimpleValueType VecTys[2] = {MVT::v2i16, MVT::v4i8};
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 123-124
```cpp
    for (const auto &VecTy : VecTys) {
      addRegisterClass(VecTy, &Mips::DSPRRegClass);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 126-128
```cpp
      // Expand all builtin opcodes.
      for (unsigned Opc = 0; Opc < ISD::BUILTIN_OP_END; ++Opc)
        setOperationAction(Opc, VecTy, Expand);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 130-135
```cpp
      setOperationAction(ISD::ADD, VecTy, Legal);
      setOperationAction(ISD::SUB, VecTy, Legal);
      setOperationAction(ISD::LOAD, VecTy, Legal);
      setOperationAction(ISD::STORE, VecTy, Legal);
      setOperationAction(ISD::BITCAST, VecTy, Legal);
    }
```
- EN: Declares `setOperationAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 137-138
```cpp
    setTargetDAGCombine(
        {ISD::SHL, ISD::SRA, ISD::SRL, ISD::SETCC, ISD::VSELECT});
```
- EN: Implements `setTargetDAGCombine`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `setTargetDAGCombine`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 140-144
```cpp
    if (Subtarget.hasMips32r2()) {
      setOperationAction(ISD::ADDC, MVT::i32, Legal);
      setOperationAction(ISD::ADDE, MVT::i32, Legal);
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 146-147
```cpp
  if (Subtarget.hasDSPR2())
    setOperationAction(ISD::MUL, MVT::v2i16, Legal);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 149-156
```cpp
  if (Subtarget.hasMSA()) {
    addMSAIntType(MVT::v16i8, &Mips::MSA128BRegClass);
    addMSAIntType(MVT::v8i16, &Mips::MSA128HRegClass);
    addMSAIntType(MVT::v4i32, &Mips::MSA128WRegClass);
    addMSAIntType(MVT::v2i64, &Mips::MSA128DRegClass);
    addMSAFloatType(MVT::v8f16, &Mips::MSA128HRegClass);
    addMSAFloatType(MVT::v4f32, &Mips::MSA128WRegClass);
    addMSAFloatType(MVT::v2f64, &Mips::MSA128DRegClass);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 158-172
```cpp
    // f16 is a storage-only type, always promote it to f32.
    addRegisterClass(MVT::f16, &Mips::MSA128HRegClass);
    setOperationAction(ISD::SETCC, MVT::f16, Promote);
    setOperationAction(ISD::BR_CC, MVT::f16, Promote);
    setOperationAction(ISD::SELECT_CC, MVT::f16, Promote);
    setOperationAction(ISD::SELECT, MVT::f16, Promote);
    setOperationAction(ISD::FADD, MVT::f16, Promote);
    setOperationAction(ISD::FSUB, MVT::f16, Promote);
    setOperationAction(ISD::FMUL, MVT::f16, Promote);
    setOperationAction(ISD::FDIV, MVT::f16, Promote);
    setOperationAction(ISD::FREM, MVT::f16, Promote);
    setOperationAction(ISD::FMA, MVT::f16, Promote);
    setOperationAction(ISD::FNEG, MVT::f16, Promote);
    setOperationAction(ISD::FABS, MVT::f16, Promote);
    setOperationAction(ISD::FCEIL, MVT::f16, Promote);
```
- EN: Declares `addRegisterClass`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `addRegisterClass`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 173-187
```cpp
    setOperationAction(ISD::FCOPYSIGN, MVT::f16, Promote);
    setOperationAction(ISD::FCOS, MVT::f16, Promote);
    setOperationAction(ISD::FP_EXTEND, MVT::f16, Promote);
    setOperationAction(ISD::FFLOOR, MVT::f16, Promote);
    setOperationAction(ISD::FNEARBYINT, MVT::f16, Promote);
    setOperationAction(ISD::FPOW, MVT::f16, Promote);
    setOperationAction(ISD::FPOWI, MVT::f16, Promote);
    setOperationAction(ISD::FRINT, MVT::f16, Promote);
    setOperationAction(ISD::FSIN, MVT::f16, Promote);
    setOperationAction(ISD::FSINCOS, MVT::f16, Promote);
    setOperationAction(ISD::FSQRT, MVT::f16, Promote);
    setOperationAction(ISD::FEXP, MVT::f16, Promote);
    setOperationAction(ISD::FEXP2, MVT::f16, Promote);
    setOperationAction(ISD::FLOG, MVT::f16, Promote);
    setOperationAction(ISD::FLOG2, MVT::f16, Promote);
```
- EN: Declares `setOperationAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 188-194
```cpp
    setOperationAction(ISD::FLOG10, MVT::f16, Promote);
    setOperationAction(ISD::FROUND, MVT::f16, Promote);
    setOperationAction(ISD::FTRUNC, MVT::f16, Promote);
    setOperationAction(ISD::FMINNUM, MVT::f16, Promote);
    setOperationAction(ISD::FMAXNUM, MVT::f16, Promote);
    setOperationAction(ISD::FMINIMUM, MVT::f16, Promote);
    setOperationAction(ISD::FMAXIMUM, MVT::f16, Promote);
```
- EN: Declares `setOperationAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 196-197
```cpp
    setTargetDAGCombine({ISD::AND, ISD::OR, ISD::SRA, ISD::VSELECT, ISD::XOR});
  }
```
- EN: Implements `setTargetDAGCombine`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `setTargetDAGCombine`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 199-200
```cpp
  if (!Subtarget.useSoftFloat()) {
    addRegisterClass(MVT::f32, &Mips::FGR32RegClass);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 202-208
```cpp
    // When dealing with single precision only, use libcalls
    if (!Subtarget.isSingleFloat()) {
      if (Subtarget.isFP64bit())
        addRegisterClass(MVT::f64, &Mips::FGR64RegClass);
      else
        addRegisterClass(MVT::f64, &Mips::AFGR64RegClass);
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 210-215
```cpp
    for (auto Op : {ISD::STRICT_FADD, ISD::STRICT_FSUB, ISD::STRICT_FMUL,
                    ISD::STRICT_FDIV, ISD::STRICT_FSQRT}) {
      setOperationAction(Op, MVT::f32, Legal);
      setOperationAction(Op, MVT::f64, Legal);
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 217-225
```cpp
  // Targets with 64bits integer registers, but no 64bit floating point register
  // do not support conversion between them
  if (Subtarget.isGP64bit() && Subtarget.isSingleFloat() &&
      !Subtarget.useSoftFloat()) {
    setOperationAction(ISD::FP_TO_SINT, MVT::i64, Expand);
    setOperationAction(ISD::FP_TO_UINT, MVT::i64, Expand);
    setOperationAction(ISD::SINT_TO_FP, MVT::i64, Expand);
    setOperationAction(ISD::UINT_TO_FP, MVT::i64, Expand);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 227-230
```cpp
  setOperationAction(ISD::SMUL_LOHI,          MVT::i32, Custom);
  setOperationAction(ISD::UMUL_LOHI,          MVT::i32, Custom);
  setOperationAction(ISD::MULHS,              MVT::i32, Custom);
  setOperationAction(ISD::MULHU,              MVT::i32, Custom);
```
- EN: Declares `setOperationAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 232-244
```cpp
  if (Subtarget.hasCnMips())
    setOperationAction(ISD::MUL,              MVT::i64, Legal);
  else if (Subtarget.isR5900()) {
    // R5900 doesn't have DMULT/DMULTU/DDIV/DDIVU - expand to 32-bit ops
    setOperationAction(ISD::MUL, MVT::i64, Expand);
    setOperationAction(ISD::SMUL_LOHI, MVT::i64, Expand);
    setOperationAction(ISD::UMUL_LOHI, MVT::i64, Expand);
    setOperationAction(ISD::MULHS, MVT::i64, Expand);
    setOperationAction(ISD::MULHU, MVT::i64, Expand);
    setOperationAction(ISD::SDIVREM, MVT::i64, Expand);
    setOperationAction(ISD::UDIVREM, MVT::i64, Expand);
  } else if (Subtarget.isGP64bit())
    setOperationAction(ISD::MUL,              MVT::i64, Custom);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 246-253
```cpp
  if (Subtarget.isGP64bit() && !Subtarget.isR5900()) {
    setOperationAction(ISD::SMUL_LOHI,        MVT::i64, Custom);
    setOperationAction(ISD::UMUL_LOHI,        MVT::i64, Custom);
    setOperationAction(ISD::MULHS,            MVT::i64, Custom);
    setOperationAction(ISD::MULHU,            MVT::i64, Custom);
    setOperationAction(ISD::SDIVREM,          MVT::i64, Custom);
    setOperationAction(ISD::UDIVREM,          MVT::i64, Custom);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 255-256
```cpp
  setOperationAction(ISD::INTRINSIC_WO_CHAIN, MVT::i64, Custom);
  setOperationAction(ISD::INTRINSIC_W_CHAIN,  MVT::i64, Custom);
```
- EN: Declares `setOperationAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 258-267
```cpp
  setOperationAction(ISD::SDIVREM, MVT::i32, Custom);
  setOperationAction(ISD::UDIVREM, MVT::i32, Custom);
  setOperationAction(ISD::ATOMIC_FENCE,       MVT::Other, Custom);
  if (Subtarget.hasMips32r6()) {
    setOperationAction(ISD::LOAD,               MVT::i32, Legal);
    setOperationAction(ISD::STORE,              MVT::i32, Legal);
  } else {
    setOperationAction(ISD::LOAD,               MVT::i32, Custom);
    setOperationAction(ISD::STORE,              MVT::i32, Custom);
  }
```
- EN: Implements `setOperationAction`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `setOperationAction`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 269-269
```cpp
  setTargetDAGCombine(ISD::MUL);
```
- EN: Declares `setTargetDAGCombine`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setTargetDAGCombine`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 271-273
```cpp
  setOperationAction(ISD::INTRINSIC_WO_CHAIN, MVT::Other, Custom);
  setOperationAction(ISD::INTRINSIC_W_CHAIN, MVT::Other, Custom);
  setOperationAction(ISD::INTRINSIC_VOID, MVT::Other, Custom);
```
- EN: Declares `setOperationAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 275-278
```cpp
  if (Subtarget.hasMips32r2() && !Subtarget.useSoftFloat() &&
      !Subtarget.hasMips64()) {
    setOperationAction(ISD::BITCAST, MVT::i64, Custom);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 280-283
```cpp
  if (NoDPLoadStore) {
    setOperationAction(ISD::LOAD, MVT::f64, Custom);
    setOperationAction(ISD::STORE, MVT::f64, Custom);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 285-292
```cpp
  if (Subtarget.hasMips32r6()) {
    // MIPS32r6 replaces the accumulator-based multiplies with a three register
    // instruction
    setOperationAction(ISD::SMUL_LOHI, MVT::i32, Expand);
    setOperationAction(ISD::UMUL_LOHI, MVT::i32, Expand);
    setOperationAction(ISD::MUL, MVT::i32, Legal);
    setOperationAction(ISD::MULHS, MVT::i32, Legal);
    setOperationAction(ISD::MULHU, MVT::i32, Legal);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 294-301
```cpp
    // MIPS32r6 replaces the accumulator-based division/remainder with separate
    // three register division and remainder instructions.
    setOperationAction(ISD::SDIVREM, MVT::i32, Expand);
    setOperationAction(ISD::UDIVREM, MVT::i32, Expand);
    setOperationAction(ISD::SDIV, MVT::i32, Legal);
    setOperationAction(ISD::UDIV, MVT::i32, Legal);
    setOperationAction(ISD::SREM, MVT::i32, Legal);
    setOperationAction(ISD::UREM, MVT::i32, Legal);
```
- EN: Declares `setOperationAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 303-307
```cpp
    // MIPS32r6 replaces conditional moves with an equivalent that removes the
    // need for three GPR read ports.
    setOperationAction(ISD::SETCC, MVT::i32, Legal);
    setOperationAction(ISD::SELECT, MVT::i32, Legal);
    setOperationAction(ISD::SELECT_CC, MVT::i32, Expand);
```
- EN: Declares `setOperationAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 309-311
```cpp
    setOperationAction(ISD::SETCC, MVT::f32, Legal);
    setOperationAction(ISD::SELECT, MVT::f32, Legal);
    setOperationAction(ISD::SELECT_CC, MVT::f32, Expand);
```
- EN: Declares `setOperationAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 313-316
```cpp
    assert(Subtarget.isFP64bit() && "FR=1 is required for MIPS32r6");
    setOperationAction(ISD::SETCC, MVT::f64, Legal);
    setOperationAction(ISD::SELECT, MVT::f64, Legal);
    setOperationAction(ISD::SELECT_CC, MVT::f64, Expand);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 318-318
```cpp
    setOperationAction(ISD::BRCOND, MVT::Other, Legal);
```
- EN: Declares `setOperationAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 320-328
```cpp
    // Floating point > and >= are supported via < and <=
    setCondCodeAction(ISD::SETOGE, MVT::f32, Expand);
    setCondCodeAction(ISD::SETOGT, MVT::f32, Expand);
    setCondCodeAction(ISD::SETUGE, MVT::f32, Expand);
    setCondCodeAction(ISD::SETUGT, MVT::f32, Expand);
    setCondCodeAction(ISD::SETONE, MVT::f32, Expand);
    setCondCodeAction(ISD::SETO, MVT::f32, Expand);
    setCondCodeAction(ISD::SETUNE, MVT::f32, Expand);
    setCondCodeAction(ISD::SETNE, MVT::f32, Expand);
```
- EN: Declares `setCondCodeAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setCondCodeAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 330-338
```cpp
    setCondCodeAction(ISD::SETOGE, MVT::f64, Expand);
    setCondCodeAction(ISD::SETOGT, MVT::f64, Expand);
    setCondCodeAction(ISD::SETUGE, MVT::f64, Expand);
    setCondCodeAction(ISD::SETUGT, MVT::f64, Expand);
    setCondCodeAction(ISD::SETONE, MVT::f64, Expand);
    setCondCodeAction(ISD::SETO, MVT::f64, Expand);
    setCondCodeAction(ISD::SETUNE, MVT::f64, Expand);
    setCondCodeAction(ISD::SETNE, MVT::f64, Expand);
  }
```
- EN: Declares `setCondCodeAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setCondCodeAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 340-347
```cpp
  if (Subtarget.hasMips64r6()) {
    // MIPS64r6 replaces the accumulator-based multiplies with a three register
    // instruction
    setOperationAction(ISD::SMUL_LOHI, MVT::i64, Expand);
    setOperationAction(ISD::UMUL_LOHI, MVT::i64, Expand);
    setOperationAction(ISD::MUL, MVT::i64, Legal);
    setOperationAction(ISD::MULHS, MVT::i64, Legal);
    setOperationAction(ISD::MULHU, MVT::i64, Legal);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 349-356
```cpp
    // MIPS32r6 replaces the accumulator-based division/remainder with separate
    // three register division and remainder instructions.
    setOperationAction(ISD::SDIVREM, MVT::i64, Expand);
    setOperationAction(ISD::UDIVREM, MVT::i64, Expand);
    setOperationAction(ISD::SDIV, MVT::i64, Legal);
    setOperationAction(ISD::UDIV, MVT::i64, Legal);
    setOperationAction(ISD::SREM, MVT::i64, Legal);
    setOperationAction(ISD::UREM, MVT::i64, Legal);
```
- EN: Declares `setOperationAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 358-363
```cpp
    // MIPS64r6 replaces conditional moves with an equivalent that removes the
    // need for three GPR read ports.
    setOperationAction(ISD::SETCC, MVT::i64, Legal);
    setOperationAction(ISD::SELECT, MVT::i64, Legal);
    setOperationAction(ISD::SELECT_CC, MVT::i64, Expand);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 365-379
```cpp
  if (Subtarget.isR5900()) {
    // R5900 FPU only supports 4 compare conditions: C.F, C.EQ, C.OLT, C.OLE
    // (and their inversions via bc1t/bc1f). Expand all conditions that would
    // require C.UN, C.UEQ, C.ULT, or C.ULE instructions (not available on
    // R5900). The legalizer resolves these via operand swapping, condition
    // inversion, and decomposition into supported conditions.
    setCondCodeAction(ISD::SETOGT, MVT::f32, Expand);
    setCondCodeAction(ISD::SETOGE, MVT::f32, Expand);
    setCondCodeAction(ISD::SETGT, MVT::f32, Expand);
    setCondCodeAction(ISD::SETGE, MVT::f32, Expand);
    setCondCodeAction(ISD::SETULT, MVT::f32, Expand);
    setCondCodeAction(ISD::SETULE, MVT::f32, Expand);
    setCondCodeAction(ISD::SETUO, MVT::f32, Expand);
    setCondCodeAction(ISD::SETO, MVT::f32, Expand);
    setCondCodeAction(ISD::SETONE, MVT::f32, Expand);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 380-381
```cpp
    setCondCodeAction(ISD::SETUEQ, MVT::f32, Expand);
    setCondCodeAction(ISD::SETNE, MVT::f32, Expand);
```
- EN: Declares `setCondCodeAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setCondCodeAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 383-391
```cpp
    // R5900 FPU does not support IEEE 754 special values (NaN, infinity). Use
    // custom lowering to decide per-instruction: hardware when nnan+ninf flags
    // guarantee no NaN or infinity, software libcall otherwise.
    setOperationAction(ISD::FADD, MVT::f32, Custom);
    setOperationAction(ISD::FSUB, MVT::f32, Custom);
    setOperationAction(ISD::FMUL, MVT::f32, Custom);
    setOperationAction(ISD::FDIV, MVT::f32, Custom);
    setOperationAction(ISD::FSQRT, MVT::f32, Custom);
  }
```
- EN: Declares `setOperationAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 393-394
```cpp
  computeRegisterProperties(Subtarget.getRegisterInfo());
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 396-400
```cpp
const MipsTargetLowering *
llvm::createMipsSETargetLowering(const MipsTargetMachine &TM,
                                 const MipsSubtarget &STI) {
  return new MipsSETargetLowering(TM, STI);
}
```
- EN: Implements `llvm::createMipsSETargetLowering`, a lowering routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::createMipsSETargetLowering`，它是一个围绕目标机器策略展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 402-405
```cpp
const TargetRegisterClass *
MipsSETargetLowering::getRepRegClassFor(MVT VT) const {
  if (VT == MVT::Untyped)
    return Subtarget.hasDSP() ? &Mips::ACC64DSPRegClass : &Mips::ACC64RegClass;
```
- EN: Implements `MipsSETargetLowering::getRepRegClassFor`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSETargetLowering::getRepRegClassFor`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 407-408
```cpp
  return TargetLowering::getRepRegClassFor(VT);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 410-413
```cpp
// Enable MSA support for the given integer type and Register class.
void MipsSETargetLowering::
addMSAIntType(MVT::SimpleValueType Ty, const TargetRegisterClass *RC) {
  addRegisterClass(Ty, RC);
```
- EN: Implements `addMSAIntType`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addMSAIntType`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 415-417
```cpp
  // Expand all builtin opcodes.
  for (unsigned Opc = 0; Opc < ISD::BUILTIN_OP_END; ++Opc)
    setOperationAction(Opc, Ty, Expand);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 419-425
```cpp
  setOperationAction(ISD::BITCAST, Ty, Legal);
  setOperationAction(ISD::LOAD, Ty, Legal);
  setOperationAction(ISD::STORE, Ty, Legal);
  setOperationAction(ISD::EXTRACT_VECTOR_ELT, Ty, Custom);
  setOperationAction(ISD::INSERT_VECTOR_ELT, Ty, Legal);
  setOperationAction(ISD::BUILD_VECTOR, Ty, Custom);
  setOperationAction(ISD::UNDEF, Ty, Legal);
```
- EN: Declares `setOperationAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 427-441
```cpp
  setOperationAction(ISD::ADD, Ty, Legal);
  setOperationAction(ISD::AND, Ty, Legal);
  setOperationAction(ISD::CTLZ, Ty, Legal);
  setOperationAction(ISD::CTPOP, Ty, Legal);
  setOperationAction(ISD::MUL, Ty, Legal);
  setOperationAction(ISD::OR, Ty, Legal);
  setOperationAction(ISD::SDIV, Ty, Legal);
  setOperationAction(ISD::SREM, Ty, Legal);
  setOperationAction(ISD::SHL, Ty, Legal);
  setOperationAction(ISD::SRA, Ty, Legal);
  setOperationAction(ISD::SRL, Ty, Legal);
  setOperationAction(ISD::SUB, Ty, Legal);
  setOperationAction(ISD::SMAX, Ty, Legal);
  setOperationAction(ISD::SMIN, Ty, Legal);
  setOperationAction(ISD::UDIV, Ty, Legal);
```
- EN: Declares `setOperationAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 442-447
```cpp
  setOperationAction(ISD::UREM, Ty, Legal);
  setOperationAction(ISD::UMAX, Ty, Legal);
  setOperationAction(ISD::UMIN, Ty, Legal);
  setOperationAction(ISD::VECTOR_SHUFFLE, Ty, Custom);
  setOperationAction(ISD::VSELECT, Ty, Legal);
  setOperationAction(ISD::XOR, Ty, Legal);
```
- EN: Declares `setOperationAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 449-454
```cpp
  if (Ty == MVT::v4i32 || Ty == MVT::v2i64) {
    setOperationAction(ISD::FP_TO_SINT, Ty, Legal);
    setOperationAction(ISD::FP_TO_UINT, Ty, Legal);
    setOperationAction(ISD::SINT_TO_FP, Ty, Legal);
    setOperationAction(ISD::UINT_TO_FP, Ty, Legal);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 456-462
```cpp
  setOperationAction(ISD::SETCC, Ty, Legal);
  setCondCodeAction(ISD::SETNE, Ty, Expand);
  setCondCodeAction(ISD::SETGE, Ty, Expand);
  setCondCodeAction(ISD::SETGT, Ty, Expand);
  setCondCodeAction(ISD::SETUGE, Ty, Expand);
  setCondCodeAction(ISD::SETUGT, Ty, Expand);
}
```
- EN: Declares `setOperationAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 464-467
```cpp
// Enable MSA support for the given floating-point type and Register class.
void MipsSETargetLowering::
addMSAFloatType(MVT::SimpleValueType Ty, const TargetRegisterClass *RC) {
  addRegisterClass(Ty, RC);
```
- EN: Implements `addMSAFloatType`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addMSAFloatType`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 469-471
```cpp
  // Expand all builtin opcodes.
  for (unsigned Opc = 0; Opc < ISD::BUILTIN_OP_END; ++Opc)
    setOperationAction(Opc, Ty, Expand);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 473-478
```cpp
  setOperationAction(ISD::LOAD, Ty, Legal);
  setOperationAction(ISD::STORE, Ty, Legal);
  setOperationAction(ISD::BITCAST, Ty, Legal);
  setOperationAction(ISD::EXTRACT_VECTOR_ELT, Ty, Legal);
  setOperationAction(ISD::INSERT_VECTOR_ELT, Ty, Legal);
  setOperationAction(ISD::BUILD_VECTOR, Ty, Custom);
```
- EN: Declares `setOperationAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 480-491
```cpp
  if (Ty != MVT::v8f16) {
    setOperationAction(ISD::FABS,  Ty, Legal);
    setOperationAction(ISD::FADD,  Ty, Legal);
    setOperationAction(ISD::FDIV,  Ty, Legal);
    setOperationAction(ISD::FEXP2, Ty, Legal);
    setOperationAction(ISD::FLOG2, Ty, Legal);
    setOperationAction(ISD::FMA,   Ty, Legal);
    setOperationAction(ISD::FMUL,  Ty, Legal);
    setOperationAction(ISD::FRINT, Ty, Legal);
    setOperationAction(ISD::FSQRT, Ty, Legal);
    setOperationAction(ISD::FSUB,  Ty, Legal);
    setOperationAction(ISD::VSELECT, Ty, Legal);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 493-501
```cpp
    setOperationAction(ISD::SETCC, Ty, Legal);
    setCondCodeAction(ISD::SETOGE, Ty, Expand);
    setCondCodeAction(ISD::SETOGT, Ty, Expand);
    setCondCodeAction(ISD::SETUGE, Ty, Expand);
    setCondCodeAction(ISD::SETUGT, Ty, Expand);
    setCondCodeAction(ISD::SETGE,  Ty, Expand);
    setCondCodeAction(ISD::SETGT,  Ty, Expand);
  }
}
```
- EN: Declares `setOperationAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 503-505
```cpp
SDValue MipsSETargetLowering::lowerSELECT(SDValue Op, SelectionDAG &DAG) const {
  if(!Subtarget.hasMips32r6())
    return MipsTargetLowering::LowerOperation(Op, DAG);
```
- EN: Implements `MipsSETargetLowering::lowerSELECT`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSETargetLowering::lowerSELECT`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 507-508
```cpp
  EVT ResTy = Op->getValueType(0);
  SDLoc DL(Op);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 510-516
```cpp
  // Although MTC1_D64 takes an i32 and writes an f64, the upper 32 bits of the
  // floating point register are undefined. Not really an issue as sel.d, which
  // is produced from an FSELECT node, only looks at bit 0.
  SDValue Tmp = DAG.getNode(MipsISD::MTC1_D64, DL, MVT::f64, Op->getOperand(0));
  return DAG.getNode(MipsISD::FSELECT, DL, ResTy, Tmp, Op->getOperand(1),
                     Op->getOperand(2));
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 518-520
```cpp
bool MipsSETargetLowering::allowsMisalignedMemoryAccesses(
    EVT VT, unsigned, Align, MachineMemOperand::Flags, unsigned *Fast) const {
  MVT::SimpleValueType SVT = VT.getSimpleVT().SimpleTy;
```
- EN: Implements `MipsSETargetLowering::allowsMisalignedMemoryAccesses`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSETargetLowering::allowsMisalignedMemoryAccesses`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 522-532
```cpp
  if (Subtarget.systemSupportsUnalignedAccess()) {
    // MIPS32r6/MIPS64r6 is required to support unaligned access. It's
    // implementation defined whether this is handled by hardware, software, or
    // a hybrid of the two but it's expected that most implementations will
    // handle the majority of cases in hardware.
    if (Fast)
      *Fast = 1;
    return true;
  } else if (Subtarget.hasMips32r6()) {
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 534-543
```cpp
  switch (SVT) {
  case MVT::i64:
  case MVT::i32:
    if (Fast)
      *Fast = 1;
    return true;
  default:
    return false;
  }
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 545-559
```cpp
SDValue MipsSETargetLowering::LowerOperation(SDValue Op,
                                             SelectionDAG &DAG) const {
  switch(Op.getOpcode()) {
  case ISD::LOAD:  return lowerLOAD(Op, DAG);
  case ISD::STORE: return lowerSTORE(Op, DAG);
  case ISD::SMUL_LOHI: return lowerMulDiv(Op, MipsISD::Mult, true, true, DAG);
  case ISD::UMUL_LOHI: return lowerMulDiv(Op, MipsISD::Multu, true, true, DAG);
  case ISD::MULHS:     return lowerMulDiv(Op, MipsISD::Mult, false, true, DAG);
  case ISD::MULHU:     return lowerMulDiv(Op, MipsISD::Multu, false, true, DAG);
  case ISD::MUL:       return lowerMulDiv(Op, MipsISD::Mult, true, false, DAG);
  case ISD::SDIVREM:   return lowerMulDiv(Op, MipsISD::DivRem, true, true, DAG);
  case ISD::UDIVREM:   return lowerMulDiv(Op, MipsISD::DivRemU, true, true,
                                          DAG);
  case ISD::INTRINSIC_WO_CHAIN: return lowerINTRINSIC_WO_CHAIN(Op, DAG);
  case ISD::INTRINSIC_W_CHAIN:  return lowerINTRINSIC_W_CHAIN(Op, DAG);
```
- EN: Implements `MipsSETargetLowering::LowerOperation`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSETargetLowering::LowerOperation`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 560-574
```cpp
  case ISD::INTRINSIC_VOID:     return lowerINTRINSIC_VOID(Op, DAG);
  case ISD::EXTRACT_VECTOR_ELT: return lowerEXTRACT_VECTOR_ELT(Op, DAG);
  case ISD::BUILD_VECTOR:       return lowerBUILD_VECTOR(Op, DAG);
  case ISD::VECTOR_SHUFFLE:     return lowerVECTOR_SHUFFLE(Op, DAG);
  case ISD::SELECT:             return lowerSELECT(Op, DAG);
  case ISD::BITCAST:            return lowerBITCAST(Op, DAG);
  case ISD::FADD:
    return lowerR5900FPOp(Op, DAG, RTLIB::ADD_F32);
  case ISD::FSUB:
    return lowerR5900FPOp(Op, DAG, RTLIB::SUB_F32);
  case ISD::FMUL:
    return lowerR5900FPOp(Op, DAG, RTLIB::MUL_F32);
  case ISD::FDIV:
    return lowerR5900FPOp(Op, DAG, RTLIB::DIV_F32);
  case ISD::FSQRT:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 575-576
```cpp
    return lowerR5900FPOp(Op, DAG, RTLIB::SQRT_F32);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 578-579
```cpp
  return MipsTargetLowering::LowerOperation(Op, DAG);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 581-584
```cpp
SDValue MipsSETargetLowering::lowerR5900FPOp(SDValue Op, SelectionDAG &DAG,
                                             RTLIB::Libcall LC) const {
  assert(Subtarget.isR5900());
  SDNodeFlags Flags = Op->getFlags();
```
- EN: Implements `MipsSETargetLowering::lowerR5900FPOp`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSETargetLowering::lowerR5900FPOp`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 586-590
```cpp
  if (Flags.hasNoNaNs() && Flags.hasNoInfs()) {
    // Use the hardware FPU instruction if the operation is guaranteed to have
    // no NaN or infinity inputs/outputs (nnan+ninf flags).
    return Op;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 592-599
```cpp
  // Fall back to a software libcall for IEEE correctness.
  SDLoc DL(Op);
  MVT VT = Op.getSimpleValueType();
  SmallVector<SDValue, 2> Ops(Op->op_begin(), Op->op_end());
  TargetLowering::MakeLibCallOptions CallOptions;
  auto [Result, Chain] = makeLibCall(DAG, LC, VT, Ops, CallOptions, DL);
  return Result;
}
```
- EN: Declares `DL`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `DL`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 601-612
```cpp
// Fold zero extensions into MipsISD::VEXTRACT_[SZ]EXT_ELT
//
// Performs the following transformations:
// - Changes MipsISD::VEXTRACT_[SZ]EXT_ELT to zero extension if its
//   sign/zero-extension is completely overwritten by the new one performed by
//   the ISD::AND.
// - Removes redundant zero extensions performed by an ISD::AND.
static SDValue performANDCombine(SDNode *N, SelectionDAG &DAG,
                                 TargetLowering::DAGCombinerInfo &DCI,
                                 const MipsSubtarget &Subtarget) {
  if (!Subtarget.hasMSA())
    return SDValue();
```
- EN: Implements `performANDCombine`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `performANDCombine`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 614-616
```cpp
  SDValue Op0 = N->getOperand(0);
  SDValue Op1 = N->getOperand(1);
  unsigned Op0Opcode = Op0->getOpcode();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 618-624
```cpp
  // (and (MipsVExtract[SZ]Ext $a, $b, $c), imm:$d)
  // where $d + 1 == 2^n and n == 32
  // or    $d + 1 == 2^n and n <= 32 and ZExt
  // -> (MipsVExtractZExt $a, $b, $c)
  if (Op0Opcode == MipsISD::VEXTRACT_SEXT_ELT ||
      Op0Opcode == MipsISD::VEXTRACT_ZEXT_ELT) {
    ConstantSDNode *Mask = dyn_cast<ConstantSDNode>(Op1);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 626-627
```cpp
    if (!Mask)
      return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 629-629
```cpp
    int32_t Log2IfPositive = (Mask->getAPIntValue() + 1).exactLogBase2();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 631-632
```cpp
    if (Log2IfPositive <= 0)
      return SDValue(); // Mask+1 is not a power of 2
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 634-637
```cpp
    SDValue Op0Op2 = Op0->getOperand(2);
    EVT ExtendTy = cast<VTSDNode>(Op0Op2)->getVT();
    unsigned ExtendTySize = ExtendTy.getSizeInBits();
    unsigned Log2 = Log2IfPositive;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 639-646
```cpp
    if ((Op0Opcode == MipsISD::VEXTRACT_ZEXT_ELT && Log2 >= ExtendTySize) ||
        Log2 == ExtendTySize) {
      SDValue Ops[] = { Op0->getOperand(0), Op0->getOperand(1), Op0Op2 };
      return DAG.getNode(MipsISD::VEXTRACT_ZEXT_ELT, SDLoc(Op0),
                         Op0->getVTList(),
                         ArrayRef(Ops, Op0->getNumOperands()));
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 648-649
```cpp
  return SDValue();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 651-661
```cpp
// Determine if the specified node is a constant vector splat.
//
// Returns true and sets Imm if:
// * N is a ISD::BUILD_VECTOR representing a constant splat
//
// This function is quite similar to MipsSEDAGToDAGISel::selectVSplat. The
// differences are that it assumes the MSA has already been checked and the
// arbitrary requirement for a maximum of 32-bit integers isn't applied (and
// must not be in order for binsri.d to be selectable).
static bool isVSplat(SDValue N, APInt &Imm, bool IsLittleEndian) {
  BuildVectorSDNode *Node = dyn_cast<BuildVectorSDNode>(N.getNode());
```
- EN: Implements `isVSplat`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isVSplat`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 663-664
```cpp
  if (!Node)
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 666-668
```cpp
  APInt SplatValue, SplatUndef;
  unsigned SplatBitSize;
  bool HasAnyUndefs;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 670-672
```cpp
  if (!Node->isConstantSplat(SplatValue, SplatUndef, SplatBitSize, HasAnyUndefs,
                             8, !IsLittleEndian))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 674-674
```cpp
  Imm = SplatValue;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 676-677
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 679-684
```cpp
// Test whether the given node is an all-ones build_vector.
static bool isVectorAllOnes(SDValue N) {
  // Look through bitcasts. Endianness doesn't matter because we are looking
  // for an all-ones value.
  if (N->getOpcode() == ISD::BITCAST)
    N = N->getOperand(0);
```
- EN: Implements `isVectorAllOnes`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isVectorAllOnes`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 686-686
```cpp
  BuildVectorSDNode *BVN = dyn_cast<BuildVectorSDNode>(N);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 688-689
```cpp
  if (!BVN)
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 691-693
```cpp
  APInt SplatValue, SplatUndef;
  unsigned SplatBitSize;
  bool HasAnyUndefs;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 695-698
```cpp
  // Endianness doesn't matter in this context because we are looking for
  // an all-ones value.
  if (BVN->isConstantSplat(SplatValue, SplatUndef, SplatBitSize, HasAnyUndefs))
    return SplatValue.isAllOnes();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 700-701
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 703-706
```cpp
// Test whether N is the bitwise inverse of OfNode.
static bool isBitwiseInverse(SDValue N, SDValue OfNode) {
  if (N->getOpcode() != ISD::XOR)
    return false;
```
- EN: Implements `isBitwiseInverse`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isBitwiseInverse`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 708-709
```cpp
  if (isVectorAllOnes(N->getOperand(0)))
    return N->getOperand(1) == OfNode;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 711-712
```cpp
  if (isVectorAllOnes(N->getOperand(1)))
    return N->getOperand(0) == OfNode;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 714-715
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 717-727
```cpp
// Perform combines where ISD::OR is the root node.
//
// Performs the following transformations:
// - (or (and $a, $mask), (and $b, $inv_mask)) => (vselect $mask, $a, $b)
//   where $inv_mask is the bitwise inverse of $mask and the 'or' has a 128-bit
//   vector type.
static SDValue performORCombine(SDNode *N, SelectionDAG &DAG,
                                TargetLowering::DAGCombinerInfo &DCI,
                                const MipsSubtarget &Subtarget) {
  if (!Subtarget.hasMSA())
    return SDValue();
```
- EN: Implements `performORCombine`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `performORCombine`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 729-729
```cpp
  EVT Ty = N->getValueType(0);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 731-732
```cpp
  if (!Ty.is128BitVector())
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 734-735
```cpp
  SDValue Op0 = N->getOperand(0);
  SDValue Op1 = N->getOperand(1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 737-742
```cpp
  if (Op0->getOpcode() == ISD::AND && Op1->getOpcode() == ISD::AND) {
    SDValue Op0Op0 = Op0->getOperand(0);
    SDValue Op0Op1 = Op0->getOperand(1);
    SDValue Op1Op0 = Op1->getOperand(0);
    SDValue Op1Op1 = Op1->getOperand(1);
    bool IsLittleEndian = !Subtarget.isLittle();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 744-746
```cpp
    SDValue IfSet, IfClr, Cond;
    bool IsConstantMask = false;
    APInt Mask, InvMask;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 748-754
```cpp
    // If Op0Op0 is an appropriate mask, try to find it's inverse in either
    // Op1Op0, or Op1Op1. Keep track of the Cond, IfSet, and IfClr nodes, while
    // looking.
    // IfClr will be set if we find a valid match.
    if (isVSplat(Op0Op0, Mask, IsLittleEndian)) {
      Cond = Op0Op0;
      IfSet = Op0Op1;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 756-761
```cpp
      if (isVSplat(Op1Op0, InvMask, IsLittleEndian) &&
          Mask.getBitWidth() == InvMask.getBitWidth() && Mask == ~InvMask)
        IfClr = Op1Op1;
      else if (isVSplat(Op1Op1, InvMask, IsLittleEndian) &&
               Mask.getBitWidth() == InvMask.getBitWidth() && Mask == ~InvMask)
        IfClr = Op1Op0;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 763-764
```cpp
      IsConstantMask = true;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 766-771
```cpp
    // If IfClr is not yet set, and Op0Op1 is an appropriate mask, try the same
    // thing again using this mask.
    // IfClr will be set if we find a valid match.
    if (!IfClr.getNode() && isVSplat(Op0Op1, Mask, IsLittleEndian)) {
      Cond = Op0Op1;
      IfSet = Op0Op0;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 773-778
```cpp
      if (isVSplat(Op1Op0, InvMask, IsLittleEndian) &&
          Mask.getBitWidth() == InvMask.getBitWidth() && Mask == ~InvMask)
        IfClr = Op1Op1;
      else if (isVSplat(Op1Op1, InvMask, IsLittleEndian) &&
               Mask.getBitWidth() == InvMask.getBitWidth() && Mask == ~InvMask)
        IfClr = Op1Op0;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 780-781
```cpp
      IsConstantMask = true;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 783-797
```cpp
    // If IfClr is not yet set, try looking for a non-constant match.
    // IfClr will be set if we find a valid match amongst the eight
    // possibilities.
    if (!IfClr.getNode()) {
      if (isBitwiseInverse(Op0Op0, Op1Op0)) {
        Cond = Op1Op0;
        IfSet = Op1Op1;
        IfClr = Op0Op1;
      } else if (isBitwiseInverse(Op0Op1, Op1Op0)) {
        Cond = Op1Op0;
        IfSet = Op1Op1;
        IfClr = Op0Op0;
      } else if (isBitwiseInverse(Op0Op0, Op1Op1)) {
        Cond = Op1Op1;
        IfSet = Op1Op0;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 798-812
```cpp
        IfClr = Op0Op1;
      } else if (isBitwiseInverse(Op0Op1, Op1Op1)) {
        Cond = Op1Op1;
        IfSet = Op1Op0;
        IfClr = Op0Op0;
      } else if (isBitwiseInverse(Op1Op0, Op0Op0)) {
        Cond = Op0Op0;
        IfSet = Op0Op1;
        IfClr = Op1Op1;
      } else if (isBitwiseInverse(Op1Op1, Op0Op0)) {
        Cond = Op0Op0;
        IfSet = Op0Op1;
        IfClr = Op1Op0;
      } else if (isBitwiseInverse(Op1Op0, Op0Op1)) {
        Cond = Op0Op1;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 813-820
```cpp
        IfSet = Op0Op0;
        IfClr = Op1Op1;
      } else if (isBitwiseInverse(Op1Op1, Op0Op1)) {
        Cond = Op0Op1;
        IfSet = Op0Op0;
        IfClr = Op1Op0;
      }
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 822-824
```cpp
    // At this point, IfClr will be set if we have a valid match.
    if (!IfClr.getNode())
      return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 826-826
```cpp
    assert(Cond.getNode() && IfSet.getNode());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 828-834
```cpp
    // Fold degenerate cases.
    if (IsConstantMask) {
      if (Mask.isAllOnes())
        return IfSet;
      else if (Mask == 0)
        return IfClr;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 836-838
```cpp
    // Transform the DAG into an equivalent VSELECT.
    return DAG.getNode(ISD::VSELECT, SDLoc(N), Ty, Cond, IfSet, IfClr);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 840-841
```cpp
  return SDValue();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 843-857
```cpp
static bool shouldTransformMulToShiftsAddsSubs(APInt C, EVT VT,
                                               SelectionDAG &DAG,
                                               const MipsSubtarget &Subtarget) {
  // Estimate the number of operations the below transform will turn a
  // constant multiply into. The number is approximately equal to the minimal
  // number of powers of two that constant can be broken down to by adding
  // or subtracting them.
  //
  // If we have taken more than 12[1] / 8[2] steps to attempt the
  // optimization for a native sized value, it is more than likely that this
  // optimization will make things worse.
  //
  // [1] MIPS64 requires 6 instructions at most to materialize any constant,
  //     multiplication requires at least 4 cycles, but another cycle (or two)
  //     to retrieve the result from the HI/LO registers.
```
- EN: Implements `shouldTransformMulToShiftsAddsSubs`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `shouldTransformMulToShiftsAddsSubs`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 858-871
```cpp
  //
  // [2] For MIPS32, more than 8 steps is expensive as the constant could be
  //     materialized in 2 instructions, multiplication requires at least 4
  //     cycles, but another cycle (or two) to retrieve the result from the
  //     HI/LO registers.
  //
  // TODO:
  // - MaxSteps needs to consider the `VT` of the constant for the current
  //   target.
  // - Consider to perform this optimization after type legalization.
  //   That allows to remove a workaround for types not supported natively.
  // - Take in account `-Os, -Oz` flags because this optimization
  //   increases code size.
  unsigned MaxSteps = Subtarget.isABI_O32() ? 8 : 12;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 873-875
```cpp
  SmallVector<APInt, 16> WorkStack(1, C);
  unsigned Steps = 0;
  unsigned BitWidth = C.getBitWidth();
```
- EN: Declares `WorkStack`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `WorkStack`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 877-878
```cpp
  while (!WorkStack.empty()) {
    APInt Val = WorkStack.pop_back_val();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 880-881
```cpp
    if (Val == 0 || Val == 1)
      continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 883-884
```cpp
    if (Steps >= MaxSteps)
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 886-889
```cpp
    if (Val.isPowerOf2()) {
      ++Steps;
      continue;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 891-900
```cpp
    APInt Floor = APInt(BitWidth, 1) << Val.logBase2();
    APInt Ceil = Val.isNegative() ? APInt(BitWidth, 0)
                                  : APInt(BitWidth, 1) << C.ceilLogBase2();
    if ((Val - Floor).ule(Ceil - Val)) {
      WorkStack.push_back(Floor);
      WorkStack.push_back(Val - Floor);
    } else {
      WorkStack.push_back(Ceil);
      WorkStack.push_back(Ceil - Val);
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 902-903
```cpp
    ++Steps;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 905-914
```cpp
  // If the value being multiplied is not supported natively, we have to pay
  // an additional legalization cost, conservatively assume an increase in the
  // cost of 3 instructions per step. This values for this heuristic were
  // determined experimentally.
  unsigned RegisterSize = DAG.getTargetLoweringInfo()
                              .getRegisterType(*DAG.getContext(), VT)
                              .getSizeInBits();
  Steps *= (VT.getSizeInBits() != RegisterSize) * 3;
  if (Steps > 27)
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 916-917
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 919-923
```cpp
static SDValue genConstMult(SDValue X, APInt C, const SDLoc &DL, EVT VT,
                            EVT ShiftTy, SelectionDAG &DAG) {
  // Return 0.
  if (C == 0)
    return DAG.getConstant(0, DL, VT);
```
- EN: Implements `genConstMult`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `genConstMult`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 925-927
```cpp
  // Return x.
  if (C == 1)
    return X;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 929-932
```cpp
  // If c is power of 2, return (shl x, log2(c)).
  if (C.isPowerOf2())
    return DAG.getNode(ISD::SHL, DL, VT, X,
                       DAG.getConstant(C.logBase2(), DL, ShiftTy));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 934-937
```cpp
  unsigned BitWidth = C.getBitWidth();
  APInt Floor = APInt(BitWidth, 1) << C.logBase2();
  APInt Ceil = C.isNegative() ? APInt(BitWidth, 0) :
                                APInt(BitWidth, 1) << C.ceilLogBase2();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 939-946
```cpp
  // If |c - floor_c| <= |c - ceil_c|,
  // where floor_c = pow(2, floor(log2(c))) and ceil_c = pow(2, ceil(log2(c))),
  // return (add constMult(x, floor_c), constMult(x, c - floor_c)).
  if ((C - Floor).ule(Ceil - C)) {
    SDValue Op0 = genConstMult(X, Floor, DL, VT, ShiftTy, DAG);
    SDValue Op1 = genConstMult(X, C - Floor, DL, VT, ShiftTy, DAG);
    return DAG.getNode(ISD::ADD, DL, VT, Op0, Op1);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 948-953
```cpp
  // If |c - floor_c| > |c - ceil_c|,
  // return (sub constMult(x, ceil_c), constMult(x, ceil_c - c)).
  SDValue Op0 = genConstMult(X, Ceil, DL, VT, ShiftTy, DAG);
  SDValue Op1 = genConstMult(X, Ceil - C, DL, VT, ShiftTy, DAG);
  return DAG.getNode(ISD::SUB, DL, VT, Op0, Op1);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 955-959
```cpp
static SDValue performMULCombine(SDNode *N, SelectionDAG &DAG,
                                 const TargetLowering::DAGCombinerInfo &DCI,
                                 const MipsSETargetLowering *TL,
                                 const MipsSubtarget &Subtarget) {
  EVT VT = N->getValueType(0);
```
- EN: Implements `performMULCombine`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `performMULCombine`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 961-966
```cpp
  if (ConstantSDNode *C = dyn_cast<ConstantSDNode>(N->getOperand(1)))
    if (!VT.isVector() && shouldTransformMulToShiftsAddsSubs(
                              C->getAPIntValue(), VT, DAG, Subtarget))
      return genConstMult(N->getOperand(0), C->getAPIntValue(), SDLoc(N), VT,
                          TL->getScalarShiftAmountTy(DAG.getDataLayout(), VT),
                          DAG);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 968-969
```cpp
  return SDValue(N, 0);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 971-979
```cpp
static SDValue performDSPShiftCombine(unsigned Opc, SDNode *N, EVT Ty,
                                      SelectionDAG &DAG,
                                      const MipsSubtarget &Subtarget) {
  // See if this is a vector splat immediate node.
  APInt SplatValue, SplatUndef;
  unsigned SplatBitSize;
  bool HasAnyUndefs;
  unsigned EltSize = Ty.getScalarSizeInBits();
  BuildVectorSDNode *BV = dyn_cast<BuildVectorSDNode>(N->getOperand(1));
```
- EN: Implements `performDSPShiftCombine`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `performDSPShiftCombine`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 981-982
```cpp
  if (!Subtarget.hasDSP())
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 984-989
```cpp
  if (!BV ||
      !BV->isConstantSplat(SplatValue, SplatUndef, SplatBitSize, HasAnyUndefs,
                           EltSize, !Subtarget.isLittle()) ||
      (SplatBitSize != EltSize) ||
      (SplatValue.getZExtValue() >= EltSize))
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 991-994
```cpp
  SDLoc DL(N);
  return DAG.getNode(Opc, DL, Ty, N->getOperand(0),
                     DAG.getConstant(SplatValue.getZExtValue(), DL, MVT::i32));
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 996-999
```cpp
static SDValue performSHLCombine(SDNode *N, SelectionDAG &DAG,
                                 TargetLowering::DAGCombinerInfo &DCI,
                                 const MipsSubtarget &Subtarget) {
  EVT Ty = N->getValueType(0);
```
- EN: Implements `performSHLCombine`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `performSHLCombine`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1001-1002
```cpp
  if ((Ty != MVT::v2i16) && (Ty != MVT::v4i8))
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1004-1005
```cpp
  return performDSPShiftCombine(MipsISD::SHLL_DSP, N, Ty, DAG, Subtarget);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1007-1021
```cpp
// Fold sign-extensions into MipsISD::VEXTRACT_[SZ]EXT_ELT for MSA and fold
// constant splats into MipsISD::SHRA_DSP for DSPr2.
//
// Performs the following transformations:
// - Changes MipsISD::VEXTRACT_[SZ]EXT_ELT to sign extension if its
//   sign/zero-extension is completely overwritten by the new one performed by
//   the ISD::SRA and ISD::SHL nodes.
// - Removes redundant sign extensions performed by an ISD::SRA and ISD::SHL
//   sequence.
//
// See performDSPShiftCombine for more information about the transformation
// used for DSPr2.
static SDValue performSRACombine(SDNode *N, SelectionDAG &DAG,
                                 TargetLowering::DAGCombinerInfo &DCI,
                                 const MipsSubtarget &Subtarget) {
```
- EN: Implements `performSRACombine`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `performSRACombine`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1022-1022
```cpp
  EVT Ty = N->getValueType(0);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1024-1026
```cpp
  if (Subtarget.hasMSA()) {
    SDValue Op0 = N->getOperand(0);
    SDValue Op1 = N->getOperand(1);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1028-1034
```cpp
    // (sra (shl (MipsVExtract[SZ]Ext $a, $b, $c), imm:$d), imm:$d)
    // where $d + sizeof($c) == 32
    // or    $d + sizeof($c) <= 32 and SExt
    // -> (MipsVExtractSExt $a, $b, $c)
    if (Op0->getOpcode() == ISD::SHL && Op1 == Op0->getOperand(1)) {
      SDValue Op0Op0 = Op0->getOperand(0);
      ConstantSDNode *ShAmount = dyn_cast<ConstantSDNode>(Op1);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1036-1037
```cpp
      if (!ShAmount)
        return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1039-1041
```cpp
      if (Op0Op0->getOpcode() != MipsISD::VEXTRACT_SEXT_ELT &&
          Op0Op0->getOpcode() != MipsISD::VEXTRACT_ZEXT_ELT)
        return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1043-1044
```cpp
      EVT ExtendTy = cast<VTSDNode>(Op0Op0->getOperand(2))->getVT();
      unsigned TotalBits = ShAmount->getZExtValue() + ExtendTy.getSizeInBits();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1046-1056
```cpp
      if (TotalBits == 32 ||
          (Op0Op0->getOpcode() == MipsISD::VEXTRACT_SEXT_ELT &&
           TotalBits <= 32)) {
        SDValue Ops[] = { Op0Op0->getOperand(0), Op0Op0->getOperand(1),
                          Op0Op0->getOperand(2) };
        return DAG.getNode(MipsISD::VEXTRACT_SEXT_ELT, SDLoc(Op0Op0),
                           Op0Op0->getVTList(),
                           ArrayRef(Ops, Op0Op0->getNumOperands()));
      }
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1058-1059
```cpp
  if ((Ty != MVT::v2i16) && ((Ty != MVT::v4i8) || !Subtarget.hasDSPR2()))
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1061-1062
```cpp
  return performDSPShiftCombine(MipsISD::SHRA_DSP, N, Ty, DAG, Subtarget);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1065-1068
```cpp
static SDValue performSRLCombine(SDNode *N, SelectionDAG &DAG,
                                 TargetLowering::DAGCombinerInfo &DCI,
                                 const MipsSubtarget &Subtarget) {
  EVT Ty = N->getValueType(0);
```
- EN: Implements `performSRLCombine`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `performSRLCombine`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1070-1071
```cpp
  if (((Ty != MVT::v2i16) || !Subtarget.hasDSPR2()) && (Ty != MVT::v4i8))
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1073-1074
```cpp
  return performDSPShiftCombine(MipsISD::SHRL_DSP, N, Ty, DAG, Subtarget);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1076-1077
```cpp
static bool isLegalDSPCondCode(EVT Ty, ISD::CondCode CC) {
  bool IsV216 = (Ty == MVT::v2i16);
```
- EN: Implements `isLegalDSPCondCode`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isLegalDSPCondCode`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1079-1092
```cpp
  switch (CC) {
  case ISD::SETEQ:
  case ISD::SETNE:  return true;
  case ISD::SETLT:
  case ISD::SETLE:
  case ISD::SETGT:
  case ISD::SETGE:  return IsV216;
  case ISD::SETULT:
  case ISD::SETULE:
  case ISD::SETUGT:
  case ISD::SETUGE: return !IsV216;
  default:          return false;
  }
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1094-1095
```cpp
static SDValue performSETCCCombine(SDNode *N, SelectionDAG &DAG) {
  EVT Ty = N->getValueType(0);
```
- EN: Implements `performSETCCCombine`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `performSETCCCombine`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1097-1098
```cpp
  if ((Ty != MVT::v2i16) && (Ty != MVT::v4i8))
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1100-1101
```cpp
  if (!isLegalDSPCondCode(Ty, cast<CondCodeSDNode>(N->getOperand(2))->get()))
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1103-1105
```cpp
  return DAG.getNode(MipsISD::SETCC_DSP, SDLoc(N), Ty, N->getOperand(0),
                     N->getOperand(1), N->getOperand(2));
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1107-1108
```cpp
static SDValue performVSELECTCombine(SDNode *N, SelectionDAG &DAG) {
  EVT Ty = N->getValueType(0);
```
- EN: Implements `performVSELECTCombine`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `performVSELECTCombine`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1110-1111
```cpp
  if (Ty == MVT::v2i16 || Ty == MVT::v4i8) {
    SDValue SetCC = N->getOperand(0);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1113-1114
```cpp
    if (SetCC.getOpcode() != MipsISD::SETCC_DSP)
      return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1116-1119
```cpp
    return DAG.getNode(MipsISD::SELECT_CC_DSP, SDLoc(N), Ty,
                       SetCC.getOperand(0), SetCC.getOperand(1),
                       N->getOperand(1), N->getOperand(2), SetCC.getOperand(2));
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1121-1122
```cpp
  return SDValue();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1124-1126
```cpp
static SDValue performXORCombine(SDNode *N, SelectionDAG &DAG,
                                 const MipsSubtarget &Subtarget) {
  EVT Ty = N->getValueType(0);
```
- EN: Implements `performXORCombine`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `performXORCombine`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1128-1134
```cpp
  if (Subtarget.hasMSA() && Ty.is128BitVector() && Ty.isInteger()) {
    // Try the following combines:
    //   (xor (or $a, $b), (build_vector allones))
    //   (xor (or $a, $b), (bitcast (build_vector allones)))
    SDValue Op0 = N->getOperand(0);
    SDValue Op1 = N->getOperand(1);
    SDValue NotOp;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1136-1141
```cpp
    if (ISD::isBuildVectorAllOnes(Op0.getNode()))
      NotOp = Op1;
    else if (ISD::isBuildVectorAllOnes(Op1.getNode()))
      NotOp = Op0;
    else
      return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1143-1146
```cpp
    if (NotOp->getOpcode() == ISD::OR)
      return DAG.getNode(MipsISD::VNOR, SDLoc(N), Ty, NotOp->getOperand(0),
                         NotOp->getOperand(1));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1148-1149
```cpp
  return SDValue();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1151-1154
```cpp
SDValue
MipsSETargetLowering::PerformDAGCombine(SDNode *N, DAGCombinerInfo &DCI) const {
  SelectionDAG &DAG = DCI.DAG;
  SDValue Val;
```
- EN: Implements `MipsSETargetLowering::PerformDAGCombine`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSETargetLowering::PerformDAGCombine`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1156-1170
```cpp
  switch (N->getOpcode()) {
  case ISD::AND:
    Val = performANDCombine(N, DAG, DCI, Subtarget);
    break;
  case ISD::OR:
    Val = performORCombine(N, DAG, DCI, Subtarget);
    break;
  case ISD::MUL:
    return performMULCombine(N, DAG, DCI, this, Subtarget);
  case ISD::SHL:
    Val = performSHLCombine(N, DAG, DCI, Subtarget);
    break;
  case ISD::SRA:
    return performSRACombine(N, DAG, DCI, Subtarget);
  case ISD::SRL:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1171-1180
```cpp
    return performSRLCombine(N, DAG, DCI, Subtarget);
  case ISD::VSELECT:
    return performVSELECTCombine(N, DAG);
  case ISD::XOR:
    Val = performXORCombine(N, DAG, Subtarget);
    break;
  case ISD::SETCC:
    Val = performSETCCCombine(N, DAG);
    break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1182-1187
```cpp
  if (Val.getNode()) {
    LLVM_DEBUG(dbgs() << "\nMipsSE DAG Combine:\n";
               N->printrWithDepth(dbgs(), &DAG); dbgs() << "\n=> \n";
               Val.getNode()->printrWithDepth(dbgs(), &DAG); dbgs() << "\n");
    return Val;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1189-1190
```cpp
  return MipsTargetLowering::PerformDAGCombine(N, DCI);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1192-1206
```cpp
MachineBasicBlock *
MipsSETargetLowering::EmitInstrWithCustomInserter(MachineInstr &MI,
                                                  MachineBasicBlock *BB) const {
  switch (MI.getOpcode()) {
  default:
    return MipsTargetLowering::EmitInstrWithCustomInserter(MI, BB);
  case Mips::BPOSGE32_PSEUDO:
    return emitBPOSGE32(MI, BB);
  case Mips::SNZ_B_PSEUDO:
    return emitMSACBranchPseudo(MI, BB, Mips::BNZ_B);
  case Mips::SNZ_H_PSEUDO:
    return emitMSACBranchPseudo(MI, BB, Mips::BNZ_H);
  case Mips::SNZ_W_PSEUDO:
    return emitMSACBranchPseudo(MI, BB, Mips::BNZ_W);
  case Mips::SNZ_D_PSEUDO:
```
- EN: Implements `MipsSETargetLowering::EmitInstrWithCustomInserter`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSETargetLowering::EmitInstrWithCustomInserter`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1207-1221
```cpp
    return emitMSACBranchPseudo(MI, BB, Mips::BNZ_D);
  case Mips::SNZ_V_PSEUDO:
    return emitMSACBranchPseudo(MI, BB, Mips::BNZ_V);
  case Mips::SZ_B_PSEUDO:
    return emitMSACBranchPseudo(MI, BB, Mips::BZ_B);
  case Mips::SZ_H_PSEUDO:
    return emitMSACBranchPseudo(MI, BB, Mips::BZ_H);
  case Mips::SZ_W_PSEUDO:
    return emitMSACBranchPseudo(MI, BB, Mips::BZ_W);
  case Mips::SZ_D_PSEUDO:
    return emitMSACBranchPseudo(MI, BB, Mips::BZ_D);
  case Mips::SZ_V_PSEUDO:
    return emitMSACBranchPseudo(MI, BB, Mips::BZ_V);
  case Mips::COPY_FW_PSEUDO:
    return emitCOPY_FW(MI, BB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1222-1236
```cpp
  case Mips::COPY_FD_PSEUDO:
    return emitCOPY_FD(MI, BB);
  case Mips::INSERT_FW_PSEUDO:
    return emitINSERT_FW(MI, BB);
  case Mips::INSERT_FD_PSEUDO:
    return emitINSERT_FD(MI, BB);
  case Mips::INSERT_B_VIDX_PSEUDO:
  case Mips::INSERT_B_VIDX64_PSEUDO:
    return emitINSERT_DF_VIDX(MI, BB, 1, false);
  case Mips::INSERT_H_VIDX_PSEUDO:
  case Mips::INSERT_H_VIDX64_PSEUDO:
    return emitINSERT_DF_VIDX(MI, BB, 2, false);
  case Mips::INSERT_W_VIDX_PSEUDO:
  case Mips::INSERT_W_VIDX64_PSEUDO:
    return emitINSERT_DF_VIDX(MI, BB, 4, false);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1237-1251
```cpp
  case Mips::INSERT_D_VIDX_PSEUDO:
  case Mips::INSERT_D_VIDX64_PSEUDO:
    return emitINSERT_DF_VIDX(MI, BB, 8, false);
  case Mips::INSERT_FW_VIDX_PSEUDO:
  case Mips::INSERT_FW_VIDX64_PSEUDO:
    return emitINSERT_DF_VIDX(MI, BB, 4, true);
  case Mips::INSERT_FD_VIDX_PSEUDO:
  case Mips::INSERT_FD_VIDX64_PSEUDO:
    return emitINSERT_DF_VIDX(MI, BB, 8, true);
  case Mips::FILL_FW_PSEUDO:
    return emitFILL_FW(MI, BB);
  case Mips::FILL_FD_PSEUDO:
    return emitFILL_FD(MI, BB);
  case Mips::FEXP2_W_1_PSEUDO:
    return emitFEXP2_W_1(MI, BB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1252-1266
```cpp
  case Mips::FEXP2_D_1_PSEUDO:
    return emitFEXP2_D_1(MI, BB);
  case Mips::ST_F16:
    return emitST_F16_PSEUDO(MI, BB);
  case Mips::LD_F16:
    return emitLD_F16_PSEUDO(MI, BB);
  case Mips::MSA_FP_EXTEND_W_PSEUDO:
    return emitFPEXTEND_PSEUDO(MI, BB, false);
  case Mips::MSA_FP_ROUND_W_PSEUDO:
    return emitFPROUND_PSEUDO(MI, BB, false);
  case Mips::MSA_FP_EXTEND_D_PSEUDO:
    return emitFPEXTEND_PSEUDO(MI, BB, true);
  case Mips::MSA_FP_ROUND_D_PSEUDO:
    return emitFPROUND_PSEUDO(MI, BB, true);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1267-1267
```cpp
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1269-1274
```cpp
bool MipsSETargetLowering::isEligibleForTailCallOptimization(
    const CCState &CCInfo, unsigned NextStackOffset,
    const MipsFunctionInfo &FI) const {
  // Exception has to be cleared with eret.
  if (FI.isISR())
    return false;
```
- EN: Implements `MipsSETargetLowering::isEligibleForTailCallOptimization`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSETargetLowering::isEligibleForTailCallOptimization`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1276-1278
```cpp
  // Return false if either the callee or caller has a byval argument.
  if (CCInfo.getInRegsParamsCount() > 0 || FI.hasByvalArg())
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1280-1282
```cpp
  // Return true if the callee's argument area is no larger than the caller's.
  return NextStackOffset <= FI.getIncomingArgSize();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1284-1294
```cpp
void MipsSETargetLowering::
getOpndList(SmallVectorImpl<SDValue> &Ops,
            std::deque<std::pair<unsigned, SDValue>> &RegsToPass,
            bool IsPICCall, bool GlobalOrExternal, bool InternalLinkage,
            bool IsCallReloc, CallLoweringInfo &CLI, SDValue Callee,
            SDValue Chain) const {
  Ops.push_back(Callee);
  MipsTargetLowering::getOpndList(Ops, RegsToPass, IsPICCall, GlobalOrExternal,
                                  InternalLinkage, IsCallReloc, CLI, Callee,
                                  Chain);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1296-1297
```cpp
SDValue MipsSETargetLowering::lowerLOAD(SDValue Op, SelectionDAG &DAG) const {
  LoadSDNode &Nd = *cast<LoadSDNode>(Op);
```
- EN: Implements `MipsSETargetLowering::lowerLOAD`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSETargetLowering::lowerLOAD`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1299-1300
```cpp
  if (Nd.getMemoryVT() != MVT::f64 || !NoDPLoadStore)
    return MipsTargetLowering::lowerLOAD(Op, DAG);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1302-1305
```cpp
  // Replace a double precision load with two i32 loads and a buildpair64.
  SDLoc DL(Op);
  SDValue Ptr = Nd.getBasePtr(), Chain = Nd.getChain();
  EVT PtrVT = Ptr.getValueType();
```
- EN: Declares `DL`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `DL`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1307-1309
```cpp
  // i32 load from lower address.
  SDValue Lo = DAG.getLoad(MVT::i32, DL, Chain, Ptr, MachinePointerInfo(),
                           Nd.getAlign(), Nd.getMemOperand()->getFlags());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1311-1315
```cpp
  // i32 load from higher address.
  Ptr = DAG.getNode(ISD::ADD, DL, PtrVT, Ptr, DAG.getConstant(4, DL, PtrVT));
  SDValue Hi = DAG.getLoad(
      MVT::i32, DL, Lo.getValue(1), Ptr, MachinePointerInfo(),
      commonAlignment(Nd.getAlign(), 4), Nd.getMemOperand()->getFlags());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1317-1318
```cpp
  if (!Subtarget.isLittle())
    std::swap(Lo, Hi);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1320-1323
```cpp
  SDValue BP = DAG.getNode(MipsISD::BuildPairF64, DL, MVT::f64, Lo, Hi);
  SDValue Ops[2] = {BP, Hi.getValue(1)};
  return DAG.getMergeValues(Ops, DL);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1325-1326
```cpp
SDValue MipsSETargetLowering::lowerSTORE(SDValue Op, SelectionDAG &DAG) const {
  StoreSDNode &Nd = *cast<StoreSDNode>(Op);
```
- EN: Implements `MipsSETargetLowering::lowerSTORE`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSETargetLowering::lowerSTORE`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1328-1329
```cpp
  if (Nd.getMemoryVT() != MVT::f64 || !NoDPLoadStore)
    return MipsTargetLowering::lowerSTORE(Op, DAG);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1331-1338
```cpp
  // Replace a double precision store with two extractelement64s and i32 stores.
  SDLoc DL(Op);
  SDValue Val = Nd.getValue(), Ptr = Nd.getBasePtr(), Chain = Nd.getChain();
  EVT PtrVT = Ptr.getValueType();
  SDValue Lo = DAG.getNode(MipsISD::ExtractElementF64, DL, MVT::i32,
                           Val, DAG.getConstant(0, DL, MVT::i32));
  SDValue Hi = DAG.getNode(MipsISD::ExtractElementF64, DL, MVT::i32,
                           Val, DAG.getConstant(1, DL, MVT::i32));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1340-1341
```cpp
  if (!Subtarget.isLittle())
    std::swap(Lo, Hi);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1343-1345
```cpp
  // i32 store to lower address.
  Chain = DAG.getStore(Chain, DL, Lo, Ptr, MachinePointerInfo(), Nd.getAlign(),
                       Nd.getMemOperand()->getFlags(), Nd.getAAInfo());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1347-1352
```cpp
  // i32 store to higher address.
  Ptr = DAG.getNode(ISD::ADD, DL, PtrVT, Ptr, DAG.getConstant(4, DL, PtrVT));
  return DAG.getStore(Chain, DL, Hi, Ptr, MachinePointerInfo(),
                      commonAlignment(Nd.getAlign(), 4),
                      Nd.getMemOperand()->getFlags(), Nd.getAAInfo());
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1354-1358
```cpp
SDValue MipsSETargetLowering::lowerBITCAST(SDValue Op,
                                           SelectionDAG &DAG) const {
  SDLoc DL(Op);
  MVT Src = Op.getOperand(0).getValueType().getSimpleVT();
  MVT Dest = Op.getValueType().getSimpleVT();
```
- EN: Implements `MipsSETargetLowering::lowerBITCAST`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSETargetLowering::lowerBITCAST`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1360-1366
```cpp
  // Bitcast i64 to double.
  if (Src == MVT::i64 && Dest == MVT::f64) {
    SDValue Lo, Hi;
    std::tie(Lo, Hi) =
        DAG.SplitScalar(Op.getOperand(0), DL, MVT::i32, MVT::i32);
    return DAG.getNode(MipsISD::BuildPairF64, DL, MVT::f64, Lo, Hi);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1368-1382
```cpp
  // Bitcast double to i64.
  if (Src == MVT::f64 && Dest == MVT::i64) {
    // Skip lower bitcast when operand0 has converted float results to integer
    // which was done by function SoftenFloatResult.
    if (getTypeAction(*DAG.getContext(), Op.getOperand(0).getValueType()) ==
        TargetLowering::TypeSoftenFloat)
      return SDValue();
    SDValue Lo =
        DAG.getNode(MipsISD::ExtractElementF64, DL, MVT::i32, Op.getOperand(0),
                    DAG.getConstant(0, DL, MVT::i32));
    SDValue Hi =
        DAG.getNode(MipsISD::ExtractElementF64, DL, MVT::i32, Op.getOperand(0),
                    DAG.getConstant(1, DL, MVT::i32));
    return DAG.getNode(ISD::BUILD_PAIR, DL, MVT::i64, Lo, Hi);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1384-1386
```cpp
  // Skip other cases of bitcast and use default lowering.
  return SDValue();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1388-1392
```cpp
SDValue MipsSETargetLowering::lowerMulDiv(SDValue Op, unsigned NewOpc,
                                          bool HasLo, bool HasHi,
                                          SelectionDAG &DAG) const {
  // MIPS32r6/MIPS64r6 removed accumulator based multiplies.
  assert(!Subtarget.hasMips32r6());
```
- EN: Implements `MipsSETargetLowering::lowerMulDiv`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSETargetLowering::lowerMulDiv`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1394-1398
```cpp
  EVT Ty = Op.getOperand(0).getValueType();
  SDLoc DL(Op);
  SDValue Mult = DAG.getNode(NewOpc, DL, MVT::Untyped,
                             Op.getOperand(0), Op.getOperand(1));
  SDValue Lo, Hi;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1400-1403
```cpp
  if (HasLo)
    Lo = DAG.getNode(MipsISD::MFLO, DL, Ty, Mult);
  if (HasHi)
    Hi = DAG.getNode(MipsISD::MFHI, DL, Ty, Mult);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1405-1406
```cpp
  if (!HasLo || !HasHi)
    return HasLo ? Lo : Hi;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1408-1410
```cpp
  SDValue Vals[] = { Lo, Hi };
  return DAG.getMergeValues(Vals, DL);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1412-1416
```cpp
static SDValue initAccumulator(SDValue In, const SDLoc &DL, SelectionDAG &DAG) {
  SDValue InLo, InHi;
  std::tie(InLo, InHi) = DAG.SplitScalar(In, DL, MVT::i32, MVT::i32);
  return DAG.getNode(MipsISD::MTLOHI, DL, MVT::Untyped, InLo, InHi);
}
```
- EN: Implements `initAccumulator`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `initAccumulator`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1418-1422
```cpp
static SDValue extractLOHI(SDValue Op, const SDLoc &DL, SelectionDAG &DAG) {
  SDValue Lo = DAG.getNode(MipsISD::MFLO, DL, MVT::i32, Op);
  SDValue Hi = DAG.getNode(MipsISD::MFHI, DL, MVT::i32, Op);
  return DAG.getNode(ISD::BUILD_PAIR, DL, MVT::i64, Lo, Hi);
}
```
- EN: Implements `extractLOHI`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `extractLOHI`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1424-1438
```cpp
// This function expands mips intrinsic nodes which have 64-bit input operands
// or output values.
//
// out64 = intrinsic-node in64
// =>
// lo = copy (extract-element (in64, 0))
// hi = copy (extract-element (in64, 1))
// mips-specific-node
// v0 = copy lo
// v1 = copy hi
// out64 = merge-values (v0, v1)
//
static SDValue lowerDSPIntr(SDValue Op, SelectionDAG &DAG, unsigned Opc) {
  SDLoc DL(Op);
  bool HasChainIn = Op->getOperand(0).getValueType() == MVT::Other;
```
- EN: Implements `lowerDSPIntr`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `lowerDSPIntr`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1439-1440
```cpp
  SmallVector<SDValue, 3> Ops;
  unsigned OpNo = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1442-1444
```cpp
  // See if Op has a chain input.
  if (HasChainIn)
    Ops.push_back(Op->getOperand(OpNo++));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1446-1447
```cpp
  // The next operand is the intrinsic opcode.
  assert(Op->getOperand(OpNo).getOpcode() == ISD::TargetConstant);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1449-1450
```cpp
  // See if the next operand has type i64.
  SDValue Opnd = Op->getOperand(++OpNo), In64;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1452-1455
```cpp
  if (Opnd.getValueType() == MVT::i64)
    In64 = initAccumulator(Opnd, DL, DAG);
  else
    Ops.push_back(Opnd);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1457-1459
```cpp
  // Push the remaining operands.
  for (++OpNo ; OpNo < Op->getNumOperands(); ++OpNo)
    Ops.push_back(Op->getOperand(OpNo));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1461-1463
```cpp
  // Add In64 to the end of the list.
  if (In64.getNode())
    Ops.push_back(In64);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1465-1466
```cpp
  // Scan output.
  SmallVector<EVT, 2> ResTys;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1468-1469
```cpp
  for (EVT Ty : Op->values())
    ResTys.push_back((Ty == MVT::i64) ? MVT::Untyped : Ty);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1471-1473
```cpp
  // Create node.
  SDValue Val = DAG.getNode(Opc, DL, ResTys, Ops);
  SDValue Out = (ResTys[0] == MVT::Untyped) ? extractLOHI(Val, DL, DAG) : Val;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1475-1476
```cpp
  if (!HasChainIn)
    return Out;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1478-1481
```cpp
  assert(Val->getValueType(1) == MVT::Other);
  SDValue Vals[] = { Out, SDValue(Val.getNode(), 1) };
  return DAG.getMergeValues(Vals, DL);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1483-1489
```cpp
// Lower an MSA copy intrinsic into the specified SelectionDAG node
static SDValue lowerMSACopyIntr(SDValue Op, SelectionDAG &DAG, unsigned Opc) {
  SDLoc DL(Op);
  SDValue Vec = Op->getOperand(1);
  SDValue Idx = Op->getOperand(2);
  EVT ResTy = Op->getValueType(0);
  EVT EltTy = Vec->getValueType(0).getVectorElementType();
```
- EN: Implements `lowerMSACopyIntr`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `lowerMSACopyIntr`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1491-1492
```cpp
  SDValue Result = DAG.getNode(Opc, DL, ResTy, Vec, Idx,
                               DAG.getValueType(EltTy));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1494-1495
```cpp
  return Result;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1497-1501
```cpp
static SDValue lowerMSASplatZExt(SDValue Op, unsigned OpNr, SelectionDAG &DAG) {
  EVT ResVecTy = Op->getValueType(0);
  EVT ViaVecTy = ResVecTy;
  bool BigEndian = !DAG.getSubtarget().getTargetTriple().isLittleEndian();
  SDLoc DL(Op);
```
- EN: Implements `lowerMSASplatZExt`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `lowerMSASplatZExt`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1503-1507
```cpp
  // When ResVecTy == MVT::v2i64, LaneA is the upper 32 bits of the lane and
  // LaneB is the lower 32-bits. Otherwise LaneA and LaneB are alternating
  // lanes.
  SDValue LaneA = Op->getOperand(OpNr);
  SDValue LaneB;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1509-1523
```cpp
  if (ResVecTy == MVT::v2i64) {
    // In case of the index being passed as an immediate value, set the upper
    // lane to 0 so that the splati.d instruction can be matched.
    if (isa<ConstantSDNode>(LaneA))
      LaneB = DAG.getConstant(0, DL, MVT::i32);
    // Having the index passed in a register, set the upper lane to the same
    // value as the lower - this results in the BUILD_VECTOR node not being
    // expanded through stack. This way we are able to pattern match the set of
    // nodes created here to splat.d.
    else
      LaneB = LaneA;
    ViaVecTy = MVT::v4i32;
    if(BigEndian)
      std::swap(LaneA, LaneB);
  } else
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1524-1524
```cpp
    LaneB = LaneA;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1526-1527
```cpp
  SDValue Ops[16] = { LaneA, LaneB, LaneA, LaneB, LaneA, LaneB, LaneA, LaneB,
                      LaneA, LaneB, LaneA, LaneB, LaneA, LaneB, LaneA, LaneB };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1529-1530
```cpp
  SDValue Result = DAG.getBuildVector(
      ViaVecTy, DL, ArrayRef(Ops, ViaVecTy.getVectorNumElements()));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1532-1536
```cpp
  if (ViaVecTy != ResVecTy) {
    SDValue One = DAG.getConstant(1, DL, ViaVecTy);
    Result = DAG.getNode(ISD::BITCAST, DL, ResVecTy,
                         DAG.getNode(ISD::AND, DL, ViaVecTy, Result, One));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1538-1539
```cpp
  return Result;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1541-1548
```cpp
static SDValue lowerMSASplatImm(SDValue Op, unsigned ImmOp, SelectionDAG &DAG,
                                bool IsSigned = false) {
  auto *CImm = cast<ConstantSDNode>(Op->getOperand(ImmOp));
  return DAG.getConstant(
      APInt(Op->getValueType(0).getScalarType().getSizeInBits(),
            IsSigned ? CImm->getSExtValue() : CImm->getZExtValue(), IsSigned),
      SDLoc(Op), Op->getValueType(0));
}
```
- EN: Implements `lowerMSASplatImm`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `lowerMSASplatImm`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1550-1555
```cpp
static SDValue getBuildVectorSplat(EVT VecTy, SDValue SplatValue,
                                   bool BigEndian, SelectionDAG &DAG) {
  EVT ViaVecTy = VecTy;
  SDValue SplatValueA = SplatValue;
  SDValue SplatValueB = SplatValue;
  SDLoc DL(SplatValue);
```
- EN: Implements `getBuildVectorSplat`, a query/helper routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getBuildVectorSplat`，它是一个围绕SelectionDAG 降级展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1557-1559
```cpp
  if (VecTy == MVT::v2i64) {
    // v2i64 BUILD_VECTOR must be performed via v4i32 so split into i32's.
    ViaVecTy = MVT::v4i32;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1561-1565
```cpp
    SplatValueA = DAG.getNode(ISD::TRUNCATE, DL, MVT::i32, SplatValue);
    SplatValueB = DAG.getNode(ISD::SRL, DL, MVT::i64, SplatValue,
                              DAG.getConstant(32, DL, MVT::i32));
    SplatValueB = DAG.getNode(ISD::TRUNCATE, DL, MVT::i32, SplatValueB);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1567-1570
```cpp
  // We currently hold the parts in little endian order. Swap them if
  // necessary.
  if (BigEndian)
    std::swap(SplatValueA, SplatValueB);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1572-1575
```cpp
  SDValue Ops[16] = { SplatValueA, SplatValueB, SplatValueA, SplatValueB,
                      SplatValueA, SplatValueB, SplatValueA, SplatValueB,
                      SplatValueA, SplatValueB, SplatValueA, SplatValueB,
                      SplatValueA, SplatValueB, SplatValueA, SplatValueB };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1577-1578
```cpp
  SDValue Result = DAG.getBuildVector(
      ViaVecTy, DL, ArrayRef(Ops, ViaVecTy.getVectorNumElements()));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1580-1581
```cpp
  if (VecTy != ViaVecTy)
    Result = DAG.getNode(ISD::BITCAST, DL, VecTy, Result);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1583-1584
```cpp
  return Result;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1586-1591
```cpp
static SDValue lowerMSABinaryBitImmIntr(SDValue Op, SelectionDAG &DAG,
                                        unsigned Opc, SDValue Imm,
                                        bool BigEndian) {
  EVT VecTy = Op->getValueType(0);
  SDValue Exp2Imm;
  SDLoc DL(Op);
```
- EN: Implements `lowerMSABinaryBitImmIntr`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `lowerMSABinaryBitImmIntr`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1593-1597
```cpp
  // The DAG Combiner can't constant fold bitcasted vectors yet so we must do it
  // here for now.
  if (VecTy == MVT::v2i64) {
    if (ConstantSDNode *CImm = dyn_cast<ConstantSDNode>(Imm)) {
      APInt BitImm = APInt(64, 1) << CImm->getAPIntValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1599-1601
```cpp
      SDValue BitImmHiOp = DAG.getConstant(BitImm.lshr(32).trunc(32), DL,
                                           MVT::i32);
      SDValue BitImmLoOp = DAG.getConstant(BitImm.trunc(32), DL, MVT::i32);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1603-1604
```cpp
      if (BigEndian)
        std::swap(BitImmLoOp, BitImmHiOp);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1606-1611
```cpp
      Exp2Imm = DAG.getNode(
          ISD::BITCAST, DL, MVT::v2i64,
          DAG.getBuildVector(MVT::v4i32, DL,
                             {BitImmLoOp, BitImmHiOp, BitImmLoOp, BitImmHiOp}));
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1613-1614
```cpp
  if (!Exp2Imm.getNode()) {
    // We couldnt constant fold, do a vector shift instead
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1616-1619
```cpp
    // Extend i32 to i64 if necessary. Sign or zero extend doesn't matter since
    // only values 0-63 are valid.
    if (VecTy == MVT::v2i64)
      Imm = DAG.getNode(ISD::ZERO_EXTEND, DL, MVT::i64, Imm);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1621-1621
```cpp
    Exp2Imm = getBuildVectorSplat(VecTy, Imm, BigEndian, DAG);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1623-1625
```cpp
    Exp2Imm = DAG.getNode(ISD::SHL, DL, VecTy, DAG.getConstant(1, DL, VecTy),
                          Exp2Imm);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1627-1628
```cpp
  return DAG.getNode(Opc, DL, VecTy, Op->getOperand(1), Exp2Imm);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1630-1638
```cpp
static SDValue truncateVecElts(SDValue Op, SelectionDAG &DAG) {
  SDLoc DL(Op);
  EVT ResTy = Op->getValueType(0);
  SDValue Vec = Op->getOperand(2);
  bool BigEndian = !DAG.getSubtarget().getTargetTriple().isLittleEndian();
  MVT ResEltTy = ResTy == MVT::v2i64 ? MVT::i64 : MVT::i32;
  SDValue ConstValue = DAG.getConstant(Vec.getScalarValueSizeInBits() - 1,
                                       DL, ResEltTy);
  SDValue SplatVec = getBuildVectorSplat(ResTy, ConstValue, BigEndian, DAG);
```
- EN: Implements `truncateVecElts`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `truncateVecElts`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1640-1641
```cpp
  return DAG.getNode(ISD::AND, DL, ResTy, Vec, SplatVec);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1643-1647
```cpp
static SDValue lowerMSABitClear(SDValue Op, SelectionDAG &DAG) {
  EVT ResTy = Op->getValueType(0);
  SDLoc DL(Op);
  SDValue One = DAG.getConstant(1, DL, ResTy);
  SDValue Bit = DAG.getNode(ISD::SHL, DL, ResTy, One, truncateVecElts(Op, DAG));
```
- EN: Implements `lowerMSABitClear`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `lowerMSABitClear`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1649-1651
```cpp
  return DAG.getNode(ISD::AND, DL, ResTy, Op->getOperand(1),
                     DAG.getNOT(DL, Bit, ResTy));
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1653-1658
```cpp
static SDValue lowerMSABitClearImm(SDValue Op, SelectionDAG &DAG) {
  SDLoc DL(Op);
  EVT ResTy = Op->getValueType(0);
  APInt BitImm = APInt(ResTy.getScalarSizeInBits(), 1)
                 << Op->getConstantOperandAPInt(2);
  SDValue BitMask = DAG.getConstant(~BitImm, DL, ResTy);
```
- EN: Implements `lowerMSABitClearImm`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `lowerMSABitClearImm`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1660-1661
```cpp
  return DAG.getNode(ISD::AND, DL, ResTy, Op->getOperand(1), BitMask);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1663-1677
```cpp
SDValue MipsSETargetLowering::lowerINTRINSIC_WO_CHAIN(SDValue Op,
                                                      SelectionDAG &DAG) const {
  SDLoc DL(Op);
  unsigned Intrinsic = Op->getConstantOperandVal(0);
  switch (Intrinsic) {
  default:
    return SDValue();
  case Intrinsic::mips_shilo:
    return lowerDSPIntr(Op, DAG, MipsISD::SHILO);
  case Intrinsic::mips_dpau_h_qbl:
    return lowerDSPIntr(Op, DAG, MipsISD::DPAU_H_QBL);
  case Intrinsic::mips_dpau_h_qbr:
    return lowerDSPIntr(Op, DAG, MipsISD::DPAU_H_QBR);
  case Intrinsic::mips_dpsu_h_qbl:
    return lowerDSPIntr(Op, DAG, MipsISD::DPSU_H_QBL);
```
- EN: Implements `MipsSETargetLowering::lowerINTRINSIC_WO_CHAIN`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSETargetLowering::lowerINTRINSIC_WO_CHAIN`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1678-1692
```cpp
  case Intrinsic::mips_dpsu_h_qbr:
    return lowerDSPIntr(Op, DAG, MipsISD::DPSU_H_QBR);
  case Intrinsic::mips_dpa_w_ph:
    return lowerDSPIntr(Op, DAG, MipsISD::DPA_W_PH);
  case Intrinsic::mips_dps_w_ph:
    return lowerDSPIntr(Op, DAG, MipsISD::DPS_W_PH);
  case Intrinsic::mips_dpax_w_ph:
    return lowerDSPIntr(Op, DAG, MipsISD::DPAX_W_PH);
  case Intrinsic::mips_dpsx_w_ph:
    return lowerDSPIntr(Op, DAG, MipsISD::DPSX_W_PH);
  case Intrinsic::mips_mulsa_w_ph:
    return lowerDSPIntr(Op, DAG, MipsISD::MULSA_W_PH);
  case Intrinsic::mips_mult:
    return lowerDSPIntr(Op, DAG, MipsISD::Mult);
  case Intrinsic::mips_multu:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1693-1707
```cpp
    return lowerDSPIntr(Op, DAG, MipsISD::Multu);
  case Intrinsic::mips_madd:
    return lowerDSPIntr(Op, DAG, MipsISD::MAdd);
  case Intrinsic::mips_maddu:
    return lowerDSPIntr(Op, DAG, MipsISD::MAddu);
  case Intrinsic::mips_msub:
    return lowerDSPIntr(Op, DAG, MipsISD::MSub);
  case Intrinsic::mips_msubu:
    return lowerDSPIntr(Op, DAG, MipsISD::MSubu);
  case Intrinsic::mips_addv_b:
  case Intrinsic::mips_addv_h:
  case Intrinsic::mips_addv_w:
  case Intrinsic::mips_addv_d:
    return DAG.getNode(ISD::ADD, DL, Op->getValueType(0), Op->getOperand(1),
                       Op->getOperand(2));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1708-1722
```cpp
  case Intrinsic::mips_addvi_b:
  case Intrinsic::mips_addvi_h:
  case Intrinsic::mips_addvi_w:
  case Intrinsic::mips_addvi_d:
    return DAG.getNode(ISD::ADD, DL, Op->getValueType(0), Op->getOperand(1),
                       lowerMSASplatImm(Op, 2, DAG));
  case Intrinsic::mips_and_v:
    return DAG.getNode(ISD::AND, DL, Op->getValueType(0), Op->getOperand(1),
                       Op->getOperand(2));
  case Intrinsic::mips_andi_b:
    return DAG.getNode(ISD::AND, DL, Op->getValueType(0), Op->getOperand(1),
                       lowerMSASplatImm(Op, 2, DAG));
  case Intrinsic::mips_bclr_b:
  case Intrinsic::mips_bclr_h:
  case Intrinsic::mips_bclr_w:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1723-1737
```cpp
  case Intrinsic::mips_bclr_d:
    return lowerMSABitClear(Op, DAG);
  case Intrinsic::mips_bclri_b:
  case Intrinsic::mips_bclri_h:
  case Intrinsic::mips_bclri_w:
  case Intrinsic::mips_bclri_d:
    return lowerMSABitClearImm(Op, DAG);
  case Intrinsic::mips_binsli_b:
  case Intrinsic::mips_binsli_h:
  case Intrinsic::mips_binsli_w:
  case Intrinsic::mips_binsli_d: {
    // binsli_x(IfClear, IfSet, nbits) -> (vselect LBitsMask, IfSet, IfClear)
    EVT VecTy = Op->getValueType(0);
    EVT EltTy = VecTy.getVectorElementType();
    if (Op->getConstantOperandVal(3) >= EltTy.getSizeInBits())
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1738-1752
```cpp
      report_fatal_error("Immediate out of range");
    APInt Mask = APInt::getHighBitsSet(EltTy.getSizeInBits(),
                                       Op->getConstantOperandVal(3) + 1);
    return DAG.getNode(ISD::VSELECT, DL, VecTy,
                       DAG.getConstant(Mask, DL, VecTy, true),
                       Op->getOperand(2), Op->getOperand(1));
  }
  case Intrinsic::mips_binsri_b:
  case Intrinsic::mips_binsri_h:
  case Intrinsic::mips_binsri_w:
  case Intrinsic::mips_binsri_d: {
    // binsri_x(IfClear, IfSet, nbits) -> (vselect RBitsMask, IfSet, IfClear)
    EVT VecTy = Op->getValueType(0);
    EVT EltTy = VecTy.getVectorElementType();
    if (Op->getConstantOperandVal(3) >= EltTy.getSizeInBits())
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1753-1767
```cpp
      report_fatal_error("Immediate out of range");
    APInt Mask = APInt::getLowBitsSet(EltTy.getSizeInBits(),
                                      Op->getConstantOperandVal(3) + 1);
    return DAG.getNode(ISD::VSELECT, DL, VecTy,
                       DAG.getConstant(Mask, DL, VecTy, true),
                       Op->getOperand(2), Op->getOperand(1));
  }
  case Intrinsic::mips_bmnz_v:
    return DAG.getNode(ISD::VSELECT, DL, Op->getValueType(0), Op->getOperand(3),
                       Op->getOperand(2), Op->getOperand(1));
  case Intrinsic::mips_bmnzi_b:
    return DAG.getNode(ISD::VSELECT, DL, Op->getValueType(0),
                       lowerMSASplatImm(Op, 3, DAG), Op->getOperand(2),
                       Op->getOperand(1));
  case Intrinsic::mips_bmz_v:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1768-1779
```cpp
    return DAG.getNode(ISD::VSELECT, DL, Op->getValueType(0), Op->getOperand(3),
                       Op->getOperand(1), Op->getOperand(2));
  case Intrinsic::mips_bmzi_b:
    return DAG.getNode(ISD::VSELECT, DL, Op->getValueType(0),
                       lowerMSASplatImm(Op, 3, DAG), Op->getOperand(1),
                       Op->getOperand(2));
  case Intrinsic::mips_bneg_b:
  case Intrinsic::mips_bneg_h:
  case Intrinsic::mips_bneg_w:
  case Intrinsic::mips_bneg_d: {
    EVT VecTy = Op->getValueType(0);
    SDValue One = DAG.getConstant(1, DL, VecTy);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1781-1795
```cpp
    return DAG.getNode(ISD::XOR, DL, VecTy, Op->getOperand(1),
                       DAG.getNode(ISD::SHL, DL, VecTy, One,
                                   truncateVecElts(Op, DAG)));
  }
  case Intrinsic::mips_bnegi_b:
  case Intrinsic::mips_bnegi_h:
  case Intrinsic::mips_bnegi_w:
  case Intrinsic::mips_bnegi_d:
    return lowerMSABinaryBitImmIntr(Op, DAG, ISD::XOR, Op->getOperand(2),
                                    !Subtarget.isLittle());
  case Intrinsic::mips_bnz_b:
  case Intrinsic::mips_bnz_h:
  case Intrinsic::mips_bnz_w:
  case Intrinsic::mips_bnz_d:
    return DAG.getNode(MipsISD::VALL_NONZERO, DL, Op->getValueType(0),
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1796-1810
```cpp
                       Op->getOperand(1));
  case Intrinsic::mips_bnz_v:
    return DAG.getNode(MipsISD::VANY_NONZERO, DL, Op->getValueType(0),
                       Op->getOperand(1));
  case Intrinsic::mips_bsel_v:
    // bsel_v(Mask, IfClear, IfSet) -> (vselect Mask, IfSet, IfClear)
    return DAG.getNode(ISD::VSELECT, DL, Op->getValueType(0),
                       Op->getOperand(1), Op->getOperand(3),
                       Op->getOperand(2));
  case Intrinsic::mips_bseli_b:
    // bseli_v(Mask, IfClear, IfSet) -> (vselect Mask, IfSet, IfClear)
    return DAG.getNode(ISD::VSELECT, DL, Op->getValueType(0),
                       Op->getOperand(1), lowerMSASplatImm(Op, 3, DAG),
                       Op->getOperand(2));
  case Intrinsic::mips_bset_b:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1811-1815
```cpp
  case Intrinsic::mips_bset_h:
  case Intrinsic::mips_bset_w:
  case Intrinsic::mips_bset_d: {
    EVT VecTy = Op->getValueType(0);
    SDValue One = DAG.getConstant(1, DL, VecTy);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1817-1831
```cpp
    return DAG.getNode(ISD::OR, DL, VecTy, Op->getOperand(1),
                       DAG.getNode(ISD::SHL, DL, VecTy, One,
                                   truncateVecElts(Op, DAG)));
  }
  case Intrinsic::mips_bseti_b:
  case Intrinsic::mips_bseti_h:
  case Intrinsic::mips_bseti_w:
  case Intrinsic::mips_bseti_d:
    return lowerMSABinaryBitImmIntr(Op, DAG, ISD::OR, Op->getOperand(2),
                                    !Subtarget.isLittle());
  case Intrinsic::mips_bz_b:
  case Intrinsic::mips_bz_h:
  case Intrinsic::mips_bz_w:
  case Intrinsic::mips_bz_d:
    return DAG.getNode(MipsISD::VALL_ZERO, DL, Op->getValueType(0),
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1832-1846
```cpp
                       Op->getOperand(1));
  case Intrinsic::mips_bz_v:
    return DAG.getNode(MipsISD::VANY_ZERO, DL, Op->getValueType(0),
                       Op->getOperand(1));
  case Intrinsic::mips_ceq_b:
  case Intrinsic::mips_ceq_h:
  case Intrinsic::mips_ceq_w:
  case Intrinsic::mips_ceq_d:
    return DAG.getSetCC(DL, Op->getValueType(0), Op->getOperand(1),
                        Op->getOperand(2), ISD::SETEQ);
  case Intrinsic::mips_ceqi_b:
  case Intrinsic::mips_ceqi_h:
  case Intrinsic::mips_ceqi_w:
  case Intrinsic::mips_ceqi_d:
    return DAG.getSetCC(DL, Op->getValueType(0), Op->getOperand(1),
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1847-1861
```cpp
                        lowerMSASplatImm(Op, 2, DAG, true), ISD::SETEQ);
  case Intrinsic::mips_cle_s_b:
  case Intrinsic::mips_cle_s_h:
  case Intrinsic::mips_cle_s_w:
  case Intrinsic::mips_cle_s_d:
    return DAG.getSetCC(DL, Op->getValueType(0), Op->getOperand(1),
                        Op->getOperand(2), ISD::SETLE);
  case Intrinsic::mips_clei_s_b:
  case Intrinsic::mips_clei_s_h:
  case Intrinsic::mips_clei_s_w:
  case Intrinsic::mips_clei_s_d:
    return DAG.getSetCC(DL, Op->getValueType(0), Op->getOperand(1),
                        lowerMSASplatImm(Op, 2, DAG, true), ISD::SETLE);
  case Intrinsic::mips_cle_u_b:
  case Intrinsic::mips_cle_u_h:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1862-1876
```cpp
  case Intrinsic::mips_cle_u_w:
  case Intrinsic::mips_cle_u_d:
    return DAG.getSetCC(DL, Op->getValueType(0), Op->getOperand(1),
                        Op->getOperand(2), ISD::SETULE);
  case Intrinsic::mips_clei_u_b:
  case Intrinsic::mips_clei_u_h:
  case Intrinsic::mips_clei_u_w:
  case Intrinsic::mips_clei_u_d:
    return DAG.getSetCC(DL, Op->getValueType(0), Op->getOperand(1),
                        lowerMSASplatImm(Op, 2, DAG), ISD::SETULE);
  case Intrinsic::mips_clt_s_b:
  case Intrinsic::mips_clt_s_h:
  case Intrinsic::mips_clt_s_w:
  case Intrinsic::mips_clt_s_d:
    return DAG.getSetCC(DL, Op->getValueType(0), Op->getOperand(1),
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1877-1891
```cpp
                        Op->getOperand(2), ISD::SETLT);
  case Intrinsic::mips_clti_s_b:
  case Intrinsic::mips_clti_s_h:
  case Intrinsic::mips_clti_s_w:
  case Intrinsic::mips_clti_s_d:
    return DAG.getSetCC(DL, Op->getValueType(0), Op->getOperand(1),
                        lowerMSASplatImm(Op, 2, DAG, true), ISD::SETLT);
  case Intrinsic::mips_clt_u_b:
  case Intrinsic::mips_clt_u_h:
  case Intrinsic::mips_clt_u_w:
  case Intrinsic::mips_clt_u_d:
    return DAG.getSetCC(DL, Op->getValueType(0), Op->getOperand(1),
                        Op->getOperand(2), ISD::SETULT);
  case Intrinsic::mips_clti_u_b:
  case Intrinsic::mips_clti_u_h:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1892-1906
```cpp
  case Intrinsic::mips_clti_u_w:
  case Intrinsic::mips_clti_u_d:
    return DAG.getSetCC(DL, Op->getValueType(0), Op->getOperand(1),
                        lowerMSASplatImm(Op, 2, DAG), ISD::SETULT);
  case Intrinsic::mips_copy_s_b:
  case Intrinsic::mips_copy_s_h:
  case Intrinsic::mips_copy_s_w:
    return lowerMSACopyIntr(Op, DAG, MipsISD::VEXTRACT_SEXT_ELT);
  case Intrinsic::mips_copy_s_d:
    if (Subtarget.hasMips64())
      // Lower directly into VEXTRACT_SEXT_ELT since i64 is legal on Mips64.
      return lowerMSACopyIntr(Op, DAG, MipsISD::VEXTRACT_SEXT_ELT);
    else {
      // Lower into the generic EXTRACT_VECTOR_ELT node and let the type
      // legalizer and EXTRACT_VECTOR_ELT lowering sort it out.
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1907-1921
```cpp
      return DAG.getNode(ISD::EXTRACT_VECTOR_ELT, SDLoc(Op),
                         Op->getValueType(0), Op->getOperand(1),
                         Op->getOperand(2));
    }
  case Intrinsic::mips_copy_u_b:
  case Intrinsic::mips_copy_u_h:
  case Intrinsic::mips_copy_u_w:
    return lowerMSACopyIntr(Op, DAG, MipsISD::VEXTRACT_ZEXT_ELT);
  case Intrinsic::mips_copy_u_d:
    if (Subtarget.hasMips64())
      // Lower directly into VEXTRACT_ZEXT_ELT since i64 is legal on Mips64.
      return lowerMSACopyIntr(Op, DAG, MipsISD::VEXTRACT_ZEXT_ELT);
    else {
      // Lower into the generic EXTRACT_VECTOR_ELT node and let the type
      // legalizer and EXTRACT_VECTOR_ELT lowering sort it out.
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1922-1936
```cpp
      // Note: When i64 is illegal, this results in copy_s.w instructions
      // instead of copy_u.w instructions. This makes no difference to the
      // behaviour since i64 is only illegal when the register file is 32-bit.
      return DAG.getNode(ISD::EXTRACT_VECTOR_ELT, SDLoc(Op),
                         Op->getValueType(0), Op->getOperand(1),
                         Op->getOperand(2));
    }
  case Intrinsic::mips_div_s_b:
  case Intrinsic::mips_div_s_h:
  case Intrinsic::mips_div_s_w:
  case Intrinsic::mips_div_s_d:
    return DAG.getNode(ISD::SDIV, DL, Op->getValueType(0), Op->getOperand(1),
                       Op->getOperand(2));
  case Intrinsic::mips_div_u_b:
  case Intrinsic::mips_div_u_h:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1937-1951
```cpp
  case Intrinsic::mips_div_u_w:
  case Intrinsic::mips_div_u_d:
    return DAG.getNode(ISD::UDIV, DL, Op->getValueType(0), Op->getOperand(1),
                       Op->getOperand(2));
  case Intrinsic::mips_fadd_w:
  case Intrinsic::mips_fadd_d:
    // TODO: If intrinsics have fast-math-flags, propagate them.
    return DAG.getNode(ISD::FADD, DL, Op->getValueType(0), Op->getOperand(1),
                       Op->getOperand(2));
  // Don't lower mips_fcaf_[wd] since LLVM folds SETFALSE condcodes away
  case Intrinsic::mips_fceq_w:
  case Intrinsic::mips_fceq_d:
    return DAG.getSetCC(DL, Op->getValueType(0), Op->getOperand(1),
                        Op->getOperand(2), ISD::SETOEQ);
  case Intrinsic::mips_fcle_w:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1952-1966
```cpp
  case Intrinsic::mips_fcle_d:
    return DAG.getSetCC(DL, Op->getValueType(0), Op->getOperand(1),
                        Op->getOperand(2), ISD::SETOLE);
  case Intrinsic::mips_fclt_w:
  case Intrinsic::mips_fclt_d:
    return DAG.getSetCC(DL, Op->getValueType(0), Op->getOperand(1),
                        Op->getOperand(2), ISD::SETOLT);
  case Intrinsic::mips_fcne_w:
  case Intrinsic::mips_fcne_d:
    return DAG.getSetCC(DL, Op->getValueType(0), Op->getOperand(1),
                        Op->getOperand(2), ISD::SETONE);
  case Intrinsic::mips_fcor_w:
  case Intrinsic::mips_fcor_d:
    return DAG.getSetCC(DL, Op->getValueType(0), Op->getOperand(1),
                        Op->getOperand(2), ISD::SETO);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1967-1981
```cpp
  case Intrinsic::mips_fcueq_w:
  case Intrinsic::mips_fcueq_d:
    return DAG.getSetCC(DL, Op->getValueType(0), Op->getOperand(1),
                        Op->getOperand(2), ISD::SETUEQ);
  case Intrinsic::mips_fcule_w:
  case Intrinsic::mips_fcule_d:
    return DAG.getSetCC(DL, Op->getValueType(0), Op->getOperand(1),
                        Op->getOperand(2), ISD::SETULE);
  case Intrinsic::mips_fcult_w:
  case Intrinsic::mips_fcult_d:
    return DAG.getSetCC(DL, Op->getValueType(0), Op->getOperand(1),
                        Op->getOperand(2), ISD::SETULT);
  case Intrinsic::mips_fcun_w:
  case Intrinsic::mips_fcun_d:
    return DAG.getSetCC(DL, Op->getValueType(0), Op->getOperand(1),
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1982-1996
```cpp
                        Op->getOperand(2), ISD::SETUO);
  case Intrinsic::mips_fcune_w:
  case Intrinsic::mips_fcune_d:
    return DAG.getSetCC(DL, Op->getValueType(0), Op->getOperand(1),
                        Op->getOperand(2), ISD::SETUNE);
  case Intrinsic::mips_fdiv_w:
  case Intrinsic::mips_fdiv_d:
    // TODO: If intrinsics have fast-math-flags, propagate them.
    return DAG.getNode(ISD::FDIV, DL, Op->getValueType(0), Op->getOperand(1),
                       Op->getOperand(2));
  case Intrinsic::mips_ffint_u_w:
  case Intrinsic::mips_ffint_u_d:
    return DAG.getNode(ISD::UINT_TO_FP, DL, Op->getValueType(0),
                       Op->getOperand(1));
  case Intrinsic::mips_ffint_s_w:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1997-2006
```cpp
  case Intrinsic::mips_ffint_s_d:
    return DAG.getNode(ISD::SINT_TO_FP, DL, Op->getValueType(0),
                       Op->getOperand(1));
  case Intrinsic::mips_fill_b:
  case Intrinsic::mips_fill_h:
  case Intrinsic::mips_fill_w:
  case Intrinsic::mips_fill_d: {
    EVT ResTy = Op->getValueType(0);
    SmallVector<SDValue, 16> Ops(ResTy.getVectorNumElements(),
                                 Op->getOperand(1));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2008-2022
```cpp
    // If ResTy is v2i64 then the type legalizer will break this node down into
    // an equivalent v4i32.
    return DAG.getBuildVector(ResTy, DL, Ops);
  }
  case Intrinsic::mips_fexp2_w:
  case Intrinsic::mips_fexp2_d: {
    // TODO: If intrinsics have fast-math-flags, propagate them.
    EVT ResTy = Op->getValueType(0);
    return DAG.getNode(
        ISD::FMUL, SDLoc(Op), ResTy, Op->getOperand(1),
        DAG.getNode(ISD::FEXP2, SDLoc(Op), ResTy, Op->getOperand(2)));
  }
  case Intrinsic::mips_flog2_w:
  case Intrinsic::mips_flog2_d:
    return DAG.getNode(ISD::FLOG2, DL, Op->getValueType(0), Op->getOperand(1));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2023-2037
```cpp
  case Intrinsic::mips_fmadd_w:
  case Intrinsic::mips_fmadd_d:
    return DAG.getNode(ISD::FMA, SDLoc(Op), Op->getValueType(0),
                       Op->getOperand(1), Op->getOperand(2), Op->getOperand(3));
  case Intrinsic::mips_fmul_w:
  case Intrinsic::mips_fmul_d:
    // TODO: If intrinsics have fast-math-flags, propagate them.
    return DAG.getNode(ISD::FMUL, DL, Op->getValueType(0), Op->getOperand(1),
                       Op->getOperand(2));
  case Intrinsic::mips_fmsub_w:
  case Intrinsic::mips_fmsub_d: {
    // TODO: If intrinsics have fast-math-flags, propagate them.
    return DAG.getNode(MipsISD::FMS, SDLoc(Op), Op->getValueType(0),
                       Op->getOperand(1), Op->getOperand(2), Op->getOperand(3));
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2038-2052
```cpp
  case Intrinsic::mips_frint_w:
  case Intrinsic::mips_frint_d:
    return DAG.getNode(ISD::FRINT, DL, Op->getValueType(0), Op->getOperand(1));
  case Intrinsic::mips_fsqrt_w:
  case Intrinsic::mips_fsqrt_d:
    return DAG.getNode(ISD::FSQRT, DL, Op->getValueType(0), Op->getOperand(1));
  case Intrinsic::mips_fsub_w:
  case Intrinsic::mips_fsub_d:
    // TODO: If intrinsics have fast-math-flags, propagate them.
    return DAG.getNode(ISD::FSUB, DL, Op->getValueType(0), Op->getOperand(1),
                       Op->getOperand(2));
  case Intrinsic::mips_ftrunc_u_w:
  case Intrinsic::mips_ftrunc_u_d:
    return DAG.getNode(ISD::FP_TO_UINT, DL, Op->getValueType(0),
                       Op->getOperand(1));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2053-2067
```cpp
  case Intrinsic::mips_ftrunc_s_w:
  case Intrinsic::mips_ftrunc_s_d:
    return DAG.getNode(ISD::FP_TO_SINT, DL, Op->getValueType(0),
                       Op->getOperand(1));
  case Intrinsic::mips_ilvev_b:
  case Intrinsic::mips_ilvev_h:
  case Intrinsic::mips_ilvev_w:
  case Intrinsic::mips_ilvev_d:
    return DAG.getNode(MipsISD::ILVEV, DL, Op->getValueType(0),
                       Op->getOperand(1), Op->getOperand(2));
  case Intrinsic::mips_ilvl_b:
  case Intrinsic::mips_ilvl_h:
  case Intrinsic::mips_ilvl_w:
  case Intrinsic::mips_ilvl_d:
    return DAG.getNode(MipsISD::ILVL, DL, Op->getValueType(0),
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2068-2082
```cpp
                       Op->getOperand(1), Op->getOperand(2));
  case Intrinsic::mips_ilvod_b:
  case Intrinsic::mips_ilvod_h:
  case Intrinsic::mips_ilvod_w:
  case Intrinsic::mips_ilvod_d:
    return DAG.getNode(MipsISD::ILVOD, DL, Op->getValueType(0),
                       Op->getOperand(1), Op->getOperand(2));
  case Intrinsic::mips_ilvr_b:
  case Intrinsic::mips_ilvr_h:
  case Intrinsic::mips_ilvr_w:
  case Intrinsic::mips_ilvr_d:
    return DAG.getNode(MipsISD::ILVR, DL, Op->getValueType(0),
                       Op->getOperand(1), Op->getOperand(2));
  case Intrinsic::mips_insert_b:
  case Intrinsic::mips_insert_h:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2083-2097
```cpp
  case Intrinsic::mips_insert_w:
  case Intrinsic::mips_insert_d:
    return DAG.getNode(ISD::INSERT_VECTOR_ELT, SDLoc(Op), Op->getValueType(0),
                       Op->getOperand(1), Op->getOperand(3), Op->getOperand(2));
  case Intrinsic::mips_insve_b:
  case Intrinsic::mips_insve_h:
  case Intrinsic::mips_insve_w:
  case Intrinsic::mips_insve_d: {
    // Report an error for out of range values.
    int64_t Max;
    switch (Intrinsic) {
    case Intrinsic::mips_insve_b: Max = 15; break;
    case Intrinsic::mips_insve_h: Max = 7; break;
    case Intrinsic::mips_insve_w: Max = 3; break;
    case Intrinsic::mips_insve_d: Max = 1; break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2098-2112
```cpp
    default: llvm_unreachable("Unmatched intrinsic");
    }
    int64_t Value = cast<ConstantSDNode>(Op->getOperand(2))->getSExtValue();
    if (Value < 0 || Value > Max)
      report_fatal_error("Immediate out of range");
    return DAG.getNode(MipsISD::INSVE, DL, Op->getValueType(0),
                       Op->getOperand(1), Op->getOperand(2), Op->getOperand(3),
                       DAG.getConstant(0, DL, MVT::i32));
    }
  case Intrinsic::mips_ldi_b:
  case Intrinsic::mips_ldi_h:
  case Intrinsic::mips_ldi_w:
  case Intrinsic::mips_ldi_d:
    return lowerMSASplatImm(Op, 1, DAG, true);
  case Intrinsic::mips_lsa:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2113-2127
```cpp
  case Intrinsic::mips_dlsa: {
    EVT ResTy = Op->getValueType(0);
    return DAG.getNode(ISD::ADD, SDLoc(Op), ResTy, Op->getOperand(1),
                       DAG.getNode(ISD::SHL, SDLoc(Op), ResTy,
                                   Op->getOperand(2), Op->getOperand(3)));
  }
  case Intrinsic::mips_maddv_b:
  case Intrinsic::mips_maddv_h:
  case Intrinsic::mips_maddv_w:
  case Intrinsic::mips_maddv_d: {
    EVT ResTy = Op->getValueType(0);
    return DAG.getNode(ISD::ADD, SDLoc(Op), ResTy, Op->getOperand(1),
                       DAG.getNode(ISD::MUL, SDLoc(Op), ResTy,
                                   Op->getOperand(2), Op->getOperand(3)));
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2128-2142
```cpp
  case Intrinsic::mips_max_s_b:
  case Intrinsic::mips_max_s_h:
  case Intrinsic::mips_max_s_w:
  case Intrinsic::mips_max_s_d:
    return DAG.getNode(ISD::SMAX, DL, Op->getValueType(0),
                       Op->getOperand(1), Op->getOperand(2));
  case Intrinsic::mips_max_u_b:
  case Intrinsic::mips_max_u_h:
  case Intrinsic::mips_max_u_w:
  case Intrinsic::mips_max_u_d:
    return DAG.getNode(ISD::UMAX, DL, Op->getValueType(0),
                       Op->getOperand(1), Op->getOperand(2));
  case Intrinsic::mips_maxi_s_b:
  case Intrinsic::mips_maxi_s_h:
  case Intrinsic::mips_maxi_s_w:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2143-2157
```cpp
  case Intrinsic::mips_maxi_s_d:
    return DAG.getNode(ISD::SMAX, DL, Op->getValueType(0),
                       Op->getOperand(1), lowerMSASplatImm(Op, 2, DAG, true));
  case Intrinsic::mips_maxi_u_b:
  case Intrinsic::mips_maxi_u_h:
  case Intrinsic::mips_maxi_u_w:
  case Intrinsic::mips_maxi_u_d:
    return DAG.getNode(ISD::UMAX, DL, Op->getValueType(0),
                       Op->getOperand(1), lowerMSASplatImm(Op, 2, DAG));
  case Intrinsic::mips_min_s_b:
  case Intrinsic::mips_min_s_h:
  case Intrinsic::mips_min_s_w:
  case Intrinsic::mips_min_s_d:
    return DAG.getNode(ISD::SMIN, DL, Op->getValueType(0),
                       Op->getOperand(1), Op->getOperand(2));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2158-2172
```cpp
  case Intrinsic::mips_min_u_b:
  case Intrinsic::mips_min_u_h:
  case Intrinsic::mips_min_u_w:
  case Intrinsic::mips_min_u_d:
    return DAG.getNode(ISD::UMIN, DL, Op->getValueType(0),
                       Op->getOperand(1), Op->getOperand(2));
  case Intrinsic::mips_mini_s_b:
  case Intrinsic::mips_mini_s_h:
  case Intrinsic::mips_mini_s_w:
  case Intrinsic::mips_mini_s_d:
    return DAG.getNode(ISD::SMIN, DL, Op->getValueType(0),
                       Op->getOperand(1), lowerMSASplatImm(Op, 2, DAG, true));
  case Intrinsic::mips_mini_u_b:
  case Intrinsic::mips_mini_u_h:
  case Intrinsic::mips_mini_u_w:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2173-2187
```cpp
  case Intrinsic::mips_mini_u_d:
    return DAG.getNode(ISD::UMIN, DL, Op->getValueType(0),
                       Op->getOperand(1), lowerMSASplatImm(Op, 2, DAG));
  case Intrinsic::mips_mod_s_b:
  case Intrinsic::mips_mod_s_h:
  case Intrinsic::mips_mod_s_w:
  case Intrinsic::mips_mod_s_d:
    return DAG.getNode(ISD::SREM, DL, Op->getValueType(0), Op->getOperand(1),
                       Op->getOperand(2));
  case Intrinsic::mips_mod_u_b:
  case Intrinsic::mips_mod_u_h:
  case Intrinsic::mips_mod_u_w:
  case Intrinsic::mips_mod_u_d:
    return DAG.getNode(ISD::UREM, DL, Op->getValueType(0), Op->getOperand(1),
                       Op->getOperand(2));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2188-2202
```cpp
  case Intrinsic::mips_mulv_b:
  case Intrinsic::mips_mulv_h:
  case Intrinsic::mips_mulv_w:
  case Intrinsic::mips_mulv_d:
    return DAG.getNode(ISD::MUL, DL, Op->getValueType(0), Op->getOperand(1),
                       Op->getOperand(2));
  case Intrinsic::mips_msubv_b:
  case Intrinsic::mips_msubv_h:
  case Intrinsic::mips_msubv_w:
  case Intrinsic::mips_msubv_d: {
    EVT ResTy = Op->getValueType(0);
    return DAG.getNode(ISD::SUB, SDLoc(Op), ResTy, Op->getOperand(1),
                       DAG.getNode(ISD::MUL, SDLoc(Op), ResTy,
                                   Op->getOperand(2), Op->getOperand(3)));
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2203-2217
```cpp
  case Intrinsic::mips_nlzc_b:
  case Intrinsic::mips_nlzc_h:
  case Intrinsic::mips_nlzc_w:
  case Intrinsic::mips_nlzc_d:
    return DAG.getNode(ISD::CTLZ, DL, Op->getValueType(0), Op->getOperand(1));
  case Intrinsic::mips_nor_v: {
    SDValue Res = DAG.getNode(ISD::OR, DL, Op->getValueType(0),
                              Op->getOperand(1), Op->getOperand(2));
    return DAG.getNOT(DL, Res, Res->getValueType(0));
  }
  case Intrinsic::mips_nori_b: {
    SDValue Res =  DAG.getNode(ISD::OR, DL, Op->getValueType(0),
                               Op->getOperand(1),
                               lowerMSASplatImm(Op, 2, DAG));
    return DAG.getNOT(DL, Res, Res->getValueType(0));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2218-2232
```cpp
  }
  case Intrinsic::mips_or_v:
    return DAG.getNode(ISD::OR, DL, Op->getValueType(0), Op->getOperand(1),
                       Op->getOperand(2));
  case Intrinsic::mips_ori_b:
    return DAG.getNode(ISD::OR, DL, Op->getValueType(0),
                       Op->getOperand(1), lowerMSASplatImm(Op, 2, DAG));
  case Intrinsic::mips_pckev_b:
  case Intrinsic::mips_pckev_h:
  case Intrinsic::mips_pckev_w:
  case Intrinsic::mips_pckev_d:
    return DAG.getNode(MipsISD::PCKEV, DL, Op->getValueType(0),
                       Op->getOperand(1), Op->getOperand(2));
  case Intrinsic::mips_pckod_b:
  case Intrinsic::mips_pckod_h:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2233-2247
```cpp
  case Intrinsic::mips_pckod_w:
  case Intrinsic::mips_pckod_d:
    return DAG.getNode(MipsISD::PCKOD, DL, Op->getValueType(0),
                       Op->getOperand(1), Op->getOperand(2));
  case Intrinsic::mips_pcnt_b:
  case Intrinsic::mips_pcnt_h:
  case Intrinsic::mips_pcnt_w:
  case Intrinsic::mips_pcnt_d:
    return DAG.getNode(ISD::CTPOP, DL, Op->getValueType(0), Op->getOperand(1));
  case Intrinsic::mips_sat_s_b:
  case Intrinsic::mips_sat_s_h:
  case Intrinsic::mips_sat_s_w:
  case Intrinsic::mips_sat_s_d:
  case Intrinsic::mips_sat_u_b:
  case Intrinsic::mips_sat_u_h:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2248-2262
```cpp
  case Intrinsic::mips_sat_u_w:
  case Intrinsic::mips_sat_u_d: {
    // Report an error for out of range values.
    int64_t Max;
    switch (Intrinsic) {
    case Intrinsic::mips_sat_s_b:
    case Intrinsic::mips_sat_u_b: Max = 7;  break;
    case Intrinsic::mips_sat_s_h:
    case Intrinsic::mips_sat_u_h: Max = 15; break;
    case Intrinsic::mips_sat_s_w:
    case Intrinsic::mips_sat_u_w: Max = 31; break;
    case Intrinsic::mips_sat_s_d:
    case Intrinsic::mips_sat_u_d: Max = 63; break;
    default: llvm_unreachable("Unmatched intrinsic");
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2263-2277
```cpp
    int64_t Value = cast<ConstantSDNode>(Op->getOperand(2))->getSExtValue();
    if (Value < 0 || Value > Max)
      report_fatal_error("Immediate out of range");
    return SDValue();
  }
  case Intrinsic::mips_shf_b:
  case Intrinsic::mips_shf_h:
  case Intrinsic::mips_shf_w: {
    int64_t Value = cast<ConstantSDNode>(Op->getOperand(2))->getSExtValue();
    if (Value < 0 || Value > 255)
      report_fatal_error("Immediate out of range");
    return DAG.getNode(MipsISD::SHF, DL, Op->getValueType(0),
                       Op->getOperand(2), Op->getOperand(1));
  }
  case Intrinsic::mips_sldi_b:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2278-2292
```cpp
  case Intrinsic::mips_sldi_h:
  case Intrinsic::mips_sldi_w:
  case Intrinsic::mips_sldi_d: {
    // Report an error for out of range values.
    int64_t Max;
    switch (Intrinsic) {
    case Intrinsic::mips_sldi_b: Max = 15; break;
    case Intrinsic::mips_sldi_h: Max = 7; break;
    case Intrinsic::mips_sldi_w: Max = 3; break;
    case Intrinsic::mips_sldi_d: Max = 1; break;
    default: llvm_unreachable("Unmatched intrinsic");
    }
    int64_t Value = cast<ConstantSDNode>(Op->getOperand(3))->getSExtValue();
    if (Value < 0 || Value > Max)
      report_fatal_error("Immediate out of range");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2293-2307
```cpp
    return SDValue();
  }
  case Intrinsic::mips_sll_b:
  case Intrinsic::mips_sll_h:
  case Intrinsic::mips_sll_w:
  case Intrinsic::mips_sll_d:
    return DAG.getNode(ISD::SHL, DL, Op->getValueType(0), Op->getOperand(1),
                       truncateVecElts(Op, DAG));
  case Intrinsic::mips_slli_b:
  case Intrinsic::mips_slli_h:
  case Intrinsic::mips_slli_w:
  case Intrinsic::mips_slli_d:
    return DAG.getNode(ISD::SHL, DL, Op->getValueType(0),
                       Op->getOperand(1), lowerMSASplatImm(Op, 2, DAG));
  case Intrinsic::mips_splat_b:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2308-2322
```cpp
  case Intrinsic::mips_splat_h:
  case Intrinsic::mips_splat_w:
  case Intrinsic::mips_splat_d:
    // We can't lower via VECTOR_SHUFFLE because it requires constant shuffle
    // masks, nor can we lower via BUILD_VECTOR & EXTRACT_VECTOR_ELT because
    // EXTRACT_VECTOR_ELT can't extract i64's on MIPS32.
    // Instead we lower to MipsISD::VSHF and match from there.
    return DAG.getNode(MipsISD::VSHF, DL, Op->getValueType(0),
                       lowerMSASplatZExt(Op, 2, DAG), Op->getOperand(1),
                       Op->getOperand(1));
  case Intrinsic::mips_splati_b:
  case Intrinsic::mips_splati_h:
  case Intrinsic::mips_splati_w:
  case Intrinsic::mips_splati_d:
    return DAG.getNode(MipsISD::VSHF, DL, Op->getValueType(0),
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2323-2337
```cpp
                       lowerMSASplatImm(Op, 2, DAG), Op->getOperand(1),
                       Op->getOperand(1));
  case Intrinsic::mips_sra_b:
  case Intrinsic::mips_sra_h:
  case Intrinsic::mips_sra_w:
  case Intrinsic::mips_sra_d:
    return DAG.getNode(ISD::SRA, DL, Op->getValueType(0), Op->getOperand(1),
                       truncateVecElts(Op, DAG));
  case Intrinsic::mips_srai_b:
  case Intrinsic::mips_srai_h:
  case Intrinsic::mips_srai_w:
  case Intrinsic::mips_srai_d:
    return DAG.getNode(ISD::SRA, DL, Op->getValueType(0),
                       Op->getOperand(1), lowerMSASplatImm(Op, 2, DAG));
  case Intrinsic::mips_srari_b:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2338-2352
```cpp
  case Intrinsic::mips_srari_h:
  case Intrinsic::mips_srari_w:
  case Intrinsic::mips_srari_d: {
    // Report an error for out of range values.
    int64_t Max;
    switch (Intrinsic) {
    case Intrinsic::mips_srari_b: Max = 7; break;
    case Intrinsic::mips_srari_h: Max = 15; break;
    case Intrinsic::mips_srari_w: Max = 31; break;
    case Intrinsic::mips_srari_d: Max = 63; break;
    default: llvm_unreachable("Unmatched intrinsic");
    }
    int64_t Value = cast<ConstantSDNode>(Op->getOperand(2))->getSExtValue();
    if (Value < 0 || Value > Max)
      report_fatal_error("Immediate out of range");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2353-2367
```cpp
    return SDValue();
  }
  case Intrinsic::mips_srl_b:
  case Intrinsic::mips_srl_h:
  case Intrinsic::mips_srl_w:
  case Intrinsic::mips_srl_d:
    return DAG.getNode(ISD::SRL, DL, Op->getValueType(0), Op->getOperand(1),
                       truncateVecElts(Op, DAG));
  case Intrinsic::mips_srli_b:
  case Intrinsic::mips_srli_h:
  case Intrinsic::mips_srli_w:
  case Intrinsic::mips_srli_d:
    return DAG.getNode(ISD::SRL, DL, Op->getValueType(0),
                       Op->getOperand(1), lowerMSASplatImm(Op, 2, DAG));
  case Intrinsic::mips_srlri_b:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2368-2382
```cpp
  case Intrinsic::mips_srlri_h:
  case Intrinsic::mips_srlri_w:
  case Intrinsic::mips_srlri_d: {
    // Report an error for out of range values.
    int64_t Max;
    switch (Intrinsic) {
    case Intrinsic::mips_srlri_b: Max = 7; break;
    case Intrinsic::mips_srlri_h: Max = 15; break;
    case Intrinsic::mips_srlri_w: Max = 31; break;
    case Intrinsic::mips_srlri_d: Max = 63; break;
    default: llvm_unreachable("Unmatched intrinsic");
    }
    int64_t Value = cast<ConstantSDNode>(Op->getOperand(2))->getSExtValue();
    if (Value < 0 || Value > Max)
      report_fatal_error("Immediate out of range");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2383-2397
```cpp
    return SDValue();
  }
  case Intrinsic::mips_subv_b:
  case Intrinsic::mips_subv_h:
  case Intrinsic::mips_subv_w:
  case Intrinsic::mips_subv_d:
    return DAG.getNode(ISD::SUB, DL, Op->getValueType(0), Op->getOperand(1),
                       Op->getOperand(2));
  case Intrinsic::mips_subvi_b:
  case Intrinsic::mips_subvi_h:
  case Intrinsic::mips_subvi_w:
  case Intrinsic::mips_subvi_d:
    return DAG.getNode(ISD::SUB, DL, Op->getValueType(0),
                       Op->getOperand(1), lowerMSASplatImm(Op, 2, DAG));
  case Intrinsic::mips_vshf_b:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2398-2412
```cpp
  case Intrinsic::mips_vshf_h:
  case Intrinsic::mips_vshf_w:
  case Intrinsic::mips_vshf_d:
    return DAG.getNode(MipsISD::VSHF, DL, Op->getValueType(0),
                       Op->getOperand(1), Op->getOperand(2), Op->getOperand(3));
  case Intrinsic::mips_xor_v:
    return DAG.getNode(ISD::XOR, DL, Op->getValueType(0), Op->getOperand(1),
                       Op->getOperand(2));
  case Intrinsic::mips_xori_b:
    return DAG.getNode(ISD::XOR, DL, Op->getValueType(0),
                       Op->getOperand(1), lowerMSASplatImm(Op, 2, DAG));
  case Intrinsic::thread_pointer: {
    EVT PtrVT = getPointerTy(DAG.getDataLayout());
    return DAG.getNode(MipsISD::ThreadPointer, DL, PtrVT);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2413-2414
```cpp
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2416-2423
```cpp
static SDValue lowerMSALoadIntr(SDValue Op, SelectionDAG &DAG, unsigned Intr,
                                const MipsSubtarget &Subtarget) {
  SDLoc DL(Op);
  SDValue ChainIn = Op->getOperand(0);
  SDValue Address = Op->getOperand(2);
  SDValue Offset  = Op->getOperand(3);
  EVT ResTy = Op->getValueType(0);
  EVT PtrTy = Address->getValueType(0);
```
- EN: Implements `lowerMSALoadIntr`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `lowerMSALoadIntr`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2425-2429
```cpp
  // For N64 addresses have the underlying type MVT::i64. This intrinsic
  // however takes an i32 signed constant offset. The actual type of the
  // intrinsic is a scaled signed i10.
  if (Subtarget.isABI_N64())
    Offset = DAG.getNode(ISD::SIGN_EXTEND, DL, PtrTy, Offset);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2431-2434
```cpp
  Address = DAG.getNode(ISD::ADD, DL, PtrTy, Address, Offset);
  return DAG.getLoad(ResTy, DL, ChainIn, Address, MachinePointerInfo(),
                     Align(16));
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2436-2450
```cpp
SDValue MipsSETargetLowering::lowerINTRINSIC_W_CHAIN(SDValue Op,
                                                     SelectionDAG &DAG) const {
  unsigned Intr = Op->getConstantOperandVal(1);
  switch (Intr) {
  default:
    return SDValue();
  case Intrinsic::mips_extp:
    return lowerDSPIntr(Op, DAG, MipsISD::EXTP);
  case Intrinsic::mips_extpdp:
    return lowerDSPIntr(Op, DAG, MipsISD::EXTPDP);
  case Intrinsic::mips_extr_w:
    return lowerDSPIntr(Op, DAG, MipsISD::EXTR_W);
  case Intrinsic::mips_extr_r_w:
    return lowerDSPIntr(Op, DAG, MipsISD::EXTR_R_W);
  case Intrinsic::mips_extr_rs_w:
```
- EN: Implements `MipsSETargetLowering::lowerINTRINSIC_W_CHAIN`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSETargetLowering::lowerINTRINSIC_W_CHAIN`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2451-2465
```cpp
    return lowerDSPIntr(Op, DAG, MipsISD::EXTR_RS_W);
  case Intrinsic::mips_extr_s_h:
    return lowerDSPIntr(Op, DAG, MipsISD::EXTR_S_H);
  case Intrinsic::mips_mthlip:
    return lowerDSPIntr(Op, DAG, MipsISD::MTHLIP);
  case Intrinsic::mips_mulsaq_s_w_ph:
    return lowerDSPIntr(Op, DAG, MipsISD::MULSAQ_S_W_PH);
  case Intrinsic::mips_maq_s_w_phl:
    return lowerDSPIntr(Op, DAG, MipsISD::MAQ_S_W_PHL);
  case Intrinsic::mips_maq_s_w_phr:
    return lowerDSPIntr(Op, DAG, MipsISD::MAQ_S_W_PHR);
  case Intrinsic::mips_maq_sa_w_phl:
    return lowerDSPIntr(Op, DAG, MipsISD::MAQ_SA_W_PHL);
  case Intrinsic::mips_maq_sa_w_phr:
    return lowerDSPIntr(Op, DAG, MipsISD::MAQ_SA_W_PHR);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2466-2480
```cpp
  case Intrinsic::mips_dpaq_s_w_ph:
    return lowerDSPIntr(Op, DAG, MipsISD::DPAQ_S_W_PH);
  case Intrinsic::mips_dpsq_s_w_ph:
    return lowerDSPIntr(Op, DAG, MipsISD::DPSQ_S_W_PH);
  case Intrinsic::mips_dpaq_sa_l_w:
    return lowerDSPIntr(Op, DAG, MipsISD::DPAQ_SA_L_W);
  case Intrinsic::mips_dpsq_sa_l_w:
    return lowerDSPIntr(Op, DAG, MipsISD::DPSQ_SA_L_W);
  case Intrinsic::mips_dpaqx_s_w_ph:
    return lowerDSPIntr(Op, DAG, MipsISD::DPAQX_S_W_PH);
  case Intrinsic::mips_dpaqx_sa_w_ph:
    return lowerDSPIntr(Op, DAG, MipsISD::DPAQX_SA_W_PH);
  case Intrinsic::mips_dpsqx_s_w_ph:
    return lowerDSPIntr(Op, DAG, MipsISD::DPSQX_S_W_PH);
  case Intrinsic::mips_dpsqx_sa_w_ph:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2481-2488
```cpp
    return lowerDSPIntr(Op, DAG, MipsISD::DPSQX_SA_W_PH);
  case Intrinsic::mips_ld_b:
  case Intrinsic::mips_ld_h:
  case Intrinsic::mips_ld_w:
  case Intrinsic::mips_ld_d:
   return lowerMSALoadIntr(Op, DAG, Intr, Subtarget);
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2490-2497
```cpp
static SDValue lowerMSAStoreIntr(SDValue Op, SelectionDAG &DAG, unsigned Intr,
                                 const MipsSubtarget &Subtarget) {
  SDLoc DL(Op);
  SDValue ChainIn = Op->getOperand(0);
  SDValue Value   = Op->getOperand(2);
  SDValue Address = Op->getOperand(3);
  SDValue Offset  = Op->getOperand(4);
  EVT PtrTy = Address->getValueType(0);
```
- EN: Implements `lowerMSAStoreIntr`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `lowerMSAStoreIntr`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2499-2503
```cpp
  // For N64 addresses have the underlying type MVT::i64. This intrinsic
  // however takes an i32 signed constant offset. The actual type of the
  // intrinsic is a scaled signed i10.
  if (Subtarget.isABI_N64())
    Offset = DAG.getNode(ISD::SIGN_EXTEND, DL, PtrTy, Offset);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2505-2505
```cpp
  Address = DAG.getNode(ISD::ADD, DL, PtrTy, Address, Offset);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2507-2509
```cpp
  return DAG.getStore(ChainIn, DL, Value, Address, MachinePointerInfo(),
                      Align(16));
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2511-2523
```cpp
SDValue MipsSETargetLowering::lowerINTRINSIC_VOID(SDValue Op,
                                                  SelectionDAG &DAG) const {
  unsigned Intr = Op->getConstantOperandVal(1);
  switch (Intr) {
  default:
    return SDValue();
  case Intrinsic::mips_st_b:
  case Intrinsic::mips_st_h:
  case Intrinsic::mips_st_w:
  case Intrinsic::mips_st_d:
    return lowerMSAStoreIntr(Op, DAG, Intr, Subtarget);
  }
}
```
- EN: Implements `MipsSETargetLowering::lowerINTRINSIC_VOID`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSETargetLowering::lowerINTRINSIC_VOID`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2525-2537
```cpp
// Lower ISD::EXTRACT_VECTOR_ELT into MipsISD::VEXTRACT_SEXT_ELT.
//
// The non-value bits resulting from ISD::EXTRACT_VECTOR_ELT are undefined. We
// choose to sign-extend but we could have equally chosen zero-extend. The
// DAGCombiner will fold any sign/zero extension of the ISD::EXTRACT_VECTOR_ELT
// result into this node later (possibly changing it to a zero-extend in the
// process).
SDValue MipsSETargetLowering::
lowerEXTRACT_VECTOR_ELT(SDValue Op, SelectionDAG &DAG) const {
  SDLoc DL(Op);
  EVT ResTy = Op->getValueType(0);
  SDValue Op0 = Op->getOperand(0);
  EVT VecTy = Op0->getValueType(0);
```
- EN: Implements `lowerEXTRACT_VECTOR_ELT`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `lowerEXTRACT_VECTOR_ELT`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2539-2540
```cpp
  if (!VecTy.is128BitVector())
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2542-2547
```cpp
  if (ResTy.isInteger()) {
    SDValue Op1 = Op->getOperand(1);
    EVT EltTy = VecTy.getVectorElementType();
    return DAG.getNode(MipsISD::VEXTRACT_SEXT_ELT, DL, ResTy, Op0, Op1,
                       DAG.getValueType(EltTy));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2549-2550
```cpp
  return Op;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2552-2560
```cpp
static bool isConstantOrUndef(const SDValue Op) {
  if (Op->isUndef())
    return true;
  if (isa<ConstantSDNode>(Op))
    return true;
  if (isa<ConstantFPSDNode>(Op))
    return true;
  return false;
}
```
- EN: Implements `isConstantOrUndef`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isConstantOrUndef`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2562-2567
```cpp
static bool isConstantOrUndefBUILD_VECTOR(const BuildVectorSDNode *Op) {
  for (unsigned i = 0; i < Op->getNumOperands(); ++i)
    if (isConstantOrUndef(Op->getOperand(i)))
      return true;
  return false;
}
```
- EN: Implements `isConstantOrUndefBUILD_VECTOR`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isConstantOrUndefBUILD_VECTOR`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2569-2583
```cpp
// Lowers ISD::BUILD_VECTOR into appropriate SelectionDAG nodes for the
// backend.
//
// Lowers according to the following rules:
// - Constant splats are legal as-is as long as the SplatBitSize is a power of
//   2 less than or equal to 64 and the value fits into a signed 10-bit
//   immediate
// - Constant splats are lowered to bitconverted BUILD_VECTORs if SplatBitSize
//   is a power of 2 less than or equal to 64 and the value does not fit into a
//   signed 10-bit immediate
// - Non-constant splats are legal as-is.
// - Non-constant non-splats are lowered to sequences of INSERT_VECTOR_ELT.
// - All others are illegal and must be expanded.
SDValue MipsSETargetLowering::lowerBUILD_VECTOR(SDValue Op,
                                                SelectionDAG &DAG) const {
```
- EN: Implements `MipsSETargetLowering::lowerBUILD_VECTOR`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSETargetLowering::lowerBUILD_VECTOR`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2584-2589
```cpp
  BuildVectorSDNode *Node = cast<BuildVectorSDNode>(Op);
  EVT ResTy = Op->getValueType(0);
  SDLoc DL(Op);
  APInt SplatValue, SplatUndef;
  unsigned SplatBitSize;
  bool HasAnyUndefs;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2591-2592
```cpp
  if (!Subtarget.hasMSA() || !ResTy.is128BitVector())
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2594-2600
```cpp
  if (Node->isConstantSplat(SplatValue, SplatUndef, SplatBitSize,
                            HasAnyUndefs, 8,
                            !Subtarget.isLittle()) && SplatBitSize <= 64) {
    // We can only cope with 8, 16, 32, or 64-bit elements
    if (SplatBitSize != 8 && SplatBitSize != 16 && SplatBitSize != 32 &&
        SplatBitSize != 64)
      return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2602-2606
```cpp
    // If the value isn't an integer type we will have to bitcast
    // from an integer type first. Also, if there are any undefs, we must
    // lower them to defined values first.
    if (ResTy.isInteger() && !HasAnyUndefs)
      return Op;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2608-2608
```cpp
    EVT ViaVecTy;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2610-2624
```cpp
    switch (SplatBitSize) {
    default:
      return SDValue();
    case 8:
      ViaVecTy = MVT::v16i8;
      break;
    case 16:
      ViaVecTy = MVT::v8i16;
      break;
    case 32:
      ViaVecTy = MVT::v4i32;
      break;
    case 64:
      // There's no fill.d to fall back on for 64-bit values
      return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2625-2625
```cpp
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2627-2628
```cpp
    // SelectionDAG::getConstant will promote SplatValue appropriately.
    SDValue Result = DAG.getConstant(SplatValue, DL, ViaVecTy);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2630-2632
```cpp
    // Bitcast to the type we originally wanted
    if (ViaVecTy != ResTy)
      Result = DAG.getNode(ISD::BITCAST, SDLoc(Node), ResTy, Result);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2634-2641
```cpp
    return Result;
  } else if (DAG.isSplatValue(Op, /* AllowUndefs */ false))
    return Op;
  else if (!isConstantOrUndefBUILD_VECTOR(Node)) {
    // Use INSERT_VECTOR_ELT operations rather than expand to stores.
    // The resulting code is the same length as the expansion, but it doesn't
    // use memory operations
    EVT ResTy = Node->getValueType(0);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2643-2643
```cpp
    assert(ResTy.isVector());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2645-2653
```cpp
    unsigned NumElts = ResTy.getVectorNumElements();
    SDValue Vector = DAG.getUNDEF(ResTy);
    for (unsigned i = 0; i < NumElts; ++i) {
      Vector = DAG.getNode(ISD::INSERT_VECTOR_ELT, DL, ResTy, Vector,
                           Node->getOperand(i),
                           DAG.getConstant(i, DL, MVT::i32));
    }
    return Vector;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2655-2656
```cpp
  return SDValue();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2658-2672
```cpp
// Lower VECTOR_SHUFFLE into SHF (if possible).
//
// SHF splits the vector into blocks of four elements, then shuffles these
// elements according to a <4 x i2> constant (encoded as an integer immediate).
//
// It is therefore possible to lower into SHF when the mask takes the form:
//   <a, b, c, d, a+4, b+4, c+4, d+4, a+8, b+8, c+8, d+8, ...>
// When undef's appear they are treated as if they were whatever value is
// necessary in order to fit the above forms.
//
// For example:
//   %2 = shufflevector <8 x i16> %0, <8 x i16> undef,
//                      <8 x i32> <i32 3, i32 2, i32 1, i32 0,
//                                 i32 7, i32 6, i32 5, i32 4>
// is lowered to:
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 2673-2679
```cpp
//   (SHF_H $w0, $w1, 27)
// where the 27 comes from:
//   3 + (2 << 2) + (1 << 4) + (0 << 6)
static SDValue lowerVECTOR_SHUFFLE_SHF(SDValue Op, EVT ResTy,
                                       SmallVector<int, 16> Indices,
                                       SelectionDAG &DAG) {
  int SHFIndices[4] = { -1, -1, -1, -1 };
```
- EN: Implements `lowerVECTOR_SHUFFLE_SHF`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `lowerVECTOR_SHUFFLE_SHF`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2681-2682
```cpp
  if (Indices.size() < 4)
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2684-2686
```cpp
  for (unsigned i = 0; i < 4; ++i) {
    for (unsigned j = i; j < Indices.size(); j += 4) {
      int Idx = Indices[j];
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2688-2694
```cpp
      // Convert from vector index to 4-element subvector index
      // If an index refers to an element outside of the subvector then give up
      if (Idx != -1) {
        Idx -= 4 * (j / 4);
        if (Idx < 0 || Idx >= 4)
          return SDValue();
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2696-2699
```cpp
      // If the mask has an undef, replace it with the current index.
      // Note that it might still be undef if the current index is also undef
      if (SHFIndices[i] == -1)
        SHFIndices[i] = Idx;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2701-2706
```cpp
      // Check that non-undef values are the same as in the mask. If they
      // aren't then give up
      if (!(Idx == -1 || Idx == SHFIndices[i]))
        return SDValue();
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2708-2711
```cpp
  // Calculate the immediate. Replace any remaining undefs with zero
  APInt Imm(32, 0);
  for (int i = 3; i >= 0; --i) {
    int Idx = SHFIndices[i];
```
- EN: Implements `Imm`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Imm`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2713-2714
```cpp
    if (Idx == -1)
      Idx = 0;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2716-2718
```cpp
    Imm <<= 2;
    Imm |= Idx & 0x3;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2720-2724
```cpp
  SDLoc DL(Op);
  return DAG.getNode(MipsISD::SHF, DL, ResTy,
                     DAG.getTargetConstant(Imm, DL, MVT::i32),
                     Op->getOperand(0));
}
```
- EN: Declares `DL`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `DL`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2726-2734
```cpp
/// Determine whether a range fits a regular pattern of values.
/// This function accounts for the possibility of jumping over the End iterator.
template <typename ValType>
static bool
fitsRegularPattern(typename SmallVectorImpl<ValType>::const_iterator Begin,
                   unsigned CheckStride,
                   typename SmallVectorImpl<ValType>::const_iterator End,
                   ValType ExpectedIndex, unsigned ExpectedIndexStride) {
  auto &I = Begin;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2736-2739
```cpp
  while (I != End) {
    if (*I != -1 && *I != ExpectedIndex)
      return false;
    ExpectedIndex += ExpectedIndexStride;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2741-2747
```cpp
    // Incrementing past End is undefined behaviour so we must increment one
    // step at a time and check for End at each step.
    for (unsigned n = 0; n < CheckStride && I != End; ++n, ++I)
      ; // Empty loop body.
  }
  return true;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2749-2760
```cpp
// Determine whether VECTOR_SHUFFLE is a SPLATI.
//
// It is a SPLATI when the mask is:
//   <x, x, x, ...>
// where x is any valid index.
//
// When undef's appear in the mask they are treated as if they were whatever
// value is necessary in order to fit the above form.
static bool isVECTOR_SHUFFLE_SPLATI(SDValue Op, EVT ResTy,
                                    SmallVector<int, 16> Indices,
                                    SelectionDAG &DAG) {
  assert((Indices.size() % 2) == 0);
```
- EN: Implements `isVECTOR_SHUFFLE_SPLATI`, a query/helper routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isVECTOR_SHUFFLE_SPLATI`，它是一个围绕SelectionDAG 降级展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2762-2768
```cpp
  int SplatIndex = -1;
  for (const auto &V : Indices) {
    if (V != -1) {
      SplatIndex = V;
      break;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2770-2772
```cpp
  return fitsRegularPattern<int>(Indices.begin(), 1, Indices.end(), SplatIndex,
                                 0);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2774-2788
```cpp
// Lower VECTOR_SHUFFLE into ILVEV (if possible).
//
// ILVEV interleaves the even elements from each vector.
//
// It is possible to lower into ILVEV when the mask consists of two of the
// following forms interleaved:
//   <0, 2, 4, ...>
//   <n, n+2, n+4, ...>
// where n is the number of elements in the vector.
// For example:
//   <0, 0, 2, 2, 4, 4, ...>
//   <0, n, 2, n+2, 4, n+4, ...>
//
// When undef's appear in the mask they are treated as if they were whatever
// value is necessary in order to fit the above forms.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 2789-2792
```cpp
static SDValue lowerVECTOR_SHUFFLE_ILVEV(SDValue Op, EVT ResTy,
                                         SmallVector<int, 16> Indices,
                                         SelectionDAG &DAG) {
  assert((Indices.size() % 2) == 0);
```
- EN: Implements `lowerVECTOR_SHUFFLE_ILVEV`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `lowerVECTOR_SHUFFLE_ILVEV`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2794-2797
```cpp
  SDValue Wt;
  SDValue Ws;
  const auto &Begin = Indices.begin();
  const auto &End = Indices.end();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2799-2806
```cpp
  // Check even elements are taken from the even elements of one half or the
  // other and pick an operand accordingly.
  if (fitsRegularPattern<int>(Begin, 2, End, 0, 2))
    Wt = Op->getOperand(0);
  else if (fitsRegularPattern<int>(Begin, 2, End, Indices.size(), 2))
    Wt = Op->getOperand(1);
  else
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2808-2815
```cpp
  // Check odd elements are taken from the even elements of one half or the
  // other and pick an operand accordingly.
  if (fitsRegularPattern<int>(Begin + 1, 2, End, 0, 2))
    Ws = Op->getOperand(0);
  else if (fitsRegularPattern<int>(Begin + 1, 2, End, Indices.size(), 2))
    Ws = Op->getOperand(1);
  else
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2817-2818
```cpp
  return DAG.getNode(MipsISD::ILVEV, SDLoc(Op), ResTy, Ws, Wt);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2820-2834
```cpp
// Lower VECTOR_SHUFFLE into ILVOD (if possible).
//
// ILVOD interleaves the odd elements from each vector.
//
// It is possible to lower into ILVOD when the mask consists of two of the
// following forms interleaved:
//   <1, 3, 5, ...>
//   <n+1, n+3, n+5, ...>
// where n is the number of elements in the vector.
// For example:
//   <1, 1, 3, 3, 5, 5, ...>
//   <1, n+1, 3, n+3, 5, n+5, ...>
//
// When undef's appear in the mask they are treated as if they were whatever
// value is necessary in order to fit the above forms.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 2835-2838
```cpp
static SDValue lowerVECTOR_SHUFFLE_ILVOD(SDValue Op, EVT ResTy,
                                         SmallVector<int, 16> Indices,
                                         SelectionDAG &DAG) {
  assert((Indices.size() % 2) == 0);
```
- EN: Implements `lowerVECTOR_SHUFFLE_ILVOD`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `lowerVECTOR_SHUFFLE_ILVOD`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2840-2843
```cpp
  SDValue Wt;
  SDValue Ws;
  const auto &Begin = Indices.begin();
  const auto &End = Indices.end();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2845-2852
```cpp
  // Check even elements are taken from the odd elements of one half or the
  // other and pick an operand accordingly.
  if (fitsRegularPattern<int>(Begin, 2, End, 1, 2))
    Wt = Op->getOperand(0);
  else if (fitsRegularPattern<int>(Begin, 2, End, Indices.size() + 1, 2))
    Wt = Op->getOperand(1);
  else
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2854-2861
```cpp
  // Check odd elements are taken from the odd elements of one half or the
  // other and pick an operand accordingly.
  if (fitsRegularPattern<int>(Begin + 1, 2, End, 1, 2))
    Ws = Op->getOperand(0);
  else if (fitsRegularPattern<int>(Begin + 1, 2, End, Indices.size() + 1, 2))
    Ws = Op->getOperand(1);
  else
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2863-2864
```cpp
  return DAG.getNode(MipsISD::ILVOD, SDLoc(Op), ResTy, Ws, Wt);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2866-2880
```cpp
// Lower VECTOR_SHUFFLE into ILVR (if possible).
//
// ILVR interleaves consecutive elements from the right (lowest-indexed) half of
// each vector.
//
// It is possible to lower into ILVR when the mask consists of two of the
// following forms interleaved:
//   <0, 1, 2, ...>
//   <n, n+1, n+2, ...>
// where n is the number of elements in the vector.
// For example:
//   <0, 0, 1, 1, 2, 2, ...>
//   <0, n, 1, n+1, 2, n+2, ...>
//
// When undef's appear in the mask they are treated as if they were whatever
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 2881-2885
```cpp
// value is necessary in order to fit the above forms.
static SDValue lowerVECTOR_SHUFFLE_ILVR(SDValue Op, EVT ResTy,
                                        SmallVector<int, 16> Indices,
                                        SelectionDAG &DAG) {
  assert((Indices.size() % 2) == 0);
```
- EN: Implements `lowerVECTOR_SHUFFLE_ILVR`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `lowerVECTOR_SHUFFLE_ILVR`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2887-2890
```cpp
  SDValue Wt;
  SDValue Ws;
  const auto &Begin = Indices.begin();
  const auto &End = Indices.end();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2892-2899
```cpp
  // Check even elements are taken from the right (lowest-indexed) elements of
  // one half or the other and pick an operand accordingly.
  if (fitsRegularPattern<int>(Begin, 2, End, 0, 1))
    Wt = Op->getOperand(0);
  else if (fitsRegularPattern<int>(Begin, 2, End, Indices.size(), 1))
    Wt = Op->getOperand(1);
  else
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2901-2908
```cpp
  // Check odd elements are taken from the right (lowest-indexed) elements of
  // one half or the other and pick an operand accordingly.
  if (fitsRegularPattern<int>(Begin + 1, 2, End, 0, 1))
    Ws = Op->getOperand(0);
  else if (fitsRegularPattern<int>(Begin + 1, 2, End, Indices.size(), 1))
    Ws = Op->getOperand(1);
  else
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2910-2911
```cpp
  return DAG.getNode(MipsISD::ILVR, SDLoc(Op), ResTy, Ws, Wt);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2913-2927
```cpp
// Lower VECTOR_SHUFFLE into ILVL (if possible).
//
// ILVL interleaves consecutive elements from the left (highest-indexed) half
// of each vector.
//
// It is possible to lower into ILVL when the mask consists of two of the
// following forms interleaved:
//   <x, x+1, x+2, ...>
//   <n+x, n+x+1, n+x+2, ...>
// where n is the number of elements in the vector and x is half n.
// For example:
//   <x, x, x+1, x+1, x+2, x+2, ...>
//   <x, n+x, x+1, n+x+1, x+2, n+x+2, ...>
//
// When undef's appear in the mask they are treated as if they were whatever
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 2928-2932
```cpp
// value is necessary in order to fit the above forms.
static SDValue lowerVECTOR_SHUFFLE_ILVL(SDValue Op, EVT ResTy,
                                        SmallVector<int, 16> Indices,
                                        SelectionDAG &DAG) {
  assert((Indices.size() % 2) == 0);
```
- EN: Implements `lowerVECTOR_SHUFFLE_ILVL`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `lowerVECTOR_SHUFFLE_ILVL`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2934-2938
```cpp
  unsigned HalfSize = Indices.size() / 2;
  SDValue Wt;
  SDValue Ws;
  const auto &Begin = Indices.begin();
  const auto &End = Indices.end();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2940-2947
```cpp
  // Check even elements are taken from the left (highest-indexed) elements of
  // one half or the other and pick an operand accordingly.
  if (fitsRegularPattern<int>(Begin, 2, End, HalfSize, 1))
    Wt = Op->getOperand(0);
  else if (fitsRegularPattern<int>(Begin, 2, End, Indices.size() + HalfSize, 1))
    Wt = Op->getOperand(1);
  else
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2949-2957
```cpp
  // Check odd elements are taken from the left (highest-indexed) elements of
  // one half or the other and pick an operand accordingly.
  if (fitsRegularPattern<int>(Begin + 1, 2, End, HalfSize, 1))
    Ws = Op->getOperand(0);
  else if (fitsRegularPattern<int>(Begin + 1, 2, End, Indices.size() + HalfSize,
                                   1))
    Ws = Op->getOperand(1);
  else
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2959-2960
```cpp
  return DAG.getNode(MipsISD::ILVL, SDLoc(Op), ResTy, Ws, Wt);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2962-2976
```cpp
// Lower VECTOR_SHUFFLE into PCKEV (if possible).
//
// PCKEV copies the even elements of each vector into the result vector.
//
// It is possible to lower into PCKEV when the mask consists of two of the
// following forms concatenated:
//   <0, 2, 4, ...>
//   <n, n+2, n+4, ...>
// where n is the number of elements in the vector.
// For example:
//   <0, 2, 4, ..., 0, 2, 4, ...>
//   <0, 2, 4, ..., n, n+2, n+4, ...>
//
// When undef's appear in the mask they are treated as if they were whatever
// value is necessary in order to fit the above forms.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 2977-2980
```cpp
static SDValue lowerVECTOR_SHUFFLE_PCKEV(SDValue Op, EVT ResTy,
                                         SmallVector<int, 16> Indices,
                                         SelectionDAG &DAG) {
  assert((Indices.size() % 2) == 0);
```
- EN: Implements `lowerVECTOR_SHUFFLE_PCKEV`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `lowerVECTOR_SHUFFLE_PCKEV`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2982-2986
```cpp
  SDValue Wt;
  SDValue Ws;
  const auto &Begin = Indices.begin();
  const auto &Mid = Indices.begin() + Indices.size() / 2;
  const auto &End = Indices.end();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2988-2993
```cpp
  if (fitsRegularPattern<int>(Begin, 1, Mid, 0, 2))
    Wt = Op->getOperand(0);
  else if (fitsRegularPattern<int>(Begin, 1, Mid, Indices.size(), 2))
    Wt = Op->getOperand(1);
  else
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2995-3000
```cpp
  if (fitsRegularPattern<int>(Mid, 1, End, 0, 2))
    Ws = Op->getOperand(0);
  else if (fitsRegularPattern<int>(Mid, 1, End, Indices.size(), 2))
    Ws = Op->getOperand(1);
  else
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3002-3003
```cpp
  return DAG.getNode(MipsISD::PCKEV, SDLoc(Op), ResTy, Ws, Wt);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3005-3019
```cpp
// Lower VECTOR_SHUFFLE into PCKOD (if possible).
//
// PCKOD copies the odd elements of each vector into the result vector.
//
// It is possible to lower into PCKOD when the mask consists of two of the
// following forms concatenated:
//   <1, 3, 5, ...>
//   <n+1, n+3, n+5, ...>
// where n is the number of elements in the vector.
// For example:
//   <1, 3, 5, ..., 1, 3, 5, ...>
//   <1, 3, 5, ..., n+1, n+3, n+5, ...>
//
// When undef's appear in the mask they are treated as if they were whatever
// value is necessary in order to fit the above forms.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 3020-3023
```cpp
static SDValue lowerVECTOR_SHUFFLE_PCKOD(SDValue Op, EVT ResTy,
                                         SmallVector<int, 16> Indices,
                                         SelectionDAG &DAG) {
  assert((Indices.size() % 2) == 0);
```
- EN: Implements `lowerVECTOR_SHUFFLE_PCKOD`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `lowerVECTOR_SHUFFLE_PCKOD`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3025-3029
```cpp
  SDValue Wt;
  SDValue Ws;
  const auto &Begin = Indices.begin();
  const auto &Mid = Indices.begin() + Indices.size() / 2;
  const auto &End = Indices.end();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3031-3036
```cpp
  if (fitsRegularPattern<int>(Begin, 1, Mid, 1, 2))
    Wt = Op->getOperand(0);
  else if (fitsRegularPattern<int>(Begin, 1, Mid, Indices.size() + 1, 2))
    Wt = Op->getOperand(1);
  else
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3038-3043
```cpp
  if (fitsRegularPattern<int>(Mid, 1, End, 1, 2))
    Ws = Op->getOperand(0);
  else if (fitsRegularPattern<int>(Mid, 1, End, Indices.size() + 1, 2))
    Ws = Op->getOperand(1);
  else
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3045-3046
```cpp
  return DAG.getNode(MipsISD::PCKOD, SDLoc(Op), ResTy, Ws, Wt);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3048-3062
```cpp
// Lower VECTOR_SHUFFLE into VSHF.
//
// This mostly consists of converting the shuffle indices in Indices into a
// BUILD_VECTOR and adding it as an operand to the resulting VSHF. There is
// also code to eliminate unused operands of the VECTOR_SHUFFLE. For example,
// if the type is v8i16 and all the indices are less than 8 then the second
// operand is unused and can be replaced with anything. We choose to replace it
// with the used operand since this reduces the number of instructions overall.
//
// NOTE: SPLATI shuffle masks may contain UNDEFs, since isSPLATI() treats
//       UNDEFs as same as SPLATI index.
//       For other instances we use the last valid index if UNDEF is
//       encountered.
static SDValue lowerVECTOR_SHUFFLE_VSHF(SDValue Op, EVT ResTy,
                                        const SmallVector<int, 16> &Indices,
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3063-3073
```cpp
                                        const bool isSPLATI,
                                        SelectionDAG &DAG) {
  SmallVector<SDValue, 16> Ops;
  SDValue Op0;
  SDValue Op1;
  EVT MaskVecTy = ResTy.changeVectorElementTypeToInteger();
  EVT MaskEltTy = MaskVecTy.getVectorElementType();
  bool Using1stVec = false;
  bool Using2ndVec = false;
  SDLoc DL(Op);
  int ResTyNumElts = ResTy.getVectorNumElements();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3075-3076
```cpp
  assert(Indices[0] >= 0 &&
         "shuffle mask starts with an UNDEF, which is not expected");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3078-3080
```cpp
  for (int i = 0; i < ResTyNumElts; ++i) {
    // Idx == -1 means UNDEF
    int Idx = Indices[i];
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3082-3096
```cpp
    if (0 <= Idx && Idx < ResTyNumElts)
      Using1stVec = true;
    if (ResTyNumElts <= Idx && Idx < ResTyNumElts * 2)
      Using2ndVec = true;
  }
  int LastValidIndex = 0;
  for (size_t i = 0; i < Indices.size(); i++) {
    int Idx = Indices[i];
    if (Idx < 0) {
      // Continue using splati index or use the last valid index.
      Idx = isSPLATI ? Indices[0] : LastValidIndex;
    } else {
      LastValidIndex = Idx;
    }
    Ops.push_back(DAG.getTargetConstant(Idx, DL, MaskEltTy));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3097-3097
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3099-3099
```cpp
  SDValue MaskVec = DAG.getBuildVector(MaskVecTy, DL, Ops);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3101-3109
```cpp
  if (Using1stVec && Using2ndVec) {
    Op0 = Op->getOperand(0);
    Op1 = Op->getOperand(1);
  } else if (Using1stVec)
    Op0 = Op1 = Op->getOperand(0);
  else if (Using2ndVec)
    Op0 = Op1 = Op->getOperand(1);
  else
    llvm_unreachable("shuffle vector mask references neither vector operand?");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3111-3119
```cpp
  // VECTOR_SHUFFLE concatenates the vectors in an vectorwise fashion.
  // <0b00, 0b01> + <0b10, 0b11> -> <0b00, 0b01, 0b10, 0b11>
  // VSHF concatenates the vectors in a bitwise fashion:
  // <0b00, 0b01> + <0b10, 0b11> ->
  // 0b0100       + 0b1110       -> 0b01001110
  //                                <0b10, 0b11, 0b00, 0b01>
  // We must therefore swap the operands to get the correct result.
  return DAG.getNode(MipsISD::VSHF, DL, ResTy, MaskVec, Op1, Op0);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3121-3126
```cpp
// Lower VECTOR_SHUFFLE into one of a number of instructions depending on the
// indices in the shuffle.
SDValue MipsSETargetLowering::lowerVECTOR_SHUFFLE(SDValue Op,
                                                  SelectionDAG &DAG) const {
  ShuffleVectorSDNode *Node = cast<ShuffleVectorSDNode>(Op);
  EVT ResTy = Op->getValueType(0);
```
- EN: Implements `MipsSETargetLowering::lowerVECTOR_SHUFFLE`, a lowering routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSETargetLowering::lowerVECTOR_SHUFFLE`，它是一个围绕SelectionDAG 降级展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3128-3129
```cpp
  if (!ResTy.is128BitVector())
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3131-3132
```cpp
  int ResTyNumElts = ResTy.getVectorNumElements();
  SmallVector<int, 16> Indices;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3134-3135
```cpp
  for (int i = 0; i < ResTyNumElts; ++i)
    Indices.push_back(Node->getMaskElt(i));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3137-3151
```cpp
  // splati.[bhwd] is preferable to the others but is matched from
  // MipsISD::VSHF.
  if (isVECTOR_SHUFFLE_SPLATI(Op, ResTy, Indices, DAG))
    return lowerVECTOR_SHUFFLE_VSHF(Op, ResTy, Indices, true, DAG);
  SDValue Result;
  if ((Result = lowerVECTOR_SHUFFLE_ILVEV(Op, ResTy, Indices, DAG)))
    return Result;
  if ((Result = lowerVECTOR_SHUFFLE_ILVOD(Op, ResTy, Indices, DAG)))
    return Result;
  if ((Result = lowerVECTOR_SHUFFLE_ILVL(Op, ResTy, Indices, DAG)))
    return Result;
  if ((Result = lowerVECTOR_SHUFFLE_ILVR(Op, ResTy, Indices, DAG)))
    return Result;
  if ((Result = lowerVECTOR_SHUFFLE_PCKEV(Op, ResTy, Indices, DAG)))
    return Result;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3152-3157
```cpp
  if ((Result = lowerVECTOR_SHUFFLE_PCKOD(Op, ResTy, Indices, DAG)))
    return Result;
  if ((Result = lowerVECTOR_SHUFFLE_SHF(Op, ResTy, Indices, DAG)))
    return Result;
  return lowerVECTOR_SHUFFLE_VSHF(Op, ResTy, Indices, false, DAG);
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3159-3173
```cpp
MachineBasicBlock *
MipsSETargetLowering::emitBPOSGE32(MachineInstr &MI,
                                   MachineBasicBlock *BB) const {
  // $bb:
  //  bposge32_pseudo $vr0
  //  =>
  // $bb:
  //  bposge32 $tbb
  // $fbb:
  //  li $vr2, 0
  //  b $sink
  // $tbb:
  //  li $vr1, 1
  // $sink:
  //  $vr0 = phi($vr2, $fbb, $vr1, $tbb)
```
- EN: Implements `MipsSETargetLowering::emitBPOSGE32`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSETargetLowering::emitBPOSGE32`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3175-3187
```cpp
  MachineRegisterInfo &RegInfo = BB->getParent()->getRegInfo();
  const TargetInstrInfo *TII = Subtarget.getInstrInfo();
  const TargetRegisterClass *RC = &Mips::GPR32RegClass;
  DebugLoc DL = MI.getDebugLoc();
  const BasicBlock *LLVM_BB = BB->getBasicBlock();
  MachineFunction::iterator It = std::next(MachineFunction::iterator(BB));
  MachineFunction *F = BB->getParent();
  MachineBasicBlock *FBB = F->CreateMachineBasicBlock(LLVM_BB);
  MachineBasicBlock *TBB = F->CreateMachineBasicBlock(LLVM_BB);
  MachineBasicBlock *Sink  = F->CreateMachineBasicBlock(LLVM_BB);
  F->insert(It, FBB);
  F->insert(It, TBB);
  F->insert(It, Sink);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3189-3192
```cpp
  // Transfer the remainder of BB and its successor edges to Sink.
  Sink->splice(Sink->begin(), BB, std::next(MachineBasicBlock::iterator(MI)),
               BB->end());
  Sink->transferSuccessorsAndUpdatePHIs(BB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3194-3198
```cpp
  // Add successors.
  BB->addSuccessor(FBB);
  BB->addSuccessor(TBB);
  FBB->addSuccessor(Sink);
  TBB->addSuccessor(Sink);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3200-3203
```cpp
  // Insert the real bposge32 instruction to $BB.
  BuildMI(BB, DL, TII->get(Mips::BPOSGE32)).addMBB(TBB);
  // Insert the real bposge32c instruction to $BB.
  BuildMI(BB, DL, TII->get(Mips::BPOSGE32C_MMR3)).addMBB(TBB);
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3205-3209
```cpp
  // Fill $FBB.
  Register VR2 = RegInfo.createVirtualRegister(RC);
  BuildMI(*FBB, FBB->end(), DL, TII->get(Mips::ADDiu), VR2)
    .addReg(Mips::ZERO).addImm(0);
  BuildMI(*FBB, FBB->end(), DL, TII->get(Mips::B)).addMBB(Sink);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3211-3214
```cpp
  // Fill $TBB.
  Register VR1 = RegInfo.createVirtualRegister(RC);
  BuildMI(*TBB, TBB->end(), DL, TII->get(Mips::ADDiu), VR1)
    .addReg(Mips::ZERO).addImm(1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3216-3222
```cpp
  // Insert phi function to $Sink.
  BuildMI(*Sink, Sink->begin(), DL, TII->get(Mips::PHI),
          MI.getOperand(0).getReg())
      .addReg(VR2)
      .addMBB(FBB)
      .addReg(VR1)
      .addMBB(TBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3224-3226
```cpp
  MI.eraseFromParent(); // The pseudo instruction is gone now.
  return Sink;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3228-3242
```cpp
MachineBasicBlock *MipsSETargetLowering::emitMSACBranchPseudo(
    MachineInstr &MI, MachineBasicBlock *BB, unsigned BranchOp) const {
  // $bb:
  //  vany_nonzero $rd, $ws
  //  =>
  // $bb:
  //  bnz.b $ws, $tbb
  //  b $fbb
  // $fbb:
  //  li $rd1, 0
  //  b $sink
  // $tbb:
  //  li $rd2, 1
  // $sink:
  //  $rd = phi($rd1, $fbb, $rd2, $tbb)
```
- EN: Implements `MipsSETargetLowering::emitMSACBranchPseudo`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSETargetLowering::emitMSACBranchPseudo`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3244-3256
```cpp
  MachineRegisterInfo &RegInfo = BB->getParent()->getRegInfo();
  const TargetInstrInfo *TII = Subtarget.getInstrInfo();
  const TargetRegisterClass *RC = &Mips::GPR32RegClass;
  DebugLoc DL = MI.getDebugLoc();
  const BasicBlock *LLVM_BB = BB->getBasicBlock();
  MachineFunction::iterator It = std::next(MachineFunction::iterator(BB));
  MachineFunction *F = BB->getParent();
  MachineBasicBlock *FBB = F->CreateMachineBasicBlock(LLVM_BB);
  MachineBasicBlock *TBB = F->CreateMachineBasicBlock(LLVM_BB);
  MachineBasicBlock *Sink  = F->CreateMachineBasicBlock(LLVM_BB);
  F->insert(It, FBB);
  F->insert(It, TBB);
  F->insert(It, Sink);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3258-3261
```cpp
  // Transfer the remainder of BB and its successor edges to Sink.
  Sink->splice(Sink->begin(), BB, std::next(MachineBasicBlock::iterator(MI)),
               BB->end());
  Sink->transferSuccessorsAndUpdatePHIs(BB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3263-3267
```cpp
  // Add successors.
  BB->addSuccessor(FBB);
  BB->addSuccessor(TBB);
  FBB->addSuccessor(Sink);
  TBB->addSuccessor(Sink);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3269-3272
```cpp
  // Insert the real bnz.b instruction to $BB.
  BuildMI(BB, DL, TII->get(BranchOp))
      .addReg(MI.getOperand(1).getReg())
      .addMBB(TBB);
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3274-3278
```cpp
  // Fill $FBB.
  Register RD1 = RegInfo.createVirtualRegister(RC);
  BuildMI(*FBB, FBB->end(), DL, TII->get(Mips::ADDiu), RD1)
    .addReg(Mips::ZERO).addImm(0);
  BuildMI(*FBB, FBB->end(), DL, TII->get(Mips::B)).addMBB(Sink);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3280-3283
```cpp
  // Fill $TBB.
  Register RD2 = RegInfo.createVirtualRegister(RC);
  BuildMI(*TBB, TBB->end(), DL, TII->get(Mips::ADDiu), RD2)
    .addReg(Mips::ZERO).addImm(1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3285-3291
```cpp
  // Insert phi function to $Sink.
  BuildMI(*Sink, Sink->begin(), DL, TII->get(Mips::PHI),
          MI.getOperand(0).getReg())
      .addReg(RD1)
      .addMBB(FBB)
      .addReg(RD2)
      .addMBB(TBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3293-3295
```cpp
  MI.eraseFromParent(); // The pseudo instruction is gone now.
  return Sink;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3297-3311
```cpp
// Emit the COPY_FW pseudo instruction.
//
// copy_fw_pseudo $fd, $ws, n
// =>
// copy_u_w $rt, $ws, $n
// mtc1     $rt, $fd
//
// When n is zero, the equivalent operation can be performed with (potentially)
// zero instructions due to register overlaps. This optimization is never valid
// for lane 1 because it would require FR=0 mode which isn't supported by MSA.
MachineBasicBlock *
MipsSETargetLowering::emitCOPY_FW(MachineInstr &MI,
                                  MachineBasicBlock *BB) const {
  const TargetInstrInfo *TII = Subtarget.getInstrInfo();
  MachineRegisterInfo &RegInfo = BB->getParent()->getRegInfo();
```
- EN: Implements `MipsSETargetLowering::emitCOPY_FW`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSETargetLowering::emitCOPY_FW`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3312-3315
```cpp
  DebugLoc DL = MI.getDebugLoc();
  Register Fd = MI.getOperand(0).getReg();
  Register Ws = MI.getOperand(1).getReg();
  unsigned Lane = MI.getOperand(2).getImm();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3317-3322
```cpp
  if (Lane == 0) {
    unsigned Wt = Ws;
    if (!Subtarget.useOddSPReg()) {
      // We must copy to an even-numbered MSA register so that the
      // single-precision sub-register is also guaranteed to be even-numbered.
      Wt = RegInfo.createVirtualRegister(&Mips::MSA128WEvensRegClass);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3324-3325
```cpp
      BuildMI(*BB, MI, DL, TII->get(Mips::COPY), Wt).addReg(Ws);
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3327-3331
```cpp
    BuildMI(*BB, MI, DL, TII->get(Mips::COPY), Fd).addReg(Wt, {}, Mips::sub_lo);
  } else {
    Register Wt = RegInfo.createVirtualRegister(
        Subtarget.useOddSPReg() ? &Mips::MSA128WRegClass
                                : &Mips::MSA128WEvensRegClass);
```
- EN: Implements `BuildMI`, a mutation/build routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `BuildMI`，它是一个围绕寄存器管理展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3333-3335
```cpp
    BuildMI(*BB, MI, DL, TII->get(Mips::SPLATI_W), Wt).addReg(Ws).addImm(Lane);
    BuildMI(*BB, MI, DL, TII->get(Mips::COPY), Fd).addReg(Wt, {}, Mips::sub_lo);
  }
```
- EN: Implements `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3337-3339
```cpp
  MI.eraseFromParent(); // The pseudo instruction is gone now.
  return BB;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3341-3354
```cpp
// Emit the COPY_FD pseudo instruction.
//
// copy_fd_pseudo $fd, $ws, n
// =>
// splati.d $wt, $ws, $n
// copy $fd, $wt:sub_64
//
// When n is zero, the equivalent operation can be performed with (potentially)
// zero instructions due to register overlaps. This optimization is always
// valid because FR=1 mode which is the only supported mode in MSA.
MachineBasicBlock *
MipsSETargetLowering::emitCOPY_FD(MachineInstr &MI,
                                  MachineBasicBlock *BB) const {
  assert(Subtarget.isFP64bit());
```
- EN: Implements `MipsSETargetLowering::emitCOPY_FD`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSETargetLowering::emitCOPY_FD`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3356-3361
```cpp
  const TargetInstrInfo *TII = Subtarget.getInstrInfo();
  MachineRegisterInfo &RegInfo = BB->getParent()->getRegInfo();
  Register Fd = MI.getOperand(0).getReg();
  Register Ws = MI.getOperand(1).getReg();
  unsigned Lane = MI.getOperand(2).getImm() * 2;
  DebugLoc DL = MI.getDebugLoc();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3363-3366
```cpp
  if (Lane == 0)
    BuildMI(*BB, MI, DL, TII->get(Mips::COPY), Fd).addReg(Ws, {}, Mips::sub_64);
  else {
    Register Wt = RegInfo.createVirtualRegister(&Mips::MSA128DRegClass);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3368-3370
```cpp
    BuildMI(*BB, MI, DL, TII->get(Mips::SPLATI_D), Wt).addReg(Ws).addImm(1);
    BuildMI(*BB, MI, DL, TII->get(Mips::COPY), Fd).addReg(Wt, {}, Mips::sub_64);
  }
```
- EN: Implements `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3372-3374
```cpp
  MI.eraseFromParent(); // The pseudo instruction is gone now.
  return BB;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3376-3390
```cpp
// Emit the INSERT_FW pseudo instruction.
//
// insert_fw_pseudo $wd, $wd_in, $n, $fs
// =>
// subreg_to_reg $wt:sub_lo, $fs
// insve_w $wd[$n], $wd_in, $wt[0]
MachineBasicBlock *
MipsSETargetLowering::emitINSERT_FW(MachineInstr &MI,
                                    MachineBasicBlock *BB) const {
  const TargetInstrInfo *TII = Subtarget.getInstrInfo();
  MachineRegisterInfo &RegInfo = BB->getParent()->getRegInfo();
  DebugLoc DL = MI.getDebugLoc();
  Register Wd = MI.getOperand(0).getReg();
  Register Wd_in = MI.getOperand(1).getReg();
  unsigned Lane = MI.getOperand(2).getImm();
```
- EN: Implements `MipsSETargetLowering::emitINSERT_FW`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSETargetLowering::emitINSERT_FW`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3391-3394
```cpp
  Register Fs = MI.getOperand(3).getReg();
  Register Wt = RegInfo.createVirtualRegister(
      Subtarget.useOddSPReg() ? &Mips::MSA128WRegClass
                              : &Mips::MSA128WEvensRegClass);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3396-3403
```cpp
  BuildMI(*BB, MI, DL, TII->get(Mips::SUBREG_TO_REG), Wt)
      .addReg(Fs)
      .addImm(Mips::sub_lo);
  BuildMI(*BB, MI, DL, TII->get(Mips::INSVE_W), Wd)
      .addReg(Wd_in)
      .addImm(Lane)
      .addReg(Wt)
      .addImm(0);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3405-3407
```cpp
  MI.eraseFromParent(); // The pseudo instruction is gone now.
  return BB;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3409-3418
```cpp
// Emit the INSERT_FD pseudo instruction.
//
// insert_fd_pseudo $wd, $fs, n
// =>
// subreg_to_reg $wt:sub_64, $fs
// insve_d $wd[$n], $wd_in, $wt[0]
MachineBasicBlock *
MipsSETargetLowering::emitINSERT_FD(MachineInstr &MI,
                                    MachineBasicBlock *BB) const {
  assert(Subtarget.isFP64bit());
```
- EN: Implements `MipsSETargetLowering::emitINSERT_FD`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSETargetLowering::emitINSERT_FD`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3420-3427
```cpp
  const TargetInstrInfo *TII = Subtarget.getInstrInfo();
  MachineRegisterInfo &RegInfo = BB->getParent()->getRegInfo();
  DebugLoc DL = MI.getDebugLoc();
  Register Wd = MI.getOperand(0).getReg();
  Register Wd_in = MI.getOperand(1).getReg();
  unsigned Lane = MI.getOperand(2).getImm();
  Register Fs = MI.getOperand(3).getReg();
  Register Wt = RegInfo.createVirtualRegister(&Mips::MSA128DRegClass);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3429-3436
```cpp
  BuildMI(*BB, MI, DL, TII->get(Mips::SUBREG_TO_REG), Wt)
      .addReg(Fs)
      .addImm(Mips::sub_64);
  BuildMI(*BB, MI, DL, TII->get(Mips::INSVE_D), Wd)
      .addReg(Wd_in)
      .addImm(Lane)
      .addReg(Wt)
      .addImm(0);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3438-3440
```cpp
  MI.eraseFromParent(); // The pseudo instruction is gone now.
  return BB;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3442-3456
```cpp
// Emit the INSERT_([BHWD]|F[WD])_VIDX pseudo instruction.
//
// For integer:
// (INSERT_([BHWD]|F[WD])_PSEUDO $wd, $wd_in, $n, $rs)
// =>
// (SLL $lanetmp1, $lane, <log2size)
// (SLD_B $wdtmp1, $wd_in, $wd_in, $lanetmp1)
// (INSERT_[BHWD], $wdtmp2, $wdtmp1, 0, $rs)
// (NEG $lanetmp2, $lanetmp1)
// (SLD_B $wd, $wdtmp2, $wdtmp2,  $lanetmp2)
//
// For floating point:
// (INSERT_([BHWD]|F[WD])_PSEUDO $wd, $wd_in, $n, $fs)
// =>
// (SUBREG_TO_REG $wt, $fs, <subreg>)
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 3457-3471
```cpp
// (SLL $lanetmp1, $lane, <log2size)
// (SLD_B $wdtmp1, $wd_in, $wd_in, $lanetmp1)
// (INSVE_[WD], $wdtmp2, 0, $wdtmp1, 0)
// (NEG $lanetmp2, $lanetmp1)
// (SLD_B $wd, $wdtmp2, $wdtmp2,  $lanetmp2)
MachineBasicBlock *MipsSETargetLowering::emitINSERT_DF_VIDX(
    MachineInstr &MI, MachineBasicBlock *BB, unsigned EltSizeInBytes,
    bool IsFP) const {
  const TargetInstrInfo *TII = Subtarget.getInstrInfo();
  MachineRegisterInfo &RegInfo = BB->getParent()->getRegInfo();
  DebugLoc DL = MI.getDebugLoc();
  Register Wd = MI.getOperand(0).getReg();
  Register SrcVecReg = MI.getOperand(1).getReg();
  Register LaneReg = MI.getOperand(2).getReg();
  Register SrcValReg = MI.getOperand(3).getReg();
```
- EN: Implements `MipsSETargetLowering::emitINSERT_DF_VIDX`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSETargetLowering::emitINSERT_DF_VIDX`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3473-3487
```cpp
  const TargetRegisterClass *VecRC = nullptr;
  // FIXME: This should be true for N32 too.
  const TargetRegisterClass *GPRRC =
      Subtarget.isABI_N64() ? &Mips::GPR64RegClass : &Mips::GPR32RegClass;
  unsigned SubRegIdx = Subtarget.isABI_N64() ? Mips::sub_32 : 0;
  unsigned ShiftOp = Subtarget.isABI_N64() ? Mips::DSLL : Mips::SLL;
  unsigned EltLog2Size;
  unsigned InsertOp = 0;
  unsigned InsveOp = 0;
  switch (EltSizeInBytes) {
  default:
    llvm_unreachable("Unexpected size");
  case 1:
    EltLog2Size = 0;
    InsertOp = Mips::INSERT_B;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3488-3502
```cpp
    InsveOp = Mips::INSVE_B;
    VecRC = &Mips::MSA128BRegClass;
    break;
  case 2:
    EltLog2Size = 1;
    InsertOp = Mips::INSERT_H;
    InsveOp = Mips::INSVE_H;
    VecRC = &Mips::MSA128HRegClass;
    break;
  case 4:
    EltLog2Size = 2;
    InsertOp = Mips::INSERT_W;
    InsveOp = Mips::INSVE_W;
    VecRC = &Mips::MSA128WRegClass;
    break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3503-3509
```cpp
  case 8:
    EltLog2Size = 3;
    InsertOp = Mips::INSERT_D;
    InsveOp = Mips::INSVE_D;
    VecRC = &Mips::MSA128DRegClass;
    break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3511-3517
```cpp
  if (IsFP) {
    Register Wt = RegInfo.createVirtualRegister(VecRC);
    BuildMI(*BB, MI, DL, TII->get(Mips::SUBREG_TO_REG), Wt)
        .addReg(SrcValReg)
        .addImm(EltSizeInBytes == 8 ? Mips::sub_64 : Mips::sub_lo);
    SrcValReg = Wt;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3519-3526
```cpp
  // Convert the lane index into a byte index
  if (EltSizeInBytes != 1) {
    Register LaneTmp1 = RegInfo.createVirtualRegister(GPRRC);
    BuildMI(*BB, MI, DL, TII->get(ShiftOp), LaneTmp1)
        .addReg(LaneReg)
        .addImm(EltLog2Size);
    LaneReg = LaneTmp1;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3528-3533
```cpp
  // Rotate bytes around so that the desired lane is element zero
  Register WdTmp1 = RegInfo.createVirtualRegister(VecRC);
  BuildMI(*BB, MI, DL, TII->get(Mips::SLD_B), WdTmp1)
      .addReg(SrcVecReg)
      .addReg(SrcVecReg)
      .addReg(LaneReg, {}, SubRegIdx);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3535-3549
```cpp
  Register WdTmp2 = RegInfo.createVirtualRegister(VecRC);
  if (IsFP) {
    // Use insve.df to insert to element zero
    BuildMI(*BB, MI, DL, TII->get(InsveOp), WdTmp2)
        .addReg(WdTmp1)
        .addImm(0)
        .addReg(SrcValReg)
        .addImm(0);
  } else {
    // Use insert.df to insert to element zero
    BuildMI(*BB, MI, DL, TII->get(InsertOp), WdTmp2)
        .addReg(WdTmp1)
        .addReg(SrcValReg)
        .addImm(0);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3551-3562
```cpp
  // Rotate elements the rest of the way for a full rotation.
  // sld.df inteprets $rt modulo the number of columns so we only need to negate
  // the lane index to do this.
  Register LaneTmp2 = RegInfo.createVirtualRegister(GPRRC);
  BuildMI(*BB, MI, DL, TII->get(Subtarget.isABI_N64() ? Mips::DSUB : Mips::SUB),
          LaneTmp2)
      .addReg(Subtarget.isABI_N64() ? Mips::ZERO_64 : Mips::ZERO)
      .addReg(LaneReg);
  BuildMI(*BB, MI, DL, TII->get(Mips::SLD_B), Wd)
      .addReg(WdTmp2)
      .addReg(WdTmp2)
      .addReg(LaneTmp2, {}, SubRegIdx);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3564-3566
```cpp
  MI.eraseFromParent(); // The pseudo instruction is gone now.
  return BB;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3568-3582
```cpp
// Emit the FILL_FW pseudo instruction.
//
// fill_fw_pseudo $wd, $fs
// =>
// implicit_def $wt1
// insert_subreg $wt2:subreg_lo, $wt1, $fs
// splati.w $wd, $wt2[0]
MachineBasicBlock *
MipsSETargetLowering::emitFILL_FW(MachineInstr &MI,
                                  MachineBasicBlock *BB) const {
  const TargetInstrInfo *TII = Subtarget.getInstrInfo();
  MachineRegisterInfo &RegInfo = BB->getParent()->getRegInfo();
  DebugLoc DL = MI.getDebugLoc();
  Register Wd = MI.getOperand(0).getReg();
  Register Fs = MI.getOperand(1).getReg();
```
- EN: Implements `MipsSETargetLowering::emitFILL_FW`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSETargetLowering::emitFILL_FW`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3583-3588
```cpp
  Register Wt1 = RegInfo.createVirtualRegister(
      Subtarget.useOddSPReg() ? &Mips::MSA128WRegClass
                              : &Mips::MSA128WEvensRegClass);
  Register Wt2 = RegInfo.createVirtualRegister(
      Subtarget.useOddSPReg() ? &Mips::MSA128WRegClass
                              : &Mips::MSA128WEvensRegClass);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3590-3595
```cpp
  BuildMI(*BB, MI, DL, TII->get(Mips::IMPLICIT_DEF), Wt1);
  BuildMI(*BB, MI, DL, TII->get(Mips::INSERT_SUBREG), Wt2)
      .addReg(Wt1)
      .addReg(Fs)
      .addImm(Mips::sub_lo);
  BuildMI(*BB, MI, DL, TII->get(Mips::SPLATI_W), Wd).addReg(Wt2).addImm(0);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3597-3599
```cpp
  MI.eraseFromParent(); // The pseudo instruction is gone now.
  return BB;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3601-3611
```cpp
// Emit the FILL_FD pseudo instruction.
//
// fill_fd_pseudo $wd, $fs
// =>
// implicit_def $wt1
// insert_subreg $wt2:subreg_64, $wt1, $fs
// splati.d $wd, $wt2[0]
MachineBasicBlock *
MipsSETargetLowering::emitFILL_FD(MachineInstr &MI,
                                  MachineBasicBlock *BB) const {
  assert(Subtarget.isFP64bit());
```
- EN: Implements `MipsSETargetLowering::emitFILL_FD`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSETargetLowering::emitFILL_FD`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3613-3619
```cpp
  const TargetInstrInfo *TII = Subtarget.getInstrInfo();
  MachineRegisterInfo &RegInfo = BB->getParent()->getRegInfo();
  DebugLoc DL = MI.getDebugLoc();
  Register Wd = MI.getOperand(0).getReg();
  Register Fs = MI.getOperand(1).getReg();
  Register Wt1 = RegInfo.createVirtualRegister(&Mips::MSA128DRegClass);
  Register Wt2 = RegInfo.createVirtualRegister(&Mips::MSA128DRegClass);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3621-3626
```cpp
  BuildMI(*BB, MI, DL, TII->get(Mips::IMPLICIT_DEF), Wt1);
  BuildMI(*BB, MI, DL, TII->get(Mips::INSERT_SUBREG), Wt2)
      .addReg(Wt1)
      .addReg(Fs)
      .addImm(Mips::sub_64);
  BuildMI(*BB, MI, DL, TII->get(Mips::SPLATI_D), Wd).addReg(Wt2).addImm(0);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3628-3630
```cpp
  MI.eraseFromParent(); // The pseudo instruction is gone now.
  return BB;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3632-3645
```cpp
// Emit the ST_F16_PSEDUO instruction to store a f16 value from an MSA
// register.
//
// STF16 MSA128F16:$wd, mem_simm10:$addr
// =>
//  copy_u.h $rtemp,$wd[0]
//  sh $rtemp, $addr
//
// Safety: We can't use st.h & co as they would over write the memory after
// the destination. It would require half floats be allocated 16 bytes(!) of
// space.
MachineBasicBlock *
MipsSETargetLowering::emitST_F16_PSEUDO(MachineInstr &MI,
                                       MachineBasicBlock *BB) const {
```
- EN: Implements `MipsSETargetLowering::emitST_F16_PSEUDO`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSETargetLowering::emitST_F16_PSEUDO`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3647-3653
```cpp
  const TargetInstrInfo *TII = Subtarget.getInstrInfo();
  MachineRegisterInfo &RegInfo = BB->getParent()->getRegInfo();
  DebugLoc DL = MI.getDebugLoc();
  Register Ws = MI.getOperand(0).getReg();
  Register Rt = MI.getOperand(1).getReg();
  const MachineMemOperand &MMO = **MI.memoperands_begin();
  unsigned Imm = MMO.getOffset();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3655-3663
```cpp
  // Caution: A load via the GOT can expand to a GPR32 operand, a load via
  //          spill and reload can expand as a GPR64 operand. Examine the
  //          operand in detail and default to ABI.
  const TargetRegisterClass *RC =
      MI.getOperand(1).isReg() ? RegInfo.getRegClass(MI.getOperand(1).getReg())
                               : (Subtarget.isABI_O32() ? &Mips::GPR32RegClass
                                                        : &Mips::GPR64RegClass);
  const bool UsingMips32 = RC == &Mips::GPR32RegClass;
  Register Rs = RegInfo.createVirtualRegister(&Mips::GPR32RegClass);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3665-3678
```cpp
  BuildMI(*BB, MI, DL, TII->get(Mips::COPY_U_H), Rs).addReg(Ws).addImm(0);
  if(!UsingMips32) {
    Register Tmp = RegInfo.createVirtualRegister(&Mips::GPR64RegClass);
    BuildMI(*BB, MI, DL, TII->get(Mips::SUBREG_TO_REG), Tmp)
        .addReg(Rs)
        .addImm(Mips::sub_32);
    Rs = Tmp;
  }
  BuildMI(*BB, MI, DL, TII->get(UsingMips32 ? Mips::SH : Mips::SH64))
      .addReg(Rs)
      .addReg(Rt)
      .addImm(Imm)
      .addMemOperand(BB->getParent()->getMachineMemOperand(
          &MMO, MMO.getOffset(), MMO.getSize()));
```
- EN: Implements `BuildMI`, a mutation/build routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `BuildMI`，它是一个围绕寄存器管理展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3680-3682
```cpp
  MI.eraseFromParent();
  return BB;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3684-3698
```cpp
// Emit the LD_F16_PSEDUO instruction to load a f16 value into an MSA register.
//
// LD_F16 MSA128F16:$wd, mem_simm10:$addr
// =>
//  lh $rtemp, $addr
//  fill.h $wd, $rtemp
//
// Safety: We can't use ld.h & co as they over-read from the source.
// Additionally, if the address is not modulo 16, 2 cases can occur:
//  a) Segmentation fault as the load instruction reads from a memory page
//     memory it's not supposed to.
//  b) The load crosses an implementation specific boundary, requiring OS
//     intervention.
MachineBasicBlock *
MipsSETargetLowering::emitLD_F16_PSEUDO(MachineInstr &MI,
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3699-3699
```cpp
                                       MachineBasicBlock *BB) const {
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3701-3704
```cpp
  const TargetInstrInfo *TII = Subtarget.getInstrInfo();
  MachineRegisterInfo &RegInfo = BB->getParent()->getRegInfo();
  DebugLoc DL = MI.getDebugLoc();
  Register Wd = MI.getOperand(0).getReg();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3706-3712
```cpp
  // Caution: A load via the GOT can expand to a GPR32 operand, a load via
  //          spill and reload can expand as a GPR64 operand. Examine the
  //          operand in detail and default to ABI.
  const TargetRegisterClass *RC =
      MI.getOperand(1).isReg() ? RegInfo.getRegClass(MI.getOperand(1).getReg())
                               : (Subtarget.isABI_O32() ? &Mips::GPR32RegClass
                                                        : &Mips::GPR64RegClass);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3714-3715
```cpp
  const bool UsingMips32 = RC == &Mips::GPR32RegClass;
  Register Rt = RegInfo.createVirtualRegister(RC);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3717-3720
```cpp
  MachineInstrBuilder MIB =
      BuildMI(*BB, MI, DL, TII->get(UsingMips32 ? Mips::LH : Mips::LH64), Rt);
  for (const MachineOperand &MO : llvm::drop_begin(MI.operands()))
    MIB.add(MO);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3722-3727
```cpp
  if(!UsingMips32) {
    Register Tmp = RegInfo.createVirtualRegister(&Mips::GPR32RegClass);
    BuildMI(*BB, MI, DL, TII->get(Mips::COPY), Tmp)
        .addReg(Rt, {}, Mips::sub_32);
    Rt = Tmp;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3729-3729
```cpp
  BuildMI(*BB, MI, DL, TII->get(Mips::FILL_H), Wd).addReg(Rt);
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3731-3733
```cpp
  MI.eraseFromParent();
  return BB;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3735-3749
```cpp
// Emit the FPROUND_PSEUDO instruction.
//
// Round an FGR64Opnd, FGR32Opnd to an f16.
//
// Safety: Cycle the operand through the GPRs so the result always ends up
//         the correct MSA register.
//
// FIXME: This copying is strictly unnecessary. If we could tie FGR32Opnd:$Fs
//        / FGR64Opnd:$Fs and MSA128F16:$Wd to the same physical register
//        (which they can be, as the MSA registers are defined to alias the
//        FPU's 64 bit and 32 bit registers) the result can be accessed using
//        the correct register class. That requires operands be tie-able across
//        register classes which have a sub/super register class relationship.
//
// For FPG32Opnd:
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 3750-3764
```cpp
//
// FPROUND MSA128F16:$wd, FGR32Opnd:$fs
// =>
//  mfc1 $rtemp, $fs
//  fill.w $rtemp, $wtemp
//  fexdo.w $wd, $wtemp, $wtemp
//
// For FPG64Opnd on mips32r2+:
//
// FPROUND MSA128F16:$wd, FGR64Opnd:$fs
// =>
//  mfc1 $rtemp, $fs
//  fill.w $rtemp, $wtemp
//  mfhc1 $rtemp2, $fs
//  insert.w $wtemp[1], $rtemp2
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 3765-3779
```cpp
//  insert.w $wtemp[3], $rtemp2
//  fexdo.w $wtemp2, $wtemp, $wtemp
//  fexdo.h $wd, $temp2, $temp2
//
// For FGR64Opnd on mips64r2+:
//
// FPROUND MSA128F16:$wd, FGR64Opnd:$fs
// =>
//  dmfc1 $rtemp, $fs
//  fill.d $rtemp, $wtemp
//  fexdo.w $wtemp2, $wtemp, $wtemp
//  fexdo.h $wd, $wtemp2, $wtemp2
//
// Safety note: As $wtemp is UNDEF, we may provoke a spurious exception if the
//              undef bits are "just right" and the exception enable bits are
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 3780-3787
```cpp
//              set. By using fill.w to replicate $fs into all elements over
//              insert.w for one element, we avoid that potiential case. If
//              fexdo.[hw] causes an exception in, the exception is valid and it
//              occurs for all elements.
MachineBasicBlock *
MipsSETargetLowering::emitFPROUND_PSEUDO(MachineInstr &MI,
                                         MachineBasicBlock *BB,
                                         bool IsFGR64) const {
```
- EN: Implements `MipsSETargetLowering::emitFPROUND_PSEUDO`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSETargetLowering::emitFPROUND_PSEUDO`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3789-3792
```cpp
  // Strictly speaking, we need MIPS32R5 to support MSA. We'll be generous
  // here. It's technically doable to support MIPS32 here, but the ISA forbids
  // it.
  assert(Subtarget.hasMSA() && Subtarget.hasMips32r2());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3794-3795
```cpp
  bool IsFGR64onMips64 = Subtarget.hasMips64() && IsFGR64;
  bool IsFGR64onMips32 = !Subtarget.hasMips64() && IsFGR64;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3797-3800
```cpp
  const TargetInstrInfo *TII = Subtarget.getInstrInfo();
  DebugLoc DL = MI.getDebugLoc();
  Register Wd = MI.getOperand(0).getReg();
  Register Fs = MI.getOperand(1).getReg();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3802-3809
```cpp
  MachineRegisterInfo &RegInfo = BB->getParent()->getRegInfo();
  Register Wtemp = RegInfo.createVirtualRegister(&Mips::MSA128WRegClass);
  const TargetRegisterClass *GPRRC =
      IsFGR64onMips64 ? &Mips::GPR64RegClass : &Mips::GPR32RegClass;
  unsigned MFC1Opc = IsFGR64onMips64
                         ? Mips::DMFC1
                         : (IsFGR64onMips32 ? Mips::MFC1_D64 : Mips::MFC1);
  unsigned FILLOpc = IsFGR64onMips64 ? Mips::FILL_D : Mips::FILL_W;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3811-3815
```cpp
  // Perform the register class copy as mentioned above.
  Register Rtemp = RegInfo.createVirtualRegister(GPRRC);
  BuildMI(*BB, MI, DL, TII->get(MFC1Opc), Rtemp).addReg(Fs);
  BuildMI(*BB, MI, DL, TII->get(FILLOpc), Wtemp).addReg(Rtemp);
  unsigned WPHI = Wtemp;
```
- EN: Declares `copy`, packaging target-specific state and APIs around `MipsSEISelLowering`.
- CN: 这里声明 `copy`，把与 `MipsSEISelLowering` 相关的目标特定状态和 API 组织在一起。

### Lines 3817-3831
```cpp
  if (IsFGR64onMips32) {
    Register Rtemp2 = RegInfo.createVirtualRegister(GPRRC);
    BuildMI(*BB, MI, DL, TII->get(Mips::MFHC1_D64), Rtemp2).addReg(Fs);
    Register Wtemp2 = RegInfo.createVirtualRegister(&Mips::MSA128WRegClass);
    Register Wtemp3 = RegInfo.createVirtualRegister(&Mips::MSA128WRegClass);
    BuildMI(*BB, MI, DL, TII->get(Mips::INSERT_W), Wtemp2)
        .addReg(Wtemp)
        .addReg(Rtemp2)
        .addImm(1);
    BuildMI(*BB, MI, DL, TII->get(Mips::INSERT_W), Wtemp3)
        .addReg(Wtemp2)
        .addReg(Rtemp2)
        .addImm(3);
    WPHI = Wtemp3;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3833-3839
```cpp
  if (IsFGR64) {
    Register Wtemp2 = RegInfo.createVirtualRegister(&Mips::MSA128WRegClass);
    BuildMI(*BB, MI, DL, TII->get(Mips::FEXDO_W), Wtemp2)
        .addReg(WPHI)
        .addReg(WPHI);
    WPHI = Wtemp2;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3841-3841
```cpp
  BuildMI(*BB, MI, DL, TII->get(Mips::FEXDO_H), Wd).addReg(WPHI).addReg(WPHI);
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3843-3845
```cpp
  MI.eraseFromParent();
  return BB;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3847-3861
```cpp
// Emit the FPEXTEND_PSEUDO instruction.
//
// Expand an f16 to either a FGR32Opnd or FGR64Opnd.
//
// Safety: Cycle the result through the GPRs so the result always ends up
//         the correct floating point register.
//
// FIXME: This copying is strictly unnecessary. If we could tie FGR32Opnd:$Fd
//        / FGR64Opnd:$Fd and MSA128F16:$Ws to the same physical register
//        (which they can be, as the MSA registers are defined to alias the
//        FPU's 64 bit and 32 bit registers) the result can be accessed using
//        the correct register class. That requires operands be tie-able across
//        register classes which have a sub/super register class relationship. I
//        haven't checked.
//
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 3862-3876
```cpp
// For FGR32Opnd:
//
// FPEXTEND FGR32Opnd:$fd, MSA128F16:$ws
// =>
//  fexupr.w $wtemp, $ws
//  copy_s.w $rtemp, $ws[0]
//  mtc1 $rtemp, $fd
//
// For FGR64Opnd on Mips64:
//
// FPEXTEND FGR64Opnd:$fd, MSA128F16:$ws
// =>
//  fexupr.w $wtemp, $ws
//  fexupr.d $wtemp2, $wtemp
//  copy_s.d $rtemp, $wtemp2s[0]
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 3877-3891
```cpp
//  dmtc1 $rtemp, $fd
//
// For FGR64Opnd on Mips32:
//
// FPEXTEND FGR64Opnd:$fd, MSA128F16:$ws
// =>
//  fexupr.w $wtemp, $ws
//  fexupr.d $wtemp2, $wtemp
//  copy_s.w $rtemp, $wtemp2[0]
//  mtc1 $rtemp, $ftemp
//  copy_s.w $rtemp2, $wtemp2[1]
//  $fd = mthc1 $rtemp2, $ftemp
MachineBasicBlock *
MipsSETargetLowering::emitFPEXTEND_PSEUDO(MachineInstr &MI,
                                          MachineBasicBlock *BB,
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3892-3892
```cpp
                                          bool IsFGR64) const {
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3894-3897
```cpp
  // Strictly speaking, we need MIPS32R5 to support MSA. We'll be generous
  // here. It's technically doable to support MIPS32 here, but the ISA forbids
  // it.
  assert(Subtarget.hasMSA() && Subtarget.hasMips32r2());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3899-3900
```cpp
  bool IsFGR64onMips64 = Subtarget.hasMips64() && IsFGR64;
  bool IsFGR64onMips32 = !Subtarget.hasMips64() && IsFGR64;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3902-3905
```cpp
  const TargetInstrInfo *TII = Subtarget.getInstrInfo();
  DebugLoc DL = MI.getDebugLoc();
  Register Fd = MI.getOperand(0).getReg();
  Register Ws = MI.getOperand(1).getReg();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3907-3913
```cpp
  MachineRegisterInfo &RegInfo = BB->getParent()->getRegInfo();
  const TargetRegisterClass *GPRRC =
      IsFGR64onMips64 ? &Mips::GPR64RegClass : &Mips::GPR32RegClass;
  unsigned MTC1Opc = IsFGR64onMips64
                         ? Mips::DMTC1
                         : (IsFGR64onMips32 ? Mips::MTC1_D64 : Mips::MTC1);
  Register COPYOpc = IsFGR64onMips64 ? Mips::COPY_S_D : Mips::COPY_S_W;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3915-3916
```cpp
  Register Wtemp = RegInfo.createVirtualRegister(&Mips::MSA128WRegClass);
  Register WPHI = Wtemp;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3918-3922
```cpp
  BuildMI(*BB, MI, DL, TII->get(Mips::FEXUPR_W), Wtemp).addReg(Ws);
  if (IsFGR64) {
    WPHI = RegInfo.createVirtualRegister(&Mips::MSA128DRegClass);
    BuildMI(*BB, MI, DL, TII->get(Mips::FEXUPR_D), WPHI).addReg(Wtemp);
  }
```
- EN: Implements `BuildMI`, a mutation/build routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `BuildMI`，它是一个围绕寄存器管理展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3924-3930
```cpp
  // Perform the safety regclass copy mentioned above.
  Register Rtemp = RegInfo.createVirtualRegister(GPRRC);
  Register FPRPHI = IsFGR64onMips32
                        ? RegInfo.createVirtualRegister(&Mips::FGR64RegClass)
                        : Fd;
  BuildMI(*BB, MI, DL, TII->get(COPYOpc), Rtemp).addReg(WPHI).addImm(0);
  BuildMI(*BB, MI, DL, TII->get(MTC1Opc), FPRPHI).addReg(Rtemp);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3932-3940
```cpp
  if (IsFGR64onMips32) {
    Register Rtemp2 = RegInfo.createVirtualRegister(GPRRC);
    BuildMI(*BB, MI, DL, TII->get(Mips::COPY_S_W), Rtemp2)
        .addReg(WPHI)
        .addImm(1);
    BuildMI(*BB, MI, DL, TII->get(Mips::MTHC1_D64), Fd)
        .addReg(FPRPHI)
        .addReg(Rtemp2);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3942-3944
```cpp
  MI.eraseFromParent();
  return BB;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3946-3960
```cpp
// Emit the FEXP2_W_1 pseudo instructions.
//
// fexp2_w_1_pseudo $wd, $wt
// =>
// ldi.w $ws, 1
// fexp2.w $wd, $ws, $wt
MachineBasicBlock *
MipsSETargetLowering::emitFEXP2_W_1(MachineInstr &MI,
                                    MachineBasicBlock *BB) const {
  const TargetInstrInfo *TII = Subtarget.getInstrInfo();
  MachineRegisterInfo &RegInfo = BB->getParent()->getRegInfo();
  const TargetRegisterClass *RC = &Mips::MSA128WRegClass;
  Register Ws1 = RegInfo.createVirtualRegister(RC);
  Register Ws2 = RegInfo.createVirtualRegister(RC);
  DebugLoc DL = MI.getDebugLoc();
```
- EN: Implements `MipsSETargetLowering::emitFEXP2_W_1`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSETargetLowering::emitFEXP2_W_1`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3962-3964
```cpp
  // Splat 1.0 into a vector
  BuildMI(*BB, MI, DL, TII->get(Mips::LDI_W), Ws1).addImm(1);
  BuildMI(*BB, MI, DL, TII->get(Mips::FFINT_U_W), Ws2).addReg(Ws1);
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3966-3969
```cpp
  // Emit 1.0 * fexp2(Wt)
  BuildMI(*BB, MI, DL, TII->get(Mips::FEXP2_W), MI.getOperand(0).getReg())
      .addReg(Ws2)
      .addReg(MI.getOperand(1).getReg());
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3971-3973
```cpp
  MI.eraseFromParent(); // The pseudo instruction is gone now.
  return BB;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3975-3989
```cpp
// Emit the FEXP2_D_1 pseudo instructions.
//
// fexp2_d_1_pseudo $wd, $wt
// =>
// ldi.d $ws, 1
// fexp2.d $wd, $ws, $wt
MachineBasicBlock *
MipsSETargetLowering::emitFEXP2_D_1(MachineInstr &MI,
                                    MachineBasicBlock *BB) const {
  const TargetInstrInfo *TII = Subtarget.getInstrInfo();
  MachineRegisterInfo &RegInfo = BB->getParent()->getRegInfo();
  const TargetRegisterClass *RC = &Mips::MSA128DRegClass;
  Register Ws1 = RegInfo.createVirtualRegister(RC);
  Register Ws2 = RegInfo.createVirtualRegister(RC);
  DebugLoc DL = MI.getDebugLoc();
```
- EN: Implements `MipsSETargetLowering::emitFEXP2_D_1`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSETargetLowering::emitFEXP2_D_1`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3991-3993
```cpp
  // Splat 1.0 into a vector
  BuildMI(*BB, MI, DL, TII->get(Mips::LDI_D), Ws1).addImm(1);
  BuildMI(*BB, MI, DL, TII->get(Mips::FFINT_U_D), Ws2).addReg(Ws1);
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3995-3998
```cpp
  // Emit 1.0 * fexp2(Wt)
  BuildMI(*BB, MI, DL, TII->get(Mips::FEXP2_D), MI.getOperand(0).getReg())
      .addReg(Ws2)
      .addReg(MI.getOperand(1).getReg());
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4000-4002
```cpp
  MI.eraseFromParent(); // The pseudo instruction is gone now.
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

- EN: Backend-local headers: `MipsSEISelLowering.h`, `MipsMachineFunction.h`, `MipsRegisterInfo.h`, `MipsSubtarget.h`.
  - CN: 后端本地头文件：`MipsSEISelLowering.h`, `MipsMachineFunction.h`, `MipsRegisterInfo.h`, `MipsSubtarget.h`。
- EN: LLVM infrastructure headers: `llvm/ADT/APInt.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVector.h`, `llvm/CodeGen/CallingConvLower.h`, `llvm/CodeGen/ISDOpcodes.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstr.h` ... (+19 more).
  - CN: LLVM 基础设施头文件：`llvm/ADT/APInt.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVector.h`, `llvm/CodeGen/CallingConvLower.h`, `llvm/CodeGen/ISDOpcodes.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstr.h` ... (+19 more)。
- EN: Standard/system headers: `algorithm`, `cassert`, `cstddef`, `cstdint`, `iterator`, `utility`.
  - CN: 标准库/系统头文件：`algorithm`, `cassert`, `cstddef`, `cstdint`, `iterator`, `utility`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
