# AMDGPURewriteUndefForPHI.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPURewriteUndefForPHI.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPURewriteUndefForPHI for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPURewriteUndefForPHI 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: File banner, license, and overview
```cpp
//===- AMDGPURewriteUndefForPHI.cpp ---------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
// This file implements the idea to rewrite undef incoming operand for certain
// PHIs in structurized CFG. This pass only works on IR that has gone through
// StructurizedCFG pass, and this pass has some additional limitation that make
// it can only run after SIAnnotateControlFlow.
//
// To achieve optimal code generation for AMDGPU, we assume that uniformity
// analysis reports the PHI in join block of divergent branch as uniform if
// it has one unique uniform value plus additional undefined/poisoned incoming
// value. That is to say the later compiler pipeline will ensure such PHI always
// return uniform value and ensure it work correctly. Let's take a look at two
// typical patterns in structured CFG that need to be taken care: (In both
// patterns, block %if terminate with divergent branch.)
//
// Pattern A: Block with undefined incoming value dominates defined predecessor
//  %if
//  | \
//  | %then
```
**EN:** This opening section establishes the file identity, licensing terms, and high-level intent before the backend-specific code begins.
**CN:** 开头部分先给出文件标识、许可证信息以及总体意图，为后续后端专用代码建立上下文。

### Lines 25-48: Implementation details and local logic
```cpp
//  | /
//  %endif: %phi = phi [%undef, %if], [%uniform, %then]
//
//  Pattern B: Block with defined incoming value dominates undefined predecessor
//  %if
//  | \
//  | %then
//  | /
//  %endif: %phi = phi [%uniform, %if], [%undef, %then]
//
// For pattern A, by reporting %phi as uniform, the later pipeline need to make
// sure it be handled correctly. The backend usually allocates a scalar register
// and if any thread in a wave takes %then path, the scalar register will get
// the %uniform value.
//
// For pattern B, we will replace the undef operand with the other defined value
// in this pass. So the scalar register allocated for such PHI will get correct
// liveness. Without this transformation, the scalar register may be overwritten
// in the %then block.
//
// Limitation note:
// If the join block of divergent threads is a loop header, the pass cannot
// handle it correctly right now. For below case, the undef in %phi should also
// be rewritten. Currently we depend on SIAnnotateControlFlow to split %header
```
**EN:** This section advances the file implementation by combining declarations, control flow, and helper operations that support the AMDGPU backend.
**CN:** 本节通过组合声明、控制流与辅助操作来推进文件实现，支撑 AMDGPU 后端的整体功能。

### Lines 49-71: Header dependencies and setup
```cpp
// block to get a separate join block, then we can rewrite the undef correctly.
//     %if
//     | \
//     | %then
//     | /
// -> %header: %phi = phi [%uniform, %if], [%undef, %then], [%uniform2, %header]
// |   |
// \---

#include "AMDGPU.h"
#include "llvm/Analysis/UniformityAnalysis.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/Dominators.h"
#include "llvm/IR/Instructions.h"
#include "llvm/InitializePasses.h"

using namespace llvm;

#define DEBUG_TYPE "amdgpu-rewrite-undef-for-phi"

namespace {

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。

### Lines 72-92: Declares class AMDGPURewriteUndefForPHILegacy
```cpp
class AMDGPURewriteUndefForPHILegacy : public FunctionPass {
public:
  static char ID;
  AMDGPURewriteUndefForPHILegacy() : FunctionPass(ID) {}
  bool runOnFunction(Function &F) override;
  StringRef getPassName() const override {
    return "AMDGPU Rewrite Undef for PHI";
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addRequired<UniformityInfoWrapperPass>();
    AU.addRequired<DominatorTreeWrapperPass>();

    AU.addPreserved<DominatorTreeWrapperPass>();
    AU.setPreservesCFG();
  }
};

} // end anonymous namespace
char AMDGPURewriteUndefForPHILegacy::ID = 0;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPURewriteUndefForPHILegacy`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPURewriteUndefForPHILegacy`。

### Lines 93-115: Registers LLVM passes
```cpp
INITIALIZE_PASS_BEGIN(AMDGPURewriteUndefForPHILegacy, DEBUG_TYPE,
                      "Rewrite undef for PHI", false, false)
INITIALIZE_PASS_DEPENDENCY(UniformityInfoWrapperPass)
INITIALIZE_PASS_DEPENDENCY(DominatorTreeWrapperPass)
INITIALIZE_PASS_END(AMDGPURewriteUndefForPHILegacy, DEBUG_TYPE,
                    "Rewrite undef for PHI", false, false)

bool rewritePHIs(Function &F, UniformityInfo &UA, DominatorTree *DT) {
  bool Changed = false;
  SmallVector<PHINode *> ToBeDeleted;
  for (auto &BB : F) {
    for (auto &PHI : BB.phis()) {
      if (UA.isDivergentAtDef(&PHI))
        continue;

      // The unique incoming value except undef/poison for the PHI node.
      Value *UniqueDefinedIncoming = nullptr;
      // The divergent block with defined incoming value that dominates all
      // other block with the same incoming value.
      BasicBlock *DominateBB = nullptr;
      // Predecessors with undefined incoming value (excluding loop backedge).
      SmallVector<BasicBlock *> Undefs;

```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。

