# WebAssemblyRefTypeMem2Local.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyRefTypeMem2Local.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Assign reference type allocas to local addrspace (addrspace(1)) so that their loads and stores can be lowered to local.gets/local.sets.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyRefTypeMem2Local.cpp`，主要负责 WebAssembly 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//=== WebAssemblyRefTypeMem2Local.cpp - WebAssembly RefType Mem2Local -----===//
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
/// Assign reference type allocas to local addrspace (addrspace(1)) so that
/// their loads and stores can be lowered to local.gets/local.sets.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file". Notable symbols in this range include `addrspace`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。 该区间中较显眼的符号包括 `addrspace`。

### Lines 13-19

```cpp
//===----------------------------------------------------------------------===//

#include "Utils/WasmAddressSpaces.h"
#include "Utils/WebAssemblyTypeUtilities.h"
#include "WebAssembly.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/InstVisitor.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 20-26

```cpp
#include "llvm/IR/ValueHandle.h"
#include "llvm/Pass.h"
using namespace llvm;

#define DEBUG_TYPE "wasm-ref-type-mem2local"

namespace {
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 27-64

```cpp
class WebAssemblyRefTypeMem2Local final
    : public FunctionPass,
      public InstVisitor<WebAssemblyRefTypeMem2Local> {
  StringRef getPassName() const override {
    return "WebAssembly Reference Types Memory to Local";
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    FunctionPass::getAnalysisUsage(AU);
  }

  bool runOnFunction(Function &F) override;
  bool Changed = false;

public:
  static char ID;
  WebAssemblyRefTypeMem2Local() : FunctionPass(ID) {}

  void visitAllocaInst(AllocaInst &AI);
};
} // End anonymous namespace

char WebAssemblyRefTypeMem2Local::ID = 0;
INITIALIZE_PASS(WebAssemblyRefTypeMem2Local, DEBUG_TYPE,
                "Assign reference type allocas to local address space", true,
                false)

FunctionPass *llvm::createWebAssemblyRefTypeMem2Local() {
  return new WebAssemblyRefTypeMem2Local();
}

void WebAssemblyRefTypeMem2Local::visitAllocaInst(AllocaInst &AI) {
  if (WebAssembly::isWebAssemblyReferenceType(AI.getAllocatedType())) {
    Changed = true;
    IRBuilder<> IRB(AI.getContext());
    IRB.SetInsertPoint(&AI);
    auto *NewAI = IRB.CreateAlloca(AI.getAllocatedType(),
```
- **EN**: Declares a backend-facing type `WebAssemblyRefTypeMem2Local`, `getPassName`, `getAnalysisUsage` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `WebAssemblyRefTypeMem2Local`, `getPassName`, `getAnalysisUsage`，并勾勒出周边代码会依赖的接口或状态。

### Lines 65-94

```cpp
                                   WebAssembly::WASM_ADDRESS_SPACE_VAR, nullptr,
                                   AI.getName() + ".var");

    // The below is basically equivalent to AI.replaceAllUsesWith(NewAI), but we
    // cannot use it because it requires the old and new types be the same,
    // which is not true here because the address spaces are different.
    if (AI.hasValueHandle())
      ValueHandleBase::ValueIsRAUWd(&AI, NewAI);
    if (AI.isUsedByMetadata())
      ValueAsMetadata::handleRAUW(&AI, NewAI);
    while (!AI.materialized_use_empty()) {
      Use &U = *AI.materialized_use_begin();
      U.set(NewAI);
    }

    AI.eraseFromParent();
  }
}

bool WebAssemblyRefTypeMem2Local::runOnFunction(Function &F) {
  LLVM_DEBUG(dbgs() << "********** WebAssembly RefType Mem2Local **********\n"
                       "********** Function: "
                    << F.getName() << '\n');

  if (F.getFnAttribute("target-features")
          .getValueAsString()
          .contains("+reference-types"))
    visit(F);
  return Changed;
}
```
- **EN**: Implements helper routine(s) `getName`, `replaceAllUsesWith`, `hasValueHandle` for this portion of the WebAssembly backend backend implementation logic.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `getName`, `replaceAllUsesWith`, `hasValueHandle`。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Backend implementation logic / 后端实现逻辑
- Object format integration / 目标文件格式集成
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `Utils/WasmAddressSpaces.h`
- `Utils/WebAssemblyTypeUtilities.h`
- `WebAssembly.h`
- `llvm/IR/IRBuilder.h`
- `llvm/IR/InstVisitor.h`
- `llvm/IR/ValueHandle.h`
- `llvm/Pass.h`

### Important Collaborators / 重要协作组件

- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
