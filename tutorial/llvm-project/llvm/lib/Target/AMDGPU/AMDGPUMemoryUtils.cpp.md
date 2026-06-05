# AMDGPUMemoryUtils.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUMemoryUtils.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUMemoryUtils for the LLVM backend utilities. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM 后端工具中的 AMDGPUMemoryUtils 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-32: File banner, includes, and setup
```cpp
//===-- AMDGPUMemoryUtils.cpp - -------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "AMDGPUMemoryUtils.h"
#include "AMDGPU.h"
#include "Utils/AMDGPUBaseInfo.h"
#include "llvm/ADT/SetOperations.h"
#include "llvm/Analysis/AliasAnalysis.h"
#include "llvm/Analysis/CallGraph.h"
#include "llvm/Analysis/MemorySSA.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/IntrinsicInst.h"
#include "llvm/IR/IntrinsicsAMDGPU.h"
#include "llvm/IR/ReplaceConstant.h"

#define DEBUG_TYPE "amdgpu-memory-utils"

using namespace llvm;

namespace llvm::AMDGPU {

Align getAlign(const DataLayout &DL, const GlobalVariable *GV) {
  return DL.getValueOrABITypeAlignment(GV->getPointerAlignment(DL),
                                       GV->getValueType());
}

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 33-66: Defines getTargetExtType
```cpp
// Returns the target extension type of a global variable,
// which can only be a TargetExtType, an array or single-element struct of it,
// or their nesting combination.
// TODO: allow struct of multiple TargetExtType elements of the same type.
// TODO: Disallow other uses of target("amdgcn.named.barrier") including:
// - Structs containing barriers in different scope/rank
// - Structs containing a mixture of barriers and other data.
// - Globals in other address spaces.
// - Allocas.
static TargetExtType *getTargetExtType(const GlobalVariable &GV) {
  Type *Ty = GV.getValueType();
  while (true) {
    if (auto *TTy = dyn_cast<TargetExtType>(Ty))
      return TTy;
    if (auto *STy = dyn_cast<StructType>(Ty)) {
      if (STy->getNumElements() != 1)
        return nullptr;
      Ty = STy->getElementType(0);
      continue;
    }
    if (auto *ATy = dyn_cast<ArrayType>(Ty)) {
      Ty = ATy->getElementType();
      continue;
    }
    return nullptr;
  }
}

TargetExtType *isNamedBarrier(const GlobalVariable &GV) {
  if (TargetExtType *Ty = getTargetExtType(GV))
    return Ty->getName() == "amdgcn.named.barrier" ? Ty : nullptr;
  return nullptr;
}

```
**EN:** This section contains concrete logic for getTargetExtType. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 getTargetExtType 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 67-96: Defines isDynamicLDS
```cpp
bool isDynamicLDS(const GlobalVariable &GV) {
  // external zero size addrspace(3) without initializer is dynlds.
  const Module *M = GV.getParent();
  const DataLayout &DL = M->getDataLayout();
  if (GV.getType()->getPointerAddressSpace() != AMDGPUAS::LOCAL_ADDRESS)
    return false;
  return GV.getGlobalSize(DL) == 0;
}

bool isLDSVariableToLower(const GlobalVariable &GV) {
  if (GV.getType()->getPointerAddressSpace() != AMDGPUAS::LOCAL_ADDRESS) {
    return false;
  }
  if (isDynamicLDS(GV)) {
    return true;
  }
  if (GV.isConstant()) {
    // A constant undef variable can't be written to, and any load is
    // undef, so it should be eliminated by the optimizer. It could be
    // dropped by the back end if not. This pass skips over it.
    return false;
  }
  if (GV.hasInitializer() && !isa<UndefValue>(GV.getInitializer())) {
    // Initializers are unimplemented for LDS address space.
    // Leave such variables in place for consistent error reporting.
    return false;
  }
  return true;
}

