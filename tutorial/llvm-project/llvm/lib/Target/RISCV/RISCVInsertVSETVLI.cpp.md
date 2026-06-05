# RISCVInsertVSETVLI.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInsertVSETVLI.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements insertion and maintenance of vsetvli/vsetivli vector-configuration instructions. / 实现插入并维护 vsetvli/vsetivli 向量配置指令。

## Line-by-Line Analysis / 逐行分析
### Lines 1-25: Commentary and design intent / 注释与设计意图
```cpp
//===- RISCVInsertVSETVLI.cpp - Insert VSETVLI instructions ---------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements a function pass that inserts VSETVLI instructions where
// needed and expands the vl outputs of VLEFF/VLSEGFF to PseudoReadVL
// instructions.
//
// This pass consists of 3 phases:
//
// Phase 1 collects how each basic block affects VL/VTYPE.
//
// Phase 2 uses the information from phase 1 to do a data flow analysis to
// propagate the VL/VTYPE changes through the function. This gives us the
// VL/VTYPE at the start of each basic block.
//
// Phase 3 inserts VSETVLI instructions in each basic block. Information from
// phase 2 is used to prevent inserting a VSETVLI before the first vector
// instruction in the block if possible.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 26-39: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp

#include "RISCV.h"
#include "RISCVSubtarget.h"
#include "RISCVVSETVLIInfoAnalysis.h"
#include "llvm/ADT/PostOrderIterator.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/CodeGen/LiveDebugVariables.h"
#include "llvm/CodeGen/LiveIntervals.h"
#include "llvm/CodeGen/LiveStacks.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include <queue>
using namespace llvm;
using namespace RISCV;
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 40-53: Header guard and interface framing / 头文件保护与接口框架
```cpp
#define DEBUG_TYPE "riscv-insert-vsetvli"
#define RISCV_INSERT_VSETVLI_NAME "RISC-V Insert VSETVLI pass"

STATISTIC(NumInsertedVSETVL, "Number of VSETVL inst inserted");
STATISTIC(NumCoalescedVSETVL, "Number of VSETVL inst coalesced");

static cl::opt<bool> EnsureWholeVectorRegisterMoveValidVTYPE(
    DEBUG_TYPE "-whole-vector-register-move-valid-vtype", cl::Hidden,
    cl::desc("Insert vsetvlis before vmvNr.vs to ensure vtype is valid and "
             "vill is cleared"),
    cl::init(true));

namespace {
```
**EN:** This block establishes include guards and the outer structure of the public interface.

**CN:** 该区段建立头文件保护并给出公共接口的外层结构。

### Lines 54-69: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
/// Given a virtual register \p Reg, return the corresponding VNInfo for it.
/// This will return nullptr if the virtual register is an implicit_def or
/// if LiveIntervals is not available.
static VNInfo *getVNInfoFromReg(Register Reg, const MachineInstr &MI,
                                const LiveIntervals *LIS) {
  assert(Reg.isVirtual());
  if (!LIS)
    return nullptr;
  auto &LI = LIS->getInterval(Reg);
  SlotIndex SI = LIS->getSlotIndexes()->getInstructionIndex(MI);
  return LI.getVNInfoBefore(SI);
}

static MachineOperand &getVLOp(MachineInstr &MI) {
  return MI.getOperand(RISCVII::getVLOpNum(MI.getDesc()));
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 70-84: Type declaration for BlockData / BlockData 的类型声明
```cpp

struct BlockData {
  // The VSETVLIInfo that represents the VL/VTYPE settings on exit from this
  // block. Calculated in Phase 2.
  VSETVLIInfo Exit;

  // The VSETVLIInfo that represents the VL/VTYPE settings from all predecessor
  // blocks. Calculated in Phase 2, and used by Phase 3.
  VSETVLIInfo Pred;

  // Keeps track of whether the block is already in the queue.
  bool InQueue = false;

  BlockData() = default;
};
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 85-98: Type declaration for TKTMMode / TKTMMode 的类型声明
```cpp

enum TKTMMode {
  VSETTK = 0,
  VSETTM = 1,
};

class RISCVInsertVSETVLI : public MachineFunctionPass {
  const RISCVSubtarget *ST;
  const TargetInstrInfo *TII;
  MachineRegisterInfo *MRI;
  // Possibly null!
  LiveIntervals *LIS;
  RISCVVSETVLIInfoAnalysis VIA;
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 99-116: Function implementation: RISCVInsertVSETVLI / 函数实现：RISCVInsertVSETVLI
```cpp
  std::vector<BlockData> BlockInfo;
  std::queue<const MachineBasicBlock *> WorkList;

public:
  static char ID;

  RISCVInsertVSETVLI() : MachineFunctionPass(ID) {}
  bool runOnMachineFunction(MachineFunction &MF) override;

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();

