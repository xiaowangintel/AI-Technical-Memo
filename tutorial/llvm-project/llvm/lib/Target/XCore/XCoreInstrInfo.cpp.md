# XCoreInstrInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/XCore/XCoreInstrInfo.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements target-specific instruction information, instruction helpers, and scheduling-related behavior.
  - **CN**: 实现目标相关的指令信息、指令辅助逻辑以及与调度相关的行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- XCoreInstrInfo.cpp - XCore Instruction Information ----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the XCore implementation of the TargetInstrInfo class.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 12-24
```cpp

#include "XCoreInstrInfo.h"
#include "XCore.h"
#include "XCoreSubtarget.h"
#include "llvm/CodeGen/MachineConstantPool.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineMemOperand.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/Function.h"
#include "llvm/MC/MCContext.h"
#include "llvm/Support/ErrorHandling.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `XCoreInstrInfo.h`, `XCore.h`, `XCoreSubtarget.h`, `llvm/CodeGen/MachineConstantPool.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `XCoreInstrInfo.h`, `XCore.h`, `XCoreSubtarget.h`, `llvm/CodeGen/MachineConstantPool.h`。

### Lines 25-41
```cpp
using namespace llvm;

#define GET_INSTRINFO_CTOR_DTOR
#include "XCoreGenInstrInfo.inc"

namespace llvm {
namespace XCore {

  // XCore Condition Codes
  enum CondCode {
    COND_TRUE,
    COND_FALSE,
    COND_INVALID
  };
}
}

```
- **EN**: Pulls in the headers needed for this implementation, including `XCoreGenInstrInfo.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `XCoreGenInstrInfo.inc`。

### Lines 42-52
```cpp
// Pin the vtable to this file.
void XCoreInstrInfo::anchor() {}

XCoreInstrInfo::XCoreInstrInfo(const XCoreSubtarget &ST)
    : XCoreGenInstrInfo(ST, RI, XCore::ADJCALLSTACKDOWN, XCore::ADJCALLSTACKUP),
      RI() {}

static bool isZeroImm(const MachineOperand &op) {
  return op.isImm() && op.getImm() == 0;
}

```
- **EN**: Implements logic around `anchor`, `XCoreInstrInfo`, `XCoreGenInstrInfo`, `RI`, ...; this block returns target-specific results.
- **CN**: 围绕 `anchor`, `XCoreInstrInfo`, `XCoreGenInstrInfo`, `RI`, ... 实现具体逻辑；这一段返回目标相关结果。

### Lines 53-70
```cpp
/// isLoadFromStackSlot - If the specified machine instruction is a direct
/// load from a stack slot, return the virtual or physical register number of
/// the destination along with the FrameIndex of the loaded stack slot.  If
/// not, return 0.  This predicate must return 0 if the instruction has
/// any side effects other than loading from the stack slot.
Register XCoreInstrInfo::isLoadFromStackSlot(const MachineInstr &MI,
                                             int &FrameIndex) const {
  int Opcode = MI.getOpcode();
  if (Opcode == XCore::LDWFI)
  {
    if ((MI.getOperand(1).isFI()) &&  // is a stack slot
        (MI.getOperand(2).isImm()) && // the imm is zero
        (isZeroImm(MI.getOperand(2)))) {
      FrameIndex = MI.getOperand(1).getIndex();
      return MI.getOperand(0).getReg();
    }
  }
  return 0;
```
- **EN**: Implements logic around `isLoadFromStackSlot`, `getOpcode`, `getOperand`, `isZeroImm`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `isLoadFromStackSlot`, `getOpcode`, `getOperand`, `isZeroImm` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 71-88
```cpp
}

  /// isStoreToStackSlot - If the specified machine instruction is a direct
  /// store to a stack slot, return the virtual or physical register number of
  /// the source reg along with the FrameIndex of the loaded stack slot.  If
  /// not, return 0.  This predicate must return 0 if the instruction has
  /// any side effects other than storing to the stack slot.
