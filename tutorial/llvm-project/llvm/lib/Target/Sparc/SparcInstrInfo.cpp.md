# SparcInstrInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Sparc/SparcInstrInfo.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements target-specific instruction information, instruction helpers, and scheduling-related behavior.
  - **CN**: 实现目标相关的指令信息、指令辅助逻辑以及与调度相关的行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===-- SparcInstrInfo.cpp - Sparc Instruction Information ----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the Sparc implementation of the TargetInstrInfo class.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 13-25
```cpp
#include "SparcInstrInfo.h"
#include "Sparc.h"
#include "SparcMachineFunctionInfo.h"
#include "SparcSubtarget.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineMemOperand.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/Support/ErrorHandling.h"

using namespace llvm;

```
- **EN**: Pulls in the headers needed for this implementation, including `SparcInstrInfo.h`, `Sparc.h`, `SparcMachineFunctionInfo.h`, `SparcSubtarget.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `SparcInstrInfo.h`, `Sparc.h`, `SparcMachineFunctionInfo.h`, `SparcSubtarget.h`。

### Lines 26-39
```cpp
#define GET_INSTRINFO_CTOR_DTOR
#include "SparcGenInstrInfo.inc"

static cl::opt<unsigned> BPccDisplacementBits(
    "sparc-bpcc-offset-bits", cl::Hidden, cl::init(19),
    cl::desc("Restrict range of BPcc/FBPfcc instructions (DEBUG)"));

static cl::opt<unsigned>
    BPrDisplacementBits("sparc-bpr-offset-bits", cl::Hidden, cl::init(16),
                        cl::desc("Restrict range of BPr instructions (DEBUG)"));

// Pin the vtable to this file.
void SparcInstrInfo::anchor() {}

```
- **EN**: Pulls in the headers needed for this implementation, including `SparcGenInstrInfo.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `SparcGenInstrInfo.inc`。

### Lines 40-63
```cpp
SparcInstrInfo::SparcInstrInfo(const SparcSubtarget &ST)
    : SparcGenInstrInfo(ST, RI, SP::ADJCALLSTACKDOWN, SP::ADJCALLSTACKUP),
      RI(ST), Subtarget(ST) {}

/// isLoadFromStackSlot - If the specified machine instruction is a direct
/// load from a stack slot, return the virtual or physical register number of
/// the destination along with the FrameIndex of the loaded stack slot.  If
/// not, return 0.  This predicate must return 0 if the instruction has
/// any side effects other than loading from the stack slot.
Register SparcInstrInfo::isLoadFromStackSlot(const MachineInstr &MI,
                                             int &FrameIndex,
                                             TypeSize &MemBytes) const {
  switch (MI.getOpcode()) {
  default:
    return 0;
  case SP::LDri:
    MemBytes = TypeSize::getFixed(4);
    break;
  case SP::LDXri:
    MemBytes = TypeSize::getFixed(8);
    break;
  case SP::LDFri:
    MemBytes = TypeSize::getFixed(4);
    break;
```
- **EN**: Implements logic around `SparcInstrInfo`, `SparcGenInstrInfo`, `RI`, `isLoadFromStackSlot`, ...; this block uses `switch`-based dispatch; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `SparcInstrInfo`, `SparcGenInstrInfo`, `RI`, `isLoadFromStackSlot`, ... 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 64-78
```cpp
  case SP::LDDFri:
    MemBytes = TypeSize::getFixed(8);
    break;
  case SP::LDQFri:
    MemBytes = TypeSize::getFixed(16);
    break;
  }
  if (MI.getOperand(1).isFI() && MI.getOperand(2).isImm() &&
      MI.getOperand(2).getImm() == 0) {
    FrameIndex = MI.getOperand(1).getIndex();
    return MI.getOperand(0).getReg();
  }
  return 0;
}

```
- **EN**: Implements logic around `getFixed`, `getOperand`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getFixed`, `getOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 79-102
```cpp
/// isStoreToStackSlot - If the specified machine instruction is a direct
/// store to a stack slot, return the virtual or physical register number of
/// the source reg along with the FrameIndex of the loaded stack slot.  If
/// not, return 0.  This predicate must return 0 if the instruction has
/// any side effects other than storing to the stack slot.
Register SparcInstrInfo::isStoreToStackSlot(const MachineInstr &MI,
                                            int &FrameIndex,
                                            TypeSize &MemBytes) const {
  switch (MI.getOpcode()) {
  default:
    return 0;
  case SP::STri:
    MemBytes = TypeSize::getFixed(4);
    break;
  case SP::STXri:
    MemBytes = TypeSize::getFixed(8);
    break;
  case SP::STFri:
    MemBytes = TypeSize::getFixed(4);
    break;
  case SP::STDFri:
    MemBytes = TypeSize::getFixed(8);
    break;
  case SP::STQFri:
```
- **EN**: Implements logic around `isStoreToStackSlot`, `getFixed`; this block uses `switch`-based dispatch; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `isStoreToStackSlot`, `getFixed` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 103-126
```cpp
    MemBytes = TypeSize::getFixed(16);
    break;
  }
  if (MI.getOperand(0).isFI() && MI.getOperand(1).isImm() &&
      MI.getOperand(1).getImm() == 0) {
    FrameIndex = MI.getOperand(0).getIndex();
    return MI.getOperand(2).getReg();
  }
  return 0;
}

static SPCC::CondCodes GetOppositeBranchCondition(SPCC::CondCodes CC)
{
  switch(CC) {
  case SPCC::ICC_A:    return SPCC::ICC_N;
  case SPCC::ICC_N:    return SPCC::ICC_A;
  case SPCC::ICC_NE:   return SPCC::ICC_E;
  case SPCC::ICC_E:    return SPCC::ICC_NE;
  case SPCC::ICC_G:    return SPCC::ICC_LE;
  case SPCC::ICC_LE:   return SPCC::ICC_G;
  case SPCC::ICC_GE:   return SPCC::ICC_L;
  case SPCC::ICC_L:    return SPCC::ICC_GE;
  case SPCC::ICC_GU:   return SPCC::ICC_LEU;
  case SPCC::ICC_LEU:  return SPCC::ICC_GU;
```
- **EN**: Implements logic around `getFixed`, `getOperand`, `GetOppositeBranchCondition`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getFixed`, `getOperand`, `GetOppositeBranchCondition` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果。

### Lines 127-150
```cpp
  case SPCC::ICC_CC:   return SPCC::ICC_CS;
  case SPCC::ICC_CS:   return SPCC::ICC_CC;
  case SPCC::ICC_POS:  return SPCC::ICC_NEG;
  case SPCC::ICC_NEG:  return SPCC::ICC_POS;
  case SPCC::ICC_VC:   return SPCC::ICC_VS;
  case SPCC::ICC_VS:   return SPCC::ICC_VC;

  case SPCC::FCC_A:    return SPCC::FCC_N;
  case SPCC::FCC_N:    return SPCC::FCC_A;
  case SPCC::FCC_U:    return SPCC::FCC_O;
  case SPCC::FCC_O:    return SPCC::FCC_U;
  case SPCC::FCC_G:    return SPCC::FCC_ULE;
  case SPCC::FCC_LE:   return SPCC::FCC_UG;
  case SPCC::FCC_UG:   return SPCC::FCC_LE;
  case SPCC::FCC_ULE:  return SPCC::FCC_G;
  case SPCC::FCC_L:    return SPCC::FCC_UGE;
  case SPCC::FCC_GE:   return SPCC::FCC_UL;
  case SPCC::FCC_UL:   return SPCC::FCC_GE;
  case SPCC::FCC_UGE:  return SPCC::FCC_L;
  case SPCC::FCC_LG:   return SPCC::FCC_UE;
  case SPCC::FCC_UE:   return SPCC::FCC_LG;
  case SPCC::FCC_NE:   return SPCC::FCC_E;
  case SPCC::FCC_E:    return SPCC::FCC_NE;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 151-172
