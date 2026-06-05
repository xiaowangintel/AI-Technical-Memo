# WebAssemblyCallLowering.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/GISel/WebAssemblyCallLowering.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file implements the lowering of LLVM calls to machine code calls for GlobalISel.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/GISel/WebAssemblyCallLowering.cpp`，主要负责 WebAssembly 后端的GlobalISel 调用约定 lowering 逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- WebAssemblyCallLowering.cpp - Call lowering for GlobalISel -*- C++ -*-//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file. This range participates in the GlobalISel pipeline.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。 这一段参与 GlobalISel 流水线。

### Lines 7-12

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// This file implements the lowering of LLVM calls to machine code calls for
/// GlobalISel.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file". This range participates in the GlobalISel pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。 这一段参与 GlobalISel 流水线。

### Lines 13-19

```cpp
//===----------------------------------------------------------------------===//

#include "WebAssemblyCallLowering.h"
#include "MCTargetDesc/WebAssemblyMCTargetDesc.h"
#include "Utils/WasmAddressSpaces.h"
#include "WebAssemblyISelLowering.h"
#include "WebAssemblyMachineFunctionInfo.h"
```
- **EN**: Pulls in direct dependencies required by this GlobalISel call lowering logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该GlobalISel 调用约定 lowering 逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 20-26

```cpp
#include "WebAssemblyRegisterInfo.h"
#include "WebAssemblySubtarget.h"
#include "WebAssemblyUtilities.h"
#include "llvm/CodeGen/Analysis.h"
#include "llvm/CodeGen/FunctionLoweringInfo.h"
#include "llvm/CodeGen/GlobalISel/MachineIRBuilder.h"
#include "llvm/CodeGen/LowLevelTypeUtils.h"
```
- **EN**: Pulls in direct dependencies required by this GlobalISel call lowering logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段引入该GlobalISel 调用约定 lowering 逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 27-33

```cpp
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/RegisterBankInfo.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/IR/DebugLoc.h"
#include "llvm/IR/Value.h"
#include "llvm/Support/ErrorHandling.h"
```
- **EN**: Pulls in direct dependencies required by this GlobalISel call lowering logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段引入该GlobalISel 调用约定 lowering 逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 34-71

```cpp
#define DEBUG_TYPE "wasm-call-lowering"

using namespace llvm;

// Test whether the given calling convention is supported.
static bool callingConvSupported(CallingConv::ID CallConv) {
  // We currently support the language-independent target-independent
  // conventions. We don't yet have a way to annotate calls with properties like
  // "cold", and we don't have any call-clobbered registers, so these are mostly
  // all handled the same.
  return CallConv == CallingConv::C || CallConv == CallingConv::Fast ||
         CallConv == CallingConv::Cold ||
         CallConv == CallingConv::PreserveMost ||
         CallConv == CallingConv::PreserveAll ||
         CallConv == CallingConv::CXX_FAST_TLS ||
         CallConv == CallingConv::WASM_EmscriptenInvoke ||
         CallConv == CallingConv::Swift;
}

static unsigned extendOpFromFlags(ISD::ArgFlagsTy Flags) {
  if (Flags.isSExt())
    return TargetOpcode::G_SEXT;
  if (Flags.isZExt())
    return TargetOpcode::G_ZEXT;
  return TargetOpcode::G_ANYEXT;
}

static LLT getLLTForWasmMVT(MVT Ty, const DataLayout &DL) {
  if (Ty == MVT::externref) {
    return LLT::pointer(
        WebAssembly::WasmAddressSpace::WASM_ADDRESS_SPACE_EXTERNREF,
        DL.getPointerSizeInBits(
            WebAssembly::WasmAddressSpace::WASM_ADDRESS_SPACE_EXTERNREF));
  }

  if (Ty == MVT::funcref) {
    return LLT::pointer(
        WebAssembly::WasmAddressSpace::WASM_ADDRESS_SPACE_FUNCREF,
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 72-109

```cpp
        DL.getPointerSizeInBits(
            WebAssembly::WasmAddressSpace::WASM_ADDRESS_SPACE_FUNCREF));
  }

  return llvm::getLLTForMVT(Ty);
}

WebAssemblyCallLowering::WebAssemblyCallLowering(
    const WebAssemblyTargetLowering &TLI)
    : CallLowering(&TLI) {}

