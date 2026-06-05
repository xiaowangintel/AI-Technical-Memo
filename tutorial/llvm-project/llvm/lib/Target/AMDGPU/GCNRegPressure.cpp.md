# GCNRegPressure.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/GCNRegPressure.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements GCNRegPressure for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 GCNRegPressure 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40: File banner, includes, and setup
```cpp
//===- GCNRegPressure.cpp -------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///
/// \file
/// This file implements the GCNRegPressure class.
///
//===----------------------------------------------------------------------===//

#include "GCNRegPressure.h"
#include "AMDGPU.h"
#include "SIMachineFunctionInfo.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineLoopInfo.h"
#include "llvm/CodeGen/RegisterPressure.h"

using namespace llvm;

#define DEBUG_TYPE "machine-scheduler"

bool llvm::isEqual(const GCNRPTracker::LiveRegSet &S1,
                   const GCNRPTracker::LiveRegSet &S2) {
  if (S1.size() != S2.size())
    return false;

  for (const auto &P : S1) {
    auto I = S2.find(P.first);
    if (I == S2.end() || I->second != P.second)
      return false;
  }
  return true;
}

///////////////////////////////////////////////////////////////////////////////
// GCNRegPressure

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `llvm::isEqual`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`llvm::isEqual`。

### Lines 41-67: Implements GCNRegPressure::getRegKind
```cpp
unsigned GCNRegPressure::getRegKind(const TargetRegisterClass *RC,
                                    const SIRegisterInfo *STI) {
  return STI->isSGPRClass(RC)
             ? SGPR
             : (STI->isAGPRClass(RC)
                    ? AGPR
                    : (STI->isVectorSuperClass(RC) ? AVGPR : VGPR));
}

void GCNRegPressure::inc(unsigned Reg,
                         LaneBitmask PrevMask,
                         LaneBitmask NewMask,
                         const MachineRegisterInfo &MRI) {
  unsigned NewNumCoveredRegs = SIRegisterInfo::getNumCoveredRegs(NewMask);
  unsigned PrevNumCoveredRegs = SIRegisterInfo::getNumCoveredRegs(PrevMask);
  if (NewNumCoveredRegs == PrevNumCoveredRegs)
    return;

  int Sign = 1;
  if (NewMask < PrevMask) {
    std::swap(NewMask, PrevMask);
    std::swap(NewNumCoveredRegs, PrevNumCoveredRegs);
    Sign = -1;
  }
  assert(PrevMask < NewMask && PrevNumCoveredRegs < NewNumCoveredRegs &&
         "prev mask should always be lesser than new");

```
**EN:** This section contains concrete logic for GCNRegPressure::getRegKind. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GCNRegPressure::getRegKind`, `GCNRegPressure::inc`, `SIRegisterInfo::getNumCoveredRegs`.
**CN:** 本节包含与 GCNRegPressure::getRegKind 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GCNRegPressure::getRegKind`, `GCNRegPressure::inc`, `SIRegisterInfo::getNumCoveredRegs`。

### Lines 68-110: Declares struct RegExcess
```cpp
  const TargetRegisterClass *RC = MRI.getRegClass(Reg);
  const TargetRegisterInfo *TRI = MRI.getTargetRegisterInfo();
  const SIRegisterInfo *STI = static_cast<const SIRegisterInfo *>(TRI);
  unsigned RegKind = getRegKind(RC, STI);
  if (TRI->getRegSizeInBits(*RC) != 32) {
    // Reg is from a tuple register class.
    if (PrevMask.none()) {
      unsigned TupleIdx = TOTAL_KINDS + RegKind;
      Value[TupleIdx] += Sign * TRI->getRegClassWeight(RC).RegWeight;
    }
    // Pressure scales with number of new registers covered by the new mask.
    // Note when true16 is enabled, we can no longer safely use the following
    // approach to calculate the difference in the number of 32-bit registers
    // between two masks:
    //
    // Sign *= SIRegisterInfo::getNumCoveredRegs(~PrevMask & NewMask);
    //
    // The issue is that the mask calculation `~PrevMask & NewMask` doesn't
    // properly account for partial usage of a 32-bit register when dealing with
    // 16-bit registers.
    //
    // Consider this example:
    // Assume PrevMask = 0b0010 and NewMask = 0b1111. Here, the correct register
    // usage difference should be 1, because even though PrevMask uses only half
    // of a 32-bit register, it should still be counted as a full register use.
    // However, the mask calculation yields `~PrevMask & NewMask = 0b1101`, and
    // calling `getNumCoveredRegs` returns 2 instead of 1. This incorrect
    // calculation can lead to integer overflow when Sign = -1.
    Sign *= NewNumCoveredRegs - PrevNumCoveredRegs;
  }
  Value[RegKind] += Sign;
}

namespace {
struct RegExcess {
  unsigned SGPR = 0;
  unsigned VGPR = 0;
  unsigned ArchVGPR = 0;
  unsigned AGPR = 0;

