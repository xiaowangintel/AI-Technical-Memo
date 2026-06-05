# AMDGPUMCResourceInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUMCResourceInfo.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUMCResourceInfo for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPUMCResourceInfo 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-27: File banner, includes, and setup
```cpp
//===- AMDGPUMCResourceInfo.cpp --- MC Resource Info ----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// \brief MC infrastructure to propagate the function level resource usage
/// info.
///
//===----------------------------------------------------------------------===//

#include "AMDGPUMCResourceInfo.h"
#include "AMDGPUTargetMachine.h"
#include "Utils/AMDGPUBaseInfo.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCSymbol.h"
#include "llvm/Target/TargetMachine.h"

#define DEBUG_TYPE "amdgpu-mc-resource-usage"

using namespace llvm;

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 28-59: Implements MCResourceInfo::getSymbol
```cpp
MCSymbol *MCResourceInfo::getSymbol(StringRef FuncName, ResourceInfoKind RIK,
                                    MCContext &OutContext) {
  auto GOCS = [FuncName, &OutContext](StringRef Suffix) {
    StringRef Prefix = OutContext.getAsmInfo().getInternalSymbolPrefix();
    return OutContext.getOrCreateSymbol(Twine(Prefix) + FuncName +
                                        Twine(Suffix));
  };
  switch (RIK) {
  case RIK_NumVGPR:
    return GOCS(".num_vgpr");
  case RIK_NumAGPR:
    return GOCS(".num_agpr");
  case RIK_NumSGPR:
    return GOCS(".numbered_sgpr");
  case RIK_NumNamedBarrier:
    return GOCS(".num_named_barrier");
  case RIK_PrivateSegSize:
    return GOCS(".private_seg_size");
  case RIK_UsesVCC:
    return GOCS(".uses_vcc");
  case RIK_UsesFlatScratch:
    return GOCS(".uses_flat_scratch");
  case RIK_HasDynSizedStack:
    return GOCS(".has_dyn_sized_stack");
  case RIK_HasRecursion:
    return GOCS(".has_recursion");
  case RIK_HasIndirectCall:
    return GOCS(".has_indirect_call");
  }
  llvm_unreachable("Unexpected ResourceInfoKind.");
}

```
**EN:** This section contains concrete logic for MCResourceInfo::getSymbol. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `MCResourceInfo::getSymbol`.
**CN:** 本节包含与 MCResourceInfo::getSymbol 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`MCResourceInfo::getSymbol`。

### Lines 60-91: Implements MCResourceInfo::getSymRefExpr
```cpp
const MCExpr *MCResourceInfo::getSymRefExpr(StringRef FuncName,
                                            ResourceInfoKind RIK,
                                            MCContext &Ctx) {
  return MCSymbolRefExpr::create(getSymbol(FuncName, RIK, Ctx), Ctx);
}

void MCResourceInfo::assignMaxRegs(MCContext &OutContext) {
  // Assign expression to get the max register use to the max_num_Xgpr symbol.
  MCSymbol *MaxVGPRSym = getMaxVGPRSymbol(OutContext);
  MCSymbol *MaxAGPRSym = getMaxAGPRSymbol(OutContext);
  MCSymbol *MaxSGPRSym = getMaxSGPRSymbol(OutContext);
  MCSymbol *MaxNamedBarrierSym = getMaxNamedBarrierSymbol(OutContext);

  auto assignMaxRegSym = [&OutContext](MCSymbol *Sym, int32_t RegCount) {
    const MCExpr *MaxExpr = MCConstantExpr::create(RegCount, OutContext);
    Sym->setVariableValue(MaxExpr);
  };

  assignMaxRegSym(MaxVGPRSym, MaxVGPR);
  assignMaxRegSym(MaxAGPRSym, MaxAGPR);
  assignMaxRegSym(MaxSGPRSym, MaxSGPR);
  assignMaxRegSym(MaxNamedBarrierSym, MaxNamedBarrier);
}

