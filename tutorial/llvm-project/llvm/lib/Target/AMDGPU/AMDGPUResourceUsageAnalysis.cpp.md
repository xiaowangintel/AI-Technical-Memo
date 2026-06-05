# AMDGPUResourceUsageAnalysis.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUResourceUsageAnalysis.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUResourceUsageAnalysis for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPUResourceUsageAnalysis 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-33: File banner, includes, and setup
```cpp
//===- AMDGPUResourceUsageAnalysis.h ---- analysis of resources -----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// \brief Analyzes how many registers and other resources are used by
/// functions.
///
/// The results of this analysis are used to fill the register usage, flat
/// usage, etc. into hardware registers.
///
//===----------------------------------------------------------------------===//

#include "AMDGPUResourceUsageAnalysis.h"
#include "AMDGPU.h"
#include "AMDGPUTargetMachine.h"
#include "GCNSubtarget.h"
#include "SIMachineFunctionInfo.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineModuleInfo.h"
#include "llvm/CodeGen/TargetPassConfig.h"
#include "llvm/IR/GlobalValue.h"
#include "llvm/Target/TargetMachine.h"

using namespace llvm;
using namespace llvm::AMDGPU;

#define DEBUG_TYPE "amdgpu-resource-usage"

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 34-62: Registers LLVM passes
```cpp
char llvm::AMDGPUResourceUsageAnalysisWrapperPass::ID = 0;
char &llvm::AMDGPUResourceUsageAnalysisID =
    AMDGPUResourceUsageAnalysisWrapperPass::ID;

// In code object v4 and older, we need to tell the runtime some amount ahead of
// time if we don't know the true stack size. Assume a smaller number if this is
// only due to dynamic / non-entry block allocas.
static cl::opt<uint32_t> clAssumedStackSizeForExternalCall(
    "amdgpu-assume-external-call-stack-size",
    cl::desc("Assumed stack use of any external call (in bytes)"), cl::Hidden,
    cl::init(16384));

static cl::opt<uint32_t> clAssumedStackSizeForDynamicSizeObjects(
    "amdgpu-assume-dynamic-stack-object-size",
    cl::desc("Assumed extra stack use if there are any "
             "variable sized objects (in bytes)"),
    cl::Hidden, cl::init(4096));

INITIALIZE_PASS(AMDGPUResourceUsageAnalysisWrapperPass, DEBUG_TYPE,
                "Function register usage analysis", true, true)

static const Function *getCalleeFunction(const MachineOperand &Op) {
  if (Op.isImm()) {
    assert(Op.getImm() == 0);
    return nullptr;
  }
  return cast<Function>(Op.getGlobal()->stripPointerCastsAndAliases());
}

```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `cl::desc`, `cl::init`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`cl::desc`, `cl::init`。

### Lines 63-95: Defines hasAnyNonFlatUseOfReg
```cpp
static bool hasAnyNonFlatUseOfReg(const MachineRegisterInfo &MRI,
                                  const SIInstrInfo &TII, unsigned Reg) {
  for (const MachineOperand &UseOp : MRI.reg_operands(Reg)) {
    if (!UseOp.isImplicit() || !TII.isFLAT(*UseOp.getParent()))
      return true;
  }

  return false;
}

bool AMDGPUResourceUsageAnalysisWrapperPass::runOnMachineFunction(
    MachineFunction &MF) {
  auto *TPC = getAnalysisIfAvailable<TargetPassConfig>();
  if (!TPC)
    return false;

  const TargetMachine &TM = TPC->getTM<TargetMachine>();
  const MCSubtargetInfo &STI = TM.getMCSubtargetInfo();

  // By default, for code object v5 and later, track only the minimum scratch
  // size
  uint32_t AssumedStackSizeForDynamicSizeObjects =
      clAssumedStackSizeForDynamicSizeObjects;
  uint32_t AssumedStackSizeForExternalCall = clAssumedStackSizeForExternalCall;
  if (AMDGPU::getAMDHSACodeObjectVersion(*MF.getFunction().getParent()) >=
          AMDGPU::AMDHSA_COV5 ||
      STI.getTargetTriple().getOS() == Triple::AMDPAL) {
    if (!clAssumedStackSizeForDynamicSizeObjects.getNumOccurrences())
      AssumedStackSizeForDynamicSizeObjects = 0;
    if (!clAssumedStackSizeForExternalCall.getNumOccurrences())
      AssumedStackSizeForExternalCall = 0;
  }

```
**EN:** This section contains concrete logic for hasAnyNonFlatUseOfReg. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUResourceUsageAnalysisWrapperPass::runOnMachineFunction`, `AMDGPU::getAMDHSACodeObjectVersion`.
**CN:** 本节包含与 hasAnyNonFlatUseOfReg 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUResourceUsageAnalysisWrapperPass::runOnMachineFunction`, `AMDGPU::getAMDHSACodeObjectVersion`。

### Lines 96-127: Defines AMDGPUResourceUsageAnalysisImpl
```cpp
  ResourceInfo = AMDGPUResourceUsageAnalysisImpl().analyzeResourceUsage(
      MF, AssumedStackSizeForDynamicSizeObjects,
      AssumedStackSizeForExternalCall);

  return false;
}

