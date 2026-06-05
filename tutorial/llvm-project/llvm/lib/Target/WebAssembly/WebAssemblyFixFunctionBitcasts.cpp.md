# WebAssemblyFixFunctionBitcasts.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyFixFunctionBitcasts.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Fix bitcasted functions.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyFixFunctionBitcasts.cpp`，主要负责 WebAssembly 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- WebAssemblyFixFunctionBitcasts.cpp - Fix function bitcasts --------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-22

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// Fix bitcasted functions.
///
/// WebAssembly requires caller and callee signatures to match, however in LLVM,
/// some amount of slop is vaguely permitted. Detect mismatch by looking for
/// bitcasts of functions and rewrite them to use wrapper functions instead.
///
/// This doesn't catch all cases, such as when a function's address is taken in
/// one place and casted in another, but it works for many common cases.
///
/// Note that LLVM already optimizes away function bitcasts in common cases by
/// dropping arguments as needed, so this pass only ends up getting used in less
/// common cases.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 23-29

```cpp
//===----------------------------------------------------------------------===//

#include "WebAssembly.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/Module.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 30-37

```cpp
#include "llvm/IR/Operator.h"
#include "llvm/Pass.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"
using namespace llvm;

#define DEBUG_TYPE "wasm-fix-function-bitcasts"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 38-75

```cpp
namespace {
class FixFunctionBitcasts final : public ModulePass {
  StringRef getPassName() const override {
    return "WebAssembly Fix Function Bitcasts";
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    ModulePass::getAnalysisUsage(AU);
  }

  bool runOnModule(Module &M) override;

public:
  static char ID;
  FixFunctionBitcasts() : ModulePass(ID) {}
};
} // End anonymous namespace

char FixFunctionBitcasts::ID = 0;
INITIALIZE_PASS(FixFunctionBitcasts, DEBUG_TYPE,
                "Fix mismatching bitcasts for WebAssembly", false, false)

ModulePass *llvm::createWebAssemblyFixFunctionBitcasts() {
  return new FixFunctionBitcasts();
}

// Recursively descend the def-use lists from V to find non-bitcast users of
// bitcasts of V.
static void findUses(Value *V, Function &F,
                     SmallVectorImpl<std::pair<CallBase *, Function *>> &Uses) {
  for (User *U : V->users()) {
    if (auto *BC = dyn_cast<BitCastOperator>(U))
      findUses(BC, F, Uses);
    else if (auto *A = dyn_cast<GlobalAlias>(U))
      findUses(A, F, Uses);
    else if (auto *CB = dyn_cast<CallBase>(U)) {
      Value *Callee = CB->getCalledOperand();
```
- **EN**: Declares a backend-facing type `FixFunctionBitcasts`, `getPassName`, `getAnalysisUsage` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `FixFunctionBitcasts`, `getPassName`, `getAnalysisUsage`，并勾勒出周边代码会依赖的接口或状态。

### Lines 76-113

