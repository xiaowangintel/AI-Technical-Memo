# AMDGPUPromoteKernelArguments.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUPromoteKernelArguments.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUPromoteKernelArguments for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPUPromoteKernelArguments 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-34: File banner, includes, and setup
```cpp
//===-- AMDGPUPromoteKernelArguments.cpp ----------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file This pass recursively promotes generic pointer arguments of a kernel
/// into the global address space.
///
/// The pass walks kernel's pointer arguments, then loads from them. If a loaded
/// value is a pointer and loaded pointer is unmodified in the kernel before the
/// load, then promote loaded pointer to global. Then recursively continue.
//
//===----------------------------------------------------------------------===//

#include "AMDGPU.h"
#include "AMDGPUMemoryUtils.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/Analysis/AliasAnalysis.h"
#include "llvm/Analysis/MemorySSA.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/InitializePasses.h"

#define DEBUG_TYPE "amdgpu-promote-kernel-arguments"

using namespace llvm;

namespace {

class AMDGPUPromoteKernelArguments : public FunctionPass {
  MemorySSA *MSSA;

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `AMDGPUPromoteKernelArguments`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`AMDGPUPromoteKernelArguments`。

### Lines 35-67: Declares enqueueUsers
```cpp
  AliasAnalysis *AA;

  Instruction *ArgCastInsertPt;

  SmallVector<Value *> Ptrs;

  void enqueueUsers(Value *Ptr);

  bool promotePointer(Value *Ptr);

  bool promoteLoad(LoadInst *LI);

public:
  static char ID;

  AMDGPUPromoteKernelArguments() : FunctionPass(ID) {}

  bool run(Function &F, MemorySSA &MSSA, AliasAnalysis &AA);

  bool runOnFunction(Function &F) override;

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addRequired<AAResultsWrapperPass>();
    AU.addRequired<MemorySSAWrapperPass>();
    AU.setPreservesAll();
  }
};

} // end anonymous namespace

void AMDGPUPromoteKernelArguments::enqueueUsers(Value *Ptr) {
  SmallVector<User *> PtrUsers(Ptr->users());

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `AMDGPUPromoteKernelArguments::enqueueUsers`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`AMDGPUPromoteKernelArguments::enqueueUsers`。

### Lines 68-100: Switch-based control flow
```cpp
  while (!PtrUsers.empty()) {
    Instruction *U = dyn_cast<Instruction>(PtrUsers.pop_back_val());
    if (!U)
      continue;

    switch (U->getOpcode()) {
    default:
      break;
    case Instruction::Load: {
      LoadInst *LD = cast<LoadInst>(U);
      if (LD->getPointerOperand()->stripInBoundsOffsets() == Ptr &&
          !AMDGPU::isClobberedInFunction(LD, MSSA, AA))
        Ptrs.push_back(LD);

      break;
    }
    case Instruction::GetElementPtr:
    case Instruction::AddrSpaceCast:
    case Instruction::BitCast:
      if (U->getOperand(0)->stripInBoundsOffsets() == Ptr)
        PtrUsers.append(U->user_begin(), U->user_end());
      break;
    }
  }
}

bool AMDGPUPromoteKernelArguments::promotePointer(Value *Ptr) {
  bool Changed = false;

  LoadInst *LI = dyn_cast<LoadInst>(Ptr);
  if (LI)
    Changed |= promoteLoad(LI);

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::isClobberedInFunction`, `AMDGPUPromoteKernelArguments::promotePointer`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::isClobberedInFunction`, `AMDGPUPromoteKernelArguments::promotePointer`。

### Lines 101-133: Declares getType
```cpp
  PointerType *PT = dyn_cast<PointerType>(Ptr->getType());
  if (!PT)
    return Changed;

  if (PT->getAddressSpace() == AMDGPUAS::FLAT_ADDRESS ||
      PT->getAddressSpace() == AMDGPUAS::GLOBAL_ADDRESS ||
      PT->getAddressSpace() == AMDGPUAS::CONSTANT_ADDRESS)
    enqueueUsers(Ptr);

  if (PT->getAddressSpace() != AMDGPUAS::FLAT_ADDRESS)
    return Changed;

  IRBuilder<> B(LI ? &*std::next(cast<Instruction>(Ptr)->getIterator())
                   : ArgCastInsertPt);

  // Cast pointer to global address space and back to flat and let
  // Infer Address Spaces pass to do all necessary rewriting.
  PointerType *NewPT =
      PointerType::get(PT->getContext(), AMDGPUAS::GLOBAL_ADDRESS);
  Value *Cast =
      B.CreateAddrSpaceCast(Ptr, NewPT, Twine(Ptr->getName(), ".global"));
  Value *CastBack =
      B.CreateAddrSpaceCast(Cast, PT, Twine(Ptr->getName(), ".flat"));
  Ptr->replaceUsesWithIf(CastBack,
                         [Cast](Use &U) { return U.getUser() != Cast; });

  return true;
}