void MCResourceInfo::reset() { *this = MCResourceInfo(); }

void MCResourceInfo::finalize(MCContext &OutContext) {
  assert(!Finalized && "Cannot finalize ResourceInfo again.");
  Finalized = true;
  assignMaxRegs(OutContext);
}

```
**EN:** This section contains concrete logic for MCResourceInfo::getSymRefExpr. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `MCResourceInfo::getSymRefExpr`, `MCSymbolRefExpr::create`, `MCResourceInfo::assignMaxRegs`.
**CN:** 本节包含与 MCResourceInfo::getSymRefExpr 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`MCResourceInfo::getSymRefExpr`, `MCSymbolRefExpr::create`, `MCResourceInfo::assignMaxRegs`。

### Lines 92-124: Implements MCResourceInfo::getMaxVGPRSymbol
```cpp
MCSymbol *MCResourceInfo::getMaxVGPRSymbol(MCContext &OutContext) {
  return OutContext.getOrCreateSymbol("amdgpu.max_num_vgpr");
}

MCSymbol *MCResourceInfo::getMaxAGPRSymbol(MCContext &OutContext) {
  return OutContext.getOrCreateSymbol("amdgpu.max_num_agpr");
}

MCSymbol *MCResourceInfo::getMaxSGPRSymbol(MCContext &OutContext) {
  return OutContext.getOrCreateSymbol("amdgpu.max_num_sgpr");
}

MCSymbol *MCResourceInfo::getMaxNamedBarrierSymbol(MCContext &OutContext) {
  return OutContext.getOrCreateSymbol("amdgpu.max_num_named_barrier");
}

