# R600MachineCFGStructurizer.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/R600MachineCFGStructurizer.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements R600MachineCFGStructurizer for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 R600MachineCFGStructurizer 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-43: File banner, includes, and setup
```cpp
//===- R600MachineCFGStructurizer.cpp - CFG Structurizer ------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//==-----------------------------------------------------------------------===//

#include "MCTargetDesc/R600MCTargetDesc.h"
#include "R600.h"
#include "R600RegisterInfo.h"
#include "R600Subtarget.h"
#include "llvm/ADT/DepthFirstIterator.h"
#include "llvm/ADT/SCCIterator.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineJumpTableInfo.h"
#include "llvm/CodeGen/MachineLoopInfo.h"
#include "llvm/CodeGen/MachinePostDominators.h"
#include "llvm/InitializePasses.h"

using namespace llvm;

#define DEBUG_TYPE "structcfg"

enum { DEFAULT_VEC_SLOTS = 8 };

// TODO: move-begin.

//===----------------------------------------------------------------------===//
//
// Statistics for CFGStructurizer.
//
//===----------------------------------------------------------------------===//

STATISTIC(numSerialPatternMatch,    "CFGStructurizer number of serial pattern "
    "matched");
STATISTIC(numIfPatternMatch,        "CFGStructurizer number of if pattern "
    "matched");
STATISTIC(numClonedBlock,           "CFGStructurizer cloned blocks");
STATISTIC(numClonedInstr,           "CFGStructurizer cloned instructions");

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 44-86: Preprocessor guards and macros
```cpp
namespace {

//===----------------------------------------------------------------------===//
//
// Miscellaneous utility for CFGStructurizer.
//
//===----------------------------------------------------------------------===//

#define SHOWNEWINSTR(i) LLVM_DEBUG(dbgs() << "New instr: " << *i << "\n");

#define SHOWNEWBLK(b, msg)                                                     \
  LLVM_DEBUG(dbgs() << msg << "BB" << b->getNumber() << "size " << b->size();  \
             dbgs() << "\n";);

#define SHOWBLK_DETAIL(b, msg)                                                 \
  LLVM_DEBUG(if (b) {                                                          \
    dbgs() << msg << "BB" << b->getNumber() << "size " << b->size();           \
    b->print(dbgs());                                                          \
    dbgs() << "\n";                                                            \
  });

#define INVALIDSCCNUM -1

//===----------------------------------------------------------------------===//
//
// supporting data structure for CFGStructurizer
//
//===----------------------------------------------------------------------===//

class BlockInformation {
public:
  bool IsRetired = false;
  int SccNum = INVALIDSCCNUM;

  BlockInformation() = default;
};

//===----------------------------------------------------------------------===//
//
// CFGStructurizer
//
//===----------------------------------------------------------------------===//

```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `BlockInformation`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`BlockInformation`。

### Lines 87-125: Declares class R600MachineCFGStructurizer
```cpp
class R600MachineCFGStructurizer : public MachineFunctionPass {
public:
  using MBBVector = SmallVector<MachineBasicBlock *, 32>;
  using MBBInfoMap = std::map<MachineBasicBlock *, BlockInformation *>;
  using LoopLandInfoMap = std::map<MachineLoop *, MachineBasicBlock *>;

  enum PathToKind {
    Not_SinglePath = 0,
    SinglePath_InPath = 1,
    SinglePath_NotInPath = 2
  };

  static char ID;

  R600MachineCFGStructurizer() : MachineFunctionPass(ID) {}

  StringRef getPassName() const override {
    return "AMDGPU Control Flow Graph structurizer Pass";
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addRequired<MachineDominatorTreeWrapperPass>();
    AU.addRequired<MachinePostDominatorTreeWrapperPass>();
    AU.addRequired<MachineLoopInfoWrapperPass>();
    MachineFunctionPass::getAnalysisUsage(AU);
  }

  /// Perform the CFG structurization
  bool run();

  /// Perform the CFG preparation
  /// This step will remove every unconditionnal/dead jump instructions and make
  /// sure all loops have an exit block
  bool prepare();

