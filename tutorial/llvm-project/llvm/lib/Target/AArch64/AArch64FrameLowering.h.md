# AArch64FrameLowering.h — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/AArch64FrameLowering.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This file covers TargetFrameLowering for AArch64. / 该文件实现 AArch64 后端中的栈帧降低。
## Line-by-Line Analysis / 逐行分析
### Lines 1-31: Documented code section
```cpp
//==-- AArch64FrameLowering.h - TargetFrameLowering for AArch64 --*- C++ -*-==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
//
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AARCH64_AARCH64FRAMELOWERING_H
#define LLVM_LIB_TARGET_AARCH64_AARCH64FRAMELOWERING_H

#include "llvm/CodeGen/MachineOptimizationRemarkEmitter.h"
#include "llvm/CodeGen/TargetFrameLowering.h"
#include "llvm/Support/TypeSize.h"

namespace llvm {

class TargetLowering;
class AArch64FunctionInfo;
class AArch64InstrInfo;
class AArch64PrologueEmitter;
class AArch64EpilogueEmitter;

struct SVEStackSizes {
  uint64_t ZPRStackSize{0};
  uint64_t PPRStackSize{0};
};
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 32-62: Class AArch64FrameLowering
```cpp

class AArch64FrameLowering : public TargetFrameLowering {
public:
  explicit AArch64FrameLowering()
      : TargetFrameLowering(StackGrowsDown, Align(16), 0, Align(16),
                            true /*StackRealignable*/) {}

  void resetCFIToInitialState(MachineBasicBlock &MBB) const override;

  MachineBasicBlock::iterator
  eliminateCallFramePseudoInstr(MachineFunction &MF, MachineBasicBlock &MBB,
                                MachineBasicBlock::iterator I) const override;

  /// emitProlog/emitEpilog - These methods insert prolog and epilog code into
  /// the function.
  void emitPrologue(MachineFunction &MF, MachineBasicBlock &MBB) const override;
  void emitEpilogue(MachineFunction &MF, MachineBasicBlock &MBB) const override;

  /// Harden the entire function with pac-ret.
  ///
  /// If pac-ret+leaf is requested, we want to harden as much code as possible.
  /// This function inserts pac-ret hardening at the points where prologue and
  /// epilogue are traditionally inserted, ignoring possible shrink-wrapping
  /// optimization.
  void emitPacRetPlusLeafHardening(MachineFunction &MF) const;

  bool enableCFIFixup(const MachineFunction &MF) const override;

  bool enableFullCFIFixup(const MachineFunction &MF) const override;

  bool canUseAsPrologue(const MachineBasicBlock &MBB) const override;
```
**EN:** This block defines AArch64FrameLowering, packaging state and behavior that the file reuses for stack frame lowering.  
**CN:** 该代码块定义 AArch64FrameLowering，把 栈帧降低 所需的状态与行为封装在一起供后续复用。
### Lines 63-95: Core AArch64 backend logic
```cpp

  StackOffset getFrameIndexReference(const MachineFunction &MF, int FI,
                                     Register &FrameReg) const override;
  StackOffset getFrameIndexReferenceFromSP(const MachineFunction &MF,
                                           int FI) const override;
  StackOffset resolveFrameIndexReference(const MachineFunction &MF, int FI,
                                         Register &FrameReg, bool PreferFP,
                                         bool ForSimm) const;
  StackOffset resolveFrameOffsetReference(const MachineFunction &MF,
                                          int64_t ObjectOffset, bool isFixed,
                                          TargetStackID::Value StackID,
                                          Register &FrameReg, bool PreferFP,
                                          bool ForSimm) const;
  bool spillCalleeSavedRegisters(MachineBasicBlock &MBB,
                                 MachineBasicBlock::iterator MI,
                                 ArrayRef<CalleeSavedInfo> CSI,
                                 const TargetRegisterInfo *TRI) const override;

  bool
  restoreCalleeSavedRegisters(MachineBasicBlock &MBB,
                              MachineBasicBlock::iterator MI,
                              MutableArrayRef<CalleeSavedInfo> CSI,
                              const TargetRegisterInfo *TRI) const override;

  /// Can this function use the red zone for local allocations.
  bool canUseRedZone(const MachineFunction &MF) const;

  bool hasReservedCallFrame(const MachineFunction &MF) const override;

  bool
  assignCalleeSavedSpillSlots(MachineFunction &MF,
                              const TargetRegisterInfo *TRI,
                              std::vector<CalleeSavedInfo> &CSI) const override;
```
**EN:** This block continues the file's main stack frame lowering logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的栈帧降低主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 96-125: Function determineCalleeSaves
```cpp