    AU.addUsedIfAvailable<LiveIntervalsWrapperPass>();
    AU.addPreserved<LiveIntervalsWrapperPass>();
    AU.addPreserved<SlotIndexesWrapperPass>();
    AU.addPreserved<LiveDebugVariablesWrapperLegacy>();
    AU.addPreserved<LiveStacksWrapperLegacy>();
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 117-130: Function implementation: MachineFunctionPass::getAnalysisUsage / 函数实现：MachineFunctionPass::getAnalysisUsage
```cpp
    MachineFunctionPass::getAnalysisUsage(AU);
  }

  StringRef getPassName() const override { return RISCV_INSERT_VSETVLI_NAME; }

private:
  bool needVSETVLI(const DemandedFields &Used, const VSETVLIInfo &Require,
                   const VSETVLIInfo &CurInfo) const;
  bool needVSETVLIPHI(const VSETVLIInfo &Require,
                      const MachineBasicBlock &MBB) const;
  void insertVSETVLI(MachineBasicBlock &MBB,
                     MachineBasicBlock::iterator InsertPt, DebugLoc DL,
                     const VSETVLIInfo &Info, const VSETVLIInfo &PrevInfo);
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 131-145: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  void transferBefore(VSETVLIInfo &Info, const MachineInstr &MI) const;
  void transferAfter(VSETVLIInfo &Info, const MachineInstr &MI) const;
  bool computeVLVTYPEChanges(const MachineBasicBlock &MBB,
                             VSETVLIInfo &Info) const;
  void computeIncomingVLVTYPE(const MachineBasicBlock &MBB);
  void emitVSETVLIs(MachineBasicBlock &MBB);
  void doPRE(MachineBasicBlock &MBB);
  void insertReadVL(MachineBasicBlock &MBB);

  bool canMutatePriorConfig(const MachineInstr &PrevMI, const MachineInstr &MI,
                            const DemandedFields &Used,
                            MachineInstr *&AVLDefToMove) const;
  void coalesceVSETVLIs(MachineBasicBlock &MBB) const;
  bool insertVSETMTK(MachineBasicBlock &MBB, TKTMMode Mode) const;
};
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 146-160: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp

} // end anonymous namespace

char RISCVInsertVSETVLI::ID = 0;
char &llvm::RISCVInsertVSETVLIID = RISCVInsertVSETVLI::ID;

INITIALIZE_PASS(RISCVInsertVSETVLI, DEBUG_TYPE, RISCV_INSERT_VSETVLI_NAME,
                false, false)

void RISCVInsertVSETVLI::insertVSETVLI(MachineBasicBlock &MBB,
                                       MachineBasicBlock::iterator InsertPt,
                                       DebugLoc DL, const VSETVLIInfo &Info,
                                       const VSETVLIInfo &PrevInfo) {
  ++NumInsertedVSETVL;
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 161-175: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  if (PrevInfo.isKnown()) {
    // Use X0, X0 form if the AVL is the same and the SEW+LMUL gives the same
    // VLMAX.
    if (Info.hasSameAVL(PrevInfo) && Info.hasSameVLMAX(PrevInfo)) {
      auto MI = BuildMI(MBB, InsertPt, DL,
                        TII->get(Info.getTWiden() ? RISCV::PseudoSF_VSETTNTX0X0
                                                  : RISCV::PseudoVSETVLIX0X0))
                    .addReg(RISCV::X0, RegState::Define | RegState::Dead)
                    .addReg(RISCV::X0, RegState::Kill)
                    .addImm(Info.encodeVTYPE())
                    .addReg(RISCV::VL, RegState::Implicit);
      if (LIS)
        LIS->InsertMachineInstrInMaps(*MI);
      return;
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 176-196: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

    // If our AVL is a virtual register, it might be defined by a VSET(I)VLI. If
    // it has the same VLMAX we want and the last VL/VTYPE we observed is the
    // same, we can use the X0, X0 form.
    if (Info.hasSameVLMAX(PrevInfo) && Info.hasAVLReg()) {
      if (const MachineInstr *DefMI = Info.getAVLDefMI(LIS);
          DefMI && RISCVInstrInfo::isVectorConfigInstr(*DefMI)) {
        VSETVLIInfo DefInfo = VIA.getInfoForVSETVLI(*DefMI);
        if (DefInfo.hasSameAVL(PrevInfo) && DefInfo.hasSameVLMAX(PrevInfo)) {
          auto MI =
              BuildMI(MBB, InsertPt, DL,
                      TII->get(Info.getTWiden() ? RISCV::PseudoSF_VSETTNTX0X0
                                                : RISCV::PseudoVSETVLIX0X0))
                  .addReg(RISCV::X0, RegState::Define | RegState::Dead)
                  .addReg(RISCV::X0, RegState::Kill)
                  .addImm(Info.encodeVTYPE())
                  .addReg(RISCV::VL, RegState::Implicit);
          if (LIS)
            LIS->InsertMachineInstrInMaps(*MI);
          return;
        }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 197-210: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
      }
    }
  }

  if (Info.hasAVLImm()) {
    auto MI = BuildMI(MBB, InsertPt, DL, TII->get(RISCV::PseudoVSETIVLI))
                  .addReg(RISCV::X0, RegState::Define | RegState::Dead)
                  .addImm(Info.getAVLImm())
                  .addImm(Info.encodeVTYPE());
    if (LIS)
      LIS->InsertMachineInstrInMaps(*MI);
    return;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 211-224: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  if (Info.hasAVLVLMAX()) {
    Register DestReg = MRI->createVirtualRegister(&RISCV::GPRNoX0RegClass);
    auto MI = BuildMI(MBB, InsertPt, DL,
                      TII->get(Info.getTWiden() ? RISCV::PseudoSF_VSETTNTX0
                                                : RISCV::PseudoVSETVLIX0))
                  .addReg(DestReg, RegState::Define | RegState::Dead)
                  .addReg(RISCV::X0, RegState::Kill)
                  .addImm(Info.encodeVTYPE());
    if (LIS) {
      LIS->InsertMachineInstrInMaps(*MI);
      LIS->createAndComputeVirtRegInterval(DestReg);
    }
    return;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 225-252: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  Register AVLReg = Info.getAVLReg();
  MRI->constrainRegClass(AVLReg, &RISCV::GPRNoX0RegClass);
  auto MI = BuildMI(MBB, InsertPt, DL,
                    TII->get(Info.getTWiden() ? RISCV::PseudoSF_VSETTNT
                                              : RISCV::PseudoVSETVLI))
                .addReg(RISCV::X0, RegState::Define | RegState::Dead)
                .addReg(AVLReg)
                .addImm(Info.encodeVTYPE());
  if (LIS) {
    LIS->InsertMachineInstrInMaps(*MI);
    LiveInterval &LI = LIS->getInterval(AVLReg);
    SlotIndex SI = LIS->getInstructionIndex(*MI).getRegSlot();
    const VNInfo *CurVNI = Info.getAVLVNInfo();
    // If the AVL value isn't live at MI, do a quick check to see if it's easily
    // extendable. Otherwise, we need to copy it.
    if (LI.getVNInfoBefore(SI) != CurVNI) {
      if (!LI.liveAt(SI) && LI.containsOneValue())
        LIS->extendToIndices(LI, SI);
      else {
        Register AVLCopyReg =
            MRI->createVirtualRegister(&RISCV::GPRNoX0RegClass);
        MachineBasicBlock *MBB = LIS->getMBBFromIndex(CurVNI->def);
        MachineBasicBlock::iterator II;
        if (CurVNI->isPHIDef())
          II = MBB->getFirstNonPHI();
        else {
          II = LIS->getInstructionFromIndex(CurVNI->def);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 253-273: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
          II = std::next(II);
        }
        assert(II.isValid());
        auto AVLCopy = BuildMI(*MBB, II, DL, TII->get(RISCV::COPY), AVLCopyReg)
                           .addReg(AVLReg);
        LIS->InsertMachineInstrInMaps(*AVLCopy);
        MI->getOperand(1).setReg(AVLCopyReg);
        LIS->createAndComputeVirtRegInterval(AVLCopyReg);
      }
    }
  }
}

/// Return true if a VSETVLI is required to transition from CurInfo to Require
/// given a set of DemandedFields \p Used.
bool RISCVInsertVSETVLI::needVSETVLI(const DemandedFields &Used,
                                     const VSETVLIInfo &Require,
                                     const VSETVLIInfo &CurInfo) const {
  if (!CurInfo.isKnown() || CurInfo.hasSEWLMULRatioOnly())
    return true;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 274-287: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  if (CurInfo.isCompatible(Used, Require, LIS))
    return false;

  return true;
}

// If we don't use LMUL or the SEW/LMUL ratio, then adjust LMUL so that we
// maintain the SEW/LMUL ratio. This allows us to eliminate VL toggles in more
// places.
static VSETVLIInfo adjustIncoming(const VSETVLIInfo &PrevInfo,
                                  const VSETVLIInfo &NewInfo,
                                  DemandedFields &Demanded) {
  VSETVLIInfo Info = NewInfo;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 288-314: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  if (!Demanded.LMUL && !Demanded.SEWLMULRatio && PrevInfo.isKnown()) {
    if (auto NewVLMul = RISCVVType::getSameRatioLMUL(PrevInfo.getSEWLMULRatio(),
                                                     Info.getSEW()))
      Info.setVLMul(*NewVLMul);
    Demanded.LMUL = DemandedFields::LMULEqual;
  }

  return Info;
}

// Given an incoming state reaching MI, minimally modifies that state so that it
// is compatible with MI. The resulting state is guaranteed to be semantically
// legal for MI, but may not be the state requested by MI.
void RISCVInsertVSETVLI::transferBefore(VSETVLIInfo &Info,
                                        const MachineInstr &MI) const {
  if (EnsureWholeVectorRegisterMoveValidVTYPE &&
      RISCV::isVectorCopy(ST->getRegisterInfo(), MI) &&
      (!Info.isKnown() || Info.hasSEWLMULRatioOnly())) {
    // Use an arbitrary but valid AVL and VTYPE so vill will be cleared. It may
    // be coalesced into another vsetvli since we won't demand any fields.
    VSETVLIInfo NewInfo; // Need a new VSETVLIInfo to clear SEWLMULRatioOnly
    NewInfo.setAVLImm(1);
    NewInfo.setVTYPE(RISCVVType::LMUL_1, /*sew*/ 8, /*ta*/ true, /*ma*/ true,
                     /*AltFmt*/ false, /*W*/ 0);
    Info = NewInfo;
    return;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 315-329: Definitions and supporting logic / 定义与支撑逻辑
```cpp

