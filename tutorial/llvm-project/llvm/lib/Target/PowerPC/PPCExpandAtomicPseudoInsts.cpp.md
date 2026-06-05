# PPCExpandAtomicPseudoInsts.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCExpandAtomicPseudoInsts.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides backend implementation logic for the PowerPC backend.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCExpandAtomicPseudoInsts.cpp`，主要负责 PowerPC 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- PPCExpandAtomicPseudoInsts.cpp - Expand atomic pseudo instrs. -----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-12

```cpp
//===----------------------------------------------------------------------===//
//
// This file contains a pass that expands atomic pseudo instructions into
// target instructions post RA. With such method, LL/SC loop is considered as
// a whole blob and make spilling unlikely happens in the LL/SC loop.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This file contains a pass that expands atomic pseudo instructions into".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This file contains a pass that expands atomic pseudo instructions into”。

### Lines 13-19

```cpp
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/PPCPredicates.h"
#include "PPC.h"
#include "PPCInstrInfo.h"

#include "llvm/CodeGen/LivePhysRegs.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Register assignment and register-class constraints matter here.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 20-26

```cpp
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"

using namespace llvm;

#define DEBUG_TYPE "ppc-atomic-expand"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 27-64

```cpp
namespace {

class PPCExpandAtomicPseudo : public MachineFunctionPass {
public:
  const PPCInstrInfo *TII;
  const PPCRegisterInfo *TRI;
  static char ID;

  PPCExpandAtomicPseudo() : MachineFunctionPass(ID) {}

