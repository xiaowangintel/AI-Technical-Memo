# PPCBranchSelector.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCBranchSelector.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides backend implementation logic for the PowerPC backend.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCBranchSelector.cpp`，主要负责 PowerPC 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- PPCBranchSelector.cpp - Emit long conditional branches ------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-14

```cpp
//===----------------------------------------------------------------------===//
//
// This file contains a pass that scans a machine function to determine which
// conditional branches need more than 16 bits of displacement to reach their
// target basic block.  It does this in two passes; a calculation of basic block
// positions pass, and a branch pseudo op to machine branch opcode pass.  This
// pass should be run last, just before the assembly printer.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This file contains a pass that scans a machine function to determine which".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This file contains a pass that scans a machine function to determine which”。

### Lines 15-21

```cpp
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/PPCPredicates.h"
#include "PPC.h"
#include "PPCInstrInfo.h"
#include "PPCSubtarget.h"
#include "llvm/ADT/Statistic.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 22-29

```cpp
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/TargetSubtargetInfo.h"
#include "llvm/Support/MathExtras.h"
#include "llvm/Target/TargetMachine.h"
#include <algorithm>
using namespace llvm;
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Register assignment and register-class constraints matter here.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 30-36

```cpp
#define DEBUG_TYPE "ppc-branch-select"

STATISTIC(NumExpanded, "Number of branches expanded to long format");
STATISTIC(NumPrefixed, "Number of prefixed instructions");
STATISTIC(NumPrefixedAligned,
          "Number of prefixed instructions that have been aligned");
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Notable symbols in this range include `STATISTIC`.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 该区间中较显眼的符号包括 `STATISTIC`。

### Lines 37-74

```cpp
namespace {
  struct PPCBSel : public MachineFunctionPass {
    static char ID;
    PPCBSel() : MachineFunctionPass(ID) {}

    // The sizes of the basic blocks in the function (the first
    // element of the pair); the second element of the pair is the amount of the
    // size that is due to potential padding.
    std::vector<std::pair<unsigned, unsigned>> BlockSizes;

    // The first block number which has imprecise instruction address.
    int FirstImpreciseBlock = -1;

    unsigned GetAlignmentAdjustment(MachineBasicBlock &MBB, unsigned Offset);
    unsigned ComputeBlockSizes(MachineFunction &Fn);
    void modifyAdjustment(MachineFunction &Fn);
    int computeBranchSize(MachineFunction &Fn,
                          const MachineBasicBlock *Src,
                          const MachineBasicBlock *Dest,
                          unsigned BrOffset);

    bool runOnMachineFunction(MachineFunction &Fn) override;

    MachineFunctionProperties getRequiredProperties() const override {
      return MachineFunctionProperties().setNoVRegs();
    }

    StringRef getPassName() const override { return "PowerPC Branch Selector"; }
  };
  char PPCBSel::ID = 0;
}

INITIALIZE_PASS(PPCBSel, "ppc-branch-select", "PowerPC Branch Selector",
                false, false)

/// createPPCBranchSelectionPass - returns an instance of the Branch Selection
/// Pass
///
```
- **EN**: Implements helper routine(s) `PPCBSel`, `MachineFunctionPass`, `function` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `PPCBSel`, `MachineFunctionPass`, `function`。

### Lines 75-112

