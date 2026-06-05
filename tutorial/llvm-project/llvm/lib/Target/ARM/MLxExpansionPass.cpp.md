# MLxExpansionPass.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/MLxExpansionPass.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `MLxExpansionPass` for the ARM backend, focusing on target-specific code-generation behavior.
- 用途 (CN): 实现 ARM 后端中的 `MLxExpansionPass`，重点处理目标相关代码生成行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===-- MLxExpansionPass.cpp - Expand MLx instrs to avoid hazards ---------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Expand VFP / NEON floating point MLA / MLS instructions (each to a pair of
// multiple and add / sub instructions) when special VMLx hazards are detected.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 14-27
```cpp
#include "ARM.h"
#include "ARMBaseInstrInfo.h"
#include "ARMSubtarget.h"
#include "llvm/ADT/SmallPtrSet.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/TargetRegisterInfo.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 29-29
```cpp
#define DEBUG_TYPE "mlx-expansion"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 31-34
```cpp
static cl::opt<bool>
ForceExpand("expand-all-fp-mlx", cl::init(false), cl::Hidden);
static cl::opt<unsigned>
ExpandLimit("expand-limit", cl::init(~0U), cl::Hidden);
```
- EN: Declares `cl::init`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::init`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 36-36
```cpp
STATISTIC(NumExpand, "Number of fp MLA / MLS instructions expanded");
```
- EN: Declares `STATISTIC`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `STATISTIC`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 38-41
```cpp
namespace {
  struct MLxExpansion : public MachineFunctionPass {
    static char ID;
    MLxExpansion() : MachineFunctionPass(ID) {}
```
- EN: Declares `MLxExpansion`, packaging target-specific state and APIs around `MLxExpansionPass`.
- CN: 这里声明 `MLxExpansion`，把与 `MLxExpansionPass` 相关的目标特定状态和 API 组织在一起。

### Lines 43-43
```cpp
    bool runOnMachineFunction(MachineFunction &Fn) override;
```
- EN: Declares `runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 45-47
```cpp
    StringRef getPassName() const override {
      return "ARM MLA / MLS expansion pass";
    }
```
- EN: Implements `getPassName`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getPassName`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 49-52
```cpp
  private:
    const ARMBaseInstrInfo *TII;
    const TargetRegisterInfo *TRI;
    MachineRegisterInfo *MRI;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 54-58
```cpp
    bool isLikeA9;
    bool isSwift;
    unsigned MIIdx;
    MachineInstr* LastMIs[4];
    SmallPtrSet<MachineInstr*, 4> IgnoreStall;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 60-73
```cpp
    void clearStack();
    void pushStack(MachineInstr *MI);
    MachineInstr *getAccDefMI(MachineInstr *MI) const;
    unsigned getDefReg(MachineInstr *MI) const;
    bool hasLoopHazard(MachineInstr *MI) const;
    bool hasRAWHazard(unsigned Reg, MachineInstr *MI) const;
    bool FindMLxHazard(MachineInstr *MI);
    void ExpandFPMLxInstruction(MachineBasicBlock &MBB, MachineInstr *MI,
                                unsigned MulOpc, unsigned AddSubOpc,
                                bool NegAcc, bool HasLane);
    bool ExpandFPMLxInstructions(MachineBasicBlock &MBB);
  };
  char MLxExpansion::ID = 0;
}
```
- EN: Declares `clearStack`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `clearStack`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 75-78
```cpp
void MLxExpansion::clearStack() {
  std::fill(LastMIs, LastMIs + 4, nullptr);
  MIIdx = 0;
}
```
- EN: Implements `MLxExpansion::clearStack`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MLxExpansion::clearStack`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 80-84
```cpp
void MLxExpansion::pushStack(MachineInstr *MI) {
  LastMIs[MIIdx] = MI;
  if (++MIIdx == 4)
    MIIdx = 0;
}
```
- EN: Implements `MLxExpansion::pushStack`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MLxExpansion::pushStack`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 86-91
```cpp
MachineInstr *MLxExpansion::getAccDefMI(MachineInstr *MI) const {
  // Look past COPY and INSERT_SUBREG instructions to find the
  // real definition MI. This is important for _sfp instructions.
  Register Reg = MI->getOperand(1).getReg();
  if (Reg.isPhysical())
    return nullptr;
```
- EN: Implements `MLxExpansion::getAccDefMI`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MLxExpansion::getAccDefMI`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 93-107
```cpp
  MachineBasicBlock *MBB = MI->getParent();
  MachineInstr *DefMI = MRI->getVRegDef(Reg);
  while (true) {
    if (DefMI->getParent() != MBB)
      break;
    if (DefMI->isCopyLike()) {
      Reg = DefMI->getOperand(1).getReg();
      if (Reg.isVirtual()) {
        DefMI = MRI->getVRegDef(Reg);
        continue;
      }
    } else if (DefMI->isInsertSubreg()) {
      Reg = DefMI->getOperand(2).getReg();
      if (Reg.isVirtual()) {
        DefMI = MRI->getVRegDef(Reg);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 108-114
```cpp
        continue;
      }
    }
    break;
  }
  return DefMI;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 116-119
