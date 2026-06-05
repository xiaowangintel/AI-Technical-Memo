# AMDGPUUnifyDivergentExitNodes.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUUnifyDivergentExitNodes.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUUnifyDivergentExitNodes for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPUUnifyDivergentExitNodes 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: File banner, license, and overview
```cpp
//===- AMDGPUUnifyDivergentExitNodes.cpp ----------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This is a variant of the UnifyFunctionExitNodes pass. Rather than ensuring
// there is at most one ret and one unreachable instruction, it ensures there is
// at most one divergent exiting block.
//
// StructurizeCFG can't deal with multi-exit regions formed by branches to
// multiple return nodes. It is not desirable to structurize regions with
// uniform branches, so unifying those to the same return block as divergent
// branches inhibits use of scalar branching. It still can't deal with the case
// where one branch goes to return, and one unreachable. Replace unreachable in
// this case with a return.
//
//===----------------------------------------------------------------------===//

```
**EN:** This opening section establishes the file identity, licensing terms, and high-level intent before the backend-specific code begins.
**CN:** 开头部分先给出文件标识、许可证信息以及总体意图，为后续后端专用代码建立上下文。

### Lines 22-54: Header dependencies and setup
```cpp
#include "AMDGPUUnifyDivergentExitNodes.h"
#include "AMDGPU.h"
#include "llvm/ADT/ArrayRef.h"
#include "llvm/ADT/SmallPtrSet.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/Analysis/DomTreeUpdater.h"
#include "llvm/Analysis/PostDominators.h"
#include "llvm/Analysis/TargetTransformInfo.h"
#include "llvm/Analysis/UniformityAnalysis.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/CFG.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/Dominators.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/InstrTypes.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/Intrinsics.h"
#include "llvm/IR/IntrinsicsAMDGPU.h"
#include "llvm/IR/Type.h"
#include "llvm/InitializePasses.h"
#include "llvm/Pass.h"
#include "llvm/Support/Casting.h"
#include "llvm/Transforms/Scalar.h"
#include "llvm/Transforms/Utils.h"
#include "llvm/Transforms/Utils/BasicBlockUtils.h"
#include "llvm/Transforms/Utils/Local.h"

using namespace llvm;

#define DEBUG_TYPE "amdgpu-unify-divergent-exit-nodes"

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。

### Lines 55-87: Declares class AMDGPUUnifyDivergentExitNodesImpl
```cpp
namespace {

class AMDGPUUnifyDivergentExitNodesImpl {
private:
  const TargetTransformInfo *TTI = nullptr;

public:
  AMDGPUUnifyDivergentExitNodesImpl() = delete;
  AMDGPUUnifyDivergentExitNodesImpl(const TargetTransformInfo *TTI)
      : TTI(TTI) {}

  // We can preserve non-critical-edgeness when we unify function exit nodes
  BasicBlock *unifyReturnBlockSet(Function &F, DomTreeUpdater &DTU,
                                  ArrayRef<BasicBlock *> ReturningBlocks,
                                  StringRef Name);
  bool run(Function &F, DominatorTree *DT, const PostDominatorTree &PDT,
           const UniformityInfo &UA);
};

class AMDGPUUnifyDivergentExitNodesLegacy : public FunctionPass {
public:
  static char ID;
  AMDGPUUnifyDivergentExitNodesLegacy() : FunctionPass(ID) {}
  void getAnalysisUsage(AnalysisUsage &AU) const override;
  bool runOnFunction(Function &F) override;
};
} // end anonymous namespace

char AMDGPUUnifyDivergentExitNodesLegacy::ID = 0;

char &llvm::AMDGPUUnifyDivergentExitNodesID =
    AMDGPUUnifyDivergentExitNodesLegacy::ID;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUUnifyDivergentExitNodesImpl`, `AMDGPUUnifyDivergentExitNodesLegacy`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUUnifyDivergentExitNodesImpl`, `AMDGPUUnifyDivergentExitNodesLegacy`。

