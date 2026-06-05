# AMDGPUWaitSGPRHazards.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUWaitSGPRHazards.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUWaitSGPRHazards for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPUWaitSGPRHazards 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-33: File banner, includes, and setup
```cpp
//===- AMDGPUWaitSGPRHazards.cpp - Insert waits for SGPR read hazards -----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// Insert s_wait_alu instructions to mitigate SGPR read hazards on GFX12.
//
//===----------------------------------------------------------------------===//

#include "AMDGPUWaitSGPRHazards.h"
#include "AMDGPU.h"
#include "GCNSubtarget.h"
#include "MCTargetDesc/AMDGPUMCTargetDesc.h"
#include "SIInstrInfo.h"
#include "llvm/ADT/SetVector.h"
#include "llvm/TargetParser/TargetParser.h"

using namespace llvm;

#define DEBUG_TYPE "amdgpu-wait-sgpr-hazards"

static cl::opt<bool> GlobalEnableSGPRHazardWaits(
    "amdgpu-sgpr-hazard-wait", cl::init(true), cl::Hidden,
    cl::desc("Enable required s_wait_alu on SGPR hazards"));

static cl::opt<bool> GlobalCullSGPRHazardsOnFunctionBoundary(
    "amdgpu-sgpr-hazard-boundary-cull", cl::init(false), cl::Hidden,
    cl::desc("Cull hazards on function boundaries"));

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `cl::init`, `cl::desc`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`cl::init`, `cl::desc`。

### Lines 34-60: Declares class AMDGPUWaitSGPRHazards
```cpp
static cl::opt<bool>
    GlobalCullSGPRHazardsAtMemWait("amdgpu-sgpr-hazard-mem-wait-cull",
                                   cl::init(false), cl::Hidden,
                                   cl::desc("Cull hazards on memory waits"));

static cl::opt<unsigned> GlobalCullSGPRHazardsMemWaitThreshold(
    "amdgpu-sgpr-hazard-mem-wait-cull-threshold", cl::init(8), cl::Hidden,
    cl::desc("Number of tracked SGPRs before initiating hazard cull on memory "
             "wait"));

namespace {

class AMDGPUWaitSGPRHazards {
public:
  const GCNSubtarget *ST;
  const SIInstrInfo *TII;
  const SIRegisterInfo *TRI;
  const MachineRegisterInfo *MRI;
  unsigned DsNopCount;

  bool EnableSGPRHazardWaits;
  bool CullSGPRHazardsOnFunctionBoundary;
  bool CullSGPRHazardsAtMemWait;
  unsigned CullSGPRHazardsMemWaitThreshold;

