# DelaySlotFiller.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Sparc/DelaySlotFiller.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements delay-slot filling logic for targets that can schedule useful instructions after branches or calls.
  - **CN**: 为带有延迟槽的目标实现填槽逻辑，在分支或调用后调度有用指令。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===-- DelaySlotFiller.cpp - SPARC delay slot filler ---------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This is a simple local pass that attempts to fill delay slots with useful
// instructions. If no instructions can be moved into the delay slot, then a
// NOP is placed.
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 13-24
```cpp

#include "Sparc.h"
#include "SparcSubtarget.h"
#include "llvm/ADT/SmallSet.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/TargetInstrInfo.h"
#include "llvm/CodeGen/TargetRegisterInfo.h"
#include "llvm/Support/CommandLine.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `Sparc.h`, `SparcSubtarget.h`, `llvm/ADT/SmallSet.h`, `llvm/ADT/Statistic.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `Sparc.h`, `SparcSubtarget.h`, `llvm/ADT/SmallSet.h`, `llvm/ADT/Statistic.h`。

### Lines 25-36
```cpp
using namespace llvm;

#define DEBUG_TYPE "delay-slot-filler"

STATISTIC(FilledSlots, "Number of delay slots filled");

static cl::opt<bool> DisableDelaySlotFiller(
  "disable-sparc-delay-filler",
  cl::init(false),
  cl::desc("Disable the Sparc delay slot filler."),
  cl::Hidden);

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 37-45
```cpp
namespace {
  struct Filler : public MachineFunctionPass {
    const SparcSubtarget *Subtarget = nullptr;

    static char ID;
    Filler() : MachineFunctionPass(ID) {}

    StringRef getPassName() const override { return "SPARC Delay Slot Filler"; }

```
- **EN**: Introduces declarations for `Filler`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `Filler` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 46-54
```cpp
    bool runOnMachineBasicBlock(MachineBasicBlock &MBB);
    bool runOnMachineFunction(MachineFunction &F) override {
      bool Changed = false;
      Subtarget = &F.getSubtarget<SparcSubtarget>();

      // This pass invalidates liveness information when it reorders
      // instructions to fill delay slot.
      F.getRegInfo().invalidateLiveness();

```
- **EN**: Implements logic around `runOnMachineBasicBlock`, `runOnMachineFunction`, `getSubtarget<SparcSubtarget>`, `getRegInfo`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `runOnMachineBasicBlock`, `runOnMachineFunction`, `getSubtarget<SparcSubtarget>`, `getRegInfo` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 55-63
```cpp
      for (MachineBasicBlock &MBB : F)
        Changed |= runOnMachineBasicBlock(MBB);
      return Changed;
    }

    MachineFunctionProperties getRequiredProperties() const override {
      return MachineFunctionProperties().setNoVRegs();
    }

```
- **EN**: Implements logic around `runOnMachineBasicBlock`, `getRequiredProperties`, `MachineFunctionProperties`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `runOnMachineBasicBlock`, `getRequiredProperties`, `MachineFunctionProperties` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 64-74
```cpp
    void insertCallDefsUses(MachineBasicBlock::iterator MI,
                            SmallSet<unsigned, 32>& RegDefs,
                            SmallSet<unsigned, 32>& RegUses);

    void insertDefsUses(MachineBasicBlock::iterator MI,
                        SmallSet<unsigned, 32>& RegDefs,
                        SmallSet<unsigned, 32>& RegUses);

    bool IsRegInSet(SmallSet<unsigned, 32>& RegSet,
                    unsigned Reg);

```
- **EN**: Implements logic around `insertCallDefsUses`, `insertDefsUses`, `IsRegInSet`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `insertCallDefsUses`, `insertDefsUses`, `IsRegInSet` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 75-84
```cpp
    bool delayHasHazard(MachineBasicBlock::iterator candidate,
                        bool &sawLoad, bool &sawStore,
                        SmallSet<unsigned, 32> &RegDefs,
                        SmallSet<unsigned, 32> &RegUses);

