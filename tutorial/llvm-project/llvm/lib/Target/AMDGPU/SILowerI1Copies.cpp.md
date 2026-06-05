# SILowerI1Copies.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/SILowerI1Copies.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements SILowerI1Copies for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 SILowerI1Copies 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-43: File banner, includes, and setup
```cpp
//===-- SILowerI1Copies.cpp - Lower I1 Copies -----------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass lowers all occurrences of i1 values (with a vreg_1 register class)
// to lane masks (32 / 64-bit scalar registers). The pass assumes machine SSA
// form and a wave-level control flow graph.
//
// Before this pass, values that are semantically i1 and are defined and used
// within the same basic block are already represented as lane masks in scalar
// registers. However, values that cross basic blocks are always transferred
// between basic blocks in vreg_1 virtual registers and are lowered by this
// pass.
//
// The only instructions that use or define vreg_1 virtual registers are COPY,
// PHI, and IMPLICIT_DEF.
//
//===----------------------------------------------------------------------===//

#include "SILowerI1Copies.h"
#include "AMDGPU.h"
#include "llvm/CodeGen/MachineIDFSSAUpdater.h"
#include "llvm/InitializePasses.h"

#define DEBUG_TYPE "si-i1-copies"

using namespace llvm;

static Register
insertUndefLaneMask(MachineBasicBlock *MBB, MachineRegisterInfo *MRI,
                    MachineRegisterInfo::VRegAttrs LaneMaskRegAttrs);

namespace {

class Vreg1LoweringHelper : public AMDGPU::PhiLoweringHelper {
public:
  Vreg1LoweringHelper(MachineFunction *MF, MachineDominatorTree *DT,
                      MachinePostDominatorTree *PDT);

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `Vreg1LoweringHelper`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`Vreg1LoweringHelper`。

### Lines 44-83: Declares markAsLaneMask
```cpp
private:
  DenseSet<Register> ConstrainRegs;

public:
  void markAsLaneMask(Register DstReg) const override;
  void getCandidatesForLowering(
      SmallVectorImpl<MachineInstr *> &Vreg1Phis) const override;
  void collectIncomingValuesFromPhi(
      const MachineInstr *MI,
      SmallVectorImpl<AMDGPU::Incoming> &Incomings) const override;
  void replaceDstReg(Register NewReg, Register OldReg,
                     MachineBasicBlock *MBB) override;
  void buildMergeLaneMasks(MachineBasicBlock &MBB,
                           MachineBasicBlock::iterator I, const DebugLoc &DL,
                           Register DstReg, Register PrevReg,
                           Register CurReg) override;
  void constrainAsLaneMask(AMDGPU::Incoming &In) override;

  bool lowerCopiesFromI1();
  bool lowerCopiesToI1();
  bool cleanConstrainRegs(bool Changed);
  bool isVreg1(Register Reg) const {
    return Reg.isVirtual() && MRI->getRegClass(Reg) == &AMDGPU::VReg_1RegClass;
  }
};

Vreg1LoweringHelper::Vreg1LoweringHelper(MachineFunction *MF,
                                         MachineDominatorTree *DT,
                                         MachinePostDominatorTree *PDT)
    : PhiLoweringHelper(MF, DT, PDT) {}

bool Vreg1LoweringHelper::cleanConstrainRegs(bool Changed) {
  assert(Changed || ConstrainRegs.empty());
  for (Register Reg : ConstrainRegs)
    MRI->constrainRegClass(Reg, TII->getRegisterInfo().getWaveMaskRegClass());
  ConstrainRegs.clear();

  return Changed;
}

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `Vreg1LoweringHelper::Vreg1LoweringHelper`, `Vreg1LoweringHelper::cleanConstrainRegs`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`Vreg1LoweringHelper::Vreg1LoweringHelper`, `Vreg1LoweringHelper::cleanConstrainRegs`。

