# AMDGPUMarkLastScratchLoad.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUMarkLastScratchLoad.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUMarkLastScratchLoad for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPUMarkLastScratchLoad 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: File banner, includes, and setup
```cpp
//===-- AMDGPUMarkLastScratchLoad.cpp -------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Mark scratch load/spill instructions which are guaranteed to be the last time
// this scratch slot is used so it can be evicted from caches.
//
// TODO: Handle general stack accesses not just spilling.
//
//===----------------------------------------------------------------------===//

#include "AMDGPU.h"
#include "GCNSubtarget.h"
#include "llvm/CodeGen/LiveIntervals.h"
#include "llvm/CodeGen/LiveStacks.h"
#include "llvm/CodeGen/MachineOperand.h"

using namespace llvm;

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 24-46: Preprocessor guards and macros
```cpp
#define DEBUG_TYPE "amdgpu-mark-last-scratch-load"

namespace {

class AMDGPUMarkLastScratchLoad {
private:
  LiveStacks *LS = nullptr;
  LiveIntervals *LIS = nullptr;
  SlotIndexes *SI = nullptr;
  const SIInstrInfo *SII = nullptr;

public:
  AMDGPUMarkLastScratchLoad(LiveStacks *LS, LiveIntervals *LIS, SlotIndexes *SI)
      : LS(LS), LIS(LIS), SI(SI) {}
  bool run(MachineFunction &MF);
};

class AMDGPUMarkLastScratchLoadLegacy : public MachineFunctionPass {
public:
  static char ID;

  AMDGPUMarkLastScratchLoadLegacy() : MachineFunctionPass(ID) {}

```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `AMDGPUMarkLastScratchLoad`, `AMDGPUMarkLastScratchLoadLegacy`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`AMDGPUMarkLastScratchLoad`, `AMDGPUMarkLastScratchLoadLegacy`。

### Lines 47-68: Declares runOnMachineFunction
```cpp
  bool runOnMachineFunction(MachineFunction &MF) override;

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addRequired<SlotIndexesWrapperPass>();
    AU.addRequired<LiveIntervalsWrapperPass>();
    AU.addRequired<LiveStacksWrapperLegacy>();
    AU.setPreservesAll();
    MachineFunctionPass::getAnalysisUsage(AU);
  }

  StringRef getPassName() const override {
    return "AMDGPU Mark Last Scratch Load";
  }
};

} // end anonymous namespace

