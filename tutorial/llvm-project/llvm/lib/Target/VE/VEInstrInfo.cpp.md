# VEInstrInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/VE/VEInstrInfo.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements target-specific instruction information, instruction helpers, and scheduling-related behavior.
  - **CN**: 实现目标相关的指令信息、指令辅助逻辑以及与调度相关的行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-26
```cpp
//===-- VEInstrInfo.cpp - VE Instruction Information ----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the VE implementation of the TargetInstrInfo class.
//
//===----------------------------------------------------------------------===//

#include "VEInstrInfo.h"
#include "VE.h"
#include "VEMachineFunctionInfo.h"
#include "VESubtarget.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineMemOperand.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/ErrorHandling.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `VEInstrInfo.h`, `VE.h`, `VEMachineFunctionInfo.h`, `VESubtarget.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `VEInstrInfo.h`, `VE.h`, `VEMachineFunctionInfo.h`, `VESubtarget.h`。

### Lines 27-58
```cpp
#define DEBUG_TYPE "ve-instr-info"

using namespace llvm;

#define GET_INSTRINFO_CTOR_DTOR
#include "VEGenInstrInfo.inc"

// Pin the vtable to this file.
void VEInstrInfo::anchor() {}

VEInstrInfo::VEInstrInfo(const VESubtarget &ST)
    : VEGenInstrInfo(ST, RI, VE::ADJCALLSTACKDOWN, VE::ADJCALLSTACKUP), RI() {}

static bool IsIntegerCC(unsigned CC) { return (CC < VECC::CC_AF); }

static VECC::CondCode GetOppositeBranchCondition(VECC::CondCode CC) {
  switch (CC) {
  case VECC::CC_IG:
    return VECC::CC_ILE;
  case VECC::CC_IL:
    return VECC::CC_IGE;
  case VECC::CC_INE:
    return VECC::CC_IEQ;
  case VECC::CC_IEQ:
    return VECC::CC_INE;
  case VECC::CC_IGE:
    return VECC::CC_IL;
  case VECC::CC_ILE:
    return VECC::CC_IG;
  case VECC::CC_AF:
    return VECC::CC_AT;
  case VECC::CC_G:
```
- **EN**: Pulls in the headers needed for this implementation, including `VEGenInstrInfo.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `VEGenInstrInfo.inc`。

### Lines 59-90
```cpp
    return VECC::CC_LENAN;
  case VECC::CC_L:
    return VECC::CC_GENAN;
  case VECC::CC_NE:
    return VECC::CC_EQNAN;
  case VECC::CC_EQ:
    return VECC::CC_NENAN;
  case VECC::CC_GE:
    return VECC::CC_LNAN;
  case VECC::CC_LE:
    return VECC::CC_GNAN;
  case VECC::CC_NUM:
    return VECC::CC_NAN;
  case VECC::CC_NAN:
    return VECC::CC_NUM;
  case VECC::CC_GNAN:
    return VECC::CC_LE;
  case VECC::CC_LNAN:
    return VECC::CC_GE;
  case VECC::CC_NENAN:
    return VECC::CC_EQ;
  case VECC::CC_EQNAN:
    return VECC::CC_NE;
  case VECC::CC_GENAN:
    return VECC::CC_L;
  case VECC::CC_LENAN:
    return VECC::CC_G;
  case VECC::CC_AT:
    return VECC::CC_AF;
  case VECC::UNKNOWN:
    return VECC::UNKNOWN;
  }
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 91-108
```cpp
  llvm_unreachable("Invalid cond code");
}

// Treat a branch relative long always instruction as unconditional branch.
// For example, br.l.t and br.l.
static bool isUncondBranchOpcode(int Opc) {
  using namespace llvm::VE;

#define BRKIND(NAME) (Opc == NAME##a || Opc == NAME##a_nt || Opc == NAME##a_t)
  // VE has other branch relative always instructions for word/double/float,
  // but we use only long branches in our lower.  So, check it here.
  assert(!BRKIND(BRCFW) && !BRKIND(BRCFD) && !BRKIND(BRCFS) &&
         "Branch relative word/double/float always instructions should not be "
         "used!");
  return BRKIND(BRCFL);
#undef BRKIND
}

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 109-125
```cpp
// Treat branch relative conditional as conditional branch instructions.
// For example, brgt.l.t and brle.s.nt.
static bool isCondBranchOpcode(int Opc) {
  using namespace llvm::VE;

#define BRKIND(NAME)                                                           \
  (Opc == NAME##rr || Opc == NAME##rr_nt || Opc == NAME##rr_t ||               \
   Opc == NAME##ir || Opc == NAME##ir_nt || Opc == NAME##ir_t)
  return BRKIND(BRCFL) || BRKIND(BRCFW) || BRKIND(BRCFD) || BRKIND(BRCFS);
#undef BRKIND
}

// Treat branch long always instructions as indirect branch.
// For example, b.l.t and b.l.
static bool isIndirectBranchOpcode(int Opc) {
  using namespace llvm::VE;

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 126-143
```cpp
#define BRKIND(NAME)                                                           \
  (Opc == NAME##ari || Opc == NAME##ari_nt || Opc == NAME##ari_t)
  // VE has other branch always instructions for word/double/float, but
  // we use only long branches in our lower.  So, check it here.
  assert(!BRKIND(BCFW) && !BRKIND(BCFD) && !BRKIND(BCFS) &&
         "Branch word/double/float always instructions should not be used!");
  return BRKIND(BCFL);
#undef BRKIND
}

static void parseCondBranch(MachineInstr *LastInst, MachineBasicBlock *&Target,
                            SmallVectorImpl<MachineOperand> &Cond) {
  Cond.push_back(MachineOperand::CreateImm(LastInst->getOperand(0).getImm()));
  Cond.push_back(LastInst->getOperand(1));
  Cond.push_back(LastInst->getOperand(2));
  Target = LastInst->getOperand(3).getMBB();
}

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 144-172
```cpp
bool VEInstrInfo::analyzeBranch(MachineBasicBlock &MBB, MachineBasicBlock *&TBB,
                                MachineBasicBlock *&FBB,
                                SmallVectorImpl<MachineOperand> &Cond,
                                bool AllowModify) const {
  MachineBasicBlock::iterator I = MBB.getLastNonDebugInstr();
  if (I == MBB.end())
    return false;

  if (!isUnpredicatedTerminator(*I))
    return false;

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
- **EN**: Implements logic around `analyzeBranch`, `getLastNonDebugInstr`, `getOpcode`, `getOperand`, ...; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `analyzeBranch`, `getLastNonDebugInstr`, `getOpcode`, `getOperand`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 173-193
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
      }
      SecondLastInst = &*I;
      SecondLastOpc = SecondLastInst->getOpcode();
    }
  }

