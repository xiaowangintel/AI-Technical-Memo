# AMDGPUSetWavePriority.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUSetWavePriority.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUSetWavePriority for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPUSetWavePriority 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-34: File banner, includes, and setup
```cpp
//===- AMDGPUSetWavePriority.cpp - Set wave priority ----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// Pass to temporarily raise the wave priority beginning the start of
/// the shader function until its last VMEM instructions to allow younger
/// waves to issue their VMEM instructions as well.
//
//===----------------------------------------------------------------------===//

#include "AMDGPU.h"
#include "GCNSubtarget.h"
#include "MCTargetDesc/AMDGPUMCTargetDesc.h"
#include "SIInstrInfo.h"
#include "llvm/ADT/PostOrderIterator.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachinePassManager.h"

using namespace llvm;

#define DEBUG_TYPE "amdgpu-set-wave-priority"

static cl::opt<unsigned> DefaultVALUInstsThreshold(
    "amdgpu-set-wave-priority-valu-insts-threshold",
    cl::desc("VALU instruction count threshold for adjusting wave priority"),
    cl::init(100), cl::Hidden);

namespace {

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `cl::desc`, `cl::init`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`cl::desc`, `cl::init`。

### Lines 35-67: Declares struct MBBInfo
```cpp
struct MBBInfo {
  MBBInfo() = default;
  unsigned NumVALUInstsAtStart = 0;
  bool MayReachVMEMLoad = false;
  MachineInstr *LastVMEMLoad = nullptr;
};

using MBBInfoSet = DenseMap<const MachineBasicBlock *, MBBInfo>;

class AMDGPUSetWavePriority {
public:
  bool run(MachineFunction &MF);

private:
  MachineInstr *BuildSetprioMI(MachineBasicBlock &MBB,
                               MachineBasicBlock::iterator I,
                               unsigned priority) const;

  const SIInstrInfo *TII;
};

class AMDGPUSetWavePriorityLegacy : public MachineFunctionPass {
public:
  static char ID;

  AMDGPUSetWavePriorityLegacy() : MachineFunctionPass(ID) {}

  StringRef getPassName() const override { return "Set wave priority"; }

