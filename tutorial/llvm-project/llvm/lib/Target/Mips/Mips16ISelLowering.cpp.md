# Mips16ISelLowering.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/Mips16ISelLowering.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `Mips16ISelLowering` for the Mips backend, focusing on SelectionDAG lowering and target-lowering rules.
- 用途 (CN): 实现 Mips 后端中的 `Mips16ISelLowering`，重点处理SelectionDAG 降级与目标降级规则。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```cpp
//===-- Mips16ISelLowering.h - Mips16 DAG Lowering Interface ----*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Subclass of MipsTargetLowering specialized for mips16.
//
//===----------------------------------------------------------------------===//
#include "Mips16ISelLowering.h"
#include "MCTargetDesc/MipsBaseInfo.h"
#include "Mips16HardFloatInfo.h"
#include "MipsMachineFunction.h"
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 16-20
```cpp
#include "MipsRegisterInfo.h"
#include "MipsTargetMachine.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/TargetInstrInfo.h"
#include "llvm/Support/CommandLine.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 22-22
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 24-24
```cpp
#define DEBUG_TYPE "mips-lower"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 26-31
```cpp
static cl::opt<bool> DontExpandCondPseudos16(
  "mips16-dont-expand-cond-pseudo",
  cl::init(false),
  cl::desc("Don't expand conditional move related "
           "pseudos for Mips 16"),
  cl::Hidden);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 33-36
```cpp
namespace {
struct Mips16IntrinsicHelperType{
  const char* Name;
  const char* Helper;
```
- EN: Declares `Mips16IntrinsicHelperType`, packaging target-specific state and APIs around `Mips16ISelLowering`.
- CN: 这里声明 `Mips16IntrinsicHelperType`，把与 `Mips16ISelLowering` 相关的目标特定状态和 API 组织在一起。

### Lines 38-45
```cpp
  bool operator<(const Mips16IntrinsicHelperType &RHS) const {
    return std::strcmp(Name, RHS.Name) < 0;
  }
  bool operator==(const Mips16IntrinsicHelperType &RHS) const {
    return std::strcmp(Name, RHS.Name) == 0;
  }
};
} // namespace
```
- EN: Implements `std::strcmp`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `std::strcmp`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 47-61
```cpp
static const Mips16IntrinsicHelperType Mips16IntrinsicHelper[] = {
  {"__fixunsdfsi", "__mips16_call_stub_2" },
  {"ceil",  "__mips16_call_stub_df_2"},
  {"ceilf", "__mips16_call_stub_sf_1"},
  {"copysign",  "__mips16_call_stub_df_10"},
  {"copysignf", "__mips16_call_stub_sf_5"},
  {"cos",  "__mips16_call_stub_df_2"},
  {"cosf", "__mips16_call_stub_sf_1"},
  {"exp2",  "__mips16_call_stub_df_2"},
  {"exp2f", "__mips16_call_stub_sf_1"},
  {"floor",  "__mips16_call_stub_df_2"},
  {"floorf", "__mips16_call_stub_sf_1"},
  {"log2",  "__mips16_call_stub_df_2"},
  {"log2f", "__mips16_call_stub_sf_1"},
  {"nearbyint",  "__mips16_call_stub_df_2"},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 62-71
```cpp
  {"nearbyintf", "__mips16_call_stub_sf_1"},
  {"rint",  "__mips16_call_stub_df_2"},
  {"rintf", "__mips16_call_stub_sf_1"},
  {"sin",  "__mips16_call_stub_df_2"},
  {"sinf", "__mips16_call_stub_sf_1"},
  {"sqrt",  "__mips16_call_stub_df_2"},
  {"sqrtf", "__mips16_call_stub_sf_1"},
  {"trunc",  "__mips16_call_stub_df_2"},
  {"truncf", "__mips16_call_stub_sf_1"},
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 73-75
```cpp
Mips16TargetLowering::Mips16TargetLowering(const MipsTargetMachine &TM,
                                           const MipsSubtarget &STI)
    : MipsTargetLowering(TM, STI) {
```
- EN: Implements `Mips16TargetLowering::Mips16TargetLowering`, a lowering routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips16TargetLowering::Mips16TargetLowering`，它是一个围绕目标机器策略展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 77-78
```cpp
  // Set up the register classes
  addRegisterClass(MVT::i32, &Mips::CPU16RegsRegClass);