```cpp
  case SPCC::CPCC_A:   return SPCC::CPCC_N;
  case SPCC::CPCC_N:   return SPCC::CPCC_A;
  case SPCC::CPCC_3:   [[fallthrough]];
  case SPCC::CPCC_2:   [[fallthrough]];
  case SPCC::CPCC_23:  [[fallthrough]];
  case SPCC::CPCC_1:   [[fallthrough]];
  case SPCC::CPCC_13:  [[fallthrough]];
  case SPCC::CPCC_12:  [[fallthrough]];
  case SPCC::CPCC_123: [[fallthrough]];
  case SPCC::CPCC_0:   [[fallthrough]];
  case SPCC::CPCC_03:  [[fallthrough]];
  case SPCC::CPCC_02:  [[fallthrough]];
  case SPCC::CPCC_023: [[fallthrough]];
  case SPCC::CPCC_01:  [[fallthrough]];
  case SPCC::CPCC_013: [[fallthrough]];
  case SPCC::CPCC_012:
      // "Opposite" code is not meaningful, as we don't know
      // what the CoProc condition means here. The cond-code will
      // only be used in inline assembler, so this code should
      // not be reached in a normal compilation pass.
      llvm_unreachable("Meaningless inversion of co-processor cond code");

```
- **EN**: Implements logic around `llvm_unreachable`; this block returns target-specific results.
- **CN**: 围绕 `llvm_unreachable` 实现具体逻辑；这一段返回目标相关结果。

### Lines 173-190
```cpp
  case SPCC::REG_BEGIN:
      llvm_unreachable("Use of reserved cond code");
  case SPCC::REG_Z:
      return SPCC::REG_NZ;
  case SPCC::REG_LEZ:
      return SPCC::REG_GZ;
  case SPCC::REG_LZ:
      return SPCC::REG_GEZ;
  case SPCC::REG_NZ:
      return SPCC::REG_Z;
  case SPCC::REG_GZ:
      return SPCC::REG_LEZ;
  case SPCC::REG_GEZ:
      return SPCC::REG_LZ;
  }
  llvm_unreachable("Invalid cond code");
}

```
- **EN**: Implements logic around `llvm_unreachable`; this block returns target-specific results.
- **CN**: 围绕 `llvm_unreachable` 实现具体逻辑；这一段返回目标相关结果。

### Lines 191-202
```cpp
static bool isUncondBranchOpcode(int Opc) { return Opc == SP::BA; }

static bool isI32CondBranchOpcode(int Opc) {
  return Opc == SP::BCOND || Opc == SP::BPICC || Opc == SP::BPICCA ||
         Opc == SP::BPICCNT || Opc == SP::BPICCANT;
}

static bool isI64CondBranchOpcode(int Opc) {
  return Opc == SP::BPXCC || Opc == SP::BPXCCA || Opc == SP::BPXCCNT ||
         Opc == SP::BPXCCANT;
}

```
- **EN**: Implements logic around `isUncondBranchOpcode`, `isI32CondBranchOpcode`, `isI64CondBranchOpcode`; this block returns target-specific results.
- **CN**: 围绕 `isUncondBranchOpcode`, `isI32CondBranchOpcode`, `isI64CondBranchOpcode` 实现具体逻辑；这一段返回目标相关结果。

### Lines 203-217
```cpp
static bool isRegCondBranchOpcode(int Opc) {
  return Opc == SP::BPR || Opc == SP::BPRA || Opc == SP::BPRNT ||
         Opc == SP::BPRANT;
}

static bool isFCondBranchOpcode(int Opc) {
  return Opc == SP::FBCOND || Opc == SP::FBCONDA || Opc == SP::FBCOND_V9 ||
         Opc == SP::FBCONDA_V9;
}

static bool isCondBranchOpcode(int Opc) {
  return isI32CondBranchOpcode(Opc) || isI64CondBranchOpcode(Opc) ||
         isRegCondBranchOpcode(Opc) || isFCondBranchOpcode(Opc);
}

```
- **EN**: Implements logic around `isRegCondBranchOpcode`, `isFCondBranchOpcode`, `isCondBranchOpcode`, `isI32CondBranchOpcode`; this block returns target-specific results.
- **CN**: 围绕 `isRegCondBranchOpcode`, `isFCondBranchOpcode`, `isCondBranchOpcode`, `isI32CondBranchOpcode` 实现具体逻辑；这一段返回目标相关结果。