  bool runOnMachineFunction(MachineFunction &MF) override {
    if (skipFunction(MF.getFunction()))
      return false;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `MBBInfo`, `AMDGPUSetWavePriority`, `AMDGPUSetWavePriorityLegacy`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`MBBInfo`, `AMDGPUSetWavePriority`, `AMDGPUSetWavePriorityLegacy`。

### Lines 68-90: Registers LLVM passes
```cpp
    return AMDGPUSetWavePriority().run(MF);
  }
};

} // End anonymous namespace.

INITIALIZE_PASS(AMDGPUSetWavePriorityLegacy, DEBUG_TYPE, "Set wave priority",
                false, false)

char AMDGPUSetWavePriorityLegacy::ID = 0;

FunctionPass *llvm::createAMDGPUSetWavePriorityPass() {
  return new AMDGPUSetWavePriorityLegacy();
}

MachineInstr *
AMDGPUSetWavePriority::BuildSetprioMI(MachineBasicBlock &MBB,
                                      MachineBasicBlock::iterator I,
                                      unsigned priority) const {
  return BuildMI(MBB, I, DebugLoc(), TII->get(AMDGPU::S_SETPRIO))
      .addImm(priority);
}

```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `llvm::createAMDGPUSetWavePriorityPass`, `AMDGPUSetWavePriority::BuildSetprioMI`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`llvm::createAMDGPUSetWavePriorityPass`, `AMDGPUSetWavePriority::BuildSetprioMI`。

### Lines 91-122: Defines CanLowerPriorityDirectlyInPredecessors
```cpp
// Checks that for every predecessor Pred that can reach a VMEM load,
// none of Pred's successors can reach a VMEM load.
static bool CanLowerPriorityDirectlyInPredecessors(const MachineBasicBlock &MBB,
                                                   MBBInfoSet &MBBInfos) {
  for (const MachineBasicBlock *Pred : MBB.predecessors()) {
    if (!MBBInfos[Pred].MayReachVMEMLoad)
      continue;
    for (const MachineBasicBlock *Succ : Pred->successors()) {
      if (MBBInfos[Succ].MayReachVMEMLoad)
        return false;
    }
  }
  return true;
}

static bool isVMEMLoad(const MachineInstr &MI) {
  return SIInstrInfo::isVMEM(MI) && MI.mayLoad();
}

PreservedAnalyses
llvm::AMDGPUSetWavePriorityPass::run(MachineFunction &MF,
                                     MachineFunctionAnalysisManager &MFAM) {
  if (!AMDGPUSetWavePriority().run(MF))
    return PreservedAnalyses::all();

  return getMachineFunctionPassPreservedAnalyses();
}

bool AMDGPUSetWavePriority::run(MachineFunction &MF) {
  const unsigned HighPriority = 3;
  const unsigned LowPriority = 0;

```
**EN:** This section contains concrete logic for CanLowerPriorityDirectlyInPredecessors. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SIInstrInfo::isVMEM`, `AMDGPUSetWavePriorityPass::run`, `PreservedAnalyses::all`.
**CN:** 本节包含与 CanLowerPriorityDirectlyInPredecessors 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SIInstrInfo::isVMEM`, `AMDGPUSetWavePriorityPass::run`, `PreservedAnalyses::all`。

### Lines 123-156: Declares getFunction
```cpp
  Function &F = MF.getFunction();
  if (!AMDGPU::isEntryFunctionCC(F.getCallingConv()))
    return false;

  const GCNSubtarget &ST = MF.getSubtarget<GCNSubtarget>();
  TII = ST.getInstrInfo();

  unsigned VALUInstsThreshold = DefaultVALUInstsThreshold;
  Attribute A = F.getFnAttribute("amdgpu-wave-priority-threshold");
  if (A.isValid())
    A.getValueAsString().getAsInteger(0, VALUInstsThreshold);

  // Find VMEM loads that may be executed before long-enough sequences of
  // VALU instructions. We currently assume that backedges/loops, branch
  // probabilities and other details can be ignored, so we essentially
  // determine the largest number of VALU instructions along every
  // possible path from the start of the function that may potentially be
  // executed provided no backedge is ever taken.
  MBBInfoSet MBBInfos;
  for (MachineBasicBlock *MBB : post_order(&MF)) {
    bool AtStart = true;
    unsigned MaxNumVALUInstsInMiddle = 0;
    unsigned NumVALUInstsAtEnd = 0;
    for (MachineInstr &MI : *MBB) {
      if (isVMEMLoad(MI)) {
        AtStart = false;
        MBBInfo &Info = MBBInfos[MBB];
        Info.NumVALUInstsAtStart = 0;
        MaxNumVALUInstsInMiddle = 0;
        NumVALUInstsAtEnd = 0;
        Info.LastVMEMLoad = &MI;
      } else if (SIInstrInfo::isDS(MI)) {
        AtStart = false;
        MaxNumVALUInstsInMiddle =
```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `AMDGPU::isEntryFunctionCC`, `SIInstrInfo::isDS`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`AMDGPU::isEntryFunctionCC`, `SIInstrInfo::isDS`。

### Lines 157-189: Conditional logic and checks
```cpp
            std::max(MaxNumVALUInstsInMiddle, NumVALUInstsAtEnd);
        NumVALUInstsAtEnd = 0;
      } else if (SIInstrInfo::isVALU(MI)) {
        if (AtStart)
          ++MBBInfos[MBB].NumVALUInstsAtStart;
        ++NumVALUInstsAtEnd;
      }
    }

    bool SuccsMayReachVMEMLoad = false;
    unsigned NumFollowingVALUInsts = 0;
    for (const MachineBasicBlock *Succ : MBB->successors()) {
      const MBBInfo &SuccInfo = MBBInfos[Succ];
      SuccsMayReachVMEMLoad |= SuccInfo.MayReachVMEMLoad;
      NumFollowingVALUInsts =
          std::max(NumFollowingVALUInsts, SuccInfo.NumVALUInstsAtStart);
    }
    MBBInfo &Info = MBBInfos[MBB];
    if (AtStart)
      Info.NumVALUInstsAtStart += NumFollowingVALUInsts;
    NumVALUInstsAtEnd += NumFollowingVALUInsts;

    unsigned MaxNumVALUInsts =
        std::max(MaxNumVALUInstsInMiddle, NumVALUInstsAtEnd);
    Info.MayReachVMEMLoad =
        SuccsMayReachVMEMLoad ||
        (Info.LastVMEMLoad && MaxNumVALUInsts >= VALUInstsThreshold);
  }

  MachineBasicBlock &Entry = MF.front();
  if (!MBBInfos[&Entry].MayReachVMEMLoad)
    return false;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::max`, `SIInstrInfo::isVALU`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::max`, `SIInstrInfo::isVALU`。

### Lines 190-222: Declares begin
```cpp
  // Raise the priority at the beginning of the shader.
  MachineBasicBlock::iterator I = Entry.begin(), E = Entry.end();
  while (I != E && !SIInstrInfo::isVALU(*I) && !I->isTerminator())
    ++I;
  BuildSetprioMI(Entry, I, HighPriority);

  // Lower the priority on edges where control leaves blocks from which
  // the VMEM loads are reachable.
  SmallPtrSet<MachineBasicBlock *, 16> PriorityLoweringBlocks;
  for (MachineBasicBlock &MBB : MF) {
    if (MBBInfos[&MBB].MayReachVMEMLoad) {
      if (MBB.succ_empty())
        PriorityLoweringBlocks.insert(&MBB);
      continue;
    }

    if (CanLowerPriorityDirectlyInPredecessors(MBB, MBBInfos)) {
      for (MachineBasicBlock *Pred : MBB.predecessors()) {
        if (MBBInfos[Pred].MayReachVMEMLoad)
          PriorityLoweringBlocks.insert(Pred);
      }
      continue;
    }

    // Where lowering the priority in predecessors is not possible, the
    // block receiving control either was not part of a loop in the first
    // place or the loop simplification/canonicalization pass should have
    // already tried to split the edge and insert a preheader, and if for
    // whatever reason it failed to do so, then this leaves us with the
    // only option of lowering the priority within the loop.
    PriorityLoweringBlocks.insert(&MBB);
  }

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `SIInstrInfo::isVALU`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`SIInstrInfo::isVALU`。

### Lines 223-233: Result computation and returns
```cpp
  for (MachineBasicBlock *MBB : PriorityLoweringBlocks) {
    MachineInstr *LastVMEMLoad = MBBInfos[MBB].LastVMEMLoad;
    BuildSetprioMI(*MBB,
                   LastVMEMLoad
                       ? std::next(MachineBasicBlock::iterator(LastVMEMLoad))
                       : MBB->begin(),
                   LowPriority);
  }

  return true;
}
```
**EN:** This section finalizes intermediate state and returns the value that the caller or pass pipeline needs. Main symbols: `std::next`, `MachineBasicBlock::iterator`.
**CN:** 本节整理中间状态并返回调用者或 Pass 流水线所需的结果。 主要符号：`std::next`, `MachineBasicBlock::iterator`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `MBBInfo`, `AMDGPUSetWavePriority`, `AMDGPUSetWavePriorityLegacy`, `cl::desc`, `cl::init`, `llvm::createAMDGPUSetWavePriorityPass`
- **Main themes / 核心主题**: instruction semantics / 指令语义; subtarget modeling / 子目标建模; lowering / 降低; LLVM pass integration / LLVM Pass 集成
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPU.h"`
- `"GCNSubtarget.h"`
- `"MCTargetDesc/AMDGPUMCTargetDesc.h"`
- `"SIInstrInfo.h"`
- `"llvm/ADT/PostOrderIterator.h"`
- `"llvm/CodeGen/MachineFunctionPass.h"`
- `"llvm/CodeGen/MachinePassManager.h"`
