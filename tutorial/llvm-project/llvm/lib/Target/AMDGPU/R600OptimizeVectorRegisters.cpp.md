# R600OptimizeVectorRegisters.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/R600OptimizeVectorRegisters.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements R600OptimizeVectorRegisters for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 R600OptimizeVectorRegisters 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28: File banner, license, and overview
```cpp
//===- R600MergeVectorRegisters.cpp ---------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// This pass merges inputs of swizzeable instructions into vector sharing
/// common data and/or have enough undef subreg using swizzle abilities.
///
/// For instance let's consider the following pseudo code :
/// %5 = REG_SEQ %1, sub0, %2, sub1, %3, sub2, undef, sub3
/// ...
/// %7 = REG_SEQ %1, sub0, %3, sub1, undef, sub2, %4, sub3
/// (swizzable Inst) %7, SwizzleMask : sub0, sub1, sub2, sub3
///
/// is turned into :
/// %5 = REG_SEQ %1, sub0, %2, sub1, %3, sub2, undef, sub3
/// ...
/// %7 = INSERT_SUBREG %4, sub3
/// (swizzable Inst) %7, SwizzleMask : sub0, sub2, sub1, sub3
///
/// This allow regalloc to reduce register pressure for vector registers and
/// to reduce MOV count.
//===----------------------------------------------------------------------===//

```
**EN:** This opening section establishes the file identity, licensing terms, and high-level intent before the backend-specific code begins.
**CN:** 开头部分先给出文件标识、许可证信息以及总体意图，为后续后端专用代码建立上下文。

### Lines 29-54: Header dependencies and setup
```cpp
#include "MCTargetDesc/R600MCTargetDesc.h"
#include "R600.h"
#include "R600Defines.h"
#include "R600Subtarget.h"
#include "llvm/CodeGen/MachineDominators.h"
#include "llvm/CodeGen/MachineLoopInfo.h"

using namespace llvm;

#define DEBUG_TYPE "vec-merger"

static bool isImplicitlyDef(MachineRegisterInfo &MRI, Register Reg) {
  if (Reg.isPhysical())
    return false;
  const MachineInstr *MI = MRI.getUniqueVRegDef(Reg);
  return MI && MI->isImplicitDef();
}

namespace {

class RegSeqInfo {
public:
  MachineInstr *Instr;
  DenseMap<Register, unsigned> RegToChan;
  std::vector<Register> UndefReg;

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available. Main symbols: `RegSeqInfo`.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。 主要符号：`RegSeqInfo`。

### Lines 55-83: Declares class R600VectorRegMerger
```cpp
  RegSeqInfo(MachineRegisterInfo &MRI, MachineInstr *MI) : Instr(MI) {
    assert(MI->getOpcode() == R600::REG_SEQUENCE);
    for (unsigned i = 1, e = Instr->getNumOperands(); i < e; i+=2) {
      MachineOperand &MO = Instr->getOperand(i);
      unsigned Chan = Instr->getOperand(i + 1).getImm();
      if (isImplicitlyDef(MRI, MO.getReg()))
        UndefReg.emplace_back(Chan);
      else
        RegToChan[MO.getReg()] = Chan;
    }
  }

  RegSeqInfo() = default;

  bool operator==(const RegSeqInfo &RSI) const {
    return RSI.Instr == Instr;
  }
};

class R600VectorRegMerger : public MachineFunctionPass {
private:
  using InstructionSetMap = DenseMap<unsigned, std::vector<MachineInstr *>>;

