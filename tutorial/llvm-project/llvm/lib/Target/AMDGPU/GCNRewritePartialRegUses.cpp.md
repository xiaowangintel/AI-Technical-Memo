# GCNRewritePartialRegUses.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/GCNRewritePartialRegUses.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements GCNRewritePartialRegUses for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 GCNRewritePartialRegUses 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: File banner, license, and overview
```cpp
//===-------------- GCNRewritePartialRegUses.cpp --------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
/// \file
/// RenameIndependentSubregs pass leaves large partially used super registers,
/// for example:
///   undef %0.sub4:VReg_1024 = ...
///   %0.sub5:VReg_1024 = ...
///   %0.sub6:VReg_1024 = ...
///   %0.sub7:VReg_1024 = ...
///   use %0.sub4_sub5_sub6_sub7
///   use %0.sub6_sub7
///
/// GCNRewritePartialRegUses goes right after RenameIndependentSubregs and
/// rewrites such partially used super registers with registers of minimal size:
///   undef %0.sub0:VReg_128 = ...
///   %0.sub1:VReg_128 = ...
///   %0.sub2:VReg_128 = ...
///   %0.sub3:VReg_128 = ...
///   use %0.sub0_sub1_sub2_sub3
///   use %0.sub2_sub3
///
/// This allows to avoid subreg lanemasks tracking during register pressure
/// calculation and creates more possibilities for the code unaware of lanemasks
//===----------------------------------------------------------------------===//

```
**EN:** This opening section establishes the file identity, licensing terms, and high-level intent before the backend-specific code begins.
**CN:** 开头部分先给出文件标识、许可证信息以及总体意图，为后续后端专用代码建立上下文。

### Lines 31-61: Header dependencies and setup
```cpp
#include "GCNRewritePartialRegUses.h"
#include "AMDGPU.h"
#include "MCTargetDesc/AMDGPUMCTargetDesc.h"
#include "SIRegisterInfo.h"
#include "llvm/CodeGen/LiveInterval.h"
#include "llvm/CodeGen/LiveIntervals.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/TargetInstrInfo.h"
#include "llvm/Pass.h"

using namespace llvm;

#define DEBUG_TYPE "rewrite-partial-reg-uses"

namespace {

class GCNRewritePartialRegUsesImpl {
  MachineRegisterInfo *MRI;
  const SIRegisterInfo *TRI;
  const TargetInstrInfo *TII;
  LiveIntervals *LIS;

  /// Rewrite partially used register Reg by shifting all its subregisters to
  /// the right and replacing the original register with a register of minimal
  /// size. Return true if the change has been made.
  bool rewriteReg(Register Reg) const;

  /// Map OldSubReg -> NewSubReg. Used as in/out container.
  using SubRegMap = SmallDenseMap<unsigned, unsigned>;

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available. Main symbols: `GCNRewritePartialRegUsesImpl`.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。 主要符号：`GCNRewritePartialRegUsesImpl`。

### Lines 62-93: Defines getMinSizeReg
```cpp
  /// Given register class RC and the set of used subregs as keys in the SubRegs
  /// map return new register class and indexes of right-shifted subregs as
  /// values in SubRegs map such that the resulting regclass would contain
  /// registers of minimal size.
  const TargetRegisterClass *getMinSizeReg(const TargetRegisterClass *RC,
                                           SubRegMap &SubRegs) const;

  /// Given regclass RC and pairs of [OldSubReg, NewSubReg] in SubRegs try to
  /// find new regclass such that:
  ///   1. It has subregs obtained by shifting each OldSubReg by RShift number
  ///      of bits to the right. Every "shifted" subreg should have the same
  ///      SubRegRC. If CoverSubregIdx is not zero it's a subreg that "covers"
  ///      all other subregs in pairs. Basically such subreg becomes a whole
  ///      register.
  ///   2. Resulting register class contains registers of minimal size.
  ///
  /// SubRegs is map of OldSubReg -> NewSubReg and is used as in/out
  /// parameter:
  ///   OldSubReg - input parameter,
  ///   NewSubReg - output, contains shifted subregs on return.
  const TargetRegisterClass *
  getRegClassWithShiftedSubregs(const TargetRegisterClass *RC, unsigned RShift,
                                unsigned CoverSubregIdx,
                                SubRegMap &SubRegs) const;

  /// Update live intervals after rewriting OldReg to NewReg with SubRegs map
  /// describing OldSubReg -> NewSubReg mapping.
  void updateLiveIntervals(Register OldReg, Register NewReg,
                           SubRegMap &SubRegs) const;

  /// Helper methods.

