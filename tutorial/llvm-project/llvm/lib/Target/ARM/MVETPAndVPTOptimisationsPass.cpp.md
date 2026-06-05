# MVETPAndVPTOptimisationsPass.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/MVETPAndVPTOptimisationsPass.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `MVETPAndVPTOptimisationsPass` for the ARM backend, focusing on target-specific code-generation behavior.
- 用途 (CN): 实现 ARM 后端中的 `MVETPAndVPTOptimisationsPass`，重点处理目标相关代码生成行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```cpp
//===-- MVETPAndVPTOptimisationsPass.cpp ----------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file This pass does a few optimisations related to Tail predicated loops
/// and MVE VPT blocks before register allocation is performed. For VPT blocks
/// the goal is to maximize the sizes of the blocks that will be created by the
/// MVE VPT Block Insertion pass (which runs after register allocation). For
/// tail predicated loops we transform the loop into something that will
/// hopefully make the backend ARMLowOverheadLoops pass's job easier.
///
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 16-16
```cpp
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 18-31
```cpp
#include "ARM.h"
#include "ARMSubtarget.h"
#include "MVETailPredUtils.h"
#include "Thumb2InstrInfo.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineDominators.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineLoopInfo.h"
#include "llvm/InitializePasses.h"
#include "llvm/Support/Debug.h"
#include <cassert>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 33-33
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 35-35
```cpp
#define DEBUG_TYPE "arm-mve-vpt-opts"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 37-40
```cpp
static cl::opt<bool>
MergeEndDec("arm-enable-merge-loopenddec", cl::Hidden,
    cl::desc("Enable merging Loop End and Dec instructions."),
    cl::init(true));
```
- EN: Declares `cl::desc`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::desc`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 42-45
```cpp
static cl::opt<bool>
SetLRPredicate("arm-set-lr-predicate", cl::Hidden,
    cl::desc("Enable setting lr as a predicate in tail predication regions."),
    cl::init(true));
```
- EN: Declares `cl::desc`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::desc`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 47-52
```cpp
namespace {
class MVETPAndVPTOptimisations : public MachineFunctionPass {
public:
  static char ID;
  const Thumb2InstrInfo *TII;
  MachineRegisterInfo *MRI;
```
- EN: Declares `MVETPAndVPTOptimisations`, packaging target-specific state and APIs around `MVETPAndVPTOptimisationsPass`.
- CN: 这里声明 `MVETPAndVPTOptimisations`，把与 `MVETPAndVPTOptimisationsPass` 相关的目标特定状态和 API 组织在一起。

### Lines 54-54
```cpp
  MVETPAndVPTOptimisations() : MachineFunctionPass(ID) {}
```
- EN: Implements `MVETPAndVPTOptimisations`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MVETPAndVPTOptimisations`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 56-56
```cpp
  bool runOnMachineFunction(MachineFunction &Fn) override;
```
- EN: Declares `runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 58-64
```cpp
  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addRequired<MachineLoopInfoWrapperPass>();
    AU.addPreserved<MachineLoopInfoWrapperPass>();
    AU.addRequired<MachineDominatorTreeWrapperPass>();
    AU.addPreserved<MachineDominatorTreeWrapperPass>();
    MachineFunctionPass::getAnalysisUsage(AU);
  }
```
- EN: Implements `getAnalysisUsage`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getAnalysisUsage`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 66-68
```cpp
  StringRef getPassName() const override {
    return "ARM MVE TailPred and VPT Optimisation Pass";
  }