### Lines 84-127: Declares class PhiIncomingAnalysis
```cpp
/// Helper class that determines the relationship between incoming values of a
/// phi in the control flow graph to determine where an incoming value can
/// simply be taken as a scalar lane mask as-is, and where it needs to be
/// merged with another, previously defined lane mask.
///
/// The approach is as follows:
///  - Determine all basic blocks which, starting from the incoming blocks,
///    a wave may reach before entering the def block (the block containing the
///    phi).
///  - If an incoming block has no predecessors in this set, we can take the
///    incoming value as a scalar lane mask as-is.
///  -- A special case of this is when the def block has a self-loop.
///  - Otherwise, the incoming value needs to be merged with a previously
///    defined lane mask.
///  - If there is a path into the set of reachable blocks that does _not_ go
///    through an incoming block where we can take the scalar lane mask as-is,
///    we need to invent an available value for the SSAUpdater. Choices are
///    0 and undef, with differing consequences for how to merge values etc.
///
/// TODO: We could use region analysis to quickly skip over SESE regions during
///       the traversal.
///
class PhiIncomingAnalysis {
  MachinePostDominatorTree &PDT;
  const SIInstrInfo *TII;

  // For each reachable basic block, whether it is a source in the induced
  // subgraph of the CFG.
  MapVector<MachineBasicBlock *, bool> ReachableMap;
  SmallVector<MachineBasicBlock *, 4> Stack;
  SmallVector<MachineBasicBlock *, 4> Predecessors;

public:
  PhiIncomingAnalysis(MachinePostDominatorTree &PDT, const SIInstrInfo *TII)
      : PDT(PDT), TII(TII) {}

  /// Returns whether \p MBB is a source in the induced subgraph of reachable
  /// blocks.
  bool isSource(MachineBasicBlock &MBB) const {
    return ReachableMap.find(&MBB)->second;
  }

  ArrayRef<MachineBasicBlock *> predecessors() const { return Predecessors; }

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `PhiIncomingAnalysis`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`PhiIncomingAnalysis`。

### Lines 128-158: Defines analyze
```cpp
  void analyze(MachineBasicBlock &DefBlock,
               ArrayRef<AMDGPU::Incoming> Incomings) {
    assert(Stack.empty());
    ReachableMap.clear();
    Predecessors.clear();

    // Insert the def block first, so that it acts as an end point for the
    // traversal.
    ReachableMap.try_emplace(&DefBlock, false);

    for (auto Incoming : Incomings) {
      MachineBasicBlock *MBB = Incoming.Block;
      if (MBB == &DefBlock) {
        ReachableMap[&DefBlock] = true; // self-loop on DefBlock
        continue;
      }

      ReachableMap.try_emplace(MBB, false);

      // If this block has a divergent terminator and the def block is its
      // post-dominator, the wave may first visit the other successors.
      if (TII->hasDivergentBranch(MBB) && PDT.dominates(&DefBlock, MBB))
        append_range(Stack, MBB->successors());
    }

    while (!Stack.empty()) {
      MachineBasicBlock *MBB = Stack.pop_back_val();
      if (ReachableMap.try_emplace(MBB, false).second)
        append_range(Stack, MBB->successors());
    }

```
**EN:** This section contains concrete logic for analyze. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 analyze 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 159-202: Conditional logic and checks
```cpp
    for (auto &[MBB, Reachable] : ReachableMap) {
      bool HaveReachablePred = false;
      for (MachineBasicBlock *Pred : MBB->predecessors()) {
        if (ReachableMap.count(Pred)) {
          HaveReachablePred = true;
        } else {
          Stack.push_back(Pred);
        }
      }
      if (!HaveReachablePred)
        Reachable = true;
      if (HaveReachablePred) {
        for (MachineBasicBlock *UnreachablePred : Stack) {
          if (!llvm::is_contained(Predecessors, UnreachablePred))
            Predecessors.push_back(UnreachablePred);
        }
      }
      Stack.clear();
    }
  }
};

/// Helper class that detects loops which require us to lower an i1 COPY into
/// bitwise manipulation.
///
/// Unfortunately, we cannot use LoopInfo because LoopInfo does not distinguish
/// between loops with the same header. Consider this example:
///
///  A-+-+
///  | | |
///  B-+ |
///  |   |
///  C---+
///
/// A is the header of a loop containing A, B, and C as far as LoopInfo is
/// concerned. However, an i1 COPY in B that is used in C must be lowered to
/// bitwise operations to combine results from different loop iterations when
/// B has a divergent branch (since by default we will compile this code such
/// that threads in a wave are merged at the entry of C).
///
/// The following rule is implemented to determine whether bitwise operations
/// are required: use the bitwise lowering for a def in block B if a backward
/// edge to B is reachable without going through the nearest common
/// post-dominator of B and all uses of the def.
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `llvm::is_contained`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`llvm::is_contained`。

