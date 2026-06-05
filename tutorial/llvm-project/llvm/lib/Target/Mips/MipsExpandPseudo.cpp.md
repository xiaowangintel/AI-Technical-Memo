# MipsExpandPseudo.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsExpandPseudo.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains a pass that expands pseudo instructions into target instructions to allow proper scheduling, if-conversion, and other late optimizations. This pass should be run after register allocation but before the post-regalloc scheduling pass.
- 用途 (CN): 实现 Mips 后端中的 `MipsExpandPseudo`，重点处理目标相关代码生成行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```cpp
//===-- MipsExpandPseudoInsts.cpp - Expand pseudo instructions ------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains a pass that expands pseudo instructions into target
// instructions to allow proper scheduling, if-conversion, and other late
// optimizations. This pass should be run after register allocation but before
// the post-regalloc scheduling pass.
//
// This is currently only used for expanding atomic pseudos after register
// allocation. We do this to avoid the fast register allocator introducing
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 16-19
```cpp
// spills between ll and sc. These stores cause some MIPS implementations to
// abort the atomic RMW sequence.
//
//===----------------------------------------------------------------------===//
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 21-26
```cpp
#include "Mips.h"
#include "MipsInstrInfo.h"
#include "MipsSubtarget.h"
#include "llvm/CodeGen/LivePhysRegs.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 28-28
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 30-30
```cpp
#define DEBUG_TYPE "mips-pseudo"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 32-36
```cpp
namespace {
  class MipsExpandPseudo : public MachineFunctionPass {
  public:
    static char ID;
    MipsExpandPseudo() : MachineFunctionPass(ID) {}
```
- EN: Declares `MipsExpandPseudo`, packaging target-specific state and APIs around `MipsExpandPseudo`.
- CN: 这里声明 `MipsExpandPseudo`，把与 `MipsExpandPseudo` 相关的目标特定状态和 API 组织在一起。

### Lines 38-39
```cpp
    const MipsInstrInfo *TII;
    const MipsSubtarget *STI;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 41-41
```cpp
    bool runOnMachineFunction(MachineFunction &Fn) override;
```
- EN: Declares `runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 43-45
```cpp
    MachineFunctionProperties getRequiredProperties() const override {
      return MachineFunctionProperties().setNoVRegs();
    }
```
- EN: Implements `getRequiredProperties`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getRequiredProperties`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 47-49
```cpp
    StringRef getPassName() const override {
      return "Mips pseudo instruction expansion pass";
    }