```cpp
FunctionPass *llvm::createPPCBranchSelectionPass() {
  return new PPCBSel();
}

/// In order to make MBB aligned, we need to add an adjustment value to the
/// original Offset.
unsigned PPCBSel::GetAlignmentAdjustment(MachineBasicBlock &MBB,
                                         unsigned Offset) {
  const Align Alignment = MBB.getAlignment();
  if (Alignment == Align(1))
    return 0;

  const Align ParentAlign = MBB.getParent()->getAlignment();

  if (Alignment <= ParentAlign)
    return offsetToAlignment(Offset, Alignment);

  // The alignment of this MBB is larger than the function's alignment, so we
  // can't tell whether or not it will insert nops. Assume that it will.
  if (FirstImpreciseBlock < 0)
    FirstImpreciseBlock = MBB.getNumber();
  return Alignment.value() + offsetToAlignment(Offset, Alignment);
}

/// We need to be careful about the offset of the first block in the function
/// because it might not have the function's alignment. This happens because,
/// under the ELFv2 ABI, for functions which require a TOC pointer, we add a
/// two-instruction sequence to the start of the function.
/// Note: This needs to be synchronized with the check in
/// PPCLinuxAsmPrinter::EmitFunctionBodyStart.
static inline unsigned GetInitialOffset(MachineFunction &Fn) {
  unsigned InitialOffset = 0;
  if (Fn.getSubtarget<PPCSubtarget>().isELFv2ABI() &&
      !Fn.getRegInfo().use_empty(PPC::X2))
    InitialOffset = 8;
  return InitialOffset;
}
```
- **EN**: Implements helper routine(s) `createPPCBranchSelectionPass`, `PPCBSel`, `GetAlignmentAdjustment` for this portion of the PowerPC backend backend implementation logic. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `createPPCBranchSelectionPass`, `PPCBSel`, `GetAlignmentAdjustment`。 子目标特性裁剪会影响这里的行为。

### Lines 113-150

```cpp
/// Measure each MBB and compute a size for the entire function.
unsigned PPCBSel::ComputeBlockSizes(MachineFunction &Fn) {
  const PPCInstrInfo *TII =
      static_cast<const PPCInstrInfo *>(Fn.getSubtarget().getInstrInfo());
  unsigned FuncSize = GetInitialOffset(Fn);

  for (MachineBasicBlock &MBB : Fn) {
    // The end of the previous block may have extra nops if this block has an
    // alignment requirement.
    if (MBB.getNumber() > 0) {
      unsigned AlignExtra = GetAlignmentAdjustment(MBB, FuncSize);

      auto &BS = BlockSizes[MBB.getNumber()-1];
      BS.first += AlignExtra;
      BS.second = AlignExtra;

      FuncSize += AlignExtra;
    }

    unsigned BlockSize = 0;
    unsigned UnalignedBytesRemaining = 0;
    for (MachineInstr &MI : MBB) {
      unsigned MINumBytes = TII->getInstSizeInBytes(MI);
      if (MI.isInlineAsm() && (FirstImpreciseBlock < 0))
        FirstImpreciseBlock = MBB.getNumber();
      if (TII->isPrefixed(MI.getOpcode())) {
        NumPrefixed++;

        // All 8 byte instructions may require alignment. Each 8 byte
        // instruction may be aligned by another 4 bytes.
        // This means that an 8 byte instruction may require 12 bytes
        // (8 for the instruction itself and 4 for the alignment nop).
        // This will happen if an 8 byte instruction can be aligned to 64 bytes
        // by only adding a 4 byte nop.
        // We don't know the alignment at this point in the code so we have to
        // adopt a more pessimistic approach. If an instruction may need
        // alignment we assume that it does need alignment and add 4 bytes to
        // it. As a result we may end up with more long branches than before
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Measure each MBB and compute a size for the entire function.". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Measure each MBB and compute a size for the entire function.”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 151-188

```cpp
        // but we are in the safe position where if we need a long branch we
        // have one.
        // The if statement checks to make sure that two 8 byte instructions
        // are at least 64 bytes away from each other. It is not possible for
        // two instructions that both need alignment to be within 64 bytes of
        // each other.
        if (!UnalignedBytesRemaining) {
          BlockSize += 4;
          UnalignedBytesRemaining = 60;
          NumPrefixedAligned++;
        }
      }
      UnalignedBytesRemaining -= std::min(UnalignedBytesRemaining, MINumBytes);
      BlockSize += MINumBytes;
    }

    BlockSizes[MBB.getNumber()].first = BlockSize;
    FuncSize += BlockSize;
  }

  return FuncSize;
}

