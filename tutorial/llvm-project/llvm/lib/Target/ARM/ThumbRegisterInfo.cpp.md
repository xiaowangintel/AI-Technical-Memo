# ThumbRegisterInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ThumbRegisterInfo.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains the Thumb-1 implementation of the TargetRegisterInfo class.
- 用途 (CN): 实现 ARM 后端中的 `ThumbRegisterInfo`，重点处理寄存器定义、分配约束以及寄存器工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===-- ThumbRegisterInfo.cpp - Thumb-1 Register Information -------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the Thumb-1 implementation of the TargetRegisterInfo
// class.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 14-28
```cpp
#include "ThumbRegisterInfo.h"
#include "ARMBaseInstrInfo.h"
#include "ARMMachineFunctionInfo.h"
#include "ARMSubtarget.h"
#include "llvm/CodeGen/MachineConstantPool.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/RegisterScavenging.h"
#include "llvm/CodeGen/TargetFrameLowering.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/LLVMContext.h"
#include "llvm/Support/CommandLine.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 29-30
```cpp
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Target/TargetMachine.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 32-34
```cpp
namespace llvm {
extern cl::opt<bool> ReuseFrameIndexVals;
}
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 36-36
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 38-39
```cpp
ThumbRegisterInfo::ThumbRegisterInfo(const ARMSubtarget &STI)
    : IsThumb1Only(STI.isThumb1Only()) {}
