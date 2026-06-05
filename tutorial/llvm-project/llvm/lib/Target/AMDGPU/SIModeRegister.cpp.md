# SIModeRegister.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/SIModeRegister.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements SIModeRegister for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 SIModeRegister 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-34: File banner, includes, and setup
```cpp
//===-- SIModeRegister.cpp - Mode Register --------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
/// \file
/// This pass inserts changes to the Mode register settings as required.
/// Note that currently it only deals with the Double Precision Floating Point
/// rounding mode setting, but is intended to be generic enough to be easily
/// expanded.
///
//===----------------------------------------------------------------------===//
//
#include "AMDGPU.h"
#include "GCNSubtarget.h"
#include "MCTargetDesc/AMDGPUMCTargetDesc.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include <queue>

#define DEBUG_TYPE "si-mode-register"

STATISTIC(NumSetregInserted, "Number of setreg of mode register inserted.");

using namespace llvm;

struct Status {
  // Mask is a bitmask where a '1' indicates the corresponding Mode bit has a
  // known value
  unsigned Mask = 0;
  unsigned Mode = 0;

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `Status`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`Status`。

### Lines 35-65: Declares Status
```cpp
  Status() = default;

  Status(unsigned NewMask, unsigned NewMode) : Mask(NewMask), Mode(NewMode) {
    Mode &= Mask;
  };

  // merge two status values such that only values that don't conflict are
  // preserved
  Status merge(const Status &S) const {
    return Status((Mask | S.Mask), ((Mode & ~S.Mask) | (S.Mode & S.Mask)));
  }

  // merge an unknown value by using the unknown value's mask to remove bits
  // from the result
  Status mergeUnknown(unsigned newMask) {
    return Status(Mask & ~newMask, Mode & ~newMask);
  }

  // intersect two Status values to produce a mode and mask that is a subset
  // of both values
  Status intersect(const Status &S) const {
    unsigned NewMask = (Mask & S.Mask) & (Mode ^ ~S.Mode);
    unsigned NewMode = (Mode & NewMask);
    return Status(NewMask, NewMode);
  }

  // produce the delta required to change the Mode to the required Mode
  Status delta(const Status &S) const {
    return Status((S.Mask & (Mode ^ S.Mode)) | (~Mask & S.Mask), S.Mode);
  }

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 66-96: Declares class BlockData
```cpp
  bool operator==(const Status &S) const {
    return (Mask == S.Mask) && (Mode == S.Mode);
  }

  bool operator!=(const Status &S) const { return !(*this == S); }

  bool isCompatible(Status &S) {
    return ((Mask & S.Mask) == S.Mask) && ((Mode & S.Mask) == S.Mode);
  }

  bool isCombinable(Status &S) { return !(Mask & S.Mask) || isCompatible(S); }
};

class BlockData {
public:
  // The Status that represents the mode register settings required by the
  // FirstInsertionPoint (if any) in this block. Calculated in Phase 1.
  Status Require;

  // The Status that represents the net changes to the Mode register made by
  // this block, Calculated in Phase 1.
  Status Change;

  // The Status that represents the mode register settings on exit from this
  // block. Calculated in Phase 2.
  Status Exit;

  // The Status that represents the intersection of exit Mode register settings
  // from all predecessor blocks. Calculated in Phase 2, and used by Phase 3.
  Status Pred;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `BlockData`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`BlockData`。

### Lines 97-129: Declares class SIModeRegister
```cpp
  // In Phase 1 we record the first instruction that has a mode requirement,
  // which is used in Phase 3 if we need to insert a mode change.
  MachineInstr *FirstInsertionPoint = nullptr;

  // A flag to indicate whether an Exit value has been set (we can't tell by
  // examining the Exit value itself as all values may be valid results).
  bool ExitSet = false;

  BlockData() = default;
};

namespace {

class SIModeRegister {
public:
  std::vector<std::unique_ptr<BlockData>> BlockInfo;
  std::queue<MachineBasicBlock *> Phase2List;

  // The default mode register setting currently only caters for the floating
  // point double precision rounding mode.
  // We currently assume the default rounding mode is Round to Nearest
  // NOTE: this should come from a per function rounding mode setting once such
  // a setting exists.
  unsigned DefaultMode = FP_ROUND_ROUND_TO_NEAREST;
  Status DefaultStatus =
      Status(FP_ROUND_MODE_DP(0x3), FP_ROUND_MODE_DP(DefaultMode));