bool WebAssemblyCallLowering::canLowerReturn(MachineFunction &MF,
                                             CallingConv::ID CallConv,
                                             SmallVectorImpl<BaseArgInfo> &Outs,
                                             bool IsVarArg) const {
  return WebAssembly::canLowerReturn(Outs.size(),
                                     &MF.getSubtarget<WebAssemblySubtarget>());
}

bool WebAssemblyCallLowering::lowerReturn(MachineIRBuilder &MIRBuilder,
                                          const Value *Val,
                                          ArrayRef<Register> VRegs,
                                          FunctionLoweringInfo &FLI,
                                          Register SwiftErrorVReg) const {
  MachineFunction &MF = MIRBuilder.getMF();
  const Function &F = MF.getFunction();
  MachineRegisterInfo &MRI = MF.getRegInfo();
  const WebAssemblyTargetLowering &TLI = *getTLI<WebAssemblyTargetLowering>();
  const DataLayout &DL = F.getDataLayout();

  MachineInstrBuilder MIB = MIRBuilder.buildInstrNoInsert(WebAssembly::RETURN);

  assert(((Val && !VRegs.empty()) || (!Val && VRegs.empty())) &&
         "Return value without a vreg or vice versa");

  if (Val) {
    LLVMContext &Ctx = Val->getType()->getContext();
```
- **EN**: Implements helper routine(s) `getPointerSizeInBits`, `getLLTForMVT`, `WebAssemblyCallLowering` for this portion of the WebAssembly backend GlobalISel call lowering logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分GlobalISel 调用约定 lowering 逻辑所需的辅助例程 `getPointerSizeInBits`, `getLLTForMVT`, `WebAssemblyCallLowering`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 110-147

```cpp
    if (!FLI.CanLowerReturn) {
      insertSRetStores(MIRBuilder, Val->getType(), VRegs, FLI.DemoteRegister);
    } else {
      SmallVector<EVT, 4> SplitEVTs;
      ComputeValueVTs(TLI, DL, Val->getType(), SplitEVTs);
      assert(VRegs.size() == SplitEVTs.size() &&
             "For each split Type there should be exactly one VReg.");

      SmallVector<ArgInfo, 8> SplitRets;
      CallingConv::ID CallConv = F.getCallingConv();

      unsigned RetIdx = 0;
      for (EVT SplitEVT : SplitEVTs) {
        Register CurVReg = VRegs[RetIdx];
        ArgInfo CurRetInfo = ArgInfo{CurVReg, SplitEVT.getTypeForEVT(Ctx), 0};
        setArgFlags(CurRetInfo, AttributeList::ReturnIndex, DL, F);

        splitToValueTypes(CurRetInfo, SplitRets, DL, CallConv);
        ++RetIdx;
      }

      for (ArgInfo &Ret : SplitRets) {
        const EVT OrigVT = TLI.getValueType(DL, Ret.Ty);
        const MVT NewVT =
            TLI.getRegisterTypeForCallingConv(Ctx, CallConv, OrigVT);
        const LLT OrigLLT =
            getLLTForType(*OrigVT.getTypeForEVT(F.getContext()), DL);
        const LLT NewLLT = getLLTForWasmMVT(NewVT, DL);

        const TargetRegisterClass &NewRegClass = *TLI.getRegClassFor(NewVT);

        // If we need to split the type over multiple regs, check it's a
        // scenario we currently support.
        const unsigned NumParts =
            TLI.getNumRegistersForCallingConv(Ctx, CallConv, OrigVT);

        const ISD::ArgFlagsTy OrigFlags = Ret.Flags[0];
        Ret.Flags.clear();
```
- **EN**: Implements helper routine(s) `insertSRetStores`, `getType`, `ComputeValueVTs` for this portion of the WebAssembly backend GlobalISel call lowering logic. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里实现了 WebAssembly 后端该部分GlobalISel 调用约定 lowering 逻辑所需的辅助例程 `insertSRetStores`, `getType`, `ComputeValueVTs`。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 148-185

```cpp

        for (unsigned Part = 0; Part < NumParts; ++Part) {
          ISD::ArgFlagsTy Flags = OrigFlags;

          if (Part == 0) {
            Flags.setSplit();
          } else {
            Flags.setOrigAlign(Align(1));
            if (Part == NumParts - 1)
              Flags.setSplitEnd();
          }

          Ret.Flags.push_back(Flags);
        }

        Ret.OrigRegs.assign(Ret.Regs.begin(), Ret.Regs.end());
        if (NumParts != 1 || OrigLLT != NewLLT) {
          // If we can't directly assign the register, we need one or more
          // intermediate values.
          Ret.Regs.resize(NumParts);

          // For each split register, create and assign a vreg that will store
          // the incoming component of the larger value. These will later be
          // merged to form the final vreg.
          for (unsigned Part = 0; Part < NumParts; ++Part) {
            Register NewReg = MRI.createVirtualRegister(&NewRegClass);
            MRI.setType(NewReg, NewLLT);
            MIB.addUse(NewReg);
            Ret.Regs[Part] = NewReg;
          }

          buildCopyToRegs(MIRBuilder, Ret.Regs, Ret.OrigRegs[0], OrigLLT,
                          NewLLT, extendOpFromFlags(Ret.Flags[0]));
        } else {
          MIB.addUse(Ret.Regs[0]);
        }
      }
    }
```
- **EN**: Implements helper routine(s) `setSplit`, `setOrigAlign`, `Align` for this portion of the WebAssembly backend GlobalISel call lowering logic. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里实现了 WebAssembly 后端该部分GlobalISel 调用约定 lowering 逻辑所需的辅助例程 `setSplit`, `setOrigAlign`, `Align`。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 186-223

```cpp
  }

  if (SwiftErrorVReg) {
    reportFatalInternalError(
        "Wasm does not `supportSwiftError`, yet SwiftErrorVReg is "
        "improperly valid.");
  }

  MIRBuilder.insertInstr(MIB);
  return true;
}