```
**EN:** This section contains concrete logic for getMinSizeReg. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 getMinSizeReg 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 94-124: Declares shiftSubReg
```cpp
  /// Find right-shifted by RShift amount version of the SubReg if it exists,
  /// return 0 otherwise.
  unsigned shiftSubReg(unsigned SubReg, unsigned RShift) const;

  /// Find subreg index with a given Offset and Size, return 0 if there is no
  /// such subregister index. The result is cached in SubRegs data-member.
  unsigned getSubReg(unsigned Offset, unsigned Size) const;

  /// Cache for getSubReg method: {Offset, Size} -> SubReg index.
  mutable SmallDenseMap<std::pair<unsigned, unsigned>, unsigned> SubRegs;

  /// Return bit mask that contains all register classes that are projected into
  /// RC by SubRegIdx. The result is cached in SuperRegMasks data-member.
  const uint32_t *getSuperRegClassMask(const TargetRegisterClass *RC,
                                       unsigned SubRegIdx) const;

  /// Cache for getSuperRegClassMask method: { RC, SubRegIdx } -> Class bitmask.
  mutable SmallDenseMap<std::pair<const TargetRegisterClass *, unsigned>,
                        const uint32_t *>
      SuperRegMasks;

  /// Return bitmask containing all allocatable register classes with registers
  /// aligned at AlignNumBits. The result is cached in
  /// AllocatableAndAlignedRegClassMasks data-member.
  const BitVector &
  getAllocatableAndAlignedRegClassMask(unsigned AlignNumBits) const;

  /// Cache for getAllocatableAndAlignedRegClassMask method:
  ///   AlignNumBits -> Class bitmask.
  mutable SmallDenseMap<unsigned, BitVector> AllocatableAndAlignedRegClassMasks;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 125-150: Declares class GCNRewritePartialRegUsesLegacy
```cpp
public:
  GCNRewritePartialRegUsesImpl(LiveIntervals *LS) : LIS(LS) {}
  bool run(MachineFunction &MF);
};

class GCNRewritePartialRegUsesLegacy : public MachineFunctionPass {
public:
  static char ID;
  GCNRewritePartialRegUsesLegacy() : MachineFunctionPass(ID) {}

  StringRef getPassName() const override {
    return "Rewrite Partial Register Uses";
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    AU.addPreserved<LiveIntervalsWrapperPass>();
    AU.addPreserved<SlotIndexesWrapperPass>();
    MachineFunctionPass::getAnalysisUsage(AU);
  }

  bool runOnMachineFunction(MachineFunction &MF) override;
};

} // end anonymous namespace

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `GCNRewritePartialRegUsesLegacy`, `MachineFunctionPass::getAnalysisUsage`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`GCNRewritePartialRegUsesLegacy`, `MachineFunctionPass::getAnalysisUsage`。

### Lines 151-172: Implements GCNRewritePartialRegUsesImpl::getSubReg
```cpp
// TODO: move this to the tablegen and use binary search by Offset.
unsigned GCNRewritePartialRegUsesImpl::getSubReg(unsigned Offset,
                                                 unsigned Size) const {
  const auto [I, Inserted] = SubRegs.try_emplace({Offset, Size}, 0);
  if (Inserted) {
    for (unsigned Idx = 1, E = TRI->getNumSubRegIndices(); Idx < E; ++Idx) {
      if (TRI->getSubRegIdxOffset(Idx) == Offset &&
          TRI->getSubRegIdxSize(Idx) == Size) {
        I->second = Idx;
        break;
      }
    }
  }
  return I->second;
}

