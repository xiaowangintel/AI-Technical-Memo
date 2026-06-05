# AMDGPUAnnotateUniformValues.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUAnnotateUniformValues.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUAnnotateUniformValues for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPUAnnotateUniformValues 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: File banner, includes, and setup
```cpp
//===-- AMDGPUAnnotateUniformValues.cpp - ---------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// This pass adds amdgpu.uniform metadata to IR values so this information
/// can be used during instruction selection.
//
//===----------------------------------------------------------------------===//

#include "AMDGPU.h"
#include "AMDGPUMemoryUtils.h"
#include "Utils/AMDGPUBaseInfo.h"
#include "llvm/Analysis/AliasAnalysis.h"
#include "llvm/Analysis/MemorySSA.h"
#include "llvm/Analysis/UniformityAnalysis.h"
#include "llvm/IR/InstVisitor.h"
#include "llvm/InitializePasses.h"

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 24-47: Preprocessor guards and macros
```cpp
#define DEBUG_TYPE "amdgpu-annotate-uniform"

using namespace llvm;

namespace {

class AMDGPUAnnotateUniformValues
    : public InstVisitor<AMDGPUAnnotateUniformValues> {
  UniformityInfo *UA;
  MemorySSA *MSSA;
  AliasAnalysis *AA;
  bool isEntryFunc;
  bool Changed = false;

  void setUniformMetadata(Instruction *I) {
    I->setMetadata("amdgpu.uniform", MDNode::get(I->getContext(), {}));
    Changed = true;
  }

  void setNoClobberMetadata(Instruction *I) {
    I->setMetadata("amdgpu.noclobber", MDNode::get(I->getContext(), {}));
    Changed = true;
  }

```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `AMDGPUAnnotateUniformValues`, `MDNode::get`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`AMDGPUAnnotateUniformValues`, `MDNode::get`。

### Lines 48-66: Defines AMDGPUAnnotateUniformValues
```cpp
public:
  AMDGPUAnnotateUniformValues(UniformityInfo &UA, MemorySSA &MSSA,
                              AliasAnalysis &AA, const Function &F)
      : UA(&UA), MSSA(&MSSA), AA(&AA),
        isEntryFunc(AMDGPU::isEntryFunctionCC(F.getCallingConv())) {}

  void visitCondBrInst(CondBrInst &I);
  void visitLoadInst(LoadInst &I);

  bool changed() const { return Changed; }
};

} // End anonymous namespace

void AMDGPUAnnotateUniformValues::visitCondBrInst(CondBrInst &I) {
  if (UA->isUniformAtDef(&I))
    setUniformMetadata(&I);
}

```
**EN:** This section contains concrete logic for AMDGPUAnnotateUniformValues. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPU::isEntryFunctionCC`, `AMDGPUAnnotateUniformValues::visitCondBrInst`.
**CN:** 本节包含与 AMDGPUAnnotateUniformValues 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPU::isEntryFunctionCC`, `AMDGPUAnnotateUniformValues::visitCondBrInst`。

### Lines 67-84: Implements AMDGPUAnnotateUniformValues::visitLoadInst
```cpp
void AMDGPUAnnotateUniformValues::visitLoadInst(LoadInst &I) {
  Value *Ptr = I.getPointerOperand();
  if (UA->isDivergentAtDef(Ptr))
    return;
  Instruction *PtrI = dyn_cast<Instruction>(Ptr);
  if (PtrI)
    setUniformMetadata(PtrI);

  // We're tracking up to the Function boundaries, and cannot go beyond because
  // of FunctionPass restrictions. We can ensure that is memory not clobbered
  // for memory operations that are live in to entry points only.
  if (!isEntryFunc)
    return;
  bool GlobalLoad = I.getPointerAddressSpace() == AMDGPUAS::GLOBAL_ADDRESS;
  if (GlobalLoad && !AMDGPU::isClobberedInFunction(&I, MSSA, AA))
    setNoClobberMetadata(&I);
}