```
- EN: Declares `addRegisterClass`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `addRegisterClass`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 80-92
```cpp
  setOperationAction(ISD::ATOMIC_FENCE, MVT::Other, LibCall);
  setOperationAction(ISD::ATOMIC_CMP_SWAP, MVT::i32, LibCall);
  setOperationAction(ISD::ATOMIC_SWAP, MVT::i32, LibCall);
  setOperationAction(ISD::ATOMIC_LOAD_ADD, MVT::i32, LibCall);
  setOperationAction(ISD::ATOMIC_LOAD_SUB, MVT::i32, LibCall);
  setOperationAction(ISD::ATOMIC_LOAD_AND, MVT::i32, LibCall);
  setOperationAction(ISD::ATOMIC_LOAD_OR, MVT::i32, LibCall);
  setOperationAction(ISD::ATOMIC_LOAD_XOR, MVT::i32, LibCall);
  setOperationAction(ISD::ATOMIC_LOAD_NAND, MVT::i32, LibCall);
  setOperationAction(ISD::ATOMIC_LOAD_MIN, MVT::i32, LibCall);
  setOperationAction(ISD::ATOMIC_LOAD_MAX, MVT::i32, LibCall);
  setOperationAction(ISD::ATOMIC_LOAD_UMIN, MVT::i32, LibCall);
  setOperationAction(ISD::ATOMIC_LOAD_UMAX, MVT::i32, LibCall);
