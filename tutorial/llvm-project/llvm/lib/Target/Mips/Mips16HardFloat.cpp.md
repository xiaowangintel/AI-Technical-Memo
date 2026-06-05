# Mips16HardFloat.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/Mips16HardFloat.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file defines a pass needed for Mips16 Hard Float.
- 用途 (CN): 实现 Mips 后端中的 `Mips16HardFloat`，重点处理目标相关代码生成行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===- Mips16HardFloat.cpp for Mips16 Hard Float --------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines a pass needed for Mips16 Hard Float
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 13-20
```cpp
#include "MipsTargetMachine.h"
#include "llvm/CodeGen/TargetPassConfig.h"
#include "llvm/IR/Module.h"
#include "llvm/IR/Value.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/ModRef.h"
#include "llvm/Support/raw_ostream.h"
#include <string>
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
#define DEBUG_TYPE "mips16-hard-float"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 26-26
```cpp
namespace {
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 28-30
```cpp
  class Mips16HardFloat : public ModulePass {
  public:
    static char ID;
```
- EN: Declares `Mips16HardFloat`, packaging target-specific state and APIs around `Mips16HardFloat`.
- CN: 这里声明 `Mips16HardFloat`，把与 `Mips16HardFloat` 相关的目标特定状态和 API 组织在一起。

### Lines 32-32
```cpp
    Mips16HardFloat() : ModulePass(ID) {}
```
- EN: Implements `Mips16HardFloat`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips16HardFloat`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 34-34
```cpp
    StringRef getPassName() const override { return "MIPS16 Hard Float Pass"; }
```
- EN: Implements `getPassName`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getPassName`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 36-39
```cpp
    void getAnalysisUsage(AnalysisUsage &AU) const override {
      AU.addRequired<TargetPassConfig>();
      ModulePass::getAnalysisUsage(AU);
    }
```
- EN: Implements `getAnalysisUsage`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getAnalysisUsage`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 41-42
```cpp
    bool runOnModule(Module &M) override;
  };
