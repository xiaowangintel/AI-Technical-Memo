# RISCVMoveMerger.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVMoveMerger.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements merging compatible move and immediate-materialization sequences on RISC-V. / 实现合并 RISC-V 上兼容的移动与立即数物化序列。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Commentary and design intent / 注释与设计意图
```cpp
//===-- RISCVMoveMerger.cpp - RISC-V move merge pass ----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains a pass that performs move related peephole optimizations
// as Zcmp has specified. This pass should be run after register allocation.
//
// This pass also supports Xqccmp, which has identical instructions.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 15-28: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp

#include "RISCVInstrInfo.h"
#include "RISCVSubtarget.h"

using namespace llvm;

#define RISCV_MOVE_MERGE_NAME "RISC-V Zcmp move merging pass"

namespace {
struct RISCVMoveMerge : public MachineFunctionPass {
  static char ID;

  RISCVMoveMerge() : MachineFunctionPass(ID) {}
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 29-48: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  const RISCVSubtarget *ST;
  const RISCVInstrInfo *TII;
  const TargetRegisterInfo *TRI;

  // Track which register units have been modified and used.
  LiveRegUnits ModifiedRegUnits, UsedRegUnits;

  bool isGPRPairCopyCandidateEven(const DestSourcePair &RegPair);
  bool isGPRPairCopyCandidateOdd(const DestSourcePair &RegPair);

  bool isCandidateToMergeMVA01S(const DestSourcePair &RegPair);
  bool isCandidateToMergeMVSA01(const DestSourcePair &RegPair);
  // Merge the two instructions indicated into a single pair instruction.
  MachineBasicBlock::iterator
  mergeGPRPairInsns(MachineBasicBlock::iterator I,
                    MachineBasicBlock::iterator Paired, bool RegPairIsEven);
  MachineBasicBlock::iterator
  mergePairedInsns(MachineBasicBlock::iterator I,
                   MachineBasicBlock::iterator Paired, bool MoveFromSToA);
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 49-62: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  MachineBasicBlock::iterator
  findMatchingInstPair(MachineBasicBlock::iterator &MBBI, bool EvenRegPair,
                       const DestSourcePair &RegPair);
  // Look for C.MV instruction that can be combined with
  // the given instruction into CM.MVA01S or CM.MVSA01. Return the matching
  // instruction if one exists.
  MachineBasicBlock::iterator
  findMatchingInst(MachineBasicBlock::iterator &MBBI, bool MoveFromSToA,
                   const DestSourcePair &RegPair);
  bool mergeMoveSARegPair(MachineBasicBlock &MBB);
  bool runOnMachineFunction(MachineFunction &Fn) override;

  StringRef getPassName() const override { return RISCV_MOVE_MERGE_NAME; }
};
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 63-77: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp

char RISCVMoveMerge::ID = 0;

} // end of anonymous namespace

INITIALIZE_PASS(RISCVMoveMerge, "riscv-move-merge", RISCV_MOVE_MERGE_NAME,
                false, false)

static unsigned getGPRPairCopyOpcode(const RISCVSubtarget &ST) {
  if (ST.hasStdExtZdinx())
    return RISCV::FSGNJ_D_IN32X;

  if (ST.hasStdExtP())
    return RISCV::PADD_DW;
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 78-94: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  llvm_unreachable("Unhandled subtarget with paired move.");
}

static unsigned getCM_MVOpcode(const RISCVSubtarget &ST, bool MoveFromSToA) {
  if (ST.hasStdExtZcmp())
    return MoveFromSToA ? RISCV::CM_MVA01S : RISCV::CM_MVSA01;

  if (ST.hasVendorXqccmp())
    return MoveFromSToA ? RISCV::QC_CM_MVA01S : RISCV::QC_CM_MVSA01;

  llvm_unreachable("Unhandled subtarget with paired move.");
}

bool RISCVMoveMerge::isGPRPairCopyCandidateEven(const DestSourcePair &RegPair) {
  Register Destination = RegPair.Destination->getReg();
  Register Source = RegPair.Source->getReg();
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 95-111: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  if (Source == Destination)
    return false;

  if ((!ST->hasStdExtZdinx() && !ST->hasStdExtP()) || ST->is64Bit())
    return false;
  Register SrcPair = TRI->getMatchingSuperReg(Source, RISCV::sub_gpr_even,
                                              &RISCV::GPRPairRegClass);
  Register DestPair = TRI->getMatchingSuperReg(Destination, RISCV::sub_gpr_even,
                                               &RISCV::GPRPairRegClass);

  return SrcPair.isValid() && DestPair.isValid();
}

bool RISCVMoveMerge::isGPRPairCopyCandidateOdd(const DestSourcePair &RegPair) {
  Register Destination = RegPair.Destination->getReg();
  Register Source = RegPair.Source->getReg();
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 112-135: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  if (Source == Destination)
    return false;

  if ((!ST->hasStdExtZdinx() && !ST->hasStdExtP()) || ST->is64Bit())
    return false;
  Register SrcPair = TRI->getMatchingSuperReg(Source, RISCV::sub_gpr_odd,
                                              &RISCV::GPRPairRegClass);
  Register DestPair = TRI->getMatchingSuperReg(Destination, RISCV::sub_gpr_odd,
                                               &RISCV::GPRPairRegClass);

  return SrcPair.isValid() && DestPair.isValid();
}

// Check if registers meet CM.MVA01S constraints.
bool RISCVMoveMerge::isCandidateToMergeMVA01S(const DestSourcePair &RegPair) {
  Register Destination = RegPair.Destination->getReg();
  Register Source = RegPair.Source->getReg();
  // If destination is not a0 or a1.
  if ((ST->hasStdExtZcmp() || ST->hasVendorXqccmp()) &&
      (Destination == RISCV::X10 || Destination == RISCV::X11) &&
      RISCV::SR07RegClass.contains(Source))
    return true;
  return false;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 136-157: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

// Check if registers meet CM.MVSA01 constraints.
bool RISCVMoveMerge::isCandidateToMergeMVSA01(const DestSourcePair &RegPair) {
  Register Destination = RegPair.Destination->getReg();
  Register Source = RegPair.Source->getReg();
  // If Source is s0 - s7.
  if ((ST->hasStdExtZcmp() || ST->hasVendorXqccmp()) &&
      (Source == RISCV::X10 || Source == RISCV::X11) &&
      RISCV::SR07RegClass.contains(Destination))
    return true;
  return false;
}

MachineBasicBlock::iterator
RISCVMoveMerge::mergeGPRPairInsns(MachineBasicBlock::iterator I,
                                  MachineBasicBlock::iterator Paired,
                                  bool RegPairIsEven) {
  MachineBasicBlock::iterator E = I->getParent()->end();
  MachineBasicBlock::iterator NextI = next_nodbg(I, E);
  DestSourcePair FirstPair = TII->isCopyInstrImpl(*I).value();
  DestSourcePair SecondPair = TII->isCopyInstrImpl(*Paired).value();
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 158-179: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  if (NextI == Paired)
    NextI = next_nodbg(NextI, E);
  DebugLoc DL = I->getDebugLoc();

  // Make a copy of the second instruction to update the kill
  // flag.
  MachineOperand PairedSource = *SecondPair.Source;

  unsigned Opcode = getGPRPairCopyOpcode(*ST);
  for (auto It = std::next(I); It != Paired && PairedSource.isKill(); ++It)
    if (It->readsRegister(PairedSource.getReg(), TRI))
      PairedSource.setIsKill(false);

  Register SrcReg1, SrcReg2, DestReg;
  unsigned GPRPairIdx =
      RegPairIsEven ? RISCV::sub_gpr_even : RISCV::sub_gpr_odd;
  SrcReg1 = TRI->getMatchingSuperReg(FirstPair.Source->getReg(), GPRPairIdx,
                                     &RISCV::GPRPairRegClass);
  SrcReg2 = ST->hasStdExtZdinx() ? SrcReg1 : Register(RISCV::X0_Pair);
  DestReg = TRI->getMatchingSuperReg(FirstPair.Destination->getReg(),
                                     GPRPairIdx, &RISCV::GPRPairRegClass);
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 180-200: Function implementation: RISCVMoveMerge::mergePairedInsns / 函数实现：RISCVMoveMerge::mergePairedInsns
```cpp
  BuildMI(*I->getParent(), I, DL, TII->get(Opcode), DestReg)
      .addReg(SrcReg1, getKillRegState(PairedSource.isKill() &&
                                       FirstPair.Source->isKill()))
      .addReg(SrcReg2, getKillRegState(PairedSource.isKill() &&
                                       FirstPair.Source->isKill()));

  I->eraseFromParent();
  Paired->eraseFromParent();
  return NextI;
}