### Lines 203-241: Declares class LoopFinder
```cpp
///
/// TODO: This rule is conservative because it does not check whether the
///       relevant branches are actually divergent.
///
/// The class is designed to cache the CFG traversal so that it can be re-used
/// for multiple defs within the same basic block.
///
/// TODO: We could use region analysis to quickly skip over SESE regions during
///       the traversal.
///
class LoopFinder {
  MachineDominatorTree &DT;
  MachinePostDominatorTree &PDT;

  // All visited / reachable block, tagged by level (level 0 is the def block,
  // level 1 are all blocks reachable including but not going through the def
  // block's IPDOM, etc.).
  DenseMap<MachineBasicBlock *, unsigned> Visited;

  // Nearest common dominator of all visited blocks by level (level 0 is the
  // def block). Used for seeding the SSAUpdater.
  SmallVector<MachineBasicBlock *, 4> CommonDominators;

  // Post-dominator of all visited blocks.
  MachineBasicBlock *VisitedPostDom = nullptr;

  // Level at which a loop was found: 0 is not possible; 1 = a backward edge is
  // reachable without going through the IPDOM of the def block (if the IPDOM
  // itself has an edge to the def block, the loop level is 2), etc.
  unsigned FoundLoopLevel = ~0u;

  MachineBasicBlock *DefBlock = nullptr;
  SmallVector<MachineBasicBlock *, 4> Stack;
  SmallVector<MachineBasicBlock *, 4> NextLevel;

public:
  LoopFinder(MachineDominatorTree &DT, MachinePostDominatorTree &PDT)
      : DT(DT), PDT(PDT) {}

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `LoopFinder`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`LoopFinder`。

### Lines 242-284: Defines initialize
```cpp
  void initialize(MachineBasicBlock &MBB) {
    Visited.clear();
    CommonDominators.clear();
    Stack.clear();
    NextLevel.clear();
    VisitedPostDom = nullptr;
    FoundLoopLevel = ~0u;

    DefBlock = &MBB;
  }

  /// Check whether a backward edge can be reached without going through the
  /// given \p PostDom of the def block.
  ///
  /// Return the level of \p PostDom if a loop was found, or 0 otherwise.
  unsigned findLoop(MachineBasicBlock *PostDom) {
    MachineDomTreeNode *PDNode = PDT.getNode(DefBlock);

    if (!VisitedPostDom)
      advanceLevel();

    unsigned Level = 0;
    while (PDNode->getBlock() != PostDom) {
      if (PDNode->getBlock() == VisitedPostDom)
        advanceLevel();
      PDNode = PDNode->getIDom();
      Level++;
      if (FoundLoopLevel == Level)
        return Level;
    }

    return 0;
  }

  /// Add undef values dominating the loop and the optionally given additional
  /// blocks, so that the SSA updater doesn't have to search all the way to the
  /// function entry.
  void addLoopEntries(unsigned LoopLevel, MachineIDFSSAUpdater &SSAUpdater,
                      MachineRegisterInfo &MRI,
                      MachineRegisterInfo::VRegAttrs LaneMaskRegAttrs,
                      ArrayRef<AMDGPU::Incoming> Incomings = {}) {
    assert(LoopLevel < CommonDominators.size());

```
**EN:** This section contains concrete logic for initialize. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 initialize 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 285-327: Conditional logic and checks
```cpp
    MachineBasicBlock *Dom = CommonDominators[LoopLevel];
    for (auto &Incoming : Incomings)
      Dom = DT.findNearestCommonDominator(Dom, Incoming.Block);

    if (!inLoopLevel(*Dom, LoopLevel, Incomings)) {
      SSAUpdater.addAvailableValue(
          Dom, insertUndefLaneMask(Dom, &MRI, LaneMaskRegAttrs));
    } else {
      // The dominator is part of the loop or the given blocks, so add the
      // undef value to unreachable predecessors instead.
      for (MachineBasicBlock *Pred : Dom->predecessors()) {
        if (!inLoopLevel(*Pred, LoopLevel, Incomings))
          SSAUpdater.addAvailableValue(
              Pred, insertUndefLaneMask(Pred, &MRI, LaneMaskRegAttrs));
      }
    }
  }