  bool Changed = false;

  bool run(MachineFunction &MF);

  void processBlockPhase1(MachineBasicBlock &MBB, const SIInstrInfo *TII);

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `SIModeRegister`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`SIModeRegister`。

### Lines 130-161: Registers LLVM passes
```cpp
  void processBlockPhase2(MachineBasicBlock &MBB, const SIInstrInfo *TII);

  void processBlockPhase3(MachineBasicBlock &MBB, const SIInstrInfo *TII);

  Status getInstructionMode(MachineInstr &MI, const SIInstrInfo *TII);

  void insertSetreg(MachineBasicBlock &MBB, MachineInstr *I,
                    const SIInstrInfo *TII, Status InstrMode);
};

class SIModeRegisterLegacy : public MachineFunctionPass {
public:
  static char ID;

  SIModeRegisterLegacy() : MachineFunctionPass(ID) {}

  bool runOnMachineFunction(MachineFunction &MF) override;

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    MachineFunctionPass::getAnalysisUsage(AU);
  }
};
} // End anonymous namespace.

INITIALIZE_PASS(SIModeRegisterLegacy, DEBUG_TYPE,
                "Insert required mode register values", false, false)

char SIModeRegisterLegacy::ID = 0;

char &llvm::SIModeRegisterID = SIModeRegisterLegacy::ID;

```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `SIModeRegisterLegacy`, `MachineFunctionPass::getAnalysisUsage`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`SIModeRegisterLegacy`, `MachineFunctionPass::getAnalysisUsage`。

### Lines 162-195: Implements llvm::createSIModeRegisterPass
```cpp
FunctionPass *llvm::createSIModeRegisterPass() {
  return new SIModeRegisterLegacy();
}

// Determine the Mode register setting required for this instruction.
// Instructions which don't use the Mode register return a null Status.
// Note this currently only deals with instructions that use the floating point
// double precision setting.
Status SIModeRegister::getInstructionMode(MachineInstr &MI,
                                          const SIInstrInfo *TII) {
  unsigned Opcode = MI.getOpcode();
  if (TII->usesFPDPRounding(MI) ||
      Opcode == AMDGPU::FPTRUNC_ROUND_F16_F32_PSEUDO ||
      Opcode == AMDGPU::FPTRUNC_ROUND_F16_F32_PSEUDO_fake16_e32 ||
      Opcode == AMDGPU::FPTRUNC_ROUND_F16_F32_PSEUDO_t16_e64 ||
      Opcode == AMDGPU::FPTRUNC_ROUND_F32_F64_PSEUDO) {
    switch (Opcode) {
    case AMDGPU::V_INTERP_P1LL_F16:
    case AMDGPU::V_INTERP_P1LV_F16:
    case AMDGPU::V_INTERP_P2_F16:
      // f16 interpolation instructions need double precision round to zero
      return Status(FP_ROUND_MODE_DP(3),
                    FP_ROUND_MODE_DP(FP_ROUND_ROUND_TO_ZERO));
    case AMDGPU::FPTRUNC_ROUND_F16_F32_PSEUDO: {
      unsigned Mode = MI.getOperand(2).getImm();
      MI.removeOperand(2);
      MI.setDesc(TII->get(AMDGPU::V_CVT_F16_F32_e32));
      return Status(FP_ROUND_MODE_DP(3), FP_ROUND_MODE_DP(Mode));
    }
    case AMDGPU::FPTRUNC_ROUND_F16_F32_PSEUDO_fake16_e32: {
      unsigned Mode = MI.getOperand(2).getImm();
      MI.removeOperand(2);
      MI.setDesc(TII->get(AMDGPU::V_CVT_F16_F32_fake16_e32));
      return Status(FP_ROUND_MODE_DP(3), FP_ROUND_MODE_DP(Mode));
```
**EN:** This section contains concrete logic for llvm::createSIModeRegisterPass. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `llvm::createSIModeRegisterPass`, `SIModeRegister::getInstructionMode`.
**CN:** 本节包含与 llvm::createSIModeRegisterPass 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`llvm::createSIModeRegisterPass`, `SIModeRegister::getInstructionMode`。

### Lines 196-215: Result computation and returns
```cpp
    }
    case AMDGPU::FPTRUNC_ROUND_F16_F32_PSEUDO_t16_e64: {
      unsigned Mode = MI.getOperand(6).getImm();
      MI.removeOperand(6);
      MI.setDesc(TII->get(AMDGPU::V_CVT_F16_F32_t16_e64));
      return Status(FP_ROUND_MODE_DP(3), FP_ROUND_MODE_DP(Mode));
    }
    case AMDGPU::FPTRUNC_ROUND_F32_F64_PSEUDO: {
      unsigned Mode = MI.getOperand(2).getImm();
      MI.removeOperand(2);
      MI.setDesc(TII->get(AMDGPU::V_CVT_F32_F64_e32));
      return Status(FP_ROUND_MODE_DP(3), FP_ROUND_MODE_DP(Mode));
    }
    default:
      return DefaultStatus;
    }
  }
  return Status();
}

```
**EN:** This section finalizes intermediate state and returns the value that the caller or pass pipeline needs.
**CN:** 本节整理中间状态并返回调用者或 Pass 流水线所需的结果。

### Lines 216-236: Implements SIModeRegister::insertSetreg
```cpp
// Insert a setreg instruction to update the Mode register.
// It is possible (though unlikely) for an instruction to require a change to
// the value of disjoint parts of the Mode register when we don't know the
// value of the intervening bits. In that case we need to use more than one
// setreg instruction.
void SIModeRegister::insertSetreg(MachineBasicBlock &MBB, MachineInstr *MI,
                                  const SIInstrInfo *TII, Status InstrMode) {
  while (InstrMode.Mask) {
    unsigned Offset = llvm::countr_zero<unsigned>(InstrMode.Mask);
    unsigned Width = llvm::countr_one<unsigned>(InstrMode.Mask >> Offset);
    unsigned Value = (InstrMode.Mode >> Offset) & ((1 << Width) - 1);
    using namespace AMDGPU::Hwreg;
    BuildMI(MBB, MI, nullptr, TII->get(AMDGPU::S_SETREG_IMM32_B32))
        .addImm(Value)
        .addImm(HwregEncoding::encode(ID_MODE, Offset, Width));
    ++NumSetregInserted;
    Changed = true;
    InstrMode.Mask &= ~(((1 << Width) - 1) << Offset);
  }
}

```
**EN:** This section contains concrete logic for SIModeRegister::insertSetreg. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SIModeRegister::insertSetreg`, `HwregEncoding::encode`.
**CN:** 本节包含与 SIModeRegister::insertSetreg 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SIModeRegister::insertSetreg`, `HwregEncoding::encode`。

### Lines 237-270: Conditional logic and checks
```cpp
// In Phase 1 we iterate through the instructions of the block and for each
// instruction we get its mode usage. If the instruction uses the Mode register
// we:
// - update the Change status, which tracks the changes to the Mode register
//   made by this block
// - if this instruction's requirements are compatible with the current setting
//   of the Mode register we merge the modes
// - if it isn't compatible and an InsertionPoint isn't set, then we set the
//   InsertionPoint to the current instruction, and we remember the current
//   mode
// - if it isn't compatible and InsertionPoint is set we insert a seteg before
//   that instruction (unless this instruction forms part of the block's
//   entry requirements in which case the insertion is deferred until Phase 3
//   when predecessor exit values are known), and move the insertion point to
//   this instruction
// - if this is a setreg instruction we treat it as an incompatible instruction.
//   This is sub-optimal but avoids some nasty corner cases, and is expected to
//   occur very rarely.
// - on exit we have set the Require, Change, and initial Exit modes.
void SIModeRegister::processBlockPhase1(MachineBasicBlock &MBB,
                                        const SIInstrInfo *TII) {
  auto NewInfo = std::make_unique<BlockData>();
  MachineInstr *InsertionPoint = nullptr;
  // RequirePending is used to indicate whether we are collecting the initial
  // requirements for the block, and need to defer the first InsertionPoint to
  // Phase 3. It is set to false once we have set FirstInsertionPoint, or when
  // we discover an explicit setreg that means this block doesn't have any
  // initial requirements.
  bool RequirePending = true;
  Status IPChange;
  for (MachineInstr &MI : MBB) {
    Status InstrMode = getInstructionMode(MI, TII);
    if (MI.getOpcode() == AMDGPU::S_SETREG_B32 ||
        MI.getOpcode() == AMDGPU::S_SETREG_B32_mode ||
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `SIModeRegister::processBlockPhase1`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`SIModeRegister::processBlockPhase1`。

### Lines 271-304: Namespace declarations and scope setup
```cpp
        MI.getOpcode() == AMDGPU::S_SETREG_IMM32_B32 ||
        MI.getOpcode() == AMDGPU::S_SETREG_IMM32_B32_mode) {
      // We preserve any explicit mode register setreg instruction we encounter,
      // as we assume it has been inserted by a higher authority (this is
      // likely to be a very rare occurrence).
      unsigned Dst = TII->getNamedOperand(MI, AMDGPU::OpName::simm16)->getImm();
      using namespace AMDGPU::Hwreg;
      auto [Id, Offset, Width] = HwregEncoding::decode(Dst);
      if (Id != ID_MODE)
        continue;

      unsigned Mask = maskTrailingOnes<unsigned>(Width) << Offset;

      // If an InsertionPoint is set we will insert a setreg there.
      if (InsertionPoint) {
        insertSetreg(MBB, InsertionPoint, TII, IPChange.delta(NewInfo->Change));
        InsertionPoint = nullptr;
      }
      // If this is an immediate then we know the value being set, but if it is
      // not an immediate then we treat the modified bits of the mode register
      // as unknown.
      if (MI.getOpcode() == AMDGPU::S_SETREG_IMM32_B32 ||
          MI.getOpcode() == AMDGPU::S_SETREG_IMM32_B32_mode) {
        unsigned Val = TII->getNamedOperand(MI, AMDGPU::OpName::imm)->getImm();
        unsigned Mode = (Val << Offset) & Mask;
        Status Setreg = Status(Mask, Mode);
        // If we haven't already set the initial requirements for the block we
        // don't need to as the requirements start from this explicit setreg.
        RequirePending = false;
        NewInfo->Change = NewInfo->Change.merge(Setreg);
      } else {
        NewInfo->Change = NewInfo->Change.mergeUnknown(Mask);
      }
    } else if (!NewInfo->Change.isCompatible(InstrMode)) {
```
**EN:** This section arranges declarations into namespace scope so related AMDGPU functionality stays organized and avoids symbol collisions. Main symbols: `HwregEncoding::decode`.
**CN:** 本节将声明组织到命名空间作用域中，使相关 AMDGPU 功能保持清晰并避免符号冲突。 主要符号：`HwregEncoding::decode`。

### Lines 305-338: Conditional logic and checks
```cpp
      // This instruction uses the Mode register and its requirements aren't
      // compatible with the current mode.
      if (InsertionPoint) {
        // If the required mode change cannot be included in the current
        // InsertionPoint changes, we need a setreg and start a new
        // InsertionPoint.
        if (!IPChange.delta(NewInfo->Change).isCombinable(InstrMode)) {
          if (RequirePending) {
            // This is the first insertionPoint in the block so we will defer
            // the insertion of the setreg to Phase 3 where we know whether or
            // not it is actually needed.
            NewInfo->FirstInsertionPoint = InsertionPoint;
            NewInfo->Require = NewInfo->Change;
            RequirePending = false;
          } else {
            insertSetreg(MBB, InsertionPoint, TII,
                         IPChange.delta(NewInfo->Change));
            IPChange = NewInfo->Change;
          }
          // Set the new InsertionPoint
          InsertionPoint = &MI;
        }
        NewInfo->Change = NewInfo->Change.merge(InstrMode);
      } else {
        // No InsertionPoint is currently set - this is either the first in
        // the block or we have previously seen an explicit setreg.
        InsertionPoint = &MI;
        IPChange = NewInfo->Change;
        NewInfo->Change = NewInfo->Change.merge(InstrMode);
      }
    }
  }
  if (RequirePending) {
    // If we haven't yet set the initial requirements for the block we set them
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 339-372: Implements std::move
```cpp
    // now.
    NewInfo->FirstInsertionPoint = InsertionPoint;
    NewInfo->Require = NewInfo->Change;
  } else if (InsertionPoint) {
    // We need to insert a setreg at the InsertionPoint
    insertSetreg(MBB, InsertionPoint, TII, IPChange.delta(NewInfo->Change));
  }
  NewInfo->Exit = NewInfo->Change;
  BlockInfo[MBB.getNumber()] = std::move(NewInfo);
}

// In Phase 2 we revisit each block and calculate the common Mode register
// value provided by all predecessor blocks. If the Exit value for the block
// is changed, then we add the successor blocks to the worklist so that the
// exit value is propagated.
void SIModeRegister::processBlockPhase2(MachineBasicBlock &MBB,
                                        const SIInstrInfo *TII) {
  bool RevisitRequired = false;
  bool ExitSet = false;
  unsigned ThisBlock = MBB.getNumber();
  if (MBB.pred_empty()) {
    // There are no predecessors, so use the default starting status.
    BlockInfo[ThisBlock]->Pred = DefaultStatus;
    ExitSet = true;
  } else {
    // Build a status that is common to all the predecessors by intersecting
    // all the predecessor exit status values.
    // Mask bits (which represent the Mode bits with a known value) can only be
    // added by explicit SETREG instructions or the initial default value -
    // the intersection process may remove Mask bits.
    // If we find a predecessor that has not yet had an exit value determined
    // (this can happen for example if a block is its own predecessor) we defer
    // use of that value as the Mask will be all zero, and we will revisit this
    // block again later (unless the only predecessor without an exit value is
```
**EN:** This section contains concrete logic for std::move. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `std::move`, `SIModeRegister::processBlockPhase2`.
**CN:** 本节包含与 std::move 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`std::move`, `SIModeRegister::processBlockPhase2`。

### Lines 373-406: Conditional logic and checks
```cpp
    // this block).
    MachineBasicBlock::pred_iterator P = MBB.pred_begin(), E = MBB.pred_end();
    MachineBasicBlock &PB = *(*P);
    unsigned PredBlock = PB.getNumber();
    if ((ThisBlock == PredBlock) && (std::next(P) == E)) {
      BlockInfo[ThisBlock]->Pred = DefaultStatus;
      ExitSet = true;
    } else if (BlockInfo[PredBlock]->ExitSet) {
      BlockInfo[ThisBlock]->Pred = BlockInfo[PredBlock]->Exit;
      ExitSet = true;
    } else if (PredBlock != ThisBlock)
      RevisitRequired = true;

    for (P = std::next(P); P != E; P = std::next(P)) {
      MachineBasicBlock *Pred = *P;
      unsigned PredBlock = Pred->getNumber();
      if (BlockInfo[PredBlock]->ExitSet) {
        if (BlockInfo[ThisBlock]->ExitSet) {
          BlockInfo[ThisBlock]->Pred =
              BlockInfo[ThisBlock]->Pred.intersect(BlockInfo[PredBlock]->Exit);
        } else {
          BlockInfo[ThisBlock]->Pred = BlockInfo[PredBlock]->Exit;
        }
        ExitSet = true;
      } else if (PredBlock != ThisBlock)
        RevisitRequired = true;
    }
  }
  Status TmpStatus =
      BlockInfo[ThisBlock]->Pred.merge(BlockInfo[ThisBlock]->Change);
  if (BlockInfo[ThisBlock]->Exit != TmpStatus) {
    BlockInfo[ThisBlock]->Exit = TmpStatus;
    // Add the successors to the work list so we can propagate the changed exit
    // status.
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::next`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::next`。

### Lines 407-434: Implements SIModeRegister::processBlockPhase3
```cpp
    for (MachineBasicBlock *Succ : MBB.successors())
      Phase2List.push(Succ);
  }
  BlockInfo[ThisBlock]->ExitSet = ExitSet;
  if (RevisitRequired)
    Phase2List.push(&MBB);
}

