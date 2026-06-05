# A15SDOptimizer.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/A15SDOptimizer.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file defines a pre-regalloc pass which looks for SPR producers which are going to be used by a DPR (or QPR) consumers and creates the more optimized access pattern.
- 用途 (CN): 实现 ARM 后端中的 `A15SDOptimizer`，重点处理目标相关代码生成行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```cpp
//=== A15SDOptimizerPass.cpp - Optimize DPR and SPR register accesses on A15==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// The Cortex-A15 processor employs a tracking scheme in its register renaming
// in order to process each instruction's micro-ops speculatively and
// out-of-order with appropriate forwarding. The ARM architecture allows VFP
// instructions to read and write 32-bit S-registers.  Each S-register
// corresponds to one half (upper or lower) of an overlaid 64-bit D-register.
//
// There are several instruction patterns which can be used to provide this
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 16-24
```cpp
// capability which can provide higher performance than other, potentially more
// direct patterns, specifically around when one micro-op reads a D-register
// operand that has recently been written as one or more S-register results.
//
// This file defines a pre-regalloc pass which looks for SPR producers which
// are going to be used by a DPR (or QPR) consumers and creates the more
// optimized access pattern.
//
//===----------------------------------------------------------------------===//
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 26-40
```cpp
#include "ARM.h"
#include "ARMBaseInstrInfo.h"
#include "ARMBaseRegisterInfo.h"
#include "ARMSubtarget.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/TargetRegisterInfo.h"
#include "llvm/CodeGen/TargetSubtargetInfo.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"
#include <map>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 41-41
```cpp
#include <set>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 43-43
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 45-45
```cpp
#define DEBUG_TYPE "a15-sd-optimizer"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 47-50
```cpp
namespace {
  struct A15SDOptimizer : public MachineFunctionPass {
    static char ID;
    A15SDOptimizer() : MachineFunctionPass(ID) {}
```
- EN: Declares `A15SDOptimizer`, packaging target-specific state and APIs around `A15SDOptimizer`.
- CN: 这里声明 `A15SDOptimizer`，把与 `A15SDOptimizer` 相关的目标特定状态和 API 组织在一起。

### Lines 52-52
```cpp
    bool runOnMachineFunction(MachineFunction &Fn) override;
```
- EN: Declares `runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 54-54
```cpp
    StringRef getPassName() const override { return "ARM A15 S->D optimizer"; }
```
- EN: Implements `getPassName`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getPassName`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 56-59
```cpp
  private:
    const ARMBaseInstrInfo *TII;
    const TargetRegisterInfo *TRI;
    MachineRegisterInfo *MRI;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 61-61
```cpp
    bool runOnInstruction(MachineInstr *MI);
```
- EN: Declares `runOnInstruction`, a pass-entry routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `runOnInstruction`，它是一个围绕机器指令展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 63-69
```cpp
    //
    // Instruction builder helpers
    //
    unsigned createDupLane(MachineBasicBlock &MBB,
                           MachineBasicBlock::iterator InsertBefore,
                           const DebugLoc &DL, unsigned Reg, unsigned Lane,
                           bool QPR = false);
```
- EN: Declares `createDupLane`, a query/helper routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `createDupLane`，它是一个围绕机器基本块展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 71-74
```cpp
    unsigned createExtractSubreg(MachineBasicBlock &MBB,
                                 MachineBasicBlock::iterator InsertBefore,
                                 const DebugLoc &DL, unsigned DReg,
                                 unsigned Lane, const TargetRegisterClass *TRC);
```
- EN: Declares `createExtractSubreg`, a query/helper routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `createExtractSubreg`，它是一个围绕机器基本块展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 76-78
```cpp
    unsigned createVExt(MachineBasicBlock &MBB,
                        MachineBasicBlock::iterator InsertBefore,
                        const DebugLoc &DL, unsigned Ssub0, unsigned Ssub1);
```
- EN: Declares `createVExt`, a query/helper routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `createVExt`，它是一个围绕机器基本块展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 80-83
```cpp
    unsigned createRegSequence(MachineBasicBlock &MBB,
                               MachineBasicBlock::iterator InsertBefore,
                               const DebugLoc &DL, unsigned Reg1,
                               unsigned Reg2);
```
- EN: Declares `createRegSequence`, a query/helper routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `createRegSequence`，它是一个围绕机器基本块展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 85-88
```cpp
    unsigned createInsertSubreg(MachineBasicBlock &MBB,
                                MachineBasicBlock::iterator InsertBefore,
                                const DebugLoc &DL, unsigned DReg,
                                unsigned Lane, unsigned ToInsert);
```
- EN: Declares `createInsertSubreg`, a query/helper routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `createInsertSubreg`，它是一个围绕机器基本块展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 90-92
```cpp
    unsigned createImplicitDef(MachineBasicBlock &MBB,
                               MachineBasicBlock::iterator InsertBefore,
                               const DebugLoc &DL);
```
- EN: Declares `createImplicitDef`, a query/helper routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `createImplicitDef`，它是一个围绕机器基本块展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 94-100
```cpp
    //
    // Various property checkers
    //
    bool usesRegClass(MachineOperand &MO, const TargetRegisterClass *TRC);
    bool hasPartialWrite(MachineInstr *MI);
    SmallVector<unsigned, 8> getReadDPRs(MachineInstr *MI);
    unsigned getDPRLaneFromSPR(unsigned SReg);
```
- EN: Declares `usesRegClass`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `usesRegClass`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 102-104
```cpp
    //
    // Methods used for getting the definitions of partial registers
    //
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 106-108
```cpp
    MachineInstr *elideCopies(MachineInstr *MI);
    void elideCopiesAndPHIs(MachineInstr *MI,
                            SmallVectorImpl<MachineInstr*> &Outs);
```
- EN: Declares `elideCopies`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `elideCopies`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 110-115
```cpp
    //
    // Pattern optimization methods
    //
    unsigned optimizeAllLanesPattern(MachineInstr *MI, unsigned Reg);
    unsigned optimizeSDPattern(MachineInstr *MI);
    unsigned getPrefSPRLane(unsigned SReg);
```
- EN: Declares `optimizeAllLanesPattern`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `optimizeAllLanesPattern`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 117-120
```cpp
    //
    // Sanitizing method - used to make sure if don't leave dead code around.
    //
    void eraseInstrWithNoUses(MachineInstr *MI);
```
- EN: Declares `eraseInstrWithNoUses`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `eraseInstrWithNoUses`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 122-129
```cpp
    //
    // A map used to track the changes done by this pass.
    //
    std::map<MachineInstr*, unsigned> Replacements;
    std::set<MachineInstr *> DeadInstr;
  };
  char A15SDOptimizer::ID = 0;
} // end anonymous namespace
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 131-136
```cpp
// Returns true if this is a use of a SPR register.
bool A15SDOptimizer::usesRegClass(MachineOperand &MO,
                                  const TargetRegisterClass *TRC) {
  if (!MO.isReg())
    return false;
  Register Reg = MO.getReg();
```
- EN: Implements `A15SDOptimizer::usesRegClass`, a target-specific routine centered on machine operands. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `A15SDOptimizer::usesRegClass`，它是一个围绕机器操作数展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 138-142
```cpp
  if (Reg.isVirtual())
    return MRI->getRegClass(Reg)->hasSuperClassEq(TRC);
  else
    return TRC->contains(Reg);
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 144-150
```cpp
unsigned A15SDOptimizer::getDPRLaneFromSPR(unsigned SReg) {
  MCRegister DReg =
      TRI->getMatchingSuperReg(SReg, ARM::ssub_1, &ARM::DPRRegClass);
  if (DReg)
    return ARM::ssub_1;
  return ARM::ssub_0;
}
```
- EN: Implements `A15SDOptimizer::getDPRLaneFromSPR`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `A15SDOptimizer::getDPRLaneFromSPR`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 152-156
```cpp
// Get the subreg type that is most likely to be coalesced
// for an SPR register that will be used in VDUP32d pseudo.
unsigned A15SDOptimizer::getPrefSPRLane(unsigned SReg) {
  if (!Register::isVirtualRegister(SReg))
    return getDPRLaneFromSPR(SReg);
```
- EN: Implements `A15SDOptimizer::getPrefSPRLane`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `A15SDOptimizer::getPrefSPRLane`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 158-162
```cpp
  MachineInstr *MI = MRI->getVRegDef(SReg);
  if (!MI) return ARM::ssub_0;
  MachineOperand *MO = MI->findRegisterDefOperand(SReg, /*TRI=*/nullptr);
  if (!MO) return ARM::ssub_0;
  assert(MO->isReg() && "Non-register operand found!");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 164-167
```cpp
  if (MI->isCopy() && usesRegClass(MI->getOperand(1),
                                    &ARM::SPRRegClass)) {
    SReg = MI->getOperand(1).getReg();
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 169-174
```cpp
  if (Register::isVirtualRegister(SReg)) {
    if (MO->getSubReg() == ARM::ssub_1) return ARM::ssub_1;
    return ARM::ssub_0;
  }
  return getDPRLaneFromSPR(SReg);
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 176-180
```cpp
// MI is known to be dead. Figure out what instructions
// are also made dead by this and mark them for removal.
void A15SDOptimizer::eraseInstrWithNoUses(MachineInstr *MI) {
  SmallVector<MachineInstr *, 8> Front;
  DeadInstr.insert(MI);
```
- EN: Implements `A15SDOptimizer::eraseInstrWithNoUses`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `A15SDOptimizer::eraseInstrWithNoUses`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 182-183
```cpp
  LLVM_DEBUG(dbgs() << "Deleting base instruction " << *MI << "\n");
  Front.push_back(MI);
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 185-186
```cpp
  while (Front.size() != 0) {
    MI = Front.pop_back_val();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 188-196
```cpp
    // MI is already known to be dead. We need to see
    // if other instructions can also be removed.
    for (MachineOperand &MO : MI->operands()) {
      if ((!MO.isReg()) || (!MO.isUse()))
        continue;
      Register Reg = MO.getReg();
      if (!Reg.isVirtual())
        continue;
      MachineOperand *Op = MI->findRegisterDefOperand(Reg, /*TRI=*/nullptr);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 198-199
```cpp
      if (!Op)
        continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 201-201
```cpp
      MachineInstr *Def = Op->getParent();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 203-206
```cpp
      // We don't need to do anything if we have already marked
      // this instruction as being dead.
      if (DeadInstr.find(Def) != DeadInstr.end())
        continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 208-222
```cpp
      // Check if all the uses of this instruction are marked as
      // dead. If so, we can also mark this instruction as being
      // dead.
      bool IsDead = true;
      for (MachineOperand &MODef : Def->operands()) {
        if ((!MODef.isReg()) || (!MODef.isDef()))
          continue;
        Register DefReg = MODef.getReg();
        if (!DefReg.isVirtual()) {
          IsDead = false;
          break;
        }
        for (MachineInstr &Use : MRI->use_instructions(Reg)) {
          // We don't care about self references.
          if (&Use == Def)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 223-229
```cpp
            continue;
          if (DeadInstr.find(&Use) == DeadInstr.end()) {
            IsDead = false;
            break;
          }
        }
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 231-231
```cpp
      if (!IsDead) continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 233-237
```cpp
      LLVM_DEBUG(dbgs() << "Deleting instruction " << *Def << "\n");
      DeadInstr.insert(Def);
    }
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 239-244
```cpp
// Creates the more optimized patterns and generally does all the code
// transformations in this pass.
unsigned A15SDOptimizer::optimizeSDPattern(MachineInstr *MI) {
  if (MI->isCopy()) {
    return optimizeAllLanesPattern(MI, MI->getOperand(1).getReg());
  }
```
- EN: Implements `A15SDOptimizer::optimizeSDPattern`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `A15SDOptimizer::optimizeSDPattern`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 246-248
```cpp
  if (MI->isInsertSubreg()) {
    Register DPRReg = MI->getOperand(1).getReg();
    Register SPRReg = MI->getOperand(2).getReg();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 250-252
```cpp
    if (DPRReg.isVirtual() && SPRReg.isVirtual()) {
      MachineInstr *DPRMI = MRI->getVRegDef(MI->getOperand(1).getReg());
      MachineInstr *SPRMI = MRI->getVRegDef(MI->getOperand(2).getReg());
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 254-259
```cpp
      if (DPRMI && SPRMI) {
        // See if the first operand of this insert_subreg is IMPLICIT_DEF
        MachineInstr *ECDef = elideCopies(DPRMI);
        if (ECDef && ECDef->isImplicitDef()) {
          // Another corner case - if we're inserting something that is purely
          // a subreg copy of a DPR, just use that DPR.
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 261-265
```cpp
          MachineInstr *EC = elideCopies(SPRMI);
          // Is it a subreg copy of ssub_0?
          if (EC && EC->isCopy() &&
              EC->getOperand(1).getSubReg() == ARM::ssub_0) {
            LLVM_DEBUG(dbgs() << "Found a subreg copy: " << *SPRMI);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 267-278
```cpp
            // Find the thing we're subreg copying out of - is it of the same
            // regclass as DPRMI? (i.e. a DPR or QPR).
            Register FullReg = SPRMI->getOperand(1).getReg();
            const TargetRegisterClass *TRC =
              MRI->getRegClass(MI->getOperand(1).getReg());
            if (TRC->hasSuperClassEq(MRI->getRegClass(FullReg))) {
              LLVM_DEBUG(dbgs() << "Subreg copy is compatible - returning ");
              LLVM_DEBUG(dbgs() << printReg(FullReg) << "\n");
              eraseInstrWithNoUses(MI);
              return FullReg;
            }
          }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 280-285
```cpp
          return optimizeAllLanesPattern(MI, MI->getOperand(2).getReg());
        }
      }
    }
    return optimizeAllLanesPattern(MI, MI->getOperand(0).getReg());
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 287-292
```cpp
  if (MI->isRegSequence() && usesRegClass(MI->getOperand(1),
                                          &ARM::SPRRegClass)) {
    // See if all bar one of the operands are IMPLICIT_DEF and insert the
    // optimizer pattern accordingly.
    unsigned NumImplicit = 0, NumTotal = 0;
    unsigned NonImplicitReg = ~0U;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 294-298
```cpp
    for (MachineOperand &MO : llvm::drop_begin(MI->explicit_operands())) {
      if (!MO.isReg())
        continue;
      ++NumTotal;
      Register OpReg = MO.getReg();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 300-301
```cpp
      if (!OpReg.isVirtual())
        break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 303-310
```cpp
      MachineInstr *Def = MRI->getVRegDef(OpReg);
      if (!Def)
        break;
      if (Def->isImplicitDef())
        ++NumImplicit;
      else
        NonImplicitReg = MO.getReg();
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 312-316
```cpp
    if (NumImplicit == NumTotal - 1)
      return optimizeAllLanesPattern(MI, NonImplicitReg);
    else
      return optimizeAllLanesPattern(MI, MI->getOperand(0).getReg());
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 318-319
```cpp
  llvm_unreachable("Unhandled update pattern!");
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 321-327
```cpp
// Return true if this MachineInstr inserts a scalar (SPR) value into
// a D or Q register.
bool A15SDOptimizer::hasPartialWrite(MachineInstr *MI) {
  // The only way we can do a partial register update is through a COPY,
  // INSERT_SUBREG or REG_SEQUENCE.
  if (MI->isCopy() && usesRegClass(MI->getOperand(1), &ARM::SPRRegClass))
    return true;
```
- EN: Implements `A15SDOptimizer::hasPartialWrite`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `A15SDOptimizer::hasPartialWrite`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 329-331
```cpp
  if (MI->isInsertSubreg() && usesRegClass(MI->getOperand(2),
                                           &ARM::SPRRegClass))
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 333-334
```cpp
  if (MI->isRegSequence() && usesRegClass(MI->getOperand(1), &ARM::SPRRegClass))
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 336-337
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 339-350
```cpp
// Looks through full copies to get the instruction that defines the input
// operand for MI.
MachineInstr *A15SDOptimizer::elideCopies(MachineInstr *MI) {
  if (!MI->isFullCopy())
    return MI;
  if (!MI->getOperand(1).getReg().isVirtual())
    return nullptr;
  MachineInstr *Def = MRI->getVRegDef(MI->getOperand(1).getReg());
  if (!Def)
    return nullptr;
  return elideCopies(Def);
}
```
- EN: Implements `A15SDOptimizer::elideCopies`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `A15SDOptimizer::elideCopies`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 352-362
```cpp
// Look through full copies and PHIs to get the set of non-copy MachineInstrs
// that can produce MI.
void A15SDOptimizer::elideCopiesAndPHIs(MachineInstr *MI,
                                        SmallVectorImpl<MachineInstr*> &Outs) {
   // Looking through PHIs may create loops so we need to track what
   // instructions we have visited before.
   std::set<MachineInstr *> Reached;
   SmallVector<MachineInstr *, 8> Front;
   Front.push_back(MI);
   while (Front.size() != 0) {
     MI = Front.pop_back_val();
```
- EN: Implements `A15SDOptimizer::elideCopiesAndPHIs`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `A15SDOptimizer::elideCopiesAndPHIs`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 364-378
```cpp
     // If we have already explored this MachineInstr, ignore it.
     if (!Reached.insert(MI).second)
       continue;
     if (MI->isPHI()) {
       for (unsigned I = 1, E = MI->getNumOperands(); I != E; I += 2) {
         Register Reg = MI->getOperand(I).getReg();
         if (!Reg.isVirtual()) {
           continue;
         }
         MachineInstr *NewMI = MRI->getVRegDef(Reg);
         if (!NewMI)
           continue;
         Front.push_back(NewMI);
       }
     } else if (MI->isFullCopy()) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 379-390
```cpp
       if (!MI->getOperand(1).getReg().isVirtual())
         continue;
       MachineInstr *NewMI = MRI->getVRegDef(MI->getOperand(1).getReg());
       if (!NewMI)
         continue;
       Front.push_back(NewMI);
     } else {
       LLVM_DEBUG(dbgs() << "Found partial copy" << *MI << "\n");
       Outs.push_back(MI);
     }
   }
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 392-397
```cpp
// Return the DPR virtual registers that are read by this machine instruction
// (if any).
SmallVector<unsigned, 8> A15SDOptimizer::getReadDPRs(MachineInstr *MI) {
  if (MI->isCopyLike() || MI->isInsertSubreg() || MI->isRegSequence() ||
      MI->isKill())
    return SmallVector<unsigned, 8>();
```
- EN: Implements `A15SDOptimizer::getReadDPRs`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `A15SDOptimizer::getReadDPRs`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 399-406
```cpp
  SmallVector<unsigned, 8> Defs;
  for (MachineOperand &MO : MI->operands()) {
    if (!MO.isReg() || !MO.isUse())
      continue;
    if (!usesRegClass(MO, &ARM::DPRRegClass) &&
        !usesRegClass(MO, &ARM::QPRRegClass) &&
        !usesRegClass(MO, &ARM::DPairRegClass)) // Treat DPair as QPR
      continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 408-411
```cpp
    Defs.push_back(MO.getReg());
  }
  return Defs;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 413-424
