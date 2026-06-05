# AMDGPULowerExecSync.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPULowerExecSync.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPULowerExecSync for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPULowerExecSync 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-34: File banner, includes, and setup
```cpp
//===----------------------------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Lower LDS global variables with target extension type "amdgpu.named.barrier"
// that require specialized address assignment. It assigns a unique
// barrier identifier to each named-barrier LDS variable and encodes
// this identifier within the !absolute_symbol metadata of that global.
// This encoding ensures that subsequent LDS lowering passes can process these
// barriers correctly without conflicts.
//
//===----------------------------------------------------------------------===//

#include "AMDGPU.h"
#include "AMDGPUMemoryUtils.h"
#include "AMDGPUTargetMachine.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/Analysis/CallGraph.h"
#include "llvm/CodeGen/TargetPassConfig.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/ReplaceConstant.h"
#include "llvm/InitializePasses.h"
#include "llvm/Pass.h"
#include "llvm/Transforms/Utils/ModuleUtils.h"

#include <algorithm>

#define DEBUG_TYPE "amdgpu-lower-exec-sync"

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 35-68: Defines uniquifyGVPerKernel
```cpp
using namespace llvm;
using namespace AMDGPU;

namespace {

// If GV is also used directly by other kernels, create a new GV
// used only by this kernel and its function.
static GlobalVariable *uniquifyGVPerKernel(Module &M, GlobalVariable *GV,
                                           Function *KF) {
  bool NeedsReplacement = false;
  for (Use &U : GV->uses()) {
    if (auto *I = dyn_cast<Instruction>(U.getUser())) {
      Function *F = I->getFunction();
      if (isKernel(*F) && F != KF) {
        NeedsReplacement = true;
        break;
      }
    }
  }
  if (!NeedsReplacement)
    return GV;
  // Create a new GV used only by this kernel and its function
  GlobalVariable *NewGV = new GlobalVariable(
      M, GV->getValueType(), GV->isConstant(), GV->getLinkage(),
      GV->getInitializer(), GV->getName() + "." + KF->getName(), nullptr,
      GV->getThreadLocalMode(), GV->getType()->getAddressSpace());
  NewGV->copyAttributesFrom(GV);
  for (Use &U : make_early_inc_range(GV->uses())) {
    if (auto *I = dyn_cast<Instruction>(U.getUser())) {
      Function *F = I->getFunction();
      if (!isKernel(*F) || F == KF) {
        U.getUser()->replaceUsesOfWith(GV, NewGV);
      }
    }
```
**EN:** This section contains concrete logic for uniquifyGVPerKernel. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 uniquifyGVPerKernel 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 69-91: Defines recordLDSAbsoluteAddress
```cpp
  }
  return NewGV;
}

// Write the specified address into metadata where it can be retrieved by
// the assembler. Format is a half open range, [Address Address+1)
static void recordLDSAbsoluteAddress(Module *M, GlobalVariable *GV,
                                     uint32_t Address) {
  LLVMContext &Ctx = M->getContext();
  auto *IntTy = M->getDataLayout().getIntPtrType(Ctx, AMDGPUAS::LOCAL_ADDRESS);
  auto *MinC = ConstantAsMetadata::get(ConstantInt::get(IntTy, Address));
  auto *MaxC = ConstantAsMetadata::get(ConstantInt::get(IntTy, Address + 1));
  GV->setMetadata(LLVMContext::MD_absolute_symbol,
                  MDNode::get(Ctx, {MinC, MaxC}));
}

template <typename T> SmallVector<T> sortByName(SmallVector<T> &&V) {
  sort(V, [](const auto *L, const auto *R) {
    return L->getName() < R->getName();
  });
  return {std::move(V)};
}

```
**EN:** This section contains concrete logic for recordLDSAbsoluteAddress. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `ConstantAsMetadata::get`, `ConstantInt::get`, `MDNode::get`.
**CN:** 本节包含与 recordLDSAbsoluteAddress 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`ConstantAsMetadata::get`, `ConstantInt::get`, `MDNode::get`。

### Lines 92-123: Defines lowerExecSyncGlobalVariables
```cpp
// Main utility function for special LDS variables lowering.
static bool lowerExecSyncGlobalVariables(
    Module &M, LDSUsesInfoTy &LDSUsesInfo,
    VariableFunctionMap &LDSToKernelsThatNeedToAccessItIndirectly) {
  bool Changed = false;
  const DataLayout &DL = M.getDataLayout();
  // The 1st round: give module-absolute assignments
  int NumAbsolutes = 0;
  SmallVector<GlobalVariable *> OrderedGVs;
  for (auto &K : LDSToKernelsThatNeedToAccessItIndirectly) {
    GlobalVariable *GV = K.first;
    if (!isNamedBarrier(*GV))
      continue;
    // give a module-absolute assignment if it is indirectly accessed by
    // multiple kernels. This is not precise, but we don't want to duplicate
    // a function when it is called by multiple kernels.
    if (LDSToKernelsThatNeedToAccessItIndirectly[GV].size() > 1) {
      OrderedGVs.push_back(GV);
    } else {
      // leave it to the 2nd round, which will give a kernel-relative
      // assignment if it is only indirectly accessed by one kernel
      LDSUsesInfo.direct_access[*K.second.begin()].insert(GV);
    }
    LDSToKernelsThatNeedToAccessItIndirectly.erase(GV);
  }
  OrderedGVs = sortByName(std::move(OrderedGVs));
  for (GlobalVariable *GV : OrderedGVs) {
    unsigned BarrierScope = AMDGPU::Barrier::BARRIER_SCOPE_WORKGROUP;
    unsigned BarId = NumAbsolutes + 1;
    unsigned BarCnt = GV->getGlobalSize(DL) / 16;
    NumAbsolutes += BarCnt;

```
**EN:** This section contains concrete logic for lowerExecSyncGlobalVariables. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `std::move`.
**CN:** 本节包含与 lowerExecSyncGlobalVariables 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`std::move`。

### Lines 124-147: Declares clear
```cpp
    // 4 bits for alignment, 5 bits for the barrier num,
    // 3 bits for the barrier scope
    unsigned Offset = 0x802000u | BarrierScope << 9 | BarId << 4;
    recordLDSAbsoluteAddress(&M, GV, Offset);
  }
  OrderedGVs.clear();

  // The 2nd round: give a kernel-relative assignment for GV that
  // either only indirectly accessed by single kernel or only directly
  // accessed by multiple kernels.
  SmallVector<Function *> OrderedKernels;
  for (auto &K : LDSUsesInfo.direct_access) {
    Function *F = K.first;
    assert(isKernel(*F));
    OrderedKernels.push_back(F);
  }
  OrderedKernels = sortByName(std::move(OrderedKernels));

  DenseMap<Function *, uint32_t> Kernel2BarId;
  for (Function *F : OrderedKernels) {
    for (GlobalVariable *GV : LDSUsesInfo.direct_access[F]) {
      if (!isNamedBarrier(*GV))
        continue;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `std::move`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`std::move`。

### Lines 148-181: Conditional logic and checks
```cpp
      LDSUsesInfo.direct_access[F].erase(GV);
      if (GV->isAbsoluteSymbolRef()) {
        // already assigned
        continue;
      }
      OrderedGVs.push_back(GV);
    }
    OrderedGVs = sortByName(std::move(OrderedGVs));
    for (GlobalVariable *GV : OrderedGVs) {
      // GV could also be used directly by other kernels. If so, we need to
      // create a new GV used only by this kernel and its function.
      auto NewGV = uniquifyGVPerKernel(M, GV, F);
      Changed |= (NewGV != GV);
      unsigned BarrierScope = AMDGPU::Barrier::BARRIER_SCOPE_WORKGROUP;
      unsigned BarId = Kernel2BarId[F];
      BarId += NumAbsolutes + 1;
      unsigned BarCnt = GV->getGlobalSize(DL) / 16;
      Kernel2BarId[F] += BarCnt;
      unsigned Offset = 0x802000u | BarrierScope << 9 | BarId << 4;
      recordLDSAbsoluteAddress(&M, NewGV, Offset);
    }
    OrderedGVs.clear();
  }
  // Also erase those special LDS variables from indirect_access.
  for (auto &K : LDSUsesInfo.indirect_access) {
    assert(isKernel(*K.first));
    for (GlobalVariable *GV : K.second) {
      if (isNamedBarrier(*GV))
        K.second.erase(GV);
    }
  }
  return Changed;
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::move`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::move`。

### Lines 182-212: Defines handleNamedBarriersForObjectLinking
```cpp
// With object linking, barrier ID assignment is deferred to the linker.
// Externalize named barrier globals and emit self-contained metadata so the
// AsmPrinter can generate the callgraph entries the linker needs.
static bool handleNamedBarriersForObjectLinking(Module &M) {
  DenseMap<GlobalVariable *, DenseSet<Function *>> BarrierToFuncs;
  for (GlobalVariable &GV : M.globals()) {
    if (!isNamedBarrier(GV) || GV.use_empty())
      continue;
    for (User *U : GV.users()) {
      if (auto *I = dyn_cast<Instruction>(U))
        BarrierToFuncs[&GV].insert(I->getFunction());
    }
  }
  if (BarrierToFuncs.empty())
    return false;

  LLVMContext &Ctx = M.getContext();
  NamedMDNode *BarMD = M.getOrInsertNamedMetadata("amdgpu.named_barrier.uses");

  std::string ModuleId;
  ModuleId = getUniqueModuleId(&M);
  assert(!ModuleId.empty() &&
         "modules with named barriers should have a unique ID");
  for (auto &[V, Funcs] : BarrierToFuncs) {
    if (V->hasLocalLinkage())
      V->setName("__amdgpu_named_barrier." + V->getName() + ModuleId);
    else if (!V->getName().starts_with("__amdgpu_named_barrier"))
      V->setName("__amdgpu_named_barrier." + V->getName());
    V->setInitializer(nullptr);
    V->setLinkage(GlobalValue::ExternalLinkage);

```
**EN:** This section contains concrete logic for handleNamedBarriersForObjectLinking. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 handleNamedBarriersForObjectLinking 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 213-243: Defines runLowerExecSyncGlobals
```cpp
    SmallVector<Metadata *, 4> Ops;
    Ops.push_back(ValueAsMetadata::get(V));
    for (Function *F : Funcs)
      Ops.push_back(ValueAsMetadata::get(F));
    BarMD->addOperand(MDNode::get(Ctx, Ops));
  }
  return true;
}

static bool runLowerExecSyncGlobals(Module &M) {
  if (AMDGPUTargetMachine::EnableObjectLinking)
    return handleNamedBarriersForObjectLinking(M);

  CallGraph CG = CallGraph(M);
  bool Changed = false;
  Changed |= eliminateConstantExprUsesOfLDSFromAllInstructions(M);

  // For each kernel, what variables does it access directly or through
  // callees
  LDSUsesInfoTy LDSUsesInfo = getTransitiveUsesOfLDS(CG, M);

  // For each variable accessed through callees, which kernels access it
  VariableFunctionMap LDSToKernelsThatNeedToAccessItIndirectly;
  for (auto &K : LDSUsesInfo.indirect_access) {
    Function *F = K.first;
    assert(isKernel(*F));
    for (GlobalVariable *GV : K.second) {
      LDSToKernelsThatNeedToAccessItIndirectly[GV].insert(F);
    }
  }

```
**EN:** This section contains concrete logic for runLowerExecSyncGlobals. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `ValueAsMetadata::get`, `MDNode::get`.
**CN:** 本节包含与 runLowerExecSyncGlobals 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`ValueAsMetadata::get`, `MDNode::get`。

### Lines 244-275: Registers LLVM passes
```cpp
  if (LDSUsesInfo.HasSpecialGVs) {
    // Special LDS variables need special address assignment
    Changed |= lowerExecSyncGlobalVariables(
        M, LDSUsesInfo, LDSToKernelsThatNeedToAccessItIndirectly);
  }
  return Changed;
}

class AMDGPULowerExecSyncLegacy : public ModulePass {
public:
  static char ID;
  AMDGPULowerExecSyncLegacy() : ModulePass(ID) {}
  bool runOnModule(Module &M) override;
};

} // namespace

char AMDGPULowerExecSyncLegacy::ID = 0;
char &llvm::AMDGPULowerExecSyncLegacyPassID = AMDGPULowerExecSyncLegacy::ID;

INITIALIZE_PASS_BEGIN(AMDGPULowerExecSyncLegacy, DEBUG_TYPE,
                      "AMDGPU lowering of execution synchronization", false,
                      false)
INITIALIZE_PASS_DEPENDENCY(TargetPassConfig)
INITIALIZE_PASS_END(AMDGPULowerExecSyncLegacy, DEBUG_TYPE,
                    "AMDGPU lowering of execution synchronization", false,
                    false)

bool AMDGPULowerExecSyncLegacy::runOnModule(Module &M) {
  return runLowerExecSyncGlobals(M);
}

```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `AMDGPULowerExecSyncLegacy`, `AMDGPULowerExecSyncLegacy::runOnModule`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`AMDGPULowerExecSyncLegacy`, `AMDGPULowerExecSyncLegacy::runOnModule`。

### Lines 276-284: Implements llvm::createAMDGPULowerExecSyncLegacyPass
```cpp
ModulePass *llvm::createAMDGPULowerExecSyncLegacyPass() {
  return new AMDGPULowerExecSyncLegacy();
}

PreservedAnalyses AMDGPULowerExecSyncPass::run(Module &M,
                                               ModuleAnalysisManager &AM) {
  return runLowerExecSyncGlobals(M) ? PreservedAnalyses::none()
                                    : PreservedAnalyses::all();
}
```
**EN:** This section contains concrete logic for llvm::createAMDGPULowerExecSyncLegacyPass. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `llvm::createAMDGPULowerExecSyncLegacyPass`, `AMDGPULowerExecSyncPass::run`, `PreservedAnalyses::none`.
**CN:** 本节包含与 llvm::createAMDGPULowerExecSyncLegacyPass 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`llvm::createAMDGPULowerExecSyncLegacyPass`, `AMDGPULowerExecSyncPass::run`, `PreservedAnalyses::none`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `AMDGPULowerExecSyncLegacy`, `ConstantAsMetadata::get`, `ConstantInt::get`, `MDNode::get`, `std::move`, `ValueAsMetadata::get`
- **Main themes / 核心主题**: instruction semantics / 指令语义; lowering / 降低; assembly handling / 汇编处理; LLVM pass integration / LLVM Pass 集成
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPU.h"`
- `"AMDGPUMemoryUtils.h"`
- `"AMDGPUTargetMachine.h"`
- `"llvm/ADT/DenseMap.h"`
- `"llvm/Analysis/CallGraph.h"`
- `"llvm/CodeGen/TargetPassConfig.h"`
- `"llvm/IR/Constants.h"`
- `"llvm/IR/Instructions.h"`
- `"llvm/IR/ReplaceConstant.h"`
- `"llvm/InitializePasses.h"`
- `"llvm/Pass.h"`
- `"llvm/Transforms/Utils/ModuleUtils.h"`
- `<algorithm>`