// In Phase 3 we revisit each block and if it has an insertion point defined we
// check whether the predecessor mode meets the block's entry requirements. If
// not we insert an appropriate setreg instruction to modify the Mode register.
void SIModeRegister::processBlockPhase3(MachineBasicBlock &MBB,
                                        const SIInstrInfo *TII) {
  unsigned ThisBlock = MBB.getNumber();
  if (!BlockInfo[ThisBlock]->Pred.isCompatible(BlockInfo[ThisBlock]->Require)) {
    Status Delta =
        BlockInfo[ThisBlock]->Pred.delta(BlockInfo[ThisBlock]->Require);
    if (BlockInfo[ThisBlock]->FirstInsertionPoint)
      insertSetreg(MBB, BlockInfo[ThisBlock]->FirstInsertionPoint, TII, Delta);
    else
      insertSetreg(MBB, &MBB.instr_front(), TII, Delta);
  }
}

bool SIModeRegisterLegacy::runOnMachineFunction(MachineFunction &MF) {
  return SIModeRegister().run(MF);
}

```
**EN:** This section contains concrete logic for SIModeRegister::processBlockPhase3. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SIModeRegister::processBlockPhase3`, `SIModeRegisterLegacy::runOnMachineFunction`.
**CN:** 本节包含与 SIModeRegister::processBlockPhase3 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SIModeRegister::processBlockPhase3`, `SIModeRegisterLegacy::runOnMachineFunction`。

### Lines 435-463: Implements SIModeRegisterPass::run
```cpp
PreservedAnalyses SIModeRegisterPass::run(MachineFunction &MF,
                                          MachineFunctionAnalysisManager &AM) {
  if (!SIModeRegister().run(MF))
    return PreservedAnalyses::all();
  auto PA = getMachineFunctionPassPreservedAnalyses();
  PA.preserveSet<CFGAnalyses>();
  return PA;
}