static unsigned getWasmArgumentOpcode(MVT ArgType) {
  switch (ArgType.SimpleTy) {
  case MVT::i32:
    return WebAssembly::ARGUMENT_i32;
  case MVT::i64:
    return WebAssembly::ARGUMENT_i64;
  case MVT::f32:
    return WebAssembly::ARGUMENT_f32;
  case MVT::f64:
    return WebAssembly::ARGUMENT_f64;

  case MVT::funcref:
    return WebAssembly::ARGUMENT_funcref;
  case MVT::externref:
    return WebAssembly::ARGUMENT_externref;
  case MVT::exnref:
    return WebAssembly::ARGUMENT_exnref;

  case MVT::v16i8:
    return WebAssembly::ARGUMENT_v16i8;
  case MVT::v8i16:
    return WebAssembly::ARGUMENT_v8i16;
  case MVT::v4i32:
    return WebAssembly::ARGUMENT_v4i32;
  case MVT::v2i64:
    return WebAssembly::ARGUMENT_v2i64;
```
- **EN**: Implements helper routine(s) `reportFatalInternalError`, `insertInstr`, `getWasmArgumentOpcode` for this portion of the WebAssembly backend GlobalISel call lowering logic.
- **CN**: 这里实现了 WebAssembly 后端该部分GlobalISel 调用约定 lowering 逻辑所需的辅助例程 `reportFatalInternalError`, `insertInstr`, `getWasmArgumentOpcode`。

### Lines 224-261

```cpp
  case MVT::v8f16:
    return WebAssembly::ARGUMENT_v8f16;
  case MVT::v4f32:
    return WebAssembly::ARGUMENT_v4f32;
  case MVT::v2f64:
    return WebAssembly::ARGUMENT_v2f64;
  default:
    break;
  }
  llvm_unreachable("Found unexpected type for Wasm argument");
}