private:
  bool inLoopLevel(MachineBasicBlock &MBB, unsigned LoopLevel,
                   ArrayRef<AMDGPU::Incoming> Incomings) const {
    auto DomIt = Visited.find(&MBB);
    if (DomIt != Visited.end() && DomIt->second <= LoopLevel)
      return true;

    for (auto &Incoming : Incomings)
      if (Incoming.Block == &MBB)
        return true;

    return false;
  }

  void advanceLevel() {
    MachineBasicBlock *VisitedDom;

    if (!VisitedPostDom) {
      VisitedPostDom = DefBlock;
      VisitedDom = DefBlock;
      Stack.push_back(DefBlock);
    } else {
      VisitedPostDom = PDT.getNode(VisitedPostDom)->getIDom()->getBlock();
      VisitedDom = CommonDominators.back();

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 328-370: Conditional logic and checks
```cpp
      for (unsigned i = 0; i < NextLevel.size();) {
        if (PDT.dominates(VisitedPostDom, NextLevel[i])) {
          Stack.push_back(NextLevel[i]);

          NextLevel[i] = NextLevel.back();
          NextLevel.pop_back();
        } else {
          i++;
        }
      }
    }

    unsigned Level = CommonDominators.size();
    while (!Stack.empty()) {
      MachineBasicBlock *MBB = Stack.pop_back_val();
      if (!PDT.dominates(VisitedPostDom, MBB))
        NextLevel.push_back(MBB);

      Visited[MBB] = Level;
      VisitedDom = DT.findNearestCommonDominator(VisitedDom, MBB);

      for (MachineBasicBlock *Succ : MBB->successors()) {
        if (Succ == DefBlock) {
          if (MBB == VisitedPostDom)
            FoundLoopLevel = std::min(FoundLoopLevel, Level + 1);
          else
            FoundLoopLevel = std::min(FoundLoopLevel, Level);
          continue;
        }

        if (Visited.try_emplace(Succ, ~0u).second) {
          if (MBB == VisitedPostDom)
            NextLevel.push_back(Succ);
          else
            Stack.push_back(Succ);
        }
      }
    }

    CommonDominators.push_back(VisitedDom);
  }
};

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::min`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::min`。

### Lines 371-412: Preprocessor guards and macros
```cpp
} // End anonymous namespace.

Register llvm::AMDGPU::createLaneMaskReg(
    MachineRegisterInfo *MRI, MachineRegisterInfo::VRegAttrs LaneMaskRegAttrs) {
  return MRI->createVirtualRegister(LaneMaskRegAttrs);
}

static Register
insertUndefLaneMask(MachineBasicBlock *MBB, MachineRegisterInfo *MRI,
                    MachineRegisterInfo::VRegAttrs LaneMaskRegAttrs) {
  MachineFunction &MF = *MBB->getParent();
  const GCNSubtarget &ST = MF.getSubtarget<GCNSubtarget>();
  const SIInstrInfo *TII = ST.getInstrInfo();
  Register UndefReg = AMDGPU::createLaneMaskReg(MRI, LaneMaskRegAttrs);
  BuildMI(*MBB, MBB->getFirstTerminator(), {}, TII->get(AMDGPU::IMPLICIT_DEF),
          UndefReg);
  return UndefReg;
}

#ifndef NDEBUG
static bool isVRegCompatibleReg(const SIRegisterInfo &TRI,
                                const MachineRegisterInfo &MRI,
                                Register Reg) {
  unsigned Size = TRI.getRegSizeInBits(Reg, MRI);
  return Size == 1 || Size == 32;
}
#endif

bool Vreg1LoweringHelper::lowerCopiesFromI1() {
  bool Changed = false;
  SmallVector<MachineInstr *, 4> DeadCopies;

  for (MachineBasicBlock &MBB : *MF) {
    for (MachineInstr &MI : MBB) {
      if (MI.getOpcode() != AMDGPU::COPY)
        continue;

      Register DstReg = MI.getOperand(0).getReg();
      Register SrcReg = MI.getOperand(1).getReg();
      if (!isVreg1(SrcReg))
        continue;

```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `AMDGPU::createLaneMaskReg`, `Vreg1LoweringHelper::lowerCopiesFromI1`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`AMDGPU::createLaneMaskReg`, `Vreg1LoweringHelper::lowerCopiesFromI1`。

### Lines 413-451: Conditional logic and checks
```cpp
      if (isLaneMaskReg(DstReg) || isVreg1(DstReg))
        continue;

      Changed = true;

      // Copy into a 32-bit vector register.
      LLVM_DEBUG(dbgs() << "Lower copy from i1: " << MI);
      const DebugLoc &DL = MI.getDebugLoc();

      assert(isVRegCompatibleReg(TII->getRegisterInfo(), *MRI, DstReg));
      assert(!MI.getOperand(0).getSubReg());

      ConstrainRegs.insert(SrcReg);
      BuildMI(MBB, MI, DL, TII->get(AMDGPU::V_CNDMASK_B32_e64), DstReg)
          .addImm(0)
          .addImm(0)
          .addImm(0)
          .addImm(-1)
          .addReg(SrcReg);
      DeadCopies.push_back(&MI);
    }

    for (MachineInstr *MI : DeadCopies)
      MI->eraseFromParent();
    DeadCopies.clear();
  }
  return Changed;
}

AMDGPU::PhiLoweringHelper::PhiLoweringHelper(MachineFunction *MF,
                                             MachineDominatorTree *DT,
                                             MachinePostDominatorTree *PDT)
    : MF(MF), DT(DT), PDT(PDT), ST(&MF->getSubtarget<GCNSubtarget>()),
      LMC(&AMDGPU::LaneMaskConstants::get(*ST)) {
  MRI = &MF->getRegInfo();

  TII = ST->getInstrInfo();
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `PhiLoweringHelper::PhiLoweringHelper`, `LaneMaskConstants::get`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`PhiLoweringHelper::PhiLoweringHelper`, `LaneMaskConstants::get`。

### Lines 452-491: Preprocessor guards and macros
```cpp
bool AMDGPU::PhiLoweringHelper::lowerPhis() {
  LoopFinder LF(*DT, *PDT);
  PhiIncomingAnalysis PIA(*PDT, TII);
  SmallVector<MachineInstr *, 4> Vreg1Phis;
  SmallVector<Incoming, 4> Incomings;

  getCandidatesForLowering(Vreg1Phis);
  if (Vreg1Phis.empty())
    return false;

  DT->updateDFSNumbers();
  MachineBasicBlock *PrevMBB = nullptr;
  for (MachineInstr *MI : Vreg1Phis) {
    MachineBasicBlock &MBB = *MI->getParent();
    if (&MBB != PrevMBB) {
      LF.initialize(MBB);
      PrevMBB = &MBB;
    }

    LLVM_DEBUG(dbgs() << "Lower PHI: " << *MI);

    Register DstReg = MI->getOperand(0).getReg();
    markAsLaneMask(DstReg);
    initializeLaneMaskRegisterAttributes(DstReg);

    collectIncomingValuesFromPhi(MI, Incomings);

    // Sort the incomings such that incoming values that dominate other incoming
    // values are sorted earlier. This allows us to do some amount of on-the-fly
    // constant folding.
    // Incoming with smaller DFSNumIn goes first, DFSNumIn is 0 for entry block.
    llvm::sort(Incomings, [this](Incoming LHS, Incoming RHS) {
      return DT->getNode(LHS.Block)->getDFSNumIn() <
             DT->getNode(RHS.Block)->getDFSNumIn();
    });

#ifndef NDEBUG
    PhiRegisters.insert(DstReg);
#endif

```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `PhiLoweringHelper::lowerPhis`, `llvm::sort`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`PhiLoweringHelper::lowerPhis`, `llvm::sort`。

### Lines 492-532: Conditional logic and checks
```cpp
    // Phis in a loop that are observed outside the loop receive a simple but
    // conservatively correct treatment.
    std::vector<MachineBasicBlock *> DomBlocks = {&MBB};
    for (MachineInstr &Use : MRI->use_instructions(DstReg))
      DomBlocks.push_back(Use.getParent());

    MachineBasicBlock *PostDomBound =
        PDT->findNearestCommonDominator(DomBlocks);

    // FIXME: This fails to find irreducible cycles. If we have a def (other
    // than a constant) in a pair of blocks that end up looping back to each
    // other, it will be mishandle. Due to structurization this shouldn't occur
    // in practice.
    unsigned FoundLoopLevel = LF.findLoop(PostDomBound);

    MachineIDFSSAUpdater SSAUpdater(*DT, *MF, DstReg);
    SSAUpdater.addUseBlock(&MBB);

    if (FoundLoopLevel) {
      LF.addLoopEntries(FoundLoopLevel, SSAUpdater, *MRI, LaneMaskRegAttrs,
                        Incomings);

      for (auto &Incoming : Incomings) {
        SSAUpdater.addUseBlock(Incoming.Block);
        Incoming.UpdatedReg = createLaneMaskReg(MRI, LaneMaskRegAttrs);
        SSAUpdater.addAvailableValue(Incoming.Block, Incoming.UpdatedReg);
      }

      SSAUpdater.calculate();

      for (auto &Incoming : Incomings) {
        MachineBasicBlock &IMBB = *Incoming.Block;
        buildMergeLaneMasks(
            IMBB, getSaluInsertionAtEnd(IMBB), {}, Incoming.UpdatedReg,
            SSAUpdater.getValueInMiddleOfBlock(&IMBB), Incoming.Reg);
      }
    } else {
      // The phi is not observed from outside a loop. Use a more accurate
      // lowering.
      PIA.analyze(MBB, Incomings);

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 533-572: Conditional logic and checks
```cpp
      for (MachineBasicBlock *MBB : PIA.predecessors())
        SSAUpdater.addAvailableValue(
            MBB, insertUndefLaneMask(MBB, MRI, LaneMaskRegAttrs));

      for (auto &Incoming : Incomings) {
        MachineBasicBlock &IMBB = *Incoming.Block;
        if (PIA.isSource(IMBB)) {
          constrainAsLaneMask(Incoming);
          SSAUpdater.addAvailableValue(&IMBB, Incoming.Reg);
        } else {
          SSAUpdater.addUseBlock(&IMBB);
          Incoming.UpdatedReg = createLaneMaskReg(MRI, LaneMaskRegAttrs);
          SSAUpdater.addAvailableValue(&IMBB, Incoming.UpdatedReg);
        }
      }

      SSAUpdater.calculate();

      for (auto &Incoming : Incomings) {
        if (!Incoming.UpdatedReg.isValid())
          continue;

        MachineBasicBlock &IMBB = *Incoming.Block;
        buildMergeLaneMasks(
            IMBB, getSaluInsertionAtEnd(IMBB), {}, Incoming.UpdatedReg,
            SSAUpdater.getValueInMiddleOfBlock(&IMBB), Incoming.Reg);
      }
    }

    Register NewReg = SSAUpdater.getValueInMiddleOfBlock(&MBB);
    if (NewReg != DstReg) {
      replaceDstReg(NewReg, DstReg, &MBB);
      MI->eraseFromParent();
    }

    Incomings.clear();
  }
  return true;
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 573-608: Implements Vreg1LoweringHelper::lowerCopiesToI1
```cpp
bool Vreg1LoweringHelper::lowerCopiesToI1() {
  bool Changed = false;
  LoopFinder LF(*DT, *PDT);
  SmallVector<MachineInstr *, 4> DeadCopies;

  for (MachineBasicBlock &MBB : *MF) {
    LF.initialize(MBB);

    for (MachineInstr &MI : MBB) {
      if (MI.getOpcode() != AMDGPU::IMPLICIT_DEF &&
          MI.getOpcode() != AMDGPU::COPY)
        continue;

      Register DstReg = MI.getOperand(0).getReg();
      if (!isVreg1(DstReg))
        continue;

      Changed = true;

      if (MRI->use_empty(DstReg)) {
        DeadCopies.push_back(&MI);
        continue;
      }

      LLVM_DEBUG(dbgs() << "Lower Other: " << MI);

      markAsLaneMask(DstReg);
      initializeLaneMaskRegisterAttributes(DstReg);

      if (MI.getOpcode() == AMDGPU::IMPLICIT_DEF)
        continue;

      const DebugLoc &DL = MI.getDebugLoc();
      Register SrcReg = MI.getOperand(1).getReg();
      assert(!MI.getOperand(1).getSubReg());

```
**EN:** This section contains concrete logic for Vreg1LoweringHelper::lowerCopiesToI1. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `Vreg1LoweringHelper::lowerCopiesToI1`.
**CN:** 本节包含与 Vreg1LoweringHelper::lowerCopiesToI1 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`Vreg1LoweringHelper::lowerCopiesToI1`。