  bool runOnMachineFunction(MachineFunction &MF) override;

private:
  bool expandMI(MachineBasicBlock &MBB, MachineInstr &MI,
                MachineBasicBlock::iterator &NMBBI);
  bool expandAtomicRMW128(MachineBasicBlock &MBB, MachineInstr &MI,
                          MachineBasicBlock::iterator &NMBBI);
  bool expandAtomicCmpSwap128(MachineBasicBlock &MBB, MachineInstr &MI,
                              MachineBasicBlock::iterator &NMBBI);
};

static void PairedCopy(const PPCInstrInfo *TII, MachineBasicBlock &MBB,
                       MachineBasicBlock::iterator MBBI, const DebugLoc &DL,
                       Register Dest0, Register Dest1, Register Src0,
                       Register Src1) {
  const MCInstrDesc &OR = TII->get(PPC::OR8);
  const MCInstrDesc &XOR = TII->get(PPC::XOR8);
  if (Dest0 == Src1 && Dest1 == Src0) {
    // The most tricky case, swapping values.
    BuildMI(MBB, MBBI, DL, XOR, Dest0).addReg(Dest0).addReg(Dest1);
    BuildMI(MBB, MBBI, DL, XOR, Dest1).addReg(Dest0).addReg(Dest1);
    BuildMI(MBB, MBBI, DL, XOR, Dest0).addReg(Dest0).addReg(Dest1);
  } else if (Dest0 != Src0 || Dest1 != Src1) {
    if (Dest0 == Src1 || Dest1 != Src0) {
      BuildMI(MBB, MBBI, DL, OR, Dest1).addReg(Src1).addReg(Src1);
      BuildMI(MBB, MBBI, DL, OR, Dest0).addReg(Src0).addReg(Src0);
    } else {
      BuildMI(MBB, MBBI, DL, OR, Dest0).addReg(Src0).addReg(Src0);
```
- **EN**: Declares a backend-facing type `PPCExpandAtomicPseudo`, `MachineFunctionPass`, `runOnMachineFunction` and outlines the API or state that nearby code will rely on. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里声明面向后端的类型 `PPCExpandAtomicPseudo`, `MachineFunctionPass`, `runOnMachineFunction`，并勾勒出周边代码会依赖的接口或状态。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 65-102

```cpp
      BuildMI(MBB, MBBI, DL, OR, Dest1).addReg(Src1).addReg(Src1);
    }
  }
}

bool PPCExpandAtomicPseudo::runOnMachineFunction(MachineFunction &MF) {
  bool Changed = false;
  TII = static_cast<const PPCInstrInfo *>(MF.getSubtarget().getInstrInfo());
  TRI = &TII->getRegisterInfo();
  for (MachineBasicBlock &MBB : MF) {
    for (MachineBasicBlock::iterator MBBI = MBB.begin(), MBBE = MBB.end();
         MBBI != MBBE;) {
      MachineInstr &MI = *MBBI;
      MachineBasicBlock::iterator NMBBI = std::next(MBBI);
      Changed |= expandMI(MBB, MI, NMBBI);
      MBBI = NMBBI;
    }
  }
  if (Changed)
    MF.RenumberBlocks();
  return Changed;
}

bool PPCExpandAtomicPseudo::expandMI(MachineBasicBlock &MBB, MachineInstr &MI,
                                     MachineBasicBlock::iterator &NMBBI) {
  switch (MI.getOpcode()) {
  case PPC::ATOMIC_SWAP_I128:
  case PPC::ATOMIC_LOAD_ADD_I128:
  case PPC::ATOMIC_LOAD_SUB_I128:
  case PPC::ATOMIC_LOAD_XOR_I128:
  case PPC::ATOMIC_LOAD_NAND_I128:
  case PPC::ATOMIC_LOAD_AND_I128:
  case PPC::ATOMIC_LOAD_OR_I128:
    return expandAtomicRMW128(MBB, MI, NMBBI);
  case PPC::ATOMIC_CMP_SWAP_I128:
    return expandAtomicCmpSwap128(MBB, MI, NMBBI);
  case PPC::BUILD_QUADWORD: {
    Register Dst = MI.getOperand(0).getReg();
```
- **EN**: Implements helper routine(s) `BuildMI`, `addReg`, `runOnMachineFunction` for this portion of the PowerPC backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `BuildMI`, `addReg`, `runOnMachineFunction`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 103-140

```cpp
    Register DstHi = TRI->getSubReg(Dst, PPC::sub_gp8_x0);
    Register DstLo = TRI->getSubReg(Dst, PPC::sub_gp8_x1);
    Register Lo = MI.getOperand(1).getReg();
    Register Hi = MI.getOperand(2).getReg();
    PairedCopy(TII, MBB, MI, MI.getDebugLoc(), DstHi, DstLo, Hi, Lo);
    MI.eraseFromParent();
    return true;
  }
  default:
    return false;
  }
}

bool PPCExpandAtomicPseudo::expandAtomicRMW128(
    MachineBasicBlock &MBB, MachineInstr &MI,
    MachineBasicBlock::iterator &NMBBI) {
  const MCInstrDesc &LL = TII->get(PPC::LQARX);
  const MCInstrDesc &SC = TII->get(PPC::STQCX);
  DebugLoc DL = MI.getDebugLoc();
  MachineFunction *MF = MBB.getParent();
  const BasicBlock *BB = MBB.getBasicBlock();
  // Create layout of control flow.
  MachineFunction::iterator MFI = ++MBB.getIterator();
  MachineBasicBlock *LoopMBB = MF->CreateMachineBasicBlock(BB);
  MachineBasicBlock *ExitMBB = MF->CreateMachineBasicBlock(BB);
  MF->insert(MFI, LoopMBB);
  MF->insert(MFI, ExitMBB);
  ExitMBB->splice(ExitMBB->begin(), &MBB, std::next(MI.getIterator()),
                  MBB.end());
  ExitMBB->transferSuccessorsAndUpdatePHIs(&MBB);
  MBB.addSuccessor(LoopMBB);

  // For non-min/max operations, control flow is kinda like:
  // MBB:
  //   ...
  // LoopMBB:
  //   lqarx in, ptr
  //   addc out.sub_x1, in.sub_x1, op.sub_x1
```
- **EN**: Implements helper routine(s) `getSubReg`, `getOperand`, `getReg` for this portion of the PowerPC backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `getSubReg`, `getOperand`, `getReg`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 141-178

```cpp
  //   adde out.sub_x0, in.sub_x0, op.sub_x0
  //   stqcx out, ptr
  //   bne- LoopMBB
  // ExitMBB:
  //   ...
  Register Old = MI.getOperand(0).getReg();
  Register OldHi = TRI->getSubReg(Old, PPC::sub_gp8_x0);
  Register OldLo = TRI->getSubReg(Old, PPC::sub_gp8_x1);
  Register Scratch = MI.getOperand(1).getReg();
  Register ScratchHi = TRI->getSubReg(Scratch, PPC::sub_gp8_x0);
  Register ScratchLo = TRI->getSubReg(Scratch, PPC::sub_gp8_x1);
  Register RA = MI.getOperand(2).getReg();
  Register RB = MI.getOperand(3).getReg();
  Register IncrLo = MI.getOperand(4).getReg();
  Register IncrHi = MI.getOperand(5).getReg();
  unsigned RMWOpcode = MI.getOpcode();

  MachineBasicBlock *CurrentMBB = LoopMBB;
  BuildMI(CurrentMBB, DL, LL, Old).addReg(RA).addReg(RB);

  switch (RMWOpcode) {
  case PPC::ATOMIC_SWAP_I128:
    PairedCopy(TII, *CurrentMBB, CurrentMBB->end(), DL, ScratchHi, ScratchLo,
               IncrHi, IncrLo);
    break;
  case PPC::ATOMIC_LOAD_ADD_I128:
    BuildMI(CurrentMBB, DL, TII->get(PPC::ADDC8), ScratchLo)
        .addReg(IncrLo)
        .addReg(OldLo);
    BuildMI(CurrentMBB, DL, TII->get(PPC::ADDE8), ScratchHi)
        .addReg(IncrHi)
        .addReg(OldHi);
    break;
  case PPC::ATOMIC_LOAD_SUB_I128:
    BuildMI(CurrentMBB, DL, TII->get(PPC::SUBFC8), ScratchLo)
        .addReg(IncrLo)
        .addReg(OldLo);
    BuildMI(CurrentMBB, DL, TII->get(PPC::SUBFE8), ScratchHi)
```
- **EN**: Implements helper routine(s) `getOperand`, `getReg`, `getSubReg` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `getOperand`, `getReg`, `getSubReg`。

### Lines 179-216

```cpp
        .addReg(IncrHi)
        .addReg(OldHi);
    break;

#define TRIVIAL_ATOMICRMW(Opcode, Instr)                                       \
  case Opcode:                                                                 \
    BuildMI(CurrentMBB, DL, TII->get((Instr)), ScratchLo)                      \
        .addReg(IncrLo)                                                        \
        .addReg(OldLo);                                                        \
    BuildMI(CurrentMBB, DL, TII->get((Instr)), ScratchHi)                      \
        .addReg(IncrHi)                                                        \
        .addReg(OldHi);                                                        \
    break

    TRIVIAL_ATOMICRMW(PPC::ATOMIC_LOAD_OR_I128, PPC::OR8);
    TRIVIAL_ATOMICRMW(PPC::ATOMIC_LOAD_XOR_I128, PPC::XOR8);
    TRIVIAL_ATOMICRMW(PPC::ATOMIC_LOAD_AND_I128, PPC::AND8);
    TRIVIAL_ATOMICRMW(PPC::ATOMIC_LOAD_NAND_I128, PPC::NAND8);
#undef TRIVIAL_ATOMICRMW
  default:
    llvm_unreachable("Unhandled atomic RMW operation");
  }
  BuildMI(CurrentMBB, DL, SC).addReg(Scratch).addReg(RA).addReg(RB);
  BuildMI(CurrentMBB, DL, TII->get(PPC::BCC))
      .addImm(PPC::PRED_NE)
      .addReg(PPC::CR0)
      .addMBB(LoopMBB);
  CurrentMBB->addSuccessor(LoopMBB);
  CurrentMBB->addSuccessor(ExitMBB);
  fullyRecomputeLiveIns({ExitMBB, LoopMBB});
  NMBBI = MBB.end();
  MI.eraseFromParent();
  return true;
}

bool PPCExpandAtomicPseudo::expandAtomicCmpSwap128(
    MachineBasicBlock &MBB, MachineInstr &MI,
    MachineBasicBlock::iterator &NMBBI) {
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 217-254

```cpp
  const MCInstrDesc &LL = TII->get(PPC::LQARX);
  const MCInstrDesc &SC = TII->get(PPC::STQCX);
  DebugLoc DL = MI.getDebugLoc();
  MachineFunction *MF = MBB.getParent();
  const BasicBlock *BB = MBB.getBasicBlock();
  Register Old = MI.getOperand(0).getReg();
  Register OldHi = TRI->getSubReg(Old, PPC::sub_gp8_x0);
  Register OldLo = TRI->getSubReg(Old, PPC::sub_gp8_x1);
  Register Scratch = MI.getOperand(1).getReg();
  Register ScratchHi = TRI->getSubReg(Scratch, PPC::sub_gp8_x0);
  Register ScratchLo = TRI->getSubReg(Scratch, PPC::sub_gp8_x1);
  Register RA = MI.getOperand(2).getReg();
  Register RB = MI.getOperand(3).getReg();
  Register CmpLo = MI.getOperand(4).getReg();
  Register CmpHi = MI.getOperand(5).getReg();
  Register NewLo = MI.getOperand(6).getReg();
  Register NewHi = MI.getOperand(7).getReg();
  // Create layout of control flow.
  // loop:
  //   old = lqarx ptr
  //   <compare old, cmp>
  //   bne 0, exit
  // succ:
  //   stqcx new ptr
  //   bne 0, loop
  // exit:
  //   ....
  MachineFunction::iterator MFI = ++MBB.getIterator();
  MachineBasicBlock *LoopCmpMBB = MF->CreateMachineBasicBlock(BB);
  MachineBasicBlock *CmpSuccMBB = MF->CreateMachineBasicBlock(BB);
  MachineBasicBlock *ExitMBB = MF->CreateMachineBasicBlock(BB);
  MF->insert(MFI, LoopCmpMBB);
  MF->insert(MFI, CmpSuccMBB);
  MF->insert(MFI, ExitMBB);
  ExitMBB->splice(ExitMBB->begin(), &MBB, std::next(MI.getIterator()),
                  MBB.end());
  ExitMBB->transferSuccessorsAndUpdatePHIs(&MBB);
  MBB.addSuccessor(LoopCmpMBB);
```
- **EN**: Declares function entry points including `get`, `getDebugLoc`, `getParent` that other backend components call later. The logic interacts with LLVM's MC layer.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `get`, `getDebugLoc`, `getParent`。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 255-292

```cpp
  // Build loop.
  MachineBasicBlock *CurrentMBB = LoopCmpMBB;
  BuildMI(CurrentMBB, DL, LL, Old).addReg(RA).addReg(RB);
  BuildMI(CurrentMBB, DL, TII->get(PPC::XOR8), ScratchLo)
      .addReg(OldLo)
      .addReg(CmpLo);
  BuildMI(CurrentMBB, DL, TII->get(PPC::XOR8), ScratchHi)
      .addReg(OldHi)
      .addReg(CmpHi);
  BuildMI(CurrentMBB, DL, TII->get(PPC::OR8_rec), ScratchLo)
      .addReg(ScratchLo)
      .addReg(ScratchHi);
  BuildMI(CurrentMBB, DL, TII->get(PPC::BCC))
      .addImm(PPC::PRED_NE)
      .addReg(PPC::CR0)
      .addMBB(ExitMBB);
  CurrentMBB->addSuccessor(CmpSuccMBB);
  CurrentMBB->addSuccessor(ExitMBB);
  // Build succ.
  CurrentMBB = CmpSuccMBB;
  PairedCopy(TII, *CurrentMBB, CurrentMBB->end(), DL, ScratchHi, ScratchLo,
             NewHi, NewLo);
  BuildMI(CurrentMBB, DL, SC).addReg(Scratch).addReg(RA).addReg(RB);
  BuildMI(CurrentMBB, DL, TII->get(PPC::BCC))
      .addImm(PPC::PRED_NE)
      .addReg(PPC::CR0)
      .addMBB(LoopCmpMBB);
  CurrentMBB->addSuccessor(LoopCmpMBB);
  CurrentMBB->addSuccessor(ExitMBB);

  fullyRecomputeLiveIns({ExitMBB, CmpSuccMBB, LoopCmpMBB});
  NMBBI = MBB.end();
  MI.eraseFromParent();
  return true;
}

} // namespace
```
- **EN**: Implements helper routine(s) `BuildMI`, `addReg`, `get` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `BuildMI`, `addReg`, `get`。

### Lines 293-299

```cpp
INITIALIZE_PASS(PPCExpandAtomicPseudo, DEBUG_TYPE, "PowerPC Expand Atomic",
                false, false)

char PPCExpandAtomicPseudo::ID = 0;
FunctionPass *llvm::createPPCExpandAtomicPseudoPass() {
  return new PPCExpandAtomicPseudo();
}
```
- **EN**: Implements helper routine(s) `INITIALIZE_PASS`, `createPPCExpandAtomicPseudoPass`, `PPCExpandAtomicPseudo` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `INITIALIZE_PASS`, `createPPCExpandAtomicPseudoPass`, `PPCExpandAtomicPseudo`。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- Backend implementation logic / 后端实现逻辑
- Register modeling / 寄存器建模
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Declarative TableGen records / 声明式 TableGen 记录
- Atomic operation support / 原子操作支持
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `MCTargetDesc/PPCPredicates.h`
- `PPC.h`
- `PPCInstrInfo.h`
- `llvm/CodeGen/LivePhysRegs.h`
- `llvm/CodeGen/MachineFunctionPass.h`
- `llvm/CodeGen/MachineInstrBuilder.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM MC layer / LLVM MC 层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