### Lines 218-231
```cpp
static bool isIndirectBranchOpcode(int Opc) {
  return Opc == SP::BINDrr || Opc == SP::BINDri;
}

static void parseCondBranch(MachineInstr *LastInst, MachineBasicBlock *&Target,
                            SmallVectorImpl<MachineOperand> &Cond) {
  unsigned Opc = LastInst->getOpcode();
  int64_t CC = LastInst->getOperand(1).getImm();

  // Push the branch opcode into Cond too so later in insertBranch
  // it can use the information to emit the correct SPARC branch opcode.
  Cond.push_back(MachineOperand::CreateImm(Opc));
  Cond.push_back(MachineOperand::CreateImm(CC));

```
- **EN**: Implements logic around `isIndirectBranchOpcode`, `parseCondBranch`, `getOpcode`, `getOperand`, ...; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `isIndirectBranchOpcode`, `parseCondBranch`, `getOpcode`, `getOperand`, ... 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 232-255
```cpp
  // Branch on register contents need another argument to indicate
  // the register it branches on.
  if (isRegCondBranchOpcode(Opc)) {
      Register Reg = LastInst->getOperand(2).getReg();
      Cond.push_back(MachineOperand::CreateReg(Reg, false));
  }

  Target = LastInst->getOperand(0).getMBB();
}

MachineBasicBlock *
SparcInstrInfo::getBranchDestBlock(const MachineInstr &MI) const {
  switch (MI.getOpcode()) {
  default:
      llvm_unreachable("unexpected opcode!");
  case SP::BA:
  case SP::BCOND:
  case SP::BCONDA:
  case SP::FBCOND:
  case SP::FBCONDA:
  case SP::BPICC:
  case SP::BPICCA:
  case SP::BPICCNT:
  case SP::BPICCANT:
```
- **EN**: Implements logic around `getOperand`, `push_back`, `getBranchDestBlock`, `llvm_unreachable`; this block uses `switch`-based dispatch; applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getOperand`, `push_back`, `getBranchDestBlock`, `llvm_unreachable` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 256-273
```cpp
  case SP::BPXCC:
  case SP::BPXCCA:
  case SP::BPXCCNT:
  case SP::BPXCCANT:
  case SP::BPFCC:
  case SP::BPFCCA:
  case SP::BPFCCNT:
  case SP::BPFCCANT:
  case SP::FBCOND_V9:
  case SP::FBCONDA_V9:
  case SP::BPR:
  case SP::BPRA:
  case SP::BPRNT:
  case SP::BPRANT:
      return MI.getOperand(0).getMBB();
  }
}

```
- **EN**: Implements logic around `getOperand`; this block returns target-specific results.
- **CN**: 围绕 `getOperand` 实现具体逻辑；这一段返回目标相关结果。

### Lines 274-285
```cpp
bool SparcInstrInfo::analyzeBranch(MachineBasicBlock &MBB,
                                   MachineBasicBlock *&TBB,
                                   MachineBasicBlock *&FBB,
                                   SmallVectorImpl<MachineOperand> &Cond,
                                   bool AllowModify) const {
  MachineBasicBlock::iterator I = MBB.getLastNonDebugInstr();
  if (I == MBB.end())
    return false;

  if (!isUnpredicatedTerminator(*I))
    return false;

```
- **EN**: Implements logic around `analyzeBranch`, `getLastNonDebugInstr`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `analyzeBranch`, `getLastNonDebugInstr` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 286-303
```cpp
  // Get the last instruction in the block.
  MachineInstr *LastInst = &*I;
  unsigned LastOpc = LastInst->getOpcode();

  // If there is only one terminator instruction, process it.
  if (I == MBB.begin() || !isUnpredicatedTerminator(*--I)) {
    if (isUncondBranchOpcode(LastOpc)) {
      TBB = LastInst->getOperand(0).getMBB();
      return false;
    }
    if (isCondBranchOpcode(LastOpc)) {
      // Block ends with fall-through condbranch.
      parseCondBranch(LastInst, TBB, Cond);
      return false;
    }
    return true; // Can't handle indirect branch.
  }

```
- **EN**: Implements logic around `getOpcode`, `getOperand`, `parseCondBranch`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getOpcode`, `getOperand`, `parseCondBranch` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 304-325
```cpp
  // Get the instruction before it if it is a terminator.
  MachineInstr *SecondLastInst = &*I;
  unsigned SecondLastOpc = SecondLastInst->getOpcode();

  // If AllowModify is true and the block ends with two or more unconditional
  // branches, delete all but the first unconditional branch.
  if (AllowModify && isUncondBranchOpcode(LastOpc)) {
    while (isUncondBranchOpcode(SecondLastOpc)) {
      LastInst->eraseFromParent();
      LastInst = SecondLastInst;
      LastOpc = LastInst->getOpcode();
      if (I == MBB.begin() || !isUnpredicatedTerminator(*--I)) {
        // Return now the only terminator is an unconditional branch.
        TBB = LastInst->getOperand(0).getMBB();
        return false;
      } else {
        SecondLastInst = &*I;
        SecondLastOpc = SecondLastInst->getOpcode();
      }
    }
  }

```
- **EN**: Implements logic around `getOpcode`, `eraseFromParent`, `getOperand`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getOpcode`, `eraseFromParent`, `getOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 326-343
```cpp
  // If there are three terminators, we don't know what sort of block this is.
  if (SecondLastInst && I != MBB.begin() && isUnpredicatedTerminator(*--I))
    return true;

  // If the block ends with a B and a Bcc, handle it.
  if (isCondBranchOpcode(SecondLastOpc) && isUncondBranchOpcode(LastOpc)) {
    parseCondBranch(SecondLastInst, TBB, Cond);
    FBB = LastInst->getOperand(0).getMBB();
    return false;
  }

  // If the block ends with two unconditional branches, handle it.  The second
  // one is not executed.
  if (isUncondBranchOpcode(SecondLastOpc) && isUncondBranchOpcode(LastOpc)) {
    TBB = SecondLastInst->getOperand(0).getMBB();
    return false;
  }

```
- **EN**: Implements logic around `parseCondBranch`, `getOperand`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `parseCondBranch`, `getOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 344-356
```cpp
  // ...likewise if it ends with an indirect branch followed by an unconditional
  // branch.
  if (isIndirectBranchOpcode(SecondLastOpc) && isUncondBranchOpcode(LastOpc)) {
    I = LastInst;
    if (AllowModify)
      I->eraseFromParent();
    return true;
  }

  // Otherwise, can't handle this.
  return true;
}