unsigned GCNRewritePartialRegUsesImpl::shiftSubReg(unsigned SubReg,
                                                   unsigned RShift) const {
  unsigned Offset = TRI->getSubRegIdxOffset(SubReg) - RShift;
  return getSubReg(Offset, TRI->getSubRegIdxSize(SubReg));
}

```
**EN:** This section contains concrete logic for GCNRewritePartialRegUsesImpl::getSubReg. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GCNRewritePartialRegUsesImpl::getSubReg`, `GCNRewritePartialRegUsesImpl::shiftSubReg`.
**CN:** 本节包含与 GCNRewritePartialRegUsesImpl::getSubReg 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GCNRewritePartialRegUsesImpl::getSubReg`, `GCNRewritePartialRegUsesImpl::shiftSubReg`。

### Lines 173-204: Implements GCNRewritePartialRegUsesImpl::getSuperRegClassMask
```cpp
const uint32_t *GCNRewritePartialRegUsesImpl::getSuperRegClassMask(
    const TargetRegisterClass *RC, unsigned SubRegIdx) const {
  const auto [I, Inserted] =
      SuperRegMasks.try_emplace({RC, SubRegIdx}, nullptr);
  if (Inserted) {
    for (SuperRegClassIterator RCI(RC, TRI); RCI.isValid(); ++RCI) {
      if (RCI.getSubReg() == SubRegIdx) {
        I->second = RCI.getMask();
        break;
      }
    }
  }
  return I->second;
}

const BitVector &
GCNRewritePartialRegUsesImpl::getAllocatableAndAlignedRegClassMask(
    unsigned AlignNumBits) const {
  const auto [I, Inserted] =
      AllocatableAndAlignedRegClassMasks.try_emplace(AlignNumBits);
  if (Inserted) {
    BitVector &BV = I->second;
    BV.resize(TRI->getNumRegClasses());
    for (unsigned ClassID = 0; ClassID < TRI->getNumRegClasses(); ++ClassID) {
      auto *RC = TRI->getRegClass(ClassID);
      if (RC->isAllocatable() && TRI->isRegClassAligned(RC, AlignNumBits))
        BV.set(ClassID);
    }
  }
  return I->second;
}