// Tries to flatten recursive call register resource gathering. Simple cycle
// avoiding dfs to find the constants in the propagated symbols.
// Assumes:
// - RecSym has been confirmed to recurse (this means the callee symbols should
//   all be populated, started at RecSym).
// - Shape of the resource symbol's MCExpr (`max` args are order agnostic):
//   RecSym.MCExpr := max(<constant>+, <callee_symbol>*)
const MCExpr *MCResourceInfo::flattenedCycleMax(MCSymbol *RecSym,
                                                ResourceInfoKind RIK,
                                                MCContext &OutContext) {
  SmallPtrSet<const MCExpr *, 8> Seen;
  SmallVector<const MCExpr *, 8> WorkList;
  int64_t Maximum = 0;

  const MCExpr *RecExpr = RecSym->getVariableValue();
  WorkList.push_back(RecExpr);

```
**EN:** This section contains concrete logic for MCResourceInfo::getMaxVGPRSymbol. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `MCResourceInfo::getMaxVGPRSymbol`, `MCResourceInfo::getMaxAGPRSymbol`, `MCResourceInfo::getMaxSGPRSymbol`.
**CN:** 本节包含与 MCResourceInfo::getMaxVGPRSymbol 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`MCResourceInfo::getMaxVGPRSymbol`, `MCResourceInfo::getMaxAGPRSymbol`, `MCResourceInfo::getMaxSGPRSymbol`。

### Lines 125-158: Type declarations and aliases
```cpp
  while (!WorkList.empty()) {
    const MCExpr *CurExpr = WorkList.pop_back_val();
    switch (CurExpr->getKind()) {
    default: {
      // Assuming the recursion is of shape `max(<constant>, <callee_symbol>)`
      // where <callee_symbol> will eventually recurse. If this condition holds,
      // the recursion occurs within some other (possibly unresolvable) MCExpr,
      // thus using the worst case value then.
      if (!AMDGPUMCExpr::isSymbolUsedInExpression(RecSym, CurExpr)) {
        LLVM_DEBUG(dbgs() << "MCResUse:   " << RecSym->getName()
                          << ": Recursion in unexpected sub-expression, using "
                             "module maximum\n");
        switch (RIK) {
        default:
          break;
        case RIK_NumVGPR:
          return MCSymbolRefExpr::create(getMaxVGPRSymbol(OutContext),
                                         OutContext);
          break;
        case RIK_NumSGPR:
          return MCSymbolRefExpr::create(getMaxSGPRSymbol(OutContext),
                                         OutContext);
          break;
        case RIK_NumAGPR:
          return MCSymbolRefExpr::create(getMaxAGPRSymbol(OutContext),
                                         OutContext);
          break;
        }
      }
      break;
    }
    case MCExpr::ExprKind::Constant: {
      int64_t Val = cast<MCConstantExpr>(CurExpr)->getValue();
      Maximum = std::max(Maximum, Val);
```
**EN:** This section introduces supporting types or aliases that simplify later declarations and backend logic. Main symbols: `AMDGPUMCExpr::isSymbolUsedInExpression`, `MCSymbolRefExpr::create`, `std::max`.
**CN:** 本节引入辅助类型或别名，以简化后续声明与后端逻辑。 主要符号：`AMDGPUMCExpr::isSymbolUsedInExpression`, `MCSymbolRefExpr::create`, `std::max`。

### Lines 159-187: Conditional logic and checks
```cpp
      break;
    }
    case MCExpr::ExprKind::SymbolRef: {
      const MCSymbolRefExpr *SymExpr = cast<MCSymbolRefExpr>(CurExpr);
      const MCSymbol &SymRef = SymExpr->getSymbol();
      if (SymRef.isVariable()) {
        const MCExpr *SymVal = SymRef.getVariableValue();
        if (Seen.insert(SymVal).second)
          WorkList.push_back(SymVal);
      }
      break;
    }
    case MCExpr::ExprKind::Target: {
      const AMDGPUMCExpr *TargetExpr = cast<AMDGPUMCExpr>(CurExpr);
      if (TargetExpr->getKind() == AMDGPUMCExpr::VariantKind::AGVK_Max) {
        for (auto &Arg : TargetExpr->getArgs())
          WorkList.push_back(Arg);
      }
      break;
    }
    }
  }

  LLVM_DEBUG(dbgs() << "MCResUse:   " << RecSym->getName()
                    << ": Using flattened max: << " << Maximum << '\n');

  return MCConstantExpr::create(Maximum, OutContext);
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `MCConstantExpr::create`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`MCConstantExpr::create`。

### Lines 188-212: Implements MCResourceInfo::assignResourceInfoExpr
```cpp
void MCResourceInfo::assignResourceInfoExpr(
    int64_t LocalValue, ResourceInfoKind RIK, AMDGPUMCExpr::VariantKind Kind,
    const MachineFunction &MF, const SmallVectorImpl<const Function *> &Callees,
    MCContext &OutContext) {
  const TargetMachine &TM = MF.getTarget();
  MCSymbol *FnSym = TM.getSymbol(&MF.getFunction());
  const MCConstantExpr *LocalConstExpr =
      MCConstantExpr::create(LocalValue, OutContext);
  const MCExpr *SymVal = LocalConstExpr;
  MCSymbol *Sym = getSymbol(FnSym->getName(), RIK, OutContext);
  LLVM_DEBUG(dbgs() << "MCResUse:   " << Sym->getName() << ": Adding "
                    << LocalValue << " as function local usage\n");
  if (!Callees.empty()) {
    SmallVector<const MCExpr *, 8> ArgExprs;
    SmallPtrSet<const Function *, 8> Seen;
    ArgExprs.push_back(LocalConstExpr);

    for (const Function *Callee : Callees) {
      if (!Seen.insert(Callee).second)
        continue;

      MCSymbol *CalleeFnSym = TM.getSymbol(&Callee->getFunction());
      MCSymbol *CalleeValSym =
          getSymbol(CalleeFnSym->getName(), RIK, OutContext);

```
**EN:** This section contains concrete logic for MCResourceInfo::assignResourceInfoExpr. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `MCResourceInfo::assignResourceInfoExpr`, `MCConstantExpr::create`.
**CN:** 本节包含与 MCResourceInfo::assignResourceInfoExpr 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`MCResourceInfo::assignResourceInfoExpr`, `MCConstantExpr::create`。

### Lines 213-246: Switch-based control flow
```cpp
      // Avoid constructing recursive definitions by detecting whether `Sym` is
      // found transitively within any of its `CalleeValSym`.
      if (!CalleeValSym->isVariable() ||
          !AMDGPUMCExpr::isSymbolUsedInExpression(
              Sym, CalleeValSym->getVariableValue())) {
        LLVM_DEBUG(dbgs() << "MCResUse:   " << Sym->getName() << ": Adding "
                          << CalleeValSym->getName() << " as callee\n");
        ArgExprs.push_back(MCSymbolRefExpr::create(CalleeValSym, OutContext));
      } else {
        LLVM_DEBUG(dbgs() << "MCResUse:   " << Sym->getName()
                          << ": Recursion found, attempt flattening of cycle "
                             "for resource usage\n");
        // In case of recursion for vgpr/sgpr/agpr resource usage: try to
        // flatten and use the max of the call cycle. May still end up emitting
        // module max if not fully resolvable.
        switch (RIK) {
        default:
          break;
        case RIK_NumVGPR:
        case RIK_NumSGPR:
        case RIK_NumAGPR:
          ArgExprs.push_back(flattenedCycleMax(CalleeValSym, RIK, OutContext));
          break;
        case RIK_NumNamedBarrier:
          ArgExprs.push_back(MCSymbolRefExpr::create(
              getMaxNamedBarrierSymbol(OutContext), OutContext));
          break;
        }
      }
    }
    if (ArgExprs.size() > 1)
      SymVal = AMDGPUMCExpr::create(Kind, ArgExprs, OutContext);
  }
  Sym->setVariableValue(SymVal);
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPUMCExpr::isSymbolUsedInExpression`, `MCSymbolRefExpr::create`, `AMDGPUMCExpr::create`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPUMCExpr::isSymbolUsedInExpression`, `MCSymbolRefExpr::create`, `AMDGPUMCExpr::create`。

### Lines 247-276: Implements MCResourceInfo::gatherResourceInfo
```cpp
}

