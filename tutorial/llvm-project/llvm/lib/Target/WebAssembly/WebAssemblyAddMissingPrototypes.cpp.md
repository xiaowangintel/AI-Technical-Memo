# WebAssemblyAddMissingPrototypes.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyAddMissingPrototypes.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Add prototypes to prototypes-less functions.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyAddMissingPrototypes.cpp`，主要负责 WebAssembly 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- WebAssemblyAddMissingPrototypes.cpp - Fix prototypeless functions -===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-18

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// Add prototypes to prototypes-less functions.
///
/// WebAssembly has strict function prototype checking so we need functions
/// declarations to match the call sites.  Clang treats prototype-less functions
/// as varargs (foo(...)) which happens to work on existing platforms but
/// doesn't under WebAssembly.  This pass will find all the call sites of each
/// prototype-less function, ensure they agree, and then set the signature
/// on the function declaration accordingly.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file". Notable symbols in this range include `varargs`, `foo`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。 该区间中较显眼的符号包括 `varargs`, `foo`。

### Lines 19-25

```cpp
//===----------------------------------------------------------------------===//

#include "WebAssembly.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/Module.h"
#include "llvm/IR/Operator.h"
#include "llvm/Pass.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 26-32

```cpp
#include "llvm/Support/Debug.h"
#include "llvm/Transforms/Utils/Local.h"
#include "llvm/Transforms/Utils/ModuleUtils.h"
using namespace llvm;

#define DEBUG_TYPE "wasm-add-missing-prototypes"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 33-70

```cpp
namespace {
class WebAssemblyAddMissingPrototypes final : public ModulePass {
  StringRef getPassName() const override {
    return "Add prototypes to prototypes-less functions";
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    ModulePass::getAnalysisUsage(AU);
  }

  bool runOnModule(Module &M) override;

public:
  static char ID;
  WebAssemblyAddMissingPrototypes() : ModulePass(ID) {}
};
} // End anonymous namespace

char WebAssemblyAddMissingPrototypes::ID = 0;
INITIALIZE_PASS(WebAssemblyAddMissingPrototypes, DEBUG_TYPE,
                "Add prototypes to prototypes-less functions", false, false)

ModulePass *llvm::createWebAssemblyAddMissingPrototypes() {
  return new WebAssemblyAddMissingPrototypes();
}

bool WebAssemblyAddMissingPrototypes::runOnModule(Module &M) {
  LLVM_DEBUG(dbgs() << "********** Add Missing Prototypes **********\n");

  std::vector<std::pair<Function *, Function *>> Replacements;

  // Find all the prototype-less function declarations
  for (Function &F : M) {
    if (!F.isDeclaration() || !F.hasFnAttribute("no-prototype"))
      continue;

    LLVM_DEBUG(dbgs() << "Found no-prototype function: " << F.getName()
```
- **EN**: Declares a backend-facing type `WebAssemblyAddMissingPrototypes`, `getPassName`, `getAnalysisUsage` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `WebAssemblyAddMissingPrototypes`, `getPassName`, `getAnalysisUsage`，并勾勒出周边代码会依赖的接口或状态。

### Lines 71-108