bool SIModeRegister::run(MachineFunction &MF) {
  // Constrained FP intrinsics are used to support non-default rounding modes.
  // strictfp attribute is required to mark functions with strict FP semantics
  // having constrained FP intrinsics. This pass fixes up operations that uses
  // a non-default rounding mode for non-strictfp functions. But it should not
  // assume or modify any default rounding modes in case of strictfp functions.
  const Function &F = MF.getFunction();
  if (F.hasFnAttribute(llvm::Attribute::StrictFP))
    return Changed;
  BlockInfo.resize(MF.getNumBlockIDs());
  const GCNSubtarget &ST = MF.getSubtarget<GCNSubtarget>();
  const SIInstrInfo *TII = ST.getInstrInfo();

  // Processing is performed in a number of phases

  // Phase 1 - determine the initial mode required by each block, and add setreg
  // instructions for intra block requirements.
  for (MachineBasicBlock &BB : MF)
    processBlockPhase1(BB, TII);

```
**EN:** This section contains concrete logic for SIModeRegisterPass::run. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SIModeRegisterPass::run`, `PreservedAnalyses::all`, `SIModeRegister::run`.
**CN:** 本节包含与 SIModeRegisterPass::run 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SIModeRegisterPass::run`, `PreservedAnalyses::all`, `SIModeRegister::run`。

### Lines 464-482: Result computation and returns
```cpp
  // Phase 2 - determine the exit mode from each block. We add all blocks to the
  // list here, but will also add any that need to be revisited during Phase 2
  // processing.
  for (MachineBasicBlock &BB : MF)
    Phase2List.push(&BB);
  while (!Phase2List.empty()) {
    processBlockPhase2(*Phase2List.front(), TII);
    Phase2List.pop();
  }

  // Phase 3 - add an initial setreg to each block where the required entry mode
  // is not satisfied by the exit mode of all its predecessors.
  for (MachineBasicBlock &BB : MF)
    processBlockPhase3(BB, TII);

  BlockInfo.clear();

  return Changed;
}
```
**EN:** This section finalizes intermediate state and returns the value that the caller or pass pipeline needs.
**CN:** 本节整理中间状态并返回调用者或 Pass 流水线所需的结果。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `Status`, `BlockData`, `SIModeRegister`, `SIModeRegisterLegacy`, `MachineFunctionPass::getAnalysisUsage`, `llvm::createSIModeRegisterPass`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; LLVM pass integration / LLVM Pass 集成
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPU.h"`
- `"GCNSubtarget.h"`
- `"MCTargetDesc/AMDGPUMCTargetDesc.h"`
- `"llvm/ADT/Statistic.h"`
- `"llvm/CodeGen/MachineFunctionPass.h"`
- `<queue>`