```
**EN:** This section contains concrete logic for GCNRewritePartialRegUsesImpl::getSuperRegClassMask. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GCNRewritePartialRegUsesImpl::getSuperRegClassMask`, `GCNRewritePartialRegUsesImpl::getAllocatableAndAlignedRegClassMask`.
**CN:** 本节包含与 GCNRewritePartialRegUsesImpl::getSuperRegClassMask 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GCNRewritePartialRegUsesImpl::getSuperRegClassMask`, `GCNRewritePartialRegUsesImpl::getAllocatableAndAlignedRegClassMask`。

### Lines 205-226: Implements GCNRewritePartialRegUsesImpl::getRegClassWithShiftedSubregs
```cpp
const TargetRegisterClass *
GCNRewritePartialRegUsesImpl::getRegClassWithShiftedSubregs(
    const TargetRegisterClass *RC, unsigned RShift, unsigned CoverSubregIdx,
    SubRegMap &SubRegs) const {

  unsigned RCAlign = TRI->getRegClassAlignmentNumBits(RC);
  LLVM_DEBUG(dbgs() << "  Shift " << RShift << ", reg align " << RCAlign
                    << '\n');

  BitVector ClassMask(getAllocatableAndAlignedRegClassMask(RCAlign));
  for (auto &[OldSubReg, NewSubReg] : SubRegs) {
    LLVM_DEBUG(dbgs() << "  " << TRI->getSubRegIndexName(OldSubReg) << ':');

    auto *SubRegRC = TRI->getSubRegisterClass(RC, OldSubReg);
    if (!SubRegRC) {
      LLVM_DEBUG(dbgs() << "couldn't find target regclass\n");
      return nullptr;
    }
    LLVM_DEBUG(dbgs() << TRI->getRegClassName(SubRegRC)
                      << (SubRegRC->isAllocatable() ? "" : " not alloc")
                      << " -> ");

```
**EN:** This section contains concrete logic for GCNRewritePartialRegUsesImpl::getRegClassWithShiftedSubregs. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GCNRewritePartialRegUsesImpl::getRegClassWithShiftedSubregs`.
**CN:** 本节包含与 GCNRewritePartialRegUsesImpl::getRegClassWithShiftedSubregs 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GCNRewritePartialRegUsesImpl::getRegClassWithShiftedSubregs`。

### Lines 227-251: Conditional logic and checks
```cpp
    if (OldSubReg == CoverSubregIdx) {
      // Covering subreg will become a full register, RC should be allocatable.
      assert(SubRegRC->isAllocatable());
      NewSubReg = AMDGPU::NoSubRegister;
      LLVM_DEBUG(dbgs() << "whole reg");
    } else {
      NewSubReg = shiftSubReg(OldSubReg, RShift);
      if (!NewSubReg) {
        LLVM_DEBUG(dbgs() << "none\n");
        return nullptr;
      }
      LLVM_DEBUG(dbgs() << TRI->getSubRegIndexName(NewSubReg));
    }

    const uint32_t *Mask = NewSubReg ? getSuperRegClassMask(SubRegRC, NewSubReg)
                                     : SubRegRC->getSubClassMask();
    if (!Mask)
      llvm_unreachable("no register class mask?");

    ClassMask.clearBitsNotInMask(Mask);
    // Don't try to early exit because checking if ClassMask has set bits isn't
    // that cheap and we expect it to pass in most cases.
    LLVM_DEBUG(dbgs() << ", num regclasses " << ClassMask.count() << '\n');
  }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 252-278: Preprocessor guards and macros
```cpp
  // ClassMask is the set of all register classes such that each class is
  // allocatable, aligned, has all shifted subregs and each subreg has required
  // register class (see SubRegRC above). Now select first (that is largest)
  // register class with registers of minimal size.
  const TargetRegisterClass *MinRC = nullptr;
  unsigned MinNumBits = std::numeric_limits<unsigned>::max();
  for (unsigned ClassID : ClassMask.set_bits()) {
    auto *RC = TRI->getRegClass(ClassID);
    unsigned NumBits = TRI->getRegSizeInBits(*RC);
    if (NumBits < MinNumBits) {
      MinNumBits = NumBits;
      MinRC = RC;
    }
  }
#ifndef NDEBUG
  if (MinRC) {
    assert(MinRC->isAllocatable() && TRI->isRegClassAligned(MinRC, RCAlign));
    for (auto [OldSubReg, NewSubReg] : SubRegs)
      // Check that all registers in MinRC support NewSubReg subregister.
      assert(TRI->isSubRegValidForRegClass(MinRC, NewSubReg));
  }
#endif
  // There might be zero RShift - in this case we just trying to find smaller
  // register.
  return (MinRC != RC || RShift != 0) ? MinRC : nullptr;
}

```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

### Lines 279-310: Implements GCNRewritePartialRegUsesImpl::getMinSizeReg
```cpp
const TargetRegisterClass *
GCNRewritePartialRegUsesImpl::getMinSizeReg(const TargetRegisterClass *RC,
                                            SubRegMap &SubRegs) const {
  unsigned CoverSubreg = AMDGPU::NoSubRegister;
  unsigned Offset = std::numeric_limits<unsigned>::max();
  unsigned End = 0;
  for (auto [SubReg, SRI] : SubRegs) {
    unsigned SubRegOffset = TRI->getSubRegIdxOffset(SubReg);
    unsigned SubRegEnd = SubRegOffset + TRI->getSubRegIdxSize(SubReg);
    if (SubRegOffset < Offset) {
      Offset = SubRegOffset;
      CoverSubreg = AMDGPU::NoSubRegister;
    }
    if (SubRegEnd > End) {
      End = SubRegEnd;
      CoverSubreg = AMDGPU::NoSubRegister;
    }
    if (SubRegOffset == Offset && SubRegEnd == End)
      CoverSubreg = SubReg;
  }
  // If covering subreg is found shift everything so the covering subreg would
  // be in the rightmost position.
  if (CoverSubreg != AMDGPU::NoSubRegister)
    return getRegClassWithShiftedSubregs(RC, Offset, CoverSubreg, SubRegs);

  // Otherwise find subreg with maximum required alignment and shift it and all
  // other subregs to the rightmost possible position with respect to the
  // alignment.
  unsigned MaxAlign = 0;
  for (auto [SubReg, SRI] : SubRegs)
    MaxAlign = std::max(MaxAlign, TRI->getSubRegAlignmentNumBits(RC, SubReg));

```
**EN:** This section contains concrete logic for GCNRewritePartialRegUsesImpl::getMinSizeReg. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GCNRewritePartialRegUsesImpl::getMinSizeReg`, `std::max`.
**CN:** 本节包含与 GCNRewritePartialRegUsesImpl::getMinSizeReg 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GCNRewritePartialRegUsesImpl::getMinSizeReg`, `std::max`。

### Lines 311-343: Declares max
```cpp
  unsigned FirstMaxAlignedSubRegOffset = std::numeric_limits<unsigned>::max();
  for (auto [SubReg, SRI] : SubRegs) {
    if (TRI->getSubRegAlignmentNumBits(RC, SubReg) != MaxAlign)
      continue;
    FirstMaxAlignedSubRegOffset =
        std::min(FirstMaxAlignedSubRegOffset, TRI->getSubRegIdxOffset(SubReg));
    if (FirstMaxAlignedSubRegOffset == Offset)
      break;
  }

  unsigned NewOffsetOfMaxAlignedSubReg =
      alignTo(FirstMaxAlignedSubRegOffset - Offset, MaxAlign);

  if (NewOffsetOfMaxAlignedSubReg > FirstMaxAlignedSubRegOffset)
    llvm_unreachable("misaligned subreg");

  unsigned RShift = FirstMaxAlignedSubRegOffset - NewOffsetOfMaxAlignedSubReg;
  return getRegClassWithShiftedSubregs(RC, RShift, 0, SubRegs);
}

// Only the subrange's lanemasks of the original interval need to be modified.
// Subrange for a covering subreg becomes the main range.
void GCNRewritePartialRegUsesImpl::updateLiveIntervals(
    Register OldReg, Register NewReg, SubRegMap &SubRegs) const {
  if (!LIS->hasInterval(OldReg))
    return;

  auto &OldLI = LIS->getInterval(OldReg);
  auto &NewLI = LIS->createEmptyInterval(NewReg);

  auto &Allocator = LIS->getVNInfoAllocator();
  NewLI.setWeight(OldLI.weight());

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `std::min`, `GCNRewritePartialRegUsesImpl::updateLiveIntervals`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`std::min`, `GCNRewritePartialRegUsesImpl::updateLiveIntervals`。

### Lines 344-373: Conditional logic and checks
```cpp
  for (auto &SR : OldLI.subranges()) {
    auto I = find_if(SubRegs, [&](auto &P) {
      return SR.LaneMask == TRI->getSubRegIndexLaneMask(P.first);
    });

    if (I == SubRegs.end()) {
      // There might be a situation when subranges don't exactly match used
      // subregs, for example:
      // %120 [160r,1392r:0) 0@160r
      //    L000000000000C000 [160r,1392r:0) 0@160r
      //    L0000000000003000 [160r,1392r:0) 0@160r
      //    L0000000000000C00 [160r,1392r:0) 0@160r
      //    L0000000000000300 [160r,1392r:0) 0@160r
      //    L0000000000000003 [160r,1104r:0) 0@160r
      //    L000000000000000C [160r,1104r:0) 0@160r
      //    L0000000000000030 [160r,1104r:0) 0@160r
      //    L00000000000000C0 [160r,1104r:0) 0@160r
      // but used subregs are:
      //    sub0_sub1_sub2_sub3_sub4_sub5_sub6_sub7, L000000000000FFFF
      //    sub0_sub1_sub2_sub3, L00000000000000FF
      //    sub4_sub5_sub6_sub7, L000000000000FF00
      // In this example subregs sub0_sub1_sub2_sub3 and sub4_sub5_sub6_sub7
      // have several subranges with the same lifetime. For such cases just
      // recreate the interval.
      LIS->removeInterval(OldReg);
      LIS->removeInterval(NewReg);
      LIS->createAndComputeVirtRegInterval(NewReg);
      return;
    }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 374-404: Declares assert
```cpp
    if (unsigned NewSubReg = I->second)
      NewLI.createSubRangeFrom(Allocator,
                               TRI->getSubRegIndexLaneMask(NewSubReg), SR);
    else // This is the covering subreg (0 index) - set it as main range.
      NewLI.assign(SR, Allocator);

    SubRegs.erase(I);
  }
  if (NewLI.empty())
    NewLI.assign(OldLI, Allocator);
  assert(NewLI.verify(MRI));
  LIS->removeInterval(OldReg);
}

bool GCNRewritePartialRegUsesImpl::rewriteReg(Register Reg) const {

  // Collect used subregs.
  SubRegMap SubRegs;
  for (MachineOperand &MO : MRI->reg_nodbg_operands(Reg)) {
    if (MO.getSubReg() == AMDGPU::NoSubRegister)
      return false; // Whole reg used.
    SubRegs.try_emplace(MO.getSubReg());
  }

  if (SubRegs.empty())
    return false;

  auto *RC = MRI->getRegClass(Reg);
  LLVM_DEBUG(dbgs() << "Try to rewrite partial reg " << printReg(Reg, TRI)
                    << ':' << TRI->getRegClassName(RC) << '\n');

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `GCNRewritePartialRegUsesImpl::rewriteReg`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`GCNRewritePartialRegUsesImpl::rewriteReg`。

### Lines 405-434: Declares getMinSizeReg
```cpp
  auto *NewRC = getMinSizeReg(RC, SubRegs);
  if (!NewRC) {
    LLVM_DEBUG(dbgs() << "  No improvement achieved\n");
    return false;
  }

  Register NewReg = MRI->createVirtualRegister(NewRC);
  LLVM_DEBUG(dbgs() << "  Success " << printReg(Reg, TRI) << ':'
                    << TRI->getRegClassName(RC) << " -> "
                    << printReg(NewReg, TRI) << ':'
                    << TRI->getRegClassName(NewRC) << '\n');

  for (auto &MO : make_early_inc_range(MRI->reg_operands(Reg))) {
    MO.setReg(NewReg);
    // Debug info can refer to the whole reg, just leave it as it is for now.
    // TODO: create some DI shift expression?
    if (MO.isDebug() && MO.getSubReg() == 0)
      continue;
    unsigned NewSubReg = SubRegs[MO.getSubReg()];
    MO.setSubReg(NewSubReg);
    if (NewSubReg == AMDGPU::NoSubRegister && MO.isDef())
      MO.setIsUndef(false);
  }

  if (LIS)
    updateLiveIntervals(Reg, NewReg, SubRegs);

  return true;
}

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 435-467: Implements GCNRewritePartialRegUsesImpl::run
```cpp
bool GCNRewritePartialRegUsesImpl::run(MachineFunction &MF) {
  MRI = &MF.getRegInfo();
  TRI = static_cast<const SIRegisterInfo *>(MRI->getTargetRegisterInfo());
  TII = MF.getSubtarget().getInstrInfo();
  bool Changed = false;
  for (size_t I = 0, E = MRI->getNumVirtRegs(); I < E; ++I) {
    Changed |= rewriteReg(Register::index2VirtReg(I));
  }
  return Changed;
}

bool GCNRewritePartialRegUsesLegacy::runOnMachineFunction(MachineFunction &MF) {
  LiveIntervalsWrapperPass *LISWrapper =
      getAnalysisIfAvailable<LiveIntervalsWrapperPass>();
  LiveIntervals *LIS = LISWrapper ? &LISWrapper->getLIS() : nullptr;
  GCNRewritePartialRegUsesImpl Impl(LIS);
  return Impl.run(MF);
}

PreservedAnalyses
GCNRewritePartialRegUsesPass::run(MachineFunction &MF,
                                  MachineFunctionAnalysisManager &MFAM) {
  auto *LIS = MFAM.getCachedResult<LiveIntervalsAnalysis>(MF);
  if (!GCNRewritePartialRegUsesImpl(LIS).run(MF))
    return PreservedAnalyses::all();

  auto PA = getMachineFunctionPassPreservedAnalyses();
  PA.preserveSet<CFGAnalyses>();
  PA.preserve<LiveIntervalsAnalysis>();
  PA.preserve<SlotIndexesAnalysis>();
  return PA;
}

```
**EN:** This section contains concrete logic for GCNRewritePartialRegUsesImpl::run. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GCNRewritePartialRegUsesImpl::run`, `Register::index2VirtReg`, `GCNRewritePartialRegUsesLegacy::runOnMachineFunction`.
**CN:** 本节包含与 GCNRewritePartialRegUsesImpl::run 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GCNRewritePartialRegUsesImpl::run`, `Register::index2VirtReg`, `GCNRewritePartialRegUsesLegacy::runOnMachineFunction`。

### Lines 468-475: Registers LLVM passes
```cpp
char GCNRewritePartialRegUsesLegacy::ID;

char &llvm::GCNRewritePartialRegUsesID = GCNRewritePartialRegUsesLegacy::ID;

INITIALIZE_PASS_BEGIN(GCNRewritePartialRegUsesLegacy, DEBUG_TYPE,
                      "Rewrite Partial Register Uses", false, false)
INITIALIZE_PASS_END(GCNRewritePartialRegUsesLegacy, DEBUG_TYPE,
                    "Rewrite Partial Register Uses", false, false)
```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `GCNRewritePartialRegUsesImpl`, `GCNRewritePartialRegUsesLegacy`, `MachineFunctionPass::getAnalysisUsage`, `GCNRewritePartialRegUsesImpl::getSubReg`, `GCNRewritePartialRegUsesImpl::shiftSubReg`, `GCNRewritePartialRegUsesImpl::getSuperRegClassMask`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; LLVM pass integration / LLVM Pass 集成; machine-function state / MachineFunction 状态
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"GCNRewritePartialRegUses.h"`
- `"AMDGPU.h"`
- `"MCTargetDesc/AMDGPUMCTargetDesc.h"`
- `"SIRegisterInfo.h"`
- `"llvm/CodeGen/LiveInterval.h"`
- `"llvm/CodeGen/LiveIntervals.h"`
- `"llvm/CodeGen/MachineFunctionPass.h"`
- `"llvm/CodeGen/MachineRegisterInfo.h"`
- `"llvm/CodeGen/TargetInstrInfo.h"`
- `"llvm/Pass.h"`