```
- **EN**: Implements logic around `getOpcode`, `eraseFromParent`, `getOperand`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getOpcode`, `eraseFromParent`, `getOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 194-211
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

### Lines 212-241
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

unsigned VEInstrInfo::insertBranch(MachineBasicBlock &MBB,
                                   MachineBasicBlock *TBB,
                                   MachineBasicBlock *FBB,
                                   ArrayRef<MachineOperand> Cond,
                                   const DebugLoc &DL, int *BytesAdded) const {
  assert(TBB && "insertBranch must not be told to insert a fallthrough");
  assert((Cond.size() == 3 || Cond.size() == 0) &&
         "VE branch conditions should have three component!");
  assert(!BytesAdded && "code size not handled");
  if (Cond.empty()) {
    // Uncondition branch
    assert(!FBB && "Unconditional branch with multiple successors!");
    BuildMI(&MBB, DL, get(VE::BRCFLa_t))
        .addMBB(TBB);
    return 1;
  }

```
- **EN**: Implements logic around `eraseFromParent`, `insertBranch`, `assert`, `BuildMI`, ...; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `eraseFromParent`, `insertBranch`, `assert`, `BuildMI`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 242-273
```cpp
  // Conditional branch
  //   (BRCFir CC sy sz addr)
  assert(Cond[0].isImm() && Cond[2].isReg() && "not implemented");

  unsigned opc[2];
  const TargetRegisterInfo *TRI = &getRegisterInfo();
  MachineFunction *MF = MBB.getParent();
  const MachineRegisterInfo &MRI = MF->getRegInfo();
  Register Reg = Cond[2].getReg();
  if (IsIntegerCC(Cond[0].getImm())) {
    if (TRI->getRegSizeInBits(Reg, MRI) == 32) {
      opc[0] = VE::BRCFWir;
      opc[1] = VE::BRCFWrr;
    } else {
      opc[0] = VE::BRCFLir;
      opc[1] = VE::BRCFLrr;
    }
  } else {
    if (TRI->getRegSizeInBits(Reg, MRI) == 32) {
      opc[0] = VE::BRCFSir;
      opc[1] = VE::BRCFSrr;
    } else {
      opc[0] = VE::BRCFDir;
      opc[1] = VE::BRCFDrr;
    }
  }
  if (Cond[1].isImm()) {
      BuildMI(&MBB, DL, get(opc[0]))
          .add(Cond[0]) // condition code
          .add(Cond[1]) // lhs
          .add(Cond[2]) // rhs
          .addMBB(TBB);
```
- **EN**: Implements logic around `assert`, `getRegisterInfo`, `getParent`, `getRegInfo`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `assert`, `getRegisterInfo`, `getParent`, `getRegInfo`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 274-289
```cpp
  } else {
      BuildMI(&MBB, DL, get(opc[1]))
          .add(Cond[0])
          .add(Cond[1])
          .add(Cond[2])
          .addMBB(TBB);
  }

  if (!FBB)
    return 1;

  BuildMI(&MBB, DL, get(VE::BRCFLa_t))
      .addMBB(FBB);
  return 2;
}

```
- **EN**: Implements logic around `BuildMI`, `add`, `addMBB`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `add`, `addMBB` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 290-305
```cpp
unsigned VEInstrInfo::removeBranch(MachineBasicBlock &MBB,
                                   int *BytesRemoved) const {
  assert(!BytesRemoved && "code size not handled");

  MachineBasicBlock::iterator I = MBB.end();
  unsigned Count = 0;
  while (I != MBB.begin()) {
    --I;

    if (I->isDebugValue())
      continue;

    if (!isUncondBranchOpcode(I->getOpcode()) &&
        !isCondBranchOpcode(I->getOpcode()))
      break; // Not a branch

```
- **EN**: Implements logic around `removeBranch`, `assert`, `end`, `isCondBranchOpcode`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `removeBranch`, `assert`, `end`, `isCondBranchOpcode` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 306-324
```cpp
    I->eraseFromParent();
    I = MBB.end();
    ++Count;
  }
  return Count;
}

bool VEInstrInfo::reverseBranchCondition(
    SmallVectorImpl<MachineOperand> &Cond) const {
  VECC::CondCode CC = static_cast<VECC::CondCode>(Cond[0].getImm());
  Cond[0].setImm(GetOppositeBranchCondition(CC));
  return false;
}

static bool IsAliasOfSX(Register Reg) {
  return VE::I32RegClass.contains(Reg) || VE::I64RegClass.contains(Reg) ||
         VE::F32RegClass.contains(Reg);
}

```
- **EN**: Implements logic around `eraseFromParent`, `end`, `reverseBranchCondition`, `CondCode>`, ...; this block returns target-specific results.
- **CN**: 围绕 `eraseFromParent`, `end`, `reverseBranchCondition`, `CondCode>`, ... 实现具体逻辑；这一段返回目标相关结果。

### Lines 325-356
```cpp
static void copyPhysSubRegs(MachineBasicBlock &MBB,
                            MachineBasicBlock::iterator I, const DebugLoc &DL,
                            MCRegister DestReg, MCRegister SrcReg, bool KillSrc,
                            const MCInstrDesc &MCID, unsigned int NumSubRegs,
                            const unsigned *SubRegIdx,
                            const TargetRegisterInfo *TRI) {
  MachineInstr *MovMI = nullptr;

  for (unsigned Idx = 0; Idx != NumSubRegs; ++Idx) {
    Register SubDest = TRI->getSubReg(DestReg, SubRegIdx[Idx]);
    Register SubSrc = TRI->getSubReg(SrcReg, SubRegIdx[Idx]);
    assert(SubDest && SubSrc && "Bad sub-register");

    if (MCID.getOpcode() == VE::ORri) {
      // generate "ORri, dest, src, 0" instruction.
      MachineInstrBuilder MIB =
          BuildMI(MBB, I, DL, MCID, SubDest).addReg(SubSrc).addImm(0);
      MovMI = MIB.getInstr();
    } else if (MCID.getOpcode() == VE::ANDMmm) {
      // generate "ANDM, dest, vm0, src" instruction.
      MachineInstrBuilder MIB =
          BuildMI(MBB, I, DL, MCID, SubDest).addReg(VE::VM0).addReg(SubSrc);
      MovMI = MIB.getInstr();
    } else {
      llvm_unreachable("Unexpected reg-to-reg copy instruction");
    }
  }
  // Add implicit super-register defs and kills to the last MovMI.
  MovMI->addRegisterDefined(DestReg, TRI);
  if (KillSrc)
    MovMI->addRegisterKilled(SrcReg, TRI, true);
}
```
- **EN**: Implements logic around `copyPhysSubRegs`, `getSubReg`, `assert`, `BuildMI`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer; works at the MC layer.
- **CN**: 围绕 `copyPhysSubRegs`, `getSubReg`, `assert`, `BuildMI`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层，工作在 MC 层。