```
**EN:** This section contains concrete logic for isDynamicLDS. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 isDynamicLDS 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 97-117: Defines eliminateConstantExprUsesOfLDSFromAllInstructions
```cpp
bool eliminateConstantExprUsesOfLDSFromAllInstructions(Module &M) {
  // Constants are uniqued within LLVM. A ConstantExpr referring to a LDS
  // global may have uses from multiple different functions as a result.
  // This pass specialises LDS variables with respect to the kernel that
  // allocates them.

  // This is semantically equivalent to (the unimplemented as slow):
  // for (auto &F : M.functions())
  //   for (auto &BB : F)
  //     for (auto &I : BB)
  //       for (Use &Op : I.operands())
  //         if (constantExprUsesLDS(Op))
  //           replaceConstantExprInFunction(I, Op);

  SmallVector<Constant *> LDSGlobals;
  for (auto &GV : M.globals())
    if (AMDGPU::isLDSVariableToLower(GV))
      LDSGlobals.push_back(&GV);
  return convertUsersOfConstantsToInstructions(LDSGlobals);
}

```
**EN:** This section contains concrete logic for eliminateConstantExprUsesOfLDSFromAllInstructions. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPU::isLDSVariableToLower`.
**CN:** 本节包含与 eliminateConstantExprUsesOfLDSFromAllInstructions 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPU::isLDSVariableToLower`。

### Lines 118-143: Defines getUsesOfLDSByFunction
```cpp
void getUsesOfLDSByFunction(const CallGraph &CG, Module &M,
                            FunctionVariableMap &kernels,
                            FunctionVariableMap &Functions) {
  // Get uses from the current function, excluding uses by called Functions
  // Two output variables to avoid walking the globals list twice
  for (auto &GV : M.globals()) {
    if (!AMDGPU::isLDSVariableToLower(GV))
      continue;
    for (User *V : GV.users()) {
      if (auto *I = dyn_cast<Instruction>(V)) {
        Function *F = I->getFunction();
        if (isKernel(*F))
          kernels[F].insert(&GV);
        else
          Functions[F].insert(&GV);
      }
    }
  }
}