Register XCoreInstrInfo::isStoreToStackSlot(const MachineInstr &MI,
                                            int &FrameIndex) const {
  int Opcode = MI.getOpcode();
  if (Opcode == XCore::STWFI)
  {
    if ((MI.getOperand(1).isFI()) &&  // is a stack slot
        (MI.getOperand(2).isImm()) && // the imm is zero
        (isZeroImm(MI.getOperand(2)))) {
      FrameIndex = MI.getOperand(1).getIndex();
      return MI.getOperand(0).getReg();
    }
```
- **EN**: Implements logic around `isStoreToStackSlot`, `getOpcode`, `getOperand`, `isZeroImm`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `isStoreToStackSlot`, `getOpcode`, `getOperand`, `isZeroImm` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 89-103
```cpp
  }
  return 0;
}

//===----------------------------------------------------------------------===//
// Branch Analysis
//===----------------------------------------------------------------------===//

static inline bool IsBRU(unsigned BrOpc) {
  return BrOpc == XCore::BRFU_u6
      || BrOpc == XCore::BRFU_lu6
      || BrOpc == XCore::BRBU_u6
      || BrOpc == XCore::BRBU_lu6;
}

```
- **EN**: Implements logic around `IsBRU`; this block returns target-specific results.
- **CN**: 围绕 `IsBRU` 实现具体逻辑；这一段返回目标相关结果。

### Lines 104-117
```cpp
static inline bool IsBRT(unsigned BrOpc) {
  return BrOpc == XCore::BRFT_ru6
      || BrOpc == XCore::BRFT_lru6
      || BrOpc == XCore::BRBT_ru6
      || BrOpc == XCore::BRBT_lru6;
}

static inline bool IsBRF(unsigned BrOpc) {
  return BrOpc == XCore::BRFF_ru6
      || BrOpc == XCore::BRFF_lru6
      || BrOpc == XCore::BRBF_ru6
      || BrOpc == XCore::BRBF_lru6;
}

```
- **EN**: Implements logic around `IsBRT`, `IsBRF`; this block returns target-specific results.
- **CN**: 围绕 `IsBRT`, `IsBRF` 实现具体逻辑；这一段返回目标相关结果。

### Lines 118-126
```cpp
static inline bool IsCondBranch(unsigned BrOpc) {
  return IsBRF(BrOpc) || IsBRT(BrOpc);
}

static inline bool IsBR_JT(unsigned BrOpc) {
  return BrOpc == XCore::BR_JT
      || BrOpc == XCore::BR_JT32;
}

```
- **EN**: Implements logic around `IsCondBranch`, `IsBRF`, `IsBR_JT`; this block returns target-specific results.
- **CN**: 围绕 `IsCondBranch`, `IsBRF`, `IsBR_JT` 实现具体逻辑；这一段返回目标相关结果。

### Lines 127-139
```cpp
/// GetCondFromBranchOpc - Return the XCore CC that matches
/// the correspondent Branch instruction opcode.
static XCore::CondCode GetCondFromBranchOpc(unsigned BrOpc)
{
  if (IsBRT(BrOpc)) {
    return XCore::COND_TRUE;
  } else if (IsBRF(BrOpc)) {
    return XCore::COND_FALSE;
  } else {
    return XCore::COND_INVALID;
  }
}

```
- **EN**: Implements logic around `GetCondFromBranchOpc`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `GetCondFromBranchOpc` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 140-150
```cpp
/// GetCondBranchFromCond - Return the Branch instruction
/// opcode that matches the cc.
static inline unsigned GetCondBranchFromCond(XCore::CondCode CC)
{
  switch (CC) {
  default: llvm_unreachable("Illegal condition code!");
  case XCore::COND_TRUE   : return XCore::BRFT_lru6;
  case XCore::COND_FALSE  : return XCore::BRFF_lru6;
  }
}

```
- **EN**: Implements logic around `GetCondBranchFromCond`, `llvm_unreachable`; this block uses `switch`-based dispatch; returns target-specific results.
- **CN**: 围绕 `GetCondBranchFromCond`, `llvm_unreachable` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果。

### Lines 151-161
```cpp
/// GetOppositeBranchCondition - Return the inverse of the specified
/// condition, e.g. turning COND_E to COND_NE.
static inline XCore::CondCode GetOppositeBranchCondition(XCore::CondCode CC)
{
  switch (CC) {
  default: llvm_unreachable("Illegal condition code!");
  case XCore::COND_TRUE   : return XCore::COND_FALSE;
  case XCore::COND_FALSE  : return XCore::COND_TRUE;
  }
}

