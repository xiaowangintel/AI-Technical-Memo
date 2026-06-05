# SIPostRABundler.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/SIPostRABundler.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements SIPostRABundler for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 SIPostRABundler 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-33: File banner, includes, and setup
```cpp
//===-- SIPostRABundler.cpp -----------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// This pass creates bundles of memory instructions to protect adjacent loads
/// and stores from being rescheduled apart from each other post-RA.
///
//===----------------------------------------------------------------------===//

#include "SIPostRABundler.h"
#include "AMDGPU.h"
#include "GCNSubtarget.h"
#include "llvm/ADT/SmallSet.h"
#include "llvm/CodeGen/MachineFunctionPass.h"

using namespace llvm;

#define DEBUG_TYPE "si-post-ra-bundler"

namespace {

class SIPostRABundlerLegacy : public MachineFunctionPass {
public:
  static char ID;

public:
  SIPostRABundlerLegacy() : MachineFunctionPass(ID) {}

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `SIPostRABundlerLegacy`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`SIPostRABundlerLegacy`。

### Lines 34-67: Declares class SIPostRABundler
```cpp
  bool runOnMachineFunction(MachineFunction &MF) override;

  StringRef getPassName() const override {
    return "SI post-RA bundler";
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesAll();
    MachineFunctionPass::getAnalysisUsage(AU);
  }
};

class SIPostRABundler {
public:
  bool run(MachineFunction &MF);

private:
  const SIRegisterInfo *TRI;

  SmallSet<Register, 16> Defs;

  void collectUsedRegUnits(const MachineInstr &MI,
                           BitVector &UsedRegUnits) const;

  bool isBundleCandidate(const MachineInstr &MI) const;
  bool isDependentLoad(const MachineInstr &MI) const;
  bool canBundle(const MachineInstr &MI, const MachineInstr &NextMI) const;
};

constexpr uint64_t MemFlags = SIInstrFlags::MTBUF | SIInstrFlags::MUBUF |
                              SIInstrFlags::SMRD | SIInstrFlags::DS |
                              SIInstrFlags::FLAT | SIInstrFlags::MIMG |
                              SIInstrFlags::VIMAGE | SIInstrFlags::VSAMPLE;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `SIPostRABundler`, `MachineFunctionPass::getAnalysisUsage`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`SIPostRABundler`, `MachineFunctionPass::getAnalysisUsage`。

### Lines 68-101: Registers LLVM passes
```cpp
} // End anonymous namespace.

INITIALIZE_PASS(SIPostRABundlerLegacy, DEBUG_TYPE, "SI post-RA bundler", false,
                false)

char SIPostRABundlerLegacy::ID = 0;

char &llvm::SIPostRABundlerLegacyID = SIPostRABundlerLegacy::ID;

FunctionPass *llvm::createSIPostRABundlerPass() {
  return new SIPostRABundlerLegacy();
}

bool SIPostRABundler::isDependentLoad(const MachineInstr &MI) const {
  if (!MI.mayLoad())
    return false;

  for (const MachineOperand &Op : MI.explicit_operands()) {
    if (!Op.isReg())
      continue;
    Register Reg = Op.getReg();
    for (Register Def : Defs)
      if (TRI->regsOverlap(Reg, Def))
        return true;
  }

  return false;
}

void SIPostRABundler::collectUsedRegUnits(const MachineInstr &MI,
                                          BitVector &UsedRegUnits) const {
  if (MI.isDebugInstr())
    return;

```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `llvm::createSIPostRABundlerPass`, `SIPostRABundler::isDependentLoad`, `SIPostRABundler::collectUsedRegUnits`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`llvm::createSIPostRABundlerPass`, `SIPostRABundler::isDependentLoad`, `SIPostRABundler::collectUsedRegUnits`。

### Lines 102-135: Conditional logic and checks
```cpp
  for (const MachineOperand &Op : MI.operands()) {
    if (!Op.isReg() || !Op.readsReg())
      continue;

    Register Reg = Op.getReg();
    assert(!Op.getSubReg() &&
           "subregister indexes should not be present after RA");

    for (MCRegUnit Unit : TRI->regunits(Reg))
      UsedRegUnits.set(static_cast<unsigned>(Unit));
  }
}

bool SIPostRABundler::isBundleCandidate(const MachineInstr &MI) const {
  const uint64_t IMemFlags = MI.getDesc().TSFlags & MemFlags;
  return IMemFlags != 0 && MI.mayLoadOrStore() && !MI.isBundled();
}

bool SIPostRABundler::canBundle(const MachineInstr &MI,
                                const MachineInstr &NextMI) const {
  const uint64_t IMemFlags = MI.getDesc().TSFlags & MemFlags;

  return (IMemFlags != 0 && MI.mayLoadOrStore() && !NextMI.isBundled() &&
          NextMI.mayLoad() == MI.mayLoad() && NextMI.mayStore() == MI.mayStore() &&
          ((NextMI.getDesc().TSFlags & MemFlags) == IMemFlags) &&
          !isDependentLoad(NextMI));
}

bool SIPostRABundlerLegacy::runOnMachineFunction(MachineFunction &MF) {
  if (skipFunction(MF.getFunction()))
    return false;
  return SIPostRABundler().run(MF);
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `SIPostRABundler::isBundleCandidate`, `SIPostRABundler::canBundle`, `SIPostRABundlerLegacy::runOnMachineFunction`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`SIPostRABundler::isBundleCandidate`, `SIPostRABundler::canBundle`, `SIPostRABundlerLegacy::runOnMachineFunction`。

### Lines 136-169: Implements SIPostRABundlerPass::run
```cpp
PreservedAnalyses SIPostRABundlerPass::run(MachineFunction &MF,
                                           MachineFunctionAnalysisManager &) {
  SIPostRABundler().run(MF);
  return PreservedAnalyses::all();
}