```cpp
                      << "\n");

    // When clang emits prototype-less C functions it uses (...), i.e. varargs
    // function that take no arguments (have no sentinel).  When we see a
    // no-prototype attribute we expect the function have these properties.
    if (!F.isVarArg())
      report_fatal_error(
          "Functions with 'no-prototype' attribute must take varargs: " +
          F.getName());
    unsigned NumParams = F.getFunctionType()->getNumParams();
    if (NumParams != 0) {
      if (!(NumParams == 1 && F.arg_begin()->hasStructRetAttr()))
        report_fatal_error("Functions with 'no-prototype' attribute should "
                           "not have params: " +
                           F.getName());
    }

    // Find calls of this function, looking through bitcasts.
    SmallVector<CallBase *> Calls;
    SmallVector<Value *> Worklist;
    Worklist.push_back(&F);
    while (!Worklist.empty()) {
      Value *V = Worklist.pop_back_val();
      for (User *U : V->users()) {
        if (auto *BC = dyn_cast<BitCastOperator>(U))
          Worklist.push_back(BC);
        else if (auto *CB = dyn_cast<CallBase>(U))
          if (CB->getCalledOperand() == V)
            Calls.push_back(CB);
      }
    }

    // Create a function prototype based on the first call site that we find.
    FunctionType *NewType = nullptr;
    for (CallBase *CB : Calls) {
      LLVM_DEBUG(dbgs() << "prototype-less call of " << F.getName() << ":\n");
      LLVM_DEBUG(dbgs() << *CB << "\n");
      FunctionType *DestType = CB->getFunctionType();
```
- **EN**: Implements helper routine(s) `uses`, `arguments`, `isVarArg` for this portion of the WebAssembly backend backend implementation logic.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `uses`, `arguments`, `isVarArg`。

### Lines 109-146

```cpp
      if (!NewType) {
        // Create a new function with the correct type
        NewType = DestType;
        LLVM_DEBUG(dbgs() << "found function type: " << *NewType << "\n");
      } else if (NewType != DestType) {
        errs() << "warning: prototype-less function used with "
                  "conflicting signatures: "
               << F.getName() << "\n";
        LLVM_DEBUG(dbgs() << "  " << *DestType << "\n");
        LLVM_DEBUG(dbgs() << "  " << *NewType << "\n");
      }
    }

    if (!NewType) {
      LLVM_DEBUG(
          dbgs() << "could not derive a function prototype from usage: " +
                        F.getName() + "\n");
      // We could not derive a type for this function.  In this case strip
      // the isVarArg and make it a simple zero-arg function.  This has more
      // chance of being correct.  The current signature of (...) is illegal in
      // C since it doesn't have any arguments before the "...", we this at
      // least makes it possible for this symbol to be resolved by the linker.
      NewType = FunctionType::get(F.getFunctionType()->getReturnType(), false);
    }

    Function *NewF =
        Function::Create(NewType, F.getLinkage(), F.getName() + ".fixed_sig");
    NewF->setAttributes(F.getAttributes());
    NewF->removeFnAttr("no-prototype");
    Replacements.emplace_back(&F, NewF);
  }

  for (auto &Pair : Replacements) {
    Function *OldF = Pair.first;
    Function *NewF = Pair.second;
    std::string Name = std::string(OldF->getName());
    M.getFunctionList().push_back(NewF);
    OldF->replaceAllUsesWith(
```
- **EN**: Implements helper routine(s) `dbgs`, `errs`, `getName` for this portion of the WebAssembly backend backend implementation logic.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `dbgs`, `errs`, `getName`。

### Lines 147-153

```cpp
        ConstantExpr::getPointerBitCastOrAddrSpaceCast(NewF, OldF->getType()));
    OldF->eraseFromParent();
    NewF->setName(Name);
  }

  return !Replacements.empty();
}
```
- **EN**: Declares function entry points including `getPointerBitCastOrAddrSpaceCast`, `getType`, `eraseFromParent` that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `getPointerBitCastOrAddrSpaceCast`, `getType`, `eraseFromParent`。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Backend implementation logic / 后端实现逻辑
- Vector or SIMD behavior / 向量或 SIMD 行为
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `WebAssembly.h`
- `llvm/IR/Constants.h`
- `llvm/IR/Module.h`
- `llvm/IR/Operator.h`
- `llvm/Pass.h`
- `llvm/Support/Debug.h`
- `llvm/Transforms/Utils/Local.h`
- `llvm/Transforms/Utils/ModuleUtils.h`

### Important Collaborators / 重要协作组件

- LLVM Support utilities / LLVM Support 工具层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