```cpp
// Creates a DPR register from an SPR one by using a VDUP.
unsigned A15SDOptimizer::createDupLane(MachineBasicBlock &MBB,
                                       MachineBasicBlock::iterator InsertBefore,
                                       const DebugLoc &DL, unsigned Reg,
                                       unsigned Lane, bool QPR) {
  Register Out =
      MRI->createVirtualRegister(QPR ? &ARM::QPRRegClass : &ARM::DPRRegClass);
  BuildMI(MBB, InsertBefore, DL,
          TII->get(QPR ? ARM::VDUPLN32q : ARM::VDUPLN32d), Out)
      .addReg(Reg)
      .addImm(Lane)
      .add(predOps(ARMCC::AL));
```
- EN: Implements `A15SDOptimizer::createDupLane`, a query/helper routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `A15SDOptimizer::createDupLane`，它是一个围绕机器基本块展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 426-427
```cpp
  return Out;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 429-436
```cpp
// Creates a SPR register from a DPR by copying the value in lane 0.
unsigned A15SDOptimizer::createExtractSubreg(
    MachineBasicBlock &MBB, MachineBasicBlock::iterator InsertBefore,
    const DebugLoc &DL, unsigned DReg, unsigned Lane,
    const TargetRegisterClass *TRC) {
  Register Out = MRI->createVirtualRegister(TRC);
  BuildMI(MBB, InsertBefore, DL, TII->get(TargetOpcode::COPY), Out)
      .addReg(DReg, {}, Lane);
```
- EN: Implements `A15SDOptimizer::createExtractSubreg`, a query/helper routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `A15SDOptimizer::createExtractSubreg`，它是一个围绕机器基本块展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 438-439
```cpp
  return Out;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 441-455
```cpp
// Takes two SPR registers and creates a DPR by using a REG_SEQUENCE.
unsigned A15SDOptimizer::createRegSequence(
    MachineBasicBlock &MBB, MachineBasicBlock::iterator InsertBefore,
    const DebugLoc &DL, unsigned Reg1, unsigned Reg2) {
  Register Out = MRI->createVirtualRegister(&ARM::QPRRegClass);
  BuildMI(MBB,
          InsertBefore,
          DL,
          TII->get(TargetOpcode::REG_SEQUENCE), Out)
    .addReg(Reg1)
    .addImm(ARM::dsub_0)
    .addReg(Reg2)
    .addImm(ARM::dsub_1);
  return Out;
}
```
- EN: Implements `A15SDOptimizer::createRegSequence`, a query/helper routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `A15SDOptimizer::createRegSequence`，它是一个围绕机器基本块展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 457-470
```cpp
// Takes two DPR registers that have previously been VDUPed (Ssub0 and Ssub1)
// and merges them into one DPR register.
unsigned A15SDOptimizer::createVExt(MachineBasicBlock &MBB,
                                    MachineBasicBlock::iterator InsertBefore,
                                    const DebugLoc &DL, unsigned Ssub0,
                                    unsigned Ssub1) {
  Register Out = MRI->createVirtualRegister(&ARM::DPRRegClass);
  BuildMI(MBB, InsertBefore, DL, TII->get(ARM::VEXTd32), Out)
      .addReg(Ssub0)
      .addReg(Ssub1)
      .addImm(1)
      .add(predOps(ARMCC::AL));
  return Out;
}
```
- EN: Implements `A15SDOptimizer::createVExt`, a query/helper routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `A15SDOptimizer::createVExt`，它是一个围绕机器基本块展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 472-482
```cpp
unsigned A15SDOptimizer::createInsertSubreg(
    MachineBasicBlock &MBB, MachineBasicBlock::iterator InsertBefore,
    const DebugLoc &DL, unsigned DReg, unsigned Lane, unsigned ToInsert) {
  Register Out = MRI->createVirtualRegister(&ARM::DPR_VFP2RegClass);
  BuildMI(MBB,
          InsertBefore,
          DL,
          TII->get(TargetOpcode::INSERT_SUBREG), Out)
    .addReg(DReg)
    .addReg(ToInsert)
    .addImm(Lane);
```
- EN: Implements `A15SDOptimizer::createInsertSubreg`, a query/helper routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `A15SDOptimizer::createInsertSubreg`，它是一个围绕机器基本块展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 484-485
```cpp
  return Out;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 487-497
```cpp
unsigned
A15SDOptimizer::createImplicitDef(MachineBasicBlock &MBB,
                                  MachineBasicBlock::iterator InsertBefore,
                                  const DebugLoc &DL) {
  Register Out = MRI->createVirtualRegister(&ARM::DPRRegClass);
  BuildMI(MBB,
          InsertBefore,
          DL,
          TII->get(TargetOpcode::IMPLICIT_DEF), Out);
  return Out;
}
```
- EN: Implements `A15SDOptimizer::createImplicitDef`, a query/helper routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `A15SDOptimizer::createImplicitDef`，它是一个围绕机器基本块展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 499-508
```cpp
// This function inserts instructions in order to optimize interactions between
// SPR registers and DPR/QPR registers. It does so by performing VDUPs on all
// lanes, and the using VEXT instructions to recompose the result.
unsigned
A15SDOptimizer::optimizeAllLanesPattern(MachineInstr *MI, unsigned Reg) {
  MachineBasicBlock::iterator InsertPt(MI);
  DebugLoc DL = MI->getDebugLoc();
  MachineBasicBlock &MBB = *MI->getParent();
  InsertPt++;
  unsigned Out;
```
- EN: Implements `A15SDOptimizer::optimizeAllLanesPattern`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `A15SDOptimizer::optimizeAllLanesPattern`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 510-517
```cpp
  // DPair has the same length as QPR and also has two DPRs as subreg.
  // Treat DPair as QPR.
  if (MRI->getRegClass(Reg)->hasSuperClassEq(&ARM::QPRRegClass) ||
      MRI->getRegClass(Reg)->hasSuperClassEq(&ARM::DPairRegClass)) {
    unsigned DSub0 = createExtractSubreg(MBB, InsertPt, DL, Reg,
                                         ARM::dsub_0, &ARM::DPRRegClass);
    unsigned DSub1 = createExtractSubreg(MBB, InsertPt, DL, Reg,
                                         ARM::dsub_1, &ARM::DPRRegClass);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 519-521
```cpp
    unsigned Out1 = createDupLane(MBB, InsertPt, DL, DSub0, 0);
    unsigned Out2 = createDupLane(MBB, InsertPt, DL, DSub0, 1);
    Out = createVExt(MBB, InsertPt, DL, Out1, Out2);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 523-525
```cpp
    unsigned Out3 = createDupLane(MBB, InsertPt, DL, DSub1, 0);
    unsigned Out4 = createDupLane(MBB, InsertPt, DL, DSub1, 1);
    Out2 = createVExt(MBB, InsertPt, DL, Out3, Out4);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 527-527
```cpp
    Out = createRegSequence(MBB, InsertPt, DL, Out, Out2);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 529-532
```cpp
  } else if (MRI->getRegClass(Reg)->hasSuperClassEq(&ARM::DPRRegClass)) {
    unsigned Out1 = createDupLane(MBB, InsertPt, DL, Reg, 0);
    unsigned Out2 = createDupLane(MBB, InsertPt, DL, Reg, 1);
    Out = createVExt(MBB, InsertPt, DL, Out1, Out2);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 534-536
```cpp
  } else {
    assert(MRI->getRegClass(Reg)->hasSuperClassEq(&ARM::SPRRegClass) &&
           "Found unexpected regclass!");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 538-544
```cpp
    unsigned PrefLane = getPrefSPRLane(Reg);
    unsigned Lane;
    switch (PrefLane) {
      case ARM::ssub_0: Lane = 0; break;
      case ARM::ssub_1: Lane = 1; break;
      default: llvm_unreachable("Unknown preferred lane!");
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 546-548
```cpp
    // Treat DPair as QPR
    bool UsesQPR = usesRegClass(MI->getOperand(0), &ARM::QPRRegClass) ||
                   usesRegClass(MI->getOperand(0), &ARM::DPairRegClass);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 550-556
```cpp
    Out = createImplicitDef(MBB, InsertPt, DL);
    Out = createInsertSubreg(MBB, InsertPt, DL, Out, PrefLane, Reg);
    Out = createDupLane(MBB, InsertPt, DL, Out, Lane, UsesQPR);
    eraseInstrWithNoUses(MI);
  }
  return Out;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 558-572
```cpp
bool A15SDOptimizer::runOnInstruction(MachineInstr *MI) {
  // We look for instructions that write S registers that are then read as
  // D/Q registers. These can only be caused by COPY, INSERT_SUBREG and
  // REG_SEQUENCE pseudos that insert an SPR value into a DPR register or
  // merge two SPR values to form a DPR register.  In order avoid false
  // positives we make sure that there is an SPR producer so we look past
  // COPY and PHI nodes to find it.
  //
  // The best code pattern for when an SPR producer is going to be used by a
  // DPR or QPR consumer depends on whether the other lanes of the
  // corresponding DPR/QPR are currently defined.
  //
  // We can handle these efficiently, depending on the type of
  // pseudo-instruction that is producing the pattern
  //
```
- EN: Implements `A15SDOptimizer::runOnInstruction`, a pass-entry routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `A15SDOptimizer::runOnInstruction`，它是一个围绕机器指令展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 573-586
```cpp
  //   * COPY:          * VDUP all lanes and merge the results together
  //                      using VEXTs.
  //
  //   * INSERT_SUBREG: * If the SPR value was originally in another DPR/QPR
  //                      lane, and the other lane(s) of the DPR/QPR register
  //                      that we are inserting in are undefined, use the
  //                      original DPR/QPR value.
  //                    * Otherwise, fall back on the same strategy as COPY.
  //
  //   * REG_SEQUENCE:  * If all except one of the input operands are
  //                      IMPLICIT_DEFs, insert the VDUP pattern for just the
  //                      defined input operand
  //                    * Otherwise, fall back on the same strategy as COPY.
  //
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 588-590
```cpp
  // First, get all the reads of D-registers done by this instruction.
  SmallVector<unsigned, 8> Defs = getReadDPRs(MI);
  bool Modified = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 592-595
```cpp
  for (unsigned I : Defs) {
    // Follow the def-use chain for this DPR through COPYs, and also through
    // PHIs (which are essentially multi-way COPYs). It is because of PHIs that
    // we can end up with multiple defs of this DPR.
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 597-602
```cpp
    SmallVector<MachineInstr *, 8> DefSrcs;
    if (!Register::isVirtualRegister(I))
      continue;
    MachineInstr *Def = MRI->getVRegDef(I);
    if (!Def)
      continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 604-604
```cpp
    elideCopiesAndPHIs(Def, DefSrcs);
```
- EN: Declares `elideCopiesAndPHIs`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `elideCopiesAndPHIs`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 606-610
```cpp
    for (MachineInstr *MI : DefSrcs) {
      // If we've already analyzed and replaced this operand, don't do
      // anything.
      if (Replacements.find(MI) != Replacements.end())
        continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 612-614
```cpp
      // Now, work out if the instruction causes a SPR->DPR dependency.
      if (!hasPartialWrite(MI))
        continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 616-619
```cpp
      // Collect all the uses of this MI's DPR def for updating later.
      Register DPRDefReg = MI->getOperand(0).getReg();
      SmallVector<MachineOperand *, 8> Uses(
          llvm::make_pointer_range(MRI->use_operands(DPRDefReg)));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 621-622
```cpp
      // We can optimize this.
      unsigned NewReg = optimizeSDPattern(MI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 624-632
```cpp
      if (NewReg != 0) {
        Modified = true;
        for (MachineOperand *Use : Uses) {
          // Make sure to constrain the register class of the new register to
          // match what we're replacing. Otherwise we can optimize a DPR_VFP2
          // reference into a plain DPR, and that will end poorly. NewReg is
          // always virtual here, so there will always be a matching subclass
          // to find.
          MRI->constrainRegClass(NewReg, MRI->getRegClass(Use->getReg()));
```
- EN: Declares `of`, packaging target-specific state and APIs around `A15SDOptimizer`.
- CN: 这里声明 `of`，把与 `A15SDOptimizer` 相关的目标特定状态和 API 组织在一起。

### Lines 634-643
```cpp
          LLVM_DEBUG(dbgs() << "Replacing operand " << *Use << " with "
                            << printReg(NewReg) << "\n");
          Use->substVirtReg(NewReg, 0, *TRI);
        }
      }
      Replacements[MI] = NewReg;
    }
  }
  return Modified;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 645-647
```cpp
bool A15SDOptimizer::runOnMachineFunction(MachineFunction &Fn) {
  if (skipFunction(Fn.getFunction()))
    return false;
```
- EN: Implements `A15SDOptimizer::runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `A15SDOptimizer::runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 649-653
```cpp
  const ARMSubtarget &STI = Fn.getSubtarget<ARMSubtarget>();
  // Since the A15SDOptimizer pass can insert VDUP instructions, it can only be
  // enabled when NEON is available.
  if (!(STI.useSplatVFPToNeon() && STI.hasNEON()))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 655-658
```cpp
  TII = STI.getInstrInfo();
  TRI = STI.getRegisterInfo();
  MRI = &Fn.getRegInfo();
  bool Modified = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 660-660
```cpp
  LLVM_DEBUG(dbgs() << "Running on function " << Fn.getName() << "\n");
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 662-663
```cpp
  DeadInstr.clear();
  Replacements.clear();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 665-669
```cpp
  for (MachineBasicBlock &MBB : Fn) {
    for (MachineInstr &MI : MBB) {
      Modified |= runOnInstruction(&MI);
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 671-673
```cpp
  for (MachineInstr *MI : DeadInstr) {
    MI->eraseFromParent();
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 675-676
```cpp
  return Modified;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 678-680
```cpp
FunctionPass *llvm::createA15SDOptimizerPass() {
  return new A15SDOptimizer();
}
```
- EN: Implements `llvm::createA15SDOptimizerPass`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::createA15SDOptimizerPass`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

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

- EN: Backend-local headers: `ARM.h`, `ARMBaseInstrInfo.h`, `ARMBaseRegisterInfo.h`, `ARMSubtarget.h`.
  - CN: 后端本地头文件：`ARM.h`, `ARMBaseInstrInfo.h`, `ARMBaseRegisterInfo.h`, `ARMSubtarget.h`。
- EN: LLVM infrastructure headers: `llvm/ADT/Statistic.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineRegisterInfo.h`, `llvm/CodeGen/TargetRegisterInfo.h`, `llvm/CodeGen/TargetSubtargetInfo.h` ... (+2 more).
  - CN: LLVM 基础设施头文件：`llvm/ADT/Statistic.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineRegisterInfo.h`, `llvm/CodeGen/TargetRegisterInfo.h`, `llvm/CodeGen/TargetSubtargetInfo.h` ... (+2 more)。
- EN: Standard/system headers: `map`, `set`.
  - CN: 标准库/系统头文件：`map`, `set`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