```
- **EN**: Implements logic around `eraseFromParent`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `eraseFromParent` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 357-374
```cpp
unsigned SparcInstrInfo::insertBranch(MachineBasicBlock &MBB,
                                      MachineBasicBlock *TBB,
                                      MachineBasicBlock *FBB,
                                      ArrayRef<MachineOperand> Cond,
                                      const DebugLoc &DL,
                                      int *BytesAdded) const {
  assert(TBB && "insertBranch must not be told to insert a fallthrough");
  assert((Cond.size() <= 3) &&
         "Sparc branch conditions should have at most three components!");

  if (Cond.empty()) {
    assert(!FBB && "Unconditional branch with multiple successors!");
    BuildMI(&MBB, DL, get(SP::BA)).addMBB(TBB);
    if (BytesAdded)
      *BytesAdded = 8;
    return 1;
  }

```
- **EN**: Implements logic around `insertBranch`, `assert`, `BuildMI`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `insertBranch`, `assert`, `BuildMI` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 375-390
```cpp
  // Conditional branch
  unsigned Opc = Cond[0].getImm();
  unsigned CC = Cond[1].getImm();
  if (isRegCondBranchOpcode(Opc)) {
    Register Reg = Cond[2].getReg();
    BuildMI(&MBB, DL, get(Opc)).addMBB(TBB).addImm(CC).addReg(Reg);
  } else {
    BuildMI(&MBB, DL, get(Opc)).addMBB(TBB).addImm(CC);
  }

  if (!FBB) {
    if (BytesAdded)
      *BytesAdded = 8;
    return 1;
  }

```
- **EN**: Implements logic around `getImm`, `getReg`, `BuildMI`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getImm`, `getReg`, `BuildMI` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 391-404
```cpp
  BuildMI(&MBB, DL, get(SP::BA)).addMBB(FBB);
  if (BytesAdded)
    *BytesAdded = 16;
  return 2;
}

unsigned SparcInstrInfo::removeBranch(MachineBasicBlock &MBB,
                                      int *BytesRemoved) const {
  MachineBasicBlock::iterator I = MBB.end();
  unsigned Count = 0;
  int Removed = 0;
  while (I != MBB.begin()) {
    --I;

```
- **EN**: Implements logic around `BuildMI`, `removeBranch`, `end`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `removeBranch`, `end` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 405-417
```cpp
    if (I->isDebugInstr())
      continue;

    if (!isCondBranchOpcode(I->getOpcode()) &&
        !isUncondBranchOpcode(I->getOpcode()))
      break; // Not a branch

    Removed += getInstSizeInBytes(*I);
    I->eraseFromParent();
    I = MBB.end();
    ++Count;
  }

```
- **EN**: Implements logic around `isUncondBranchOpcode`, `getInstSizeInBytes`, `eraseFromParent`, `end`; this block applies conditional target rules.
- **CN**: 围绕 `isUncondBranchOpcode`, `getInstSizeInBytes`, `eraseFromParent`, `end` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 418-430
```cpp
  if (BytesRemoved)
    *BytesRemoved = Removed;
  return Count;
}

bool SparcInstrInfo::reverseBranchCondition(
    SmallVectorImpl<MachineOperand> &Cond) const {
  assert(Cond.size() <= 3);
  SPCC::CondCodes CC = static_cast<SPCC::CondCodes>(Cond[1].getImm());
  Cond[1].setImm(GetOppositeBranchCondition(CC));
  return false;
}

```
- **EN**: Implements logic around `reverseBranchCondition`, `assert`, `CondCodes>`, `setImm`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `reverseBranchCondition`, `assert`, `CondCodes>`, `setImm` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 431-454
```cpp
bool SparcInstrInfo::isBranchOffsetInRange(unsigned BranchOpc,
                                           int64_t Offset) const {
  assert((Offset & 0b11) == 0 && "Malformed branch offset");
  switch (BranchOpc) {
  case SP::BA:
  case SP::BCOND:
  case SP::BCONDA:
  case SP::FBCOND:
  case SP::FBCONDA:
    return isIntN(22, Offset >> 2);

  case SP::BPICC:
  case SP::BPICCA:
  case SP::BPICCNT:
  case SP::BPICCANT:
  case SP::BPXCC:
  case SP::BPXCCA:
  case SP::BPXCCNT:
  case SP::BPXCCANT:
  case SP::BPFCC:
  case SP::BPFCCA:
  case SP::BPFCCNT:
  case SP::BPFCCANT:
  case SP::FBCOND_V9:
```
- **EN**: Implements logic around `isBranchOffsetInRange`, `assert`, `isIntN`; this block uses `switch`-based dispatch; returns target-specific results.
- **CN**: 围绕 `isBranchOffsetInRange`, `assert`, `isIntN` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果。

### Lines 455-467
```cpp
  case SP::FBCONDA_V9:
    return isIntN(BPccDisplacementBits, Offset >> 2);

  case SP::BPR:
  case SP::BPRA:
  case SP::BPRNT:
  case SP::BPRANT:
    return isIntN(BPrDisplacementBits, Offset >> 2);
  }

  llvm_unreachable("Unknown branch instruction!");
}

```
- **EN**: Implements logic around `isIntN`, `llvm_unreachable`; this block returns target-specific results.
- **CN**: 围绕 `isIntN`, `llvm_unreachable` 实现具体逻辑；这一段返回目标相关结果。

### Lines 468-484
```cpp
void SparcInstrInfo::copyPhysReg(MachineBasicBlock &MBB,
                                 MachineBasicBlock::iterator I,
                                 const DebugLoc &DL, Register DestReg,
                                 Register SrcReg, bool KillSrc,
                                 bool RenamableDest, bool RenamableSrc) const {
  unsigned numSubRegs = 0;
  unsigned movOpc     = 0;
  const unsigned *subRegIdx = nullptr;
  bool ExtraG0 = false;

  const unsigned DW_SubRegsIdx[]  = { SP::sub_even, SP::sub_odd };
  const unsigned DFP_FP_SubRegsIdx[]  = { SP::sub_even, SP::sub_odd };
  const unsigned QFP_DFP_SubRegsIdx[] = { SP::sub_even64, SP::sub_odd64 };
  const unsigned QFP_FP_SubRegsIdx[]  = { SP::sub_even, SP::sub_odd,
                                          SP::sub_odd64_then_sub_even,
                                          SP::sub_odd64_then_sub_odd };

```
- **EN**: Implements logic around `copyPhysReg`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `copyPhysReg` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 485-508
```cpp
  if (SP::IntRegsRegClass.contains(DestReg, SrcReg))
    BuildMI(MBB, I, DL, get(SP::ORrr), DestReg).addReg(SP::G0)
      .addReg(SrcReg, getKillRegState(KillSrc));
  else if (SP::IntPairRegClass.contains(DestReg, SrcReg)) {
    subRegIdx  = DW_SubRegsIdx;
    numSubRegs = 2;
    movOpc     = SP::ORrr;
    ExtraG0 = true;
  } else if (SP::FPRegsRegClass.contains(DestReg, SrcReg))
    BuildMI(MBB, I, DL, get(SP::FMOVS), DestReg)
      .addReg(SrcReg, getKillRegState(KillSrc));
  else if (SP::DFPRegsRegClass.contains(DestReg, SrcReg)) {
    if (Subtarget.isV9()) {
      BuildMI(MBB, I, DL, get(SP::FMOVD), DestReg)
        .addReg(SrcReg, getKillRegState(KillSrc));
    } else {
      // Use two FMOVS instructions.
      subRegIdx  = DFP_FP_SubRegsIdx;
      numSubRegs = 2;
      movOpc     = SP::FMOVS;
    }
  } else if (SP::QFPRegsRegClass.contains(DestReg, SrcReg)) {
    if (Subtarget.isV9()) {
      if (Subtarget.hasHardQuad()) {
```
- **EN**: Implements logic around `BuildMI`, `addReg`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `addReg` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 509-532
```cpp
        BuildMI(MBB, I, DL, get(SP::FMOVQ), DestReg)
          .addReg(SrcReg, getKillRegState(KillSrc));
      } else {
        // Use two FMOVD instructions.
        subRegIdx  = QFP_DFP_SubRegsIdx;
        numSubRegs = 2;
        movOpc     = SP::FMOVD;
      }
    } else {
      // Use four FMOVS instructions.
      subRegIdx  = QFP_FP_SubRegsIdx;
      numSubRegs = 4;
      movOpc     = SP::FMOVS;
    }
  } else if (SP::ASRRegsRegClass.contains(DestReg) &&
             SP::IntRegsRegClass.contains(SrcReg)) {
    BuildMI(MBB, I, DL, get(SP::WRASRrr), DestReg)
        .addReg(SP::G0)
        .addReg(SrcReg, getKillRegState(KillSrc));
  } else if (SP::IntRegsRegClass.contains(DestReg) &&
             SP::ASRRegsRegClass.contains(SrcReg)) {
    BuildMI(MBB, I, DL, get(SP::RDASR), DestReg)
        .addReg(SrcReg, getKillRegState(KillSrc));
  } else
```
- **EN**: Implements logic around `BuildMI`, `addReg`, `contains`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `addReg`, `contains` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 533-545
```cpp
    llvm_unreachable("Impossible reg-to-reg copy");

  if (numSubRegs == 0 || subRegIdx == nullptr || movOpc == 0)
    return;

  const TargetRegisterInfo *TRI = &getRegisterInfo();
  MachineInstr *MovMI = nullptr;

  for (unsigned i = 0; i != numSubRegs; ++i) {
    Register Dst = TRI->getSubReg(DestReg, subRegIdx[i]);
    Register Src = TRI->getSubReg(SrcReg, subRegIdx[i]);
    assert(Dst && Src && "Bad sub-register");

```
- **EN**: Implements logic around `llvm_unreachable`, `getRegisterInfo`, `getSubReg`, `assert`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `llvm_unreachable`, `getRegisterInfo`, `getSubReg`, `assert` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 546-557
```cpp
    MachineInstrBuilder MIB = BuildMI(MBB, I, DL, get(movOpc), Dst);
    if (ExtraG0)
      MIB.addReg(SP::G0);
    MIB.addReg(Src);
    MovMI = MIB.getInstr();
  }
  // Add implicit super-register defs and kills to the last MovMI.
  MovMI->addRegisterDefined(DestReg, TRI);
  if (KillSrc)
    MovMI->addRegisterKilled(SrcReg, TRI);
}

```
- **EN**: Implements logic around `BuildMI`, `addReg`, `getInstr`, `addRegisterDefined`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `addReg`, `getInstr`, `addRegisterDefined`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 558-572
```cpp
void SparcInstrInfo::storeRegToStackSlot(MachineBasicBlock &MBB,
                                         MachineBasicBlock::iterator I,
                                         Register SrcReg, bool isKill, int FI,
                                         const TargetRegisterClass *RC,
                                         Register VReg,
                                         MachineInstr::MIFlag Flags) const {
  DebugLoc DL;
  if (I != MBB.end()) DL = I->getDebugLoc();

  MachineFunction *MF = MBB.getParent();
  const MachineFrameInfo &MFI = MF->getFrameInfo();
  MachineMemOperand *MMO = MF->getMachineMemOperand(
      MachinePointerInfo::getFixedStack(*MF, FI), MachineMemOperand::MOStore,
      MFI.getObjectSize(FI), MFI.getObjectAlign(FI));

```
- **EN**: Implements logic around `storeRegToStackSlot`, `getParent`, `getFrameInfo`, `getMachineMemOperand`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `storeRegToStackSlot`, `getParent`, `getFrameInfo`, `getMachineMemOperand`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 573-596
```cpp
  // On the order of operands here: think "[FrameIdx + 0] = SrcReg".
  if (RC == &SP::I64RegsRegClass)
    BuildMI(MBB, I, DL, get(SP::STXri)).addFrameIndex(FI).addImm(0)
      .addReg(SrcReg, getKillRegState(isKill)).addMemOperand(MMO);
  else if (RC == &SP::IntRegsRegClass)
    BuildMI(MBB, I, DL, get(SP::STri)).addFrameIndex(FI).addImm(0)
      .addReg(SrcReg, getKillRegState(isKill)).addMemOperand(MMO);
  else if (RC == &SP::IntPairRegClass)
    BuildMI(MBB, I, DL, get(SP::STDri)).addFrameIndex(FI).addImm(0)
      .addReg(SrcReg, getKillRegState(isKill)).addMemOperand(MMO);
  else if (RC == &SP::FPRegsRegClass)
    BuildMI(MBB, I, DL, get(SP::STFri)).addFrameIndex(FI).addImm(0)
      .addReg(SrcReg,  getKillRegState(isKill)).addMemOperand(MMO);
  else if (SP::DFPRegsRegClass.hasSubClassEq(RC))
    BuildMI(MBB, I, DL, get(SP::STDFri)).addFrameIndex(FI).addImm(0)
      .addReg(SrcReg,  getKillRegState(isKill)).addMemOperand(MMO);
  else if (SP::QFPRegsRegClass.hasSubClassEq(RC))
    // Use STQFri irrespective of its legality. If STQ is not legal, it will be
    // lowered into two STDs in eliminateFrameIndex.
    BuildMI(MBB, I, DL, get(SP::STQFri)).addFrameIndex(FI).addImm(0)
      .addReg(SrcReg,  getKillRegState(isKill)).addMemOperand(MMO);
  else
    llvm_unreachable("Can't store this register to stack slot");
}
```
- **EN**: Implements logic around `BuildMI`, `addReg`, `llvm_unreachable`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `addReg`, `llvm_unreachable` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 597-612
```cpp

void SparcInstrInfo::loadRegFromStackSlot(MachineBasicBlock &MBB,
                                          MachineBasicBlock::iterator I,
                                          Register DestReg, int FI,
                                          const TargetRegisterClass *RC,
                                          Register VReg, unsigned SubReg,
                                          MachineInstr::MIFlag Flags) const {
  DebugLoc DL;
  if (I != MBB.end()) DL = I->getDebugLoc();

  MachineFunction *MF = MBB.getParent();
  const MachineFrameInfo &MFI = MF->getFrameInfo();
  MachineMemOperand *MMO = MF->getMachineMemOperand(
      MachinePointerInfo::getFixedStack(*MF, FI), MachineMemOperand::MOLoad,
      MFI.getObjectSize(FI), MFI.getObjectAlign(FI));

```
- **EN**: Implements logic around `loadRegFromStackSlot`, `getParent`, `getFrameInfo`, `getMachineMemOperand`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `loadRegFromStackSlot`, `getParent`, `getFrameInfo`, `getMachineMemOperand`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 613-636
```cpp
  if (RC == &SP::I64RegsRegClass)
    BuildMI(MBB, I, DL, get(SP::LDXri), DestReg).addFrameIndex(FI).addImm(0)
      .addMemOperand(MMO);
  else if (RC == &SP::IntRegsRegClass)
    BuildMI(MBB, I, DL, get(SP::LDri), DestReg).addFrameIndex(FI).addImm(0)
      .addMemOperand(MMO);
  else if (RC == &SP::IntPairRegClass)
    BuildMI(MBB, I, DL, get(SP::LDDri), DestReg).addFrameIndex(FI).addImm(0)
      .addMemOperand(MMO);
  else if (RC == &SP::FPRegsRegClass)
    BuildMI(MBB, I, DL, get(SP::LDFri), DestReg).addFrameIndex(FI).addImm(0)
      .addMemOperand(MMO);
  else if (SP::DFPRegsRegClass.hasSubClassEq(RC))
    BuildMI(MBB, I, DL, get(SP::LDDFri), DestReg).addFrameIndex(FI).addImm(0)
      .addMemOperand(MMO);
  else if (SP::QFPRegsRegClass.hasSubClassEq(RC))
    // Use LDQFri irrespective of its legality. If LDQ is not legal, it will be
    // lowered into two LDDs in eliminateFrameIndex.
    BuildMI(MBB, I, DL, get(SP::LDQFri), DestReg).addFrameIndex(FI).addImm(0)
      .addMemOperand(MMO);
  else
    llvm_unreachable("Can't load this register from stack slot");
}

```
- **EN**: Implements logic around `BuildMI`, `addMemOperand`, `llvm_unreachable`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `addMemOperand`, `llvm_unreachable` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 637-651
```cpp
Register SparcInstrInfo::getGlobalBaseReg(MachineFunction *MF) const {
  SparcMachineFunctionInfo *SparcFI = MF->getInfo<SparcMachineFunctionInfo>();
  Register GlobalBaseReg = SparcFI->getGlobalBaseReg();
  if (GlobalBaseReg)
    return GlobalBaseReg;

  // Insert the set of GlobalBaseReg into the first MBB of the function
  MachineBasicBlock &FirstMBB = MF->front();
  MachineBasicBlock::iterator MBBI = FirstMBB.begin();
  MachineRegisterInfo &RegInfo = MF->getRegInfo();

  const TargetRegisterClass *PtrRC =
    Subtarget.is64Bit() ? &SP::I64RegsRegClass : &SP::IntRegsRegClass;
  GlobalBaseReg = RegInfo.createVirtualRegister(PtrRC);

```
- **EN**: Implements logic around `getGlobalBaseReg`, `getInfo<SparcMachineFunctionInfo>`, `front`, `begin`, ...; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getGlobalBaseReg`, `getInfo<SparcMachineFunctionInfo>`, `front`, `begin`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 652-667
```cpp
  DebugLoc dl;

  BuildMI(FirstMBB, MBBI, dl, get(SP::GETPCX), GlobalBaseReg);
  SparcFI->setGlobalBaseReg(GlobalBaseReg);
  return GlobalBaseReg;
}