AnalysisKey AMDGPUResourceUsageAnalysis::Key;
AMDGPUResourceUsageAnalysis::Result
AMDGPUResourceUsageAnalysis::run(MachineFunction &MF,
                                 MachineFunctionAnalysisManager &MFAM) {
  const MCSubtargetInfo &STI = TM.getMCSubtargetInfo();

  // By default, for code object v5 and later, track only the minimum scratch
  // size
  uint32_t AssumedStackSizeForDynamicSizeObjects =
      clAssumedStackSizeForDynamicSizeObjects;
  uint32_t AssumedStackSizeForExternalCall = clAssumedStackSizeForExternalCall;
  if (AMDGPU::getAMDHSACodeObjectVersion(*MF.getFunction().getParent()) >=
          AMDGPU::AMDHSA_COV5 ||
      STI.getTargetTriple().getOS() == Triple::AMDPAL) {
    if (!clAssumedStackSizeForDynamicSizeObjects.getNumOccurrences())
      AssumedStackSizeForDynamicSizeObjects = 0;
    if (!clAssumedStackSizeForExternalCall.getNumOccurrences())
      AssumedStackSizeForExternalCall = 0;
  }

  return AMDGPUResourceUsageAnalysisImpl().analyzeResourceUsage(
      MF, AssumedStackSizeForDynamicSizeObjects,
      AssumedStackSizeForExternalCall);
}