### Lines 609-650: Conditional logic and checks
```cpp
      if (!SrcReg.isVirtual() || (!isLaneMaskReg(SrcReg) && !isVreg1(SrcReg))) {
        assert(TII->getRegisterInfo().getRegSizeInBits(SrcReg, *MRI) == 32);
        Register TmpReg = AMDGPU::createLaneMaskReg(MRI, LaneMaskRegAttrs);
        BuildMI(MBB, MI, DL, TII->get(AMDGPU::V_CMP_NE_U32_e64), TmpReg)
            .addReg(SrcReg)
            .addImm(0);
        MI.getOperand(1).setReg(TmpReg);
        SrcReg = TmpReg;
      } else {
        // SrcReg needs to be live beyond copy.
        MI.getOperand(1).setIsKill(false);
      }

      // Defs in a loop that are observed outside the loop must be transformed
      // into appropriate bit manipulation.
      std::vector<MachineBasicBlock *> DomBlocks = {&MBB};
      for (MachineInstr &Use : MRI->use_instructions(DstReg))
        DomBlocks.push_back(Use.getParent());

      MachineBasicBlock *PostDomBound =
          PDT->findNearestCommonDominator(DomBlocks);
      unsigned FoundLoopLevel = LF.findLoop(PostDomBound);
      if (FoundLoopLevel) {
        MachineIDFSSAUpdater SSAUpdater(*DT, *MF, DstReg);
        SSAUpdater.addUseBlock(&MBB);
        SSAUpdater.addAvailableValue(&MBB, DstReg);
        LF.addLoopEntries(FoundLoopLevel, SSAUpdater, *MRI, LaneMaskRegAttrs);

        SSAUpdater.calculate();
        buildMergeLaneMasks(MBB, MI, DL, DstReg,
                            SSAUpdater.getValueInMiddleOfBlock(&MBB), SrcReg);
        DeadCopies.push_back(&MI);
      }
    }

    for (MachineInstr *MI : DeadCopies)
      MI->eraseFromParent();
    DeadCopies.clear();
  }
  return Changed;
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::createLaneMaskReg`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::createLaneMaskReg`。

### Lines 651-691: Implements PhiLoweringHelper::isConstantLaneMask
```cpp
bool AMDGPU::PhiLoweringHelper::isConstantLaneMask(Register Reg,
                                                   bool &Val) const {
  const MachineInstr *MI;
  for (;;) {
    MI = MRI->getUniqueVRegDef(Reg);
    if (MI->getOpcode() == AMDGPU::IMPLICIT_DEF)
      return true;

    if (MI->getOpcode() != AMDGPU::COPY)
      break;

    Reg = MI->getOperand(1).getReg();
    if (!Reg.isVirtual())
      return false;
    if (!isLaneMaskReg(Reg))
      return false;
  }

  if (MI->getOpcode() != LMC->MovOpc)
    return false;

  if (!MI->getOperand(1).isImm())
    return false;

  int64_t Imm = MI->getOperand(1).getImm();
  if (Imm == 0) {
    Val = false;
    return true;
  }
  if (Imm == -1) {
    Val = true;
    return true;
  }

  return false;
}