  if (!RISCVII::hasSEWOp(MI.getDesc().TSFlags))
    return;

  DemandedFields Demanded = getDemanded(MI, ST);

  const VSETVLIInfo NewInfo = VIA.computeInfoForInstr(MI);
  assert(NewInfo.isKnown());
  if (Info.isValid() && !needVSETVLI(Demanded, NewInfo, Info))
    return;

  const VSETVLIInfo PrevInfo = Info;
  if (!Info.isKnown())
    Info = NewInfo;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 330-343: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  const VSETVLIInfo IncomingInfo = adjustIncoming(PrevInfo, NewInfo, Demanded);

  // If MI only demands that VL has the same zeroness, we only need to set the
  // AVL if the zeroness differs.  This removes a vsetvli entirely if the types
  // match or allows use of cheaper avl preserving variant if VLMAX doesn't
  // change. If VLMAX might change, we couldn't use the 'vsetvli x0, x0, vtype"
  // variant, so we avoid the transform to prevent extending live range of an
  // avl register operand.
  // TODO: We can probably relax this for immediates.
  bool EquallyZero = IncomingInfo.hasEquallyZeroAVL(PrevInfo, LIS) &&
                     IncomingInfo.hasSameVLMAX(PrevInfo);
  if (Demanded.VLAny || (Demanded.VLZeroness && !EquallyZero))
    Info.setAVL(IncomingInfo);
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 344-366: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // If we only knew the sew/lmul ratio previously, replace the VTYPE.
  if (Info.hasSEWLMULRatioOnly()) {
    VSETVLIInfo RatiolessInfo = IncomingInfo;
    RatiolessInfo.setAVL(Info);
    Info = RatiolessInfo;
  } else {
    unsigned SEW =
        ((Demanded.SEW || Demanded.SEWLMULRatio) ? IncomingInfo : Info)
            .getSEW();
    Info.setVTYPE(
        ((Demanded.LMUL || Demanded.SEWLMULRatio) ? IncomingInfo : Info)
            .getVLMUL(),
        SEW,
        // Prefer tail/mask agnostic since it can be relaxed to undisturbed
        // later if needed.
        (Demanded.TailPolicy ? IncomingInfo : Info).getTailAgnostic() ||
            IncomingInfo.getTailAgnostic(),
        (Demanded.MaskPolicy ? IncomingInfo : Info).getMaskAgnostic() ||
            IncomingInfo.getMaskAgnostic(),
        // AltFmt requires SEW < 32.
        (Demanded.AltFmt ? IncomingInfo : Info).getAltFmt() && SEW < 32,
        Demanded.TWiden ? IncomingInfo.getTWiden() : 0);
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 367-391: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
}

// Given a state with which we evaluated MI (see transferBefore above for why
// this might be different that the state MI requested), modify the state to
// reflect the changes MI might make.
void RISCVInsertVSETVLI::transferAfter(VSETVLIInfo &Info,
                                       const MachineInstr &MI) const {
  if (RISCVInstrInfo::isVectorConfigInstr(MI)) {
    Info = VIA.getInfoForVSETVLI(MI);
    return;
  }

  if (RISCVInstrInfo::isFaultOnlyFirstLoad(MI)) {
    // Update AVL to vl-output of the fault first load.
    assert(MI.getOperand(1).getReg().isVirtual());
    if (LIS) {
      auto &LI = LIS->getInterval(MI.getOperand(1).getReg());
      SlotIndex SI =
          LIS->getSlotIndexes()->getInstructionIndex(MI).getRegSlot();
      VNInfo *VNI = LI.getVNInfoAt(SI);
      Info.setAVLRegDef(VNI, MI.getOperand(1).getReg());
    } else
      Info.setAVLRegDef(nullptr, MI.getOperand(1).getReg());
    return;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 392-408: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  // If this is something that updates VL/VTYPE that we don't know about, set
  // the state to unknown.
  if (MI.isCall() || MI.isInlineAsm() ||
      MI.modifiesRegister(RISCV::VL, /*TRI=*/nullptr) ||
      MI.modifiesRegister(RISCV::VTYPE, /*TRI=*/nullptr))
    Info = VSETVLIInfo::getUnknown();
}

bool RISCVInsertVSETVLI::computeVLVTYPEChanges(const MachineBasicBlock &MBB,
                                               VSETVLIInfo &Info) const {
  bool HadVectorOp = false;

  Info = BlockInfo[MBB.getNumber()].Pred;
  for (const MachineInstr &MI : MBB) {
    transferBefore(Info, MI);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 409-422: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    if (RISCVInstrInfo::isVectorConfigInstr(MI) ||
        RISCVII::hasSEWOp(MI.getDesc().TSFlags) ||
        RISCV::isVectorCopy(ST->getRegisterInfo(), MI) ||
        RISCVInstrInfo::isXSfmmVectorConfigInstr(MI))
      HadVectorOp = true;

    transferAfter(Info, MI);
  }

  return HadVectorOp;
}

void RISCVInsertVSETVLI::computeIncomingVLVTYPE(const MachineBasicBlock &MBB) {
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 423-436: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  BlockData &BBInfo = BlockInfo[MBB.getNumber()];

  BBInfo.InQueue = false;

  // Start with the previous entry so that we keep the most conservative state
  // we have ever found.
  VSETVLIInfo InInfo = BBInfo.Pred;
  if (MBB.pred_empty()) {
    // There are no predecessors, so use the default starting status.
    InInfo.setUnknown();
  } else {
    for (MachineBasicBlock *P : MBB.predecessors())
      InInfo = InInfo.intersect(BlockInfo[P->getNumber()].Exit);
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 437-456: Definitions and supporting logic / 定义与支撑逻辑
```cpp

  // If we don't have any valid predecessor value, wait until we do.
  if (!InInfo.isValid())
    return;

  // If no change, no need to rerun block
  if (InInfo == BBInfo.Pred)
    return;

  BBInfo.Pred = InInfo;
  LLVM_DEBUG(dbgs() << "Entry state of " << printMBBReference(MBB)
                    << " changed to " << BBInfo.Pred << "\n");

  // Note: It's tempting to cache the state changes here, but due to the
  // compatibility checks performed a blocks output state can change based on
  // the input state.  To cache, we'd have to add logic for finding
  // never-compatible state changes.
  VSETVLIInfo TmpStatus;
  computeVLVTYPEChanges(MBB, TmpStatus);
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 457-472: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // If the new exit value matches the old exit value, we don't need to revisit
  // any blocks.
  if (BBInfo.Exit == TmpStatus)
    return;

  BBInfo.Exit = TmpStatus;
  LLVM_DEBUG(dbgs() << "Exit state of " << printMBBReference(MBB)
                    << " changed to " << BBInfo.Exit << "\n");

  // Add the successors to the work list so we can propagate the changed exit
  // status.
  for (MachineBasicBlock *S : MBB.successors())
    if (!BlockInfo[S->getNumber()].InQueue) {
      BlockInfo[S->getNumber()].InQueue = true;
      WorkList.push(S);
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 473-490: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
}

// If we weren't able to prove a vsetvli was directly unneeded, it might still
// be unneeded if the AVL was a phi node where all incoming values are VL
// outputs from the last VSETVLI in their respective basic blocks.
bool RISCVInsertVSETVLI::needVSETVLIPHI(const VSETVLIInfo &Require,
                                        const MachineBasicBlock &MBB) const {
  if (!Require.hasAVLReg())
    return true;

  if (!LIS)
    return true;

  // We need the AVL to have been produced by a PHI node in this basic block.
  const VNInfo *Valno = Require.getAVLVNInfo();
  if (!Valno->isPHIDef() || LIS->getMBBFromIndex(Valno->def) != &MBB)
    return true;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 491-509: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  const LiveRange &LR = LIS->getInterval(Require.getAVLReg());

  for (auto *PBB : MBB.predecessors()) {
    const VSETVLIInfo &PBBExit = BlockInfo[PBB->getNumber()].Exit;

    // We need the PHI input to the be the output of a VSET(I)VLI.
    const VNInfo *Value = LR.getVNInfoBefore(LIS->getMBBEndIdx(PBB));
    if (!Value)
      return true;
    MachineInstr *DefMI = LIS->getInstructionFromIndex(Value->def);
    if (!DefMI || !RISCVInstrInfo::isVectorConfigInstr(*DefMI))
      return true;

    // We found a VSET(I)VLI make sure it matches the output of the
    // predecessor block.
    VSETVLIInfo DefInfo = VIA.getInfoForVSETVLI(*DefMI);
    if (DefInfo != PBBExit)
      return true;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 510-530: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    // Require has the same VL as PBBExit, so if the exit from the
    // predecessor has the VTYPE we are looking for we might be able
    // to avoid a VSETVLI.
    if (PBBExit.isUnknown() || !PBBExit.hasSameVTYPE(Require))
      return true;
  }

  // If all the incoming values to the PHI checked out, we don't need
  // to insert a VSETVLI.
  return false;
}

void RISCVInsertVSETVLI::emitVSETVLIs(MachineBasicBlock &MBB) {
  VSETVLIInfo CurInfo = BlockInfo[MBB.getNumber()].Pred;
  // Track whether the prefix of the block we've scanned is transparent
  // (meaning has not yet changed the abstract state).
  bool PrefixTransparent = true;
  for (MachineInstr &MI : MBB) {
    const VSETVLIInfo PrevInfo = CurInfo;
    transferBefore(CurInfo, MI);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 531-547: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    // If this is an explicit VSETVLI or VSETIVLI, update our state.
    if (RISCVInstrInfo::isVectorConfigInstr(MI)) {
      // Conservatively, mark the VL and VTYPE as live.
      assert(MI.getOperand(3).getReg() == RISCV::VL &&
             MI.getOperand(4).getReg() == RISCV::VTYPE &&
             "Unexpected operands where VL and VTYPE should be");
      MI.getOperand(3).setIsDead(false);
      MI.getOperand(4).setIsDead(false);
      PrefixTransparent = false;
    }

    if (EnsureWholeVectorRegisterMoveValidVTYPE &&
        RISCV::isVectorCopy(ST->getRegisterInfo(), MI)) {
      if (!PrevInfo.isCompatible(DemandedFields::all(), CurInfo, LIS)) {
        insertVSETVLI(MBB, MI, MI.getDebugLoc(), CurInfo, PrevInfo);
        PrefixTransparent = false;
      }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 548-565: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
      MI.addOperand(MachineOperand::CreateReg(RISCV::VTYPE, /*isDef*/ false,
                                              /*isImp*/ true));
    }

    uint64_t TSFlags = MI.getDesc().TSFlags;
    if (RISCVII::hasSEWOp(TSFlags)) {
      if (!PrevInfo.isCompatible(DemandedFields::all(), CurInfo, LIS)) {
        // If this is the first implicit state change, and the state change
        // requested can be proven to produce the same register contents, we
        // can skip emitting the actual state change and continue as if we
        // had since we know the GPR result of the implicit state change
        // wouldn't be used and VL/VTYPE registers are correct.  Note that
        // we *do* need to model the state as if it changed as while the
        // register contents are unchanged, the abstract model can change.
        if (!PrefixTransparent || needVSETVLIPHI(CurInfo, MBB))
          insertVSETVLI(MBB, MI, MI.getDebugLoc(), CurInfo, PrevInfo);
        PrefixTransparent = false;
      }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 566-583: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

      if (RISCVII::hasVLOp(TSFlags)) {
        MachineOperand &VLOp = getVLOp(MI);
        if (VLOp.isReg()) {
          Register Reg = VLOp.getReg();

          // Erase the AVL operand from the instruction.
          VLOp.setReg(Register());
          VLOp.setIsKill(false);
          if (LIS) {
            LiveInterval &LI = LIS->getInterval(Reg);
            SmallVector<MachineInstr *> DeadMIs;
            LIS->shrinkToUses(&LI, &DeadMIs);
            // We might have separate components that need split due to
            // needVSETVLIPHI causing us to skip inserting a new VL def.
            SmallVector<LiveInterval *> SplitLIs;
            LIS->splitSeparateComponents(LI, SplitLIs);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 584-597: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
            // If the AVL was an immediate > 31, then it would have been emitted
            // as an ADDI. However, the ADDI might not have been used in the
            // vsetvli, or a vsetvli might not have been emitted, so it may be
            // dead now.
            for (MachineInstr *DeadMI : DeadMIs) {
              if (!TII->isAddImmediate(*DeadMI, Reg))
                continue;
              LIS->RemoveMachineInstrFromMaps(*DeadMI);
              Register AddReg = DeadMI->getOperand(1).getReg();
              DeadMI->eraseFromParent();
              if (AddReg.isVirtual())
                LIS->shrinkToUses(&LIS->getInterval(AddReg));
            }
          }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 598-611: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
        }
        MI.addOperand(MachineOperand::CreateReg(RISCV::VL, /*isDef*/ false,
                                                /*isImp*/ true));
      }
      MI.addOperand(MachineOperand::CreateReg(RISCV::VTYPE, /*isDef*/ false,
                                              /*isImp*/ true));
    }

    if (MI.isInlineAsm()) {
      MI.addOperand(MachineOperand::CreateReg(RISCV::VL, /*isDef*/ true,
                                              /*isImp*/ true));
      MI.addOperand(MachineOperand::CreateReg(RISCV::VTYPE, /*isDef*/ true,
                                              /*isImp*/ true));
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 612-627: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

    if (MI.isCall() || MI.isInlineAsm() ||
        MI.modifiesRegister(RISCV::VL, /*TRI=*/nullptr) ||
        MI.modifiesRegister(RISCV::VTYPE, /*TRI=*/nullptr))
      PrefixTransparent = false;

    transferAfter(CurInfo, MI);
  }

  const auto &Info = BlockInfo[MBB.getNumber()];
  if (CurInfo != Info.Exit) {
    LLVM_DEBUG(dbgs() << "in block " << printMBBReference(MBB) << "\n");
    LLVM_DEBUG(dbgs() << "  begin        state: " << Info.Pred << "\n");
    LLVM_DEBUG(dbgs() << "  expected end state: " << Info.Exit << "\n");
    LLVM_DEBUG(dbgs() << "  actual   end state: " << CurInfo << "\n");
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 628-652: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  assert(CurInfo == Info.Exit && "InsertVSETVLI dataflow invariant violated");
}

/// Perform simple partial redundancy elimination of the VSETVLI instructions
/// we're about to insert by looking for cases where we can PRE from the
/// beginning of one block to the end of one of its predecessors.  Specifically,
/// this is geared to catch the common case of a fixed length vsetvl in a single
/// block loop when it could execute once in the preheader instead.
void RISCVInsertVSETVLI::doPRE(MachineBasicBlock &MBB) {
  if (!BlockInfo[MBB.getNumber()].Pred.isUnknown())
    return;

  MachineBasicBlock *UnavailablePred = nullptr;
  VSETVLIInfo AvailableInfo;
  for (MachineBasicBlock *P : MBB.predecessors()) {
    const VSETVLIInfo &PredInfo = BlockInfo[P->getNumber()].Exit;
    if (PredInfo.isUnknown()) {
      if (UnavailablePred)
        return;
      UnavailablePred = P;
    } else if (!AvailableInfo.isValid()) {
      AvailableInfo = PredInfo;
    } else if (AvailableInfo != PredInfo) {
      return;
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 653-667: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  }

  // Unreachable, single pred, or full redundancy. Note that FRE is handled by
  // phase 3.
  if (!UnavailablePred || !AvailableInfo.isValid())
    return;

  if (!LIS)
    return;

  // If we don't know the exact VTYPE, we can't copy the vsetvli to the exit of
  // the unavailable pred.
  if (AvailableInfo.hasSEWLMULRatioOnly())
    return;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 668-684: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // Critical edge - TODO: consider splitting?
  if (UnavailablePred->succ_size() != 1)
    return;

  // If the AVL value is a register (other than our VLMAX sentinel),
  // we need to prove the value is available at the point we're going
  // to insert the vsetvli at.
  if (AvailableInfo.hasAVLReg()) {
    SlotIndex SI = AvailableInfo.getAVLVNInfo()->def;
    // This is an inline dominance check which covers the case of
    // UnavailablePred being the preheader of a loop.
    if (LIS->getMBBFromIndex(SI) != UnavailablePred)
      return;
    if (!UnavailablePred->terminators().empty() &&
        SI >= LIS->getInstructionIndex(*UnavailablePred->getFirstTerminator()))
      return;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 685-711: Commentary and design intent / 注释与设计意图
```cpp

  // Model the effect of changing the input state of the block MBB to
  // AvailableInfo.  We're looking for two issues here; one legality,
  // one profitability.
  // 1) If the block doesn't use some of the fields from VL or VTYPE, we
  //    may hit the end of the block with a different end state.  We can
  //    not make this change without reflowing later blocks as well.
  // 2) If we don't actually remove a transition, inserting a vsetvli
  //    into the predecessor block would be correct, but unprofitable.
  VSETVLIInfo OldInfo = BlockInfo[MBB.getNumber()].Pred;
  VSETVLIInfo CurInfo = AvailableInfo;
  int TransitionsRemoved = 0;
  for (const MachineInstr &MI : MBB) {
    const VSETVLIInfo LastInfo = CurInfo;
    const VSETVLIInfo LastOldInfo = OldInfo;
    transferBefore(CurInfo, MI);
    transferBefore(OldInfo, MI);
    if (CurInfo == LastInfo)
      TransitionsRemoved++;
    if (LastOldInfo == OldInfo)
      TransitionsRemoved--;
    transferAfter(CurInfo, MI);
    transferAfter(OldInfo, MI);
    if (CurInfo == OldInfo)
      // Convergence.  All transitions after this must match by construction.
      break;
  }
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 712-725: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  if (CurInfo != OldInfo || TransitionsRemoved <= 0)
    // Issues 1 and 2 above
    return;

  // Finally, update both data flow state and insert the actual vsetvli.
  // Doing both keeps the code in sync with the dataflow results, which
  // is critical for correctness of phase 3.
  auto OldExit = BlockInfo[UnavailablePred->getNumber()].Exit;
  LLVM_DEBUG(dbgs() << "PRE VSETVLI from " << MBB.getName() << " to "
                    << UnavailablePred->getName() << " with state "
                    << AvailableInfo << "\n");
  BlockInfo[UnavailablePred->getNumber()].Exit = AvailableInfo;
  BlockInfo[MBB.getNumber()].Pred = AvailableInfo;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 726-748: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // Note there's an implicit assumption here that terminators never use
  // or modify VL or VTYPE.  Also, fallthrough will return end().
  auto InsertPt = UnavailablePred->getFirstInstrTerminator();
  insertVSETVLI(*UnavailablePred, InsertPt,
                UnavailablePred->findDebugLoc(InsertPt),
                AvailableInfo, OldExit);
}

// Return true if we can mutate PrevMI to match MI without changing any the
// fields which would be observed.
// If AVLDefToMove is non-null after the call, it points to an ADDI
// instruction that needs to be moved before PrevMI.
bool RISCVInsertVSETVLI::canMutatePriorConfig(
    const MachineInstr &PrevMI, const MachineInstr &MI,
    const DemandedFields &Used, MachineInstr *&AVLDefToMove) const {
  AVLDefToMove = nullptr;
  // If the VL values aren't equal, return false if either a) the former is
  // demanded, or b) we can't rewrite the former to be the later for
  // implementation reasons.
  if (!RISCVInstrInfo::isVLPreservingConfig(MI)) {
    if (Used.VLAny)
      return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 749-773: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    if (Used.VLZeroness) {
      if (RISCVInstrInfo::isVLPreservingConfig(PrevMI))
        return false;
      if (!VIA.getInfoForVSETVLI(PrevMI).hasEquallyZeroAVL(
              VIA.getInfoForVSETVLI(MI), LIS))
        return false;
    }

    auto &AVL = MI.getOperand(1);

    // If the AVL is a register, we need to make sure its definition is the same
    // at PrevMI as it was at MI.
    if (AVL.isReg() && AVL.getReg() != RISCV::X0) {
      VNInfo *VNI = getVNInfoFromReg(AVL.getReg(), MI, LIS);
      VNInfo *PrevVNI = getVNInfoFromReg(AVL.getReg(), PrevMI, LIS);
      if (!VNI || !PrevVNI || VNI != PrevVNI) {
        // If LIS is null, we were not able to get the VNInfo so we don't know
        // if the AVL def needs to be moved.
        if (!LIS)
          return false;
        // If the AVL is defined by a load immediate instruction (ADDI x0, imm),
        // it can be moved earlier since it has no register dependencies.
        if (!AVL.getReg().isVirtual())
          return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 774-791: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
        MachineInstr *DefMI = MRI->getUniqueVRegDef(AVL.getReg());
        if (!DefMI || !RISCVInstrInfo::isLoadImmediate(*DefMI) ||
            DefMI->getParent() != PrevMI.getParent()) {
          return false;
        }
        // Mark that this ADDI needs to be moved.
        AVLDefToMove = DefMI;
      }
    }

    // If we define VL and need to move the definition up, check we can extend
    // the live interval upwards from MI to PrevMI.
    Register VL = MI.getOperand(0).getReg();
    if (VL.isVirtual() && LIS &&
        LIS->getInterval(VL).overlaps(LIS->getInstructionIndex(PrevMI),
                                      LIS->getInstructionIndex(MI)))
      return false;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 792-807: Function implementation: RISCVInsertVSETVLI::coalesceVSETVLIs / 函数实现：RISCVInsertVSETVLI::coalesceVSETVLIs
```cpp

  assert(PrevMI.getOperand(2).isImm() && MI.getOperand(2).isImm());
  auto PriorVType = PrevMI.getOperand(2).getImm();
  auto VType = MI.getOperand(2).getImm();
  return areCompatibleVTYPEs(PriorVType, VType, Used);
}

void RISCVInsertVSETVLI::coalesceVSETVLIs(MachineBasicBlock &MBB) const {
  MachineInstr *NextMI = nullptr;
  // We can have arbitrary code in successors, so VL and VTYPE
  // must be considered demanded.
  DemandedFields Used;
  Used.demandVL();
  Used.demandVTYPE();
  SmallVector<MachineInstr*> ToDelete;
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 808-821: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  auto dropAVLUse = [&](MachineOperand &MO) {
    if (!MO.isReg() || !MO.getReg().isVirtual())
      return;
    Register OldVLReg = MO.getReg();
    MO.setReg(Register());

    if (LIS)
      LIS->shrinkToUses(&LIS->getInterval(OldVLReg));

    MachineInstr *VLOpDef = MRI->getUniqueVRegDef(OldVLReg);
    if (VLOpDef && TII->isAddImmediate(*VLOpDef, OldVLReg) &&
        MRI->use_nodbg_empty(OldVLReg))
      ToDelete.push_back(VLOpDef);
  };
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 822-838: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  for (MachineInstr &MI : make_early_inc_range(reverse(MBB))) {
    // TODO: Support XSfmm.
    if (RISCVII::hasTWidenOp(MI.getDesc().TSFlags) ||
        RISCVInstrInfo::isXSfmmVectorConfigInstr(MI)) {
      NextMI = nullptr;
      continue;
    }

    if (!RISCVInstrInfo::isVectorConfigInstr(MI)) {
      Used.doUnion(getDemanded(MI, ST));
      if (MI.isCall() || MI.isInlineAsm() ||
          MI.modifiesRegister(RISCV::VL, /*TRI=*/nullptr) ||
          MI.modifiesRegister(RISCV::VTYPE, /*TRI=*/nullptr))
        NextMI = nullptr;
      continue;
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 839-852: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

    if (!MI.getOperand(0).isDead())
      Used.demandVL();

    if (NextMI) {
      if (!Used.usedVL() && !Used.usedVTYPE()) {
        dropAVLUse(MI.getOperand(1));
        if (LIS)
          LIS->RemoveMachineInstrFromMaps(MI);
        MI.eraseFromParent();
        NumCoalescedVSETVL++;
        // Leave NextMI unchanged
        continue;
      }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 853-869: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

      MachineInstr *AVLDefToMove = nullptr;
      if (canMutatePriorConfig(MI, *NextMI, Used, AVLDefToMove)) {
        if (!RISCVInstrInfo::isVLPreservingConfig(*NextMI)) {
          Register DefReg = NextMI->getOperand(0).getReg();

          MI.getOperand(0).setReg(DefReg);
          MI.getOperand(0).setIsDead(false);

          // Move the AVL from NextMI to MI
          dropAVLUse(MI.getOperand(1));
          if (NextMI->getOperand(1).isImm())
            MI.getOperand(1).ChangeToImmediate(NextMI->getOperand(1).getImm());
          else {
            MI.getOperand(1).ChangeToRegister(NextMI->getOperand(1).getReg(),
                                              false);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 870-893: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
            // If canMutatePriorConfig indicated that an ADDI needs to be moved,
            // move it now.
            if (AVLDefToMove) {
              AVLDefToMove->moveBefore(&MI);
              if (LIS)
                LIS->handleMove(*AVLDefToMove);
            }
          }
          dropAVLUse(NextMI->getOperand(1));

          // The def of DefReg moved to MI, so extend the LiveInterval up to
          // it.
          if (DefReg.isVirtual() && LIS) {
            LiveInterval &DefLI = LIS->getInterval(DefReg);
            SlotIndex MISlot = LIS->getInstructionIndex(MI).getRegSlot();
            SlotIndex NextMISlot =
                LIS->getInstructionIndex(*NextMI).getRegSlot();
            VNInfo *DefVNI = DefLI.getVNInfoAt(NextMISlot);
            LiveInterval::Segment S(MISlot, NextMISlot, DefVNI);
            DefLI.addSegment(S);
            DefVNI->def = MISlot;
            // Mark DefLI as spillable if it was previously unspillable
            DefLI.setWeight(0);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 894-909: Definitions and supporting logic / 定义与支撑逻辑
```cpp
            // DefReg may have had no uses, in which case we need to shrink
            // the LiveInterval up to MI.
            LIS->shrinkToUses(&DefLI);
          }

          MI.setDesc(NextMI->getDesc());
        }
        MI.getOperand(2).setImm(NextMI->getOperand(2).getImm());

        dropAVLUse(NextMI->getOperand(1));
        if (LIS)
          LIS->RemoveMachineInstrFromMaps(*NextMI);
        NextMI->eraseFromParent();
        NumCoalescedVSETVL++;
        // fallthrough
      }
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 910-923: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    }
    NextMI = &MI;
    Used = getDemanded(MI, ST);
  }

  // Loop over the dead AVL values, and delete them now.  This has
  // to be outside the above loop to avoid invalidating iterators.
  for (auto *MI : ToDelete) {
    assert(MI->getOpcode() == RISCV::ADDI);
    Register AddReg = MI->getOperand(1).getReg();
    if (LIS) {
      LIS->removeInterval(MI->getOperand(0).getReg());
      LIS->RemoveMachineInstrFromMaps(*MI);
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 924-948: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    MI->eraseFromParent();
    if (LIS && AddReg.isVirtual())
      LIS->shrinkToUses(&LIS->getInterval(AddReg));
  }
}

