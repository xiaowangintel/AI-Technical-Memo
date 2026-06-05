# MVEVPTBlockPass.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/MVEVPTBlockPass.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `MVEVPTBlockPass` for the ARM backend, focusing on target-specific code-generation behavior.
- 用途 (CN): 实现 ARM 后端中的 `MVEVPTBlockPass`，重点处理目标相关代码生成行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- MVEVPTBlockPass.cpp - Insert MVE VPT blocks -----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 9-23
```cpp
#include "ARM.h"
#include "ARMMachineFunctionInfo.h"
#include "ARMSubtarget.h"
#include "Thumb2InstrInfo.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineInstrBundle.h"
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/IR/DebugLoc.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 24-26
```cpp
#include "llvm/Support/Debug.h"
#include <cassert>
#include <new>
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
#define DEBUG_TYPE "arm-mve-vpt"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 32-37
```cpp
namespace {
class MVEVPTBlock : public MachineFunctionPass {
public:
  static char ID;
  const Thumb2InstrInfo *TII;
  const TargetRegisterInfo *TRI;
```
- EN: Declares `MVEVPTBlock`, packaging target-specific state and APIs around `MVEVPTBlockPass`.
- CN: 这里声明 `MVEVPTBlock`，把与 `MVEVPTBlockPass` 相关的目标特定状态和 API 组织在一起。

### Lines 39-39
```cpp
  MVEVPTBlock() : MachineFunctionPass(ID) {}
```
- EN: Implements `MVEVPTBlock`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MVEVPTBlock`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

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
    return "MVE VPT block insertion pass";
  }
```
- EN: Implements `getPassName`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getPassName`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 51-53
```cpp
private:
  bool InsertVPTBlocks(MachineBasicBlock &MBB);
};
```
- EN: Declares `InsertVPTBlocks`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `InsertVPTBlocks`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 55-55
```cpp
char MVEVPTBlock::ID = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 57-57
```cpp
} // end anonymous namespace
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 59-59
```cpp
INITIALIZE_PASS(MVEVPTBlock, DEBUG_TYPE, "ARM MVE VPT block pass", false, false)
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 61-74
```cpp
static MachineInstr *findVCMPToFoldIntoVPST(MachineBasicBlock::iterator MI,
                                            const TargetRegisterInfo *TRI,
                                            unsigned &NewOpcode) {
  // Search backwards to the instruction that defines VPR. This may or not
  // be a VCMP, we check that after this loop. If we find another instruction
  // that reads cpsr, we return nullptr.
  MachineBasicBlock::iterator CmpMI = MI;
  while (CmpMI != MI->getParent()->begin()) {
    --CmpMI;
    if (CmpMI->modifiesRegister(ARM::VPR, TRI))
      break;
    if (CmpMI->readsRegister(ARM::VPR, TRI))
      break;
  }
```
- EN: Implements `findVCMPToFoldIntoVPST`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `findVCMPToFoldIntoVPST`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 76-80
```cpp
  if (CmpMI == MI)
    return nullptr;
  NewOpcode = VCMPOpcodeToVPT(CmpMI->getOpcode());
  if (NewOpcode == 0)
    return nullptr;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 82-90
```cpp
  // Search forward from CmpMI to MI, checking if either register was def'd
  if (registerDefinedBetween(CmpMI->getOperand(1).getReg(), std::next(CmpMI),
                             MI, TRI))
    return nullptr;
  if (registerDefinedBetween(CmpMI->getOperand(2).getReg(), std::next(CmpMI),
                             MI, TRI))
    return nullptr;
  return &*CmpMI;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 92-102