### Lines 88-117: Registers LLVM passes
```cpp
INITIALIZE_PASS_BEGIN(AMDGPUUnifyDivergentExitNodesLegacy, DEBUG_TYPE,
                      "Unify divergent function exit nodes", false, false)
INITIALIZE_PASS_DEPENDENCY(DominatorTreeWrapperPass)
INITIALIZE_PASS_DEPENDENCY(PostDominatorTreeWrapperPass)
INITIALIZE_PASS_DEPENDENCY(UniformityInfoWrapperPass)
INITIALIZE_PASS_END(AMDGPUUnifyDivergentExitNodesLegacy, DEBUG_TYPE,
                    "Unify divergent function exit nodes", false, false)

void AMDGPUUnifyDivergentExitNodesLegacy::getAnalysisUsage(
    AnalysisUsage &AU) const {
  if (RequireAndPreserveDomTree)
    AU.addRequired<DominatorTreeWrapperPass>();

  AU.addRequired<PostDominatorTreeWrapperPass>();

  AU.addRequired<UniformityInfoWrapperPass>();

  if (RequireAndPreserveDomTree) {
    AU.addPreserved<DominatorTreeWrapperPass>();
    // FIXME: preserve PostDominatorTreeWrapperPass
  }

  // We preserve the non-critical-edgeness property
  AU.addPreservedID(BreakCriticalEdgesID);

  FunctionPass::getAnalysisUsage(AU);

  AU.addRequired<TargetTransformInfoWrapperPass>();
}

```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `AMDGPUUnifyDivergentExitNodesLegacy::getAnalysisUsage`, `FunctionPass::getAnalysisUsage`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`AMDGPUUnifyDivergentExitNodesLegacy::getAnalysisUsage`, `FunctionPass::getAnalysisUsage`。

### Lines 118-146: Defines isUniformlyReached
```cpp
/// \returns true if \p BB is reachable through only uniform branches.
/// XXX - Is there a more efficient way to find this?
static bool isUniformlyReached(const UniformityInfo &UA, BasicBlock &BB) {
  SmallVector<BasicBlock *, 8> Stack(predecessors(&BB));
  SmallPtrSet<BasicBlock *, 8> Visited;

  while (!Stack.empty()) {
    BasicBlock *Top = Stack.pop_back_val();
    if (UA.isDivergentAtDef(Top->getTerminator()))
      return false;

    for (BasicBlock *Pred : predecessors(Top)) {
      if (Visited.insert(Pred).second)
        Stack.push_back(Pred);
    }
  }

  return true;
}

BasicBlock *AMDGPUUnifyDivergentExitNodesImpl::unifyReturnBlockSet(
    Function &F, DomTreeUpdater &DTU, ArrayRef<BasicBlock *> ReturningBlocks,
    StringRef Name) {
  // Otherwise, we need to insert a new basic block into the function, add a PHI
  // nodes (if the function returns values), and convert all of the return
  // instructions into unconditional branches.
  BasicBlock *NewRetBlock = BasicBlock::Create(F.getContext(), Name, &F);
  IRBuilder<> B(NewRetBlock);

```
**EN:** This section contains concrete logic for isUniformlyReached. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUUnifyDivergentExitNodesImpl::unifyReturnBlockSet`, `BasicBlock::Create`.
**CN:** 本节包含与 isUniformlyReached 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUUnifyDivergentExitNodesImpl::unifyReturnBlockSet`, `BasicBlock::Create`。

### Lines 147-176: Conditional logic and checks
```cpp
  PHINode *PN = nullptr;
  if (F.getReturnType()->isVoidTy()) {
    B.CreateRetVoid();
  } else {
    // If the function doesn't return void... add a PHI node to the block...
    PN = B.CreatePHI(F.getReturnType(), ReturningBlocks.size(),
                     "UnifiedRetVal");
    B.CreateRet(PN);
  }

  // Loop over all of the blocks, replacing the return instruction with an
  // unconditional branch.
  std::vector<DominatorTree::UpdateType> Updates;
  Updates.reserve(ReturningBlocks.size());
  for (BasicBlock *BB : ReturningBlocks) {
    // Add an incoming element to the PHI node for every return instruction that
    // is merging into this new block...
    if (PN)
      PN->addIncoming(BB->getTerminator()->getOperand(0), BB);

    // Remove and delete the return inst.
    BB->getTerminator()->eraseFromParent();
    UncondBrInst::Create(NewRetBlock, BB);
    Updates.emplace_back(DominatorTree::Insert, BB, NewRetBlock);
  }

  if (RequireAndPreserveDomTree)
    DTU.applyUpdates(Updates);
  Updates.clear();

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `UncondBrInst::Create`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`UncondBrInst::Create`。

### Lines 177-209: Defines createDummyReturnBlock
```cpp
  for (BasicBlock *BB : ReturningBlocks) {
    // Cleanup possible branch to unconditional branch to the return.
    simplifyCFG(BB, *TTI, RequireAndPreserveDomTree ? &DTU : nullptr,
                SimplifyCFGOptions().bonusInstThreshold(2));
  }

  return NewRetBlock;
}

