# AArch64CompressJumpTables.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/AArch64CompressJumpTables.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This file covers Compress jump tables for AArch64. / 该文件实现 AArch64 后端中的AArch64 后端逻辑。
## Line-by-Line Analysis / 逐行分析
### Lines 1-26: Documented code section
```cpp
//==-- AArch64CompressJumpTables.cpp - Compress jump tables for AArch64 --====//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
// This pass looks at the basic blocks each jump-table refers to and works out
// whether they can be emitted in a compressed form (with 8 or 16-bit
// entries). If so, it changes the opcode and flags them in the associated
// AArch64FunctionInfo.
//
//===----------------------------------------------------------------------===//

#include "AArch64.h"
#include "AArch64MachineFunctionInfo.h"
#include "AArch64Subtarget.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineJumpTableInfo.h"
#include "llvm/CodeGen/TargetInstrInfo.h"
#include "llvm/CodeGen/TargetSubtargetInfo.h"
#include "llvm/Support/Alignment.h"

using namespace llvm;

#define DEBUG_TYPE "aarch64-jump-tables"
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 27-50: Core AArch64 backend logic
```cpp

STATISTIC(NumJT8, "Number of jump-tables with 1-byte entries");
STATISTIC(NumJT16, "Number of jump-tables with 2-byte entries");
STATISTIC(NumJT32, "Number of jump-tables with 4-byte entries");

namespace {
class AArch64CompressJumpTablesImpl {
  const TargetInstrInfo *TII = nullptr;
  MachineFunction *MF = nullptr;
  SmallVector<int, 8> BlockInfo;

  /// Returns the size of instructions in the block \p MBB, or std::nullopt if
  /// we couldn't get a safe upper bound.
  std::optional<int> computeBlockSize(MachineBasicBlock &MBB);

  /// Gather information about the function, returns false if we can't perform
  /// this optimization for some reason.
  bool scanFunction();

  bool compressJumpTable(MachineInstr &MI, int Offset);

public:
  bool run(MachineFunction &MF);
};
```
**EN:** This block continues the file's main AArch64 backend logic logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的AArch64 后端逻辑主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 51-70: Class AArch64CompressJumpTablesLegacy
```cpp

class AArch64CompressJumpTablesLegacy : public MachineFunctionPass {
public:
  static char ID;
  AArch64CompressJumpTablesLegacy() : MachineFunctionPass(ID) {}

  bool runOnMachineFunction(MachineFunction &MF) override;

  MachineFunctionProperties getRequiredProperties() const override {
    return MachineFunctionProperties().setNoVRegs();
  }
  StringRef getPassName() const override {
    return "AArch64 Compress Jump Tables";
  }
};
char AArch64CompressJumpTablesLegacy::ID = 0;
} // namespace

INITIALIZE_PASS(AArch64CompressJumpTablesLegacy, DEBUG_TYPE,
                "AArch64 compress jump tables pass", false, false)
```
**EN:** This block defines AArch64CompressJumpTablesLegacy, packaging state and behavior that the file reuses for AArch64 backend logic.  
**CN:** 该代码块定义 AArch64CompressJumpTablesLegacy，把 AArch64 后端逻辑 所需的状态与行为封装在一起供后续复用。
### Lines 71-91: Function AArch64CompressJumpTablesImpl::computeBlockSize
```cpp

std::optional<int>
AArch64CompressJumpTablesImpl::computeBlockSize(MachineBasicBlock &MBB) {
  int Size = 0;
  for (const MachineInstr &MI : MBB) {
    // Inline asm may contain some directives like .bytes which we don't
    // currently have the ability to parse accurately. To be safe, just avoid
    // computing a size and bail out.
    if (MI.getOpcode() == AArch64::INLINEASM ||
        MI.getOpcode() == AArch64::INLINEASM_BR)
      return std::nullopt;
    Size += TII->getInstSizeInBytes(MI);
  }
  return Size;
}

bool AArch64CompressJumpTablesImpl::scanFunction() {
  BlockInfo.clear();
  BlockInfo.resize(MF->getNumBlockIDs());

  // NOTE: BlockSize, Offset, OffsetAfterAlignment are all upper bounds.
```
**EN:** This block implements AArch64CompressJumpTablesImpl::computeBlockSize, advancing the file's AArch64 backend logic flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 AArch64CompressJumpTablesImpl::computeBlockSize，通过 AArch64 专用的决策与数据处理推进本文件的AArch64 后端逻辑流程。
### Lines 92-116: Core AArch64 backend logic
```cpp

  unsigned Offset = 0;
  for (MachineBasicBlock &MBB : *MF) {
    const Align Alignment = MBB.getAlignment();
    unsigned OffsetAfterAlignment = Offset;
    // We don't know the exact size of MBB so assume worse case padding.
    if (Alignment > Align(4))
      OffsetAfterAlignment += Alignment.value() - 4;
    BlockInfo[MBB.getNumber()] = OffsetAfterAlignment;
    auto BlockSize = computeBlockSize(MBB);
    if (!BlockSize)
      return false;
    Offset = OffsetAfterAlignment + *BlockSize;
  }
  return true;
}