bool AMDGPUPromoteKernelArguments::promoteLoad(LoadInst *LI) {
  if (!LI->isSimple())
    return false;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `std::next`, `PointerType::get`, `AMDGPUPromoteKernelArguments::promoteLoad`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`std::next`, `PointerType::get`, `AMDGPUPromoteKernelArguments::promoteLoad`。

### Lines 134-165: Implements MDNode::get
```cpp
  LI->setMetadata("amdgpu.noclobber", MDNode::get(LI->getContext(), {}));
  return true;
}

// skip allocas
static BasicBlock::iterator getInsertPt(BasicBlock &BB) {
  BasicBlock::iterator InsPt = BB.getFirstInsertionPt();
  for (BasicBlock::iterator E = BB.end(); InsPt != E; ++InsPt) {
    AllocaInst *AI = dyn_cast<AllocaInst>(&*InsPt);

    // If this is a dynamic alloca, the value may depend on the loaded kernargs,
    // so loads will need to be inserted before it.
    if (!AI || !AI->isStaticAlloca())
      break;
  }

  return InsPt;
}

bool AMDGPUPromoteKernelArguments::run(Function &F, MemorySSA &MSSA,
                                       AliasAnalysis &AA) {
  if (skipFunction(F))
    return false;

  CallingConv::ID CC = F.getCallingConv();
  if (CC != CallingConv::AMDGPU_KERNEL || F.arg_empty())
    return false;

  ArgCastInsertPt = &*getInsertPt(*F.begin());
  this->MSSA = &MSSA;
  this->AA = &AA;

```
**EN:** This section contains concrete logic for MDNode::get. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `MDNode::get`, `AMDGPUPromoteKernelArguments::run`.
**CN:** 本节包含与 MDNode::get 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`MDNode::get`, `AMDGPUPromoteKernelArguments::run`。

### Lines 166-193: Conditional logic and checks
```cpp
  for (Argument &Arg : F.args()) {
    if (Arg.use_empty())
      continue;

    PointerType *PT = dyn_cast<PointerType>(Arg.getType());
    if (!PT || (PT->getAddressSpace() != AMDGPUAS::FLAT_ADDRESS &&
                PT->getAddressSpace() != AMDGPUAS::GLOBAL_ADDRESS &&
                PT->getAddressSpace() != AMDGPUAS::CONSTANT_ADDRESS))
      continue;

    Ptrs.push_back(&Arg);
  }

  bool Changed = false;
  while (!Ptrs.empty()) {
    Value *Ptr = Ptrs.pop_back_val();
    Changed |= promotePointer(Ptr);
  }

  return Changed;
}

bool AMDGPUPromoteKernelArguments::runOnFunction(Function &F) {
  MemorySSA &MSSA = getAnalysis<MemorySSAWrapperPass>().getMSSA();
  AliasAnalysis &AA = getAnalysis<AAResultsWrapperPass>().getAAResults();
  return run(F, MSSA, AA);
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPUPromoteKernelArguments::runOnFunction`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPUPromoteKernelArguments::runOnFunction`。

### Lines 194-219: Registers LLVM passes
```cpp
INITIALIZE_PASS_BEGIN(AMDGPUPromoteKernelArguments, DEBUG_TYPE,
                      "AMDGPU Promote Kernel Arguments", false, false)
INITIALIZE_PASS_DEPENDENCY(AAResultsWrapperPass)
INITIALIZE_PASS_DEPENDENCY(MemorySSAWrapperPass)
INITIALIZE_PASS_END(AMDGPUPromoteKernelArguments, DEBUG_TYPE,
                    "AMDGPU Promote Kernel Arguments", false, false)

char AMDGPUPromoteKernelArguments::ID = 0;

FunctionPass *llvm::createAMDGPUPromoteKernelArgumentsPass() {
  return new AMDGPUPromoteKernelArguments();
}

PreservedAnalyses
AMDGPUPromoteKernelArgumentsPass::run(Function &F,
                                      FunctionAnalysisManager &AM) {
  MemorySSA &MSSA = AM.getResult<MemorySSAAnalysis>(F).getMSSA();
  AliasAnalysis &AA = AM.getResult<AAManager>(F);
  if (AMDGPUPromoteKernelArguments().run(F, MSSA, AA)) {
    PreservedAnalyses PA;
    PA.preserveSet<CFGAnalyses>();
    PA.preserve<MemorySSAAnalysis>();
    return PA;
  }
  return PreservedAnalyses::all();
}
```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `llvm::createAMDGPUPromoteKernelArgumentsPass`, `AMDGPUPromoteKernelArgumentsPass::run`, `PreservedAnalyses::all`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`llvm::createAMDGPUPromoteKernelArgumentsPass`, `AMDGPUPromoteKernelArgumentsPass::run`, `PreservedAnalyses::all`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `AMDGPUPromoteKernelArguments`, `AMDGPUPromoteKernelArguments::enqueueUsers`, `AMDGPU::isClobberedInFunction`, `AMDGPUPromoteKernelArguments::promotePointer`, `std::next`, `PointerType::get`
- **Main themes / 核心主题**: alias analysis / 别名分析; instruction semantics / 指令语义; LLVM pass integration / LLVM Pass 集成
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPU.h"`
- `"AMDGPUMemoryUtils.h"`
- `"llvm/ADT/SmallVector.h"`
- `"llvm/Analysis/AliasAnalysis.h"`
- `"llvm/Analysis/MemorySSA.h"`
- `"llvm/IR/IRBuilder.h"`
- `"llvm/InitializePasses.h"`