static Register buildWasmArgument(unsigned Idx, MVT ArgVT, LLT ArgLLT,
                                  MachineIRBuilder &MIRBuilder,
                                  Register Def = Register()) {
  unsigned Op = getWasmArgumentOpcode(ArgVT);

  const TargetInstrInfo &TII = MIRBuilder.getTII();
  MachineFunction &MF = MIRBuilder.getMF();
  MachineRegisterInfo &MRI = MF.getRegInfo();
  const TargetRegisterClass &RegClass = *TII.getRegClass(TII.get(Op), 0);

  Register NewReg;

  if (Def.isValid()) {
    assert(MRI.getRegClassOrRegBank(Def).isNull() &&
           "Def already has reg bank or reg class?");
    MRI.setRegClass(Def, &RegClass);

    NewReg = Def;
  } else {
    NewReg = MRI.createVirtualRegister(&RegClass);
    MRI.setType(NewReg, ArgLLT);
  }

  MIRBuilder.buildInstr(Op).addDef(NewReg).addImm(Idx);

  return NewReg;
```
- **EN**: Implements helper routine(s) `llvm_unreachable`, `buildWasmArgument`, `Register` for this portion of the WebAssembly backend GlobalISel call lowering logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分GlobalISel 调用约定 lowering 逻辑所需的辅助例程 `llvm_unreachable`, `buildWasmArgument`, `Register`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 262-299

```cpp
}

