# AMDGPUCtorDtorLowering.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUCtorDtorLowering.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUCtorDtorLowering for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPUCtorDtorLowering 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: File banner, includes, and setup
```cpp
//===-- AMDGPUCtorDtorLowering.cpp - Handle global ctors and dtors --------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///
/// \file
/// This pass creates a unified init and fini kernel with the required metadata
//===----------------------------------------------------------------------===//

#include "AMDGPUCtorDtorLowering.h"
#include "AMDGPU.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/GlobalVariable.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/Module.h"
#include "llvm/IR/Value.h"
#include "llvm/Pass.h"
#include "llvm/Transforms/Utils/ModuleUtils.h"

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 24-36: Preprocessor guards and macros
```cpp
using namespace llvm;

#define DEBUG_TYPE "amdgpu-lower-ctor-dtor"

namespace {

static Function *createInitOrFiniKernelFunction(Module &M, bool IsCtor) {
  StringRef InitOrFiniKernelName = "amdgcn.device.init";
  if (!IsCtor)
    InitOrFiniKernelName = "amdgcn.device.fini";
  if (M.getFunction(InitOrFiniKernelName))
    return nullptr;

```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

### Lines 37-60: Implements Function::createWithDefaultAttr
```cpp
  Function *InitOrFiniKernel = Function::createWithDefaultAttr(
      FunctionType::get(Type::getVoidTy(M.getContext()), false),
      GlobalValue::WeakODRLinkage, 0, InitOrFiniKernelName, &M);
  InitOrFiniKernel->setCallingConv(CallingConv::AMDGPU_KERNEL);
  InitOrFiniKernel->addFnAttr("amdgpu-flat-work-group-size", "1,1");
  if (IsCtor)
    InitOrFiniKernel->addFnAttr("device-init");
  else
    InitOrFiniKernel->addFnAttr("device-fini");
  return InitOrFiniKernel;
}

// The linker will provide the associated symbols to allow us to traverse the
// global constructors / destructors in priority order. We create the IR
// required to call each callback in this section. This is equivalent to the
// following code.
//
// extern "C" void * __init_array_start[];
// extern "C" void * __init_array_end[];
// extern "C" void * __fini_array_start[];
// extern "C" void * __fini_array_end[];
//
// using InitCallback = void();
// using FiniCallback = void(void);
```
**EN:** This section contains concrete logic for Function::createWithDefaultAttr. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `Function::createWithDefaultAttr`, `FunctionType::get`, `Type::getVoidTy`.
**CN:** 本节包含与 Function::createWithDefaultAttr 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`Function::createWithDefaultAttr`, `FunctionType::get`, `Type::getVoidTy`。

### Lines 61-81: Defines createInitOrFiniCalls
```cpp
//
// void call_init_array_callbacks() {
//   for (auto start = __init_array_start; start != __init_array_end; ++start)
//     reinterpret_cast<InitCallback *>(*start)();
// }
//
// void call_fini_array_callbacks() {
//  size_t fini_array_size = __fini_array_end - __fini_array_start;
//  for (size_t i = fini_array_size; i > 0; --i)
//    reinterpret_cast<FiniCallback *>(__fini_array_start[i - 1])();
// }
static void createInitOrFiniCalls(Function &F, bool IsCtor) {
  Module &M = *F.getParent();
  LLVMContext &C = M.getContext();

  IRBuilder<> IRB(BasicBlock::Create(C, "entry", &F));
  auto *LoopBB = BasicBlock::Create(C, "while.entry", &F);
  auto *ExitBB = BasicBlock::Create(C, "while.end", &F);
  Type *PtrTy = IRB.getPtrTy(AMDGPUAS::GLOBAL_ADDRESS);
  ArrayType *PtrArrayTy = ArrayType::get(PtrTy, 0);

```
**EN:** This section contains concrete logic for createInitOrFiniCalls. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `BasicBlock::Create`, `ArrayType::get`.
**CN:** 本节包含与 createInitOrFiniCalls 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`BasicBlock::Create`, `ArrayType::get`。

### Lines 82-102: Defines getOrInsertGlobal
```cpp
  auto *Begin = M.getOrInsertGlobal(
      IsCtor ? "__init_array_start" : "__fini_array_start", PtrArrayTy, [&]() {
        return new GlobalVariable(
            M, PtrArrayTy,
            /*isConstant=*/true, GlobalValue::ExternalLinkage,
            /*Initializer=*/nullptr,
            IsCtor ? "__init_array_start" : "__fini_array_start",
            /*InsertBefore=*/nullptr, GlobalVariable::NotThreadLocal,
            /*AddressSpace=*/AMDGPUAS::GLOBAL_ADDRESS);
      });
  auto *End = M.getOrInsertGlobal(
      IsCtor ? "__init_array_end" : "__fini_array_end", PtrArrayTy, [&]() {
        return new GlobalVariable(
            M, PtrArrayTy,
            /*isConstant=*/true, GlobalValue::ExternalLinkage,
            /*Initializer=*/nullptr,
            IsCtor ? "__init_array_end" : "__fini_array_end",
            /*InsertBefore=*/nullptr, GlobalVariable::NotThreadLocal,
            /*AddressSpace=*/AMDGPUAS::GLOBAL_ADDRESS);
      });

```
**EN:** This section contains concrete logic for getOrInsertGlobal. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 getOrInsertGlobal 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 103-116: Implements FunctionType::get
```cpp
  // The constructor type is suppoed to allow using the argument vectors, but
  // for now we just call them with no arguments.
  auto *CallBackTy = FunctionType::get(IRB.getVoidTy(), {});

  Value *Start = Begin;
  Value *Stop = End;
  // The destructor array must be called in reverse order. Get a constant
  // expression to the end of the array and iterate backwards instead.
  if (!IsCtor) {
    Start = IRB.CreateInBoundsGEP(
        PtrTy, End, ConstantInt::getAllOnesValue(IRB.getInt64Ty()));
    Stop = Begin;
  }

```
**EN:** This section contains concrete logic for FunctionType::get. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `FunctionType::get`, `ConstantInt::getAllOnesValue`.
**CN:** 本节包含与 FunctionType::get 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`FunctionType::get`, `ConstantInt::getAllOnesValue`。

### Lines 117-135: Defines CreateCondBr
```cpp
  IRB.CreateCondBr(
      IRB.CreateCmp(IsCtor ? ICmpInst::ICMP_NE : ICmpInst::ICMP_UGE, Start,
                    Stop),
      LoopBB, ExitBB);
  IRB.SetInsertPoint(LoopBB);
  auto *CallBackPHI = IRB.CreatePHI(PtrTy, 2, "ptr");
  auto *CallBack = IRB.CreateLoad(F.getType(), CallBackPHI, "callback");
  IRB.CreateCall(CallBackTy, CallBack);
  auto *NewCallBack =
      IRB.CreateConstGEP1_64(PtrTy, CallBackPHI, IsCtor ? 1 : -1, "next");
  auto *EndCmp = IRB.CreateCmp(IsCtor ? ICmpInst::ICMP_EQ : ICmpInst::ICMP_ULT,
                               NewCallBack, Stop, "end");
  CallBackPHI->addIncoming(Start, &F.getEntryBlock());
  CallBackPHI->addIncoming(NewCallBack, LoopBB);
  IRB.CreateCondBr(EndCmp, ExitBB, LoopBB);
  IRB.SetInsertPoint(ExitBB);
  IRB.CreateRetVoid();
}

```
**EN:** This section contains concrete logic for CreateCondBr. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 CreateCondBr 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 136-154: Defines createInitOrFiniKernel
```cpp
static bool createInitOrFiniKernel(Module &M, StringRef GlobalName,
                                   bool IsCtor) {
  GlobalVariable *GV = M.getGlobalVariable(GlobalName);
  if (!GV || !GV->hasInitializer())
    return false;
  ConstantArray *GA = dyn_cast<ConstantArray>(GV->getInitializer());
  if (!GA || GA->getNumOperands() == 0)
    return false;

  Function *InitOrFiniKernel = createInitOrFiniKernelFunction(M, IsCtor);
  if (!InitOrFiniKernel)
    return false;

  createInitOrFiniCalls(*InitOrFiniKernel, IsCtor);

  appendToUsed(M, {InitOrFiniKernel});
  return true;
}

```
**EN:** This section contains concrete logic for createInitOrFiniKernel. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 createInitOrFiniKernel 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 155-176: Declares class AMDGPUCtorDtorLoweringLegacy
```cpp
static bool lowerCtorsAndDtors(Module &M) {
  bool Modified = false;
  Modified |= createInitOrFiniKernel(M, "llvm.global_ctors", /*IsCtor =*/true);
  Modified |= createInitOrFiniKernel(M, "llvm.global_dtors", /*IsCtor =*/false);
  return Modified;
}

class AMDGPUCtorDtorLoweringLegacy final : public ModulePass {
public:
  static char ID;
  AMDGPUCtorDtorLoweringLegacy() : ModulePass(ID) {}
  bool runOnModule(Module &M) override { return lowerCtorsAndDtors(M); }
};

} // End anonymous namespace

