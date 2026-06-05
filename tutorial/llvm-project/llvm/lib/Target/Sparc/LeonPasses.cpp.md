# LeonPasses.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Sparc/LeonPasses.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Defines SPARC/LEON-specific optimization or cleanup passes and their registration glue.
  - **CN**: 定义 SPARC/LEON 专用优化或清理 Pass 及其注册胶水代码。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10
```cpp
//===------ LeonPasses.cpp - Define passes specific to LEON ---------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 11-19
```cpp

#include "LeonPasses.h"
#include "SparcSubtarget.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/Support/raw_ostream.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `LeonPasses.h`, `SparcSubtarget.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineFunction.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `LeonPasses.h`, `SparcSubtarget.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineFunction.h`。

### Lines 20-30
```cpp
using namespace llvm;

char ErrataWorkaround::ID = 0;

ErrataWorkaround::ErrataWorkaround() : MachineFunctionPass(ID) {
  initializeErrataWorkaroundPass(*PassRegistry::getPassRegistry());
}

INITIALIZE_PASS(ErrataWorkaround, "errata-workaround", "Errata workaround pass",
                false, false)

```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 31-40
```cpp
// Move iterator to the next instruction in the function, ignoring
// meta instructions and inline assembly. Returns false when reaching
// the end of the function.
bool ErrataWorkaround::moveNext(MachineBasicBlock::iterator &I) {

  MachineBasicBlock *MBB = I->getParent();

  do {
    I++;

```
- **EN**: Implements logic around `moveNext`, `getParent`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `moveNext`, `getParent` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 41-51
```cpp
    while (I == MBB->end()) {
      if (MBB->getFallThrough() == nullptr)
        return false;
      MBB = MBB->getFallThrough();
      I = MBB->begin();
    }
  } while (I->isMetaInstruction() || I->isInlineAsm());

  return true;
}

```
- **EN**: Implements logic around `getFallThrough`, `begin`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getFallThrough`, `begin` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 52-62
```cpp
void ErrataWorkaround::insertNop(MachineBasicBlock::iterator I) {
  BuildMI(*I->getParent(), I, I->getDebugLoc(), TII->get(SP::NOP));
}

bool ErrataWorkaround::isFloat(MachineBasicBlock::iterator I) {
  if (I->getNumOperands() == 0)
    return false;

  if (!I->getOperand(0).isReg())
    return false;

```
- **EN**: Implements logic around `insertNop`, `BuildMI`, `isFloat`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `insertNop`, `BuildMI`, `isFloat` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 63-80
```cpp
  unsigned reg = I->getOperand(0).getReg();

  if (!SP::FPRegsRegClass.contains(reg) && !SP::DFPRegsRegClass.contains(reg))
    return false;

  return true;
}

bool ErrataWorkaround::isDivSqrt(MachineBasicBlock::iterator I) {
  switch (I->getOpcode()) {
  case SP::FDIVS:
  case SP::FDIVD:
  case SP::FSQRTS:
  case SP::FSQRTD:
    return true;
  }
  return false;
}
```
- **EN**: Implements logic around `getOperand`, `isDivSqrt`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getOperand`, `isDivSqrt` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 81-98
```cpp

// Prevents the following code sequence from being generated:
// (stb/sth/st/stf) -> (single non-store/load instruction) -> (any store)
// If the sequence is detected a NOP instruction is inserted after
// the first store instruction.
bool ErrataWorkaround::checkSeqTN0009A(MachineBasicBlock::iterator I) {
  switch (I->getOpcode()) {
  case SP::STrr:
  case SP::STri:
  case SP::STBrr:
  case SP::STBri:
  case SP::STHrr:
  case SP::STHri:
  case SP::STFrr:
  case SP::STFri:
    break;
  default:
    return false;
```
- **EN**: Implements logic around `checkSeqTN0009A`; this block uses `switch`-based dispatch; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `checkSeqTN0009A` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 99-107
```cpp
  }

  MachineBasicBlock::iterator MI = I;
  if (!moveNext(MI))
    return false;

  if (MI->mayStore() || MI->mayLoad())
    return false;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 108-119
```cpp
  MachineBasicBlock::iterator PatchHere = MI;

  if (!moveNext(MI))
    return false;

  if (!MI->mayStore())
    return false;

  insertNop(PatchHere);
  return true;
}