LDSUsesInfoTy getTransitiveUsesOfLDS(const CallGraph &CG, Module &M) {

  FunctionVariableMap DirectMapKernel;
  FunctionVariableMap DirectMapFunction;
  getUsesOfLDSByFunction(CG, M, DirectMapKernel, DirectMapFunction);

```
**EN:** This section contains concrete logic for getUsesOfLDSByFunction. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPU::isLDSVariableToLower`.
**CN:** 本节包含与 getUsesOfLDSByFunction 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPU::isLDSVariableToLower`。

### Lines 144-174: Conditional logic and checks
```cpp
  // Collect functions whose address has escaped
  DenseSet<Function *> AddressTakenFuncs;
  for (Function &F : M.functions()) {
    if (!isKernel(F))
      if (F.hasAddressTaken(nullptr,
                            /* IgnoreCallbackUses */ false,
                            /* IgnoreAssumeLikeCalls */ false,
                            /* IgnoreLLVMUsed */ true,
                            /* IgnoreArcAttachedCall */ false)) {
        AddressTakenFuncs.insert(&F);
      }
  }

  // Collect variables that are used by functions whose address has escaped
  DenseSet<GlobalVariable *> VariablesReachableThroughFunctionPointer;
  for (Function *F : AddressTakenFuncs) {
    set_union(VariablesReachableThroughFunctionPointer, DirectMapFunction[F]);
  }

  auto FunctionMakesUnknownCall = [&](const Function *F) -> bool {
    assert(!F->isDeclaration());
    for (const CallGraphNode::CallRecord &R : *CG[F]) {
      if (!R.second->getFunction())
        return true;
    }
    return false;
  };

  // Work out which variables are reachable through function calls
  FunctionVariableMap TransitiveMapFunction = DirectMapFunction;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 175-201: Conditional logic and checks
```cpp
  // If the function makes any unknown call, assume the worst case that it can
  // access all variables accessed by functions whose address escaped
  for (Function &F : M.functions()) {
    if (!F.isDeclaration() && FunctionMakesUnknownCall(&F)) {
      if (!isKernel(F)) {
        set_union(TransitiveMapFunction[&F],
                  VariablesReachableThroughFunctionPointer);
      }
    }
  }

  // Direct implementation of collecting all variables reachable from each
  // function
  for (Function &Func : M.functions()) {
    if (Func.isDeclaration() || isKernel(Func))
      continue;

    DenseSet<Function *> seen; // catches cycles
    SmallVector<Function *, 4> wip = {&Func};

    while (!wip.empty()) {
      Function *F = wip.pop_back_val();

      // Can accelerate this by referring to transitive map for functions that
      // have already been computed, with more care than this
      set_union(TransitiveMapFunction[&Func], DirectMapFunction[F]);

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 202-235: Conditional logic and checks
```cpp
      for (const CallGraphNode::CallRecord &R : *CG[F]) {
        Function *Ith = R.second->getFunction();
        if (Ith) {
          if (!seen.contains(Ith)) {
            seen.insert(Ith);
            wip.push_back(Ith);
          }
        }
      }
    }
  }

  // Collect variables that are transitively used by functions whose address has
  // escaped
  for (Function *F : AddressTakenFuncs) {
    set_union(VariablesReachableThroughFunctionPointer,
              TransitiveMapFunction[F]);
  }

  // DirectMapKernel lists which variables are used by the kernel
  // find the variables which are used through a function call
  FunctionVariableMap IndirectMapKernel;

  for (Function &Func : M.functions()) {
    if (Func.isDeclaration() || !isKernel(Func))
      continue;

    for (const CallGraphNode::CallRecord &R : *CG[&Func]) {
      Function *Ith = R.second->getFunction();
      if (Ith) {
        set_union(IndirectMapKernel[&Func], TransitiveMapFunction[Ith]);
      }
    }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 236-265: Conditional logic and checks
```cpp
    // Check if the kernel encounters unknows calls, wheher directly or
    // indirectly.
    bool SeesUnknownCalls = [&]() {
      SmallVector<Function *> WorkList = {CG[&Func]->getFunction()};
      SmallPtrSet<Function *, 8> Visited;

      while (!WorkList.empty()) {
        Function *F = WorkList.pop_back_val();

        for (const CallGraphNode::CallRecord &CallRecord : *CG[F]) {
          if (!CallRecord.second)
            continue;

          Function *Callee = CallRecord.second->getFunction();
          if (!Callee)
            return true;

          if (Visited.insert(Callee).second)
            WorkList.push_back(Callee);
        }
      }
      return false;
    }();

    if (SeesUnknownCalls) {
      set_union(IndirectMapKernel[&Func],
                VariablesReachableThroughFunctionPointer);
    }
  }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 266-299: Conditional logic and checks
```cpp
  // Verify that we fall into one of 2 cases:
  //    - All variables are either absolute
  //      or direct mapped dynamic LDS that is not lowered.
  //      this is a re-run of the pass
  //      so we don't have anything to do.
  //    - No variables are absolute.
  // Named-barriers which are absolute symbols are removed
  // from the maps.
  std::optional<bool> HasAbsoluteGVs;
  bool HasSpecialGVs = false;
  for (auto &Map : {DirectMapKernel, IndirectMapKernel}) {
    for (auto &[Fn, GVs] : Map) {
      for (auto *GV : GVs) {
        bool IsAbsolute = GV->isAbsoluteSymbolRef();
        bool IsDirectMapDynLDSGV =
            AMDGPU::isDynamicLDS(*GV) && DirectMapKernel.contains(Fn);
        if (IsDirectMapDynLDSGV)
          continue;
        if (isNamedBarrier(*GV)) {
          if (IsAbsolute) {
            DirectMapKernel[Fn].erase(GV);
            IndirectMapKernel[Fn].erase(GV);
          }
          HasSpecialGVs = true;
          continue;
        }
        if (HasAbsoluteGVs.has_value()) {
          if (*HasAbsoluteGVs != IsAbsolute) {
            reportFatalUsageError(
                "module cannot mix absolute and non-absolute LDS GVs");
          }
        } else
          HasAbsoluteGVs = IsAbsolute;
      }
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::isDynamicLDS`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::isDynamicLDS`。

### Lines 300-332: Conditional logic and checks
```cpp
    }
  }

  // If we only had absolute GVs, we have nothing to do, return an empty
  // result.
  if (HasAbsoluteGVs && *HasAbsoluteGVs)
    return {FunctionVariableMap(), FunctionVariableMap(), false};

  return {std::move(DirectMapKernel), std::move(IndirectMapKernel),
          HasSpecialGVs};
}

void removeFnAttrFromReachable(CallGraph &CG, Function *KernelRoot,
                               ArrayRef<StringRef> FnAttrs) {
  for (StringRef Attr : FnAttrs)
    KernelRoot->removeFnAttr(Attr);

  SmallVector<Function *> WorkList = {CG[KernelRoot]->getFunction()};
  SmallPtrSet<Function *, 8> Visited;
  bool SeenUnknownCall = false;

  while (!WorkList.empty()) {
    Function *F = WorkList.pop_back_val();

    for (auto &CallRecord : *CG[F]) {
      if (!CallRecord.second)
        continue;

      Function *Callee = CallRecord.second->getFunction();
      if (!Callee) {
        if (!SeenUnknownCall) {
          SeenUnknownCall = true;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::move`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::move`。

### Lines 333-361: Conditional logic and checks
```cpp
          // If we see any indirect calls, assume nothing about potential
          // targets.
          // TODO: This could be refined to possible LDS global users.
          for (auto &ExternalCallRecord : *CG.getExternalCallingNode()) {
            Function *PotentialCallee =
                ExternalCallRecord.second->getFunction();
            assert(PotentialCallee);
            if (!isKernel(*PotentialCallee)) {
              for (StringRef Attr : FnAttrs)
                PotentialCallee->removeFnAttr(Attr);
            }
          }
        }
      } else {
        for (StringRef Attr : FnAttrs)
          Callee->removeFnAttr(Attr);
        if (Visited.insert(Callee).second)
          WorkList.push_back(Callee);
      }
    }
  }
}