### Lines 357-388
```cpp

void VEInstrInfo::copyPhysReg(MachineBasicBlock &MBB,
                              MachineBasicBlock::iterator I, const DebugLoc &DL,
                              Register DestReg, Register SrcReg, bool KillSrc,
                              bool RenamableDest, bool RenamableSrc) const {

  if (IsAliasOfSX(SrcReg) && IsAliasOfSX(DestReg)) {
    BuildMI(MBB, I, DL, get(VE::ORri), DestReg)
        .addReg(SrcReg, getKillRegState(KillSrc))
        .addImm(0);
  } else if (VE::V64RegClass.contains(DestReg, SrcReg)) {
    // Generate following instructions
    //   %sw16 = LEA32zii 256
    //   VORmvl %dest, (0)1, %src, %sw16
    // TODO: reuse a register if vl is already assigned to a register
    // FIXME: it would be better to scavenge a register here instead of
    // reserving SX16 all of the time.
    const TargetRegisterInfo *TRI = &getRegisterInfo();
    Register TmpReg = VE::SX16;
    Register SubTmp = TRI->getSubReg(TmpReg, VE::sub_i32);
    BuildMI(MBB, I, DL, get(VE::LEAzii), TmpReg)
        .addImm(0)
        .addImm(0)
        .addImm(256);
    MachineInstrBuilder MIB = BuildMI(MBB, I, DL, get(VE::VORmvl), DestReg)
                                  .addImm(M1(0)) // Represent (0)1.
                                  .addReg(SrcReg, getKillRegState(KillSrc))
                                  .addReg(SubTmp, getKillRegState(true));
    MIB.getInstr()->addRegisterKilled(TmpReg, TRI, true);
  } else if (VE::VMRegClass.contains(DestReg, SrcReg)) {
    BuildMI(MBB, I, DL, get(VE::ANDMmm), DestReg)
        .addReg(VE::VM0)
```
- **EN**: Implements logic around `copyPhysReg`, `BuildMI`, `addReg`, `addImm`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `copyPhysReg`, `BuildMI`, `addReg`, `addImm`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 389-409
```cpp
        .addReg(SrcReg, getKillRegState(KillSrc));
  } else if (VE::VM512RegClass.contains(DestReg, SrcReg)) {
    // Use two instructions.
    const unsigned SubRegIdx[] = {VE::sub_vm_even, VE::sub_vm_odd};
    unsigned int NumSubRegs = 2;
    copyPhysSubRegs(MBB, I, DL, DestReg, SrcReg, KillSrc, get(VE::ANDMmm),
                    NumSubRegs, SubRegIdx, &getRegisterInfo());
  } else if (VE::F128RegClass.contains(DestReg, SrcReg)) {
    // Use two instructions.
    const unsigned SubRegIdx[] = {VE::sub_even, VE::sub_odd};
    unsigned int NumSubRegs = 2;
    copyPhysSubRegs(MBB, I, DL, DestReg, SrcReg, KillSrc, get(VE::ORri),
                    NumSubRegs, SubRegIdx, &getRegisterInfo());
  } else {
    const TargetRegisterInfo *TRI = &getRegisterInfo();
    dbgs() << "Impossible reg-to-reg copy from " << printReg(SrcReg, TRI)
           << " to " << printReg(DestReg, TRI) << "\n";
    llvm_unreachable("Impossible reg-to-reg copy");
  }
}

```
- **EN**: Implements logic around `addReg`, `copyPhysSubRegs`, `getRegisterInfo`, `dbgs`, ...; this block applies conditional target rules.
- **CN**: 围绕 `addReg`, `copyPhysSubRegs`, `getRegisterInfo`, `dbgs`, ... 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 410-433
```cpp
/// isLoadFromStackSlot - If the specified machine instruction is a direct
/// load from a stack slot, return the virtual or physical register number of
/// the destination along with the FrameIndex of the loaded stack slot.  If
/// not, return 0.  This predicate must return 0 if the instruction has
/// any side effects other than loading from the stack slot.
Register VEInstrInfo::isLoadFromStackSlot(const MachineInstr &MI,
                                          int &FrameIndex) const {
  if (MI.getOpcode() == VE::LDrii ||    // I64
      MI.getOpcode() == VE::LDLSXrii || // I32
      MI.getOpcode() == VE::LDUrii ||   // F32
      MI.getOpcode() == VE::LDQrii ||   // F128 (pseudo)
      MI.getOpcode() == VE::LDVMrii ||  // VM (pseudo)
      MI.getOpcode() == VE::LDVM512rii  // VM512 (pseudo)
  ) {
    if (MI.getOperand(1).isFI() && MI.getOperand(2).isImm() &&
        MI.getOperand(2).getImm() == 0 && MI.getOperand(3).isImm() &&
        MI.getOperand(3).getImm() == 0) {
      FrameIndex = MI.getOperand(1).getIndex();
      return MI.getOperand(0).getReg();
    }
  }
  return 0;
}

```
- **EN**: Implements logic around `isLoadFromStackSlot`, `getOpcode`, `getOperand`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `isLoadFromStackSlot`, `getOpcode`, `getOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 434-457
```cpp
/// isStoreToStackSlot - If the specified machine instruction is a direct
/// store to a stack slot, return the virtual or physical register number of
/// the source reg along with the FrameIndex of the loaded stack slot.  If
/// not, return 0.  This predicate must return 0 if the instruction has
/// any side effects other than storing to the stack slot.
Register VEInstrInfo::isStoreToStackSlot(const MachineInstr &MI,
                                         int &FrameIndex) const {
  if (MI.getOpcode() == VE::STrii ||   // I64
      MI.getOpcode() == VE::STLrii ||  // I32
      MI.getOpcode() == VE::STUrii ||  // F32
      MI.getOpcode() == VE::STQrii ||  // F128 (pseudo)
      MI.getOpcode() == VE::STVMrii || // VM (pseudo)
      MI.getOpcode() == VE::STVM512rii // VM512 (pseudo)
  ) {
    if (MI.getOperand(0).isFI() && MI.getOperand(1).isImm() &&
        MI.getOperand(1).getImm() == 0 && MI.getOperand(2).isImm() &&
        MI.getOperand(2).getImm() == 0) {
      FrameIndex = MI.getOperand(0).getIndex();
      return MI.getOperand(3).getReg();
    }
  }
  return 0;
}