  void determineCalleeSaves(MachineFunction &MF, BitVector &SavedRegs,
                            RegScavenger *RS) const override;

  /// Returns true if the target will correctly handle shrink wrapping.
  bool enableShrinkWrapping(const MachineFunction &MF) const override {
    return true;
  }

  bool enableStackSlotScavenging(const MachineFunction &MF) const override;
  TargetStackID::Value getStackIDForScalableVectors() const override;

  void processFunctionBeforeFrameFinalized(MachineFunction &MF,
                                           RegScavenger *RS) const override;

  void
  processFunctionBeforeFrameIndicesReplaced(MachineFunction &MF,
                                            RegScavenger *RS) const override;

  unsigned getWinEHParentFrameOffset(const MachineFunction &MF) const override;

  unsigned getWinEHFuncletFrameSize(const MachineFunction &MF) const;

  StackOffset
  getFrameIndexReferencePreferSP(const MachineFunction &MF, int FI,
                                 Register &FrameReg,
                                 bool IgnoreSPUpdates) const override;
  StackOffset getNonLocalFrameIndexReference(const MachineFunction &MF,
                                             int FI) const override;
  int getSEHFrameIndexOffset(const MachineFunction &MF, int FI) const;
```
**EN:** This block implements determineCalleeSaves, advancing the file's stack frame lowering flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 determineCalleeSaves，通过 AArch64 专用的决策与数据处理推进本文件的栈帧降低流程。
### Lines 126-157: Function isSupportedStackID
```cpp

  bool isSupportedStackID(TargetStackID::Value ID) const override {
    switch (ID) {
    default:
      return false;
    case TargetStackID::Default:
    case TargetStackID::ScalableVector:
    case TargetStackID::ScalablePredicateVector:
    case TargetStackID::NoAlloc:
      return true;
    }
  }

  bool isStackIdSafeForLocalArea(unsigned StackId) const override {
    // We don't support putting SVE objects into the pre-allocated local
    // frame block at the moment.
    return (StackId != TargetStackID::ScalableVector &&
            StackId != TargetStackID::ScalablePredicateVector);
  }

  void
  orderFrameObjects(const MachineFunction &MF,
                    SmallVectorImpl<int> &ObjectsToAllocate) const override;

  bool isFPReserved(const MachineFunction &MF) const;

  bool needsWinCFI(const MachineFunction &MF) const;

  bool requiresSaveVG(const MachineFunction &MF) const;

  /// Returns the size of the entire ZPR stackframe (calleesaves + spills).
  StackOffset getZPRStackSize(const MachineFunction &MF) const;
```
**EN:** This block implements isSupportedStackID, advancing the file's stack frame lowering flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 isSupportedStackID，通过 AArch64 专用的决策与数据处理推进本文件的栈帧降低流程。
### Lines 158-187: Function stackframe
```cpp

  /// Returns the size of the entire PPR stackframe (calleesaves + spills +
  /// hazard padding).
  StackOffset getPPRStackSize(const MachineFunction &MF) const;

  /// Returns the size of the entire SVE stackframe (PPRs + ZPRs).
  StackOffset getSVEStackSize(const MachineFunction &MF) const {
    return getZPRStackSize(MF) + getPPRStackSize(MF);
  }

  friend class AArch64PrologueEpilogueCommon;
  friend class AArch64PrologueEmitter;
  friend class AArch64EpilogueEmitter;

  // Windows unwind can't represent the required stack adjustments if we have
  // both SVE callee-saves and dynamic stack allocations, and the frame
  // pointer is before the SVE spills.  The allocation of the frame pointer
  // must be the last instruction in the prologue so the unwinder can restore
  // the stack pointer correctly. (And there isn't any unwind opcode for
  // `addvl sp, x29, -17`.)
  //
  // Because of this, we do spills in the opposite order on Windows: first SVE,
  // then GPRs. The main side-effect of this is that it makes accessing
  // parameters passed on the stack more expensive.
  //
  // We could consider rearranging the spills for simpler cases.
  bool hasSVECalleeSavesAboveFrameRecord(const MachineFunction &MF) const;

protected:
  bool hasFPImpl(const MachineFunction &MF) const override;
```
**EN:** This block implements stackframe, advancing the file's stack frame lowering flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 stackframe，通过 AArch64 专用的决策与数据处理推进本文件的栈帧降低流程。
### Lines 188-220: Core AArch64 backend logic
```cpp

private:
  /// Returns true if a homogeneous prolog or epilog code can be emitted
  /// for the size optimization. If so, HOM_Prolog/HOM_Epilog pseudo
  /// instructions are emitted in place. When Exit block is given, this check is
  /// for epilog.
  bool homogeneousPrologEpilog(MachineFunction &MF,
                               MachineBasicBlock *Exit = nullptr) const;

