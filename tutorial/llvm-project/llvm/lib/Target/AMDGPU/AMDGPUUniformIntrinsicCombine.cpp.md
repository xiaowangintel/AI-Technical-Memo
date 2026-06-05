# AMDGPUUniformIntrinsicCombine.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUUniformIntrinsicCombine.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUUniformIntrinsicCombine for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPUUniformIntrinsicCombine 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: File banner, license, and overview
```cpp
//===-- AMDGPUUniformIntrinsicCombine.cpp ---------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// This pass simplifies certain intrinsic calls when the arguments are uniform.
/// It's true that this pass has transforms that can lead to a situation where
/// some instruction whose operand was previously recognized as statically
/// uniform is later on no longer recognized as statically uniform. However, the
/// semantics of how programs execute don't (and must not, for this precise
/// reason) care about static uniformity, they only ever care about dynamic
/// uniformity. And every instruction that's downstream and cares about dynamic
/// uniformity must be convergent (and isel will introduce v_readfirstlane for
/// them if their operands can't be proven statically uniform).
//===----------------------------------------------------------------------===//

```
**EN:** This opening section establishes the file identity, licensing terms, and high-level intent before the backend-specific code begins.
**CN:** 开头部分先给出文件标识、许可证信息以及总体意图，为后续后端专用代码建立上下文。