void RISCVInsertVSETVLI::insertReadVL(MachineBasicBlock &MBB) {
  for (auto I = MBB.begin(), E = MBB.end(); I != E;) {
    MachineInstr &MI = *I++;
    if (RISCVInstrInfo::isFaultOnlyFirstLoad(MI)) {
      Register VLOutput = MI.getOperand(1).getReg();
      assert(VLOutput.isVirtual());
      if (!MI.getOperand(1).isDead()) {
        auto ReadVLMI = BuildMI(MBB, I, MI.getDebugLoc(),
                                TII->get(RISCV::PseudoReadVL), VLOutput);
        // Move the LiveInterval's definition down to PseudoReadVL.
        if (LIS) {
          SlotIndex NewDefSI =
              LIS->InsertMachineInstrInMaps(*ReadVLMI).getRegSlot();
          LiveInterval &DefLI = LIS->getInterval(VLOutput);
          LiveRange::Segment *DefSeg = DefLI.getSegmentContaining(NewDefSI);
          VNInfo *DefVNI = DefLI.getVNInfoAt(DefSeg->start);
          DefLI.removeSegment(DefSeg->start, NewDefSI);
          DefVNI->def = NewDefSI;
        }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 949-966: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
      }
      // We don't use the vl output of the VLEFF/VLSEGFF anymore.
      MI.getOperand(1).setReg(RISCV::X0);
      MI.addRegisterDefined(RISCV::VL, MRI->getTargetRegisterInfo());
    }
  }
}