```
- **EN**: Implements logic around `GetOppositeBranchCondition`, `llvm_unreachable`; this block uses `switch`-based dispatch; returns target-specific results.
- **CN**: 围绕 `GetOppositeBranchCondition`, `llvm_unreachable` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果。

### Lines 162-170
```cpp
/// analyzeBranch - Analyze the branching code at the end of MBB, returning
/// true if it cannot be understood (e.g. it's a switch dispatch or isn't
/// implemented for a target).  Upon success, this returns false and returns
/// with the following information in various cases:
///
/// 1. If this block ends with no branches (it just falls through to its succ)
///    just return false, leaving TBB/FBB null.
/// 2. If this block ends with only an unconditional branch, it sets TBB to be
///    the destination block.
```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 171-179
```cpp
/// 3. If this block ends with an conditional branch and it falls through to
///    an successor block, it sets TBB to be the branch destination block and a
///    list of operands that evaluate the condition. These
///    operands can be passed to other TargetInstrInfo methods to create new
///    branches.
/// 4. If this block ends with an conditional branch and an unconditional
///    block, it returns the 'true' destination in TBB, the 'false' destination
///    in FBB, and a list of operands that evaluate the condition. These
///    operands can be passed to other TargetInstrInfo methods to create new
```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 180-194
```cpp
///    branches.
///
/// Note that removeBranch and insertBranch must be implemented to support
/// cases where this method returns success.
///
bool XCoreInstrInfo::analyzeBranch(MachineBasicBlock &MBB,
                                   MachineBasicBlock *&TBB,
                                   MachineBasicBlock *&FBB,
                                   SmallVectorImpl<MachineOperand> &Cond,
                                   bool AllowModify) const {
  // If the block has no terminators, it just falls into the block after it.
  MachineBasicBlock::iterator I = MBB.getLastNonDebugInstr();
  if (I == MBB.end())
    return false;

```
- **EN**: Implements logic around `analyzeBranch`, `getLastNonDebugInstr`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `analyzeBranch`, `getLastNonDebugInstr` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 195-207
```cpp
  if (!isUnpredicatedTerminator(*I))
    return false;

  // Get the last instruction in the block.
  MachineInstr *LastInst = &*I;

  // If there is only one terminator instruction, process it.
  if (I == MBB.begin() || !isUnpredicatedTerminator(*--I)) {
    if (IsBRU(LastInst->getOpcode())) {
      TBB = LastInst->getOperand(0).getMBB();
      return false;
    }

```
- **EN**: Implements logic around `getOperand`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 208-220
```cpp
    XCore::CondCode BranchCode = GetCondFromBranchOpc(LastInst->getOpcode());
    if (BranchCode == XCore::COND_INVALID)
      return true;  // Can't handle indirect branch.

    // Conditional branch
    // Block ends with fall-through condbranch.

    TBB = LastInst->getOperand(1).getMBB();
    Cond.push_back(MachineOperand::CreateImm(BranchCode));
    Cond.push_back(LastInst->getOperand(0));
    return false;
  }

```
- **EN**: Implements logic around `GetCondFromBranchOpc`, `getOperand`, `push_back`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `GetCondFromBranchOpc`, `getOperand`, `push_back` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 221-230
```cpp
  // Get the instruction before it if it's a terminator.
  MachineInstr *SecondLastInst = &*I;

  // If there are three terminators, we don't know what sort of block this is.
  if (SecondLastInst && I != MBB.begin() && isUnpredicatedTerminator(*--I))
    return true;

  unsigned SecondLastOpc    = SecondLastInst->getOpcode();
  XCore::CondCode BranchCode = GetCondFromBranchOpc(SecondLastOpc);

```
- **EN**: Implements logic around `getOpcode`, `GetCondFromBranchOpc`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getOpcode`, `GetCondFromBranchOpc` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 231-239
```cpp
  // If the block ends with conditional branch followed by unconditional,
  // handle it.
  if (BranchCode != XCore::COND_INVALID
    && IsBRU(LastInst->getOpcode())) {

    TBB = SecondLastInst->getOperand(1).getMBB();
    Cond.push_back(MachineOperand::CreateImm(BranchCode));
    Cond.push_back(SecondLastInst->getOperand(0));

```
- **EN**: Implements logic around `IsBRU`, `getOperand`, `push_back`; this block applies conditional target rules.
- **CN**: 围绕 `IsBRU`, `getOperand`, `push_back` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 240-254
```cpp
    FBB = LastInst->getOperand(0).getMBB();
    return false;
  }

  // If the block ends with two unconditional branches, handle it.  The second
  // one is not executed, so remove it.
  if (IsBRU(SecondLastInst->getOpcode()) &&
      IsBRU(LastInst->getOpcode())) {
    TBB = SecondLastInst->getOperand(0).getMBB();
    I = LastInst;
    if (AllowModify)
      I->eraseFromParent();
    return false;
  }

```
- **EN**: Implements logic around `getOperand`, `IsBRU`, `eraseFromParent`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getOperand`, `IsBRU`, `eraseFromParent` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 255-266
```cpp
  // Likewise if it ends with a branch table followed by an unconditional branch.
  if (IsBR_JT(SecondLastInst->getOpcode()) && IsBRU(LastInst->getOpcode())) {
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

### Lines 267-278
```cpp
unsigned XCoreInstrInfo::insertBranch(MachineBasicBlock &MBB,
                                      MachineBasicBlock *TBB,
                                      MachineBasicBlock *FBB,
                                      ArrayRef<MachineOperand> Cond,
                                      const DebugLoc &DL,
                                      int *BytesAdded) const {
  // Shouldn't be a fall through.
  assert(TBB && "insertBranch must not be told to insert a fallthrough");
  assert((Cond.size() == 2 || Cond.size() == 0) &&
         "Unexpected number of components!");
  assert(!BytesAdded && "code size not handled");

```
- **EN**: Implements logic around `insertBranch`, `assert`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `insertBranch`, `assert` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 279-291
```cpp
  if (!FBB) { // One way branch.
    if (Cond.empty()) {
      // Unconditional branch
      BuildMI(&MBB, DL, get(XCore::BRFU_lu6)).addMBB(TBB);
    } else {
      // Conditional branch.
      unsigned Opc = GetCondBranchFromCond((XCore::CondCode)Cond[0].getImm());
      BuildMI(&MBB, DL, get(Opc)).addReg(Cond[1].getReg())
                             .addMBB(TBB);
    }
    return 1;
  }

```
- **EN**: Implements logic around `BuildMI`, `GetCondBranchFromCond`, `addMBB`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `GetCondBranchFromCond`, `addMBB` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 292-300
```cpp
  // Two-way Conditional branch.
  assert(Cond.size() == 2 && "Unexpected number of components!");
  unsigned Opc = GetCondBranchFromCond((XCore::CondCode)Cond[0].getImm());
  BuildMI(&MBB, DL, get(Opc)).addReg(Cond[1].getReg())
                         .addMBB(TBB);
  BuildMI(&MBB, DL, get(XCore::BRFU_lu6)).addMBB(FBB);
  return 2;
}

```
- **EN**: Implements logic around `assert`, `GetCondBranchFromCond`, `BuildMI`, `addMBB`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `assert`, `GetCondBranchFromCond`, `BuildMI`, `addMBB` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 301-311
```cpp
unsigned
XCoreInstrInfo::removeBranch(MachineBasicBlock &MBB, int *BytesRemoved) const {
  assert(!BytesRemoved && "code size not handled");

  MachineBasicBlock::iterator I = MBB.getLastNonDebugInstr();
  if (I == MBB.end())
    return 0;

  if (!IsBRU(I->getOpcode()) && !IsCondBranch(I->getOpcode()))
    return 0;

```
- **EN**: Implements logic around `removeBranch`, `assert`, `getLastNonDebugInstr`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `removeBranch`, `assert`, `getLastNonDebugInstr` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 312-321
```cpp
  // Remove the branch.
  I->eraseFromParent();

  I = MBB.end();

  if (I == MBB.begin()) return 1;
  --I;
  if (!IsCondBranch(I->getOpcode()))
    return 1;

```
- **EN**: Implements logic around `eraseFromParent`, `end`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `eraseFromParent`, `end` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 322-334
```cpp
  // Remove the branch.
  I->eraseFromParent();
  return 2;
}

void XCoreInstrInfo::copyPhysReg(MachineBasicBlock &MBB,
                                 MachineBasicBlock::iterator I,
                                 const DebugLoc &DL, Register DestReg,
                                 Register SrcReg, bool KillSrc,
                                 bool RenamableDest, bool RenamableSrc) const {
  bool GRDest = XCore::GRRegsRegClass.contains(DestReg);
  bool GRSrc  = XCore::GRRegsRegClass.contains(SrcReg);

```
- **EN**: Implements logic around `eraseFromParent`, `copyPhysReg`, `contains`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `eraseFromParent`, `copyPhysReg`, `contains` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 335-346
```cpp
  if (GRDest && GRSrc) {
    BuildMI(MBB, I, DL, get(XCore::ADD_2rus), DestReg)
      .addReg(SrcReg, getKillRegState(KillSrc))
      .addImm(0);
    return;
  }

  if (GRDest && SrcReg == XCore::SP) {
    BuildMI(MBB, I, DL, get(XCore::LDAWSP_ru6), DestReg).addImm(0);
    return;
  }

```
- **EN**: Implements logic around `BuildMI`, `addReg`, `addImm`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `addReg`, `addImm` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 347-358
```cpp
  if (DestReg == XCore::SP && GRSrc) {
    BuildMI(MBB, I, DL, get(XCore::SETSP_1r))
      .addReg(SrcReg, getKillRegState(KillSrc));
    return;
  }
  llvm_unreachable("Impossible reg-to-reg copy");
}

void XCoreInstrInfo::storeRegToStackSlot(
    MachineBasicBlock &MBB, MachineBasicBlock::iterator I, Register SrcReg,
    bool isKill, int FrameIndex, const TargetRegisterClass *RC,

```
- **EN**: Implements logic around `BuildMI`, `addReg`, `llvm_unreachable`, `storeRegToStackSlot`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `addReg`, `llvm_unreachable`, `storeRegToStackSlot` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 359-375
```cpp
    Register VReg, MachineInstr::MIFlag Flags) const {
  DebugLoc DL;
  if (I != MBB.end() && !I->isDebugInstr())
    DL = I->getDebugLoc();
  MachineFunction *MF = MBB.getParent();
  const MachineFrameInfo &MFI = MF->getFrameInfo();
  MachineMemOperand *MMO = MF->getMachineMemOperand(
      MachinePointerInfo::getFixedStack(*MF, FrameIndex),
      MachineMemOperand::MOStore, MFI.getObjectSize(FrameIndex),
      MFI.getObjectAlign(FrameIndex));
  BuildMI(MBB, I, DL, get(XCore::STWFI))
    .addReg(SrcReg, getKillRegState(isKill))
    .addFrameIndex(FrameIndex)
    .addImm(0)
    .addMemOperand(MMO);
}

```
- **EN**: Implements logic around `getDebugLoc`, `getParent`, `getFrameInfo`, `getMachineMemOperand`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getDebugLoc`, `getParent`, `getFrameInfo`, `getMachineMemOperand`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 376-393
```cpp
void XCoreInstrInfo::loadRegFromStackSlot(MachineBasicBlock &MBB,
                                          MachineBasicBlock::iterator I,
                                          Register DestReg, int FrameIndex,
                                          const TargetRegisterClass *RC,
                                          Register VReg, unsigned SubReg,
                                          MachineInstr::MIFlag Flags) const {
  DebugLoc DL;
  if (I != MBB.end() && !I->isDebugInstr())
    DL = I->getDebugLoc();
  MachineFunction *MF = MBB.getParent();
  const MachineFrameInfo &MFI = MF->getFrameInfo();
  MachineMemOperand *MMO = MF->getMachineMemOperand(
      MachinePointerInfo::getFixedStack(*MF, FrameIndex),
      MachineMemOperand::MOLoad, MFI.getObjectSize(FrameIndex),
      MFI.getObjectAlign(FrameIndex));
  BuildMI(MBB, I, DL, get(XCore::LDWFI), DestReg)
    .addFrameIndex(FrameIndex)
    .addImm(0)
```
- **EN**: Implements logic around `loadRegFromStackSlot`, `getDebugLoc`, `getParent`, `getFrameInfo`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `loadRegFromStackSlot`, `getDebugLoc`, `getParent`, `getFrameInfo`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 394-404
```cpp
    .addMemOperand(MMO);
}

bool XCoreInstrInfo::
reverseBranchCondition(SmallVectorImpl<MachineOperand> &Cond) const {
  assert((Cond.size() == 2) &&
          "Invalid XCore branch condition!");
  Cond[0].setImm(GetOppositeBranchCondition((XCore::CondCode)Cond[0].getImm()));
  return false;
}

```
- **EN**: Implements logic around `addMemOperand`, `reverseBranchCondition`, `assert`, `setImm`; this block returns target-specific results.
- **CN**: 围绕 `addMemOperand`, `reverseBranchCondition`, `assert`, `setImm` 实现具体逻辑；这一段返回目标相关结果。

### Lines 405-420
```cpp
static inline bool isImmU6(unsigned val) {
  return val < (1 << 6);
}

static inline bool isImmU16(unsigned val) {
  return val < (1 << 16);
}

static bool isImmMskBitp(unsigned val) {
  if (!isMask_32(val)) {
    return false;
  }
  int N = llvm::bit_width(val);
  return (N >= 1 && N <= 8) || N == 16 || N == 24 || N == 32;
}

```
- **EN**: Implements logic around `isImmU6`, `isImmU16`, `isImmMskBitp`, `bit_width`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `isImmU6`, `isImmU16`, `isImmMskBitp`, `bit_width` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 421-438
```cpp
MachineBasicBlock::iterator XCoreInstrInfo::loadImmediate(
                                              MachineBasicBlock &MBB,
                                              MachineBasicBlock::iterator MI,
                                              unsigned Reg, uint64_t Value) const {
  DebugLoc dl;
  if (MI != MBB.end() && !MI->isDebugInstr())
    dl = MI->getDebugLoc();
  if (isImmMskBitp(Value)) {
    int N = llvm::bit_width(Value);
    return BuildMI(MBB, MI, dl, get(XCore::MKMSK_rus), Reg)
        .addImm(N)
        .getInstr();
  }
  if (isImmU16(Value)) {
    int Opcode = isImmU6(Value) ? XCore::LDC_ru6 : XCore::LDC_lru6;
    return BuildMI(MBB, MI, dl, get(Opcode), Reg).addImm(Value).getInstr();
  }
  MachineConstantPool *ConstantPool = MBB.getParent()->getConstantPool();
```
- **EN**: Implements logic around `loadImmediate`, `getDebugLoc`, `bit_width`, `BuildMI`, ...; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `loadImmediate`, `getDebugLoc`, `bit_width`, `BuildMI`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 439-445
```cpp
  const Constant *C = ConstantInt::get(
        Type::getInt32Ty(MBB.getParent()->getFunction().getContext()), Value);
  unsigned Idx = ConstantPool->getConstantPoolIndex(C, Align(4));
  return BuildMI(MBB, MI, dl, get(XCore::LDWCP_lru6), Reg)
      .addConstantPoolIndex(Idx)
      .getInstr();
}
```
- **EN**: Implements logic around `get`, `getInt32Ty`, `getConstantPoolIndex`, `BuildMI`, ...; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `get`, `getInt32Ty`, `getConstantPoolIndex`, `BuildMI`, ... 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

## Key Concepts / 关键概念

- **Instruction semantics / 指令语义**:
  - **EN**: Encodes instruction behavior and helper routines
  - **CN**: 描述指令行为与辅助例程
- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `XCoreInstrInfo.h`, `XCore.h`, `XCoreSubtarget.h`, `llvm/CodeGen/MachineConstantPool.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineMemOperand.h`, `llvm/IR/Constants.h`, `llvm/IR/Function.h`, `llvm/MC/MCContext.h`, `llvm/Support/ErrorHandling.h`, `XCoreGenInstrInfo.inc`
- **LLVM subsystems / LLVM 子系统**: MC, CodeGen, IR, Support
- **Generated macros / 生成宏**: `GET_INSTRINFO_CTOR_DTOR`