bool AArch64CompressJumpTablesImpl::compressJumpTable(MachineInstr &MI,
                                                      int Offset) {
  if (MI.getOpcode() != AArch64::JumpTableDest32)
    return false;

  int JTIdx = MI.getOperand(4).getIndex();
  auto &JTInfo = *MF->getJumpTableInfo();
  const MachineJumpTableEntry &JT = JTInfo.getJumpTables()[JTIdx];
```
**EN:** This block continues the file's main AArch64 backend logic logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的AArch64 后端逻辑主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 117-142: Documented code section
```cpp

  // The jump-table might have been optimized away.
  if (JT.MBBs.empty())
    return false;

  int MaxOffset = std::numeric_limits<int>::min(),
      MinOffset = std::numeric_limits<int>::max();
  MachineBasicBlock *MinBlock = nullptr;
  for (auto *Block : JT.MBBs) {
    int BlockOffset = BlockInfo[Block->getNumber()];
    assert(BlockOffset % 4 == 0 && "misaligned basic block");

    MaxOffset = std::max(MaxOffset, BlockOffset);
    if (BlockOffset <= MinOffset) {
      MinOffset = BlockOffset;
      MinBlock = Block;
    }
  }
  assert(MinBlock && "Failed to find minimum offset block");

  // The ADR instruction needed to calculate the address of the first reachable
  // basic block can address +/-1MB.
  if (!isInt<21>(MinOffset - Offset)) {
    ++NumJT32;
    return false;
  }
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 143-168: Core AArch64 backend logic
```cpp

  int Span = MaxOffset - MinOffset;
  auto *AFI = MF->getInfo<AArch64FunctionInfo>();
  if (isUInt<8>(Span / 4)) {
    AFI->setJumpTableEntryInfo(JTIdx, 1, MinBlock->getSymbol());
    MI.setDesc(TII->get(AArch64::JumpTableDest8));
    ++NumJT8;
    return true;
  }
  if (isUInt<16>(Span / 4)) {
    AFI->setJumpTableEntryInfo(JTIdx, 2, MinBlock->getSymbol());
    MI.setDesc(TII->get(AArch64::JumpTableDest16));
    ++NumJT16;
    return true;
  }

  ++NumJT32;
  return false;
}

bool AArch64CompressJumpTablesImpl::run(MachineFunction &MFIn) {
  bool Changed = false;
  MF = &MFIn;

  const auto &ST = MF->getSubtarget<AArch64Subtarget>();
  TII = ST.getInstrInfo();
```
**EN:** This block continues the file's main AArch64 backend logic logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的AArch64 后端逻辑主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 169-190: Core AArch64 backend logic
```cpp

  if (ST.force32BitJumpTables() && !MF->getFunction().hasMinSize())
    return false;

  if (!scanFunction())
    return false;

  for (MachineBasicBlock &MBB : *MF) {
    int Offset = BlockInfo[MBB.getNumber()];
    for (MachineInstr &MI : MBB) {
      Changed |= compressJumpTable(MI, Offset);
      Offset += TII->getInstSizeInBytes(MI);
    }
  }

  return Changed;
}

bool AArch64CompressJumpTablesLegacy::runOnMachineFunction(
    MachineFunction &MF) {
  return AArch64CompressJumpTablesImpl().run(MF);
}
```
**EN:** This block continues the file's main AArch64 backend logic logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的AArch64 后端逻辑主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 191-205: Function AArch64CompressJumpTablesPass::run
```cpp

PreservedAnalyses
AArch64CompressJumpTablesPass::run(MachineFunction &MF,
                                   MachineFunctionAnalysisManager &MFAM) {
  const bool Changed = AArch64CompressJumpTablesImpl().run(MF);
  if (!Changed)
    return PreservedAnalyses::all();
  PreservedAnalyses PA = getMachineFunctionPassPreservedAnalyses();
  PA.preserveSet<CFGAnalyses>();
  return PA;
}

FunctionPass *llvm::createAArch64CompressJumpTablesPass() {
  return new AArch64CompressJumpTablesLegacy();
}
```
**EN:** This block implements AArch64CompressJumpTablesPass::run, advancing the file's AArch64 backend logic flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 AArch64CompressJumpTablesPass::run，通过 AArch64 专用的决策与数据处理推进本文件的AArch64 后端逻辑流程。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
## Dependencies / 依赖关系
- **EN:** Target-local includes: AArch64.h, AArch64MachineFunctionInfo.h, AArch64Subtarget.h **CN:** 目标本地依赖：AArch64.h, AArch64MachineFunctionInfo.h, AArch64Subtarget.h
- **EN:** Core LLVM interfaces: llvm/ADT/Statistic.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineJumpTableInfo.h, llvm/CodeGen/TargetInstrInfo.h, llvm/CodeGen/TargetSubtargetInfo.h, llvm/Support/Alignment.h **CN:** 核心 LLVM 接口：llvm/ADT/Statistic.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineJumpTableInfo.h, llvm/CodeGen/TargetInstrInfo.h, llvm/CodeGen/TargetSubtargetInfo.h, llvm/Support/Alignment.h
- **EN:** Closely connected with neighboring AArch64 backend components responsible for AArch64 backend logic. **CN:** 与周边负责AArch64 后端逻辑的 AArch64 后端组件紧密协作。