```
**EN:** This section contains concrete logic for AMDGPUResourceUsageAnalysisImpl. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUResourceUsageAnalysis::run`, `AMDGPU::getAMDHSACodeObjectVersion`.
**CN:** 本节包含与 AMDGPUResourceUsageAnalysisImpl 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUResourceUsageAnalysis::run`, `AMDGPU::getAMDHSACodeObjectVersion`。

### Lines 128-160: Implements AMDGPUResourceUsageAnalysisImpl::analyzeResourceUsage
```cpp
AMDGPUResourceUsageAnalysisImpl::SIFunctionResourceInfo
AMDGPUResourceUsageAnalysisImpl::analyzeResourceUsage(
    const MachineFunction &MF, uint32_t AssumedStackSizeForDynamicSizeObjects,
    uint32_t AssumedStackSizeForExternalCall) const {
  SIFunctionResourceInfo Info;

  const SIMachineFunctionInfo *MFI = MF.getInfo<SIMachineFunctionInfo>();
  const GCNSubtarget &ST = MF.getSubtarget<GCNSubtarget>();
  const MachineFrameInfo &FrameInfo = MF.getFrameInfo();
  const MachineRegisterInfo &MRI = MF.getRegInfo();
  const SIInstrInfo *TII = ST.getInstrInfo();
  const SIRegisterInfo &TRI = TII->getRegisterInfo();

  Info.UsesFlatScratch = MRI.isPhysRegUsed(AMDGPU::FLAT_SCR_LO) ||
                         MRI.isPhysRegUsed(AMDGPU::FLAT_SCR_HI) ||
                         MRI.isLiveIn(MFI->getPreloadedReg(
                             AMDGPUFunctionArgInfo::FLAT_SCRATCH_INIT));

  Info.NumNamedBarrier = MFI->getNumNamedBarriers();

  // Even if FLAT_SCRATCH is implicitly used, it has no effect if flat
  // instructions aren't used to access the scratch buffer. Inline assembly may
  // need it though.
  //
  // If we only have implicit uses of flat_scr on flat instructions, it is not
  // really needed.
  if (Info.UsesFlatScratch && !MFI->getUserSGPRInfo().hasFlatScratchInit() &&
      (!hasAnyNonFlatUseOfReg(MRI, *TII, AMDGPU::FLAT_SCR) &&
       !hasAnyNonFlatUseOfReg(MRI, *TII, AMDGPU::FLAT_SCR_LO) &&
       !hasAnyNonFlatUseOfReg(MRI, *TII, AMDGPU::FLAT_SCR_HI))) {
    Info.UsesFlatScratch = false;
  }

```
**EN:** This section contains concrete logic for AMDGPUResourceUsageAnalysisImpl::analyzeResourceUsage. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUResourceUsageAnalysisImpl::analyzeResourceUsage`.
**CN:** 本节包含与 AMDGPUResourceUsageAnalysisImpl::analyzeResourceUsage 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUResourceUsageAnalysisImpl::analyzeResourceUsage`。

### Lines 161-190: Declares getStackSize
```cpp
  Info.PrivateSegmentSize = FrameInfo.getStackSize();

  // Assume a big number if there are any unknown sized objects.
  Info.HasDynamicallySizedStack = FrameInfo.hasVarSizedObjects();
  if (Info.HasDynamicallySizedStack)
    Info.PrivateSegmentSize += AssumedStackSizeForDynamicSizeObjects;

  if (MFI->isStackRealigned())
    Info.PrivateSegmentSize += FrameInfo.getMaxAlign().value();

  Info.UsesVCC =
      MRI.isPhysRegUsed(AMDGPU::VCC_LO) || MRI.isPhysRegUsed(AMDGPU::VCC_HI);
  Info.NumExplicitSGPR = TRI.getNumUsedPhysRegs(MRI, AMDGPU::SGPR_32RegClass,
                                                /*IncludeCalls=*/false);
  if (ST.hasMAIInsts())
    Info.NumAGPR = TRI.getNumUsedPhysRegs(MRI, AMDGPU::AGPR_32RegClass,
                                          /*IncludeCalls=*/false);

  // If there are no calls, MachineRegisterInfo can tell us the used register
  // count easily.
  // A tail call isn't considered a call for MachineFrameInfo's purposes.
  if (!FrameInfo.hasCalls() && !FrameInfo.hasTailCall()) {
    Info.NumVGPR = TRI.getNumUsedPhysRegs(MRI, AMDGPU::VGPR_32RegClass,
                                          /*IncludeCalls=*/false);
    return Info;
  }

  int32_t MaxVGPR = -1;
  Info.CalleeSegmentSize = 0;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 191-224: Switch-based control flow
```cpp
  for (const MachineBasicBlock &MBB : MF) {
    for (const MachineInstr &MI : MBB) {
      for (unsigned I = 0; I < MI.getNumOperands(); ++I) {
        const MachineOperand &MO = MI.getOperand(I);

        if (!MO.isReg())
          continue;

        Register Reg = MO.getReg();
        switch (Reg) {
        case AMDGPU::NoRegister:
          assert(MI.isDebugInstr() &&
                 "Instruction uses invalid noreg register");
          continue;

        case AMDGPU::XNACK_MASK:
        case AMDGPU::XNACK_MASK_LO:
        case AMDGPU::XNACK_MASK_HI:
          llvm_unreachable("xnack_mask registers should not be used");

        case AMDGPU::LDS_DIRECT:
          llvm_unreachable("lds_direct register should not be used");

        case AMDGPU::TBA:
        case AMDGPU::TBA_LO:
        case AMDGPU::TBA_HI:
        case AMDGPU::TMA:
        case AMDGPU::TMA_LO:
        case AMDGPU::TMA_HI:
          llvm_unreachable("trap handler registers should not be used");

        case AMDGPU::SRC_VCCZ:
          llvm_unreachable("src_vccz register should not be used");

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 225-255: Conditional logic and checks
```cpp
        case AMDGPU::SRC_EXECZ:
          llvm_unreachable("src_execz register should not be used");

        case AMDGPU::SRC_SCC:
          llvm_unreachable("src_scc register should not be used");

        default:
          break;
        }

        const TargetRegisterClass *RC = TRI.getPhysRegBaseClass(Reg);
        assert((!RC || TRI.isVGPRClass(RC) || TRI.isSGPRClass(RC) ||
                TRI.isAGPRClass(RC) || AMDGPU::TTMP_32RegClass.contains(Reg) ||
                AMDGPU::TTMP_64RegClass.contains(Reg) ||
                AMDGPU::TTMP_128RegClass.contains(Reg) ||
                AMDGPU::TTMP_256RegClass.contains(Reg) ||
                AMDGPU::TTMP_512RegClass.contains(Reg)) &&
               "Unknown register class");

        if (!RC || !TRI.isVGPRClass(RC))
          continue;

        if (MI.isCall() || MI.isMetaInstruction())
          continue;

        unsigned Width = divideCeil(TRI.getRegSizeInBits(*RC), 32);
        unsigned HWReg = TRI.getHWRegIndex(Reg);
        int MaxUsed = HWReg + Width - 1;
        MaxVGPR = std::max(MaxUsed, MaxVGPR);
      }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::max`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::max`。