  AMDGPUWaitSGPRHazards() = default;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUWaitSGPRHazards`, `cl::init`, `cl::desc`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUWaitSGPRHazards`, `cl::init`, `cl::desc`。

### Lines 61-90: Defines sgprNumber
```cpp
  // Return the numeric ID 0-127 for a given SGPR.
  static std::optional<unsigned> sgprNumber(Register Reg,
                                            const SIRegisterInfo &TRI) {
    switch (Reg) {
    case AMDGPU::M0:
    case AMDGPU::EXEC:
    case AMDGPU::EXEC_LO:
    case AMDGPU::EXEC_HI:
    case AMDGPU::SGPR_NULL:
    case AMDGPU::SGPR_NULL64:
      return {};
    default:
      break;
    }
    unsigned RegN = TRI.getHWRegIndex(Reg);
    if (RegN > 127)
      return {};
    return RegN;
  }

  static inline bool isVCC(Register Reg) {
    return Reg == AMDGPU::VCC || Reg == AMDGPU::VCC_LO || Reg == AMDGPU::VCC_HI;
  }

  // Adjust global offsets for instructions bundled with S_GETPC_B64 after
  // insertion of a new instruction.
  static void updateGetPCBundle(MachineInstr *NewMI) {
    if (!NewMI->isBundled())
      return;

```
**EN:** This section contains concrete logic for sgprNumber. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 sgprNumber 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 91-121: Declares struct HazardState
```cpp
    // Find start of bundle.
    auto I = NewMI->getIterator();
    while (I->isBundledWithPred())
      I--;
    if (I->isBundle())
      I++;

    // Bail if this is not an S_GETPC bundle.
    if (I->getOpcode() != AMDGPU::S_GETPC_B64)
      return;

    // Update offsets of any references in the bundle.
    const unsigned NewBytes = 4;
    assert(NewMI->getOpcode() == AMDGPU::S_WAITCNT_DEPCTR &&
           "Unexpected instruction insertion in bundle");
    auto NextMI = std::next(NewMI->getIterator());
    auto End = NewMI->getParent()->end();
    while (NextMI != End && NextMI->isBundledWithPred()) {
      for (auto &Operand : NextMI->operands()) {
        if (Operand.isGlobal())
          Operand.setOffset(Operand.getOffset() + NewBytes);
      }
      NextMI++;
    }
  }

  struct HazardState {
    static constexpr unsigned None = 0;
    static constexpr unsigned SALU = (1 << 0);
    static constexpr unsigned VALU = (1 << 1);

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `HazardState`, `std::next`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`HazardState`, `std::next`。

### Lines 122-155: Declares struct BlockHazardState
```cpp
    std::bitset<64> Tracked;      // SGPR banks ever read by VALU
    std::bitset<128> SALUHazards; // SGPRs with uncommitted values from SALU
    std::bitset<128> VALUHazards; // SGPRs with uncommitted values from VALU
    unsigned VCCHazard = None;    // Source of current VCC writes
    bool ActiveFlat = false;      // Has unwaited flat instructions

    bool merge(const HazardState &RHS) {
      HazardState Orig(*this);
      *this |= RHS;
      return (*this != Orig);
    }

    bool operator==(const HazardState &RHS) const {
      return Tracked == RHS.Tracked && SALUHazards == RHS.SALUHazards &&
             VALUHazards == RHS.VALUHazards && VCCHazard == RHS.VCCHazard &&
             ActiveFlat == RHS.ActiveFlat;
    }

    bool operator!=(const HazardState &RHS) const { return !(*this == RHS); }

    void operator|=(const HazardState &RHS) {
      Tracked |= RHS.Tracked;
      SALUHazards |= RHS.SALUHazards;
      VALUHazards |= RHS.VALUHazards;
      VCCHazard |= RHS.VCCHazard;
      ActiveFlat |= RHS.ActiveFlat;
    }
  };