  MachineRegisterInfo *MRI;
  const R600InstrInfo *TII = nullptr;
  DenseMap<MachineInstr *, RegSeqInfo> PreviousRegSeq;
  InstructionSetMap PreviousRegSeqByReg;
  InstructionSetMap PreviousRegSeqByUndefCount;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `R600VectorRegMerger`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`R600VectorRegMerger`。

### Lines 84-116: Declares canSwizzle
```cpp
  bool canSwizzle(const MachineInstr &MI) const;
  bool areAllUsesSwizzeable(Register Reg) const;
  void SwizzleInput(MachineInstr &,
      const std::vector<std::pair<unsigned, unsigned>> &RemapChan) const;
  bool tryMergeVector(const RegSeqInfo *Untouched, RegSeqInfo *ToMerge,
      std::vector<std::pair<unsigned, unsigned>> &Remap) const;
  bool tryMergeUsingCommonSlot(RegSeqInfo &RSI, RegSeqInfo &CompatibleRSI,
      std::vector<std::pair<unsigned, unsigned>> &RemapChan);
  bool tryMergeUsingFreeSlot(RegSeqInfo &RSI, RegSeqInfo &CompatibleRSI,
      std::vector<std::pair<unsigned, unsigned>> &RemapChan);
  MachineInstr *RebuildVector(RegSeqInfo *MI, const RegSeqInfo *BaseVec,
      const std::vector<std::pair<unsigned, unsigned>> &RemapChan) const;
  void RemoveMI(MachineInstr *);
  void trackRSI(const RegSeqInfo &RSI);

public:
  static char ID;

  R600VectorRegMerger() : MachineFunctionPass(ID) {}

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    AU.addRequired<MachineDominatorTreeWrapperPass>();
    AU.addPreserved<MachineDominatorTreeWrapperPass>();
    AU.addRequired<MachineLoopInfoWrapperPass>();
    AU.addPreserved<MachineLoopInfoWrapperPass>();
    MachineFunctionPass::getAnalysisUsage(AU);
  }

  MachineFunctionProperties getRequiredProperties() const override {
    return MachineFunctionProperties().setIsSSA();
  }

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `MachineFunctionPass::getAnalysisUsage`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`MachineFunctionPass::getAnalysisUsage`。

### Lines 117-147: Registers LLVM passes
```cpp
  StringRef getPassName() const override {
    return "R600 Vector Registers Merge Pass";
  }

  bool runOnMachineFunction(MachineFunction &Fn) override;
};

} // end anonymous namespace

INITIALIZE_PASS_BEGIN(R600VectorRegMerger, DEBUG_TYPE,
                     "R600 Vector Reg Merger", false, false)
INITIALIZE_PASS_END(R600VectorRegMerger, DEBUG_TYPE,
                    "R600 Vector Reg Merger", false, false)

char R600VectorRegMerger::ID = 0;

char &llvm::R600VectorRegMergerID = R600VectorRegMerger::ID;

bool R600VectorRegMerger::canSwizzle(const MachineInstr &MI)
    const {
  if (TII->get(MI.getOpcode()).TSFlags & R600_InstFlag::TEX_INST)
    return true;
  switch (MI.getOpcode()) {
  case R600::R600_ExportSwz:
  case R600::EG_ExportSwz:
    return true;
  default:
    return false;
  }
}

```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `R600VectorRegMerger::canSwizzle`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`R600VectorRegMerger::canSwizzle`。

### Lines 148-176: Implements R600VectorRegMerger::tryMergeVector
```cpp
bool R600VectorRegMerger::tryMergeVector(const RegSeqInfo *Untouched,
    RegSeqInfo *ToMerge, std::vector< std::pair<unsigned, unsigned>> &Remap)
    const {
  unsigned CurrentUndexIdx = 0;
  for (auto &It : ToMerge->RegToChan) {
    auto PosInUntouched = Untouched->RegToChan.find(It.first);
    if (PosInUntouched != Untouched->RegToChan.end()) {
      Remap.emplace_back(It.second, (*PosInUntouched).second);
      continue;
    }
    if (CurrentUndexIdx >= Untouched->UndefReg.size())
      return false;
    Remap.emplace_back(It.second, Untouched->UndefReg[CurrentUndexIdx++]);
  }

  return true;
}

static
unsigned getReassignedChan(
    const std::vector<std::pair<unsigned, unsigned>> &RemapChan,
    unsigned Chan) {
  for (const auto &J : RemapChan) {
    if (J.first == Chan)
      return J.second;
  }
  llvm_unreachable("Chan wasn't reassigned");
}

