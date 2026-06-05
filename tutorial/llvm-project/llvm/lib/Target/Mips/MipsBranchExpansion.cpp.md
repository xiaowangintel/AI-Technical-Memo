# MipsBranchExpansion.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsBranchExpansion.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `MipsBranchExpansion` for the Mips backend, focusing on target-specific code-generation behavior.
- 用途 (CN): 实现 Mips 后端中的 `MipsBranchExpansion`，重点处理目标相关代码生成行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```cpp
//===----------------------- MipsBranchExpansion.cpp ----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
/// \file
///
/// This pass do two things:
/// - it expands a branch or jump instruction into a long branch if its offset
///   is too large to fit into its immediate field,
/// - it inserts nops to prevent forbidden slot hazards.
///
/// The reason why this pass combines these two tasks is that one of these two
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 16-30
```cpp
/// tasks can break the result of the previous one.
///
/// Example of that is a situation where at first, no branch should be expanded,
/// but after adding at least one nop somewhere in the code to prevent a
/// forbidden slot hazard, offset of some branches may go out of range. In that
/// case it is necessary to check again if there is some branch that needs
/// expansion. On the other hand, expanding some branch may cause a control
/// transfer instruction to appear in the forbidden slot, which is a hazard that
/// should be fixed. This pass alternates between this two tasks untill no
/// changes are made. Only then we can be sure that all branches are expanded
/// properly, and no hazard situations exist.
///
/// Regarding branch expanding:
///
/// When branch instruction like beqzc or bnezc has offset that is too large
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 31-45
```cpp
/// to fit into its immediate field, it has to be expanded to another
/// instruction or series of instructions.
///
/// FIXME: Fix pc-region jump instructions which cross 256MB segment boundaries.
/// TODO: Handle out of range bc, b (pseudo) instructions.
///
/// Regarding compact branch hazard prevention:
///
/// Hazards handled: forbidden slots for MIPSR6, FPU slots for MIPS3 and below,
/// load delay slots for MIPS1.
///
/// A forbidden slot hazard occurs when a compact branch instruction is executed
/// and the adjacent instruction in memory is a control transfer instruction
/// such as a branch or jump, ERET, ERETNC, DERET, WAIT and PAUSE.
///
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 46-60
```cpp
/// For example:
///
/// 0x8004      bnec    a1,v0,<P+0x18>
/// 0x8008      beqc    a1,a2,<P+0x54>
///
/// In such cases, the processor is required to signal a Reserved Instruction
/// exception.
///
/// Here, if the instruction at 0x8004 is executed, the processor will raise an
/// exception as there is a control transfer instruction at 0x8008.
///
/// There are two sources of forbidden slot hazards:
///
/// A) A previous pass has created a compact branch directly.
/// B) Transforming a delay slot branch into compact branch. This case can be
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 61-73
```cpp
///    difficult to process as lookahead for hazards is insufficient, as
///    backwards delay slot fillling can also produce hazards in previously
///    processed instuctions.
///
/// In future this pass can be extended (or new pass can be created) to handle
/// other pipeline hazards, such as various MIPS1 hazards, processor errata that
/// require instruction reorganization, etc.
///
/// This pass has to run after the delay slot filler as that pass can introduce
/// pipeline hazards such as compact branch hazard, hence the existing hazard
/// recognizer is not suitable.
///
//===----------------------------------------------------------------------===//
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 75-89
```cpp
#include "MCTargetDesc/MipsABIInfo.h"
#include "MCTargetDesc/MipsBaseInfo.h"
#include "MCTargetDesc/MipsMCTargetDesc.h"
#include "Mips.h"
#include "MipsInstrInfo.h"
#include "MipsMachineFunction.h"
#include "MipsSubtarget.h"
#include "MipsTargetMachine.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstr.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 90-102
```cpp
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineModuleInfo.h"
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/CodeGen/TargetSubtargetInfo.h"
#include "llvm/IR/DebugLoc.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Target/TargetMachine.h"
#include <algorithm>
#include <cassert>
#include <cstdint>
#include <iterator>
#include <utility>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 104-104
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 106-106
```cpp
#define DEBUG_TYPE "mips-branch-expansion"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 108-109
```cpp
STATISTIC(NumInsertedNops, "Number of nops inserted");
STATISTIC(LongBranches, "Number of long branches.");
```
- EN: Declares `STATISTIC`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `STATISTIC`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 111-113
```cpp
static cl::opt<bool>
    SkipLongBranch("skip-mips-long-branch", cl::init(false),
                   cl::desc("MIPS: Skip branch expansion pass."), cl::Hidden);
```
- EN: Declares `cl::init`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::init`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 115-118
```cpp
static cl::opt<bool>
    ForceLongBranch("force-mips-long-branch", cl::init(false),
                    cl::desc("MIPS: Expand all branches to long format."),
                    cl::Hidden);
```
- EN: Declares `cl::init`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::init`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 120-120
```cpp
namespace {
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 122-123
```cpp
using Iter = MachineBasicBlock::iterator;
using ReverseIter = MachineBasicBlock::reverse_iterator;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 125-131
```cpp
struct MBBInfo {
  uint64_t Size = 0;
  bool HasLongBranch = false;
  MachineInstr *Br = nullptr;
  uint64_t Offset = 0;
  MBBInfo() = default;
};
```
- EN: Declares `MBBInfo`, packaging target-specific state and APIs around `MipsBranchExpansion`.
- CN: 这里声明 `MBBInfo`，把与 `MipsBranchExpansion` 相关的目标特定状态和 API 组织在一起。

### Lines 133-135
```cpp
class MipsBranchExpansion : public MachineFunctionPass {
public:
  static char ID;
```
- EN: Declares `MipsBranchExpansion`, packaging target-specific state and APIs around `MipsBranchExpansion`.
- CN: 这里声明 `MipsBranchExpansion`，把与 `MipsBranchExpansion` 相关的目标特定状态和 API 组织在一起。