static void instrDefsUsesSCC(const MachineInstr &MI, bool &Def, bool &Use) {
  Def = false;
  Use = false;

```
**EN:** This section contains concrete logic for PhiLoweringHelper::isConstantLaneMask. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `PhiLoweringHelper::isConstantLaneMask`.
**CN:** 本节包含与 PhiLoweringHelper::isConstantLaneMask 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`PhiLoweringHelper::isConstantLaneMask`。

### Lines 692-735: Conditional logic and checks
```cpp
  for (const MachineOperand &MO : MI.operands()) {
    if (MO.isReg() && MO.getReg() == AMDGPU::SCC) {
      if (MO.isUse())
        Use = true;
      else
        Def = true;
    }
  }
}

/// Return a point at the end of the given \p MBB to insert SALU instructions
/// for lane mask calculation. Take terminators and SCC into account.
MachineBasicBlock::iterator
AMDGPU::PhiLoweringHelper::getSaluInsertionAtEnd(MachineBasicBlock &MBB) const {
  auto InsertionPt = MBB.getFirstTerminator();
  bool TerminatorsUseSCC = false;
  for (auto I = InsertionPt, E = MBB.end(); I != E; ++I) {
    bool DefsSCC;
    instrDefsUsesSCC(*I, DefsSCC, TerminatorsUseSCC);
    if (TerminatorsUseSCC || DefsSCC)
      break;
  }

  if (!TerminatorsUseSCC)
    return InsertionPt;

  while (InsertionPt != MBB.begin()) {
    InsertionPt--;

    bool DefSCC, UseSCC;
    instrDefsUsesSCC(*InsertionPt, DefSCC, UseSCC);
    if (DefSCC)
      return InsertionPt;
  }

  // We should have at least seen an IMPLICIT_DEF or COPY
  llvm_unreachable("SCC used by terminator but no def in block");
}