bool AMDGPUMarkLastScratchLoadLegacy::runOnMachineFunction(
    MachineFunction &MF) {
  if (skipFunction(MF.getFunction()))
    return false;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `MachineFunctionPass::getAnalysisUsage`, `AMDGPUMarkLastScratchLoadLegacy::runOnMachineFunction`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`MachineFunctionPass::getAnalysisUsage`, `AMDGPUMarkLastScratchLoadLegacy::runOnMachineFunction`。

### Lines 69-91: Declares getLS
```cpp
  auto &LS = getAnalysis<LiveStacksWrapperLegacy>().getLS();
  auto &LIS = getAnalysis<LiveIntervalsWrapperPass>().getLIS();
  auto &SI = getAnalysis<SlotIndexesWrapperPass>().getSI();

  return AMDGPUMarkLastScratchLoad(&LS, &LIS, &SI).run(MF);
}

PreservedAnalyses
AMDGPUMarkLastScratchLoadPass::run(MachineFunction &MF,
                                   MachineFunctionAnalysisManager &MFAM) {
  auto &LS = MFAM.getResult<LiveStacksAnalysis>(MF);
  auto &LIS = MFAM.getResult<LiveIntervalsAnalysis>(MF);
  auto &SI = MFAM.getResult<SlotIndexesAnalysis>(MF);

  AMDGPUMarkLastScratchLoad(&LS, &LIS, &SI).run(MF);
  return PreservedAnalyses::all();
}

bool AMDGPUMarkLastScratchLoad::run(MachineFunction &MF) {
  const GCNSubtarget &ST = MF.getSubtarget<GCNSubtarget>();
  if (ST.getGeneration() < AMDGPUSubtarget::GFX12)
    return false;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `AMDGPUMarkLastScratchLoadPass::run`, `PreservedAnalyses::all`, `AMDGPUMarkLastScratchLoad::run`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`AMDGPUMarkLastScratchLoadPass::run`, `PreservedAnalyses::all`, `AMDGPUMarkLastScratchLoad::run`。

### Lines 92-115: Declares getInstrInfo
```cpp
  SII = ST.getInstrInfo();
  SlotIndexes &Slots = *LIS->getSlotIndexes();

  const unsigned NumSlots = LS->getNumIntervals();
  if (NumSlots == 0) {
    LLVM_DEBUG(dbgs() << "No live slots, skipping\n");
    return false;
  }

  LLVM_DEBUG(dbgs() << LS->getNumIntervals() << " intervals\n");

  bool Changed = false;

  for (auto &[SS, LI] : *LS) {
    for (const LiveRange::Segment &Segment : LI.segments) {

      // Ignore segments that run to the end of basic block because in this case
      // slot is still live at the end of it.
      if (Segment.end.isBlock())
        continue;

      const int FrameIndex = LI.reg().stackSlotIndex();
      MachineInstr *LastLoad = nullptr;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 116-136: Conditional logic and checks
```cpp
      MachineInstr *MISegmentEnd = SI->getInstructionFromIndex(Segment.end);

      // If there is no instruction at this slot because it was deleted take the
      // instruction from the next slot.
      if (!MISegmentEnd) {
        SlotIndex NextSlot = Slots.getNextNonNullIndex(Segment.end);
        MISegmentEnd = SI->getInstructionFromIndex(NextSlot);
      }

      MachineInstr *MISegmentStart = SI->getInstructionFromIndex(Segment.start);
      MachineBasicBlock *BB = MISegmentEnd->getParent();

      // Start iteration backwards from segment end until the start of basic
      // block or start of segment if it is in the same basic block.
      auto End = BB->rend();
      if (MISegmentStart && MISegmentStart->getParent() == BB)
        End = MISegmentStart->getReverseIterator();

      for (auto MI = MISegmentEnd->getReverseIterator(); MI != End; ++MI) {
        int LoadFI = 0;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 137-158: Conditional logic and checks
```cpp
        if (SII->isLoadFromStackSlot(*MI, LoadFI) && LoadFI == FrameIndex) {
          LastLoad = &*MI;
          break;
        }
      }

      if (LastLoad && !LastLoad->memoperands_empty()) {
        MachineMemOperand *MMO = *LastLoad->memoperands_begin();
        MMO->setFlags(MOLastUse);
        Changed = true;
        LLVM_DEBUG(dbgs() << "  Found last load: " << *LastLoad);
      }
    }
  }

  return Changed;
}

char AMDGPUMarkLastScratchLoadLegacy::ID = 0;

char &llvm::AMDGPUMarkLastScratchLoadID = AMDGPUMarkLastScratchLoadLegacy::ID;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 159-164: Registers LLVM passes
```cpp
INITIALIZE_PASS_BEGIN(AMDGPUMarkLastScratchLoadLegacy, DEBUG_TYPE,
                      "AMDGPU Mark last scratch load", false, false)
INITIALIZE_PASS_DEPENDENCY(SlotIndexesWrapperPass)
INITIALIZE_PASS_DEPENDENCY(LiveStacksWrapperLegacy)
INITIALIZE_PASS_END(AMDGPUMarkLastScratchLoadLegacy, DEBUG_TYPE,
                    "AMDGPU Mark last scratch load", false, false)
```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `AMDGPUMarkLastScratchLoad`, `AMDGPUMarkLastScratchLoadLegacy`, `MachineFunctionPass::getAnalysisUsage`, `AMDGPUMarkLastScratchLoadLegacy::runOnMachineFunction`, `AMDGPUMarkLastScratchLoadPass::run`, `PreservedAnalyses::all`
- **Main themes / 核心主题**: instruction semantics / 指令语义; subtarget modeling / 子目标建模; LLVM pass integration / LLVM Pass 集成; machine-function state / MachineFunction 状态
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPU.h"`
- `"GCNSubtarget.h"`
- `"llvm/CodeGen/LiveIntervals.h"`
- `"llvm/CodeGen/LiveStacks.h"`
- `"llvm/CodeGen/MachineOperand.h"`