```
- EN: Implements `getPassName`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getPassName`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 51-57
```cpp
  private:
    bool expandAtomicCmpSwap(MachineBasicBlock &MBB,
                             MachineBasicBlock::iterator MBBI,
                             MachineBasicBlock::iterator &NextMBBI);
    bool expandAtomicCmpSwapSubword(MachineBasicBlock &MBB,
                                    MachineBasicBlock::iterator MBBI,
                                    MachineBasicBlock::iterator &NextMBBI);
```
- EN: Declares `expandAtomicCmpSwap`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandAtomicCmpSwap`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 59-64
```cpp
    bool expandAtomicBinOp(MachineBasicBlock &BB,
                           MachineBasicBlock::iterator I,
                           MachineBasicBlock::iterator &NMBBI, unsigned Size);
    bool expandAtomicBinOpSubword(MachineBasicBlock &BB,
                                  MachineBasicBlock::iterator I,
                                  MachineBasicBlock::iterator &NMBBI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 66-71
```cpp
    bool expandMI(MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
                  MachineBasicBlock::iterator &NMBB);
    bool expandMBB(MachineBasicBlock &MBB);
   };
  char MipsExpandPseudo::ID = 0;
}
```
- EN: Declares `expandMI`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandMI`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 73-75
```cpp
bool MipsExpandPseudo::expandAtomicCmpSwapSubword(
    MachineBasicBlock &BB, MachineBasicBlock::iterator I,
    MachineBasicBlock::iterator &NMBBI) {
```
- EN: Implements `MipsExpandPseudo::expandAtomicCmpSwapSubword`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsExpandPseudo::expandAtomicCmpSwapSubword`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 77-77
```cpp
  MachineFunction *MF = BB.getParent();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 79-81
```cpp
  const bool ArePtrs64bit = STI->getABI().ArePtrs64bit();
  DebugLoc DL = I->getDebugLoc();
  unsigned LL, SC;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 83-87
```cpp
  unsigned ZERO = Mips::ZERO;
  unsigned BNE = Mips::BNE;
  unsigned BEQ = Mips::BEQ;
  unsigned SEOp =
      I->getOpcode() == Mips::ATOMIC_CMP_SWAP_I8_POSTRA ? Mips::SEB : Mips::SEH;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 89-99
```cpp
  if (STI->inMicroMipsMode()) {
      LL = STI->hasMips32r6() ? Mips::LL_MMR6 : Mips::LL_MM;
      SC = STI->hasMips32r6() ? Mips::SC_MMR6 : Mips::SC_MM;
      BNE = STI->hasMips32r6() ? Mips::BNEC_MMR6 : Mips::BNE_MM;
      BEQ = STI->hasMips32r6() ? Mips::BEQC_MMR6 : Mips::BEQ_MM;
  } else {
    LL = STI->hasMips32r6() ? (ArePtrs64bit ? Mips::LL64_R6 : Mips::LL_R6)
                            : (ArePtrs64bit ? Mips::LL64 : Mips::LL);
    SC = STI->hasMips32r6() ? (ArePtrs64bit ? Mips::SC64_R6 : Mips::SC_R6)
                            : (ArePtrs64bit ? Mips::SC64 : Mips::SC);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 101-109
```cpp
  Register Dest = I->getOperand(0).getReg();
  Register Ptr = I->getOperand(1).getReg();
  Register Mask = I->getOperand(2).getReg();
  Register ShiftCmpVal = I->getOperand(3).getReg();
  Register Mask2 = I->getOperand(4).getReg();
  Register ShiftNewVal = I->getOperand(5).getReg();
  Register ShiftAmnt = I->getOperand(6).getReg();
  Register Scratch = I->getOperand(7).getReg();
  Register Scratch2 = I->getOperand(8).getReg();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 111-121
```cpp
  // insert new blocks after the current block
  const BasicBlock *LLVM_BB = BB.getBasicBlock();
  MachineBasicBlock *loop1MBB = MF->CreateMachineBasicBlock(LLVM_BB);
  MachineBasicBlock *loop2MBB = MF->CreateMachineBasicBlock(LLVM_BB);
  MachineBasicBlock *sinkMBB = MF->CreateMachineBasicBlock(LLVM_BB);
  MachineBasicBlock *exitMBB = MF->CreateMachineBasicBlock(LLVM_BB);
  MachineFunction::iterator It = ++BB.getIterator();
  MF->insert(It, loop1MBB);
  MF->insert(It, loop2MBB);
  MF->insert(It, sinkMBB);
  MF->insert(It, exitMBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 123-126
```cpp
  // Transfer the remainder of BB and its successor edges to exitMBB.
  exitMBB->splice(exitMBB->begin(), &BB,
                  std::next(MachineBasicBlock::iterator(I)), BB.end());
  exitMBB->transferSuccessorsAndUpdatePHIs(&BB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 128-138
```cpp
  //  thisMBB:
  //    ...
  //    fallthrough --> loop1MBB
  BB.addSuccessor(loop1MBB, BranchProbability::getOne());
  loop1MBB->addSuccessor(sinkMBB);
  loop1MBB->addSuccessor(loop2MBB);
  loop1MBB->normalizeSuccProbs();
  loop2MBB->addSuccessor(loop1MBB);
  loop2MBB->addSuccessor(sinkMBB);
  loop2MBB->normalizeSuccProbs();
  sinkMBB->addSuccessor(exitMBB, BranchProbability::getOne());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 140-149
```cpp
  // loop1MBB:
  //   ll dest, 0(ptr)
  //   and Mask', dest, Mask
  //   bne Mask', ShiftCmpVal, exitMBB
  BuildMI(loop1MBB, DL, TII->get(LL), Scratch).addReg(Ptr).addImm(0);
  BuildMI(loop1MBB, DL, TII->get(Mips::AND), Scratch2)
      .addReg(Scratch)
      .addReg(Mask);
  BuildMI(loop1MBB, DL, TII->get(BNE))
    .addReg(Scratch2).addReg(ShiftCmpVal).addMBB(sinkMBB);
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 151-165
```cpp
  // loop2MBB:
  //   and dest, dest, mask2
  //   or dest, dest, ShiftNewVal
  //   sc dest, dest, 0(ptr)
  //   beq dest, $0, loop1MBB
  BuildMI(loop2MBB, DL, TII->get(Mips::AND), Scratch)
      .addReg(Scratch, RegState::Kill)
      .addReg(Mask2);
  BuildMI(loop2MBB, DL, TII->get(Mips::OR), Scratch)
      .addReg(Scratch, RegState::Kill)
      .addReg(ShiftNewVal);
  BuildMI(loop2MBB, DL, TII->get(SC), Scratch)
      .addReg(Scratch, RegState::Kill)
      .addReg(Ptr)
      .addImm(0);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 166-169
```cpp
  BuildMI(loop2MBB, DL, TII->get(BEQ))
      .addReg(Scratch, RegState::Kill)
      .addReg(ZERO)
      .addMBB(loop1MBB);
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 171-185
```cpp
  //  sinkMBB:
  //    srl     srlres, Mask', shiftamt
  //    sign_extend dest,srlres
  BuildMI(sinkMBB, DL, TII->get(Mips::SRLV), Dest)
      .addReg(Scratch2)
      .addReg(ShiftAmnt);
  if (STI->hasMips32r2()) {
    BuildMI(sinkMBB, DL, TII->get(SEOp), Dest).addReg(Dest);
  } else {
    const unsigned ShiftImm =
        I->getOpcode() == Mips::ATOMIC_CMP_SWAP_I16_POSTRA ? 16 : 24;
    BuildMI(sinkMBB, DL, TII->get(Mips::SLL), Dest)
        .addReg(Dest, RegState::Kill)
        .addImm(ShiftImm);
    BuildMI(sinkMBB, DL, TII->get(Mips::SRA), Dest)
```
- EN: Implements `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 186-188
```cpp
        .addReg(Dest, RegState::Kill)
        .addImm(ShiftImm);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 190-194
```cpp
  LivePhysRegs LiveRegs;
  computeAndAddLiveIns(LiveRegs, *loop1MBB);
  computeAndAddLiveIns(LiveRegs, *loop2MBB);
  computeAndAddLiveIns(LiveRegs, *sinkMBB);
  computeAndAddLiveIns(LiveRegs, *exitMBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 196-199
```cpp
  NMBBI = BB.end();
  I->eraseFromParent();
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 201-203
```cpp
bool MipsExpandPseudo::expandAtomicCmpSwap(MachineBasicBlock &BB,
                                           MachineBasicBlock::iterator I,
                                           MachineBasicBlock::iterator &NMBBI) {
```
- EN: Implements `MipsExpandPseudo::expandAtomicCmpSwap`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsExpandPseudo::expandAtomicCmpSwap`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 205-207
```cpp
  const unsigned Size =
      I->getOpcode() == Mips::ATOMIC_CMP_SWAP_I32_POSTRA ? 4 : 8;
  MachineFunction *MF = BB.getParent();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 209-210
```cpp
  const bool ArePtrs64bit = STI->getABI().ArePtrs64bit();
  DebugLoc DL = I->getDebugLoc();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 212-212
```cpp
  unsigned LL, SC, ZERO, BNE, BEQ, MOVE;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 214-228
```cpp
  if (Size == 4) {
    if (STI->inMicroMipsMode()) {
      LL = STI->hasMips32r6() ? Mips::LL_MMR6 : Mips::LL_MM;
      SC = STI->hasMips32r6() ? Mips::SC_MMR6 : Mips::SC_MM;
      BNE = STI->hasMips32r6() ? Mips::BNEC_MMR6 : Mips::BNE_MM;
      BEQ = STI->hasMips32r6() ? Mips::BEQC_MMR6 : Mips::BEQ_MM;
    } else {
      LL = STI->hasMips32r6()
               ? (ArePtrs64bit ? Mips::LL64_R6 : Mips::LL_R6)
               : (ArePtrs64bit ? Mips::LL64 : Mips::LL);
      SC = STI->hasMips32r6()
               ? (ArePtrs64bit ? Mips::SC64_R6 : Mips::SC_R6)
               : (ArePtrs64bit ? Mips::SC64 : Mips::SC);
      BNE = Mips::BNE;
      BEQ = Mips::BEQ;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 229-229
```cpp
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 231-240
```cpp
    ZERO = Mips::ZERO;
    MOVE = Mips::OR;
  } else {
    LL = STI->hasMips64r6() ? Mips::LLD_R6 : Mips::LLD;
    SC = STI->hasMips64r6() ? Mips::SCD_R6 : Mips::SCD;
    ZERO = Mips::ZERO_64;
    BNE = Mips::BNE64;
    BEQ = Mips::BEQ64;
    MOVE = Mips::OR64;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 242-246
```cpp
  Register Dest = I->getOperand(0).getReg();
  Register Ptr = I->getOperand(1).getReg();
  Register OldVal = I->getOperand(2).getReg();
  Register NewVal = I->getOperand(3).getReg();
  Register Scratch = I->getOperand(4).getReg();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 248-256
```cpp
  // insert new blocks after the current block
  const BasicBlock *LLVM_BB = BB.getBasicBlock();
  MachineBasicBlock *loop1MBB = MF->CreateMachineBasicBlock(LLVM_BB);
  MachineBasicBlock *loop2MBB = MF->CreateMachineBasicBlock(LLVM_BB);
  MachineBasicBlock *exitMBB = MF->CreateMachineBasicBlock(LLVM_BB);
  MachineFunction::iterator It = ++BB.getIterator();
  MF->insert(It, loop1MBB);
  MF->insert(It, loop2MBB);
  MF->insert(It, exitMBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 258-261
```cpp
  // Transfer the remainder of BB and its successor edges to exitMBB.
  exitMBB->splice(exitMBB->begin(), &BB,
                  std::next(MachineBasicBlock::iterator(I)), BB.end());
  exitMBB->transferSuccessorsAndUpdatePHIs(&BB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 263-272
```cpp
  //  thisMBB:
  //    ...
  //    fallthrough --> loop1MBB
  BB.addSuccessor(loop1MBB, BranchProbability::getOne());
  loop1MBB->addSuccessor(exitMBB);
  loop1MBB->addSuccessor(loop2MBB);
  loop1MBB->normalizeSuccProbs();
  loop2MBB->addSuccessor(loop1MBB);
  loop2MBB->addSuccessor(exitMBB);
  loop2MBB->normalizeSuccProbs();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 274-279
```cpp
  // loop1MBB:
  //   ll dest, 0(ptr)
  //   bne dest, oldval, exitMBB
  BuildMI(loop1MBB, DL, TII->get(LL), Dest).addReg(Ptr).addImm(0);
  BuildMI(loop1MBB, DL, TII->get(BNE))
    .addReg(Dest, RegState::Kill).addReg(OldVal).addMBB(exitMBB);
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 281-289
```cpp
  // loop2MBB:
  //   move scratch, NewVal
  //   sc Scratch, Scratch, 0(ptr)
  //   beq Scratch, $0, loop1MBB
  BuildMI(loop2MBB, DL, TII->get(MOVE), Scratch).addReg(NewVal).addReg(ZERO);
  BuildMI(loop2MBB, DL, TII->get(SC), Scratch)
    .addReg(Scratch).addReg(Ptr).addImm(0);
  BuildMI(loop2MBB, DL, TII->get(BEQ))
    .addReg(Scratch, RegState::Kill).addReg(ZERO).addMBB(loop1MBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 291-294
```cpp
  LivePhysRegs LiveRegs;
  computeAndAddLiveIns(LiveRegs, *loop1MBB);
  computeAndAddLiveIns(LiveRegs, *loop2MBB);
  computeAndAddLiveIns(LiveRegs, *exitMBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 296-299
```cpp
  NMBBI = BB.end();
  I->eraseFromParent();
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 301-303
```cpp
bool MipsExpandPseudo::expandAtomicBinOpSubword(
    MachineBasicBlock &BB, MachineBasicBlock::iterator I,
    MachineBasicBlock::iterator &NMBBI) {
```
- EN: Implements `MipsExpandPseudo::expandAtomicBinOpSubword`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsExpandPseudo::expandAtomicBinOpSubword`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 305-305
```cpp
  MachineFunction *MF = BB.getParent();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 307-308
```cpp
  const bool ArePtrs64bit = STI->getABI().ArePtrs64bit();
  DebugLoc DL = I->getDebugLoc();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 310-312
```cpp
  unsigned LL, SC, SLT, SLTu, OR, MOVN, MOVZ, SELNEZ, SELEQZ;
  unsigned BEQ = Mips::BEQ;
  unsigned SEOp = Mips::SEH;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 314-328
```cpp
  if (STI->inMicroMipsMode()) {
      LL = STI->hasMips32r6() ? Mips::LL_MMR6 : Mips::LL_MM;
      SC = STI->hasMips32r6() ? Mips::SC_MMR6 : Mips::SC_MM;
      BEQ = STI->hasMips32r6() ? Mips::BEQC_MMR6 : Mips::BEQ_MM;
      SLT = Mips::SLT_MM;
      SLTu = Mips::SLTu_MM;
      OR = STI->hasMips32r6() ? Mips::OR_MMR6 : Mips::OR_MM;
      MOVN = Mips::MOVN_I_MM;
      MOVZ = Mips::MOVZ_I_MM;
      SELNEZ = STI->hasMips32r6() ? Mips::SELNEZ_MMR6 : Mips::SELNEZ;
      SELEQZ = STI->hasMips32r6() ? Mips::SELEQZ_MMR6 : Mips::SELEQZ;
  } else {
    LL = STI->hasMips32r6() ? (ArePtrs64bit ? Mips::LL64_R6 : Mips::LL_R6)
                            : (ArePtrs64bit ? Mips::LL64 : Mips::LL);
    SC = STI->hasMips32r6() ? (ArePtrs64bit ? Mips::SC64_R6 : Mips::SC_R6)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 329-337
```cpp
                            : (ArePtrs64bit ? Mips::SC64 : Mips::SC);
    SLT = Mips::SLT;
    SLTu = Mips::SLTu;
    OR = Mips::OR;
    MOVN = Mips::MOVN_I_I;
    MOVZ = Mips::MOVZ_I_I;
    SELNEZ = Mips::SELNEZ;
    SELEQZ = Mips::SELEQZ;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 339-344
```cpp
  bool IsSwap = false;
  bool IsNand = false;
  bool IsMin = false;
  bool IsMax = false;
  bool IsUnsigned = false;
  bool DestOK = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 346-360
```cpp
  unsigned Opcode = 0;
  switch (I->getOpcode()) {
  case Mips::ATOMIC_LOAD_NAND_I8_POSTRA:
    SEOp = Mips::SEB;
    [[fallthrough]];
  case Mips::ATOMIC_LOAD_NAND_I16_POSTRA:
    IsNand = true;
    break;
  case Mips::ATOMIC_SWAP_I8_POSTRA:
    SEOp = Mips::SEB;
    [[fallthrough]];
  case Mips::ATOMIC_SWAP_I16_POSTRA:
    IsSwap = true;
    break;
  case Mips::ATOMIC_LOAD_ADD_I8_POSTRA:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 361-375
```cpp
    SEOp = Mips::SEB;
    [[fallthrough]];
  case Mips::ATOMIC_LOAD_ADD_I16_POSTRA:
    Opcode = Mips::ADDu;
    break;
  case Mips::ATOMIC_LOAD_SUB_I8_POSTRA:
    SEOp = Mips::SEB;
    [[fallthrough]];
  case Mips::ATOMIC_LOAD_SUB_I16_POSTRA:
    Opcode = Mips::SUBu;
    break;
  case Mips::ATOMIC_LOAD_AND_I8_POSTRA:
    SEOp = Mips::SEB;
    [[fallthrough]];
  case Mips::ATOMIC_LOAD_AND_I16_POSTRA:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 376-390
```cpp
    Opcode = Mips::AND;
    break;
  case Mips::ATOMIC_LOAD_OR_I8_POSTRA:
    SEOp = Mips::SEB;
    [[fallthrough]];
  case Mips::ATOMIC_LOAD_OR_I16_POSTRA:
    Opcode = Mips::OR;
    break;
  case Mips::ATOMIC_LOAD_XOR_I8_POSTRA:
    SEOp = Mips::SEB;
    [[fallthrough]];
  case Mips::ATOMIC_LOAD_XOR_I16_POSTRA:
    Opcode = Mips::XOR;
    break;
  case Mips::ATOMIC_LOAD_UMIN_I8_POSTRA:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 391-405
```cpp
    SEOp = Mips::SEB;
    IsUnsigned = true;
    IsMin = true;
    break;
  case Mips::ATOMIC_LOAD_UMIN_I16_POSTRA:
    IsUnsigned = true;
    IsMin = true;
    break;
  case Mips::ATOMIC_LOAD_MIN_I8_POSTRA:
    SEOp = Mips::SEB;
    IsMin = true;
    break;
  case Mips::ATOMIC_LOAD_MIN_I16_POSTRA:
    IsMin = true;
    break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 406-420
```cpp
  case Mips::ATOMIC_LOAD_UMAX_I8_POSTRA:
    SEOp = Mips::SEB;
    IsUnsigned = true;
    IsMax = true;
    break;
  case Mips::ATOMIC_LOAD_UMAX_I16_POSTRA:
    IsUnsigned = true;
    IsMax = true;
    break;
  case Mips::ATOMIC_LOAD_MAX_I8_POSTRA:
    SEOp = Mips::SEB;
    IsMax = true;
    break;
  case Mips::ATOMIC_LOAD_MAX_I16_POSTRA:
    IsMax = true;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 421-424
```cpp
    break;
  default:
    llvm_unreachable("Unknown subword atomic pseudo for expansion!");
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 426-435
```cpp
  Register Dest = I->getOperand(0).getReg();
  Register Ptr = I->getOperand(1).getReg();
  Register Incr = I->getOperand(2).getReg();
  Register Mask = I->getOperand(3).getReg();
  Register Mask2 = I->getOperand(4).getReg();
  Register ShiftAmnt = I->getOperand(5).getReg();
  Register OldVal = I->getOperand(6).getReg();
  Register BinOpRes = I->getOperand(7).getReg();
  Register StoreVal = I->getOperand(8).getReg();
  bool NoMovnInstr = (IsMin || IsMax) && !STI->hasMips4() && !STI->hasMips32();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 437-451
```cpp
  const BasicBlock *LLVM_BB = BB.getBasicBlock();
  MachineBasicBlock *loopMBB = MF->CreateMachineBasicBlock(LLVM_BB);
  MachineBasicBlock *loop1MBB = nullptr;
  MachineBasicBlock *loop2MBB = nullptr;
  if (NoMovnInstr) {
    loop1MBB = MF->CreateMachineBasicBlock(LLVM_BB);
    loop2MBB = MF->CreateMachineBasicBlock(LLVM_BB);
  }
  MachineBasicBlock *sinkMBB = MF->CreateMachineBasicBlock(LLVM_BB);
  MachineBasicBlock *exitMBB = MF->CreateMachineBasicBlock(LLVM_BB);
  MachineFunction::iterator It = ++BB.getIterator();
  MF->insert(It, loopMBB);
  if (NoMovnInstr) {
    MF->insert(It, loop1MBB);
    MF->insert(It, loop2MBB);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 452-454
```cpp
  }
  MF->insert(It, sinkMBB);
  MF->insert(It, exitMBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 456-457
```cpp
  exitMBB->splice(exitMBB->begin(), &BB, std::next(I), BB.end());
  exitMBB->transferSuccessorsAndUpdatePHIs(&BB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 459-472
```cpp
  BB.addSuccessor(loopMBB, BranchProbability::getOne());
  if (NoMovnInstr) {
    loopMBB->addSuccessor(loop1MBB);
    loopMBB->addSuccessor(loop2MBB);
  } else {
    loopMBB->addSuccessor(sinkMBB);
    loopMBB->addSuccessor(loopMBB);
    loopMBB->normalizeSuccProbs();
  }
  if (NoMovnInstr) {
    loop1MBB->addSuccessor(loop2MBB);
    loop2MBB->addSuccessor(loopMBB);
    loop2MBB->addSuccessor(sinkMBB);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 474-488
```cpp
  BuildMI(loopMBB, DL, TII->get(LL), OldVal).addReg(Ptr).addImm(0);
  if (IsNand) {
    //  and andres, oldval, incr2
    //  nor binopres, $0, andres
    //  and newval, binopres, mask
    BuildMI(loopMBB, DL, TII->get(Mips::AND), BinOpRes)
        .addReg(OldVal)
        .addReg(Incr);
    BuildMI(loopMBB, DL, TII->get(Mips::NOR), BinOpRes)
        .addReg(Mips::ZERO)
        .addReg(BinOpRes);
    BuildMI(loopMBB, DL, TII->get(Mips::AND), BinOpRes)
        .addReg(BinOpRes)
        .addReg(Mask);
  } else if (IsMin || IsMax) {
```
- EN: Implements `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 490-492
```cpp
    assert(I->getNumOperands() == 10 &&
           "Atomics min|max|umin|umax use an additional register");
    Register Scratch4 = I->getOperand(9).getReg();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 494-497
```cpp
    unsigned SLTScratch4 = IsUnsigned ? SLTu : SLT;
    unsigned SELIncr = IsMax ? SELNEZ : SELEQZ;
    unsigned SELOldVal = IsMax ? SELEQZ : SELNEZ;
    unsigned MOVIncr = IsMax ? MOVN : MOVZ;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 499-513
```cpp
    BuildMI(loopMBB, DL, TII->get(Mips::SRAV), StoreVal)
        .addReg(OldVal)
        .addReg(ShiftAmnt);
    if (IsUnsigned) {
      const unsigned OpMask = SEOp == Mips::SEH ? 0xffff : 0xff;
      BuildMI(loopMBB, DL, TII->get(Mips::ANDi), StoreVal)
          .addReg(StoreVal)
          .addImm(OpMask);
    } else if (STI->hasMips32r2()) {
      BuildMI(loopMBB, DL, TII->get(SEOp), StoreVal).addReg(StoreVal);
    } else {
      const unsigned ShiftImm = SEOp == Mips::SEH ? 16 : 24;
      const unsigned SROp = IsUnsigned ? Mips::SRL : Mips::SRA;
      BuildMI(loopMBB, DL, TII->get(Mips::SLL), StoreVal)
          .addReg(StoreVal, RegState::Kill)
```
- EN: Implements `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 514-525
```cpp
          .addImm(ShiftImm);
      BuildMI(loopMBB, DL, TII->get(SROp), StoreVal)
          .addReg(StoreVal, RegState::Kill)
          .addImm(ShiftImm);
    }
    BuildMI(loopMBB, DL, TII->get(Mips::OR), Dest)
        .addReg(Mips::ZERO)
        .addReg(StoreVal);
    DestOK = true;
    BuildMI(loopMBB, DL, TII->get(Mips::SLLV), StoreVal)
        .addReg(StoreVal)
        .addReg(ShiftAmnt);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 527-531
```cpp
    // unsigned: sltu Scratch4, StoreVal, Incr
    // signed:   slt Scratch4, StoreVal, Incr
    BuildMI(loopMBB, DL, TII->get(SLTScratch4), Scratch4)
        .addReg(StoreVal)
        .addReg(Incr);
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 533-547
```cpp
    if (STI->hasMips64r6() || STI->hasMips32r6()) {
      // max: seleqz BinOpRes, OldVal, Scratch4
      //      selnez Scratch4, Incr, Scratch4
      //      or BinOpRes, BinOpRes, Scratch4
      // min: selnqz BinOpRes, OldVal, Scratch4
      //      seleqz Scratch4, Incr, Scratch4
      //      or BinOpRes, BinOpRes, Scratch4
      BuildMI(loopMBB, DL, TII->get(SELOldVal), BinOpRes)
          .addReg(StoreVal)
          .addReg(Scratch4);
      BuildMI(loopMBB, DL, TII->get(SELIncr), Scratch4)
          .addReg(Incr)
          .addReg(Scratch4);
      BuildMI(loopMBB, DL, TII->get(OR), BinOpRes)
          .addReg(BinOpRes)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 548-562
```cpp
          .addReg(Scratch4);
    } else if (STI->hasMips4() || STI->hasMips32()) {
      // max: move BinOpRes, StoreVal
      //      movn BinOpRes, Incr, Scratch4, BinOpRes
      // min: move BinOpRes, StoreVal
      //      movz BinOpRes, Incr, Scratch4, BinOpRes
      BuildMI(loopMBB, DL, TII->get(OR), BinOpRes)
          .addReg(StoreVal)
          .addReg(Mips::ZERO);
      BuildMI(loopMBB, DL, TII->get(MOVIncr), BinOpRes)
          .addReg(Incr)
          .addReg(Scratch4)
          .addReg(BinOpRes);
    } else {
      // if min:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 563-577
```cpp
      // loopMBB:  move BinOpRes, StoreVal
      //           beq Scratch4, 0, loop1MBB
      //           j loop2MBB
      // loop1MBB: move BinOpRes, Incr
      // loop2MBB: and BinOpRes, BinOpRes, Mask
      //           and StoreVal, OlddVal, Mask2
      //           or StoreVal, StoreVal, BinOpRes
      //           StoreVal<tied1> = sc StoreVal, 0(Ptr)
      //           beq StoreVal, zero, loopMBB
      //
      // if max:
      // loopMBB:  move BinOpRes, Incr
      //           beq Scratch4, 0, loop1MBB
      //           j loop2MBB
      // loop1MBB: move BinOpRes, StoreVal
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 578-592
```cpp
      // loop2MBB: and BinOpRes, BinOpRes, Mask
      //           and StoreVal, OlddVal, Mask2
      //           or StoreVal, StoreVal, BinOpRes
      //           StoreVal<tied1> = sc StoreVal, 0(Ptr)
      //           beq StoreVal, zero, loopMBB
      if (IsMin) {
        BuildMI(loopMBB, DL, TII->get(OR), BinOpRes)
            .addReg(StoreVal)
            .addReg(Mips::ZERO);
        BuildMI(loop1MBB, DL, TII->get(OR), BinOpRes)
            .addReg(Incr)
            .addReg(Mips::ZERO);
      } else {
        BuildMI(loopMBB, DL, TII->get(OR), BinOpRes)
            .addReg(Incr)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 593-603
```cpp
            .addReg(Mips::ZERO);
        BuildMI(loop1MBB, DL, TII->get(OR), BinOpRes)
            .addReg(StoreVal)
            .addReg(Mips::ZERO);
      }
      BuildMI(loopMBB, DL, TII->get(BEQ))
          .addReg(Scratch4)
          .addReg(Mips::ZERO)
          .addMBB(loop1MBB);
      BuildMI(loopMBB, DL, TII->get(Mips::J)).addMBB(loop2MBB);
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 605-613
```cpp
    //  and BinOpRes, BinOpRes, Mask
    if (NoMovnInstr)
      BuildMI(loop2MBB, DL, TII->get(Mips::AND), BinOpRes)
          .addReg(BinOpRes)
          .addReg(Mask);
    else
      BuildMI(loopMBB, DL, TII->get(Mips::AND), BinOpRes)
          .addReg(BinOpRes)
          .addReg(Mask);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 615-629
```cpp
  } else if (!IsSwap) {
    //  <binop> binopres, oldval, incr2
    //  and newval, binopres, mask
    BuildMI(loopMBB, DL, TII->get(Opcode), BinOpRes)
        .addReg(OldVal)
        .addReg(Incr);
    BuildMI(loopMBB, DL, TII->get(Mips::AND), BinOpRes)
        .addReg(BinOpRes)
        .addReg(Mask);
  } else { // atomic.swap
    //  and newval, incr2, mask
    BuildMI(loopMBB, DL, TII->get(Mips::AND), BinOpRes)
        .addReg(Incr)
        .addReg(Mask);
  }
```
- EN: Implements `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 631-645
```cpp
  // and StoreVal, OlddVal, Mask2
  // or StoreVal, StoreVal, BinOpRes
  // StoreVal<tied1> = sc StoreVal, 0(Ptr)
  // beq StoreVal, zero, loopMBB
  if (NoMovnInstr) {
    BuildMI(loop2MBB, DL, TII->get(Mips::AND), StoreVal)
        .addReg(OldVal)
        .addReg(Mask2);
    BuildMI(loop2MBB, DL, TII->get(Mips::OR), StoreVal)
        .addReg(StoreVal)
        .addReg(BinOpRes);
    BuildMI(loop2MBB, DL, TII->get(SC), StoreVal)
        .addReg(StoreVal)
        .addReg(Ptr)
        .addImm(0);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 646-660
```cpp
    BuildMI(loop2MBB, DL, TII->get(BEQ))
        .addReg(StoreVal)
        .addReg(Mips::ZERO)
        .addMBB(loopMBB);
  } else {
    BuildMI(loopMBB, DL, TII->get(Mips::AND), StoreVal)
        .addReg(OldVal)
        .addReg(Mask2);
    BuildMI(loopMBB, DL, TII->get(Mips::OR), StoreVal)
        .addReg(StoreVal)
        .addReg(BinOpRes);
    BuildMI(loopMBB, DL, TII->get(SC), StoreVal)
        .addReg(StoreVal)
        .addReg(Ptr)
        .addImm(0);
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 661-665
```cpp
    BuildMI(loopMBB, DL, TII->get(BEQ))
        .addReg(StoreVal)
        .addReg(Mips::ZERO)
        .addMBB(loopMBB);
  }
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 667-670
```cpp
  //  sinkMBB:
  //    and     maskedoldval1,oldval,mask
  //    srl     srlres,maskedoldval1,shiftamt
  //    sign_extend dest,srlres
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 672-677
```cpp
  if (!DestOK) {
    sinkMBB->addSuccessor(exitMBB, BranchProbability::getOne());
    BuildMI(sinkMBB, DL, TII->get(Mips::AND), Dest).addReg(OldVal).addReg(Mask);
    BuildMI(sinkMBB, DL, TII->get(Mips::SRLV), Dest)
        .addReg(Dest)
        .addReg(ShiftAmnt);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 679-690
```cpp
    if (STI->hasMips32r2()) {
      BuildMI(sinkMBB, DL, TII->get(SEOp), Dest).addReg(Dest);
    } else {
      const unsigned ShiftImm = SEOp == Mips::SEH ? 16 : 24;
      BuildMI(sinkMBB, DL, TII->get(Mips::SLL), Dest)
          .addReg(Dest, RegState::Kill)
          .addImm(ShiftImm);
      BuildMI(sinkMBB, DL, TII->get(Mips::SRA), Dest)
          .addReg(Dest, RegState::Kill)
          .addImm(ShiftImm);
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 692-700
```cpp
  LivePhysRegs LiveRegs;
  computeAndAddLiveIns(LiveRegs, *loopMBB);
  if (loop1MBB) {
    assert(loop2MBB && "should have 2 loop blocks");
    computeAndAddLiveIns(LiveRegs, *loop1MBB);
    computeAndAddLiveIns(LiveRegs, *loop2MBB);
  }
  computeAndAddLiveIns(LiveRegs, *sinkMBB);
  computeAndAddLiveIns(LiveRegs, *exitMBB);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 702-703
```cpp
  NMBBI = BB.end();
  I->eraseFromParent();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 705-706
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 708-712
```cpp
bool MipsExpandPseudo::expandAtomicBinOp(MachineBasicBlock &BB,
                                         MachineBasicBlock::iterator I,
                                         MachineBasicBlock::iterator &NMBBI,
                                         unsigned Size) {
  MachineFunction *MF = BB.getParent();
```
- EN: Implements `MipsExpandPseudo::expandAtomicBinOp`, a mutation/build routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsExpandPseudo::expandAtomicBinOp`，它是一个围绕机器函数状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 714-715
```cpp
  const bool ArePtrs64bit = STI->getABI().ArePtrs64bit();
  DebugLoc DL = I->getDebugLoc();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 717-717
```cpp
  unsigned LL, SC, ZERO, BEQ, SLT, SLTu, OR, MOVN, MOVZ, SELNEZ, SELEQZ;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 719-733
```cpp
  if (Size == 4) {
    if (STI->inMicroMipsMode()) {
      LL = STI->hasMips32r6() ? Mips::LL_MMR6 : Mips::LL_MM;
      SC = STI->hasMips32r6() ? Mips::SC_MMR6 : Mips::SC_MM;
      BEQ = STI->hasMips32r6() ? Mips::BEQC_MMR6 : Mips::BEQ_MM;
      SLT = Mips::SLT_MM;
      SLTu = Mips::SLTu_MM;
      OR = STI->hasMips32r6() ? Mips::OR_MMR6 : Mips::OR_MM;
      MOVN = Mips::MOVN_I_MM;
      MOVZ = Mips::MOVZ_I_MM;
      SELNEZ = STI->hasMips32r6() ? Mips::SELNEZ_MMR6 : Mips::SELNEZ;
      SELEQZ = STI->hasMips32r6() ? Mips::SELEQZ_MMR6 : Mips::SELEQZ;
    } else {
      LL = STI->hasMips32r6()
               ? (ArePtrs64bit ? Mips::LL64_R6 : Mips::LL_R6)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 734-746
```cpp
               : (ArePtrs64bit ? Mips::LL64 : Mips::LL);
      SC = STI->hasMips32r6()
               ? (ArePtrs64bit ? Mips::SC64_R6 : Mips::SC_R6)
               : (ArePtrs64bit ? Mips::SC64 : Mips::SC);
      BEQ = Mips::BEQ;
      SLT = Mips::SLT;
      SLTu = Mips::SLTu;
      OR = Mips::OR;
      MOVN = Mips::MOVN_I_I;
      MOVZ = Mips::MOVZ_I_I;
      SELNEZ = Mips::SELNEZ;
      SELEQZ = Mips::SELEQZ;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 748-761
```cpp
    ZERO = Mips::ZERO;
  } else {
    LL = STI->hasMips64r6() ? Mips::LLD_R6 : Mips::LLD;
    SC = STI->hasMips64r6() ? Mips::SCD_R6 : Mips::SCD;
    ZERO = Mips::ZERO_64;
    BEQ = Mips::BEQ64;
    SLT = Mips::SLT64;
    SLTu = Mips::SLTu64;
    OR = Mips::OR64;
    MOVN = Mips::MOVN_I64_I64;
    MOVZ = Mips::MOVZ_I64_I64;
    SELNEZ = Mips::SELNEZ64;
    SELEQZ = Mips::SELEQZ64;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 763-766
```cpp
  Register OldVal = I->getOperand(0).getReg();
  Register Ptr = I->getOperand(1).getReg();
  Register Incr = I->getOperand(2).getReg();
  Register Scratch = I->getOperand(3).getReg();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 768-770
```cpp
  unsigned Opcode = 0;
  unsigned AND = 0;
  unsigned NOR = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 772-776
```cpp
  bool IsOr = false;
  bool IsNand = false;
  bool IsMin = false;
  bool IsMax = false;
  bool IsUnsigned = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 778-792
```cpp
  switch (I->getOpcode()) {
  case Mips::ATOMIC_LOAD_ADD_I32_POSTRA:
    Opcode = Mips::ADDu;
    break;
  case Mips::ATOMIC_LOAD_SUB_I32_POSTRA:
    Opcode = Mips::SUBu;
    break;
  case Mips::ATOMIC_LOAD_AND_I32_POSTRA:
    Opcode = Mips::AND;
    break;
  case Mips::ATOMIC_LOAD_OR_I32_POSTRA:
    Opcode = Mips::OR;
    break;
  case Mips::ATOMIC_LOAD_XOR_I32_POSTRA:
    Opcode = Mips::XOR;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 793-807
```cpp
    break;
  case Mips::ATOMIC_LOAD_NAND_I32_POSTRA:
    IsNand = true;
    AND = Mips::AND;
    NOR = Mips::NOR;
    break;
  case Mips::ATOMIC_SWAP_I32_POSTRA:
    IsOr = true;
    break;
  case Mips::ATOMIC_LOAD_ADD_I64_POSTRA:
    Opcode = Mips::DADDu;
    break;
  case Mips::ATOMIC_LOAD_SUB_I64_POSTRA:
    Opcode = Mips::DSUBu;
    break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 808-822
```cpp
  case Mips::ATOMIC_LOAD_AND_I64_POSTRA:
    Opcode = Mips::AND64;
    break;
  case Mips::ATOMIC_LOAD_OR_I64_POSTRA:
    Opcode = Mips::OR64;
    break;
  case Mips::ATOMIC_LOAD_XOR_I64_POSTRA:
    Opcode = Mips::XOR64;
    break;
  case Mips::ATOMIC_LOAD_NAND_I64_POSTRA:
    IsNand = true;
    AND = Mips::AND64;
    NOR = Mips::NOR64;
    break;
  case Mips::ATOMIC_SWAP_I64_POSTRA:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 823-837
```cpp
    IsOr = true;
    break;
  case Mips::ATOMIC_LOAD_UMIN_I32_POSTRA:
  case Mips::ATOMIC_LOAD_UMIN_I64_POSTRA:
    IsUnsigned = true;
    [[fallthrough]];
  case Mips::ATOMIC_LOAD_MIN_I32_POSTRA:
  case Mips::ATOMIC_LOAD_MIN_I64_POSTRA:
    IsMin = true;
    break;
  case Mips::ATOMIC_LOAD_UMAX_I32_POSTRA:
  case Mips::ATOMIC_LOAD_UMAX_I64_POSTRA:
    IsUnsigned = true;
    [[fallthrough]];
  case Mips::ATOMIC_LOAD_MAX_I32_POSTRA:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 838-843
```cpp
  case Mips::ATOMIC_LOAD_MAX_I64_POSTRA:
    IsMax = true;
    break;
  default:
    llvm_unreachable("Unknown pseudo atomic!");
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 845-859
```cpp
  bool NoMovnInstr = (IsMin || IsMax) && !STI->hasMips4() && !STI->hasMips32();
  const BasicBlock *LLVM_BB = BB.getBasicBlock();
  MachineBasicBlock *loopMBB = MF->CreateMachineBasicBlock(LLVM_BB);
  MachineBasicBlock *loop1MBB = nullptr;
  MachineBasicBlock *loop2MBB = nullptr;
  if (NoMovnInstr) {
    loop1MBB = MF->CreateMachineBasicBlock(LLVM_BB);
    loop2MBB = MF->CreateMachineBasicBlock(LLVM_BB);
  }
  MachineBasicBlock *exitMBB = MF->CreateMachineBasicBlock(LLVM_BB);
  MachineFunction::iterator It = ++BB.getIterator();
  MF->insert(It, loopMBB);
  if (NoMovnInstr) {
    MF->insert(It, loop1MBB);
    MF->insert(It, loop2MBB);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 860-861
```cpp
  }
  MF->insert(It, exitMBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 863-864
```cpp
  exitMBB->splice(exitMBB->begin(), &BB, std::next(I), BB.end());
  exitMBB->transferSuccessorsAndUpdatePHIs(&BB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 866-879
```cpp
  BB.addSuccessor(loopMBB, BranchProbability::getOne());
  if (NoMovnInstr) {
    loopMBB->addSuccessor(loop1MBB);
    loopMBB->addSuccessor(loop2MBB);
  } else {
    loopMBB->addSuccessor(exitMBB);
    loopMBB->addSuccessor(loopMBB);
  }
  loopMBB->normalizeSuccProbs();
  if (NoMovnInstr) {
    loop1MBB->addSuccessor(loop2MBB);
    loop2MBB->addSuccessor(loopMBB);
    loop2MBB->addSuccessor(exitMBB);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 881-895
```cpp
  BuildMI(loopMBB, DL, TII->get(LL), OldVal).addReg(Ptr).addImm(0);
  assert((OldVal != Ptr) && "Clobbered the wrong ptr reg!");
  assert((OldVal != Incr) && "Clobbered the wrong reg!");
  if (IsMin || IsMax) {
    // Remove trailing kill/dead register operands added by
    // MachineInstr::addRegisterKilled. These are super-register markers that
    // must correspond to one of the physical registers in operands 1-4.
    // The kill/dead markers may also appear on preceding subregs.
    const TargetRegisterInfo *TRI = STI->getRegisterInfo();
    while (I->getNumOperands() > 5) {
      auto &Op = I->getOperand(I->getNumOperands() - 1);
      // Check if this register overlaps with any physical register in
      // operands 1-4 that has kill/dead marker (i.e., it's a super-register
      // marker for subregs).
      bool HasOverlapWithKill = false;
```
- EN: Implements `BuildMI`, a mutation/build routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `BuildMI`，它是一个围绕机器指令展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 896-910
```cpp
      for (unsigned i = 1; i <= 4; ++i) {
        auto &RefOp = I->getOperand(i);
        if (RefOp.isReg() && RefOp.getReg().isPhysical() &&
            TRI->regsOverlap(Op.getReg(), RefOp.getReg()) &&
            (RefOp.isKill() || RefOp.isDead())) {
          HasOverlapWithKill = true;
          break;
        }
      }
      // Remove if HasOverlapWithKill is true or Op has kill/dead marker.
      bool HasKillOrDead = Op.isReg() && Op.getReg().isPhysical() &&
                           (Op.isKill() || Op.isDead());
      if (!HasOverlapWithKill && !HasKillOrDead)
        break;
      I->removeOperand(I->getNumOperands() - 1);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 911-911
```cpp
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 913-915
```cpp
    assert(I->getNumOperands() == 5 &&
           "Atomics min|max|umin|umax use an additional register");
    MCRegister Scratch2 = I->getOperand(4).getReg().asMCReg();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 917-920
```cpp
    // On Mips64 result of slt is GPR32.
    MCRegister Scratch2_32 =
        (Size == 8) ? STI->getRegisterInfo()->getSubReg(Scratch2, Mips::sub_32)
                    : Scratch2;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 922-925
```cpp
    unsigned SLTScratch2 = IsUnsigned ? SLTu : SLT;
    unsigned SELIncr = IsMax ? SELNEZ : SELEQZ;
    unsigned SELOldVal = IsMax ? SELEQZ : SELNEZ;
    unsigned MOVIncr = IsMax ? MOVN : MOVZ;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 927-931
```cpp
    // unsigned: sltu Scratch2, oldVal, Incr
    // signed:   slt Scratch2, oldVal, Incr
    BuildMI(loopMBB, DL, TII->get(SLTScratch2), Scratch2_32)
        .addReg(OldVal)
        .addReg(Incr);
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 933-947
```cpp
    if (STI->hasMips64r6() || STI->hasMips32r6()) {
      // max: seleqz Scratch, OldVal, Scratch2
      //      selnez Scratch2, Incr, Scratch2
      //      or Scratch, Scratch, Scratch2
      // min: selnez Scratch, OldVal, Scratch2
      //      seleqz Scratch2, Incr, Scratch2
      //      or Scratch, Scratch, Scratch2
      BuildMI(loopMBB, DL, TII->get(SELOldVal), Scratch)
          .addReg(OldVal)
          .addReg(Scratch2);
      BuildMI(loopMBB, DL, TII->get(SELIncr), Scratch2)
          .addReg(Incr)
          .addReg(Scratch2);
      BuildMI(loopMBB, DL, TII->get(OR), Scratch)
          .addReg(Scratch)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 948-962
```cpp
          .addReg(Scratch2);
    } else if (STI->hasMips4() || STI->hasMips32()) {
      // max: move Scratch, OldVal
      //      movn Scratch, Incr, Scratch2, Scratch
      // min: move Scratch, OldVal
      //      movz Scratch, Incr, Scratch2, Scratch
      BuildMI(loopMBB, DL, TII->get(OR), Scratch)
          .addReg(OldVal)
          .addReg(ZERO);
      BuildMI(loopMBB, DL, TII->get(MOVIncr), Scratch)
          .addReg(Incr)
          .addReg(Scratch2)
          .addReg(Scratch);
    } else {
      // if min:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 963-977
```cpp
      // loopMBB:  move Scratch, OldVal
      //           beq Scratch2_32, 0, loop1MBB
      //           j loop2MBB
      // loop1MBB: move Scratch, Incr
      // loop2MBB: sc $2, 0($4)
      //           beqz	$2, $BB0_1
      //           nop
      //
      // if max:
      // loopMBB:  move Scratch, Incr
      //           beq Scratch2_32, 0, loop1MBB
      //           j loop2MBB
      // loop1MBB: move Scratch, OldVal
      // loop2MBB: sc $2, 0($4)
      //           beqz	$2, $BB0_1
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 978-992
```cpp
      //           nop
      if (IsMin) {
        BuildMI(loopMBB, DL, TII->get(OR), Scratch).addReg(OldVal).addReg(ZERO);
        BuildMI(loop1MBB, DL, TII->get(OR), Scratch).addReg(Incr).addReg(ZERO);
      } else {
        BuildMI(loopMBB, DL, TII->get(OR), Scratch).addReg(Incr).addReg(ZERO);
        BuildMI(loop1MBB, DL, TII->get(OR), Scratch)
            .addReg(OldVal)
            .addReg(ZERO);
      }
      BuildMI(loopMBB, DL, TII->get(BEQ))
          .addReg(Scratch2_32)
          .addReg(ZERO)
          .addMBB(loop1MBB);
      BuildMI(loopMBB, DL, TII->get(Mips::J)).addMBB(loop2MBB);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 993-993
```cpp
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 995-1006
```cpp
  } else if (Opcode) {
    BuildMI(loopMBB, DL, TII->get(Opcode), Scratch).addReg(OldVal).addReg(Incr);
  } else if (IsNand) {
    assert(AND && NOR &&
           "Unknown nand instruction for atomic pseudo expansion");
    BuildMI(loopMBB, DL, TII->get(AND), Scratch).addReg(OldVal).addReg(Incr);
    BuildMI(loopMBB, DL, TII->get(NOR), Scratch).addReg(ZERO).addReg(Scratch);
  } else {
    assert(IsOr && OR && "Unknown instruction for atomic pseudo expansion!");
    (void)IsOr;
    BuildMI(loopMBB, DL, TII->get(OR), Scratch).addReg(Incr).addReg(ZERO);
  }
```
- EN: Implements `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1008-1022
```cpp
  if (NoMovnInstr) {
    BuildMI(loop2MBB, DL, TII->get(SC), Scratch)
        .addReg(Scratch)
        .addReg(Ptr)
        .addImm(0);
    BuildMI(loop2MBB, DL, TII->get(BEQ))
        .addReg(Scratch)
        .addReg(ZERO)
        .addMBB(loopMBB);
  } else {
    BuildMI(loopMBB, DL, TII->get(SC), Scratch)
        .addReg(Scratch)
        .addReg(Ptr)
        .addImm(0);
    BuildMI(loopMBB, DL, TII->get(BEQ))
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1023-1026
```cpp
        .addReg(Scratch)
        .addReg(ZERO)
        .addMBB(loopMBB);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1028-1029
```cpp
  NMBBI = BB.end();
  I->eraseFromParent();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1031-1038
```cpp
  LivePhysRegs LiveRegs;
  computeAndAddLiveIns(LiveRegs, *loopMBB);
  if (loop1MBB) {
    assert(loop2MBB && "should have 2 loop blocks");
    computeAndAddLiveIns(LiveRegs, *loop1MBB);
    computeAndAddLiveIns(LiveRegs, *loop2MBB);
  }
  computeAndAddLiveIns(LiveRegs, *exitMBB);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1040-1041
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1043-1045
```cpp
bool MipsExpandPseudo::expandMI(MachineBasicBlock &MBB,
                                MachineBasicBlock::iterator MBBI,
                                MachineBasicBlock::iterator &NMBB) {
```
- EN: Implements `MipsExpandPseudo::expandMI`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsExpandPseudo::expandMI`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1047-1047
```cpp
  bool Modified = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1049-1063
```cpp
  switch (MBBI->getOpcode()) {
  case Mips::ATOMIC_CMP_SWAP_I32_POSTRA:
  case Mips::ATOMIC_CMP_SWAP_I64_POSTRA:
    return expandAtomicCmpSwap(MBB, MBBI, NMBB);
  case Mips::ATOMIC_CMP_SWAP_I8_POSTRA:
  case Mips::ATOMIC_CMP_SWAP_I16_POSTRA:
    return expandAtomicCmpSwapSubword(MBB, MBBI, NMBB);
  case Mips::ATOMIC_SWAP_I8_POSTRA:
  case Mips::ATOMIC_SWAP_I16_POSTRA:
  case Mips::ATOMIC_LOAD_NAND_I8_POSTRA:
  case Mips::ATOMIC_LOAD_NAND_I16_POSTRA:
  case Mips::ATOMIC_LOAD_ADD_I8_POSTRA:
  case Mips::ATOMIC_LOAD_ADD_I16_POSTRA:
  case Mips::ATOMIC_LOAD_SUB_I8_POSTRA:
  case Mips::ATOMIC_LOAD_SUB_I16_POSTRA:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1064-1078
```cpp
  case Mips::ATOMIC_LOAD_AND_I8_POSTRA:
  case Mips::ATOMIC_LOAD_AND_I16_POSTRA:
  case Mips::ATOMIC_LOAD_OR_I8_POSTRA:
  case Mips::ATOMIC_LOAD_OR_I16_POSTRA:
  case Mips::ATOMIC_LOAD_XOR_I8_POSTRA:
  case Mips::ATOMIC_LOAD_XOR_I16_POSTRA:
  case Mips::ATOMIC_LOAD_MIN_I8_POSTRA:
  case Mips::ATOMIC_LOAD_MIN_I16_POSTRA:
  case Mips::ATOMIC_LOAD_MAX_I8_POSTRA:
  case Mips::ATOMIC_LOAD_MAX_I16_POSTRA:
  case Mips::ATOMIC_LOAD_UMIN_I8_POSTRA:
  case Mips::ATOMIC_LOAD_UMIN_I16_POSTRA:
  case Mips::ATOMIC_LOAD_UMAX_I8_POSTRA:
  case Mips::ATOMIC_LOAD_UMAX_I16_POSTRA:
    return expandAtomicBinOpSubword(MBB, MBBI, NMBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1079-1093
```cpp
  case Mips::ATOMIC_LOAD_ADD_I32_POSTRA:
  case Mips::ATOMIC_LOAD_SUB_I32_POSTRA:
  case Mips::ATOMIC_LOAD_AND_I32_POSTRA:
  case Mips::ATOMIC_LOAD_OR_I32_POSTRA:
  case Mips::ATOMIC_LOAD_XOR_I32_POSTRA:
  case Mips::ATOMIC_LOAD_NAND_I32_POSTRA:
  case Mips::ATOMIC_SWAP_I32_POSTRA:
  case Mips::ATOMIC_LOAD_MIN_I32_POSTRA:
  case Mips::ATOMIC_LOAD_MAX_I32_POSTRA:
  case Mips::ATOMIC_LOAD_UMIN_I32_POSTRA:
  case Mips::ATOMIC_LOAD_UMAX_I32_POSTRA:
    return expandAtomicBinOp(MBB, MBBI, NMBB, 4);
  case Mips::ATOMIC_LOAD_ADD_I64_POSTRA:
  case Mips::ATOMIC_LOAD_SUB_I64_POSTRA:
  case Mips::ATOMIC_LOAD_AND_I64_POSTRA:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1094-1106
```cpp
  case Mips::ATOMIC_LOAD_OR_I64_POSTRA:
  case Mips::ATOMIC_LOAD_XOR_I64_POSTRA:
  case Mips::ATOMIC_LOAD_NAND_I64_POSTRA:
  case Mips::ATOMIC_SWAP_I64_POSTRA:
  case Mips::ATOMIC_LOAD_MIN_I64_POSTRA:
  case Mips::ATOMIC_LOAD_MAX_I64_POSTRA:
  case Mips::ATOMIC_LOAD_UMIN_I64_POSTRA:
  case Mips::ATOMIC_LOAD_UMAX_I64_POSTRA:
    return expandAtomicBinOp(MBB, MBBI, NMBB, 8);
  default:
    return Modified;
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1108-1109
```cpp
bool MipsExpandPseudo::expandMBB(MachineBasicBlock &MBB) {
  bool Modified = false;
```
- EN: Implements `MipsExpandPseudo::expandMBB`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsExpandPseudo::expandMBB`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1111-1116
```cpp
  MachineBasicBlock::iterator MBBI = MBB.begin(), E = MBB.end();
  while (MBBI != E) {
    MachineBasicBlock::iterator NMBBI = std::next(MBBI);
    Modified |= expandMI(MBB, MBBI, NMBBI);
    MBBI = NMBBI;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1118-1119
```cpp
  return Modified;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1121-1123
```cpp
bool MipsExpandPseudo::runOnMachineFunction(MachineFunction &MF) {
  STI = &MF.getSubtarget<MipsSubtarget>();
  TII = STI->getInstrInfo();
```
- EN: Implements `MipsExpandPseudo::runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsExpandPseudo::runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1125-1127
```cpp
  bool Modified = false;
  for (MachineBasicBlock &MBB : MF)
    Modified |= expandMBB(MBB);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1129-1130
```cpp
  if (Modified)
    MF.RenumberBlocks();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1132-1133
```cpp
  return Modified;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1135-1139
```cpp
/// createMipsExpandPseudoPass - returns an instance of the pseudo instruction
/// expansion pass.
FunctionPass *llvm::createMipsExpandPseudoPass() {
  return new MipsExpandPseudo();
}
```
- EN: Implements `llvm::createMipsExpandPseudoPass`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::createMipsExpandPseudoPass`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

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

- EN: Backend-local headers: `Mips.h`, `MipsInstrInfo.h`, `MipsSubtarget.h`.
  - CN: 后端本地头文件：`Mips.h`, `MipsInstrInfo.h`, `MipsSubtarget.h`。
- EN: LLVM infrastructure headers: `llvm/CodeGen/LivePhysRegs.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineInstrBuilder.h`.
  - CN: LLVM 基础设施头文件：`llvm/CodeGen/LivePhysRegs.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineInstrBuilder.h`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