```cpp
// Advances Iter past a block of predicated instructions.
// Returns true if it successfully skipped the whole block of predicated
// instructions. Returns false when it stopped early (due to MaxSteps), or if
// Iter didn't point to a predicated instruction.
static bool StepOverPredicatedInstrs(MachineBasicBlock::instr_iterator &Iter,
                                     MachineBasicBlock::instr_iterator EndIter,
                                     unsigned MaxSteps,
                                     unsigned &NumInstrsSteppedOver) {
  ARMVCC::VPTCodes NextPred = ARMVCC::None;
  Register PredReg;
  NumInstrsSteppedOver = 0;
```
- EN: Implements `StepOverPredicatedInstrs`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `StepOverPredicatedInstrs`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 104-109
```cpp
  while (Iter != EndIter) {
    if (Iter->isDebugInstr()) {
      // Skip debug instructions
      ++Iter;
      continue;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 111-119
```cpp
    NextPred = getVPTInstrPredicate(*Iter, PredReg);
    assert(NextPred != ARMVCC::Else &&
           "VPT block pass does not expect Else preds");
    if (NextPred == ARMVCC::None || MaxSteps == 0)
      break;
    --MaxSteps;
    ++Iter;
    ++NumInstrsSteppedOver;
  };
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 121-123
```cpp
  return NumInstrsSteppedOver != 0 &&
         (NextPred == ARMVCC::None || Iter == EndIter);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 125-134
```cpp
// Returns true if at least one instruction in the range [Iter, End) defines
// or kills VPR.
static bool IsVPRDefinedOrKilledByBlock(MachineBasicBlock::iterator Iter,
                                        MachineBasicBlock::iterator End) {
  for (; Iter != End; ++Iter)
    if (Iter->definesRegister(ARM::VPR, /*TRI=*/nullptr) ||
        Iter->killsRegister(ARM::VPR, /*TRI=*/nullptr))
      return true;
  return false;
}
```
- EN: Implements `IsVPRDefinedOrKilledByBlock`, a query/helper routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `IsVPRDefinedOrKilledByBlock`，它是一个围绕机器基本块展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 136-150
```cpp
// Creates a T, TT, TTT or TTTT BlockMask depending on BlockSize.
static ARM::PredBlockMask GetInitialBlockMask(unsigned BlockSize) {
  switch (BlockSize) {
  case 1:
    return ARM::PredBlockMask::T;
  case 2:
    return ARM::PredBlockMask::TT;
  case 3:
    return ARM::PredBlockMask::TTT;
  case 4:
    return ARM::PredBlockMask::TTTT;
  default:
    llvm_unreachable("Invalid BlockSize!");
  }
}
```
- EN: Implements `GetInitialBlockMask`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `GetInitialBlockMask`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 152-165
```cpp
// Given an iterator (Iter) that points at an instruction with a "Then"
// predicate, tries to create the largest block of continuous predicated
// instructions possible, and returns the VPT Block Mask of that block.
//
// This will try to perform some minor optimization in order to maximize the
// size of the block.
static ARM::PredBlockMask
CreateVPTBlock(MachineBasicBlock::instr_iterator &Iter,
               MachineBasicBlock::instr_iterator EndIter,
               SmallVectorImpl<MachineInstr *> &DeadInstructions) {
  MachineBasicBlock::instr_iterator BlockBeg = Iter;
  (void)BlockBeg;
  assert(getVPTInstrPredicate(*Iter) == ARMVCC::Then &&
         "Expected a Predicated Instruction");
```
- EN: Implements `CreateVPTBlock`, a query/helper routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `CreateVPTBlock`，它是一个围绕机器基本块展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 167-167
```cpp
  LLVM_DEBUG(dbgs() << "VPT block created for: "; Iter->dump());
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 169-170
```cpp
  unsigned BlockSize;
  StepOverPredicatedInstrs(Iter, EndIter, 4, BlockSize);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 172-179
```cpp
  LLVM_DEBUG(for (MachineBasicBlock::instr_iterator AddedInstIter =
                      std::next(BlockBeg);
                  AddedInstIter != Iter; ++AddedInstIter) {
    if (AddedInstIter->isDebugInstr())
      continue;
    dbgs() << "  adding: ";
    AddedInstIter->dump();
  });
```
- EN: Implements `LLVM_DEBUG`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `LLVM_DEBUG`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 181-182
```cpp
  // Generate the initial BlockMask
  ARM::PredBlockMask BlockMask = GetInitialBlockMask(BlockSize);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 184-188
```cpp
  // Remove VPNOTs while there's still room in the block, so we can make the
  // largest block possible.
  ARMVCC::VPTCodes CurrentPredicate = ARMVCC::Else;
  while (BlockSize < 4 && Iter != EndIter &&
         Iter->getOpcode() == ARM::MVE_VPNOT) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 190-196
```cpp
    // Try to skip all of the predicated instructions after the VPNOT, stopping
    // after (4 - BlockSize). If we can't skip them all, stop.
    unsigned ElseInstCnt = 0;
    MachineBasicBlock::instr_iterator VPNOTBlockEndIter = std::next(Iter);
    if (!StepOverPredicatedInstrs(VPNOTBlockEndIter, EndIter, (4 - BlockSize),
                                  ElseInstCnt))
      break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 198-202
```cpp
    // Check if this VPNOT can be removed or not: It can only be removed if at
    // least one of the predicated instruction that follows it kills or sets
    // VPR.
    if (!IsVPRDefinedOrKilledByBlock(Iter, VPNOTBlockEndIter))
      break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 204-204
```cpp
    LLVM_DEBUG(dbgs() << "  removing VPNOT: "; Iter->dump());
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 206-208
```cpp
    // Record the new size of the block
    BlockSize += ElseInstCnt;
    assert(BlockSize <= 4 && "Block is too large!");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 210-212
```cpp
    // Record the VPNot to remove it later.
    DeadInstructions.push_back(&*Iter);
    ++Iter;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 214-219
```cpp
    // Replace the predicates of the instructions we're adding.
    // Note that we are using "Iter" to iterate over the block so we can update
    // it at the same time.
    for (; Iter != VPNOTBlockEndIter; ++Iter) {
      if (Iter->isDebugInstr())
        continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 221-223
```cpp
      // Find the register in which the predicate is
      int OpIdx = findFirstVPTPredOperandIdx(*Iter);
      assert(OpIdx != -1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 225-227
```cpp
      // Change the predicate and update the mask
      Iter->getOperand(OpIdx).setImm(CurrentPredicate);
      BlockMask = expandPredBlockMask(BlockMask, CurrentPredicate);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 229-230
```cpp
      LLVM_DEBUG(dbgs() << "  adding : "; Iter->dump());
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 232-236
```cpp
    CurrentPredicate =
        (CurrentPredicate == ARMVCC::Then ? ARMVCC::Else : ARMVCC::Then);
  }
  return BlockMask;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 238-241
```cpp
bool MVEVPTBlock::InsertVPTBlocks(MachineBasicBlock &Block) {
  bool Modified = false;
  MachineBasicBlock::instr_iterator MBIter = Block.instr_begin();
  MachineBasicBlock::instr_iterator EndIter = Block.instr_end();
```
- EN: Implements `MVEVPTBlock::InsertVPTBlocks`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MVEVPTBlock::InsertVPTBlocks`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 243-243
```cpp
  SmallVector<MachineInstr *, 4> DeadInstructions;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 245-248
```cpp
  while (MBIter != EndIter) {
    MachineInstr *MI = &*MBIter;
    Register PredReg;
    DebugLoc DL = MI->getDebugLoc();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 250-250
```cpp
    ARMVCC::VPTCodes Pred = getVPTInstrPredicate(*MI, PredReg);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 252-258
```cpp
    // The idea of the predicate is that None, Then and Else are for use when
    // handling assembly language: they correspond to the three possible
    // suffixes "", "t" and "e" on the mnemonic. So when instructions are read
    // from assembly source or disassembled from object code, you expect to
    // see a mixture whenever there's a long VPT block. But in code
    // generation, we hope we'll never generate an Else as input to this pass.
    assert(Pred != ARMVCC::Else && "VPT block pass does not expect Else preds");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 260-263
```cpp
    if (Pred == ARMVCC::None) {
      ++MBIter;
      continue;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 265-266
```cpp
    ARM::PredBlockMask BlockMask =
        CreateVPTBlock(MBIter, EndIter, DeadInstructions);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 268-279
```cpp
    // Search back for a VCMP that can be folded to create a VPT, or else
    // create a VPST directly
    MachineInstrBuilder MIBuilder;
    unsigned NewOpcode;
    LLVM_DEBUG(dbgs() << "  final block mask: " << (unsigned)BlockMask << "\n");
    if (MachineInstr *VCMP = findVCMPToFoldIntoVPST(MI, TRI, NewOpcode)) {
      LLVM_DEBUG(dbgs() << "  folding VCMP into VPST: "; VCMP->dump());
      MIBuilder = BuildMI(Block, MI, DL, TII->get(NewOpcode));
      MIBuilder.addImm((uint64_t)BlockMask);
      MIBuilder.add(VCMP->getOperand(1));
      MIBuilder.add(VCMP->getOperand(2));
      MIBuilder.add(VCMP->getOperand(3));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 281-287
```cpp
      // We need to remove any kill flags between the original VCMP and the new
      // insertion point.
      for (MachineInstr &MII :
           make_range(VCMP->getIterator(), MI->getIterator())) {
        MII.clearRegisterKills(VCMP->getOperand(1).getReg(), TRI);
        MII.clearRegisterKills(VCMP->getOperand(2).getReg(), TRI);
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 289-293
```cpp
      VCMP->eraseFromParent();
    } else {
      MIBuilder = BuildMI(Block, MI, DL, TII->get(ARM::MVE_VPST));
      MIBuilder.addImm((uint64_t)BlockMask);
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 295-299
```cpp
    // Erase all dead instructions (VPNOT's). Do that now so that they do not
    // mess with the bundle creation.
    for (MachineInstr *DeadMI : DeadInstructions)
      DeadMI->eraseFromParent();
    DeadInstructions.clear();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 301-302
```cpp
    finalizeBundle(
        Block, MachineBasicBlock::instr_iterator(MIBuilder.getInstr()), MBIter);
```
- EN: Declares `MachineBasicBlock::instr_iterator`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `MachineBasicBlock::instr_iterator`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 304-305
```cpp
    Modified = true;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 307-308
```cpp
  return Modified;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 310-311
```cpp
bool MVEVPTBlock::runOnMachineFunction(MachineFunction &Fn) {
  const ARMSubtarget &STI = Fn.getSubtarget<ARMSubtarget>();
```
- EN: Implements `MVEVPTBlock::runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MVEVPTBlock::runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 313-314
```cpp
  if (!STI.isThumb2() || !STI.hasMVEIntegerOps())
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 316-317
```cpp
  TII = static_cast<const Thumb2InstrInfo *>(STI.getInstrInfo());
  TRI = STI.getRegisterInfo();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 319-320
```cpp
  LLVM_DEBUG(dbgs() << "********** ARM MVE VPT BLOCKS **********\n"
                    << "********** Function: " << Fn.getName() << '\n');
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 322-324
```cpp
  bool Modified = false;
  for (MachineBasicBlock &MBB : Fn)
    Modified |= InsertVPTBlocks(MBB);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 326-328
```cpp
  LLVM_DEBUG(dbgs() << "**************************************\n");
  return Modified;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 330-332
```cpp
/// createMVEVPTBlock - Returns an instance of the MVE VPT block
/// insertion pass.
FunctionPass *llvm::createMVEVPTBlockPass() { return new MVEVPTBlock(); }
```
- EN: Implements `llvm::createMVEVPTBlockPass`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::createMVEVPTBlockPass`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

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

- EN: Backend-local headers: `ARM.h`, `ARMMachineFunctionInfo.h`, `ARMSubtarget.h`, `Thumb2InstrInfo.h`.
  - CN: 后端本地头文件：`ARM.h`, `ARMMachineFunctionInfo.h`, `ARMSubtarget.h`, `Thumb2InstrInfo.h`。
- EN: LLVM infrastructure headers: `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h`, `llvm/ADT/StringRef.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/MachineInstrBuilder.h` ... (+4 more).
  - CN: LLVM 基础设施头文件：`llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h`, `llvm/ADT/StringRef.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/MachineInstrBuilder.h` ... (+4 more)。
- EN: Standard/system headers: `cassert`, `new`.
  - CN: 标准库/系统头文件：`cassert`, `new`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