  bool anyExcess() const { return SGPR || VGPR || ArchVGPR || AGPR; }
  bool hasVectorRegisterExcess() const { return VGPR || ArchVGPR || AGPR; }

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `RegExcess`, `SIRegisterInfo::getNumCoveredRegs`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`RegExcess`, `SIRegisterInfo::getNumCoveredRegs`。

### Lines 111-151: Defines RegExcess
```cpp
  RegExcess(const MachineFunction &MF, const GCNRegPressure &RP)
      : RegExcess(MF, RP, GCNRPTarget(MF, RP)) {}
  RegExcess(const MachineFunction &MF, const GCNRegPressure &RP,
            const GCNRPTarget &Target) {
    unsigned MaxSGPRs = Target.getMaxSGPRs();
    unsigned MaxVGPRs = Target.getMaxVGPRs();

    const GCNSubtarget &ST = MF.getSubtarget<GCNSubtarget>();
    SGPR = std::max(static_cast<int>(RP.getSGPRNum() - MaxSGPRs), 0);

    // The number of virtual VGPRs required to handle excess SGPR
    unsigned WaveSize = ST.getWavefrontSize();
    unsigned VGPRForSGPRSpills = divideCeil(SGPR, WaveSize);

    unsigned MaxArchVGPRs = ST.getAddressableNumArchVGPRs();

    // Unified excess pressure conditions, accounting for VGPRs used for SGPR
    // spills
    VGPR = std::max(static_cast<int>(RP.getVGPRNum(ST.hasGFX90AInsts()) +
                                     VGPRForSGPRSpills - MaxVGPRs),
                    0);

    unsigned ArchVGPRLimit = ST.hasGFX90AInsts() ? MaxArchVGPRs : MaxVGPRs;
    // Arch VGPR excess pressure conditions, accounting for VGPRs used for SGPR
    // spills
    ArchVGPR = std::max(static_cast<int>(RP.getArchVGPRNum() +
                                         VGPRForSGPRSpills - ArchVGPRLimit),
                        0);

    // AGPR excess pressure conditions
    AGPR = std::max(static_cast<int>(RP.getAGPRNum() - ArchVGPRLimit), 0);
  }
};
} // namespace

bool GCNRegPressure::less(const MachineFunction &MF, const GCNRegPressure &O,
                          unsigned MaxOccupancy) const {
  const GCNSubtarget &ST = MF.getSubtarget<GCNSubtarget>();
  unsigned DynamicVGPRBlockSize =
      MF.getInfo<SIMachineFunctionInfo>()->getDynamicVGPRBlockSize();

```
**EN:** This section contains concrete logic for RegExcess. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `std::max`, `GCNRegPressure::less`.
**CN:** 本节包含与 RegExcess 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`std::max`, `GCNRegPressure::less`。

### Lines 152-191: Implements std::min
```cpp
  const auto SGPROcc = std::min(MaxOccupancy,
                                ST.getOccupancyWithNumSGPRs(getSGPRNum()));
  const auto VGPROcc = std::min(
      MaxOccupancy, ST.getOccupancyWithNumVGPRs(getVGPRNum(ST.hasGFX90AInsts()),
                                                DynamicVGPRBlockSize));
  const auto OtherSGPROcc = std::min(MaxOccupancy,
                                ST.getOccupancyWithNumSGPRs(O.getSGPRNum()));
  const auto OtherVGPROcc =
      std::min(MaxOccupancy,
               ST.getOccupancyWithNumVGPRs(O.getVGPRNum(ST.hasGFX90AInsts()),
                                           DynamicVGPRBlockSize));

  const auto Occ = std::min(SGPROcc, VGPROcc);
  const auto OtherOcc = std::min(OtherSGPROcc, OtherVGPROcc);

  // Give first precedence to the better occupancy.
  if (Occ != OtherOcc)
    return Occ > OtherOcc;

  unsigned MaxVGPRs = ST.getMaxNumVGPRs(MF);

  RegExcess Excess(MF, *this);
  RegExcess OtherExcess(MF, O);

  unsigned MaxArchVGPRs = ST.getAddressableNumArchVGPRs();

  bool ExcessRP = Excess.anyExcess();
  bool OtherExcessRP = OtherExcess.anyExcess();

  // Give second precedence to the reduced number of spills to hold the register
  // pressure.
  if (ExcessRP || OtherExcessRP) {
    // The difference in excess VGPR pressure, after including VGPRs used for
    // SGPR spills
    int VGPRDiff =
        ((OtherExcess.VGPR + OtherExcess.ArchVGPR + OtherExcess.AGPR) -
         (Excess.VGPR + Excess.ArchVGPR + Excess.AGPR));

    int SGPRDiff = OtherExcess.SGPR - Excess.SGPR;

```
**EN:** This section contains concrete logic for std::min. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `std::min`.
**CN:** 本节包含与 std::min 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`std::min`。

### Lines 192-223: Conditional logic and checks
```cpp
    if (VGPRDiff != 0)
      return VGPRDiff > 0;
    if (SGPRDiff != 0) {
      unsigned PureExcessVGPR =
          std::max(static_cast<int>(getVGPRNum(ST.hasGFX90AInsts()) - MaxVGPRs),
                   0) +
          std::max(static_cast<int>(getVGPRNum(false) - MaxArchVGPRs), 0);
      unsigned OtherPureExcessVGPR =
          std::max(
              static_cast<int>(O.getVGPRNum(ST.hasGFX90AInsts()) - MaxVGPRs),
              0) +
          std::max(static_cast<int>(O.getVGPRNum(false) - MaxArchVGPRs), 0);

      // If we have a special case where there is a tie in excess VGPR, but one
      // of the pressures has VGPR usage from SGPR spills, prefer the pressure
      // with SGPR spills.
      if (PureExcessVGPR != OtherPureExcessVGPR)
        return SGPRDiff < 0;
      // If both pressures have the same excess pressure before and after
      // accounting for SGPR spills, prefer fewer SGPR spills.
      return SGPRDiff > 0;
    }
  }

  bool SGPRImportant = SGPROcc < VGPROcc;
  const bool OtherSGPRImportant = OtherSGPROcc < OtherVGPROcc;

  // If both pressures disagree on what is more important compare vgprs.
  if (SGPRImportant != OtherSGPRImportant) {
    SGPRImportant = false;
  }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::max`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::max`。

### Lines 224-266: Conditional logic and checks
```cpp
  // Give third precedence to lower register tuple pressure.
  bool SGPRFirst = SGPRImportant;
  for (int I = 2; I > 0; --I, SGPRFirst = !SGPRFirst) {
    if (SGPRFirst) {
      auto SW = getSGPRTuplesWeight();
      auto OtherSW = O.getSGPRTuplesWeight();
      if (SW != OtherSW)
        return SW < OtherSW;
    } else {
      auto VW = getVGPRTuplesWeight();
      auto OtherVW = O.getVGPRTuplesWeight();
      if (VW != OtherVW)
        return VW < OtherVW;
    }
  }