```
- **EN**: Implements logic around `insertNop`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `insertNop` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 120-135
```cpp
// Prevents the following code sequence from being generated:
// (std/stdf) -> (any store)
// If the sequence is detected a NOP instruction is inserted after
// the first store instruction.
bool ErrataWorkaround::checkSeqTN0009B(MachineBasicBlock::iterator I) {

  switch (I->getOpcode()) {
  case SP::STDrr:
  case SP::STDri:
  case SP::STDFrr:
  case SP::STDFri:
    break;
  default:
    return false;
  }

```
- **EN**: Implements logic around `checkSeqTN0009B`; this block uses `switch`-based dispatch; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `checkSeqTN0009B` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 136-147
```cpp
  MachineBasicBlock::iterator MI = I;

  if (!moveNext(MI))
    return false;

  if (!MI->mayStore())
    return false;

  insertNop(MI);
  return true;
}

```
- **EN**: Implements logic around `insertNop`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `insertNop` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 148-156
```cpp
// Insert a NOP at branch target if load in delay slot and atomic
// instruction at branch target. Also insert a NOP between load
// instruction and atomic instruction (swap or casa).
bool ErrataWorkaround::checkSeqTN0010(MachineBasicBlock::iterator I) {

  // Check for load instruction or branch bundled with load instruction
  if (!I->mayLoad())
    return false;

```
- **EN**: Implements logic around `checkSeqTN0010`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `checkSeqTN0010` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 157-167
```cpp
  // Check for branch to atomic instruction with load in delay slot
  if (I->isBranch()) {
    MachineBasicBlock *TargetMBB = I->getOperand(0).getMBB();
    MachineBasicBlock::iterator MI = TargetMBB->begin();

    while (MI != TargetMBB->end() && MI->isMetaInstruction())
      MI++;

    if (MI == TargetMBB->end())
      return false;

```
- **EN**: Implements logic around `getOperand`, `begin`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getOperand`, `begin` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 168-178
```cpp
    switch (MI->getOpcode()) {
    case SP::SWAPrr:
    case SP::SWAPri:
    case SP::CASArr:
      insertNop(MI);
      break;
    default:
      break;
    }
  }

```
- **EN**: Implements logic around `insertNop`; this block uses `switch`-based dispatch.
- **CN**: 围绕 `insertNop` 实现具体逻辑；这一段使用 `switch` 分派。

### Lines 179-195
```cpp
  // Check for load followed by atomic instruction
  MachineBasicBlock::iterator MI = I;
  if (!moveNext(MI))
    return false;

  switch (MI->getOpcode()) {
  case SP::SWAPrr:
  case SP::SWAPri:
  case SP::CASArr:
    break;
  default:
    return false;
  }
  insertNop(MI);
  return true;
}

```
- **EN**: Implements logic around `insertNop`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `insertNop` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 196-212
```cpp
// Do not allow functions to begin with an atomic instruction
bool ErrataWorkaround::checkSeqTN0010First(MachineBasicBlock &MBB) {
  MachineBasicBlock::iterator I = MBB.begin();
  while (I != MBB.end() && I->isMetaInstruction())
    I++;
  switch (I->getOpcode()) {
  case SP::SWAPrr:
  case SP::SWAPri:
  case SP::CASArr:
    break;
  default:
    return false;
  }
  insertNop(I);
  return true;
}

```
- **EN**: Implements logic around `checkSeqTN0010First`, `begin`, `insertNop`; this block uses `switch`-based dispatch; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `checkSeqTN0010First`, `begin`, `insertNop` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 213-222
```cpp
// Inserts a NOP instruction at the target of an integer branch if the
// target is a floating-point instruction or floating-point branch.
bool ErrataWorkaround::checkSeqTN0012(MachineBasicBlock::iterator I) {

  if (I->getOpcode() != SP::BCOND && I->getOpcode() != SP::BCONDA)
    return false;

  MachineBasicBlock *TargetMBB = I->getOperand(0).getMBB();
  MachineBasicBlock::iterator MI = TargetMBB->begin();

```
- **EN**: Implements logic around `checkSeqTN0012`, `getOperand`, `begin`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `checkSeqTN0012`, `getOperand`, `begin` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 223-231
```cpp
  while (MI != TargetMBB->end() && MI->isMetaInstruction())
    MI++;

  if (MI == TargetMBB->end())
    return false;

  if (!isFloat(MI) && MI->getOpcode() != SP::FBCOND)
    return false;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 232-246
```cpp
  insertNop(MI);
  return true;
}

// Prevents the following code sequence from being generated:
// (div/sqrt) -> (2 to 3 floating-point operations or loads) -> (div/sqrt)
// If the sequence is detected one or two NOP instruction are inserted after
// the first div/sqrt instruction. No NOPs are inserted if one of the floating-
// point instructions in the middle of the sequence is a (div/sqrt), or if
// they have dependency on the destination register of the first (div/sqrt).
//
// The function also prevents the following code sequence from being generated,
// (div/sqrt) -> (branch), by inserting a NOP instruction after the (div/sqrt).
bool ErrataWorkaround::checkSeqTN0013(MachineBasicBlock::iterator I) {

```
- **EN**: Implements logic around `insertNop`, `checkSeqTN0013`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `insertNop`, `checkSeqTN0013` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 247-255
```cpp
  if (!isDivSqrt(I))
    return false;

  unsigned dstReg = I->getOperand(0).getReg();

  MachineBasicBlock::iterator MI = I;
  if (!moveNext(MI))
    return false;

```
- **EN**: Implements logic around `getOperand`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 256-265
```cpp
  if (MI->isBranch()) {
    insertNop(MI);
    return true;
  }

  MachineBasicBlock::iterator PatchHere = MI;

  unsigned fpFound = 0;
  for (unsigned i = 0; i < 4; i++) {

```
- **EN**: Implements logic around `insertNop`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `insertNop` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 266-274
```cpp
    if (!isFloat(MI)) {
      if (!moveNext(MI))
        return false;
      continue;
    }

    if (MI->readsRegister(dstReg, TRI))
      return false;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 275-286
```cpp
    if (isDivSqrt(MI)) {
      if (i < 2)
        return false;
      if (fpFound < 2)
        return false;

      insertNop(PatchHere);
      if (i == 2)
        insertNop(PatchHere);
      return true;
    }

```
- **EN**: Implements logic around `insertNop`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `insertNop` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 287-298
```cpp
    fpFound++;
    if (!moveNext(MI))
      return false;
  }

  return false;
}

bool ErrataWorkaround::runOnMachineFunction(MachineFunction &MF) {
  bool Changed = false;
  ST = &MF.getSubtarget<SparcSubtarget>();

```
- **EN**: Implements logic around `runOnMachineFunction`, `getSubtarget<SparcSubtarget>`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `runOnMachineFunction`, `getSubtarget<SparcSubtarget>` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 299-308
```cpp
  if (!(ST->fixTN0009() || ST->fixTN0010() || ST->fixTN0012() ||
        ST->fixTN0013()))
    return false;

  TII = ST->getInstrInfo();
  TRI = ST->getRegisterInfo();

  if (ST->fixTN0010())
    Changed |= checkSeqTN0010First(MF.front());

```
- **EN**: Implements logic around `fixTN0013`, `getInstrInfo`, `getRegisterInfo`, `checkSeqTN0010First`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `fixTN0013`, `getInstrInfo`, `getRegisterInfo`, `checkSeqTN0010First` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 309-325
```cpp
  for (auto &MBB : MF) {
    for (auto &I : MBB) {
      if (ST->fixTN0009()) {
        Changed |= checkSeqTN0009A(I);
        Changed |= checkSeqTN0009B(I);
      }
      if (ST->fixTN0010())
        Changed |= checkSeqTN0010(I);
      if (ST->fixTN0012())
        Changed |= checkSeqTN0012(I);
      if (ST->fixTN0013())
        Changed |= checkSeqTN0013(I);
    }
  }
  return Changed;
}

```
- **EN**: Implements logic around `checkSeqTN0009A`, `checkSeqTN0009B`, `checkSeqTN0010`, `checkSeqTN0012`, ...; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `checkSeqTN0009A`, `checkSeqTN0009B`, `checkSeqTN0010`, `checkSeqTN0012`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 326-340
```cpp
LEONMachineFunctionPass::LEONMachineFunctionPass(char &ID)
    : MachineFunctionPass(ID) {}

//*****************************************************************************
//**** InsertNOPLoad pass
//*****************************************************************************
// This pass fixes the incorrectly working Load instructions that exists for
// some earlier versions of the LEON processor line. NOP instructions must
// be inserted after the load instruction to ensure that the Load instruction
// behaves as expected for these processors.
//
// This pass inserts a NOP after any LD or LDF instruction.
//
char InsertNOPLoad::ID = 0;

```
- **EN**: Implements logic around `LEONMachineFunctionPass`, `MachineFunctionPass`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `LEONMachineFunctionPass`, `MachineFunctionPass` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 341-350
```cpp
InsertNOPLoad::InsertNOPLoad() : LEONMachineFunctionPass(ID) {}

bool InsertNOPLoad::runOnMachineFunction(MachineFunction &MF) {
  Subtarget = &MF.getSubtarget<SparcSubtarget>();
  if (!Subtarget->insertNOPLoad())
    return false;

  const TargetInstrInfo &TII = *Subtarget->getInstrInfo();
  DebugLoc DL = DebugLoc();

```
- **EN**: Implements logic around `InsertNOPLoad`, `runOnMachineFunction`, `getSubtarget<SparcSubtarget>`, `getInstrInfo`, ...; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `InsertNOPLoad`, `runOnMachineFunction`, `getSubtarget<SparcSubtarget>`, `getInstrInfo`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 351-363
```cpp
  bool Modified = false;
  for (MachineBasicBlock &MBB : MF) {
    for (auto MBBI = MBB.begin(), E = MBB.end(); MBBI != E; ++MBBI) {
      MachineInstr &MI = *MBBI;
      unsigned Opcode = MI.getOpcode();
      if (Opcode >= SP::LDDArr && Opcode <= SP::LDrr) {
        MachineBasicBlock::iterator NMBBI = std::next(MBBI);
        BuildMI(MBB, NMBBI, DL, TII.get(SP::NOP));
        Modified = true;
      }
    }
  }

```
- **EN**: Implements logic around `getOpcode`, `next`, `BuildMI`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getOpcode`, `next`, `BuildMI` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 364-377
```cpp
  return Modified;
}



//*****************************************************************************
//**** DetectRoundChange pass
//*****************************************************************************
// To prevent any explicit change of the default rounding mode, this pass
// detects any call of the fesetround function.
// A warning is generated to ensure the user knows this has happened.
//
// Detects an erratum in UT699 LEON 3 processor

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 378-386
```cpp
char DetectRoundChange::ID = 0;

DetectRoundChange::DetectRoundChange() : LEONMachineFunctionPass(ID) {}

bool DetectRoundChange::runOnMachineFunction(MachineFunction &MF) {
  Subtarget = &MF.getSubtarget<SparcSubtarget>();
  if (!Subtarget->detectRoundChange())
    return false;

```
- **EN**: Implements logic around `DetectRoundChange`, `runOnMachineFunction`, `getSubtarget<SparcSubtarget>`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `DetectRoundChange`, `runOnMachineFunction`, `getSubtarget<SparcSubtarget>` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 387-404
```cpp
  bool Modified = false;
  for (MachineBasicBlock &MBB : MF) {
    for (MachineInstr &MI : MBB) {
      unsigned Opcode = MI.getOpcode();
      if (Opcode == SP::CALL && MI.getNumOperands() > 0) {
        MachineOperand &MO = MI.getOperand(0);

        if (MO.isGlobal()) {
          StringRef FuncName = MO.getGlobal()->getName();
          if (FuncName.compare_insensitive("fesetround") == 0) {
            errs() << "Error: You are using the detectroundchange "
                      "option to detect rounding changes that will "
                      "cause LEON errata. The only way to fix this "
                      "is to remove the call to fesetround from "
                      "the source code.\n";
          }
        }
      }
```
- **EN**: Implements logic around `getOpcode`, `getOperand`, `getGlobal`, `errs`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getOpcode`, `getOperand`, `getGlobal`, `errs` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 405-422
```cpp
    }
  }

  return Modified;
}