  struct BlockHazardState {
    HazardState In;
    HazardState Out;
  };

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `BlockHazardState`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`BlockHazardState`。

### Lines 156-189: Defines insertHazardCull
```cpp
  DenseMap<const MachineBasicBlock *, BlockHazardState> BlockState;

  static constexpr unsigned WAVE32_NOPS = 4;
  static constexpr unsigned WAVE64_NOPS = 8;

  void insertHazardCull(MachineBasicBlock &MBB,
                        MachineBasicBlock::instr_iterator &MI) {
    assert(!MI->isBundled());
    unsigned Count = DsNopCount;
    while (Count--)
      BuildMI(MBB, MI, MI->getDebugLoc(), TII->get(AMDGPU::DS_NOP));
  }

  unsigned mergeMasks(unsigned Mask1, unsigned Mask2) {
    unsigned Mask = AMDGPU::DepCtr::getDefaultDepCtrEncoding(*ST);
    Mask = AMDGPU::DepCtr::encodeFieldSaSdst(
        Mask, std::min(AMDGPU::DepCtr::decodeFieldSaSdst(Mask1),
                       AMDGPU::DepCtr::decodeFieldSaSdst(Mask2)));
    Mask = AMDGPU::DepCtr::encodeFieldVaVcc(
        Mask, std::min(AMDGPU::DepCtr::decodeFieldVaVcc(Mask1),
                       AMDGPU::DepCtr::decodeFieldVaVcc(Mask2)));
    Mask = AMDGPU::DepCtr::encodeFieldVmVsrc(
        Mask, std::min(AMDGPU::DepCtr::decodeFieldVmVsrc(Mask1),
                       AMDGPU::DepCtr::decodeFieldVmVsrc(Mask2)));
    Mask = AMDGPU::DepCtr::encodeFieldVaSdst(
        Mask, std::min(AMDGPU::DepCtr::decodeFieldVaSdst(Mask1),
                       AMDGPU::DepCtr::decodeFieldVaSdst(Mask2)));
    Mask = AMDGPU::DepCtr::encodeFieldVaVdst(
        Mask, std::min(AMDGPU::DepCtr::decodeFieldVaVdst(Mask1),
                       AMDGPU::DepCtr::decodeFieldVaVdst(Mask2)));
    const AMDGPU::IsaVersion &Version = AMDGPU::getIsaVersion(ST->getCPU());
    Mask = AMDGPU::DepCtr::encodeFieldHoldCnt(
        Mask,
        std::min(AMDGPU::DepCtr::decodeFieldHoldCnt(Mask1, Version),
```
**EN:** This section contains concrete logic for insertHazardCull. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `DepCtr::getDefaultDepCtrEncoding`, `DepCtr::encodeFieldSaSdst`, `std::min`.
**CN:** 本节包含与 insertHazardCull 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`DepCtr::getDefaultDepCtrEncoding`, `DepCtr::encodeFieldSaSdst`, `std::min`。

### Lines 190-219: Defines mergeConsecutiveWaitAlus
```cpp
                 AMDGPU::DepCtr::decodeFieldHoldCnt(Mask2, Version)),
        Version);
    Mask = AMDGPU::DepCtr::encodeFieldVaSsrc(
        Mask, std::min(AMDGPU::DepCtr::decodeFieldVaSsrc(Mask1),
                       AMDGPU::DepCtr::decodeFieldVaSsrc(Mask2)));
    return Mask;
  }

  bool mergeConsecutiveWaitAlus(MachineBasicBlock::instr_iterator &MI,
                                unsigned Mask) {
    auto MBB = MI->getParent();
    if (MI == MBB->instr_begin())
      return false;

    auto It = prev_nodbg(MI, MBB->instr_begin());
    if (It->getOpcode() != AMDGPU::S_WAITCNT_DEPCTR)
      return false;

    It->getOperand(0).setImm(mergeMasks(Mask, It->getOperand(0).getImm()));
    return true;
  }

  bool runOnMachineBasicBlock(MachineBasicBlock &MBB, bool Emit) {
    enum { WA_VALU = 0x1, WA_SALU = 0x2, WA_VCC = 0x4 };

    HazardState State = BlockState[&MBB].In;
    SmallSet<Register, 8> SeenRegs;
    bool Emitted = false;
    unsigned DsNops = 0;

```
**EN:** This section contains concrete logic for mergeConsecutiveWaitAlus. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `DepCtr::decodeFieldHoldCnt`, `DepCtr::encodeFieldVaSsrc`, `std::min`.
**CN:** 本节包含与 mergeConsecutiveWaitAlus 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`DepCtr::decodeFieldHoldCnt`, `DepCtr::encodeFieldVaSsrc`, `std::min`。

### Lines 220-248: Conditional logic and checks
```cpp
    for (MachineBasicBlock::instr_iterator MI = MBB.instr_begin(),
                                           E = MBB.instr_end();
         MI != E; ++MI) {
      if (MI->isMetaInstruction())
        continue;

      // Clear tracked SGPRs if sufficient DS_NOPs occur
      if (MI->getOpcode() == AMDGPU::DS_NOP) {
        if (++DsNops >= DsNopCount)
          State.Tracked.reset();
        continue;
      }
      DsNops = 0;

      // Snoop FLAT instructions to avoid adding culls before scratch/lds loads.
      // Culls could be disproportionate in cost to load time.
      if (SIInstrInfo::isFLAT(*MI) && !SIInstrInfo::isFLATGlobal(*MI))
        State.ActiveFlat = true;

      // SMEM or VMEM clears hazards
      // FIXME: adapt to add FLAT without VALU (so !isLDSDMA())?
      if ((SIInstrInfo::isVMEM(*MI) && !SIInstrInfo::isFLAT(*MI)) ||
          SIInstrInfo::isSMRD(*MI)) {
        State.VCCHazard = HazardState::None;
        State.SALUHazards.reset();
        State.VALUHazards.reset();
        continue;
      }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `SIInstrInfo::isFLAT`, `SIInstrInfo::isFLATGlobal`, `SIInstrInfo::isVMEM`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`SIInstrInfo::isFLAT`, `SIInstrInfo::isFLATGlobal`, `SIInstrInfo::isVMEM`。

### Lines 249-279: Conditional logic and checks
```cpp
      // Existing S_WAITALU can clear hazards
      if (MI->getOpcode() == AMDGPU::S_WAITCNT_DEPCTR) {
        unsigned int Mask = MI->getOperand(0).getImm();
        if (AMDGPU::DepCtr::decodeFieldVaVcc(Mask) == 0)
          State.VCCHazard &= ~HazardState::VALU;
        if (AMDGPU::DepCtr::decodeFieldSaSdst(Mask) == 0) {
          State.SALUHazards.reset();
          State.VCCHazard &= ~HazardState::SALU;
        }
        if (AMDGPU::DepCtr::decodeFieldVaSdst(Mask) == 0)
          State.VALUHazards.reset();
        continue;
      }

