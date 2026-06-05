# GCNVOPDUtils.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/GCNVOPDUtils.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements GCNVOPDUtils for the LLVM backend utilities. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM 后端工具中的 GCNVOPDUtils 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-34: File banner, includes, and setup
```cpp
//===- GCNVOPDUtils.cpp - GCN VOPD Utils  ------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file This file contains the AMDGPU DAG scheduling
/// mutation to pair VOPD instructions back to back. It also contains
//  subroutines useful in the creation of VOPD instructions
//
//===----------------------------------------------------------------------===//

#include "GCNVOPDUtils.h"
#include "AMDGPUSubtarget.h"
#include "GCNSubtarget.h"
#include "MCTargetDesc/AMDGPUMCTargetDesc.h"
#include "SIInstrInfo.h"
#include "Utils/AMDGPUBaseInfo.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/MacroFusion.h"
#include "llvm/CodeGen/ScheduleDAG.h"
#include "llvm/CodeGen/ScheduleDAGMutation.h"
#include "llvm/CodeGen/TargetInstrInfo.h"
#include "llvm/MC/MCInst.h"

using namespace llvm;

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 35-65: Preprocessor guards and macros
```cpp
#define DEBUG_TYPE "gcn-vopd-utils"

bool llvm::checkVOPDRegConstraints(const SIInstrInfo &TII,
                                   const MachineInstr &MIX,
                                   const MachineInstr &MIY, bool IsVOPD3,
                                   bool AllowSameVGPR) {
  namespace VOPD = AMDGPU::VOPD;

  const MachineFunction *MF = MIX.getMF();
  const GCNSubtarget &ST = MF->getSubtarget<GCNSubtarget>();

  if (IsVOPD3 && !ST.hasVOPD3())
    return false;
  if (!IsVOPD3 && (TII.isVOP3(MIX) || TII.isVOP3(MIY)))
    return false;
  if (TII.isDPP(MIX) || TII.isDPP(MIY))
    return false;

  const SIRegisterInfo *TRI = ST.getRegisterInfo();
  const MachineRegisterInfo &MRI = MF->getRegInfo();
  // Literals also count against scalar bus limit
  SmallVector<const MachineOperand *> UniqueLiterals;
  auto addLiteral = [&](const MachineOperand &Op) {
    for (auto &Literal : UniqueLiterals) {
      if (Literal->isIdenticalTo(Op))
        return;
    }
    UniqueLiterals.push_back(&Op);
  };
  SmallSet<Register, 4> UniqueScalarRegs;

```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `llvm::checkVOPDRegConstraints`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`llvm::checkVOPDRegConstraints`。

### Lines 66-99: Implements AMDGPU::getVOPDInstInfo
```cpp
  auto getVRegIdx = [&](unsigned OpcodeIdx, unsigned OperandIdx) {
    const MachineInstr &MI = (OpcodeIdx == VOPD::X) ? MIX : MIY;
    const MachineOperand &Operand = MI.getOperand(OperandIdx);
    if (Operand.isReg() && TRI->isVectorRegister(MRI, Operand.getReg()))
      return Operand.getReg();
    return Register();
  };

  auto InstInfo = AMDGPU::getVOPDInstInfo(MIX.getDesc(), MIY.getDesc());

  for (auto CompIdx : VOPD::COMPONENTS) {
    const MachineInstr &MI = (CompIdx == VOPD::X) ? MIX : MIY;

    const MachineOperand &Src0 = *TII.getNamedOperand(MI, AMDGPU::OpName::src0);
    if (Src0.isReg()) {
      if (!TRI->isVectorRegister(MRI, Src0.getReg())) {
        UniqueScalarRegs.insert(Src0.getReg());
      }
    } else if (!TII.isInlineConstant(Src0)) {
      if (IsVOPD3)
        return false;
      addLiteral(Src0);
    }

    if (InstInfo[CompIdx].hasMandatoryLiteral()) {
      if (IsVOPD3)
        return false;

      auto CompOprIdx = InstInfo[CompIdx].getMandatoryLiteralCompOperandIndex();
      addLiteral(MI.getOperand(CompOprIdx));
    }
    if (MI.getDesc().hasImplicitUseOfPhysReg(AMDGPU::VCC))
      UniqueScalarRegs.insert(AMDGPU::VCC_LO);

```
**EN:** This section contains concrete logic for AMDGPU::getVOPDInstInfo. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPU::getVOPDInstInfo`.
**CN:** 本节包含与 AMDGPU::getVOPDInstInfo 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPU::getVOPDInstInfo`。

### Lines 100-122: Conditional logic and checks
```cpp
    if (IsVOPD3) {
      for (auto OpName : {AMDGPU::OpName::src1, AMDGPU::OpName::src2}) {
        const MachineOperand *Src = TII.getNamedOperand(MI, OpName);
        if (!Src)
          continue;
        if (OpName == AMDGPU::OpName::src2) {
          if (AMDGPU::hasNamedOperand(MI.getOpcode(), AMDGPU::OpName::bitop3))
            continue;
          if (MI.getOpcode() == AMDGPU::V_CNDMASK_B32_e64) {
            UniqueScalarRegs.insert(Src->getReg());
            continue;
          }
        }
        if (!Src->isReg() || !TRI->isVGPR(MRI, Src->getReg()))
          return false;
      }

      for (auto OpName : {AMDGPU::OpName::clamp, AMDGPU::OpName::omod,
                          AMDGPU::OpName::op_sel}) {
        if (TII.hasModifiersSet(MI, OpName))
          return false;
      }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::hasNamedOperand`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::hasNamedOperand`。

### Lines 123-149: Conditional logic and checks
```cpp
      // Neg is allowed, other modifiers are not. NB: even though sext has the
      // same value as neg, there are no combinable instructions with sext.
      for (auto OpName :
           {AMDGPU::OpName::src0_modifiers, AMDGPU::OpName::src1_modifiers,
            AMDGPU::OpName::src2_modifiers}) {
        const MachineOperand *Mods = TII.getNamedOperand(MI, OpName);
        if (Mods && (Mods->getImm() & ~SISrcMods::NEG))
          return false;
      }
    }
  }

  if (UniqueLiterals.size() > 1)
    return false;
  if ((UniqueLiterals.size() + UniqueScalarRegs.size()) > 2)
    return false;

  // On GFX1170+ if both OpX and OpY are V_MOV_B32 then OPY uses SRC2
  // source-cache.
  bool SkipSrc = (ST.hasGFX11_7Insts() || ST.hasGFX12Insts()) &&
                 MIX.getOpcode() == AMDGPU::V_MOV_B32_e32 &&
                 MIY.getOpcode() == AMDGPU::V_MOV_B32_e32;

  if (InstInfo.hasInvalidOperand(getVRegIdx, *TRI, SkipSrc, AllowSameVGPR,
                                 IsVOPD3))
    return false;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 150-171: Conditional logic and checks
```cpp
  if (IsVOPD3) {
    // BITOP3 can be converted to DUAL_BITOP2 only if src2 is zero.
    // MIX check is only relevant to scheduling?
    if (AMDGPU::hasNamedOperand(MIX.getOpcode(), AMDGPU::OpName::bitop3)) {
      const MachineOperand &Src2 =
          *TII.getNamedOperand(MIX, AMDGPU::OpName::src2);
      if (!Src2.isImm() || Src2.getImm())
        return false;
    }
    if (AMDGPU::hasNamedOperand(MIY.getOpcode(), AMDGPU::OpName::bitop3)) {
      const MachineOperand &Src2 =
          *TII.getNamedOperand(MIY, AMDGPU::OpName::src2);
      if (!Src2.isImm() || Src2.getImm())
        return false;
    }
  }

  LLVM_DEBUG(dbgs() << "VOPD Reg Constraints Passed\n\tX: " << MIX
                    << "\n\tY: " << MIY << "\n");
  return true;
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::hasNamedOperand`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::hasNamedOperand`。

### Lines 172-201: Defines tryMatchVOPDPairVariant
```cpp
/// Core pair-eligibility check for a single VOPD encoding variant (VOPD or
/// VOPD3).  Returns the X/Y assignment on success, or std::nullopt otherwise.
static std::optional<VOPDMatchInfo>
tryMatchVOPDPairVariant(const SIInstrInfo &TII, unsigned EncodingFamily,
                        MachineInstr &FirstMI, MachineInstr &SecondMI,
                        bool IsVOPD3) {
  unsigned Opc = FirstMI.getOpcode();
  unsigned Opc2 = SecondMI.getOpcode();
  AMDGPU::CanBeVOPD FirstCanBeVOPD =
      AMDGPU::getCanBeVOPD(Opc, EncodingFamily, IsVOPD3);
  AMDGPU::CanBeVOPD SecondCanBeVOPD =
      AMDGPU::getCanBeVOPD(Opc2, EncodingFamily, IsVOPD3);

  // If SecondMI depends on FirstMI they cannot execute at the same time.
  if (TII.hasRAWDependency(FirstMI, SecondMI))
    return std::nullopt;

  const GCNSubtarget &ST = TII.getSubtarget();
  bool AllowSameVGPR = ST.hasGFX12Insts();

  if (FirstCanBeVOPD.X && SecondCanBeVOPD.Y) {
    if (checkVOPDRegConstraints(TII, FirstMI, SecondMI, IsVOPD3, AllowSameVGPR))
      return VOPDMatchInfo{&FirstMI, &SecondMI, IsVOPD3};
  }

  // AllowSameVGPR relaxes the VGPR bank overlap check for source operands.
  // Only enable it when there is no antidependency.
  bool IsAntiDep = TII.hasRAWDependency(SecondMI, FirstMI);
  AllowSameVGPR &= !IsAntiDep;

```
**EN:** This section contains concrete logic for tryMatchVOPDPairVariant. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPU::getCanBeVOPD`.
**CN:** 本节包含与 tryMatchVOPDPairVariant 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPU::getCanBeVOPD`。

### Lines 202-235: Implements llvm::tryMatchVOPDPair
```cpp
  if (FirstCanBeVOPD.Y && SecondCanBeVOPD.X) {
    if (IsAntiDep && !TII.isVOPDAntidependencyAllowed(SecondMI))
      return std::nullopt;
    if (checkVOPDRegConstraints(TII, SecondMI, FirstMI, IsVOPD3, AllowSameVGPR))
      return VOPDMatchInfo{&SecondMI, &FirstMI, IsVOPD3};
  }

  return std::nullopt;
}

std::optional<VOPDMatchInfo> llvm::tryMatchVOPDPair(const SIInstrInfo &TII,
                                                    MachineInstr &FirstMI,
                                                    MachineInstr &SecondMI) {
  const GCNSubtarget &ST = TII.getSubtarget();
  unsigned EncodingFamily = AMDGPU::getVOPDEncodingFamily(ST);
  if (auto Match = tryMatchVOPDPairVariant(TII, EncodingFamily, FirstMI,
                                           SecondMI, /*IsVOPD3=*/false))
    return Match;
  if (ST.hasVOPD3())
    return tryMatchVOPDPairVariant(TII, EncodingFamily, FirstMI, SecondMI,
                                   /*IsVOPD3=*/true);
  return std::nullopt;
}

/// Check if the instr pair, FirstMI and SecondMI, should be scheduled
/// together. Given SecondMI, when FirstMI is unspecified, then check if
/// SecondMI may be part of a fused pair at all.
static bool shouldScheduleVOPDAdjacent(const TargetInstrInfo &TII,
                                       const TargetSubtargetInfo &TSI,
                                       const MachineInstr *FirstMI,
                                       const MachineInstr &SecondMI) {
  const SIInstrInfo &STII = static_cast<const SIInstrInfo &>(TII);
  const GCNSubtarget &ST = STII.getSubtarget();

```
**EN:** This section contains concrete logic for llvm::tryMatchVOPDPair. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `llvm::tryMatchVOPDPair`, `AMDGPU::getVOPDEncodingFamily`.
**CN:** 本节包含与 llvm::tryMatchVOPDPair 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`llvm::tryMatchVOPDPair`, `AMDGPU::getVOPDEncodingFamily`。

### Lines 236-263: Preprocessor guards and macros
```cpp
  // One instruction case: just check whether SecondMI is eligible at all.
  if (!FirstMI) {
    unsigned EncodingFamily = AMDGPU::getVOPDEncodingFamily(ST);
    unsigned Opc2 = SecondMI.getOpcode();
    auto checkCanBeVOPD = [&](bool VOPD3) {
      AMDGPU::CanBeVOPD CanBeVOPD =
          AMDGPU::getCanBeVOPD(Opc2, EncodingFamily, VOPD3);
      return CanBeVOPD.Y || CanBeVOPD.X;
    };
    return checkCanBeVOPD(false) || (ST.hasVOPD3() && checkCanBeVOPD(true));
  }

#ifdef EXPENSIVE_CHECKS
  assert([&]() -> bool {
    for (auto MII = MachineBasicBlock::const_iterator(FirstMI);
         MII != FirstMI->getParent()->instr_end(); ++MII) {
      if (&*MII == &SecondMI)
        return true;
    }
    return false;
  }() && "Expected FirstMI to precede SecondMI");
#endif

  return tryMatchVOPDPair(STII, *const_cast<MachineInstr *>(FirstMI),
                          const_cast<MachineInstr &>(SecondMI))
      .has_value();
}

```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `AMDGPU::getVOPDEncodingFamily`, `AMDGPU::getCanBeVOPD`, `MachineBasicBlock::const_iterator`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`AMDGPU::getVOPDEncodingFamily`, `AMDGPU::getCanBeVOPD`, `MachineBasicBlock::const_iterator`。

### Lines 264-291: Declares struct VOPDPairingMutation
```cpp
namespace {
/// Adapts design from MacroFusion
/// Puts valid candidate instructions back-to-back so they can easily
/// be turned into VOPD instructions
/// Greedily pairs instruction candidates. O(n^2) algorithm.
struct VOPDPairingMutation : ScheduleDAGMutation {
  MacroFusionPredTy shouldScheduleAdjacent; // NOLINT: function pointer

  VOPDPairingMutation(
      MacroFusionPredTy shouldScheduleAdjacent) // NOLINT: function pointer
      : shouldScheduleAdjacent(shouldScheduleAdjacent) {}

  void apply(ScheduleDAGInstrs *DAG) override {
    const TargetInstrInfo &TII = *DAG->TII;
    const GCNSubtarget &ST = DAG->MF.getSubtarget<GCNSubtarget>();
    if (!AMDGPU::hasVOPD(ST) || !ST.isWave32()) {
      LLVM_DEBUG(dbgs() << "Target does not support VOPDPairingMutation\n");
      return;
    }

    std::vector<SUnit>::iterator ISUI, JSUI;
    for (ISUI = DAG->SUnits.begin(); ISUI != DAG->SUnits.end(); ++ISUI) {
      const MachineInstr *IMI = ISUI->getInstr();
      if (!shouldScheduleAdjacent(TII, ST, nullptr, *IMI))
        continue;
      if (!hasLessThanNumFused(*ISUI, 2))
        continue;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `VOPDPairingMutation`, `AMDGPU::hasVOPD`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`VOPDPairingMutation`, `AMDGPU::hasVOPD`。

### Lines 292-310: Conditional logic and checks
```cpp
      for (JSUI = ISUI + 1; JSUI != DAG->SUnits.end(); ++JSUI) {
        if (JSUI->isBoundaryNode())
          continue;
        const MachineInstr *JMI = JSUI->getInstr();
        if (!hasLessThanNumFused(*JSUI, 2) ||
            !shouldScheduleAdjacent(TII, ST, IMI, *JMI))
          continue;
        if (fuseInstructionPair(*DAG, *ISUI, *JSUI))
          break;
      }
    }
    LLVM_DEBUG(dbgs() << "Completed VOPDPairingMutation\n");
  }
};
} // namespace