```
**EN:** This section contains concrete logic for R600VectorRegMerger::tryMergeVector. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600VectorRegMerger::tryMergeVector`.
**CN:** 本节包含与 R600VectorRegMerger::tryMergeVector 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600VectorRegMerger::tryMergeVector`。

### Lines 177-210: Implements R600VectorRegMerger::RebuildVector
```cpp
MachineInstr *R600VectorRegMerger::RebuildVector(
    RegSeqInfo *RSI, const RegSeqInfo *BaseRSI,
    const std::vector<std::pair<unsigned, unsigned>> &RemapChan) const {
  Register Reg = RSI->Instr->getOperand(0).getReg();
  MachineBasicBlock::iterator Pos = RSI->Instr;
  MachineBasicBlock &MBB = *Pos->getParent();
  const DebugLoc &DL = Pos->getDebugLoc();

  Register SrcVec = BaseRSI->Instr->getOperand(0).getReg();
  DenseMap<Register, unsigned> UpdatedRegToChan = BaseRSI->RegToChan;
  std::vector<Register> UpdatedUndef = BaseRSI->UndefReg;
  for (const auto &It : RSI->RegToChan) {
    Register DstReg = MRI->createVirtualRegister(&R600::R600_Reg128RegClass);
    unsigned SubReg = It.first;
    unsigned Swizzle = It.second;
    unsigned Chan = getReassignedChan(RemapChan, Swizzle);

    MachineInstr *Tmp = BuildMI(MBB, Pos, DL, TII->get(R600::INSERT_SUBREG),
        DstReg)
        .addReg(SrcVec)
        .addReg(SubReg)
        .addImm(Chan);
    UpdatedRegToChan[SubReg] = Chan;
    std::vector<Register>::iterator ChanPos = llvm::find(UpdatedUndef, Chan);
    if (ChanPos != UpdatedUndef.end())
      UpdatedUndef.erase(ChanPos);
    assert(!is_contained(UpdatedUndef, Chan) &&
           "UpdatedUndef shouldn't contain Chan more than once!");
    LLVM_DEBUG(dbgs() << "    ->"; Tmp->dump(););
    (void)Tmp;
    SrcVec = DstReg;
  }
  MachineInstr *NewMI =
      BuildMI(MBB, Pos, DL, TII->get(R600::COPY), Reg).addReg(SrcVec);
```
**EN:** This section contains concrete logic for R600VectorRegMerger::RebuildVector. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600VectorRegMerger::RebuildVector`, `llvm::find`.
**CN:** 本节包含与 R600VectorRegMerger::RebuildVector 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600VectorRegMerger::RebuildVector`, `llvm::find`。

### Lines 211-240: Declares LLVM_DEBUG
```cpp
  LLVM_DEBUG(dbgs() << "    ->"; NewMI->dump(););

  LLVM_DEBUG(dbgs() << "  Updating Swizzle:\n");
  for (MachineRegisterInfo::use_instr_iterator It = MRI->use_instr_begin(Reg),
      E = MRI->use_instr_end(); It != E; ++It) {
    LLVM_DEBUG(dbgs() << "    "; (*It).dump(); dbgs() << "    ->");
    SwizzleInput(*It, RemapChan);
    LLVM_DEBUG((*It).dump());
  }
  RSI->Instr->eraseFromParent();

  // Update RSI
  RSI->Instr = NewMI;
  RSI->RegToChan = std::move(UpdatedRegToChan);
  RSI->UndefReg = std::move(UpdatedUndef);

  return NewMI;
}

void R600VectorRegMerger::RemoveMI(MachineInstr *MI) {
  for (auto &It : PreviousRegSeqByReg) {
    std::vector<MachineInstr *> &MIs = It.second;
    MIs.erase(llvm::find(MIs, MI), MIs.end());
  }
  for (auto &It : PreviousRegSeqByUndefCount) {
    std::vector<MachineInstr *> &MIs = It.second;
    MIs.erase(llvm::find(MIs, MI), MIs.end());
  }
}

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `std::move`, `R600VectorRegMerger::RemoveMI`, `llvm::find`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`std::move`, `R600VectorRegMerger::RemoveMI`, `llvm::find`。