    MachineBasicBlock::iterator
    findDelayInstr(MachineBasicBlock &MBB, MachineBasicBlock::iterator slot);

    bool needsUnimp(MachineBasicBlock::iterator I, unsigned &StructSize);

```
- **EN**: Implements logic around `delayHasHazard`, `findDelayInstr`, `needsUnimp`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `delayHasHazard`, `findDelayInstr`, `needsUnimp` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 85-93
```cpp
    bool tryCombineRestoreWithPrevInst(MachineBasicBlock &MBB,
                                       MachineBasicBlock::iterator MBBI);

  };
  char Filler::ID = 0;
} // end of anonymous namespace

/// createSparcDelaySlotFillerPass - Returns a pass that fills in delay
/// slots in Sparc MachineFunctions
```
- **EN**: Implements logic around `tryCombineRestoreWithPrevInst`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `tryCombineRestoreWithPrevInst` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 94-102
```cpp
///
FunctionPass *llvm::createSparcDelaySlotFillerPass() {
  return new Filler;
}


/// runOnMachineBasicBlock - Fill in delay slots for the given basic block.
/// We assume there is only one delay slot per delayed instruction.
///
```
- **EN**: Implements logic around `createSparcDelaySlotFillerPass`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `createSparcDelaySlotFillerPass` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 103-111
```cpp
bool Filler::runOnMachineBasicBlock(MachineBasicBlock &MBB) {
  bool Changed = false;
  Subtarget = &MBB.getParent()->getSubtarget<SparcSubtarget>();
  const TargetInstrInfo *TII = Subtarget->getInstrInfo();

  for (MachineBasicBlock::iterator I = MBB.begin(); I != MBB.end(); ) {
    MachineBasicBlock::iterator MI = I;
    ++I;

```
- **EN**: Implements logic around `runOnMachineBasicBlock`, `getParent`, `getInstrInfo`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `runOnMachineBasicBlock`, `getParent`, `getInstrInfo` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 112-129
```cpp
    // If MI is restore, try combining it with previous inst.
    if (!DisableDelaySlotFiller &&
        (MI->getOpcode() == SP::RESTORErr
         || MI->getOpcode() == SP::RESTOREri)) {
      Changed |= tryCombineRestoreWithPrevInst(MBB, MI);
      continue;
    }

    // TODO: If we ever want to support v7, this needs to be extended
    // to cover all floating point operations.
    if (!Subtarget->isV9() &&
        (MI->getOpcode() == SP::FCMPS || MI->getOpcode() == SP::FCMPD
         || MI->getOpcode() == SP::FCMPQ)) {
      BuildMI(MBB, I, MI->getDebugLoc(), TII->get(SP::NOP));
      Changed = true;
      continue;
    }

```
- **EN**: Implements logic around `getOpcode`, `tryCombineRestoreWithPrevInst`, `BuildMI`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getOpcode`, `tryCombineRestoreWithPrevInst`, `BuildMI` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 130-138
```cpp
    // If MI has no delay slot, skip.
    if (!MI->hasDelaySlot())
      continue;

    MachineBasicBlock::iterator D = MBB.end();

    if (!DisableDelaySlotFiller)
      D = findDelayInstr(MBB, MI);

```
- **EN**: Implements logic around `end`, `findDelayInstr`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `end`, `findDelayInstr` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 139-156
```cpp
    ++FilledSlots;
    Changed = true;

    if (D == MBB.end())
      BuildMI(MBB, I, MI->getDebugLoc(), TII->get(SP::NOP));
    else
      MBB.splice(I, &MBB, D);

    unsigned structSize = 0;
    if (needsUnimp(MI, structSize)) {
      MachineBasicBlock::iterator J = MI;
      ++J; // skip the delay filler.
      assert (J != MBB.end() && "MI needs a delay instruction.");
      BuildMI(MBB, ++J, MI->getDebugLoc(),
              TII->get(SP::UNIMP)).addImm(structSize);
      // Bundle the delay filler and unimp with the instruction.
      MIBundleBuilder(MBB, MachineBasicBlock::iterator(MI), J);
    } else {
```
- **EN**: Implements logic around `BuildMI`, `splice`, `assert`, `get`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `splice`, `assert`, `get`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 157-171
```cpp
      MIBundleBuilder(MBB, MachineBasicBlock::iterator(MI), I);
    }
  }
  return Changed;
}

MachineBasicBlock::iterator
Filler::findDelayInstr(MachineBasicBlock &MBB,
                       MachineBasicBlock::iterator slot)
{
  SmallSet<unsigned, 32> RegDefs;
  SmallSet<unsigned, 32> RegUses;
  bool sawLoad = false;
  bool sawStore = false;

```
- **EN**: Implements logic around `MIBundleBuilder`, `findDelayInstr`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `MIBundleBuilder`, `findDelayInstr` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 172-183
```cpp
  if (slot == MBB.begin())
    return MBB.end();

  unsigned Opc = slot->getOpcode();

  if (Opc == SP::RET || Opc == SP::TLS_CALL)
    return MBB.end();

  if (Opc == SP::RETL || Opc == SP::TAIL_CALL || Opc == SP::TAIL_CALLri) {
    MachineBasicBlock::iterator J = slot;
    --J;

```
- **EN**: Implements logic around `end`, `getOpcode`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `end`, `getOpcode` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 184-192
```cpp
    if (J->getOpcode() == SP::RESTORErr
        || J->getOpcode() == SP::RESTOREri) {
      // change retl to ret.
      if (Opc == SP::RETL)
        slot->setDesc(Subtarget->getInstrInfo()->get(SP::RET));
      return J;
    }
  }

```
- **EN**: Implements logic around `getOpcode`, `setDesc`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getOpcode`, `setDesc` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 193-202
```cpp
  // Call's delay filler can def some of call's uses.
  if (slot->isCall())
    insertCallDefsUses(slot, RegDefs, RegUses);
  else
    insertDefsUses(slot, RegDefs, RegUses);

  bool done = false;

  MachineBasicBlock::iterator I = slot;

```
- **EN**: Implements logic around `insertCallDefsUses`, `insertDefsUses`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `insertCallDefsUses`, `insertDefsUses` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 203-212
```cpp
  while (!done) {
    done = (I == MBB.begin());

    if (!done)
      --I;

    // Skip meta instructions.
    if (I->isMetaInstruction())
      continue;

```
- **EN**: Implements logic around `begin`; this block applies conditional target rules.
- **CN**: 围绕 `begin` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 213-221
```cpp
    if (I->hasUnmodeledSideEffects() || I->isInlineAsm() || I->isPosition() ||
        I->hasDelaySlot() || I->isBundledWithSucc())
      break;

    if (delayHasHazard(I, sawLoad, sawStore, RegDefs, RegUses)) {
      insertDefsUses(I, RegDefs, RegUses);
      continue;
    }

```
- **EN**: Implements logic around `hasDelaySlot`, `insertDefsUses`; this block applies conditional target rules.
- **CN**: 围绕 `hasDelaySlot`, `insertDefsUses` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 222-233
```cpp
    return I;
  }
  return MBB.end();
}

bool Filler::delayHasHazard(MachineBasicBlock::iterator candidate,
                            bool &sawLoad,
                            bool &sawStore,
                            SmallSet<unsigned, 32> &RegDefs,
                            SmallSet<unsigned, 32> &RegUses)
{

```
- **EN**: Implements logic around `end`, `delayHasHazard`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `end`, `delayHasHazard` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 234-242
```cpp
  if (candidate->isImplicitDef() || candidate->isKill())
    return true;

  if (candidate->mayLoad()) {
    sawLoad = true;
    if (sawStore)
      return true;
  }

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 243-254
```cpp
  if (candidate->mayStore()) {
    if (sawStore)
      return true;
    sawStore = true;
    if (sawLoad)
      return true;
  }

  for (const MachineOperand &MO : candidate->operands()) {
    if (!MO.isReg())
      continue; // skip

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 255-268
```cpp
    Register Reg = MO.getReg();

    if (MO.isDef()) {
      // check whether Reg is defined or used before delay slot.
      if (IsRegInSet(RegDefs, Reg) || IsRegInSet(RegUses, Reg))
        return true;
    }
    if (MO.isUse()) {
      // check whether Reg is defined before delay slot.
      if (IsRegInSet(RegDefs, Reg))
        return true;
    }
  }

```
- **EN**: Implements logic around `getReg`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getReg` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 269-282
```cpp
  unsigned Opcode = candidate->getOpcode();
  // LD and LDD may have NOPs inserted afterwards in the case of some LEON
  // processors, so we can't use the delay slot if this feature is switched-on.
  if (Subtarget->insertNOPLoad()
      &&
      Opcode >=  SP::LDDArr && Opcode <= SP::LDrr)
    return true;

  // Same as above for FDIV and FSQRT on some LEON processors.
  if (Subtarget->fixAllFDIVSQRT()
      &&
      Opcode >=  SP::FDIVD && Opcode <= SP::FSQRTD)
    return true;

```
- **EN**: Implements logic around `getOpcode`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getOpcode` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果。

### Lines 283-297
```cpp
  if (Subtarget->fixTN0009() && candidate->mayStore())
    return true;

  if (Subtarget->fixTN0013()) {
    switch (Opcode) {
    case SP::FDIVS:
    case SP::FDIVD:
    case SP::FSQRTS:
    case SP::FSQRTD:
      return true;
    default:
      break;
    }
  }

```
- **EN**: Implements target-specific case analysis using a `switch` over opcodes, fixups, or enum values.
- **CN**: 通过对 opcode、fixup 或枚举值执行 `switch` 分析来实现目标相关逻辑。

### Lines 298-315
```cpp
  return false;
}