```
- EN: Declares `runOnModule`, a pass-entry routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `runOnModule`，它是一个围绕目标相关状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 44-44
```cpp
} // end anonymous namespace
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 46-48
```cpp
static void emitInlineAsm(LLVMContext &C, BasicBlock *BB, StringRef AsmText) {
  std::vector<Type *> AsmArgTypes;
  std::vector<Value *> AsmArgs;
```
- EN: Implements `emitInlineAsm`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `emitInlineAsm`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 50-55
```cpp
  FunctionType *AsmFTy =
      FunctionType::get(Type::getVoidTy(C), AsmArgTypes, false);
  InlineAsm *IA = InlineAsm::get(AsmFTy, AsmText, "", true,
                                 /* IsAlignStack */ false, InlineAsm::AD_ATT);
  CallInst::Create(IA, AsmArgs, "", BB);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 57-57
```cpp
char Mips16HardFloat::ID = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 59-65
```cpp
//
// Return types that matter for hard float are:
// float, double, complex float, and complex double
//
enum FPReturnVariant {
  FRet, DRet, CFRet, CDRet, NoFPRet
};
```
- EN: Defines enumeration `FPReturnVariant` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `FPReturnVariant`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 67-81
```cpp
//
// Determine which FP return type this function has
//
static FPReturnVariant whichFPReturnVariant(Type *T) {
  switch (T->getTypeID()) {
  case Type::FloatTyID:
    return FRet;
  case Type::DoubleTyID:
    return DRet;
  case Type::StructTyID: {
    StructType *ST = cast<StructType>(T);
    if (ST->getNumElements() != 2)
      break;
    if ((ST->getElementType(0)->isFloatTy()) &&
        (ST->getElementType(1)->isFloatTy()))
```
- EN: Implements `whichFPReturnVariant`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `whichFPReturnVariant`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 82-92
```cpp
      return CFRet;
    if ((ST->getElementType(0)->isDoubleTy()) &&
        (ST->getElementType(1)->isDoubleTy()))
      return CDRet;
    break;
  }
  default:
    break;
  }
  return NoFPRet;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 94-99
```cpp
// Parameter type that matter are float, (float, float), (float, double),
// double, (double, double), (double, float)
enum FPParamVariant {
  FSig, FFSig, FDSig,
  DSig, DDSig, DFSig, NoSig
};
```
- EN: Defines enumeration `FPParamVariant` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `FPParamVariant`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 101-104
```cpp
// which floating point parameter signature variant we are dealing with
using TypeID = Type::TypeID;
const Type::TypeID FloatTyID = Type::FloatTyID;
const Type::TypeID DoubleTyID = Type::DoubleTyID;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 106-120
```cpp
static FPParamVariant whichFPParamVariantNeeded(Function &F) {
  switch (F.arg_size()) {
  case 0:
    return NoSig;
  case 1:{
    TypeID ArgTypeID = F.getFunctionType()->getParamType(0)->getTypeID();
    switch (ArgTypeID) {
    case FloatTyID:
      return FSig;
    case DoubleTyID:
      return DSig;
    default:
      return NoSig;
    }
  }
```
- EN: Implements `whichFPParamVariantNeeded`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `whichFPParamVariantNeeded`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 121-135
```cpp
  default: {
    TypeID ArgTypeID0 = F.getFunctionType()->getParamType(0)->getTypeID();
    TypeID ArgTypeID1 = F.getFunctionType()->getParamType(1)->getTypeID();
    switch(ArgTypeID0) {
    case FloatTyID: {
      switch (ArgTypeID1) {
      case FloatTyID:
        return FFSig;
      case DoubleTyID:
        return FDSig;
      default:
        return FSig;
      }
    }
    case DoubleTyID: {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 136-150
```cpp
      switch (ArgTypeID1) {
      case FloatTyID:
        return DFSig;
      case DoubleTyID:
        return DDSig;
      default:
        return DSig;
      }
    }
    default:
      return NoSig;
    }
  }
  }
  llvm_unreachable("can't get here");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 151-151