### Lines 256-289: Conditional logic and checks
```cpp
      if (MI.isCall()) {
        // Pseudo used just to encode the underlying global. Is there a better
        // way to track this?

        // TODO: Some of the generic call-like pseudos do not encode the callee,
        // so we overly conservatively treat this as an indirect call.
        const MachineOperand *CalleeOp =
            TII->getNamedOperand(MI, AMDGPU::OpName::callee);

        const Function *Callee =
            CalleeOp ? getCalleeFunction(*CalleeOp) : nullptr;

        auto isSameFunction = [](const MachineFunction &MF, const Function *F) {
          return F == &MF.getFunction();
        };

        if (Callee && !isSameFunction(MF, Callee))
          Info.Callees.push_back(Callee);

        bool IsIndirect = !Callee || Callee->isDeclaration();
        Info.HasIndirectCall |= IsIndirect;

        // In object linking mode the linker has the full cross-TU view. It
        // propagates resource usage across both direct calls to external
        // declarations and true indirect calls. Skip the compile-time
        // conservative assumptions so that the locally emitted metadata
        // describes this function's own usage only.
        if (AMDGPUTargetMachine::EnableObjectLinking)
          continue;

        // FIXME: Call site could have norecurse on it
        if (!Callee || !Callee->doesNotRecurse()) {
          Info.HasRecursion = true;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 290-322: Conditional logic and checks
```cpp
          // TODO: If we happen to know there is no stack usage in the
          // callgraph, we don't need to assume an infinitely growing stack.
          if (!MI.isReturn()) {
            // We don't need to assume an unknown stack size for tail calls.

            // FIXME: This only benefits in the case where the kernel does not
            // directly call the tail called function. If a kernel directly
            // calls a tail recursive function, we'll assume maximum stack size
            // based on the regular call instruction.
            Info.CalleeSegmentSize = std::max(
                Info.CalleeSegmentSize,
                static_cast<uint64_t>(AssumedStackSizeForExternalCall));
          }
        }

        if (IsIndirect) {
          Info.CalleeSegmentSize =
              std::max(Info.CalleeSegmentSize,
                       static_cast<uint64_t>(AssumedStackSizeForExternalCall));

          // Register usage of indirect calls gets handled later
          Info.UsesVCC = true;
          Info.UsesFlatScratch = ST.hasFlatAddressSpace();
          Info.HasDynamicallySizedStack = true;
        }
      }
    }
  }

  Info.NumVGPR = MaxVGPR + 1;

  return Info;
}
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::max`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::max`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `cl::desc`, `cl::init`, `AMDGPUResourceUsageAnalysisWrapperPass::runOnMachineFunction`, `AMDGPU::getAMDHSACodeObjectVersion`, `AMDGPUResourceUsageAnalysis::run`, `AMDGPUResourceUsageAnalysisImpl::analyzeResourceUsage`
- **Main themes / 核心主题**: alias analysis / 别名分析; register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPUResourceUsageAnalysis.h"`
- `"AMDGPU.h"`
- `"AMDGPUTargetMachine.h"`
- `"GCNSubtarget.h"`
- `"SIMachineFunctionInfo.h"`
- `"llvm/CodeGen/MachineFrameInfo.h"`
- `"llvm/CodeGen/MachineModuleInfo.h"`
- `"llvm/CodeGen/TargetPassConfig.h"`
- `"llvm/IR/GlobalValue.h"`
- `"llvm/Target/TargetMachine.h"`
