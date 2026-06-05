# PPCCTRLoops.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCCTRLoops.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides backend implementation logic for the PowerPC backend.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCCTRLoops.cpp`，主要负责 PowerPC 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- PPCCTRLoops.cpp - Generate CTR loops ------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-27

```cpp
//===----------------------------------------------------------------------===//
//
// This pass generates machine instructions for the CTR loops related pseudos:
// 1: MTCTRloop/DecreaseCTRloop
// 2: MTCTR8loop/DecreaseCTR8loop
//
// If a CTR loop can be generated:
// 1: MTCTRloop/MTCTR8loop will be converted to "mtctr"
// 2: DecreaseCTRloop/DecreaseCTR8loop will be converted to "bdnz/bdz" and
//    its user branch instruction can be deleted.
//
// If a CTR loop can not be generated due to clobber of CTR:
// 1: MTCTRloop/MTCTR8loop can be deleted.
// 2: DecreaseCTRloop/DecreaseCTR8loop will be converted to "addi -1" and
//    a "cmplwi/cmpldi".
//
// This pass runs just before register allocation, because we don't want
// register allocator to allocate register for DecreaseCTRloop if a CTR can be
// generated or if a CTR loop can not be generated, we don't have any condition
// register for the new added "cmplwi/cmpldi".
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This pass generates machine instructions for the CTR loops related pseudos:".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This pass generates machine instructions for the CTR loops related pseudos:”。

### Lines 28-34

```cpp
//===----------------------------------------------------------------------===//

#include "PPC.h"
#include "PPCInstrInfo.h"
#include "PPCSubtarget.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 35-41

```cpp
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineLoopInfo.h"
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/Register.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 42-48

```cpp
#include "llvm/InitializePasses.h"
#include "llvm/Pass.h"
#include "llvm/Support/ErrorHandling.h"
#include <cassert>

using namespace llvm;
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 49-86

```cpp
#define DEBUG_TYPE "ppc-ctrloops"

STATISTIC(NumCTRLoops, "Number of CTR loops generated");
STATISTIC(NumNormalLoops, "Number of normal compare + branch loops generated");

namespace {
class PPCCTRLoops : public MachineFunctionPass {
public:
  static char ID;

  PPCCTRLoops() : MachineFunctionPass(ID) {}

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addRequired<MachineLoopInfoWrapperPass>();
    MachineFunctionPass::getAnalysisUsage(AU);
  }

  bool runOnMachineFunction(MachineFunction &MF) override;

private:
  const PPCInstrInfo *TII = nullptr;
  MachineRegisterInfo *MRI = nullptr;

  bool processLoop(MachineLoop *ML);
  bool isCTRClobber(MachineInstr *MI, bool CheckReads) const;
  void expandNormalLoops(MachineLoop *ML, MachineInstr *Start,
                         MachineInstr *Dec);
  void expandCTRLoops(MachineLoop *ML, MachineInstr *Start, MachineInstr *Dec);
};
} // namespace

char PPCCTRLoops::ID = 0;

INITIALIZE_PASS_BEGIN(PPCCTRLoops, DEBUG_TYPE, "PowerPC CTR loops generation",
                      false, false)
INITIALIZE_PASS_DEPENDENCY(MachineLoopInfoWrapperPass)
INITIALIZE_PASS_END(PPCCTRLoops, DEBUG_TYPE, "PowerPC CTR loops generation",
                    false, false)
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 87-101

```cpp

FunctionPass *llvm::createPPCCTRLoopsPass() { return new PPCCTRLoops(); }