      // Snoop counter waits to insert culls
      if (CullSGPRHazardsAtMemWait &&
          (MI->getOpcode() == AMDGPU::S_WAIT_LOADCNT ||
           MI->getOpcode() == AMDGPU::S_WAIT_SAMPLECNT ||
           MI->getOpcode() == AMDGPU::S_WAIT_BVHCNT) &&
          (MI->getOperand(0).isImm() && MI->getOperand(0).getImm() == 0) &&
          (State.Tracked.count() >= CullSGPRHazardsMemWaitThreshold)) {
        if (MI->getOpcode() == AMDGPU::S_WAIT_LOADCNT && State.ActiveFlat) {
          State.ActiveFlat = false;
        } else {
          State.Tracked.reset();
          if (Emit)
            insertHazardCull(MBB, MI);
          continue;
        }
      }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `DepCtr::decodeFieldVaVcc`, `DepCtr::decodeFieldSaSdst`, `DepCtr::decodeFieldVaSdst`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`DepCtr::decodeFieldVaVcc`, `DepCtr::decodeFieldSaSdst`, `DepCtr::decodeFieldVaSdst`。

### Lines 280-308: Conditional logic and checks
```cpp
      // Process only VALUs and SALUs
      bool IsVALU = SIInstrInfo::isVALU(*MI);
      bool IsSALU = SIInstrInfo::isSALU(*MI);
      if (!IsVALU && !IsSALU)
        continue;

      unsigned Wait = 0;

      auto processOperand = [&](const MachineOperand &Op, bool IsUse) {
        if (!Op.isReg())
          return;
        Register Reg = Op.getReg();
        assert(!Op.getSubReg());
        if (!TRI->isSGPRReg(*MRI, Reg))
          return;

        // Only visit each register once
        if (!SeenRegs.insert(Reg).second)
          return;

        auto RegNumber = sgprNumber(Reg, *TRI);
        if (!RegNumber)
          return;

        // Track SGPRs by pair -- numeric ID of an 64b SGPR pair.
        // i.e. SGPR0 = SGPR0_SGPR1 = 0, SGPR3 = SGPR2_SGPR3 = 1, etc
        unsigned RegN = *RegNumber;
        unsigned PairN = (RegN >> 1) & 0x3f;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `SIInstrInfo::isVALU`, `SIInstrInfo::isSALU`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`SIInstrInfo::isVALU`, `SIInstrInfo::isSALU`。

### Lines 309-342: Conditional logic and checks
```cpp
        // Read/write of untracked register is safe; but must record any new
        // reads.
        if (!State.Tracked[PairN]) {
          if (IsVALU && IsUse)
            State.Tracked.set(PairN);
          return;
        }