### Lines 21-54: Header dependencies and setup
```cpp
#include "AMDGPU.h"
#include "GCNSubtarget.h"
#include "llvm/Analysis/DomTreeUpdater.h"
#include "llvm/Analysis/LoopInfo.h"
#include "llvm/Analysis/ScalarEvolution.h"
#include "llvm/Analysis/TargetLibraryInfo.h"
#include "llvm/Analysis/UniformityAnalysis.h"
#include "llvm/CodeGen/TargetPassConfig.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/InstIterator.h"
#include "llvm/IR/InstVisitor.h"
#include "llvm/IR/IntrinsicsAMDGPU.h"
#include "llvm/IR/PatternMatch.h"
#include "llvm/InitializePasses.h"
#include "llvm/Target/TargetMachine.h"
#include "llvm/Transforms/Utils/BasicBlockUtils.h"

#define DEBUG_TYPE "amdgpu-uniform-intrinsic-combine"

using namespace llvm;
using namespace llvm::AMDGPU;
using namespace llvm::PatternMatch;

/// Wrapper for querying uniformity info that first checks locally tracked
/// instructions.
static bool
isDivergentUseWithNew(const Use &U, const UniformityInfo &UI,
                      const ValueMap<const Value *, bool> &Tracker) {
  Value *V = U.get();
  if (auto It = Tracker.find(V); It != Tracker.end())
    return !It->second; // divergent if marked false
  return UI.isDivergentAtUse(U);
}

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。

### Lines 55-87: Defines optimizeUniformIntrinsic
```cpp
/// Optimizes uniform intrinsics calls if their operand can be proven uniform.
static bool optimizeUniformIntrinsic(IntrinsicInst &II,
                                     const UniformityInfo &UI,
                                     ValueMap<const Value *, bool> &Tracker) {
  llvm::Intrinsic::ID IID = II.getIntrinsicID();
  /// We deliberately do not simplify readfirstlane with a uniform argument, so
  /// that frontends can use it to force a copy to SGPR and thereby prevent the
  /// backend from generating unwanted waterfall loops.
  switch (IID) {
  case Intrinsic::amdgcn_permlane64:
  case Intrinsic::amdgcn_readlane: {
    Value *Src = II.getArgOperand(0);
    if (isDivergentUseWithNew(II.getOperandUse(0), UI, Tracker))
      return false;
    LLVM_DEBUG(dbgs() << "Replacing " << II << " with " << *Src << '\n');
    II.replaceAllUsesWith(Src);
    II.eraseFromParent();
    return true;
  }
  case Intrinsic::amdgcn_ballot: {
    Value *Src = II.getArgOperand(0);
    if (isDivergentUseWithNew(II.getOperandUse(0), UI, Tracker))
      return false;
    LLVM_DEBUG(dbgs() << "Found uniform ballot intrinsic: " << II << '\n');

    bool Changed = false;
    for (User *U : make_early_inc_range(II.users())) {
      if (auto *ICmp = dyn_cast<ICmpInst>(U)) {
        Value *Op0 = ICmp->getOperand(0);
        Value *Op1 = ICmp->getOperand(1);
        ICmpInst::Predicate Pred = ICmp->getPredicate();
        Value *OtherOp = Op0 == &II ? Op1 : Op0;

```
**EN:** This section contains concrete logic for optimizeUniformIntrinsic. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 optimizeUniformIntrinsic 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 88-120: Conditional logic and checks
```cpp
        if (Pred == ICmpInst::ICMP_EQ && match(OtherOp, m_Zero())) {
          // Case: (icmp eq %ballot, 0) -> xor %ballot_arg, 1
          Instruction *NotOp =
              BinaryOperator::CreateNot(Src, "", ICmp->getIterator());
          Tracker[NotOp] = true; // NOT preserves uniformity
          LLVM_DEBUG(dbgs() << "Replacing ICMP_EQ: " << *NotOp << '\n');
          ICmp->replaceAllUsesWith(NotOp);
          Changed = true;
        } else if (Pred == ICmpInst::ICMP_NE && match(OtherOp, m_Zero())) {
          // Case: (icmp ne %ballot, 0) -> %ballot_arg
          LLVM_DEBUG(dbgs() << "Replacing ICMP_NE with ballot argument: "
                            << *Src << '\n');
          ICmp->replaceAllUsesWith(Src);
          Changed = true;
        }
      }
    }
    // Erase the intrinsic if it has no remaining uses.
    if (II.use_empty())
      II.eraseFromParent();
    return Changed;
  }
  case Intrinsic::amdgcn_wave_shuffle: {
    Use &Val = II.getOperandUse(0);
    Use &Idx = II.getOperandUse(1);

    // Like with readlane, if Value is uniform then just propagate it
    if (!isDivergentUseWithNew(Val, UI, Tracker)) {
      II.replaceAllUsesWith(Val);
      II.eraseFromParent();
      return true;
    }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `BinaryOperator::CreateNot`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`BinaryOperator::CreateNot`。

### Lines 121-151: Conditional logic and checks
```cpp
    // Otherwise, when Index is uniform, this is just a readlane operation
    if (isDivergentUseWithNew(Idx, UI, Tracker))
      return false;

    // The readlane intrinsic we want to call has the exact same function
    // signature, so we can quickly modify the instruction in-place
    Module *Mod = II.getModule();
    II.setCalledFunction(Intrinsic::getOrInsertDeclaration(
        Mod, Intrinsic::amdgcn_readlane, II.getType()));
    return true;
  }
  default:
    return false;
  }
  return false;
}

/// Iterates over intrinsic calls in the Function to optimize.
static bool runUniformIntrinsicCombine(Function &F, const UniformityInfo &UI) {
  bool IsChanged = false;
  ValueMap<const Value *, bool> Tracker;

  for (Instruction &I : make_early_inc_range(instructions(F))) {
    auto *II = dyn_cast<IntrinsicInst>(&I);
    if (!II)
      continue;
    IsChanged |= optimizeUniformIntrinsic(*II, UI, Tracker);
  }
  return IsChanged;
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `Intrinsic::getOrInsertDeclaration`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`Intrinsic::getOrInsertDeclaration`。

### Lines 152-183: Declares class AMDGPUUniformIntrinsicCombineLegacy
```cpp
PreservedAnalyses
AMDGPUUniformIntrinsicCombinePass::run(Function &F,
                                       FunctionAnalysisManager &AM) {
  const auto &UI = AM.getResult<UniformityInfoAnalysis>(F);
  if (!runUniformIntrinsicCombine(F, UI))
    return PreservedAnalyses::all();

  PreservedAnalyses PA;
  PA.preserve<UniformityInfoAnalysis>();
  return PA;
}

namespace {
class AMDGPUUniformIntrinsicCombineLegacy : public FunctionPass {
public:
  static char ID;
  AMDGPUUniformIntrinsicCombineLegacy() : FunctionPass(ID) {}

private:
  bool runOnFunction(Function &F) override;
  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    AU.addRequired<UniformityInfoWrapperPass>();
    AU.addRequired<TargetPassConfig>();
  }
};
} // namespace

char AMDGPUUniformIntrinsicCombineLegacy::ID = 0;
char &llvm::AMDGPUUniformIntrinsicCombineLegacyPassID =
    AMDGPUUniformIntrinsicCombineLegacy::ID;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUUniformIntrinsicCombineLegacy`, `AMDGPUUniformIntrinsicCombinePass::run`, `PreservedAnalyses::all`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUUniformIntrinsicCombineLegacy`, `AMDGPUUniformIntrinsicCombinePass::run`, `PreservedAnalyses::all`。

### Lines 184-201: Registers LLVM passes
```cpp
bool AMDGPUUniformIntrinsicCombineLegacy::runOnFunction(Function &F) {
  if (skipFunction(F))
    return false;
  const UniformityInfo &UI =
      getAnalysis<UniformityInfoWrapperPass>().getUniformityInfo();
  return runUniformIntrinsicCombine(F, UI);
}

INITIALIZE_PASS_BEGIN(AMDGPUUniformIntrinsicCombineLegacy, DEBUG_TYPE,
                      "AMDGPU Uniform Intrinsic Combine", false, false)
INITIALIZE_PASS_DEPENDENCY(UniformityInfoWrapperPass)
INITIALIZE_PASS_DEPENDENCY(TargetPassConfig)
INITIALIZE_PASS_END(AMDGPUUniformIntrinsicCombineLegacy, DEBUG_TYPE,
                    "AMDGPU Uniform Intrinsic Combine", false, false)

FunctionPass *llvm::createAMDGPUUniformIntrinsicCombineLegacyPass() {
  return new AMDGPUUniformIntrinsicCombineLegacy();
}
```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `AMDGPUUniformIntrinsicCombineLegacy::runOnFunction`, `llvm::createAMDGPUUniformIntrinsicCombineLegacyPass`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`AMDGPUUniformIntrinsicCombineLegacy::runOnFunction`, `llvm::createAMDGPUUniformIntrinsicCombineLegacyPass`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `AMDGPUUniformIntrinsicCombineLegacy`, `BinaryOperator::CreateNot`, `Intrinsic::getOrInsertDeclaration`, `AMDGPUUniformIntrinsicCombinePass::run`, `PreservedAnalyses::all`, `AMDGPUUniformIntrinsicCombineLegacy::runOnFunction`
- **Main themes / 核心主题**: instruction semantics / 指令语义; subtarget modeling / 子目标建模; predicates and constraints / 谓词与约束; LLVM pass integration / LLVM Pass 集成
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPU.h"`
- `"GCNSubtarget.h"`
- `"llvm/Analysis/DomTreeUpdater.h"`
- `"llvm/Analysis/LoopInfo.h"`
- `"llvm/Analysis/ScalarEvolution.h"`
- `"llvm/Analysis/TargetLibraryInfo.h"`
- `"llvm/Analysis/UniformityAnalysis.h"`
- `"llvm/CodeGen/TargetPassConfig.h"`
- `"llvm/IR/IRBuilder.h"`
- `"llvm/IR/InstIterator.h"`
- `"llvm/IR/InstVisitor.h"`
- `"llvm/IR/IntrinsicsAMDGPU.h"`
- `"llvm/IR/PatternMatch.h"`
- `"llvm/InitializePasses.h"`
- `"llvm/Target/TargetMachine.h"`
- `"llvm/Transforms/Utils/BasicBlockUtils.h"`