```
- **EN**: Implements logic around `isStoreToStackSlot`, `getOpcode`, `getOperand`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `isStoreToStackSlot`, `getOpcode`, `getOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 458-473
```cpp
void VEInstrInfo::storeRegToStackSlot(MachineBasicBlock &MBB,
                                      MachineBasicBlock::iterator I,
                                      Register SrcReg, bool isKill, int FI,
                                      const TargetRegisterClass *RC,
                                      Register VReg,
                                      MachineInstr::MIFlag Flags) const {
  DebugLoc DL;
  if (I != MBB.end())
    DL = I->getDebugLoc();

  MachineFunction *MF = MBB.getParent();
  const MachineFrameInfo &MFI = MF->getFrameInfo();
  MachineMemOperand *MMO = MF->getMachineMemOperand(
      MachinePointerInfo::getFixedStack(*MF, FI), MachineMemOperand::MOStore,
      MFI.getObjectSize(FI), MFI.getObjectAlign(FI));

```
- **EN**: Implements logic around `storeRegToStackSlot`, `getDebugLoc`, `getParent`, `getFrameInfo`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `storeRegToStackSlot`, `getDebugLoc`, `getParent`, `getFrameInfo`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 474-505
```cpp
  // On the order of operands here: think "[FrameIdx + 0] = SrcReg".
  if (RC == &VE::I64RegClass) {
    BuildMI(MBB, I, DL, get(VE::STrii))
        .addFrameIndex(FI)
        .addImm(0)
        .addImm(0)
        .addReg(SrcReg, getKillRegState(isKill))
        .addMemOperand(MMO);
  } else if (RC == &VE::I32RegClass) {
    BuildMI(MBB, I, DL, get(VE::STLrii))
        .addFrameIndex(FI)
        .addImm(0)
        .addImm(0)
        .addReg(SrcReg, getKillRegState(isKill))
        .addMemOperand(MMO);
  } else if (RC == &VE::F32RegClass) {
    BuildMI(MBB, I, DL, get(VE::STUrii))
        .addFrameIndex(FI)
        .addImm(0)
        .addImm(0)
        .addReg(SrcReg, getKillRegState(isKill))
        .addMemOperand(MMO);
  } else if (VE::F128RegClass.hasSubClassEq(RC)) {
    BuildMI(MBB, I, DL, get(VE::STQrii))
        .addFrameIndex(FI)
        .addImm(0)
        .addImm(0)
        .addReg(SrcReg, getKillRegState(isKill))
        .addMemOperand(MMO);
  } else if (RC == &VE::VMRegClass) {
    BuildMI(MBB, I, DL, get(VE::STVMrii))
        .addFrameIndex(FI)
```
- **EN**: Implements logic around `BuildMI`, `addFrameIndex`, `addImm`, `addReg`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `addFrameIndex`, `addImm`, `addReg`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 506-530
```cpp
        .addImm(0)
        .addImm(0)
        .addReg(SrcReg, getKillRegState(isKill))
        .addMemOperand(MMO);
  } else if (VE::VM512RegClass.hasSubClassEq(RC)) {
    BuildMI(MBB, I, DL, get(VE::STVM512rii))
        .addFrameIndex(FI)
        .addImm(0)
        .addImm(0)
        .addReg(SrcReg, getKillRegState(isKill))
        .addMemOperand(MMO);
  } else
    report_fatal_error("Can't store this register to stack slot");
}

void VEInstrInfo::loadRegFromStackSlot(MachineBasicBlock &MBB,
                                       MachineBasicBlock::iterator I,
                                       Register DestReg, int FI,
                                       const TargetRegisterClass *RC,
                                       Register VReg, unsigned SubReg,
                                       MachineInstr::MIFlag Flags) const {
  DebugLoc DL;
  if (I != MBB.end())
    DL = I->getDebugLoc();

```
- **EN**: Implements logic around `addImm`, `addReg`, `addMemOperand`, `BuildMI`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `addImm`, `addReg`, `addMemOperand`, `BuildMI`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 531-562
```cpp
  MachineFunction *MF = MBB.getParent();
  const MachineFrameInfo &MFI = MF->getFrameInfo();
  MachineMemOperand *MMO = MF->getMachineMemOperand(
      MachinePointerInfo::getFixedStack(*MF, FI), MachineMemOperand::MOLoad,
      MFI.getObjectSize(FI), MFI.getObjectAlign(FI));

  if (RC == &VE::I64RegClass) {
    BuildMI(MBB, I, DL, get(VE::LDrii), DestReg)
        .addFrameIndex(FI)
        .addImm(0)
        .addImm(0)
        .addMemOperand(MMO);
  } else if (RC == &VE::I32RegClass) {
    BuildMI(MBB, I, DL, get(VE::LDLSXrii), DestReg)
        .addFrameIndex(FI)
        .addImm(0)
        .addImm(0)
        .addMemOperand(MMO);
  } else if (RC == &VE::F32RegClass) {
    BuildMI(MBB, I, DL, get(VE::LDUrii), DestReg)
        .addFrameIndex(FI)
        .addImm(0)
        .addImm(0)
        .addMemOperand(MMO);
  } else if (VE::F128RegClass.hasSubClassEq(RC)) {
    BuildMI(MBB, I, DL, get(VE::LDQrii), DestReg)
        .addFrameIndex(FI)
        .addImm(0)
        .addImm(0)
        .addMemOperand(MMO);
  } else if (RC == &VE::VMRegClass) {
    BuildMI(MBB, I, DL, get(VE::LDVMrii), DestReg)
```
- **EN**: Implements logic around `getParent`, `getFrameInfo`, `getMachineMemOperand`, `getFixedStack`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getParent`, `getFrameInfo`, `getMachineMemOperand`, `getFixedStack`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 563-580
```cpp
        .addFrameIndex(FI)
        .addImm(0)
        .addImm(0)
        .addMemOperand(MMO);
  } else if (VE::VM512RegClass.hasSubClassEq(RC)) {
    BuildMI(MBB, I, DL, get(VE::LDVM512rii), DestReg)
        .addFrameIndex(FI)
        .addImm(0)
        .addImm(0)
        .addMemOperand(MMO);
  } else
    report_fatal_error("Can't load this register from stack slot");
}