//*****************************************************************************
//**** FixAllFDIVSQRT pass
//*****************************************************************************
// This pass fixes the incorrectly working FDIVx and FSQRTx instructions that
// exist for some earlier versions of the LEON processor line. Five NOP
// instructions need to be inserted after these instructions to ensure the
// correct result is placed in the destination registers before they are used.
//
// This pass implements two fixes:
//  1) fixing the FSQRTS and FSQRTD instructions.
//  2) fixing the FDIVS and FDIVD instructions.
//
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 423-431
```cpp
// FSQRTS and FDIVS are converted to FDIVD and FSQRTD respectively earlier in
// the pipeline when this option is enabled, so this pass needs only to deal
// with the changes that still need implementing for the "double" versions
// of these instructions.
//
char FixAllFDIVSQRT::ID = 0;

FixAllFDIVSQRT::FixAllFDIVSQRT() : LEONMachineFunctionPass(ID) {}

```
- **EN**: Implements logic around `FixAllFDIVSQRT`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `FixAllFDIVSQRT` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 432-445
```cpp
bool FixAllFDIVSQRT::runOnMachineFunction(MachineFunction &MF) {
  Subtarget = &MF.getSubtarget<SparcSubtarget>();
  if (!Subtarget->fixAllFDIVSQRT())
    return false;

  const TargetInstrInfo &TII = *Subtarget->getInstrInfo();
  DebugLoc DL = DebugLoc();

  bool Modified = false;
  for (MachineBasicBlock &MBB : MF) {
    for (auto MBBI = MBB.begin(), E = MBB.end(); MBBI != E; ++MBBI) {
      MachineInstr &MI = *MBBI;
      unsigned Opcode = MI.getOpcode();

```
- **EN**: Implements logic around `runOnMachineFunction`, `getSubtarget<SparcSubtarget>`, `getInstrInfo`, `DebugLoc`, ...; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `runOnMachineFunction`, `getSubtarget<SparcSubtarget>`, `getInstrInfo`, `DebugLoc`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 446-457
```cpp
      // Note: FDIVS and FSQRTS cannot be generated when this erratum fix is
      // switched on so we don't need to check for them here. They will
      // already have been converted to FSQRTD or FDIVD earlier in the
      // pipeline.
      if (Opcode == SP::FSQRTD || Opcode == SP::FDIVD) {
        for (int InsertedCount = 0; InsertedCount < 5; InsertedCount++)
          BuildMI(MBB, MBBI, DL, TII.get(SP::NOP));

        MachineBasicBlock::iterator NMBBI = std::next(MBBI);
        for (int InsertedCount = 0; InsertedCount < 28; InsertedCount++)
          BuildMI(MBB, NMBBI, DL, TII.get(SP::NOP));

```
- **EN**: Implements logic around `BuildMI`, `next`; this block uses `switch`-based dispatch; applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `next` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 458-464
```cpp
        Modified = true;
      }
    }
  }

  return Modified;
}
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

## Key Concepts / 关键概念

- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `LeonPasses.h`, `SparcSubtarget.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/Support/raw_ostream.h`
- **LLVM subsystems / LLVM 子系统**: CodeGen, Support