bool SIPostRABundler::run(MachineFunction &MF) {

  TRI = MF.getSubtarget<GCNSubtarget>().getRegisterInfo();
  BitVector BundleUsedRegUnits(TRI->getNumRegUnits());
  BitVector KillUsedRegUnits(TRI->getNumRegUnits());

  bool Changed = false;
  for (MachineBasicBlock &MBB : MF) {
    bool HasIGLPInstrs = llvm::any_of(MBB.instrs(), [](MachineInstr &MI) {
      unsigned Opc = MI.getOpcode();
      return Opc == AMDGPU::SCHED_GROUP_BARRIER || Opc == AMDGPU::IGLP_OPT;
    });

    // Don't cluster with IGLP instructions.
    if (HasIGLPInstrs)
      continue;

    MachineBasicBlock::instr_iterator Next;
    MachineBasicBlock::instr_iterator B = MBB.instr_begin();
    MachineBasicBlock::instr_iterator E = MBB.instr_end();

    for (auto I = B; I != E; I = Next) {
      Next = std::next(I);
      if (!isBundleCandidate(*I))
        continue;

      assert(Defs.empty());

```
**EN:** This section contains concrete logic for SIPostRABundlerPass::run. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SIPostRABundlerPass::run`, `PreservedAnalyses::all`, `SIPostRABundler::run`.
**CN:** 本节包含与 SIPostRABundlerPass::run 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SIPostRABundlerPass::run`, `PreservedAnalyses::all`, `SIPostRABundler::run`。

### Lines 170-201: Conditional logic and checks
```cpp
      if (I->getNumExplicitDefs() != 0)
        Defs.insert(I->defs().begin()->getReg());

      MachineBasicBlock::instr_iterator BundleStart = I;
      MachineBasicBlock::instr_iterator BundleEnd = I;
      unsigned ClauseLength = 1;
      for (I = Next; I != E; I = Next) {
        Next = std::next(I);

        assert(BundleEnd != I);
        if (canBundle(*BundleEnd, *I)) {
          BundleEnd = I;
          if (I->getNumExplicitDefs() != 0)
            Defs.insert(I->defs().begin()->getReg());
          ++ClauseLength;
        } else if (!I->isMetaInstruction() ||
                   I->getOpcode() == AMDGPU::SCHED_BARRIER) {
          // SCHED_BARRIER is not bundled to be honored by scheduler later.
          // Allow other meta instructions in between bundle candidates, but do
          // not start or end a bundle on one.
          //
          // TODO: It may be better to move meta instructions like dbg_value
          // after the bundle. We're relying on the memory legalizer to unbundle
          // these.
          break;
        }
      }

      Next = std::next(BundleEnd);
      if (ClauseLength > 1) {
        Changed = true;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::next`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::next`。

### Lines 202-234: Conditional logic and checks
```cpp
        // Before register allocation, kills are inserted after potential soft
        // clauses to hint register allocation. Look for kills that look like
        // this, and erase them.
        if (Next != E && Next->isKill()) {

          // TODO: Should maybe back-propagate kill flags to the bundle.
          for (const MachineInstr &BundleMI : make_range(BundleStart, Next))
            collectUsedRegUnits(BundleMI, BundleUsedRegUnits);

          BundleUsedRegUnits.flip();

          while (Next != E && Next->isKill()) {
            MachineInstr &Kill = *Next;
            collectUsedRegUnits(Kill, KillUsedRegUnits);

            KillUsedRegUnits &= BundleUsedRegUnits;

            // Erase the kill if it's a subset of the used registers.
            //
            // TODO: Should we just remove all kills? Is there any real reason to
            // keep them after RA?
            if (KillUsedRegUnits.none()) {
              ++Next;
              Kill.eraseFromParent();
            } else
              break;

            KillUsedRegUnits.reset();
          }

          BundleUsedRegUnits.reset();
        }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 235-243: Result computation and returns
```cpp
        finalizeBundle(MBB, BundleStart, Next);
      }

      Defs.clear();
    }
  }

  return Changed;
}
```
**EN:** This section finalizes intermediate state and returns the value that the caller or pass pipeline needs.
**CN:** 本节整理中间状态并返回调用者或 Pass 流水线所需的结果。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `SIPostRABundlerLegacy`, `SIPostRABundler`, `MachineFunctionPass::getAnalysisUsage`, `llvm::createSIPostRABundlerPass`, `SIPostRABundler::isDependentLoad`, `SIPostRABundler::collectUsedRegUnits`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; scheduling / 调度
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"SIPostRABundler.h"`
- `"AMDGPU.h"`
- `"GCNSubtarget.h"`
- `"llvm/ADT/SmallSet.h"`
- `"llvm/CodeGen/MachineFunctionPass.h"`