void Filler::insertCallDefsUses(MachineBasicBlock::iterator MI,
                                SmallSet<unsigned, 32>& RegDefs,
                                SmallSet<unsigned, 32>& RegUses)
{
  // Regular calls define o7, which is visible to the instruction in delay slot.
  // On the other hand, tail calls preserve it.
  switch(MI->getOpcode()) {
  default: llvm_unreachable("Unknown opcode.");
  case SP::CALL:
    RegDefs.insert(SP::O7);
    break;
  case SP::TAIL_CALL:
    break;
  case SP::CALLrr:
```
- **EN**: Implements logic around `insertCallDefsUses`, `llvm_unreachable`, `insert`; this block uses `switch`-based dispatch; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `insertCallDefsUses`, `llvm_unreachable`, `insert` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 316-325
```cpp
  case SP::CALLri:
    RegDefs.insert(SP::O7);
    [[fallthrough]];
  case SP::TAIL_CALLri:
    assert(MI->getNumOperands() >= 2);
    const MachineOperand &Reg = MI->getOperand(0);
    assert(Reg.isReg() && "CALL first operand is not a register.");
    assert(Reg.isUse() && "CALL first operand is not a use.");
    RegUses.insert(Reg.getReg());

```
- **EN**: Implements logic around `insert`, `assert`, `getOperand`.
- **CN**: 围绕 `insert`, `assert`, `getOperand` 实现具体逻辑。

### Lines 326-335
```cpp
    const MachineOperand &Operand1 = MI->getOperand(1);
    if (Operand1.isImm() || Operand1.isGlobal())
        break;
    assert(Operand1.isReg() && "CALLrr second operand is not a register.");
    assert(Operand1.isUse() && "CALLrr second operand is not a use.");
    RegUses.insert(Operand1.getReg());
    break;
  }
}

