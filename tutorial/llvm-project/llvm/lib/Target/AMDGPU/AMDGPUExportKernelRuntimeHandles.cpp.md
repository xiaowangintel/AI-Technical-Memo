# AMDGPUExportKernelRuntimeHandles.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUExportKernelRuntimeHandles.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUExportKernelRuntimeHandles for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPUExportKernelRuntimeHandles 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: File banner, license, and overview
```cpp
//===- AMDGPUExportKernelRuntimeHandles.cpp - Lower enqueued block --------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// \file
//
// Give any globals used for OpenCL block enqueue runtime handles external
// linkage so the runtime may access them. These should behave like internal
// functions for purposes of linking, but need to have an external symbol in the
// final object for the runtime to access them.
//
// TODO: This could be replaced with a new linkage type or global object
// metadata that produces an external symbol in the final object, but allows
// rename on IR linking. Alternatively if we can rely on
// GlobalValue::getGlobalIdentifier we can just make these external symbols to
// begin with.
//
//===----------------------------------------------------------------------===//

```
**EN:** This opening section establishes the file identity, licensing terms, and high-level intent before the backend-specific code begins.
**CN:** 开头部分先给出文件标识、许可证信息以及总体意图，为后续后端专用代码建立上下文。

### Lines 24-47: Header dependencies and setup
```cpp
#include "AMDGPUExportKernelRuntimeHandles.h"
#include "AMDGPU.h"
#include "llvm/IR/Module.h"
#include "llvm/Pass.h"

#define DEBUG_TYPE "amdgpu-export-kernel-runtime-handles"

using namespace llvm;

namespace {

/// Lower enqueued blocks.
class AMDGPUExportKernelRuntimeHandlesLegacy : public ModulePass {
public:
  static char ID;

  explicit AMDGPUExportKernelRuntimeHandlesLegacy() : ModulePass(ID) {}

private:
  bool runOnModule(Module &M) override;
};

} // end anonymous namespace

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available. Main symbols: `AMDGPUExportKernelRuntimeHandlesLegacy`.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。 主要符号：`AMDGPUExportKernelRuntimeHandlesLegacy`。

### Lines 48-64: Registers LLVM passes
```cpp
char AMDGPUExportKernelRuntimeHandlesLegacy::ID = 0;

char &llvm::AMDGPUExportKernelRuntimeHandlesLegacyID =
    AMDGPUExportKernelRuntimeHandlesLegacy::ID;

INITIALIZE_PASS(AMDGPUExportKernelRuntimeHandlesLegacy, DEBUG_TYPE,
                "Externalize enqueued block runtime handles", false, false)

ModulePass *llvm::createAMDGPUExportKernelRuntimeHandlesLegacyPass() {
  return new AMDGPUExportKernelRuntimeHandlesLegacy();
}

static bool exportKernelRuntimeHandles(Module &M) {
  bool Changed = false;

  const StringLiteral HandleSectionName(".amdgpu.kernel.runtime.handle");

```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `llvm::createAMDGPUExportKernelRuntimeHandlesLegacyPass`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`llvm::createAMDGPUExportKernelRuntimeHandlesLegacyPass`。

### Lines 65-85: Conditional logic and checks
```cpp
  for (GlobalVariable &GV : M.globals()) {
    if (GV.getSection() == HandleSectionName) {
      GV.setLinkage(GlobalValue::ExternalLinkage);
      GV.setDSOLocal(false);
      Changed = true;
    }
  }

  if (!Changed)
    return false;

  // FIXME: We shouldn't really need to export the kernel address. We can
  // initialize the runtime handle with the kernel descriptor.
  for (Function &F : M) {
    if (F.getCallingConv() != CallingConv::AMDGPU_KERNEL)
      continue;

    const MDNode *Associated = F.getMetadata(LLVMContext::MD_associated);
    if (!Associated)
      continue;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 86-106: Implements AMDGPUExportKernelRuntimeHandlesLegacy::runOnModule
```cpp
    auto *VM = cast<ValueAsMetadata>(Associated->getOperand(0));
    auto *Handle = dyn_cast<GlobalObject>(VM->getValue());
    if (Handle && Handle->getSection() == HandleSectionName) {
      F.setLinkage(GlobalValue::ExternalLinkage);
      F.setVisibility(GlobalValue::ProtectedVisibility);
    }
  }

  return Changed;
}

bool AMDGPUExportKernelRuntimeHandlesLegacy::runOnModule(Module &M) {
  return exportKernelRuntimeHandles(M);
}

PreservedAnalyses
AMDGPUExportKernelRuntimeHandlesPass::run(Module &M,
                                          ModuleAnalysisManager &MAM) {
  if (!exportKernelRuntimeHandles(M))
    return PreservedAnalyses::all();

```
**EN:** This section contains concrete logic for AMDGPUExportKernelRuntimeHandlesLegacy::runOnModule. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUExportKernelRuntimeHandlesLegacy::runOnModule`, `AMDGPUExportKernelRuntimeHandlesPass::run`, `PreservedAnalyses::all`.
**CN:** 本节包含与 AMDGPUExportKernelRuntimeHandlesLegacy::runOnModule 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUExportKernelRuntimeHandlesLegacy::runOnModule`, `AMDGPUExportKernelRuntimeHandlesPass::run`, `PreservedAnalyses::all`。

### Lines 107-110: Result computation and returns
```cpp
  PreservedAnalyses PA;
  PA.preserveSet<AllAnalysesOn<Function>>();
  return PA;
}
```
**EN:** This section finalizes intermediate state and returns the value that the caller or pass pipeline needs.
**CN:** 本节整理中间状态并返回调用者或 Pass 流水线所需的结果。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `AMDGPUExportKernelRuntimeHandlesLegacy`, `llvm::createAMDGPUExportKernelRuntimeHandlesLegacyPass`, `AMDGPUExportKernelRuntimeHandlesLegacy::runOnModule`, `AMDGPUExportKernelRuntimeHandlesPass::run`, `PreservedAnalyses::all`
- **Main themes / 核心主题**: lowering / 降低; assembly handling / 汇编处理; LLVM pass integration / LLVM Pass 集成
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPUExportKernelRuntimeHandles.h"`
- `"AMDGPU.h"`
- `"llvm/IR/Module.h"`
- `"llvm/Pass.h"`