### Lines 241-263: Implements R600VectorRegMerger::SwizzleInput
```cpp
void R600VectorRegMerger::SwizzleInput(MachineInstr &MI,
    const std::vector<std::pair<unsigned, unsigned>> &RemapChan) const {
  unsigned Offset;
  if (TII->get(MI.getOpcode()).TSFlags & R600_InstFlag::TEX_INST)
    Offset = 2;
  else
    Offset = 3;
  for (unsigned i = 0; i < 4; i++) {
    unsigned Swizzle = MI.getOperand(i + Offset).getImm() + 1;
    for (const auto &J : RemapChan) {
      if (J.first == Swizzle) {
        MI.getOperand(i + Offset).setImm(J.second - 1);
        break;
      }
    }
  }
}

bool R600VectorRegMerger::areAllUsesSwizzeable(Register Reg) const {
  return llvm::all_of(MRI->use_instructions(Reg),
                      [&](const MachineInstr &MI) { return canSwizzle(MI); });
}

```
**EN:** This section contains concrete logic for R600VectorRegMerger::SwizzleInput. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600VectorRegMerger::SwizzleInput`, `R600VectorRegMerger::areAllUsesSwizzeable`, `llvm::all_of`.
**CN:** 本节包含与 R600VectorRegMerger::SwizzleInput 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600VectorRegMerger::SwizzleInput`, `R600VectorRegMerger::areAllUsesSwizzeable`, `llvm::all_of`。

### Lines 264-297: Implements R600VectorRegMerger::tryMergeUsingCommonSlot
```cpp
bool R600VectorRegMerger::tryMergeUsingCommonSlot(RegSeqInfo &RSI,
    RegSeqInfo &CompatibleRSI,
    std::vector<std::pair<unsigned, unsigned>> &RemapChan) {
  for (MachineInstr::mop_iterator MOp = RSI.Instr->operands_begin(),
      MOE = RSI.Instr->operands_end(); MOp != MOE; ++MOp) {
    if (!MOp->isReg())
      continue;
    auto &Insts = PreviousRegSeqByReg[MOp->getReg()];
    if (Insts.empty())
      continue;
    for (MachineInstr *MI : Insts) {
      CompatibleRSI = PreviousRegSeq[MI];
      if (RSI == CompatibleRSI)
        continue;
      if (tryMergeVector(&CompatibleRSI, &RSI, RemapChan))
        return true;
    }
  }
  return false;
}

bool R600VectorRegMerger::tryMergeUsingFreeSlot(RegSeqInfo &RSI,
    RegSeqInfo &CompatibleRSI,
    std::vector<std::pair<unsigned, unsigned>> &RemapChan) {
  unsigned NeededUndefs = 4 - RSI.UndefReg.size();
  std::vector<MachineInstr *> &MIs =
      PreviousRegSeqByUndefCount[NeededUndefs];
  if (MIs.empty())
    return false;
  CompatibleRSI = PreviousRegSeq[MIs.back()];
  tryMergeVector(&CompatibleRSI, &RSI, RemapChan);
  return true;
}

```
**EN:** This section contains concrete logic for R600VectorRegMerger::tryMergeUsingCommonSlot. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600VectorRegMerger::tryMergeUsingCommonSlot`, `R600VectorRegMerger::tryMergeUsingFreeSlot`.
**CN:** 本节包含与 R600VectorRegMerger::tryMergeUsingCommonSlot 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600VectorRegMerger::tryMergeUsingCommonSlot`, `R600VectorRegMerger::tryMergeUsingFreeSlot`。