unsigned SparcInstrInfo::getInstSizeInBytes(const MachineInstr &MI) const {
  unsigned Opcode = MI.getOpcode();

  if (MI.isInlineAsm()) {
    const MachineFunction *MF = MI.getParent()->getParent();
    const char *AsmStr = MI.getOperand(0).getSymbolName();
    return getInlineAsmLength(AsmStr, MF->getTarget().getMCAsmInfo());
  }

```
- **EN**: Implements logic around `BuildMI`, `setGlobalBaseReg`, `getInstSizeInBytes`, `getOpcode`, ...; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `setGlobalBaseReg`, `getInstSizeInBytes`, `getOpcode`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 668-686
```cpp
  if (Opcode == TargetOpcode::BUNDLE)
    return getInstBundleSize(MI);

  if (MI.getOpcode() == SP::GETPCX) {
    const TargetMachine &TM = MI.getParent()->getParent()->getTarget();
    if (TM.isPositionIndependent())
      return 16;
    switch (TM.getCodeModel()) {
    default:
      llvm_unreachable("Unsupported absolute code model");
    case CodeModel::Small:
      return 8;
    case CodeModel::Medium:
      return 16;
    case CodeModel::Large:
      return 24;
    }
  }

```
- **EN**: Implements logic around `getInstBundleSize`, `getParent`, `llvm_unreachable`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getInstBundleSize`, `getParent`, `llvm_unreachable` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果。

### Lines 687-710
```cpp
  // If the instruction has a delay slot, be conservative and also include
  // it for sizing purposes. This is done so that the BranchRelaxation pass
  // will not mistakenly mark out-of-range branches as in-range.
  if (MI.hasDelaySlot())
    return get(Opcode).getSize() * 2;
  return get(Opcode).getSize();
}

bool SparcInstrInfo::analyzeCompare(const MachineInstr &MI, Register &SrcReg,
                                    Register &SrcReg2, int64_t &CmpMask,
                                    int64_t &CmpValue) const {
  Register DstReg;
  switch (MI.getOpcode()) {
  default:
    break;
  case SP::SUBCCri:
    DstReg = MI.getOperand(0).getReg();
    SrcReg = MI.getOperand(1).getReg();
    SrcReg2 = 0;
    CmpMask = ~0;
    CmpValue = MI.getOperand(2).getImm();
    return DstReg == SP::G0 && CmpValue == 0;
  case SP::SUBCCrr:
    DstReg = MI.getOperand(0).getReg();
```
- **EN**: Implements logic around `get`, `analyzeCompare`, `getOperand`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `get`, `analyzeCompare`, `getOperand` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 711-724
```cpp
    SrcReg = MI.getOperand(1).getReg();
    SrcReg2 = MI.getOperand(2).getReg();
    CmpMask = ~0;
    CmpValue = 0;
    return DstReg == SP::G0 && SrcReg2 == SP::G0;
  }

  return false;
}

bool SparcInstrInfo::optimizeCompareInstr(
    MachineInstr &CmpInstr, Register SrcReg, Register SrcReg2, int64_t CmpMask,
    int64_t CmpValue, const MachineRegisterInfo *MRI) const {

```
- **EN**: Implements logic around `getOperand`, `optimizeCompareInstr`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getOperand`, `optimizeCompareInstr` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 725-748
```cpp
  // Get the unique definition of SrcReg.
  MachineInstr *MI = MRI->getUniqueVRegDef(SrcReg);
  if (!MI)
    return false;

  // Only optimize if defining and comparing instruction in same block.
  if (MI->getParent() != CmpInstr.getParent())
    return false;

  unsigned NewOpcode;
  switch (MI->getOpcode()) {
  case SP::ANDNrr:
    NewOpcode = SP::ANDNCCrr;
    break;
  case SP::ANDNri:
    NewOpcode = SP::ANDNCCri;
    break;
  case SP::ANDrr:
    NewOpcode = SP::ANDCCrr;
    break;
  case SP::ANDri:
    NewOpcode = SP::ANDCCri;
    break;
  case SP::ORrr:
```
- **EN**: Implements logic around `getUniqueVRegDef`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getUniqueVRegDef` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 749-772
```cpp
    NewOpcode = SP::ORCCrr;
    break;
  case SP::ORri:
    NewOpcode = SP::ORCCri;
    break;
  case SP::ORNCCrr:
    NewOpcode = SP::ORNCCrr;
    break;
  case SP::ORNri:
    NewOpcode = SP::ORNCCri;
    break;
  case SP::XORrr:
    NewOpcode = SP::XORCCrr;
    break;
  case SP::XNORri:
    NewOpcode = SP::XNORCCri;
    break;
  case SP::XNORrr:
    NewOpcode = SP::XNORCCrr;
    break;
  case SP::ADDrr:
    NewOpcode = SP::ADDCCrr;
    break;
  case SP::ADDri:
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 773-784
```cpp
    NewOpcode = SP::ADDCCri;
    break;
  case SP::SUBrr:
    NewOpcode = SP::SUBCCrr;
    break;
  case SP::SUBri:
    NewOpcode = SP::SUBCCri;
    break;
  default:
    return false;
  }

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 785-796
```cpp
  bool IsICCModified = false;
  MachineBasicBlock::iterator I = MI;
  MachineBasicBlock::iterator C = CmpInstr;
  MachineBasicBlock::iterator E = CmpInstr.getParent()->end();
  const TargetRegisterInfo *TRI = &getRegisterInfo();

  // If ICC is used or modified between MI and CmpInstr we cannot optimize.
  while (++I != C) {
    if (I->modifiesRegister(SP::ICC, TRI) || I->readsRegister(SP::ICC, TRI))
      return false;
  }

```
- **EN**: Implements logic around `getParent`, `getRegisterInfo`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getParent`, `getRegisterInfo` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 797-816
```cpp
  while (++I != E) {
    // Only allow conditionals on equality.
    if (I->readsRegister(SP::ICC, TRI)) {
      bool IsICCBranch = I->getOpcode() == SP::BCOND ||
                         I->getOpcode() == SP::BPICC ||
                         I->getOpcode() == SP::BPXCC;
      bool IsICCMove =
          I->getOpcode() == SP::MOVICCrr || I->getOpcode() == SP::MOVICCri ||
          I->getOpcode() == SP::MOVXCCrr || I->getOpcode() == SP::MOVXCCri;
      bool IsICCConditional = IsICCBranch || IsICCMove;
      if (!IsICCConditional ||
          (I->getOperand(IsICCBranch ? 1 : 3).getImm() != SPCC::ICC_E &&
           I->getOperand(IsICCBranch ? 1 : 3).getImm() != SPCC::ICC_NE))
        return false;
    } else if (I->modifiesRegister(SP::ICC, TRI)) {
      IsICCModified = true;
      break;
    }
  }

```
- **EN**: Implements logic around `getOpcode`, `getOperand`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getOpcode`, `getOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 817-830
```cpp
  if (!IsICCModified) {
    MachineBasicBlock *MBB = CmpInstr.getParent();
    if (any_of(MBB->successors(),
               [](MachineBasicBlock *Succ) { return Succ->isLiveIn(SP::ICC); }))
      return false;
  }

  if (MRI->hasOneNonDBGUse(SrcReg))
    MI->getOperand(0).setReg(SP::G0);

  MI->setDesc(get(NewOpcode));
  MI->addRegisterDefined(SP::ICC);
  CmpInstr.eraseFromParent();

```
- **EN**: Implements logic around `getParent`, `isLiveIn`, `getOperand`, `setDesc`, ...; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getParent`, `isLiveIn`, `getOperand`, `setDesc`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 831-850
```cpp
  return true;
}

bool SparcInstrInfo::expandPostRAPseudo(MachineInstr &MI) const {
  switch (MI.getOpcode()) {
  case TargetOpcode::LOAD_STACK_GUARD: {
    assert(Subtarget.getTargetTriple().isOSLinux() &&
           "Only Linux target is expected to contain LOAD_STACK_GUARD");
    // offsetof(tcbhead_t, stack_guard) from sysdeps/sparc/nptl/tls.h in glibc.
    const int64_t Offset = Subtarget.is64Bit() ? 0x28 : 0x14;
    MI.setDesc(get(Subtarget.is64Bit() ? SP::LDXri : SP::LDri));
    MachineInstrBuilder(*MI.getParent()->getParent(), MI)
        .addReg(SP::G7)
        .addImm(Offset);
    return true;
  }
  case SP::V8BAR: {
    assert(!Subtarget.isV9() &&
           "V8BAR should not be emitted on V9 processors!");

```
- **EN**: Implements logic around `expandPostRAPseudo`, `assert`, `is64Bit`, `setDesc`, ...; this block uses `switch`-based dispatch; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `expandPostRAPseudo`, `assert`, `is64Bit`, `setDesc`, ... 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 851-866
```cpp
    // Emit stbar; ldstub [%sp-1], %g0
    // The sequence acts as a full barrier on V8 systems.
    MachineBasicBlock &MBB = *MI.getParent();
    MachineInstr &InstSTBAR =
        *BuildMI(MBB, MI, MI.getDebugLoc(), get(SP::STBAR));
    MachineInstr &InstLDSTUB =
        *BuildMI(MBB, MI, MI.getDebugLoc(), get(SP::LDSTUBri), SP::G0)
             .addReg(SP::O6)
             .addImm(-1);
    MIBundleBuilder(MBB, InstSTBAR, InstLDSTUB);
    MBB.erase(MI);
    return true;
  }
  }
  return false;
}
```
- **EN**: Implements logic around `getParent`, `BuildMI`, `addReg`, `addImm`, ...; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getParent`, `BuildMI`, `addReg`, `addImm`, ... 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

## Key Concepts / 关键概念

- **Instruction semantics / 指令语义**:
  - **EN**: Encodes instruction behavior and helper routines
  - **CN**: 描述指令行为与辅助例程
- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `SparcInstrInfo.h`, `Sparc.h`, `SparcMachineFunctionInfo.h`, `SparcSubtarget.h`, `llvm/ADT/SmallVector.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineMemOperand.h`, `llvm/CodeGen/MachineRegisterInfo.h`, `llvm/Support/ErrorHandling.h`, `SparcGenInstrInfo.inc`
- **LLVM subsystems / LLVM 子系统**: CodeGen, Support
- **Generated macros / 生成宏**: `GET_INSTRINFO_CTOR_DTOR`