        uint8_t SGPRCount =
            AMDGPU::getRegBitWidth(*TRI->getRegClassForReg(*MRI, Reg)) / 32;

        if (IsUse) {
          // SALU reading SGPR clears VALU hazards
          if (IsSALU) {
            if (isVCC(Reg)) {
              if (State.VCCHazard & HazardState::VALU)
                State.VCCHazard = HazardState::None;
            } else {
              State.VALUHazards.reset();
            }
          }
          // Compute required waits
          for (uint8_t RegIdx = 0; RegIdx < SGPRCount; ++RegIdx) {
            Wait |= State.SALUHazards[RegN + RegIdx] ? WA_SALU : 0;
            Wait |= IsVALU && State.VALUHazards[RegN + RegIdx] ? WA_VALU : 0;
          }
          if (isVCC(Reg) && State.VCCHazard) {
            // Note: it's possible for both SALU and VALU to exist if VCC
            // was updated differently by merged predecessors.
            if (State.VCCHazard & HazardState::SALU)
              Wait |= WA_SALU;
            if (State.VCCHazard & HazardState::VALU)
              Wait |= WA_VCC;
          }
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::getRegBitWidth`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::getRegBitWidth`。

### Lines 343-367: Conditional logic and checks
```cpp
        } else {
          // Update hazards
          if (isVCC(Reg)) {
            State.VCCHazard = IsSALU ? HazardState::SALU : HazardState::VALU;
          } else {
            for (uint8_t RegIdx = 0; RegIdx < SGPRCount; ++RegIdx) {
              if (IsSALU)
                State.SALUHazards.set(RegN + RegIdx);
              else
                State.VALUHazards.set(RegN + RegIdx);
            }
          }
        }
      };

      const bool IsSetPC =
          (MI->isCall() || MI->isReturn() || MI->isIndirectBranch()) &&
          MI->getOpcode() != AMDGPU::S_ENDPGM &&
          MI->getOpcode() != AMDGPU::S_ENDPGM_SAVED;

      // Only consider implicit VCC specified by instruction descriptor.
      const bool HasImplicitVCC =
          llvm::any_of(MI->getDesc().implicit_uses(), isVCC) ||
          llvm::any_of(MI->getDesc().implicit_defs(), isVCC);

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `llvm::any_of`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`llvm::any_of`。

### Lines 368-392: Conditional logic and checks
```cpp
      if (IsSetPC) {
        // All SGPR writes before a call/return must be flushed as the
        // callee/caller will not will not see the hazard chain.
        if (State.VCCHazard & HazardState::VALU)
          Wait |= WA_VCC;
        if (State.SALUHazards.any() || (State.VCCHazard & HazardState::SALU))
          Wait |= WA_SALU;
        if (State.VALUHazards.any())
          Wait |= WA_VALU;
        if (CullSGPRHazardsOnFunctionBoundary && State.Tracked.any()) {
          State.Tracked.reset();
          if (Emit)
            insertHazardCull(MBB, MI);
        }
      } else {
        // Process uses to determine required wait.
        SeenRegs.clear();
        for (const MachineOperand &Op : MI->all_uses()) {
          if (Op.isImplicit() &&
              (!HasImplicitVCC || !Op.isReg() || !isVCC(Op.getReg())))
            continue;
          processOperand(Op, true);
        }
      }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 393-423: Conditional logic and checks
```cpp
      // Apply wait
      if (Wait) {
        unsigned Mask = AMDGPU::DepCtr::getDefaultDepCtrEncoding(*ST);
        if (Wait & WA_VCC) {
          State.VCCHazard &= ~HazardState::VALU;
          Mask = AMDGPU::DepCtr::encodeFieldVaVcc(Mask, 0);
        }
        if (Wait & WA_SALU) {
          State.SALUHazards.reset();
          State.VCCHazard &= ~HazardState::SALU;
          Mask = AMDGPU::DepCtr::encodeFieldSaSdst(Mask, 0);
        }
        if (Wait & WA_VALU) {
          State.VALUHazards.reset();
          Mask = AMDGPU::DepCtr::encodeFieldVaSdst(Mask, 0);
        }
        if (Emit) {
          if (!mergeConsecutiveWaitAlus(MI, Mask)) {
            auto NewMI = BuildMI(MBB, MI, MI->getDebugLoc(),
                                 TII->get(AMDGPU::S_WAITCNT_DEPCTR))
                             .addImm(Mask);
            updateGetPCBundle(NewMI);
          }
          Emitted = true;
        }
      }

      // On return from a call SGPR state is unknown, so all potential hazards.
      if (MI->isCall() && !CullSGPRHazardsOnFunctionBoundary)
        State.Tracked.set();

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `DepCtr::getDefaultDepCtrEncoding`, `DepCtr::encodeFieldVaVcc`, `DepCtr::encodeFieldSaSdst`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`DepCtr::getDefaultDepCtrEncoding`, `DepCtr::encodeFieldVaVcc`, `DepCtr::encodeFieldSaSdst`。

### Lines 424-455: Conditional logic and checks
```cpp
      // Update hazards based on defs.
      SeenRegs.clear();
      for (const MachineOperand &Op : MI->all_defs()) {
        if (Op.isImplicit() &&
            (!HasImplicitVCC || !Op.isReg() || !isVCC(Op.getReg())))
          continue;
        processOperand(Op, false);
      }
    }