```
- EN: Implements `getPassName`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getPassName`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 70-84
```cpp
private:
  bool LowerWhileLoopStart(MachineLoop *ML);
  bool MergeLoopEnd(MachineLoop *ML);
  bool ConvertTailPredLoop(MachineLoop *ML, MachineDominatorTree *DT);
  MachineInstr &ReplaceRegisterUseWithVPNOT(MachineBasicBlock &MBB,
                                            MachineInstr &Instr,
                                            MachineOperand &User,
                                            Register Target);
  bool ReduceOldVCCRValueUses(MachineBasicBlock &MBB);
  bool ReplaceVCMPsByVPNOTs(MachineBasicBlock &MBB);
  bool ReplaceConstByVPNOTs(MachineBasicBlock &MBB, MachineDominatorTree *DT);
  bool ConvertVPSEL(MachineBasicBlock &MBB);
  bool HintDoLoopStartReg(MachineBasicBlock &MBB);
  MachineInstr *CheckForLRUseInPredecessors(MachineBasicBlock *PreHeader,
                                            MachineInstr *LoopStart);
```
- EN: Declares `LowerWhileLoopStart`, a lowering routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LowerWhileLoopStart`，它是一个围绕机器基本块展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 85-85
```cpp
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 87-87
```cpp
char MVETPAndVPTOptimisations::ID = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 89-89
```cpp
} // end anonymous namespace
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 91-97
```cpp
INITIALIZE_PASS_BEGIN(MVETPAndVPTOptimisations, DEBUG_TYPE,
                      "ARM MVE TailPred and VPT Optimisations pass", false,
                      false)
INITIALIZE_PASS_DEPENDENCY(MachineLoopInfoWrapperPass)
INITIALIZE_PASS_DEPENDENCY(MachineDominatorTreeWrapperPass)
INITIALIZE_PASS_END(MVETPAndVPTOptimisations, DEBUG_TYPE,
                    "ARM MVE TailPred and VPT Optimisations pass", false, false)
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 99-105
```cpp
static MachineInstr *LookThroughCOPY(MachineInstr *MI,
                                     MachineRegisterInfo *MRI) {
  while (MI && MI->getOpcode() == TargetOpcode::COPY &&
         MI->getOperand(1).getReg().isVirtual())
    MI = MRI->getVRegDef(MI->getOperand(1).getReg());
  return MI;
}
```
- EN: Implements `LookThroughCOPY`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `LookThroughCOPY`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 107-118
```cpp
// Given a loop ML, this attempts to find the t2LoopEnd, t2LoopDec and
// corresponding PHI that make up a low overhead loop. Only handles 'do' loops
// at the moment, returning a t2DoLoopStart in LoopStart.
static bool findLoopComponents(MachineLoop *ML, MachineRegisterInfo *MRI,
                               MachineInstr *&LoopStart, MachineInstr *&LoopPhi,
                               MachineInstr *&LoopDec, MachineInstr *&LoopEnd) {
  MachineBasicBlock *Header = ML->getHeader();
  MachineBasicBlock *Latch = ML->getLoopLatch();
  if (!Header || !Latch) {
    LLVM_DEBUG(dbgs() << "  no Loop Latch or Header\n");
    return false;
  }
```
- EN: Implements `findLoopComponents`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `findLoopComponents`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 120-134
```cpp
  // Find the loop end from the terminators.
  LoopEnd = nullptr;
  for (auto &T : Latch->terminators()) {
    if (T.getOpcode() == ARM::t2LoopEnd && T.getOperand(1).getMBB() == Header) {
      LoopEnd = &T;
      break;
    }
    if (T.getOpcode() == ARM::t2LoopEndDec &&
        T.getOperand(2).getMBB() == Header) {
      LoopEnd = &T;
      break;
    }
  }
  if (!LoopEnd) {
    LLVM_DEBUG(dbgs() << "  no LoopEnd\n");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 135-137
```cpp
    return false;
  }
  LLVM_DEBUG(dbgs() << "  found loop end: " << *LoopEnd);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 139-153
```cpp
  // Find the dec from the use of the end. There may be copies between
  // instructions. We expect the loop to loop like:
  //   $vs = t2DoLoopStart ...
  // loop:
  //   $vp = phi [ $vs ], [ $vd ]
  //   ...
  //   $vd = t2LoopDec $vp
  //   ...
  //   t2LoopEnd $vd, loop
  if (LoopEnd->getOpcode() == ARM::t2LoopEndDec)
    LoopDec = LoopEnd;
  else {
    LoopDec =
        LookThroughCOPY(MRI->getVRegDef(LoopEnd->getOperand(0).getReg()), MRI);
    if (!LoopDec || LoopDec->getOpcode() != ARM::t2LoopDec) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 154-158
```cpp
      LLVM_DEBUG(dbgs() << "  didn't find LoopDec where we expected!\n");
      return false;
    }
  }
  LLVM_DEBUG(dbgs() << "  found loop dec: " << *LoopDec);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 160-169
```cpp
  LoopPhi =
      LookThroughCOPY(MRI->getVRegDef(LoopDec->getOperand(1).getReg()), MRI);
  if (!LoopPhi || LoopPhi->getOpcode() != TargetOpcode::PHI ||
      LoopPhi->getNumOperands() != 5 ||
      (LoopPhi->getOperand(2).getMBB() != Latch &&
       LoopPhi->getOperand(4).getMBB() != Latch)) {
    LLVM_DEBUG(dbgs() << "  didn't find PHI where we expected!\n");
    return false;
  }
  LLVM_DEBUG(dbgs() << "  found loop phi: " << *LoopPhi);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 171-181
```cpp
  Register StartReg = LoopPhi->getOperand(2).getMBB() == Latch
                          ? LoopPhi->getOperand(3).getReg()
                          : LoopPhi->getOperand(1).getReg();
  LoopStart = LookThroughCOPY(MRI->getVRegDef(StartReg), MRI);
  if (!LoopStart || (LoopStart->getOpcode() != ARM::t2DoLoopStart &&
                     LoopStart->getOpcode() != ARM::t2WhileLoopSetup &&
                     LoopStart->getOpcode() != ARM::t2WhileLoopStartLR)) {
    LLVM_DEBUG(dbgs() << "  didn't find Start where we expected!\n");
    return false;
  }
  LLVM_DEBUG(dbgs() << "  found loop start: " << *LoopStart);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 183-184
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 186-189
```cpp
static void RevertWhileLoopSetup(MachineInstr *MI, const TargetInstrInfo *TII) {
  MachineBasicBlock *MBB = MI->getParent();
  assert(MI->getOpcode() == ARM::t2WhileLoopSetup &&
         "Only expected a t2WhileLoopSetup in RevertWhileLoopStart!");
```
- EN: Implements `RevertWhileLoopSetup`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `RevertWhileLoopSetup`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 191-199
```cpp
  // Subs
  MachineInstrBuilder MIB =
      BuildMI(*MBB, MI, MI->getDebugLoc(), TII->get(ARM::t2SUBri));
  MIB.add(MI->getOperand(0));
  MIB.add(MI->getOperand(1));
  MIB.addImm(0);
  MIB.addImm(ARMCC::AL);
  MIB.addReg(ARM::NoRegister);
  MIB.addReg(ARM::CPSR, RegState::Define);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 201-212
```cpp
  // Attempt to find a t2WhileLoopStart and revert to a t2Bcc.
  for (MachineInstr &I : MBB->terminators()) {
    if (I.getOpcode() == ARM::t2WhileLoopStart) {
      MachineInstrBuilder MIB =
          BuildMI(*MBB, &I, I.getDebugLoc(), TII->get(ARM::t2Bcc));
      MIB.add(MI->getOperand(1)); // branch target
      MIB.addImm(ARMCC::EQ);
      MIB.addReg(ARM::CPSR);
      I.eraseFromParent();
      break;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 214-215
```cpp
  MI->eraseFromParent();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 217-229
```cpp
// The Hardware Loop insertion and ISel Lowering produce the pseudos for the
// start of a while loop:
//   %a:gprlr = t2WhileLoopSetup %Cnt
//   t2WhileLoopStart %a, %BB
// We want to convert those to a single instruction which, like t2LoopEndDec and
// t2DoLoopStartTP is both a terminator and produces a value:
//   %a:grplr: t2WhileLoopStartLR %Cnt, %BB
//
// Otherwise if we can't, we revert the loop. t2WhileLoopSetup and
// t2WhileLoopStart are not valid past regalloc.
bool MVETPAndVPTOptimisations::LowerWhileLoopStart(MachineLoop *ML) {
  LLVM_DEBUG(dbgs() << "LowerWhileLoopStart on loop "
                    << ML->getHeader()->getName() << "\n");
```
- EN: Implements `MVETPAndVPTOptimisations::LowerWhileLoopStart`, a lowering routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MVETPAndVPTOptimisations::LowerWhileLoopStart`，它是一个围绕目标相关状态展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 231-233
```cpp
  MachineInstr *LoopEnd, *LoopPhi, *LoopStart, *LoopDec;
  if (!findLoopComponents(ML, MRI, LoopStart, LoopPhi, LoopDec, LoopEnd))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 235-236
```cpp
  if (LoopStart->getOpcode() != ARM::t2WhileLoopSetup)
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 238-247
```cpp
  Register LR = LoopStart->getOperand(0).getReg();
  auto WLSIt = find_if(MRI->use_nodbg_instructions(LR), [](auto &MI) {
    return MI.getOpcode() == ARM::t2WhileLoopStart;
  });
  if (!MergeEndDec || WLSIt == MRI->use_instr_nodbg_end()) {
    RevertWhileLoopSetup(LoopStart, TII);
    RevertLoopDec(LoopStart, TII);
    RevertLoopEnd(LoopStart, TII);
    return true;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 249-255
```cpp
  MachineInstrBuilder MI =
      BuildMI(*WLSIt->getParent(), *WLSIt, WLSIt->getDebugLoc(),
              TII->get(ARM::t2WhileLoopStartLR), LR)
          .add(LoopStart->getOperand(1))
          .add(WLSIt->getOperand(1));
  (void)MI;
  LLVM_DEBUG(dbgs() << "Lowered WhileLoopStart into: " << *MI.getInstr());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 257-260
```cpp
  WLSIt->eraseFromParent();
  LoopStart->eraseFromParent();
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 262-266
```cpp
// Return true if this instruction is invalid in a low overhead loop, usually
// because it clobbers LR.
static bool IsInvalidTPInstruction(MachineInstr &MI) {
  return MI.isCall() || isLoopStart(MI);
}
```
- EN: Implements `IsInvalidTPInstruction`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `IsInvalidTPInstruction`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 268-277
```cpp
// Starting from PreHeader, search for invalid instructions back until the
// LoopStart block is reached. If invalid instructions are found, the loop start
// is reverted from a WhileLoopStart to a DoLoopStart on the same loop. Will
// return the new DLS LoopStart if updated.
MachineInstr *MVETPAndVPTOptimisations::CheckForLRUseInPredecessors(
    MachineBasicBlock *PreHeader, MachineInstr *LoopStart) {
  SmallVector<MachineBasicBlock *> Worklist;
  SmallPtrSet<MachineBasicBlock *, 4> Visited;
  Worklist.push_back(PreHeader);
  Visited.insert(LoopStart->getParent());
```
- EN: Implements `MVETPAndVPTOptimisations::CheckForLRUseInPredecessors`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MVETPAndVPTOptimisations::CheckForLRUseInPredecessors`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 279-282
```cpp
  while (!Worklist.empty()) {
    MachineBasicBlock *MBB = Worklist.pop_back_val();
    if (Visited.count(MBB))
      continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 284-286
```cpp
    for (MachineInstr &MI : *MBB) {
      if (!IsInvalidTPInstruction(MI))
        continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 288-288
```cpp
      LLVM_DEBUG(dbgs() << "Found LR use in predecessors, reverting: " << MI);
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 290-295
```cpp
      // Create a t2DoLoopStart at the end of the preheader.
      MachineInstrBuilder MIB =
          BuildMI(*PreHeader, PreHeader->getFirstTerminator(),
                  LoopStart->getDebugLoc(), TII->get(ARM::t2DoLoopStart));
      MIB.add(LoopStart->getOperand(0));
      MIB.add(LoopStart->getOperand(1));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 297-298
```cpp
      // Make sure to remove the kill flags, to prevent them from being invalid.
      LoopStart->getOperand(1).setIsKill(false);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 300-303
```cpp
      // Revert the t2WhileLoopStartLR to a CMP and Br.
      RevertWhileLoopStartLR(LoopStart, TII, ARM::t2Bcc, true);
      return MIB;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 305-309
```cpp
    Visited.insert(MBB);
    llvm::append_range(Worklist, MBB->predecessors());
  }
  return LoopStart;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 311-320
```cpp
// This function converts loops with t2LoopEnd and t2LoopEnd instructions into
// a single t2LoopEndDec instruction. To do that it needs to make sure that LR
// will be valid to be used for the low overhead loop, which means nothing else
// is using LR (especially calls) and there are no superfluous copies in the
// loop. The t2LoopEndDec is a branching terminator that produces a value (the
// decrement) around the loop edge, which means we need to be careful that they
// will be valid to allocate without any spilling.
bool MVETPAndVPTOptimisations::MergeLoopEnd(MachineLoop *ML) {
  if (!MergeEndDec)
    return false;
```
- EN: Implements `MVETPAndVPTOptimisations::MergeLoopEnd`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MVETPAndVPTOptimisations::MergeLoopEnd`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 322-323
```cpp
  LLVM_DEBUG(dbgs() << "MergeLoopEnd on loop " << ML->getHeader()->getName()
                    << "\n");
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 325-327
```cpp
  MachineInstr *LoopEnd, *LoopPhi, *LoopStart, *LoopDec;
  if (!findLoopComponents(ML, MRI, LoopStart, LoopPhi, LoopDec, LoopEnd))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 329-334
```cpp
  // Check if there is an illegal instruction (a call) in the low overhead loop
  // and if so revert it now before we get any further. While loops also need to
  // check the preheaders, but can be reverted to a DLS loop if needed.
  auto *PreHeader = ML->getLoopPreheader();
  if (LoopStart->getOpcode() == ARM::t2WhileLoopStartLR && PreHeader)
    LoopStart = CheckForLRUseInPredecessors(PreHeader, LoopStart);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 336-349
```cpp
  for (MachineBasicBlock *MBB : ML->blocks()) {
    for (MachineInstr &MI : *MBB) {
      if (IsInvalidTPInstruction(MI)) {
        LLVM_DEBUG(dbgs() << "Found LR use in loop, reverting: " << MI);
        if (LoopStart->getOpcode() == ARM::t2DoLoopStart)
          RevertDoLoopStart(LoopStart, TII);
        else
          RevertWhileLoopStartLR(LoopStart, TII);
        RevertLoopDec(LoopDec, TII);
        RevertLoopEnd(LoopEnd, TII);
        return true;
      }
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 351-365
```cpp
  // Remove any copies from the loop, to ensure the phi that remains is both
  // simpler and contains no extra uses. Because t2LoopEndDec is a terminator
  // that cannot spill, we need to be careful what remains in the loop.
  Register PhiReg = LoopPhi->getOperand(0).getReg();
  Register DecReg = LoopDec->getOperand(0).getReg();
  Register StartReg = LoopStart->getOperand(0).getReg();
  // Ensure the uses are expected, and collect any copies we want to remove.
  SmallVector<MachineInstr *, 4> Copies;
  auto CheckUsers = [&Copies](Register BaseReg,
                              ArrayRef<MachineInstr *> ExpectedUsers,
                              MachineRegisterInfo *MRI) {
    SmallVector<Register, 4> Worklist;
    Worklist.push_back(BaseReg);
    while (!Worklist.empty()) {
      Register Reg = Worklist.pop_back_val();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 366-380
```cpp
      for (MachineInstr &MI : MRI->use_nodbg_instructions(Reg)) {
        if (llvm::is_contained(ExpectedUsers, &MI))
          continue;
        if (MI.getOpcode() != TargetOpcode::COPY ||
            !MI.getOperand(0).getReg().isVirtual()) {
          LLVM_DEBUG(dbgs() << "Extra users of register found: " << MI);
          return false;
        }
        Worklist.push_back(MI.getOperand(0).getReg());
        Copies.push_back(&MI);
      }
    }
    return true;
  };
  if (!CheckUsers(PhiReg, {LoopDec}, MRI) ||
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 381-391
```cpp
      !CheckUsers(DecReg, {LoopPhi, LoopEnd}, MRI) ||
      !CheckUsers(StartReg, {LoopPhi}, MRI)) {
    // Don't leave a t2WhileLoopStartLR without the LoopDecEnd.
    if (LoopStart->getOpcode() == ARM::t2WhileLoopStartLR) {
      RevertWhileLoopStartLR(LoopStart, TII);
      RevertLoopDec(LoopDec, TII);
      RevertLoopEnd(LoopEnd, TII);
      return true;
    }
    return false;
  }
```
- EN: Implements `CheckUsers`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `CheckUsers`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 393-395
```cpp
  MRI->constrainRegClass(StartReg, &ARM::GPRlrRegClass);
  MRI->constrainRegClass(PhiReg, &ARM::GPRlrRegClass);
  MRI->constrainRegClass(DecReg, &ARM::GPRlrRegClass);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 397-403
```cpp
  if (LoopPhi->getOperand(2).getMBB() == ML->getLoopLatch()) {
    LoopPhi->getOperand(3).setReg(StartReg);
    LoopPhi->getOperand(1).setReg(DecReg);
  } else {
    LoopPhi->getOperand(1).setReg(StartReg);
    LoopPhi->getOperand(3).setReg(DecReg);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 405-414
```cpp
  SmallVector<MachineOperand, 4> Cond;              // For analyzeBranch.
  MachineBasicBlock *TBB = nullptr, *FBB = nullptr; // For analyzeBranch.
  if (!TII->analyzeBranch(*LoopEnd->getParent(), TBB, FBB, Cond) && !FBB) {
    // If the LoopEnd falls through, need to insert a t2B to the fall-through
    // block so that the non-analyzable t2LoopEndDec doesn't fall through.
    MachineFunction::iterator MBBI = ++LoopEnd->getParent()->getIterator();
    BuildMI(LoopEnd->getParent(), DebugLoc(), TII->get(ARM::t2B))
        .addMBB(&*MBBI)
        .add(predOps(ARMCC::AL));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 416-423
```cpp
  // Replace the loop dec and loop end as a single instruction.
  MachineInstrBuilder MI =
      BuildMI(*LoopEnd->getParent(), *LoopEnd, LoopEnd->getDebugLoc(),
              TII->get(ARM::t2LoopEndDec), DecReg)
          .addReg(PhiReg)
          .add(LoopEnd->getOperand(1));
  (void)MI;
  LLVM_DEBUG(dbgs() << "Merged LoopDec and End into: " << *MI.getInstr());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 425-430
```cpp
  LoopDec->eraseFromParent();
  LoopEnd->eraseFromParent();
  for (auto *MI : Copies)
    MI->eraseFromParent();
  return true;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 432-439
```cpp
// Convert t2DoLoopStart to t2DoLoopStartTP if the loop contains VCTP
// instructions. This keeps the VCTP count reg operand on the t2DoLoopStartTP
// instruction, making the backend ARMLowOverheadLoops passes job of finding the
// VCTP operand much simpler.
bool MVETPAndVPTOptimisations::ConvertTailPredLoop(MachineLoop *ML,
                                              MachineDominatorTree *DT) {
  LLVM_DEBUG(dbgs() << "ConvertTailPredLoop on loop "
                    << ML->getHeader()->getName() << "\n");
```
- EN: Implements `MVETPAndVPTOptimisations::ConvertTailPredLoop`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MVETPAndVPTOptimisations::ConvertTailPredLoop`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 441-448
```cpp
  // Find some loop components including the LoopEnd/Dec/Start, and any VCTP's
  // in the loop.
  MachineInstr *LoopEnd, *LoopPhi, *LoopStart, *LoopDec;
  if (!findLoopComponents(ML, MRI, LoopStart, LoopPhi, LoopDec, LoopEnd))
    return false;
  if (LoopDec != LoopEnd || (LoopStart->getOpcode() != ARM::t2DoLoopStart &&
                             LoopStart->getOpcode() != ARM::t2WhileLoopStartLR))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 450-458
```cpp
  SmallVector<MachineInstr *, 4> VCTPs;
  SmallVector<MachineInstr *, 4> MVEInstrs;
  for (MachineBasicBlock *BB : ML->blocks()) {
    for (MachineInstr &MI : *BB)
      if (isVCTP(&MI))
        VCTPs.push_back(&MI);
      else if (findFirstVPTPredOperandIdx(MI) != -1)
        MVEInstrs.push_back(&MI);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 460-463
```cpp
  if (VCTPs.empty()) {
    LLVM_DEBUG(dbgs() << "  no VCTPs\n");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 465-474
```cpp
  // Check all VCTPs are the same.
  MachineInstr *FirstVCTP = *VCTPs.begin();
  for (MachineInstr *VCTP : VCTPs) {
    LLVM_DEBUG(dbgs() << "  with VCTP " << *VCTP);
    if (VCTP->getOpcode() != FirstVCTP->getOpcode() ||
        VCTP->getOperand(0).getReg() != FirstVCTP->getOperand(0).getReg()) {
      LLVM_DEBUG(dbgs() << "  VCTP's are not identical\n");
      return false;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 476-490
```cpp
  // Check for the register being used can be setup before the loop. We expect
  // this to be:
  //   $vx = ...
  // loop:
  //   $vp = PHI [ $vx ], [ $vd ]
  //   ..
  //   $vpr = VCTP $vp
  //   ..
  //   $vd = t2SUBri $vp, #n
  //   ..
  Register CountReg = FirstVCTP->getOperand(1).getReg();
  if (!CountReg.isVirtual()) {
    LLVM_DEBUG(dbgs() << "  cannot determine VCTP PHI\n");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 491-501
```cpp
  MachineInstr *Phi = LookThroughCOPY(MRI->getVRegDef(CountReg), MRI);
  if (!Phi || Phi->getOpcode() != TargetOpcode::PHI ||
      Phi->getNumOperands() != 5 ||
      (Phi->getOperand(2).getMBB() != ML->getLoopLatch() &&
       Phi->getOperand(4).getMBB() != ML->getLoopLatch())) {
    LLVM_DEBUG(dbgs() << "  cannot determine VCTP Count\n");
    return false;
  }
  CountReg = Phi->getOperand(2).getMBB() == ML->getLoopLatch()
                 ? Phi->getOperand(3).getReg()
                 : Phi->getOperand(1).getReg();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 503-514
```cpp
  // Replace the t2DoLoopStart with the t2DoLoopStartTP, move it to the end of
  // the preheader and add the new CountReg to it. We attempt to place it late
  // in the preheader, but may need to move that earlier based on uses.
  MachineBasicBlock *MBB = LoopStart->getParent();
  MachineBasicBlock::iterator InsertPt = MBB->getFirstTerminator();
  for (MachineInstr &Use :
       MRI->use_instructions(LoopStart->getOperand(0).getReg()))
    if ((InsertPt != MBB->end() && !DT->dominates(&*InsertPt, &Use)) ||
        !DT->dominates(ML->getHeader(), Use.getParent())) {
      LLVM_DEBUG(dbgs() << "  InsertPt could not be a terminator!\n");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 516-529
```cpp
  unsigned NewOpc = LoopStart->getOpcode() == ARM::t2DoLoopStart
                        ? ARM::t2DoLoopStartTP
                        : ARM::t2WhileLoopStartTP;
  MachineInstrBuilder MI =
      BuildMI(*MBB, InsertPt, LoopStart->getDebugLoc(), TII->get(NewOpc))
          .add(LoopStart->getOperand(0))
          .add(LoopStart->getOperand(1))
          .addReg(CountReg);
  if (NewOpc == ARM::t2WhileLoopStartTP)
    MI.add(LoopStart->getOperand(2));
  LLVM_DEBUG(dbgs() << "Replacing " << *LoopStart << "  with "
                    << *MI.getInstr());
  MRI->constrainRegClass(CountReg, &ARM::rGPRRegClass);
  LoopStart->eraseFromParent();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 531-539
```cpp
  if (SetLRPredicate) {
    // Each instruction in the loop needs to be using LR as the predicate from
    // the Phi as the predicate.
    Register LR = LoopPhi->getOperand(0).getReg();
    for (MachineInstr *MI : MVEInstrs) {
      int Idx = findFirstVPTPredOperandIdx(*MI);
      MI->getOperand(Idx + ARM::SUBOP_vpred_n_tp_reg).setReg(LR);
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 541-542
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 544-545
```cpp
// Returns true if Opcode is any VCMP Opcode.
static bool IsVCMP(unsigned Opcode) { return VCMPOpcodeToVPT(Opcode) != 0; }
```
- EN: Implements `IsVCMP`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `IsVCMP`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 547-561
```cpp
// Returns true if a VCMP with this Opcode can have its operands swapped.
// There is 2 kind of VCMP that can't have their operands swapped: Float VCMPs,
// and VCMPr instructions (since the r is always on the right).
static bool CanHaveSwappedOperands(unsigned Opcode) {
  switch (Opcode) {
  default:
    return true;
  case ARM::MVE_VCMPf32:
  case ARM::MVE_VCMPf16:
  case ARM::MVE_VCMPf32r:
  case ARM::MVE_VCMPf16r:
  case ARM::MVE_VCMPi8r:
  case ARM::MVE_VCMPi16r:
  case ARM::MVE_VCMPi32r:
  case ARM::MVE_VCMPu8r:
```
- EN: Implements `CanHaveSwappedOperands`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `CanHaveSwappedOperands`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 562-569
```cpp
  case ARM::MVE_VCMPu16r:
  case ARM::MVE_VCMPu32r:
  case ARM::MVE_VCMPs8r:
  case ARM::MVE_VCMPs16r:
  case ARM::MVE_VCMPs32r:
    return false;
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 571-575
```cpp
// Returns the CondCode of a VCMP Instruction.
static ARMCC::CondCodes GetCondCode(MachineInstr &Instr) {
  assert(IsVCMP(Instr.getOpcode()) && "Inst must be a VCMP");
  return ARMCC::CondCodes(Instr.getOperand(3).getImm());
}
```
- EN: Implements `GetCondCode`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `GetCondCode`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 577-580
```cpp
// Returns true if Cond is equivalent to a VPNOT instruction on the result of
// Prev. Cond and Prev must be VCMPs.
static bool IsVPNOTEquivalent(MachineInstr &Cond, MachineInstr &Prev) {
  assert(IsVCMP(Cond.getOpcode()) && IsVCMP(Prev.getOpcode()));
```
- EN: Implements `IsVPNOTEquivalent`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `IsVPNOTEquivalent`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 582-584
```cpp
  // Opcodes must match.
  if (Cond.getOpcode() != Prev.getOpcode())
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 586-587
```cpp
  MachineOperand &CondOP1 = Cond.getOperand(1), &CondOP2 = Cond.getOperand(2);
  MachineOperand &PrevOP1 = Prev.getOperand(1), &PrevOP2 = Prev.getOperand(2);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 589-602
```cpp
  // If the VCMP has the opposite condition with the same operands, we can
  // replace it with a VPNOT
  ARMCC::CondCodes ExpectedCode = GetCondCode(Cond);
  ExpectedCode = ARMCC::getOppositeCondition(ExpectedCode);
  if (ExpectedCode == GetCondCode(Prev))
    if (CondOP1.isIdenticalTo(PrevOP1) && CondOP2.isIdenticalTo(PrevOP2))
      return true;
  // Check again with operands swapped if possible
  if (!CanHaveSwappedOperands(Cond.getOpcode()))
    return false;
  ExpectedCode = ARMCC::getSwappedCondition(ExpectedCode);
  return ExpectedCode == GetCondCode(Prev) && CondOP1.isIdenticalTo(PrevOP2) &&
         CondOP2.isIdenticalTo(PrevOP1);
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 604-617
```cpp
// Returns true if Instr writes to VCCR.
static bool IsWritingToVCCR(MachineInstr &Instr) {
  if (Instr.getNumOperands() == 0)
    return false;
  MachineOperand &Dst = Instr.getOperand(0);
  if (!Dst.isReg())
    return false;
  Register DstReg = Dst.getReg();
  if (!DstReg.isVirtual())
    return false;
  MachineRegisterInfo &RegInfo = Instr.getMF()->getRegInfo();
  const TargetRegisterClass *RegClass = RegInfo.getRegClassOrNull(DstReg);
  return RegClass && (RegClass->getID() == ARM::VCCRRegClassID);
}
```
- EN: Implements `IsWritingToVCCR`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `IsWritingToVCCR`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 619-630
```cpp
// Transforms
//    <Instr that uses %A ('User' Operand)>
// Into
//    %K = VPNOT %Target
//    <Instr that uses %K ('User' Operand)>
// And returns the newly inserted VPNOT.
// This optimization is done in the hopes of preventing spills/reloads of VPR by
// reducing the number of VCCR values with overlapping lifetimes.
MachineInstr &MVETPAndVPTOptimisations::ReplaceRegisterUseWithVPNOT(
    MachineBasicBlock &MBB, MachineInstr &Instr, MachineOperand &User,
    Register Target) {
  Register NewResult = MRI->createVirtualRegister(MRI->getRegClass(Target));
```
- EN: Implements `MVETPAndVPTOptimisations::ReplaceRegisterUseWithVPNOT`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MVETPAndVPTOptimisations::ReplaceRegisterUseWithVPNOT`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 632-636
```cpp
  MachineInstrBuilder MIBuilder =
      BuildMI(MBB, &Instr, Instr.getDebugLoc(), TII->get(ARM::MVE_VPNOT))
          .addDef(NewResult)
          .addReg(Target);
  addUnpredicatedMveVpredNOp(MIBuilder);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 638-640
```cpp
  // Make the user use NewResult instead, and clear its kill flag.
  User.setReg(NewResult);
  User.setIsKill(false);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 642-643
```cpp
  LLVM_DEBUG(dbgs() << "  Inserting VPNOT (for spill prevention): ";
             MIBuilder.getInstr()->dump());
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 645-646
```cpp
  return *MIBuilder.getInstr();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 648-656
```cpp
// Moves a VPNOT before its first user if an instruction that uses Reg is found
// in-between the VPNOT and its user.
// Returns true if there is at least one user of the VPNOT in the block.
static bool MoveVPNOTBeforeFirstUser(MachineBasicBlock &MBB,
                                     MachineBasicBlock::iterator Iter,
                                     Register Reg) {
  assert(Iter->getOpcode() == ARM::MVE_VPNOT && "Not a VPNOT!");
  assert(getVPTInstrPredicate(*Iter) == ARMVCC::None &&
         "The VPNOT cannot be predicated");
```
- EN: Implements `MoveVPNOTBeforeFirstUser`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MoveVPNOTBeforeFirstUser`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 658-660
```cpp
  MachineInstr &VPNOT = *Iter;
  Register VPNOTResult = VPNOT.getOperand(0).getReg();
  Register VPNOTOperand = VPNOT.getOperand(1).getReg();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 662-672
```cpp
  // Whether the VPNOT will need to be moved, and whether we found a user of the
  // VPNOT.
  bool MustMove = false, HasUser = false;
  MachineOperand *VPNOTOperandKiller = nullptr;
  for (; Iter != MBB.end(); ++Iter) {
    if (MachineOperand *MO =
            Iter->findRegisterUseOperand(VPNOTOperand, /*TRI=*/nullptr,
                                         /*isKill*/ true)) {
      // If we find the operand that kills the VPNOTOperand's result, save it.
      VPNOTOperandKiller = MO;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 674-677
```cpp
    if (Iter->findRegisterUseOperandIdx(Reg, /*TRI=*/nullptr) != -1) {
      MustMove = true;
      continue;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 679-680
```cpp
    if (Iter->findRegisterUseOperandIdx(VPNOTResult, /*TRI=*/nullptr) == -1)
      continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 682-684
```cpp
    HasUser = true;
    if (!MustMove)
      break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 686-693
```cpp
    // Move the VPNOT right before Iter
    LLVM_DEBUG(dbgs() << "Moving: "; VPNOT.dump(); dbgs() << "  Before: ";
               Iter->dump());
    MBB.splice(Iter, &MBB, VPNOT.getIterator());
    // If we move the instr, and its operand was killed earlier, remove the kill
    // flag.
    if (VPNOTOperandKiller)
      VPNOTOperandKiller->setIsKill(false);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 695-698
```cpp
    break;
  }
  return HasUser;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 700-714
```cpp
// This optimisation attempts to reduce the number of overlapping lifetimes of
// VCCR values by replacing uses of old VCCR values with VPNOTs. For example,
// this replaces
//    %A:vccr = (something)
//    %B:vccr = VPNOT %A
//    %Foo = (some op that uses %B)
//    %Bar = (some op that uses %A)
// With
//    %A:vccr = (something)
//    %B:vccr = VPNOT %A
//    %Foo = (some op that uses %B)
//    %TMP2:vccr = VPNOT %B
//    %Bar = (some op that uses %A)
bool MVETPAndVPTOptimisations::ReduceOldVCCRValueUses(MachineBasicBlock &MBB) {
  MachineBasicBlock::iterator Iter = MBB.begin(), End = MBB.end();
```
- EN: Implements `MVETPAndVPTOptimisations::ReduceOldVCCRValueUses`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MVETPAndVPTOptimisations::ReduceOldVCCRValueUses`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 715-716
```cpp
  SmallVector<MachineInstr *, 4> DeadInstructions;
  bool Modified = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 718-728
```cpp
  while (Iter != End) {
    Register VCCRValue, OppositeVCCRValue;
    // The first loop looks for 2 unpredicated instructions:
    //    %A:vccr = (instr)     ; A is stored in VCCRValue
    //    %B:vccr = VPNOT %A    ; B is stored in OppositeVCCRValue
    for (; Iter != End; ++Iter) {
      // We're only interested in unpredicated instructions that write to VCCR.
      if (!IsWritingToVCCR(*Iter) ||
          getVPTInstrPredicate(*Iter) != ARMVCC::None)
        continue;
      Register Dst = Iter->getOperand(0).getReg();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 730-737
```cpp
      // If we already have a VCCRValue, and this is a VPNOT on VCCRValue, we've
      // found what we were looking for.
      if (VCCRValue && Iter->getOpcode() == ARM::MVE_VPNOT &&
          Iter->findRegisterUseOperandIdx(VCCRValue, /*TRI=*/nullptr) != -1) {
        // Move the VPNOT closer to its first user if needed, and ignore if it
        // has no users.
        if (!MoveVPNOTBeforeFirstUser(MBB, Iter, VCCRValue))
          continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 739-742
```cpp
        OppositeVCCRValue = Dst;
        ++Iter;
        break;
      }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 744-746
```cpp
      // Else, just set VCCRValue.
      VCCRValue = Dst;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 748-750
```cpp
    // If the first inner loop didn't find anything, stop here.
    if (Iter == End)
      break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 752-756
```cpp
    assert(VCCRValue && OppositeVCCRValue &&
           "VCCRValue and OppositeVCCRValue shouldn't be empty if the loop "
           "stopped before the end of the block!");
    assert(VCCRValue != OppositeVCCRValue &&
           "VCCRValue should not be equal to OppositeVCCRValue!");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 758-759
```cpp
    // LastVPNOTResult always contains the same value as OppositeVCCRValue.
    Register LastVPNOTResult = OppositeVCCRValue;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 761-763
```cpp
    // This second loop tries to optimize the remaining instructions.
    for (; Iter != End; ++Iter) {
      bool IsInteresting = false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 765-767
```cpp
      if (MachineOperand *MO =
              Iter->findRegisterUseOperand(VCCRValue, /*TRI=*/nullptr)) {
        IsInteresting = true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 769-773
```cpp
        // - If the instruction is a VPNOT, it can be removed, and we can just
        //   replace its uses with LastVPNOTResult.
        // - Else, insert a new VPNOT on LastVPNOTResult to recompute VCCRValue.
        if (Iter->getOpcode() == ARM::MVE_VPNOT) {
          Register Result = Iter->getOperand(0).getReg();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 775-777
```cpp
          MRI->replaceRegWith(Result, LastVPNOTResult);
          DeadInstructions.push_back(&*Iter);
          Modified = true;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 779-785
```cpp
          LLVM_DEBUG(dbgs()
                     << "Replacing all uses of '" << printReg(Result)
                     << "' with '" << printReg(LastVPNOTResult) << "'\n");
        } else {
          MachineInstr &VPNOT =
              ReplaceRegisterUseWithVPNOT(MBB, *Iter, *MO, LastVPNOTResult);
          Modified = true;
```
- EN: Implements `LLVM_DEBUG`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `LLVM_DEBUG`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 787-788
```cpp
          LastVPNOTResult = VPNOT.getOperand(0).getReg();
          std::swap(VCCRValue, OppositeVCCRValue);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 790-799
```cpp
          LLVM_DEBUG(dbgs() << "Replacing use of '" << printReg(VCCRValue)
                            << "' with '" << printReg(LastVPNOTResult)
                            << "' in instr: " << *Iter);
        }
      } else {
        // If the instr uses OppositeVCCRValue, make it use LastVPNOTResult
        // instead as they contain the same value.
        if (MachineOperand *MO = Iter->findRegisterUseOperand(
                OppositeVCCRValue, /*TRI=*/nullptr)) {
          IsInteresting = true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 801-809
```cpp
          // This is pointless if LastVPNOTResult == OppositeVCCRValue.
          if (LastVPNOTResult != OppositeVCCRValue) {
            LLVM_DEBUG(dbgs() << "Replacing usage of '"
                              << printReg(OppositeVCCRValue) << "' with '"
                              << printReg(LastVPNOTResult) << " for instr: ";
                       Iter->dump());
            MO->setReg(LastVPNOTResult);
            Modified = true;
          }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 811-812
```cpp
          MO->setIsKill(false);
        }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 814-821
```cpp
        // If this is an unpredicated VPNOT on
        // LastVPNOTResult/OppositeVCCRValue, we can act like we inserted it.
        if (Iter->getOpcode() == ARM::MVE_VPNOT &&
            getVPTInstrPredicate(*Iter) == ARMVCC::None) {
          Register VPNOTOperand = Iter->getOperand(1).getReg();
          if (VPNOTOperand == LastVPNOTResult ||
              VPNOTOperand == OppositeVCCRValue) {
            IsInteresting = true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 823-827
```cpp
            std::swap(VCCRValue, OppositeVCCRValue);
            LastVPNOTResult = Iter->getOperand(0).getReg();
          }
        }
      }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 829-833
```cpp
      // If this instruction was not interesting, and it writes to VCCR, stop.
      if (!IsInteresting && IsWritingToVCCR(*Iter))
        break;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 835-836
```cpp
  for (MachineInstr *DeadInstruction : DeadInstructions)
    DeadInstruction->eraseFromParent();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 838-839
```cpp
  return Modified;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 841-843
```cpp
// This optimisation replaces VCMPs with VPNOTs when they are equivalent.
bool MVETPAndVPTOptimisations::ReplaceVCMPsByVPNOTs(MachineBasicBlock &MBB) {
  SmallVector<MachineInstr *, 4> DeadInstructions;
```
- EN: Implements `MVETPAndVPTOptimisations::ReplaceVCMPsByVPNOTs`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MVETPAndVPTOptimisations::ReplaceVCMPsByVPNOTs`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 845-854
```cpp
  // The last VCMP that we have seen and that couldn't be replaced.
  // This is reset when an instruction that writes to VCCR/VPR is found, or when
  // a VCMP is replaced with a VPNOT.
  // We'll only replace VCMPs with VPNOTs when this is not null, and when the
  // current VCMP is the opposite of PrevVCMP.
  MachineInstr *PrevVCMP = nullptr;
  // If we find an instruction that kills the result of PrevVCMP, we save the
  // operand here to remove the kill flag in case we need to use PrevVCMP's
  // result.
  MachineOperand *PrevVCMPResultKiller = nullptr;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 856-865
```cpp
  for (MachineInstr &Instr : MBB.instrs()) {
    if (PrevVCMP) {
      if (MachineOperand *MO =
              Instr.findRegisterUseOperand(PrevVCMP->getOperand(0).getReg(),
                                           /*TRI=*/nullptr, /*isKill*/ true)) {
        // If we come across the instr that kills PrevVCMP's result, record it
        // so we can remove the kill flag later if we need to.
        PrevVCMPResultKiller = MO;
      }
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 867-869
```cpp
    // Ignore predicated instructions.
    if (getVPTInstrPredicate(Instr) != ARMVCC::None)
      continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 871-877
```cpp
    // Only look at VCMPs
    if (!IsVCMP(Instr.getOpcode())) {
      // If the instruction writes to VCCR, forget the previous VCMP.
      if (IsWritingToVCCR(Instr))
        PrevVCMP = nullptr;
      continue;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 879-882
```cpp
    if (!PrevVCMP || !IsVPNOTEquivalent(Instr, *PrevVCMP)) {
      PrevVCMP = &Instr;
      continue;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 884-886
```cpp
    // The register containing the result of the VCMP that we're going to
    // replace.
    Register PrevVCMPResultReg = PrevVCMP->getOperand(0).getReg();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 888-896
```cpp
    // Build a VPNOT to replace the VCMP, reusing its operands.
    MachineInstrBuilder MIBuilder =
        BuildMI(MBB, &Instr, Instr.getDebugLoc(), TII->get(ARM::MVE_VPNOT))
            .add(Instr.getOperand(0))
            .addReg(PrevVCMPResultReg);
    addUnpredicatedMveVpredNOp(MIBuilder);
    LLVM_DEBUG(dbgs() << "Inserting VPNOT (to replace VCMP): ";
               MIBuilder.getInstr()->dump(); dbgs() << "  Removed VCMP: ";
               Instr.dump());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 898-901
```cpp
    // If we found an instruction that uses, and kills PrevVCMP's result,
    // remove the kill flag.
    if (PrevVCMPResultKiller)
      PrevVCMPResultKiller->setIsKill(false);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 903-908
```cpp
    // Finally, mark the old VCMP for removal and reset
    // PrevVCMP/PrevVCMPResultKiller.
    DeadInstructions.push_back(&Instr);
    PrevVCMP = nullptr;
    PrevVCMPResultKiller = nullptr;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 910-911
```cpp
  for (MachineInstr *DeadInstruction : DeadInstructions)
    DeadInstruction->eraseFromParent();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 913-914
```cpp
  return !DeadInstructions.empty();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 916-925
```cpp
bool MVETPAndVPTOptimisations::ReplaceConstByVPNOTs(MachineBasicBlock &MBB,
                                                    MachineDominatorTree *DT) {
  // Scan through the block, looking for instructions that use constants moves
  // into VPR that are the negative of one another. These are expected to be
  // COPY's to VCCRRegClass, from a t2MOVi or t2MOVi16. The last seen constant
  // mask is kept it or and VPNOT's of it are added or reused as we scan through
  // the function.
  unsigned LastVPTImm = 0;
  Register LastVPTReg = 0;
  SmallPtrSet<MachineInstr *, 4> DeadInstructions;
```
- EN: Implements `MVETPAndVPTOptimisations::ReplaceConstByVPNOTs`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MVETPAndVPTOptimisations::ReplaceConstByVPNOTs`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 927-934
```cpp
  for (MachineInstr &Instr : MBB.instrs()) {
    // Look for predicated MVE instructions.
    int PIdx = llvm::findFirstVPTPredOperandIdx(Instr);
    if (PIdx == -1)
      continue;
    Register VPR = Instr.getOperand(PIdx + 1).getReg();
    if (!VPR.isVirtual())
      continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 936-944
```cpp
    // From that we are looking for an instruction like %11:vccr = COPY %9:rgpr.
    MachineInstr *Copy = MRI->getVRegDef(VPR);
    if (!Copy || Copy->getOpcode() != TargetOpcode::COPY ||
        !Copy->getOperand(1).getReg().isVirtual() ||
        MRI->getRegClass(Copy->getOperand(1).getReg()) == &ARM::VCCRRegClass) {
      LastVPTReg = 0;
      continue;
    }
    Register GPR = Copy->getOperand(1).getReg();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 946-958
```cpp
    // Find the Immediate used by the copy.
    auto getImm = [&](Register GPR) -> unsigned {
      MachineInstr *Def = MRI->getVRegDef(GPR);
      if (Def && (Def->getOpcode() == ARM::t2MOVi ||
                  Def->getOpcode() == ARM::t2MOVi16))
        return Def->getOperand(1).getImm();
      return -1U;
    };
    unsigned Imm = getImm(GPR);
    if (Imm == -1U) {
      LastVPTReg = 0;
      continue;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 960-974
```cpp
    unsigned NotImm = ~Imm & 0xffff;
    if (LastVPTReg != 0 && LastVPTReg != VPR && LastVPTImm == Imm) {
      MRI->clearKillFlags(LastVPTReg);
      Instr.getOperand(PIdx + 1).setReg(LastVPTReg);
      if (MRI->use_empty(VPR)) {
        DeadInstructions.insert(Copy);
        if (MRI->hasOneUse(GPR))
          DeadInstructions.insert(MRI->getVRegDef(GPR));
      }
      LLVM_DEBUG(dbgs() << "Reusing predicate: in  " << Instr);
      VPR = LastVPTReg;
    } else if (LastVPTReg != 0 && LastVPTImm == NotImm) {
      // We have found the not of a previous constant. Create a VPNot of the
      // earlier predicate reg and use it instead of the copy.
      Register NewVPR = MRI->createVirtualRegister(&ARM::VCCRRegClass);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 975-978
```cpp
      auto VPNot = BuildMI(MBB, &Instr, Instr.getDebugLoc(),
                           TII->get(ARM::MVE_VPNOT), NewVPR)
                       .addReg(LastVPTReg);
      addUnpredicatedMveVpredNOp(VPNot);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 980-991
```cpp
      // Use the new register and check if the def is now dead.
      Instr.getOperand(PIdx + 1).setReg(NewVPR);
      if (MRI->use_empty(VPR)) {
        DeadInstructions.insert(Copy);
        if (MRI->hasOneUse(GPR))
          DeadInstructions.insert(MRI->getVRegDef(GPR));
      }
      MRI->clearKillFlags(LastVPTReg);
      LLVM_DEBUG(dbgs() << "Adding VPNot: " << *VPNot << "  to replace use at "
                        << Instr);
      VPR = NewVPR;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 993-995
```cpp
    LastVPTImm = Imm;
    LastVPTReg = VPR;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 997-998
```cpp
  for (MachineInstr *DI : DeadInstructions)
    DI->eraseFromParent();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1000-1001
```cpp
  return !DeadInstructions.empty();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1003-1010
```cpp
// Replace VPSEL with a predicated VMOV in blocks with a VCTP. This is a
// somewhat blunt approximation to allow tail predicated with vpsel
// instructions. We turn a vselect into a VPSEL in ISEL, but they have slightly
// different semantics under tail predication. Until that is modelled we just
// convert to a VMOVT (via a predicated VORR) instead.
bool MVETPAndVPTOptimisations::ConvertVPSEL(MachineBasicBlock &MBB) {
  bool HasVCTP = false;
  SmallVector<MachineInstr *, 4> DeadInstructions;
```
- EN: Implements `MVETPAndVPTOptimisations::ConvertVPSEL`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MVETPAndVPTOptimisations::ConvertVPSEL`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1012-1016
```cpp
  for (MachineInstr &MI : MBB.instrs()) {
    if (isVCTP(&MI)) {
      HasVCTP = true;
      continue;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1018-1019
```cpp
    if (!HasVCTP || MI.getOpcode() != ARM::MVE_VPSEL)
      continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1021-1035
```cpp
    MachineInstrBuilder MIBuilder =
        BuildMI(MBB, &MI, MI.getDebugLoc(), TII->get(ARM::MVE_VORR))
            .add(MI.getOperand(0))
            .add(MI.getOperand(1))
            .add(MI.getOperand(1))
            .addImm(ARMVCC::Then)
            .add(MI.getOperand(4))
            .add(MI.getOperand(5))
            .add(MI.getOperand(2));
    // Silence unused variable warning in release builds.
    (void)MIBuilder;
    LLVM_DEBUG(dbgs() << "Replacing VPSEL: "; MI.dump();
               dbgs() << "     with VMOVT: "; MIBuilder.getInstr()->dump());
    DeadInstructions.push_back(&MI);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1037-1038
```cpp
  for (MachineInstr *DeadInstruction : DeadInstructions)
    DeadInstruction->eraseFromParent();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1040-1041
```cpp
  return !DeadInstructions.empty();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1043-1056
```cpp
// Add a registry allocation hint for t2DoLoopStart to hint it towards LR, as
// the instruction may be removable as a noop.
bool MVETPAndVPTOptimisations::HintDoLoopStartReg(MachineBasicBlock &MBB) {
  bool Changed = false;
  for (MachineInstr &MI : MBB.instrs()) {
    if (MI.getOpcode() != ARM::t2DoLoopStart)
      continue;
    Register R = MI.getOperand(1).getReg();
    MachineFunction *MF = MI.getParent()->getParent();
    MF->getRegInfo().setRegAllocationHint(R, ARMRI::RegLR, 0);
    Changed = true;
  }
  return Changed;
}
```
- EN: Implements `MVETPAndVPTOptimisations::HintDoLoopStartReg`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MVETPAndVPTOptimisations::HintDoLoopStartReg`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1058-1059
```cpp
bool MVETPAndVPTOptimisations::runOnMachineFunction(MachineFunction &Fn) {
  const ARMSubtarget &STI = Fn.getSubtarget<ARMSubtarget>();
```
- EN: Implements `MVETPAndVPTOptimisations::runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MVETPAndVPTOptimisations::runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1061-1062
```cpp
  if (!STI.isThumb2() || !STI.hasLOB())
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1064-1068
```cpp
  TII = static_cast<const Thumb2InstrInfo *>(STI.getInstrInfo());
  MRI = &Fn.getRegInfo();
  MachineLoopInfo *MLI = &getAnalysis<MachineLoopInfoWrapperPass>().getLI();
  MachineDominatorTree *DT =
      &getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1070-1071
```cpp
  LLVM_DEBUG(dbgs() << "********** ARM MVE VPT Optimisations **********\n"
                    << "********** Function: " << Fn.getName() << '\n');
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1073-1078
```cpp
  bool Modified = false;
  for (MachineLoop *ML : MLI->getLoopsInPreorder()) {
    Modified |= LowerWhileLoopStart(ML);
    Modified |= MergeLoopEnd(ML);
    Modified |= ConvertTailPredLoop(ML, DT);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1080-1086
```cpp
  for (MachineBasicBlock &MBB : Fn) {
    Modified |= HintDoLoopStartReg(MBB);
    Modified |= ReplaceConstByVPNOTs(MBB, DT);
    Modified |= ReplaceVCMPsByVPNOTs(MBB);
    Modified |= ReduceOldVCCRValueUses(MBB);
    Modified |= ConvertVPSEL(MBB);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1088-1090
```cpp
  LLVM_DEBUG(dbgs() << "**************************************\n");
  return Modified;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1092-1095
```cpp
/// createMVETPAndVPTOptimisationsPass
FunctionPass *llvm::createMVETPAndVPTOptimisationsPass() {
  return new MVETPAndVPTOptimisations();
}
```
- EN: Implements `llvm::createMVETPAndVPTOptimisationsPass`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::createMVETPAndVPTOptimisationsPass`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

## Key Concepts / 关键概念

- EN: Primary role: target-specific code-generation behavior.
  - CN: 核心职责：目标相关代码生成行为。
- EN: MachineInstr-level manipulation is central to this file.
  - CN: 该文件以 MachineInstr 层面的操作为核心。
- EN: MachineBasicBlock flow and branch structure matter to the implementation.
  - CN: 实现高度依赖 MachineBasicBlock 流程与分支结构。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。
- EN: The component plugs into LLVM machine-function passes or codegen pipelines.
  - CN: 该组件会接入 LLVM 的机器函数 Pass 或代码生成流水线。

## Dependencies / 依赖关系

- EN: Backend-local headers: `ARM.h`, `ARMSubtarget.h`, `MVETailPredUtils.h`, `Thumb2InstrInfo.h`.
  - CN: 后端本地头文件：`ARM.h`, `ARMSubtarget.h`, `MVETailPredUtils.h`, `Thumb2InstrInfo.h`。
- EN: LLVM infrastructure headers: `llvm/ADT/SmallVector.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineDominators.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/MachineLoopInfo.h`, `llvm/InitializePasses.h` ... (+1 more).
  - CN: LLVM 基础设施头文件：`llvm/ADT/SmallVector.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineDominators.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/MachineLoopInfo.h`, `llvm/InitializePasses.h` ... (+1 more)。
- EN: Standard/system headers: `cassert`.
  - CN: 标准库/系统头文件：`cassert`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