```
- EN: Declares `setOperationAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 94-97
```cpp
  setOperationAction(ISD::ROTR, MVT::i32,  Expand);
  setOperationAction(ISD::ROTR, MVT::i64,  Expand);
  setOperationAction(ISD::BSWAP, MVT::i32, Expand);
  setOperationAction(ISD::BSWAP, MVT::i64, Expand);
```
- EN: Declares `setOperationAction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setOperationAction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 99-100
```cpp
  computeRegisterProperties(STI.getRegisterInfo());
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 102-106
```cpp
const MipsTargetLowering *
llvm::createMips16TargetLowering(const MipsTargetMachine &TM,
                                 const MipsSubtarget &STI) {
  return new Mips16TargetLowering(TM, STI);
}
```
- EN: Implements `llvm::createMips16TargetLowering`, a lowering routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::createMips16TargetLowering`，它是一个围绕目标机器策略展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 108-111
```cpp
bool Mips16TargetLowering::allowsMisalignedMemoryAccesses(
    EVT VT, unsigned, Align, MachineMemOperand::Flags, unsigned *Fast) const {
  return false;
}
```
- EN: Implements `Mips16TargetLowering::allowsMisalignedMemoryAccesses`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips16TargetLowering::allowsMisalignedMemoryAccesses`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 113-127
```cpp
MachineBasicBlock *
Mips16TargetLowering::EmitInstrWithCustomInserter(MachineInstr &MI,
                                                  MachineBasicBlock *BB) const {
  switch (MI.getOpcode()) {
  default:
    return MipsTargetLowering::EmitInstrWithCustomInserter(MI, BB);
  case Mips::SelBeqZ:
    return emitSel16(Mips::BeqzRxImm16, MI, BB);
  case Mips::SelBneZ:
    return emitSel16(Mips::BnezRxImm16, MI, BB);
  case Mips::SelTBteqZCmpi:
    return emitSeliT16(Mips::Bteqz16, Mips::CmpiRxImmX16, MI, BB);
  case Mips::SelTBteqZSlti:
    return emitSeliT16(Mips::Bteqz16, Mips::SltiRxImmX16, MI, BB);
  case Mips::SelTBteqZSltiu:
```
- EN: Implements `Mips16TargetLowering::EmitInstrWithCustomInserter`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips16TargetLowering::EmitInstrWithCustomInserter`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 128-142
```cpp
    return emitSeliT16(Mips::Bteqz16, Mips::SltiuRxImmX16, MI, BB);
  case Mips::SelTBtneZCmpi:
    return emitSeliT16(Mips::Btnez16, Mips::CmpiRxImmX16, MI, BB);
  case Mips::SelTBtneZSlti:
    return emitSeliT16(Mips::Btnez16, Mips::SltiRxImmX16, MI, BB);
  case Mips::SelTBtneZSltiu:
    return emitSeliT16(Mips::Btnez16, Mips::SltiuRxImmX16, MI, BB);
  case Mips::SelTBteqZCmp:
    return emitSelT16(Mips::Bteqz16, Mips::CmpRxRy16, MI, BB);
  case Mips::SelTBteqZSlt:
    return emitSelT16(Mips::Bteqz16, Mips::SltRxRy16, MI, BB);
  case Mips::SelTBteqZSltu:
    return emitSelT16(Mips::Bteqz16, Mips::SltuRxRy16, MI, BB);
  case Mips::SelTBtneZCmp:
    return emitSelT16(Mips::Btnez16, Mips::CmpRxRy16, MI, BB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 143-157
```cpp
  case Mips::SelTBtneZSlt:
    return emitSelT16(Mips::Btnez16, Mips::SltRxRy16, MI, BB);
  case Mips::SelTBtneZSltu:
    return emitSelT16(Mips::Btnez16, Mips::SltuRxRy16, MI, BB);
  case Mips::BteqzT8CmpX16:
    return emitFEXT_T8I816_ins(Mips::Bteqz16, Mips::CmpRxRy16, MI, BB);
  case Mips::BteqzT8SltX16:
    return emitFEXT_T8I816_ins(Mips::Bteqz16, Mips::SltRxRy16, MI, BB);
  case Mips::BteqzT8SltuX16:
    // TBD: figure out a way to get this or remove the instruction
    // altogether.
    return emitFEXT_T8I816_ins(Mips::Bteqz16, Mips::SltuRxRy16, MI, BB);
  case Mips::BtnezT8CmpX16:
    return emitFEXT_T8I816_ins(Mips::Btnez16, Mips::CmpRxRy16, MI, BB);
  case Mips::BtnezT8SltX16:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 158-172
```cpp
    return emitFEXT_T8I816_ins(Mips::Btnez16, Mips::SltRxRy16, MI, BB);
  case Mips::BtnezT8SltuX16:
    // TBD: figure out a way to get this or remove the instruction
    // altogether.
    return emitFEXT_T8I816_ins(Mips::Btnez16, Mips::SltuRxRy16, MI, BB);
  case Mips::BteqzT8CmpiX16: return emitFEXT_T8I8I16_ins(
    Mips::Bteqz16, Mips::CmpiRxImm16, Mips::CmpiRxImmX16, false, MI, BB);
  case Mips::BteqzT8SltiX16: return emitFEXT_T8I8I16_ins(
    Mips::Bteqz16, Mips::SltiRxImm16, Mips::SltiRxImmX16, true, MI, BB);
  case Mips::BteqzT8SltiuX16: return emitFEXT_T8I8I16_ins(
    Mips::Bteqz16, Mips::SltiuRxImm16, Mips::SltiuRxImmX16, false, MI, BB);
  case Mips::BtnezT8CmpiX16: return emitFEXT_T8I8I16_ins(
    Mips::Btnez16, Mips::CmpiRxImm16, Mips::CmpiRxImmX16, false, MI, BB);
  case Mips::BtnezT8SltiX16: return emitFEXT_T8I8I16_ins(
    Mips::Btnez16, Mips::SltiRxImm16, Mips::SltiRxImmX16, true, MI, BB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 173-187
```cpp
  case Mips::BtnezT8SltiuX16: return emitFEXT_T8I8I16_ins(
    Mips::Btnez16, Mips::SltiuRxImm16, Mips::SltiuRxImmX16, false, MI, BB);
    break;
  case Mips::SltCCRxRy16:
    return emitFEXT_CCRX16_ins(Mips::SltRxRy16, MI, BB);
    break;
  case Mips::SltiCCRxImmX16:
    return emitFEXT_CCRXI16_ins
      (Mips::SltiRxImm16, Mips::SltiRxImmX16, MI, BB);
  case Mips::SltiuCCRxImmX16:
    return emitFEXT_CCRXI16_ins
      (Mips::SltiuRxImm16, Mips::SltiuRxImmX16, MI, BB);
  case Mips::SltuCCRxRy16:
    return emitFEXT_CCRX16_ins
      (Mips::SltuRxRy16, MI, BB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 188-189
```cpp
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 191-196
```cpp
bool Mips16TargetLowering::isEligibleForTailCallOptimization(
    const CCState &CCInfo, unsigned NextStackOffset,
    const MipsFunctionInfo &FI) const {
  // No tail call optimization for mips16.
  return false;
}
```
- EN: Implements `Mips16TargetLowering::isEligibleForTailCallOptimization`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips16TargetLowering::isEligibleForTailCallOptimization`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 198-212
```cpp
//
// The Mips16 hard float is a crazy quilt inherited from gcc. I have a much
// cleaner way to do all of this but it will have to wait until the traditional
// gcc mechanism is completed.
//
// For Pic, in order for Mips16 code to call Mips32 code which according the abi
// have either arguments or returned values placed in floating point registers,
// we use a set of helper functions. (This includes functions which return type
//  complex which on Mips are returned in a pair of floating point registers).
//
// This is an encoding that we inherited from gcc.
// In Mips traditional O32, N32 ABI, floating point numbers are passed in
// floating point argument registers 1,2 only when the first and optionally
// the second arguments are float (sf) or double (df).
// For Mips16 we are only concerned with the situations where floating point
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 213-227
```cpp
// arguments are being passed in floating point registers by the ABI, because
// Mips16 mode code cannot execute floating point instructions to load those
// values and hence helper functions are needed.
// The possibilities are (), (sf), (sf, sf), (sf, df), (df), (df, sf), (df, df)
// the helper function suffixs for these are:
//                        0,  1,    5,        9,         2,   6,        10
// this suffix can then be calculated as follows:
// for a given argument Arg:
//     Arg1x, Arg2x = 1 :  Arg is sf
//                    2 :  Arg is df
//                    0:   Arg is neither sf or df
// So this stub is the string for number Arg1x + Arg2x*4.
// However not all numbers between 0 and 10 are possible, we check anyway and
// assert if the impossible exists.
//
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 229-243
```cpp
unsigned int Mips16TargetLowering::getMips16HelperFunctionStubNumber
  (ArgListTy &Args) const {
  unsigned int resultNum = 0;
  if (Args.size() >= 1) {
    Type *t = Args[0].Ty;
    if (t->isFloatTy()) {
      resultNum = 1;
    }
    else if (t->isDoubleTy()) {
      resultNum = 2;
    }
  }
  if (resultNum) {
    if (Args.size() >=2) {
      Type *t = Args[1].Ty;
```
- EN: Implements `Mips16TargetLowering::getMips16HelperFunctionStubNumber`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips16TargetLowering::getMips16HelperFunctionStubNumber`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 244-253
```cpp
      if (t->isFloatTy()) {
        resultNum += 4;
      }
      else if (t->isDoubleTy()) {
        resultNum += 8;
      }
    }
  }
  return resultNum;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 255-269
```cpp
//
// Prefixes are attached to stub numbers depending on the return type.
// return type: float  sf_
//              double df_
//              single complex sc_
//              double complext dc_
//              others  NO PREFIX
//
//
// The full name of a helper function is__mips16_call_stub +
//    return type dependent prefix + stub number
//
// FIXME: This is something that probably should be in a different source file
// and perhaps done differently but my main purpose is to not waste runtime
// on something that we can enumerate in the source. Another possibility is
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 270-284
```cpp
// to have a python script to generate these mapping tables. This will do
// for now. There are a whole series of helper function mapping arrays, one
// for each return type class as outlined above. There there are 11 possible
// entries. Ones with 0 are ones which should never be selected.
//
// All the arrays are similar except for ones which return neither
// sf, df, sc, dc, in which we only care about ones which have sf or df as a
// first parameter.
//
#define P_ "__mips16_call_stub_"
#define MAX_STUB_NUMBER 10
#define T1 P "1", P "2", 0, 0, P "5", P "6", 0, 0, P "9", P "10"
#define T P "0" , T1
#define P P_
static char const * vMips16Helper[MAX_STUB_NUMBER+1] =
```
- EN: Declares `as`, packaging target-specific state and APIs around `Mips16ISelLowering`.
- CN: 这里声明 `as`，把与 `Mips16ISelLowering` 相关的目标特定状态和 API 组织在一起。

### Lines 285-299
```cpp
  {nullptr, T1 };
#undef P
#define P P_ "sf_"
static char const * sfMips16Helper[MAX_STUB_NUMBER+1] =
  { T };
#undef P
#define P P_ "df_"
static char const * dfMips16Helper[MAX_STUB_NUMBER+1] =
  { T };
#undef P
#define P P_ "sc_"
static char const * scMips16Helper[MAX_STUB_NUMBER+1] =
  { T };
#undef P
#define P P_ "dc_"
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 300-303
```cpp
static char const * dcMips16Helper[MAX_STUB_NUMBER+1] =
  { T };
#undef P
#undef P_
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 306-320
```cpp
const char* Mips16TargetLowering::
  getMips16HelperFunction
    (Type* RetTy, ArgListTy &Args, bool &needHelper) const {
  const unsigned int stubNum = getMips16HelperFunctionStubNumber(Args);
#ifndef NDEBUG
  const unsigned int maxStubNum = 10;
  assert(stubNum <= maxStubNum);
  const bool validStubNum[maxStubNum+1] =
    {true, true, true, false, false, true, true, false, false, true, true};
  assert(validStubNum[stubNum]);
#endif
  const char *result;
  if (RetTy->isFloatTy()) {
    result = sfMips16Helper[stubNum];
  }
```
- EN: Implements `getMips16HelperFunction`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getMips16HelperFunction`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 321-335
```cpp
  else if (RetTy ->isDoubleTy()) {
    result = dfMips16Helper[stubNum];
  } else if (StructType *SRetTy = dyn_cast<StructType>(RetTy)) {
    // check if it's complex
    if (SRetTy->getNumElements() == 2) {
      if ((SRetTy->getElementType(0)->isFloatTy()) &&
          (SRetTy->getElementType(1)->isFloatTy())) {
        result = scMips16Helper[stubNum];
      } else if ((SRetTy->getElementType(0)->isDoubleTy()) &&
                 (SRetTy->getElementType(1)->isDoubleTy())) {
        result = dcMips16Helper[stubNum];
      } else {
        llvm_unreachable("Uncovered condition");
      }
    } else {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 336-347
```cpp
      llvm_unreachable("Uncovered condition");
    }
  } else {
    if (stubNum == 0) {
      needHelper = false;
      return "";
    }
    result = vMips16Helper[stubNum];
  }
  needHelper = true;
  return result;
}
```
- EN: Implements `llvm_unreachable`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm_unreachable`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 349-356
```cpp
static bool isMips16HardFloatLibcall(StringRef Name) {
  // FIXME: Use getSupportedLibcallImpl instead of blindly parsing the name.
  iota_range<RTLIB::LibcallImpl> ParsedLibcalls =
      RTLIB::RuntimeLibcallsInfo::lookupLibcallImplName(Name);
  return !ParsedLibcalls.empty() &&
         binary_search(MipsSubtarget::HardFloatLibCalls,
                       *ParsedLibcalls.begin());
}
```
- EN: Implements `isMips16HardFloatLibcall`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isMips16HardFloatLibcall`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 358-368
```cpp
void Mips16TargetLowering::
getOpndList(SmallVectorImpl<SDValue> &Ops,
            std::deque< std::pair<unsigned, SDValue> > &RegsToPass,
            bool IsPICCall, bool GlobalOrExternal, bool InternalLinkage,
            bool IsCallReloc, CallLoweringInfo &CLI, SDValue Callee,
            SDValue Chain) const {
  SelectionDAG &DAG = CLI.DAG;
  MachineFunction &MF = DAG.getMachineFunction();
  MipsFunctionInfo *FuncInfo = MF.getInfo<MipsFunctionInfo>();
  const char* Mips16HelperFunction = nullptr;
  bool NeedMips16Helper = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 370-384
```cpp
  if (Subtarget.inMips16HardFloat()) {
    //
    // currently we don't have symbols tagged with the mips16 or mips32
    // qualifier so we will assume that we don't know what kind it is.
    // and generate the helper
    //
    bool LookupHelper = true;
    if (ExternalSymbolSDNode *S = dyn_cast<ExternalSymbolSDNode>(CLI.Callee)) {
      if (isMips16HardFloatLibcall(S->getSymbol()))
        LookupHelper = false;
      else {
        const char *Symbol = S->getSymbol();
        Mips16IntrinsicHelperType IntrinsicFind = { Symbol, "" };
        const Mips16HardFloatInfo::FuncSignature *Signature =
            Mips16HardFloatInfo::findFuncSignature(Symbol);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 385-399
```cpp
        if (!IsPICCall && Signature &&
            FuncInfo->StubsNeeded.try_emplace(Symbol, Signature).second) {
          //
          // S2 is normally saved if the stub is for a function which
          // returns a float or double value and is not otherwise. This is
          // because more work is required after the function the stub
          // is calling completes, and so the stub cannot directly return
          // and the stub has no stack space to store the return address so
          // S2 is used for that purpose.
          // In order to take advantage of not saving S2, we need to also
          // optimize the call in the stub and this requires some further
          // functionality in MipsAsmPrinter which we don't have yet.
          // So for now we always save S2. The optimization will be done
          // in a follow-on patch.
          //
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 400-411
```cpp
          if (true || (Signature->RetSig != Mips16HardFloatInfo::NoFPRet))
            FuncInfo->setSaveS2();
        }
        // one more look at list of intrinsics
        const Mips16IntrinsicHelperType *Helper =
            llvm::lower_bound(Mips16IntrinsicHelper, IntrinsicFind);
        if (Helper != std::end(Mips16IntrinsicHelper) &&
            *Helper == IntrinsicFind) {
          Mips16HelperFunction = Helper->Helper;
          NeedMips16Helper = true;
          LookupHelper = false;
        }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 413-415
```cpp
      }
    } else if (GlobalAddressSDNode *G =
                   dyn_cast<GlobalAddressSDNode>(CLI.Callee)) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 417-423
```cpp
      if (isMips16HardFloatLibcall(G->getGlobal()->getName()))
        LookupHelper = false;
    }
    if (LookupHelper)
      Mips16HelperFunction =
        getMips16HelperFunction(CLI.RetTy, CLI.getArgs(), NeedMips16Helper);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 425-425
```cpp
  SDValue JumpTarget = Callee;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 427-441
```cpp
  // T9 should contain the address of the callee function if
  // -relocation-model=pic or it is an indirect call.
  if (IsPICCall || !GlobalOrExternal) {
    unsigned V0Reg = Mips::V0;
    if (NeedMips16Helper) {
      RegsToPass.push_front(std::make_pair(V0Reg, Callee));
      JumpTarget = DAG.getExternalSymbol(Mips16HelperFunction,
                                         getPointerTy(DAG.getDataLayout()));
      ExternalSymbolSDNode *S = cast<ExternalSymbolSDNode>(JumpTarget);
      JumpTarget = getAddrGlobal(S, CLI.DL, JumpTarget.getValueType(), DAG,
                                 MipsII::MO_GOT, Chain,
                                 FuncInfo->callPtrInfo(MF, S->getSymbol()));
    } else
      RegsToPass.push_front(std::make_pair((unsigned)Mips::T9, Callee));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 443-443
```cpp
  Ops.push_back(JumpTarget);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 445-448
```cpp
  MipsTargetLowering::getOpndList(Ops, RegsToPass, IsPICCall, GlobalOrExternal,
                                  InternalLinkage, IsCallReloc, CLI, Callee,
                                  Chain);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 450-462
```cpp
MachineBasicBlock *
Mips16TargetLowering::emitSel16(unsigned Opc, MachineInstr &MI,
                                MachineBasicBlock *BB) const {
  if (DontExpandCondPseudos16)
    return BB;
  const TargetInstrInfo *TII = Subtarget.getInstrInfo();
  DebugLoc DL = MI.getDebugLoc();
  // To "insert" a SELECT_CC instruction, we actually have to insert the
  // diamond control-flow pattern.  The incoming instruction knows the
  // destination vreg to set, the condition code register to branch on, the
  // true/false values to select between, and a branch opcode to use.
  const BasicBlock *LLVM_BB = BB->getBasicBlock();
  MachineFunction::iterator It = ++BB->getIterator();
```
- EN: Implements `Mips16TargetLowering::emitSel16`, a emission/printing routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips16TargetLowering::emitSel16`，它是一个围绕机器函数状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 464-475
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

### Lines 477-480
```cpp
  // Transfer the remainder of BB and its successor edges to sinkMBB.
  sinkMBB->splice(sinkMBB->begin(), BB,
                  std::next(MachineBasicBlock::iterator(MI)), BB->end());
  sinkMBB->transferSuccessorsAndUpdatePHIs(BB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 482-484
```cpp
  // Next, add the true and fallthrough blocks as its successors.
  BB->addSuccessor(copy0MBB);
  BB->addSuccessor(sinkMBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 486-488
```cpp
  BuildMI(BB, DL, TII->get(Opc))
      .addReg(MI.getOperand(3).getReg())
      .addMBB(sinkMBB);
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 490-493
```cpp
  //  copy0MBB:
  //   %FalseValue = ...
  //   # fallthrough to sinkMBB
  BB = copy0MBB;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 495-496
```cpp
  // Update machine-CFG edges
  BB->addSuccessor(sinkMBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 498-501
```cpp
  //  sinkMBB:
  //   %Result = phi [ %TrueValue, thisMBB ], [ %FalseValue, copy0MBB ]
  //  ...
  BB = sinkMBB;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 503-507
```cpp
  BuildMI(*BB, BB->begin(), DL, TII->get(Mips::PHI), MI.getOperand(0).getReg())
      .addReg(MI.getOperand(1).getReg())
      .addMBB(thisMBB)
      .addReg(MI.getOperand(2).getReg())
      .addMBB(copy0MBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 509-511
```cpp
  MI.eraseFromParent(); // The pseudo instruction is gone now.
  return BB;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 513-525
```cpp
MachineBasicBlock *
Mips16TargetLowering::emitSelT16(unsigned Opc1, unsigned Opc2, MachineInstr &MI,
                                 MachineBasicBlock *BB) const {
  if (DontExpandCondPseudos16)
    return BB;
  const TargetInstrInfo *TII = Subtarget.getInstrInfo();
  DebugLoc DL = MI.getDebugLoc();
  // To "insert" a SELECT_CC instruction, we actually have to insert the
  // diamond control-flow pattern.  The incoming instruction knows the
  // destination vreg to set, the condition code register to branch on, the
  // true/false values to select between, and a branch opcode to use.
  const BasicBlock *LLVM_BB = BB->getBasicBlock();
  MachineFunction::iterator It = ++BB->getIterator();
```
- EN: Implements `Mips16TargetLowering::emitSelT16`, a emission/printing routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips16TargetLowering::emitSelT16`，它是一个围绕机器函数状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 527-538
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

### Lines 540-543
```cpp
  // Transfer the remainder of BB and its successor edges to sinkMBB.
  sinkMBB->splice(sinkMBB->begin(), BB,
                  std::next(MachineBasicBlock::iterator(MI)), BB->end());
  sinkMBB->transferSuccessorsAndUpdatePHIs(BB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 545-547
```cpp
  // Next, add the true and fallthrough blocks as its successors.
  BB->addSuccessor(copy0MBB);
  BB->addSuccessor(sinkMBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 549-552
```cpp
  BuildMI(BB, DL, TII->get(Opc2))
      .addReg(MI.getOperand(3).getReg())
      .addReg(MI.getOperand(4).getReg());
  BuildMI(BB, DL, TII->get(Opc1)).addMBB(sinkMBB);
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 554-557
```cpp
  //  copy0MBB:
  //   %FalseValue = ...
  //   # fallthrough to sinkMBB
  BB = copy0MBB;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 559-560
```cpp
  // Update machine-CFG edges
  BB->addSuccessor(sinkMBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 562-565
```cpp
  //  sinkMBB:
  //   %Result = phi [ %TrueValue, thisMBB ], [ %FalseValue, copy0MBB ]
  //  ...
  BB = sinkMBB;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 567-571
```cpp
  BuildMI(*BB, BB->begin(), DL, TII->get(Mips::PHI), MI.getOperand(0).getReg())
      .addReg(MI.getOperand(1).getReg())
      .addMBB(thisMBB)
      .addReg(MI.getOperand(2).getReg())
      .addMBB(copy0MBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 573-574
```cpp
  MI.eraseFromParent(); // The pseudo instruction is gone now.
  return BB;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 576-576
```cpp
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 578-591
```cpp
MachineBasicBlock *
Mips16TargetLowering::emitSeliT16(unsigned Opc1, unsigned Opc2,
                                  MachineInstr &MI,
                                  MachineBasicBlock *BB) const {
  if (DontExpandCondPseudos16)
    return BB;
  const TargetInstrInfo *TII = Subtarget.getInstrInfo();
  DebugLoc DL = MI.getDebugLoc();
  // To "insert" a SELECT_CC instruction, we actually have to insert the
  // diamond control-flow pattern.  The incoming instruction knows the
  // destination vreg to set, the condition code register to branch on, the
  // true/false values to select between, and a branch opcode to use.
  const BasicBlock *LLVM_BB = BB->getBasicBlock();
  MachineFunction::iterator It = ++BB->getIterator();
```
- EN: Implements `Mips16TargetLowering::emitSeliT16`, a emission/printing routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips16TargetLowering::emitSeliT16`，它是一个围绕机器函数状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 593-604
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

### Lines 606-609
```cpp
  // Transfer the remainder of BB and its successor edges to sinkMBB.
  sinkMBB->splice(sinkMBB->begin(), BB,
                  std::next(MachineBasicBlock::iterator(MI)), BB->end());
  sinkMBB->transferSuccessorsAndUpdatePHIs(BB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 611-613
```cpp
  // Next, add the true and fallthrough blocks as its successors.
  BB->addSuccessor(copy0MBB);
  BB->addSuccessor(sinkMBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 615-618
```cpp
  BuildMI(BB, DL, TII->get(Opc2))
      .addReg(MI.getOperand(3).getReg())
      .addImm(MI.getOperand(4).getImm());
  BuildMI(BB, DL, TII->get(Opc1)).addMBB(sinkMBB);
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 620-623
```cpp
  //  copy0MBB:
  //   %FalseValue = ...
  //   # fallthrough to sinkMBB
  BB = copy0MBB;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 625-626
```cpp
  // Update machine-CFG edges
  BB->addSuccessor(sinkMBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 628-631
```cpp
  //  sinkMBB:
  //   %Result = phi [ %TrueValue, thisMBB ], [ %FalseValue, copy0MBB ]
  //  ...
  BB = sinkMBB;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 633-637
```cpp
  BuildMI(*BB, BB->begin(), DL, TII->get(Mips::PHI), MI.getOperand(0).getReg())
      .addReg(MI.getOperand(1).getReg())
      .addMBB(thisMBB)
      .addReg(MI.getOperand(2).getReg())
      .addMBB(copy0MBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 639-640
```cpp
  MI.eraseFromParent(); // The pseudo instruction is gone now.
  return BB;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 642-642
```cpp
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 644-658
```cpp
MachineBasicBlock *
Mips16TargetLowering::emitFEXT_T8I816_ins(unsigned BtOpc, unsigned CmpOpc,
                                          MachineInstr &MI,
                                          MachineBasicBlock *BB) const {
  if (DontExpandCondPseudos16)
    return BB;
  const TargetInstrInfo *TII = Subtarget.getInstrInfo();
  Register regX = MI.getOperand(0).getReg();
  Register regY = MI.getOperand(1).getReg();
  MachineBasicBlock *target = MI.getOperand(2).getMBB();
  BuildMI(*BB, MI, MI.getDebugLoc(), TII->get(CmpOpc))
      .addReg(regX)
      .addReg(regY);
  BuildMI(*BB, MI, MI.getDebugLoc(), TII->get(BtOpc)).addMBB(target);
  MI.eraseFromParent(); // The pseudo instruction is gone now.
```
- EN: Implements `Mips16TargetLowering::emitFEXT_T8I816_ins`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips16TargetLowering::emitFEXT_T8I816_ins`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 659-660
```cpp
  return BB;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 662-676
```cpp
MachineBasicBlock *Mips16TargetLowering::emitFEXT_T8I8I16_ins(
    unsigned BtOpc, unsigned CmpiOpc, unsigned CmpiXOpc, bool ImmSigned,
    MachineInstr &MI, MachineBasicBlock *BB) const {
  if (DontExpandCondPseudos16)
    return BB;
  const TargetInstrInfo *TII = Subtarget.getInstrInfo();
  Register regX = MI.getOperand(0).getReg();
  int64_t imm = MI.getOperand(1).getImm();
  MachineBasicBlock *target = MI.getOperand(2).getMBB();
  unsigned CmpOpc;
  if (isUInt<8>(imm))
    CmpOpc = CmpiOpc;
  else if ((!ImmSigned && isUInt<16>(imm)) ||
           (ImmSigned && isInt<16>(imm)))
    CmpOpc = CmpiXOpc;
```
- EN: Implements `Mips16TargetLowering::emitFEXT_T8I8I16_ins`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips16TargetLowering::emitFEXT_T8I8I16_ins`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 677-683
```cpp
  else
    llvm_unreachable("immediate field not usable");
  BuildMI(*BB, MI, MI.getDebugLoc(), TII->get(CmpOpc)).addReg(regX).addImm(imm);
  BuildMI(*BB, MI, MI.getDebugLoc(), TII->get(BtOpc)).addMBB(target);
  MI.eraseFromParent(); // The pseudo instruction is gone now.
  return BB;
}
```
- EN: Declares `llvm_unreachable`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `llvm_unreachable`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 685-693
```cpp
static unsigned Mips16WhichOp8uOr16simm
  (unsigned shortOp, unsigned longOp, int64_t Imm) {
  if (isUInt<8>(Imm))
    return shortOp;
  else if (isInt<16>(Imm))
    return longOp;
  else
    llvm_unreachable("immediate field not usable");
}
```
- EN: Implements `Mips16WhichOp8uOr16simm`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips16WhichOp8uOr16simm`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 695-709
```cpp
MachineBasicBlock *
Mips16TargetLowering::emitFEXT_CCRX16_ins(unsigned SltOpc, MachineInstr &MI,
                                          MachineBasicBlock *BB) const {
  if (DontExpandCondPseudos16)
    return BB;
  const TargetInstrInfo *TII = Subtarget.getInstrInfo();
  Register CC = MI.getOperand(0).getReg();
  Register regX = MI.getOperand(1).getReg();
  Register regY = MI.getOperand(2).getReg();
  BuildMI(*BB, MI, MI.getDebugLoc(), TII->get(SltOpc))
      .addReg(regX)
      .addReg(regY);
  BuildMI(*BB, MI, MI.getDebugLoc(), TII->get(Mips::MoveR3216), CC)
      .addReg(Mips::T8);
  MI.eraseFromParent(); // The pseudo instruction is gone now.
```
- EN: Implements `Mips16TargetLowering::emitFEXT_CCRX16_ins`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips16TargetLowering::emitFEXT_CCRX16_ins`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 710-711
```cpp
  return BB;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 713-727
```cpp
MachineBasicBlock *
Mips16TargetLowering::emitFEXT_CCRXI16_ins(unsigned SltiOpc, unsigned SltiXOpc,
                                           MachineInstr &MI,
                                           MachineBasicBlock *BB) const {
  if (DontExpandCondPseudos16)
    return BB;
  const TargetInstrInfo *TII = Subtarget.getInstrInfo();
  Register CC = MI.getOperand(0).getReg();
  Register regX = MI.getOperand(1).getReg();
  int64_t Imm = MI.getOperand(2).getImm();
  unsigned SltOpc = Mips16WhichOp8uOr16simm(SltiOpc, SltiXOpc, Imm);
  BuildMI(*BB, MI, MI.getDebugLoc(), TII->get(SltOpc)).addReg(regX).addImm(Imm);
  BuildMI(*BB, MI, MI.getDebugLoc(), TII->get(Mips::MoveR3216), CC)
      .addReg(Mips::T8);
  MI.eraseFromParent(); // The pseudo instruction is gone now.
```
- EN: Implements `Mips16TargetLowering::emitFEXT_CCRXI16_ins`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips16TargetLowering::emitFEXT_CCRXI16_ins`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 728-728
```cpp
  return BB;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 730-730
```cpp
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

- EN: Backend-local headers: `Mips16ISelLowering.h`, `MCTargetDesc/MipsBaseInfo.h`, `Mips16HardFloatInfo.h`, `MipsMachineFunction.h`, `MipsRegisterInfo.h`, `MipsTargetMachine.h`.
  - CN: 后端本地头文件：`Mips16ISelLowering.h`, `MCTargetDesc/MipsBaseInfo.h`, `Mips16HardFloatInfo.h`, `MipsMachineFunction.h`, `MipsRegisterInfo.h`, `MipsTargetMachine.h`。
- EN: LLVM infrastructure headers: `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/TargetInstrInfo.h`, `llvm/Support/CommandLine.h`.
  - CN: LLVM 基础设施头文件：`llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/TargetInstrInfo.h`, `llvm/Support/CommandLine.h`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
