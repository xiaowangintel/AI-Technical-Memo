# SIPreEmitPeephole.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/SIPreEmitPeephole.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements SIPreEmitPeephole for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 SIPreEmitPeephole 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-42: File banner, includes, and setup
```cpp
//===-- SIPreEmitPeephole.cpp ------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// This pass performs the peephole optimizations before code emission.
///
/// Additionally, this pass also unpacks packed instructions (V_PK_MUL_F32/F16,
/// V_PK_ADD_F32/F16, V_PK_FMA_F32) adjacent to MFMAs such that they can be
/// co-issued. This helps with overlapping MFMA and certain vector instructions
/// in machine schedules and is expected to improve performance. Only those
/// packed instructions are unpacked that are overlapped by the MFMA latency.
/// Rest should remain untouched.
/// TODO: Add support for F16 packed instructions
//===----------------------------------------------------------------------===//

#include "AMDGPU.h"
#include "GCNSubtarget.h"
#include "MCTargetDesc/AMDGPUMCTargetDesc.h"
#include "llvm/ADT/SetVector.h"
#include "llvm/CodeGen/MachineDominators.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineLoopInfo.h"
#include "llvm/CodeGen/MachinePostDominators.h"
#include "llvm/CodeGen/TargetSchedule.h"
#include "llvm/Support/BranchProbability.h"
using namespace llvm;

#define DEBUG_TYPE "si-pre-emit-peephole"

namespace {

class SIPreEmitPeephole {
private:
  const SIInstrInfo *TII = nullptr;
  const SIRegisterInfo *TRI = nullptr;
  MachineLoopInfo *MLI = nullptr;

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `SIPreEmitPeephole`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`SIPreEmitPeephole`。

### Lines 43-83: Declares optimizeVccBranch
```cpp
  bool optimizeVccBranch(MachineInstr &MI) const;
  void updateMLIBeforeRemovingEdge(MachineBasicBlock *From,
                                   MachineBasicBlock *To) const;
  bool optimizeSetGPR(MachineInstr &First, MachineInstr &MI) const;
  bool getBlockDestinations(MachineBasicBlock &SrcMBB,
                            MachineBasicBlock *&TrueMBB,
                            MachineBasicBlock *&FalseMBB,
                            SmallVectorImpl<MachineOperand> &Cond);
  bool mustRetainExeczBranch(const MachineInstr &Branch,
                             const MachineBasicBlock &From,
                             const MachineBasicBlock &To) const;
  bool removeExeczBranch(MachineInstr &MI, MachineBasicBlock &SrcMBB);
  // Creates a list of packed instructions following an MFMA that are suitable
  // for unpacking.
  void collectUnpackingCandidates(MachineInstr &BeginMI,
                                  SetVector<MachineInstr *> &InstrsToUnpack,
                                  uint16_t NumMFMACycles);
  // v_pk_fma_f32 v[0:1], v[0:1], v[2:3], v[2:3] op_sel:[1,1,1]
  // op_sel_hi:[0,0,0]
  // ==>
  // v_fma_f32 v0, v1, v3, v3
  // v_fma_f32 v1, v0, v2, v2
  // Here, we have overwritten v0 before we use it. This function checks if
  // unpacking can lead to such a situation.
  bool canUnpackingClobberRegister(const MachineInstr &MI);
  // Unpack and insert F32 packed instructions, such as V_PK_MUL, V_PK_ADD, and
  // V_PK_FMA. Currently, only V_PK_MUL, V_PK_ADD, V_PK_FMA are supported for
  // this transformation.
  void performF32Unpacking(MachineInstr &I);
  // Select corresponding unpacked instruction
  uint32_t mapToUnpackedOpcode(MachineInstr &I);
  // Creates the unpacked instruction to be inserted. Adds source modifiers to
  // the unpacked instructions based on the source modifiers in the packed
  // instruction.
  MachineInstrBuilder createUnpackedMI(MachineInstr &I, uint32_t UnpackedOpcode,
                                       bool IsHiBits);
  // Process operands/source modifiers from packed instructions and insert the
  // appropriate source modifers and operands into the unpacked instructions.
  void addOperandAndMods(MachineInstrBuilder &NewMI, unsigned SrcMods,
                         bool IsHiBits, const MachineOperand &SrcMO);

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 84-125: Registers LLVM passes
```cpp
public:
  bool run(MachineFunction &MF, MachineLoopInfo *MLI);
};

class SIPreEmitPeepholeLegacy : public MachineFunctionPass {
public:
  static char ID;

  SIPreEmitPeepholeLegacy() : MachineFunctionPass(ID) {}

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addUsedIfAvailable<MachineLoopInfoWrapperPass>();
    AU.addPreserved<MachineLoopInfoWrapperPass>();
    MachineFunctionPass::getAnalysisUsage(AU);
  }

  bool runOnMachineFunction(MachineFunction &MF) override {
    auto *MLIWrapper = getAnalysisIfAvailable<MachineLoopInfoWrapperPass>();
    MachineLoopInfo *MLI = MLIWrapper ? &MLIWrapper->getLI() : nullptr;
    return SIPreEmitPeephole().run(MF, MLI);
  }
};

} // End anonymous namespace.

INITIALIZE_PASS(SIPreEmitPeepholeLegacy, DEBUG_TYPE,
                "SI peephole optimizations", false, false)

char SIPreEmitPeepholeLegacy::ID = 0;

char &llvm::SIPreEmitPeepholeID = SIPreEmitPeepholeLegacy::ID;

void SIPreEmitPeephole::updateMLIBeforeRemovingEdge(
    MachineBasicBlock *From, MachineBasicBlock *To) const {
  if (!MLI)
    return;

  // Only handle back-edges: To must be a loop header with From inside the loop.
  MachineLoop *Loop = MLI->getLoopFor(To);
  if (!Loop || Loop->getHeader() != To || !Loop->contains(From))
    return;

```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `SIPreEmitPeepholeLegacy`, `MachineFunctionPass::getAnalysisUsage`, `SIPreEmitPeephole::updateMLIBeforeRemovingEdge`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`SIPreEmitPeepholeLegacy`, `MachineFunctionPass::getAnalysisUsage`, `SIPreEmitPeephole::updateMLIBeforeRemovingEdge`。