### Lines 137-138
```cpp
  MipsBranchExpansion()
      : MachineFunctionPass(ID), ABI(MipsABIInfo::Unknown()) {}
```
- EN: Implements `MipsABIInfo::Unknown`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsABIInfo::Unknown`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 140-142
```cpp
  StringRef getPassName() const override {
    return "Mips Branch Expansion Pass";
  }
```
- EN: Implements `getPassName`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getPassName`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 144-144
```cpp
  bool runOnMachineFunction(MachineFunction &F) override;
```
- EN: Declares `runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 146-148
```cpp
  MachineFunctionProperties getRequiredProperties() const override {
    return MachineFunctionProperties().setNoVRegs();
  }
```
- EN: Implements `getRequiredProperties`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getRequiredProperties`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 150-164
```cpp
private:
  void splitMBB(MachineBasicBlock *MBB);
  void initMBBInfo();
  int64_t computeOffset(const MachineInstr *Br);
  uint64_t computeOffsetFromTheBeginning(int MBB);
  void replaceBranch(MachineBasicBlock &MBB, Iter Br, const DebugLoc &DL,
                     MachineBasicBlock *MBBOpnd);
  bool buildProperJumpMI(MachineBasicBlock *MBB,
                         MachineBasicBlock::iterator Pos, DebugLoc DL);
  void expandToLongBranch(MBBInfo &Info);
  template <typename Pred, typename Safe>
  bool handleSlot(Pred Predicate, Safe SafeInSlot);
  bool handleForbiddenSlot();
  bool handleFPUDelaySlot();
  bool handleLoadDelaySlot();
```
- EN: Declares `splitMBB`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `splitMBB`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 165-168
```cpp
  bool handlePossibleLongBranch();
  bool handleMFLO();
  template <typename Pred, typename Safe>
  bool handleMFLOSlot(Pred Predicate, Safe SafeInSlot);
```
- EN: Declares `handlePossibleLongBranch`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `handlePossibleLongBranch`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 170-171
```cpp
  const MipsSubtarget *STI;
  const MipsInstrInfo *TII;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 173-178
```cpp
  MachineFunction *MFp;
  SmallVector<MBBInfo, 16> MBBInfos;
  bool IsPIC;
  MipsABIInfo ABI;
  bool ForceLongBranchFirstPass = false;
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 180-180
```cpp
} // end of anonymous namespace
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 182-182
```cpp
char MipsBranchExpansion::ID = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 184-187
```cpp
INITIALIZE_PASS(MipsBranchExpansion, DEBUG_TYPE,
                "Expand out of range branch instructions and fix forbidden"
                " slot hazards",
                false, false)
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 189-192
```cpp
/// Returns a pass that clears pipeline hazards.
FunctionPass *llvm::createMipsBranchExpansion() {
  return new MipsBranchExpansion();
}
```
- EN: Implements `llvm::createMipsBranchExpansion`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::createMipsBranchExpansion`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 194-199
```cpp
// Find the next real instruction from the current position in current basic
// block.
static Iter getNextMachineInstrInBB(Iter Position) {
  Iter I = Position, E = Position->getParent()->end();
  I = std::find_if_not(I, E,
                       [](const Iter &Insn) { return Insn->isTransient(); });
```
- EN: Implements `getNextMachineInstrInBB`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getNextMachineInstrInBB`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 201-202
```cpp
  return I;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 204-218