void MCResourceInfo::gatherResourceInfo(
    const MachineFunction &MF,
    const AMDGPUResourceUsageAnalysisWrapperPass::FunctionResourceInfo &FRI,
    MCContext &OutContext) {
  // Worst case VGPR use for non-hardware-entrypoints.
  MCSymbol *MaxVGPRSym = getMaxVGPRSymbol(OutContext);
  MCSymbol *MaxAGPRSym = getMaxAGPRSymbol(OutContext);
  MCSymbol *MaxSGPRSym = getMaxSGPRSymbol(OutContext);
  MCSymbol *MaxNamedBarrierSym = getMaxNamedBarrierSymbol(OutContext);

  if (!AMDGPU::isEntryFunctionCC(MF.getFunction().getCallingConv())) {
    addMaxVGPRCandidate(FRI.NumVGPR);
    addMaxAGPRCandidate(FRI.NumAGPR);
    addMaxSGPRCandidate(FRI.NumExplicitSGPR);
    addMaxNamedBarrierCandidate(FRI.NumNamedBarrier);
  }

  const TargetMachine &TM = MF.getTarget();
  MCSymbol *FnSym = TM.getSymbol(&MF.getFunction());

  LLVM_DEBUG(dbgs() << "MCResUse: Gathering resource information for "
                    << FnSym->getName() << '\n');

  auto SetToLocal = [&](int64_t Value, ResourceInfoKind RIK) {
    MCSymbol *Sym = getSymbol(FnSym->getName(), RIK, OutContext);
    Sym->setVariableValue(MCConstantExpr::create(Value, OutContext));
  };

```
**EN:** This section contains concrete logic for MCResourceInfo::gatherResourceInfo. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `MCResourceInfo::gatherResourceInfo`, `AMDGPU::isEntryFunctionCC`, `MCConstantExpr::create`.
**CN:** 本节包含与 MCResourceInfo::gatherResourceInfo 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`MCResourceInfo::gatherResourceInfo`, `AMDGPU::isEntryFunctionCC`, `MCConstantExpr::create`。

### Lines 277-304: Conditional logic and checks
```cpp
  // When link-time object linking is enabled, set all resource symbols to
  // concrete local values.
  if (AMDGPUTargetMachine::EnableObjectLinking) {
    LLVM_DEBUG(dbgs() << "MCResUse:   object linking enabled, no call-graph "
                         "propagation; emitting local resource values only\n");
    SetToLocal(FRI.NumVGPR, RIK_NumVGPR);
    SetToLocal(FRI.NumAGPR, RIK_NumAGPR);
    SetToLocal(FRI.NumExplicitSGPR, RIK_NumSGPR);
    SetToLocal(FRI.NumNamedBarrier, RIK_NumNamedBarrier);
    SetToLocal(FRI.PrivateSegmentSize, RIK_PrivateSegSize);
    SetToLocal(FRI.UsesVCC, RIK_UsesVCC);
    SetToLocal(FRI.UsesFlatScratch, RIK_UsesFlatScratch);
    SetToLocal(FRI.HasDynamicallySizedStack, RIK_HasDynSizedStack);
    SetToLocal(FRI.HasRecursion, RIK_HasRecursion);
    SetToLocal(FRI.HasIndirectCall, RIK_HasIndirectCall);
    return;
  }

  LLVM_DEBUG({
    if (!FRI.Callees.empty()) {
      dbgs() << "MCResUse: Callees:\n";
      for (const Function *Callee : FRI.Callees) {
        MCSymbol *CalleeFnSym = TM.getSymbol(&Callee->getFunction());
        dbgs() << "MCResUse:   " << CalleeFnSym->getName() << '\n';
      }
    }
  });

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 305-326: Type declarations and aliases
```cpp
  auto SetMaxReg = [&](MCSymbol *MaxSym, int32_t numRegs,
                       ResourceInfoKind RIK) {
    if (!FRI.HasIndirectCall) {
      assignResourceInfoExpr(numRegs, RIK, AMDGPUMCExpr::AGVK_Max, MF,
                             FRI.Callees, OutContext);
    } else {
      const MCExpr *SymRef = MCSymbolRefExpr::create(MaxSym, OutContext);
      MCSymbol *LocalNumSym = getSymbol(FnSym->getName(), RIK, OutContext);
      const MCExpr *MaxWithLocal = AMDGPUMCExpr::createMax(
          {MCConstantExpr::create(numRegs, OutContext), SymRef}, OutContext);
      LocalNumSym->setVariableValue(MaxWithLocal);
      LLVM_DEBUG(dbgs() << "MCResUse:   " << LocalNumSym->getName()
                        << ": Indirect callee within, using module maximum\n");
    }
  };

  LLVM_DEBUG(dbgs() << "MCResUse: " << FnSym->getName() << '\n');
  SetMaxReg(MaxVGPRSym, FRI.NumVGPR, RIK_NumVGPR);
  SetMaxReg(MaxAGPRSym, FRI.NumAGPR, RIK_NumAGPR);
  SetMaxReg(MaxSGPRSym, FRI.NumExplicitSGPR, RIK_NumSGPR);
  SetMaxReg(MaxNamedBarrierSym, FRI.NumNamedBarrier, RIK_NumNamedBarrier);

```
**EN:** This section introduces supporting types or aliases that simplify later declarations and backend logic. Main symbols: `MCSymbolRefExpr::create`, `AMDGPUMCExpr::createMax`, `MCConstantExpr::create`.
**CN:** 本节引入辅助类型或别名，以简化后续声明与后端逻辑。 主要符号：`MCSymbolRefExpr::create`, `AMDGPUMCExpr::createMax`, `MCConstantExpr::create`。

### Lines 327-349: Conditional logic and checks
```cpp
  {
    // The expression for private segment size should be: FRI.PrivateSegmentSize
    // + max(FRI.Callees, FRI.CalleeSegmentSize)
    SmallVector<const MCExpr *, 8> ArgExprs;
    MCSymbol *Sym = getSymbol(FnSym->getName(), RIK_PrivateSegSize, OutContext);
    if (FRI.CalleeSegmentSize) {
      LLVM_DEBUG(dbgs() << "MCResUse:   " << Sym->getName() << ": Adding "
                        << FRI.CalleeSegmentSize
                        << " for indirect/recursive callees within\n");
      ArgExprs.push_back(
          MCConstantExpr::create(FRI.CalleeSegmentSize, OutContext));
    }

    SmallPtrSet<const Function *, 8> Seen;
    Seen.insert(&MF.getFunction());
    for (const Function *Callee : FRI.Callees) {
      if (!Seen.insert(Callee).second)
        continue;
      if (!Callee->isDeclaration()) {
        MCSymbol *CalleeFnSym = TM.getSymbol(&Callee->getFunction());
        MCSymbol *CalleeValSym =
            getSymbol(CalleeFnSym->getName(), RIK_PrivateSegSize, OutContext);

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `MCConstantExpr::create`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`MCConstantExpr::create`。

### Lines 350-374: Conditional logic and checks
```cpp
        // Avoid constructing recursive definitions by detecting whether `Sym`
        // is found transitively within any of its `CalleeValSym`.
        if (!CalleeValSym->isVariable() ||
            !AMDGPUMCExpr::isSymbolUsedInExpression(
                Sym, CalleeValSym->getVariableValue())) {
          LLVM_DEBUG(dbgs() << "MCResUse:   " << Sym->getName() << ": Adding "
                            << CalleeValSym->getName() << " as callee\n");
          ArgExprs.push_back(MCSymbolRefExpr::create(CalleeValSym, OutContext));
        }
      }
    }
    const MCExpr *localConstExpr =
        MCConstantExpr::create(FRI.PrivateSegmentSize, OutContext);
    LLVM_DEBUG(dbgs() << "MCResUse:   " << Sym->getName() << ": Adding "
                      << FRI.PrivateSegmentSize
                      << " as function local usage\n");
    if (!ArgExprs.empty()) {
      const AMDGPUMCExpr *transitiveExpr =
          AMDGPUMCExpr::createMax(ArgExprs, OutContext);
      localConstExpr =
          MCBinaryExpr::createAdd(localConstExpr, transitiveExpr, OutContext);
    }
    Sym->setVariableValue(localConstExpr);
  }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPUMCExpr::isSymbolUsedInExpression`, `MCSymbolRefExpr::create`, `MCConstantExpr::create`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPUMCExpr::isSymbolUsedInExpression`, `MCSymbolRefExpr::create`, `MCConstantExpr::create`。

