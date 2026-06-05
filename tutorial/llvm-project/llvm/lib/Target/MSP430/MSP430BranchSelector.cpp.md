# MSP430BranchSelector.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/MSP430/MSP430BranchSelector.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements a backend pass that relaxes or rewrites branches after block layout is known.
  - **CN**: 实现一种后端 Pass，在基本块布局已知后放宽或重写分支。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- MSP430BranchSelector.cpp - Emit long conditional branches ---------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-15
```cpp
//
// This file contains a pass that scans a machine function to determine which
// conditional branches need more than 10 bits of displacement to reach their
// target basic block.  It does this in two passes; a calculation of basic block
// positions pass, and a branch pseudo op to machine branch opcode pass.  This
// pass should be run last, just before the assembly printer.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 16-27
```cpp

#include "MSP430.h"
#include "MSP430InstrInfo.h"
#include "MSP430Subtarget.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/MathExtras.h"
#include "llvm/Target/TargetMachine.h"
using namespace llvm;

```
- **EN**: Pulls in the headers needed for this implementation, including `MSP430.h`, `MSP430InstrInfo.h`, `MSP430Subtarget.h`, `llvm/ADT/Statistic.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `MSP430.h`, `MSP430InstrInfo.h`, `MSP430Subtarget.h`, `llvm/ADT/Statistic.h`。

### Lines 28-36
```cpp
#define DEBUG_TYPE "msp430-branch-select"

static cl::opt<bool>
    BranchSelectEnabled("msp430-branch-select", cl::Hidden, cl::init(true),
                        cl::desc("Expand out of range branches"));

STATISTIC(NumSplit, "Number of machine basic blocks split");
STATISTIC(NumExpanded, "Number of branches expanded to long format");

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 37-44
```cpp
namespace {
class MSP430BSel : public MachineFunctionPass {

  typedef SmallVector<int, 16> OffsetVector;

  MachineFunction *MF;
  const MSP430InstrInfo *TII;

```
- **EN**: Introduces declarations for `MSP430BSel`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `MSP430BSel` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 45-52
```cpp
  unsigned measureFunction(OffsetVector &BlockOffsets,
                           MachineBasicBlock *FromBB = nullptr);
  bool expandBranches(OffsetVector &BlockOffsets);

public:
  static char ID;
  MSP430BSel() : MachineFunctionPass(ID) {}

```
- **EN**: Implements logic around `measureFunction`, `expandBranches`, `MSP430BSel`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `measureFunction`, `expandBranches`, `MSP430BSel` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 53-63
```cpp
  bool runOnMachineFunction(MachineFunction &MF) override;

  MachineFunctionProperties getRequiredProperties() const override {
    return MachineFunctionProperties().setNoVRegs();
  }

  StringRef getPassName() const override { return "MSP430 Branch Selector"; }
};
char MSP430BSel::ID = 0;
}

```
- **EN**: Implements logic around `runOnMachineFunction`, `getRequiredProperties`, `MachineFunctionProperties`, `getPassName`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `runOnMachineFunction`, `getRequiredProperties`, `MachineFunctionProperties`, `getPassName` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 64-70
```cpp
static bool isInRage(int DistanceInBytes) {
  // According to CC430 Family User's Guide, Section 4.5.1.3, branch
  // instructions have the signed 10-bit word offset field, so first we need to
  // convert the distance from bytes to words, then check if it fits in 10-bit
  // signed integer.
  const int WordSize = 2;

```
- **EN**: Implements logic around `isInRage`.
- **CN**: 围绕 `isInRage` 实现具体逻辑。

### Lines 71-77
```cpp
  assert((DistanceInBytes % WordSize == 0) &&
         "Branch offset should be word aligned!");

  int Words = DistanceInBytes / WordSize;
  return isInt<10>(Words);
}

```
- **EN**: Implements logic around `assert`, `isInt<10>`; this block returns target-specific results.
- **CN**: 围绕 `assert`, `isInt<10>` 实现具体逻辑；这一段返回目标相关结果。

### Lines 78-84
```cpp
/// Measure each basic block, fill the BlockOffsets, and return the size of
/// the function, starting with BB
unsigned MSP430BSel::measureFunction(OffsetVector &BlockOffsets,
                                     MachineBasicBlock *FromBB) {
  // Give the blocks of the function a dense, in-order, numbering.
  MF->RenumberBlocks(FromBB);

```
- **EN**: Implements logic around `measureFunction`, `RenumberBlocks`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `measureFunction`, `RenumberBlocks` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 85-91
```cpp
  MachineFunction::iterator Begin;
  if (FromBB == nullptr) {
    Begin = MF->begin();
  } else {
    Begin = FromBB->getIterator();
  }

```
- **EN**: Implements logic around `begin`, `getIterator`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `begin`, `getIterator` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 92-103
```cpp
  BlockOffsets.resize(MF->getNumBlockIDs());

  unsigned TotalSize = BlockOffsets[Begin->getNumber()];
  for (auto &MBB : make_range(Begin, MF->end())) {
    BlockOffsets[MBB.getNumber()] = TotalSize;
    for (MachineInstr &MI : MBB) {
      TotalSize += TII->getInstSizeInBytes(MI);
    }
  }
  return TotalSize;
}