static BasicBlock *
createDummyReturnBlock(Function &F,
                       SmallVector<BasicBlock *, 4> &ReturningBlocks) {
  BasicBlock *DummyReturnBB =
      BasicBlock::Create(F.getContext(), "DummyReturnBlock", &F);
  Type *RetTy = F.getReturnType();
  Value *RetVal = RetTy->isVoidTy() ? nullptr : PoisonValue::get(RetTy);
  ReturnInst::Create(F.getContext(), RetVal, DummyReturnBB);
  ReturningBlocks.push_back(DummyReturnBB);
  return DummyReturnBB;
}

/// Handle conditional branch instructions (-> 2 targets) and callbr
/// instructions with N targets.
static void handleNBranch(Function &F, BasicBlock *BB, Instruction *BI,
                          BasicBlock *DummyReturnBB,
                          std::vector<DominatorTree::UpdateType> &Updates) {
  SmallVector<BasicBlock *, 2> Successors(successors(BB));

  // Create a new transition block to hold the conditional branch.
  BasicBlock *TransitionBB = BB->splitBasicBlock(BI, "TransitionBlock");

  Updates.reserve(Updates.size() + 2 * Successors.size() + 2);

```
**EN:** This section contains concrete logic for createDummyReturnBlock. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `BasicBlock::Create`, `PoisonValue::get`, `ReturnInst::Create`.
**CN:** 本节包含与 createDummyReturnBlock 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`BasicBlock::Create`, `PoisonValue::get`, `ReturnInst::Create`。

### Lines 210-241: Declares emplace_back
```cpp
  // 'Successors' become successors of TransitionBB instead of BB,
  // and TransitionBB becomes a single successor of BB.
  Updates.emplace_back(DominatorTree::Insert, BB, TransitionBB);
  for (BasicBlock *Successor : Successors) {
    Updates.emplace_back(DominatorTree::Insert, TransitionBB, Successor);
    Updates.emplace_back(DominatorTree::Delete, BB, Successor);
  }

  // Create a branch that will always branch to the transition block and
  // references DummyReturnBB.
  BB->getTerminator()->eraseFromParent();
  CondBrInst::Create(ConstantInt::getTrue(F.getContext()), TransitionBB,
                     DummyReturnBB, BB);
  Updates.emplace_back(DominatorTree::Insert, BB, DummyReturnBB);
}

