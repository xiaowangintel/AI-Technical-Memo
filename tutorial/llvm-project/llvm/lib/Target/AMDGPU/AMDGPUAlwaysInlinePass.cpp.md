# AMDGPUAlwaysInlinePass.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUAlwaysInlinePass.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUAlwaysInlinePass for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPUAlwaysInlinePass 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: File banner, includes, and setup
```cpp
//===-- AMDGPUAlwaysInlinePass.cpp - Promote Allocas ----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// This pass marks all internal functions as always_inline and creates
/// duplicates of all other functions and marks the duplicates as always_inline.
//
//===----------------------------------------------------------------------===//

#include "AMDGPU.h"
#include "AMDGPUTargetMachine.h"
#include "Utils/AMDGPUBaseInfo.h"
#include "llvm/CodeGen/CommandFlags.h"
#include "llvm/IR/Module.h"
#include "llvm/Pass.h"
#include "llvm/Support/CommandLine.h"

using namespace llvm;

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 25-47: Declares class AMDGPUAlwaysInline
```cpp
namespace {

static cl::opt<bool> StressCalls(
  "amdgpu-stress-function-calls",
  cl::Hidden,
  cl::desc("Force all functions to be noinline"),
  cl::init(false));

class AMDGPUAlwaysInline : public ModulePass {
  bool GlobalOpt;

public:
  static char ID;

  AMDGPUAlwaysInline(bool GlobalOpt = false) :
    ModulePass(ID), GlobalOpt(GlobalOpt) { }
  bool runOnModule(Module &M) override;

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesAll();
  }
};

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUAlwaysInline`, `cl::desc`, `cl::init`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUAlwaysInline`, `cl::desc`, `cl::init`。

### Lines 48-66: Registers LLVM passes
```cpp
} // End anonymous namespace

INITIALIZE_PASS(AMDGPUAlwaysInline, "amdgpu-always-inline",
                "AMDGPU Inline All Functions", false, false)

char AMDGPUAlwaysInline::ID = 0;

static void
recursivelyVisitUsers(GlobalValue &GV,
                      SmallPtrSetImpl<Function *> &FuncsToAlwaysInline) {
  SmallVector<User *, 16> Stack(GV.users());

  SmallPtrSet<const Value *, 8> Visited;

  while (!Stack.empty()) {
    User *U = Stack.pop_back_val();
    if (!Visited.insert(U).second)
      continue;

```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。