  bool runOnMachineFunction(MachineFunction &MF) override {
    // FIXME: This pass causes verification failures.
    MF.getProperties().setFailsVerification();

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `R600MachineCFGStructurizer`, `PathToKind`, `MachineFunctionPass::getAnalysisUsage`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`R600MachineCFGStructurizer`, `PathToKind`, `MachineFunctionPass::getAnalysisUsage`。

### Lines 126-166: Conditional logic and checks
```cpp
    TII = MF.getSubtarget<R600Subtarget>().getInstrInfo();
    TRI = &TII->getRegisterInfo();
    LLVM_DEBUG(MF.dump(););
    OrderedBlks.clear();
    Visited.clear();
    FuncRep = &MF;
    MLI = &getAnalysis<MachineLoopInfoWrapperPass>().getLI();
    LLVM_DEBUG(dbgs() << "LoopInfo:\n"; PrintLoopinfo(*MLI););
    MDT = &getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
    LLVM_DEBUG(MDT->print(dbgs()););
    PDT = &getAnalysis<MachinePostDominatorTreeWrapperPass>().getPostDomTree();
    LLVM_DEBUG(PDT->print(dbgs()););
    prepare();
    run();
    LLVM_DEBUG(MF.dump(););
    return true;
  }

protected:
  MachineDominatorTree *MDT;
  MachinePostDominatorTree *PDT;
  MachineLoopInfo *MLI;
  const R600InstrInfo *TII = nullptr;
  const R600RegisterInfo *TRI = nullptr;

  // PRINT FUNCTIONS
  /// Print the ordered Blocks.
  void printOrderedBlocks() const {
    size_t i = 0;
    for (MBBVector::const_iterator iterBlk = OrderedBlks.begin(),
        iterBlkEnd = OrderedBlks.end(); iterBlk != iterBlkEnd; ++iterBlk, ++i) {
      dbgs() << "BB" << (*iterBlk)->getNumber();
      dbgs() << "(" << getSCCNum(*iterBlk) << "," << (*iterBlk)->size() << ")";
      if (i != 0 && i % 10 == 0) {
        dbgs() << "\n";
      } else {
        dbgs() << " ";
      }
    }
  }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 167-201: Defines PrintLoopinfo
```cpp
  static void PrintLoopinfo(const MachineLoopInfo &LoopInfo) {
    for (const MachineLoop *L : LoopInfo)
      L->print(dbgs());
  }

  // UTILITY FUNCTIONS
  int getSCCNum(MachineBasicBlock *MBB) const;
  MachineBasicBlock *getLoopLandInfo(MachineLoop *LoopRep) const;
  bool hasBackEdge(MachineBasicBlock *MBB) const;
  bool isRetiredBlock(MachineBasicBlock *MBB) const;
  bool isActiveLoophead(MachineBasicBlock *MBB) const;
  PathToKind singlePathTo(MachineBasicBlock *SrcMBB, MachineBasicBlock *DstMBB,
      bool AllowSideEntry = true) const;
  int countActiveBlock(MBBVector::const_iterator It,
      MBBVector::const_iterator E) const;
  bool needMigrateBlock(MachineBasicBlock *MBB) const;

  // Utility Functions
  void reversePredicateSetter(MachineBasicBlock::iterator I,
                              MachineBasicBlock &MBB);
  /// Compute the reversed DFS post order of Blocks
  void orderBlocks(MachineFunction *MF);

  // Function originally from CFGStructTraits
  void insertInstrEnd(MachineBasicBlock *MBB, int NewOpcode,
                      const DebugLoc &DL = DebugLoc());
  MachineInstr *insertInstrBefore(MachineBasicBlock *MBB, int NewOpcode,
                                  const DebugLoc &DL = DebugLoc());
  MachineInstr *insertInstrBefore(MachineBasicBlock::iterator I, int NewOpcode);
  void insertCondBranchBefore(MachineBasicBlock::iterator I, int NewOpcode,
                              const DebugLoc &DL);
  void insertCondBranchBefore(MachineBasicBlock *MBB,
                              MachineBasicBlock::iterator I, int NewOpcode,
                              int RegNum, const DebugLoc &DL);

```
**EN:** This section contains concrete logic for PrintLoopinfo. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 PrintLoopinfo 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 202-241: Declares getBranchNzeroOpcode
```cpp
  static int getBranchNzeroOpcode(int OldOpcode);
  static int getBranchZeroOpcode(int OldOpcode);
  static int getContinueNzeroOpcode(int OldOpcode);
  static int getContinueZeroOpcode(int OldOpcode);
  static MachineBasicBlock *getTrueBranch(MachineInstr *MI);
  static void setTrueBranch(MachineInstr *MI, MachineBasicBlock *MBB);
  static MachineBasicBlock *getFalseBranch(MachineBasicBlock *MBB,
      MachineInstr *MI);
  static bool isCondBranch(MachineInstr *MI);
  static bool isUncondBranch(MachineInstr *MI);
  static DebugLoc getLastDebugLocInBB(MachineBasicBlock *MBB);
  static MachineInstr *getNormalBlockBranchInstr(MachineBasicBlock *MBB);

  /// The correct naming for this is getPossibleLoopendBlockBranchInstr.
  ///
  /// BB with backward-edge could have move instructions after the branch
  /// instruction.  Such move instruction "belong to" the loop backward-edge.
  MachineInstr *getLoopendBlockBranchInstr(MachineBasicBlock *MBB);

  static MachineInstr *getReturnInstr(MachineBasicBlock *MBB);
  static bool isReturnBlock(MachineBasicBlock *MBB);
  static void cloneSuccessorList(MachineBasicBlock *DstMBB,
      MachineBasicBlock *SrcMBB);
  static MachineBasicBlock *clone(MachineBasicBlock *MBB);

  /// MachineBasicBlock::ReplaceUsesOfBlockWith doesn't serve the purpose
  /// because the AMDGPU instruction is not recognized as terminator fix this
  /// and retire this routine
  void replaceInstrUseOfBlockWith(MachineBasicBlock *SrcMBB,
      MachineBasicBlock *OldMBB, MachineBasicBlock *NewBlk);

  static void wrapup(MachineBasicBlock *MBB);

  int patternMatch(MachineBasicBlock *MBB);
  int patternMatchGroup(MachineBasicBlock *MBB);
  int serialPatternMatch(MachineBasicBlock *MBB);
  int ifPatternMatch(MachineBasicBlock *MBB);
  int loopendPatternMatch();
  int mergeLoop(MachineLoop *LoopRep);

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 242-282: Defines isSameloopDetachedContbreak
```cpp
  /// return true iff src1Blk->succ_empty() && src1Blk and src2Blk are in
  /// the same loop with LoopLandInfo without explicitly keeping track of
  /// loopContBlks and loopBreakBlks, this is a method to get the information.
  bool isSameloopDetachedContbreak(MachineBasicBlock *Src1MBB,
      MachineBasicBlock *Src2MBB);
  int handleJumpintoIf(MachineBasicBlock *HeadMBB,
      MachineBasicBlock *TrueMBB, MachineBasicBlock *FalseMBB);
  int handleJumpintoIfImp(MachineBasicBlock *HeadMBB,
      MachineBasicBlock *TrueMBB, MachineBasicBlock *FalseMBB);
  int improveSimpleJumpintoIf(MachineBasicBlock *HeadMBB,
      MachineBasicBlock *TrueMBB, MachineBasicBlock *FalseMBB,
      MachineBasicBlock **LandMBBPtr);
  void showImproveSimpleJumpintoIf(MachineBasicBlock *HeadMBB,
      MachineBasicBlock *TrueMBB, MachineBasicBlock *FalseMBB,
      MachineBasicBlock *LandMBB, bool Detail = false);
  int cloneOnSideEntryTo(MachineBasicBlock *PreMBB,
      MachineBasicBlock *SrcMBB, MachineBasicBlock *DstMBB);
  void mergeSerialBlock(MachineBasicBlock *DstMBB,
      MachineBasicBlock *SrcMBB);

  void mergeIfthenelseBlock(MachineInstr *BranchMI,
      MachineBasicBlock *MBB, MachineBasicBlock *TrueMBB,
      MachineBasicBlock *FalseMBB, MachineBasicBlock *LandMBB);
  void mergeLooplandBlock(MachineBasicBlock *DstMBB,
      MachineBasicBlock *LandMBB);
  void mergeLoopbreakBlock(MachineBasicBlock *ExitingMBB,
      MachineBasicBlock *LandMBB);
  void settleLoopcontBlock(MachineBasicBlock *ContingMBB,
      MachineBasicBlock *ContMBB);

  /// normalizeInfiniteLoopExit change
  ///   B1:
  ///        uncond_br LoopHeader
  ///
  /// to
  ///   B1:
  ///        cond_br 1 LoopHeader dummyExit
  /// and return the newly added dummy exit block
  MachineBasicBlock *normalizeInfiniteLoopExit(MachineLoop *LoopRep);
  void removeUnconditionalBranch(MachineBasicBlock *MBB);

```
**EN:** This section contains concrete logic for isSameloopDetachedContbreak. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 isSameloopDetachedContbreak 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 283-320: Declares removeRedundantConditionalBranch
```cpp
  /// Remove duplicate branches instructions in a block.
  /// For instance
  /// B0:
  ///    cond_br X B1 B2
  ///    cond_br X B1 B2
  /// is transformed to
  /// B0:
  ///    cond_br X B1 B2
  void removeRedundantConditionalBranch(MachineBasicBlock *MBB);

  void addDummyExitBlock(SmallVectorImpl<MachineBasicBlock *> &RetMBB);
  void removeSuccessor(MachineBasicBlock *MBB);
  MachineBasicBlock *cloneBlockForPredecessor(MachineBasicBlock *MBB,
      MachineBasicBlock *PredMBB);
  void migrateInstruction(MachineBasicBlock *SrcMBB,
      MachineBasicBlock *DstMBB, MachineBasicBlock::iterator I);
  void recordSccnum(MachineBasicBlock *MBB, int SCCNum);
  void retireBlock(MachineBasicBlock *MBB);

private:
  MBBInfoMap BlockInfoMap;
  LoopLandInfoMap LLInfoMap;
  std::map<MachineLoop *, bool> Visited;
  MachineFunction *FuncRep;
  SmallVector<MachineBasicBlock *, DEFAULT_VEC_SLOTS> OrderedBlks;
};

} // end anonymous namespace

char R600MachineCFGStructurizer::ID = 0;

int R600MachineCFGStructurizer::getSCCNum(MachineBasicBlock *MBB) const {
  MBBInfoMap::const_iterator It = BlockInfoMap.find(MBB);
  if (It == BlockInfoMap.end())
    return INVALIDSCCNUM;
  return (*It).second->SccNum;
}

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `R600MachineCFGStructurizer::getSCCNum`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`R600MachineCFGStructurizer::getSCCNum`。

### Lines 321-356: Implements R600MachineCFGStructurizer::getLoopLandInfo
```cpp
MachineBasicBlock *R600MachineCFGStructurizer::getLoopLandInfo(MachineLoop *LoopRep)
    const {
  LoopLandInfoMap::const_iterator It = LLInfoMap.find(LoopRep);
  if (It == LLInfoMap.end())
    return nullptr;
  return (*It).second;
}

bool R600MachineCFGStructurizer::hasBackEdge(MachineBasicBlock *MBB) const {
  MachineLoop *LoopRep = MLI->getLoopFor(MBB);
  if (!LoopRep)
    return false;
  MachineBasicBlock *LoopHeader = LoopRep->getHeader();
  return MBB->isSuccessor(LoopHeader);
}

bool R600MachineCFGStructurizer::isRetiredBlock(MachineBasicBlock *MBB) const {
  MBBInfoMap::const_iterator It = BlockInfoMap.find(MBB);
  if (It == BlockInfoMap.end())
    return false;
  return (*It).second->IsRetired;
}

bool R600MachineCFGStructurizer::isActiveLoophead(MachineBasicBlock *MBB) const {
  MachineLoop *LoopRep = MLI->getLoopFor(MBB);
  while (LoopRep && LoopRep->getHeader() == MBB) {
    MachineBasicBlock *LoopLand = getLoopLandInfo(LoopRep);
    if(!LoopLand)
      return true;
    if (!isRetiredBlock(LoopLand))
      return true;
    LoopRep = LoopRep->getParentLoop();
  }
  return false;
}

```
**EN:** This section contains concrete logic for R600MachineCFGStructurizer::getLoopLandInfo. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600MachineCFGStructurizer::getLoopLandInfo`, `R600MachineCFGStructurizer::hasBackEdge`, `R600MachineCFGStructurizer::isRetiredBlock`.
**CN:** 本节包含与 R600MachineCFGStructurizer::getLoopLandInfo 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600MachineCFGStructurizer::getLoopLandInfo`, `R600MachineCFGStructurizer::hasBackEdge`, `R600MachineCFGStructurizer::isRetiredBlock`。

### Lines 357-397: Implements R600MachineCFGStructurizer::singlePathTo
```cpp
R600MachineCFGStructurizer::PathToKind R600MachineCFGStructurizer::singlePathTo(
    MachineBasicBlock *SrcMBB, MachineBasicBlock *DstMBB,
    bool AllowSideEntry) const {
  assert(DstMBB);
  if (SrcMBB == DstMBB)
    return SinglePath_InPath;
  while (SrcMBB && SrcMBB->succ_size() == 1) {
    SrcMBB = *SrcMBB->succ_begin();
    if (SrcMBB == DstMBB)
      return SinglePath_InPath;
    if (!AllowSideEntry && SrcMBB->pred_size() > 1)
      return Not_SinglePath;
  }
  if (SrcMBB && SrcMBB->succ_size()==0)
    return SinglePath_NotInPath;
  return Not_SinglePath;
}

int R600MachineCFGStructurizer::countActiveBlock(MBBVector::const_iterator It,
    MBBVector::const_iterator E) const {
  int Count = 0;
  while (It != E) {
    if (!isRetiredBlock(*It))
      ++Count;
    ++It;
  }
  return Count;
}

bool R600MachineCFGStructurizer::needMigrateBlock(MachineBasicBlock *MBB) const {
  unsigned BlockSizeThreshold = 30;
  unsigned CloneInstrThreshold = 100;
  bool MultiplePreds = MBB && (MBB->pred_size() > 1);

  if(!MultiplePreds)
    return false;
  unsigned BlkSize = MBB->size();
  return ((BlkSize > BlockSizeThreshold) &&
      (BlkSize * (MBB->pred_size() - 1) > CloneInstrThreshold));
}

```
**EN:** This section contains concrete logic for R600MachineCFGStructurizer::singlePathTo. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600MachineCFGStructurizer::singlePathTo`, `R600MachineCFGStructurizer::countActiveBlock`, `R600MachineCFGStructurizer::needMigrateBlock`.
**CN:** 本节包含与 R600MachineCFGStructurizer::singlePathTo 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600MachineCFGStructurizer::singlePathTo`, `R600MachineCFGStructurizer::countActiveBlock`, `R600MachineCFGStructurizer::needMigrateBlock`。

### Lines 398-433: Implements R600MachineCFGStructurizer::reversePredicateSetter
```cpp
void R600MachineCFGStructurizer::reversePredicateSetter(
    MachineBasicBlock::iterator I, MachineBasicBlock &MBB) {
  assert(I.isValid() && "Expected valid iterator");
  for (;; --I) {
    if (I == MBB.end())
      continue;
    if (I->getOpcode() == R600::PRED_X) {
      switch (I->getOperand(2).getImm()) {
      case R600::PRED_SETE_INT:
        I->getOperand(2).setImm(R600::PRED_SETNE_INT);
        return;
      case R600::PRED_SETNE_INT:
        I->getOperand(2).setImm(R600::PRED_SETE_INT);
        return;
      case R600::PRED_SETE:
        I->getOperand(2).setImm(R600::PRED_SETNE);
        return;
      case R600::PRED_SETNE:
        I->getOperand(2).setImm(R600::PRED_SETE);
        return;
      default:
        llvm_unreachable("PRED_X Opcode invalid!");
      }
    }
  }
}

void R600MachineCFGStructurizer::insertInstrEnd(MachineBasicBlock *MBB,
                                           int NewOpcode, const DebugLoc &DL) {
  MachineInstr *MI =
      MBB->getParent()->CreateMachineInstr(TII->get(NewOpcode), DL);
  MBB->push_back(MI);
  //assume the instruction doesn't take any reg operand ...
  SHOWNEWINSTR(MI);
}

```
**EN:** This section contains concrete logic for R600MachineCFGStructurizer::reversePredicateSetter. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600MachineCFGStructurizer::reversePredicateSetter`, `R600MachineCFGStructurizer::insertInstrEnd`.
**CN:** 本节包含与 R600MachineCFGStructurizer::reversePredicateSetter 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600MachineCFGStructurizer::reversePredicateSetter`, `R600MachineCFGStructurizer::insertInstrEnd`。

### Lines 434-471: Implements R600MachineCFGStructurizer::insertInstrBefore
```cpp
MachineInstr *R600MachineCFGStructurizer::insertInstrBefore(MachineBasicBlock *MBB,
                                                       int NewOpcode,
                                                       const DebugLoc &DL) {
  MachineInstr *MI =
      MBB->getParent()->CreateMachineInstr(TII->get(NewOpcode), DL);
  if (!MBB->empty())
    MBB->insert(MBB->begin(), MI);
  else
    MBB->push_back(MI);
  SHOWNEWINSTR(MI);
  return MI;
}

MachineInstr *R600MachineCFGStructurizer::insertInstrBefore(
    MachineBasicBlock::iterator I, int NewOpcode) {
  MachineInstr *OldMI = &(*I);
  MachineBasicBlock *MBB = OldMI->getParent();
  MachineInstr *NewMBB =
      MBB->getParent()->CreateMachineInstr(TII->get(NewOpcode), DebugLoc());
  MBB->insert(I, NewMBB);
  //assume the instruction doesn't take any reg operand ...
  SHOWNEWINSTR(NewMBB);
  return NewMBB;
}

void R600MachineCFGStructurizer::insertCondBranchBefore(
    MachineBasicBlock::iterator I, int NewOpcode, const DebugLoc &DL) {
  MachineInstr *OldMI = &(*I);
  MachineBasicBlock *MBB = OldMI->getParent();
  MachineFunction *MF = MBB->getParent();
  MachineInstr *NewMI = MF->CreateMachineInstr(TII->get(NewOpcode), DL);
  MBB->insert(I, NewMI);
  MachineInstrBuilder MIB(*MF, NewMI);
  MIB.addReg(OldMI->getOperand(1).getReg());
  SHOWNEWINSTR(NewMI);
  //erase later oldInstr->eraseFromParent();
}

```
**EN:** This section contains concrete logic for R600MachineCFGStructurizer::insertInstrBefore. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600MachineCFGStructurizer::insertInstrBefore`, `R600MachineCFGStructurizer::insertCondBranchBefore`.
**CN:** 本节包含与 R600MachineCFGStructurizer::insertInstrBefore 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600MachineCFGStructurizer::insertInstrBefore`, `R600MachineCFGStructurizer::insertCondBranchBefore`。

### Lines 472-513: Implements R600MachineCFGStructurizer::insertCondBranchBefore
```cpp
void R600MachineCFGStructurizer::insertCondBranchBefore(
    MachineBasicBlock *blk, MachineBasicBlock::iterator I, int NewOpcode,
    int RegNum, const DebugLoc &DL) {
  MachineFunction *MF = blk->getParent();
  MachineInstr *NewInstr = MF->CreateMachineInstr(TII->get(NewOpcode), DL);
  //insert before
  blk->insert(I, NewInstr);
  MachineInstrBuilder(*MF, NewInstr).addReg(RegNum);
  SHOWNEWINSTR(NewInstr);
}

int R600MachineCFGStructurizer::getBranchNzeroOpcode(int OldOpcode) {
  switch(OldOpcode) {
  case R600::JUMP_COND:
  case R600::JUMP: return R600::IF_PREDICATE_SET;
  case R600::BRANCH_COND_i32:
  case R600::BRANCH_COND_f32: return R600::IF_LOGICALNZ_f32;
  default: llvm_unreachable("internal error");
  }
  return -1;
}

int R600MachineCFGStructurizer::getBranchZeroOpcode(int OldOpcode) {
  switch(OldOpcode) {
  case R600::JUMP_COND:
  case R600::JUMP: return R600::IF_PREDICATE_SET;
  case R600::BRANCH_COND_i32:
  case R600::BRANCH_COND_f32: return R600::IF_LOGICALZ_f32;
  default: llvm_unreachable("internal error");
  }
  return -1;
}

int R600MachineCFGStructurizer::getContinueNzeroOpcode(int OldOpcode) {
  switch(OldOpcode) {
  case R600::JUMP_COND:
  case R600::JUMP: return R600::CONTINUE_LOGICALNZ_i32;
  default: llvm_unreachable("internal error");
  }
  return -1;
}

```
**EN:** This section contains concrete logic for R600MachineCFGStructurizer::insertCondBranchBefore. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600MachineCFGStructurizer::insertCondBranchBefore`, `R600MachineCFGStructurizer::getBranchNzeroOpcode`, `R600MachineCFGStructurizer::getBranchZeroOpcode`.
**CN:** 本节包含与 R600MachineCFGStructurizer::insertCondBranchBefore 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600MachineCFGStructurizer::insertCondBranchBefore`, `R600MachineCFGStructurizer::getBranchNzeroOpcode`, `R600MachineCFGStructurizer::getBranchZeroOpcode`。

### Lines 514-553: Implements R600MachineCFGStructurizer::getContinueZeroOpcode
```cpp
int R600MachineCFGStructurizer::getContinueZeroOpcode(int OldOpcode) {
  switch(OldOpcode) {
  case R600::JUMP_COND:
  case R600::JUMP: return R600::CONTINUE_LOGICALZ_i32;
  default: llvm_unreachable("internal error");
  }
  return -1;
}

MachineBasicBlock *R600MachineCFGStructurizer::getTrueBranch(MachineInstr *MI) {
  return MI->getOperand(0).getMBB();
}

void R600MachineCFGStructurizer::setTrueBranch(MachineInstr *MI,
    MachineBasicBlock *MBB) {
  MI->getOperand(0).setMBB(MBB);
}

MachineBasicBlock *
R600MachineCFGStructurizer::getFalseBranch(MachineBasicBlock *MBB,
    MachineInstr *MI) {
  assert(MBB->succ_size() == 2);
  MachineBasicBlock *TrueBranch = getTrueBranch(MI);
  MachineBasicBlock::succ_iterator It = MBB->succ_begin();
  MachineBasicBlock::succ_iterator Next = It;
  ++Next;
  return (*It == TrueBranch) ? *Next : *It;
}

bool R600MachineCFGStructurizer::isCondBranch(MachineInstr *MI) {
  switch (MI->getOpcode()) {
    case R600::JUMP_COND:
    case R600::BRANCH_COND_i32:
    case R600::BRANCH_COND_f32: return true;
  default:
    return false;
  }
  return false;
}

```
**EN:** This section contains concrete logic for R600MachineCFGStructurizer::getContinueZeroOpcode. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600MachineCFGStructurizer::getContinueZeroOpcode`, `R600MachineCFGStructurizer::getTrueBranch`, `R600MachineCFGStructurizer::setTrueBranch`.
**CN:** 本节包含与 R600MachineCFGStructurizer::getContinueZeroOpcode 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600MachineCFGStructurizer::getContinueZeroOpcode`, `R600MachineCFGStructurizer::getTrueBranch`, `R600MachineCFGStructurizer::setTrueBranch`。

### Lines 554-582: Implements R600MachineCFGStructurizer::isUncondBranch
```cpp
bool R600MachineCFGStructurizer::isUncondBranch(MachineInstr *MI) {
  switch (MI->getOpcode()) {
  case R600::JUMP:
  case R600::BRANCH:
    return true;
  default:
    return false;
  }
  return false;
}

DebugLoc R600MachineCFGStructurizer::getLastDebugLocInBB(MachineBasicBlock *MBB) {
  //get DebugLoc from the first MachineBasicBlock instruction with debug info
  DebugLoc DL;
  for (MachineInstr &MI : *MBB)
    if (MI.getDebugLoc())
      DL = MI.getDebugLoc();
  return DL;
}

MachineInstr *R600MachineCFGStructurizer::getNormalBlockBranchInstr(
    MachineBasicBlock *MBB) {
  MachineBasicBlock::reverse_iterator It = MBB->rbegin();
  MachineInstr *MI = &*It;
  if (MI && (isCondBranch(MI) || isUncondBranch(MI)))
    return MI;
  return nullptr;
}

```
**EN:** This section contains concrete logic for R600MachineCFGStructurizer::isUncondBranch. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600MachineCFGStructurizer::isUncondBranch`, `R600MachineCFGStructurizer::getLastDebugLocInBB`, `R600MachineCFGStructurizer::getNormalBlockBranchInstr`.
**CN:** 本节包含与 R600MachineCFGStructurizer::isUncondBranch 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600MachineCFGStructurizer::isUncondBranch`, `R600MachineCFGStructurizer::getLastDebugLocInBB`, `R600MachineCFGStructurizer::getNormalBlockBranchInstr`。

### Lines 583-625: Implements R600MachineCFGStructurizer::getLoopendBlockBranchInstr
```cpp
MachineInstr *R600MachineCFGStructurizer::getLoopendBlockBranchInstr(
    MachineBasicBlock *MBB) {
  for (MachineBasicBlock::reverse_iterator It = MBB->rbegin(), E = MBB->rend();
      It != E; ++It) {
    // FIXME: Simplify
    MachineInstr *MI = &*It;
    if (MI) {
      if (isCondBranch(MI) || isUncondBranch(MI))
        return MI;
      if (!TII->isMov(MI->getOpcode()))
        break;
    }
  }
  return nullptr;
}

MachineInstr *R600MachineCFGStructurizer::getReturnInstr(MachineBasicBlock *MBB) {
  MachineBasicBlock::reverse_iterator It = MBB->rbegin();
  if (It != MBB->rend()) {
    MachineInstr *instr = &(*It);
    if (instr->getOpcode() == R600::RETURN)
      return instr;
  }
  return nullptr;
}

bool R600MachineCFGStructurizer::isReturnBlock(MachineBasicBlock *MBB) {
  MachineInstr *MI = getReturnInstr(MBB);
  bool IsReturn = MBB->succ_empty();
  if (MI)
    assert(IsReturn);
  else if (IsReturn)
    LLVM_DEBUG(dbgs() << "BB" << MBB->getNumber()
                      << " is return block without RETURN instr\n";);
  return  IsReturn;
}

void R600MachineCFGStructurizer::cloneSuccessorList(MachineBasicBlock *DstMBB,
    MachineBasicBlock *SrcMBB) {
  for (MachineBasicBlock *Succ : SrcMBB->successors())
    DstMBB->addSuccessor(Succ);  // *iter's predecessor is also taken care of
}

```
**EN:** This section contains concrete logic for R600MachineCFGStructurizer::getLoopendBlockBranchInstr. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600MachineCFGStructurizer::getLoopendBlockBranchInstr`, `R600MachineCFGStructurizer::getReturnInstr`, `R600MachineCFGStructurizer::isReturnBlock`.
**CN:** 本节包含与 R600MachineCFGStructurizer::getLoopendBlockBranchInstr 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600MachineCFGStructurizer::getLoopendBlockBranchInstr`, `R600MachineCFGStructurizer::getReturnInstr`, `R600MachineCFGStructurizer::isReturnBlock`。

### Lines 626-665: Implements R600MachineCFGStructurizer::clone
```cpp
MachineBasicBlock *R600MachineCFGStructurizer::clone(MachineBasicBlock *MBB) {
  MachineFunction *Func = MBB->getParent();
  MachineBasicBlock *NewMBB = Func->CreateMachineBasicBlock();
  Func->push_back(NewMBB);  //insert to function
  for (const MachineInstr &It : *MBB)
    NewMBB->push_back(Func->CloneMachineInstr(&It));
  return NewMBB;
}

void R600MachineCFGStructurizer::replaceInstrUseOfBlockWith(
    MachineBasicBlock *SrcMBB, MachineBasicBlock *OldMBB,
    MachineBasicBlock *NewBlk) {
  MachineInstr *BranchMI = getLoopendBlockBranchInstr(SrcMBB);
  if (BranchMI && isCondBranch(BranchMI) &&
      getTrueBranch(BranchMI) == OldMBB)
    setTrueBranch(BranchMI, NewBlk);
}

void R600MachineCFGStructurizer::wrapup(MachineBasicBlock *MBB) {
  assert((!MBB->getParent()->getJumpTableInfo()
          || MBB->getParent()->getJumpTableInfo()->isEmpty())
         && "found a jump table");

   //collect continue right before endloop
   SmallVector<MachineInstr *, DEFAULT_VEC_SLOTS> ContInstr;
   MachineBasicBlock::iterator Pre = MBB->begin();
   MachineBasicBlock::iterator E = MBB->end();
   MachineBasicBlock::iterator It = Pre;
   while (It != E) {
     if (Pre->getOpcode() == R600::CONTINUE
         && It->getOpcode() == R600::ENDLOOP)
       ContInstr.push_back(&*Pre);
     Pre = It;
     ++It;
   }

   //delete continue right before endloop
   for (auto *MI : ContInstr)
     MI->eraseFromParent();

```
**EN:** This section contains concrete logic for R600MachineCFGStructurizer::clone. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600MachineCFGStructurizer::clone`, `R600MachineCFGStructurizer::replaceInstrUseOfBlockWith`, `R600MachineCFGStructurizer::wrapup`.
**CN:** 本节包含与 R600MachineCFGStructurizer::clone 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600MachineCFGStructurizer::clone`, `R600MachineCFGStructurizer::replaceInstrUseOfBlockWith`, `R600MachineCFGStructurizer::wrapup`。

### Lines 666-705: Implements R600MachineCFGStructurizer::prepare
```cpp
   // TODO to fix up jump table so later phase won't be confused.  if
   // (jumpTableInfo->isEmpty() == false) { need to clean the jump table, but
   // there isn't such an interface yet.  alternatively, replace all the other
   // blocks in the jump table with the entryBlk //}
}

bool R600MachineCFGStructurizer::prepare() {
  bool Changed = false;

  //FIXME: if not reducible flow graph, make it so ???

  LLVM_DEBUG(dbgs() << "R600MachineCFGStructurizer::prepare\n";);

  orderBlocks(FuncRep);

  SmallVector<MachineBasicBlock *, DEFAULT_VEC_SLOTS> RetBlks;

  // Add an ExitBlk to loop that don't have one
  for (MachineLoop *LoopRep : *MLI) {
    MBBVector ExitingMBBs;
    LoopRep->getExitingBlocks(ExitingMBBs);

    if (ExitingMBBs.empty()) {
      MachineBasicBlock* DummyExitBlk = normalizeInfiniteLoopExit(LoopRep);
      if (DummyExitBlk)
        RetBlks.push_back(DummyExitBlk);
    }
  }

  // Remove unconditional branch instr.
  // Add dummy exit block iff there are multiple returns.
  for (MachineBasicBlock *MBB : OrderedBlks) {
    removeUnconditionalBranch(MBB);
    removeRedundantConditionalBranch(MBB);
    if (isReturnBlock(MBB)) {
      RetBlks.push_back(MBB);
    }
    assert(MBB->succ_size() <= 2);
  }

```
**EN:** This section contains concrete logic for R600MachineCFGStructurizer::prepare. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600MachineCFGStructurizer::prepare`.
**CN:** 本节包含与 R600MachineCFGStructurizer::prepare 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600MachineCFGStructurizer::prepare`。

### Lines 706-748: Preprocessor guards and macros
```cpp
  if (RetBlks.size() >= 2) {
    addDummyExitBlock(RetBlks);
    Changed = true;
  }

  return Changed;
}

bool R600MachineCFGStructurizer::run() {
  //Assume reducible CFG...
  LLVM_DEBUG(dbgs() << "R600MachineCFGStructurizer::run\n");

#ifdef STRESSTEST
  //Use the worse block ordering to test the algorithm.
  ReverseVector(orderedBlks);
#endif

  LLVM_DEBUG(dbgs() << "Ordered blocks:\n"; printOrderedBlocks(););
  int NumIter = 0;
  bool Finish = false;
  MachineBasicBlock *MBB;
  bool MakeProgress = false;
  int NumRemainedBlk = countActiveBlock(OrderedBlks.begin(),
                                        OrderedBlks.end());

  do {
    ++NumIter;
    LLVM_DEBUG(dbgs() << "numIter = " << NumIter
                      << ", numRemaintedBlk = " << NumRemainedBlk << "\n";);
    (void)NumIter;

    SmallVectorImpl<MachineBasicBlock *>::const_iterator It =
        OrderedBlks.begin();
    SmallVectorImpl<MachineBasicBlock *>::const_iterator E =
        OrderedBlks.end();

    SmallVectorImpl<MachineBasicBlock *>::const_iterator SccBeginIter =
        It;
    MachineBasicBlock *SccBeginMBB = nullptr;
    int SccNumBlk = 0;  // The number of active blocks, init to a
                        // maximum possible number.
    int SccNumIter;     // Number of iteration in this SCC.

```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `R600MachineCFGStructurizer::run`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`R600MachineCFGStructurizer::run`。

### Lines 749-792: Conditional logic and checks
```cpp
    while (It != E) {
      MBB = *It;

      if (!SccBeginMBB) {
        SccBeginIter = It;
        SccBeginMBB = MBB;
        SccNumIter = 0;
        SccNumBlk = NumRemainedBlk; // Init to maximum possible number.
        LLVM_DEBUG(dbgs() << "start processing SCC" << getSCCNum(SccBeginMBB);
                   dbgs() << "\n";);
      }

      if (!isRetiredBlock(MBB))
        patternMatch(MBB);

      ++It;

      bool ContNextScc = true;
      if (It == E
          || getSCCNum(SccBeginMBB) != getSCCNum(*It)) {
        // Just finish one scc.
        ++SccNumIter;
        int sccRemainedNumBlk = countActiveBlock(SccBeginIter, It);
        if (sccRemainedNumBlk != 1 && sccRemainedNumBlk >= SccNumBlk) {
          LLVM_DEBUG(dbgs() << "Can't reduce SCC " << getSCCNum(MBB)
                            << ", sccNumIter = " << SccNumIter;
                     dbgs() << "doesn't make any progress\n";);
          (void)SccNumIter;
          ContNextScc = true;
        } else if (sccRemainedNumBlk != 1 && sccRemainedNumBlk < SccNumBlk) {
          SccNumBlk = sccRemainedNumBlk;
          It = SccBeginIter;
          ContNextScc = false;
          LLVM_DEBUG(dbgs() << "repeat processing SCC" << getSCCNum(MBB)
                            << "sccNumIter = " << SccNumIter << '\n';);
        } else {
          // Finish the current scc.
          ContNextScc = true;
        }
      } else {
        // Continue on next component in the current scc.
        ContNextScc = false;
      }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 793-835: Conditional logic and checks
```cpp
      if (ContNextScc)
        SccBeginMBB = nullptr;
    } //while, "one iteration" over the function.

    MachineBasicBlock *EntryMBB =
        *GraphTraits<MachineFunction *>::nodes_begin(FuncRep);
    if (EntryMBB->succ_empty()) {
      Finish = true;
      LLVM_DEBUG(dbgs() << "Reduce to one block\n";);
    } else {
      int NewnumRemainedBlk
        = countActiveBlock(OrderedBlks.begin(), OrderedBlks.end());
      // consider cloned blocks ??
      if (NewnumRemainedBlk == 1 || NewnumRemainedBlk < NumRemainedBlk) {
        MakeProgress = true;
        NumRemainedBlk = NewnumRemainedBlk;
      } else {
        MakeProgress = false;
        LLVM_DEBUG(dbgs() << "No progress\n";);
      }
    }
  } while (!Finish && MakeProgress);

  // Misc wrap up to maintain the consistency of the Function representation.
  wrapup(*GraphTraits<MachineFunction *>::nodes_begin(FuncRep));

  // Detach retired Block, release memory.
  for (auto &It : BlockInfoMap) {
    if (It.second && It.second->IsRetired) {
      assert((It.first)->getNumber() != -1);
      LLVM_DEBUG(dbgs() << "Erase BB" << (It.first)->getNumber() << "\n";);
      It.first->eraseFromParent(); // Remove from the parent Function.
    }
    delete It.second;
  }
  BlockInfoMap.clear();
  LLInfoMap.clear();

  if (!Finish) {
    LLVM_DEBUG(FuncRep->viewCFG());
    report_fatal_error("IRREDUCIBLE_CFG");
  }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 836-872: Implements R600MachineCFGStructurizer::orderBlocks
```cpp
  return true;
}

void R600MachineCFGStructurizer::orderBlocks(MachineFunction *MF) {
  int SccNum = 0;
  for (scc_iterator<MachineFunction *> It = scc_begin(MF); !It.isAtEnd();
       ++It, ++SccNum) {
    const std::vector<MachineBasicBlock *> &SccNext = *It;
    for (MachineBasicBlock *MBB : SccNext) {
      OrderedBlks.push_back(MBB);
      recordSccnum(MBB, SccNum);
    }
  }

  // walk through all the block in func to check for unreachable
  for (auto *MBB : nodes(MF)) {
    SccNum = getSCCNum(MBB);
    if (SccNum == INVALIDSCCNUM)
      dbgs() << "unreachable block BB" << MBB->getNumber() << "\n";
  }
}

int R600MachineCFGStructurizer::patternMatch(MachineBasicBlock *MBB) {
  int NumMatch = 0;
  int CurMatch;

  LLVM_DEBUG(dbgs() << "Begin patternMatch BB" << MBB->getNumber() << "\n";);

  while ((CurMatch = patternMatchGroup(MBB)) > 0)
    NumMatch += CurMatch;

  LLVM_DEBUG(dbgs() << "End patternMatch BB" << MBB->getNumber()
                    << ", numMatch = " << NumMatch << "\n";);

  return NumMatch;
}

```
**EN:** This section contains concrete logic for R600MachineCFGStructurizer::orderBlocks. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600MachineCFGStructurizer::orderBlocks`, `R600MachineCFGStructurizer::patternMatch`.
**CN:** 本节包含与 R600MachineCFGStructurizer::orderBlocks 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600MachineCFGStructurizer::orderBlocks`, `R600MachineCFGStructurizer::patternMatch`。

### Lines 873-915: Implements R600MachineCFGStructurizer::patternMatchGroup
```cpp
int R600MachineCFGStructurizer::patternMatchGroup(MachineBasicBlock *MBB) {
  int NumMatch = 0;
  NumMatch += loopendPatternMatch();
  NumMatch += serialPatternMatch(MBB);
  NumMatch += ifPatternMatch(MBB);
  return NumMatch;
}

int R600MachineCFGStructurizer::serialPatternMatch(MachineBasicBlock *MBB) {
  if (MBB->succ_size() != 1)
    return 0;

  MachineBasicBlock *childBlk = *MBB->succ_begin();
  if (childBlk->pred_size() != 1 || isActiveLoophead(childBlk))
    return 0;

  mergeSerialBlock(MBB, childBlk);
  ++numSerialPatternMatch;
  return 1;
}

int R600MachineCFGStructurizer::ifPatternMatch(MachineBasicBlock *MBB) {
  //two edges
  if (MBB->succ_size() != 2)
    return 0;
  if (hasBackEdge(MBB))
    return 0;
  MachineInstr *BranchMI = getNormalBlockBranchInstr(MBB);
  if (!BranchMI)
    return 0;

  assert(isCondBranch(BranchMI));
  int NumMatch = 0;

  MachineBasicBlock *TrueMBB = getTrueBranch(BranchMI);
  NumMatch += serialPatternMatch(TrueMBB);
  NumMatch += ifPatternMatch(TrueMBB);
  MachineBasicBlock *FalseMBB = getFalseBranch(MBB, BranchMI);
  NumMatch += serialPatternMatch(FalseMBB);
  NumMatch += ifPatternMatch(FalseMBB);
  MachineBasicBlock *LandBlk;
  int Cloned = 0;

```
**EN:** This section contains concrete logic for R600MachineCFGStructurizer::patternMatchGroup. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600MachineCFGStructurizer::patternMatchGroup`, `R600MachineCFGStructurizer::serialPatternMatch`, `R600MachineCFGStructurizer::ifPatternMatch`.
**CN:** 本节包含与 R600MachineCFGStructurizer::patternMatchGroup 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600MachineCFGStructurizer::patternMatchGroup`, `R600MachineCFGStructurizer::serialPatternMatch`, `R600MachineCFGStructurizer::ifPatternMatch`。

### Lines 916-957: Declares assert
```cpp
  assert (!TrueMBB->succ_empty() || !FalseMBB->succ_empty());
  // TODO: Simplify
  if (TrueMBB->succ_size() == 1 && FalseMBB->succ_size() == 1
    && *TrueMBB->succ_begin() == *FalseMBB->succ_begin()) {
    // Diamond pattern
    LandBlk = *TrueMBB->succ_begin();
  } else if (TrueMBB->succ_size() == 1 && *TrueMBB->succ_begin() == FalseMBB) {
    // Triangle pattern, false is empty
    LandBlk = FalseMBB;
    FalseMBB = nullptr;
  } else if (FalseMBB->succ_size() == 1
             && *FalseMBB->succ_begin() == TrueMBB) {
    // Triangle pattern, true is empty
    // We reverse the predicate to make a triangle, empty false pattern;
    std::swap(TrueMBB, FalseMBB);
    reversePredicateSetter(MBB->end(), *MBB);
    LandBlk = FalseMBB;
    FalseMBB = nullptr;
  } else if (FalseMBB->succ_size() == 1
             && isSameloopDetachedContbreak(TrueMBB, FalseMBB)) {
    LandBlk = *FalseMBB->succ_begin();
  } else if (TrueMBB->succ_size() == 1
    && isSameloopDetachedContbreak(FalseMBB, TrueMBB)) {
    LandBlk = *TrueMBB->succ_begin();
  } else {
    return NumMatch + handleJumpintoIf(MBB, TrueMBB, FalseMBB);
  }

  // improveSimpleJumpinfoIf can handle the case where landBlk == NULL but the
  // new BB created for landBlk==NULL may introduce new challenge to the
  // reduction process.
  if (LandBlk &&
      ((TrueMBB && TrueMBB->pred_size() > 1)
      || (FalseMBB && FalseMBB->pred_size() > 1))) {
     Cloned += improveSimpleJumpintoIf(MBB, TrueMBB, FalseMBB, &LandBlk);
  }

  if (TrueMBB && TrueMBB->pred_size() > 1) {
    TrueMBB = cloneBlockForPredecessor(TrueMBB, MBB);
    ++Cloned;
  }

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `std::swap`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`std::swap`。

### Lines 958-996: Declares mergeIfthenelseBlock
```cpp
  if (FalseMBB && FalseMBB->pred_size() > 1) {
    FalseMBB = cloneBlockForPredecessor(FalseMBB, MBB);
    ++Cloned;
  }

  mergeIfthenelseBlock(BranchMI, MBB, TrueMBB, FalseMBB, LandBlk);

  ++numIfPatternMatch;

  numClonedBlock += Cloned;

  return 1 + Cloned + NumMatch;
}

int R600MachineCFGStructurizer::loopendPatternMatch() {
  std::deque<MachineLoop *> NestedLoops;
  for (auto &It: *MLI)
    for (MachineLoop *ML : depth_first(It))
      NestedLoops.push_front(ML);

  if (NestedLoops.empty())
    return 0;

  // Process nested loop outside->inside (we did push_front),
  // so "continue" to a outside loop won't be mistaken as "break"
  // of the current loop.
  int Num = 0;
  for (MachineLoop *ExaminedLoop : NestedLoops) {
    if (ExaminedLoop->getNumBlocks() == 0 || Visited[ExaminedLoop])
      continue;
    LLVM_DEBUG(dbgs() << "Processing:\n"; ExaminedLoop->dump(););
    int NumBreak = mergeLoop(ExaminedLoop);
    if (NumBreak == -1)
      break;
    Num += NumBreak;
  }
  return Num;
}

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `R600MachineCFGStructurizer::loopendPatternMatch`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`R600MachineCFGStructurizer::loopendPatternMatch`。

### Lines 997-1035: Implements R600MachineCFGStructurizer::mergeLoop
```cpp
int R600MachineCFGStructurizer::mergeLoop(MachineLoop *LoopRep) {
  MachineBasicBlock *LoopHeader = LoopRep->getHeader();
  MBBVector ExitingMBBs;
  LoopRep->getExitingBlocks(ExitingMBBs);
  assert(!ExitingMBBs.empty() && "Infinite Loop not supported");
  LLVM_DEBUG(dbgs() << "Loop has " << ExitingMBBs.size()
                    << " exiting blocks\n";);
  // We assume a single ExitBlk
  MBBVector ExitBlks;
  LoopRep->getExitBlocks(ExitBlks);
  SmallPtrSet<MachineBasicBlock *, 2> ExitBlkSet(llvm::from_range, ExitBlks);
  assert(ExitBlkSet.size() == 1);
  MachineBasicBlock *ExitBlk = *ExitBlks.begin();
  assert(ExitBlk && "Loop has several exit block");
  MBBVector LatchBlks;
  for (auto *LB : inverse_children<MachineBasicBlock*>(LoopHeader))
    if (LoopRep->contains(LB))
      LatchBlks.push_back(LB);

  for (MachineBasicBlock *MBB : ExitingMBBs)
    mergeLoopbreakBlock(MBB, ExitBlk);
  for (MachineBasicBlock *MBB : LatchBlks)
    settleLoopcontBlock(MBB, LoopHeader);
  int Match = 0;
  do {
    Match = 0;
    Match += serialPatternMatch(LoopHeader);
    Match += ifPatternMatch(LoopHeader);
  } while (Match > 0);
  mergeLooplandBlock(LoopHeader, ExitBlk);
  MachineLoop *ParentLoop = LoopRep->getParentLoop();
  if (ParentLoop)
    MLI->changeLoopFor(LoopHeader, ParentLoop);
  else
    MLI->removeBlock(LoopHeader);
  Visited[LoopRep] = true;
  return 1;
}

```
**EN:** This section contains concrete logic for R600MachineCFGStructurizer::mergeLoop. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600MachineCFGStructurizer::mergeLoop`.
**CN:** 本节包含与 R600MachineCFGStructurizer::mergeLoop 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600MachineCFGStructurizer::mergeLoop`。

### Lines 1036-1079: Implements R600MachineCFGStructurizer::isSameloopDetachedContbreak
```cpp
bool R600MachineCFGStructurizer::isSameloopDetachedContbreak(
    MachineBasicBlock *Src1MBB, MachineBasicBlock *Src2MBB) {
  if (Src1MBB->succ_empty()) {
    MachineLoop *LoopRep = MLI->getLoopFor(Src1MBB);
    if (LoopRep&& LoopRep == MLI->getLoopFor(Src2MBB)) {
      MachineBasicBlock *&TheEntry = LLInfoMap[LoopRep];
      if (TheEntry) {
        LLVM_DEBUG(dbgs() << "isLoopContBreakBlock yes src1 = BB"
                          << Src1MBB->getNumber() << " src2 = BB"
                          << Src2MBB->getNumber() << "\n";);
        return true;
      }
    }
  }
  return false;
}

int R600MachineCFGStructurizer::handleJumpintoIf(MachineBasicBlock *HeadMBB,
    MachineBasicBlock *TrueMBB, MachineBasicBlock *FalseMBB) {
  int Num = handleJumpintoIfImp(HeadMBB, TrueMBB, FalseMBB);
  if (Num == 0) {
    LLVM_DEBUG(dbgs() << "handleJumpintoIf swap trueBlk and FalseBlk"
                      << "\n";);
    Num = handleJumpintoIfImp(HeadMBB, FalseMBB, TrueMBB);
  }
  return Num;
}

int R600MachineCFGStructurizer::handleJumpintoIfImp(MachineBasicBlock *HeadMBB,
    MachineBasicBlock *TrueMBB, MachineBasicBlock *FalseMBB) {
  int Num = 0;
  MachineBasicBlock *DownBlk;

  //trueBlk could be the common post dominator
  DownBlk = TrueMBB;

  LLVM_DEBUG(dbgs() << "handleJumpintoIfImp head = BB" << HeadMBB->getNumber()
                    << " true = BB" << TrueMBB->getNumber()
                    << ", numSucc=" << TrueMBB->succ_size() << " false = BB"
                    << FalseMBB->getNumber() << "\n";);

  while (DownBlk) {
    LLVM_DEBUG(dbgs() << "check down = BB" << DownBlk->getNumber(););

```
**EN:** This section contains concrete logic for R600MachineCFGStructurizer::isSameloopDetachedContbreak. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600MachineCFGStructurizer::isSameloopDetachedContbreak`, `R600MachineCFGStructurizer::handleJumpintoIf`, `R600MachineCFGStructurizer::handleJumpintoIfImp`.
**CN:** 本节包含与 R600MachineCFGStructurizer::isSameloopDetachedContbreak 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600MachineCFGStructurizer::isSameloopDetachedContbreak`, `R600MachineCFGStructurizer::handleJumpintoIf`, `R600MachineCFGStructurizer::handleJumpintoIfImp`。

### Lines 1080-1112: Preprocessor guards and macros
```cpp
    if (singlePathTo(FalseMBB, DownBlk) == SinglePath_InPath) {
      LLVM_DEBUG(dbgs() << " working\n";);

      Num += cloneOnSideEntryTo(HeadMBB, TrueMBB, DownBlk);
      Num += cloneOnSideEntryTo(HeadMBB, FalseMBB, DownBlk);

      numClonedBlock += Num;
      Num += serialPatternMatch(*HeadMBB->succ_begin());
      Num += serialPatternMatch(*std::next(HeadMBB->succ_begin()));
      Num += ifPatternMatch(HeadMBB);
      assert(Num > 0);

      break;
    }
    LLVM_DEBUG(dbgs() << " not working\n";);
    DownBlk = (DownBlk->succ_size() == 1) ? (*DownBlk->succ_begin()) : nullptr;
  } // walk down the postDomTree

  return Num;
}

#ifndef NDEBUG
void R600MachineCFGStructurizer::showImproveSimpleJumpintoIf(
    MachineBasicBlock *HeadMBB, MachineBasicBlock *TrueMBB,
    MachineBasicBlock *FalseMBB, MachineBasicBlock *LandMBB, bool Detail) {
  dbgs() << "head = BB" << HeadMBB->getNumber()
         << " size = " << HeadMBB->size();
  if (Detail) {
    dbgs() << "\n";
    HeadMBB->print(dbgs());
    dbgs() << "\n";
  }

```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `std::next`, `R600MachineCFGStructurizer::showImproveSimpleJumpintoIf`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`std::next`, `R600MachineCFGStructurizer::showImproveSimpleJumpintoIf`。

### Lines 1113-1155: Preprocessor guards and macros
```cpp
  if (TrueMBB) {
    dbgs() << ", true = BB" << TrueMBB->getNumber() << " size = "
           << TrueMBB->size() << " numPred = " << TrueMBB->pred_size();
    if (Detail) {
      dbgs() << "\n";
      TrueMBB->print(dbgs());
      dbgs() << "\n";
    }
  }
  if (FalseMBB) {
    dbgs() << ", false = BB" << FalseMBB->getNumber() << " size = "
           << FalseMBB->size() << " numPred = " << FalseMBB->pred_size();
    if (Detail) {
      dbgs() << "\n";
      FalseMBB->print(dbgs());
      dbgs() << "\n";
    }
  }
  if (LandMBB) {
    dbgs() << ", land = BB" << LandMBB->getNumber() << " size = "
           << LandMBB->size() << " numPred = " << LandMBB->pred_size();
    if (Detail) {
      dbgs() << "\n";
      LandMBB->print(dbgs());
      dbgs() << "\n";
    }
  }

  dbgs() << "\n";
}
#endif

int R600MachineCFGStructurizer::improveSimpleJumpintoIf(MachineBasicBlock *HeadMBB,
    MachineBasicBlock *TrueMBB, MachineBasicBlock *FalseMBB,
    MachineBasicBlock **LandMBBPtr) {
  bool MigrateTrue = false;
  bool MigrateFalse = false;

  MachineBasicBlock *LandBlk = *LandMBBPtr;

  assert((!TrueMBB || TrueMBB->succ_size() <= 1)
         && (!FalseMBB || FalseMBB->succ_size() <= 1));

```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `R600MachineCFGStructurizer::improveSimpleJumpintoIf`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`R600MachineCFGStructurizer::improveSimpleJumpintoIf`。

### Lines 1156-1187: Declares needMigrateBlock
```cpp
  if (TrueMBB == FalseMBB)
    return 0;

  MigrateTrue = needMigrateBlock(TrueMBB);
  MigrateFalse = needMigrateBlock(FalseMBB);

  if (!MigrateTrue && !MigrateFalse)
    return 0;

  // If we need to migrate either trueBlk and falseBlk, migrate the rest that
  // have more than one predecessors.  without doing this, its predecessor
  // rather than headBlk will have undefined value in initReg.
  if (!MigrateTrue && TrueMBB && TrueMBB->pred_size() > 1)
    MigrateTrue = true;
  if (!MigrateFalse && FalseMBB && FalseMBB->pred_size() > 1)
    MigrateFalse = true;

  LLVM_DEBUG(
      dbgs() << "before improveSimpleJumpintoIf: ";
      showImproveSimpleJumpintoIf(HeadMBB, TrueMBB, FalseMBB, LandBlk, 0););

  // org: headBlk => if () {trueBlk} else {falseBlk} => landBlk
  //
  // new: headBlk => if () {initReg = 1; org trueBlk branch} else
  //      {initReg = 0; org falseBlk branch }
  //      => landBlk => if (initReg) {org trueBlk} else {org falseBlk}
  //      => org landBlk
  //      if landBlk->pred_size() > 2, put the about if-else inside
  //      if (initReg !=2) {...}
  //
  // add initReg = initVal to headBlk

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 1188-1231: Declares getCFGStructurizerRegClass
```cpp
  const TargetRegisterClass * I32RC = TRI->getCFGStructurizerRegClass(MVT::i32);
  if (!MigrateTrue || !MigrateFalse) {
    // XXX: We have an opportunity here to optimize the "branch into if" case
    // here.  Branch into if looks like this:
    //                        entry
    //                       /     |
    //           diamond_head       branch_from
    //             /      \           |
    // diamond_false        diamond_true
    //             \      /
    //               done
    //
    // The diamond_head block begins the "if" and the diamond_true block
    // is the block being "branched into".
    //
    // If MigrateTrue is true, then TrueBB is the block being "branched into"
    // and if MigrateFalse is true, then FalseBB is the block being
    // "branched into"
    //
    // Here is the pseudo code for how I think the optimization should work:
    // 1. Insert MOV GPR0, 0 before the branch instruction in diamond_head.
    // 2. Insert MOV GPR0, 1 before the branch instruction in branch_from.
    // 3. Move the branch instruction from diamond_head into its own basic
    //    block (new_block).
    // 4. Add an unconditional branch from diamond_head to new_block
    // 5. Replace the branch instruction in branch_from with an unconditional
    //    branch to new_block.  If branch_from has multiple predecessors, then
    //    we need to replace the True/False block in the branch
    //    instruction instead of replacing it.
    // 6. Change the condition of the branch instruction in new_block from
    //    COND to (COND || GPR0)
    //
    // In order insert these MOV instruction, we will need to use the
    // RegisterScavenger.  Usually liveness stops being tracked during
    // the late machine optimization passes, however if we implement
    // bool TargetRegisterInfo::requiresRegisterScavenging(
    //                                                const MachineFunction &MF)
    // and have it return true, liveness will be tracked correctly
    // by generic optimization passes.  We will also need to make sure that
    // all of our target-specific passes that run after regalloc and before
    // the CFGStructurizer track liveness and we will need to modify this pass
    // to correctly track liveness.
    //
    // After the above changes, the new CFG should look like this:
```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `TargetRegisterInfo::requiresRegisterScavenging`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`TargetRegisterInfo::requiresRegisterScavenging`。

### Lines 1232-1273: Conditional logic and checks
```cpp
    //                        entry
    //                       /     |
    //           diamond_head       branch_from
    //                       \     /
    //                      new_block
    //                      /      |
    //         diamond_false        diamond_true
    //                      \      /
    //                        done
    //
    // Without this optimization, we are forced to duplicate the diamond_true
    // block and we will end up with a CFG like this:
    //
    //                        entry
    //                       /     |
    //           diamond_head       branch_from
    //             /      \                   |
    // diamond_false        diamond_true      diamond_true (duplicate)
    //             \      /                   |
    //               done --------------------|
    //
    // Duplicating diamond_true can be very costly especially if it has a
    // lot of instructions.
    return 0;
  }

  int NumNewBlk = 0;

  bool LandBlkHasOtherPred = (LandBlk->pred_size() > 2);

  //insert R600::ENDIF to avoid special case "input landBlk == NULL"
  MachineBasicBlock::iterator I = insertInstrBefore(LandBlk, R600::ENDIF);

  if (LandBlkHasOtherPred) {
    report_fatal_error("Extra register needed to handle CFG");
    Register CmpResReg =
        HeadMBB->getParent()->getRegInfo().createVirtualRegister(I32RC);
    report_fatal_error("Extra compare instruction needed to handle CFG");
    insertCondBranchBefore(LandBlk, I, R600::IF_PREDICATE_SET,
        CmpResReg, DebugLoc());
  }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 1274-1316: Defines insertCondBranchBefore
```cpp
  // XXX: We are running this after RA, so creating virtual registers will
  // cause an assertion failure in the PostRA scheduling pass.
  Register InitReg =
      HeadMBB->getParent()->getRegInfo().createVirtualRegister(I32RC);
  insertCondBranchBefore(LandBlk, I, R600::IF_PREDICATE_SET, InitReg,
      DebugLoc());

  if (MigrateTrue) {
    migrateInstruction(TrueMBB, LandBlk, I);
    // need to uncondionally insert the assignment to ensure a path from its
    // predecessor rather than headBlk has valid value in initReg if
    // (initVal != 1).
    report_fatal_error("Extra register needed to handle CFG");
  }
  insertInstrBefore(I, R600::ELSE);

  if (MigrateFalse) {
    migrateInstruction(FalseMBB, LandBlk, I);
    // need to uncondionally insert the assignment to ensure a path from its
    // predecessor rather than headBlk has valid value in initReg if
    // (initVal != 0)
    report_fatal_error("Extra register needed to handle CFG");
  }

  if (LandBlkHasOtherPred) {
    // add endif
    insertInstrBefore(I, R600::ENDIF);

    // put initReg = 2 to other predecessors of landBlk
    for (MachineBasicBlock *MBB : LandBlk->predecessors())
      if (MBB != TrueMBB && MBB != FalseMBB)
        report_fatal_error("Extra register needed to handle CFG");
  }
  LLVM_DEBUG(
      dbgs() << "result from improveSimpleJumpintoIf: ";
      showImproveSimpleJumpintoIf(HeadMBB, TrueMBB, FalseMBB, LandBlk, 0););

  // update landBlk
  *LandMBBPtr = LandBlk;

  return NumNewBlk;
}

```
**EN:** This section contains concrete logic for insertCondBranchBefore. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 insertCondBranchBefore 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 1317-1359: Implements R600MachineCFGStructurizer::mergeSerialBlock
```cpp
void R600MachineCFGStructurizer::mergeSerialBlock(MachineBasicBlock *DstMBB,
    MachineBasicBlock *SrcMBB) {
  LLVM_DEBUG(dbgs() << "serialPattern BB" << DstMBB->getNumber() << " <= BB"
                    << SrcMBB->getNumber() << "\n";);
  DstMBB->splice(DstMBB->end(), SrcMBB, SrcMBB->begin(), SrcMBB->end());

  DstMBB->removeSuccessor(SrcMBB, true);
  cloneSuccessorList(DstMBB, SrcMBB);

  removeSuccessor(SrcMBB);
  MLI->removeBlock(SrcMBB);
  retireBlock(SrcMBB);
}

void R600MachineCFGStructurizer::mergeIfthenelseBlock(MachineInstr *BranchMI,
    MachineBasicBlock *MBB, MachineBasicBlock *TrueMBB,
    MachineBasicBlock *FalseMBB, MachineBasicBlock *LandMBB) {
  assert (TrueMBB);
  LLVM_DEBUG(dbgs() << "ifPattern BB" << MBB->getNumber(); dbgs() << "{  ";
             if (TrueMBB) { dbgs() << "BB" << TrueMBB->getNumber(); } dbgs()
             << "  } else ";
             dbgs() << "{  "; if (FalseMBB) {
               dbgs() << "BB" << FalseMBB->getNumber();
             } dbgs() << "  }\n ";
             dbgs() << "landBlock: "; if (!LandMBB) { dbgs() << "NULL"; } else {
               dbgs() << "BB" << LandMBB->getNumber();
             } dbgs() << "\n";);

  int OldOpcode = BranchMI->getOpcode();
  DebugLoc BranchDL = BranchMI->getDebugLoc();

//    transform to
//    if cond
//       trueBlk
//    else
//       falseBlk
//    endif
//    landBlk

  MachineBasicBlock::iterator I = BranchMI;
  insertCondBranchBefore(I, getBranchNzeroOpcode(OldOpcode),
      BranchDL);

```
**EN:** This section contains concrete logic for R600MachineCFGStructurizer::mergeSerialBlock. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600MachineCFGStructurizer::mergeSerialBlock`, `R600MachineCFGStructurizer::mergeIfthenelseBlock`.
**CN:** 本节包含与 R600MachineCFGStructurizer::mergeSerialBlock 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600MachineCFGStructurizer::mergeSerialBlock`, `R600MachineCFGStructurizer::mergeIfthenelseBlock`。

### Lines 1360-1396: Conditional logic and checks
```cpp
  if (TrueMBB) {
    MBB->splice(I, TrueMBB, TrueMBB->begin(), TrueMBB->end());
    MBB->removeSuccessor(TrueMBB, true);
    if (LandMBB && TrueMBB->succ_size()!=0)
      TrueMBB->removeSuccessor(LandMBB, true);
    retireBlock(TrueMBB);
    MLI->removeBlock(TrueMBB);
  }

  if (FalseMBB) {
    insertInstrBefore(I, R600::ELSE);
    MBB->splice(I, FalseMBB, FalseMBB->begin(),
                   FalseMBB->end());
    MBB->removeSuccessor(FalseMBB, true);
    if (LandMBB && !FalseMBB->succ_empty())
      FalseMBB->removeSuccessor(LandMBB, true);
    retireBlock(FalseMBB);
    MLI->removeBlock(FalseMBB);
  }
  insertInstrBefore(I, R600::ENDIF);

  BranchMI->eraseFromParent();

  if (LandMBB && TrueMBB && FalseMBB)
    MBB->addSuccessor(LandMBB);
}

void R600MachineCFGStructurizer::mergeLooplandBlock(MachineBasicBlock *DstBlk,
    MachineBasicBlock *LandMBB) {
  LLVM_DEBUG(dbgs() << "loopPattern header = BB" << DstBlk->getNumber()
                    << " land = BB" << LandMBB->getNumber() << "\n";);

  insertInstrBefore(DstBlk, R600::WHILELOOP, DebugLoc());
  insertInstrEnd(DstBlk, R600::ENDLOOP, DebugLoc());
  DstBlk->replaceSuccessor(DstBlk, LandMBB);
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `R600MachineCFGStructurizer::mergeLooplandBlock`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`R600MachineCFGStructurizer::mergeLooplandBlock`。

### Lines 1397-1433: Implements R600MachineCFGStructurizer::mergeLoopbreakBlock
```cpp
void R600MachineCFGStructurizer::mergeLoopbreakBlock(MachineBasicBlock *ExitingMBB,
    MachineBasicBlock *LandMBB) {
  LLVM_DEBUG(dbgs() << "loopbreakPattern exiting = BB"
                    << ExitingMBB->getNumber() << " land = BB"
                    << LandMBB->getNumber() << "\n";);
  MachineInstr *BranchMI = getLoopendBlockBranchInstr(ExitingMBB);
  assert(BranchMI && isCondBranch(BranchMI));
  const DebugLoc &DL = BranchMI->getDebugLoc();
  MachineBasicBlock *TrueBranch = getTrueBranch(BranchMI);
  MachineBasicBlock::iterator I = BranchMI;
  if (TrueBranch != LandMBB)
    reversePredicateSetter(I, *I->getParent());
  insertCondBranchBefore(ExitingMBB, I, R600::IF_PREDICATE_SET, R600::PREDICATE_BIT, DL);
  insertInstrBefore(I, R600::BREAK);
  insertInstrBefore(I, R600::ENDIF);
  //now branchInst can be erase safely
  BranchMI->eraseFromParent();
  //now take care of successors, retire blocks
  ExitingMBB->removeSuccessor(LandMBB, true);
}

void R600MachineCFGStructurizer::settleLoopcontBlock(MachineBasicBlock *ContingMBB,
    MachineBasicBlock *ContMBB) {
  LLVM_DEBUG(dbgs() << "settleLoopcontBlock conting = BB"
                    << ContingMBB->getNumber() << ", cont = BB"
                    << ContMBB->getNumber() << "\n";);

  MachineInstr *MI = getLoopendBlockBranchInstr(ContingMBB);
  if (MI) {
    assert(isCondBranch(MI));
    MachineBasicBlock::iterator I = MI;
    MachineBasicBlock *TrueBranch = getTrueBranch(MI);
    int OldOpcode = MI->getOpcode();
    const DebugLoc &DL = MI->getDebugLoc();

    bool UseContinueLogical = ((&*ContingMBB->rbegin()) == MI);

```
**EN:** This section contains concrete logic for R600MachineCFGStructurizer::mergeLoopbreakBlock. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600MachineCFGStructurizer::mergeLoopbreakBlock`, `R600MachineCFGStructurizer::settleLoopcontBlock`.
**CN:** 本节包含与 R600MachineCFGStructurizer::mergeLoopbreakBlock 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600MachineCFGStructurizer::mergeLoopbreakBlock`, `R600MachineCFGStructurizer::settleLoopcontBlock`。

### Lines 1434-1475: Conditional logic and checks
```cpp
    if (!UseContinueLogical) {
      int BranchOpcode =
          TrueBranch == ContMBB ? getBranchNzeroOpcode(OldOpcode) :
          getBranchZeroOpcode(OldOpcode);
      insertCondBranchBefore(I, BranchOpcode, DL);
      // insertEnd to ensure phi-moves, if exist, go before the continue-instr.
      insertInstrEnd(ContingMBB, R600::CONTINUE, DL);
      insertInstrEnd(ContingMBB, R600::ENDIF, DL);
    } else {
      int BranchOpcode =
          TrueBranch == ContMBB ? getContinueNzeroOpcode(OldOpcode) :
          getContinueZeroOpcode(OldOpcode);
      insertCondBranchBefore(I, BranchOpcode, DL);
    }

    MI->eraseFromParent();
  } else {
    // if we've arrived here then we've already erased the branch instruction
    // travel back up the basic block to see the last reference of our debug
    // location we've just inserted that reference here so it should be
    // representative insertEnd to ensure phi-moves, if exist, go before the
    // continue-instr.
    insertInstrEnd(ContingMBB, R600::CONTINUE,
        getLastDebugLocInBB(ContingMBB));
  }
}

int R600MachineCFGStructurizer::cloneOnSideEntryTo(MachineBasicBlock *PreMBB,
    MachineBasicBlock *SrcMBB, MachineBasicBlock *DstMBB) {
  int Cloned = 0;
  assert(PreMBB->isSuccessor(SrcMBB));
  while (SrcMBB && SrcMBB != DstMBB) {
    assert(SrcMBB->succ_size() == 1);
    if (SrcMBB->pred_size() > 1) {
      SrcMBB = cloneBlockForPredecessor(SrcMBB, PreMBB);
      ++Cloned;
    }

    PreMBB = SrcMBB;
    SrcMBB = *SrcMBB->succ_begin();
  }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `R600MachineCFGStructurizer::cloneOnSideEntryTo`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`R600MachineCFGStructurizer::cloneOnSideEntryTo`。

### Lines 1476-1519: Implements R600MachineCFGStructurizer::cloneBlockForPredecessor
```cpp
  return Cloned;
}

MachineBasicBlock *
R600MachineCFGStructurizer::cloneBlockForPredecessor(MachineBasicBlock *MBB,
    MachineBasicBlock *PredMBB) {
  assert(PredMBB->isSuccessor(MBB) && "succBlk is not a predecessor of curBlk");

  MachineBasicBlock *CloneMBB = clone(MBB);  //clone instructions
  replaceInstrUseOfBlockWith(PredMBB, MBB, CloneMBB);
  //srcBlk, oldBlk, newBlk

  PredMBB->replaceSuccessor(MBB, CloneMBB);

  // add all successor to cloneBlk
  cloneSuccessorList(CloneMBB, MBB);

  numClonedInstr += MBB->size();

  LLVM_DEBUG(dbgs() << "Cloned block: "
                    << "BB" << MBB->getNumber() << "size " << MBB->size()
                    << "\n";);

  SHOWNEWBLK(CloneMBB, "result of Cloned block: ");

  return CloneMBB;
}

void R600MachineCFGStructurizer::migrateInstruction(MachineBasicBlock *SrcMBB,
    MachineBasicBlock *DstMBB, MachineBasicBlock::iterator I) {
  MachineBasicBlock::iterator SpliceEnd;
  //look for the input branchinstr, not the AMDGPU branchinstr
  MachineInstr *BranchMI = getNormalBlockBranchInstr(SrcMBB);
  if (!BranchMI) {
    LLVM_DEBUG(dbgs() << "migrateInstruction don't see branch instr\n";);
    SpliceEnd = SrcMBB->end();
  } else {
    LLVM_DEBUG(dbgs() << "migrateInstruction see branch instr: " << *BranchMI);
    SpliceEnd = BranchMI;
  }
  LLVM_DEBUG(dbgs() << "migrateInstruction before splice dstSize = "
                    << DstMBB->size() << "srcSize = " << SrcMBB->size()
                    << "\n";);

```
**EN:** This section contains concrete logic for R600MachineCFGStructurizer::cloneBlockForPredecessor. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600MachineCFGStructurizer::cloneBlockForPredecessor`, `R600MachineCFGStructurizer::migrateInstruction`.
**CN:** 本节包含与 R600MachineCFGStructurizer::cloneBlockForPredecessor 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600MachineCFGStructurizer::cloneBlockForPredecessor`, `R600MachineCFGStructurizer::migrateInstruction`。

### Lines 1520-1560: Declares splice
```cpp
  //splice insert before insertPos
  DstMBB->splice(I, SrcMBB, SrcMBB->begin(), SpliceEnd);

  LLVM_DEBUG(dbgs() << "migrateInstruction after splice dstSize = "
                    << DstMBB->size() << "srcSize = " << SrcMBB->size()
                    << '\n';);
}

MachineBasicBlock *
R600MachineCFGStructurizer::normalizeInfiniteLoopExit(MachineLoop* LoopRep) {
  MachineBasicBlock *LoopHeader = LoopRep->getHeader();
  MachineBasicBlock *LoopLatch = LoopRep->getLoopLatch();

  if (!LoopHeader || !LoopLatch)
    return nullptr;
  MachineInstr *BranchMI = getLoopendBlockBranchInstr(LoopLatch);
  // Is LoopRep an infinite loop ?
  if (!BranchMI || !isUncondBranch(BranchMI))
    return nullptr;

  MachineBasicBlock *DummyExitBlk = FuncRep->CreateMachineBasicBlock();
  FuncRep->push_back(DummyExitBlk);  //insert to function
  SHOWNEWBLK(DummyExitBlk, "DummyExitBlock to normalize infiniteLoop: ");
  LLVM_DEBUG(dbgs() << "Old branch instr: " << *BranchMI << "\n";);
  LLVMContext &Ctx = LoopHeader->getParent()->getFunction().getContext();
  Ctx.emitError("Extra register needed to handle CFG");
  return nullptr;
}

void R600MachineCFGStructurizer::removeUnconditionalBranch(MachineBasicBlock *MBB) {
  MachineInstr *BranchMI;

  // I saw two unconditional branch in one basic block in example
  // test_fc_do_while_or.c need to fix the upstream on this to remove the loop.
  while ((BranchMI = getLoopendBlockBranchInstr(MBB))
          && isUncondBranch(BranchMI)) {
    LLVM_DEBUG(dbgs() << "Removing uncond branch instr: " << *BranchMI);
    BranchMI->eraseFromParent();
  }
}

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `R600MachineCFGStructurizer::normalizeInfiniteLoopExit`, `R600MachineCFGStructurizer::removeUnconditionalBranch`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`R600MachineCFGStructurizer::normalizeInfiniteLoopExit`, `R600MachineCFGStructurizer::removeUnconditionalBranch`。

### Lines 1561-1598: Implements R600MachineCFGStructurizer::removeRedundantConditionalBranch
```cpp
void R600MachineCFGStructurizer::removeRedundantConditionalBranch(
    MachineBasicBlock *MBB) {
  if (MBB->succ_size() != 2)
    return;
  MachineBasicBlock *MBB1 = *MBB->succ_begin();
  MachineBasicBlock *MBB2 = *std::next(MBB->succ_begin());
  if (MBB1 != MBB2)
    return;

  MachineInstr *BranchMI = getNormalBlockBranchInstr(MBB);
  assert(BranchMI && isCondBranch(BranchMI));
  LLVM_DEBUG(dbgs() << "Removing unneeded cond branch instr: " << *BranchMI);
  BranchMI->eraseFromParent();
  SHOWNEWBLK(MBB1, "Removing redundant successor");
  MBB->removeSuccessor(MBB1, true);
}

void R600MachineCFGStructurizer::addDummyExitBlock(
    SmallVectorImpl<MachineBasicBlock*> &RetMBB) {
  MachineBasicBlock *DummyExitBlk = FuncRep->CreateMachineBasicBlock();
  FuncRep->push_back(DummyExitBlk);  //insert to function
  insertInstrEnd(DummyExitBlk, R600::RETURN);

  for (MachineBasicBlock *MBB : RetMBB) {
    if (MachineInstr *MI = getReturnInstr(MBB))
      MI->eraseFromParent();
    MBB->addSuccessor(DummyExitBlk);
    LLVM_DEBUG(dbgs() << "Add dummyExitBlock to BB" << MBB->getNumber()
                      << " successors\n";);
  }
  SHOWNEWBLK(DummyExitBlk, "DummyExitBlock: ");
}

void R600MachineCFGStructurizer::removeSuccessor(MachineBasicBlock *MBB) {
  while (MBB->succ_size())
    MBB->removeSuccessor(*MBB->succ_begin());
}

```
**EN:** This section contains concrete logic for R600MachineCFGStructurizer::removeRedundantConditionalBranch. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600MachineCFGStructurizer::removeRedundantConditionalBranch`, `std::next`, `R600MachineCFGStructurizer::addDummyExitBlock`.
**CN:** 本节包含与 R600MachineCFGStructurizer::removeRedundantConditionalBranch 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600MachineCFGStructurizer::removeRedundantConditionalBranch`, `std::next`, `R600MachineCFGStructurizer::addDummyExitBlock`。

### Lines 1599-1629: Registers LLVM passes
```cpp
void R600MachineCFGStructurizer::recordSccnum(MachineBasicBlock *MBB,
    int SccNum) {
  BlockInformation *&srcBlkInfo = BlockInfoMap[MBB];
  if (!srcBlkInfo)
    srcBlkInfo = new BlockInformation();
  srcBlkInfo->SccNum = SccNum;
}

void R600MachineCFGStructurizer::retireBlock(MachineBasicBlock *MBB) {
  LLVM_DEBUG(dbgs() << "Retiring BB" << MBB->getNumber() << "\n";);

  BlockInformation *&SrcBlkInfo = BlockInfoMap[MBB];

  if (!SrcBlkInfo)
    SrcBlkInfo = new BlockInformation();

  SrcBlkInfo->IsRetired = true;
  assert(MBB->succ_empty() && MBB->pred_empty() && "can't retire block yet");
}

INITIALIZE_PASS_BEGIN(R600MachineCFGStructurizer, "amdgpustructurizer",
                      "AMDGPU CFG Structurizer", false, false)
INITIALIZE_PASS_DEPENDENCY(MachineDominatorTreeWrapperPass)
INITIALIZE_PASS_DEPENDENCY(MachinePostDominatorTreeWrapperPass)
INITIALIZE_PASS_DEPENDENCY(MachineLoopInfoWrapperPass)
INITIALIZE_PASS_END(R600MachineCFGStructurizer, "amdgpustructurizer",
                      "AMDGPU CFG Structurizer", false, false)

FunctionPass *llvm::createR600MachineCFGStructurizerPass() {
  return new R600MachineCFGStructurizer();
}
```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `R600MachineCFGStructurizer::recordSccnum`, `R600MachineCFGStructurizer::retireBlock`, `llvm::createR600MachineCFGStructurizerPass`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`R600MachineCFGStructurizer::recordSccnum`, `R600MachineCFGStructurizer::retireBlock`, `llvm::createR600MachineCFGStructurizerPass`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `BlockInformation`, `R600MachineCFGStructurizer`, `PathToKind`, `MachineFunctionPass::getAnalysisUsage`, `R600MachineCFGStructurizer::getSCCNum`, `R600MachineCFGStructurizer::getLoopLandInfo`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; LLVM pass integration / LLVM Pass 集成
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"MCTargetDesc/R600MCTargetDesc.h"`
- `"R600.h"`
- `"R600RegisterInfo.h"`
- `"R600Subtarget.h"`
- `"llvm/ADT/DepthFirstIterator.h"`
- `"llvm/ADT/SCCIterator.h"`
- `"llvm/ADT/Statistic.h"`
- `"llvm/CodeGen/MachineFunction.h"`
- `"llvm/CodeGen/MachineFunctionPass.h"`
- `"llvm/CodeGen/MachineJumpTableInfo.h"`
- `"llvm/CodeGen/MachineLoopInfo.h"`
- `"llvm/CodeGen/MachinePostDominators.h"`
- `"llvm/InitializePasses.h"`