bool PPCCTRLoops::runOnMachineFunction(MachineFunction &MF) {
  bool Changed = false;

  auto &MLI = getAnalysis<MachineLoopInfoWrapperPass>().getLI();
  TII = static_cast<const PPCInstrInfo *>(MF.getSubtarget().getInstrInfo());
  MRI = &MF.getRegInfo();

  for (auto *ML : MLI) {
    if (ML->isOutermost())
      Changed |= processLoop(ML);
  }
```
- **EN**: Implements helper routine(s) `createPPCCTRLoopsPass`, `PPCCTRLoops`, `runOnMachineFunction` for this portion of the PowerPC backend backend implementation logic. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `createPPCCTRLoopsPass`, `PPCCTRLoops`, `runOnMachineFunction`。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 102-108

```cpp
#ifndef NDEBUG
  for (const MachineBasicBlock &BB : MF) {
    for (const MachineInstr &I : BB)
      assert((I.getOpcode() != PPC::DecreaseCTRloop &&
              I.getOpcode() != PPC::DecreaseCTR8loop) &&
             "CTR loop pseudo is not expanded!");
  }
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 109-146

```cpp
#endif

  return Changed;
}

bool PPCCTRLoops::isCTRClobber(MachineInstr *MI, bool CheckReads) const {
  if (!CheckReads) {
    // If we are only checking for defs, that is we are going to find
    // definitions before MTCTRloop, for this case:
    // CTR defination inside the callee of a call instruction will not impact
    // the defination of MTCTRloop, so we can use definesRegister() for the
    // check, no need to check the regmask.
    return MI->definesRegister(PPC::CTR, /*TRI=*/nullptr) ||
           MI->definesRegister(PPC::CTR8, /*TRI=*/nullptr);
  }

  if (MI->modifiesRegister(PPC::CTR, /*TRI=*/nullptr) ||
      MI->modifiesRegister(PPC::CTR8, /*TRI=*/nullptr))
    return true;

  if (MI->getDesc().isCall())
    return true;

  // We define the CTR in the loop preheader, so if there is any CTR reader in
  // the loop, we also can not use CTR loop form.
  if (MI->readsRegister(PPC::CTR, /*TRI=*/nullptr) ||
      MI->readsRegister(PPC::CTR8, /*TRI=*/nullptr))
    return true;

  return false;
}

bool PPCCTRLoops::processLoop(MachineLoop *ML) {
  bool Changed = false;

  // Align with HardwareLoop pass, process inner loops first.
  for (MachineLoop *I : *ML)
    Changed |= processLoop(I);
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 147-184

```cpp

  // If any inner loop is changed, outter loop must be without hardware loop
  // intrinsics.
  if (Changed)
    return true;

  auto IsLoopStart = [](MachineInstr &MI) {
    return MI.getOpcode() == PPC::MTCTRloop ||
           MI.getOpcode() == PPC::MTCTR8loop;
  };

  auto SearchForStart =
      [&IsLoopStart](MachineBasicBlock *MBB) -> MachineInstr * {
    for (auto &MI : *MBB) {
      if (IsLoopStart(MI))
        return &MI;
    }
    return nullptr;
  };

  MachineInstr *Start = nullptr;
  MachineInstr *Dec = nullptr;
  bool InvalidCTRLoop = false;

  MachineBasicBlock *Preheader = ML->getLoopPreheader();
  // If there is no preheader for this loop, there must be no MTCTRloop
  // either.
  if (!Preheader)
    return false;

  Start = SearchForStart(Preheader);
  // This is not a CTR loop candidate.
  if (!Start)
    return false;

  // If CTR is live to the preheader, we can not redefine the CTR register.
  if (Preheader->isLiveIn(PPC::CTR) || Preheader->isLiveIn(PPC::CTR8))
    InvalidCTRLoop = true;
```
- **EN**: Implements helper routine(s) `getOpcode`, `IsLoopStart`, `getLoopPreheader` for this portion of the PowerPC backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `getOpcode`, `IsLoopStart`, `getLoopPreheader`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 185-222

```cpp

  // Make sure there is also no CTR clobber in the block preheader between the
  // begin and MTCTR.
  for (MachineBasicBlock::reverse_instr_iterator I =
           std::next(Start->getReverseIterator());
       I != Preheader->instr_rend(); ++I)
    // Only check the definitions of CTR. If there is non-dead definition for
    // the CTR, we conservatively don't generate a CTR loop.
    if (isCTRClobber(&*I, /* CheckReads */ false)) {
      InvalidCTRLoop = true;
      break;
    }

  // Make sure there is also no CTR clobber/user in the block preheader between
  // MTCTR and the end.
  for (MachineBasicBlock::instr_iterator I = std::next(Start->getIterator());
       I != Preheader->instr_end(); ++I)
    if (isCTRClobber(&*I, /* CheckReads */ true)) {
      InvalidCTRLoop = true;
      break;
    }

  // Find the CTR loop components and decide whether or not to fall back to a
  // normal loop.
  for (auto *MBB : reverse(ML->getBlocks())) {
    for (auto &MI : *MBB) {
      if (MI.getOpcode() == PPC::DecreaseCTRloop ||
          MI.getOpcode() == PPC::DecreaseCTR8loop)
        Dec = &MI;
      else if (!InvalidCTRLoop)
        // If any instruction clobber CTR, then we can not generate a CTR loop.
        InvalidCTRLoop |= isCTRClobber(&MI, /* CheckReads */ true);
    }
    if (Dec && InvalidCTRLoop)
      break;
  }

  assert(Dec && "CTR loop is not complete!");
```
- **EN**: Implements helper routine(s) `next`, `getReverseIterator`, `instr_rend` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `next`, `getReverseIterator`, `instr_rend`。

### Lines 223-260

```cpp

  if (InvalidCTRLoop) {
    expandNormalLoops(ML, Start, Dec);
    ++NumNormalLoops;
  }
  else {
    expandCTRLoops(ML, Start, Dec);
    ++NumCTRLoops;
  }
  return true;
}