// VReg_1 -> SReg_32 or SReg_64
void Vreg1LoweringHelper::markAsLaneMask(Register DstReg) const {
  MRI->setRegClass(DstReg, ST->getBoolRC());
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `PhiLoweringHelper::getSaluInsertionAtEnd`, `Vreg1LoweringHelper::markAsLaneMask`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`PhiLoweringHelper::getSaluInsertionAtEnd`, `Vreg1LoweringHelper::markAsLaneMask`。

### Lines 736-773: Implements Vreg1LoweringHelper::getCandidatesForLowering
```cpp
void Vreg1LoweringHelper::getCandidatesForLowering(
    SmallVectorImpl<MachineInstr *> &Vreg1Phis) const {
  for (MachineBasicBlock &MBB : *MF) {
    for (MachineInstr &MI : MBB.phis()) {
      if (isVreg1(MI.getOperand(0).getReg()))
        Vreg1Phis.push_back(&MI);
    }
  }
}

void Vreg1LoweringHelper::collectIncomingValuesFromPhi(
    const MachineInstr *MI,
    SmallVectorImpl<AMDGPU::Incoming> &Incomings) const {
  for (unsigned i = 1; i < MI->getNumOperands(); i += 2) {
    assert(i + 1 < MI->getNumOperands());
    Register IncomingReg = MI->getOperand(i).getReg();
    MachineBasicBlock *IncomingMBB = MI->getOperand(i + 1).getMBB();
    MachineInstr *IncomingDef = MRI->getUniqueVRegDef(IncomingReg);

    if (IncomingDef->getOpcode() == AMDGPU::COPY) {
      IncomingReg = IncomingDef->getOperand(1).getReg();
      assert(isLaneMaskReg(IncomingReg) || isVreg1(IncomingReg));
      assert(!IncomingDef->getOperand(1).getSubReg());
    } else if (IncomingDef->getOpcode() == AMDGPU::IMPLICIT_DEF) {
      continue;
    } else {
      assert(IncomingDef->isPHI() || PhiRegisters.count(IncomingReg));
    }

    Incomings.emplace_back(IncomingReg, IncomingMBB, Register());
  }
}

void Vreg1LoweringHelper::replaceDstReg(Register NewReg, Register OldReg,
                                        MachineBasicBlock *MBB) {
  MRI->replaceRegWith(NewReg, OldReg);
}

```
**EN:** This section contains concrete logic for Vreg1LoweringHelper::getCandidatesForLowering. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `Vreg1LoweringHelper::getCandidatesForLowering`, `Vreg1LoweringHelper::collectIncomingValuesFromPhi`, `Vreg1LoweringHelper::replaceDstReg`.
**CN:** 本节包含与 Vreg1LoweringHelper::getCandidatesForLowering 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`Vreg1LoweringHelper::getCandidatesForLowering`, `Vreg1LoweringHelper::collectIncomingValuesFromPhi`, `Vreg1LoweringHelper::replaceDstReg`。

### Lines 774-796: Implements Vreg1LoweringHelper::buildMergeLaneMasks
```cpp
void Vreg1LoweringHelper::buildMergeLaneMasks(MachineBasicBlock &MBB,
                                              MachineBasicBlock::iterator I,
                                              const DebugLoc &DL,
                                              Register DstReg, Register PrevReg,
                                              Register CurReg) {
  bool PrevVal = false;
  bool PrevConstant = isConstantLaneMask(PrevReg, PrevVal);
  bool CurVal = false;
  bool CurConstant = isConstantLaneMask(CurReg, CurVal);

  if (PrevConstant && CurConstant) {
    if (PrevVal == CurVal) {
      BuildMI(MBB, I, DL, TII->get(AMDGPU::COPY), DstReg).addReg(CurReg);
    } else if (CurVal) {
      BuildMI(MBB, I, DL, TII->get(AMDGPU::COPY), DstReg).addReg(LMC->ExecReg);
    } else {
      BuildMI(MBB, I, DL, TII->get(LMC->XorOpc), DstReg)
          .addReg(LMC->ExecReg)
          .addImm(-1);
    }
    return;
  }

```
**EN:** This section contains concrete logic for Vreg1LoweringHelper::buildMergeLaneMasks. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `Vreg1LoweringHelper::buildMergeLaneMasks`.
**CN:** 本节包含与 Vreg1LoweringHelper::buildMergeLaneMasks 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`Vreg1LoweringHelper::buildMergeLaneMasks`。

### Lines 797-839: Conditional logic and checks
```cpp
  Register PrevMaskedReg;
  Register CurMaskedReg;
  if (!PrevConstant) {
    if (CurConstant && CurVal) {
      PrevMaskedReg = PrevReg;
    } else {
      PrevMaskedReg = AMDGPU::createLaneMaskReg(MRI, LaneMaskRegAttrs);
      BuildMI(MBB, I, DL, TII->get(LMC->AndN2Opc), PrevMaskedReg)
          .addReg(PrevReg)
          .addReg(LMC->ExecReg);
    }
  }
  if (!CurConstant) {
    // TODO: check whether CurReg is already masked by EXEC
    if (PrevConstant && PrevVal) {
      CurMaskedReg = CurReg;
    } else {
      CurMaskedReg = AMDGPU::createLaneMaskReg(MRI, LaneMaskRegAttrs);
      BuildMI(MBB, I, DL, TII->get(LMC->AndOpc), CurMaskedReg)
          .addReg(CurReg)
          .addReg(LMC->ExecReg);
    }
  }

  if (PrevConstant && !PrevVal) {
    BuildMI(MBB, I, DL, TII->get(AMDGPU::COPY), DstReg)
        .addReg(CurMaskedReg);
  } else if (CurConstant && !CurVal) {
    BuildMI(MBB, I, DL, TII->get(AMDGPU::COPY), DstReg)
        .addReg(PrevMaskedReg);
  } else if (PrevConstant && PrevVal) {
    BuildMI(MBB, I, DL, TII->get(LMC->OrN2Opc), DstReg)
        .addReg(CurMaskedReg)
        .addReg(LMC->ExecReg);
  } else {
    BuildMI(MBB, I, DL, TII->get(LMC->OrOpc), DstReg)
        .addReg(PrevMaskedReg)
        .addReg(CurMaskedReg ? CurMaskedReg : LMC->ExecReg);
  }
}

void Vreg1LoweringHelper::constrainAsLaneMask(AMDGPU::Incoming &In) {}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::createLaneMaskReg`, `Vreg1LoweringHelper::constrainAsLaneMask`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::createLaneMaskReg`, `Vreg1LoweringHelper::constrainAsLaneMask`。

### Lines 840-882: Declares class SILowerI1CopiesLegacy
```cpp
/// Lower all instructions that def or use vreg_1 registers.
///
/// In a first pass, we lower COPYs from vreg_1 to vector registers, as can
/// occur around inline assembly. We do this first, before vreg_1 registers
/// are changed to scalar mask registers.
///
/// Then we lower all defs of vreg_1 registers. Phi nodes are lowered before
/// all others, because phi lowering looks through copies and can therefore
/// often make copy lowering unnecessary.
static bool runFixI1Copies(MachineFunction &MF, MachineDominatorTree &MDT,
                           MachinePostDominatorTree &MPDT) {
  // Only need to run this in SelectionDAG path.
  if (MF.getProperties().hasSelected())
    return false;

  Vreg1LoweringHelper Helper(&MF, &MDT, &MPDT);
  bool Changed = false;
  Changed |= Helper.lowerCopiesFromI1();
  Changed |= Helper.lowerPhis();
  Changed |= Helper.lowerCopiesToI1();
  return Helper.cleanConstrainRegs(Changed);
}

PreservedAnalyses
SILowerI1CopiesPass::run(MachineFunction &MF,
                         MachineFunctionAnalysisManager &MFAM) {
  MachineDominatorTree &MDT = MFAM.getResult<MachineDominatorTreeAnalysis>(MF);
  MachinePostDominatorTree &MPDT =
      MFAM.getResult<MachinePostDominatorTreeAnalysis>(MF);
  bool Changed = runFixI1Copies(MF, MDT, MPDT);
  if (!Changed)
    return PreservedAnalyses::all();

  // TODO: Probably preserves most.
  return getMachineFunctionPassPreservedAnalyses().preserveSet<CFGAnalyses>();
}

class SILowerI1CopiesLegacy : public MachineFunctionPass {
public:
  static char ID;

  SILowerI1CopiesLegacy() : MachineFunctionPass(ID) {}

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `SILowerI1CopiesLegacy`, `SILowerI1CopiesPass::run`, `PreservedAnalyses::all`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`SILowerI1CopiesLegacy`, `SILowerI1CopiesPass::run`, `PreservedAnalyses::all`。

### Lines 883-916: Registers LLVM passes
```cpp
  bool runOnMachineFunction(MachineFunction &MF) override;

  StringRef getPassName() const override { return "SI Lower i1 Copies"; }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    AU.addRequired<MachineDominatorTreeWrapperPass>();
    AU.addRequired<MachinePostDominatorTreeWrapperPass>();
    MachineFunctionPass::getAnalysisUsage(AU);
  }
};

bool SILowerI1CopiesLegacy::runOnMachineFunction(MachineFunction &MF) {
  MachineDominatorTree &MDT =
      getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
  MachinePostDominatorTree &MPDT =
      getAnalysis<MachinePostDominatorTreeWrapperPass>().getPostDomTree();
  return runFixI1Copies(MF, MDT, MPDT);
}

INITIALIZE_PASS_BEGIN(SILowerI1CopiesLegacy, DEBUG_TYPE, "SI Lower i1 Copies",
                      false, false)
INITIALIZE_PASS_DEPENDENCY(MachineDominatorTreeWrapperPass)
INITIALIZE_PASS_DEPENDENCY(MachinePostDominatorTreeWrapperPass)
INITIALIZE_PASS_END(SILowerI1CopiesLegacy, DEBUG_TYPE, "SI Lower i1 Copies",
                    false, false)

char SILowerI1CopiesLegacy::ID = 0;

char &llvm::SILowerI1CopiesLegacyID = SILowerI1CopiesLegacy::ID;

FunctionPass *llvm::createSILowerI1CopiesLegacyPass() {
  return new SILowerI1CopiesLegacy();
}
```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `MachineFunctionPass::getAnalysisUsage`, `SILowerI1CopiesLegacy::runOnMachineFunction`, `llvm::createSILowerI1CopiesLegacyPass`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`MachineFunctionPass::getAnalysisUsage`, `SILowerI1CopiesLegacy::runOnMachineFunction`, `llvm::createSILowerI1CopiesLegacyPass`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `Vreg1LoweringHelper`, `PhiIncomingAnalysis`, `LoopFinder`, `SILowerI1CopiesLegacy`, `Vreg1LoweringHelper::Vreg1LoweringHelper`, `Vreg1LoweringHelper::cleanConstrainRegs`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; lowering / 降低; LLVM pass integration / LLVM Pass 集成
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"SILowerI1Copies.h"`
- `"AMDGPU.h"`
- `"llvm/CodeGen/MachineIDFSSAUpdater.h"`
- `"llvm/InitializePasses.h"`