```cpp
// Find the next real instruction from the current position, looking through
// basic block boundaries.
static std::pair<Iter, bool> getNextMachineInstr(Iter Position,
                                                 MachineBasicBlock *Parent) {
  if (Position == Parent->end()) {
    do {
      MachineBasicBlock *Succ = Parent->getNextNode();
      if (Succ != nullptr && Parent->isSuccessor(Succ)) {
        Position = Succ->begin();
        Parent = Succ;
      } else {
        return std::make_pair(Position, true);
      }
    } while (Parent->empty());
  }
```
- EN: Implements `getNextMachineInstr`, a query/helper routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getNextMachineInstr`，它是一个围绕机器基本块展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 220-225
```cpp
  Iter Instr = getNextMachineInstrInBB(Position);
  if (Instr == Parent->end()) {
    return getNextMachineInstr(Instr, Parent);
  }
  return std::make_pair(Instr, false);
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 227-231
```cpp
/// Iterate over list of Br's operands and search for a MachineBasicBlock
/// operand.
static MachineBasicBlock *getTargetMBB(const MachineInstr &Br) {
  for (unsigned I = 0, E = Br.getDesc().getNumOperands(); I < E; ++I) {
    const MachineOperand &MO = Br.getOperand(I);
```
- EN: Implements `getTargetMBB`, a query/helper routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getTargetMBB`，它是一个围绕机器基本块展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 233-235
```cpp
    if (MO.isMBB())
      return MO.getMBB();
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 237-238
```cpp
  llvm_unreachable("This instruction does not have an MBB operand.");
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 240-245
```cpp
// Traverse the list of instructions backwards until a non-debug instruction is
// found or it reaches E.
static ReverseIter getNonDebugInstr(ReverseIter B, const ReverseIter &E) {
  for (; B != E; ++B)
    if (!B->isDebugInstr())
      return B;
```
- EN: Implements `getNonDebugInstr`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getNonDebugInstr`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 247-248
```cpp
  return E;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 250-253
```cpp
// Split MBB if it has two direct jumps/branches.
void MipsBranchExpansion::splitMBB(MachineBasicBlock *MBB) {
  ReverseIter End = MBB->rend();
  ReverseIter LastBr = getNonDebugInstr(MBB->rbegin(), End);
```
- EN: Implements `MipsBranchExpansion::splitMBB`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsBranchExpansion::splitMBB`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 255-258
```cpp
  // Return if MBB has no branch instructions.
  if ((LastBr == End) ||
      (!LastBr->isConditionalBranch() && !LastBr->isUnconditionalBranch()))
    return;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 260-260
```cpp
  ReverseIter FirstBr = getNonDebugInstr(std::next(LastBr), End);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 262-266
```cpp
  // MBB has only one branch instruction if FirstBr is not a branch
  // instruction.
  if ((FirstBr == End) ||
      (!FirstBr->isConditionalBranch() && !FirstBr->isUnconditionalBranch()))
    return;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 268-268
```cpp
  assert(!FirstBr->isIndirectBranch() && "Unexpected indirect branch found.");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 270-272
```cpp
  // Create a new MBB. Move instructions in MBB to the newly created MBB.
  MachineBasicBlock *NewMBB =
      MFp->CreateMachineBasicBlock(MBB->getBasicBlock());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 274-281
```cpp
  // Insert NewMBB and fix control flow.
  MachineBasicBlock *Tgt = getTargetMBB(*FirstBr);
  NewMBB->transferSuccessors(MBB);
  if (Tgt != getTargetMBB(*LastBr))
    NewMBB->removeSuccessor(Tgt, true);
  MBB->addSuccessor(NewMBB);
  MBB->addSuccessor(Tgt);
  MFp->insert(std::next(MachineFunction::iterator(MBB)), NewMBB);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 283-284
```cpp
  NewMBB->splice(NewMBB->end(), MBB, LastBr.getReverse(), MBB->end());
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 286-291
```cpp
// Fill MBBInfos.
void MipsBranchExpansion::initMBBInfo() {
  // Split the MBBs if they have two branches. Each basic block should have at
  // most one branch after this loop is executed.
  for (auto &MBB : *MFp)
    splitMBB(&MBB);
```
- EN: Implements `MipsBranchExpansion::initMBBInfo`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsBranchExpansion::initMBBInfo`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 293-295
```cpp
  MFp->RenumberBlocks();
  MBBInfos.clear();
  MBBInfos.resize(MFp->size());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 297-298
```cpp
  for (unsigned I = 0, E = MBBInfos.size(); I < E; ++I) {
    MachineBasicBlock *MBB = MFp->getBlockNumbered(I);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 300-304
```cpp
    // Compute size of MBB.
    for (MachineInstr &MI : *MBB)
      MBBInfos[I].Size += TII->getInstSizeInBytes(MI);
  }
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 306-310
```cpp
// Compute offset of branch in number of bytes.
int64_t MipsBranchExpansion::computeOffset(const MachineInstr *Br) {
  int64_t Offset = 0;
  int ThisMBB = Br->getParent()->getNumber();
  int TargetMBB = getTargetMBB(*Br)->getNumber();
```
- EN: Implements `MipsBranchExpansion::computeOffset`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsBranchExpansion::computeOffset`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 312-315
```cpp
  // Compute offset of a forward branch.
  if (ThisMBB < TargetMBB) {
    for (int N = ThisMBB + 1; N < TargetMBB; ++N)
      Offset += MBBInfos[N].Size;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 317-318
```cpp
    return Offset + 4;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 320-322
```cpp
  // Compute offset of a backward branch.
  for (int N = ThisMBB; N >= TargetMBB; --N)
    Offset += MBBInfos[N].Size;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 324-325
```cpp
  return -Offset + 4;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 327-333
```cpp
// Returns the distance in bytes up until MBB
uint64_t MipsBranchExpansion::computeOffsetFromTheBeginning(int MBB) {
  uint64_t Offset = 0;
  for (int N = 0; N < MBB; ++N)
    Offset += MBBInfos[N].Size;
  return Offset;
}
```
- EN: Implements `MipsBranchExpansion::computeOffsetFromTheBeginning`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsBranchExpansion::computeOffsetFromTheBeginning`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 335-341
```cpp
// Replace Br with a branch which has the opposite condition code and a
// MachineBasicBlock operand MBBOpnd.
void MipsBranchExpansion::replaceBranch(MachineBasicBlock &MBB, Iter Br,
                                        const DebugLoc &DL,
                                        MachineBasicBlock *MBBOpnd) {
  unsigned NewOpc = TII->getOppositeBranchOpc(Br->getOpcode());
  const MCInstrDesc &NewDesc = TII->get(NewOpc);
```
- EN: Implements `MipsBranchExpansion::replaceBranch`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsBranchExpansion::replaceBranch`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 343-343
```cpp
  MachineInstrBuilder MIB = BuildMI(MBB, Br, DL, NewDesc);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 345-346
```cpp
  for (unsigned I = 0, E = Br->getDesc().getNumOperands(); I < E; ++I) {
    MachineOperand &MO = Br->getOperand(I);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 348-362
```cpp
    switch (MO.getType()) {
    case MachineOperand::MO_Register:
      MIB.addReg(MO.getReg());
      break;
    case MachineOperand::MO_Immediate:
      // Octeon BBIT family of branch has an immediate operand
      // (e.g. BBIT0 $v0, 3, %bb.1).
      if (!TII->isBranchWithImm(Br->getOpcode()))
        llvm_unreachable("Unexpected immediate in branch instruction");
      MIB.addImm(MO.getImm());
      break;
    case MachineOperand::MO_MachineBasicBlock:
      MIB.addMBB(MBBOpnd);
      break;
    default:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 363-365
```cpp
      llvm_unreachable("Unexpected operand type in branch instruction");
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 367-375
```cpp
  if (Br->hasDelaySlot()) {
    // Bundle the instruction in the delay slot to the newly created branch
    // and erase the original branch.
    assert(Br->isBundledWithSucc());
    MachineBasicBlock::instr_iterator II = Br.getInstrIterator();
    MIBundleBuilder(&*MIB).append((++II)->removeFromBundle());
  }
  Br->eraseFromParent();
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 377-381
```cpp
bool MipsBranchExpansion::buildProperJumpMI(MachineBasicBlock *MBB,
                                            MachineBasicBlock::iterator Pos,
                                            DebugLoc DL) {
  bool HasR6 = ABI.IsN64() ? STI->hasMips64r6() : STI->hasMips32r6();
  bool AddImm = HasR6 && !STI->useIndirectJumpsHazard();
```
- EN: Implements `MipsBranchExpansion::buildProperJumpMI`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsBranchExpansion::buildProperJumpMI`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 383-386
```cpp
  unsigned JR = ABI.IsN64() ? Mips::JR64 : Mips::JR;
  unsigned JIC = ABI.IsN64() ? Mips::JIC64 : Mips::JIC;
  unsigned JR_HB = ABI.IsN64() ? Mips::JR_HB64 : Mips::JR_HB;
  unsigned JR_HB_R6 = ABI.IsN64() ? Mips::JR_HB64_R6 : Mips::JR_HB_R6;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 388-392
```cpp
  unsigned JumpOp;
  if (STI->useIndirectJumpsHazard())
    JumpOp = HasR6 ? JR_HB_R6 : JR_HB;
  else
    JumpOp = HasR6 ? JIC : JR;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 394-395
```cpp
  if (JumpOp == Mips::JIC && STI->inMicroMipsMode())
    JumpOp = Mips::JIC_MMR6;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 397-401
```cpp
  unsigned ATReg = ABI.IsN64() ? Mips::AT_64 : Mips::AT;
  MachineInstrBuilder Instr =
      BuildMI(*MBB, Pos, DL, TII->get(JumpOp)).addReg(ATReg);
  if (AddImm)
    Instr.addImm(0);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 403-404
```cpp
  return !AddImm;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 406-417
```cpp
// Expand branch instructions to long branches.
// TODO: This function has to be fixed for beqz16 and bnez16, because it
// currently assumes that all branches have 16-bit offsets, and will produce
// wrong code if branches whose allowed offsets are [-128, -126, ..., 126]
// are present.
void MipsBranchExpansion::expandToLongBranch(MBBInfo &I) {
  MachineBasicBlock::iterator Pos;
  MachineBasicBlock *MBB = I.Br->getParent(), *TgtMBB = getTargetMBB(*I.Br);
  DebugLoc DL = I.Br->getDebugLoc();
  const BasicBlock *BB = MBB->getBasicBlock();
  MachineFunction::iterator FallThroughMBB = ++MachineFunction::iterator(MBB);
  MachineBasicBlock *LongBrMBB = MFp->CreateMachineBasicBlock(BB);
```
- EN: Implements `MipsBranchExpansion::expandToLongBranch`, a mutation/build routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsBranchExpansion::expandToLongBranch`，它是一个围绕机器函数状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 419-420
```cpp
  MFp->insert(FallThroughMBB, LongBrMBB);
  MBB->replaceSuccessor(TgtMBB, LongBrMBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 422-426
```cpp
  if (IsPIC) {
    MachineBasicBlock *BalTgtMBB = MFp->CreateMachineBasicBlock(BB);
    MFp->insert(FallThroughMBB, BalTgtMBB);
    LongBrMBB->addSuccessor(BalTgtMBB);
    BalTgtMBB->addSuccessor(TgtMBB);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 428-434
```cpp
    // We must select between the MIPS32r6/MIPS64r6 BALC (which is a normal
    // instruction) and the pre-MIPS32r6/MIPS64r6 definition (which is an
    // pseudo-instruction wrapping BGEZAL).
    const unsigned BalOp =
        STI->hasMips32r6()
            ? STI->inMicroMipsMode() ? Mips::BALC_MMR6 : Mips::BALC
            : STI->inMicroMipsMode() ? Mips::BAL_BR_MM : Mips::BAL_BR;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 436-450
```cpp
    if (!ABI.IsN64()) {
      // Pre R6:
      // $longbr:
      //  addiu $sp, $sp, -8
      //  sw $ra, 0($sp)
      //  lui $at, %hi($tgt - $baltgt)
      //  bal $baltgt
      //  addiu $at, $at, %lo($tgt - $baltgt)
      // $baltgt:
      //  addu $at, $ra, $at
      //  lw $ra, 0($sp)
      //  jr $at
      //  addiu $sp, $sp, 8
      // $fallthrough:
      //
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 452-464
```cpp
      // R6:
      // $longbr:
      //  addiu $sp, $sp, -8
      //  sw $ra, 0($sp)
      //  lui $at, %hi($tgt - $baltgt)
      //  addiu $at, $at, %lo($tgt - $baltgt)
      //  balc $baltgt
      // $baltgt:
      //  addu $at, $ra, $at
      //  lw $ra, 0($sp)
      //  addiu $sp, $sp, 8
      //  jic $at, 0
      // $fallthrough:
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 466-466
```cpp
      Pos = LongBrMBB->begin();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 468-474
```cpp
      BuildMI(*LongBrMBB, Pos, DL, TII->get(Mips::ADDiu), Mips::SP)
          .addReg(Mips::SP)
          .addImm(-8);
      BuildMI(*LongBrMBB, Pos, DL, TII->get(Mips::SW))
          .addReg(Mips::RA)
          .addReg(Mips::SP)
          .addImm(0);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 476-490
```cpp
      // LUi and ADDiu instructions create 32-bit offset of the target basic
      // block from the target of BAL(C) instruction.  We cannot use immediate
      // value for this offset because it cannot be determined accurately when
      // the program has inline assembly statements.  We therefore use the
      // relocation expressions %hi($tgt-$baltgt) and %lo($tgt-$baltgt) which
      // are resolved during the fixup, so the values will always be correct.
      //
      // Since we cannot create %hi($tgt-$baltgt) and %lo($tgt-$baltgt)
      // expressions at this point (it is possible only at the MC layer),
      // we replace LUi and ADDiu with pseudo instructions
      // LONG_BRANCH_LUi and LONG_BRANCH_ADDiu, and add both basic
      // blocks as operands to these instructions.  When lowering these pseudo
      // instructions to LUi and ADDiu in the MC layer, we will create
      // %hi($tgt-$baltgt) and %lo($tgt-$baltgt) expressions and add them as
      // operands to lowered instructions.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 492-494
```cpp
      BuildMI(*LongBrMBB, Pos, DL, TII->get(Mips::LONG_BRANCH_LUi), Mips::AT)
          .addMBB(TgtMBB, MipsII::MO_ABS_HI)
          .addMBB(BalTgtMBB);
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 496-510
```cpp
      MachineInstrBuilder BalInstr =
          BuildMI(*MFp, DL, TII->get(BalOp)).addMBB(BalTgtMBB);
      MachineInstrBuilder ADDiuInstr =
          BuildMI(*MFp, DL, TII->get(Mips::LONG_BRANCH_ADDiu), Mips::AT)
              .addReg(Mips::AT)
              .addMBB(TgtMBB, MipsII::MO_ABS_LO)
              .addMBB(BalTgtMBB);
      if (STI->hasMips32r6()) {
        LongBrMBB->insert(Pos, ADDiuInstr);
        LongBrMBB->insert(Pos, BalInstr);
      } else {
        LongBrMBB->insert(Pos, BalInstr);
        LongBrMBB->insert(Pos, ADDiuInstr);
        LongBrMBB->rbegin()->bundleWithPred();
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 512-512
```cpp
      Pos = BalTgtMBB->begin();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 514-519
```cpp
      BuildMI(*BalTgtMBB, Pos, DL, TII->get(Mips::ADDu), Mips::AT)
          .addReg(Mips::RA)
          .addReg(Mips::AT);
      BuildMI(*BalTgtMBB, Pos, DL, TII->get(Mips::LW), Mips::RA)
          .addReg(Mips::SP)
          .addImm(0);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 521-522
```cpp
      // For MIPS32R6, we can skip using a delay slot branch.
      bool hasDelaySlot = buildProperJumpMI(BalTgtMBB, Pos, DL);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 524-538
```cpp
      if (!hasDelaySlot) {
        BuildMI(*BalTgtMBB, std::prev(Pos), DL, TII->get(Mips::ADDiu), Mips::SP)
            .addReg(Mips::SP)
            .addImm(8);
      }
      if (hasDelaySlot) {
        BuildMI(*BalTgtMBB, Pos, DL, TII->get(Mips::ADDiu), Mips::SP)
            .addReg(Mips::SP)
            .addImm(8);
        BalTgtMBB->rbegin()->bundleWithPred();
      }
    } else {
      // Pre R6:
      // $longbr:
      //  daddiu $sp, $sp, -16
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 539-549
```cpp
      //  sd $ra, 0($sp)
      //  daddiu $at, $zero, %hi($tgt - $baltgt)
      //  dsll $at, $at, 16
      //  bal $baltgt
      //  daddiu $at, $at, %lo($tgt - $baltgt)
      // $baltgt:
      //  daddu $at, $ra, $at
      //  ld $ra, 0($sp)
      //  jr64 $at
      //  daddiu $sp, $sp, 16
      // $fallthrough:
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 551-564
```cpp
      // R6:
      // $longbr:
      //  daddiu $sp, $sp, -16
      //  sd $ra, 0($sp)
      //  daddiu $at, $zero, %hi($tgt - $baltgt)
      //  dsll $at, $at, 16
      //  daddiu $at, $at, %lo($tgt - $baltgt)
      //  balc $baltgt
      // $baltgt:
      //  daddu $at, $ra, $at
      //  ld $ra, 0($sp)
      //  daddiu $sp, $sp, 16
      //  jic $at, 0
      // $fallthrough:
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 566-567
```cpp
      // We assume the branch is within-function, and that offset is within
      // +/- 2GB.  High 32 bits will therefore always be zero.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 569-579
```cpp
      // Note that this will work even if the offset is negative, because
      // of the +1 modification that's added in that case.  For example, if the
      // offset is -1MB (0xFFFFFFFFFFF00000), the computation for %higher is
      //
      // 0xFFFFFFFFFFF00000 + 0x80008000 = 0x000000007FF08000
      //
      // and the bits [47:32] are zero.  For %highest
      //
      // 0xFFFFFFFFFFF00000 + 0x800080008000 = 0x000080007FF08000
      //
      // and the bits [63:48] are zero.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 581-581
```cpp
      Pos = LongBrMBB->begin();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 583-597
```cpp
      BuildMI(*LongBrMBB, Pos, DL, TII->get(Mips::DADDiu), Mips::SP_64)
          .addReg(Mips::SP_64)
          .addImm(-16);
      BuildMI(*LongBrMBB, Pos, DL, TII->get(Mips::SD))
          .addReg(Mips::RA_64)
          .addReg(Mips::SP_64)
          .addImm(0);
      BuildMI(*LongBrMBB, Pos, DL, TII->get(Mips::LONG_BRANCH_DADDiu),
              Mips::AT_64)
          .addReg(Mips::ZERO_64)
          .addMBB(TgtMBB, MipsII::MO_ABS_HI)
          .addMBB(BalTgtMBB);
      BuildMI(*LongBrMBB, Pos, DL, TII->get(Mips::DSLL), Mips::AT_64)
          .addReg(Mips::AT_64)
          .addImm(16);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 599-613
```cpp
      MachineInstrBuilder BalInstr =
          BuildMI(*MFp, DL, TII->get(BalOp)).addMBB(BalTgtMBB);
      MachineInstrBuilder DADDiuInstr =
          BuildMI(*MFp, DL, TII->get(Mips::LONG_BRANCH_DADDiu), Mips::AT_64)
              .addReg(Mips::AT_64)
              .addMBB(TgtMBB, MipsII::MO_ABS_LO)
              .addMBB(BalTgtMBB);
      if (STI->hasMips32r6()) {
        LongBrMBB->insert(Pos, DADDiuInstr);
        LongBrMBB->insert(Pos, BalInstr);
      } else {
        LongBrMBB->insert(Pos, BalInstr);
        LongBrMBB->insert(Pos, DADDiuInstr);
        LongBrMBB->rbegin()->bundleWithPred();
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 615-615
```cpp
      Pos = BalTgtMBB->begin();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 617-622
```cpp
      BuildMI(*BalTgtMBB, Pos, DL, TII->get(Mips::DADDu), Mips::AT_64)
          .addReg(Mips::RA_64)
          .addReg(Mips::AT_64);
      BuildMI(*BalTgtMBB, Pos, DL, TII->get(Mips::LD), Mips::RA_64)
          .addReg(Mips::SP_64)
          .addImm(0);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 624-638
```cpp
      bool hasDelaySlot = buildProperJumpMI(BalTgtMBB, Pos, DL);
      // If there is no delay slot, Insert stack adjustment before
      if (!hasDelaySlot) {
        BuildMI(*BalTgtMBB, std::prev(Pos), DL, TII->get(Mips::DADDiu),
                Mips::SP_64)
            .addReg(Mips::SP_64)
            .addImm(16);
      } else {
        BuildMI(*BalTgtMBB, Pos, DL, TII->get(Mips::DADDiu), Mips::SP_64)
            .addReg(Mips::SP_64)
            .addImm(16);
        BalTgtMBB->rbegin()->bundleWithPred();
      }
    }
  } else { // Not PIC
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 639-640
```cpp
    Pos = LongBrMBB->begin();
    LongBrMBB->addSuccessor(TgtMBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 642-652
```cpp
    // Compute the position of the potentiall jump instruction (basic blocks
    // before + 4 for the instruction)
    uint64_t JOffset = computeOffsetFromTheBeginning(MBB->getNumber()) +
                       MBBInfos[MBB->getNumber()].Size + 4;
    uint64_t TgtMBBOffset = computeOffsetFromTheBeginning(TgtMBB->getNumber());
    // If it's a forward jump, then TgtMBBOffset will be shifted by two
    // instructions
    if (JOffset < TgtMBBOffset)
      TgtMBBOffset += 2 * 4;
    // Compare 4 upper bits to check if it's the same segment
    bool SameSegmentJump = JOffset >> 28 == TgtMBBOffset >> 28;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 654-668
```cpp
    if (STI->hasMips32r6() && TII->isBranchOffsetInRange(Mips::BC, I.Offset)) {
      // R6:
      // $longbr:
      //  bc $tgt
      // $fallthrough:
      //
      BuildMI(*LongBrMBB, Pos, DL,
              TII->get(STI->inMicroMipsMode() ? Mips::BC_MMR6 : Mips::BC))
          .addMBB(TgtMBB);
    } else if (SameSegmentJump) {
      // Pre R6:
      // $longbr:
      //  j $tgt
      //  nop
      // $fallthrough:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 669-683
```cpp
      //
      BuildMI(*LongBrMBB, Pos, DL, TII->get(Mips::J)).addMBB(TgtMBB);
      TII->insertNop(*LongBrMBB, Pos, DL)->bundleWithPred();
    } else {
      // At this point, offset where we need to branch does not fit into
      // immediate field of the branch instruction and is not in the same
      // segment as jump instruction. Therefore we will break it into couple
      // instructions, where we first load the offset into register, and then we
      // do branch register.
      if (ABI.IsN64()) {
        BuildMI(*LongBrMBB, Pos, DL, TII->get(Mips::LONG_BRANCH_LUi2Op_64),
                Mips::AT_64)
            .addMBB(TgtMBB, MipsII::MO_HIGHEST);
        BuildMI(*LongBrMBB, Pos, DL, TII->get(Mips::LONG_BRANCH_DADDiu2Op),
                Mips::AT_64)
```
- EN: Implements `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 684-698
```cpp
            .addReg(Mips::AT_64)
            .addMBB(TgtMBB, MipsII::MO_HIGHER);
        BuildMI(*LongBrMBB, Pos, DL, TII->get(Mips::DSLL), Mips::AT_64)
            .addReg(Mips::AT_64)
            .addImm(16);
        BuildMI(*LongBrMBB, Pos, DL, TII->get(Mips::LONG_BRANCH_DADDiu2Op),
                Mips::AT_64)
            .addReg(Mips::AT_64)
            .addMBB(TgtMBB, MipsII::MO_ABS_HI);
        BuildMI(*LongBrMBB, Pos, DL, TII->get(Mips::DSLL), Mips::AT_64)
            .addReg(Mips::AT_64)
            .addImm(16);
        BuildMI(*LongBrMBB, Pos, DL, TII->get(Mips::LONG_BRANCH_DADDiu2Op),
                Mips::AT_64)
            .addReg(Mips::AT_64)
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 699-711
```cpp
            .addMBB(TgtMBB, MipsII::MO_ABS_LO);
      } else {
        BuildMI(*LongBrMBB, Pos, DL, TII->get(Mips::LONG_BRANCH_LUi2Op),
                Mips::AT)
            .addMBB(TgtMBB, MipsII::MO_ABS_HI);
        BuildMI(*LongBrMBB, Pos, DL, TII->get(Mips::LONG_BRANCH_ADDiu2Op),
                Mips::AT)
            .addReg(Mips::AT)
            .addMBB(TgtMBB, MipsII::MO_ABS_LO);
      }
      buildProperJumpMI(LongBrMBB, Pos, DL);
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 713-721
```cpp
  if (I.Br->isUnconditionalBranch()) {
    // Change branch destination.
    assert(I.Br->getDesc().getNumOperands() == 1);
    I.Br->removeOperand(0);
    I.Br->addOperand(MachineOperand::CreateMBB(LongBrMBB));
  } else
    // Change branch destination and reverse condition.
    replaceBranch(*MBB, I.Br, DL, &*FallThroughMBB);
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 723-733
```cpp
static void emitGPDisp(MachineFunction &F, const MipsInstrInfo *TII) {
  MachineBasicBlock &MBB = F.front();
  MachineBasicBlock::iterator I = MBB.begin();
  DebugLoc DL = MBB.findDebugLoc(MBB.begin());
  BuildMI(MBB, I, DL, TII->get(Mips::LUi), Mips::V0)
      .addExternalSymbol("_gp_disp", MipsII::MO_ABS_HI);
  BuildMI(MBB, I, DL, TII->get(Mips::ADDiu), Mips::V0)
      .addReg(Mips::V0)
      .addExternalSymbol("_gp_disp", MipsII::MO_ABS_LO);
  MBB.removeLiveIn(Mips::V0);
}
```
- EN: Implements `emitGPDisp`, a emission/printing routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `emitGPDisp`，它是一个围绕机器函数状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 735-738
```cpp
template <typename Pred, typename Safe>
bool MipsBranchExpansion::handleMFLOSlot(Pred Predicate, Safe SafeInSlot) {
  bool Changed = false;
  bool hasPendingMFLO = false;
```
- EN: Implements `MipsBranchExpansion::handleMFLOSlot`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsBranchExpansion::handleMFLOSlot`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 740-741
```cpp
  for (MachineFunction::iterator FI = MFp->begin(); FI != MFp->end(); ++FI) {
    for (Iter I = FI->begin(); I != FI->end(); ++I) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 743-745
```cpp
      if (!Predicate(*I) && !hasPendingMFLO) {
        continue;
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 747-761
```cpp
      Iter IInSlot;
      bool LastInstInFunction =
          std::next(I) == FI->end() && std::next(FI) == MFp->end();
      // We need process several situations:
      // mflo is last instruction, do not process;
      // mflo + div, add two nop between them;
      // mflo + none-div + none-div, do not process;
      // mflo + none-div + div, add nop between none-div and div.
      if (!LastInstInFunction) {
        std::pair<Iter, bool> Res = getNextMachineInstr(std::next(I), &*FI);
        LastInstInFunction |= Res.second;
        IInSlot = Res.first;
        if (LastInstInFunction)
          continue;
        if (!SafeInSlot(*IInSlot, *I)) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 762-776
```cpp
          Changed = true;
          TII->insertNop(*(I->getParent()), std::next(I), I->getDebugLoc())
              ->bundleWithPred();
          NumInsertedNops++;
          if (IsMFLOMFHI(I->getOpcode())) {
            TII->insertNop(*(I->getParent()), std::next(I), I->getDebugLoc())
                ->bundleWithPred();
            NumInsertedNops++;
          }
          if (hasPendingMFLO)
            hasPendingMFLO = false;
        } else if (hasPendingMFLO)
          hasPendingMFLO = false;
        else if (IsMFLOMFHI(I->getOpcode()))
          hasPendingMFLO = true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 777-779
```cpp
      }
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 781-782
```cpp
  return Changed;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 784-786
```cpp
template <typename Pred, typename Safe>
bool MipsBranchExpansion::handleSlot(Pred Predicate, Safe SafeInSlot) {
  bool Changed = false;
```
- EN: Implements `MipsBranchExpansion::handleSlot`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsBranchExpansion::handleSlot`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 788-789
```cpp
  for (MachineFunction::iterator FI = MFp->begin(); FI != MFp->end(); ++FI) {
    for (Iter I = FI->begin(); I != FI->end(); ++I) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 791-793
```cpp
      // Delay slot hazard handling. Use lookahead over state.
      if (!Predicate(*I))
        continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 795-802
```cpp
      Iter IInSlot;
      bool LastInstInFunction =
          std::next(I) == FI->end() && std::next(FI) == MFp->end();
      if (!LastInstInFunction) {
        std::pair<Iter, bool> Res = getNextMachineInstr(std::next(I), &*FI);
        LastInstInFunction |= Res.second;
        IInSlot = Res.first;
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 804-815
```cpp
      if (LastInstInFunction || !SafeInSlot(*IInSlot, *I)) {
        MachineBasicBlock::instr_iterator Iit = I->getIterator();
        if (std::next(Iit) == FI->end() ||
            std::next(Iit)->getOpcode() != Mips::NOP) {
          Changed = true;
          TII->insertNop(*(I->getParent()), std::next(I), I->getDebugLoc())
              ->bundleWithPred();
          NumInsertedNops++;
        }
      }
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 817-818
```cpp
  return Changed;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 820-824
```cpp
bool MipsBranchExpansion::handleMFLO() {
  // mips1-4 require a minimum of 2 instructions between a mflo/mfhi
  // and the next mul/div instruction.
  if (STI->hasMips32() || STI->hasMips5())
    return false;
```
- EN: Implements `MipsBranchExpansion::handleMFLO`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsBranchExpansion::handleMFLO`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 826-831
```cpp
  return handleMFLOSlot(
      [this](auto &I) -> bool { return TII->IsMfloOrMfhi(I); },
      [this](auto &IInSlot, auto &I) -> bool {
        return TII->SafeAfterMflo(IInSlot);
      });
}
```
- EN: Implements `handleMFLOSlot`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `handleMFLOSlot`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 833-836
```cpp
bool MipsBranchExpansion::handleForbiddenSlot() {
  // Forbidden slot hazards are only defined for MIPSR6 but not microMIPSR6.
  if (!STI->hasMips32r6() || STI->inMicroMipsMode())
    return false;
```
- EN: Implements `MipsBranchExpansion::handleForbiddenSlot`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsBranchExpansion::handleForbiddenSlot`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 838-843
```cpp
  return handleSlot(
      [this](auto &I) -> bool { return TII->HasForbiddenSlot(I); },
      [this](auto &IInSlot, auto &I) -> bool {
        return TII->SafeInForbiddenSlot(IInSlot);
      });
}
```
- EN: Implements `handleSlot`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `handleSlot`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 845-848
```cpp
bool MipsBranchExpansion::handleFPUDelaySlot() {
  // FPU delay slots are only defined for MIPS3 and below.
  if (STI->hasMips32() || STI->hasMips4())
    return false;
```
- EN: Implements `MipsBranchExpansion::handleFPUDelaySlot`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsBranchExpansion::handleFPUDelaySlot`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 850-854
```cpp
  return handleSlot([this](auto &I) -> bool { return TII->HasFPUDelaySlot(I); },
                    [this](auto &IInSlot, auto &I) -> bool {
                      return TII->SafeInFPUDelaySlot(IInSlot, I);
                    });
}
```
- EN: Implements `handleSlot`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `handleSlot`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 856-859
```cpp
bool MipsBranchExpansion::handleLoadDelaySlot() {
  // Load delay slot hazards are only for MIPS1.
  if (STI->hasMips2())
    return false;
```
- EN: Implements `MipsBranchExpansion::handleLoadDelaySlot`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsBranchExpansion::handleLoadDelaySlot`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 861-866
```cpp
  return handleSlot(
      [this](auto &I) -> bool { return TII->HasLoadDelaySlot(I); },
      [this](auto &IInSlot, auto &I) -> bool {
        return TII->SafeInLoadDelaySlot(IInSlot, I);
      });
}
```
- EN: Implements `handleSlot`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `handleSlot`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 868-870
```cpp
bool MipsBranchExpansion::handlePossibleLongBranch() {
  if (STI->inMips16Mode() || !STI->enableLongBranchPass())
    return false;
```
- EN: Implements `MipsBranchExpansion::handlePossibleLongBranch`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsBranchExpansion::handlePossibleLongBranch`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 872-873
```cpp
  if (SkipLongBranch)
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 875-875
```cpp
  bool EverMadeChange = false, MadeChange = true;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 877-878
```cpp
  while (MadeChange) {
    MadeChange = false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 880-880
```cpp
    initMBBInfo();
```
- EN: Declares `initMBBInfo`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `initMBBInfo`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 882-886
```cpp
    for (unsigned I = 0, E = MBBInfos.size(); I < E; ++I) {
      MachineBasicBlock *MBB = MFp->getBlockNumbered(I);
      // Search for MBB's branch instruction.
      ReverseIter End = MBB->rend();
      ReverseIter Br = getNonDebugInstr(MBB->rbegin(), End);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 888-891
```cpp
      if ((Br != End) && Br->isBranch() && !Br->isIndirectBranch() &&
          (Br->isConditionalBranch() ||
           (Br->isUnconditionalBranch() && IsPIC))) {
        int64_t Offset = computeOffset(&*Br);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 893-899
```cpp
        if (ForceLongBranchFirstPass ||
            !TII->isBranchOffsetInRange(Br->getOpcode(), Offset)) {
          MBBInfos[I].Offset = Offset;
          MBBInfos[I].Br = &*Br;
        }
      }
    } // End for
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 901-901
```cpp
    ForceLongBranchFirstPass = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 903-903
```cpp
    SmallVectorImpl<MBBInfo>::iterator I, E = MBBInfos.end();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 905-909
```cpp
    for (I = MBBInfos.begin(); I != E; ++I) {
      // Skip if this MBB doesn't have a branch or the branch has already been
      // converted to a long branch.
      if (!I->Br)
        continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 911-914
```cpp
      expandToLongBranch(*I);
      ++LongBranches;
      EverMadeChange = MadeChange = true;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 916-917
```cpp
    MFp->RenumberBlocks();
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 919-920
```cpp
  return EverMadeChange;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 922-927
```cpp
bool MipsBranchExpansion::runOnMachineFunction(MachineFunction &MF) {
  const TargetMachine &TM = MF.getTarget();
  IsPIC = TM.isPositionIndependent();
  ABI = static_cast<const MipsTargetMachine &>(TM).getABI();
  STI = &MF.getSubtarget<MipsSubtarget>();
  TII = STI->getInstrInfo();
```
- EN: Implements `MipsBranchExpansion::runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsBranchExpansion::runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 929-931
```cpp
  if (IsPIC && ABI.IsO32() &&
      MF.getInfo<MipsFunctionInfo>()->globalBaseRegSet())
    emitGPDisp(MF, TII);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 933-933
```cpp
  MFp = &MF;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 935-941
```cpp
  ForceLongBranchFirstPass = ForceLongBranch;
  // Run these at least once.
  bool longBranchChanged = handlePossibleLongBranch();
  bool forbiddenSlotChanged = handleForbiddenSlot();
  bool fpuDelaySlotChanged = handleFPUDelaySlot();
  bool loadDelaySlotChanged = handleLoadDelaySlot();
  bool MfloChanged = handleMFLO();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 943-944
```cpp
  bool Changed = longBranchChanged || forbiddenSlotChanged ||
                 fpuDelaySlotChanged || loadDelaySlotChanged || MfloChanged;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 946-956
```cpp
  // Then run them alternatively while there are changes.
  while (forbiddenSlotChanged) {
    longBranchChanged = handlePossibleLongBranch();
    fpuDelaySlotChanged = handleFPUDelaySlot();
    loadDelaySlotChanged = handleLoadDelaySlot();
    MfloChanged = handleMFLO();
    if (!longBranchChanged && !fpuDelaySlotChanged && !loadDelaySlotChanged &&
        !MfloChanged)
      break;
    forbiddenSlotChanged = handleForbiddenSlot();
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 958-959
```cpp
  return Changed;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

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

- EN: Backend-local headers: `MCTargetDesc/MipsABIInfo.h`, `MCTargetDesc/MipsBaseInfo.h`, `MCTargetDesc/MipsMCTargetDesc.h`, `Mips.h`, `MipsInstrInfo.h`, `MipsMachineFunction.h`, `MipsSubtarget.h`, `MipsTargetMachine.h`.
  - CN: 后端本地头文件：`MCTargetDesc/MipsABIInfo.h`, `MCTargetDesc/MipsBaseInfo.h`, `MCTargetDesc/MipsMCTargetDesc.h`, `Mips.h`, `MipsInstrInfo.h`, `MipsMachineFunction.h`, `MipsSubtarget.h`, `MipsTargetMachine.h`。
- EN: LLVM infrastructure headers: `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h`, `llvm/ADT/StringRef.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/MachineInstrBuilder.h` ... (+7 more).
  - CN: LLVM 基础设施头文件：`llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h`, `llvm/ADT/StringRef.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/MachineInstrBuilder.h` ... (+7 more)。
- EN: Standard/system headers: `algorithm`, `cassert`, `cstdint`, `iterator`, `utility`.
  - CN: 标准库/系统头文件：`algorithm`, `cassert`, `cstdint`, `iterator`, `utility`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
- EN: The implementation also depends on LLVM MC layer classes for final assembly/object emission.
  - CN: 实现还依赖 LLVM 的 MC 层类来完成最终汇编/目标文件输出。