void PPCCTRLoops::expandNormalLoops(MachineLoop *ML, MachineInstr *Start,
                                    MachineInstr *Dec) {
  bool Is64Bit =
      Start->getParent()->getParent()->getSubtarget<PPCSubtarget>().isPPC64();

  MachineBasicBlock *Preheader = Start->getParent();
  MachineBasicBlock *Exiting = Dec->getParent();
  assert((Preheader && Exiting) &&
         "Preheader and exiting should exist for CTR loop!");

  assert(Dec->getOperand(1).getImm() == 1 &&
         "Loop decrement stride must be 1");

  unsigned ADDIOpcode = Is64Bit ? PPC::ADDI8 : PPC::ADDI;
  unsigned CMPOpcode = Is64Bit ? PPC::CMPLDI : PPC::CMPLWI;

  Register PHIDef =
      MRI->createVirtualRegister(Is64Bit ? &PPC::G8RC_and_G8RC_NOX0RegClass
                                         : &PPC::GPRC_and_GPRC_NOR0RegClass);

  Start->getParent()->getParent()->getProperties().resetNoPHIs();

  // Generate "PHI" in the header block.
  auto PHIMIB = BuildMI(*ML->getHeader(), ML->getHeader()->getFirstNonPHI(),
                        DebugLoc(), TII->get(TargetOpcode::PHI), PHIDef);
  PHIMIB.addReg(Start->getOperand(0).getReg()).addMBB(Preheader);
```
- **EN**: Implements helper routine(s) `expandNormalLoops`, `expandCTRLoops`, `getParent` for this portion of the PowerPC backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `expandNormalLoops`, `expandCTRLoops`, `getParent`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 261-298

```cpp

  Register ADDIDef =
      MRI->createVirtualRegister(Is64Bit ? &PPC::G8RC_and_G8RC_NOX0RegClass
                                         : &PPC::GPRC_and_GPRC_NOR0RegClass);
  // Generate "addi -1" in the exiting block.
  BuildMI(*Exiting, Dec, Dec->getDebugLoc(), TII->get(ADDIOpcode), ADDIDef)
      .addReg(PHIDef)
      .addImm(-1);

  // Add other inputs for the PHI node.
  if (ML->isLoopLatch(Exiting)) {
    // There must be only two predecessors for the loop header, one is the
    // Preheader and the other one is loop latch Exiting. In hardware loop
    // insertion pass, the block containing DecreaseCTRloop must dominate all
    // loop latches. So there must be only one latch.
    assert(ML->getHeader()->pred_size() == 2 &&
           "Loop header predecessor is not right!");
    PHIMIB.addReg(ADDIDef).addMBB(Exiting);
  } else {
    // If the block containing DecreaseCTRloop is not a loop latch, we can use
    // ADDIDef as the value for all other blocks for the PHI. In hardware loop
    // insertion pass, the block containing DecreaseCTRloop must dominate all
    // loop latches.
    for (MachineBasicBlock *P : ML->getHeader()->predecessors()) {
      if (ML->contains(P)) {
        assert(ML->isLoopLatch(P) &&
               "Loop's header in-loop predecessor is not loop latch!");
        PHIMIB.addReg(ADDIDef).addMBB(P);
      } else
        assert(P == Preheader &&
               "CTR loop should not be generated for irreducible loop!");
    }
  }

  // Generate the compare in the exiting block.
  Register CMPDef = MRI->createVirtualRegister(&PPC::CRRCRegClass);
  auto CMPMIB =
      BuildMI(*Exiting, Dec, Dec->getDebugLoc(), TII->get(CMPOpcode), CMPDef)
```
- **EN**: Implements helper routine(s) `createVirtualRegister`, `BuildMI`, `getDebugLoc` for this portion of the PowerPC backend backend implementation logic. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `createVirtualRegister`, `BuildMI`, `getDebugLoc`。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 299-336

```cpp
          .addReg(ADDIDef)
          .addImm(0);

  BuildMI(*Exiting, Dec, Dec->getDebugLoc(), TII->get(TargetOpcode::COPY),
          Dec->getOperand(0).getReg())
      .addReg(CMPMIB->getOperand(0).getReg(), {}, PPC::sub_gt);

  // Remove the pseudo instructions.
  Start->eraseFromParent();
  Dec->eraseFromParent();
}