std::unique_ptr<ScheduleDAGMutation> llvm::createVOPDPairingMutation() {
  return std::make_unique<VOPDPairingMutation>(shouldScheduleVOPDAdjacent);
}
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `llvm::createVOPDPairingMutation`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`llvm::createVOPDPairingMutation`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `VOPDPairingMutation`, `llvm::checkVOPDRegConstraints`, `AMDGPU::getVOPDInstInfo`, `AMDGPU::hasNamedOperand`, `AMDGPU::getCanBeVOPD`, `llvm::tryMatchVOPDPair`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; scheduling / 调度
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"GCNVOPDUtils.h"`
- `"AMDGPUSubtarget.h"`
- `"GCNSubtarget.h"`
- `"MCTargetDesc/AMDGPUMCTargetDesc.h"`
- `"SIInstrInfo.h"`
- `"Utils/AMDGPUBaseInfo.h"`
- `"llvm/ADT/STLExtras.h"`
- `"llvm/ADT/SmallVector.h"`
- `"llvm/CodeGen/MachineBasicBlock.h"`
- `"llvm/CodeGen/MachineInstr.h"`
- `"llvm/CodeGen/MachineOperand.h"`
- `"llvm/CodeGen/MachineRegisterInfo.h"`
- `"llvm/CodeGen/MacroFusion.h"`
- `"llvm/CodeGen/ScheduleDAG.h"`
- `"llvm/CodeGen/ScheduleDAGMutation.h"`
- `"llvm/CodeGen/TargetInstrInfo.h"`
- `"llvm/MC/MCInst.h"`
