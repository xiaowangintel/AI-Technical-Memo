# PPCPrepareIFuncsOnAIX.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCPrepareIFuncsOnAIX.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides backend implementation logic for the PowerPC backend.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCPrepareIFuncsOnAIX.cpp`，主要负责 PowerPC 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- PPCPrepareIFuncsOnAIX.cpp - Prepare for ifunc lowering in codegen ===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-10

```cpp
//===----------------------------------------------------------------------===//
//
// This pass generates...
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This pass generates...".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This pass generates...”。

### Lines 11-17

```cpp
//===----------------------------------------------------------------------===//

#include "PPC.h"
#include "PPCSubtarget.h"
#include "PPCTargetMachine.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/Statistic.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 18-25

```cpp
#include "llvm/Analysis/TargetTransformInfo.h"
#include "llvm/CodeGen/TargetPassConfig.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/Module.h"
#include <cassert>

using namespace llvm;
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 26-63

```cpp
#define DEBUG_TYPE "ppc-prep-ifunc-aix"

STATISTIC(NumIFuncs, "Number of IFuncs prepared");

namespace {
class PPCPrepareIFuncsOnAIX : public ModulePass {
public:
  static char ID;

  PPCPrepareIFuncsOnAIX() : ModulePass(ID) {}

  bool runOnModule(Module &M) override;

  StringRef getPassName() const override {
    return "PPC Prepare for AIX IFunc lowering";
  }
};
} // namespace

char PPCPrepareIFuncsOnAIX::ID = 0;

INITIALIZE_PASS(PPCPrepareIFuncsOnAIX, DEBUG_TYPE,
                "PPC Prepare for AIX IFunc lowering", false, false)

ModulePass *llvm::createPPCPrepareIFuncsOnAIXPass() {
  return new PPCPrepareIFuncsOnAIX();
}

// For each ifunc `foo` with a resolver `foo_resolver`, create a global variable
// `__update_foo` in the `ifunc_sec` section, representing the pair:
//   { ptr @foo, ptr @foo_resolver }
// The compiler arranges for the constructor function `__init_ifuncs` to be
// included on the link step. The constructor walks the `ifunc_sec` section,
// calling the resolver function and storing the result in foo's descriptor.
// On AIX, the address of a function is the address of its descriptor, so the
// constructor accesses foo's descriptor from the first field of the pair.
//
// Since the global `__update_foo` is unreferenced, it's liveness needs to be
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Notable symbols in this range include `PPCPrepareIFuncsOnAIX`, `STATISTIC`, `ModulePass`.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 该区间中较显眼的符号包括 `PPCPrepareIFuncsOnAIX`, `STATISTIC`, `ModulePass`。

### Lines 64-101

```cpp
// associated to the liveness of ifunc `foo`
//
bool PPCPrepareIFuncsOnAIX::runOnModule(Module &M) {
  if (M.ifuncs().empty())
    return false;

  const DataLayout &DL = M.getDataLayout();
  LLVMContext &Ctx = M.getContext();
  auto *PtrTy = PointerType::getUnqual(Ctx);
  StringRef IFuncUpdatePrefix = "__update_";
  StringRef IFuncUpdateSectionName = "__ifunc_sec";
  StructType *IFuncPairType = StructType::get(PtrTy, PtrTy);

  StringRef IFuncConstructorName = "__init_ifuncs";
  auto *IFuncConstructorFnType =
      FunctionType::get(Type::getVoidTy(Ctx), {}, /*isVarArg=*/false);
  auto *IFuncConstructorDecl = cast<Function>(
      M.getOrInsertFunction(IFuncConstructorName, IFuncConstructorFnType)
          .getCallee());

  for (GlobalIFunc &IFunc : M.ifuncs()) {
    NumIFuncs++;
    LLVM_DEBUG(dbgs() << "expanding ifunc " << IFunc.getName() << "\n");
    // @__update_foo = private global { ptr @foo, ptr @foo_resolver },
    //   section "ifunc_sec"
    std::string Name = (Twine(IFuncUpdatePrefix) + IFunc.getName()).str();
    auto *GV = new GlobalVariable(M, IFuncPairType, /*isConstant*/ false,
                                  GlobalValue::PrivateLinkage, nullptr, Name);
    GV->setAlignment(DL.getPointerPrefAlignment());
    GV->setSection(IFuncUpdateSectionName);

    // Note that on AIX, the address of a function is the address of it's
    // function descriptor, which is what these two values end up being
    // in assembly.
    Constant *InitVals[] = {&IFunc, IFunc.getResolver()};
    GV->setInitializer(ConstantStruct::get(IFuncPairType, InitVals));

    // Liveness of __update_foo is dependent on liveness of ifunc foo.
```
- **EN**: Implements helper routine(s) `runOnModule`, `ifuncs`, `empty` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `runOnModule`, `ifuncs`, `empty`。

### Lines 102-114

```cpp
    IFunc.setMetadata(LLVMContext::MD_implicit_ref,
                      MDNode::get(Ctx, ValueAsMetadata::get(GV)));

    // An implicit.ref creates linkage dependency, so make function foo require
    // the constructor that calls each ifunc's resolver and saves the result in
    // the ifunc's function descriptor.
    IFunc.addMetadata(
        LLVMContext::MD_implicit_ref,
        *MDNode::get(Ctx, ValueAsMetadata::get(IFuncConstructorDecl)));
  }

  return true;
}
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "An implicit.ref creates linkage dependency, so make function foo require". Notable symbols in this range include `setMetadata`, `get`, `addMetadata`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“An implicit.ref creates linkage dependency, so make function foo require”。 该区间中较显眼的符号包括 `setMetadata`, `get`, `addMetadata`。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- Backend implementation logic / 后端实现逻辑
- Subtarget features / 子目标特性
- Target machine configuration / 目标机器配置
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `PPC.h`
- `PPCSubtarget.h`
- `PPCTargetMachine.h`
- `llvm/ADT/STLExtras.h`
- `llvm/ADT/Statistic.h`
- `llvm/Analysis/TargetTransformInfo.h`
- `llvm/CodeGen/TargetPassConfig.h`
- `llvm/IR/Instructions.h`
- `llvm/IR/Module.h`
- `cassert`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
