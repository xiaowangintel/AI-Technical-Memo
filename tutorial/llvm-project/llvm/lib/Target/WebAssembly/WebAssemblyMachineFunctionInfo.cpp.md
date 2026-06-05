# WebAssemblyMachineFunctionInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyMachineFunctionInfo.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file implements WebAssembly-specific per-machine-function information.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyMachineFunctionInfo.cpp`，主要负责 WebAssembly 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//=- WebAssemblyMachineFunctionInfo.cpp - WebAssembly Machine Function Info -=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-12

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// This file implements WebAssembly-specific per-machine-function
/// information.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 13-19

```cpp
//===----------------------------------------------------------------------===//

#include "WebAssemblyMachineFunctionInfo.h"
#include "Utils/WebAssemblyTypeUtilities.h"
#include "WebAssemblyISelLowering.h"
#include "WebAssemblySubtarget.h"
#include "WebAssemblyUtilities.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 20-57

```cpp
#include "llvm/CodeGen/Analysis.h"
#include "llvm/Target/TargetMachine.h"
using namespace llvm;

WebAssemblyFunctionInfo::~WebAssemblyFunctionInfo() = default; // anchor.

MachineFunctionInfo *WebAssemblyFunctionInfo::clone(
    BumpPtrAllocator &Allocator, MachineFunction &DestMF,
    const DenseMap<MachineBasicBlock *, MachineBasicBlock *> &Src2DstMBB)
    const {
  return DestMF.cloneInfo<WebAssemblyFunctionInfo>(*this);
}

void WebAssemblyFunctionInfo::initWARegs(MachineRegisterInfo &MRI) {
  assert(WARegs.empty());
  unsigned Reg = WebAssembly::UnusedReg;
  WARegs.resize(MRI.getNumVirtRegs(), Reg);
}

void llvm::computeLegalValueVTs(const WebAssemblyTargetLowering &TLI,
                                LLVMContext &Ctx, const DataLayout &DL,
                                Type *Ty, SmallVectorImpl<MVT> &ValueVTs) {
  SmallVector<EVT, 4> VTs;
  ComputeValueVTs(TLI, DL, Ty, VTs);

  for (EVT VT : VTs) {
    unsigned NumRegs = TLI.getNumRegisters(Ctx, VT);
    MVT RegisterVT = TLI.getRegisterType(Ctx, VT);
    for (unsigned I = 0; I != NumRegs; ++I)
      ValueVTs.push_back(RegisterVT);
  }
}