MachineBasicBlock::iterator
RISCVMoveMerge::mergePairedInsns(MachineBasicBlock::iterator I,
                                 MachineBasicBlock::iterator Paired,
                                 bool MoveFromSToA) {
  const MachineOperand *Sreg1, *Sreg2;
  MachineBasicBlock::iterator E = I->getParent()->end();
  MachineBasicBlock::iterator NextI = next_nodbg(I, E);
  DestSourcePair FirstPair = TII->isCopyInstrImpl(*I).value();
  DestSourcePair PairedRegs = TII->isCopyInstrImpl(*Paired).value();
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 201-226: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  if (NextI == Paired)
    NextI = next_nodbg(NextI, E);
  DebugLoc DL = I->getDebugLoc();

  // Make a copy so we can update the kill flag in the MoveFromSToA case. The
  // copied operand needs to be scoped outside the if since we make a pointer
  // to it.
  MachineOperand PairedSource = *PairedRegs.Source;

  // The order of S-reg depends on which instruction holds A0, instead of
  // the order of register pair.
  // e,g.
  //   mv a1, s1
  //   mv a0, s2    =>  cm.mva01s s2,s1
  //
  //   mv a0, s2
  //   mv a1, s1    =>  cm.mva01s s2,s1
  unsigned Opcode = getCM_MVOpcode(*ST, MoveFromSToA);
  if (MoveFromSToA) {
    // We are moving one of the copies earlier so its kill flag may become
    // invalid. Clear the copied kill flag if there are any reads of the
    // register between the new location and the old location.
    for (auto It = std::next(I); It != Paired && PairedSource.isKill(); ++It)
      if (It->readsRegister(PairedSource.getReg(), TRI))
        PairedSource.setIsKill(false);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 227-243: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    Sreg1 = FirstPair.Source;
    Sreg2 = &PairedSource;
    if (FirstPair.Destination->getReg() != RISCV::X10)
      std::swap(Sreg1, Sreg2);
  } else {
    Sreg1 = FirstPair.Destination;
    Sreg2 = PairedRegs.Destination;
    if (FirstPair.Source->getReg() != RISCV::X10)
      std::swap(Sreg1, Sreg2);
  }

  BuildMI(*I->getParent(), I, DL, TII->get(Opcode)).add(*Sreg1).add(*Sreg2);

  I->eraseFromParent();
  Paired->eraseFromParent();
  return NextI;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 244-257: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

MachineBasicBlock::iterator
RISCVMoveMerge::findMatchingInstPair(MachineBasicBlock::iterator &MBBI,
                                     bool EvenRegPair,
                                     const DestSourcePair &RegPair) {
  MachineBasicBlock::iterator E = MBBI->getParent()->end();
  ModifiedRegUnits.clear();
  UsedRegUnits.clear();

  for (MachineBasicBlock::iterator I = next_nodbg(MBBI, E); I != E;
       I = next_nodbg(I, E)) {

    MachineInstr &MI = *I;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 258-274: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    if (auto SecondPair = TII->isCopyInstrImpl(MI)) {
      Register SourceReg = SecondPair->Source->getReg();
      Register DestReg = SecondPair->Destination->getReg();

      if (RegPair.Destination->getReg() == DestReg ||
          RegPair.Source->getReg() == SourceReg)
        return E;

      unsigned RegPairIdx =
          EvenRegPair ? RISCV::sub_gpr_even : RISCV::sub_gpr_odd;
      unsigned SecondPairIdx =
          !EvenRegPair ? RISCV::sub_gpr_even : RISCV::sub_gpr_odd;

      // Get the register GPRPair.
      Register SrcGPRPair = TRI->getMatchingSuperReg(
          RegPair.Source->getReg(), RegPairIdx, &RISCV::GPRPairRegClass);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 275-288: Definitions and supporting logic / 定义与支撑逻辑
```cpp
      Register DestGPRPair = TRI->getMatchingSuperReg(
          RegPair.Destination->getReg(), RegPairIdx, &RISCV::GPRPairRegClass);

      // Check if the second pair's registers match the other lane of the
      // GPRPairs.
      if (SourceReg != TRI->getSubReg(SrcGPRPair, SecondPairIdx) ||
          DestReg != TRI->getSubReg(DestGPRPair, SecondPairIdx))
        return E;

      if (!ModifiedRegUnits.available(DestReg) ||
          !UsedRegUnits.available(DestReg) ||
          !ModifiedRegUnits.available(SourceReg))
        return E;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 289-302: Function implementation: LiveRegUnits::accumulateUsedDefed / 函数实现：LiveRegUnits::accumulateUsedDefed
```cpp
      return I;
    }
    // Update modified / used register units.
    LiveRegUnits::accumulateUsedDefed(MI, ModifiedRegUnits, UsedRegUnits, TRI);
  }
  return E;
}

MachineBasicBlock::iterator
RISCVMoveMerge::findMatchingInst(MachineBasicBlock::iterator &MBBI,
                                 bool MoveFromSToA,
                                 const DestSourcePair &RegPair) {
  MachineBasicBlock::iterator E = MBBI->getParent()->end();
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 303-316: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // Track which register units have been modified and used between the first
  // insn and the second insn.
  ModifiedRegUnits.clear();
  UsedRegUnits.clear();

  for (MachineBasicBlock::iterator I = next_nodbg(MBBI, E); I != E;
       I = next_nodbg(I, E)) {

    MachineInstr &MI = *I;

    if (auto SecondPair = TII->isCopyInstrImpl(MI)) {
      Register SourceReg = SecondPair->Source->getReg();
      Register DestReg = SecondPair->Destination->getReg();
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 317-335: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
      bool IsCandidate = MoveFromSToA ? isCandidateToMergeMVA01S(*SecondPair)
                                      : isCandidateToMergeMVSA01(*SecondPair);
      if (IsCandidate) {
        // Second destination must be different.
        if (RegPair.Destination->getReg() == DestReg)
          return E;

        // For AtoS the source must also be different.
        if (!MoveFromSToA && RegPair.Source->getReg() == SourceReg)
          return E;

        // If paired destination register was modified or used, the source reg
        // was modified, there is no possibility of finding matching
        // instruction so exit early.
        if (!ModifiedRegUnits.available(DestReg) ||
            !UsedRegUnits.available(DestReg) ||
            !ModifiedRegUnits.available(SourceReg))
          return E;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 336-349: Function implementation: LiveRegUnits::accumulateUsedDefed / 函数实现：LiveRegUnits::accumulateUsedDefed
```cpp
        return I;
      }
    }
    // Update modified / used register units.
    LiveRegUnits::accumulateUsedDefed(MI, ModifiedRegUnits, UsedRegUnits, TRI);
  }
  return E;
}

// Finds instructions, which could be represented as C.MV instructions and
// merged into CM.MVA01S or CM.MVSA01.
bool RISCVMoveMerge::mergeMoveSARegPair(MachineBasicBlock &MBB) {
  bool Modified = false;
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 350-363: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  for (MachineBasicBlock::iterator MBBI = MBB.begin(), E = MBB.end();
       MBBI != E;) {
    // Check if the instruction can be compressed to C.MV instruction. If it
    // can, return Dest/Src register pair.
    auto RegPair = TII->isCopyInstrImpl(*MBBI);
    if (RegPair.has_value()) {
      bool MoveFromSToA = isCandidateToMergeMVA01S(*RegPair);
      bool IsEven = isGPRPairCopyCandidateEven(*RegPair);
      bool IsOdd = isGPRPairCopyCandidateOdd(*RegPair);
      if (!MoveFromSToA && !isCandidateToMergeMVSA01(*RegPair) && !IsEven &&
          !IsOdd) {
        ++MBBI;
        continue;
      }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 364-380: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

      MachineBasicBlock::iterator Paired = E;
      if (ST->hasStdExtZcmp() || ST->hasVendorXqccmp()) {
        Paired = findMatchingInst(MBBI, MoveFromSToA, RegPair.value());
        if (Paired != E) {
          MBBI = mergePairedInsns(MBBI, Paired, MoveFromSToA);
          Modified = true;
          continue;
        }
      }
      if (IsEven != IsOdd) {
        Paired = findMatchingInstPair(MBBI, IsEven, RegPair.value());
        if (Paired != E) {
          MBBI = mergeGPRPairInsns(MBBI, Paired, IsEven);
          Modified = true;
          continue;
        }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 381-397: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
      }
    }
    ++MBBI;
  }
  return Modified;
}

bool RISCVMoveMerge::runOnMachineFunction(MachineFunction &Fn) {
  if (skipFunction(Fn.getFunction()))
    return false;

  ST = &Fn.getSubtarget<RISCVSubtarget>();
  bool HasGPRPairCopy =
      !ST->is64Bit() && (ST->hasStdExtZdinx() || ST->hasStdExtP());
  if (!ST->hasStdExtZcmp() && !ST->hasVendorXqccmp() && !HasGPRPairCopy)
    return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 398-413: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  TII = ST->getInstrInfo();
  TRI = ST->getRegisterInfo();
  // Resize the modified and used register unit trackers.  We do this once
  // per function and then clear the register units each time we optimize a
  // move.
  ModifiedRegUnits.init(*TRI);
  UsedRegUnits.init(*TRI);
  bool Modified = false;
  for (auto &MBB : Fn)
    Modified |= mergeMoveSARegPair(MBB);
  return Modified;
}

/// createRISCVMoveMergePass - returns an instance of the
/// move merge pass.
FunctionPass *llvm::createRISCVMoveMergePass() { return new RISCVMoveMerge(); }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

## Key Concepts / 关键概念
- **RISC-V backend structure** / **RISC-V 后端结构**

## Dependencies / 依赖关系
- `RISCVInstrInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVSubtarget.h` — Directly referenced by this file. / 该文件直接引用的依赖。