### Lines 116-129: Conditional logic and checks
```cpp
      for (unsigned i = 0; i < PHI.getNumIncomingValues(); i++) {
        Value *Incoming = PHI.getIncomingValue(i);
        BasicBlock *IncomingBB = PHI.getIncomingBlock(i);

        if (Incoming == &PHI)
          continue;

        if (isa<UndefValue>(Incoming)) {
          // Undef from loop backedge will not be replaced.
          if (!DT->dominates(&BB, IncomingBB))
            Undefs.push_back(IncomingBB);
          continue;
        }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 130-149: Conditional logic and checks
```cpp
        if (!UniqueDefinedIncoming) {
          UniqueDefinedIncoming = Incoming;
          DominateBB = IncomingBB;
        } else if (Incoming == UniqueDefinedIncoming) {
          // Update DominateBB if necessary.
          if (DT->dominates(IncomingBB, DominateBB))
            DominateBB = IncomingBB;
        } else {
          UniqueDefinedIncoming = nullptr;
          break;
        }
      }
      // We only need to replace the undef for the PHI which is merging
      // defined/undefined values from divergent threads.
      // TODO: We should still be able to replace undef value if the unique
      // value is a Constant.
      if (!UniqueDefinedIncoming || Undefs.empty() ||
          UA.isUniformAtDef(DominateBB->getTerminator()))
        continue;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 150-169: Conditional logic and checks
```cpp
      // We only replace the undef when DominateBB truly dominates all the
      // other predecessors with undefined incoming value. Make sure DominateBB
      // dominates BB so that UniqueDefinedIncoming is available in BB and
      // afterwards.
      if (DT->dominates(DominateBB, &BB) && all_of(Undefs, [&](BasicBlock *UD) {
            return DT->dominates(DominateBB, UD);
          })) {
        PHI.replaceAllUsesWith(UniqueDefinedIncoming);
        ToBeDeleted.push_back(&PHI);
        Changed = true;
      }
    }
  }

  for (auto *PHI : ToBeDeleted)
    PHI->eraseFromParent();

  return Changed;
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 170-193: Implements AMDGPURewriteUndefForPHILegacy::runOnFunction
```cpp
bool AMDGPURewriteUndefForPHILegacy::runOnFunction(Function &F) {
  UniformityInfo &UA =
      getAnalysis<UniformityInfoWrapperPass>().getUniformityInfo();
  DominatorTree *DT = &getAnalysis<DominatorTreeWrapperPass>().getDomTree();
  return rewritePHIs(F, UA, DT);
}

PreservedAnalyses
AMDGPURewriteUndefForPHIPass::run(Function &F, FunctionAnalysisManager &AM) {
  UniformityInfo &UA = AM.getResult<UniformityInfoAnalysis>(F);
  DominatorTree *DT = &AM.getResult<DominatorTreeAnalysis>(F);
  bool Changed = rewritePHIs(F, UA, DT);
  if (Changed) {
    PreservedAnalyses PA;
    PA.preserveSet<CFGAnalyses>();
    return PA;
  }

  return PreservedAnalyses::all();
}

FunctionPass *llvm::createAMDGPURewriteUndefForPHILegacyPass() {
  return new AMDGPURewriteUndefForPHILegacy();
}
```
**EN:** This section contains concrete logic for AMDGPURewriteUndefForPHILegacy::runOnFunction. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPURewriteUndefForPHILegacy::runOnFunction`, `AMDGPURewriteUndefForPHIPass::run`, `PreservedAnalyses::all`.
**CN:** 本节包含与 AMDGPURewriteUndefForPHILegacy::runOnFunction 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPURewriteUndefForPHILegacy::runOnFunction`, `AMDGPURewriteUndefForPHIPass::run`, `PreservedAnalyses::all`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `AMDGPURewriteUndefForPHILegacy`, `AMDGPURewriteUndefForPHILegacy::runOnFunction`, `AMDGPURewriteUndefForPHIPass::run`, `PreservedAnalyses::all`, `llvm::createAMDGPURewriteUndefForPHILegacyPass`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; LLVM pass integration / LLVM Pass 集成
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPU.h"`
- `"llvm/Analysis/UniformityAnalysis.h"`
- `"llvm/IR/BasicBlock.h"`
- `"llvm/IR/Constants.h"`
- `"llvm/IR/Dominators.h"`
- `"llvm/IR/Instructions.h"`
- `"llvm/InitializePasses.h"`