void PPCCTRLoops::expandCTRLoops(MachineLoop *ML, MachineInstr *Start,
                                 MachineInstr *Dec) {
  bool Is64Bit =
      Start->getParent()->getParent()->getSubtarget<PPCSubtarget>().isPPC64();

  MachineBasicBlock *Preheader = Start->getParent();
  MachineBasicBlock *Exiting = Dec->getParent();

  (void)Preheader;
  assert((Preheader && Exiting) &&
         "Preheader and exiting should exist for CTR loop!");

  assert(Dec->getOperand(1).getImm() == 1 && "Loop decrement must be 1!");

  unsigned BDNZOpcode = Is64Bit ? PPC::BDNZ8 : PPC::BDNZ;
  unsigned BDZOpcode = Is64Bit ? PPC::BDZ8 : PPC::BDZ;
  auto BrInstr = MRI->use_instr_begin(Dec->getOperand(0).getReg());
  assert(MRI->hasOneUse(Dec->getOperand(0).getReg()) &&
         "There should be only one user for loop decrement pseudo!");

  unsigned Opcode = 0;
  switch (BrInstr->getOpcode()) {
  case PPC::BC:
    Opcode = BDNZOpcode;
    (void) ML;
    assert(ML->contains(BrInstr->getOperand(1).getMBB()) &&
```
- **EN**: Implements helper routine(s) `addReg`, `addImm`, `BuildMI` for this portion of the PowerPC backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `addReg`, `addImm`, `BuildMI`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 337-355

```cpp
           "Invalid ctr loop!");
    break;
  case PPC::BCn:
    Opcode = BDZOpcode;
    assert(!ML->contains(BrInstr->getOperand(1).getMBB()) &&
           "Invalid ctr loop!");
    break;
  default:
    llvm_unreachable("Unhandled branch user for DecreaseCTRloop.");
  }

  // Generate "bdnz/bdz" in the exiting block just before the terminator.
  BuildMI(*Exiting, &*BrInstr, BrInstr->getDebugLoc(), TII->get(Opcode))
      .addMBB(BrInstr->getOperand(1).getMBB());

  // Remove the pseudo instructions.
  BrInstr->eraseFromParent();
  Dec->eraseFromParent();
}
```
- **EN**: Declares function entry points including `contains`, `getOperand`, `getMBB` that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `contains`, `getOperand`, `getMBB`。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- Backend implementation logic / 后端实现逻辑
- Register modeling / 寄存器建模
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Declarative TableGen records / 声明式 TableGen 记录
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `PPC.h`
- `PPCInstrInfo.h`
- `PPCSubtarget.h`
- `llvm/ADT/Statistic.h`
- `llvm/CodeGen/MachineBasicBlock.h`
- `llvm/CodeGen/MachineFunction.h`
- `llvm/CodeGen/MachineFunctionPass.h`
- `llvm/CodeGen/MachineInstr.h`
- `llvm/CodeGen/MachineLoopInfo.h`
- `llvm/CodeGen/MachineOperand.h`
- `llvm/CodeGen/MachineRegisterInfo.h`
- `llvm/CodeGen/Register.h`
- `llvm/InitializePasses.h`
- `llvm/Pass.h`
- `llvm/Support/ErrorHandling.h`
- `cassert`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM Support utilities / LLVM Support 工具层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