```
- **EN**: Implements logic around `getOperand`, `assert`, `insert`; this block applies conditional target rules.
- **CN**: 围绕 `getOperand`, `assert`, `insert` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 336-344
```cpp
// Insert Defs and Uses of MI into the sets RegDefs and RegUses.
void Filler::insertDefsUses(MachineBasicBlock::iterator MI,
                            SmallSet<unsigned, 32>& RegDefs,
                            SmallSet<unsigned, 32>& RegUses)
{
  for (const MachineOperand &MO : MI->operands()) {
    if (!MO.isReg())
      continue;

```
- **EN**: Implements logic around `insertDefsUses`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `insertDefsUses` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 345-359
```cpp
    Register Reg = MO.getReg();
    if (Reg == 0)
      continue;
    if (MO.isDef())
      RegDefs.insert(Reg);
    if (MO.isUse()) {
      // Implicit register uses of retl are return values and
      // retl does not use them.
      if (MO.isImplicit() && MI->getOpcode() == SP::RETL)
        continue;
      RegUses.insert(Reg);
    }
  }
}

```
- **EN**: Implements logic around `getReg`, `insert`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getReg`, `insert` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 360-370
```cpp
// returns true if the Reg or its alias is in the RegSet.
bool Filler::IsRegInSet(SmallSet<unsigned, 32>& RegSet, unsigned Reg)
{
  // Check Reg and all aliased Registers.
  for (MCRegAliasIterator AI(Reg, Subtarget->getRegisterInfo(), true);
       AI.isValid(); ++AI)
    if (RegSet.count(*AI))
      return true;
  return false;
}

```
- **EN**: Implements logic around `IsRegInSet`, `isValid`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `IsRegInSet`, `isValid` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 371-388
```cpp
bool Filler::needsUnimp(MachineBasicBlock::iterator I, unsigned &StructSize)
{
  if (!I->isCall())
    return false;

  unsigned structSizeOpNum = 0;
  switch (I->getOpcode()) {
  default: llvm_unreachable("Unknown call opcode.");
  case SP::CALL:
    structSizeOpNum = 1;
    break;
  case SP::CALLrr:
  case SP::CALLri:
    structSizeOpNum = 2;
    break;
  case SP::TLS_CALL: return false;
  case SP::TAIL_CALLri:
  case SP::TAIL_CALL: return false;
```
- **EN**: Implements logic around `needsUnimp`, `llvm_unreachable`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `needsUnimp`, `llvm_unreachable` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 389-397
```cpp
  }

  const MachineOperand &MO = I->getOperand(structSizeOpNum);
  if (!MO.isImm())
    return false;
  StructSize = MO.getImm();
  return true;
}

```
- **EN**: Implements logic around `getOperand`, `getImm`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getOperand`, `getImm` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 398-406
```cpp
static bool combineRestoreADD(MachineBasicBlock &MBB,
                              MachineBasicBlock::iterator RestoreMI,
                              MachineBasicBlock::iterator AddMI,
                              const TargetInstrInfo *TII) {
  // Before:  add  <op0>, <op1>, %i[0-7]
  //          restore %g0, %g0, %i[0-7]
  //
  // After :  restore <op0>, <op1>, %o[0-7]

```
- **EN**: Implements logic around `combineRestoreADD`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `combineRestoreADD` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 407-416
```cpp
  const TargetRegisterInfo *TRI = &TII->getRegisterInfo();
  Register reg = AddMI->getOperand(0).getReg();
  if (reg < SP::I0 || reg > SP::I7)
    return false;

  // Check whether it uses %o7 as its source and the corresponding branch
  // instruction is a call.
  MachineBasicBlock::iterator LastInst = MBB.getFirstTerminator();
  bool IsCall = LastInst != MBB.end() && LastInst->isCall();

```
- **EN**: Implements logic around `getRegisterInfo`, `getOperand`, `getFirstTerminator`, `end`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getRegisterInfo`, `getOperand`, `getFirstTerminator`, `end` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 417-427
```cpp
  if (IsCall && AddMI->getOpcode() == SP::ADDrr &&
      AddMI->readsRegister(SP::O7, TRI))
    return false;

  if (IsCall && AddMI->getOpcode() == SP::ADDri &&
      AddMI->readsRegister(SP::O7, TRI))
    return false;

  // Erase RESTORE.
  RestoreMI->eraseFromParent();

```
- **EN**: Implements logic around `readsRegister`, `eraseFromParent`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `readsRegister`, `eraseFromParent` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 428-438
```cpp
  // Change ADD to RESTORE.
  AddMI->setDesc(TII->get((AddMI->getOpcode() == SP::ADDrr)
                          ? SP::RESTORErr
                          : SP::RESTOREri));

  // Map the destination register.
  AddMI->getOperand(0).setReg(reg - SP::I0 + SP::O0);

  return true;
}

```
- **EN**: Implements logic around `setDesc`, `getOperand`; this block returns target-specific results.
- **CN**: 围绕 `setDesc`, `getOperand` 实现具体逻辑；这一段返回目标相关结果。

### Lines 439-448
```cpp
static bool combineRestoreOR(MachineBasicBlock &MBB,
                             MachineBasicBlock::iterator RestoreMI,
                             MachineBasicBlock::iterator OrMI,
                             const TargetInstrInfo *TII) {
  // Before:  or  <op0>, <op1>, %i[0-7]
  //          restore %g0, %g0, %i[0-7]
  //    and <op0> or <op1> is zero,
  //
  // After :  restore <op0>, <op1>, %o[0-7]

```
- **EN**: Implements logic around `combineRestoreOR`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `combineRestoreOR` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 449-459
```cpp
  const TargetRegisterInfo *TRI = &TII->getRegisterInfo();
  Register reg = OrMI->getOperand(0).getReg();
  if (reg < SP::I0 || reg > SP::I7)
    return false;

  // check whether it is a copy.
  if (OrMI->getOpcode() == SP::ORrr
      && OrMI->getOperand(1).getReg() != SP::G0
      && OrMI->getOperand(2).getReg() != SP::G0)
    return false;

```
- **EN**: Implements logic around `getRegisterInfo`, `getOperand`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getRegisterInfo`, `getOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 460-469
```cpp
  if (OrMI->getOpcode() == SP::ORri
      && OrMI->getOperand(1).getReg() != SP::G0
      && (!OrMI->getOperand(2).isImm() || OrMI->getOperand(2).getImm() != 0))
    return false;

  // Check whether it uses %o7 as its source and the corresponding branch
  // instruction is a call.
  MachineBasicBlock::iterator LastInst = MBB.getFirstTerminator();
  bool IsCall = LastInst != MBB.end() && LastInst->isCall();

```
- **EN**: Implements logic around `getOperand`, `getFirstTerminator`, `end`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getOperand`, `getFirstTerminator`, `end` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 470-481
```cpp
  if (IsCall && OrMI->getOpcode() == SP::ORrr &&
      OrMI->readsRegister(SP::O7, TRI))
    return false;

  // Erase RESTORE.
  RestoreMI->eraseFromParent();

  // Change OR to RESTORE.
  OrMI->setDesc(TII->get((OrMI->getOpcode() == SP::ORrr)
                         ? SP::RESTORErr
                         : SP::RESTOREri));

```
- **EN**: Implements logic around `readsRegister`, `eraseFromParent`, `setDesc`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `readsRegister`, `eraseFromParent`, `setDesc` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 482-496
```cpp
  // Map the destination register.
  OrMI->getOperand(0).setReg(reg - SP::I0 + SP::O0);

  return true;
}