bool isReallyAClobber(const Value *Ptr, MemoryDef *Def, AAResults *AA) {
  Instruction *DefInst = Def->getMemoryInst();

  if (isa<FenceInst>(DefInst))
    return false;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 362-394: Switch-based control flow
```cpp
  if (const IntrinsicInst *II = dyn_cast<IntrinsicInst>(DefInst)) {
    switch (II->getIntrinsicID()) {
    case Intrinsic::amdgcn_s_barrier:
    case Intrinsic::amdgcn_s_cluster_barrier:
    case Intrinsic::amdgcn_s_barrier_signal:
    case Intrinsic::amdgcn_s_barrier_signal_var:
    case Intrinsic::amdgcn_s_barrier_signal_isfirst:
    case Intrinsic::amdgcn_s_barrier_init:
    case Intrinsic::amdgcn_s_barrier_join:
    case Intrinsic::amdgcn_s_barrier_wait:
    case Intrinsic::amdgcn_s_barrier_leave:
    case Intrinsic::amdgcn_s_get_barrier_state:
    case Intrinsic::amdgcn_s_wakeup_barrier:
    case Intrinsic::amdgcn_wave_barrier:
    case Intrinsic::amdgcn_sched_barrier:
    case Intrinsic::amdgcn_sched_group_barrier:
    case Intrinsic::amdgcn_iglp_opt:
      return false;
    default:
      break;
    }
  }

  // Ignore atomics not aliasing with the original load, any atomic is a
  // universal MemoryDef from MSSA's point of view too, just like a fence.
  const auto checkNoAlias = [AA, Ptr](auto I) -> bool {
    return I && AA->isNoAlias(I->getPointerOperand(), Ptr);
  };

  if (checkNoAlias(dyn_cast<AtomicCmpXchgInst>(DefInst)) ||
      checkNoAlias(dyn_cast<AtomicRMWInst>(DefInst)))
    return false;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 395-425: Defines isClobberedInFunction
```cpp
  return true;
}