### Lines 375-407: Conditional logic and checks
```cpp
  if (!FRI.HasIndirectCall) {
    assignResourceInfoExpr(FRI.UsesVCC, ResourceInfoKind::RIK_UsesVCC,
                           AMDGPUMCExpr::AGVK_Or, MF, FRI.Callees, OutContext);
    assignResourceInfoExpr(FRI.UsesFlatScratch,
                           ResourceInfoKind::RIK_UsesFlatScratch,
                           AMDGPUMCExpr::AGVK_Or, MF, FRI.Callees, OutContext);
    assignResourceInfoExpr(FRI.HasDynamicallySizedStack,
                           ResourceInfoKind::RIK_HasDynSizedStack,
                           AMDGPUMCExpr::AGVK_Or, MF, FRI.Callees, OutContext);
    assignResourceInfoExpr(FRI.HasRecursion, ResourceInfoKind::RIK_HasRecursion,
                           AMDGPUMCExpr::AGVK_Or, MF, FRI.Callees, OutContext);
    assignResourceInfoExpr(FRI.HasIndirectCall,
                           ResourceInfoKind::RIK_HasIndirectCall,
                           AMDGPUMCExpr::AGVK_Or, MF, FRI.Callees, OutContext);
  } else {
    SetToLocal(FRI.UsesVCC, ResourceInfoKind::RIK_UsesVCC);
    SetToLocal(FRI.UsesFlatScratch, ResourceInfoKind::RIK_UsesFlatScratch);
    SetToLocal(FRI.HasDynamicallySizedStack,
               ResourceInfoKind::RIK_HasDynSizedStack);
    SetToLocal(FRI.HasRecursion, ResourceInfoKind::RIK_HasRecursion);
    SetToLocal(FRI.HasIndirectCall, ResourceInfoKind::RIK_HasIndirectCall);
  }
}

const MCExpr *MCResourceInfo::createTotalNumVGPRs(const MachineFunction &MF,
                                                  MCContext &Ctx) {
  const TargetMachine &TM = MF.getTarget();
  MCSymbol *FnSym = TM.getSymbol(&MF.getFunction());
  return AMDGPUMCExpr::createTotalNumVGPR(
      getSymRefExpr(FnSym->getName(), RIK_NumAGPR, Ctx),
      getSymRefExpr(FnSym->getName(), RIK_NumVGPR, Ctx), Ctx);
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `MCResourceInfo::createTotalNumVGPRs`, `AMDGPUMCExpr::createTotalNumVGPR`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`MCResourceInfo::createTotalNumVGPRs`, `AMDGPUMCExpr::createTotalNumVGPR`。

### Lines 408-420: Implements MCResourceInfo::createTotalNumSGPRs
```cpp
const MCExpr *MCResourceInfo::createTotalNumSGPRs(const MachineFunction &MF,
                                                  bool hasXnack,
                                                  MCContext &Ctx) {
  const TargetMachine &TM = MF.getTarget();
  MCSymbol *FnSym = TM.getSymbol(&MF.getFunction());
  return MCBinaryExpr::createAdd(
      getSymRefExpr(FnSym->getName(), RIK_NumSGPR, Ctx),
      AMDGPUMCExpr::createExtraSGPRs(
          getSymRefExpr(FnSym->getName(), RIK_UsesVCC, Ctx),
          getSymRefExpr(FnSym->getName(), RIK_UsesFlatScratch, Ctx), hasXnack,
          Ctx),
      Ctx);
}
```
**EN:** This section contains concrete logic for MCResourceInfo::createTotalNumSGPRs. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `MCResourceInfo::createTotalNumSGPRs`, `MCBinaryExpr::createAdd`, `AMDGPUMCExpr::createExtraSGPRs`.
**CN:** 本节包含与 MCResourceInfo::createTotalNumSGPRs 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`MCResourceInfo::createTotalNumSGPRs`, `MCBinaryExpr::createAdd`, `AMDGPUMCExpr::createExtraSGPRs`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `MCResourceInfo::getSymbol`, `MCResourceInfo::getSymRefExpr`, `MCSymbolRefExpr::create`, `MCResourceInfo::assignMaxRegs`, `MCConstantExpr::create`, `MCResourceInfo::reset`
- **Main themes / 核心主题**: register management / 寄存器管理; assembly handling / 汇编处理; hardware resource modeling / 硬件资源建模
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPUMCResourceInfo.h"`
- `"AMDGPUTargetMachine.h"`
- `"Utils/AMDGPUBaseInfo.h"`
- `"llvm/ADT/StringRef.h"`
- `"llvm/MC/MCAsmInfo.h"`
- `"llvm/MC/MCContext.h"`
- `"llvm/MC/MCSymbol.h"`
- `"llvm/Target/TargetMachine.h"`