### Lines 298-331: Implements R600VectorRegMerger::trackRSI
```cpp
void R600VectorRegMerger::trackRSI(const RegSeqInfo &RSI) {
  for (DenseMap<Register, unsigned>::const_iterator
  It = RSI.RegToChan.begin(), E = RSI.RegToChan.end(); It != E; ++It) {
    PreviousRegSeqByReg[(*It).first].push_back(RSI.Instr);
  }
  PreviousRegSeqByUndefCount[RSI.UndefReg.size()].push_back(RSI.Instr);
  PreviousRegSeq[RSI.Instr] = RSI;
}

bool R600VectorRegMerger::runOnMachineFunction(MachineFunction &Fn) {
  if (skipFunction(Fn.getFunction()))
    return false;

  const R600Subtarget &ST = Fn.getSubtarget<R600Subtarget>();
  TII = ST.getInstrInfo();
  MRI = &Fn.getRegInfo();

  for (MachineBasicBlock &MB : Fn) {
    PreviousRegSeq.clear();
    PreviousRegSeqByReg.clear();
    PreviousRegSeqByUndefCount.clear();

    for (MachineBasicBlock::iterator MII = MB.begin(), MIIE = MB.end();
         MII != MIIE; ++MII) {
      MachineInstr &MI = *MII;
      if (MI.getOpcode() != R600::REG_SEQUENCE) {
        if (TII->get(MI.getOpcode()).TSFlags & R600_InstFlag::TEX_INST) {
          Register Reg = MI.getOperand(1).getReg();
          for (MachineInstr &DefMI : MRI->def_instructions(Reg))
            RemoveMI(&DefMI);
        }
        continue;
      }

```
**EN:** This section contains concrete logic for R600VectorRegMerger::trackRSI. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600VectorRegMerger::trackRSI`, `R600VectorRegMerger::runOnMachineFunction`.
**CN:** 本节包含与 R600VectorRegMerger::trackRSI 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600VectorRegMerger::trackRSI`, `R600VectorRegMerger::runOnMachineFunction`。

### Lines 332-365: Conditional logic and checks
```cpp
      RegSeqInfo RSI(*MRI, &MI);

      // All uses of MI are swizzeable ?
      Register Reg = MI.getOperand(0).getReg();
      if (!areAllUsesSwizzeable(Reg))
        continue;

      LLVM_DEBUG({
        dbgs() << "Trying to optimize ";
        MI.dump();
      });

      RegSeqInfo CandidateRSI;
      std::vector<std::pair<unsigned, unsigned>> RemapChan;
      LLVM_DEBUG(dbgs() << "Using common slots...\n";);
      if (tryMergeUsingCommonSlot(RSI, CandidateRSI, RemapChan)) {
        // Remove CandidateRSI mapping
        RemoveMI(CandidateRSI.Instr);
        MII = RebuildVector(&RSI, &CandidateRSI, RemapChan);
        trackRSI(RSI);
        continue;
      }
      LLVM_DEBUG(dbgs() << "Using free slots...\n";);
      RemapChan.clear();
      if (tryMergeUsingFreeSlot(RSI, CandidateRSI, RemapChan)) {
        RemoveMI(CandidateRSI.Instr);
        MII = RebuildVector(&RSI, &CandidateRSI, RemapChan);
        trackRSI(RSI);
        continue;
      }
      //Failed to merge
      trackRSI(RSI);
    }
  }
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 366-371: Implements llvm::createR600VectorRegMerger
```cpp
  return false;
}

llvm::FunctionPass *llvm::createR600VectorRegMerger() {
  return new R600VectorRegMerger();
}
```
**EN:** This section contains concrete logic for llvm::createR600VectorRegMerger. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `llvm::createR600VectorRegMerger`.
**CN:** 本节包含与 llvm::createR600VectorRegMerger 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`llvm::createR600VectorRegMerger`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `RegSeqInfo`, `R600VectorRegMerger`, `MachineFunctionPass::getAnalysisUsage`, `R600VectorRegMerger::canSwizzle`, `R600VectorRegMerger::tryMergeVector`, `R600VectorRegMerger::RebuildVector`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; LLVM pass integration / LLVM Pass 集成
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"MCTargetDesc/R600MCTargetDesc.h"`
- `"R600.h"`
- `"R600Defines.h"`
- `"R600Subtarget.h"`
- `"llvm/CodeGen/MachineDominators.h"`
- `"llvm/CodeGen/MachineLoopInfo.h"`