```
- **EN**: Implements logic around `resize`, `getNumber`, `getInstSizeInBytes`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `resize`, `getNumber`, `getInstSizeInBytes` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 104-117
```cpp
/// Do expand branches and split the basic blocks if necessary.
/// Returns true if made any change.
bool MSP430BSel::expandBranches(OffsetVector &BlockOffsets) {
  // For each conditional branch, if the offset to its destination is larger
  // than the offset field allows, transform it into a long branch sequence
  // like this:
  //   short branch:
  //     bCC MBB
  //   long branch:
  //     b!CC $PC+6
  //     b MBB
  //
  bool MadeChange = false;
  for (auto MBB = MF->begin(), E = MF->end(); MBB != E; ++MBB) {
```
- **EN**: Implements logic around `expandBranches`.
- **CN**: 围绕 `expandBranches` 实现具体逻辑。

### Lines 118-126
```cpp
    unsigned MBBStartOffset = 0;
    for (auto MI = MBB->begin(), EE = MBB->end(); MI != EE; ++MI) {
      MBBStartOffset += TII->getInstSizeInBytes(*MI);

      // If this instruction is not a short branch then skip it.
      if (MI->getOpcode() != MSP430::JCC && MI->getOpcode() != MSP430::JMP) {
        continue;
      }

```
- **EN**: Implements logic around `getInstSizeInBytes`; this block applies conditional target rules.
- **CN**: 围绕 `getInstSizeInBytes` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 127-134
```cpp
      MachineBasicBlock *DestBB = MI->getOperand(0).getMBB();
      // Determine the distance from the current branch to the destination
      // block. MBBStartOffset already includes the size of the current branch
      // instruction.
      int BlockDistance =
          BlockOffsets[DestBB->getNumber()] - BlockOffsets[MBB->getNumber()];
      int BranchDistance = BlockDistance - MBBStartOffset;

```
- **EN**: Implements logic around `getOperand`, `getNumber`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getOperand`, `getNumber` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 135-143
```cpp
      // If this branch is in range, ignore it.
      if (isInRage(BranchDistance)) {
        continue;
      }

      LLVM_DEBUG(dbgs() << "  Found a branch that needs expanding, "
                        << printMBBReference(*DestBB) << ", Distance "
                        << BranchDistance << "\n");

```
- **EN**: Implements logic around `printMBBReference`; this block applies conditional target rules.
- **CN**: 围绕 `printMBBReference` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 144-154
```cpp
      // If JCC is not the last instruction we need to split the MBB.
      if (MI->getOpcode() == MSP430::JCC && std::next(MI) != EE) {

        LLVM_DEBUG(dbgs() << "  Found a basic block that needs to be split, "
                          << printMBBReference(*MBB) << "\n");

        // Create a new basic block.
        MachineBasicBlock *NewBB =
            MF->CreateMachineBasicBlock(MBB->getBasicBlock());
        MF->insert(std::next(MBB), NewBB);

```
- **EN**: Implements logic around `printMBBReference`, `CreateMachineBasicBlock`, `insert`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `printMBBReference`, `CreateMachineBasicBlock`, `insert` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 155-166
```cpp
        // Splice the instructions following MI over to the NewBB.
        NewBB->splice(NewBB->end(), &*MBB, std::next(MI), MBB->end());

        // Update the successor lists.
        for (MachineBasicBlock *Succ : MBB->successors()) {
          if (Succ == DestBB) {
            continue;
          }
          MBB->replaceSuccessor(Succ, NewBB);
          NewBB->addSuccessor(Succ);
        }

```
- **EN**: Implements logic around `splice`, `replaceSuccessor`, `addSuccessor`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `splice`, `replaceSuccessor`, `addSuccessor` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 167-177
```cpp
        // We introduced a new MBB so all following blocks should be numbered
        // and measured again.
        measureFunction(BlockOffsets, &*MBB);

        ++NumSplit;

        // It may be not necessary to start all over at this point, but it's
        // safer do this anyway.
        return true;
      }

```
- **EN**: Implements logic around `measureFunction`; this block returns target-specific results.
- **CN**: 围绕 `measureFunction` 实现具体逻辑；这一段返回目标相关结果。

### Lines 178-186
```cpp
      MachineInstr &OldBranch = *MI;
      DebugLoc dl = OldBranch.getDebugLoc();
      int InstrSizeDiff = -TII->getInstSizeInBytes(OldBranch);

      if (MI->getOpcode() == MSP430::JCC) {
        MachineBasicBlock *NextMBB = &*std::next(MBB);
        assert(MBB->isSuccessor(NextMBB) &&
               "This block must have a layout successor!");

```
- **EN**: Implements logic around `getDebugLoc`, `getInstSizeInBytes`, `next`, `assert`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getDebugLoc`, `getInstSizeInBytes`, `next`, `assert` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 187-200
```cpp
        // The BCC operands are:
        // 0. Target MBB
        // 1. MSP430 branch predicate
        SmallVector<MachineOperand, 1> Cond;
        Cond.push_back(MI->getOperand(1));

        // Jump over the long branch on the opposite condition
        TII->reverseBranchCondition(Cond);
        MI = BuildMI(*MBB, MI, dl, TII->get(MSP430::JCC))
                 .addMBB(NextMBB)
                 .add(Cond[0]);
        InstrSizeDiff += TII->getInstSizeInBytes(*MI);
        ++MI;
      }
```
- **EN**: Implements logic around `push_back`, `reverseBranchCondition`, `BuildMI`, `addMBB`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `push_back`, `reverseBranchCondition`, `BuildMI`, `addMBB`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 201-208
```cpp

      // Unconditional branch to the real destination.
      MI = BuildMI(*MBB, MI, dl, TII->get(MSP430::Bi)).addMBB(DestBB);
      InstrSizeDiff += TII->getInstSizeInBytes(*MI);

      // Remove the old branch from the function.
      OldBranch.eraseFromParent();

```
- **EN**: Implements logic around `BuildMI`, `getInstSizeInBytes`, `eraseFromParent`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `getInstSizeInBytes`, `eraseFromParent` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 209-215
```cpp
      // The size of a new instruction is different from the old one, so we need
      // to correct all block offsets.
      for (int i = MBB->getNumber() + 1, e = BlockOffsets.size(); i < e; ++i) {
        BlockOffsets[i] += InstrSizeDiff;
      }
      MBBStartOffset += InstrSizeDiff;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 216-222
```cpp
      ++NumExpanded;
      MadeChange = true;
    }
  }
  return MadeChange;
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 223-230
```cpp
bool MSP430BSel::runOnMachineFunction(MachineFunction &mf) {
  MF = &mf;
  TII = static_cast<const MSP430InstrInfo *>(MF->getSubtarget().getInstrInfo());

  // If the pass is disabled, just bail early.
  if (!BranchSelectEnabled)
    return false;

```
- **EN**: Implements logic around `runOnMachineFunction`, `getSubtarget`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `runOnMachineFunction`, `getSubtarget` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 231-244
```cpp
  LLVM_DEBUG(dbgs() << "\n********** " << getPassName() << " **********\n");

  // BlockOffsets - Contains the distance from the beginning of the function to
  // the beginning of each basic block.
  OffsetVector BlockOffsets;

  unsigned FunctionSize = measureFunction(BlockOffsets);
  // If the entire function is smaller than the displacement of a branch field,
  // we know we don't need to expand any branches in this
  // function. This is a common case.
  if (isInRage(FunctionSize)) {
    return false;
  }

```
- **EN**: Implements logic around `measureFunction`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `measureFunction` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 245-252
```cpp
  // Iteratively expand branches until we reach a fixed point.
  bool MadeChange = false;
  while (expandBranches(BlockOffsets))
    MadeChange = true;

  return MadeChange;
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 253-256
```cpp
/// Returns an instance of the Branch Selection Pass
FunctionPass *llvm::createMSP430BranchSelectionPass() {
  return new MSP430BSel();
}
```
- **EN**: Implements logic around `createMSP430BranchSelectionPass`, `MSP430BSel`; this block returns target-specific results.
- **CN**: 围绕 `createMSP430BranchSelectionPass`, `MSP430BSel` 实现具体逻辑；这一段返回目标相关结果。

## Key Concepts / 关键概念

- **Branch relaxation / 分支放宽**:
  - **EN**: Adjusts branches after layout to satisfy range or encoding limits
  - **CN**: 在布局完成后调整分支以满足范围或编码限制
- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `MSP430.h`, `MSP430InstrInfo.h`, `MSP430Subtarget.h`, `llvm/ADT/Statistic.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/Support/Debug.h`, `llvm/Support/MathExtras.h`, `llvm/Target/TargetMachine.h`
- **LLVM subsystems / LLVM 子系统**: CodeGen, Support