```
- EN: Implements `ThumbRegisterInfo::ThumbRegisterInfo`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ThumbRegisterInfo::ThumbRegisterInfo`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 41-45
```cpp
const TargetRegisterClass *
ThumbRegisterInfo::getLargestLegalSuperClass(const TargetRegisterClass *RC,
                                              const MachineFunction &MF) const {
  if (!IsThumb1Only)
    return ARMBaseRegisterInfo::getLargestLegalSuperClass(RC, MF);
```
- EN: Implements `ThumbRegisterInfo::getLargestLegalSuperClass`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ThumbRegisterInfo::getLargestLegalSuperClass`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 47-50
```cpp
  if (ARM::tGPRRegClass.hasSubClassEq(RC))
    return &ARM::tGPRRegClass;
  return ARMBaseRegisterInfo::getLargestLegalSuperClass(RC, MF);
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 52-57
```cpp
const TargetRegisterClass *
ThumbRegisterInfo::getPointerRegClass(unsigned Kind) const {
  if (!IsThumb1Only)
    return ARMBaseRegisterInfo::getPointerRegClass(Kind);
  return &ARM::tGPRRegClass;
}
```
- EN: Implements `ThumbRegisterInfo::getPointerRegClass`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ThumbRegisterInfo::getPointerRegClass`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 59-71
```cpp
static void emitThumb1LoadConstPool(MachineBasicBlock &MBB,
                                    MachineBasicBlock::iterator &MBBI,
                                    const DebugLoc &dl, unsigned DestReg,
                                    unsigned SubIdx, int Val,
                                    ARMCC::CondCodes Pred, unsigned PredReg,
                                    unsigned MIFlags) {
  MachineFunction &MF = *MBB.getParent();
  const ARMSubtarget &STI = MF.getSubtarget<ARMSubtarget>();
  const TargetInstrInfo &TII = *STI.getInstrInfo();
  MachineConstantPool *ConstantPool = MF.getConstantPool();
  const Constant *C = ConstantInt::getSigned(
      Type::getInt32Ty(MBB.getParent()->getFunction().getContext()), Val);
  unsigned Idx = ConstantPool->getConstantPoolIndex(C, Align(4));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 73-77
```cpp
  BuildMI(MBB, MBBI, dl, TII.get(ARM::tLDRpci))
    .addReg(DestReg, getDefRegState(true), SubIdx)
    .addConstantPoolIndex(Idx).addImm(Pred).addReg(PredReg)
    .setMIFlags(MIFlags);
}
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 79-90
```cpp
static void emitThumb2LoadConstPool(MachineBasicBlock &MBB,
                                    MachineBasicBlock::iterator &MBBI,
                                    const DebugLoc &dl, unsigned DestReg,
                                    unsigned SubIdx, int Val,
                                    ARMCC::CondCodes Pred, unsigned PredReg,
                                    unsigned MIFlags) {
  MachineFunction &MF = *MBB.getParent();
  const TargetInstrInfo &TII = *MF.getSubtarget().getInstrInfo();
  MachineConstantPool *ConstantPool = MF.getConstantPool();
  const Constant *C = ConstantInt::getSigned(
      Type::getInt32Ty(MBB.getParent()->getFunction().getContext()), Val);
  unsigned Idx = ConstantPool->getConstantPoolIndex(C, Align(4));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 92-97
```cpp
  BuildMI(MBB, MBBI, dl, TII.get(ARM::t2LDRpci))
      .addReg(DestReg, getDefRegState(true), SubIdx)
      .addConstantPoolIndex(Idx)
      .add(predOps(ARMCC::AL))
      .setMIFlags(MIFlags);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 99-113
```cpp
/// emitLoadConstPool - Emits a load from constpool to materialize the
/// specified immediate.
void ThumbRegisterInfo::emitLoadConstPool(
    MachineBasicBlock &MBB, MachineBasicBlock::iterator &MBBI,
    const DebugLoc &dl, Register DestReg, unsigned SubIdx, int Val,
    ARMCC::CondCodes Pred, Register PredReg, unsigned MIFlags) const {
  MachineFunction &MF = *MBB.getParent();
  const ARMSubtarget &STI = MF.getSubtarget<ARMSubtarget>();
  if (STI.isThumb1Only()) {
    assert((DestReg.isVirtual() || isARMLowRegister(DestReg)) &&
           "Thumb1 does not have ldr to high register");
    return emitThumb1LoadConstPool(MBB, MBBI, dl, DestReg, SubIdx, Val, Pred,
                                   PredReg, MIFlags);
  }
  return emitThumb2LoadConstPool(MBB, MBBI, dl, DestReg, SubIdx, Val, Pred,
```
- EN: Implements `ThumbRegisterInfo::emitLoadConstPool`, a emission/printing routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ThumbRegisterInfo::emitLoadConstPool`，它是一个围绕机器函数状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 114-115
```cpp
                                 PredReg, MIFlags);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 117-128
```cpp
/// emitThumbRegPlusImmInReg - Emits a series of instructions to materialize a
/// destreg = basereg + immediate in Thumb code. Materialize the immediate in a
/// register using mov / mvn (armv6-M >) sequences, movs / lsls / adds / lsls /
/// adds / lsls / adds sequences (armv6-M) or load the immediate from a
/// constpool entry.
static void emitThumbRegPlusImmInReg(
    MachineBasicBlock &MBB, MachineBasicBlock::iterator &MBBI,
    const DebugLoc &dl, Register DestReg, Register BaseReg, int NumBytes,
    bool CanChangeCC, const TargetInstrInfo &TII,
    const ARMBaseRegisterInfo &MRI, unsigned MIFlags = MachineInstr::NoFlags) {
  MachineFunction &MF = *MBB.getParent();
  const ARMSubtarget &ST = MF.getSubtarget<ARMSubtarget>();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 130-140
```cpp
  // Use a single sp-relative add if the immediate is small enough.
  if (BaseReg == ARM::SP &&
      (DestReg.isVirtual() || isARMLowRegister(DestReg)) && NumBytes >= 0 &&
      NumBytes <= 1020 && (NumBytes % 4) == 0) {
    BuildMI(MBB, MBBI, dl, TII.get(ARM::tADDrSPi), DestReg)
        .addReg(ARM::SP)
        .addImm(NumBytes / 4)
        .add(predOps(ARMCC::AL))
        .setMIFlags(MIFlags);
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 142-156
```cpp
  bool isHigh = DestReg.isVirtual() || !isARMLowRegister(DestReg) ||
                (BaseReg != 0 && !isARMLowRegister(BaseReg));
  bool isSub = false;
  // Subtract doesn't have high register version. Load the negative value
  // if either base or dest register is a high register. Also, if do not
  // issue sub as part of the sequence if condition register is to be
  // preserved.
  if (NumBytes < 0 && !isHigh && CanChangeCC) {
    isSub = true;
    NumBytes = -NumBytes;
  }
  Register LdReg = DestReg;
  if (DestReg == ARM::SP)
    assert(BaseReg == ARM::SP && "Unexpected!");
  if (!DestReg.isVirtual() && !isARMLowRegister(DestReg))
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 157-157
```cpp
    LdReg = MF.getRegInfo().createVirtualRegister(&ARM::tGPRRegClass);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 159-173
```cpp
  if (NumBytes <= 255 && NumBytes >= 0 && CanChangeCC) {
    BuildMI(MBB, MBBI, dl, TII.get(ARM::tMOVi8), LdReg)
        .add(t1CondCodeOp())
        .addImm(NumBytes)
        .setMIFlags(MIFlags);
  } else if (NumBytes < 0 && NumBytes >= -255 && CanChangeCC) {
    BuildMI(MBB, MBBI, dl, TII.get(ARM::tMOVi8), LdReg)
        .add(t1CondCodeOp())
        .addImm(NumBytes)
        .setMIFlags(MIFlags);
    BuildMI(MBB, MBBI, dl, TII.get(ARM::tRSB), LdReg)
        .add(t1CondCodeOp())
        .addReg(LdReg, RegState::Kill)
        .setMIFlags(MIFlags);
  } else if (ST.genExecuteOnly()) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 174-188
```cpp
    if (ST.useMovt()) {
      BuildMI(MBB, MBBI, dl, TII.get(ARM::t2MOVi32imm ), LdReg)
          .addImm(NumBytes)
          .setMIFlags(MIFlags);
    } else if (!CanChangeCC) {
      // tMOVi32imm is lowered to a sequence of flag-setting instructions, so
      // if CPSR is live we need to save and restore CPSR around it.
      // TODO Try inserting the tMOVi32imm at an earlier point, where CPSR is
      // dead.
      bool LiveCpsr = false, CpsrWrite = false;
      auto isCpsr = [](auto &MO) { return MO.getReg() == ARM::CPSR; };
      for (auto Iter = MBBI; Iter != MBB.instr_end(); ++Iter) {
        // If CPSR is used after this instruction (and there's not a def before
        // that) then CPSR is live.
        if (any_of(Iter->all_uses(), isCpsr)) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 189-201
```cpp
          LiveCpsr = true;
          break;
        }
        if (any_of(Iter->all_defs(), isCpsr)) {
          CpsrWrite = true;
          break;
        }
      }
      // If there's no use or def of CPSR then it may be live if it's a
      // live-out value.
      auto liveOutIsCpsr = [](auto &Out) { return Out.PhysReg == ARM::CPSR; };
      if (!LiveCpsr && !CpsrWrite)
        LiveCpsr = any_of(MBB.liveouts(), liveOutIsCpsr);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 203-217
```cpp
      Register CPSRSaveReg;
      unsigned APSREncoding;
      if (LiveCpsr) {
        CPSRSaveReg = MF.getRegInfo().createVirtualRegister(&ARM::tGPRRegClass);
        APSREncoding =
            ARMSysReg::lookupMClassSysRegByName("apsr_nzcvq")->Encoding;
        BuildMI(MBB, MBBI, dl, TII.get(ARM::t2MRS_M), CPSRSaveReg)
            .addImm(APSREncoding)
            .add(predOps(ARMCC::AL))
            .addReg(ARM::CPSR, RegState::Implicit);
      }
      BuildMI(MBB, MBBI, dl, TII.get(ARM::tMOVi32imm), LdReg)
          .addImm(NumBytes)
          .setMIFlags(MIFlags);
      if (LiveCpsr) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 218-230
```cpp
        BuildMI(MBB, MBBI, dl, TII.get(ARM::t2MSR_M))
            .addImm(APSREncoding)
            .addReg(CPSRSaveReg, RegState::Kill)
            .add(predOps(ARMCC::AL));
      }
    } else {
      BuildMI(MBB, MBBI, dl, TII.get(ARM::tMOVi32imm), LdReg)
          .addImm(NumBytes)
          .setMIFlags(MIFlags);
    }
  } else
    MRI.emitLoadConstPool(MBB, MBBI, dl, LdReg, 0, NumBytes, ARMCC::AL, 0,
                          MIFlags);
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 232-243
```cpp
  // Emit add / sub.
  int Opc = (isSub) ? ARM::tSUBrr
                    : ((isHigh || !CanChangeCC) ? ARM::tADDhirr : ARM::tADDrr);
  MachineInstrBuilder MIB = BuildMI(MBB, MBBI, dl, TII.get(Opc), DestReg);
  if (Opc != ARM::tADDhirr)
    MIB = MIB.add(t1CondCodeOp());
  if (DestReg == ARM::SP || isSub)
    MIB.addReg(BaseReg).addReg(LdReg, RegState::Kill);
  else
    MIB.addReg(LdReg).addReg(BaseReg, RegState::Kill);
  MIB.add(predOps(ARMCC::AL));
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 245-258
```cpp
/// emitThumbRegPlusImmediate - Emits a series of instructions to materialize
/// a destreg = basereg + immediate in Thumb code. Tries a series of ADDs or
/// SUBs first, and uses a constant pool value if the instruction sequence would
/// be too long. This is allowed to modify the condition flags.
void llvm::emitThumbRegPlusImmediate(MachineBasicBlock &MBB,
                                     MachineBasicBlock::iterator &MBBI,
                                     const DebugLoc &dl, Register DestReg,
                                     Register BaseReg, int NumBytes,
                                     const TargetInstrInfo &TII,
                                     const ARMBaseRegisterInfo &MRI,
                                     unsigned MIFlags) {
  bool isSub = NumBytes < 0;
  unsigned Bytes = (unsigned)NumBytes;
  if (isSub) Bytes = -NumBytes;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 260-267
```cpp
  int CopyOpc = 0;
  unsigned CopyBits = 0;
  unsigned CopyScale = 1;
  bool CopyNeedsCC = false;
  int ExtraOpc = 0;
  unsigned ExtraBits = 0;
  unsigned ExtraScale = 1;
  bool ExtraNeedsCC = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 269-283
```cpp
  // Strategy:
  // We need to select two types of instruction, maximizing the available
  // immediate range of each. The instructions we use will depend on whether
  // DestReg and BaseReg are low, high or the stack pointer.
  // * CopyOpc  - DestReg = BaseReg + imm
  //              This will be emitted once if DestReg != BaseReg, and never if
  //              DestReg == BaseReg.
  // * ExtraOpc - DestReg = DestReg + imm
  //              This will be emitted as many times as necessary to add the
  //              full immediate.
  // If the immediate ranges of these instructions are not large enough to cover
  // NumBytes with a reasonable number of instructions, we fall back to using a
  // value loaded from a constant pool.
  if (DestReg == ARM::SP) {
    if (BaseReg == ARM::SP) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 284-298
```cpp
      // sp -> sp
      // Already in right reg, no copy needed
    } else {
      // low -> sp or high -> sp
      CopyOpc = ARM::tMOVr;
      CopyBits = 0;
    }
    ExtraOpc = isSub ? ARM::tSUBspi : ARM::tADDspi;
    ExtraBits = 7;
    ExtraScale = 4;
  } else if (isARMLowRegister(DestReg)) {
    if (BaseReg == ARM::SP) {
      // sp -> low
      assert(!isSub && "Thumb1 does not have tSUBrSPi");
      CopyOpc = ARM::tADDrSPi;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 299-313
```cpp
      CopyBits = 8;
      CopyScale = 4;
    } else if (DestReg == BaseReg) {
      // low -> same low
      // Already in right reg, no copy needed
    } else if (isARMLowRegister(BaseReg)) {
      // low -> different low
      CopyOpc = isSub ? ARM::tSUBi3 : ARM::tADDi3;
      CopyBits = 3;
      CopyNeedsCC = true;
    } else {
      // high -> low
      CopyOpc = ARM::tMOVr;
      CopyBits = 0;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 314-327
```cpp
    ExtraOpc = isSub ? ARM::tSUBi8 : ARM::tADDi8;
    ExtraBits = 8;
    ExtraNeedsCC = true;
  } else /* DestReg is high */ {
    if (DestReg == BaseReg) {
      // high -> same high
      // Already in right reg, no copy needed
    } else {
      // {low,high,sp} -> high
      CopyOpc = ARM::tMOVr;
      CopyBits = 0;
    }
    ExtraOpc = 0;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 329-332
```cpp
  // We could handle an unaligned immediate with an unaligned copy instruction
  // and an aligned extra instruction, but this case is not currently needed.
  assert(((Bytes & 3) == 0 || ExtraScale == 1) &&
         "Unaligned offset, but all instructions require alignment");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 334-344
```cpp
  unsigned CopyRange = ((1 << CopyBits) - 1) * CopyScale;
  // If we would emit the copy with an immediate of 0, just use tMOVr.
  if (CopyOpc && Bytes < CopyScale) {
    CopyOpc = ARM::tMOVr;
    CopyScale = 1;
    CopyNeedsCC = false;
    CopyRange = 0;
  }
  unsigned ExtraRange = ((1 << ExtraBits) - 1) * ExtraScale; // per instruction
  unsigned RequiredCopyInstrs = CopyOpc ? 1 : 0;
  unsigned RangeAfterCopy = (CopyRange > Bytes) ? 0 : (Bytes - CopyRange);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 346-349
```cpp
  // We could handle this case when the copy instruction does not require an
  // aligned immediate, but we do not currently do this.
  assert(RangeAfterCopy % ExtraScale == 0 &&
         "Extra instruction requires immediate to be aligned");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 351-360
```cpp
  unsigned RequiredExtraInstrs;
  if (ExtraRange)
    RequiredExtraInstrs = alignTo(RangeAfterCopy, ExtraRange) / ExtraRange;
  else if (RangeAfterCopy > 0)
    // We need an extra instruction but none is available
    RequiredExtraInstrs = 1000000;
  else
    RequiredExtraInstrs = 0;
  unsigned RequiredInstrs = RequiredCopyInstrs + RequiredExtraInstrs;
  unsigned Threshold = (DestReg == ARM::SP) ? 3 : 2;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 362-368
```cpp
  // Use a constant pool, if the sequence of ADDs/SUBs is too expensive.
  if (RequiredInstrs > Threshold) {
    emitThumbRegPlusImmInReg(MBB, MBBI, dl,
                             DestReg, BaseReg, NumBytes, true,
                             TII, MRI, MIFlags);
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 370-373
```cpp
  // Emit zero or one copy instructions
  if (CopyOpc) {
    unsigned CopyImm = std::min(Bytes, CopyRange) / CopyScale;
    Bytes -= CopyImm * CopyScale;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 375-382
```cpp
    MachineInstrBuilder MIB = BuildMI(MBB, MBBI, dl, TII.get(CopyOpc), DestReg);
    if (CopyNeedsCC)
      MIB = MIB.add(t1CondCodeOp());
    MIB.addReg(BaseReg, RegState::Kill);
    if (CopyOpc != ARM::tMOVr) {
      MIB.addImm(CopyImm);
    }
    MIB.setMIFlags(MIFlags).add(predOps(ARMCC::AL));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 384-385
```cpp
    BaseReg = DestReg;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 387-390
```cpp
  // Emit zero or more in-place add/sub instructions
  while (Bytes) {
    unsigned ExtraImm = std::min(Bytes, ExtraRange) / ExtraScale;
    Bytes -= ExtraImm * ExtraScale;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 392-400
```cpp
    MachineInstrBuilder MIB = BuildMI(MBB, MBBI, dl, TII.get(ExtraOpc), DestReg);
    if (ExtraNeedsCC)
      MIB = MIB.add(t1CondCodeOp());
    MIB.addReg(BaseReg)
       .addImm(ExtraImm)
       .add(predOps(ARMCC::AL))
       .setMIFlags(MIFlags);
  }
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 402-406
```cpp
static void removeOperands(MachineInstr &MI, unsigned i) {
  unsigned Op = i;
  for (unsigned e = MI.getNumOperands(); i != e; ++i)
    MI.removeOperand(Op);
}
```
- EN: Implements `removeOperands`, a mutation/build routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `removeOperands`，它是一个围绕机器指令展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 408-413
```cpp
/// convertToNonSPOpcode - Change the opcode to the non-SP version, because
/// we're replacing the frame index with a non-SP register.
static unsigned convertToNonSPOpcode(unsigned Opcode) {
  switch (Opcode) {
  case ARM::tLDRspi:
    return ARM::tLDRi;
```
- EN: Implements `convertToNonSPOpcode`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `convertToNonSPOpcode`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 415-417
```cpp
  case ARM::tSTRspi:
    return ARM::tSTRi;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 419-420
```cpp
  return Opcode;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 422-435
```cpp
bool ThumbRegisterInfo::rewriteFrameIndex(MachineBasicBlock::iterator II,
                                          unsigned FrameRegIdx,
                                          Register FrameReg, int &Offset,
                                          const ARMBaseInstrInfo &TII) const {
  MachineInstr &MI = *II;
  MachineBasicBlock &MBB = *MI.getParent();
  MachineFunction &MF = *MBB.getParent();
  assert(MBB.getParent()->getSubtarget<ARMSubtarget>().isThumb1Only() &&
         "This isn't needed for thumb2!");
  DebugLoc dl = MI.getDebugLoc();
  MachineInstrBuilder MIB(*MBB.getParent(), &MI);
  unsigned Opcode = MI.getOpcode();
  const MCInstrDesc &Desc = MI.getDesc();
  unsigned AddrMode = (Desc.TSFlags & ARMII::AddrModeMask);
```
- EN: Implements `ThumbRegisterInfo::rewriteFrameIndex`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ThumbRegisterInfo::rewriteFrameIndex`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 437-439
```cpp
  if (Opcode == ARM::tADDframe) {
    Offset += MI.getOperand(FrameRegIdx+1).getImm();
    Register DestReg = MI.getOperand(0).getReg();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 441-447
```cpp
    emitThumbRegPlusImmediate(MBB, II, dl, DestReg, FrameReg, Offset, TII,
                              *this);
    MBB.erase(II);
    return true;
  } else {
    if (AddrMode != ARMII::AddrModeT1_s)
      llvm_unreachable("Unsupported addressing mode!");
```
- EN: Declares `emitThumbRegPlusImmediate`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitThumbRegPlusImmediate`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 449-452
```cpp
    unsigned ImmIdx = FrameRegIdx + 1;
    int InstrOffs = MI.getOperand(ImmIdx).getImm();
    unsigned NumBits = (FrameReg == ARM::SP) ? 8 : 5;
    unsigned Scale = 4;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 454-455
```cpp
    Offset += InstrOffs * Scale;
    assert((Offset & (Scale - 1)) == 0 && "Can't encode this offset!");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 457-460
```cpp
    // Common case: small offset, fits into instruction.
    MachineOperand &ImmOp = MI.getOperand(ImmIdx);
    int ImmedOffset = Offset / Scale;
    unsigned Mask = (1 << NumBits) - 1;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 462-464
```cpp
    if ((unsigned)Offset <= Mask * Scale) {
      // Replace the FrameIndex with the frame register (e.g., sp).
      Register DestReg = FrameReg;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 466-473
```cpp
      // In case FrameReg is a high register, move it to a low reg to ensure it
      // can be used as an operand.
      if (ARM::hGPRRegClass.contains(FrameReg) && FrameReg != ARM::SP) {
        DestReg = MF.getRegInfo().createVirtualRegister(&ARM::tGPRRegClass);
        BuildMI(MBB, II, dl, TII.get(ARM::tMOVr), DestReg)
            .addReg(FrameReg)
            .add(predOps(ARMCC::AL));
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 475-476
```cpp
      MI.getOperand(FrameRegIdx).ChangeToRegister(DestReg, false);
      ImmOp.ChangeToImmediate(ImmedOffset);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 478-482
```cpp
      // If we're using a register where sp was stored, convert the instruction
      // to the non-SP version.
      unsigned NewOpc = convertToNonSPOpcode(Opcode);
      if (NewOpc != Opcode && FrameReg != ARM::SP)
        MI.setDesc(TII.get(NewOpc));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 484-485
```cpp
      return true;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 487-501
```cpp
    // The offset doesn't fit, but we may be able to put some of the offset into
    // the ldr to simplify the generation of the rest of it.
    NumBits = 5;
    Mask = (1 << NumBits) - 1;
    InstrOffs = 0;
    auto &ST = MF.getSubtarget<ARMSubtarget>();
    // If using the maximum ldr offset will put the rest into the range of a
    // single sp-relative add then do so.
    if (FrameReg == ARM::SP && Offset - (Mask * Scale) <= 1020) {
      InstrOffs = Mask;
    } else if (ST.genExecuteOnly()) {
      // With execute-only the offset is generated either with movw+movt or an
      // add+lsl sequence. If subtracting an offset will make the top half zero
      // then that saves a movt or lsl+add. Otherwise if we don't have movw then
      // we may be able to subtract a value such that it makes the bottom byte
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 502-514
```cpp
      // zero, saving an add.
      unsigned BottomBits = (Offset / Scale) & Mask;
      bool CanMakeBottomByteZero = ((Offset - BottomBits * Scale) & 0xff) == 0;
      bool TopHalfZero = (Offset & 0xffff0000) == 0;
      bool CanMakeTopHalfZero = ((Offset - Mask * Scale) & 0xffff0000) == 0;
      if (!TopHalfZero && CanMakeTopHalfZero)
        InstrOffs = Mask;
      else if (!ST.useMovt() && CanMakeBottomByteZero)
        InstrOffs = BottomBits;
    }
    ImmOp.ChangeToImmediate(InstrOffs);
    Offset -= InstrOffs * Scale;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 516-517
```cpp
  return Offset == 0;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 519-524
```cpp
void ThumbRegisterInfo::resolveFrameIndex(MachineInstr &MI, Register BaseReg,
                                          int64_t Offset) const {
  const MachineFunction &MF = *MI.getParent()->getParent();
  const ARMSubtarget &STI = MF.getSubtarget<ARMSubtarget>();
  if (!STI.isThumb1Only())
    return ARMBaseRegisterInfo::resolveFrameIndex(MI, BaseReg, Offset);
```
- EN: Implements `ThumbRegisterInfo::resolveFrameIndex`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ThumbRegisterInfo::resolveFrameIndex`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 526-528
```cpp
  const ARMBaseInstrInfo &TII = *STI.getInstrInfo();
  int Off = Offset; // ARM doesn't need the general 64-bit offsets
  unsigned i = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 530-537
```cpp
  while (!MI.getOperand(i).isFI()) {
    ++i;
    assert(i < MI.getNumOperands() && "Instr doesn't have FrameIndex operand!");
  }
  bool Done = rewriteFrameIndex(MI, i, BaseReg, Off, TII);
  assert (Done && "Unable to resolve frame index!");
  (void)Done;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 539-548
```cpp
bool ThumbRegisterInfo::eliminateFrameIndex(MachineBasicBlock::iterator II,
                                            int SPAdj, unsigned FIOperandNum,
                                            RegScavenger *RS) const {
  MachineInstr &MI = *II;
  MachineBasicBlock &MBB = *MI.getParent();
  MachineFunction &MF = *MBB.getParent();
  const ARMSubtarget &STI = MF.getSubtarget<ARMSubtarget>();
  if (!STI.isThumb1Only())
    return ARMBaseRegisterInfo::eliminateFrameIndex(II, SPAdj, FIOperandNum,
                                                    RS);
```
- EN: Implements `ThumbRegisterInfo::eliminateFrameIndex`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ThumbRegisterInfo::eliminateFrameIndex`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 550-553
```cpp
  Register VReg;
  const ARMBaseInstrInfo &TII = *STI.getInstrInfo();
  DebugLoc dl = MI.getDebugLoc();
  MachineInstrBuilder MIB(*MBB.getParent(), &MI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 555-558
```cpp
  Register FrameReg;
  int FrameIndex = MI.getOperand(FIOperandNum).getIndex();
  const ARMFrameLowering *TFI = getFrameLowering(MF);
  int Offset = TFI->ResolveFrameIndexReference(MF, FrameIndex, FrameReg, SPAdj);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 560-573
```cpp
  // PEI::scavengeFrameVirtualRegs() cannot accurately track SPAdj because the
  // call frame setup/destroy instructions have already been eliminated.  That
  // means the stack pointer cannot be used to access the emergency spill slot
  // when !hasReservedCallFrame().
#ifndef NDEBUG
  if (RS && FrameReg == ARM::SP && RS->isScavengingFrameIndex(FrameIndex)){
    assert(STI.getFrameLowering()->hasReservedCallFrame(MF) &&
           "Cannot use SP to access the emergency spill slot in "
           "functions without a reserved call frame");
    assert(!MF.getFrameInfo().hasVarSizedObjects() &&
           "Cannot use SP to access the emergency spill slot in "
           "functions with variable sized frame objects");
  }
#endif // NDEBUG
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 575-580
```cpp
  // Special handling of dbg_value instructions.
  if (MI.isDebugValue()) {
    MI.getOperand(FIOperandNum).  ChangeToRegister(FrameReg, false /*isDef*/);
    MI.getOperand(FIOperandNum+1).ChangeToImmediate(Offset);
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 582-586
```cpp
  // Modify MI as necessary to handle as much of 'Offset' as possible
  assert(MF.getInfo<ARMFunctionInfo>()->isThumbFunction() &&
         "This eliminateFrameIndex only supports Thumb1!");
  if (rewriteFrameIndex(MI, FIOperandNum, FrameReg, Offset, TII))
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 588-591
```cpp
  // If we get here, the immediate doesn't fit into the instruction.  We folded
  // as much as possible above, handle the rest, providing a register that is
  // SP+LargeImm.
  assert(Offset && "This code isn't needed if offset already handled!");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 593-593
```cpp
  unsigned Opcode = MI.getOpcode();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 595-598
```cpp
  // Remove predicate first.
  int PIdx = MI.findFirstPredOperandIdx();
  if (PIdx != -1)
    removeOperands(MI, PIdx);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 600-614
```cpp
  if (MI.mayLoad()) {
    // Use the destination register to materialize sp + offset.
    Register TmpReg = MI.getOperand(0).getReg();
    bool UseRR = false;
    if (Opcode == ARM::tLDRspi) {
      if (FrameReg == ARM::SP || STI.genExecuteOnly())
        emitThumbRegPlusImmInReg(MBB, II, dl, TmpReg, FrameReg,
                                 Offset, false, TII, *this);
      else {
        emitLoadConstPool(MBB, II, dl, TmpReg, 0, Offset);
        if (!ARM::hGPRRegClass.contains(FrameReg)) {
          UseRR = true;
        } else {
          // If FrameReg is a high register, add the reg values in a separate
          // instruction as the load won't be able to access it.
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 615-624
```cpp
          BuildMI(MBB, II, dl, TII.get(ARM::tADDhirr), TmpReg)
              .addReg(TmpReg)
              .addReg(FrameReg)
              .add(predOps(ARMCC::AL));
        }
      }
    } else {
      emitThumbRegPlusImmediate(MBB, II, dl, TmpReg, FrameReg, Offset, TII,
                                *this);
    }
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 626-638
```cpp
    MI.setDesc(TII.get(UseRR ? ARM::tLDRr : ARM::tLDRi));
    MI.getOperand(FIOperandNum).ChangeToRegister(TmpReg, false, false, true);
    if (UseRR) {
      assert(!ARM::hGPRRegClass.contains(FrameReg) &&
             "Thumb1 loads can't use high register");
      // Use [reg, reg] addrmode. Replace the immediate operand w/ the frame
      // register. The offset is already handled in the vreg value.
      MI.getOperand(FIOperandNum+1).ChangeToRegister(FrameReg, false, false,
                                                     false);
    }
  } else if (MI.mayStore()) {
      VReg = MF.getRegInfo().createVirtualRegister(&ARM::tGPRRegClass);
      bool UseRR = false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 640-654
```cpp
      if (Opcode == ARM::tSTRspi) {
        if (FrameReg == ARM::SP || STI.genExecuteOnly())
          emitThumbRegPlusImmInReg(MBB, II, dl, VReg, FrameReg,
                                   Offset, false, TII, *this);
        else {
          emitLoadConstPool(MBB, II, dl, VReg, 0, Offset);
          if (!ARM::hGPRRegClass.contains(FrameReg)) {
            UseRR = true;
          } else {
            // If FrameReg is a high register, add the reg values in a separate
            // instruction as the load won't be able to access it.
            BuildMI(MBB, II, dl, TII.get(ARM::tADDhirr), VReg)
                .addReg(VReg)
                .addReg(FrameReg)
                .add(predOps(ARMCC::AL));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 655-669
```cpp
          }
        }
      } else
        emitThumbRegPlusImmediate(MBB, II, dl, VReg, FrameReg, Offset, TII,
                                  *this);
      MI.setDesc(TII.get(UseRR ? ARM::tSTRr : ARM::tSTRi));
      MI.getOperand(FIOperandNum).ChangeToRegister(VReg, false, false, true);
      if (UseRR) {
        assert(!ARM::hGPRRegClass.contains(FrameReg) &&
               "Thumb1 stores can't use high register");
        // Use [reg, reg] addrmode. Replace the immediate operand w/ the frame
        // register. The offset is already handled in the vreg value.
        MI.getOperand(FIOperandNum+1).ChangeToRegister(FrameReg, false, false,
                                                       false);
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 670-672
```cpp
  } else {
    llvm_unreachable("Unexpected opcode!");
  }
```
- EN: Implements `llvm_unreachable`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm_unreachable`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 674-678
```cpp
  // Add predicate back if it's needed.
  if (MI.isPredicable())
    MIB.add(predOps(ARMCC::AL));
  return false;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 680-689
```cpp
bool
ThumbRegisterInfo::useFPForScavengingIndex(const MachineFunction &MF) const {
  if (MF.getSubtarget<ARMSubtarget>().isThumb1Only()) {
    // For Thumb1, the emergency spill slot must be some small positive
    // offset from the base/stack pointer.
    return false;
  }
  // For Thumb2, put the emergency spill slot next to FP.
  return true;
}
```
- EN: Implements `ThumbRegisterInfo::useFPForScavengingIndex`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ThumbRegisterInfo::useFPForScavengingIndex`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

## Key Concepts / 关键概念

- EN: Primary role: register definitions, allocation constraints, and register utilities.
  - CN: 核心职责：寄存器定义、分配约束以及寄存器工具。
- EN: MachineInstr-level manipulation is central to this file.
  - CN: 该文件以 MachineInstr 层面的操作为核心。
- EN: MachineBasicBlock flow and branch structure matter to the implementation.
  - CN: 实现高度依赖 MachineBasicBlock 流程与分支结构。
- EN: The MC layer is used to print, encode, or stream target instructions and symbols.
  - CN: MC 层被用于打印、编码或流式输出目标指令与符号。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。

## Dependencies / 依赖关系

- EN: Backend-local headers: `ThumbRegisterInfo.h`, `ARMBaseInstrInfo.h`, `ARMMachineFunctionInfo.h`, `ARMSubtarget.h`.
  - CN: 后端本地头文件：`ThumbRegisterInfo.h`, `ARMBaseInstrInfo.h`, `ARMMachineFunctionInfo.h`, `ARMSubtarget.h`。
- EN: LLVM infrastructure headers: `llvm/CodeGen/MachineConstantPool.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineRegisterInfo.h`, `llvm/CodeGen/RegisterScavenging.h`, `llvm/CodeGen/TargetFrameLowering.h`, `llvm/IR/Constants.h` ... (+5 more).
  - CN: LLVM 基础设施头文件：`llvm/CodeGen/MachineConstantPool.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineRegisterInfo.h`, `llvm/CodeGen/RegisterScavenging.h`, `llvm/CodeGen/TargetFrameLowering.h`, `llvm/IR/Constants.h` ... (+5 more)。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
- EN: The implementation also depends on LLVM MC layer classes for final assembly/object emission.
  - CN: 实现还依赖 LLVM 的 MC 层类来完成最终汇编/目标文件输出。