void llvm::computeLegalValueVTs(const Function &F, const TargetMachine &TM,
                                Type *Ty, SmallVectorImpl<MVT> &ValueVTs) {
  const DataLayout &DL(F.getDataLayout());
  const WebAssemblyTargetLowering &TLI =
      *TM.getSubtarget<WebAssemblySubtarget>(F).getTargetLowering();
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Register assignment and register-class constraints matter here.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 58-95

```cpp
  computeLegalValueVTs(TLI, F.getContext(), DL, Ty, ValueVTs);
}

void llvm::computeSignatureVTs(const FunctionType *Ty,
                               const Function *TargetFunc,
                               const Function &ContextFunc,
                               const TargetMachine &TM,
                               SmallVectorImpl<MVT> &Params,
                               SmallVectorImpl<MVT> &Results) {
  computeLegalValueVTs(ContextFunc, TM, Ty->getReturnType(), Results);

  MVT PtrVT = MVT::getIntegerVT(TM.createDataLayout().getPointerSizeInBits());
  if (!WebAssembly::canLowerReturn(
          Results.size(),
          &TM.getSubtarget<WebAssemblySubtarget>(ContextFunc))) {
    // WebAssembly can't lower returns of multiple values without demoting to
    // sret unless multivalue is enabled (see
    // WebAssemblyTargetLowering::CanLowerReturn). So replace multiple return
    // values with a poitner parameter.
    Results.clear();
    Params.push_back(PtrVT);
  }

  for (auto *Param : Ty->params())
    computeLegalValueVTs(ContextFunc, TM, Param, Params);
  if (Ty->isVarArg())
    Params.push_back(PtrVT);

  // For swiftcc and swifttailcc, emit additional swiftself, swifterror, and
  // (for swifttailcc) swiftasync parameters if there aren't. These additional
  // parameters are also passed for caller. They are necessary to match callee
  // and caller signature for indirect call.

  if (TargetFunc && (TargetFunc->getCallingConv() == CallingConv::Swift ||
                     TargetFunc->getCallingConv() == CallingConv::SwiftTail)) {
    MVT PtrVT = MVT::getIntegerVT(TM.createDataLayout().getPointerSizeInBits());
    bool HasSwiftErrorArg = false;
    bool HasSwiftSelfArg = false;
```
- **EN**: Implements helper routine(s) `computeLegalValueVTs`, `getContext`, `computeSignatureVTs` for this portion of the WebAssembly backend backend implementation logic. ABI and calling-convention details are important in this part of the code.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `computeLegalValueVTs`, `getContext`, `computeSignatureVTs`。 ABI 与调用约定细节是这一部分代码的重要约束。

### Lines 96-133

```cpp
    bool HasSwiftAsyncArg = false;
    for (const auto &Arg : TargetFunc->args()) {
      HasSwiftErrorArg |= Arg.hasAttribute(Attribute::SwiftError);
      HasSwiftSelfArg |= Arg.hasAttribute(Attribute::SwiftSelf);
      HasSwiftAsyncArg |= Arg.hasAttribute(Attribute::SwiftAsync);
    }
    if (!HasSwiftSelfArg)
      Params.push_back(PtrVT);
    if (!HasSwiftErrorArg)
      Params.push_back(PtrVT);
    if (TargetFunc->getCallingConv() == CallingConv::SwiftTail &&
        !HasSwiftAsyncArg)
      Params.push_back(PtrVT);
  }
}

void llvm::valTypesFromMVTs(ArrayRef<MVT> In,
                            SmallVectorImpl<wasm::ValType> &Out) {
  for (MVT Ty : In)
    Out.push_back(WebAssembly::toValType(Ty));
}

wasm::WasmSignature *
llvm::signatureFromMVTs(MCContext &Ctx, const SmallVectorImpl<MVT> &Results,
                        const SmallVectorImpl<MVT> &Params) {
  auto Sig = Ctx.createWasmSignature();
  valTypesFromMVTs(Results, Sig->Returns);
  valTypesFromMVTs(Params, Sig->Params);
  return Sig;
}

yaml::WebAssemblyFunctionInfo::WebAssemblyFunctionInfo(
    const llvm::MachineFunction &MF, const llvm::WebAssemblyFunctionInfo &MFI)
    : CFGStackified(MFI.isCFGStackified()) {
  for (auto VT : MFI.getParams())
    Params.push_back(EVT(VT).getEVTString());
  for (auto VT : MFI.getResults())
    Results.push_back(EVT(VT).getEVTString());
```
- **EN**: Implements helper routine(s) `args`, `hasAttribute`, `push_back` for this portion of the WebAssembly backend backend implementation logic. ABI and calling-convention details are important in this part of the code.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `args`, `hasAttribute`, `push_back`。 ABI 与调用约定细节是这一部分代码的重要约束。

### Lines 134-147

```cpp
}

void yaml::WebAssemblyFunctionInfo::mappingImpl(yaml::IO &YamlIO) {
  MappingTraits<WebAssemblyFunctionInfo>::mapping(YamlIO, *this);
}

void WebAssemblyFunctionInfo::initializeBaseYamlFields(
    MachineFunction &MF, const yaml::WebAssemblyFunctionInfo &YamlMFI) {
  CFGStackified = YamlMFI.CFGStackified;
  for (auto VT : YamlMFI.Params)
    addParam(WebAssembly::parseMVT(VT.Value));
  for (auto VT : YamlMFI.Results)
    addResult(WebAssembly::parseMVT(VT.Value));
}
```
- **EN**: Implements helper routine(s) `mappingImpl`, `mapping`, `initializeBaseYamlFields` for this portion of the WebAssembly backend backend implementation logic. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `mappingImpl`, `mapping`, `initializeBaseYamlFields`。 该区间与栈帧布局或栈访问相关。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Backend implementation logic / 后端实现逻辑
- SelectionDAG lowering / SelectionDAG lowering
- Calling convention handling / 调用约定处理
- Register modeling / 寄存器建模
- Stack frame management / 栈帧管理
- Subtarget features / 子目标特性
- Target machine configuration / 目标机器配置

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `WebAssemblyMachineFunctionInfo.h`
- `Utils/WebAssemblyTypeUtilities.h`
- `WebAssemblyISelLowering.h`
- `WebAssemblySubtarget.h`
- `WebAssemblyUtilities.h`
- `llvm/CodeGen/Analysis.h`
- `llvm/Target/TargetMachine.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