bool isClobberedInFunction(const LoadInst *Load, MemorySSA *MSSA,
                           AAResults *AA) {
  MemorySSAWalker *Walker = MSSA->getWalker();
  SmallVector<MemoryAccess *> WorkList{Walker->getClobberingMemoryAccess(Load)};
  SmallPtrSet<MemoryAccess *, 8> Visited;
  MemoryLocation Loc(MemoryLocation::get(Load));

  LLVM_DEBUG(dbgs() << "Checking clobbering of: " << *Load << '\n');

  // Start with a nearest dominating clobbering access, it will be either
  // live on entry (nothing to do, load is not clobbered), MemoryDef, or
  // MemoryPhi if several MemoryDefs can define this memory state. In that
  // case add all Defs to WorkList and continue going up and checking all
  // the definitions of this memory location until the root. When all the
  // defs are exhausted and came to the entry state we have no clobber.
  // Along the scan ignore barriers and fences which are considered clobbers
  // by the MemorySSA, but not really writing anything into the memory.
  while (!WorkList.empty()) {
    MemoryAccess *MA = WorkList.pop_back_val();
    if (!Visited.insert(MA).second)
      continue;

    if (MSSA->isLiveOnEntryDef(MA))
      continue;

    if (MemoryDef *Def = dyn_cast<MemoryDef>(MA)) {
      LLVM_DEBUG(dbgs() << "  Def: " << *Def->getMemoryInst() << '\n');

```
**EN:** This section contains concrete logic for isClobberedInFunction. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `MemoryLocation::get`.
**CN:** 本节包含与 isClobberedInFunction 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`MemoryLocation::get`。

### Lines 426-445: Namespace declarations and scope setup
```cpp
      if (isReallyAClobber(Load->getPointerOperand(), Def, AA)) {
        LLVM_DEBUG(dbgs() << "      -> load is clobbered\n");
        return true;
      }

      WorkList.push_back(
          Walker->getClobberingMemoryAccess(Def->getDefiningAccess(), Loc));
      continue;
    }

    const MemoryPhi *Phi = cast<MemoryPhi>(MA);
    for (const auto &Use : Phi->incoming_values())
      WorkList.push_back(cast<MemoryAccess>(&Use));
  }

  LLVM_DEBUG(dbgs() << "      -> no clobber\n");
  return false;
}

} // end namespace llvm::AMDGPU
```
**EN:** This section arranges declarations into namespace scope so related AMDGPU functionality stays organized and avoids symbol collisions.
**CN:** 本节将声明组织到命名空间作用域中，使相关 AMDGPU 功能保持清晰并避免符号冲突。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `AMDGPU::isLDSVariableToLower`, `AMDGPU::isDynamicLDS`, `std::move`, `MemoryLocation::get`
- **Main themes / 核心主题**: alias analysis / 别名分析; instruction semantics / 指令语义; lowering / 降低; LLVM pass integration / LLVM Pass 集成
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPUMemoryUtils.h"`
- `"AMDGPU.h"`
- `"Utils/AMDGPUBaseInfo.h"`
- `"llvm/ADT/SetOperations.h"`
- `"llvm/Analysis/AliasAnalysis.h"`
- `"llvm/Analysis/CallGraph.h"`
- `"llvm/Analysis/MemorySSA.h"`
- `"llvm/IR/DataLayout.h"`
- `"llvm/IR/Instructions.h"`
- `"llvm/IR/IntrinsicInst.h"`
- `"llvm/IR/IntrinsicsAMDGPU.h"`
- `"llvm/IR/ReplaceConstant.h"`