### Lines 67-88: Conditional logic and checks
```cpp
    if (Instruction *I = dyn_cast<Instruction>(U)) {
      Function *F = I->getFunction();
      if (!AMDGPU::isEntryFunctionCC(F->getCallingConv())) {
        // FIXME: This is a horrible hack. We should always respect noinline,
        // and just let us hit the error when we can't handle this.
        //
        // Unfortunately, clang adds noinline to all functions at -O0. We have
        // to override this here until that's fixed.
        F->removeFnAttr(Attribute::NoInline);

        FuncsToAlwaysInline.insert(F);
        Stack.push_back(F);
      }

      // No need to look at further users, but we do need to inline any callers.
      continue;
    }

    append_range(Stack, U->users());
  }
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::isEntryFunctionCC`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::isEntryFunctionCC`。

### Lines 89-109: Defines alwaysInlineImpl
```cpp
static bool alwaysInlineImpl(Module &M, bool GlobalOpt) {
  std::vector<GlobalAlias*> AliasesToRemove;

  bool Changed = false;
  SmallPtrSet<Function *, 8> FuncsToAlwaysInline;
  SmallPtrSet<Function *, 8> FuncsToNoInline;
  Triple TT(M.getTargetTriple());

  for (GlobalAlias &A : M.aliases()) {
    if (Function* F = dyn_cast<Function>(A.getAliasee())) {
      if (TT.isAMDGCN() && A.getLinkage() != GlobalValue::InternalLinkage)
        continue;
      Changed = true;
      A.replaceAllUsesWith(F);
      AliasesToRemove.push_back(&A);
    }

    // FIXME: If the aliasee isn't a function, it's some kind of constant expr
    // cast that won't be inlined through.
  }

```
**EN:** This section contains concrete logic for alwaysInlineImpl. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 alwaysInlineImpl 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 110-125: Conditional logic and checks
```cpp
  if (GlobalOpt) {
    for (GlobalAlias* A : AliasesToRemove) {
      A->eraseFromParent();
    }
  }

  // Always force inlining of any function that uses an LDS global address. This
  // is something of a workaround because we don't have a way of supporting LDS
  // objects defined in functions. LDS is always allocated by a kernel, and it
  // is difficult to manage LDS usage if a function may be used by multiple
  // kernels.
  //
  // OpenCL doesn't allow declaring LDS in non-kernels, so in practice this
  // should only appear when IPO passes manages to move LDs defined in a kernel
  // into a single user function.

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 126-138: Conditional logic and checks
```cpp
  for (GlobalVariable &GV : M.globals()) {
    // TODO: Region address
    unsigned AS = GV.getAddressSpace();
    if ((AS == AMDGPUAS::REGION_ADDRESS) ||
        (AS == AMDGPUAS::LOCAL_ADDRESS &&
         (!AMDGPUTargetMachine::EnableLowerModuleLDS)))
      recursivelyVisitUsers(GV, FuncsToAlwaysInline);
  }

  if (!AMDGPUTargetMachine::EnableFunctionCalls || StressCalls) {
    auto IncompatAttr
      = StressCalls ? Attribute::AlwaysInline : Attribute::NoInline;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 139-159: Conditional logic and checks
```cpp
    for (Function &F : M) {
      if (!F.isDeclaration() && !F.use_empty() &&
          !F.hasFnAttribute(IncompatAttr)) {
        if (StressCalls) {
          if (!FuncsToAlwaysInline.count(&F))
            FuncsToNoInline.insert(&F);
        } else
          FuncsToAlwaysInline.insert(&F);
      }
    }
  }

  for (Function *F : FuncsToAlwaysInline)
    F->addFnAttr(Attribute::AlwaysInline);

  for (Function *F : FuncsToNoInline)
    F->addFnAttr(Attribute::NoInline);

  return Changed || !FuncsToAlwaysInline.empty() || !FuncsToNoInline.empty();
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 160-172: Implements AMDGPUAlwaysInline::runOnModule
```cpp
bool AMDGPUAlwaysInline::runOnModule(Module &M) {
  return alwaysInlineImpl(M, GlobalOpt);
}

ModulePass *llvm::createAMDGPUAlwaysInlinePass(bool GlobalOpt) {
  return new AMDGPUAlwaysInline(GlobalOpt);
}

PreservedAnalyses AMDGPUAlwaysInlinePass::run(Module &M,
                                              ModuleAnalysisManager &AM) {
  const bool Changed = alwaysInlineImpl(M, GlobalOpt);
  return Changed ? PreservedAnalyses::none() : PreservedAnalyses::all();
}
```
**EN:** This section contains concrete logic for AMDGPUAlwaysInline::runOnModule. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUAlwaysInline::runOnModule`, `llvm::createAMDGPUAlwaysInlinePass`, `AMDGPUAlwaysInlinePass::run`.
**CN:** 本节包含与 AMDGPUAlwaysInline::runOnModule 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUAlwaysInline::runOnModule`, `llvm::createAMDGPUAlwaysInlinePass`, `AMDGPUAlwaysInlinePass::run`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `AMDGPUAlwaysInline`, `cl::desc`, `cl::init`, `AMDGPU::isEntryFunctionCC`, `AMDGPUAlwaysInline::runOnModule`, `llvm::createAMDGPUAlwaysInlinePass`
- **Main themes / 核心主题**: alias analysis / 别名分析; instruction semantics / 指令语义; lowering / 降低; LLVM pass integration / LLVM Pass 集成
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPU.h"`
- `"AMDGPUTargetMachine.h"`
- `"Utils/AMDGPUBaseInfo.h"`
- `"llvm/CodeGen/CommandFlags.h"`
- `"llvm/IR/Module.h"`
- `"llvm/Pass.h"`
- `"llvm/Support/CommandLine.h"`