bool VEInstrInfo::foldImmediate(MachineInstr &UseMI, MachineInstr &DefMI,
                                Register Reg, MachineRegisterInfo *MRI) const {
  LLVM_DEBUG(dbgs() << "foldImmediate\n");

```
- **EN**: Implements logic around `addFrameIndex`, `addImm`, `addMemOperand`, `BuildMI`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `addFrameIndex`, `addImm`, `addMemOperand`, `BuildMI`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 581-610
```cpp
  LLVM_DEBUG(dbgs() << "checking DefMI\n");
  int64_t ImmVal;
  switch (DefMI.getOpcode()) {
  default:
    return false;
  case VE::ORim:
    // General move small immediate instruction on VE.
    LLVM_DEBUG(dbgs() << "checking ORim\n");
    LLVM_DEBUG(DefMI.dump());
    // FIXME: We may need to support FPImm too.
    assert(DefMI.getOperand(1).isImm());
    assert(DefMI.getOperand(2).isImm());
    ImmVal =
        DefMI.getOperand(1).getImm() + mimm2Val(DefMI.getOperand(2).getImm());
    LLVM_DEBUG(dbgs() << "ImmVal is " << ImmVal << "\n");
    break;
  case VE::LEAzii:
    // General move immediate instruction on VE.
    LLVM_DEBUG(dbgs() << "checking LEAzii\n");
    LLVM_DEBUG(DefMI.dump());
    // FIXME: We may need to support FPImm too.
    assert(DefMI.getOperand(2).isImm());
    if (!DefMI.getOperand(3).isImm())
      // LEAzii may refer label
      return false;
    ImmVal = DefMI.getOperand(2).getImm() + DefMI.getOperand(3).getImm();
    LLVM_DEBUG(dbgs() << "ImmVal is " << ImmVal << "\n");
    break;
  }

```
- **EN**: Implements logic around `assert`, `getOperand`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `assert`, `getOperand` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果。

### Lines 611-636
```cpp
  // Try to fold like below:
  //   %1:i64 = ORim 0, 0(1)
  //   %2:i64 = CMPSLrr %0, %1
  // To
  //   %2:i64 = CMPSLrm %0, 0(1)
  //
  // Another example:
  //   %1:i64 = ORim 6, 0(1)
  //   %2:i64 = CMPSLrr %1, %0
  // To
  //   %2:i64 = CMPSLir 6, %0
  //
  // Support commutable instructions like below:
  //   %1:i64 = ORim 6, 0(1)
  //   %2:i64 = ADDSLrr %1, %0
  // To
  //   %2:i64 = ADDSLri %0, 6
  //
  // FIXME: Need to support i32.  Current implementtation requires
  //        EXTRACT_SUBREG, so input has following COPY and it avoids folding:
  //   %1:i64 = ORim 6, 0(1)
  //   %2:i32 = COPY %1.sub_i32
  //   %3:i32 = ADDSWSXrr %0, %2
  // FIXME: Need to support shift, cmov, and more instructions.
  // FIXME: Need to support lvl too, but LVLGen runs after peephole-opt.

```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 637-659
```cpp
  LLVM_DEBUG(dbgs() << "checking UseMI\n");
  LLVM_DEBUG(UseMI.dump());
  unsigned NewUseOpcSImm7;
  unsigned NewUseOpcMImm;
  enum InstType {
    rr2ri_rm, // rr -> ri or rm, commutable
    rr2ir_rm, // rr -> ir or rm
  } InstType;

  using namespace llvm::VE;
#define INSTRKIND(NAME)                                                        \
  case NAME##rr:                                                               \
    NewUseOpcSImm7 = NAME##ri;                                                 \
    NewUseOpcMImm = NAME##rm;                                                  \
    InstType = rr2ri_rm;                                                       \
    break
#define NCINSTRKIND(NAME)                                                      \
  case NAME##rr:                                                               \
    NewUseOpcSImm7 = NAME##ir;                                                 \
    NewUseOpcMImm = NAME##rm;                                                  \
    InstType = rr2ir_rm;                                                       \
    break

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 660-691
```cpp
  switch (UseMI.getOpcode()) {
  default:
    return false;

    INSTRKIND(ADDUL);
    INSTRKIND(ADDSWSX);
    INSTRKIND(ADDSWZX);
    INSTRKIND(ADDSL);
    NCINSTRKIND(SUBUL);
    NCINSTRKIND(SUBSWSX);
    NCINSTRKIND(SUBSWZX);
    NCINSTRKIND(SUBSL);
    INSTRKIND(MULUL);
    INSTRKIND(MULSWSX);
    INSTRKIND(MULSWZX);
    INSTRKIND(MULSL);
    NCINSTRKIND(DIVUL);
    NCINSTRKIND(DIVSWSX);
    NCINSTRKIND(DIVSWZX);
    NCINSTRKIND(DIVSL);
    NCINSTRKIND(CMPUL);
    NCINSTRKIND(CMPSWSX);
    NCINSTRKIND(CMPSWZX);
    NCINSTRKIND(CMPSL);
    INSTRKIND(MAXSWSX);
    INSTRKIND(MAXSWZX);
    INSTRKIND(MAXSL);
    INSTRKIND(MINSWSX);
    INSTRKIND(MINSWZX);
    INSTRKIND(MINSL);
    INSTRKIND(AND);
    INSTRKIND(OR);
```
- **EN**: Implements logic around `INSTRKIND`, `NCINSTRKIND`; this block uses `switch`-based dispatch; returns target-specific results.
- **CN**: 围绕 `INSTRKIND`, `NCINSTRKIND` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果。

### Lines 692-723
```cpp
    INSTRKIND(XOR);
    INSTRKIND(EQV);
    NCINSTRKIND(NND);
    NCINSTRKIND(MRG);
  }

#undef INSTRKIND

  unsigned NewUseOpc;
  unsigned UseIdx;
  bool Commute = false;
  LLVM_DEBUG(dbgs() << "checking UseMI operands\n");
  switch (InstType) {
  case rr2ri_rm:
    UseIdx = 2;
    if (UseMI.getOperand(1).getReg() == Reg) {
      Commute = true;
    } else {
      assert(UseMI.getOperand(2).getReg() == Reg);
    }
    if (isInt<7>(ImmVal)) {
      // This ImmVal matches to SImm7 slot, so change UseOpc to an instruction
      // holds a simm7 slot.
      NewUseOpc = NewUseOpcSImm7;
    } else if (isMImmVal(ImmVal)) {
      // Similarly, change UseOpc to an instruction holds a mimm slot.
      NewUseOpc = NewUseOpcMImm;
      ImmVal = val2MImm(ImmVal);
    } else
      return false;
    break;
  case rr2ir_rm:
```
- **EN**: Implements logic around `INSTRKIND`, `NCINSTRKIND`, `assert`, `val2MImm`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `INSTRKIND`, `NCINSTRKIND`, `assert`, `val2MImm` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果。

### Lines 724-741
```cpp
    if (UseMI.getOperand(1).getReg() == Reg) {
      // Check immediate value whether it matchs to the UseMI instruction.
      if (!isInt<7>(ImmVal))
        return false;
      NewUseOpc = NewUseOpcSImm7;
      UseIdx = 1;
    } else {
      assert(UseMI.getOperand(2).getReg() == Reg);
      // Check immediate value whether it matchs to the UseMI instruction.
      if (!isMImmVal(ImmVal))
        return false;
      NewUseOpc = NewUseOpcMImm;
      ImmVal = val2MImm(ImmVal);
      UseIdx = 2;
    }
    break;
  }

```
- **EN**: Implements logic around `assert`, `val2MImm`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `assert`, `val2MImm` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 742-760
```cpp
  LLVM_DEBUG(dbgs() << "modifying UseMI\n");
  bool DeleteDef = MRI->hasOneNonDBGUse(Reg);
  UseMI.setDesc(get(NewUseOpc));
  if (Commute) {
    UseMI.getOperand(1).setReg(UseMI.getOperand(UseIdx).getReg());
  }
  UseMI.getOperand(UseIdx).ChangeToImmediate(ImmVal);
  if (DeleteDef)
    DefMI.eraseFromParent();

  return true;
}

Register VEInstrInfo::getGlobalBaseReg(MachineFunction *MF) const {
  VEMachineFunctionInfo *VEFI = MF->getInfo<VEMachineFunctionInfo>();
  Register GlobalBaseReg = VEFI->getGlobalBaseReg();
  if (GlobalBaseReg != 0)
    return GlobalBaseReg;

```
- **EN**: Implements logic around `hasOneNonDBGUse`, `setDesc`, `getOperand`, `eraseFromParent`, ...; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `hasOneNonDBGUse`, `setDesc`, `getOperand`, `eraseFromParent`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 761-777
```cpp
  // We use %s15 (%got) as a global base register
  GlobalBaseReg = VE::SX15;

  // Insert a pseudo instruction to set the GlobalBaseReg into the first
  // MBB of the function
  MachineBasicBlock &FirstMBB = MF->front();
  MachineBasicBlock::iterator MBBI = FirstMBB.begin();
  DebugLoc dl;
  BuildMI(FirstMBB, MBBI, dl, get(VE::GETGOT), GlobalBaseReg);
  VEFI->setGlobalBaseReg(GlobalBaseReg);
  return GlobalBaseReg;
}

static Register getVM512Upper(Register reg) {
  return (reg - VE::VMP0) * 2 + VE::VM0;
}

```
- **EN**: Implements logic around `front`, `begin`, `BuildMI`, `setGlobalBaseReg`, ...; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `front`, `begin`, `BuildMI`, `setGlobalBaseReg`, ... 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 778-805
```cpp
static Register getVM512Lower(Register reg) { return getVM512Upper(reg) + 1; }

// Expand pseudo logical vector instructions for VM512 registers.
static void expandPseudoLogM(MachineInstr &MI, const MCInstrDesc &MCID) {
  MachineBasicBlock *MBB = MI.getParent();
  DebugLoc DL = MI.getDebugLoc();

  Register VMXu = getVM512Upper(MI.getOperand(0).getReg());
  Register VMXl = getVM512Lower(MI.getOperand(0).getReg());
  Register VMYu = getVM512Upper(MI.getOperand(1).getReg());
  Register VMYl = getVM512Lower(MI.getOperand(1).getReg());

  switch (MI.getOpcode()) {
  default: {
    Register VMZu = getVM512Upper(MI.getOperand(2).getReg());
    Register VMZl = getVM512Lower(MI.getOperand(2).getReg());
    BuildMI(*MBB, MI, DL, MCID).addDef(VMXu).addUse(VMYu).addUse(VMZu);
    BuildMI(*MBB, MI, DL, MCID).addDef(VMXl).addUse(VMYl).addUse(VMZl);
    break;
  }
  case VE::NEGMy:
    BuildMI(*MBB, MI, DL, MCID).addDef(VMXu).addUse(VMYu);
    BuildMI(*MBB, MI, DL, MCID).addDef(VMXl).addUse(VMYl);
    break;
  }
  MI.eraseFromParent();
}

```
- **EN**: Implements logic around `getVM512Lower`, `expandPseudoLogM`, `getParent`, `getDebugLoc`, ...; this block uses `switch`-based dispatch; returns target-specific results; works at the MachineInstr/MachineFunction layer; works at the MC layer.
- **CN**: 围绕 `getVM512Lower`, `expandPseudoLogM`, `getParent`, `getDebugLoc`, ... 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，工作在 MachineInstr/MachineFunction 层，工作在 MC 层。

### Lines 806-837
```cpp
static void addOperandsForVFMK(MachineInstrBuilder &MIB, MachineInstr &MI,
                               bool Upper) {
  // VM512
  MIB.addReg(Upper ? getVM512Upper(MI.getOperand(0).getReg())
                   : getVM512Lower(MI.getOperand(0).getReg()));

  switch (MI.getNumExplicitOperands()) {
  default:
    report_fatal_error("unexpected number of operands for pvfmk");
  case 2: // _Ml: VM512, VL
    // VL
    MIB.addReg(MI.getOperand(1).getReg());
    break;
  case 4: // _Mvl: VM512, CC, VR, VL
    // CC
    MIB.addImm(MI.getOperand(1).getImm());
    // VR
    MIB.addReg(MI.getOperand(2).getReg());
    // VL
    MIB.addReg(MI.getOperand(3).getReg());
    break;
  case 5: // _MvMl: VM512, CC, VR, VM512, VL
    // CC
    MIB.addImm(MI.getOperand(1).getImm());
    // VR
    MIB.addReg(MI.getOperand(2).getReg());
    // VM512
    MIB.addReg(Upper ? getVM512Upper(MI.getOperand(3).getReg())
                     : getVM512Lower(MI.getOperand(3).getReg()));
    // VL
    MIB.addReg(MI.getOperand(4).getReg());
    break;
```
- **EN**: Implements logic around `addOperandsForVFMK`, `addReg`, `getVM512Lower`, `report_fatal_error`, ...; this block uses `switch`-based dispatch; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `addOperandsForVFMK`, `addReg`, `getVM512Lower`, `report_fatal_error`, ... 实现具体逻辑；这一段使用 `switch` 分派，工作在 MachineInstr/MachineFunction 层。

### Lines 838-853
```cpp
  }
}

static void expandPseudoVFMK(const TargetInstrInfo &TI, MachineInstr &MI) {
  // replace to pvfmk.w.up and pvfmk.w.lo
  // replace to pvfmk.s.up and pvfmk.s.lo

  static const std::pair<unsigned, std::pair<unsigned, unsigned>> VFMKMap[] = {
      {VE::VFMKyal, {VE::VFMKLal, VE::VFMKLal}},
      {VE::VFMKynal, {VE::VFMKLnal, VE::VFMKLnal}},
      {VE::VFMKWyvl, {VE::PVFMKWUPvl, VE::PVFMKWLOvl}},
      {VE::VFMKWyvyl, {VE::PVFMKWUPvml, VE::PVFMKWLOvml}},
      {VE::VFMKSyvl, {VE::PVFMKSUPvl, VE::PVFMKSLOvl}},
      {VE::VFMKSyvyl, {VE::PVFMKSUPvml, VE::PVFMKSLOvml}},
  };

```
- **EN**: Implements logic around `expandPseudoVFMK`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `expandPseudoVFMK` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 854-871
```cpp
  unsigned Opcode = MI.getOpcode();

  const auto *Found =
      llvm::find_if(VFMKMap, [&](auto P) { return P.first == Opcode; });
  if (Found == std::end(VFMKMap))
    report_fatal_error("unexpected opcode for pseudo vfmk");

  unsigned OpcodeUpper = (*Found).second.first;
  unsigned OpcodeLower = (*Found).second.second;

  MachineBasicBlock *MBB = MI.getParent();
  DebugLoc DL = MI.getDebugLoc();

  MachineInstrBuilder Bu = BuildMI(*MBB, MI, DL, TI.get(OpcodeUpper));
  addOperandsForVFMK(Bu, MI, /* Upper */ true);
  MachineInstrBuilder Bl = BuildMI(*MBB, MI, DL, TI.get(OpcodeLower));
  addOperandsForVFMK(Bl, MI, /* Upper */ false);

```
- **EN**: Implements logic around `getOpcode`, `find_if`, `report_fatal_error`, `getParent`, ...; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getOpcode`, `find_if`, `report_fatal_error`, `getParent`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 872-887
```cpp
  MI.eraseFromParent();
}