static bool combineRestoreSETHIi(MachineBasicBlock::iterator RestoreMI,
                                 MachineBasicBlock::iterator SetHiMI,
                                 const TargetInstrInfo *TII)
{
  // Before:  sethi imm3, %i[0-7]
  //          restore %g0, %g0, %g0
  //
  // After :  restore %g0, (imm3<<10), %o[0-7]

```
- **EN**: Implements logic around `getOperand`, `combineRestoreSETHIi`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getOperand`, `combineRestoreSETHIi` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 497-505
```cpp
  Register reg = SetHiMI->getOperand(0).getReg();
  if (reg < SP::I0 || reg > SP::I7)
    return false;

  if (!SetHiMI->getOperand(1).isImm())
    return false;

  int64_t imm = SetHiMI->getOperand(1).getImm();

```
- **EN**: Implements logic around `getOperand`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 506-514
```cpp
  // Is it a 3 bit immediate?
  if (!isInt<3>(imm))
    return false;

  // Make it a 13 bit immediate.
  imm = (imm << 10) & 0x1FFF;

  assert(RestoreMI->getOpcode() == SP::RESTORErr);

```
- **EN**: Implements logic around `assert`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `assert` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 515-524
```cpp
  RestoreMI->setDesc(TII->get(SP::RESTOREri));

  RestoreMI->getOperand(0).setReg(reg - SP::I0 + SP::O0);
  RestoreMI->getOperand(1).setReg(SP::G0);
  RestoreMI->getOperand(2).ChangeToImmediate(imm);


  // Erase the original SETHI.
  SetHiMI->eraseFromParent();

```
- **EN**: Implements logic around `setDesc`, `getOperand`, `eraseFromParent`.
- **CN**: 围绕 `setDesc`, `getOperand`, `eraseFromParent` 实现具体逻辑。

### Lines 525-534
```cpp
  return true;
}

bool Filler::tryCombineRestoreWithPrevInst(MachineBasicBlock &MBB,
                                        MachineBasicBlock::iterator MBBI)
{
  // No previous instruction.
  if (MBBI == MBB.begin())
    return false;

```
- **EN**: Implements logic around `tryCombineRestoreWithPrevInst`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `tryCombineRestoreWithPrevInst` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 535-546
```cpp
  // assert that MBBI is a "restore %g0, %g0, %g0".
  assert(MBBI->getOpcode() == SP::RESTORErr
         && MBBI->getOperand(0).getReg() == SP::G0
         && MBBI->getOperand(1).getReg() == SP::G0
         && MBBI->getOperand(2).getReg() == SP::G0);

  MachineBasicBlock::iterator PrevInst = std::prev(MBBI);

  // It cannot be combined with a bundled instruction.
  if (PrevInst->isBundledWithSucc())
    return false;

```
- **EN**: Implements logic around `assert`, `getOperand`, `prev`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `assert`, `getOperand`, `prev` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 547-561
```cpp
  const TargetInstrInfo *TII = Subtarget->getInstrInfo();

  switch (PrevInst->getOpcode()) {
  default: break;
  case SP::ADDrr:
  case SP::ADDri:
    return combineRestoreADD(MBB, MBBI, PrevInst, TII);
  case SP::ORrr:
  case SP::ORri:
    return combineRestoreOR(MBB, MBBI, PrevInst, TII);
  case SP::SETHIi: return combineRestoreSETHIi(MBBI, PrevInst, TII); break;
  }
  // It cannot combine with the previous instruction.
  return false;
}
```
- **EN**: Implements logic around `getInstrInfo`, `combineRestoreADD`, `combineRestoreOR`, `combineRestoreSETHIi`; this block uses `switch`-based dispatch; returns target-specific results.
- **CN**: 围绕 `getInstrInfo`, `combineRestoreADD`, `combineRestoreOR`, `combineRestoreSETHIi` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果。

## Key Concepts / 关键概念

- **Delay slots / 延迟槽**:
  - **EN**: Finds safe instructions to occupy architectural delay slots
  - **CN**: 找到可安全填入体系结构延迟槽的指令
- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `Sparc.h`, `SparcSubtarget.h`, `llvm/ADT/SmallSet.h`, `llvm/ADT/Statistic.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineRegisterInfo.h`, `llvm/CodeGen/TargetInstrInfo.h`, `llvm/CodeGen/TargetRegisterInfo.h`, `llvm/Support/CommandLine.h`
- **LLVM subsystems / LLVM 子系统**: CodeGen, Support