bool WebAssemblyCallLowering::lowerFormalArguments(
    MachineIRBuilder &MIRBuilder, const Function &F,
    ArrayRef<ArrayRef<Register>> VRegs, FunctionLoweringInfo &FLI) const {
  MachineFunction &MF = MIRBuilder.getMF();
  MachineRegisterInfo &MRI = MF.getRegInfo();
  WebAssemblyFunctionInfo *MFI = MF.getInfo<WebAssemblyFunctionInfo>();
  const DataLayout &DL = F.getDataLayout();
  const WebAssemblyTargetLowering &TLI = *getTLI<WebAssemblyTargetLowering>();

  LLVMContext &Ctx = MIRBuilder.getContext();
  const CallingConv::ID CallConv = F.getCallingConv();

  if (!callingConvSupported(CallConv))
    return false;

  MF.getRegInfo().addLiveIn(WebAssembly::ARGUMENTS);
  MF.front().addLiveIn(WebAssembly::ARGUMENTS);

  SmallVector<ArgInfo, 8> SplitArgs;

  if (!FLI.CanLowerReturn)
    insertSRetIncomingArgument(F, SplitArgs, FLI.DemoteRegister, MRI, DL);

  unsigned ArgIdx = 0;
  bool HasSwiftErrorArg = false;
  bool HasSwiftSelfArg = false;
  for (const Argument &Arg : F.args()) {
    ArgInfo OrigArg{VRegs[ArgIdx], Arg.getType(), ArgIdx};
    setArgFlags(OrigArg, ArgIdx + AttributeList::FirstArgIndex, DL, F);

    HasSwiftSelfArg |= Arg.hasSwiftSelfAttr();
    HasSwiftErrorArg |= Arg.hasSwiftErrorAttr();
    if (Arg.hasInAllocaAttr())
      return false;
    if (Arg.hasNestAttr())
      return false;
```
- **EN**: Implements helper routine(s) `lowerFormalArguments`, `getMF`, `getRegInfo` for this portion of the WebAssembly backend GlobalISel call lowering logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分GlobalISel 调用约定 lowering 逻辑所需的辅助例程 `lowerFormalArguments`, `getMF`, `getRegInfo`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 300-337

```cpp

    splitToValueTypes(OrigArg, SplitArgs, DL, F.getCallingConv());
    ++ArgIdx;
  }

  unsigned FinalArgIdx = 0;
  for (ArgInfo &Arg : SplitArgs) {
    const EVT OrigVT = TLI.getValueType(DL, Arg.Ty);
    const MVT NewVT = TLI.getRegisterTypeForCallingConv(Ctx, CallConv, OrigVT);
    const LLT OrigLLT =
        getLLTForType(*OrigVT.getTypeForEVT(F.getContext()), DL);
    const LLT NewLLT = getLLTForWasmMVT(NewVT, DL);

    // If we need to split the type over multiple regs, check it's a scenario
    // we currently support.
    const unsigned NumParts =
        TLI.getNumRegistersForCallingConv(Ctx, CallConv, OrigVT);

    const ISD::ArgFlagsTy OrigFlags = Arg.Flags[0];
    Arg.Flags.clear();

    for (unsigned Part = 0; Part < NumParts; ++Part) {
      ISD::ArgFlagsTy Flags = OrigFlags;
      if (Part == 0) {
        Flags.setSplit();
      } else {
        Flags.setOrigAlign(Align(1));
        if (Part == NumParts - 1)
          Flags.setSplitEnd();
      }

      Arg.Flags.push_back(Flags);
    }

    Arg.OrigRegs.assign(Arg.Regs.begin(), Arg.Regs.end());
    if (NumParts != 1 || OrigLLT != NewLLT) {
      // If we can't directly assign the register, we need one or more
      // intermediate values.
```
- **EN**: Implements helper routine(s) `splitToValueTypes`, `getCallingConv`, `getValueType` for this portion of the WebAssembly backend GlobalISel call lowering logic. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里实现了 WebAssembly 后端该部分GlobalISel 调用约定 lowering 逻辑所需的辅助例程 `splitToValueTypes`, `getCallingConv`, `getValueType`。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 338-375

```cpp
      Arg.Regs.resize(NumParts);

      // For each split register, create and assign a vreg that will store
      // the incoming component of the larger value. These will later be
      // merged to form the final vreg.
      for (unsigned Part = 0; Part < NumParts; ++Part) {
        Arg.Regs[Part] =
            buildWasmArgument(FinalArgIdx++, NewVT, NewLLT, MIRBuilder);
        MFI->addParam(NewVT);
      }

      buildCopyFromRegs(MIRBuilder, Arg.OrigRegs, Arg.Regs, OrigLLT, NewLLT,
                        Arg.Flags[0]);
    } else {
      buildWasmArgument(FinalArgIdx++, NewVT, NewLLT, MIRBuilder, Arg.Regs[0]);
      MFI->addParam(NewVT);
    }
  }

  // For swiftcc, emit additional swiftself and swifterror arguments
  // if there aren't. These additional arguments are also added for callee
  // signature They are necessary to match callee and caller signature for
  // indirect call.
  if (CallConv == CallingConv::Swift) {
    const MVT PtrVT = TLI.getPointerTy(DL);

    if (!HasSwiftSelfArg)
      MFI->addParam(PtrVT);
    if (!HasSwiftErrorArg)
      MFI->addParam(PtrVT);
  }

  // Varargs are copied into a buffer allocated by the caller, and a pointer to
  // the buffer is passed as an argument.
  if (F.isVarArg()) {
    const MVT PtrVT = TLI.getPointerTy(DL, 0);
    const LLT PtrLLT = LLT::pointer(0, DL.getPointerSizeInBits(0));
```
- **EN**: Implements helper routine(s) `resize`, `buildWasmArgument`, `addParam` for this portion of the WebAssembly backend GlobalISel call lowering logic. ABI and calling-convention details are important in this part of the code.
- **CN**: 这里实现了 WebAssembly 后端该部分GlobalISel 调用约定 lowering 逻辑所需的辅助例程 `resize`, `buildWasmArgument`, `addParam`。 ABI 与调用约定细节是这一部分代码的重要约束。

### Lines 376-413

```cpp
    Register VarargVreg =
        buildWasmArgument(FinalArgIdx++, PtrVT, PtrLLT, MIRBuilder);
    MFI->setVarargBufferVreg(VarargVreg);

    MFI->addParam(PtrVT);
    ++FinalArgIdx;
  }

  // Record the number and types of arguments and results.
  SmallVector<MVT, 4> Params;
  SmallVector<MVT, 4> Results;
  computeSignatureVTs(MF.getFunction().getFunctionType(), &MF.getFunction(),
                      MF.getFunction(), MF.getTarget(), Params, Results);
  for (MVT VT : Results)
    MFI->addResult(VT);

  // TODO: Use signatures in WebAssemblyMachineFunctionInfo too and unify
  // the param logic here with ComputeSignatureVTs
  assert(MFI->getParams().size() == Params.size() &&
         std::equal(MFI->getParams().begin(), MFI->getParams().end(),
                    Params.begin()));
  return true;
}

