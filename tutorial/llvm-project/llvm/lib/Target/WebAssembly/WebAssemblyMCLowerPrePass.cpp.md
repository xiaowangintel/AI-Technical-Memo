# WebAssemblyMCLowerPrePass.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyMCLowerPrePass.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Some information in MC lowering / asm printing gets generated as instructions get emitted, but may be necessary at the start, such as for .globaltype declarations. This pass collects this information.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyMCLowerPrePass.cpp`，主要负责 WebAssembly 后端的该后端的 MC 层支持。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- WebAssemblyMCLowerPrePass.cpp - Prepare for MC lower --------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-13

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// Some information in MC lowering / asm printing gets generated as
/// instructions get emitted, but may be necessary at the start, such as for
/// .globaltype declarations. This pass collects this information.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 14-20

```cpp
//===----------------------------------------------------------------------===//

#include "WebAssembly.h"
#include "WebAssemblyUtilities.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineModuleInfoImpls.h"
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 该区间与栈帧布局或栈访问相关。

### Lines 21-28

```cpp
#include "llvm/CodeGen/Passes.h"
#include "llvm/IR/Module.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"
using namespace llvm;

#define DEBUG_TYPE "wasm-mclower-prepass"
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 29-66

```cpp
namespace {
class WebAssemblyMCLowerPrePass final : public ModulePass {
  StringRef getPassName() const override {
    return "WebAssembly MC Lower Pre Pass";
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    ModulePass::getAnalysisUsage(AU);
  }

  bool runOnModule(Module &M) override;

public:
  static char ID; // Pass identification, replacement for typeid
  WebAssemblyMCLowerPrePass() : ModulePass(ID) {}
};
} // end anonymous namespace

char WebAssemblyMCLowerPrePass::ID = 0;
INITIALIZE_PASS(
    WebAssemblyMCLowerPrePass, DEBUG_TYPE,
    "Collects information ahead of time for MC lowering",
    false, false)

ModulePass *llvm::createWebAssemblyMCLowerPrePass() {
  return new WebAssemblyMCLowerPrePass();
}

// NOTE: this is a ModulePass since we need to enforce that this code has run
// for all functions before AsmPrinter. If this way of doing things is ever
// suboptimal, we could opt to make it a MachineFunctionPass and instead use
// something like createBarrierNoopPass() to enforce ordering.
//
// The information stored here is essential for emitExternalDecls in the Wasm
// AsmPrinter
bool WebAssemblyMCLowerPrePass::runOnModule(Module &M) {
  auto *MMIWP = getAnalysisIfAvailable<MachineModuleInfoWrapperPass>();
```
- **EN**: Declares a backend-facing type `WebAssemblyMCLowerPrePass`, `getPassName`, `getAnalysisUsage` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `WebAssemblyMCLowerPrePass`, `getPassName`, `getAnalysisUsage`，并勾勒出周边代码会依赖的接口或状态。

### Lines 67-96

```cpp
  if (!MMIWP)
    return true;

  MachineModuleInfo &MMI = MMIWP->getMMI();
  MachineModuleInfoWasm &MMIW = MMI.getObjFileInfo<MachineModuleInfoWasm>();

  for (Function &F : M) {
    MachineFunction *MF = MMI.getMachineFunction(F);
    if (!MF)
      continue;

    LLVM_DEBUG(dbgs() << "********** MC Lower Pre Pass **********\n"
                         "********** Function: "
                      << MF->getName() << '\n');

    for (MachineBasicBlock &MBB : *MF) {
      for (auto &MI : MBB) {
        // FIXME: what should all be filtered out beyond these?
        if (MI.isDebugInstr() || MI.isInlineAsm())
          continue;
        for (MachineOperand &MO : MI.uses()) {
          if (MO.isSymbol()) {
            MMIW.MachineSymbolsUsed.insert(MO.getSymbolName());
          }
        }
      }
    }
  }
  return true;
}
```
- **EN**: Implements helper routine(s) `getMMI`, `getMachineFunction`, `dbgs` for this portion of the WebAssembly backend MC layer support for the backend. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分该后端的 MC 层支持所需的辅助例程 `getMMI`, `getMachineFunction`, `dbgs`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- MC layer support for the backend / 该后端的 MC 层支持
- Stack frame management / 栈帧管理
- Assembly or MC emission / 汇编或 MC 发射
- Object format integration / 目标文件格式集成
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `WebAssembly.h`
- `WebAssemblyUtilities.h`
- `llvm/CodeGen/MachineFrameInfo.h`
- `llvm/CodeGen/MachineFunction.h`
- `llvm/CodeGen/MachineModuleInfoImpls.h`
- `llvm/CodeGen/Passes.h`
- `llvm/IR/Module.h`
- `llvm/Support/Debug.h`
- `llvm/Support/raw_ostream.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM Support utilities / LLVM Support 工具层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