/// Modify the basic block align adjustment.
void PPCBSel::modifyAdjustment(MachineFunction &Fn) {
  unsigned Offset = GetInitialOffset(Fn);
  for (MachineBasicBlock &MBB : Fn) {
    if (MBB.getNumber() > 0) {
      auto &BS = BlockSizes[MBB.getNumber()-1];
      BS.first -= BS.second;
      Offset -= BS.second;

      unsigned AlignExtra = GetAlignmentAdjustment(MBB, Offset);

      BS.first += AlignExtra;
      BS.second = AlignExtra;

      Offset += AlignExtra;
```
- **EN**: Implements helper routine(s) `min`, `getNumber`, `modifyAdjustment` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `min`, `getNumber`, `modifyAdjustment`。

### Lines 189-226

```cpp
    }

    Offset += BlockSizes[MBB.getNumber()].first;
  }
}

/// Determine the offset from the branch in Src block to the Dest block.
/// BrOffset is the offset of the branch instruction inside Src block.
int PPCBSel::computeBranchSize(MachineFunction &Fn,
                               const MachineBasicBlock *Src,
                               const MachineBasicBlock *Dest,
                               unsigned BrOffset) {
  int BranchSize;
  Align MaxAlign = Align(4);
  bool NeedExtraAdjustment = false;
  if (Dest->getNumber() <= Src->getNumber()) {
    // If this is a backwards branch, the delta is the offset from the
    // start of this block to this branch, plus the sizes of all blocks
    // from this block to the dest.
    BranchSize = BrOffset;
    MaxAlign = std::max(MaxAlign, Src->getAlignment());

    int DestBlock = Dest->getNumber();
    BranchSize += BlockSizes[DestBlock].first;
    for (unsigned i = DestBlock+1, e = Src->getNumber(); i < e; ++i) {
      BranchSize += BlockSizes[i].first;
      MaxAlign = std::max(MaxAlign, Fn.getBlockNumbered(i)->getAlignment());
    }

    NeedExtraAdjustment = (FirstImpreciseBlock >= 0) &&
                          (DestBlock >= FirstImpreciseBlock);
  } else {
    // Otherwise, add the size of the blocks between this block and the
    // dest to the number of bytes left in this block.
    unsigned StartBlock = Src->getNumber();
    BranchSize = BlockSizes[StartBlock].first - BrOffset;

    MaxAlign = std::max(MaxAlign, Dest->getAlignment());
```
- **EN**: Implements helper routine(s) `getNumber`, `computeBranchSize`, `Align` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `getNumber`, `computeBranchSize`, `Align`。

### Lines 227-264

```cpp
    for (unsigned i = StartBlock+1, e = Dest->getNumber(); i != e; ++i) {
      BranchSize += BlockSizes[i].first;
      MaxAlign = std::max(MaxAlign, Fn.getBlockNumbered(i)->getAlignment());
    }

    NeedExtraAdjustment = (FirstImpreciseBlock >= 0) &&
                          (Src->getNumber() >= FirstImpreciseBlock);
  }

  // We tend to over estimate code size due to large alignment and
  // inline assembly. Usually it causes larger computed branch offset.
  // But sometimes it may also causes smaller computed branch offset
  // than actual branch offset. If the offset is close to the limit of
  // encoding, it may cause problem at run time.
  // Following is a simplified example.
  //
  //              actual        estimated
  //              address        address
  //    ...
  //   bne Far      100            10c
  //   .p2align 4
  //   Near:        110            110
  //    ...
  //   Far:        8108           8108
  //
  //   Actual offset:    0x8108 - 0x100 = 0x8008
  //   Computed offset:  0x8108 - 0x10c = 0x7ffc
  //
  // This example also shows when we can get the largest gap between
  // estimated offset and actual offset. If there is an aligned block
  // ABB between branch and target, assume its alignment is <align>
  // bits. Now consider the accumulated function size FSIZE till the end
  // of previous block PBB. If the estimated FSIZE is multiple of
  // 2^<align>, we don't need any padding for the estimated address of
  // ABB. If actual FSIZE at the end of PBB is 4 bytes more than
  // multiple of 2^<align>, then we need (2^<align> - 4) bytes of
  // padding. It also means the actual branch offset is (2^<align> - 4)
  // larger than computed offset. Other actual FSIZE needs less padding
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "We tend to over estimate code size due to large alignment and". Notable symbols in this range include `getNumber`, `max`, `getBlockNumbered`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“We tend to over estimate code size due to large alignment and”。 该区间中较显眼的符号包括 `getNumber`, `max`, `getBlockNumbered`。

### Lines 265-302

```cpp
  // bytes, so causes smaller gap between actual and computed offset.
  //
  // On the other hand, if the inline asm or large alignment occurs
  // between the branch block and destination block, the estimated address
  // can be <delta> larger than actual address. If padding bytes are
  // needed for a later aligned block, the actual number of padding bytes
  // is at most <delta> more than estimated padding bytes. So the actual
  // aligned block address is less than or equal to the estimated aligned
  // block address. So the actual branch offset is less than or equal to
  // computed branch offset.
  //
  // The computed offset is at most ((1 << alignment) - 4) bytes smaller
  // than actual offset. So we add this number to the offset for safety.
  if (NeedExtraAdjustment)
    BranchSize += MaxAlign.value() - 4;

  return BranchSize;
}

bool PPCBSel::runOnMachineFunction(MachineFunction &Fn) {
  const PPCInstrInfo *TII =
      static_cast<const PPCInstrInfo *>(Fn.getSubtarget().getInstrInfo());
  // Give the blocks of the function a dense, in-order, numbering.
  Fn.RenumberBlocks();
  BlockSizes.resize(Fn.getNumBlockIDs());
  FirstImpreciseBlock = -1;

  // Measure each MBB and compute a size for the entire function.
  unsigned FuncSize = ComputeBlockSizes(Fn);

  // If the entire function is smaller than the displacement of a branch field,
  // we know we don't need to shrink any branches in this function.  This is a
  // common case.
  if (FuncSize < (1 << 15)) {
    BlockSizes.clear();
    return false;
  }
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "bytes, so causes smaller gap between actual and computed offset.". Subtarget feature gating influences the behavior here.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“bytes, so causes smaller gap between actual and computed offset.”。 子目标特性裁剪会影响这里的行为。

### Lines 303-340

```cpp
  // For each conditional branch, if the offset to its destination is larger
  // than the offset field allows, transform it into a long branch sequence
  // like this:
  //   short branch:
  //     bCC MBB
  //   long branch:
  //     b!CC $PC+8
  //     b MBB
  //
  bool MadeChange = true;
  bool EverMadeChange = false;
  while (MadeChange) {
    // Iteratively expand branches until we reach a fixed point.
    MadeChange = false;

    for (MachineFunction::iterator MFI = Fn.begin(), E = Fn.end(); MFI != E;
         ++MFI) {
      MachineBasicBlock &MBB = *MFI;
      unsigned MBBStartOffset = 0;
      for (MachineBasicBlock::iterator I = MBB.begin(), E = MBB.end();
           I != E; ++I) {
        MachineBasicBlock *Dest = nullptr;
        if (I->getOpcode() == PPC::BCC && !I->getOperand(2).isImm())
          Dest = I->getOperand(2).getMBB();
        else if ((I->getOpcode() == PPC::BC || I->getOpcode() == PPC::BCn) &&
                 !I->getOperand(1).isImm())
          Dest = I->getOperand(1).getMBB();
        else if ((I->getOpcode() == PPC::BDNZ8 || I->getOpcode() == PPC::BDNZ ||
                  I->getOpcode() == PPC::BDZ8  || I->getOpcode() == PPC::BDZ) &&
                 !I->getOperand(0).isImm())
          Dest = I->getOperand(0).getMBB();

        if (!Dest) {
          MBBStartOffset += TII->getInstSizeInBytes(*I);
          continue;
        }

        // Determine the offset from the current branch to the destination
```
- **EN**: Implements helper routine(s) `begin`, `end`, `getOpcode` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `begin`, `end`, `getOpcode`。

### Lines 341-378

```cpp
        // block.
        int BranchSize = computeBranchSize(Fn, &MBB, Dest, MBBStartOffset);

        // If this branch is in range, ignore it.
        if (isInt<16>(BranchSize)) {
          MBBStartOffset += 4;
          continue;
        }

        // Otherwise, we have to expand it to a long branch.
        MachineInstr &OldBranch = *I;
        DebugLoc dl = OldBranch.getDebugLoc();

        if (I->getOpcode() == PPC::BCC) {
          // The BCC operands are:
          // 0. PPC branch predicate
          // 1. CR register
          // 2. Target MBB
          PPC::Predicate Pred = (PPC::Predicate)I->getOperand(0).getImm();
          Register CRReg = I->getOperand(1).getReg();

          // Jump over the uncond branch inst (i.e. $PC+8) on opposite condition.
          BuildMI(MBB, I, dl, TII->get(PPC::BCC))
            .addImm(PPC::InvertPredicate(Pred)).addReg(CRReg).addImm(2);
        } else if (I->getOpcode() == PPC::BC) {
          Register CRBit = I->getOperand(0).getReg();
          BuildMI(MBB, I, dl, TII->get(PPC::BCn)).addReg(CRBit).addImm(2);
        } else if (I->getOpcode() == PPC::BCn) {
          Register CRBit = I->getOperand(0).getReg();
          BuildMI(MBB, I, dl, TII->get(PPC::BC)).addReg(CRBit).addImm(2);
        } else if (I->getOpcode() == PPC::BDNZ) {
          BuildMI(MBB, I, dl, TII->get(PPC::BDZ)).addImm(2);
        } else if (I->getOpcode() == PPC::BDNZ8) {
          BuildMI(MBB, I, dl, TII->get(PPC::BDZ8)).addImm(2);
        } else if (I->getOpcode() == PPC::BDZ) {
          BuildMI(MBB, I, dl, TII->get(PPC::BDNZ)).addImm(2);
        } else if (I->getOpcode() == PPC::BDZ8) {
          BuildMI(MBB, I, dl, TII->get(PPC::BDNZ8)).addImm(2);
```
- **EN**: Implements helper routine(s) `computeBranchSize`, `getDebugLoc`, `getOpcode` for this portion of the PowerPC backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `computeBranchSize`, `getDebugLoc`, `getOpcode`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 379-409

```cpp
        } else {
           llvm_unreachable("Unhandled branch type!");
        }

        // Uncond branch to the real destination.
        I = BuildMI(MBB, I, dl, TII->get(PPC::B)).addMBB(Dest);

        // Remove the old branch from the function.
        OldBranch.eraseFromParent();

        // Remember that this instruction is 8-bytes, increase the size of the
        // block by 4, remember to iterate.
        BlockSizes[MBB.getNumber()].first += 4;
        MBBStartOffset += 8;
        ++NumExpanded;
        MadeChange = true;
      }
    }

    if (MadeChange) {
      // If we're going to iterate again, make sure we've updated our
      // padding-based contributions to the block sizes.
      modifyAdjustment(Fn);
    }

    EverMadeChange |= MadeChange;
  }

  BlockSizes.clear();
  return EverMadeChange;
}
```
- **EN**: Implements helper routine(s) `llvm_unreachable`, `BuildMI`, `get` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `llvm_unreachable`, `BuildMI`, `get`。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- Backend implementation logic / 后端实现逻辑
- Register modeling / 寄存器建模
- Subtarget features / 子目标特性
- Assembly or MC emission / 汇编或 MC 发射
- Instruction selection or opcode handling / 指令选择或操作码处理
- Target machine configuration / 目标机器配置
- Object format integration / 目标文件格式集成

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `MCTargetDesc/PPCPredicates.h`
- `PPC.h`
- `PPCInstrInfo.h`
- `PPCSubtarget.h`
- `llvm/ADT/Statistic.h`
- `llvm/CodeGen/MachineFunctionPass.h`
- `llvm/CodeGen/MachineRegisterInfo.h`
- `llvm/CodeGen/TargetSubtargetInfo.h`
- `llvm/Support/MathExtras.h`
- `llvm/Target/TargetMachine.h`
- `algorithm`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM Support utilities / LLVM Support 工具层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