```cpp
unsigned MLxExpansion::getDefReg(MachineInstr *MI) const {
  Register Reg = MI->getOperand(0).getReg();
  if (Reg.isPhysical() || !MRI->hasOneNonDBGUse(Reg))
    return Reg;
```
- EN: Implements `MLxExpansion::getDefReg`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MLxExpansion::getDefReg`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 121-124
```cpp
  MachineBasicBlock *MBB = MI->getParent();
  MachineInstr *UseMI = &*MRI->use_instr_nodbg_begin(Reg);
  if (UseMI->getParent() != MBB)
    return Reg;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 126-133
```cpp
  while (UseMI->isCopy() || UseMI->isInsertSubreg()) {
    Reg = UseMI->getOperand(0).getReg();
    if (Reg.isPhysical() || !MRI->hasOneNonDBGUse(Reg))
      return Reg;
    UseMI = &*MRI->use_instr_nodbg_begin(Reg);
    if (UseMI->getParent() != MBB)
      return Reg;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 135-136
```cpp
  return Reg;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 138-143
```cpp
/// hasLoopHazard - Check whether an MLx instruction is chained to itself across
/// a single-MBB loop.
bool MLxExpansion::hasLoopHazard(MachineInstr *MI) const {
  Register Reg = MI->getOperand(1).getReg();
  if (Reg.isPhysical())
    return false;
```
- EN: Implements `MLxExpansion::hasLoopHazard`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MLxExpansion::hasLoopHazard`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 145-150
```cpp
  MachineBasicBlock *MBB = MI->getParent();
  MachineInstr *DefMI = MRI->getVRegDef(Reg);
  while (true) {
outer_continue:
    if (DefMI->getParent() != MBB)
      break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 152-166
```cpp
    if (DefMI->isPHI()) {
      for (unsigned i = 1, e = DefMI->getNumOperands(); i < e; i += 2) {
        if (DefMI->getOperand(i + 1).getMBB() == MBB) {
          Register SrcReg = DefMI->getOperand(i).getReg();
          if (SrcReg.isVirtual()) {
            DefMI = MRI->getVRegDef(SrcReg);
            goto outer_continue;
          }
        }
      }
    } else if (DefMI->isCopyLike()) {
      Reg = DefMI->getOperand(1).getReg();
      if (Reg.isVirtual()) {
        DefMI = MRI->getVRegDef(Reg);
        continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 167-174
```cpp
      }
    } else if (DefMI->isInsertSubreg()) {
      Reg = DefMI->getOperand(2).getReg();
      if (Reg.isVirtual()) {
        DefMI = MRI->getVRegDef(Reg);
        continue;
      }
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 176-177
```cpp
    break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 179-180
```cpp
  return DefMI == MI;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 182-194
```cpp
bool MLxExpansion::hasRAWHazard(unsigned Reg, MachineInstr *MI) const {
  // FIXME: Detect integer instructions properly.
  const MCInstrDesc &MCID = MI->getDesc();
  unsigned Domain = MCID.TSFlags & ARMII::DomainMask;
  if (MI->mayStore())
    return false;
  unsigned Opcode = MCID.getOpcode();
  if (Opcode == ARM::VMOVRS || Opcode == ARM::VMOVRRD)
    return false;
  if ((Domain & ARMII::DomainVFP) || (Domain & ARMII::DomainNEON))
    return MI->readsRegister(Reg, TRI);
  return false;
}
```
- EN: Implements `MLxExpansion::hasRAWHazard`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MLxExpansion::hasRAWHazard`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 196-208
```cpp
static bool isFpMulInstruction(unsigned Opcode) {
  switch (Opcode) {
  case ARM::VMULS:
  case ARM::VMULfd:
  case ARM::VMULfq:
  case ARM::VMULD:
  case ARM::VMULslfd:
  case ARM::VMULslfq:
    return true;
  default:
    return false;
  }
}
```
- EN: Implements `isFpMulInstruction`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isFpMulInstruction`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 210-212
```cpp
bool MLxExpansion::FindMLxHazard(MachineInstr *MI) {
  if (NumExpand >= ExpandLimit)
    return false;
```
- EN: Implements `MLxExpansion::FindMLxHazard`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MLxExpansion::FindMLxHazard`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 214-215
```cpp
  if (ForceExpand)
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 217-229
```cpp
  MachineInstr *DefMI = getAccDefMI(MI);
  if (TII->isFpMLxInstruction(DefMI->getOpcode())) {
    // r0 = vmla
    // r3 = vmla r0, r1, r2
    // takes 16 - 17 cycles
    //
    // r0 = vmla
    // r4 = vmul r1, r2
    // r3 = vadd r0, r4
    // takes about 14 - 15 cycles even with vmul stalling for 4 cycles.
    IgnoreStall.insert(DefMI);
    return true;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 231-235
```cpp
  // On Swift, we mostly care about hazards from multiplication instructions
  // writing the accumulator and the pipelining of loop iterations by out-of-
  // order execution.
  if (isSwift)
    return isFpMulInstruction(DefMI->getOpcode()) || hasLoopHazard(MI);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 237-238
```cpp
  if (IgnoreStall.count(MI))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 240-251
```cpp
  // If a VMLA.F is followed by an VADD.F or VMUL.F with no RAW hazard, the
  // VADD.F or VMUL.F will stall 4 cycles before issue. The 4 cycle stall
  // preserves the in-order retirement of the instructions.
  // Look at the next few instructions, if *most* of them can cause hazards,
  // then the scheduler can't *fix* this, we'd better break up the VMLA.
  unsigned Limit1 = isLikeA9 ? 1 : 4;
  unsigned Limit2 = isLikeA9 ? 1 : 4;
  for (unsigned i = 1; i <= 4; ++i) {
    int Idx = ((int)MIIdx - i + 4) % 4;
    MachineInstr *NextMI = LastMIs[Idx];
    if (!NextMI)
      continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 253-256
```cpp
    if (TII->canCauseFpMLxStall(NextMI->getOpcode())) {
      if (i <= Limit1)
        return true;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 258-261
```cpp
    // Look for VMLx RAW hazard.
    if (i <= Limit2 && hasRAWHazard(getDefReg(MI), NextMI))
      return true;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 263-264
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 266-280
```cpp
/// ExpandFPMLxInstructions - Expand a MLA / MLS instruction into a pair
/// of MUL + ADD / SUB instructions.
void
MLxExpansion::ExpandFPMLxInstruction(MachineBasicBlock &MBB, MachineInstr *MI,
                                     unsigned MulOpc, unsigned AddSubOpc,
                                     bool NegAcc, bool HasLane) {
  Register DstReg = MI->getOperand(0).getReg();
  bool DstDead = MI->getOperand(0).isDead();
  Register AccReg = MI->getOperand(1).getReg();
  Register Src1Reg = MI->getOperand(2).getReg();
  Register Src2Reg = MI->getOperand(3).getReg();
  bool Src1Kill = MI->getOperand(2).isKill();
  bool Src2Kill = MI->getOperand(3).isKill();
  unsigned LaneImm = HasLane ? MI->getOperand(4).getImm() : 0;
  unsigned NextOp = HasLane ? 5 : 4;
```
- EN: Implements `MLxExpansion::ExpandFPMLxInstruction`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MLxExpansion::ExpandFPMLxInstruction`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 281-282
```cpp
  ARMCC::CondCodes Pred = (ARMCC::CondCodes)MI->getOperand(NextOp).getImm();
  Register PredReg = MI->getOperand(++NextOp).getReg();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 284-286
```cpp
  const MCInstrDesc &MCID1 = TII->get(MulOpc);
  const MCInstrDesc &MCID2 = TII->get(AddSubOpc);
  Register TmpReg = MRI->createVirtualRegister(TII->getRegClass(MCID1, 0));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 288-293
```cpp
  MachineInstrBuilder MIB = BuildMI(MBB, MI, MI->getDebugLoc(), MCID1, TmpReg)
    .addReg(Src1Reg, getKillRegState(Src1Kill))
    .addReg(Src2Reg, getKillRegState(Src2Kill));
  if (HasLane)
    MIB.addImm(LaneImm);
  MIB.addImm(Pred).addReg(PredReg);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 295-296
```cpp
  MIB = BuildMI(MBB, MI, MI->getDebugLoc(), MCID2)
    .addReg(DstReg, getDefRegState(true) | getDeadRegState(DstDead));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 298-305
```cpp
  if (NegAcc) {
    bool AccKill = MRI->hasOneNonDBGUse(AccReg);
    MIB.addReg(TmpReg, getKillRegState(true))
       .addReg(AccReg, getKillRegState(AccKill));
  } else {
    MIB.addReg(AccReg).addReg(TmpReg, getKillRegState(true));
  }
  MIB.addImm(Pred).addReg(PredReg);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 307-317
```cpp
  LLVM_DEBUG({
    dbgs() << "Expanding: " << *MI;
    dbgs() << "  to:\n";
    MachineBasicBlock::iterator MII = MI;
    MII = std::prev(MII);
    MachineInstr &MI2 = *MII;
    MII = std::prev(MII);
    MachineInstr &MI1 = *MII;
    dbgs() << "    " << MI1;
    dbgs() << "    " << MI2;
  });
```
- EN: Implements `LLVM_DEBUG`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `LLVM_DEBUG`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 319-321
```cpp
  MI->eraseFromParent();
  ++NumExpand;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 323-324
```cpp
bool MLxExpansion::ExpandFPMLxInstructions(MachineBasicBlock &MBB) {
  bool Changed = false;
```
- EN: Implements `MLxExpansion::ExpandFPMLxInstructions`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MLxExpansion::ExpandFPMLxInstructions`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 326-327
```cpp
  clearStack();
  IgnoreStall.clear();
```
- EN: Declares `clearStack`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `clearStack`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 329-332
```cpp
  unsigned Skip = 0;
  MachineBasicBlock::reverse_iterator MII = MBB.rbegin(), E = MBB.rend();
  while (MII != E) {
    MachineInstr *MI = &*MII++;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 334-335
```cpp
    if (MI->isPosition() || MI->isImplicitDef() || MI->isCopy())
      continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 337-342
```cpp
    const MCInstrDesc &MCID = MI->getDesc();
    if (MI->isBarrier()) {
      clearStack();
      Skip = 0;
      continue;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 344-350
```cpp
    unsigned Domain = MCID.TSFlags & ARMII::DomainMask;
    if (Domain == ARMII::DomainGeneral) {
      if (++Skip == 2)
        // Assume dual issues of non-VFP / NEON instructions.
        pushStack(nullptr);
    } else {
      Skip = 0;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 352-363
```cpp
      unsigned MulOpc, AddSubOpc;
      bool NegAcc, HasLane;
      if (!TII->isFpMLxInstruction(MCID.getOpcode(),
                                   MulOpc, AddSubOpc, NegAcc, HasLane) ||
          !FindMLxHazard(MI))
        pushStack(MI);
      else {
        ExpandFPMLxInstruction(MBB, MI, MulOpc, AddSubOpc, NegAcc, HasLane);
        Changed = true;
      }
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 365-366
```cpp
  return Changed;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 368-370
```cpp
bool MLxExpansion::runOnMachineFunction(MachineFunction &Fn) {
  if (skipFunction(Fn.getFunction()))
    return false;
```
- EN: Implements `MLxExpansion::runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MLxExpansion::runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 372-379
```cpp
  TII = static_cast<const ARMBaseInstrInfo *>(Fn.getSubtarget().getInstrInfo());
  TRI = Fn.getSubtarget().getRegisterInfo();
  MRI = &Fn.getRegInfo();
  const ARMSubtarget *STI = &Fn.getSubtarget<ARMSubtarget>();
  if (!STI->expandMLx())
    return false;
  isLikeA9 = STI->isLikeA9() || STI->isSwift();
  isSwift = STI->isSwift();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 381-383
```cpp
  bool Modified = false;
  for (MachineBasicBlock &MBB : Fn)
    Modified |= ExpandFPMLxInstructions(MBB);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 385-386
```cpp
  return Modified;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 388-390
```cpp
FunctionPass *llvm::createMLxExpansionPass() {
  return new MLxExpansion();
}
```
- EN: Implements `llvm::createMLxExpansionPass`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::createMLxExpansionPass`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

## Key Concepts / 关键概念

- EN: Primary role: target-specific code-generation behavior.
  - CN: 核心职责：目标相关代码生成行为。
- EN: MachineInstr-level manipulation is central to this file.
  - CN: 该文件以 MachineInstr 层面的操作为核心。
- EN: MachineBasicBlock flow and branch structure matter to the implementation.
  - CN: 实现高度依赖 MachineBasicBlock 流程与分支结构。
- EN: The MC layer is used to print, encode, or stream target instructions and symbols.
  - CN: MC 层被用于打印、编码或流式输出目标指令与符号。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。

## Dependencies / 依赖关系

- EN: Backend-local headers: `ARM.h`, `ARMBaseInstrInfo.h`, `ARMSubtarget.h`.
  - CN: 后端本地头文件：`ARM.h`, `ARMBaseInstrInfo.h`, `ARMSubtarget.h`。
- EN: LLVM infrastructure headers: `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/Statistic.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineRegisterInfo.h`, `llvm/CodeGen/TargetRegisterInfo.h`, `llvm/Support/CommandLine.h` ... (+2 more).
  - CN: LLVM 基础设施头文件：`llvm/ADT/SmallPtrSet.h`, `llvm/ADT/Statistic.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineRegisterInfo.h`, `llvm/CodeGen/TargetRegisterInfo.h`, `llvm/Support/CommandLine.h` ... (+2 more)。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
- EN: The implementation also depends on LLVM MC layer classes for final assembly/object emission.
  - CN: 实现还依赖 LLVM 的 MC 层类来完成最终汇编/目标文件输出。