    BlockHazardState &BS = BlockState[&MBB];
    bool Changed = State != BS.Out;
    if (Emit) {
      assert(!Changed && "Hazard state should not change on emit pass");
      return Emitted;
    }
    if (Changed)
      BS.Out = State;
    return Changed;
  }

  bool run(MachineFunction &MF) {
    ST = &MF.getSubtarget<GCNSubtarget>();
    if (!ST->hasVALUReadSGPRHazard())
      return false;

    // Parse settings
    EnableSGPRHazardWaits = GlobalEnableSGPRHazardWaits;
    CullSGPRHazardsOnFunctionBoundary = GlobalCullSGPRHazardsOnFunctionBoundary;
    CullSGPRHazardsAtMemWait = GlobalCullSGPRHazardsAtMemWait;
    CullSGPRHazardsMemWaitThreshold = GlobalCullSGPRHazardsMemWaitThreshold;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 456-488: Conditional logic and checks
```cpp
    if (!GlobalEnableSGPRHazardWaits.getNumOccurrences())
      EnableSGPRHazardWaits = MF.getFunction().getFnAttributeAsParsedInteger(
          "amdgpu-sgpr-hazard-wait", EnableSGPRHazardWaits);
    if (!GlobalCullSGPRHazardsOnFunctionBoundary.getNumOccurrences())
      CullSGPRHazardsOnFunctionBoundary =
          MF.getFunction().hasFnAttribute("amdgpu-sgpr-hazard-boundary-cull");
    if (!GlobalCullSGPRHazardsAtMemWait.getNumOccurrences())
      CullSGPRHazardsAtMemWait =
          MF.getFunction().hasFnAttribute("amdgpu-sgpr-hazard-mem-wait-cull");
    if (!GlobalCullSGPRHazardsMemWaitThreshold.getNumOccurrences())
      CullSGPRHazardsMemWaitThreshold =
          MF.getFunction().getFnAttributeAsParsedInteger(
              "amdgpu-sgpr-hazard-mem-wait-cull-threshold",
              CullSGPRHazardsMemWaitThreshold);

    // Bail if disabled
    if (!EnableSGPRHazardWaits)
      return false;

    TII = ST->getInstrInfo();
    TRI = ST->getRegisterInfo();
    MRI = &MF.getRegInfo();
    DsNopCount = ST->isWave64() ? WAVE64_NOPS : WAVE32_NOPS;

    auto CallingConv = MF.getFunction().getCallingConv();
    if (!AMDGPU::isEntryFunctionCC(CallingConv) &&
        !CullSGPRHazardsOnFunctionBoundary) {
      // Callee must consider all SGPRs as tracked.
      LLVM_DEBUG(dbgs() << "Is called function, track all SGPRs.\n");
      MachineBasicBlock &EntryBlock = MF.front();
      BlockState[&EntryBlock].In.Tracked.set();
    }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::isEntryFunctionCC`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::isEntryFunctionCC`。

### Lines 489-521: Conditional logic and checks
```cpp
    // Calculate the hazard state for each basic block.
    // Iterate until a fixed point is reached.
    // Fixed point is guaranteed as merge function only ever increases
    // the hazard set, and all backedges will cause a merge.
    //
    // Note: we have to take care of the entry block as this technically
    // has an edge from outside the function. Failure to treat this as
    // a merge could prevent fixed point being reached.
    SetVector<MachineBasicBlock *> Worklist;
    for (auto &MBB : reverse(MF))
      Worklist.insert(&MBB);
    while (!Worklist.empty()) {
      auto &MBB = *Worklist.pop_back_val();
      bool Changed = runOnMachineBasicBlock(MBB, false);
      if (Changed) {
        // Note: take a copy of state here in case it is reallocated by map
        HazardState NewState = BlockState[&MBB].Out;
        // Propagate to all successor blocks
        for (auto Succ : MBB.successors()) {
          // We only need to merge hazards at CFG merge points.
          auto &SuccState = BlockState[Succ];
          if (Succ->getSinglePredecessor() && !Succ->isEntryBlock()) {
            if (SuccState.In != NewState) {
              SuccState.In = NewState;
              Worklist.insert(Succ);
            }
          } else if (SuccState.In.merge(NewState)) {
            Worklist.insert(Succ);
          }
        }
      }
    }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 522-555: Declares class AMDGPUWaitSGPRHazardsLegacy
```cpp
    LLVM_DEBUG(dbgs() << "Emit s_wait_alu instructions\n");

    // Final to emit wait instructions.
    bool Changed = false;
    for (auto &MBB : MF)
      Changed |= runOnMachineBasicBlock(MBB, true);

    BlockState.clear();
    return Changed;
  }
};

class AMDGPUWaitSGPRHazardsLegacy : public MachineFunctionPass {
public:
  static char ID;

  AMDGPUWaitSGPRHazardsLegacy() : MachineFunctionPass(ID) {}

  bool runOnMachineFunction(MachineFunction &MF) override {
    return AMDGPUWaitSGPRHazards().run(MF);
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    MachineFunctionPass::getAnalysisUsage(AU);
  }
};

} // namespace

char AMDGPUWaitSGPRHazardsLegacy::ID = 0;

char &llvm::AMDGPUWaitSGPRHazardsLegacyID = AMDGPUWaitSGPRHazardsLegacy::ID;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUWaitSGPRHazardsLegacy`, `MachineFunctionPass::getAnalysisUsage`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUWaitSGPRHazardsLegacy`, `MachineFunctionPass::getAnalysisUsage`。

### Lines 556-565: Registers LLVM passes
```cpp
INITIALIZE_PASS(AMDGPUWaitSGPRHazardsLegacy, DEBUG_TYPE,
                "AMDGPU Insert waits for SGPR read hazards", false, false)

PreservedAnalyses
AMDGPUWaitSGPRHazardsPass::run(MachineFunction &MF,
                               MachineFunctionAnalysisManager &MFAM) {
  if (AMDGPUWaitSGPRHazards().run(MF))
    return getMachineFunctionPassPreservedAnalyses();
  return PreservedAnalyses::all();
}
```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `AMDGPUWaitSGPRHazardsPass::run`, `PreservedAnalyses::all`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`AMDGPUWaitSGPRHazardsPass::run`, `PreservedAnalyses::all`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `AMDGPUWaitSGPRHazards`, `HazardState`, `BlockHazardState`, `AMDGPUWaitSGPRHazardsLegacy`, `cl::init`, `cl::desc`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPUWaitSGPRHazards.h"`
- `"AMDGPU.h"`
- `"GCNSubtarget.h"`
- `"MCTargetDesc/AMDGPUMCTargetDesc.h"`
- `"SIInstrInfo.h"`
- `"llvm/ADT/SetVector.h"`
- `"llvm/TargetParser/TargetParser.h"`