  // Give final precedence to lower general RP.
  return SGPRImportant ? (getSGPRNum() < O.getSGPRNum()):
                         (getVGPRNum(ST.hasGFX90AInsts()) <
                          O.getVGPRNum(ST.hasGFX90AInsts()));
}

Printable llvm::print(const GCNRegPressure &RP, const GCNSubtarget *ST,
                      unsigned DynamicVGPRBlockSize) {
  return Printable([&RP, ST, DynamicVGPRBlockSize](raw_ostream &OS) {
    OS << "VGPRs: " << RP.getArchVGPRNum() << ' '
       << "AGPRs: " << RP.getAGPRNum();
    if (ST)
      OS << "(O"
         << ST->getOccupancyWithNumVGPRs(RP.getVGPRNum(ST->hasGFX90AInsts()),
                                         DynamicVGPRBlockSize)
         << ')';
    OS << ", SGPRs: " << RP.getSGPRNum();
    if (ST)
      OS << "(O" << ST->getOccupancyWithNumSGPRs(RP.getSGPRNum()) << ')';
    OS << ", LVGPR WT: " << RP.getVGPRTuplesWeight()
       << ", LSGPR WT: " << RP.getSGPRTuplesWeight();
    if (ST)
      OS << " -> Occ: " << RP.getOccupancy(*ST, DynamicVGPRBlockSize);
    OS << '\n';
  });
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `llvm::print`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`llvm::print`。

### Lines 267-310: Defines getDefRegMask
```cpp
static LaneBitmask getDefRegMask(const MachineOperand &MO,
                                 const MachineRegisterInfo &MRI) {
  assert(MO.isDef() && MO.isReg() && MO.getReg().isVirtual());

  // We don't rely on read-undef flag because in case of tentative schedule
  // tracking it isn't set correctly yet. This works correctly however since
  // use mask has been tracked before using LIS.
  return MO.getSubReg() == 0 ?
    MRI.getMaxLaneMaskForVReg(MO.getReg()) :
    MRI.getTargetRegisterInfo()->getSubRegIndexLaneMask(MO.getSubReg());
}

static void
collectVirtualRegUses(SmallVectorImpl<VRegMaskOrUnit> &VRegMaskOrUnits,
                      const MachineInstr &MI, const LiveIntervals &LIS,
                      const MachineRegisterInfo &MRI) {

  auto &TRI = *MRI.getTargetRegisterInfo();
  for (const auto &MO : MI.operands()) {
    if (!MO.isReg() || !MO.getReg().isVirtual())
      continue;
    if (!MO.isUse() || !MO.readsReg())
      continue;

    Register Reg = MO.getReg();
    auto I = llvm::find_if(VRegMaskOrUnits, [Reg](const VRegMaskOrUnit &RM) {
      return RM.VRegOrUnit.asVirtualReg() == Reg;
    });

    auto &P = I == VRegMaskOrUnits.end()
                  ? VRegMaskOrUnits.emplace_back(VirtRegOrUnit(Reg),
                                                 LaneBitmask::getNone())
                  : *I;

    P.LaneMask |= MO.getSubReg() ? TRI.getSubRegIndexLaneMask(MO.getSubReg())
                                 : MRI.getMaxLaneMaskForVReg(Reg);
  }

  SlotIndex InstrSI;
  for (auto &P : VRegMaskOrUnits) {
    auto &LI = LIS.getInterval(P.VRegOrUnit.asVirtualReg());
    if (!LI.hasSubRanges())
      continue;

```
**EN:** This section contains concrete logic for getDefRegMask. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `llvm::find_if`, `LaneBitmask::getNone`.
**CN:** 本节包含与 getDefRegMask 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`llvm::find_if`, `LaneBitmask::getNone`。

### Lines 311-341: Defines getLanesWithProperty
```cpp
    // For a tentative schedule LIS isn't updated yet but livemask should
    // remain the same on any schedule. Subreg defs can be reordered but they
    // all must dominate uses anyway.
    if (!InstrSI)
      InstrSI = LIS.getInstructionIndex(MI).getBaseIndex();

    P.LaneMask = getLiveLaneMask(LI, InstrSI, MRI, P.LaneMask);
  }
}

/// Mostly copy/paste from CodeGen/RegisterPressure.cpp
static LaneBitmask getLanesWithProperty(
    const LiveIntervals &LIS, const MachineRegisterInfo &MRI,
    bool TrackLaneMasks, Register Reg, SlotIndex Pos,
    function_ref<bool(const LiveRange &LR, SlotIndex Pos)> Property) {
  assert(Reg.isVirtual());
  const LiveInterval &LI = LIS.getInterval(Reg);
  LaneBitmask Result;
  if (TrackLaneMasks && LI.hasSubRanges()) {
    for (const LiveInterval::SubRange &SR : LI.subranges()) {
      if (Property(SR, Pos))
        Result |= SR.LaneMask;
    }
  } else if (Property(LI, Pos)) {
    Result =
        TrackLaneMasks ? MRI.getMaxLaneMaskForVReg(Reg) : LaneBitmask::getAll();
  }

  return Result;
}

```
**EN:** This section contains concrete logic for getLanesWithProperty. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `LaneBitmask::getAll`.
**CN:** 本节包含与 getLanesWithProperty 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`LaneBitmask::getAll`。

### Lines 342-380: Defines findUseBetween
```cpp
/// Mostly copy/paste from CodeGen/RegisterPressure.cpp
/// Helper to find a vreg use between two indices {PriorUseIdx, NextUseIdx}.
/// The query starts with a lane bitmask which gets lanes/bits removed for every
/// use we find.
static LaneBitmask findUseBetween(unsigned Reg, LaneBitmask LastUseMask,
                                  SlotIndex PriorUseIdx, SlotIndex NextUseIdx,
                                  const MachineRegisterInfo &MRI,
                                  const SIRegisterInfo *TRI,
                                  const LiveIntervals *LIS,
                                  bool Upward = false) {
  for (const MachineOperand &MO : MRI.use_nodbg_operands(Reg)) {
    if (MO.isUndef())
      continue;
    const MachineInstr *MI = MO.getParent();
    SlotIndex InstSlot = LIS->getInstructionIndex(*MI).getRegSlot();
    bool InRange = Upward ? (InstSlot > PriorUseIdx && InstSlot <= NextUseIdx)
                          : (InstSlot >= PriorUseIdx && InstSlot < NextUseIdx);
    if (!InRange)
      continue;

    unsigned SubRegIdx = MO.getSubReg();
    LaneBitmask UseMask = TRI->getSubRegIndexLaneMask(SubRegIdx);
    LastUseMask &= ~UseMask;
    if (LastUseMask.none())
      return LaneBitmask::getNone();
  }
  return LastUseMask;
}

////////////////////////////////////////////////////////////////////////////////
// GCNRPTarget

GCNRPTarget::GCNRPTarget(const MachineFunction &MF, const GCNRegPressure &RP)
    : GCNRPTarget(RP, MF) {
  const Function &F = MF.getFunction();
  const GCNSubtarget &ST = MF.getSubtarget<GCNSubtarget>();
  setTarget(ST.getMaxNumSGPRs(F), ST.getMaxNumVGPRs(F));
}

```
**EN:** This section contains concrete logic for findUseBetween. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `LaneBitmask::getNone`, `GCNRPTarget::GCNRPTarget`.
**CN:** 本节包含与 findUseBetween 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`LaneBitmask::getNone`, `GCNRPTarget::GCNRPTarget`。

### Lines 381-424: Implements GCNRPTarget::GCNRPTarget
```cpp
GCNRPTarget::GCNRPTarget(unsigned NumSGPRs, unsigned NumVGPRs,
                         const MachineFunction &MF, const GCNRegPressure &RP)
    : GCNRPTarget(RP, MF) {
  setTarget(NumSGPRs, NumVGPRs);
}

GCNRPTarget::GCNRPTarget(unsigned Occupancy, const MachineFunction &MF,
                         const GCNRegPressure &RP)
    : GCNRPTarget(RP, MF) {
  const GCNSubtarget &ST = MF.getSubtarget<GCNSubtarget>();
  unsigned DynamicVGPRBlockSize =
      MF.getInfo<SIMachineFunctionInfo>()->getDynamicVGPRBlockSize();
  setTarget(ST.getMaxNumSGPRs(Occupancy, /*Addressable=*/false),
            ST.getMaxNumVGPRs(Occupancy, DynamicVGPRBlockSize));
}

void GCNRPTarget::setTarget(unsigned NumSGPRs, unsigned NumVGPRs) {
  const GCNSubtarget &ST = MF.getSubtarget<GCNSubtarget>();
  MaxSGPRs = std::min(ST.getAddressableNumSGPRs(), NumSGPRs);
  MaxVGPRs = std::min(ST.getAddressableNumArchVGPRs(), NumVGPRs);
  if (UnifiedRF) {
    unsigned DynamicVGPRBlockSize =
        MF.getInfo<SIMachineFunctionInfo>()->getDynamicVGPRBlockSize();
    MaxUnifiedVGPRs =
        std::min(ST.getAddressableNumVGPRs(DynamicVGPRBlockSize), NumVGPRs);
  } else {
    MaxUnifiedVGPRs = 0;
  }
}

bool GCNRPTarget::isSaveBeneficial(Register Reg) const {
  const MachineRegisterInfo &MRI = MF.getRegInfo();
  const TargetRegisterClass *RC = MRI.getRegClass(Reg);
  const TargetRegisterInfo *TRI = MRI.getTargetRegisterInfo();
  const SIRegisterInfo *SRI = static_cast<const SIRegisterInfo *>(TRI);

  RegExcess Excess(MF, RP, *this);

  if (SRI->isSGPRClass(RC))
    return Excess.SGPR;

  if (SRI->isAGPRClass(RC))
    return (UnifiedRF && Excess.VGPR) || Excess.AGPR;

```
**EN:** This section contains concrete logic for GCNRPTarget::GCNRPTarget. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GCNRPTarget::GCNRPTarget`, `GCNRPTarget::setTarget`, `std::min`.
**CN:** 本节包含与 GCNRPTarget::GCNRPTarget 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GCNRPTarget::GCNRPTarget`, `GCNRPTarget::setTarget`, `std::min`。

### Lines 425-464: Implements GCNRPTarget::isSaveBeneficial
```cpp
  return (UnifiedRF && Excess.VGPR) || Excess.ArchVGPR;
}

bool GCNRPTarget::isSaveBeneficial(const GCNRegPressure &SaveRP) const {
  RegExcess Excess(MF, RP, *this);
  if (SaveRP.getSGPRNum() != 0 && Excess.SGPR != 0)
    return true;
  if (SaveRP.getArchVGPRNum() != 0 && Excess.ArchVGPR != 0)
    return true;
  if (SaveRP.getAGPRNum() != 0 && Excess.AGPR != 0)
    return true;
  if (UnifiedRF && Excess.VGPR != 0)
    return SaveRP.getArchVGPRNum() != 0 || SaveRP.getAGPRNum() != 0;
  return false;
}

unsigned GCNRPTarget::getNumRegsBenefit(const GCNRegPressure &SaveRP) const {
  RegExcess Excess(MF, RP, *this);
  const unsigned NumVGPRAboveAddrLimit =
      std::min(Excess.ArchVGPR, SaveRP.getArchVGPRNum()) +
      std::min(Excess.AGPR, SaveRP.getAGPRNum());
  unsigned NumRegsSaved =
      std::min(Excess.SGPR, SaveRP.getSGPRNum()) + NumVGPRAboveAddrLimit;

  if (UnifiedRF && Excess.VGPR) {
    // We have already accounted for excess pressure above addressive limits for
    // the individual VGPR classes. However for targets with unified RFs there
    // is also a unified VGPR pressure (ArchVGPR + AGPR combination) limit to
    // honor that may be more restrictive that the per-VGPR-class limits. We
    // must also be careful not to double-count VGPR saves that may contribute
    // to lowering pressure both above the addressable limit in their respective
    // class as well as in the unified VGPR limit.
    const unsigned VGPRSave = SaveRP.getArchVGPRNum() + SaveRP.getAGPRNum();
    if (NumVGPRAboveAddrLimit < VGPRSave)
      NumRegsSaved += std::min(Excess.VGPR, VGPRSave - NumVGPRAboveAddrLimit);
  }

  return NumRegsSaved;
}

```
**EN:** This section contains concrete logic for GCNRPTarget::isSaveBeneficial. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GCNRPTarget::isSaveBeneficial`, `GCNRPTarget::getNumRegsBenefit`, `std::min`.
**CN:** 本节包含与 GCNRPTarget::isSaveBeneficial 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GCNRPTarget::isSaveBeneficial`, `GCNRPTarget::getNumRegsBenefit`, `std::min`。

### Lines 465-504: Implements GCNRPTarget::satisfied
```cpp
bool GCNRPTarget::satisfied(const GCNRegPressure &TestRP) const {
  if (TestRP.getSGPRNum() > MaxSGPRs || TestRP.getVGPRNum(false) > MaxVGPRs)
    return false;
  if (UnifiedRF && TestRP.getVGPRNum(true) > MaxUnifiedVGPRs)
    return false;
  return true;
}

bool GCNRPTarget::hasVectorRegisterExcess() const {
  RegExcess Excess(MF, RP, *this);
  return Excess.hasVectorRegisterExcess();
}

///////////////////////////////////////////////////////////////////////////////
// GCNRPTracker

LaneBitmask llvm::getLiveLaneMask(unsigned Reg, SlotIndex SI,
                                  const LiveIntervals &LIS,
                                  const MachineRegisterInfo &MRI,
                                  LaneBitmask LaneMaskFilter) {
  return getLiveLaneMask(LIS.getInterval(Reg), SI, MRI, LaneMaskFilter);
}

LaneBitmask llvm::getLiveLaneMask(const LiveInterval &LI, SlotIndex SI,
                                  const MachineRegisterInfo &MRI,
                                  LaneBitmask LaneMaskFilter) {
  LaneBitmask LiveMask;
  if (LI.hasSubRanges()) {
    for (const auto &S : LI.subranges())
      if ((S.LaneMask & LaneMaskFilter).any() && S.liveAt(SI)) {
        LiveMask |= S.LaneMask;
        assert(LiveMask == (LiveMask & MRI.getMaxLaneMaskForVReg(LI.reg())));
      }
  } else if (LI.liveAt(SI)) {
    LiveMask = MRI.getMaxLaneMaskForVReg(LI.reg());
  }
  LiveMask &= LaneMaskFilter;
  return LiveMask;
}

```
**EN:** This section contains concrete logic for GCNRPTarget::satisfied. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GCNRPTarget::satisfied`, `GCNRPTarget::hasVectorRegisterExcess`, `llvm::getLiveLaneMask`.
**CN:** 本节包含与 GCNRPTarget::satisfied 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GCNRPTarget::satisfied`, `GCNRPTarget::hasVectorRegisterExcess`, `llvm::getLiveLaneMask`。

### Lines 505-533: Implements llvm::getLiveRegs
```cpp
GCNRPTracker::LiveRegSet llvm::getLiveRegs(SlotIndex SI,
                                           const LiveIntervals &LIS,
                                           const MachineRegisterInfo &MRI,
                                           GCNRegPressure::RegKind RegKind) {
  GCNRPTracker::LiveRegSet LiveRegs;
  for (unsigned I = 0, E = MRI.getNumVirtRegs(); I != E; ++I) {
    auto Reg = Register::index2VirtReg(I);
    if (RegKind != GCNRegPressure::TOTAL_KINDS &&
        GCNRegPressure::getRegKind(Reg, MRI) != RegKind)
      continue;
    if (!LIS.hasInterval(Reg))
      continue;
    auto LiveMask = getLiveLaneMask(Reg, SI, LIS, MRI);
    if (LiveMask.any())
      LiveRegs[Reg] = LiveMask;
  }
  return LiveRegs;
}

void GCNRPTracker::reset(const MachineInstr &MI, bool After) {
  const MachineRegisterInfo &MRI = MI.getMF()->getRegInfo();
  if (!MI.isDebugInstr()) {
    SlotIndex SI = LIS.getInstructionIndex(MI);
    if (After)
      SI = SI.getDeadSlot();
    reset(MRI, SI);
    return;
  }

```
**EN:** This section contains concrete logic for llvm::getLiveRegs. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `llvm::getLiveRegs`, `Register::index2VirtReg`, `GCNRegPressure::getRegKind`.
**CN:** 本节包含与 llvm::getLiveRegs 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`llvm::getLiveRegs`, `Register::index2VirtReg`, `GCNRegPressure::getRegKind`。

### Lines 534-573: Defines getIterator
```cpp
  // Look for the first valid index after the provided debug MI.
  MachineBasicBlock::const_iterator It = MI.getIterator(),
                                    MBBEnd = MI.getParent()->end();
  MachineBasicBlock::const_iterator NonDbgMI =
      skipDebugInstructionsForward(It, MBBEnd);
  if (NonDbgMI == MBBEnd) {
    // There are no non-debug instruction between MI and the end of the
    // block, so we reset the tracker at the end of the block.
    reset(*MI.getParent(), /*End=*/true);
    return;
  }
  // MI is a debug instruction so register pressure before or after it is
  // identical. Since we moved forward to finding a non-debug instruction
  // in the block, we reset the tracker before that instruction i.e., at its
  // base index.
  reset(MRI, LIS.getInstructionIndex(*NonDbgMI));
}

void GCNRPTracker::reset(const MachineBasicBlock &MBB, bool End) {
  SlotIndex SI = End ? LIS.getSlotIndexes()->getMBBLastIdx(&MBB)
                     : LIS.getMBBStartIdx(&MBB);
  reset(MBB.getParent()->getRegInfo(), SI);
}

void GCNRPTracker::reset(const MachineRegisterInfo &MRI, SlotIndex SI) {
  this->MRI = &MRI;
  LastTrackedMI = nullptr;
  LiveRegs = llvm::getLiveRegs(SI, LIS, MRI);
  MaxPressure = CurPressure = getRegPressure(MRI, LiveRegs);
}

void GCNRPTracker::reset(const MachineRegisterInfo &MRI,
                         const LiveRegSet &LiveRegs) {
  this->MRI = &MRI;
  LastTrackedMI = nullptr;
  if (&this->LiveRegs != &LiveRegs)
    this->LiveRegs = LiveRegs;
  MaxPressure = CurPressure = getRegPressure(MRI, LiveRegs);
}

```
**EN:** This section contains concrete logic for getIterator. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GCNRPTracker::reset`, `llvm::getLiveRegs`.
**CN:** 本节包含与 getIterator 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GCNRPTracker::reset`, `llvm::getLiveRegs`。

### Lines 574-615: Implements GCNRPTracker::getLastUsedLanes
```cpp
/// Mostly copy/paste from CodeGen/RegisterPressure.cpp
LaneBitmask GCNRPTracker::getLastUsedLanes(Register Reg, SlotIndex Pos) const {
  return getLanesWithProperty(
      LIS, *MRI, true, Reg, Pos.getBaseIndex(),
      [](const LiveRange &LR, SlotIndex Pos) {
        const LiveRange::Segment *S = LR.getSegmentContaining(Pos);
        return S != nullptr && S->end == Pos.getRegSlot();
      });
}

////////////////////////////////////////////////////////////////////////////////
// GCNUpwardRPTracker

void GCNUpwardRPTracker::recede(const MachineInstr &MI) {
  assert(MRI && "call reset first");

  LastTrackedMI = &MI;

  if (MI.isDebugInstr())
    return;

  // Kill all defs.
  GCNRegPressure DefPressure, ECDefPressure;
  bool HasECDefs = false;
  for (const MachineOperand &MO : MI.all_defs()) {
    if (!MO.getReg().isVirtual())
      continue;

    Register Reg = MO.getReg();
    LaneBitmask DefMask = getDefRegMask(MO, *MRI);

    // Treat a def as fully live at the moment of definition: keep a record.
    if (MO.isEarlyClobber()) {
      ECDefPressure.inc(Reg, LaneBitmask::getNone(), DefMask, *MRI);
      HasECDefs = true;
    } else
      DefPressure.inc(Reg, LaneBitmask::getNone(), DefMask, *MRI);

    auto I = LiveRegs.find(Reg);
    if (I == LiveRegs.end())
      continue;

```
**EN:** This section contains concrete logic for GCNRPTracker::getLastUsedLanes. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GCNRPTracker::getLastUsedLanes`, `GCNUpwardRPTracker::recede`, `LaneBitmask::getNone`.
**CN:** 本节包含与 GCNRPTracker::getLastUsedLanes 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GCNRPTracker::getLastUsedLanes`, `GCNUpwardRPTracker::recede`, `LaneBitmask::getNone`。

### Lines 616-658: Conditional logic and checks
```cpp
    LaneBitmask &LiveMask = I->second;
    LaneBitmask PrevMask = LiveMask;
    LiveMask &= ~DefMask;
    CurPressure.inc(Reg, PrevMask, LiveMask, *MRI);
    if (LiveMask.none())
      LiveRegs.erase(I);
  }

  // Update MaxPressure with defs pressure.
  DefPressure += CurPressure;
  if (HasECDefs)
    DefPressure += ECDefPressure;
  MaxPressure = max(DefPressure, MaxPressure);

  // Make uses alive.
  SmallVector<VRegMaskOrUnit, 8> RegUses;
  collectVirtualRegUses(RegUses, MI, LIS, *MRI);
  for (const VRegMaskOrUnit &U : RegUses) {
    LaneBitmask &LiveMask = LiveRegs[U.VRegOrUnit.asVirtualReg()];
    LaneBitmask PrevMask = LiveMask;
    LiveMask |= U.LaneMask;
    CurPressure.inc(U.VRegOrUnit.asVirtualReg(), PrevMask, LiveMask, *MRI);
  }

  // Update MaxPressure with uses plus early-clobber defs pressure.
  MaxPressure = HasECDefs ? max(CurPressure + ECDefPressure, MaxPressure)
                          : max(CurPressure, MaxPressure);

  assert(CurPressure == getRegPressure(*MRI, LiveRegs));
}

////////////////////////////////////////////////////////////////////////////////
// GCNDownwardRPTracker

bool GCNDownwardRPTracker::reset(const MachineInstr &MI,
                                 MachineBasicBlock::const_iterator End,
                                 const LiveRegSet *LiveRegsCopy) {
  MBBEnd = MI.getParent()->end();
  assert(End == MBBEnd ||
         End->getParent()->end() == MBBEnd && "end unrelated to MI block");
  NextMI = &MI;
  NextMI = skipDebugInstructionsForward(NextMI, End);

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `GCNDownwardRPTracker::reset`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`GCNDownwardRPTracker::reset`。

### Lines 659-693: Conditional logic and checks
```cpp
  // Do not use the MI to compute live registers when a set is provided.
  // Otherwise the first non-debug instruction after the provided one (or the
  // end of the block, if no such instruction exists) serves as the basis to
  // compute a live register set.
  if (LiveRegsCopy)
    GCNRPTracker::reset(MI.getMF()->getRegInfo(), *LiveRegsCopy);
  else if (NextMI != MBBEnd)
    GCNRPTracker::reset(*NextMI, /*After=*/false);
  else
    GCNRPTracker::reset(*MI.getParent(), /*End=*/true);
  return NextMI != End;
}

bool GCNDownwardRPTracker::advanceBeforeNext(MachineInstr *MI,
                                             bool UseInternalIterator) {
  assert(MRI && "call reset first");
  SlotIndex SI;
  const MachineInstr *CurrMI;
  if (UseInternalIterator) {
    if (!LastTrackedMI)
      return NextMI == MBBEnd;

    assert(NextMI == MBBEnd || !NextMI->isDebugInstr());
    CurrMI = LastTrackedMI;

    SI = NextMI == MBBEnd
             ? LIS.getInstructionIndex(*LastTrackedMI).getDeadSlot()
             : LIS.getInstructionIndex(*NextMI).getBaseIndex();
  } else { //! UseInternalIterator
    SI = LIS.getInstructionIndex(*MI).getBaseIndex();
    CurrMI = MI;
  }

  assert(SI.isValid());

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `GCNRPTracker::reset`, `GCNDownwardRPTracker::advanceBeforeNext`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`GCNRPTracker::reset`, `GCNDownwardRPTracker::advanceBeforeNext`。

### Lines 694-737: Conditional logic and checks
```cpp
  // Remove dead registers or mask bits.
  SmallSet<Register, 8> SeenRegs;
  for (auto &MO : CurrMI->operands()) {
    if (!MO.isReg() || !MO.getReg().isVirtual())
      continue;
    if (MO.isUse() && !MO.readsReg())
      continue;
    if (!UseInternalIterator && MO.isDef())
      continue;
    if (!SeenRegs.insert(MO.getReg()).second)
      continue;
    const LiveInterval &LI = LIS.getInterval(MO.getReg());
    if (LI.hasSubRanges()) {
      auto It = LiveRegs.end();
      for (const auto &S : LI.subranges()) {
        if (!S.liveAt(SI)) {
          if (It == LiveRegs.end()) {
            It = LiveRegs.find(MO.getReg());
            if (It == LiveRegs.end())
              llvm_unreachable("register isn't live");
          }
          auto PrevMask = It->second;
          It->second &= ~S.LaneMask;
          CurPressure.inc(MO.getReg(), PrevMask, It->second, *MRI);
        }
      }
      if (It != LiveRegs.end() && It->second.none())
        LiveRegs.erase(It);
    } else if (!LI.liveAt(SI)) {
      auto It = LiveRegs.find(MO.getReg());
      if (It == LiveRegs.end())
        llvm_unreachable("register isn't live");
      CurPressure.inc(MO.getReg(), It->second, LaneBitmask::getNone(), *MRI);
      LiveRegs.erase(It);
    }
  }

  MaxPressure = max(MaxPressure, CurPressure);

  LastTrackedMI = nullptr;

  return UseInternalIterator && (NextMI == MBBEnd);
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `LaneBitmask::getNone`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`LaneBitmask::getNone`。

### Lines 738-775: Implements GCNDownwardRPTracker::advanceToNext
```cpp
void GCNDownwardRPTracker::advanceToNext(MachineInstr *MI,
                                         bool UseInternalIterator) {
  if (UseInternalIterator) {
    LastTrackedMI = &*NextMI++;
    NextMI = skipDebugInstructionsForward(NextMI, MBBEnd);
  } else {
    LastTrackedMI = MI;
  }

  const MachineInstr *CurrMI = LastTrackedMI;

  // Add new registers or mask bits.
  for (const auto &MO : CurrMI->all_defs()) {
    Register Reg = MO.getReg();
    if (!Reg.isVirtual())
      continue;
    auto &LiveMask = LiveRegs[Reg];
    auto PrevMask = LiveMask;
    LiveMask |= getDefRegMask(MO, *MRI);
    CurPressure.inc(Reg, PrevMask, LiveMask, *MRI);
  }

  MaxPressure = max(MaxPressure, CurPressure);
}

bool GCNDownwardRPTracker::advance(MachineInstr *MI, bool UseInternalIterator) {
  if (UseInternalIterator && NextMI == MBBEnd)
    return false;

  advanceBeforeNext(MI, UseInternalIterator);
  advanceToNext(MI, UseInternalIterator);
  if (!UseInternalIterator) {
    // We must remove any dead def lanes from the current RP
    advanceBeforeNext(MI, true);
  }
  return true;
}

```
**EN:** This section contains concrete logic for GCNDownwardRPTracker::advanceToNext. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GCNDownwardRPTracker::advanceToNext`, `GCNDownwardRPTracker::advance`.
**CN:** 本节包含与 GCNDownwardRPTracker::advanceToNext 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GCNDownwardRPTracker::advanceToNext`, `GCNDownwardRPTracker::advance`。

### Lines 776-815: Implements GCNDownwardRPTracker::advance
```cpp
bool GCNDownwardRPTracker::advance(MachineBasicBlock::const_iterator End) {
  bool AnyAdvance = false;
  while (NextMI != End && advance())
    AnyAdvance = true;
  return AnyAdvance;
}

bool GCNDownwardRPTracker::advance(MachineBasicBlock::const_iterator Begin,
                                   MachineBasicBlock::const_iterator End,
                                   const LiveRegSet *LiveRegsCopy) {
  if (!reset(*Begin, End, LiveRegsCopy))
    return false;
  return advance(End);
}

Printable llvm::reportMismatch(const GCNRPTracker::LiveRegSet &LISLR,
                               const GCNRPTracker::LiveRegSet &TrackedLR,
                               const TargetRegisterInfo *TRI, StringRef Pfx) {
  return Printable([&LISLR, &TrackedLR, TRI, Pfx](raw_ostream &OS) {
    for (auto const &P : TrackedLR) {
      auto I = LISLR.find(P.first);
      if (I == LISLR.end()) {
        OS << Pfx << printReg(P.first, TRI) << ":L" << PrintLaneMask(P.second)
           << " isn't found in LIS reported set\n";
      } else if (I->second != P.second) {
        OS << Pfx << printReg(P.first, TRI)
           << " masks doesn't match: LIS reported " << PrintLaneMask(I->second)
           << ", tracked " << PrintLaneMask(P.second) << '\n';
      }
    }
    for (auto const &P : LISLR) {
      auto I = TrackedLR.find(P.first);
      if (I == TrackedLR.end()) {
        OS << Pfx << printReg(P.first, TRI) << ":L" << PrintLaneMask(P.second)
           << " isn't found in tracked set\n";
      }
    }
  });
}

```
**EN:** This section contains concrete logic for GCNDownwardRPTracker::advance. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GCNDownwardRPTracker::advance`, `llvm::reportMismatch`.
**CN:** 本节包含与 GCNDownwardRPTracker::advance 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GCNDownwardRPTracker::advance`, `llvm::reportMismatch`。

### Lines 816-856: Implements GCNDownwardRPTracker::bumpDownwardPressure
```cpp
GCNRegPressure
GCNDownwardRPTracker::bumpDownwardPressure(const MachineInstr *MI,
                                           const SIRegisterInfo *TRI) const {
  assert(!MI->isDebugOrPseudoInstr() && "Expect a nondebug instruction.");

  SlotIndex SlotIdx;
  SlotIdx = LIS.getInstructionIndex(*MI).getRegSlot();

  // Account for register pressure similar to RegPressureTracker::recede().
  RegisterOperands RegOpers;
  RegOpers.collect(*MI, *TRI, *MRI, true, /*IgnoreDead=*/false);
  RegOpers.adjustLaneLiveness(LIS, *MRI, SlotIdx);
  GCNRegPressure TempPressure = CurPressure;

  for (const VRegMaskOrUnit &Use : RegOpers.Uses) {
    if (!Use.VRegOrUnit.isVirtualReg())
      continue;
    Register Reg = Use.VRegOrUnit.asVirtualReg();
    LaneBitmask LastUseMask = getLastUsedLanes(Reg, SlotIdx);
    if (LastUseMask.none())
      continue;
    // The LastUseMask is queried from the liveness information of instruction
    // which may be further down the schedule. Some lanes may actually not be
    // last uses for the current position.
    // FIXME: allow the caller to pass in the list of vreg uses that remain
    // to be bottom-scheduled to avoid searching uses at each query.
    SlotIndex CurrIdx;
    const MachineBasicBlock *MBB = MI->getParent();
    MachineBasicBlock::const_iterator IdxPos = skipDebugInstructionsForward(
        LastTrackedMI ? LastTrackedMI : MBB->begin(), MBB->end());
    if (IdxPos == MBB->end()) {
      CurrIdx = LIS.getMBBEndIdx(MBB);
    } else {
      CurrIdx = LIS.getInstructionIndex(*IdxPos).getRegSlot();
    }

    LastUseMask =
        findUseBetween(Reg, LastUseMask, CurrIdx, SlotIdx, *MRI, TRI, &LIS);
    if (LastUseMask.none())
      continue;

```
**EN:** This section contains concrete logic for GCNDownwardRPTracker::bumpDownwardPressure. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GCNDownwardRPTracker::bumpDownwardPressure`, `RegPressureTracker::recede`.
**CN:** 本节包含与 GCNDownwardRPTracker::bumpDownwardPressure 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GCNDownwardRPTracker::bumpDownwardPressure`, `RegPressureTracker::recede`。

### Lines 857-898: Conditional logic and checks
```cpp
    auto It = LiveRegs.find(Reg);
    LaneBitmask LiveMask = It != LiveRegs.end() ? It->second : LaneBitmask(0);
    LaneBitmask NewMask = LiveMask & ~LastUseMask;
    TempPressure.inc(Reg, LiveMask, NewMask, *MRI);
  }

  // Generate liveness for defs.
  for (const VRegMaskOrUnit &Def : RegOpers.Defs) {
    if (!Def.VRegOrUnit.isVirtualReg())
      continue;
    Register Reg = Def.VRegOrUnit.asVirtualReg();
    auto It = LiveRegs.find(Reg);
    LaneBitmask LiveMask = It != LiveRegs.end() ? It->second : LaneBitmask(0);
    LaneBitmask NewMask = LiveMask | Def.LaneMask;
    TempPressure.inc(Reg, LiveMask, NewMask, *MRI);
  }

  return TempPressure;
}

bool GCNUpwardRPTracker::isValid() const {
  const auto &SI = LIS.getInstructionIndex(*LastTrackedMI).getBaseIndex();
  const auto LISLR = llvm::getLiveRegs(SI, LIS, *MRI);
  const auto &TrackedLR = LiveRegs;

  if (!isEqual(LISLR, TrackedLR)) {
    dbgs() << "\nGCNUpwardRPTracker error: Tracked and"
              " LIS reported livesets mismatch:\n"
           << print(LISLR, *MRI);
    reportMismatch(LISLR, TrackedLR, MRI->getTargetRegisterInfo());
    return false;
  }

  auto LISPressure = getRegPressure(*MRI, LISLR);
  if (LISPressure != CurPressure) {
    dbgs() << "GCNUpwardRPTracker error: Pressure sets different\nTracked: "
           << print(CurPressure) << "LIS rpt: " << print(LISPressure);
    return false;
  }
  return true;
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `GCNUpwardRPTracker::isValid`, `llvm::getLiveRegs`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`GCNUpwardRPTracker::isValid`, `llvm::getLiveRegs`。

### Lines 899-936: Registers LLVM passes
```cpp
Printable llvm::print(const GCNRPTracker::LiveRegSet &LiveRegs,
                      const MachineRegisterInfo &MRI) {
  return Printable([&LiveRegs, &MRI](raw_ostream &OS) {
    const TargetRegisterInfo *TRI = MRI.getTargetRegisterInfo();
    for (unsigned I = 0, E = MRI.getNumVirtRegs(); I != E; ++I) {
      Register Reg = Register::index2VirtReg(I);
      auto It = LiveRegs.find(Reg);
      if (It != LiveRegs.end() && It->second.any())
        OS << ' ' << printReg(Reg, TRI) << ':' << PrintLaneMask(It->second);
    }
    OS << '\n';
  });
}

void GCNRegPressure::dump() const { dbgs() << print(*this); }

static cl::opt<bool> UseDownwardTracker(
    "amdgpu-print-rp-downward",
    cl::desc("Use GCNDownwardRPTracker for GCNRegPressurePrinter pass"),
    cl::init(false), cl::Hidden);

char llvm::GCNRegPressurePrinter::ID = 0;
char &llvm::GCNRegPressurePrinterID = GCNRegPressurePrinter::ID;

INITIALIZE_PASS(GCNRegPressurePrinter, "amdgpu-print-rp", "", true, true)

// Return lanemask of Reg's subregs that are live-through at [Begin, End] and
// are fully covered by Mask.
static LaneBitmask
getRegLiveThroughMask(const MachineRegisterInfo &MRI, const LiveIntervals &LIS,
                      Register Reg, SlotIndex Begin, SlotIndex End,
                      LaneBitmask Mask = LaneBitmask::getAll()) {

  auto IsInOneSegment = [Begin, End](const LiveRange &LR) -> bool {
    auto *Segment = LR.getSegmentContaining(Begin);
    return Segment && Segment->contains(End);
  };

```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `llvm::print`, `Register::index2VirtReg`, `GCNRegPressure::dump`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`llvm::print`, `Register::index2VirtReg`, `GCNRegPressure::dump`。

### Lines 937-979: Preprocessor guards and macros
```cpp
  LaneBitmask LiveThroughMask;
  const LiveInterval &LI = LIS.getInterval(Reg);
  if (LI.hasSubRanges()) {
    for (auto &SR : LI.subranges()) {
      if ((SR.LaneMask & Mask) == SR.LaneMask && IsInOneSegment(SR))
        LiveThroughMask |= SR.LaneMask;
    }
  } else {
    LaneBitmask RegMask = MRI.getMaxLaneMaskForVReg(Reg);
    if ((RegMask & Mask) == RegMask && IsInOneSegment(LI))
      LiveThroughMask = RegMask;
  }

  return LiveThroughMask;
}

bool GCNRegPressurePrinter::runOnMachineFunction(MachineFunction &MF) {
  const MachineRegisterInfo &MRI = MF.getRegInfo();
  const TargetRegisterInfo *TRI = MRI.getTargetRegisterInfo();
  const LiveIntervals &LIS = getAnalysis<LiveIntervalsWrapperPass>().getLIS();

  auto &OS = dbgs();

// Leading spaces are important for YAML syntax.
#define PFX "  "

  OS << "---\nname: " << MF.getName() << "\nbody:             |\n";

  auto printRP = [](const GCNRegPressure &RP) {
    return Printable([&RP](raw_ostream &OS) {
      OS << format(PFX "  %-5d", RP.getSGPRNum())
         << format(" %-5d", RP.getVGPRNum(false));
    });
  };

  auto ReportLISMismatchIfAny = [&](const GCNRPTracker::LiveRegSet &TrackedLR,
                                    const GCNRPTracker::LiveRegSet &LISLR) {
    if (LISLR != TrackedLR) {
      OS << PFX "  mis LIS: " << llvm::print(LISLR, MRI)
         << reportMismatch(LISLR, TrackedLR, TRI, PFX "    ");
    }
  };

```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `GCNRegPressurePrinter::runOnMachineFunction`, `llvm::print`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`GCNRegPressurePrinter::runOnMachineFunction`, `llvm::print`。

### Lines 980-1022: Conditional logic and checks
```cpp
  // Register pressure before and at an instruction (in program order).
  SmallVector<std::pair<GCNRegPressure, GCNRegPressure>, 16> RP;

  for (auto &MBB : MF) {
    RP.clear();
    RP.reserve(MBB.size());

    OS << PFX;
    MBB.printName(OS);
    OS << ":\n";

    SlotIndex MBBStartSlot = LIS.getSlotIndexes()->getMBBStartIdx(&MBB);
    SlotIndex MBBLastSlot = LIS.getSlotIndexes()->getMBBLastIdx(&MBB);

    GCNRPTracker::LiveRegSet LiveIn, LiveOut;
    GCNRegPressure RPAtMBBEnd;

    if (UseDownwardTracker) {
      if (MBB.empty()) {
        LiveIn = LiveOut = getLiveRegs(MBBStartSlot, LIS, MRI);
        RPAtMBBEnd = getRegPressure(MRI, LiveIn);
      } else {
        GCNDownwardRPTracker RPT(LIS);
        RPT.reset(MBB.front(), MBB.end());

        LiveIn = RPT.getLiveRegs();

        while (!RPT.advanceBeforeNext()) {
          GCNRegPressure RPBeforeMI = RPT.getPressure();
          RPT.advanceToNext();
          RP.emplace_back(RPBeforeMI, RPT.getPressure());
        }

        LiveOut = RPT.getLiveRegs();
        RPAtMBBEnd = RPT.getPressure();
      }
    } else {
      GCNUpwardRPTracker RPT(LIS);
      RPT.reset(MRI, MBBLastSlot);

      LiveOut = RPT.getLiveRegs();
      RPAtMBBEnd = RPT.getPressure();

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 1023-1054: Conditional logic and checks
```cpp
      for (auto &MI : reverse(MBB)) {
        RPT.resetMaxPressure();
        RPT.recede(MI);
        if (!MI.isDebugInstr())
          RP.emplace_back(RPT.getPressure(), RPT.getMaxPressure());
      }

      LiveIn = RPT.getLiveRegs();
    }

    OS << PFX "  Live-in: " << llvm::print(LiveIn, MRI);
    if (!UseDownwardTracker)
      ReportLISMismatchIfAny(LiveIn, getLiveRegs(MBBStartSlot, LIS, MRI));

    OS << PFX "  SGPR  VGPR\n";
    int I = 0;
    for (auto &MI : MBB) {
      if (!MI.isDebugInstr()) {
        auto &[RPBeforeInstr, RPAtInstr] =
            RP[UseDownwardTracker ? I : (RP.size() - 1 - I)];
        ++I;
        OS << printRP(RPBeforeInstr) << '\n' << printRP(RPAtInstr) << "  ";
      } else
        OS << PFX "               ";
      MI.print(OS);
    }
    OS << printRP(RPAtMBBEnd) << '\n';

    OS << PFX "  Live-out:" << llvm::print(LiveOut, MRI);
    if (UseDownwardTracker)
      ReportLISMismatchIfAny(LiveOut, getLiveRegs(MBBLastSlot, LIS, MRI));

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `llvm::print`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`llvm::print`。

### Lines 1055-1084: Preprocessor guards and macros
```cpp
    GCNRPTracker::LiveRegSet LiveThrough;
    for (auto [Reg, Mask] : LiveIn) {
      LaneBitmask MaskIntersection = Mask & LiveOut.lookup(Reg);
      if (MaskIntersection.any()) {
        LaneBitmask LTMask = getRegLiveThroughMask(
            MRI, LIS, Reg, MBBStartSlot, MBBLastSlot, MaskIntersection);
        if (LTMask.any())
          LiveThrough[Reg] = LTMask;
      }
    }
    OS << PFX "  Live-thr:" << llvm::print(LiveThrough, MRI);
    OS << printRP(getRegPressure(MRI, LiveThrough)) << '\n';
  }
  OS << "...\n";
  return false;

#undef PFX
}

#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
LLVM_DUMP_METHOD void llvm::dumpMaxRegPressure(MachineFunction &MF,
                                               GCNRegPressure::RegKind Kind,
                                               LiveIntervals &LIS,
                                               const MachineLoopInfo *MLI) {

  const MachineRegisterInfo &MRI = MF.getRegInfo();
  const TargetRegisterInfo *TRI = MRI.getTargetRegisterInfo();
  auto &OS = dbgs();
  const char *RegName = GCNRegPressure::getName(Kind);

```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `llvm::print`, `llvm::dumpMaxRegPressure`, `GCNRegPressure::getName`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`llvm::print`, `llvm::dumpMaxRegPressure`, `GCNRegPressure::getName`。

### Lines 1085-1115: Declares RPT
```cpp
  unsigned MaxNumRegs = 0;
  const MachineInstr *MaxPressureMI = nullptr;
  GCNUpwardRPTracker RPT(LIS);
  for (const MachineBasicBlock &MBB : MF) {
    RPT.reset(MRI, LIS.getSlotIndexes()->getMBBEndIdx(&MBB).getPrevSlot());
    for (const MachineInstr &MI : reverse(MBB)) {
      RPT.recede(MI);
      unsigned NumRegs = RPT.getMaxPressure().getNumRegs(Kind);
      if (NumRegs > MaxNumRegs) {
        MaxNumRegs = NumRegs;
        MaxPressureMI = &MI;
      }
    }
  }

  SlotIndex MISlot = LIS.getInstructionIndex(*MaxPressureMI);

  // Max pressure can occur at either the early-clobber or register slot.
  // Choose the maximum liveset between both slots. This is ugly but this is
  // diagnostic code.
  SlotIndex ECSlot = MISlot.getRegSlot(true);
  SlotIndex RSlot = MISlot.getRegSlot(false);
  GCNRPTracker::LiveRegSet ECLiveSet = getLiveRegs(ECSlot, LIS, MRI, Kind);
  GCNRPTracker::LiveRegSet RLiveSet = getLiveRegs(RSlot, LIS, MRI, Kind);
  unsigned ECNumRegs = getRegPressure(MRI, ECLiveSet).getNumRegs(Kind);
  unsigned RNumRegs = getRegPressure(MRI, RLiveSet).getNumRegs(Kind);
  GCNRPTracker::LiveRegSet *LiveSet =
      ECNumRegs > RNumRegs ? &ECLiveSet : &RLiveSet;
  SlotIndex MaxPressureSlot = ECNumRegs > RNumRegs ? ECSlot : RSlot;
  assert(getRegPressure(MRI, *LiveSet).getNumRegs(Kind) == MaxNumRegs);

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 1116-1155: Conditional logic and checks
```cpp
  // Split live registers into single-def and multi-def sets.
  GCNRegPressure SDefPressure, MDefPressure;
  SmallVector<Register, 16> SDefRegs, MDefRegs;
  for (auto [Reg, LaneMask] : *LiveSet) {
    assert(GCNRegPressure::getRegKind(Reg, MRI) == Kind);
    LiveInterval &LI = LIS.getInterval(Reg);
    if (LI.getNumValNums() == 1 ||
        (LI.hasSubRanges() &&
         llvm::all_of(LI.subranges(), [](const LiveInterval::SubRange &SR) {
           return SR.getNumValNums() == 1;
         }))) {
      SDefPressure.inc(Reg, LaneBitmask::getNone(), LaneMask, MRI);
      SDefRegs.push_back(Reg);
    } else {
      MDefPressure.inc(Reg, LaneBitmask::getNone(), LaneMask, MRI);
      MDefRegs.push_back(Reg);
    }
  }
  unsigned SDefNumRegs = SDefPressure.getNumRegs(Kind);
  unsigned MDefNumRegs = MDefPressure.getNumRegs(Kind);
  assert(SDefNumRegs + MDefNumRegs == MaxNumRegs);

  auto printLoc = [&](const MachineBasicBlock *MBB, SlotIndex SI) {
    return Printable([&, MBB, SI](raw_ostream &OS) {
      OS << SI << ':' << printMBBReference(*MBB);
      if (MLI)
        if (const MachineLoop *ML = MLI->getLoopFor(MBB))
          OS << " (LoopHdr " << printMBBReference(*ML->getHeader())
             << ", Depth " << ML->getLoopDepth() << ")";
    });
  };

  auto PrintRegInfo = [&](Register Reg, LaneBitmask LiveMask) {
    GCNRegPressure RegPressure;
    RegPressure.inc(Reg, LaneBitmask::getNone(), LiveMask, MRI);
    OS << "  " << printReg(Reg, TRI) << ':'
       << TRI->getRegClassName(MRI.getRegClass(Reg)) << ", LiveMask "
       << PrintLaneMask(LiveMask) << " (" << RegPressure.getNumRegs(Kind) << ' '
       << RegName << "s)\n";

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `GCNRegPressure::getRegKind`, `llvm::all_of`, `LaneBitmask::getNone`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`GCNRegPressure::getRegKind`, `llvm::all_of`, `LaneBitmask::getNone`。

### Lines 1156-1197: Preprocessor guards and macros
```cpp
    // Use std::map to sort def/uses by SlotIndex.
    std::map<SlotIndex, const MachineInstr *> Instrs;
    for (const MachineInstr &MI : MRI.reg_nodbg_instructions(Reg)) {
      Instrs[LIS.getInstructionIndex(MI).getRegSlot()] = &MI;
    }

    for (const auto &[SI, MI] : Instrs) {
      OS << "    ";
      if (MI->definesRegister(Reg, TRI))
        OS << "def ";
      if (MI->readsRegister(Reg, TRI))
        OS << "use ";
      OS << printLoc(MI->getParent(), SI) << ": " << *MI;
    }
  };

  OS << "\n*** Register pressure info (" << RegName << "s) for " << MF.getName()
     << " ***\n";
  OS << "Max pressure is " << MaxNumRegs << ' ' << RegName << "s at "
     << printLoc(MaxPressureMI->getParent(), MaxPressureSlot) << ": "
     << *MaxPressureMI;

  OS << "\nLive registers with single definition (" << SDefNumRegs << ' '
     << RegName << "s):\n";

  // Sort SDefRegs by number of uses (smallest first)
  llvm::sort(SDefRegs, [&](Register A, Register B) {
    return std::distance(MRI.use_nodbg_begin(A), MRI.use_nodbg_end()) <
           std::distance(MRI.use_nodbg_begin(B), MRI.use_nodbg_end());
  });

  for (const Register Reg : SDefRegs) {
    PrintRegInfo(Reg, LiveSet->lookup(Reg));
  }

  OS << "\nLive registers with multiple definitions (" << MDefNumRegs << ' '
     << RegName << "s):\n";
  for (const Register Reg : MDefRegs) {
    PrintRegInfo(Reg, LiveSet->lookup(Reg));
  }
}
#endif
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `llvm::sort`, `std::distance`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`llvm::sort`, `std::distance`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `RegExcess`, `llvm::isEqual`, `GCNRegPressure::getRegKind`, `GCNRegPressure::inc`, `SIRegisterInfo::getNumCoveredRegs`, `std::swap`
- **Main themes / 核心主题**: register management / 寄存器管理; subtarget modeling / 子目标建模; scheduling / 调度; machine-function state / MachineFunction 状态
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"GCNRegPressure.h"`
- `"AMDGPU.h"`
- `"SIMachineFunctionInfo.h"`
- `"llvm/CodeGen/MachineBasicBlock.h"`
- `"llvm/CodeGen/MachineLoopInfo.h"`
- `"llvm/CodeGen/RegisterPressure.h"`