bool AMDGPUUnifyDivergentExitNodesImpl::run(Function &F, DominatorTree *DT,
                                            const PostDominatorTree &PDT,
                                            const UniformityInfo &UA) {
  if (PDT.root_size() == 0 ||
      (PDT.root_size() == 1 && !isa<UncondBrInst, CondBrInst, CallBrInst>(
                                   PDT.getRoot()->getTerminator())))
    return false;

  // Loop over all of the blocks in a function, tracking all of the blocks that
  // return.
  SmallVector<BasicBlock *, 4> ReturningBlocks;
  SmallVector<BasicBlock *, 4> UnreachableBlocks;

  // Dummy return block for infinite loop.
  BasicBlock *DummyReturnBB = nullptr;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `CondBrInst::Create`, `ConstantInt::getTrue`, `AMDGPUUnifyDivergentExitNodesImpl::run`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`CondBrInst::Create`, `ConstantInt::getTrue`, `AMDGPUUnifyDivergentExitNodesImpl::run`。

### Lines 242-268: Implements llvm::any_of
```cpp
  bool Changed = false;
  std::vector<DominatorTree::UpdateType> Updates;

  // TODO: For now we unify all exit blocks, even though they are uniformly
  // reachable, if there are any exits not uniformly reached. This is to
  // workaround the limitation of structurizer, which can not handle multiple
  // function exits. After structurizer is able to handle multiple function
  // exits, we should only unify UnreachableBlocks that are not uniformly
  // reachable.
  bool HasDivergentExitBlock = llvm::any_of(
      PDT.roots(), [&](auto BB) { return !isUniformlyReached(UA, *BB); });

  for (BasicBlock *BB : PDT.roots()) {
    Instruction *Term = BB->getTerminator();
    if (auto *RI = dyn_cast<ReturnInst>(Term)) {
      auto *CI = dyn_cast_or_null<CallInst>(RI->getPrevNode());
      if (CI && CI->isMustTailCall())
        continue;
      if (HasDivergentExitBlock)
        ReturningBlocks.push_back(BB);
    } else if (isa<UnreachableInst>(Term)) {
      if (HasDivergentExitBlock)
        UnreachableBlocks.push_back(BB);
    } else if (UncondBrInst *BI = dyn_cast<UncondBrInst>(Term)) {
      if (!DummyReturnBB)
        DummyReturnBB = createDummyReturnBlock(F, ReturningBlocks);

```
**EN:** This section contains concrete logic for llvm::any_of. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `llvm::any_of`.
**CN:** 本节包含与 llvm::any_of 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`llvm::any_of`。

### Lines 269-296: Conditional logic and checks
```cpp
      BasicBlock *LoopHeaderBB = BI->getSuccessor();
      BI->eraseFromParent(); // Delete the unconditional branch.
      // Add a new conditional branch with a dummy edge to the return block.
      CondBrInst::Create(ConstantInt::getTrue(F.getContext()), LoopHeaderBB,
                         DummyReturnBB, BB);
      Updates.emplace_back(DominatorTree::Insert, BB, DummyReturnBB);
      Changed = true;
    } else if (isa<CondBrInst, CallBrInst>(Term)) {
      if (!DummyReturnBB)
        DummyReturnBB = createDummyReturnBlock(F, ReturningBlocks);

      handleNBranch(F, BB, Term, DummyReturnBB, Updates);
      Changed = true;
    } else {
      llvm_unreachable("unsupported block terminator");
    }
  }

  if (!UnreachableBlocks.empty()) {
    BasicBlock *UnreachableBlock = nullptr;

    if (UnreachableBlocks.size() == 1) {
      UnreachableBlock = UnreachableBlocks.front();
    } else {
      UnreachableBlock = BasicBlock::Create(F.getContext(),
                                            "UnifiedUnreachableBlock", &F);
      new UnreachableInst(F.getContext(), UnreachableBlock);

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `CondBrInst::Create`, `ConstantInt::getTrue`, `BasicBlock::Create`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`CondBrInst::Create`, `ConstantInt::getTrue`, `BasicBlock::Create`。

### Lines 297-322: Conditional logic and checks
```cpp
      Updates.reserve(Updates.size() + UnreachableBlocks.size());
      for (BasicBlock *BB : UnreachableBlocks) {
        // Remove and delete the unreachable inst.
        BB->getTerminator()->eraseFromParent();
        UncondBrInst::Create(UnreachableBlock, BB);
        Updates.emplace_back(DominatorTree::Insert, BB, UnreachableBlock);
      }
      Changed = true;
    }

    if (!ReturningBlocks.empty()) {
      // Don't create a new unreachable inst if we have a return. The
      // structurizer/annotator can't handle the multiple exits

      Type *RetTy = F.getReturnType();
      Value *RetVal = RetTy->isVoidTy() ? nullptr : PoisonValue::get(RetTy);
      // Remove and delete the unreachable inst.
      UnreachableBlock->getTerminator()->eraseFromParent();

      Function *UnreachableIntrin = Intrinsic::getOrInsertDeclaration(
          F.getParent(), Intrinsic::amdgcn_unreachable);

      // Insert a call to an intrinsic tracking that this is an unreachable
      // point, in case we want to kill the active lanes or something later.
      CallInst::Create(UnreachableIntrin, {}, "", UnreachableBlock);

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `UncondBrInst::Create`, `PoisonValue::get`, `Intrinsic::getOrInsertDeclaration`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`UncondBrInst::Create`, `PoisonValue::get`, `Intrinsic::getOrInsertDeclaration`。

### Lines 323-348: Declares DTU
```cpp
      // Don't create a scalar trap. We would only want to trap if this code was
      // really reached, but a scalar trap would happen even if no lanes
      // actually reached here.
      ReturnInst::Create(F.getContext(), RetVal, UnreachableBlock);
      ReturningBlocks.push_back(UnreachableBlock);
      Changed = true;
    }
  }

  // FIXME: add PDT here once simplifycfg is ready.
  DomTreeUpdater DTU(DT, DomTreeUpdater::UpdateStrategy::Eager);
  if (RequireAndPreserveDomTree)
    DTU.applyUpdates(Updates);
  Updates.clear();

  // Now handle return blocks.
  if (ReturningBlocks.empty())
    return Changed; // No blocks return

  if (ReturningBlocks.size() == 1)
    return Changed; // Already has a single return block

  unifyReturnBlockSet(F, DTU, ReturningBlocks, "UnifiedReturnBlock");
  return true;
}

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `ReturnInst::Create`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`ReturnInst::Create`。

### Lines 349-374: Implements AMDGPUUnifyDivergentExitNodesLegacy::runOnFunction
```cpp
bool AMDGPUUnifyDivergentExitNodesLegacy::runOnFunction(Function &F) {
  DominatorTree *DT = nullptr;
  if (RequireAndPreserveDomTree)
    DT = &getAnalysis<DominatorTreeWrapperPass>().getDomTree();
  const auto &PDT =
      getAnalysis<PostDominatorTreeWrapperPass>().getPostDomTree();
  const auto &UA = getAnalysis<UniformityInfoWrapperPass>().getUniformityInfo();
  const auto *TranformInfo =
      &getAnalysis<TargetTransformInfoWrapperPass>().getTTI(F);
  return AMDGPUUnifyDivergentExitNodesImpl(TranformInfo).run(F, DT, PDT, UA);
}

PreservedAnalyses
AMDGPUUnifyDivergentExitNodesPass::run(Function &F,
                                       FunctionAnalysisManager &AM) {
  DominatorTree *DT = nullptr;
  if (RequireAndPreserveDomTree)
    DT = &AM.getResult<DominatorTreeAnalysis>(F);

  const auto &PDT = AM.getResult<PostDominatorTreeAnalysis>(F);
  const auto &UA = AM.getResult<UniformityInfoAnalysis>(F);
  const auto *TransformInfo = &AM.getResult<TargetIRAnalysis>(F);
  return AMDGPUUnifyDivergentExitNodesImpl(TransformInfo).run(F, DT, PDT, UA)
             ? PreservedAnalyses::none()
             : PreservedAnalyses::all();
}
```
**EN:** This section contains concrete logic for AMDGPUUnifyDivergentExitNodesLegacy::runOnFunction. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUUnifyDivergentExitNodesLegacy::runOnFunction`, `AMDGPUUnifyDivergentExitNodesPass::run`, `PreservedAnalyses::none`.
**CN:** 本节包含与 AMDGPUUnifyDivergentExitNodesLegacy::runOnFunction 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUUnifyDivergentExitNodesLegacy::runOnFunction`, `AMDGPUUnifyDivergentExitNodesPass::run`, `PreservedAnalyses::none`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `AMDGPUUnifyDivergentExitNodesImpl`, `AMDGPUUnifyDivergentExitNodesLegacy`, `AMDGPUUnifyDivergentExitNodesLegacy::getAnalysisUsage`, `FunctionPass::getAnalysisUsage`, `AMDGPUUnifyDivergentExitNodesImpl::unifyReturnBlockSet`, `BasicBlock::Create`
- **Main themes / 核心主题**: instruction semantics / 指令语义; LLVM pass integration / LLVM Pass 集成
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPUUnifyDivergentExitNodes.h"`
- `"AMDGPU.h"`
- `"llvm/ADT/ArrayRef.h"`
- `"llvm/ADT/SmallPtrSet.h"`
- `"llvm/ADT/SmallVector.h"`
- `"llvm/ADT/StringRef.h"`
- `"llvm/Analysis/DomTreeUpdater.h"`
- `"llvm/Analysis/PostDominators.h"`
- `"llvm/Analysis/TargetTransformInfo.h"`
- `"llvm/Analysis/UniformityAnalysis.h"`
- `"llvm/IR/BasicBlock.h"`
- `"llvm/IR/CFG.h"`
- `"llvm/IR/Constants.h"`
- `"llvm/IR/Dominators.h"`
- `"llvm/IR/Function.h"`
- `"llvm/IR/IRBuilder.h"`
- `"llvm/IR/InstrTypes.h"`
- `"llvm/IR/Instructions.h"`
- `"llvm/IR/Intrinsics.h"`
- `"llvm/IR/IntrinsicsAMDGPU.h"`
- ... and 8 more direct dependencies / 以及另外 8 个直接依赖