bool WebAssemblyCallLowering::lowerCall(MachineIRBuilder &MIRBuilder,
                                        CallLoweringInfo &Info) const {
  MachineFunction &MF = MIRBuilder.getMF();
  const DataLayout &DL = MIRBuilder.getDataLayout();
  LLVMContext &Ctx = MIRBuilder.getContext();
  const WebAssemblyTargetLowering &TLI = *getTLI<WebAssemblyTargetLowering>();
  MachineRegisterInfo &MRI = *MIRBuilder.getMRI();
  const Function &F = MF.getFunction();

  CallingConv::ID CallConv = Info.CallConv;
  if (!callingConvSupported(CallConv))
    return false;

  // TODO: tail calls
```
- **EN**: Implements helper routine(s) `buildWasmArgument`, `setVarargBufferVreg`, `addParam` for this portion of the WebAssembly backend GlobalISel call lowering logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分GlobalISel 调用约定 lowering 逻辑所需的辅助例程 `buildWasmArgument`, `setVarargBufferVreg`, `addParam`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 414-451

```cpp
  if (Info.IsMustTailCall)
    return false;

  // TODO: varargs
  if (Info.IsVarArg)
    return false;

  // TODO: swiftcc
  if (CallConv == CallingConv::Swift)
    return false;

  MachineInstrBuilder CallInst;

  // TODO: indirect calls
  if (Info.Callee.isReg())
    return false;

  CallInst = MIRBuilder.buildInstrNoInsert(WebAssembly::CALL);

  SmallVector<ArgInfo, 8> SplitArgs;

  for (const ArgInfo &Arg : Info.OrigArgs) {
    if (Arg.Flags[0].isNest())
      return false;
    if (Arg.Flags[0].isInAlloca())
      return false;
    if (Arg.Flags[0].isInConsecutiveRegs())
      return false;
    if (Arg.Flags[0].isInConsecutiveRegsLast())
      return false;

    // TODO: bulk memory, then byval
    if (Arg.Flags[0].isByVal() && Arg.Flags[0].getByValSize() != 0)
      return false;

    splitToValueTypes(Arg, SplitArgs, DL, CallConv);
  }
```
- **EN**: Implements helper routine(s) `isReg`, `buildInstrNoInsert`, `isNest` for this portion of the WebAssembly backend GlobalISel call lowering logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分GlobalISel 调用约定 lowering 逻辑所需的辅助例程 `isReg`, `buildInstrNoInsert`, `isNest`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 452-489

```cpp
  SmallVector<Register> CallUseRegs;

  for (ArgInfo &Arg : SplitArgs) {
    const EVT OrigVT = TLI.getValueType(DL, Arg.Ty);
    const MVT NewVT = TLI.getRegisterTypeForCallingConv(Ctx, CallConv, OrigVT);
    const LLT OrigLLT =
        getLLTForType(*OrigVT.getTypeForEVT(F.getContext()), DL);
    const LLT NewLLT = getLLTForWasmMVT(NewVT, DL);

    const TargetRegisterClass &NewRegClass = *TLI.getRegClassFor(NewVT);

    // If we need to split the type over multiple regs, check it's a scenario
    // we currently support.
    const unsigned NumParts =
        TLI.getNumRegistersForCallingConv(Ctx, CallConv, OrigVT);

    const ISD::ArgFlagsTy OrigFlags = Arg.Flags[0];
    Arg.Flags.clear();

    for (unsigned Part = 0; Part < NumParts; ++Part) {
      ISD::ArgFlagsTy Flags = OrigFlags;
      if (Part == 0) {
        Flags.setSplit();
      } else {
        Flags.setOrigAlign(Align(1));
        if (Part == NumParts - 1)
          Flags.setSplitEnd();
      }

      Arg.Flags.push_back(Flags);
    }

    Arg.OrigRegs.assign(Arg.Regs.begin(), Arg.Regs.end());
    if (NumParts != 1 || OrigLLT != NewLLT) {
      // If we can't directly assign the register, we need one or more
      // intermediate values.
      Arg.Regs.resize(NumParts);
```
- **EN**: Implements helper routine(s) `getValueType`, `getRegisterTypeForCallingConv`, `getLLTForType` for this portion of the WebAssembly backend GlobalISel call lowering logic. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里实现了 WebAssembly 后端该部分GlobalISel 调用约定 lowering 逻辑所需的辅助例程 `getValueType`, `getRegisterTypeForCallingConv`, `getLLTForType`。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 490-527

```cpp
      // For each split register, create and assign a vreg that will store
      // the incoming component of the larger value. These will later be
      // merged to form the final vreg.
      for (unsigned Part = 0; Part < NumParts; ++Part) {
        Register NewReg = MRI.createVirtualRegister(&NewRegClass);
        MRI.setType(NewReg, NewLLT);
        CallUseRegs.push_back(NewReg);
        Arg.Regs[Part] = NewReg;
      }

      buildCopyToRegs(MIRBuilder, Arg.Regs, Arg.OrigRegs[0], OrigLLT, NewLLT,
                      extendOpFromFlags(Arg.Flags[0]));
    } else {
      CallUseRegs.push_back(Arg.Regs[0]);
    }
  }

  // Analyze operands of the call, assigning locations to each operand.
  SmallVector<CCValAssign, 16> ArgLocs;
  CCState CCInfo(CallConv, Info.IsVarArg, MF, ArgLocs, Ctx);

  MIRBuilder.insertInstr(CallInst);

  if (Info.CanLowerReturn && !Info.OrigRet.Ty->isVoidTy()) {
    SmallVector<EVT, 4> SplitEVTs;
    ComputeValueVTs(TLI, DL, Info.OrigRet.Ty, SplitEVTs);
    assert(Info.OrigRet.Regs.size() == SplitEVTs.size() &&
           "For each split Type there should be exactly one VReg.");

    SmallVector<ArgInfo, 8> SplitRets;

    unsigned RetIdx = 0;
    for (EVT SplitEVT : SplitEVTs) {
      Register CurVReg = Info.OrigRet.Regs[RetIdx];
      ArgInfo CurArgInfo = ArgInfo{CurVReg, SplitEVT.getTypeForEVT(Ctx), 0};

      if (Info.CB) {
        setArgFlags(CurArgInfo, AttributeList::ReturnIndex, DL, *Info.CB);
```
- **EN**: Implements helper routine(s) `createVirtualRegister`, `setType`, `push_back` for this portion of the WebAssembly backend GlobalISel call lowering logic. ABI and calling-convention details are important in this part of the code.
- **CN**: 这里实现了 WebAssembly 后端该部分GlobalISel 调用约定 lowering 逻辑所需的辅助例程 `createVirtualRegister`, `setType`, `push_back`。 ABI 与调用约定细节是这一部分代码的重要约束。

### Lines 528-565

```cpp
      } else {
        // We don't have a call base, so chances are we're looking at a
        // libcall (external symbol).

        // TODO: figure out how to get ALL the correct attributes
        ISD::ArgFlagsTy &Flags = CurArgInfo.Flags[0];
        PointerType *PtrTy =
            dyn_cast<PointerType>(CurArgInfo.Ty->getScalarType());
        if (PtrTy) {
          Flags.setPointer();
          Flags.setPointerAddrSpace(PtrTy->getPointerAddressSpace());
        }
        Align MemAlign = DL.getABITypeAlign(CurArgInfo.Ty);
        Flags.setMemAlign(MemAlign);
        Flags.setOrigAlign(MemAlign);
      }

      splitToValueTypes(CurArgInfo, SplitRets, DL, CallConv);
      ++RetIdx;
    }

    for (ArgInfo &Ret : SplitRets) {
      const EVT OrigVT = TLI.getValueType(DL, Ret.Ty);
      const MVT NewVT =
          TLI.getRegisterTypeForCallingConv(Ctx, CallConv, OrigVT);
      const LLT OrigLLT =
          getLLTForType(*OrigVT.getTypeForEVT(F.getContext()), DL);
      const LLT NewLLT = getLLTForWasmMVT(NewVT, DL);

      const TargetRegisterClass &NewRegClass = *TLI.getRegClassFor(NewVT);

      // If we need to split the type over multiple regs, check it's a scenario
      // we currently support.
      const unsigned NumParts =
          TLI.getNumRegistersForCallingConv(Ctx, CallConv, OrigVT);

      const ISD::ArgFlagsTy OrigFlags = Ret.Flags[0];
      Ret.Flags.clear();
```
- **EN**: Implements helper routine(s) `libcall`, `getScalarType`, `setPointer` for this portion of the WebAssembly backend GlobalISel call lowering logic. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里实现了 WebAssembly 后端该部分GlobalISel 调用约定 lowering 逻辑所需的辅助例程 `libcall`, `getScalarType`, `setPointer`。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 566-603

```cpp

      for (unsigned Part = 0; Part < NumParts; ++Part) {
        ISD::ArgFlagsTy Flags = OrigFlags;
        if (Part == 0) {
          Flags.setSplit();
        } else {
          Flags.setOrigAlign(Align(1));
          if (Part == NumParts - 1)
            Flags.setSplitEnd();
        }

        Ret.Flags.push_back(Flags);
      }

      Ret.OrigRegs.assign(Ret.Regs.begin(), Ret.Regs.end());

      if (NumParts != 1 || OrigLLT != NewLLT) {
        // If we can't directly assign the register, we need one or more
        // intermediate values.
        Ret.Regs.resize(NumParts);

        // For each split register, create and assign a vreg that will store
        // the incoming component of the larger value. These will later be
        // merged to form the final vreg.
        for (unsigned Part = 0; Part < NumParts; ++Part) {
          Register NewReg = MRI.createVirtualRegister(&NewRegClass);
          MRI.setType(NewReg, NewLLT);
          CallInst.addDef(NewReg);
          Ret.Regs[Part] = NewReg;
        }

        buildCopyFromRegs(MIRBuilder, Ret.OrigRegs, Ret.Regs, OrigLLT, NewLLT,
                          Ret.Flags[0]);
      } else {
        MRI.setRegClass(Ret.Regs[0], &NewRegClass);
        CallInst.addDef(Ret.Regs[0]);
      }
    }
```
- **EN**: Implements helper routine(s) `setSplit`, `setOrigAlign`, `Align` for this portion of the WebAssembly backend GlobalISel call lowering logic. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里实现了 WebAssembly 后端该部分GlobalISel 调用约定 lowering 逻辑所需的辅助例程 `setSplit`, `setOrigAlign`, `Align`。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 604-623

```cpp
  }

  if (Info.Callee.isGlobal()) {
    CallInst.addGlobalAddress(Info.Callee.getGlobal());
  } else if (Info.Callee.isSymbol()) {
    CallInst.addExternalSymbol(Info.Callee.getSymbolName());
  } else {
    return false;
  }

  for (Register Reg : CallUseRegs) {
    CallInst.addUse(Reg);
  }

  if (!Info.CanLowerReturn)
    insertSRetLoads(MIRBuilder, Info.OrigRet.Ty, Info.OrigRet.Regs,
                    Info.DemoteRegister, Info.DemoteStackIndex);

  return true;
}
```
- **EN**: Implements helper routine(s) `isGlobal`, `addGlobalAddress`, `getGlobal` for this portion of the WebAssembly backend GlobalISel call lowering logic. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里实现了 WebAssembly 后端该部分GlobalISel 调用约定 lowering 逻辑所需的辅助例程 `isGlobal`, `addGlobalAddress`, `getGlobal`。 该区间与栈帧布局或栈访问相关。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- GlobalISel call lowering logic / GlobalISel 调用约定 lowering 逻辑
- GlobalISel pipeline / GlobalISel 流水线
- SelectionDAG lowering / SelectionDAG lowering
- Calling convention handling / 调用约定处理
- Register modeling / 寄存器建模
- Stack frame management / 栈帧管理
- Subtarget features / 子目标特性

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `WebAssemblyCallLowering.h`
- `MCTargetDesc/WebAssemblyMCTargetDesc.h`
- `Utils/WasmAddressSpaces.h`
- `WebAssemblyISelLowering.h`
- `WebAssemblyMachineFunctionInfo.h`
- `WebAssemblyRegisterInfo.h`
- `WebAssemblySubtarget.h`
- `WebAssemblyUtilities.h`
- `llvm/CodeGen/Analysis.h`
- `llvm/CodeGen/FunctionLoweringInfo.h`
- `llvm/CodeGen/GlobalISel/MachineIRBuilder.h`
- `llvm/CodeGen/LowLevelTypeUtils.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
- `llvm/CodeGen/RegisterBankInfo.h`
- `llvm/IR/DataLayout.h`
- `llvm/IR/DebugLoc.h`
- `llvm/IR/Value.h`
- `llvm/Support/ErrorHandling.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM Support utilities / LLVM Support 工具层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