```
**EN:** This section contains concrete logic for AMDGPUAnnotateUniformValues::visitLoadInst. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUAnnotateUniformValues::visitLoadInst`, `AMDGPU::isClobberedInFunction`.
**CN:** 本节包含与 AMDGPUAnnotateUniformValues::visitLoadInst 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUAnnotateUniformValues::visitLoadInst`, `AMDGPU::isClobberedInFunction`。

### Lines 85-107: Declares class AMDGPUAnnotateUniformValuesLegacy
```cpp
PreservedAnalyses
AMDGPUAnnotateUniformValuesPass::run(Function &F,
                                     FunctionAnalysisManager &FAM) {
  UniformityInfo &UI = FAM.getResult<UniformityInfoAnalysis>(F);
  MemorySSA &MSSA = FAM.getResult<MemorySSAAnalysis>(F).getMSSA();
  AAResults &AA = FAM.getResult<AAManager>(F);

  AMDGPUAnnotateUniformValues Impl(UI, MSSA, AA, F);
  Impl.visit(F);

  if (!Impl.changed())
    return PreservedAnalyses::all();

  PreservedAnalyses PA = PreservedAnalyses::none();
  // TODO: Should preserve nearly everything
  PA.preserveSet<CFGAnalyses>();
  return PA;
}

class AMDGPUAnnotateUniformValuesLegacy : public FunctionPass {
public:
  static char ID;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUAnnotateUniformValuesLegacy`, `AMDGPUAnnotateUniformValuesPass::run`, `PreservedAnalyses::all`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUAnnotateUniformValuesLegacy`, `AMDGPUAnnotateUniformValuesPass::run`, `PreservedAnalyses::all`。

### Lines 108-128: Defines AMDGPUAnnotateUniformValuesLegacy
```cpp
  AMDGPUAnnotateUniformValuesLegacy() : FunctionPass(ID) {}

  bool doInitialization(Module &M) override { return false; }

  bool runOnFunction(Function &F) override;
  StringRef getPassName() const override {
    return "AMDGPU Annotate Uniform Values";
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addRequired<UniformityInfoWrapperPass>();
    AU.addRequired<MemorySSAWrapperPass>();
    AU.addRequired<AAResultsWrapperPass>();
    AU.setPreservesAll();
  }
};

bool AMDGPUAnnotateUniformValuesLegacy::runOnFunction(Function &F) {
  if (skipFunction(F))
    return false;

```
**EN:** This section contains concrete logic for AMDGPUAnnotateUniformValuesLegacy. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUAnnotateUniformValuesLegacy::runOnFunction`.
**CN:** 本节包含与 AMDGPUAnnotateUniformValuesLegacy 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUAnnotateUniformValuesLegacy::runOnFunction`。

### Lines 129-151: Registers LLVM passes
```cpp
  UniformityInfo &UI =
      getAnalysis<UniformityInfoWrapperPass>().getUniformityInfo();
  MemorySSA &MSSA = getAnalysis<MemorySSAWrapperPass>().getMSSA();
  AliasAnalysis &AA = getAnalysis<AAResultsWrapperPass>().getAAResults();

  AMDGPUAnnotateUniformValues Impl(UI, MSSA, AA, F);
  Impl.visit(F);
  return Impl.changed();
}

INITIALIZE_PASS_BEGIN(AMDGPUAnnotateUniformValuesLegacy, DEBUG_TYPE,
                      "Add AMDGPU uniform metadata", false, false)
INITIALIZE_PASS_DEPENDENCY(UniformityInfoWrapperPass)
INITIALIZE_PASS_DEPENDENCY(MemorySSAWrapperPass)
INITIALIZE_PASS_DEPENDENCY(AAResultsWrapperPass)
INITIALIZE_PASS_END(AMDGPUAnnotateUniformValuesLegacy, DEBUG_TYPE,
                    "Add AMDGPU uniform metadata", false, false)

char AMDGPUAnnotateUniformValuesLegacy::ID = 0;

FunctionPass *llvm::createAMDGPUAnnotateUniformValuesLegacy() {
  return new AMDGPUAnnotateUniformValuesLegacy();
}
```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `llvm::createAMDGPUAnnotateUniformValuesLegacy`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`llvm::createAMDGPUAnnotateUniformValuesLegacy`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `AMDGPUAnnotateUniformValues`, `AMDGPUAnnotateUniformValuesLegacy`, `MDNode::get`, `AMDGPU::isEntryFunctionCC`, `AMDGPUAnnotateUniformValues::visitCondBrInst`, `AMDGPUAnnotateUniformValues::visitLoadInst`
- **Main themes / 核心主题**: alias analysis / 别名分析; instruction semantics / 指令语义; LLVM pass integration / LLVM Pass 集成
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPU.h"`
- `"AMDGPUMemoryUtils.h"`
- `"Utils/AMDGPUBaseInfo.h"`
- `"llvm/Analysis/AliasAnalysis.h"`
- `"llvm/Analysis/MemorySSA.h"`
- `"llvm/Analysis/UniformityAnalysis.h"`
- `"llvm/IR/InstVisitor.h"`
- `"llvm/InitializePasses.h"`