bool VEInstrInfo::expandPostRAPseudo(MachineInstr &MI) const {
  switch (MI.getOpcode()) {
  case VE::EXTEND_STACK: {
    return expandExtendStackPseudo(MI);
  }
  case VE::EXTEND_STACK_GUARD: {
    MI.eraseFromParent(); // The pseudo instruction is gone now.
    return true;
  }
  case VE::GETSTACKTOP: {
    return expandGetStackTopPseudo(MI);
  }

```
- **EN**: Implements logic around `eraseFromParent`, `expandPostRAPseudo`, `expandExtendStackPseudo`, `expandGetStackTopPseudo`; this block uses `switch`-based dispatch; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `eraseFromParent`, `expandPostRAPseudo`, `expandExtendStackPseudo`, `expandGetStackTopPseudo` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 888-906
```cpp
  case VE::ANDMyy:
    expandPseudoLogM(MI, get(VE::ANDMmm));
    return true;
  case VE::ORMyy:
    expandPseudoLogM(MI, get(VE::ORMmm));
    return true;
  case VE::XORMyy:
    expandPseudoLogM(MI, get(VE::XORMmm));
    return true;
  case VE::EQVMyy:
    expandPseudoLogM(MI, get(VE::EQVMmm));
    return true;
  case VE::NNDMyy:
    expandPseudoLogM(MI, get(VE::NNDMmm));
    return true;
  case VE::NEGMy:
    expandPseudoLogM(MI, get(VE::NEGMm));
    return true;

```
- **EN**: Implements logic around `expandPseudoLogM`; this block returns target-specific results.
- **CN**: 围绕 `expandPseudoLogM` 实现具体逻辑；这一段返回目标相关结果。

### Lines 907-938
```cpp
  case VE::LVMyir:
  case VE::LVMyim:
  case VE::LVMyir_y:
  case VE::LVMyim_y: {
    Register VMXu = getVM512Upper(MI.getOperand(0).getReg());
    Register VMXl = getVM512Lower(MI.getOperand(0).getReg());
    int64_t Imm = MI.getOperand(1).getImm();
    bool IsSrcReg =
        MI.getOpcode() == VE::LVMyir || MI.getOpcode() == VE::LVMyir_y;
    Register Src = IsSrcReg ? MI.getOperand(2).getReg() : VE::NoRegister;
    int64_t MImm = IsSrcReg ? 0 : MI.getOperand(2).getImm();
    bool KillSrc = IsSrcReg ? MI.getOperand(2).isKill() : false;
    Register VMX = VMXl;
    if (Imm >= 4) {
      VMX = VMXu;
      Imm -= 4;
    }
    MachineBasicBlock *MBB = MI.getParent();
    DebugLoc DL = MI.getDebugLoc();
    switch (MI.getOpcode()) {
    case VE::LVMyir:
      BuildMI(*MBB, MI, DL, get(VE::LVMir))
          .addDef(VMX)
          .addImm(Imm)
          .addReg(Src, getKillRegState(KillSrc));
      break;
    case VE::LVMyim:
      BuildMI(*MBB, MI, DL, get(VE::LVMim))
          .addDef(VMX)
          .addImm(Imm)
          .addImm(MImm);
      break;
```
- **EN**: Implements logic around `getVM512Upper`, `getVM512Lower`, `getOperand`, `getOpcode`, ...; this block uses `switch`-based dispatch; applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getVM512Upper`, `getVM512Lower`, `getOperand`, `getOpcode`, ... 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 939-970
```cpp
    case VE::LVMyir_y:
      assert(MI.getOperand(0).getReg() == MI.getOperand(3).getReg() &&
             "LVMyir_y has different register in 3rd operand");
      BuildMI(*MBB, MI, DL, get(VE::LVMir_m))
          .addDef(VMX)
          .addImm(Imm)
          .addReg(Src, getKillRegState(KillSrc))
          .addReg(VMX);
      break;
    case VE::LVMyim_y:
      assert(MI.getOperand(0).getReg() == MI.getOperand(3).getReg() &&
             "LVMyim_y has different register in 3rd operand");
      BuildMI(*MBB, MI, DL, get(VE::LVMim_m))
          .addDef(VMX)
          .addImm(Imm)
          .addImm(MImm)
          .addReg(VMX);
      break;
    }
    MI.eraseFromParent();
    return true;
  }
  case VE::SVMyi: {
    Register Dest = MI.getOperand(0).getReg();
    Register VMZu = getVM512Upper(MI.getOperand(1).getReg());
    Register VMZl = getVM512Lower(MI.getOperand(1).getReg());
    bool KillSrc = MI.getOperand(1).isKill();
    int64_t Imm = MI.getOperand(2).getImm();
    Register VMZ = VMZl;
    if (Imm >= 4) {
      VMZ = VMZu;
      Imm -= 4;
```
- **EN**: Implements logic around `assert`, `BuildMI`, `addDef`, `addImm`, ...; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `assert`, `BuildMI`, `addDef`, `addImm`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 971-995
```cpp
    }
    MachineBasicBlock *MBB = MI.getParent();
    DebugLoc DL = MI.getDebugLoc();
    MachineInstrBuilder MIB =
        BuildMI(*MBB, MI, DL, get(VE::SVMmi), Dest).addReg(VMZ).addImm(Imm);
    MachineInstr *Inst = MIB.getInstr();
    if (KillSrc) {
      const TargetRegisterInfo *TRI = &getRegisterInfo();
      Inst->addRegisterKilled(MI.getOperand(1).getReg(), TRI, true);
    }
    MI.eraseFromParent();
    return true;
  }
  case VE::VFMKyal:
  case VE::VFMKynal:
  case VE::VFMKWyvl:
  case VE::VFMKWyvyl:
  case VE::VFMKSyvl:
  case VE::VFMKSyvyl:
    expandPseudoVFMK(*this, MI);
    return true;
  }
  return false;
}

```
- **EN**: Implements logic around `getParent`, `getDebugLoc`, `BuildMI`, `getInstr`, ...; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getParent`, `getDebugLoc`, `BuildMI`, `getInstr`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 996-1017
```cpp
bool VEInstrInfo::expandExtendStackPseudo(MachineInstr &MI) const {
  MachineBasicBlock &MBB = *MI.getParent();
  MachineFunction &MF = *MBB.getParent();
  const VESubtarget &STI = MF.getSubtarget<VESubtarget>();
  const VEInstrInfo &TII = *STI.getInstrInfo();
  DebugLoc dl = MBB.findDebugLoc(MI);

  // Create following instructions and multiple basic blocks.
  //
  // thisBB:
  //   brge.l.t %sp, %sl, sinkBB
  // syscallBB:
  //   ld      %s61, 0x18(, %tp)        // load param area
  //   or      %s62, 0, %s0             // spill the value of %s0
  //   lea     %s63, 0x13b              // syscall # of grow
  //   shm.l   %s63, 0x0(%s61)          // store syscall # at addr:0
  //   shm.l   %sl, 0x8(%s61)           // store old limit at addr:8
  //   shm.l   %sp, 0x10(%s61)          // store new limit at addr:16
  //   monc                             // call monitor
  //   or      %s0, 0, %s62             // restore the value of %s0
  // sinkBB:

```
- **EN**: Implements logic around `expandExtendStackPseudo`, `getParent`, `getSubtarget<VESubtarget>`, `getInstrInfo`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `expandExtendStackPseudo`, `getParent`, `getSubtarget<VESubtarget>`, `getInstrInfo`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 1018-1041
```cpp
  // Create new MBB
  MachineBasicBlock *BB = &MBB;
  const BasicBlock *LLVM_BB = BB->getBasicBlock();
  MachineBasicBlock *syscallMBB = MF.CreateMachineBasicBlock(LLVM_BB);
  MachineBasicBlock *sinkMBB = MF.CreateMachineBasicBlock(LLVM_BB);
  MachineFunction::iterator It = ++(BB->getIterator());
  MF.insert(It, syscallMBB);
  MF.insert(It, sinkMBB);

  // Transfer the remainder of BB and its successor edges to sinkMBB.
  sinkMBB->splice(sinkMBB->begin(), BB,
                  std::next(std::next(MachineBasicBlock::iterator(MI))),
                  BB->end());
  sinkMBB->transferSuccessorsAndUpdatePHIs(BB);

  // Next, add the true and fallthrough blocks as its successors.
  BB->addSuccessor(syscallMBB);
  BB->addSuccessor(sinkMBB);
  BuildMI(BB, dl, TII.get(VE::BRCFLrr_t))
      .addImm(VECC::CC_IGE)
      .addReg(VE::SX11) // %sp
      .addReg(VE::SX8)  // %sl
      .addMBB(sinkMBB);

```
- **EN**: Implements logic around `getBasicBlock`, `CreateMachineBasicBlock`, `getIterator`, `insert`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getBasicBlock`, `CreateMachineBasicBlock`, `getIterator`, `insert`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 1042-1071
```cpp
  BB = syscallMBB;

  // Update machine-CFG edges
  BB->addSuccessor(sinkMBB);

  BuildMI(BB, dl, TII.get(VE::LDrii), VE::SX61)
      .addReg(VE::SX14)
      .addImm(0)
      .addImm(0x18);
  BuildMI(BB, dl, TII.get(VE::ORri), VE::SX62)
      .addReg(VE::SX0)
      .addImm(0);
  BuildMI(BB, dl, TII.get(VE::LEAzii), VE::SX63)
      .addImm(0)
      .addImm(0)
      .addImm(0x13b);
  BuildMI(BB, dl, TII.get(VE::SHMLri))
      .addReg(VE::SX61)
      .addImm(0)
      .addReg(VE::SX63);
  BuildMI(BB, dl, TII.get(VE::SHMLri))
      .addReg(VE::SX61)
      .addImm(8)
      .addReg(VE::SX8);
  BuildMI(BB, dl, TII.get(VE::SHMLri))
      .addReg(VE::SX61)
      .addImm(16)
      .addReg(VE::SX11);
  BuildMI(BB, dl, TII.get(VE::MONC));

```
- **EN**: Implements logic around `addSuccessor`, `BuildMI`, `addReg`, `addImm`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `addSuccessor`, `BuildMI`, `addReg`, `addImm` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 1072-1090
```cpp
  BuildMI(BB, dl, TII.get(VE::ORri), VE::SX0)
      .addReg(VE::SX62)
      .addImm(0);

  MI.eraseFromParent(); // The pseudo instruction is gone now.
  return true;
}

bool VEInstrInfo::expandGetStackTopPseudo(MachineInstr &MI) const {
  MachineBasicBlock *MBB = MI.getParent();
  MachineFunction &MF = *MBB->getParent();
  const VESubtarget &STI = MF.getSubtarget<VESubtarget>();
  const VEInstrInfo &TII = *STI.getInstrInfo();
  DebugLoc DL = MBB->findDebugLoc(MI);

  // Create following instruction
  //
  //   dst = %sp + target specific frame + the size of parameter area

```
- **EN**: Implements logic around `BuildMI`, `addReg`, `addImm`, `eraseFromParent`, ...; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `addReg`, `addImm`, `eraseFromParent`, ... 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 1091-1107
```cpp
  const MachineFrameInfo &MFI = MF.getFrameInfo();
  const VEFrameLowering &TFL = *STI.getFrameLowering();

  // The VE ABI requires a reserved area at the top of stack as described
  // in VEFrameLowering.cpp.  So, we adjust it here.
  unsigned NumBytes = STI.getAdjustedFrameSize(0);

  // Also adds the size of parameter area.
  if (MFI.adjustsStack() && TFL.hasReservedCallFrame(MF))
    NumBytes += MFI.getMaxCallFrameSize();

  BuildMI(*MBB, MI, DL, TII.get(VE::LEArii))
      .addDef(MI.getOperand(0).getReg())
      .addReg(VE::SX11)
      .addImm(0)
      .addImm(NumBytes);

```
- **EN**: Implements logic around `getFrameInfo`, `getFrameLowering`, `getAdjustedFrameSize`, `getMaxCallFrameSize`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getFrameInfo`, `getFrameLowering`, `getAdjustedFrameSize`, `getMaxCallFrameSize`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 1108-1110
```cpp
  MI.eraseFromParent(); // The pseudo instruction is gone now.
  return true;
}
```
- **EN**: Implements logic around `eraseFromParent`; this block returns target-specific results.
- **CN**: 围绕 `eraseFromParent` 实现具体逻辑；这一段返回目标相关结果。

## Key Concepts / 关键概念

- **Instruction semantics / 指令语义**:
  - **EN**: Encodes instruction behavior and helper routines
  - **CN**: 描述指令行为与辅助例程
- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `VEInstrInfo.h`, `VE.h`, `VEMachineFunctionInfo.h`, `VESubtarget.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVector.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineMemOperand.h`, `llvm/CodeGen/MachineRegisterInfo.h`, `llvm/MC/TargetRegistry.h`, `llvm/Support/Debug.h` ... (+2 more)
- **LLVM subsystems / LLVM 子系统**: MC, CodeGen, Support
- **Generated macros / 生成宏**: `GET_INSTRINFO_CTOR_DTOR`