### Lines 126-160: Declares getParentLoop
```cpp
  // Count back-edges
  unsigned BackEdgeCount = 0;
  for (MachineBasicBlock *Pred : To->predecessors()) {
    if (Loop->contains(Pred))
      BackEdgeCount++;
  }

  if (BackEdgeCount > 1)
    return;

  MachineLoop *ParentLoop = Loop->getParentLoop();

  // Re-map blocks directly owned by this loop to the parent.
  for (MachineBasicBlock *BB : Loop->blocks()) {
    if (MLI->getLoopFor(BB) == Loop)
      MLI->changeLoopFor(BB, ParentLoop);
  }

  // Reparent all child loops.
  while (!Loop->isInnermost()) {
    MachineLoop *Child = Loop->removeChildLoop(std::prev(Loop->end()));
    if (ParentLoop)
      ParentLoop->addChildLoop(Child);
    else
      MLI->addTopLevelLoop(Child);
  }

  if (ParentLoop)
    ParentLoop->removeChildLoop(Loop);
  else
    MLI->removeLoop(llvm::find(*MLI, Loop));

  MLI->destroy(Loop);
}

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `std::prev`, `llvm::find`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`std::prev`, `llvm::find`。

### Lines 161-189: Implements SIPreEmitPeephole::optimizeVccBranch
```cpp
bool SIPreEmitPeephole::optimizeVccBranch(MachineInstr &MI) const {
  // Match:
  // sreg = -1 or 0
  // vcc = S_AND_B64 exec, sreg or S_ANDN2_B64 exec, sreg
  // S_CBRANCH_VCC[N]Z
  // =>
  // S_CBRANCH_EXEC[N]Z
  // We end up with this pattern sometimes after basic block placement.
  // It happens while combining a block which assigns -1 or 0 to a saved mask
  // and another block which consumes that saved mask and then a branch.
  //
  // While searching this also performs the following substitution:
  // vcc = V_CMP
  // vcc = S_AND exec, vcc
  // S_CBRANCH_VCC[N]Z
  // =>
  // vcc = V_CMP
  // S_CBRANCH_VCC[N]Z

  bool Changed = false;
  MachineBasicBlock &MBB = *MI.getParent();
  const GCNSubtarget &ST = MBB.getParent()->getSubtarget<GCNSubtarget>();
  const bool IsWave32 = ST.isWave32();
  const unsigned CondReg = TRI->getVCC();
  const unsigned ExecReg = IsWave32 ? AMDGPU::EXEC_LO : AMDGPU::EXEC;
  const unsigned And = IsWave32 ? AMDGPU::S_AND_B32 : AMDGPU::S_AND_B64;
  const unsigned AndN2 = IsWave32 ? AMDGPU::S_ANDN2_B32 : AMDGPU::S_ANDN2_B64;
  const unsigned Mov = IsWave32 ? AMDGPU::S_MOV_B32 : AMDGPU::S_MOV_B64;

```
**EN:** This section contains concrete logic for SIPreEmitPeephole::optimizeVccBranch. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SIPreEmitPeephole::optimizeVccBranch`.
**CN:** 本节包含与 SIPreEmitPeephole::optimizeVccBranch 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SIPreEmitPeephole::optimizeVccBranch`。

### Lines 190-221: Defines getReverseIterator
```cpp
  MachineBasicBlock::reverse_iterator A = MI.getReverseIterator(),
                                      E = MBB.rend();
  bool ReadsCond = false;
  unsigned Threshold = 5;
  for (++A; A != E; ++A) {
    if (!--Threshold)
      return false;
    if (A->modifiesRegister(ExecReg, TRI))
      return false;
    if (A->modifiesRegister(CondReg, TRI)) {
      if (!A->definesRegister(CondReg, TRI) ||
          (A->getOpcode() != And && A->getOpcode() != AndN2))
        return false;
      break;
    }
    ReadsCond |= A->readsRegister(CondReg, TRI);
  }
  if (A == E)
    return false;

  MachineOperand &Op1 = A->getOperand(1);
  MachineOperand &Op2 = A->getOperand(2);
  if ((!Op1.isReg() || Op1.getReg() != ExecReg) && Op2.isReg() &&
      Op2.getReg() == ExecReg) {
    TII->commuteInstruction(*A);
    Changed = true;
  }
  if (!Op1.isReg() || Op1.getReg() != ExecReg)
    return Changed;
  if (Op2.isImm() && !(Op2.getImm() == -1 || Op2.getImm() == 0))
    return Changed;

```
**EN:** This section contains concrete logic for getReverseIterator. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 getReverseIterator 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 222-263: Conditional logic and checks
```cpp
  int64_t MaskValue = 0;
  Register SReg;
  if (Op2.isReg()) {
    SReg = Op2.getReg();
    auto M = std::next(A);
    bool ReadsSreg = false;
    bool ModifiesExec = false;
    for (; M != E; ++M) {
      if (M->definesRegister(SReg, TRI))
        break;
      if (M->modifiesRegister(SReg, TRI))
        return Changed;
      ReadsSreg |= M->readsRegister(SReg, TRI);
      ModifiesExec |= M->modifiesRegister(ExecReg, TRI);
    }
    if (M == E)
      return Changed;
    // If SReg is VCC and SReg definition is a VALU comparison.
    // This means S_AND with EXEC is not required.
    // Erase the S_AND and return.
    // Note: isVOPC is used instead of isCompare to catch V_CMP_CLASS
    if (A->getOpcode() == And && SReg == CondReg && !ModifiesExec &&
        TII->isVOPC(*M)) {
      A->eraseFromParent();
      return true;
    }
    if (!M->isMoveImmediate() || !M->getOperand(1).isImm() ||
        (M->getOperand(1).getImm() != -1 && M->getOperand(1).getImm() != 0))
      return Changed;
    MaskValue = M->getOperand(1).getImm();
    // First if sreg is only used in the AND instruction fold the immediate
    // into the AND.
    if (!ReadsSreg && Op2.isKill()) {
      A->getOperand(2).ChangeToImmediate(MaskValue);
      M->eraseFromParent();
    }
  } else if (Op2.isImm()) {
    MaskValue = Op2.getImm();
  } else {
    llvm_unreachable("Op2 must be register or immediate");
  }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::next`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::next`。

### Lines 264-307: Declares assert
```cpp
  // Invert mask for s_andn2
  assert(MaskValue == 0 || MaskValue == -1);
  if (A->getOpcode() == AndN2)
    MaskValue = ~MaskValue;

  if (!ReadsCond && A->registerDefIsDead(AMDGPU::SCC, /*TRI=*/nullptr)) {
    if (!MI.killsRegister(CondReg, TRI)) {
      // Replace AND with MOV
      if (MaskValue == 0) {
        BuildMI(*A->getParent(), *A, A->getDebugLoc(), TII->get(Mov), CondReg)
            .addImm(0);
      } else {
        BuildMI(*A->getParent(), *A, A->getDebugLoc(), TII->get(Mov), CondReg)
            .addReg(ExecReg);
      }
    }
    // Remove AND instruction
    A->eraseFromParent();
  }

  bool IsVCCZ = MI.getOpcode() == AMDGPU::S_CBRANCH_VCCZ;
  if (SReg == ExecReg) {
    // EXEC is updated directly
    if (IsVCCZ) {
      MI.eraseFromParent();
      return true;
    }
    MI.setDesc(TII->get(AMDGPU::S_BRANCH));
  } else if (IsVCCZ && MaskValue == 0) {
    // Will always branch
    // Remove all successors shadowed by new unconditional branch
    MachineBasicBlock *Parent = MI.getParent();
    SmallVector<MachineInstr *, 4> ToRemove;
    bool Found = false;
    for (MachineInstr &Term : Parent->terminators()) {
      if (Found) {
        if (Term.isBranch())
          ToRemove.push_back(&Term);
      } else {
        Found = Term.isIdenticalTo(MI);
      }
    }
    assert(Found && "conditional branch is not terminator");
    for (auto *BranchMI : ToRemove) {
```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 308-342: Conditional logic and checks
```cpp
      MachineOperand &Dst = BranchMI->getOperand(0);
      assert(Dst.isMBB() && "destination is not basic block");
      updateMLIBeforeRemovingEdge(Parent, Dst.getMBB());
      Parent->removeSuccessor(Dst.getMBB());
      BranchMI->eraseFromParent();
    }

    if (MachineBasicBlock *Succ = Parent->getFallThrough()) {
      updateMLIBeforeRemovingEdge(Parent, Succ);
      Parent->removeSuccessor(Succ);
    }

    // Rewrite to unconditional branch
    MI.setDesc(TII->get(AMDGPU::S_BRANCH));
  } else if (!IsVCCZ && MaskValue == 0) {
    // Will never branch
    MachineOperand &Dst = MI.getOperand(0);
    assert(Dst.isMBB() && "destination is not basic block");
    MachineBasicBlock *Parent = MI.getParent();
    updateMLIBeforeRemovingEdge(Parent, Dst.getMBB());
    Parent->removeSuccessor(Dst.getMBB());
    MI.eraseFromParent();
    return true;
  } else if (MaskValue == -1) {
    // Depends only on EXEC
    MI.setDesc(
        TII->get(IsVCCZ ? AMDGPU::S_CBRANCH_EXECZ : AMDGPU::S_CBRANCH_EXECNZ));
  }

  MI.removeOperand(MI.findRegisterUseOperandIdx(CondReg, TRI, false /*Kill*/));
  MI.addImplicitDefUseOperands(*MBB.getParent());

  return true;
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 343-385: Implements SIPreEmitPeephole::optimizeSetGPR
```cpp
bool SIPreEmitPeephole::optimizeSetGPR(MachineInstr &First,
                                       MachineInstr &MI) const {
  MachineBasicBlock &MBB = *MI.getParent();
  const MachineFunction &MF = *MBB.getParent();
  const MachineRegisterInfo &MRI = MF.getRegInfo();
  MachineOperand *Idx = TII->getNamedOperand(MI, AMDGPU::OpName::src0);
  Register IdxReg = Idx->isReg() ? Idx->getReg() : Register();
  SmallVector<MachineInstr *, 4> ToRemove;
  bool IdxOn = true;

  if (!MI.isIdenticalTo(First))
    return false;

  // Scan back to find an identical S_SET_GPR_IDX_ON
  for (MachineBasicBlock::instr_iterator I = std::next(First.getIterator()),
                                         E = MI.getIterator();
       I != E; ++I) {
    if (I->isBundle() || I->isDebugInstr())
      continue;
    switch (I->getOpcode()) {
    case AMDGPU::S_SET_GPR_IDX_MODE:
      return false;
    case AMDGPU::S_SET_GPR_IDX_OFF:
      IdxOn = false;
      ToRemove.push_back(&*I);
      break;
    default:
      if (I->modifiesRegister(AMDGPU::M0, TRI))
        return false;
      if (IdxReg && I->modifiesRegister(IdxReg, TRI))
        return false;
      if (llvm::any_of(I->operands(), [&MRI, this](const MachineOperand &MO) {
            return MO.isReg() && TRI->isVectorRegister(MRI, MO.getReg());
          })) {
        // The only exception allowed here is another indirect vector move
        // with the same mode.
        if (!IdxOn || !(I->getOpcode() == AMDGPU::V_MOV_B32_indirect_write ||
                        I->getOpcode() == AMDGPU::V_MOV_B32_indirect_read))
          return false;
      }
    }
  }

```
**EN:** This section contains concrete logic for SIPreEmitPeephole::optimizeSetGPR. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SIPreEmitPeephole::optimizeSetGPR`, `std::next`, `llvm::any_of`.
**CN:** 本节包含与 SIPreEmitPeephole::optimizeSetGPR 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SIPreEmitPeephole::optimizeSetGPR`, `std::next`, `llvm::any_of`。

### Lines 386-426: Declares class BranchWeightCostModel
```cpp
  MI.eraseFromBundle();
  for (MachineInstr *RI : ToRemove)
    RI->eraseFromBundle();
  return true;
}

bool SIPreEmitPeephole::getBlockDestinations(
    MachineBasicBlock &SrcMBB, MachineBasicBlock *&TrueMBB,
    MachineBasicBlock *&FalseMBB, SmallVectorImpl<MachineOperand> &Cond) {
  if (TII->analyzeBranch(SrcMBB, TrueMBB, FalseMBB, Cond))
    return false;

  if (!FalseMBB)
    FalseMBB = SrcMBB.getNextNode();

  return true;
}

namespace {
class BranchWeightCostModel {
  const SIInstrInfo &TII;
  const TargetSchedModel &SchedModel;
  BranchProbability BranchProb;
  static constexpr uint64_t BranchNotTakenCost = 1;
  uint64_t BranchTakenCost;
  uint64_t ThenCyclesCost = 0;

public:
  BranchWeightCostModel(const SIInstrInfo &TII, const MachineInstr &Branch,
                        const MachineBasicBlock &Succ)
      : TII(TII), SchedModel(TII.getSchedModel()) {
    const MachineBasicBlock &Head = *Branch.getParent();
    const auto *FromIt = find(Head.successors(), &Succ);
    assert(FromIt != Head.succ_end());

    BranchProb = Head.getSuccProbability(FromIt);
    if (BranchProb.isUnknown())
      BranchProb = BranchProbability::getZero();
    BranchTakenCost = SchedModel.computeInstrLatency(&Branch);
  }

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `BranchWeightCostModel`, `SIPreEmitPeephole::getBlockDestinations`, `BranchProbability::getZero`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`BranchWeightCostModel`, `SIPreEmitPeephole::getBlockDestinations`, `BranchProbability::getZero`。

### Lines 427-470: Defines isProfitable
```cpp
  bool isProfitable(const MachineInstr &MI) {
    if (TII.isWaitcnt(MI.getOpcode()))
      return false;

    ThenCyclesCost += SchedModel.computeInstrLatency(&MI);

    // Consider `P = N/D` to be the probability of execz being false (skipping
    // the then-block) The transformation is profitable if always executing the
    // 'then' block is cheaper than executing sometimes 'then' and always
    // executing s_cbranch_execz:
    // * ThenCost <= P*ThenCost + (1-P)*BranchTakenCost + P*BranchNotTakenCost
    // * (1-P) * ThenCost <= (1-P)*BranchTakenCost + P*BranchNotTakenCost
    // * (D-N)/D * ThenCost <= (D-N)/D * BranchTakenCost + N/D *
    // BranchNotTakenCost
    uint64_t Numerator = BranchProb.getNumerator();
    uint64_t Denominator = BranchProb.getDenominator();
    return (Denominator - Numerator) * ThenCyclesCost <=
           ((Denominator - Numerator) * BranchTakenCost +
            Numerator * BranchNotTakenCost);
  }
};

bool SIPreEmitPeephole::mustRetainExeczBranch(
    const MachineInstr &Branch, const MachineBasicBlock &From,
    const MachineBasicBlock &To) const {
  assert(is_contained(Branch.getParent()->successors(), &From));
  BranchWeightCostModel CostModel{*TII, Branch, From};

  const MachineFunction *MF = From.getParent();
  for (MachineFunction::const_iterator MBBI(&From), ToI(&To), End = MF->end();
       MBBI != End && MBBI != ToI; ++MBBI) {
    const MachineBasicBlock &MBB = *MBBI;

    for (const MachineInstr &MI : MBB) {
      // When a uniform loop is inside non-uniform control flow, the branch
      // leaving the loop might never be taken when EXEC = 0.
      // Hence we should retain cbranch out of the loop lest it become infinite.
      if (MI.isConditionalBranch())
        return true;

      if (MI.isUnconditionalBranch() &&
          TII->getBranchDestBlock(MI) != MBB.getNextNode())
        return true;

```
**EN:** This section contains concrete logic for isProfitable. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SIPreEmitPeephole::mustRetainExeczBranch`.
**CN:** 本节包含与 isProfitable 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SIPreEmitPeephole::mustRetainExeczBranch`。

### Lines 471-514: Conditional logic and checks
```cpp
      if (MI.isMetaInstruction())
        continue;

      if (TII->hasUnwantedEffectsWhenEXECEmpty(MI))
        return true;

      if (!CostModel.isProfitable(MI))
        return true;
    }
  }

  return false;
}
} // namespace

// Returns true if the skip branch instruction is removed.
bool SIPreEmitPeephole::removeExeczBranch(MachineInstr &MI,
                                          MachineBasicBlock &SrcMBB) {

  if (!TII->getSchedModel().hasInstrSchedModel())
    return false;

  MachineBasicBlock *TrueMBB = nullptr;
  MachineBasicBlock *FalseMBB = nullptr;
  SmallVector<MachineOperand, 1> Cond;

  if (!getBlockDestinations(SrcMBB, TrueMBB, FalseMBB, Cond))
    return false;

  // Consider only the forward branches.
  if (SrcMBB.getNumber() >= TrueMBB->getNumber())
    return false;

  // Consider only when it is legal and profitable
  if (mustRetainExeczBranch(MI, *FalseMBB, *TrueMBB))
    return false;

  LLVM_DEBUG(dbgs() << "Removing the execz branch: " << MI);
  MI.eraseFromParent();
  SrcMBB.removeSuccessor(TrueMBB);

  return true;
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `SIPreEmitPeephole::removeExeczBranch`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`SIPreEmitPeephole::removeExeczBranch`。

### Lines 515-552: Implements SIPreEmitPeephole::canUnpackingClobberRegister
```cpp
bool SIPreEmitPeephole::canUnpackingClobberRegister(const MachineInstr &MI) {
  unsigned OpCode = MI.getOpcode();
  Register DstReg = MI.getOperand(0).getReg();
  // Only the first register in the register pair needs to be checked due to the
  // unpacking order. Packed instructions are unpacked such that the lower 32
  // bits (i.e., the first register in the pair) are written first. This can
  // introduce dependencies if the first register is written in one instruction
  // and then read as part of the higher 32 bits in the subsequent instruction.
  // Such scenarios can arise due to specific combinations of op_sel and
  // op_sel_hi modifiers.
  Register UnpackedDstReg = TRI->getSubReg(DstReg, AMDGPU::sub0);

  const MachineOperand *Src0MO = TII->getNamedOperand(MI, AMDGPU::OpName::src0);
  if (Src0MO && Src0MO->isReg()) {
    Register SrcReg0 = Src0MO->getReg();
    unsigned Src0Mods =
        TII->getNamedOperand(MI, AMDGPU::OpName::src0_modifiers)->getImm();
    Register HiSrc0Reg = (Src0Mods & SISrcMods::OP_SEL_1)
                             ? TRI->getSubReg(SrcReg0, AMDGPU::sub1)
                             : TRI->getSubReg(SrcReg0, AMDGPU::sub0);
    // Check if the register selected by op_sel_hi is the same as the first
    // register in the destination register pair.
    if (TRI->regsOverlap(UnpackedDstReg, HiSrc0Reg))
      return true;
  }

  const MachineOperand *Src1MO = TII->getNamedOperand(MI, AMDGPU::OpName::src1);
  if (Src1MO && Src1MO->isReg()) {
    Register SrcReg1 = Src1MO->getReg();
    unsigned Src1Mods =
        TII->getNamedOperand(MI, AMDGPU::OpName::src1_modifiers)->getImm();
    Register HiSrc1Reg = (Src1Mods & SISrcMods::OP_SEL_1)
                             ? TRI->getSubReg(SrcReg1, AMDGPU::sub1)
                             : TRI->getSubReg(SrcReg1, AMDGPU::sub0);
    if (TRI->regsOverlap(UnpackedDstReg, HiSrc1Reg))
      return true;
  }

```
**EN:** This section contains concrete logic for SIPreEmitPeephole::canUnpackingClobberRegister. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SIPreEmitPeephole::canUnpackingClobberRegister`.
**CN:** 本节包含与 SIPreEmitPeephole::canUnpackingClobberRegister 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SIPreEmitPeephole::canUnpackingClobberRegister`。

### Lines 553-589: Switch-based control flow
```cpp
  // Applicable for packed instructions with 3 source operands, such as
  // V_PK_FMA.
  if (AMDGPU::hasNamedOperand(OpCode, AMDGPU::OpName::src2)) {
    const MachineOperand *Src2MO =
        TII->getNamedOperand(MI, AMDGPU::OpName::src2);
    if (Src2MO && Src2MO->isReg()) {
      Register SrcReg2 = Src2MO->getReg();
      unsigned Src2Mods =
          TII->getNamedOperand(MI, AMDGPU::OpName::src2_modifiers)->getImm();
      Register HiSrc2Reg = (Src2Mods & SISrcMods::OP_SEL_1)
                               ? TRI->getSubReg(SrcReg2, AMDGPU::sub1)
                               : TRI->getSubReg(SrcReg2, AMDGPU::sub0);
      if (TRI->regsOverlap(UnpackedDstReg, HiSrc2Reg))
        return true;
    }
  }
  return false;
}

uint32_t SIPreEmitPeephole::mapToUnpackedOpcode(MachineInstr &I) {
  unsigned Opcode = I.getOpcode();
  // Use 64 bit encoding to allow use of VOP3 instructions.
  // VOP3 e64 instructions allow source modifiers
  // e32 instructions don't allow source modifiers.
  switch (Opcode) {
  case AMDGPU::V_PK_ADD_F32:
    return AMDGPU::V_ADD_F32_e64;
  case AMDGPU::V_PK_MUL_F32:
    return AMDGPU::V_MUL_F32_e64;
  case AMDGPU::V_PK_FMA_F32:
    return AMDGPU::V_FMA_F32_e64;
  default:
    return std::numeric_limits<uint32_t>::max();
  }
  llvm_unreachable("Fully covered switch");
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::hasNamedOperand`, `SIPreEmitPeephole::mapToUnpackedOpcode`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::hasNamedOperand`, `SIPreEmitPeephole::mapToUnpackedOpcode`。

### Lines 590-619: Implements SIPreEmitPeephole::addOperandAndMods
```cpp
void SIPreEmitPeephole::addOperandAndMods(MachineInstrBuilder &NewMI,
                                          unsigned SrcMods, bool IsHiBits,
                                          const MachineOperand &SrcMO) {
  unsigned NewSrcMods = 0;
  unsigned NegModifier = IsHiBits ? SISrcMods::NEG_HI : SISrcMods::NEG;
  unsigned OpSelModifier = IsHiBits ? SISrcMods::OP_SEL_1 : SISrcMods::OP_SEL_0;
  // Packed instructions (VOP3P) do not support ABS. Hence, no checks are done
  // for ABS modifiers.
  // If NEG or NEG_HI is true, we need to negate the corresponding 32 bit
  // lane.
  // NEG_HI shares the same bit position with ABS. But packed instructions do
  // not support ABS. Therefore, NEG_HI must be translated to NEG source
  // modifier for the higher 32 bits. Unpacked VOP3 instructions support
  // ABS, but do not support NEG_HI. Therefore we need to explicitly add the
  // NEG modifier if present in the packed instruction.
  if (SrcMods & NegModifier)
    NewSrcMods |= SISrcMods::NEG;
  // Src modifiers. Only negative modifiers are added if needed. Unpacked
  // operations do not have op_sel, therefore it must be handled explicitly as
  // done below.
  NewMI.addImm(NewSrcMods);
  if (SrcMO.isImm()) {
    NewMI.addImm(SrcMO.getImm());
    return;
  }
  // If op_sel == 0, select register 0 of reg:sub0_sub1.
  Register UnpackedSrcReg = (SrcMods & OpSelModifier)
                                ? TRI->getSubReg(SrcMO.getReg(), AMDGPU::sub1)
                                : TRI->getSubReg(SrcMO.getReg(), AMDGPU::sub0);

```
**EN:** This section contains concrete logic for SIPreEmitPeephole::addOperandAndMods. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SIPreEmitPeephole::addOperandAndMods`.
**CN:** 本节包含与 SIPreEmitPeephole::addOperandAndMods 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SIPreEmitPeephole::addOperandAndMods`。

### Lines 620-653: Conditional logic and checks
```cpp
  MachineOperand UnpackedSrcMO =
      MachineOperand::CreateReg(UnpackedSrcReg, /*isDef=*/false);
  if (SrcMO.isKill()) {
    // For each unpacked instruction, mark its source registers as killed if the
    // corresponding source register in the original packed instruction was
    // marked as killed.
    //
    // Exception:
    // If the op_sel and op_sel_hi modifiers require both unpacked instructions
    // to use the same register (e.g., due to overlapping access to low/high
    // bits of the same packed register), then only the *second* (latter)
    // instruction should mark the register as killed. This is because the
    // second instruction handles the higher bits and is effectively the last
    // user of the full register pair.

    bool OpSel = SrcMods & SISrcMods::OP_SEL_0;
    bool OpSelHi = SrcMods & SISrcMods::OP_SEL_1;
    bool KillState = true;
    if ((OpSel == OpSelHi) && !IsHiBits)
      KillState = false;
    UnpackedSrcMO.setIsKill(KillState);
  }
  NewMI.add(UnpackedSrcMO);
}

void SIPreEmitPeephole::collectUnpackingCandidates(
    MachineInstr &BeginMI, SetVector<MachineInstr *> &InstrsToUnpack,
    uint16_t NumMFMACycles) {
  auto *BB = BeginMI.getParent();
  auto E = BB->end();
  int TotalCyclesBetweenCandidates = 0;
  auto SchedModel = TII->getSchedModel();
  Register MFMADef = BeginMI.getOperand(0).getReg();

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `MachineOperand::CreateReg`, `SIPreEmitPeephole::collectUnpackingCandidates`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`MachineOperand::CreateReg`, `SIPreEmitPeephole::collectUnpackingCandidates`。

### Lines 654-688: Conditional logic and checks
```cpp
  for (auto I = std::next(BeginMI.getIterator()); I != E; ++I) {
    MachineInstr &Instr = *I;
    uint32_t UnpackedOpCode = mapToUnpackedOpcode(Instr);
    bool IsUnpackable =
        !(UnpackedOpCode == std::numeric_limits<uint32_t>::max());
    if (Instr.isMetaInstruction())
      continue;
    if ((Instr.isTerminator()) ||
        (TII->isNeverCoissue(Instr) && !IsUnpackable) ||
        (SIInstrInfo::modifiesModeRegister(Instr) &&
         Instr.modifiesRegister(AMDGPU::EXEC, TRI)))
      return;

    const MCSchedClassDesc *InstrSchedClassDesc =
        SchedModel.resolveSchedClass(&Instr);
    uint16_t Latency =
        SchedModel.getWriteProcResBegin(InstrSchedClassDesc)->ReleaseAtCycle;
    TotalCyclesBetweenCandidates += Latency;

    if (TotalCyclesBetweenCandidates >= NumMFMACycles - 1)
      return;
    // Identify register dependencies between those used by the MFMA
    // instruction and the following packed instructions. Also checks for
    // transitive dependencies between the MFMA def and candidate instruction
    // def and uses. Conservatively ensures that we do not incorrectly
    // read/write registers.
    for (const MachineOperand &InstrMO : Instr.operands()) {
      if (!InstrMO.isReg() || !InstrMO.getReg().isValid())
        continue;
      if (TRI->regsOverlap(MFMADef, InstrMO.getReg()))
        return;
    }
    if (!IsUnpackable)
      continue;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::next`, `SIInstrInfo::modifiesModeRegister`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::next`, `SIInstrInfo::modifiesModeRegister`。

### Lines 689-728: Conditional logic and checks
```cpp
    if (canUnpackingClobberRegister(Instr))
      return;
    // If it's a packed instruction, adjust latency: remove the packed
    // latency, add latency of two unpacked instructions (currently estimated
    // as 2 cycles).
    TotalCyclesBetweenCandidates -= Latency;
    // TODO: improve latency handling based on instruction modeling.
    TotalCyclesBetweenCandidates += 2;
    // Subtract 1 to account for MFMA issue latency.
    if (TotalCyclesBetweenCandidates < NumMFMACycles - 1)
      InstrsToUnpack.insert(&Instr);
  }
}

void SIPreEmitPeephole::performF32Unpacking(MachineInstr &I) {
  const MachineOperand &DstOp = I.getOperand(0);

  uint32_t UnpackedOpcode = mapToUnpackedOpcode(I);
  assert(UnpackedOpcode != std::numeric_limits<uint32_t>::max() &&
         "Unsupported Opcode");

  MachineInstrBuilder Op0LOp1L =
      createUnpackedMI(I, UnpackedOpcode, /*IsHiBits=*/false);
  MachineOperand LoDstOp = Op0LOp1L->getOperand(0);

  LoDstOp.setIsUndef(DstOp.isUndef());

  MachineInstrBuilder Op0HOp1H =
      createUnpackedMI(I, UnpackedOpcode, /*IsHiBits=*/true);
  MachineOperand HiDstOp = Op0HOp1H->getOperand(0);

  uint32_t IFlags = I.getFlags();
  Op0LOp1L->setFlags(IFlags);
  Op0HOp1H->setFlags(IFlags);
  LoDstOp.setIsRenamable(DstOp.isRenamable());
  HiDstOp.setIsRenamable(DstOp.isRenamable());

  I.eraseFromParent();
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `SIPreEmitPeephole::performF32Unpacking`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`SIPreEmitPeephole::performF32Unpacking`。

### Lines 729-771: Implements SIPreEmitPeephole::createUnpackedMI
```cpp
MachineInstrBuilder SIPreEmitPeephole::createUnpackedMI(MachineInstr &I,
                                                        uint32_t UnpackedOpcode,
                                                        bool IsHiBits) {
  MachineBasicBlock &MBB = *I.getParent();
  const DebugLoc &DL = I.getDebugLoc();
  const MachineOperand *SrcMO0 = TII->getNamedOperand(I, AMDGPU::OpName::src0);
  const MachineOperand *SrcMO1 = TII->getNamedOperand(I, AMDGPU::OpName::src1);
  Register DstReg = I.getOperand(0).getReg();
  unsigned OpCode = I.getOpcode();
  Register UnpackedDstReg = IsHiBits ? TRI->getSubReg(DstReg, AMDGPU::sub1)
                                     : TRI->getSubReg(DstReg, AMDGPU::sub0);

  int64_t ClampVal = TII->getNamedOperand(I, AMDGPU::OpName::clamp)->getImm();
  unsigned Src0Mods =
      TII->getNamedOperand(I, AMDGPU::OpName::src0_modifiers)->getImm();
  unsigned Src1Mods =
      TII->getNamedOperand(I, AMDGPU::OpName::src1_modifiers)->getImm();

  MachineInstrBuilder NewMI = BuildMI(MBB, I, DL, TII->get(UnpackedOpcode));
  NewMI.addDef(UnpackedDstReg); // vdst
  addOperandAndMods(NewMI, Src0Mods, IsHiBits, *SrcMO0);
  addOperandAndMods(NewMI, Src1Mods, IsHiBits, *SrcMO1);

  if (AMDGPU::hasNamedOperand(OpCode, AMDGPU::OpName::src2)) {
    const MachineOperand *SrcMO2 =
        TII->getNamedOperand(I, AMDGPU::OpName::src2);
    unsigned Src2Mods =
        TII->getNamedOperand(I, AMDGPU::OpName::src2_modifiers)->getImm();
    addOperandAndMods(NewMI, Src2Mods, IsHiBits, *SrcMO2);
  }
  NewMI.addImm(ClampVal); // clamp
  // Packed instructions do not support output modifiers. safe to assign them 0
  // for this use case
  NewMI.addImm(0); // omod
  return NewMI;
}

PreservedAnalyses
llvm::SIPreEmitPeepholePass::run(MachineFunction &MF,
                                 MachineFunctionAnalysisManager &MFAM) {
  auto *MLI = MFAM.getCachedResult<MachineLoopAnalysis>(MF);
  SIPreEmitPeephole Impl;

```
**EN:** This section contains concrete logic for SIPreEmitPeephole::createUnpackedMI. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SIPreEmitPeephole::createUnpackedMI`, `AMDGPU::hasNamedOperand`, `SIPreEmitPeepholePass::run`.
**CN:** 本节包含与 SIPreEmitPeephole::createUnpackedMI 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SIPreEmitPeephole::createUnpackedMI`, `AMDGPU::hasNamedOperand`, `SIPreEmitPeepholePass::run`。

### Lines 772-808: Implements SIPreEmitPeephole::run
```cpp
  if (Impl.run(MF, MLI)) {
    auto PA = getMachineFunctionPassPreservedAnalyses();
    PA.preserve<MachineLoopAnalysis>();
    return PA;
  }

  return PreservedAnalyses::all();
}

bool SIPreEmitPeephole::run(MachineFunction &MF, MachineLoopInfo *LoopInfo) {
  const GCNSubtarget &ST = MF.getSubtarget<GCNSubtarget>();
  TII = ST.getInstrInfo();
  TRI = &TII->getRegisterInfo();
  MLI = LoopInfo;
  bool Changed = false;

  MF.RenumberBlocks();

  for (MachineBasicBlock &MBB : MF) {
    MachineBasicBlock::iterator TermI = MBB.getFirstTerminator();
    // Check first terminator for branches to optimize
    if (TermI != MBB.end()) {
      MachineInstr &MI = *TermI;
      switch (MI.getOpcode()) {
      case AMDGPU::S_CBRANCH_VCCZ:
      case AMDGPU::S_CBRANCH_VCCNZ:
        Changed |= optimizeVccBranch(MI);
        break;
      case AMDGPU::S_CBRANCH_EXECZ:
        Changed |= removeExeczBranch(MI, MBB);
        break;
      }
    }

    if (!ST.hasVGPRIndexMode())
      continue;

```
**EN:** This section contains concrete logic for SIPreEmitPeephole::run. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `PreservedAnalyses::all`, `SIPreEmitPeephole::run`.
**CN:** 本节包含与 SIPreEmitPeephole::run 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`PreservedAnalyses::all`, `SIPreEmitPeephole::run`。

### Lines 809-841: Conditional logic and checks
```cpp
    MachineInstr *SetGPRMI = nullptr;
    const unsigned Threshold = 20;
    unsigned Count = 0;
    // Scan the block for two S_SET_GPR_IDX_ON instructions to see if a
    // second is not needed. Do expensive checks in the optimizeSetGPR()
    // and limit the distance to 20 instructions for compile time purposes.
    // Note: this needs to work on bundles as S_SET_GPR_IDX* instructions
    // may be bundled with the instructions they modify.
    for (auto &MI : make_early_inc_range(MBB.instrs())) {
      if (Count == Threshold)
        SetGPRMI = nullptr;
      else
        ++Count;

      if (MI.getOpcode() != AMDGPU::S_SET_GPR_IDX_ON)
        continue;

      Count = 0;
      if (!SetGPRMI) {
        SetGPRMI = &MI;
        continue;
      }

      if (optimizeSetGPR(*SetGPRMI, MI))
        Changed = true;
      else
        SetGPRMI = &MI;
    }
  }

  // TODO: Fold this into previous block, if possible. Evaluate and handle any
  // side effects.

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 842-865: Conditional logic and checks
```cpp
  // Perform the extra MF scans only for supported archs
  if (!ST.hasGFX940Insts())
    return Changed;
  for (MachineBasicBlock &MBB : MF) {
    // Unpack packed instructions overlapped by MFMAs. This allows the
    // compiler to co-issue unpacked instructions with MFMA
    auto SchedModel = TII->getSchedModel();
    SetVector<MachineInstr *> InstrsToUnpack;
    for (auto &MI : make_early_inc_range(MBB.instrs())) {
      if (!SIInstrInfo::isMFMA(MI))
        continue;
      const MCSchedClassDesc *SchedClassDesc =
          SchedModel.resolveSchedClass(&MI);
      uint16_t NumMFMACycles =
          SchedModel.getWriteProcResBegin(SchedClassDesc)->ReleaseAtCycle;
      collectUnpackingCandidates(MI, InstrsToUnpack, NumMFMACycles);
    }
    for (MachineInstr *MI : InstrsToUnpack) {
      performF32Unpacking(*MI);
    }
  }

  return Changed;
}
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `SIInstrInfo::isMFMA`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`SIInstrInfo::isMFMA`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `SIPreEmitPeephole`, `SIPreEmitPeepholeLegacy`, `BranchWeightCostModel`, `MachineFunctionPass::getAnalysisUsage`, `SIPreEmitPeephole::updateMLIBeforeRemovingEdge`, `std::prev`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; scheduling / 调度
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPU.h"`
- `"GCNSubtarget.h"`
- `"MCTargetDesc/AMDGPUMCTargetDesc.h"`
- `"llvm/ADT/SetVector.h"`
- `"llvm/CodeGen/MachineDominators.h"`
- `"llvm/CodeGen/MachineFunctionPass.h"`
- `"llvm/CodeGen/MachineLoopInfo.h"`
- `"llvm/CodeGen/MachinePostDominators.h"`
- `"llvm/CodeGen/TargetSchedule.h"`
- `"llvm/Support/BranchProbability.h"`