  /// Returns true if CSRs should be paired.
  bool producePairRegisters(MachineFunction &MF) const;

  /// Make a determination whether a Hazard slot is used and create it if
  /// needed.
  void determineStackHazardSlot(MachineFunction &MF,
                                BitVector &SavedRegs) const;

  /// Emit target zero call-used regs.
  void emitZeroCallUsedRegs(BitVector RegsToZero,
                            MachineBasicBlock &MBB) const override;

  /// Replace a StackProbe stub (if any) with the actual probe code inline
  void inlineStackProbe(MachineFunction &MF,
                        MachineBasicBlock &PrologueMBB) const override;

  void inlineStackProbeFixed(MachineBasicBlock::iterator MBBI,
                             Register ScratchReg, int64_t FrameSize,
                             StackOffset CFAOffset) const;

  MachineBasicBlock::iterator
  inlineStackProbeLoopExactMultiple(MachineBasicBlock::iterator MBBI,
                                    int64_t NegProbeSize,
                                    Register TargetReg) const;
```
**EN:** This block continues the file's main stack frame lowering logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的栈帧降低主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 221-249: Core AArch64 backend logic
```cpp

  void emitRemarks(const MachineFunction &MF,
                   MachineOptimizationRemarkEmitter *ORE) const override;

  bool windowsRequiresStackProbe(const MachineFunction &MF,
                                 uint64_t StackSizeInBytes) const;

  bool shouldSignReturnAddressEverywhere(const MachineFunction &MF) const;

  StackOffset getFPOffset(const MachineFunction &MF,
                          int64_t ObjectOffset) const;

  StackOffset getStackOffset(const MachineFunction &MF,
                             int64_t ObjectOffset) const;

  // Given a load or a store instruction, generate an appropriate unwinding SEH
  // code on Windows.
  MachineBasicBlock::iterator insertSEH(MachineBasicBlock::iterator MBBI,
                                        const AArch64InstrInfo &TII,
                                        MachineInstr::MIFlag Flag) const;

  /// Returns how much of the incoming argument stack area (in bytes) we should
  /// clean up in an epilogue. For the C calling convention this will be 0, for
  /// guaranteed tail call conventions it can be positive (a normal return or a
  /// tail call to a function that uses less stack space for arguments) or
  /// negative (for a tail call to a function that needs more stack space than
  /// us for arguments).
  int64_t getArgumentStackToRestore(MachineFunction &MF,
                                    MachineBasicBlock &MBB) const;
```
**EN:** This block continues the file's main stack frame lowering logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的栈帧降低主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 250-274: Documented code section
```cpp

  // Find a scratch register that we can use at the start of the prologue to
  // re-align the stack pointer.  We avoid using callee-save registers since
  // they may appear to be free when this is called from canUseAsPrologue
  // (during shrink wrapping), but then no longer be free when this is called
  // from emitPrologue.
  //
  // FIXME: This is a bit conservative, since in the above case we could use one
  // of the callee-save registers as a scratch temp to re-align the stack
  // pointer, but we would then have to make sure that we were in fact saving at
  // least one callee-save register in the prologue, which is additional
  // complexity that doesn't seem worth the benefit.
  Register findScratchNonCalleeSaveRegister(MachineBasicBlock *MBB,
                                            bool HasCall = false) const;

  /// Returns the size of the fixed object area (allocated next to sp on entry)
  /// On Win64 this may include a var args area and an UnwindHelp object for EH.
  unsigned getFixedObjectSize(const MachineFunction &MF,
                              const AArch64FunctionInfo *AFI, bool IsWin64,
                              bool IsFunclet) const;
};

} // End llvm namespace

#endif
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
- **EN:** Prologue/epilogue and stack-frame construction **CN:** 序言/尾声与栈帧构造
## Dependencies / 依赖关系
- **EN:** Core LLVM interfaces: llvm/CodeGen/MachineOptimizationRemarkEmitter.h, llvm/CodeGen/TargetFrameLowering.h, llvm/Support/TypeSize.h **CN:** 核心 LLVM 接口：llvm/CodeGen/MachineOptimizationRemarkEmitter.h, llvm/CodeGen/TargetFrameLowering.h, llvm/Support/TypeSize.h
- **EN:** Closely connected with neighboring AArch64 backend components responsible for stack frame lowering. **CN:** 与周边负责栈帧降低的 AArch64 后端组件紧密协作。