```cpp
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 153-167
```cpp
// Figure out if we need float point based on the function parameters.
// We need to move variables in and/or out of floating point
// registers because of the ABI
static bool needsFPStubFromParams(Function &F) {
  if (F.arg_size() >=1) {
    Type *ArgType = F.getFunctionType()->getParamType(0);
    switch (ArgType->getTypeID()) {
    case Type::FloatTyID:
    case Type::DoubleTyID:
      return true;
    default:
      break;
    }
  }
  return false;
```
- EN: Implements `needsFPStubFromParams`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `needsFPStubFromParams`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 168-168
```cpp
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 170-173
```cpp
static bool needsFPReturnHelper(Function &F) {
  Type* RetType = F.getReturnType();
  return whichFPReturnVariant(RetType) != NoFPRet;
}
```
- EN: Implements `needsFPReturnHelper`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `needsFPReturnHelper`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 175-178
```cpp
static bool needsFPReturnHelper(FunctionType &FT) {
  Type* RetType = FT.getReturnType();
  return whichFPReturnVariant(RetType) != NoFPRet;
}
```
- EN: Implements `needsFPReturnHelper`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `needsFPReturnHelper`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 180-182
```cpp
static bool needsFPHelperFromSig(Function &F) {
  return needsFPStubFromParams(F) || needsFPReturnHelper(F);
}
```
- EN: Implements `needsFPHelperFromSig`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `needsFPHelperFromSig`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 184-189
```cpp
// We swap between FP and Integer registers to allow Mips16 and Mips32 to
// interoperate
static std::string swapFPIntParams(FPParamVariant PV, Module *M, bool LE,
                                   bool ToFP) {
  std::string MI = ToFP ? "mtc1 ": "mfc1 ";
  std::string AsmText;
```
- EN: Implements `swapFPIntParams`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `swapFPIntParams`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 191-194
```cpp
  switch (PV) {
  case FSig:
    AsmText += MI + "$$4, $$f12\n";
    break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 196-199
```cpp
  case FFSig:
    AsmText += MI + "$$4, $$f12\n";
    AsmText += MI + "$$5, $$f14\n";
    break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 201-210
```cpp
  case FDSig:
    AsmText += MI + "$$4, $$f12\n";
    if (LE) {
      AsmText += MI + "$$6, $$f14\n";
      AsmText += MI + "$$7, $$f15\n";
    } else {
      AsmText += MI + "$$7, $$f14\n";
      AsmText += MI + "$$6, $$f15\n";
    }
    break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 212-220
```cpp
  case DSig:
    if (LE) {
      AsmText += MI + "$$4, $$f12\n";
      AsmText += MI + "$$5, $$f13\n";
    } else {
      AsmText += MI + "$$5, $$f12\n";
      AsmText += MI + "$$4, $$f13\n";
    }
    break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 222-234
```cpp
  case DDSig:
    if (LE) {
      AsmText += MI + "$$4, $$f12\n";
      AsmText += MI + "$$5, $$f13\n";
      AsmText += MI + "$$6, $$f14\n";
      AsmText += MI + "$$7, $$f15\n";
    } else {
      AsmText += MI + "$$5, $$f12\n";
      AsmText += MI + "$$4, $$f13\n";
      AsmText += MI + "$$7, $$f14\n";
      AsmText += MI + "$$6, $$f15\n";
    }
    break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 236-245
```cpp
  case DFSig:
    if (LE) {
      AsmText += MI + "$$4, $$f12\n";
      AsmText += MI + "$$5, $$f13\n";
    } else {
      AsmText += MI + "$$5, $$f12\n";
      AsmText += MI + "$$4, $$f13\n";
    }
    AsmText += MI + "$$6, $$f14\n";
    break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 247-249
```cpp
  case NoSig:
    break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 251-252
```cpp
  return AsmText;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 254-268
```cpp
// Make sure that we know we already need a stub for this function.
// Having called needsFPHelperFromSig
static void assureFPCallStub(Function &F, Module *M,
                             const MipsTargetMachine &TM) {
  // for now we only need them for static relocation
  if (TM.isPositionIndependent())
    return;
  LLVMContext &Context = M->getContext();
  bool LE = TM.isLittleEndian();
  std::string Name(F.getName());
  std::string SectionName = ".mips16.call.fp." + Name;
  std::string StubName = "__call_stub_fp_" + Name;
  //
  // see if we already have the stub
  //
```
- EN: Implements `assureFPCallStub`, a target-specific routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `assureFPCallStub`，它是一个围绕目标机器策略展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 269-281
```cpp
  Function *FStub = M->getFunction(StubName);
  if (FStub && !FStub->isDeclaration()) return;
  FStub = Function::Create(F.getFunctionType(),
                           Function::InternalLinkage, StubName, M);
  FStub->addFnAttr("mips16_fp_stub");
  FStub->addFnAttr(Attribute::Naked);
  FStub->addFnAttr(Attribute::NoInline);
  FStub->addFnAttr(Attribute::NoUnwind);
  FStub->addFnAttr("nomips16");
  FStub->setSection(SectionName);
  BasicBlock *BB = BasicBlock::Create(Context, "entry", FStub);
  FPReturnVariant RV = whichFPReturnVariant(FStub->getReturnType());
  FPParamVariant PV = whichFPParamVariantNeeded(F);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 283-292
```cpp
  std::string AsmText;
  AsmText += ".set reorder\n";
  AsmText += swapFPIntParams(PV, M, LE, true);
  if (RV != NoFPRet) {
    AsmText += "move $$18, $$31\n";
    AsmText += "jal " + Name + "\n";
  } else {
    AsmText += "lui  $$25, %hi(" + Name + ")\n";
    AsmText += "addiu  $$25, $$25, %lo(" + Name + ")\n";
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 294-297
```cpp
  switch (RV) {
  case FRet:
    AsmText += "mfc1 $$2, $$f0\n";
    break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 299-307
```cpp
  case DRet:
    if (LE) {
      AsmText += "mfc1 $$2, $$f0\n";
      AsmText += "mfc1 $$3, $$f1\n";
    } else {
      AsmText += "mfc1 $$3, $$f0\n";
      AsmText += "mfc1 $$2, $$f1\n";
    }
    break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 309-317
```cpp
  case CFRet:
    if (LE) {
      AsmText += "mfc1 $$2, $$f0\n";
      AsmText += "mfc1 $$3, $$f2\n";
    } else {
      AsmText += "mfc1 $$3, $$f0\n";
      AsmText += "mfc1 $$3, $$f2\n";
    }
    break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 319-324
```cpp
  case CDRet:
    if (LE) {
      AsmText += "mfc1 $$4, $$f2\n";
      AsmText += "mfc1 $$5, $$f3\n";
      AsmText += "mfc1 $$2, $$f0\n";
      AsmText += "mfc1 $$3, $$f1\n";
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 326-332
```cpp
    } else {
      AsmText += "mfc1 $$5, $$f2\n";
      AsmText += "mfc1 $$4, $$f3\n";
      AsmText += "mfc1 $$3, $$f0\n";
      AsmText += "mfc1 $$2, $$f1\n";
    }
    break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 334-336
```cpp
  case NoFPRet:
    break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 338-342
```cpp
  if (RV != NoFPRet)
    AsmText += "jr $$18\n";
  else
    AsmText += "jr $$25\n";
  emitInlineAsm(Context, BB, AsmText);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 344-345
```cpp
  new UnreachableInst(Context, BB);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 347-361
```cpp
// Functions that are llvm intrinsics and don't need helpers.
static const char *const IntrinsicInline[] = {
  "fabs", "fabsf",
  "llvm.ceil.f32", "llvm.ceil.f64",
  "llvm.copysign.f32", "llvm.copysign.f64",
  "llvm.cos.f32", "llvm.cos.f64",
  "llvm.exp.f32", "llvm.exp.f64",
  "llvm.exp2.f32", "llvm.exp2.f64",
  "llvm.fabs.f32", "llvm.fabs.f64",
  "llvm.floor.f32", "llvm.floor.f64",
  "llvm.fma.f32", "llvm.fma.f64",
  "llvm.log.f32", "llvm.log.f64",
  "llvm.log10.f32", "llvm.log10.f64",
  "llvm.nearbyint.f32", "llvm.nearbyint.f64",
  "llvm.pow.f32", "llvm.pow.f64",
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 362-368
```cpp
  "llvm.powi.f32.i32", "llvm.powi.f64.i32",
  "llvm.rint.f32", "llvm.rint.f64",
  "llvm.round.f32", "llvm.round.f64",
  "llvm.sin.f32", "llvm.sin.f64",
  "llvm.sqrt.f32", "llvm.sqrt.f64",
  "llvm.trunc.f32", "llvm.trunc.f64",
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 370-372
```cpp
static bool isIntrinsicInline(Function *F) {
  return llvm::binary_search(IntrinsicInline, F->getName());
}
```
- EN: Implements `llvm::binary_search`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::binary_search`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 374-388
```cpp
// Returns of float, double and complex need to be handled with a helper
// function.
static bool fixupFPReturnAndCall(Function &F, Module *M,
                                 const MipsTargetMachine &TM) {
  bool Modified = false;
  LLVMContext &C = M->getContext();
  Type *MyVoid = Type::getVoidTy(C);
  for (auto &BB: F)
    for (auto &I: BB) {
      if (const ReturnInst *RI = dyn_cast<ReturnInst>(&I)) {
        Value *RVal = RI->getReturnValue();
        if (!RVal) continue;
        //
        // If there is a return value and it needs a helper function,
        // figure out which one and add a call before the actual
```
- EN: Implements `fixupFPReturnAndCall`, a target-specific routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `fixupFPReturnAndCall`，它是一个围绕目标机器策略展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 389-403
```cpp
        // return to this helper. The purpose of the helper is to move
        // floating point values from their soft float return mapping to
        // where they would have been mapped to in floating point registers.
        //
        Type *T = RVal->getType();
        FPReturnVariant RV = whichFPReturnVariant(T);
        if (RV == NoFPRet) continue;
        static const char *const Helper[NoFPRet] = {
          "__mips16_ret_sf", "__mips16_ret_df", "__mips16_ret_sc",
          "__mips16_ret_dc"
        };
        const char *Name = Helper[RV];
        AttributeList A;
        Value *Params[] = {RVal};
        Modified = true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 404-418
```cpp
        //
        // These helper functions have a different calling ABI so
        // this __Mips16RetHelper indicates that so that later
        // during call setup, the proper call lowering to the helper
        // functions will take place.
        //
        A = A.addFnAttribute(C, "__Mips16RetHelper");
        A = A.addFnAttribute(
            C, Attribute::getWithMemoryEffects(C, MemoryEffects::none()));
        A = A.addFnAttribute(C, Attribute::NoInline);
        FunctionCallee F = (M->getOrInsertFunction(Name, A, MyVoid, T));
        CallInst::Create(F, Params, "", I.getIterator());
      } else if (const CallInst *CI = dyn_cast<CallInst>(&I)) {
        FunctionType *FT = CI->getFunctionType();
        Function *F_ =  CI->getCalledFunction();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 419-433
```cpp
        if (needsFPReturnHelper(*FT) &&
            !(F_ && isIntrinsicInline(F_))) {
          Modified=true;
          F.addFnAttr("saveS2");
        }
        if (F_ && !isIntrinsicInline(F_)) {
          // pic mode calls are handled by already defined
          // helper functions
          if (needsFPReturnHelper(*F_)) {
            Modified=true;
            F.addFnAttr("saveS2");
          }
          if (!TM.isPositionIndependent()) {
            if (needsFPHelperFromSig(*F_)) {
              assureFPCallStub(*F_, M, TM);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 434-441
```cpp
              Modified=true;
            }
          }
        }
      }
    }
  return Modified;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 443-457
```cpp
static void createFPFnStub(Function *F, Module *M, FPParamVariant PV,
                           const MipsTargetMachine &TM) {
  bool PicMode = TM.isPositionIndependent();
  bool LE = TM.isLittleEndian();
  LLVMContext &Context = M->getContext();
  std::string Name(F->getName());
  std::string SectionName = ".mips16.fn." + Name;
  std::string StubName = "__fn_stub_" + Name;
  std::string LocalName = "$$__fn_local_" + Name;
  Function *FStub = Function::Create
    (F->getFunctionType(),
     Function::InternalLinkage, StubName, M);
  FStub->addFnAttr("mips16_fp_stub");
  FStub->addFnAttr(Attribute::Naked);
  FStub->addFnAttr(Attribute::NoUnwind);
```
- EN: Implements `createFPFnStub`, a query/helper routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `createFPFnStub`，它是一个围绕目标机器策略展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 458-461
```cpp
  FStub->addFnAttr(Attribute::NoInline);
  FStub->addFnAttr("nomips16");
  FStub->setSection(SectionName);
  BasicBlock *BB = BasicBlock::Create(Context, "entry", FStub);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 463-475
```cpp
  std::string AsmText;
  if (PicMode) {
    AsmText += ".set noreorder\n";
    AsmText += ".cpload $$25\n";
    AsmText += ".set reorder\n";
    AsmText += ".reloc 0, R_MIPS_NONE, " + Name + "\n";
    AsmText += "la $$25, " + LocalName + "\n";
  } else
    AsmText += "la $$25, " + Name + "\n";
  AsmText += swapFPIntParams(PV, M, LE, false);
  AsmText += "jr $$25\n";
  AsmText += LocalName + " = " + Name + "\n";
  emitInlineAsm(Context, BB, AsmText);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 477-478
```cpp
  new UnreachableInst(FStub->getContext(), BB);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 480-488
```cpp
// remove the use-soft-float attribute
static void removeUseSoftFloat(Function &F) {
  LLVM_DEBUG(errs() << "removing -use-soft-float\n");
  F.removeFnAttr("use-soft-float");
  if (F.hasFnAttribute("use-soft-float")) {
    LLVM_DEBUG(errs() << "still has -use-soft-float\n");
  }
  F.addFnAttr("use-soft-float", "false");
}
```
- EN: Implements `removeUseSoftFloat`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `removeUseSoftFloat`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 490-504
```cpp
// This pass only makes sense when the underlying chip has floating point but
// we are compiling as mips16.
// For all mips16 functions (that are not stubs we have already generated), or
// declared via attributes as nomips16, we must:
//    1) fixup all returns of float, double, single and double complex
//       by calling a helper function before the actual return.
//    2) generate helper functions (stubs) that can be called by mips32
//       functions that will move parameters passed normally passed in
//       floating point
//       registers the soft float equivalents.
//    3) in the case of static relocation, generate helper functions so that
//       mips16 functions can call extern functions of unknown type (mips16 or
//       mips32).
//    4) TBD. For pic, calls to extern functions of unknown type are handled by
//       predefined helper functions in libc but this work is currently done
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 505-519
```cpp
//       during call lowering but it should be moved here in the future.
bool Mips16HardFloat::runOnModule(Module &M) {
  auto &TM = static_cast<const MipsTargetMachine &>(
      getAnalysis<TargetPassConfig>().getTM<TargetMachine>());
  LLVM_DEBUG(errs() << "Run on Module Mips16HardFloat\n");
  bool Modified = false;
  for (Module::iterator F = M.begin(), E = M.end(); F != E; ++F) {
    if (F->hasFnAttribute("nomips16") &&
        F->hasFnAttribute("use-soft-float")) {
      removeUseSoftFloat(*F);
      continue;
    }
    if (F->isDeclaration() || F->hasFnAttribute("mips16_fp_stub") ||
        F->hasFnAttribute("nomips16")) continue;
    Modified |= fixupFPReturnAndCall(*F, &M, TM);
```
- EN: Implements `Mips16HardFloat::runOnModule`, a pass-entry routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips16HardFloat::runOnModule`，它是一个围绕目标机器策略展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 520-527
```cpp
    FPParamVariant V = whichFPParamVariantNeeded(*F);
    if (V != NoSig) {
      Modified = true;
      createFPFnStub(&*F, &M, V, TM);
    }
  }
  return Modified;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 529-531
```cpp
ModulePass *llvm::createMips16HardFloatPass() {
  return new Mips16HardFloat();
}
```
- EN: Implements `llvm::createMips16HardFloatPass`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::createMips16HardFloatPass`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

## Key Concepts / 关键概念

- EN: Primary role: target-specific code-generation behavior.
  - CN: 核心职责：目标相关代码生成行为。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。

## Dependencies / 依赖关系

- EN: Backend-local headers: `MipsTargetMachine.h`.
  - CN: 后端本地头文件：`MipsTargetMachine.h`。
- EN: LLVM infrastructure headers: `llvm/CodeGen/TargetPassConfig.h`, `llvm/IR/Module.h`, `llvm/IR/Value.h`, `llvm/Support/Debug.h`, `llvm/Support/ModRef.h`, `llvm/Support/raw_ostream.h`.
  - CN: LLVM 基础设施头文件：`llvm/CodeGen/TargetPassConfig.h`, `llvm/IR/Module.h`, `llvm/IR/Value.h`, `llvm/Support/Debug.h`, `llvm/Support/ModRef.h`, `llvm/Support/raw_ostream.h`。
- EN: Standard/system headers: `string`.
  - CN: 标准库/系统头文件：`string`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