```cpp
      if (Callee != V)
        // Skip calls where the function isn't the callee
        continue;
      if (CB->getFunctionType() == F.getFunctionType())
        // Skip uses that are immediately called
        continue;
      Uses.push_back(std::make_pair(CB, &F));
    }
  }
}

// Create a wrapper function with type Ty that calls F (which may have a
// different type). Attempt to support common bitcasted function idioms:
//  - Call with more arguments than needed: arguments are dropped
//  - Call with fewer arguments than needed: arguments are filled in with poison
//  - Return value is not needed: drop it
//  - Return value needed but not present: supply a poison value
//
// If the all the argument types of trivially castable to one another (i.e.
// I32 vs pointer type) then we don't create a wrapper at all (return nullptr
// instead).
//
// If there is a type mismatch that we know would result in an invalid wasm
// module then generate wrapper that contains unreachable (i.e. abort at
// runtime).  Such programs are deep into undefined behaviour territory,
// but we choose to fail at runtime rather than generate and invalid module
// or fail at compiler time.  The reason we delay the error is that we want
// to support the CMake which expects to be able to compile and link programs
// that refer to functions with entirely incorrect signatures (this is how
// CMake detects the existence of a function in a toolchain).
//
// For bitcasts that involve struct types we don't know at this stage if they
// would be equivalent at the wasm level and so we can't know if we need to
// generate a wrapper.
static Function *createWrapper(Function *F, FunctionType *Ty) {
  Module *M = F->getParent();

  Function *Wrapper = Function::Create(Ty, Function::PrivateLinkage,
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Skip calls where the function isn't the callee". Notable symbols in this range include `getFunctionType`, `push_back`, `make_pair`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Skip calls where the function isn't the callee”。 该区间中较显眼的符号包括 `getFunctionType`, `push_back`, `make_pair`。

### Lines 114-151

```cpp
                                       F->getName() + "_bitcast", M);
  Wrapper->setAttributes(F->getAttributes());
  BasicBlock *BB = BasicBlock::Create(M->getContext(), "body", Wrapper);
  const DataLayout &DL = BB->getDataLayout();
  IRBuilder<> Builder(BB);

  // Determine what arguments to pass.
  SmallVector<Value *, 4> Args;
  Function::arg_iterator AI = Wrapper->arg_begin();
  Function::arg_iterator AE = Wrapper->arg_end();
  FunctionType::param_iterator PI = F->getFunctionType()->param_begin();
  FunctionType::param_iterator PE = F->getFunctionType()->param_end();
  bool TypeMismatch = false;
  bool WrapperNeeded = false;

  Type *ExpectedRtnType = F->getFunctionType()->getReturnType();
  Type *RtnType = Ty->getReturnType();

  if ((F->getFunctionType()->getNumParams() != Ty->getNumParams()) ||
      (F->getFunctionType()->isVarArg() != Ty->isVarArg()) ||
      (ExpectedRtnType != RtnType))
    WrapperNeeded = true;

  for (; AI != AE && PI != PE; ++AI, ++PI) {
    Type *ArgType = AI->getType();
    Type *ParamType = *PI;

    if (ArgType == ParamType) {
      Args.push_back(&*AI);
    } else {
      if (CastInst::isBitOrNoopPointerCastable(ArgType, ParamType, DL)) {
        Args.push_back(Builder.CreateBitOrPointerCast(AI, ParamType, "cast"));
      } else if (ArgType->isStructTy() || ParamType->isStructTy()) {
        LLVM_DEBUG(dbgs() << "createWrapper: struct param type in bitcast: "
                          << F->getName() << "\n");
        WrapperNeeded = false;
      } else {
        LLVM_DEBUG(dbgs() << "createWrapper: arg type mismatch calling: "
```
- **EN**: Implements helper routine(s) `getName`, `setAttributes`, `getAttributes` for this portion of the WebAssembly backend backend implementation logic.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `getName`, `setAttributes`, `getAttributes`。

### Lines 152-189

```cpp
                          << F->getName() << "\n");
        LLVM_DEBUG(dbgs() << "Arg[" << Args.size() << "] Expected: "
                          << *ParamType << " Got: " << *ArgType << "\n");
        TypeMismatch = true;
        break;
      }
    }
  }

  if (WrapperNeeded && !TypeMismatch) {
    for (; PI != PE; ++PI)
      Args.push_back(PoisonValue::get(*PI));
    if (F->isVarArg())
      for (; AI != AE; ++AI)
        Args.push_back(&*AI);

    CallInst *Call = Builder.CreateCall(F, Args);

    // Determine what value to return.
    if (RtnType->isVoidTy()) {
      Builder.CreateRetVoid();
    } else if (ExpectedRtnType->isVoidTy()) {
      LLVM_DEBUG(dbgs() << "Creating dummy return: " << *RtnType << "\n");
      Builder.CreateRet(PoisonValue::get(RtnType));
    } else if (RtnType == ExpectedRtnType) {
      Builder.CreateRet(Call);
    } else if (CastInst::isBitOrNoopPointerCastable(ExpectedRtnType, RtnType,
                                                    DL)) {
      Builder.CreateRet(Builder.CreateBitOrPointerCast(Call, RtnType, "cast"));
    } else if (RtnType->isStructTy() || ExpectedRtnType->isStructTy()) {
      LLVM_DEBUG(dbgs() << "createWrapper: struct return type in bitcast: "
                        << F->getName() << "\n");
      WrapperNeeded = false;
    } else {
      LLVM_DEBUG(dbgs() << "createWrapper: return type mismatch calling: "
                        << F->getName() << "\n");
      LLVM_DEBUG(dbgs() << "Expected: " << *ExpectedRtnType
                        << " Got: " << *RtnType << "\n");
```
- **EN**: Implements helper routine(s) `getName`, `dbgs`, `size` for this portion of the WebAssembly backend backend implementation logic.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `getName`, `dbgs`, `size`。

### Lines 190-227

```cpp
      TypeMismatch = true;
    }
  }

  if (TypeMismatch) {
    // Create a new wrapper that simply contains `unreachable`.
    Wrapper->eraseFromParent();
    Wrapper = Function::Create(Ty, Function::PrivateLinkage,
                               F->getName() + "_bitcast_invalid", M);
    Wrapper->setAttributes(F->getAttributes());
    IRBuilder<> Builder(BasicBlock::Create(M->getContext(), "body", Wrapper));
    Builder.CreateUnreachable();
  } else if (!WrapperNeeded) {
    LLVM_DEBUG(dbgs() << "createWrapper: no wrapper needed: " << F->getName()
                      << "\n");
    Wrapper->eraseFromParent();
    return nullptr;
  }
  LLVM_DEBUG(dbgs() << "createWrapper: " << F->getName() << "\n");
  return Wrapper;
}