PreservedAnalyses AMDGPUCtorDtorLoweringPass::run(Module &M,
                                                  ModuleAnalysisManager &AM) {
  return lowerCtorsAndDtors(M) ? PreservedAnalyses::none()
                               : PreservedAnalyses::all();
}

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUCtorDtorLoweringLegacy`, `AMDGPUCtorDtorLoweringPass::run`, `PreservedAnalyses::none`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUCtorDtorLoweringLegacy`, `AMDGPUCtorDtorLoweringPass::run`, `PreservedAnalyses::none`。

### Lines 177-185: Registers LLVM passes
```cpp
char AMDGPUCtorDtorLoweringLegacy::ID = 0;
char &llvm::AMDGPUCtorDtorLoweringLegacyPassID =
    AMDGPUCtorDtorLoweringLegacy::ID;
INITIALIZE_PASS(AMDGPUCtorDtorLoweringLegacy, DEBUG_TYPE,
                "Lower ctors and dtors for AMDGPU", false, false)

ModulePass *llvm::createAMDGPUCtorDtorLoweringLegacyPass() {
  return new AMDGPUCtorDtorLoweringLegacy();
}
```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `llvm::createAMDGPUCtorDtorLoweringLegacyPass`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`llvm::createAMDGPUCtorDtorLoweringLegacyPass`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `AMDGPUCtorDtorLoweringLegacy`, `Function::createWithDefaultAttr`, `FunctionType::get`, `Type::getVoidTy`, `BasicBlock::Create`, `ArrayType::get`
- **Main themes / 核心主题**: lowering / 降低; LLVM pass integration / LLVM Pass 集成
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPUCtorDtorLowering.h"`
- `"AMDGPU.h"`
- `"llvm/IR/Constants.h"`
- `"llvm/IR/Function.h"`
- `"llvm/IR/GlobalVariable.h"`
- `"llvm/IR/IRBuilder.h"`
- `"llvm/IR/Module.h"`
- `"llvm/IR/Value.h"`
- `"llvm/Pass.h"`
- `"llvm/Transforms/Utils/ModuleUtils.h"`