bool RISCVInsertVSETVLI::insertVSETMTK(MachineBasicBlock &MBB,
                                       TKTMMode Mode) const {

  bool Changed = false;
  for (auto &MI : MBB) {
    uint64_t TSFlags = MI.getDesc().TSFlags;
    if (RISCVInstrInfo::isXSfmmVectorConfigTMTKInstr(MI) ||
        !RISCVII::hasSEWOp(TSFlags) || !RISCVII::hasTWidenOp(TSFlags))
      continue;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 967-983: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    VSETVLIInfo CurrInfo = VIA.computeInfoForInstr(MI);

    unsigned Opcode = 0, OpNum = 0;
    switch (Mode) {
    case VSETTK:
      if (!RISCVII::hasTKOp(TSFlags))
        continue;
      OpNum = RISCVII::getTKOpNum(MI.getDesc());
      Opcode = RISCV::PseudoSF_VSETTK;
      break;
    case VSETTM:
      if (!RISCVII::hasTMOp(TSFlags))
        continue;
      OpNum = RISCVII::getTMOpNum(MI.getDesc());
      Opcode = RISCV::PseudoSF_VSETTM;
      break;
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 984-1002: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

    assert(OpNum && Opcode && "Invalid OpNum or Opcode");

    MachineOperand &Op = MI.getOperand(OpNum);

    auto TmpMI = BuildMI(MBB, MI, MI.getDebugLoc(), TII->get(Opcode))
                     .addReg(RISCV::X0, RegState::Define | RegState::Dead)
                     .addReg(Op.getReg())
                     .addImm(Log2_32(CurrInfo.getSEW()))
                     .addImm(CurrInfo.getTWiden());

    Changed = true;
    Register Reg = Op.getReg();
    Op.setReg(Register());
    Op.setIsKill(false);
    if (LIS) {
      LIS->InsertMachineInstrInMaps(*TmpMI);
      LiveInterval &LI = LIS->getInterval(Reg);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 1003-1018: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
      // Erase the AVL operand from the instruction.
      LIS->shrinkToUses(&LI);
      // TODO: Enable this once needVSETVLIPHI is supported.
      // SmallVector<LiveInterval *> SplitLIs;
      // LIS->splitSeparateComponents(LI, SplitLIs);
    }
  }
  return Changed;
}

bool RISCVInsertVSETVLI::runOnMachineFunction(MachineFunction &MF) {
  // Skip if the vector extension is not enabled.
  ST = &MF.getSubtarget<RISCVSubtarget>();
  if (!ST->hasVInstructions())
    return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 1019-1041: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  LLVM_DEBUG(dbgs() << "Entering InsertVSETVLI for " << MF.getName() << "\n");

  TII = ST->getInstrInfo();
  MRI = &MF.getRegInfo();
  auto *LISWrapper = getAnalysisIfAvailable<LiveIntervalsWrapperPass>();
  LIS = LISWrapper ? &LISWrapper->getLIS() : nullptr;
  VIA = RISCVVSETVLIInfoAnalysis(ST, LIS);

  assert(BlockInfo.empty() && "Expect empty block infos");
  BlockInfo.resize(MF.getNumBlockIDs());

  bool HaveVectorOp = false;

  // Phase 1 - determine how VL/VTYPE are affected by the each block.
  for (const MachineBasicBlock &MBB : MF) {
    VSETVLIInfo TmpStatus;
    HaveVectorOp |= computeVLVTYPEChanges(MBB, TmpStatus);
    // Initial exit state is whatever change we found in the block.
    BlockData &BBInfo = BlockInfo[MBB.getNumber()];
    BBInfo.Exit = TmpStatus;
    LLVM_DEBUG(dbgs() << "Initial exit state of " << printMBBReference(MBB)
                      << " is " << BBInfo.Exit << "\n");
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 1042-1056: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  }

  // If we didn't find any instructions that need VSETVLI, we're done.
  if (!HaveVectorOp) {
    BlockInfo.clear();
    return false;
  }

  // Phase 2 - determine the exit VL/VTYPE from each block. We add all
  // blocks to the list here, but will also add any that need to be revisited
  // during Phase 2 processing.
  for (const MachineBasicBlock &MBB : MF) {
    WorkList.push(&MBB);
    BlockInfo[MBB.getNumber()].InQueue = true;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 1057-1073: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  while (!WorkList.empty()) {
    const MachineBasicBlock &MBB = *WorkList.front();
    WorkList.pop();
    computeIncomingVLVTYPE(MBB);
  }

  // Perform partial redundancy elimination of vsetvli transitions.
  for (MachineBasicBlock &MBB : MF)
    doPRE(MBB);

  // Phase 3 - add any vsetvli instructions needed in the block. Use the
  // Phase 2 information to avoid adding vsetvlis before the first vector
  // instruction in the block if the VL/VTYPE is satisfied by its
  // predecessors.
  for (MachineBasicBlock &MBB : MF)
    emitVSETVLIs(MBB);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 1074-1090: Commentary and design intent / 注释与设计意图
```cpp
  // Now that all vsetvlis are explicit, go through and do block local
  // DSE and peephole based demanded fields based transforms.  Note that
  // this *must* be done outside the main dataflow so long as we allow
  // any cross block analysis within the dataflow.  We can't have both
  // demanded fields based mutation and non-local analysis in the
  // dataflow at the same time without introducing inconsistencies.
  // We're visiting blocks from the bottom up because a VSETVLI in the
  // earlier block might become dead when its uses in later blocks are
  // optimized away.
  for (MachineBasicBlock *MBB : post_order(&MF))
    coalesceVSETVLIs(*MBB);

  // Insert PseudoReadVL after VLEFF/VLSEGFF and replace it with the vl output
  // of VLEFF/VLSEGFF.
  for (MachineBasicBlock &MBB : MF)
    insertReadVL(MBB);
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 1091-1103: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  for (MachineBasicBlock &MBB : MF) {
    insertVSETMTK(MBB, VSETTM);
    insertVSETMTK(MBB, VSETTK);
  }

  BlockInfo.clear();
  return HaveVectorOp;
}

/// Returns an instance of the Insert VSETVLI pass.
FunctionPass *llvm::createRISCVInsertVSETVLIPass() {
  return new RISCVInsertVSETVLI();
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

## Key Concepts / 关键概念
- **Vector configuration tracking** / **向量配置跟踪**

## Dependencies / 依赖关系
- `RISCV.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVSubtarget.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVVSETVLIInfoAnalysis.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/ADT/PostOrderIterator.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/ADT/Statistic.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/LiveDebugVariables.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/LiveIntervals.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/LiveStacks.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/MachineFunctionPass.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `queue` — Directly referenced by this file. / 该文件直接引用的依赖。