// Test whether a main function with type FuncTy should be rewritten to have
// type MainTy.
static bool shouldFixMainFunction(FunctionType *FuncTy, FunctionType *MainTy) {
  // Only fix the main function if it's the standard zero-arg form. That way,
  // the standard cases will work as expected, and users will see signature
  // mismatches from the linker for non-standard cases.
  return FuncTy->getReturnType() == MainTy->getReturnType() &&
         FuncTy->getNumParams() == 0 &&
         !FuncTy->isVarArg();
}

bool FixFunctionBitcasts::runOnModule(Module &M) {
  LLVM_DEBUG(dbgs() << "********** Fix Function Bitcasts **********\n");

  Function *Main = nullptr;
  CallInst *CallMain = nullptr;
```
- **EN**: Implements helper routine(s) `eraseFromParent`, `Create`, `getName` for this portion of the WebAssembly backend backend implementation logic.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `eraseFromParent`, `Create`, `getName`。

### Lines 228-265

```cpp
  SmallVector<std::pair<CallBase *, Function *>, 0> Uses;

  // Collect all the places that need wrappers.
  for (Function &F : M) {
    // Skip to fix when the function is swiftcc or swifttailcc because these
    // calling conventions allow bitcast type difference for swiftself,
    // swifterror, and swiftasync.
    if (F.getCallingConv() == CallingConv::Swift ||
        F.getCallingConv() == CallingConv::SwiftTail)
      continue;
    findUses(&F, F, Uses);

    // If we have a "main" function, and its type isn't
    // "int main(int argc, char *argv[])", create an artificial call with it
    // bitcasted to that type so that we generate a wrapper for it, so that
    // the C runtime can call it.
    if (F.getName() == "main") {
      Main = &F;
      LLVMContext &C = M.getContext();
      Type *MainArgTys[] = {Type::getInt32Ty(C), PointerType::get(C, 0)};
      FunctionType *MainTy = FunctionType::get(Type::getInt32Ty(C), MainArgTys,
                                               /*isVarArg=*/false);
      if (shouldFixMainFunction(F.getFunctionType(), MainTy)) {
        LLVM_DEBUG(dbgs() << "Found `main` function with incorrect type: "
                          << *F.getFunctionType() << "\n");
        Value *Args[] = {PoisonValue::get(MainArgTys[0]),
                         PoisonValue::get(MainArgTys[1])};
        CallMain = CallInst::Create(MainTy, Main, Args, "call_main");
        Uses.push_back(std::make_pair(CallMain, &F));
      }
    }
  }

  DenseMap<std::pair<Function *, FunctionType *>, Function *> Wrappers;

  for (auto &UseFunc : Uses) {
    CallBase *CB = UseFunc.first;
    Function *F = UseFunc.second;
```
- **EN**: Implements helper routine(s) `getCallingConv`, `findUses`, `main` for this portion of the WebAssembly backend backend implementation logic. ABI and calling-convention details are important in this part of the code.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `getCallingConv`, `findUses`, `main`。 ABI 与调用约定细节是这一部分代码的重要约束。

### Lines 266-300

```cpp
    FunctionType *Ty = CB->getFunctionType();

    auto Pair = Wrappers.try_emplace(std::make_pair(F, Ty));
    if (Pair.second)
      Pair.first->second = createWrapper(F, Ty);

    Function *Wrapper = Pair.first->second;
    if (!Wrapper)
      continue;

    CB->setCalledOperand(Wrapper);
  }

  // If we created a wrapper for main, rename the wrapper so that it's the
  // one that gets called from startup.
  if (CallMain) {
    Main->setName("__original_main");
    auto *MainWrapper =
        cast<Function>(CallMain->getCalledOperand()->stripPointerCasts());
    delete CallMain;
    if (Main->isDeclaration()) {
      // The wrapper is not needed in this case as we don't need to export
      // it to anyone else.
      MainWrapper->eraseFromParent();
    } else {
      // Otherwise give the wrapper the same linkage as the original main
      // function, so that it can be called from the same places.
      MainWrapper->setName("main");
      MainWrapper->setLinkage(Main->getLinkage());
      MainWrapper->setVisibility(Main->getVisibility());
    }
  }

  return true;
}
```
- **EN**: Implements helper routine(s) `getFunctionType`, `try_emplace`, `make_pair` for this portion of the WebAssembly backend backend implementation logic.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `getFunctionType`, `try_emplace`, `make_pair`。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Backend implementation logic / 后端实现逻辑
- Calling convention handling / 调用约定处理
- Vector or SIMD behavior / 向量或 SIMD 行为
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `WebAssembly.h`
- `llvm/IR/Constants.h`
- `llvm/IR/IRBuilder.h`
- `llvm/IR/Instructions.h`
- `llvm/IR/Module.h`
- `llvm/IR/Operator.h`
- `llvm/Pass.h`
- `llvm/Support/Debug.h`
- `llvm/Support/raw_ostream.h`

### Important Collaborators / 重要协作组件

- LLVM Support utilities / LLVM Support 工具层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
